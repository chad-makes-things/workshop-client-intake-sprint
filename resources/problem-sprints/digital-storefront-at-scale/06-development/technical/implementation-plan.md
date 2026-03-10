# Implementation Plan: The Practice Concierge

**Role**: Tech Lead
**Sprint**: Digital Storefront at Scale
**Date**: 2026-03-10
**Source Artifacts**: `04-architect/product-architecture.md`, `01-frame/problem-map.md`, `05-validate/learnings.md`

---

## 1. Stack Validation

### PostgreSQL — Practice Data Hub

**Decision**: PostgreSQL 16 as the primary data store.

**Justification**: Practice data is inherently relational. Each practice has structured relationships to team members, services, enrollment records, conversation logs, and health scores. PostgreSQL handles JSONB columns for semi-structured fields (brand assets, digital presence metadata) without sacrificing query performance. At 260+ practices with ~50 related records each, we are well within PostgreSQL's sweet spot. Full-text search for practice lookup, row-level security for multi-tenant access, and mature tooling (Prisma ORM, pg_dump for backups) make this a low-risk choice.

**Alternatives considered**: MongoDB (document model fits practice profiles but loses relational integrity for cross-practice queries and dashboard aggregations), Supabase (adds hosted Postgres + auth but introduces vendor coupling we do not need yet).

### Redis — Cache & Session Layer

**Decision**: Redis 7 for hot-path caching, session storage, and BullMQ job backing store.

**Justification**: The Concierge Agent must load practice context on every interaction. A cold PostgreSQL query for a full practice profile (team, services, brand, digital presence, health scores) takes 15-40ms. Redis brings this to <2ms. At 260+ practices with concurrent interactions, cache hit rates will exceed 95% since practice data changes infrequently (days/weeks between updates). Redis also serves as the backing store for BullMQ job queues (Channel Sync, Harvester tasks) and session storage for the HQ dashboard.

**Cache invalidation strategy**: Write-through — every Practice Data Hub write updates Redis immediately. TTL of 1 hour as safety net. Cache key pattern: `practice:{id}:profile`, `practice:{id}:health`.

### Claude API — Concierge Agent & Data Harvester

**Decision**: Claude API (claude-sonnet-4-20250514 for conversations, claude-sonnet-4-20250514 for harvester extraction).

**Justification**: The Concierge Agent requires nuanced conversation, opinionated recommendations with reasoning, and tool use (triggering backend actions from conversation context). Claude's tool_use capability maps directly to the Action Router pattern in the architecture. For the Data Harvester, Claude parses unstructured website HTML into structured practice data — dental practice websites have no standard HTML structure, making rule-based scrapers brittle. Claude handles arbitrary page layouts reliably.

**Cost model**: Estimated ~$0.15-0.30 per enrollment conversation (10-15 turns, ~4K tokens/turn). At 260 practices: ~$50-75 for full enrollment. Ongoing interactions (change management, Q&A): ~$500-1000/month at scale. Well within budget.

**Rate limiting**: Claude API rate limits (requests per minute) require a queue-based conversation handler. BullMQ manages concurrent requests; no more than 20 concurrent API calls.

### Next.js — Template Engine, Dashboard, Agent Interface

**Decision**: Next.js 14 (App Router) for three surfaces: generated practice sites, HQ dashboard, and concierge chat interface.

**Justification**: Next.js serves all three interface needs with one framework. Practice sites use Static Site Generation (SSG) — pages are pre-rendered at build time from practice data, resulting in fast load times and zero runtime cost. The HQ dashboard uses Server Components for real-time data rendering. The concierge chat interface uses client-side React with streaming responses. Tailwind CSS provides the design system. The team likely has React/Next.js experience, reducing ramp-up time.

**Template architecture**: Each specialty template (GP, Perio, Full Arch, Pediatric, Multi) is a Next.js layout with configurable slots. Practice data populates slots at build time. Template updates trigger re-generation for all practices using that template via a build pipeline.

### BullMQ — Event-Driven Job Processing

**Decision**: BullMQ (Node.js) backed by Redis for the Channel Sync Engine, Harvester tasks, and site generation jobs.

**Justification**: The architecture requires event-driven processing (ADR-3): when practice data changes, updates must propagate to websites, GMB, social channels within minutes, not on a nightly batch. BullMQ provides reliable job queues with retry logic, exponential backoff (critical for API rate limits on GMB and Meta), job prioritization (enrollment jobs > routine sync), and a built-in dashboard (Bull Board) for monitoring queue health. BullMQ runs on the same Redis instance as the cache layer, simplifying infrastructure.

**Queue topology**:
- `harvest` — Data harvester jobs (web scraping, GMB API pulls)
- `generate` — Site generation and deployment
- `sync:gmb` — Google My Business sync
- `sync:social` — Facebook/Instagram sync
- `sync:directory` — Healthgrades, Zocdoc, Yelp sync
- `notify` — Practice and HQ notifications

