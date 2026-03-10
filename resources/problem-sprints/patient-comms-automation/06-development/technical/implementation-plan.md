# Implementation Plan: The Comms Engine

**Role**: Tech Lead
**Sprint**: Patient Communications Automation
**Date**: 2026-03-10
**Source Artifacts**: `04-architect/architecture.md`, `01-frame/map.md`, `05-validate/learnings.md`

---

## 1. Stack Validation

### NexHealth Synchronizer API — PMS Integration Layer

**Decision**: NexHealth as the exclusive PMS integration layer for appointment events, treatment plan triggers, and appointment write-back.

**Justification**: SGA's 260+ practices run 20+ different PMS systems — Dentrix, Eaglesoft, Open Dental, Denticon, Curve Dental, and others. Building direct integrations to each is years of work and an ongoing maintenance burden. NexHealth provides a single REST API that covers 20+ dental PMS systems and handles the PMS-specific quirks, version management, and authentication. More importantly for the Comms Engine, NexHealth supports real-time webhooks (not just polling), which is the requirement for the <60-second lead response target. The write-back API enables the Voice AI to book appointments directly into the PMS during a live call.

**API surface used**:
- Webhooks: `appointment.created`, `appointment.completed`, `appointment.cancelled`, `patient.form_submitted`, `treatment_plan.created`, `treatment_plan.updated`
- Read: `GET /locations/{id}/providers` (for voice AI scheduling), `GET /locations/{id}/availabilities` (real-time slot lookup)
- Write: `POST /locations/{id}/appointments` (booking write-back from Voice AI)

**Coverage gap**: Estimated 15–20% of SGA practices on PMS systems not supported by NexHealth. For these practices, build a lightweight adapter: office staff manually trigger events via a simple webhook endpoint, or the system falls back to daily PMS export parsing. Exact gap to be quantified in the Stage 1 PMS audit spike.

**Alternatives considered**: Direct PMS APIs (Dentrix Developer Program, Patterson API for Eaglesoft, Open Dental REST API) — feasible for the top 3 systems, but adds 3 separate integration maintenance streams. Clearinghouse APIs (Doctible, DrChrono) — limited webhook support. **Decision**: NexHealth covers the majority case; direct adapters for edge cases.

---

### Twilio (HIPAA-eligible, BAA) — Communication Transport

**Decision**: Twilio as the exclusive communication transport for SMS, MMS, voice, and email.

**Justification**: The Comms Engine's business case rests on $47/practice/month vs. $189–299/month for vendor platforms. Twilio commodity pricing gets us there. At 260 practices × ~200 messages/month average, monthly Twilio cost is approximately $3,000–5,000 total ($11–19/practice/month), well within the target. More importantly, Twilio is HIPAA-eligible with a Business Associate Agreement available — critical for dental patient communications. Using Twilio instead of a vendor platform means SGA owns the intelligence layer (the playbook, the templates, the learning engine). The vendor's "intelligence" stays with the vendor.

**Twilio products used**:
- **Twilio Messaging (A2P 10DLC)**: SMS/MMS outbound. All 260+ practice numbers must be registered under 10DLC for A2P compliance. Throughput: 10 msgs/sec per registered number (sufficient).
- **Twilio Voice + Media Streams**: Inbound call answering + Voice AI. Media Streams enables real-time audio streaming to the Claude API for conversational AI. Critical latency requirement: <500ms per turn.
- **Twilio SendGrid**: Transactional email (form fill responses, treatment follow-up email variants). HIPAA-eligible tier.
- **Twilio Verify**: Opt-in confirmation for A2P compliance (first SMS includes consent confirmation).

**Concern**: 10DLC registration at scale (260+ numbers) requires careful throughput management and carrier compliance review. Mitigation: dedicated Twilio account support engagement at Stage 1; evaluate Toll-Free SMS as an alternative for high-volume practices (higher throughput, faster registration).

**Alternatives considered**: Bandwidth (competitive pricing, HIPAA-eligible, but smaller ecosystem), AWS Pinpoint (cheaper but less flexible for voice AI), vendor platforms (TrueLark, Weave, RevenueWell — eliminated due to cost and IP ownership concerns).

---

### Claude API (Anthropic) — LLM for Message Generation and Voice AI

**Decision**: Claude as the primary LLM for message composition, voice conversation, and sentiment analysis.

