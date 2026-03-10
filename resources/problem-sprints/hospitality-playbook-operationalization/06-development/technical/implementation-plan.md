# Technical Implementation Plan: The Hospitality Playbook Platform

**Sprint**: Hospitality Playbook Operationalization
**Phase**: Phase 1 — Pilot (20-30 practices, 90 days)
**Scope**: Morning Huddle OS + H-Score Engine + CCA Bronze + Trust Transfer Module + Practice Dashboard

---

## Stack Recommendation

### Backend

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Runtime | Node.js 20 (TypeScript) | Strong async support for webhook processing and real-time scoring; TypeScript enforces data model correctness across the multi-entity schema |
| API Framework | Fastify | Higher throughput than Express; schema-first request/response validation prevents data quality issues in webhook ingestion |
| Database | PostgreSQL 16 (via Supabase or Neon) | Relational integrity required for the multi-entity data model; time-series scoring data fits well in partitioned tables; managed hosting removes ops burden |
| ORM | Prisma | Type-safe queries; auto-generated migration; schema-first development matches the well-defined data model |
| Background Jobs | BullMQ (Redis) | Daily H-Score recalculation, review NLP processing, and certificate generation are all async jobs; BullMQ provides reliable queue with retry and dead letter |
| NLP / Text Analysis | OpenAI API (gpt-4o-mini) | Trust transfer phrase detection and hospitality sentiment scoring; cost-effective for batch review processing; switchable to local model later |
| Video Delivery | Mux | Purpose-built video hosting for LMS-style use cases; adaptive bitrate streaming; per-view analytics; thumbnail generation; cost-effective at SGA scale |
| Credential Issuance | Credly API | Purpose-built external credentialing; LinkedIn integration; verification URL generation; avoids custom-building credential infrastructure |
| Auth | Supabase Auth (JWT) | Multi-tenant auth with role-based access (staff / practice owner / regional director / SGA admin); SSO pathway for enterprise rollout |
| Cache | Redis (via Upstash) | H-Score leaderboard, practice dashboard data, content package assembly |

### Frontend

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Framework | Next.js 14 (App Router) | Server components for SEO and initial page performance; client components for interactive huddle player and H-Score gauge; strong TypeScript support |
| UI | Tailwind CSS + custom design tokens | Rapid development; design token system maps to the gold/dark color system; no heavy component library (custom components per design specs) |
| State | Zustand | Lightweight client state for huddle session (video progress, exercise completion, review data); avoids Redux overhead for a focused use case |
| Video Player | Mux Player (React) | First-party Mux React component; handles adaptive streaming, captions, analytics; matches the video delivery choice |
| Animations | CSS transitions only | Per ADR from design specs: no JS animation libraries; keeps bundle light for tablet performance |
| Mobile | Progressive Web App | No App Store approval required for B2B rollout; push notifications via Web Push API; offline video pre-loading via Service Worker |

### Infrastructure

| Component | Service | Notes |
|-----------|---------|-------|
| Hosting | Vercel (frontend) + Railway (backend) | Vercel edge deployment for global performance; Railway for managed Node.js and PostgreSQL |
| CDN | Mux + Cloudflare | Mux handles video CDN; Cloudflare handles static assets and API edge caching |
| Monitoring | Sentry + Datadog | Sentry for error tracking; Datadog for H-Score calculation job monitoring |
| Secrets | Doppler | Centralized secret management for Cinnamon/Swell/Dental Intel API keys |

---

## 10-Component Build Sequence

### Phase 1 (Months 1-3) — Build in this order