### Cloudflare Pages — Practice Site Hosting

**Decision**: Cloudflare Pages for hosting generated practice websites.

**Justification**: 260+ static sites need edge-hosted delivery with automatic SSL, custom domain support, and near-zero operational overhead. Cloudflare Pages provides: free SSL for all custom domains, global CDN (sub-100ms load times worldwide), 500 deploys/month on Pro plan (sufficient for daily re-generation of changed sites), preview deployments for QC review before go-live, and no per-site hosting cost. The alternative (Vercel) charges per-project at scale, making it cost-prohibitive for 260+ individual site projects. Cloudflare Pages treats each practice site as a deployment within a single project using path-based routing or individual projects on the Enterprise plan.

**Application hosting**: The dashboard, API, and concierge interface run on Vercel (single application, standard pricing). Data infrastructure (PostgreSQL, Redis, worker processes) runs on Railway or Render for managed hosting with simple scaling.

---

## 2. Database Schema

### Core Tables

```sql
-- Enum types
CREATE TYPE platform_enum AS ENUM ('SGA', 'Gen4', 'MODIS');
CREATE TYPE specialty_enum AS ENUM ('GP', 'Perio', 'FullArch', 'Pediatric', 'Multi');
CREATE TYPE enrollment_status AS ENUM ('pending', 'invited', 'in_progress', 'enrolled', 'blitz_deployed', 'inactive');
CREATE TYPE enrollment_method AS ENUM ('concierge', 'blitz');
CREATE TYPE channel_status AS ENUM ('active', 'pending', 'disconnected', 'not_setup');
CREATE TYPE team_role AS ENUM ('dentist', 'hygienist', 'assistant', 'office_manager', 'specialist', 'admin', 'other');
CREATE TYPE confidence_level AS ENUM ('high', 'medium', 'low');
CREATE TYPE data_source AS ENUM ('harvested', 'concierge', 'manual', 'isolved', 'api_sync');
CREATE TYPE conversation_channel AS ENUM ('web', 'sms', 'email');
CREATE TYPE message_role AS ENUM ('user', 'assistant', 'system', 'tool');

-- Practice (core entity)
CREATE TABLE practice (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug            VARCHAR(100) UNIQUE NOT NULL,
  name            VARCHAR(255) NOT NULL,
  legal_name      VARCHAR(255),
  platform        platform_enum NOT NULL DEFAULT 'SGA',
  specialty       specialty_enum NOT NULL DEFAULT 'GP',

  -- Location
  address_line1   VARCHAR(255),
  address_line2   VARCHAR(255),
  city            VARCHAR(100),
  state           VARCHAR(2),
  zip             VARCHAR(10),
  phone           VARCHAR(20),
  email           VARCHAR(255),
  hours           JSONB,           -- { "mon": "8:00-17:00", "tue": "8:00-17:00", ... }
  timezone        VARCHAR(50) DEFAULT 'America/Chicago',

  -- Brand
  brand           JSONB NOT NULL DEFAULT '{}',
  -- Structure: { primary_color, secondary_color, logo_url, photos: [] }

  -- Digital Presence
  website_url         VARCHAR(500),
  website_template_id VARCHAR(50),
  website_status      channel_status DEFAULT 'not_setup',
  website_last_deployed TIMESTAMPTZ,
  gmb_listing_id      VARCHAR(100),
  gmb_status          channel_status DEFAULT 'not_setup',
  gmb_completeness    INTEGER DEFAULT 0,     -- 0-100
  social_instagram    VARCHAR(100),
  social_facebook_url VARCHAR(500),
  social_status       channel_status DEFAULT 'not_setup',
  crm_system          VARCHAR(50),
  crm_status          channel_status DEFAULT 'not_setup',

  -- Enrollment
  enrollment_status   enrollment_status DEFAULT 'pending',
  enrollment_invited  TIMESTAMPTZ,
  enrollment_enrolled TIMESTAMPTZ,
  enrollment_method   enrollment_method,

  -- Health Scores (0-100 each)
  health_overall      INTEGER DEFAULT 0,
  health_website      INTEGER DEFAULT 0,
  health_gmb          INTEGER DEFAULT 0,
  health_social       INTEGER DEFAULT 0,
  health_reviews      INTEGER DEFAULT 0,
  health_leads        INTEGER DEFAULT 0,
  health_updated_at   TIMESTAMPTZ,

  -- Concierge Interaction Data
  concierge_last_interaction TIMESTAMPTZ,
  concierge_interaction_count INTEGER DEFAULT 0,
  concierge_satisfaction     DECIMAL(3,2),  -- 0.00 - 5.00

  -- Metadata
  data_source     data_source DEFAULT 'manual',
  data_confidence confidence_level DEFAULT 'low',
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_practice_slug ON practice(slug);
CREATE INDEX idx_practice_platform ON practice(platform);
CREATE INDEX idx_practice_specialty ON practice(specialty);
CREATE INDEX idx_practice_enrollment ON practice(enrollment_status);
CREATE INDEX idx_practice_health ON practice(health_overall);

-- Team Members
CREATE TABLE team_member (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  name        VARCHAR(255) NOT NULL,
  role        team_role NOT NULL,
  title       VARCHAR(255),
  bio         TEXT,
  photo_url   VARCHAR(500),
  start_date  DATE,
  end_date    DATE,           -- NULL = currently active
  is_active   BOOLEAN DEFAULT TRUE,
  sort_order  INTEGER DEFAULT 0,
  source      data_source DEFAULT 'manual',
  confidence  confidence_level DEFAULT 'medium',
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_team_practice ON team_member(practice_id);
CREATE INDEX idx_team_active ON team_member(practice_id, is_active);

-- Services
CREATE TABLE service (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  name        VARCHAR(255) NOT NULL,
  description TEXT,
  is_primary  BOOLEAN DEFAULT FALSE,
  sort_order  INTEGER DEFAULT 0,
  source      data_source DEFAULT 'manual',
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_service_practice ON service(practice_id);

-- Templates
CREATE TABLE template (
  id          VARCHAR(50) PRIMARY KEY,   -- e.g., 'gp-v1', 'perio-v1'
  name        VARCHAR(255) NOT NULL,
  specialty   specialty_enum NOT NULL,
  description TEXT,
  version     INTEGER DEFAULT 1,
  is_active   BOOLEAN DEFAULT TRUE,
  config      JSONB DEFAULT '{}',        -- Template-specific configuration
  preview_url VARCHAR(500),
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Enrollment Tracking
CREATE TABLE enrollment_event (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  event_type  VARCHAR(50) NOT NULL,      -- 'invited', 'opened', 'verified_data', 'selected_template', 'approved', 'deployed', 'blitz_triggered'
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_enrollment_practice ON enrollment_event(practice_id);
CREATE INDEX idx_enrollment_type ON enrollment_event(event_type);

-- Conversation Logs
CREATE TABLE conversation (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  channel     conversation_channel DEFAULT 'web',
  started_at  TIMESTAMPTZ DEFAULT NOW(),
  ended_at    TIMESTAMPTZ,
  summary     TEXT,
  intent      VARCHAR(100),              -- 'enrollment', 'change_report', 'question', 'content_approval'
  resolved    BOOLEAN DEFAULT FALSE,
  metadata    JSONB DEFAULT '{}'
);

CREATE INDEX idx_conversation_practice ON conversation(practice_id);

CREATE TABLE conversation_message (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES conversation(id) ON DELETE CASCADE,
  role            message_role NOT NULL,
  content         TEXT NOT NULL,
  tool_calls      JSONB,                 -- Claude tool_use calls/results
  tokens_input    INTEGER,
  tokens_output   INTEGER,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_message_conversation ON conversation_message(conversation_id);

-- Health Score History (for trend tracking)
CREATE TABLE health_score_snapshot (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  overall     INTEGER NOT NULL,
  website     INTEGER NOT NULL,
  gmb         INTEGER NOT NULL,
  social      INTEGER NOT NULL,
  reviews     INTEGER NOT NULL,
  leads       INTEGER NOT NULL,
  captured_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_health_practice_date ON health_score_snapshot(practice_id, captured_at);

-- Sync Log (Channel Sync Engine audit trail)
CREATE TABLE sync_log (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  channel     VARCHAR(50) NOT NULL,      -- 'website', 'gmb', 'facebook', 'instagram', 'healthgrades'
  action      VARCHAR(50) NOT NULL,      -- 'create', 'update', 'delete', 'verify'
  status      VARCHAR(20) NOT NULL,      -- 'success', 'failed', 'pending', 'retrying'
  payload     JSONB,
  error       TEXT,
  attempts    INTEGER DEFAULT 1,
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  completed_at TIMESTAMPTZ
);

CREATE INDEX idx_sync_practice ON sync_log(practice_id);
CREATE INDEX idx_sync_status ON sync_log(status);

-- Change Events (event bus audit trail)
CREATE TABLE change_event (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id UUID NOT NULL REFERENCES practice(id) ON DELETE CASCADE,
  entity_type VARCHAR(50) NOT NULL,      -- 'team_member', 'service', 'practice', 'brand'
  entity_id   UUID,
  action      VARCHAR(20) NOT NULL,      -- 'created', 'updated', 'deleted'
  changes     JSONB NOT NULL,            -- { field: { old, new } }
  triggered_by data_source NOT NULL,
  sync_status VARCHAR(20) DEFAULT 'pending',  -- 'pending', 'syncing', 'complete'
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_change_practice ON change_event(practice_id);
CREATE INDEX idx_change_sync ON change_event(sync_status);
```

