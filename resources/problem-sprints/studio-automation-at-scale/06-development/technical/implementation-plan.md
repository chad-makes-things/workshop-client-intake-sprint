# Technical Implementation Plan

**Tech Lead:** Development Team — Technical Lead Role
**Sprint source:** Studio Automation at Scale
**Date:** 2026-03-10
**Architecture source:** product-architecture.md (8 components, 7 ADRs)

---

## Stack Validation

| Layer | Recommended | Status | Notes |
|-------|-------------|--------|-------|
| LLM | Claude API (claude-sonnet-4-6) | ✅ Validated | Anthropic SDK available for Node/Python; streaming supported |
| Teams integration | Microsoft Bot Framework SDK v4 | ✅ Validated | Azure Bot Service handles registration; Node.js or C# SDK |
| Email integration | Microsoft Graph API (M365) | ✅ Validated | Supports email read/send via OAuth; webhook via change notifications |
| Design generation | Canva Connect API | ⚠️ Spike required | Template variable API in beta; verify image field support and PDF/X-1a export |
| DAM | Canva Enterprise brand kits | ⚠️ Spike required | Programmatic brand kit management via Connect API — verify per-practice isolation |
| Print vendor | Gelato API | ✅ Validated | REST API, Stripe-style auth, sandbox environment available |
| Patient comms | Weave API | ⚠️ Spike required | Requires existing Weave account per practice; verify Gen4 pilot practice coverage |
| Hosting | Azure App Service | ✅ Validated | Consistent with M365/Teams ecosystem; Bot Service lives in Azure |
| Database | PostgreSQL (Azure Database for PostgreSQL) | ✅ Validated | Practice profiles, request log, analytics events |
| Analytics | PostHog (self-hosted on Azure) | ✅ Validated | Event tracking; no PII sent to PostHog cloud |

---

## Database Schema

### Table: `practices`
```sql
CREATE TABLE practices (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name              VARCHAR(255) NOT NULL,
  platform          VARCHAR(20) NOT NULL CHECK (platform IN ('gen4', 'modis', 'sga')),
  canva_brand_kit_id VARCHAR(255),
  brand_health_score INT DEFAULT 0 CHECK (brand_health_score BETWEEN 0 AND 100),
  primary_contact_name VARCHAR(255),
  primary_contact_email VARCHAR(255),
  address_line1     VARCHAR(255),
  address_city      VARCHAR(100),
  address_state     CHAR(2),
  address_zip       VARCHAR(10),
  phone             VARCHAR(20),
  weave_account_id  VARCHAR(255),
  email_domains     TEXT[], -- e.g. {'townlakedental.com', 'tld-dental.com'}
  last_request_at   TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_practices_email_domains ON practices USING gin(email_domains);
CREATE INDEX idx_practices_platform ON practices(platform);
```

### Table: `requests`
```sql
CREATE TABLE requests (
  id                UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID NOT NULL REFERENCES practices(id),
  channel           VARCHAR(20) NOT NULL CHECK (channel IN ('email', 'teams', 'slack')),
  channel_thread_id VARCHAR(255), -- Teams thread ID or email message-id chain
  sender_email      VARCHAR(255) NOT NULL,
  status            VARCHAR(30) NOT NULL DEFAULT 'intake'
                    CHECK (status IN ('intake', 'brief_complete', 'generating',
                                      'practice_review', 'cd_review', 'approved',
                                      'deployed', 'escalated', 'rejected')),
  collateral_types  TEXT[], -- e.g. {'email_header', 'tv_slide', 'flyer_4x6'}
  brief_json        JSONB, -- structured brief: offer, cta, audience, deadline, etc.
  medium_challenge_sent BOOLEAN DEFAULT FALSE,
  medium_challenge_accepted VARCHAR(10), -- 'digital', 'print', 'both', null
  canva_design_ids  JSONB, -- {email_header: 'canvaId', tv_slide: 'canvaId', ...}
  revision_count    INT DEFAULT 0,
  cd_reviewer_id    VARCHAR(255),
  cd_reviewed_at    TIMESTAMPTZ,
  print_order_id    VARCHAR(255), -- Gelato order ID
  print_ordered_at  TIMESTAMPTZ,
  deployed_channels TEXT[], -- which channels assets were deployed to
  escalation_reason TEXT,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_requests_practice_id ON requests(practice_id);
CREATE INDEX idx_requests_status ON requests(status);
CREATE INDEX idx_requests_created_at ON requests(created_at DESC);
```

### Table: `conversations`
```sql
CREATE TABLE conversations (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id  UUID NOT NULL REFERENCES requests(id),
  role        VARCHAR(20) NOT NULL CHECK (role IN ('user', 'agent')),
  content     TEXT NOT NULL,
  channel     VARCHAR(20) NOT NULL,
  sent_at     TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_conversations_request_id ON conversations(request_id);
```