```
Week 1-2:   Foundation
  ├── Database schema for Practice, StaffMember, HScore, HuddleSession, Review
  ├── Auth service (multi-tenant, role-based)
  ├── Practice activation workflow (H-Score baseline generation on first login)
  └── API scaffolding (Fastify + Prisma)

Week 3-4:   Integration Spikes
  ├── Cinnamon webhook listener + call analytics ingestion
  ├── Swell webhook listener + review ingestion
  ├── Dental Intel API assessment (content push or pull cadence)
  └── Mux video upload pipeline (for CCA module content)

Week 5-6:   H-Score Engine
  ├── Scoring calculation service (Cinnamon 35% + Swell 45% + Dental Intel 20%)
  ├── Practice-type normalized scoring (rural/suburban/urban/cosmetic/pediatric)
  ├── Daily recalculation BullMQ job
  ├── Anomaly detection (>10 point drop in 7 days)
  └── Practice dashboard API endpoints

Week 7-8:   CCA Academy Bronze
  ├── Module sequencer (10 modules, gated progression)
  ├── Scenario assessment engine (response options, scoring, retry logic)
  ├── Certification record management
  ├── Credly API integration (certificate issuance, LinkedIn publishing)
  └── Staff roster import

Week 9-10:  Morning Huddle OS
  ├── Content package assembly (behavior taxonomy filtering by practice type)
  ├── Dental Intel content delivery integration (or fallback web URL)
  ├── Live Swell review surface in huddle (last 24-hour review pull)
  ├── Completion event handling
  └── Huddle streak tracking

Week 11-12: Trust Transfer + Polish
  ├── Review NLP pipeline (trust transfer phrase detection via OpenAI)
  ├── Trust transfer sentiment scoring (30-day rolling)
  ├── Daily huddle commitment tap logging
  ├── Practice dashboard frontend integration
  └── Pilot onboarding flow

Week 13:    Pilot Launch Prep
  ├── Load testing (20-30 practices, concurrent huddles)
  ├── Data migration for pilot practice Swell/Cinnamon backfill
  ├── Onboarding guide for practice owners
  └── Regional director pilot dashboard (basic view)
```

### Phase 2 (Months 4-9) — After pilot validation

- Component 7: Rewards Engine (leaderboard, bonus pool, jacket fulfillment)
- Component 9: Network Intelligence Dashboard (regional director portfolio view)
- CCA Silver track (requires 6 months of H-Score data for eligibility criteria)
- Component 6: Champion Network Layer (designation workflow, regional cohorts)
- Component 8: Playbook Distribution System (content versioning)

### Phase 3 (Months 10-18) — Credential and academic pathway

- CCA Gold track (requires panel review system)
- Academic publication data export
- Content Production Pipeline tooling
- External credential recognition (LinkedIn institution application)

---

## Database Schema

### Core Tables