**Justification**: The Comms Engine requires LLM capabilities in two modes: (1) non-real-time message generation (treatment follow-up texts, reactivation campaigns) where quality is paramount, and (2) real-time voice conversation where latency is paramount (<500ms per conversational turn for natural speech). Claude excels at both. For constrained message generation — warm dental communication, HIPAA-compliant, practice-specific personalization — Claude's instruction-following is best-in-class. For voice AI, Claude's streaming API enables low-latency conversational responses.

**Model selection**:
- `claude-sonnet-4-6`: Default for message generation (cost-efficient, high quality for structured prompts), sentiment analysis, form fill responses, rebooking sequences.
- `claude-opus-4-6`: High-value treatment follow-up personalization (>$15K cases) where the extra quality justifies cost; Voice AI during live calls (best conversational ability).

**Prompt architecture**:
- Message generation: structured prompt with `PracticeContext` (name, specialty, provider roster, tone guidelines), `PatientContext` (first name, treatment type, value, demographics tier, note-taker flags), `PlaybookContext` (touch number, cadence stage, network-optimized template), and HIPAA constraint instructions (no PHI in logs, no diagnosis language).
- Voice AI: system prompt with practice identity, provider schedule, booking instructions, escalation triggers. Stateful conversation with full turn history in context.

**Data handling**: No PHI passes through the Claude API beyond what is strictly necessary for personalization (patient first name, treatment type, provider name). PHI is never logged in API request/response logs. HIPAA compliance review required before production deployment.

**Alternatives considered**: GPT-4o (OpenAI) — comparable capability, but OpenAI's data handling for HIPAA healthcare use cases requires additional review and the BAA is a paid add-on. Fine-tuned smaller model — lower cost but lower quality for natural-sounding dental communication. **Decision**: Claude is the right capability/cost/compliance balance for this use case.

---

### Temporal — Workflow Orchestration for Long-Running Cadences

**Decision**: Temporal (self-hosted on AWS ECS) for all multi-touch communication workflow execution.

**Justification**: Treatment follow-up sequences span 30 days with 4 touch points. Reactivation campaigns run across weeks. Each workflow has complex branching: patient responds → cancel remaining touches; patient books → end sequence; patient opts out → terminate and record; message fails → retry with backoff. This is exactly what Temporal was built for: durable execution of long-running workflows with state persistence, automatic retry, sleep timers (Temporal can sleep a workflow for 30 days without consuming resources), and full audit visibility.

The alternative — cron jobs with a database scheduler — means building custom state machines, managing concurrency, handling retries, and debugging failed workflows with no visibility tooling. At 260 practices × hundreds of concurrent patient sequences, a fragile cron-based system becomes unmaintainable. Temporal's workflow visibility UI alone is worth the operational overhead during debugging.

**Workflow definitions** (one per playbook stage):
- `LeadResponseWorkflow`: fires on lead event; executes 3-touch sequence with 4-hour and 24-hour timers; branches on response.
- `TreatmentFollowUpWorkflow`: fires on treatment_plan_presented; executes 4- or 5-touch cadence with Day 3/7/14/30 timers; branches on booking, rejection, or non-response.
- `RebookingWorkflow`: fires on appointment_completed + no_future_appointment; executes 3-touch sequence with 30-min, Day 1, Day 3 timers.
- `ReactivationWorkflow`: monthly batch trigger; executes tiered sequence per patient segment with voice call or text cadences.

**Alternatives considered**: Celery + Redis (simpler but no durable execution, limited visibility), AWS Step Functions (managed but higher cost and HIPAA configuration complexity), custom PostgreSQL-based scheduler (too brittle for 30-day workflows at scale).

---

### PostgreSQL 16 (AWS RDS Multi-AZ) — Operational Data Store

**Decision**: PostgreSQL 16 as the operational database for all configuration, event logs, patient records, and playbook state.

**Justification**: All Comms Engine data is relational: practice configurations, agent instances, patient contact records (PHI-minimal — hashed identifiers), communication events (append-only audit log), playbook cadences, pending approvals, graduation state, A/B test assignments. PostgreSQL handles this comfortably with row-level security enabling multi-tenant access (practices see only their data), JSONB columns for semi-structured metadata (autonomy levels as JSON, campaign results as JSON), and built-in UUID support for event IDs.

The communication_events table is the HIPAA audit trail — it must be append-only (no updates, no deletes), with timestamps, and retained for 6 years per HIPAA requirements. PostgreSQL's WAL-based replication and RDS automated backups cover durability. Row-level security policies on practice_id columns enforce data isolation.