### Database Views (Dashboard)

```sql
-- Network enrollment summary
CREATE VIEW v_enrollment_summary AS
SELECT
  enrollment_status,
  COUNT(*) as count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) as percentage
FROM practice
GROUP BY enrollment_status;

-- Health score distribution
CREATE VIEW v_health_distribution AS
SELECT
  CASE
    WHEN health_overall >= 85 THEN 'Optimized'
    WHEN health_overall >= 60 THEN 'Baseline'
    ELSE 'At Risk'
  END as tier,
  COUNT(*) as count,
  ROUND(AVG(health_overall), 1) as avg_score
FROM practice
WHERE enrollment_status IN ('enrolled', 'blitz_deployed')
GROUP BY tier;

-- Practices needing attention (alerts)
CREATE VIEW v_practice_alerts AS
SELECT p.id, p.name, p.slug,
  CASE
    WHEN p.health_overall < 40 THEN 'critical'
    WHEN p.health_reviews < 50 THEN 'review_drop'
    WHEN p.concierge_last_interaction < NOW() - INTERVAL '30 days' THEN 'inactive'
    ELSE 'ok'
  END as alert_type,
  p.health_overall,
  p.enrollment_status
FROM practice p
WHERE p.health_overall < 40
   OR p.health_reviews < 50
   OR (p.enrollment_status = 'enrolled'
       AND p.concierge_last_interaction < NOW() - INTERVAL '30 days');
```