```sql
-- Multi-tenant foundation
CREATE TABLE organizations (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name          TEXT NOT NULL,
  created_at    TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE practices (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id                UUID REFERENCES organizations(id),
  name                  TEXT NOT NULL,
  practice_type         TEXT CHECK (practice_type IN ('rural','suburban','urban','cosmetic','pediatric')),
  region                TEXT,
  dental_intel_id       TEXT,
  cinnamon_account_id   TEXT,
  swell_account_id      TEXT,
  h_score_current       NUMERIC(5,2) DEFAULT 0,
  h_score_level         TEXT DEFAULT 'emerging',
  champion_staff_id     UUID,
  playbook_version      TEXT DEFAULT 'v1.0',
  activated_at          TIMESTAMPTZ,
  created_at            TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE staff_members (
  id                        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id               UUID REFERENCES practices(id),
  full_name                 TEXT NOT NULL,
  email                     TEXT,
  role                      TEXT CHECK (role IN ('front_desk','hygienist','da','doctor','office_manager','admin')),
  hire_date                 DATE,
  cca_tier                  TEXT DEFAULT 'none' CHECK (cca_tier IN ('none','bronze','silver','gold')),
  bronze_completed_at       TIMESTAMPTZ,
  silver_completed_at       TIMESTAMPTZ,
  gold_completed_at         TIMESTAMPTZ,
  is_champion               BOOLEAN DEFAULT false,
  champion_tier             TEXT CHECK (champion_tier IN ('candidate','certified','master')),
  credly_credential_id      TEXT,
  bonus_multiplier          NUMERIC(3,2) DEFAULT 1.0,
  created_at                TIMESTAMPTZ DEFAULT now()
);

-- H-Score Engine
CREATE TABLE h_score_records (
  id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id             UUID REFERENCES practices(id),
  recorded_date           DATE NOT NULL,
  h_score                 NUMERIC(5,2),
  swell_component         NUMERIC(5,2),
  cinnamon_component      NUMERIC(5,2),
  dental_intel_component  NUMERIC(5,2),
  -- Swell sub-scores
  swell_star_rating       NUMERIC(5,2),
  swell_velocity          NUMERIC(5,2),
  swell_sentiment         NUMERIC(5,2),
  -- Cinnamon sub-scores
  cinnamon_answer_rate    NUMERIC(5,2),
  cinnamon_greeting       NUMERIC(5,2),
  cinnamon_hold           NUMERIC(5,2),
  -- Dental Intel sub-scores
  di_huddle_completion    NUMERIC(5,2),
  di_certification_rate   NUMERIC(5,2),
  -- Metadata
  anomaly_flag            BOOLEAN DEFAULT false,
  anomaly_type            TEXT,
  practice_type_cohort    TEXT,
  cohort_percentile       NUMERIC(5,2),
  created_at              TIMESTAMPTZ DEFAULT now(),
  UNIQUE(practice_id, recorded_date)
);

CREATE INDEX idx_h_score_practice_date ON h_score_records(practice_id, recorded_date DESC);

-- Huddle Sessions
CREATE TABLE huddle_sessions (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id           UUID REFERENCES practices(id),
  session_date          DATE NOT NULL,
  content_package_id    TEXT,
  status                TEXT DEFAULT 'not_started' CHECK (status IN ('not_started','in_progress','complete','partial','skipped')),
  facilitator_id        UUID REFERENCES staff_members(id),
  started_at            TIMESTAMPTZ,
  completed_at          TIMESTAMPTZ,
  duration_seconds      INTEGER,
  staff_count           INTEGER,
  -- Segment completion
  culture_moment_done   BOOLEAN DEFAULT false,
  wins_segment_done     BOOLEAN DEFAULT false,
  focus_segment_done    BOOLEAN DEFAULT false,
  accountability_done   BOOLEAN DEFAULT false,
  intention_done        BOOLEAN DEFAULT false,
  -- Trust transfer
  trust_transfer_committed BOOLEAN DEFAULT false,
  created_at            TIMESTAMPTZ DEFAULT now(),
  UNIQUE(practice_id, session_date)
);

-- Reviews (from Swell and other sources)
CREATE TABLE reviews (
  id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id                 UUID REFERENCES practices(id),
  external_id                 TEXT,
  source                      TEXT DEFAULT 'swell',
  received_at                 TIMESTAMPTZ,
  star_rating                 INTEGER CHECK (star_rating BETWEEN 1 AND 5),
  review_text                 TEXT,
  reviewer_first_name         TEXT,
  -- NLP output
  hospitality_sentiment       NUMERIC(4,3),
  trust_transfer_signal       TEXT CHECK (trust_transfer_signal IN ('positive','neutral','negative','unprocessed')),
  hospitality_phrases_found   TEXT[],
  -- Staff attribution
  staff_mention               BOOLEAN DEFAULT false,
  mentioned_staff_id          UUID REFERENCES staff_members(id),
  -- Flags
  flagged_for_huddle          BOOLEAN DEFAULT false,
  wow_story_candidate         BOOLEAN DEFAULT false,
  nlp_processed_at            TIMESTAMPTZ,
  created_at                  TIMESTAMPTZ DEFAULT now()
);

-- CCA Certification
CREATE TABLE cca_module_completions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  staff_id        UUID REFERENCES staff_members(id),
  module_id       TEXT NOT NULL,  -- e.g., 'bronze-01', 'bronze-02'
  tier            TEXT NOT NULL,
  module_number   INTEGER,
  started_at      TIMESTAMPTZ,
  completed_at    TIMESTAMPTZ,
  scenario_score  INTEGER CHECK (scenario_score BETWEEN 0 AND 100),
  passed          BOOLEAN,
  attempts        INTEGER DEFAULT 1,
  playbook_version TEXT DEFAULT 'v1.0',
  created_at      TIMESTAMPTZ DEFAULT now(),
  UNIQUE(staff_id, module_id)
);

CREATE TABLE certifications (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  staff_id              UUID REFERENCES staff_members(id),
  tier                  TEXT CHECK (tier IN ('bronze','silver','gold')),
  issued_at             TIMESTAMPTZ DEFAULT now(),
  expires_at            TIMESTAMPTZ,
  credly_badge_id       TEXT,
  credly_issued_at      TIMESTAMPTZ,
  verification_url      TEXT,
  revoked_at            TIMESTAMPTZ,
  playbook_version      TEXT DEFAULT 'v1.0',
  created_at            TIMESTAMPTZ DEFAULT now()
);

-- Champions
CREATE TABLE champions (
  id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  staff_id                UUID REFERENCES staff_members(id),
  practice_id             UUID REFERENCES practices(id),
  tier                    TEXT DEFAULT 'candidate' CHECK (tier IN ('candidate','certified','master')),
  designated_at           TIMESTAMPTZ DEFAULT now(),
  huddles_facilitated     INTEGER DEFAULT 0,
  staff_onboarded_count   INTEGER DEFAULT 0,
  wow_stories_submitted   INTEGER DEFAULT 0,
  regional_cohort_id      UUID,
  created_at              TIMESTAMPTZ DEFAULT now()
);

-- Cinnamon call analytics (ingested daily)
CREATE TABLE cinnamon_daily_analytics (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id           UUID REFERENCES practices(id),
  analytics_date        DATE NOT NULL,
  total_calls           INTEGER,
  calls_answered_3ring  INTEGER,
  calls_with_greeting   INTEGER,
  calls_over_hold       INTEGER,
  answer_rate           NUMERIC(5,4),
  greeting_rate         NUMERIC(5,4),
  hold_compliance_rate  NUMERIC(5,4),
  created_at            TIMESTAMPTZ DEFAULT now(),
  UNIQUE(practice_id, analytics_date)
);

-- Playbook versioning
CREATE TABLE playbook_versions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  version_number  TEXT UNIQUE NOT NULL,
  released_at     TIMESTAMPTZ,
  is_current      BOOLEAN DEFAULT false,
  major_changes   TEXT,
  behaviors_added TEXT[],
  created_at      TIMESTAMPTZ DEFAULT now()
);
```