**Key design constraints**:
- PHI minimization: patient records store only hashed identifiers (SHA-256 of phone + email). Plain-text PHI (names, DOB) is never stored in this database — it is fetched from NexHealth at message generation time and not persisted.
- Append-only event log: `communication_events` has no UPDATE or DELETE grants; only INSERT.
- Encryption at rest: RDS encryption enabled (AES-256) for all tables.

**Alternatives considered**: MongoDB (flexible schema but weaker HIPAA audit trail story), CockroachDB (distributed but overkill at this scale), SQLite (insufficient for multi-practice concurrent writes).

---

### FastAPI (Python 3.12) — API Layer

**Decision**: FastAPI for the REST API serving webhooks, the practice approval interface, and HQ admin operations.

**Justification**: Python is the right language for this stack — it's the language of the Anthropic SDK (Claude API client), the NexHealth Python client, and typical ML/data pipelines. FastAPI provides automatic OpenAPI docs (critical for the backend/frontend handoff), async support for non-blocking I/O (Twilio webhook handlers must be fast — Twilio expects a response within 15 seconds), and Pydantic models for request/response validation that double as HIPAA data boundary enforcement (if a field isn't in the Pydantic model, it can't leak through).

**Service organization**: FastAPI app with router modules per domain — `/webhooks` (NexHealth, Twilio inbound), `/practices`, `/agent`, `/playbook`, `/communications`, `/approvals`, `/network`. Each router is an independent module with its own service layer.

**Alternatives considered**: Node.js/Express (familiar to frontend team, fast webhooks, but less alignment with ML/data stack), Django (heavier, more overhead), Go (fast webhooks but less alignment with Claude/ML tooling).

---

### React 18 + Vite + PWA — Frontend Applications

**Decision**: React 18 (with Vite build tooling) for both the practice-facing PWA (extending Practice Concierge) and the HQ Playbook Manager.

**Justification**: The Practice Concierge is already a React PWA — the Comms Engine approval interface is a new tab/module in that same app. Shared component library (shadcn/ui base, Concierge-specific extensions), shared auth (existing practice login), shared React Query data fetching layer. Building in the same framework means one codebase to maintain, one design system, one deployment pipeline for practice-facing code.

The HQ Playbook Manager is a separate React app (different users, different access patterns, different deployment cadence) but shares the same component primitives.

**State management**: Zustand (same as Practice Concierge for consistency). React Query for server state (approval lists, weekly summaries, practice health data).

**PWA requirements**:
- Service Worker: Workbox-based, caches daily digest for offline access, queues approval actions offline.
- Web Push: Firebase Cloud Messaging (FCM) for cross-platform push notification delivery.
- iOS Safari: works without native install prompt (manual Add to Home Screen flow).

---

### AWS (ECS Fargate + RDS + S3 + MSK) — Infrastructure

**Decision**: AWS as the cloud platform, ECS Fargate for containerized services, RDS for PostgreSQL, S3 for template/audit storage.

**Justification**: AWS offers HIPAA-eligible services with Business Associate Agreement coverage across the full stack: ECS, RDS, S3, Lambda, MSK. ECS Fargate eliminates server management — each service runs as a containerized task with auto-scaling. Per-practice agent isolation can be achieved at the service level (one ECS service per practice cluster) if needed, though Stage 1 will use a shared service with practice_id isolation.

**Kafka (AWS MSK)**: Event streaming between webhook receivers and workflow orchestrators. Decouples NexHealth/Twilio webhooks (high-frequency, must respond fast) from Temporal workflow triggers (can tolerate brief lag). Kafka's durability guarantees that no event is lost even if Temporal is temporarily unavailable. A 3-broker MSK cluster handles well below our volume.

**Cost estimate**:
- Stage 1 (5 pilot practices): ~$3,000–4,500/month (ECS tasks, RDS db.t3.medium, MSK small, Twilio messaging/voice)
- Stage 3 (50 practices): ~$5,000–7,500/month
- Stage 5 (260+ practices): ~$9,000–14,000/month total ($35–54/practice/month infrastructure cost at full scale — leaves room for the $47/practice target)

---

## 2. Database Schema