---

## 3. API Endpoints

### Base URL: `/api/v1`

### Practice CRUD

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/practices` | List practices (paginated, filterable by status, specialty, platform, health tier) | API key |
| `GET` | `/practices/:id` | Get full practice profile (includes team, services, health scores) | API key |
| `GET` | `/practices/slug/:slug` | Get practice by slug | API key |
| `POST` | `/practices` | Create new practice record | API key |
| `PATCH` | `/practices/:id` | Update practice fields (triggers change_event) | API key |
| `DELETE` | `/practices/:id` | Soft-delete practice (set inactive) | API key |

### Team Members

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/practices/:id/team` | List team members | API key |
| `POST` | `/practices/:id/team` | Add team member (triggers sync) | API key |
| `PATCH` | `/practices/:id/team/:memberId` | Update team member | API key |
| `DELETE` | `/practices/:id/team/:memberId` | Remove team member (set end_date, triggers sync) | API key |

### Services

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/practices/:id/services` | List services | API key |
| `POST` | `/practices/:id/services` | Add service | API key |
| `PATCH` | `/practices/:id/services/:serviceId` | Update service | API key |
| `DELETE` | `/practices/:id/services/:serviceId` | Remove service | API key |

### Enrollment Flow

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/enrollment/invite` | Send enrollment invitation to practice (or batch) | API key |
| `GET` | `/enrollment/:practiceId/status` | Get enrollment progress | API key |
| `POST` | `/enrollment/:practiceId/verify` | Submit verified practice data | Session |
| `POST` | `/enrollment/:practiceId/select-template` | Practice selects a template | Session |
| `POST` | `/enrollment/:practiceId/approve` | Practice approves storefront for deployment | Session |
| `POST` | `/enrollment/:practiceId/deploy` | Trigger site deployment | API key |
| `POST` | `/enrollment/blitz` | Trigger blitz deploy for non-engaged practices (batch) | API key |

### Harvester

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/harvester/run` | Trigger harvest for a practice (by name + address or URL) | API key |
| `POST` | `/harvester/batch` | Trigger harvest for multiple practices | API key |
| `GET` | `/harvester/:practiceId/results` | Get harvest results with confidence scores | API key |
| `GET` | `/harvester/jobs` | List active/completed harvest jobs | API key |

### Template Engine

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/templates` | List available templates | API key |
| `GET` | `/templates/:id` | Get template details | API key |
| `GET` | `/templates/:id/preview/:practiceId` | Preview template with practice data | Session |
| `POST` | `/sites/generate/:practiceId` | Generate site for practice | API key |
| `POST` | `/sites/deploy/:practiceId` | Deploy generated site | API key |
| `POST` | `/sites/batch-regenerate/:templateId` | Regenerate all sites using a template | API key |
| `GET` | `/sites/:practiceId/status` | Get site deployment status | API key |

### Concierge Agent

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/concierge/:practiceId/message` | Send message to concierge (returns streaming response) | Session |
| `GET` | `/concierge/:practiceId/conversations` | List conversations for practice | API key |
| `GET` | `/concierge/:practiceId/conversations/:convId` | Get full conversation history | API key |
| `POST` | `/concierge/:practiceId/action` | Execute concierge action (approve change, select template) | Session |

### Change Management

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/changes/:practiceId` | Report a change (staff, hours, services) | Session / API key |
| `GET` | `/changes/:practiceId` | List pending/completed changes | API key |
| `GET` | `/changes/:practiceId/:changeId/impact` | Get cascading impact analysis | API key |
| `POST` | `/changes/:practiceId/:changeId/execute` | Execute change across all channels | API key |