---

## API Endpoints

### Practice Management
```
GET    /api/v1/practices/:id/dashboard         — Full dashboard data (H-Score + components + huddle + certs)
GET    /api/v1/practices/:id/h-score           — Current H-Score with component breakdown
GET    /api/v1/practices/:id/h-score/history   — H-Score trend (7/30/90 days)
POST   /api/v1/practices/:id/activate          — Trigger baseline H-Score generation
GET    /api/v1/practices/:id/opportunity       — "This Week's Opportunity" recommendation
```

### Huddle Session Management
```
GET    /api/v1/huddles/:practiceId/today       — Today's content package (assembled, ready to play)
POST   /api/v1/huddles/:practiceId/start       — Create session record, mark in_progress
PATCH  /api/v1/huddles/:sessionId/segment      — Update segment completion (body: {segment, complete: true})
POST   /api/v1/huddles/:sessionId/complete     — Mark session complete, trigger H-Score update
POST   /api/v1/huddles/:sessionId/commit-trust — Log trust transfer commitment tap
GET    /api/v1/huddles/:practiceId/streak      — Current huddle streak and milestones
```

### H-Score Calculation
```
POST   /api/v1/scoring/recalculate/:practiceId — Trigger manual recalculation (admin + cron)
GET    /api/v1/scoring/algorithm               — Returns the full algorithm documentation (ADR-02: transparency)
GET    /api/v1/scoring/leaderboard             — Network leaderboard (practice-type filtered, anonymized)
GET    /api/v1/scoring/benchmarks/:type        — Practice-type cohort benchmarks
```

### Certification
```
GET    /api/v1/staff/:staffId/certification    — Full certification status and progress
GET    /api/v1/staff/:staffId/modules          — Module list with completion status
GET    /api/v1/modules/:moduleId               — Module content (video URL, scenario questions)
POST   /api/v1/modules/:moduleId/complete      — Submit scenario assessment, record completion
POST   /api/v1/certifications/issue            — Trigger certificate generation + Credly issuance
GET    /api/v1/certifications/:certId/verify   — Public verification endpoint (no auth required)
```