```sql
-- PRACTICES
CREATE TABLE practices (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    nex_health_location_id VARCHAR(100) NOT NULL UNIQUE,
    twilio_phone_number    VARCHAR(20),
    specialty       VARCHAR(50),   -- general | cosmetic | specialty | family
    timezone        VARCHAR(50) NOT NULL DEFAULT 'America/Chicago',
    quiet_hours_start TIME DEFAULT '20:00',
    quiet_hours_end   TIME DEFAULT '07:00',
    active          BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- AGENT INSTANCES
-- One per practice. Stores current autonomy levels per playbook stage.
CREATE TABLE agent_instances (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id             UUID NOT NULL REFERENCES practices(id),
    lead_response_level     VARCHAR(20) DEFAULT 'monitored'
                                CHECK (lead_response_level IN ('supervised','monitored','autonomous')),
    treatment_followup_level VARCHAR(20) DEFAULT 'supervised'
                                CHECK (treatment_followup_level IN ('supervised','monitored','autonomous')),
    rebooking_level         VARCHAR(20) DEFAULT 'monitored'
                                CHECK (rebooking_level IN ('supervised','monitored','autonomous')),
    reactivation_level      VARCHAR(20) DEFAULT 'supervised'
                                CHECK (reactivation_level IN ('supervised','monitored','autonomous')),
    active                  BOOLEAN DEFAULT TRUE,
    last_activity           TIMESTAMPTZ,
    created_at              TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (practice_id)
);

-- PATIENTS
-- Minimal PHI: hashed identifiers only. Plain-text names/DOB fetched from NexHealth at send time.
CREATE TABLE patients (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id             UUID NOT NULL REFERENCES practices(id),
    nex_health_patient_id   VARCHAR(100),
    phone_hash              VARCHAR(64),  -- SHA-256 of E.164 phone number
    email_hash              VARCHAR(64),  -- SHA-256 of lowercase email
    preferred_channel       VARCHAR(20) DEFAULT 'sms'
                                CHECK (preferred_channel IN ('sms','email','voice')),
    demographics_tier       VARCHAR(10) DEFAULT 'under_55'
                                CHECK (demographics_tier IN ('under_55','over_55')),
    opt_out_at              TIMESTAMPTZ,
    opt_out_channel         VARCHAR(20),
    created_at              TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (practice_id, nex_health_patient_id)
);

-- COMMUNICATION EVENTS (append-only HIPAA audit log)
-- No PHI in this table. No updates or deletes permitted.
CREATE TABLE communication_events (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id     UUID NOT NULL REFERENCES practices(id),
    patient_id      UUID REFERENCES patients(id),
    event_type      VARCHAR(50) NOT NULL,
    -- event_type values: inbound_call | missed_call | form_fill | sms_sent | sms_delivered
    --                    sms_replied | sms_failed | call_answered | call_booked | call_outcome
    --                    email_sent | email_opened | opt_out_received | sequence_started
    --                    sequence_completed | sequence_cancelled | approval_sent | approval_approved
    --                    approval_edited | approval_rejected | approval_auto_sent | graduation_recommended
    --                    graduation_approved | weekly_summary_sent | weekly_summary_opened
    stage           VARCHAR(30),
    -- stage values: lead_response | treatment_followup | rebooking | reactivation | admin
    channel         VARCHAR(20),
    direction       VARCHAR(10) CHECK (direction IN ('inbound','outbound')),
    status          VARCHAR(30),
    twilio_sid      VARCHAR(100),    -- Twilio message SID or call SID for correlation
    template_id     UUID,
    ab_test_id      UUID,
    touch_number    INTEGER,
    sequence_id     UUID,            -- links all events in a sequence together
    outcome         VARCHAR(50),
    -- outcome values: booked | responded_positive | responded_negative | no_response
    --                 opted_out | voicemail | transferred | compliance_flag
    outcome_at      TIMESTAMPTZ,
    revenue_attributed NUMERIC(10,2),
    created_at      TIMESTAMPTZ DEFAULT NOW()
    -- NO UPDATE/DELETE: enforced via role grants
);
CREATE INDEX idx_comm_events_practice_created ON communication_events (practice_id, created_at DESC);
CREATE INDEX idx_comm_events_patient ON communication_events (patient_id, created_at DESC);
CREATE INDEX idx_comm_events_sequence ON communication_events (sequence_id);

-- PLAYBOOK CADENCES
-- Per-practice cadence overrides. If no practice override, use template defaults.
CREATE TABLE playbook_cadences (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id     UUID REFERENCES practices(id), -- NULL = network default
    stage           VARCHAR(30) NOT NULL,
    touch_number    INTEGER NOT NULL,
    delay_days      INTEGER NOT NULL,
    channel         VARCHAR(20) NOT NULL,
    template_id     UUID NOT NULL,
    active          BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (practice_id, stage, touch_number)
);

-- PLAYBOOK TEMPLATES
CREATE TABLE playbook_templates (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(255) NOT NULL,
    stage           VARCHAR(30) NOT NULL,
    touch_number    INTEGER,
    body_template   TEXT NOT NULL,     -- Jinja2-style template with {{variables}}
    subject_template VARCHAR(255),     -- for email
    channel         VARCHAR(20) NOT NULL,
    variables       JSONB,             -- list of variable names used in template
    tone_notes      TEXT,
    specialty_variant VARCHAR(50),     -- NULL = all specialties
    value_tier      VARCHAR(20),       -- NULL | standard | high_value (>$15K)
    version         INTEGER DEFAULT 1,
    active          BOOLEAN DEFAULT TRUE,
    created_by      VARCHAR(100),
    compliance_reviewed_at TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ACTIVE SEQUENCES (Temporal workflow tracking)
CREATE TABLE active_sequences (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id         UUID NOT NULL REFERENCES practices(id),
    patient_id          UUID NOT NULL REFERENCES patients(id),
    stage               VARCHAR(30) NOT NULL,
    status              VARCHAR(20) DEFAULT 'active'
                            CHECK (status IN ('active','completed','cancelled','opted_out','booked')),
    temporal_workflow_id VARCHAR(255) UNIQUE,
    lead_source         VARCHAR(50),
    treatment_type      VARCHAR(100),
    value_estimate      NUMERIC(10,2),
    touch_count         INTEGER DEFAULT 0,
    last_touch_at       TIMESTAMPTZ,
    completed_at        TIMESTAMPTZ,
    booked_at           TIMESTAMPTZ,
    created_at          TIMESTAMPTZ DEFAULT NOW()
);

-- PENDING APPROVALS (supervised mode message queue)
CREATE TABLE pending_approvals (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id         UUID NOT NULL REFERENCES practices(id),
    sequence_id         UUID NOT NULL REFERENCES active_sequences(id),
    patient_id          UUID NOT NULL REFERENCES patients(id),
    stage               VARCHAR(30) NOT NULL,
    touch_number        INTEGER NOT NULL,
    proposed_channel    VARCHAR(20) NOT NULL,
    template_id         UUID NOT NULL,
    scheduled_send_at   TIMESTAMPTZ NOT NULL,
    auto_send_at        TIMESTAMPTZ NOT NULL,   -- 2 hours after digest sent
    digest_sent_at      TIMESTAMPTZ,
    status              VARCHAR(20) DEFAULT 'pending'
                            CHECK (status IN ('pending','approved','edited','rejected','auto_sent')),
    edited_content_enc  BYTEA,   -- AES-256 encrypted if office manager edited
    resolved_at         TIMESTAMPTZ,
    resolved_by         VARCHAR(50),  -- 'office_manager' | 'auto_send'
    created_at          TIMESTAMPTZ DEFAULT NOW()
);

-- GRADUATION STATE
CREATE TABLE graduation_state (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id             UUID NOT NULL REFERENCES practices(id),
    stage                   VARCHAR(30) NOT NULL,
    current_level           VARCHAR(20) NOT NULL,
    eval_period_start       DATE NOT NULL,
    eval_period_end         DATE NOT NULL,
    total_messages          INTEGER DEFAULT 0,
    edited_count            INTEGER DEFAULT 0,
    approved_count          INTEGER DEFAULT 0,
    patient_responses       INTEGER DEFAULT 0,
    compliance_flags        INTEGER DEFAULT 0,
    edit_rate               NUMERIC(5,4),
    approval_rate           NUMERIC(5,4),
    response_rate           NUMERIC(5,4),
    eligible_for_graduation BOOLEAN DEFAULT FALSE,
    recommended_at          TIMESTAMPTZ,
    graduated_at            TIMESTAMPTZ,
    updated_at              TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (practice_id, stage)
);

-- AB TESTS
CREATE TABLE ab_tests (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name                    VARCHAR(255) NOT NULL,
    stage                   VARCHAR(30) NOT NULL,
    touch_number            INTEGER,
    metric                  VARCHAR(50) NOT NULL,  -- response_rate | booking_rate
    control_template_id     UUID NOT NULL,
    treatment_template_id   UUID NOT NULL,
    practice_cohorts        JSONB,  -- {control: [practice_ids], treatment: [practice_ids]}
    min_sample_size         INTEGER DEFAULT 500,
    status                  VARCHAR(20) DEFAULT 'active',
    started_at              TIMESTAMPTZ DEFAULT NOW(),
    ended_at                TIMESTAMPTZ,
    winner                  VARCHAR(20),   -- 'control' | 'treatment' | 'no_winner'
    results                 JSONB,
    created_by              VARCHAR(100)
);
```