### Dashboard & Analytics

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `GET` | `/dashboard/summary` | Network-level summary (enrollment, health, alerts) | Session |
| `GET` | `/dashboard/enrollment` | Enrollment progress breakdown | Session |
| `GET` | `/dashboard/health` | Health score distribution and trends | Session |
| `GET` | `/dashboard/alerts` | Active alerts requiring attention | Session |
| `GET` | `/dashboard/practices/:id/detail` | Detailed practice dashboard data | Session |
| `GET` | `/dashboard/practices/:id/health-history` | Health score trend (30/60/90 day) | Session |

### Batch Operations

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| `POST` | `/batch/harvest` | Harvest data for multiple practices | API key |
| `POST` | `/batch/invite` | Send enrollment invitations to batch | API key |
| `POST` | `/batch/regenerate` | Regenerate sites for batch of practices | API key |
| `POST` | `/batch/sync` | Force sync across channels for batch | API key |
| `GET` | `/batch/jobs/:jobId` | Get batch job status | API key |

---

## 4. Engineering Tasks

### Phase 1: MVP (Days 1-30)

**Goal**: 20-30 practices enrolled and live through Concierge

#### Week 1: Foundation (Days 1-7)

| # | Task | Est | Depends On |
|---|------|-----|------------|
| 1.1 | Set up monorepo (Turborepo): `apps/api`, `apps/dashboard`, `apps/concierge`, `packages/db`, `packages/shared` | 4h | — |
| 1.2 | PostgreSQL schema migration (Prisma) — practice, team_member, service, template, enrollment_event tables | 6h | 1.1 |
| 1.3 | Redis setup + cache service wrapper (get/set/invalidate practice profiles) | 3h | 1.1 |
| 1.4 | API scaffold: Express/Fastify with auth middleware (API key + session), error handling, logging | 4h | 1.1 |
| 1.5 | Practice CRUD endpoints + validation | 6h | 1.2, 1.4 |
| 1.6 | Team member + services CRUD endpoints | 4h | 1.5 |
| 1.7 | Seed database with SGA practice list (names, addresses, platforms, specialties from internal data) | 4h | 1.2 |
| 1.8 | CI/CD pipeline: GitHub Actions, lint, test, deploy to staging | 4h | 1.1 |

#### Week 2: Harvester + Enrollment (Days 8-14)

| # | Task | Est | Depends On |
|---|------|-----|------------|
| 2.1 | Data Harvester service: website scraper (Playwright headless browser + Claude extraction) | 12h | 1.2 |
| 2.2 | GMB data pull (Google Places API for public listing data) | 6h | 1.2 |
| 2.3 | Harvest results → Practice Data Hub writer (with confidence scores per field) | 4h | 2.1, 2.2 |
| 2.4 | BullMQ queue setup: harvest queue, generate queue | 4h | 1.3 |
| 2.5 | Enrollment flow API: invite, verify, select-template, approve, deploy | 8h | 1.5 |
| 2.6 | Run initial harvest on top 30 pilot practices | 4h | 2.1, 2.2, 2.3 |

#### Week 3: Concierge + Templates (Days 15-21)

| # | Task | Est | Depends On |
|---|------|-----|------------|
| 3.1 | Concierge Agent service: Claude API integration with system prompt + practice context injection | 10h | 1.5 |
| 3.2 | Concierge tool definitions: verify_data, update_team, select_template, approve_storefront | 8h | 3.1, 2.5 |
| 3.3 | Concierge conversation persistence (conversation + message tables) | 4h | 3.1 |
| 3.4 | Concierge streaming endpoint (SSE for real-time message streaming) | 4h | 3.1 |
| 3.5 | GP website template (Next.js + Tailwind, static generation from practice data) | 10h | 1.5 |
| 3.6 | Perio website template (referral-optimized variant) | 6h | 3.5 |
| 3.7 | Template preview endpoint (generate on-demand preview with practice data) | 4h | 3.5 |
| 3.8 | Site deployment pipeline: generate static build → push to Cloudflare Pages | 6h | 3.5 |

#### Week 4: Dashboard + Integration (Days 22-30)

| # | Task | Est | Depends On |
|---|------|-----|------------|
| 4.1 | Concierge chat frontend: message interface, streaming responses, action cards | 10h | 3.4 |
| 4.2 | HQ Dashboard: practice list, enrollment progress, drill-down to practice detail | 10h | 1.5 |
| 4.3 | Dashboard auth (session-based, HQ team accounts) | 4h | 4.2 |
| 4.4 | Enrollment invitation system (email via SendGrid/Resend + unique practice link) | 4h | 2.5 |
| 4.5 | Change event system: practice data write → emit change_event → trigger sync queue | 6h | 1.5, 2.4 |
| 4.6 | End-to-end testing: harvest → enroll → verify → template → deploy for 5 pilot practices | 8h | All above |
| 4.7 | Pilot launch: invite 20-30 practices, monitor, iterate | Ongoing | 4.6 |