### Champion Management
```
GET    /api/v1/practices/:id/champion-readiness — Readiness score + recommended staff member
POST   /api/v1/champions/designate              — Designate a champion (practice owner action)
GET    /api/v1/champions/:championId            — Champion profile + impact stats
POST   /api/v1/champions/:championId/wow-story  — Submit a wow story
```

### Webhooks (Inbound)
```
POST   /webhooks/swell/review                  — Swell review webhook (new review received)
POST   /webhooks/swell/nps                     — Swell NPS survey webhook
POST   /webhooks/cinnamon/daily                — Cinnamon daily analytics push
POST   /webhooks/dental-intel/completion       — Dental Intel huddle completion event
```

---

## Integration Specifications

### Cinnamon Integration

**Connection type**: Webhook push (preferred) or daily scheduled pull
**Data required per practice per day**:
```json
{
  "practice_id": "cinnamon-account-id",
  "date": "2026-03-10",
  "total_calls": 42,
  "answered_within_3_rings": 34,
  "calls_with_greeting_phrase": 28,
  "calls_with_hold_over_2min": 4,
  "call_sentiments": ["positive", "neutral", "positive", ...],
  "greeting_phrases_detected": ["good morning this is dr chen's office", ...]
}
```
**Implementation notes**:
- If Cinnamon supports webhooks, register endpoint at `/webhooks/cinnamon/daily`
- If Cinnamon only supports pull, create a nightly BullMQ job at 2:00 AM per practice timezone to fetch prior day's analytics
- Greeting phrase detection: Cinnamon already has NLP; verify if their API returns phrase match data or if we must process raw audio
- Fallback: if Cinnamon API does not support phrase detection, use their raw call recording URLs with our own Whisper transcription pipeline

### Swell Integration

**Connection type**: Webhook push
**Webhook events to subscribe**:
- `review.created` — New review received
- `review.updated` — Review updated (e.g., star rating changed)
- `nps.submitted` — NPS survey response received

**Webhook payload (review.created)**:
```json
{
  "event": "review.created",
  "review_id": "swell-review-id",
  "practice_id": "swell-location-id",
  "star_rating": 5,
  "review_text": "Everyone I talked to seemed to know my whole story...",
  "reviewer_name": "Jim T.",
  "created_at": "2026-03-10T14:23:00Z"
}
```
**Processing pipeline on review receipt**:
1. Store raw review to `reviews` table
2. Queue NLP processing job (BullMQ, max 5 min delay)
3. NLP job: send review text to OpenAI → extract `hospitality_sentiment`, `trust_transfer_signal`, `hospitality_phrases_found`, `staff_mention`
4. Update review record with NLP output
5. If `trust_transfer_signal = 'positive'` → flag for huddle display
6. Trigger incremental H-Score Swell component update (not full recalculation)

### Dental Intel Integration

**Connection type**: Requires API assessment spike (see Technical Spikes)
**Two scenarios**:
- **Scenario A (preferred)**: Dental Intel exposes a content injection API → we push today's content package (video URLs, segment scripts) to their system; they display it; they send a completion webhook back
- **Scenario B (fallback)**: Dental Intel does not support content injection → we build a standalone web URL (`app.playbook.sga.com/huddle/:practiceId`) that practice owners open in a browser on their Dental Intel screen; completion is tracked via session analytics in our system

**Completion event (from Dental Intel or our fallback URL)**:
```json
{
  "practice_id": "sga-practice-id",
  "session_date": "2026-03-10",
  "completed_at": "2026-03-10T08:03:22Z",
  "duration_seconds": 298,
  "segments_completed": ["culture_moment", "wins", "focus", "accountability", "intention"]
}
```

---

## Background Job Architecture

All async jobs run on BullMQ with Redis. Job queues:

| Queue | Job | Schedule | SLA |
|-------|-----|----------|-----|
| `h-score-daily` | Recalculate H-Score for all practices | 6:00 AM daily (practice timezone) | < 5 min per practice |
| `review-nlp` | Process new review through NLP pipeline | On webhook receipt, max 5 min delay | < 3 min |
| `huddle-package` | Assemble next 7 days of content packages per practice | Sunday 11 PM weekly | < 30 min total |
| `cinnamon-pull` | Fetch prior day analytics if webhook not configured | 2:00 AM daily | < 10 min |
| `cert-issue` | Trigger Credly certificate issuance | On certification completion event | < 15 min |
| `anomaly-check` | Detect H-Score anomalies (>10 drop in 7 days) | 8:00 AM daily | < 2 min |
| `intervention-alert` | Generate intervention alerts for missed huddles | 9:00 AM daily | < 2 min |
| `cohort-benchmarks` | Recalculate practice-type cohort benchmarks | Sunday 12 AM weekly | < 15 min |