---

## 3. API Endpoint Reference

### Webhook Endpoints (high-priority, must respond in < 3 seconds)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/webhooks/nexhealth` | NexHealth event dispatcher (appointment, treatment plan, patient form events) |
| `POST` | `/webhooks/twilio/voice` | Twilio inbound call webhook — returns TwiML to connect Voice AI |
| `POST` | `/webhooks/twilio/voice/status` | Twilio call status updates (completed, no-answer, busy) |
| `POST` | `/webhooks/twilio/message/status` | Twilio SMS delivery status (delivered, failed, undeliverable) |
| `POST` | `/webhooks/twilio/message/inbound` | Inbound SMS reply from patient (responses, STOP, YES) |
| `POST` | `/webhooks/forms/{practice_id}` | Web form submission from practice website |

### Practice Agent API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/practices` | List all practices (HQ only) |
| `GET` | `/practices/{id}` | Get practice details + agent status |
| `PUT` | `/practices/{id}/config` | Update practice config (timezone, quiet hours, specialty) |
| `GET` | `/practices/{id}/agent` | Get agent instance autonomy levels + last activity |
| `PUT` | `/practices/{id}/agent/override` | HQ override: force autonomy level for a stage |

### Approvals API (supervised mode)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/practices/{id}/approvals/digest` | Today's pending approvals for digest |
| `POST` | `/practices/{id}/approvals/{approval_id}/decide` | Approve, edit, or reject a single message |
| `POST` | `/practices/{id}/approvals/bulk-approve` | Approve all pending messages |
| `GET` | `/practices/{id}/approvals/history` | Past digests with outcomes |