**Phase 1 total estimated effort**: ~168 hours (~4.2 engineer-weeks)

### Phase 2: Scale (Days 31-60)

**Goal**: All 260+ practices at baseline (enrolled or blitz-deployed)

| # | Task | Est | Depends On |
|---|------|-----|------------|
| 5.1 | Full Arch template | 8h | 3.5 |
| 5.2 | Pediatric template | 8h | 3.5 |
| 5.3 | Multi-Specialty template | 8h | 3.5 |
| 5.4 | Channel Sync: GMB API integration (create/update listings) | 12h | 4.5 |
| 5.5 | Channel Sync: Facebook Graph API (profile management, post publishing) | 10h | 4.5 |
| 5.6 | Channel Sync: Instagram Basic Display API | 8h | 4.5 |
| 5.7 | Blitz deploy system: auto-deploy defaults for non-engaged practices after 30-day window | 8h | 3.8, 2.3 |
| 5.8 | Concierge: change management flows (staff change → impact analysis → scheduled propagation) | 10h | 3.2, 4.5 |
| 5.9 | Health score calculation engine (automated scoring per the formula in architecture doc) | 8h | 5.4, 5.5 |
| 5.10 | Dashboard: health scores, tier distribution, alert cards | 8h | 5.9 |
| 5.11 | Dashboard: batch operations UI (batch invite, batch regenerate) | 6h | 4.2 |
| 5.12 | iSolved webhook integration (staff change detection → concierge notification) | 10h | 3.2 |
| 5.13 | Nightly reconciliation batch: safety-net sync to catch anything events missed | 4h | 5.4, 5.5 |
| 5.14 | Full network rollout: invite remaining practices, trigger blitz for non-responders | Ongoing | 5.7 |

**Phase 2 total estimated effort**: ~108 hours (~2.7 engineer-weeks)

---

## 5. Infrastructure Plan

### Hosting Architecture

| Component | Host | Rationale |
|-----------|------|-----------|
| Practice websites (260+) | Cloudflare Pages | Edge-hosted static sites, free SSL, custom domains, global CDN |
| API server | Railway (or Render) | Managed Node.js hosting, auto-scaling, zero-config deploys |
| Dashboard + Concierge UI | Vercel | Next.js-optimized hosting, edge functions, preview deploys |
| PostgreSQL | Railway (or Neon) | Managed Postgres, automatic backups, connection pooling |
| Redis | Railway (or Upstash) | Managed Redis, low-latency, BullMQ-compatible |
| Harvester workers | Railway (background workers) | Long-running Python processes, isolated from API |
| File storage (photos, logos) | Cloudflare R2 | S3-compatible, no egress fees, edge-cached |

### CI/CD Pipeline

```
GitHub (monorepo)
  ├─ Push to main → GitHub Actions
  │   ├─ Lint + Type Check (Biome/ESLint + TypeScript)
  │   ├─ Unit Tests (Vitest)
  │   ├─ Integration Tests (against test DB)
  │   ├─ Build all apps
  │   ├─ Deploy API → Railway (staging → production)
  │   ├─ Deploy Dashboard → Vercel (preview → production)
  │   └─ Deploy Template changes → trigger site regeneration
  │
  ├─ Pull Request → GitHub Actions
  │   ├─ Lint + Test (blocking)
  │   ├─ Preview deploy (Vercel)
  │   └─ Schema diff check (Prisma migrate diff)
  │
  └─ Scheduled (nightly)
      ├─ Health score recalculation
      ├─ Reconciliation sync
      └─ Database backup verification
```

### Monitoring & Observability

| Layer | Tool | What It Monitors |
|-------|------|-----------------|
| Application logs | Axiom (or Logtail) | API requests, errors, concierge conversations, sync events |
| Error tracking | Sentry | Runtime exceptions, unhandled rejections, error grouping |
| Uptime monitoring | Better Uptime | API health, dashboard availability, practice site sampling |
| Queue monitoring | Bull Board (self-hosted) | Job queue depth, failure rates, processing times |
| Database | Railway metrics + pg_stat_statements | Query performance, connection pool usage, storage growth |
| Site performance | Cloudflare Analytics | Practice site load times, cache hit rates, error rates |
| Business metrics | Custom dashboard (built in Phase 1) | Enrollment rate, health scores, concierge usage |

### Secrets Management

| Environment | Strategy |
|-------------|----------|
| Local development | `.env.local` files (gitignored), documented in `.env.example` |
| CI/CD | GitHub Actions secrets (encrypted, scoped per environment) |
| Staging | Railway environment variables (encrypted at rest) |
| Production | Railway environment variables + Doppler (centralized secret management) |