---

## Phase 1 Engineering Task Breakdown

### Backend (estimated story points)

| Task | Points | Dependencies |
|------|--------|-------------|
| Database schema + migrations | 5 | — |
| Auth service (multi-tenant, role-based) | 8 | Schema |
| Practice activation + H-Score baseline | 8 | Swell + Cinnamon integrations |
| Swell webhook listener + ingestion | 5 | Schema |
| Swell NLP pipeline (review processing) | 8 | OpenAI API access |
| Cinnamon webhook/pull + ingestion | 5 | Cinnamon API access |
| H-Score calculation service | 13 | Swell + Cinnamon + DI integrations |
| H-Score daily recalculation job | 5 | Calculation service |
| Anomaly detection + alert generation | 5 | H-Score service |
| Practice dashboard API | 5 | H-Score service |
| Dental Intel integration (post-spike) | 8-13 | DI API assessment |
| Huddle content package assembly | 8 | Content CMS + DI integration |
| Huddle session lifecycle API | 5 | Schema + DI integration |
| CCA module sequencer | 8 | Mux video hosting |
| Scenario assessment engine | 5 | Module sequencer |
| Certification record + issuance | 5 | Schema |
| Credly API integration | 8 | Credly account |
| Trust transfer NLP tagging | 3 | NLP pipeline (reuse Swell) |
| Champion designation workflow | 5 | Schema |
| **Total** | **~134 pts** | |

### Frontend (estimated story points)

| Task | Points | Dependencies |
|------|--------|-------------|
| Design system (tokens, base components) | 8 | Design specs |
| Practice dashboard page | 13 | Dashboard API |
| H-Score gauge component | 5 | Dashboard API |
| Morning huddle player | 13 | Content package API + Mux Player |
| CCA module viewer | 10 | Module API + Mux Player |
| Certification progress page | 8 | Certification API |
| Certification achievement screen | 5 | Certification API |
| Staff profile + certification badge | 5 | Staff API |
| Mobile PWA setup + service worker | 8 | All screens |
| **Total** | **~75 pts** | |

---

## Risk Register

| Risk | Severity | Probability | Mitigation |
|------|----------|-------------|-----------|
| Dental Intel API does not support content injection | HIGH | MEDIUM | Build fallback web URL on day 1; spike in week 3 to determine path |
| Cinnamon does not expose phrase detection data | MEDIUM | LOW | Use Whisper transcription fallback pipeline on call recordings |
| OpenAI rate limits delay NLP processing at scale | MEDIUM | LOW | BullMQ queue with concurrency limits; batch processing mode |
| Credly API latency for certificate issuance | LOW | LOW | Async issuance; certificates confirmed within 15 min, not instant |
| Practice staff don't adopt mobile module viewing | MEDIUM | MEDIUM | PWA install prompt in onboarding; QR code poster in break room |
| H-Score perceived as unfair by practice owners | HIGH | MEDIUM | Full algorithm documentation at /scoring/algorithm; cohort segmentation from day 1 |

---

## Definition of "Development-Ready" Per Component

| Component | Definition |
|-----------|-----------|
| Morning Huddle OS | Practice can start a huddle in 1 tap, complete all 5 segments, and see completion recorded in dashboard within 5 minutes |
| H-Score Engine | Practice has a calculated H-Score with component breakdown visible in dashboard; score updates within 2 hours of new data arrival |
| CCA Bronze | Staff can complete Module 1 end-to-end on mobile; scenario scored; completion recorded; certificate issued via Credly within 15 min |
| Trust Transfer Module | NLP pipeline tags 100% of incoming reviews; trust transfer signals surface in huddle |
| Practice Dashboard | All dashboard data loads in < 2 seconds; H-Score, huddle status, cert rates, and opportunity visible without additional navigation |