### Table: `templates`
```sql
CREATE TABLE templates (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            VARCHAR(100) NOT NULL,
  collateral_type VARCHAR(50) NOT NULL, -- 'business_card', 'flyer_4x6', 'tv_slide', etc.
  canva_template_id VARCHAR(255) NOT NULL,
  variable_fields JSONB NOT NULL, -- schema: [{name, type, required, default}]
  print_spec      JSONB, -- {width_in, height_in, bleed, color_profile, finish}
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

## API Endpoints

### Internal API (Agent → Backend)

All endpoints are internal — not exposed publicly. Auth via service-to-service JWT.

```
POST   /api/intake/email          # Process inbound email, return reply payload
POST   /api/intake/teams          # Process Teams message, return reply payload
GET    /api/practices/lookup      # ?email_domain=townlakedental.com → practice profile
GET    /api/practices/:id         # Full practice profile
PATCH  /api/practices/:id         # Update practice (admin only)
POST   /api/requests              # Create new request record
PATCH  /api/requests/:id          # Update request status / brief
POST   /api/requests/:id/generate # Trigger Canva generation
POST   /api/requests/:id/revise   # Apply text revision, regenerate
POST   /api/requests/:id/approve  # CD approval → trigger deployment
POST   /api/requests/:id/escalate # Route to human designer queue
POST   /api/requests/:id/print    # Submit Gelato print order
GET    /api/requests              # List requests (admin dashboard)
GET    /api/analytics/summary     # Dashboard KPIs
GET    /api/analytics/medium-split # Weekly medium split data
GET    /api/analytics/brand-health # Brand kit health by practice
```

### Webhook Endpoints (External → Backend)

```
POST   /webhooks/email/inbound    # M365 Graph API change notification
POST   /webhooks/teams/message    # Bot Framework activity endpoint
POST   /webhooks/gelato/order     # Gelato order status updates
```

---

## Service Architecture

### Service 1: Agent Orchestrator
**Responsibility:** Receives normalized request events from channel adapters. Manages conversation state. Calls Claude API. Decides routing (generate vs. escalate vs. ask follow-up). Returns reply payload to channel adapter.

**Key design decisions:**
- Conversation state stored in `conversations` table — not in-memory. Allows recovery if service restarts mid-conversation.
- Claude API called with sliding window of the last 10 conversation turns + practice context + system prompt
- System prompt includes: SGA brand voice guidelines, medium challenge script, escalation triggers, max 3 question rule
- Claude returns structured JSON for routing decisions, not free-form text that needs further parsing:
```json
{
  "action": "ask_questions | generate | escalate | medium_challenge",
  "questions": ["...", "..."],  // max 3
  "medium_challenge": "...",    // null if not triggered
  "brief": { "collateral_types": [...], "offer": "...", ... },
  "escalation_reason": null
}
```

### Service 2: Email Adapter
**Responsibility:** Polls or receives webhooks for new emails to `studio@sgadental.com`. Normalizes to standard request event. Sends reply emails via Graph API.

**Threading:** Uses the original email's `Message-ID` as `In-Reply-To` for all replies — keeps conversation in a single email thread.

**Attachment handling:** Inline images embedded as base64 in HTML email body for preview delivery. Print-ready PDFs sent as attachments with the approval confirmation.

### Service 3: Teams Adapter
**Responsibility:** Receives Bot Framework activity webhooks. Normalizes to standard request event. Sends replies to the correct Teams thread.

**Threading:** Uses `activity.conversation.id` + `activity.replyToId` to maintain thread context.

**Notification for CD review:** Uses Teams Adaptive Card format for the review notification — supports inline images and action buttons natively.

### Service 4: Canva Integration Service
**Responsibility:** Manages all Canva Connect API interactions: brand kit reads, template generation with variables, preview image export, print-ready PDF export.

**Rate limiting:** Canva Connect API rate limits TBD from spike. Build retry logic with exponential backoff.

**Preview generation:** Request PNG export at 800px width for practice preview. Request PDF/X-1a for print fulfillment (verify availability in spike SP-1).

### Service 5: Deployment Service
**Responsibility:** After CD approval, routes approved assets to the correct deployment targets.

**Routing logic:**
```
For each approved asset:
  if channel = 'patient_email' AND practice.weave_account_id IS NOT NULL:
    → Weave API: upload asset + schedule send
  if channel = 'tv_slide':
    → Email delivery: send as attachment with upload instructions (Phase 1)
  if channel = 'print' AND print_order_requested:
    → Gelato API: submit order
```

### Service 6: Analytics Collector
**Responsibility:** Writes analytics events to PostHog on key state transitions.

**Key events:**
- `request_created` (channel, practice_platform)
- `medium_challenge_sent` (collateral_type_requested)
- `medium_challenge_accepted` (acceptance: digital/print/both)
- `generation_started` (collateral_types, template_ids)
- `generation_completed` (duration_ms)
- `revision_requested` (revision_count)
- `cd_review_started` (time_since_request_created)
- `cd_review_completed` (duration_ms, outcome: approved/change/rejected)
- `asset_deployed` (channels: email/tv_slide/print)
- `print_order_placed` (collateral_type, quantity)

---

## Infrastructure Plan

```
Azure Resource Group: sga-studio-agent-prod