**Secrets inventory**:
- `DATABASE_URL` — PostgreSQL connection string
- `REDIS_URL` — Redis connection string
- `CLAUDE_API_KEY` — Anthropic API key
- `CLOUDFLARE_API_TOKEN` — Pages deployment
- `CLOUDFLARE_ACCOUNT_ID` — Pages project management
- `GOOGLE_API_KEY` — Places API (harvester)
- `GOOGLE_GMB_CREDENTIALS` — GMB API service account (Phase 2)
- `META_APP_ID` / `META_APP_SECRET` — Facebook/Instagram APIs (Phase 2)
- `SENDGRID_API_KEY` (or `RESEND_API_KEY`) — Enrollment invitation emails
- `SESSION_SECRET` — Dashboard session encryption
- `API_KEY_HASH_SALT` — Internal API key hashing

---

## 6. Risk Register

| # | Risk | Probability | Impact | Mitigation |
|---|------|:-----------:|:------:|------------|
| R1 | Practice websites have wildly inconsistent HTML, causing harvester extraction failures | HIGH | MEDIUM | Use Claude for extraction (handles arbitrary HTML). Log confidence scores per field. Fall back to manual entry for low-confidence fields. Iteratively improve prompts based on failure patterns. |
| R2 | Low practice engagement — offices too busy to interact with concierge | HIGH | HIGH | Blitz deploy as safety net (ADR-4). Design 5-minute interaction windows. Pre-populate everything possible. The first interaction must deliver visible value, not ask for data entry. 30-day auto-deploy deadline. |
| R3 | Claude API rate limits throttle concurrent enrollments | MEDIUM | MEDIUM | BullMQ queue with concurrency limits (max 20 concurrent API calls). Prioritize active conversations over batch jobs. Cache common responses. Monitor token usage and adjust model selection if costs spike. |
| R4 | GMB API access restrictions (Google Business Profile API requires business verification) | MEDIUM | HIGH | Phase 1 uses Google Places API (public data, read-only) for harvesting. GMB write access (Phase 2) requires verified business manager account — work with SGA operations to establish account authority. Fallback: manual GMB updates for Phase 1. |
| R5 | 260+ Cloudflare Pages deployments hit account limits | LOW | MEDIUM | Cloudflare Pages supports 100 projects on Pro plan. Use path-based routing within fewer projects (group by region or platform) or negotiate Enterprise agreement. Alternative: Vercel with monorepo deploy, or self-hosted Caddy with static file serving. |
| R6 | Practice data accuracy — harvested data may be outdated or wrong | HIGH | MEDIUM | Always present harvested data as "draft" with confidence scores. Verification step in enrollment is mandatory. Never auto-deploy unverified data to live channels without blitz-deploy flagging. Dashboard marks auto-deployed sites as "unverified." |
| R7 | Template changes break existing practice sites | LOW | HIGH | Preview pipeline: template change → regenerate all affected sites in preview → HQ QC review → deploy. Never auto-deploy template changes without preview. Automated visual regression testing (Percy or Playwright screenshots). |
| R8 | Claude conversation quality degrades with complex practice scenarios | MEDIUM | MEDIUM | Comprehensive system prompt with edge case handling. Escalation path to human HQ team member when confidence is low. Conversation quality review during pilot (sample 20% of conversations). Iterative prompt refinement. |
| R9 | Scope creep from HQ feature requests during Phase 1 | HIGH | MEDIUM | Phase 1 scope is locked: enrollment flow end-to-end for 20-30 practices. All other features are Phase 2+. Feature requests go into a backlog. Weekly stakeholder sync to manage expectations. |
| R10 | Single point of failure — Redis or Postgres outage takes down all systems | LOW | HIGH | Managed hosting with automatic failover (Railway/Neon). Redis is cache-only (system degrades gracefully, falls back to Postgres). Database backups every 6 hours. Documented runbook for recovery. |

---

## 7. Architecture Decision Records (ADRs)

### ADR-T1: Generated Sites, Not CMS

**Status**: Accepted (affirms ADR-2 from architecture doc)

**Context**: 260+ practice websites need to be created, maintained, and batch-updated by a small team. Options include WordPress Multisite, Webflow, or custom static site generation.

**Decision**: Generate static sites from Next.js templates + practice data. Deploy as static assets to Cloudflare Pages.

**Rationale**:
- **Batch updates**: Template change = one code change + regeneration pipeline. Not 260 manual CMS updates.
- **Performance**: Static sites serve from CDN edge. No server-side rendering, no database queries at request time. Sub-second load times globally.
- **Security**: No CMS admin panels exposed. No plugin vulnerabilities. No database to compromise.
- **Cost**: Static hosting is effectively free at scale (Cloudflare Pages, no per-request charges).
- **Tradeoff**: Content updates require regeneration + deployment (1-3 minutes), not instant CMS edits. Acceptable for this use case — practice websites change weekly at most.

### ADR-T2: Event-Driven Sync, Not Batch

**Status**: Accepted (affirms ADR-3 from architecture doc)

**Context**: When practice data changes (staff removed, hours updated, new service added), the change must propagate to website, GMB, social, and directories.

**Decision**: Event-driven propagation via BullMQ, with nightly batch reconciliation as safety net.