### Communications API

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/practices/{id}/sequences` | Active sequences (leads, treatment, rebooking, reactivation) |
| `GET` | `/practices/{id}/sequences/{seq_id}` | Sequence detail + touch history |
| `DELETE` | `/practices/{id}/sequences/{seq_id}` | Cancel an active sequence (HQ or office manager) |
| `GET` | `/practices/{id}/communications/events` | Communication event log (paginated, date-filtered) |

### Network & Analytics API (HQ only)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/network/metrics` | Network-wide KPIs by stage (response rates, conversion, revenue) |
| `GET` | `/network/practices/health` | Practice health grid data (autonomy levels + performance by practice) |
| `GET` | `/network/learning/report` | Latest Playbook Evolution Report |

### Playbook API (HQ only)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/playbook/cadences` | Network default cadences |
| `PUT` | `/playbook/cadences/{stage}/{touch}` | Update cadence definition |
| `GET` | `/playbook/templates` | List message templates |
| `POST` | `/playbook/templates` | Create new template |
| `PUT` | `/playbook/templates/{id}` | Update template |
| `POST` | `/playbook/templates/{id}/preview` | Generate sample messages using Claude |
| `POST` | `/playbook/templates/{id}/compliance-check` | Run HIPAA compliance check |
| `GET` | `/playbook/ab-tests` | List A/B tests |
| `POST` | `/playbook/ab-tests` | Create new A/B test |

---

## 4. Engineering Team Structure

### Stage 1 Team (5 pilot practices, 8 weeks)

| Role | FTE | Focus |
|------|-----|-------|
| Tech Lead | 1 | Architecture, infrastructure, code review, NexHealth + Twilio integration design |
| Backend Engineer | 2 | Webhook handlers, Lead Response Orchestrator, Voice AI service, Temporal workflows |
| Frontend Engineer | 1 | Practice approval interface (PWA, extends Concierge), basic HQ admin view |
| DevOps / Infrastructure | 0.5 | AWS ECS setup, RDS provisioning, MSK Kafka, HIPAA BAA configuration |
| QA | 0.5 | Integration testing, HIPAA compliance validation, Twilio + NexHealth sandbox testing |