├── App Service Plan (Standard S2)
│   ├── Agent Orchestrator (Node.js 20)
│   ├── Email Adapter (Node.js 20)
│   └── Teams Adapter (Node.js 20)
│
├── Azure Bot Service
│   └── Teams bot registration + channel configuration
│
├── Azure Database for PostgreSQL (Flexible Server)
│   └── Primary database (practices, requests, conversations, templates)
│
├── Azure Service Bus
│   └── Request event queue (decouples adapters from orchestrator)
│
├── Azure Key Vault
│   └── Secrets: Claude API key, Canva API key, Gelato API key,
│              Weave API key, M365 app credentials
│
└── Azure Monitor + Application Insights
    └── Logs, alerts, performance tracking
```

**Environments:** dev, staging, prod. Each environment has its own resource group. Staging uses Canva sandbox + Gelato sandbox.

---

## Engineering Task Breakdown

### Sprint 1 (Weeks 1-2): Infrastructure + Adapters

| Task | Owner | Est. |
|------|-------|------|
| Azure resource group setup (dev + staging) | Backend | 1d |
| PostgreSQL schema + migrations | Backend | 1d |
| M365 email adapter: inbound email ingestion + reply | Backend | 3d |
| Teams Bot Framework registration + inbound handler | Backend | 2d |
| Practice lookup service (email domain → practice profile) | Backend | 1d |
| Spike SP-1: Canva Connect API capabilities | Tech Lead | 2d |
| Spike SP-2: M365 webhook latency test | Backend | 1d |
| Spike SP-3: Weave API access verification | Backend | 1d |
| Spike SP-4: Gelato API onboarding | Backend | 1d |
| Seed practice DB: 15 Gen4 pilot practices | Studio Admin | 1d |

### Sprint 2 (Weeks 3-5): Agent Core + Generation

| Task | Owner | Est. |
|------|-------|------|
| Agent Orchestrator: Claude API integration | Backend | 3d |
| System prompt: intake, medium challenge, escalation logic | Tech Lead + UX | 2d |
| Conversation state management (sessions, threading) | Backend | 2d |
| Canva Integration Service: brand kit read + template generation | Backend | 3d |
| Template library: load 8 Phase 1 templates into Canva + DB | Studio Designer | 3d |
| Preview delivery: email inline images, Teams attachments | Backend | 2d |
| Revision handling: variable swap + regeneration | Backend | 1d |
| Escalation routing: human designer queue notification | Backend | 1d |

### Sprint 3 (Weeks 6-8): Approval + Deployment + Analytics

| Task | Owner | Est. |
|------|-------|------|
| CD review: Teams Adaptive Card notification | Frontend | 2d |
| CD review: approve/change/reject action handlers | Backend | 1d |
| CD review: escalation at 2h + 4h | Backend | 1d |
| Weave API integration: asset upload + schedule send | Backend | 2d |
| Gelato API integration: order submission + confirmation | Backend | 2d |
| Print order one-click: approval email CTA + Gelato trigger | Backend | 1d |
| Analytics event pipeline (PostHog) | Backend | 2d |
| Studio Analytics Dashboard (frontend) | Frontend | 4d |
| Studio Admin interface: practice profile editor | Frontend | 3d |
| End-to-end integration test: full request lifecycle | Tech Lead | 3d |

### Sprint 4 (Week 9): Pilot Launch Prep

| Task | Owner | Est. |
|------|-------|------|
| Canva brand kits for 15 pilot practices (verify completeness) | Studio Admin | 3d |
| 15-practice email domain config in practice DB | Backend | 0.5d |
| CD review notification configuration (Marcus's Teams account) | Backend | 0.5d |
| Teams Bot deployment to Gen4 Teams workspace | Backend | 1d |
| `studio@sgadental.com` mailbox setup + M365 API auth | Backend | 1d |
| Smoke test with 3 practice managers (internal) | QA | 2d |
| Pilot launch to 15 Gen4 practices | All | 1d |

---

## Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Canva Connect API doesn't support PDF/X-1a export | Medium | High | Spike SP-1 first. Fallback: export high-res PNG and let Gelato accept PNG for print. |
| Weave API not available for all 15 pilot practices | Medium | Medium | Phase 1 success criteria accept manual TV slide delivery. Same fallback applies to Weave: deliver email asset as attachment. |
| Claude API rate limits cause slow response | Low | Medium | Implement request queue with backpressure. Add retry logic. Claude's rate limits are generous at current usage levels. |
| Practice managers don't adopt email channel; continue texting designer | Medium | High | The designer should reply to all direct messages with: "Got it — I'm going to route this through the Studio Agent so we can track it. One sec." Then submit the request themselves. This trains the behavior. |
| CD review becomes bottleneck (Marcus is unavailable) | Low | High | Backup reviewer configured in Week 3. 4-hour escalation is automatic. |
| Brand kit data for pilot practices is incomplete | Medium | Medium | Verify all 15 practice brand kits before pilot launch (Sprint 4, Week 9). Block launch if <12 practices are ready. |