**Rationale**:
- **The Dr. Johnson scenario**: Staff removal must propagate within minutes, not on a nightly batch. A departed doctor appearing on the website overnight is a real problem for practice credibility.
- **BullMQ provides**: Reliable delivery (Redis-backed persistence), retry with exponential backoff (handles GMB/Meta API rate limits), job prioritization (active user changes > background sync), and observable queue metrics.
- **Nightly reconciliation**: A scheduled job compares Practice Data Hub state against each channel's current state. Catches anything the event system missed (network failures, API outages during sync).

### ADR-T3: One Agent Per Practice (Logical Isolation)

**Status**: Accepted (affirms ADR-1 from architecture doc)

**Context**: The Concierge Agent serves 260+ practices. Architecture options: one shared agent with practice selection, or one logical agent instance per practice.

**Decision**: One logical agent per practice. Same Claude model, same base system prompt, different practice-specific context window.

**Implementation**:
```
System Prompt = BASE_PROMPT + SPECIALTY_CONTEXT + PRACTICE_DATA
```
- `BASE_PROMPT`: SGA brand standards, personality (Sage + Caregiver), available tools, guardrails
- `SPECIALTY_CONTEXT`: Specialty-specific best practices (GP new-patient focus, Perio referral optimization, etc.)
- `PRACTICE_DATA`: Full practice profile from Redis cache (team, services, brand, enrollment status, health scores, conversation history summary)

**Rationale**: The practice experience must feel personal. "My concierge knows my practice" is fundamentally different from "I logged into a system and selected my practice." Context-switching errors (recommending GP advice to a periodontist) are eliminated because the agent is always operating in a single practice's context. The overhead is minimal — only the context window differs per practice, not the model or infrastructure.

### ADR-T4: Prisma ORM (Not Drizzle, Not Raw SQL)

**Status**: Accepted

**Context**: Need an ORM for type-safe database access in a TypeScript codebase.

**Decision**: Prisma ORM.

**Rationale**:
- **Type safety**: Prisma generates TypeScript types from the schema. Every query is fully typed. Catches data model errors at compile time.
- **Migrations**: `prisma migrate` provides deterministic, version-controlled schema migrations. Critical for a schema that will evolve rapidly in Phase 1.
- **Ecosystem**: Most widely adopted TypeScript ORM. Team familiarity is likely. Extensive documentation.
- **Tradeoff vs. Drizzle**: Drizzle is lighter and closer to SQL. Prisma adds a query engine process. At our scale (260 practices, not 260K), Prisma's overhead is negligible and the developer experience benefits outweigh performance costs.

### ADR-T5: Monorepo with Turborepo

**Status**: Accepted

**Context**: The system has multiple deployable units (API, dashboard, concierge UI, practice sites) sharing code (database client, types, utilities).

**Decision**: Turborepo monorepo with shared packages.

**Structure**:
```
practice-concierge/
├── apps/
│   ├── api/              # Express/Fastify API server
│   ├── dashboard/        # Next.js HQ dashboard
│   ├── concierge/        # Next.js concierge chat interface
│   └── site-generator/   # Next.js static site generator (templates)
├── packages/
│   ├── db/               # Prisma schema + client + migrations
│   ├── shared/           # Shared types, constants, utilities
│   └── ui/               # Shared React components (design system)
├── workers/
│   ├── harvester/        # Python harvester service
│   └── sync/             # Node.js channel sync workers
├── turbo.json
└── package.json
```

**Rationale**: Shared `packages/db` means one Prisma schema, one set of generated types, used by API, dashboard, and sync workers. Changes to the data model are reflected everywhere immediately. Turborepo caches builds and runs tasks in parallel, keeping CI fast as the codebase grows.

---

## 8. Phase 1 Success Criteria

| Metric | Target |
|--------|--------|
| Practices enrolled via Concierge | 20-30 |
| Average enrollment time (invitation → live site) | < 2 weeks |
| Concierge conversation completion rate | > 70% |
| Practice data verification accuracy | > 90% of fields confirmed correct |
| Site deployment success rate | > 95% |
| API uptime | > 99.5% |
| Concierge response latency (p95) | < 5 seconds |
| HQ dashboard load time | < 2 seconds |

---

## Appendix: Technology Version Pinning

| Technology | Version | Notes |
|------------|---------|-------|
| Node.js | 20 LTS | Runtime for API, dashboard, sync workers |
| Python | 3.12 | Harvester service |
| PostgreSQL | 16 | Primary data store |
| Redis | 7 | Cache + BullMQ backing store |
| Next.js | 14 | Dashboard, concierge UI, site generator |
| Prisma | 5.x | ORM + migrations |
| BullMQ | 5.x | Job queue |
| Tailwind CSS | 3.x | Design system |
| TypeScript | 5.4+ | All Node.js code |
| Playwright | 1.42+ | Harvester browser automation |
| Claude API | claude-sonnet-4-20250514 | Concierge + harvester extraction |