### Stage 2–3 Team Additions

| Role | FTE | Focus |
|------|-----|-------|
| Backend Engineer | +1 | Treatment Follow-Up + Rebooking workflows, AI Note-Taker bridge |
| Data Engineer | 0.5 | Outcome attribution pipeline, graduation metrics, A/B test data |
| ML Engineer | 0.5 | Network Learning Engine (joins at Stage 4) |

---

## 5. Infrastructure Plan

```
AWS Region: us-east-1 (HIPAA-eligible)

VPC: Private subnets for all services. Public subnet only for ALB.

ECS Fargate Services:
  ├── comms-api (FastAPI — webhooks + REST API)
  │     CPU: 0.5vCPU / Mem: 1GB per task, 2-4 tasks auto-scaled
  ├── voice-ai-service (FastAPI — Twilio Voice + Claude streaming)
  │     CPU: 1vCPU / Mem: 2GB per task, 4-8 tasks (voice latency sensitive)
  ├── workflow-worker (Temporal workers — Temporal SDK)
  │     CPU: 0.5vCPU / Mem: 1GB, 4+ tasks (one per workflow type)
  └── temporal-server (Temporal orchestration service)
        CPU: 1vCPU / Mem: 2GB, 2 tasks

AWS MSK (Kafka):
  3-broker cluster, kafka.t3.small
  Topics: nexhealth-events, twilio-events, form-fill-events, comms-dispatch, outcome-events

AWS RDS (PostgreSQL 16):
  Instance: db.t3.medium (Stage 1), db.r6g.large (Stage 3+)
  Multi-AZ for production
  Automated backups: 7-day retention (extend to 2557 days / 7 years for HIPAA audit log)
  Encryption: enabled (AES-256)

AWS S3:
  Buckets: message-templates (Jinja2 template storage), audit-log-archive (long-term HIPAA retention)
  Server-side encryption: SSE-S3

Application Load Balancer:
  HTTPS only (TLS 1.2+), ACM certificate
  Routes: /webhooks/* → comms-api, /api/* → comms-api

Secrets Manager:
  NexHealth API credentials, Twilio Account SID + Auth Token, Anthropic API Key, DB password
  No secrets in environment variables or source code

CloudWatch:
  Alarms: API error rate >1%, webhook response time >5s, Kafka consumer lag, RDS CPU >80%
```

---

## 6. Risk Register

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|-----------|
| R-1 | NexHealth webhook reliability is insufficient for real-time lead response at scale | Medium | High | Stage 1 spike: measure webhook latency and reliability for each NexHealth-supported PMS. For practices on unreliable PMS connections, implement a parallel polling fallback (1-minute intervals) as a backstop. |
| R-2 | Twilio Voice + Claude latency exceeds 500ms, making Voice AI sound robotic | Medium | High | Stage 1 latency prototype: test Twilio Media Streams + Claude streaming end-to-end with actual dental conversation. If >500ms, evaluate caching common responses, reducing context window, or switching to a faster TTS provider. |
| R-3 | 10DLC carrier registration for 260+ practice numbers causes send throughput bottlenecks or registration delays | Medium | Medium | Engage Twilio enterprise support early (Stage 1). Pre-register all practice numbers before practices go live. Evaluate Toll-Free SMS as a higher-throughput alternative for multi-practice outreach. |
| R-4 | Practice staff in supervised mode fail to review the daily digest, causing auto-sends of messages they haven't approved | Low | Medium | Auto-send is by design (with notification), but: (a) default auto-send window is 2 hours after digest sent, not 30 minutes; (b) all auto-sends are flagged in the weekly summary; (c) if edit rate after auto-send is high, the graduation evaluator delays graduation — the system self-corrects. |
| R-5 | Patient opt-out is received on one channel (e.g., STOP via SMS) but not propagated to other channels (voice, email) | Low | High | Centralized opt-out registry in PostgreSQL. Any opt-out event (from any channel) sets `opt_out_at` on the patient record. All outreach checks this field before sending, regardless of channel. Tested as P0 compliance requirement. |
| R-6 | The AI note-taker bridge cannot access transcription data from the specific vendors deployed in SGA practices | High | Low | Note-taker bridge is a Stage 2 enhancement, not a Stage 1 requirement. Treatment follow-up works without it (PMS treatment plan webhook is sufficient). Bridge is additive context, not the trigger. Vendor audit happens in Stage 1 while core system is live. |
