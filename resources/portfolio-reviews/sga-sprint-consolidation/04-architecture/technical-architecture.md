# SGA Dental Partners — Technical Architecture

**Date**: 2026-03-14
**Author**: Tech Lead Agent
**Purpose**: Define the shared infrastructure layer, cross-workstream event contracts, architecture decision records, build sequence, and risk register for the consolidated 3-workstream platform.
**Addresses QC-1 Revision Items**: #3 (Twilio ownership), #4 (event contracts), #5 (compliance/audit), #7 (build sequence), #8 (Canva abstraction)
**Addresses QC-2/QC-3 Carry-Forward Items**: F-01 (non-PMS data sources), F-09 (breach notification / incident response), F-10 (disaster recovery / backup), Prototype Lead #3 (shared design system)

---

## Section 1: Shared Infrastructure Layer

All three workstreams share the following cross-cutting services. Each service is owned by the Platform Engineering team and consumed by workstream teams via well-defined APIs.

---

### 1.1 Practice Data Hub

**Responsibility**: Single source of truth for every practice in the SGA network. Canonical record for practice identity, configuration, staff roster, and operational metadata. Every workstream reads practice data from here; none maintain their own copy.

**Technology**: PostgreSQL (AWS RDS Multi-AZ) with a `shared.practices` schema.

**Rationale**: PostgreSQL is already the operational database for all three workstreams. A shared schema with row-level security ensures consistent practice data without cross-service duplication.

**Data Model**:
```sql
-- shared.practices
CREATE TABLE shared.practices (
  practice_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  external_id       TEXT UNIQUE,            -- SGA internal ID (e.g., "SGA-0142")
  name              TEXT NOT NULL,
  legal_name        TEXT,
  brand_universe    TEXT NOT NULL CHECK (brand_universe IN ('sga_legacy', 'gen4', 'modis')),
  practice_type     TEXT NOT NULL CHECK (practice_type IN ('gp', 'perio', 'cosmetic', 'pediatric', 'full_arch', 'multi_specialty')),
  location_type     TEXT CHECK (location_type IN ('rural', 'suburban', 'urban')),
  region_id         UUID REFERENCES shared.regions(region_id),
  address           JSONB NOT NULL,         -- {street, city, state, zip, lat, lng}
  timezone          TEXT NOT NULL DEFAULT 'America/New_York',
  phone             TEXT,
  email             TEXT,
  website_url       TEXT,
  pms_system        TEXT,                   -- e.g., 'dentrix', 'eaglesoft', 'open_dental'
  pms_connector     TEXT CHECK (pms_connector IN ('nexhealth', 'sikka_one')),
  nexhealth_id      TEXT,                   -- NexHealth location ID
  sikka_practice_id TEXT,                   -- Sikka ONE practice ID
  onboarding_status TEXT DEFAULT 'pending' CHECK (onboarding_status IN ('pending', 'enrolling', 'active', 'suspended')),
  enrollment_date   TIMESTAMPTZ,
  config            JSONB DEFAULT '{}',     -- practice-level feature flags, preferences
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);

-- shared.practice_staff
CREATE TABLE shared.practice_staff (
  staff_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  name              TEXT NOT NULL,
  role              TEXT NOT NULL CHECK (role IN ('dentist', 'hygienist', 'da', 'office_manager', 'front_desk', 'treatment_coordinator', 'other')),
  is_provider       BOOLEAN DEFAULT FALSE,
  npi               TEXT,                   -- National Provider Identifier (providers only)
  is_champion       BOOLEAN DEFAULT FALSE,  -- Culture/Content champion
  champion_type     TEXT[],                 -- ['culture', 'content'] or both
  contact_phone     TEXT,
  contact_email     TEXT,
  active            BOOLEAN DEFAULT TRUE,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);

-- shared.regions
CREATE TABLE shared.regions (
  region_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name              TEXT NOT NULL,
  director_name     TEXT,
  director_email    TEXT,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);
```

**API Surface** (Node.js/Fastify service):
```
GET    /api/v1/practices                    -- List all practices (paginated, filterable)
GET    /api/v1/practices/:id                -- Get practice detail
PUT    /api/v1/practices/:id                -- Update practice (emits practice.profile.updated)
POST   /api/v1/practices                    -- Create practice (emits practice.created)
GET    /api/v1/practices/:id/staff          -- List staff for practice
PUT    /api/v1/practices/:id/staff/:staffId -- Update staff member
GET    /api/v1/practices/by-region/:regionId -- List practices in region
GET    /api/v1/practices/:id/config         -- Get practice configuration
PATCH  /api/v1/practices/:id/config         -- Merge config updates
```

**Consumers**: WS1 (primary owner + consumer), WS2 (reads practice profiles for brand/template selection), WS3 (reads practice profiles for benchmarking segmentation and huddle personalization).

**Scaling Strategy**: Read replicas for dashboard queries. ClickHouse materialized views for analytics aggregations. Redis cache (5-minute TTL) for practice profiles since they change infrequently. At 260 practices, the data volume is small — scaling concern is query concurrency from 260 agent instances, handled by connection pooling (PgBouncer) and read replicas.

---

### 1.2 PMS Integration Service

**Responsibility**: Unified interface for all Practice Management System connectivity. Abstracts NexHealth (real-time, read-write) and Sikka ONE (batch, read-only) behind a single API. All workstreams consume PMS data through this service — none connect to NexHealth or Sikka ONE directly.

**Technology**: Node.js/TypeScript Fastify service + Temporal workflows for batch sync orchestration.

**Rationale**: Three of six original sprints independently designed PMS integrations. Centralizing eliminates connector duplication and ensures a single credential vault, single webhook receiver, and single event normalization layer.

**Architecture**:
```
                        ┌────────────────────────────┐
                        │   PMS Integration Service   │
                        │                              │
                        │  ┌────────────────────────┐ │
                        │  │  Unified PMS API        │ │
                        │  │  (practice-scoped)      │ │
                        │  └───────┬────────────────┘ │
                        │          │                   │
                        │  ┌───────┴────────────────┐ │
                        │  │  Router / Normalizer    │ │
                        │  │  (maps to canonical     │ │
                        │  │   event schema)         │ │
                        │  └───┬──────────┬─────────┘ │
                        │      │          │            │
                        │  ┌───┴───┐  ┌───┴───────┐   │
                        │  │NexHlth│  │Sikka ONE  │   │
                        │  │Adapter│  │Adapter    │   │
                        │  │       │  │           │   │
                        │  │Real-  │  │Nightly    │   │
                        │  │time   │  │batch sync │   │
                        │  │R/W    │  │Read-only  │   │
                        │  └───────┘  └───────────┘   │
                        └────────────────────────────┘
```

**API Surface**:
```
-- Appointments
GET    /api/v1/pms/:practiceId/appointments          -- List appointments (date range)
GET    /api/v1/pms/:practiceId/appointments/tomorrow  -- Tomorrow's schedule
POST   /api/v1/pms/:practiceId/appointments           -- Book appointment (NexHealth only)
PUT    /api/v1/pms/:practiceId/appointments/:id       -- Update appointment (NexHealth only)

-- Patients
GET    /api/v1/pms/:practiceId/patients               -- Search patients
GET    /api/v1/pms/:practiceId/patients/:id           -- Patient detail

-- Providers
GET    /api/v1/pms/:practiceId/providers              -- List providers

-- Events (webhook receiver)
POST   /api/v1/pms/webhooks/nexhealth                 -- NexHealth webhook endpoint
POST   /api/v1/pms/webhooks/sikka                     -- Sikka ONE webhook endpoint

-- Health
GET    /api/v1/pms/:practiceId/status                 -- Connector health check
```

**Event Emissions** (to Kafka):
- `pms.appointment.created` — new appointment booked
- `pms.appointment.updated` — appointment rescheduled/modified
- `pms.appointment.cancelled` — appointment cancelled
- `pms.appointment.completed` — appointment marked complete (procedure codes attached)
- `pms.patient.created` — new patient registered
- `pms.patient.updated` — patient record changed

**Consumers**: WS1 (real-time appointment events for funnel tracking, agent booking), WS2 (nightly appointment sync for content capture prompts), WS3 (appointment completion rates for H-Score Dental Intel component).

**Scaling Strategy**: One NexHealth connection per practice (260 webhook subscriptions). Sikka ONE batch sync runs as a Temporal workflow — 260 practices synced nightly in parallel batches of 50 (to respect Sikka rate limits). Redis cache for frequently-accessed appointment data (today's schedule, provider availability).

---

### 1.3 Communications Service

**Responsibility**: Centralized Twilio account management, number provisioning, message routing, opt-in/opt-out compliance, and delivery tracking. Both WS1 (patient communications) and WS2 (content capture SMS) send messages through this service. No workstream interacts with Twilio directly.

*Addresses QC-1 Item #3: Twilio channel ownership moved to shared infrastructure.*

**Technology**: Node.js/TypeScript Fastify service + Redis (rate limiting, delivery state) + PostgreSQL (message log, consent registry).

**Rationale**: WS1 sends patient-facing SMS/voice (appointment confirmations, follow-ups, reactivation). WS2 sends content capture prompts to champions via SMS/WhatsApp. Without centralization, two workstreams would independently manage Twilio numbers, risking duplicate sends to the same person, conflicting opt-out handling, and compliance gaps.

**Architecture**:
```
                    ┌─────────────────────────────────────────┐
                    │        Communications Service            │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  Message Router                      │ │
                    │  │  (purpose-based routing, dedup,      │ │
                    │  │   rate limiting, quiet hours)        │ │
                    │  └──────────┬──────────────────────────┘ │
                    │             │                             │
                    │  ┌──────────┴──────────────────────────┐ │
                    │  │  Consent & Compliance Engine         │ │
                    │  │  (opt-in/out registry, TCPA,         │ │
                    │  │   quiet hours, DNC list, HIPAA)      │ │
                    │  └──────────┬──────────────────────────┘ │
                    │             │                             │
                    │  ┌──────┬───┴────┬──────┬──────────────┐ │
                    │  │ SMS  │ Voice  │Email │  WhatsApp    │ │
                    │  │Twilio│ Twilio │ SES  │  Twilio      │ │
                    │  └──────┘────────┘──────┘──────────────┘ │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  Number Management                   │ │
                    │  │  (per-practice number assignment,    │ │
                    │  │   10DLC registration, A2P trust)    │ │
                    │  └─────────────────────────────────────┘ │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  Delivery Tracker                    │ │
                    │  │  (delivery status, bounce handling,  │ │
                    │  │   delivery receipts, analytics)      │ │
                    │  └─────────────────────────────────────┘ │
                    └─────────────────────────────────────────┘
```

**Data Model**:
```sql
-- shared.consent_registry
CREATE TABLE shared.consent_registry (
  consent_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone_number      TEXT NOT NULL,
  practice_id       UUID REFERENCES shared.practices(practice_id),
  channel           TEXT NOT NULL CHECK (channel IN ('sms', 'voice', 'email', 'whatsapp')),
  consent_status    TEXT NOT NULL CHECK (consent_status IN ('opted_in', 'opted_out', 'pending')),
  consent_source    TEXT NOT NULL,         -- 'intake_form', 'web_form', 'sms_keyword', 'voice_ivr'
  consent_timestamp TIMESTAMPTZ NOT NULL,
  revoke_timestamp  TIMESTAMPTZ,
  ip_address        INET,                  -- for web-based consent
  created_at        TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_consent_phone ON shared.consent_registry(phone_number, channel);

-- shared.message_log (audit trail — HIPAA required)
CREATE TABLE shared.message_log (
  message_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  direction         TEXT NOT NULL CHECK (direction IN ('outbound', 'inbound')),
  channel           TEXT NOT NULL CHECK (channel IN ('sms', 'voice', 'email', 'whatsapp')),
  purpose           TEXT NOT NULL,          -- 'patient_comms', 'content_capture', 'internal'
  source_workstream TEXT NOT NULL,          -- 'ws1', 'ws2', 'ws3'
  from_number       TEXT,
  to_number         TEXT,
  to_email          TEXT,
  message_body_hash TEXT,                   -- SHA-256 hash (never store PHI in plain text in logs)
  contains_phi      BOOLEAN DEFAULT FALSE,
  template_id       TEXT,
  twilio_sid        TEXT,                   -- Twilio message SID for tracking
  delivery_status   TEXT DEFAULT 'queued',  -- queued, sent, delivered, failed, bounced
  status_updated_at TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_msg_practice ON shared.message_log(practice_id, created_at DESC);
CREATE INDEX idx_msg_to ON shared.message_log(to_number, created_at DESC);

-- shared.phone_numbers
CREATE TABLE shared.phone_numbers (
  number_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone_number      TEXT UNIQUE NOT NULL,
  practice_id       UUID REFERENCES shared.practices(practice_id),
  purpose           TEXT NOT NULL CHECK (purpose IN ('patient_comms', 'content_capture', 'voice_ai')),
  twilio_sid        TEXT NOT NULL,
  a2p_campaign_id   TEXT,                   -- 10DLC campaign registration
  active            BOOLEAN DEFAULT TRUE,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);
```

**API Surface**:
```
-- Sending
POST   /api/v1/comms/send                    -- Send message (SMS, email, WhatsApp)
  Body: {
    practice_id: UUID,
    channel: "sms" | "email" | "whatsapp",
    to: string,                              -- phone or email
    purpose: "patient_comms" | "content_capture",
    source_workstream: "ws1" | "ws2",
    template_id?: string,
    template_vars?: object,
    body?: string,                           -- raw body (alternative to template)
    contains_phi: boolean,
    scheduled_at?: ISO8601                   -- for scheduled sends
  }

POST   /api/v1/comms/send-batch              -- Batch send (up to 1000 recipients)
POST   /api/v1/comms/voice/initiate          -- Initiate outbound voice call

-- Consent
GET    /api/v1/comms/consent/:phoneNumber     -- Check consent status across channels
POST   /api/v1/comms/consent/opt-in           -- Record opt-in
POST   /api/v1/comms/consent/opt-out          -- Record opt-out (immediate, all channels)

-- Numbers
GET    /api/v1/comms/numbers/:practiceId      -- List numbers assigned to practice
POST   /api/v1/comms/numbers/provision        -- Provision new number for practice

-- Delivery
GET    /api/v1/comms/status/:messageId        -- Delivery status
POST   /api/v1/comms/webhooks/twilio          -- Twilio status callback
POST   /api/v1/comms/webhooks/twilio/inbound  -- Inbound message handler
```

**Message Routing Rules**:
1. Check consent registry — reject if opted out
2. Check quiet hours (no messages before 8 AM or after 9 PM practice local time)
3. Check rate limits (max 3 messages per recipient per day across all purposes)
4. Check DNC list
5. Deduplicate (same recipient + same template within 24 hours = reject)
6. Route to channel transport (Twilio SMS, Twilio Voice, AWS SES, Twilio WhatsApp)
7. Log to `shared.message_log` with audit trail

**Consumers**: WS1 (patient appointment confirmations, follow-ups, reactivation, Voice AI), WS2 (content capture SMS prompts to champions, WhatsApp media collection).

**Scaling Strategy**: Twilio handles transport scaling. The service itself scales horizontally behind a load balancer. Redis handles rate limiting state. At peak: 260 practices x ~50 messages/day = ~13,000 messages/day — well within Twilio's A2P throughput. Number provisioning: one dedicated number per practice for patient comms, one shared short code for content capture prompts.

---

### 1.4 AI Service

**Responsibility**: Centralized LLM access layer providing rate limiting, token accounting, prompt routing, model abstraction, and response caching. All workstreams access Claude API and OpenAI Whisper through this service.

**Technology**: Node.js/TypeScript Fastify service + Redis (response cache, rate limiting) + PostgreSQL (token usage tracking, prompt registry).

**Rationale**: All three workstreams use Claude API. Centralizing prevents each team from independently managing API keys, hitting rate limits, and duplicating prompt engineering patterns. The abstraction layer also enables model switching without workstream code changes (see ADR-07).

**Architecture**:
```
                    ┌─────────────────────────────────────────┐
                    │            AI Service                     │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  Request Router                      │ │
                    │  │  (model selection, priority queue,    │ │
                    │  │   cost-tier routing)                  │ │
                    │  └──────────┬──────────────────────────┘ │
                    │             │                             │
                    │  ┌──────────┴──────────────────────────┐ │
                    │  │  Prompt Registry                     │ │
                    │  │  (versioned prompts, A/B testing,    │ │
                    │  │   system prompt library)              │ │
                    │  └──────────┬──────────────────────────┘ │
                    │             │                             │
                    │  ┌──────────┴──────────────────────────┐ │
                    │  │  Token Accountant                    │ │
                    │  │  (per-workstream budgets, usage       │ │
                    │  │   tracking, cost allocation)          │ │
                    │  └──────────┬──────────────────────────┘ │
                    │             │                             │
                    │  ┌──────┬───┴─────────────────────────┐  │
                    │  │Claude│  OpenAI Whisper              │  │
                    │  │ API  │  (audio transcription)       │  │
                    │  └──────┘─────────────────────────────┘  │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  Response Cache (Redis)              │ │
                    │  │  (deterministic requests cached      │ │
                    │  │   for 24h — e.g., template gen)      │ │
                    │  └─────────────────────────────────────┘ │
                    │                                           │
                    │  ┌─────────────────────────────────────┐ │
                    │  │  PHI Filter                          │ │
                    │  │  (strips PHI from prompts before     │ │
                    │  │   sending to LLM, re-injects after)  │ │
                    │  └─────────────────────────────────────┘ │
                    └─────────────────────────────────────────┘
```

**API Surface**:
```
-- Text generation
POST   /api/v1/ai/complete
  Body: {
    model: "claude-sonnet" | "claude-haiku" | "claude-opus",
    system_prompt_id?: string,               -- reference to prompt registry
    system_prompt?: string,                  -- or inline system prompt
    messages: [{ role, content }],
    max_tokens?: number,
    temperature?: number,
    workstream: "ws1" | "ws2" | "ws3",
    cost_tier: "low" | "standard" | "high",  -- routes to appropriate model
    contains_phi: boolean,                    -- triggers PHI filter
    cache_key?: string                        -- optional cache key
  }

-- Audio transcription
POST   /api/v1/ai/transcribe
  Body: multipart/form-data {
    audio: File,
    language?: string,
    workstream: "ws1" | "ws2" | "ws3"
  }

-- Prompt management
GET    /api/v1/ai/prompts                     -- List registered prompts
GET    /api/v1/ai/prompts/:id                 -- Get prompt by ID (with version)
POST   /api/v1/ai/prompts                     -- Register new prompt
PUT    /api/v1/ai/prompts/:id                 -- Update prompt (creates new version)

-- Usage
GET    /api/v1/ai/usage                       -- Token usage by workstream/period
GET    /api/v1/ai/usage/:workstream            -- Detailed usage for workstream
```

**PHI Filter Pipeline**:
1. Incoming request is scanned for PHI patterns (SSN, DOB, insurance ID, patient names when `contains_phi: true`)
2. PHI tokens are replaced with placeholders: `[PATIENT_NAME_1]`, `[DOB_1]`, etc.
3. Mapping is stored in Redis (request-scoped, 5-minute TTL)
4. LLM receives sanitized prompt
5. Response is post-processed to re-inject PHI values from the mapping
6. Mapping is deleted from Redis

**Consumers**: WS1 (message generation, agent reasoning, anomaly analysis, voice AI), WS2 (brief extraction, caption generation, content classification), WS3 (trust transfer NLP verification, call analysis via Whisper + Claude).

**Scaling Strategy**: Claude API rate limits are the binding constraint. Priority queue ensures patient-facing real-time requests (WS1 Voice AI, WS1 lead response) take precedence over batch workloads (WS2 caption generation, WS3 call analysis). Cost-tier routing sends low-complexity tasks to Claude Haiku, standard tasks to Claude Sonnet, complex reasoning to Claude Opus. Token budget alerts at 80% of monthly allocation per workstream.

---

### 1.5 Asset Registry

**Responsibility**: Centralized storage and metadata management for all brand assets, content media, generated designs, and shared templates. Single source of truth for "what visual assets exist for practice X."

**Technology**: AWS S3 (storage) + CloudFront CDN (delivery) + PostgreSQL (metadata catalog) + Cloudinary (image/video transformations).

**Rationale**: WS2 produces content assets that WS1 displays on storefronts and WS3 uses for recognition posts. A shared registry prevents asset duplication and ensures brand-compliant assets are discoverable across workstreams.

**Data Model**:
```sql
-- shared.assets
CREATE TABLE shared.assets (
  asset_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  brand_universe    TEXT NOT NULL,
  asset_type        TEXT NOT NULL CHECK (asset_type IN (
    'logo', 'photo', 'video', 'social_post', 'flyer', 'tv_slide',
    'before_after', 'testimonial', 'template', 'brand_guideline'
  )),
  title             TEXT,
  description       TEXT,
  s3_key            TEXT NOT NULL,
  s3_bucket         TEXT NOT NULL,
  cdn_url           TEXT,
  cloudinary_id     TEXT,                   -- for transformed versions
  mime_type         TEXT NOT NULL,
  file_size_bytes   BIGINT,
  dimensions        JSONB,                  -- {width, height, duration_seconds}
  metadata          JSONB DEFAULT '{}',     -- tags, AI-generated descriptions, etc.
  source_workstream TEXT NOT NULL,
  approval_status   TEXT DEFAULT 'pending' CHECK (approval_status IN ('pending', 'approved', 'rejected')),
  approved_by       TEXT,
  approved_at       TIMESTAMPTZ,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_asset_practice ON shared.assets(practice_id, asset_type);
```

**API Surface**:
```
POST   /api/v1/assets/upload                  -- Upload asset (returns presigned S3 URL)
GET    /api/v1/assets/:id                     -- Get asset metadata + CDN URL
GET    /api/v1/assets?practice_id=X&type=Y    -- Search assets
PUT    /api/v1/assets/:id/approve             -- Approve asset
DELETE /api/v1/assets/:id                     -- Soft delete
GET    /api/v1/assets/:id/transforms/:spec    -- Get transformed version (via Cloudinary)
```

**Consumers**: WS2 (writes assets from content pipeline), WS1 (reads assets for storefronts, pulls approved social posts), WS3 (reads brand assets for recognition posts and certificates).

**Scaling Strategy**: S3 scales indefinitely. CloudFront caches hot assets at edge. At 260 practices producing ~5 assets/week each, storage grows ~67,600 assets/year — manageable. Cloudinary handles on-the-fly image transformations (resize, crop, format conversion) to avoid storing multiple versions.

---

### 1.6 Event Bus (Kafka)

**Responsibility**: Cross-workstream event streaming. All events that flow between workstreams are published to Kafka topics. Each workstream owns its topics and publishes events; other workstreams subscribe to the topics they need.

**Technology**: Apache Kafka via AWS MSK (Managed Streaming for Kafka). Schema Registry (Confluent or AWS Glue) for Avro/JSON Schema enforcement.

**Rationale**: See ADR-04. Kafka provides durable, ordered, replayable event streams. Unlike point-to-point API calls, events allow workstreams to evolve independently — a new consumer can subscribe to existing events without modifying the producer.

**Topic Architecture**:
```
Namespace: sgadental

-- Practice Data Hub events
sgadental.practice.created
sgadental.practice.profile-updated
sgadental.practice.staff-updated
sgadental.practice.onboarding-status-changed

-- PMS events (from PMS Integration Service)
sgadental.pms.appointment-created
sgadental.pms.appointment-updated
sgadental.pms.appointment-cancelled
sgadental.pms.appointment-completed
sgadental.pms.patient-created
sgadental.pms.patient-updated

-- WS1: Practice Platform events
sgadental.ws1.funnel-event                   -- funnel stage transitions
sgadental.ws1.comms-message-sent
sgadental.ws1.comms-treatment-completed
sgadental.ws1.comms-reactivation-success
sgadental.ws1.funnel-metrics-weekly          -- weekly rollup
sgadental.ws1.agent-action-taken
sgadental.ws1.onboarding-status-changed

-- WS2: Content Engine events
sgadental.ws2.content-asset-created
sgadental.ws2.content-asset-approved
sgadental.ws2.content-asset-published
sgadental.ws2.content-engagement-updated
sgadental.ws2.capture-prompt-sent
sgadental.ws2.capture-media-received

-- WS3: Culture OS events
sgadental.ws3.hscore-calculated
sgadental.ws3.champion-assigned
sgadental.ws3.champion-updated
sgadental.ws3.training-completed
sgadental.ws3.huddle-completed
sgadental.ws3.certification-earned
sgadental.ws3.recognition-awarded
```

**Schema Registry**: Every topic has a registered JSON Schema. Producers cannot publish events that don't conform to the schema. Schema evolution follows forward-compatibility rules (new fields are optional with defaults; existing fields are never removed or renamed in the same major version).

**Configuration**:
- Retention: 7 days for high-volume topics (funnel events, comms messages), 30 days for low-volume topics (champion updates, H-Score)
- Partitions: Keyed by `practice_id` to ensure per-practice ordering. 16 partitions per topic (supports up to 16 concurrent consumers per consumer group).
- Replication factor: 3 (across AZs)
- Consumer groups: One per consuming service (e.g., `ws2-content-engine`, `ws3-culture-os`)

**Consumers**: All three workstreams produce and consume events. See Section 2 for complete event contracts.

**Scaling Strategy**: MSK handles scaling. At 260 practices with ~1,000 events/day each, throughput is ~260K events/day (~3 events/second average, ~50/second peak). This is well within MSK's capacity. Partitioning by practice_id ensures even distribution.

---

### 1.7 Auth & Identity Service

**Responsibility**: Single authentication and authorization layer across all platform interfaces. Manages user identities, role-based access control, API key management, and session state.

**Technology**: AWS Cognito (identity provider) + custom RBAC service (Node.js/TypeScript) + Redis (session cache).

**Rationale**: Three workstreams produce at least 5 distinct user-facing interfaces (Executive Dashboard, Practice Nerve Center, Content Engine Dashboard, Culture OS Dashboard, Mobile App). Users must authenticate once and access all interfaces they're authorized for. AWS Cognito integrates natively with the AWS stack and supports HIPAA-eligible workloads.

**Role Model**:
```
Roles:
  sga_admin           -- Full access to everything
  regional_director   -- Region-scoped access to all workstreams
  hq_operations       -- Network-wide operations (Practice Platform + Content Engine admin)
  practice_manager    -- Single practice: Practice Nerve Center + Content approval
  front_desk          -- Single practice: limited comms view
  champion            -- Single practice: content capture + culture dashboard
  content_team        -- HQ: Content Engine admin
  culture_team        -- HQ: Culture OS admin

Permissions (resource-based):
  practices:read, practices:write
  dashboard:executive, dashboard:practice
  comms:send, comms:approve, comms:admin
  content:capture, content:approve, content:admin
  culture:view, culture:admin
  reports:read, reports:export
```

**API Surface**:
```
POST   /api/v1/auth/login                     -- Cognito-backed login (returns JWT)
POST   /api/v1/auth/refresh                   -- Refresh token
POST   /api/v1/auth/logout                    -- Invalidate session
GET    /api/v1/auth/me                        -- Current user + permissions
GET    /api/v1/auth/users                     -- List users (admin)
POST   /api/v1/auth/users                     -- Create user
PUT    /api/v1/auth/users/:id/roles           -- Assign roles
POST   /api/v1/auth/api-keys                  -- Generate API key (service-to-service)
```

**Consumers**: All workstreams + all user-facing interfaces.

**Scaling Strategy**: Cognito scales automatically. RBAC checks are cached in Redis (per-session, 15-minute TTL). JWT validation is stateless at the edge (Cloudflare Workers validate JWT signature without hitting the auth service).

---

### 1.8 Compliance & Audit Layer

**Responsibility**: HIPAA compliance infrastructure including audit logging, PHI access tracking, encryption management, data retention enforcement, and BAA tracking. This is not a standalone service but a cross-cutting layer embedded in every service.

*Addresses QC-1 Item #5: HIPAA compliance as foundational architecture.*

**Technology**: Embedded middleware (Node.js/TypeScript) + AWS CloudTrail + PostgreSQL (audit log) + AWS KMS (encryption key management) + S3 (audit log archive).

**Architecture** (cross-cutting, not a standalone service):
```
Every service includes:
┌─────────────────────────────────────────────────────────────┐
│  Compliance Middleware (Fastify plugin)                       │
│                                                               │
│  1. Request Logger    — logs who accessed what, when          │
│  2. PHI Detector      — flags requests containing PHI        │
│  3. Consent Checker   — validates consent before patient ops │
│  4. Encryption Guard  — ensures PHI fields encrypted at rest │
│  5. Retention Timer   — marks records for expiry per policy  │
└─────────────────────────────────────────────────────────────┘
```

**Audit Log Schema**:
```sql
-- shared.audit_log (append-only, immutable)
CREATE TABLE shared.audit_log (
  log_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp         TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  actor_id          TEXT NOT NULL,           -- user ID or service account
  actor_type        TEXT NOT NULL CHECK (actor_type IN ('user', 'service', 'system')),
  actor_ip          INET,
  action            TEXT NOT NULL,           -- 'read', 'create', 'update', 'delete', 'export'
  resource_type     TEXT NOT NULL,           -- 'patient', 'appointment', 'message', 'practice'
  resource_id       TEXT,
  practice_id       UUID,
  contains_phi      BOOLEAN DEFAULT FALSE,
  phi_fields        TEXT[],                  -- which PHI fields were accessed: ['name', 'dob', 'phone']
  request_id        UUID,                    -- correlation ID across services
  service_name      TEXT NOT NULL,           -- which service generated this log
  outcome           TEXT NOT NULL CHECK (outcome IN ('success', 'failure', 'denied')),
  metadata          JSONB DEFAULT '{}'
);

-- Partition by month for efficient retention management
-- Index on actor_id, resource_type, timestamp for audit queries
CREATE INDEX idx_audit_actor ON shared.audit_log(actor_id, timestamp DESC);
CREATE INDEX idx_audit_resource ON shared.audit_log(resource_type, resource_id);
CREATE INDEX idx_audit_phi ON shared.audit_log(contains_phi, timestamp DESC) WHERE contains_phi = TRUE;
```

**Data Retention Policies**:
```
Patient records:          7 years after last visit (state-dependent, max applies)
Communication logs:       7 years (HIPAA minimum 6 years)
Audit logs:               7 years (archived to S3 Glacier after 1 year)
Content assets:           Indefinite (non-PHI)
Session data:             24 hours (Redis TTL)
Analytics events:         3 years in ClickHouse, then archived to S3
Consent records:          Indefinite (never delete consent history)
PHI in LLM cache:         5 minutes max (Redis TTL, then purged)
```

**Encryption Requirements**:
```
At rest:
  - PostgreSQL RDS: AES-256 (AWS managed, enabled by default)
  - S3: AES-256 SSE-S3 or SSE-KMS (PHI buckets use SSE-KMS)
  - ClickHouse: AES-256 encrypted EBS volumes
  - Redis: TLS + at-rest encryption enabled
  - Backups: Encrypted with AWS KMS customer-managed key

In transit:
  - All inter-service: mTLS (service mesh or AWS App Mesh)
  - All external APIs: TLS 1.2+ minimum
  - Twilio: TLS (Twilio enforces this)
  - NexHealth/Sikka: TLS + API key auth

PHI-specific:
  - Patient names, DOB, SSN, insurance IDs: encrypted at application level
    (AES-256-GCM) before storage, in addition to database-level encryption
  - Encryption keys rotated annually via AWS KMS
  - PHI never logged in plaintext to application logs
  - PHI never sent to LLM without PHI filter (see AI Service 1.4)
```

**BAA Requirements** (Business Associate Agreements):
```
Vendor                BAA Required   BAA Status      Notes
──────────────────────────────────────────────────────────────
AWS                   Yes            Available        Standard AWS BAA
Twilio                Yes            Available        Twilio HIPAA product tier
NexHealth             Yes            Required         Verify before contract
Sikka ONE             Yes            Required         Verify before contract
Anthropic (Claude)    Yes            Required         PHI filter reduces exposure
OpenAI (Whisper)      Yes            Required         Audio may contain PHI
Canva                 No             N/A              No PHI in design assets
Cloudinary            No             N/A              Before/after photos are PHI
                                                      — use de-identified versions
Vercel                Conditional    Verify           Only if PHI rendered server-side
Cloudflare            No             N/A              Edge caching of public sites only
Datadog               Yes            Available        If ingesting logs with PHI refs
```

**CRITICAL**: Cloudinary processes before/after photos which may constitute PHI (patient face, dental work). Resolution: all patient media processed through Cloudinary must be de-identified (no patient name in filename or metadata). Patient-media mapping is stored only in PostgreSQL, never in Cloudinary metadata.

**Consumers**: All services embed the compliance middleware. HQ Compliance team accesses audit logs via a read-only dashboard.

---

### 1.9 Workflow Engine

**Responsibility**: Orchestrates long-running, multi-step workflows that span multiple services. Provides durability (workflows survive service restarts), visibility (workflow state is inspectable), and compensation (failed steps can be rolled back).

**Technology**: Temporal (self-hosted on AWS ECS or Temporal Cloud).

**Rationale**: Multiple cross-service workflows exist: practice enrollment (2-week pipeline), content production (capture → process → approve → distribute), communication cadences (multi-day follow-up sequences), nightly PMS sync. BullMQ handles intra-service job queues; Temporal handles cross-service orchestration.

**Key Workflows**:
```
practice-enrollment       -- 12-step enrollment pipeline, 2-week duration
  Steps: create practice → harvest public data → provision Twilio numbers →
         configure PMS connector → setup agent instance → ...

content-production        -- capture → AI process → QC → approve → distribute
  Steps: receive media → classify → enhance (Cloudinary) → generate caption →
         create variants → route to approval → publish to channels

communication-cadence     -- multi-day patient follow-up sequences
  Steps: trigger event → wait(3 days) → send message → wait for response →
         if no response → wait(4 days) → send follow-up → ...

nightly-pms-sync          -- batch sync for Sikka ONE practices
  Steps: for each practice → fetch appointments → normalize → emit events →
         update practice data hub

hscore-daily-calc         -- daily H-Score recalculation
  Steps: fetch Cinnamon data → fetch Swell data → fetch Dental Intel data →
         calculate composite → emit hscore-calculated event
```

**Consumers**: All workstreams define workflows; all are executed by the shared Temporal cluster.

**Scaling Strategy**: Temporal workers scale horizontally. Separate worker pools per workstream to isolate resource consumption. Temporal Cloud is an option if self-hosting operational burden is too high.

---

### 1.10 Design Tool Abstraction Layer

**Responsibility**: Abstracts design tool operations behind a provider-agnostic interface, allowing the platform to use Canva today and swap providers in the future without workstream code changes.

*Addresses QC-1 Item #8: Canva single-vendor risk mitigation.*

**Technology**: Node.js/TypeScript adapter layer with provider interface.

**Interface**:
```typescript
// design-provider.interface.ts
interface DesignProvider {
  // Template operations
  listTemplates(brandUniverse: string, category: string): Promise<Template[]>;
  getTemplate(templateId: string): Promise<Template>;
  createFromTemplate(templateId: string, variables: TemplateVars): Promise<Design>;

  // Design operations
  getDesign(designId: string): Promise<Design>;
  updateDesign(designId: string, changes: DesignChanges): Promise<Design>;
  exportDesign(designId: string, format: ExportFormat): Promise<ExportResult>;

  // Asset operations
  uploadAsset(file: Buffer, metadata: AssetMetadata): Promise<DesignAsset>;
  listAssets(brandUniverse: string): Promise<DesignAsset[]>;
}

// canva-provider.ts (current implementation)
class CanvaProvider implements DesignProvider {
  // Implements via Canva Connect API
  // Manages 3 brand universe workspaces (SGA Legacy, Gen4, MODIS)
  // Handles Canva OAuth, rate limiting, webhook subscriptions
}

// Future: figma-provider.ts, adobe-express-provider.ts, etc.

type ExportFormat = 'png' | 'jpg' | 'pdf' | 'mp4' | 'svg';

interface Template {
  id: string;
  providerTemplateId: string;  // Canva template ID
  name: string;
  category: string;
  brandUniverse: string;
  dimensions: { width: number; height: number };
  variables: TemplateVariable[];
}

interface TemplateVariable {
  name: string;
  type: 'text' | 'image' | 'color';
  required: boolean;
  defaultValue?: string;
}
```

**API Surface**:
```
GET    /api/v1/design/templates                -- List templates (filterable by brand, category)
POST   /api/v1/design/generate                 -- Generate design from template + variables
GET    /api/v1/design/:id                      -- Get design status
GET    /api/v1/design/:id/export/:format        -- Export design
POST   /api/v1/design/assets                   -- Upload asset to design tool
```

**Consumers**: WS2 (primary — template generation, batch content production), WS1 (storefront asset generation).

---

### 1.11 Non-PMS Data Source Integration Service

*Addresses QC-2/QC-3 Carry-Forward Item F-01: Non-PMS data source integrations.*

**Responsibility**: Unified integration layer for all external data sources beyond PMS systems. Provides normalized APIs for phone systems, advertising platforms, AI phone agents, HR systems, and social platform analytics. Prevents each workstream from independently building one-off integrations to these sources.

**Technology**: Node.js/TypeScript Fastify service + Temporal workflows (for batch sync) + Redis (rate limiting, response cache).

**Rationale**: The platform depends on data from many systems beyond PMS. WS3's H-Score already integrates Cinnamon (phone analytics) and Swell (reviews). WS1 needs ad spend data for funnel attribution. WS2 needs social platform APIs for engagement metrics. AI phone agents are emerging as a practice-level tool that feeds data back to the platform. Without centralization, each workstream builds bespoke integrations — duplicating auth management, rate limiting, and data normalization.

**Integration Categories**:

```
┌─────────────────────────────────────────────────────────────┐
│         Non-PMS Data Source Integration Service              │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Unified Query API (source-agnostic)                     │ │
│  │  GET /api/v1/sources/:practiceId/:category/:metric       │ │
│  └──────────────────────┬──────────────────────────────────┘ │
│                         │                                     │
│  ┌──────────────────────┴──────────────────────────────────┐ │
│  │  Adapter Registry + Router                               │ │
│  └──┬─────┬─────┬──────┬──────┬──────┬─────────────────────┘ │
│     │     │     │      │      │      │                        │
│  ┌──┴──┐┌─┴──┐┌─┴───┐┌─┴───┐┌─┴───┐┌─┴──────────────────┐   │
│  │Phone││Ads ││AI   ││HR   ││Socl ││Future Adapters     │   │
│  │Sys  ││Plat││Phone││Sys  ││Plat ││(plug-in pattern)   │   │
│  └─────┘└────┘└─────┘└─────┘└─────┘└────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Adapter Inventory**:

| Category | Vendor | Integration Type | Data Direction | Primary Consumer |
|----------|--------|-----------------|----------------|-----------------|
| **Phone Systems** | RingCentral | REST API + Webhooks | Read | WS3 (call analytics for H-Score), WS1 (missed call detection) |
| **Phone Systems** | 8x8 | REST API | Read (batch) | WS3 (call analytics), WS1 (missed call detection) |
| **Phone Systems** | Weave | REST API + Webhooks | Read | WS3 (call analytics), WS1 (patient comms attribution) |
| **Ad Platforms** | Google Ads | Google Ads API v17 | Read | WS1 (funnel attribution — ad spend vs. bookings) |
| **Ad Platforms** | Meta Ads (Facebook/Instagram) | Marketing API | Read | WS1 (funnel attribution), WS2 (content-to-booking ROI) |
| **AI Phone Agents** | Sierra | REST API + Webhooks | Read/Write | WS1 (AI agent call outcomes, appointment booking confirmations) |
| **AI Phone Agents** | Agnes | REST API + Webhooks | Read/Write | WS1 (AI agent call outcomes, patient routing) |
| **HR Systems** | iSolved | REST API | Read | WS3 (staff roster sync, onboarding status, certifications) |
| **Social Platforms** | Instagram (Graph API) | REST API | Read | WS2 (engagement metrics — already in Event Contract 8) |
| **Social Platforms** | Facebook (Graph API) | REST API | Read | WS2 (engagement metrics, GBP reviews) |
| **Social Platforms** | Google Business Profile | REST API | Read | WS2 (review monitoring), WS1 (storefront health) |

**API Surface**:
```
-- Unified query interface
GET    /api/v1/sources/:practiceId/phone/calls           -- Call records (any phone system)
GET    /api/v1/sources/:practiceId/phone/missed           -- Missed calls (real-time or batch)
GET    /api/v1/sources/:practiceId/ads/spend              -- Ad spend by platform/campaign
GET    /api/v1/sources/:practiceId/ads/conversions        -- Conversion data by platform
GET    /api/v1/sources/:practiceId/ai-phone/sessions      -- AI phone agent call sessions
GET    /api/v1/sources/:practiceId/hr/staff               -- Staff roster from HR system
GET    /api/v1/sources/:practiceId/social/engagement      -- Social engagement metrics

-- Webhook receivers
POST   /api/v1/sources/webhooks/ringcentral              -- RingCentral call events
POST   /api/v1/sources/webhooks/weave                    -- Weave call/text events
POST   /api/v1/sources/webhooks/sierra                   -- Sierra AI agent call outcomes
POST   /api/v1/sources/webhooks/agnes                    -- Agnes AI agent call outcomes

-- Admin
GET    /api/v1/sources/:practiceId/status                -- Integration health for all sources
PUT    /api/v1/sources/:practiceId/config                -- Configure which adapters are active
```

**Event Emissions** (to Kafka):
- `sgadental.sources.call-completed` — phone call completed (normalized across RingCentral/8x8/Weave)
- `sgadental.sources.call-missed` — missed call detected
- `sgadental.sources.ai-agent-session-completed` — AI phone agent completed a call session
- `sgadental.sources.ad-spend-synced` — daily ad spend data synced from Google/Meta
- `sgadental.sources.staff-roster-synced` — HR roster synced from iSolved

**Practice-Level Configuration**: Not all practices use the same systems. Each practice's `config` in the Practice Data Hub specifies which phone system, ad accounts, and AI phone agent (if any) are active. The adapter registry only activates adapters relevant to each practice.

**Credential Management**: All vendor API keys and OAuth tokens are stored in AWS Secrets Manager, scoped per practice. Token refresh is handled by the service — workstreams never touch vendor credentials.

**Scaling Strategy**: Most adapters are batch-oriented (daily or hourly sync via Temporal workflows). Real-time adapters (RingCentral/Weave webhooks, Sierra/Agnes callbacks) are lightweight event receivers that normalize and emit to Kafka. At 260 practices, even with all adapters active, the volume is manageable — phone systems generate ~100-500 calls/day per practice, ad platforms sync daily, HR syncs weekly.

---

### 1.12 Breach Notification & Incident Response

*Addresses QC-2/QC-3 Carry-Forward Item F-09: HIPAA breach notification and incident response architecture.*

**Responsibility**: Provides the architectural framework, automated detection, and documented procedures for identifying, containing, investigating, and reporting PHI breaches as required by the HIPAA Breach Notification Rule (45 CFR 164.400-414).

**Technology**: Embedded detection rules (integrated with Compliance Middleware from Section 1.8) + AWS GuardDuty (anomaly detection) + PagerDuty (incident alerting) + PostgreSQL (incident log) + S3 (evidence preservation).

**Rationale**: HIPAA requires covered entities and their business associates to notify affected individuals, HHS, and (in some cases) the media within 60 days of discovering a breach of unsecured PHI. SGA processes PHI across multiple services — a breach in any service triggers notification obligations. Without a pre-built incident response architecture, teams would scramble ad hoc during a breach, risking missed deadlines and incomplete notifications.

**Automated Breach Detection Rules**:
```
Detection Layer 1: Compliance Middleware (real-time, per-request)
  - Unusual PHI access volume: > 3x baseline for any actor within 1 hour
  - PHI access outside business hours (configurable per practice timezone)
  - PHI bulk export: > 50 patient records accessed in single session
  - Failed authentication attempts: > 10 failures for same actor in 5 minutes
  - PHI access from unrecognized IP / geographic anomaly

Detection Layer 2: AWS GuardDuty (infrastructure-level)
  - Unauthorized API calls to S3 buckets containing PHI
  - Unusual database query patterns on RDS (PHI tables)
  - Credential exfiltration attempts
  - EC2/ECS instance compromise indicators

Detection Layer 3: Audit Log Analysis (batch, daily)
  - Temporal workflow: scans audit_log for anomalous patterns
  - Cross-references PHI access with role permissions (RBAC)
  - Identifies access patterns inconsistent with job function
  - Flags terminated employees with active sessions
```

**Incident Response Workflow**:
```
┌──────────────────────────────────────────────────────────────────┐
│  Phase 1: DETECTION (automated, 0-15 minutes)                    │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Detection rule triggers → create incident record →        │    │
│  │ PagerDuty alert to Security Lead + Tech Lead →            │    │
│  │ auto-freeze affected actor's access (if high severity)    │    │
│  └──────────────────────────────────────────────────────────┘    │
├──────────────────────────────────────────────────────────────────┤
│  Phase 2: CONTAINMENT (manual + automated, 15 min - 4 hours)    │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Incident Commander assigned → isolate affected system →   │    │
│  │ preserve evidence (snapshot logs, freeze audit trail) →   │    │
│  │ revoke compromised credentials → block attack vector      │    │
│  └──────────────────────────────────────────────────────────┘    │
├──────────────────────────────────────────────────────────────────┤
│  Phase 3: INVESTIGATION (manual, 4 hours - 7 days)               │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Determine scope: which PHI, how many individuals →        │    │
│  │ apply 4-factor risk assessment (per 45 CFR 164.402):     │    │
│  │   1. Nature and extent of PHI involved                    │    │
│  │   2. Unauthorized person who accessed PHI                 │    │
│  │   3. Whether PHI was actually acquired or viewed          │    │
│  │   4. Extent to which risk has been mitigated              │    │
│  │ Determine if breach exception applies (encryption,        │    │
│  │   unintentional access, good-faith belief)                │    │
│  └──────────────────────────────────────────────────────────┘    │
├──────────────────────────────────────────────────────────────────┤
│  Phase 4: NOTIFICATION (if breach confirmed, within 60 days)     │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ < 500 individuals: notify individuals + log with HHS     │    │
│  │   annually                                                │    │
│  │ ≥ 500 individuals: notify individuals + HHS within 60    │    │
│  │   days + prominent media outlets in affected states       │    │
│  │ Notification content: description of breach, PHI types,   │    │
│  │   steps individuals should take, contact info,            │    │
│  │   description of mitigation actions                       │    │
│  └──────────────────────────────────────────────────────────┘    │
├──────────────────────────────────────────────────────────────────┤
│  Phase 5: REMEDIATION & POST-MORTEM (ongoing)                    │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │ Fix root cause → update detection rules →                 │    │
│  │ post-mortem document (blameless) → update training →      │    │
│  │ retain all incident documentation for 6 years (HIPAA)     │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

**Incident Log Schema**:
```sql
-- shared.security_incidents (append-only)
CREATE TABLE shared.security_incidents (
  incident_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  severity          TEXT NOT NULL CHECK (severity IN ('critical', 'high', 'medium', 'low')),
  status            TEXT NOT NULL CHECK (status IN (
    'detected', 'investigating', 'contained', 'resolved', 'closed'
  )),
  detection_source  TEXT NOT NULL,           -- 'compliance_middleware', 'guardduty', 'audit_analysis', 'manual_report'
  detection_rule    TEXT,                     -- which rule triggered
  incident_commander TEXT,                   -- assigned responder
  affected_services TEXT[],                  -- ['pms_integration', 'communications_service']
  affected_practices UUID[],                 -- practice_ids
  phi_involved      BOOLEAN DEFAULT FALSE,
  phi_types         TEXT[],                  -- ['patient_name', 'dob', 'insurance_id']
  individuals_affected INTEGER,             -- count of affected individuals
  is_reportable_breach BOOLEAN,             -- determined after investigation
  breach_notification_deadline TIMESTAMPTZ, -- 60 days from discovery
  breach_notification_sent_at TIMESTAMPTZ,
  hhs_reported_at   TIMESTAMPTZ,
  timeline          JSONB DEFAULT '[]',     -- [{timestamp, action, actor, notes}]
  root_cause        TEXT,
  remediation       TEXT,
  evidence_s3_path  TEXT,                   -- S3 path to preserved logs/evidence
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_incident_status ON shared.security_incidents(status, severity);
CREATE INDEX idx_incident_phi ON shared.security_incidents(is_reportable_breach) WHERE is_reportable_breach = TRUE;
```

**HIPAA Documentation Requirements**: All incident records, investigation findings, risk assessments, notification copies, and remediation evidence must be retained for a minimum of 6 years from the date of creation or the date the policy was last in effect, whichever is later (45 CFR 164.530(j)). Incident records are archived to S3 Glacier after 1 year but remain retrievable.

**Consumers**: Security Lead (incident management), Legal (breach determination and notification), Platform Engineering (containment and remediation), HQ Compliance (audit and reporting).

---

### 1.13 Disaster Recovery & Backup Architecture

*Addresses QC-2/QC-3 Carry-Forward Item F-10: DR/backup architecture with RPO/RTO targets for healthcare data.*

**Responsibility**: Ensures platform recoverability in the event of infrastructure failure, data corruption, or regional outage. Defines backup strategies, replication architecture, and recovery procedures with RPO/RTO targets appropriate for healthcare operations.

**Rationale**: SGA's platform processes patient appointment data, communications, and clinical photos. An extended outage means missed patient follow-ups, stalled content pipelines, and blind spots in practice operations. HIPAA's Security Rule (45 CFR 164.308(a)(7)) requires a contingency plan including data backup, disaster recovery, and emergency mode operation.

**RPO/RTO Targets by Service Tier**:

| Tier | Services | RPO (Max Data Loss) | RTO (Max Downtime) | Rationale |
|------|----------|--------------------|--------------------|-----------|
| **Tier 1: Patient-Critical** | Communications Service, PMS Integration, Voice AI, Auth & Identity | **5 minutes** | **15 minutes** | Patient-facing communications and appointment booking cannot tolerate extended outages. Missed calls and failed appointment confirmations directly impact revenue and patient experience. |
| **Tier 2: Operations-Critical** | Practice Data Hub, AI Service, Event Bus (Kafka), Compliance Middleware | **15 minutes** | **1 hour** | Core data and intelligence layer. Workstreams degrade gracefully without these but cannot function for extended periods. |
| **Tier 3: Business-Important** | Asset Registry, Design Tool Abstraction, Content Engine pipeline, Culture OS (H-Score, Huddle) | **1 hour** | **4 hours** | Content and culture operations are important but not patient-facing. A 4-hour delay in content production or H-Score calculation is acceptable. |
| **Tier 4: Analytics / Batch** | ClickHouse (analytics), Workflow Engine (Temporal), Non-PMS Data Sources | **4 hours** | **8 hours** | Analytics and batch operations tolerate longer recovery. Temporal workflows resume from last checkpoint. |

**Backup Strategy by Data Store**:

```
PostgreSQL (RDS Multi-AZ) — ALL operational data
  - Automated backups: continuous (transaction log streaming)
  - Point-in-time recovery: any second within 35-day retention window
  - Cross-region replica: us-east-1 → us-west-2 (async, < 1 min lag)
  - Manual snapshots: weekly, retained 90 days
  - Backup encryption: AWS KMS customer-managed key

Kafka (MSK) — Event streams
  - Topic retention: 7-30 days (inherent replay capability)
  - MirrorMaker 2: cross-region replication to standby MSK cluster
  - Consumer offsets backed up: enables consumer group resumption

ClickHouse — Analytics data
  - Incremental backups to S3: every 4 hours
  - Full backup: weekly, retained 90 days
  - Rebuild strategy: can be rebuilt from Kafka event replay + backfill scripts

Redis (ElastiCache) — Cache and session data
  - Multi-AZ with automatic failover (< 30 seconds)
  - Daily snapshots retained 7 days
  - Accepted data loss: cache is ephemeral; sessions require re-login

S3 — Media assets, audit archives, evidence
  - Cross-region replication enabled for PHI buckets
  - Versioning enabled (protects against accidental deletion)
  - S3 Object Lock for audit log archives (WORM — immutable)
  - 99.999999999% durability (inherent)

Secrets Manager — API keys, OAuth tokens, encryption keys
  - Multi-region secret replication enabled
  - Automatic rotation schedules per vendor credential
```

**Disaster Recovery Architecture**:

```
PRIMARY REGION: us-east-1                    DR REGION: us-west-2
┌──────────────────────────┐                ┌──────────────────────────┐
│  ECS Services (active)    │                │  ECS Services (standby)   │
│  RDS Multi-AZ (primary)   │──async repl──→│  RDS Read Replica          │
│  MSK Kafka (active)       │──MirrorMaker─→│  MSK Kafka (standby)       │
│  ElastiCache (active)     │                │  ElastiCache (standby)     │
│  S3 (primary)             │──CRR────────→│  S3 (replica)              │
│  Secrets Manager          │──replication─→│  Secrets Manager           │
└──────────────────────────┘                └──────────────────────────┘
         │                                             │
         └──────────── Route 53 health checks ─────────┘
                       (automated DNS failover)
```

**Failover Procedure**:
1. Route 53 health checks detect primary region failure (3 consecutive failures, 30-second intervals)
2. DNS failover routes traffic to DR region (< 60 seconds for DNS propagation)
3. RDS replica in us-west-2 is promoted to primary (< 5 minutes)
4. ECS services in DR region scale from standby to active (pre-warmed containers, < 2 minutes)
5. MSK standby cluster becomes active; consumers resume from replicated offsets
6. Incident Commander verifies all services healthy in DR region
7. Post-recovery: replicate data back to primary region before failback

**Regular DR Testing**:
- **Monthly**: Automated backup restoration test — restore RDS snapshot to isolated environment, verify data integrity
- **Quarterly**: Tabletop DR exercise — walk through failover procedure with engineering and operations teams
- **Bi-annually**: Full failover drill — actually fail over to DR region during a maintenance window, run for 2 hours, fail back

**Emergency Mode Operations** (HIPAA 45 CFR 164.308(a)(7)(ii)(C)): If both primary and DR regions are unavailable, the platform operates in emergency mode:
- Communications Service: Twilio continues receiving inbound calls/SMS independently (Twilio is SaaS, not hosted in our infrastructure). Voicemail and missed-call notifications queue in Twilio until service restores.
- PMS systems: Practices continue using their PMS directly (NexHealth/Sikka are independent SaaS platforms).
- Manual fallback procedures documented for practice managers: paper-based appointment confirmation, manual content approval via email.

**Consumers**: Platform Engineering (DR operations), Security Lead (incident response integration), HQ Operations (business continuity).

---

### 1.14 Shared Frontend Design System

*Addresses QC-2/QC-3 Carry-Forward Prototype Lead #3: Shared design system as Phase 1 infrastructure.*

**Responsibility**: A shared component library, design token system, and pattern library used by all frontend applications across the platform. Ensures visual consistency, accelerates development, and prevents each workstream from independently building duplicate UI components.

**Technology**: React component library (TypeScript) + Storybook (documentation/visual testing) + Tailwind CSS design tokens + npm package (published to private registry).

**Rationale**: The platform produces at least 5 distinct frontend surfaces (Practice Nerve Center, Executive Dashboard, Content Engine Dashboard, Culture OS Dashboard, Mobile App). ADR-06 already noted that WS1's two interfaces are "mitigated by shared component library." Without a design system built in Phase 1, each workstream will independently create buttons, tables, charts, modals, and form components — leading to visual inconsistency, duplicated effort, and costly retroactive unification.

**Design System Structure**:
```
@sgadental/design-system (npm package)
│
├── tokens/
│   ├── colors.ts          -- Brand palette (SGA Legacy, Gen4, MODIS + platform neutral)
│   ├── typography.ts      -- Font scales, line heights, font families
│   ├── spacing.ts         -- Spacing scale (4px grid)
│   ├── breakpoints.ts     -- Responsive breakpoints
│   ├── shadows.ts         -- Elevation system
│   └── motion.ts          -- Animation timing, easing curves
│
├── components/
│   ├── primitives/        -- Button, Input, Select, Checkbox, Radio, Toggle, Badge
│   ├── layout/            -- Container, Grid, Stack, Sidebar, Card, Divider
│   ├── navigation/        -- NavBar, Sidebar, Tabs, Breadcrumb, Pagination
│   ├── data-display/      -- Table, DataGrid, StatCard, KPITile, ProgressBar, Avatar
│   ├── feedback/          -- Toast, Alert, Modal, Dialog, Spinner, Skeleton
│   ├── charts/            -- LineChart, BarChart, DonutChart, Sparkline, Heatmap
│   │                         (wrappers around Recharts or Nivo, styled to token system)
│   ├── forms/             -- FormField, SearchInput, DatePicker, FileUpload
│   └── domain/            -- PracticeSelector, StaffAvatar, HScoreBadge, FunnelStage
│                             (domain-specific composites used across workstreams)
│
├── patterns/
│   ├── page-layouts/      -- DashboardLayout, DetailPageLayout, SettingsLayout
│   ├── data-patterns/     -- FilterBar + DataGrid, KPI row + detail table
│   └── empty-states/      -- No data, loading, error, permission denied
│
├── hooks/
│   ├── useAuth.ts         -- Auth context consumer (wraps Auth & Identity Service)
│   ├── usePractice.ts     -- Current practice context
│   └── useBreakpoint.ts   -- Responsive breakpoint detection
│
└── tailwind-preset.ts     -- Tailwind CSS preset that maps tokens to Tailwind classes
```

**Brand Universe Support**: The token system supports three brand universes (SGA Legacy, Gen4, MODIS) via CSS custom properties. Components render identically but with brand-appropriate colors, logos, and typography. A `<BrandProvider universe="gen4">` context wrapper applies the correct token set.

**API (Component Usage)**:
```typescript
// Example: KPI tile used by WS1 Executive Dashboard, WS2 Content Dashboard, WS3 Culture Dashboard
import { KPITile, StatCard, DataGrid, DashboardLayout } from '@sgadental/design-system';

<DashboardLayout>
  <KPITile
    label="Network H-Score"
    value={78.5}
    trend="improving"
    trendDelta={2.3}
    format="score"
  />
</DashboardLayout>
```

**Storybook Documentation**: Every component is documented in Storybook with:
- Interactive examples with all prop variations
- Accessibility annotations (WCAG 2.1 AA compliance)
- Responsive behavior demonstrations
- Brand universe switching (toggle between SGA Legacy / Gen4 / MODIS themes)

**Versioning**: Semantic versioning via npm. Workstream teams pin to a major version. Breaking changes require major version bump + migration guide. Non-breaking additions (new components, new props) are minor versions.

**Consumers**: WS1 Practice Operations (React), WS1 Network Intelligence (React), WS2 Content Engine Dashboard (React), WS3 Culture OS Dashboard (React), WS2 Mobile App (React Native — shared tokens, separate component implementations).

---

## Section 2: Cross-Workstream Event Contracts

*Addresses QC-1 Item #4: All 10 integration points explicitly defined as event contracts.*

Every event flowing between workstreams is defined here with schema, trigger, latency requirement, and contract owner.

---

### Event Contract 1: Champion Roster → Dashboard

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws3.champion-assigned` / `sgadental.ws3.champion-updated` |
| **Source** | WS3: Culture OS |
| **Consumer(s)** | WS1: Practice Platform (Executive Dashboard), WS2: Content Engine |
| **Trigger** | Champion is assigned, updated, or deactivated in Culture OS |
| **Latency** | Near-real-time (< 5 minutes) |
| **Contract Owner** | WS3 team (Culture OS) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "champion.assigned | champion.updated | champion.deactivated",
  "timestamp": "2026-03-14T08:00:00Z",
  "practice_id": "uuid",
  "champion": {
    "staff_id": "uuid",
    "name": "string",
    "role": "hygienist | da | front_desk | office_manager",
    "champion_types": ["culture", "content"],
    "engagement_score": 85,
    "streak_days": 14,
    "certification_tier": "bronze | silver | gold | none",
    "contact_phone": "string",
    "contact_email": "string"
  },
  "previous_champion_staff_id": "uuid | null"
}
```

---

### Event Contract 2: H-Score → Executive Dashboard

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws3.hscore-calculated` |
| **Source** | WS3: Culture OS |
| **Consumer(s)** | WS1: Practice Platform (Executive Dashboard, Network Intelligence) |
| **Trigger** | Daily H-Score recalculation completes (6 AM ET daily) |
| **Latency** | Batch (daily) |
| **Contract Owner** | WS3 team (Culture OS) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "hscore.calculated",
  "timestamp": "2026-03-14T06:00:00Z",
  "practice_id": "uuid",
  "region_id": "uuid",
  "hscore": {
    "composite": 78.5,
    "components": {
      "cinnamon_phone": { "score": 82.0, "weight": 0.35, "calls_analyzed": 47 },
      "swell_reviews": { "score": 76.0, "weight": 0.45, "reviews_period": 30, "avg_rating": 4.6 },
      "dental_intel": { "score": 74.0, "weight": 0.20, "huddle_completion_rate": 0.85 }
    },
    "trend": "improving | stable | declining",
    "trend_delta": 2.3,
    "percentile_in_type": 72,
    "practice_type": "gp | perio | cosmetic | pediatric"
  },
  "certification_breakdown": {
    "gold": 1,
    "silver": 3,
    "bronze": 8,
    "none": 4
  },
  "period": "2026-03-14"
}
```

---

### Event Contract 3: Benchmarking Taxonomy → Shared

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws3.benchmark-taxonomy-updated` |
| **Source** | WS3: Culture OS |
| **Consumer(s)** | WS1: Practice Platform, WS2: Content Engine |
| **Trigger** | Benchmarking taxonomy updated (rare — quarterly or when new practice types added) |
| **Latency** | Batch (not time-sensitive) |
| **Contract Owner** | WS3 team (Culture OS) — co-owned with WS1 for taxonomy alignment |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "benchmark.taxonomy-updated",
  "timestamp": "2026-03-14T10:00:00Z",
  "taxonomy_version": "2.1",
  "practice_types": [
    {
      "type_id": "gp",
      "label": "General Practice",
      "subtypes": ["growth_gp", "standard_gp"],
      "benchmarks": {
        "target_hscore": 75,
        "target_show_rate": 0.88,
        "target_rebooking_rate": 0.72
      }
    }
  ],
  "location_types": ["rural", "suburban", "urban"],
  "effective_date": "2026-04-01"
}
```

---

### Event Contract 4: Champion Stories → Content Pipeline

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws3.champion-story-submitted` |
| **Source** | WS3: Culture OS |
| **Consumer(s)** | WS2: Content Engine |
| **Trigger** | Champion submits a "wow story" or culture moment via the Culture OS |
| **Latency** | Near-real-time (< 10 minutes) |
| **Contract Owner** | WS3 team (Culture OS) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "champion.story-submitted",
  "timestamp": "2026-03-14T14:30:00Z",
  "practice_id": "uuid",
  "champion_staff_id": "uuid",
  "story": {
    "story_id": "uuid",
    "story_type": "wow_moment | team_highlight | patient_testimonial | culture_win",
    "text_content": "string",
    "media_urls": ["s3://bucket/path"],
    "hospitality_behaviors": ["warm_greeting", "trust_transfer"],
    "suggested_channels": ["instagram", "internal_recognition"],
    "patient_consent_obtained": true
  }
}
```

---

### Event Contract 5: Patient Data → Content Personalization

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.pms.appointment-completed` (from shared PMS Integration Service) |
| **Source** | Shared Infrastructure (PMS Integration Service, originating from WS1 practice data) |
| **Consumer(s)** | WS2: Content Engine (for capture prompt personalization) |
| **Trigger** | Appointment marked complete in PMS (procedure codes attached) |
| **Latency** | Near-real-time (< 5 minutes) |
| **Contract Owner** | Platform Engineering (PMS Integration Service) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "appointment.completed",
  "timestamp": "2026-03-14T16:00:00Z",
  "practice_id": "uuid",
  "appointment": {
    "appointment_id": "uuid",
    "provider_name": "Dr. Smith",
    "procedure_codes": ["D6010", "D6065"],
    "procedure_category": "implant",
    "is_high_value": true,
    "patient_id_hash": "sha256-hash",
    "media_consent_status": "granted | denied | unknown"
  }
}
```

**Note**: Patient PII is NOT included. Only a hashed patient ID is transmitted. The Content Engine uses procedure category and consent status to generate appropriate capture prompts — it never receives patient names, DOB, or contact info via this event. If the Content Engine needs to send a capture prompt, it requests the champion's contact info from the Practice Data Hub (authorized via Auth & Identity).

---

### Event Contract 6: Approved Assets → Storefront

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws2.content-asset-approved` |
| **Source** | WS2: Content Engine |
| **Consumer(s)** | WS1: Practice Platform (Digital Storefront, Channel Sync Engine) |
| **Trigger** | Content asset passes HQ QC review and is approved for publication |
| **Latency** | Near-real-time (< 5 minutes) |
| **Contract Owner** | WS2 team (Content Engine) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "content.asset-approved",
  "timestamp": "2026-03-14T11:00:00Z",
  "practice_id": "uuid",
  "asset": {
    "asset_id": "uuid",
    "asset_type": "social_post | before_after | testimonial_video | photo",
    "title": "string",
    "description": "string",
    "cdn_url": "https://cdn.sgadental.com/assets/...",
    "thumbnail_url": "https://cdn.sgadental.com/assets/.../thumb",
    "dimensions": { "width": 1080, "height": 1080 },
    "mime_type": "image/jpeg",
    "brand_universe": "gen4",
    "captions": {
      "instagram": "string",
      "facebook": "string",
      "google_business": "string"
    },
    "tags": ["implant", "before_after", "dr_smith"],
    "storefront_eligible": true,
    "approved_by": "string",
    "approved_at": "2026-03-14T10:58:00Z"
  }
}
```

---

### Event Contract 7: Funnel Events → Content Performance

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws1.funnel-event` |
| **Source** | WS1: Practice Platform (Funnel OS) |
| **Consumer(s)** | WS2: Content Engine (ML booking attribution) |
| **Trigger** | Patient transitions between funnel stages |
| **Latency** | Near-real-time (< 1 minute) |
| **Contract Owner** | WS1 team (Practice Platform) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "funnel.stage-transition",
  "timestamp": "2026-03-14T09:30:00Z",
  "practice_id": "uuid",
  "patient_id_hash": "sha256-hash",
  "funnel": {
    "from_stage": "lead_generated | lead_connected | lead_booked | patient_showed | null",
    "to_stage": "lead_connected | lead_booked | patient_showed | production_completed",
    "source": "organic | paid_google | paid_meta | referral | reactivation",
    "source_detail": "google_lsa | meta_implant_campaign | ...",
    "service_type": "preventive | restorative | implant | full_arch | cosmetic",
    "estimated_value": 4500.00,
    "attribution": {
      "last_content_id": "uuid | null",
      "last_content_channel": "instagram | facebook | google_business | null",
      "days_since_content_exposure": 3
    }
  }
}
```

---

### Event Contract 8: Social Engagement → Dashboard

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws2.content-engagement-updated` |
| **Source** | WS2: Content Engine |
| **Consumer(s)** | WS1: Practice Platform (Executive Dashboard, Practice Nerve Center) |
| **Trigger** | Social engagement metrics polled from platform APIs (every 6 hours) |
| **Latency** | 6-hour batch |
| **Contract Owner** | WS2 team (Content Engine) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "content.engagement-updated",
  "timestamp": "2026-03-14T12:00:00Z",
  "practice_id": "uuid",
  "period": "2026-03-14",
  "engagement": {
    "total_posts_period": 12,
    "total_reach": 8450,
    "total_impressions": 15200,
    "total_engagements": 342,
    "engagement_rate": 0.0225,
    "by_channel": [
      {
        "channel": "instagram",
        "posts": 5,
        "reach": 3200,
        "engagements": 180,
        "top_post_id": "uuid"
      },
      {
        "channel": "facebook",
        "posts": 4,
        "reach": 2800,
        "engagements": 95,
        "top_post_id": "uuid"
      },
      {
        "channel": "google_business",
        "posts": 3,
        "reach": 2450,
        "engagements": 67,
        "top_post_id": "uuid"
      }
    ],
    "brand_health_score": 82,
    "content_health_score": 71
  }
}
```

---

### Event Contract 9: Training Completion → Readiness Indicators

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws3.training-completed` |
| **Source** | WS3: Culture OS |
| **Consumer(s)** | WS1: Practice Platform (Executive Dashboard — readiness indicators) |
| **Trigger** | Staff member completes a CCA certification module or earns a tier |
| **Latency** | Near-real-time (< 5 minutes) |
| **Contract Owner** | WS3 team (Culture OS) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "training.completed",
  "timestamp": "2026-03-14T15:00:00Z",
  "practice_id": "uuid",
  "staff_id": "uuid",
  "training": {
    "module_id": "string",
    "module_name": "Trust Transfer Fundamentals",
    "certification_tier": "bronze | silver | gold",
    "is_tier_completion": true,
    "tier_earned": "bronze",
    "credly_badge_id": "string | null",
    "completion_score": 92,
    "time_spent_minutes": 45
  },
  "practice_readiness": {
    "total_staff": 16,
    "certified_staff": 12,
    "certification_rate": 0.75,
    "tier_distribution": { "gold": 1, "silver": 3, "bronze": 8, "none": 4 }
  }
}
```

---

### Event Contract 10: Onboarding Status → Activation Triggers

| Field | Value |
|-------|-------|
| **Event Name** | `sgadental.ws1.onboarding-status-changed` |
| **Source** | WS1: Practice Platform |
| **Consumer(s)** | WS3: Culture OS (activation triggers for culture onboarding) |
| **Trigger** | Practice transitions between onboarding stages |
| **Latency** | Near-real-time (< 5 minutes) |
| **Contract Owner** | WS1 team (Practice Platform) |

**Payload Schema**:
```json
{
  "event_id": "uuid",
  "event_type": "onboarding.status-changed",
  "timestamp": "2026-03-14T10:00:00Z",
  "practice_id": "uuid",
  "onboarding": {
    "previous_status": "enrolling",
    "new_status": "active",
    "milestone": "pms_connected | comms_live | storefront_live | fully_active",
    "enrollment_start_date": "2026-03-01",
    "days_in_enrollment": 14,
    "completed_steps": [
      "practice_created",
      "public_data_harvested",
      "pms_connected",
      "twilio_provisioned",
      "agent_configured",
      "storefront_deployed"
    ],
    "pending_steps": []
  }
}
```

**WS3 Consumption Logic**: When `new_status` = `"active"` and `milestone` = `"fully_active"`, the Culture OS triggers:
1. Champion assignment workflow
2. Morning Huddle activation for the practice
3. CCA Academy enrollment for staff
4. Initial H-Score baseline calculation

---

### Event Contract Summary Matrix

| # | Event | Source | Consumer(s) | Latency | Owner |
|---|-------|--------|-------------|---------|-------|
| 1 | `ws3.champion-assigned/updated` | WS3 | WS1, WS2 | < 5 min | WS3 |
| 2 | `ws3.hscore-calculated` | WS3 | WS1 | Daily batch | WS3 |
| 3 | `ws3.benchmark-taxonomy-updated` | WS3 | WS1, WS2 | Batch | WS3/WS1 |
| 4 | `ws3.champion-story-submitted` | WS3 | WS2 | < 10 min | WS3 |
| 5 | `pms.appointment-completed` | Shared | WS2 | < 5 min | Platform Eng |
| 6 | `ws2.content-asset-approved` | WS2 | WS1 | < 5 min | WS2 |
| 7 | `ws1.funnel-event` | WS1 | WS2 | < 1 min | WS1 |
| 8 | `ws2.content-engagement-updated` | WS2 | WS1 | 6-hour batch | WS2 |
| 9 | `ws3.training-completed` | WS3 | WS1 | < 5 min | WS3 |
| 10 | `ws1.onboarding-status-changed` | WS1 | WS3 | < 5 min | WS1 |

---

## Section 3: Architecture Decision Records (ADRs)

---

### ADR-01: NexHealth + Sikka ONE Dual-PMS Strategy

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead, Product Strategist

**Context**:
Four of six original sprints independently designed PMS integrations using different approaches:
- Sprint 4 (Funnel OS): Direct Airbyte connectors for Dentrix, Eaglesoft, Open Dental (3 systems, read-only)
- Sprint 5 (Comms Engine): NexHealth Synchronizer (20+ systems, read-write)
- Sprint 6 (Content Concierge): Sikka ONE API (400+ systems, read-only)
- Sprint 1 (Studio Automation): Planned direct PMS triggers for 3 systems

SGA operates 260+ practices running at least 8 distinct PMS systems. The Comms Engine requires read-write access (Voice AI must book appointments directly). The Content Engine requires nightly appointment schedules for capture prompts. The Funnel OS requires real-time event streams for funnel tracking.

**Decision**:
Adopt a dual-layer PMS strategy:
1. **NexHealth (primary, real-time)**: All read-write operations. NexHealth provides a universal API covering 20+ PMS systems with real-time webhooks and write capability. It is the only vendor offering both real-time event streams AND appointment booking APIs across multiple PMS platforms.
2. **Sikka ONE (secondary, batch)**: Read-only appointment data for practices whose PMS is not covered by NexHealth. Sikka ONE's 400+ system coverage ensures no practice is excluded. Nightly batch sync via Temporal workflow.

Both are consumed through the shared PMS Integration Service. No workstream connects directly to either vendor.

**Consequences**:
- (+) Single API surface for all PMS operations regardless of backend connector
- (+) Write capability (appointment booking) via NexHealth for Voice AI and automated scheduling
- (+) 400+ PMS coverage ensures no practice is excluded from content capture and analytics
- (+) Eliminated 4 independent connector approaches, reducing maintenance surface from ~12 connectors to 2 vendor integrations
- (-) Two vendor relationships to manage (two BAAs, two billing accounts, two support channels)
- (-) Data normalization complexity — NexHealth and Sikka ONE use different schemas that must be mapped to a canonical model
- (-) Practices on Sikka-only coverage lack real-time events (batch-only)

**Alternatives Considered**:
1. **NexHealth only**: Would leave ~15% of practices without PMS connectivity (those on niche PMS systems NexHealth does not support). Unacceptable for a network-wide platform.
2. **Sikka ONE only**: No write capability. Cannot book appointments from Voice AI. Cannot get real-time event streams for funnel tracking.
3. **Direct connectors (Airbyte)**: Maximum control but massive maintenance burden — each PMS requires a custom connector. Not viable for 8+ PMS systems across 260 practices.
4. **Henry Schein One / Planet DDS APIs**: Limited to their own PMS ecosystems. Not universal.

---

### ADR-02: Twilio as Shared Infrastructure Service

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead, Product Strategist

**Context**:
Two workstreams independently chose Twilio for different purposes:
- WS1 (Practice Platform / Comms Engine): Patient-facing SMS, MMS, voice calls. HIPAA-eligible with BAA. Supports Voice AI (Twilio Voice + Media Streams for real-time audio).
- WS2 (Content Engine): SMS/WhatsApp capture prompts to champions. Sends daily procedure-matched capture briefs.

Without centralization, each workstream would manage its own Twilio account, phone numbers, and compliance rules. A patient who is also a champion could receive messages from two different numbers for the same practice. Opt-out handling could be inconsistent — opting out of WS1 patient messages might not stop WS2 capture prompts (different number, different Twilio project).

**Decision**:
Centralize all Twilio operations in a shared Communications Service (Section 1.3). Single Twilio account, single number inventory, unified consent registry. Both workstreams send messages via the Communications Service API — neither interacts with Twilio directly.

Message routing is purpose-based:
- `patient_comms`: Routed through WS1's message generation and compliance rules
- `content_capture`: Routed through WS2's capture prompt logic
- Both share opt-out compliance, quiet hours, and rate limiting

**Consequences**:
- (+) Single opt-in/opt-out registry — TCPA and HIPAA compliance is centralized and consistent
- (+) Single phone number per practice (or one per purpose with clear routing) — patients see a consistent sender
- (+) Unified delivery tracking and analytics across all message types
- (+) Single Twilio BAA covering all uses
- (+) Cross-workstream deduplication — prevents bombarding a person with messages from multiple sources
- (-) Coupling — both workstreams depend on the Communications Service; outage affects both
- (-) Coordination overhead — changes to message routing rules require cross-team alignment
- (-) The Communications Service is a critical path service requiring high availability (99.9%+)

**Alternatives Considered**:
1. **Separate Twilio accounts per workstream**: Simpler operationally per team, but creates compliance gaps (duplicate opt-out lists, inconsistent quiet hours, potential patient confusion from multiple numbers).
2. **AWS SNS + Pinpoint instead of Twilio**: Lower cost but no Voice AI support, weaker HIPAA compliance tooling, no WhatsApp support.
3. **Shared Twilio account without abstraction service**: Both workstreams use the same Twilio account but call Twilio APIs directly. Fails to centralize compliance logic and number management.

---

### ADR-03: HIPAA Compliance as Foundational Layer

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead, Product Strategist, (requires Legal review)

**Context**:
SGA Dental Partners processes Protected Health Information (PHI) including patient names, dates of birth, appointment records, treatment plans, insurance IDs, and clinical photos. HIPAA's Security Rule requires administrative, physical, and technical safeguards. HIPAA's Privacy Rule requires minimum necessary access and patient consent for communications.

The QC-1 review (Item #5) identified that the consolidation architecture had no explicit compliance or audit layer. Compliance was mentioned in individual sprints (Sprint 5's Compliance Auditor) but was not designed as a shared concern.

**Decision**:
Embed HIPAA compliance as a cross-cutting infrastructure layer, not an afterthought bolted onto individual workstreams:

1. **Compliance Middleware**: Every Fastify service includes a compliance plugin that logs access, detects PHI, validates consent, and enforces encryption (see Section 1.8).
2. **Audit Log**: Immutable, append-only audit log capturing every PHI access with actor, resource, timestamp, and outcome.
3. **PHI Filter in AI Service**: All LLM requests containing PHI are sanitized before transmission to Anthropic/OpenAI servers. PHI is replaced with placeholders, processed, and re-injected in the response.
4. **BAA Coverage**: Every vendor touching PHI or PHI-adjacent data must have an executed BAA before production use.
5. **Encryption**: Double encryption for PHI — database-level (RDS encryption) AND application-level (AES-256-GCM for sensitive fields).
6. **Data Retention**: Automated retention policies enforced via database triggers and S3 lifecycle policies.

**Consequences**:
- (+) Compliance is guaranteed by architecture, not by developer discipline
- (+) Audit trail supports both HIPAA audit requirements and internal security reviews
- (+) PHI filter reduces exposure to LLM vendors (PHI never leaves AWS in plaintext)
- (+) Centralized consent registry eliminates per-workstream consent tracking
- (-) Performance overhead from double encryption and audit logging (~5-10ms per request)
- (-) Development velocity impact — every service must integrate the compliance middleware
- (-) PHI filter adds complexity and latency to AI requests (~50ms for tokenization/detokenization)
- (-) BAA procurement may delay vendor onboarding (NexHealth, Sikka ONE, Anthropic, OpenAI)

**Alternatives Considered**:
1. **Per-workstream compliance**: Each workstream manages its own HIPAA compliance. Rejected — inconsistent implementation, audit gaps, and triple the compliance engineering effort.
2. **Compliance as a separate service (proxy)**: All requests route through a compliance proxy. Rejected — too much latency for real-time operations (Voice AI, lead response). Middleware approach is faster.
3. **Defer compliance to post-MVP**: Rejected — retrofitting HIPAA compliance is 3-5x more expensive than building it in. PHI handling patterns must be correct from day one.

---

### ADR-04: Kafka Event Bus for Cross-Workstream Integration

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead

**Context**:
The consolidation defined 10 cross-workstream integration points. These integrations range from real-time (< 1 minute) to daily batch. Options considered:
- Direct API calls between workstreams
- Redis Pub/Sub
- AWS SQS/SNS
- Apache Kafka (AWS MSK)

**Decision**:
Adopt Apache Kafka (via AWS MSK) as the cross-workstream event bus. Workstreams communicate through events published to Kafka topics, not through direct API calls. Each workstream owns its topics and schemas.

**Consequences**:
- (+) Decoupled producers and consumers — workstreams evolve independently
- (+) Event replay — new consumers can replay historical events (7-30 day retention)
- (+) Durable and ordered — events survive consumer downtime, processed in order per partition
- (+) Schema registry enforces contract compatibility across teams
- (+) One consumer failing does not affect the producer or other consumers
- (+) Natural audit trail — all cross-workstream data flows are captured as events
- (-) Operational complexity — Kafka (even managed MSK) requires monitoring, partition management, and consumer group management
- (-) Eventual consistency — consumers may lag behind producers (acceptable for all 10 integration points)
- (-) Overkill for 260K events/day — Kafka is designed for millions/second. But MSK Serverless eliminates over-provisioning.
- (-) Team learning curve — engineers unfamiliar with Kafka need training

**Alternatives Considered**:
1. **Direct API calls**: Simplest. But creates tight coupling — WS2 calling WS1 API creates a runtime dependency. If WS1 is down, WS2 content distribution fails. Also makes adding new consumers expensive (each new consumer requires producer changes).
2. **Redis Pub/Sub**: No durability. If a consumer is offline when an event is published, the event is lost. Unacceptable for daily H-Score calculations and champion roster syncs.
3. **AWS SQS/SNS**: Durable but lacks ordering guarantees, schema registry, and replay capability. Fan-out via SNS + SQS per consumer is operationally complex at 20+ topics.
4. **AWS EventBridge**: Good for event routing but limited replay (24 hours), no partition-based ordering, higher per-event cost at volume.

---

### ADR-05: Design Tool Abstraction (Canva as Provider)

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead, Product Strategist

**Context**:
WS2 (Content Engine) relies on Canva Connect API for all design generation — template management, asset generation, brand workspace organization (3 brand universes: SGA Legacy, Gen4, MODIS). Canva is the only design tool integration.

The QC-1 review (Item #8) flagged Canva as a single-vendor risk. If Canva changes its API, pricing, or terms, the entire Content Engine's design capability is affected.

**Decision**:
Implement a Design Provider interface (Section 1.10) that abstracts design tool operations. Canva is the initial (and currently only) provider. The interface defines: template listing, design generation from template, design export, and asset upload. WS2 code calls the abstraction layer, not Canva APIs directly.

This is a **lightweight abstraction**, not a full design tool emulation. It covers the 4 operations the platform actually uses — it does not attempt to abstract Canva's full feature set (collaborative editing, comments, etc.).

**Consequences**:
- (+) Provider swappable via configuration — future migration to Figma, Adobe Express, or a custom renderer requires implementing the interface, not rewriting WS2
- (+) Forces clean separation between "what design do we need" (business logic) and "how do we generate it" (provider logic)
- (+) Enables A/B testing of design providers in the future
- (-) Abstraction adds a thin layer of indirection (~2ms per call, negligible)
- (-) Some Canva-specific features (collaborative editing, Canva Pro features) cannot be abstracted — they are used only for internal designer workflows, not automated generation
- (-) If only one provider is ever used, the abstraction is unnecessary overhead (acceptable as insurance)

**Alternatives Considered**:
1. **Direct Canva integration (no abstraction)**: Fastest to build. Accepted risk of vendor lock-in. Rejected because the abstraction cost is minimal (one interface, one adapter class) and the insurance value is high.
2. **Multi-provider from day one**: Build Canva + Figma adapters immediately. Rejected — premature. No evidence Canva will fail. Build the interface now; add providers when needed.
3. **Custom rendering engine**: Replace Canva entirely with Puppeteer/Playwright rendering HTML templates. Rejected — massive scope increase, loses Canva's design quality and template ecosystem.

---

### ADR-06: WS1 Split into Practice Operations + Network Intelligence

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Product Strategist, Tech Lead

**Context**:
WS1 (The Practice Platform) consolidated three sprints into 32 features serving two distinct user groups:
1. **Practice-facing** (Office Managers, Front Desk, Treatment Coordinators): Practice Nerve Center, Concierge Agent, Automation Engine, Voice AI — features that help a single practice operate better.
2. **Executive-facing** (CEO, VP Ops, Regional Directors): Executive Dashboard, Network Health, AI Alerts, HQ Ops Queue — features that provide network-wide visibility and intervention.

Building these as a monolithic "Practice Platform" creates a product that serves nobody perfectly. Practice users do not need network views. Executives do not need practice-level automation controls.

**Decision**:
Split WS1's product surface into two distinct interfaces sharing a common data layer:
1. **Practice Operations**: Practice Nerve Center, Concierge Agent, Automation Engine, Voice AI, Communication management. One instance per practice. Accessed by Office Managers and practice staff.
2. **Network Intelligence**: Executive Dashboard, Network Health, AI Alerts, HQ Ops Queue, Regional drill-downs. Single instance for the network. Accessed by executives and HQ operations.

Both are backed by the same services (Practice Data Hub, PMS Integration, Funnel Data Lake, AI Analysis Engine). The split is at the frontend/API-gateway level, not at the service level.

**Consequences**:
- (+) Each product surface is optimized for its user group (practice managers get a 20-minute daily tool; executives get a real-time command center)
- (+) Independent deployment — Practice Operations can ship practice-facing features without affecting the executive dashboard, and vice versa
- (+) Clearer team ownership — one team owns the practice experience, another owns the executive experience
- (+) Security isolation — practice users cannot accidentally access network-wide data; access is enforced at the API gateway
- (-) Two frontend codebases to maintain (mitigated by shared component library)
- (-) Shared services must serve both audiences without bias — API design must accommodate both practice-scoped and network-scoped queries
- (-) Feature requests that span both surfaces (e.g., "executive wants to see what the practice manager sees") require coordination

**Alternatives Considered**:
1. **Single monolithic UI with role-based views**: One app, different views based on role. Rejected — the two user groups have fundamentally different workflows (daily 20-minute ritual vs. real-time monitoring). A single app tries to do both and does neither well.
2. **Full service-level split**: Separate backend services for Practice Operations and Network Intelligence. Rejected — over-engineering. They share the same data and the same analytical engine. Splitting services would create data synchronization problems.

---

### ADR-07: Claude as Primary AI Provider Across All Workstreams

**Status**: Accepted
**Date**: 2026-03-14
**Deciders**: Tech Lead, Product Strategist

**Context**:
Four of six original sprints chose Claude API for text generation, classification, and analysis. Sprint 2 chose OpenAI NLP for trust transfer verification. Sprint 5 listed both Claude and GPT as options. All sprints requiring audio transcription chose OpenAI Whisper (no Claude equivalent).

Maintaining multiple LLM providers means maintaining multiple prompt libraries, multiple billing relationships, multiple rate limit strategies, and inconsistent output formatting.

**Decision**:
Standardize on Claude API as the primary LLM for all text workloads across all three workstreams. OpenAI Whisper remains for audio transcription (no Claude equivalent). All access is through the shared AI Service (Section 1.4), which provides a model abstraction layer enabling future provider changes without workstream code changes.

Model routing by cost tier:
- **Claude Haiku**: Low-complexity tasks (intent classification, simple extraction, template variable filling). ~70% of requests.
- **Claude Sonnet**: Standard tasks (message generation, content analysis, caption generation). ~25% of requests.
- **Claude Opus**: Complex reasoning (anomaly root cause analysis, agent decision-making, trust transfer NLP verification). ~5% of requests.

**Consequences**:
- (+) Single prompt engineering methodology across all teams — knowledge transfers, shared prompt library
- (+) Single billing account with volume discounts
- (+) Consistent output quality and formatting across workstreams
- (+) AI Service abstracts the provider — if Claude performance degrades or pricing changes, the switch to an alternative (GPT-4, Gemini, Llama) is a configuration change at the AI Service level
- (+) PHI filter only needs to be built for one provider's API format
- (-) Vendor concentration risk — if Anthropic has an outage, all three workstreams are affected (mitigated by AI Service's ability to failover to a backup provider)
- (-) Sprint 2's Trust Transfer NLP prompts must be re-engineered for Claude (one-time migration effort, estimated 1-2 weeks)
- (-) Claude may not be optimal for every task (e.g., some benchmarks show GPT-4 stronger at structured output). Accepted — consistency value outweighs marginal per-task optimization.

**Alternatives Considered**:
1. **Best-of-breed per task**: Use Claude for agent reasoning, GPT-4 for structured output, Gemini for large context windows. Rejected — operational complexity of managing 3 providers outweighs marginal quality gains.
2. **OpenAI GPT-4 as primary**: Strong alternative. Rejected because 4 of 6 sprints already chose Claude, and Claude's longer context window supports the agent pattern better.
3. **Open-source (Llama 3, Mistral)**: Lower cost, full control. Rejected — inference infrastructure management is a distraction for a product team. Revisit when the platform is profitable and AI costs are a significant line item.

---

## Section 4: Build Sequence

*Addresses QC-1 Item #7: Extended infrastructure phase with event contract definition.*

---

### Phase 0: Pre-Build (Weeks 1-2)

**Objective**: Team formation, vendor procurement, development environment setup.

| Task | Owner | Duration | Exit Criteria |
|------|-------|----------|---------------|
| Finalize team hiring / allocation | Engineering Manager | 2 weeks | Teams staffed per recommendations below |
| Execute BAAs with NexHealth, Sikka ONE, Anthropic, OpenAI | Legal + Tech Lead | 2 weeks | All BAAs signed |
| Provision AWS accounts (dev, staging, prod) | Platform Engineering | 1 week | Accounts created, IAM configured |
| Set up CI/CD pipeline (GitHub Actions → AWS ECS) | Platform Engineering | 1 week | Pipeline deploys to dev environment |
| Set up Temporal Cloud or self-hosted cluster | Platform Engineering | 1 week | Temporal accessible from dev |
| Provision MSK Kafka cluster (dev) | Platform Engineering | 1 week | Kafka topics creatable |
| Provision Twilio HIPAA account + first 10DLC campaign | Platform Engineering | 2 weeks | Account active, test numbers provisioned |
| Set up monitoring (Datadog or CloudWatch) | Platform Engineering | 1 week | Basic dashboards and alerts |

---

### Phase 1: Shared Infrastructure (Month 1 — Weeks 1-4)

**Objective**: Build the shared services that all workstreams depend on. No workstream feature work starts until Phase 1 exit criteria are met.

| Week | Service | Deliverables |
|------|---------|-------------|
| 1-2 | Practice Data Hub | PostgreSQL schema deployed. CRUD API live. Practice seed data loaded (260 practices). Read replica configured. |
| 1-2 | Auth & Identity | Cognito user pool created. RBAC service deployed. JWT middleware available as shared npm package. Test users for each role created. |
| 2-3 | Compliance Middleware | Fastify plugin published as shared npm package. Audit log table deployed. PHI detection regex library. Encryption helpers. |
| 2-3 | Event Bus (Kafka) | MSK cluster operational. Schema Registry deployed. All topic schemas from Section 2 registered. Producer/consumer SDK published as shared npm package. |
| 3-4 | PMS Integration Service | NexHealth adapter operational for top 3 PMS systems (Dentrix, Eaglesoft, Open Dental). Webhook receiver deployed. Canonical event schema emitting to Kafka. |
| 3-4 | Communications Service | Twilio integration live. Consent registry deployed. Send API operational (SMS + email). Number provisioning for 10 test practices. |
| 4 | AI Service | Claude API integration live. PHI filter operational. Token accounting. Prompt registry seeded with initial prompts. Whisper integration for audio. |
| 4 | Asset Registry | S3 buckets + CloudFront distribution. Upload/download API. Metadata catalog in PostgreSQL. |
| 4 | Shared Design System | `@sgadental/design-system` npm package published. Design tokens (colors, typography, spacing) defined for all 3 brand universes. Primitive components (Button, Input, Card, Table, Modal) built and documented in Storybook. Tailwind preset available. `DashboardLayout` and `KPITile` components ready for WS consumption. |

**Phase 1 Exit Criteria** (ALL must pass):
- [ ] Practice Data Hub API returns data for 260 practices
- [ ] A test service can authenticate via Cognito, receive a JWT, and call Practice Data Hub API with RBAC enforcement
- [ ] Compliance middleware logs a PHI access event to the audit log
- [ ] A test producer can publish a `practice.profile-updated` event to Kafka and a test consumer receives it with schema validation
- [ ] PMS Integration Service receives a webhook from NexHealth and emits a canonical `pms.appointment-created` event to Kafka
- [ ] Communications Service sends an SMS via Twilio and logs it to the message log with consent check
- [ ] AI Service processes a Claude API request with PHI filter (tokenize → send → detokenize) and returns a response
- [ ] Asset Registry accepts an upload, stores in S3, and returns a CDN URL
- [ ] `@sgadental/design-system` package installable from private npm registry; Storybook renders all primitive components across all 3 brand universes

---

### Phase 2: Event Contracts + WS3 Early Start (Month 2 — Weeks 5-8)

**Objective**: Harden event contracts with integration tests. WS3 starts building (it has the fewest infrastructure dependencies). Extend PMS coverage.

**Event Contract Hardening (Platform Engineering)**:

| Week | Task |
|------|------|
| 5 | Deploy contract testing framework (Pact or similar). Write producer tests for all 10 event contracts. |
| 5-6 | Deploy consumer contract tests — verify each consumer can deserialize and process events from each producer. |
| 6 | Load test: simulate 260 practices emitting events concurrently. Verify Kafka consumer lag stays < 30 seconds under load. |
| 7 | Deploy dead-letter queue (DLQ) handling for each consumer group. Alert on DLQ depth > 0. |
| 7-8 | Deploy event monitoring dashboard: events/sec by topic, consumer lag by group, schema validation failures. |

**WS3: Culture OS (Early Start — Weeks 5-8)**:

WS3 can start in Month 2 because it has minimal infrastructure dependencies:
- Reads practice profiles from Practice Data Hub (available Week 2)
- Publishes events to Kafka (available Week 3)
- Does NOT depend on PMS Integration (H-Score reads from Cinnamon, Swell, Dental Intel — external APIs, not PMS)
- Does NOT depend on Communications Service (Culture OS uses Dental Intel for huddles, not Twilio)

| Week | WS3 Deliverable |
|------|-----------------|
| 5 | H-Score Engine v1: Cinnamon API integration (phone score). Swell API integration (review score). Composite calculation. |
| 6 | Morning Huddle OS v1: Dental Intel integration. Content library schema. First 12 weeks of huddle content loaded. |
| 7 | CCA Academy v1: Bronze tier module structure. Credly API integration for badge issuance. |
| 8 | Champion Network v1: Champion assignment CRUD. `champion-assigned` event emitting to Kafka. H-Score daily calc workflow in Temporal. |

**PMS Coverage Extension (Platform Engineering — Weeks 5-8)**:
| Week | Task |
|------|------|
| 5-6 | NexHealth adapters for remaining PMS systems in SGA network (Denticon, Curve Dental, etc.) |
| 7-8 | Sikka ONE batch adapter: Temporal workflow for nightly sync. Canonical mapping. Test with 50 practices. |

---

### Phase 3: WS1 + WS2 Parallel Start (Month 3 — Weeks 9-12)

**Objective**: Both WS1 and WS2 begin feature development using shared infrastructure. WS3 continues building.

**WS1: Practice Platform (Weeks 9-12)**:

| Week | WS1 Deliverable | Depends On |
|------|-----------------|------------|
| 9 | Funnel Data Lake: ClickHouse schema, funnel event ingestion from Kafka PMS events | PMS Integration Service, Kafka |
| 9-10 | Practice Nerve Center v1: Daily agenda UI (React), 3 action cards, PMS data display | Practice Data Hub, Auth, PMS Integration |
| 10 | AI Analysis Engine v1: Per-practice baseline calculation, basic anomaly detection | ClickHouse, AI Service |
| 11 | Automation Engine v1: Appointment confirmation sequence (first playbook). Sends via Communications Service. | Communications Service, PMS Integration |
| 11-12 | Executive Dashboard v1: Network funnel view, region drill-down, practice detail | ClickHouse, Practice Data Hub, Auth |
| 12 | Concierge Agent v1: Lead response automation (missed call → SMS follow-up within 1 hour) | Communications Service, PMS Integration, AI Service |

**WS2: Content Engine (Weeks 9-12)**:

| Week | WS2 Deliverable | Depends On |
|------|-----------------|------------|
| 9 | Design Tool Abstraction + Canva adapter: Template listing, design generation | Asset Registry |
| 9-10 | Content Capture Bot v1: SMS capture prompts via Communications Service. Media upload pipeline. | Communications Service, PMS Integration (appointment events), Asset Registry |
| 10-11 | AI Processing Pipeline v1: Photo enhancement (Cloudinary), caption generation (AI Service), before/after pairing | AI Service, Asset Registry |
| 11 | Approval Workflow v1: HQ QC queue. Approve/reject via web UI. `content-asset-approved` event emitted. | Auth, Kafka |
| 12 | Distribution Engine v1: Publish approved assets to Instagram + Facebook (Meta Graph API). | Kafka, Asset Registry |

**WS3: Culture OS (Continues — Weeks 9-12)**:

| Week | WS3 Deliverable |
|------|-----------------|
| 9-10 | Trust Transfer OS v1: Cinnamon call analysis + Claude NLP verification via AI Service |
| 10-11 | Recognition & Rewards Engine v1: Leaderboard, basic rewards tracking |
| 11-12 | Playbook Distribution System v1: Versioned playbook content delivery. Practice-type filtering. |
| 12 | Cross-workstream integration testing: H-Score → WS1 Executive Dashboard. Champion roster → WS2 Content Engine. |

---

### Dependency Graph

```
Week:  1    2    3    4    5    6    7    8    9    10   11   12
       ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
       │         PHASE 1              │  PHASE 2       │  PHASE 3              │
       │    Shared Infrastructure     │  Contracts+WS3 │  WS1+WS2+WS3         │
       │                              │                │                        │
Infra: ██████████████████████████████████████████████████
       Practice Hub ████
       Auth ████████
       Compliance ████████
       Kafka ██████████
       PMS Integ ████████████████████████
       Comms Svc ████████████
       AI Service ████████
       Asset Reg ████████
       Design System ████████
       Contract Tests ██████████████████

WS3:                          ████████████████████████████████████████
       H-Score Engine ████████
       Huddle OS ████████
       CCA Academy ████████
       Champion Net ████████
       Trust Transfer ████████
       Recognition ████████
       Playbook Dist ████████

WS1:                                          ████████████████████████
       Funnel Data Lake ████
       Nerve Center ████████
       AI Analysis ████
       Automation Eng ████████
       Exec Dashboard ████████
       Concierge Agent ████

WS2:                                          ████████████████████████
       Design Abstraction ████
       Capture Bot ████████
       AI Processing ████████
       Approval Flow ████
       Distribution ████

Dependency arrows:
  Practice Hub ──→ ALL workstreams
  Auth ──→ ALL user-facing services
  Kafka ──→ ALL cross-workstream events
  PMS Integration ──→ WS1 Funnel, WS1 Automation, WS2 Capture Bot
  Communications ──→ WS1 Automation, WS1 Concierge, WS2 Capture Bot
  AI Service ──→ WS1 AI Analysis, WS1 Concierge, WS2 AI Processing, WS3 Trust Transfer
  Asset Registry ──→ WS2 Design, WS2 Distribution
  Design System ──→ ALL frontend applications (WS1 Nerve Center, WS1 Exec Dashboard, WS2 Dashboard, WS3 Dashboard)
  WS3 H-Score ──→ WS1 Exec Dashboard (via Kafka)
  WS3 Champions ──→ WS2 Capture Bot (via Kafka)
  WS1 Onboarding ──→ WS3 Activation (via Kafka)
```

---

### Team Structure Recommendation

| Team | Size | Skills | Owns |
|------|------|--------|------|
| **Platform Engineering** | 4 engineers + 1 lead | Node.js/TypeScript, PostgreSQL, Kafka, AWS, Temporal, security | All shared services (Sections 1.1-1.10). Event bus operations. Infrastructure-as-code. |
| **WS1: Practice Operations** | 3 engineers + 1 lead | Node.js/TypeScript, React, ClickHouse, real-time systems | Practice Nerve Center, Concierge Agent, Automation Engine, Voice AI. Practice-facing APIs. |
| **WS1: Network Intelligence** | 2 engineers + 1 lead | React, data visualization, ClickHouse, Python (ML) | Executive Dashboard, AI Analysis Engine, HQ Ops Queue. Executive-facing APIs. |
| **WS2: Content Engine** | 3 engineers + 1 lead | Node.js/TypeScript, React Native, Canva API, media processing, social platform APIs | Content capture, AI processing, approval workflow, distribution, mobile app. |
| **WS3: Culture OS** | 2 engineers + 1 lead | Node.js/TypeScript, React, external API integrations (Cinnamon, Swell, Dental Intel, Credly) | H-Score, Morning Huddle, CCA Academy, Champion Network, Trust Transfer OS. |
| **QA / DevOps** | 2 engineers | Testing, CI/CD, monitoring, load testing | Cross-team test infrastructure. Contract tests. Performance testing. |

**Total**: 15 engineers + 5 leads = 20 people

**Rationale for team sizes**:
- Platform Engineering is largest (5) because shared infrastructure is the critical path and must be built first with high reliability.
- WS1 is split into two sub-teams (5 total) because it has the most features (32) and serves two distinct user groups.
- WS2 has 4 people because the content pipeline has the most complex media processing and external API integrations.
- WS3 has 3 people because it has the fewest features (13) and many are integrations with existing SaaS tools (Cinnamon, Swell, Dental Intel, Credly) rather than custom builds.

---

## Section 5: Risk Register

| # | Risk | Severity | Likelihood | Impact | Mitigation |
|---|------|----------|------------|--------|------------|
| R1 | **NexHealth API coverage gap**: NexHealth may not support all PMS systems in the SGA network, leaving some practices without real-time PMS connectivity | HIGH | MEDIUM | Practices on unsupported PMS systems are limited to batch data (Sikka ONE). Voice AI cannot book appointments at those practices. | Audit all 260 practices' PMS systems against NexHealth's supported list before Phase 1 exit. Prioritize NexHealth expansion requests for any gaps. Design Voice AI to gracefully degrade (provide callback number instead of direct booking) for Sikka-only practices. |
| R2 | **Kafka operational complexity**: MSK requires monitoring of partition lag, consumer group rebalancing, and schema registry versioning. Team may lack Kafka expertise. | MEDIUM | MEDIUM | Consumer lag causes stale data in dashboards. Schema evolution errors break consumers. | Use MSK Serverless to reduce ops burden. Hire or train one Kafka specialist on Platform Engineering. Deploy automated alerts for consumer lag > 60 seconds. Schema registry enforces forward compatibility by default. |
| R3 | **Twilio 10DLC registration delays**: A2P 10DLC campaign registration can take 2-8 weeks per campaign. 260 practices need numbers. | HIGH | HIGH | Patient SMS delivery is blocked or filtered as spam until 10DLC is approved. | Start 10DLC registration in Phase 0. Register one campaign for "healthcare appointment communications" (covers all patient-facing SMS). Content capture prompts use a separate campaign. Register both before Phase 1 starts. |
| R4 | **Claude API rate limits**: At 260 practices with concurrent agent operations, the platform may hit Anthropic's API rate limits during peak hours. | MEDIUM | MEDIUM | Degraded AI response times. Message generation delays. Voice AI latency increases. | AI Service implements priority queue (Voice AI > lead response > batch generation). Request Anthropic rate limit increase during BAA negotiation. Implement response caching for deterministic requests. Claude Haiku for low-complexity tasks reduces load on Sonnet/Opus tiers. |
| R5 | **PHI exposure via LLM**: Despite the PHI filter, edge cases may leak PHI to Anthropic/OpenAI servers (e.g., PHI embedded in freeform text that the regex filter does not catch). | HIGH | LOW | HIPAA violation. Potential breach notification requirement. | PHI filter uses multi-layered detection: regex patterns + NER model + manual allowlist of safe fields. Quarterly red-team exercises to test PHI filter bypass. BAA with Anthropic covers residual risk. Log all LLM requests (sanitized) for audit review. |
| R6 | **Canva API changes or deprecation**: Canva Connect API is relatively new. Breaking changes could disrupt content generation. | MEDIUM | LOW | Content Engine design generation halted until adapter is updated. | Design Tool Abstraction Layer (Section 1.10) isolates Canva-specific code to one adapter class. Pin Canva API version. Monitor Canva developer changelog. Maintain a manual fallback workflow (designer creates in Canva UI). |
| R7 | **Team capacity / hiring risk**: 20-person team may be difficult to staff in the required timeframe, particularly for specialists (Kafka, ClickHouse, Temporal). | HIGH | MEDIUM | Phase delays if critical positions are unfilled. | Identify which roles can be filled by contractors. Platform Engineering lead and Kafka specialist are the highest-priority hires. WS3 can start with 2 engineers (smallest workstream). Consider Temporal Cloud (managed service) if Temporal expertise is unavailable. |
| R8 | **Cross-workstream coordination overhead**: Three workstreams + Platform Engineering = 5 teams sharing infrastructure. Schema changes, API changes, and event contract changes require multi-team coordination. | MEDIUM | HIGH | Slower velocity. Integration bugs from mismatched assumptions. | Weekly cross-team sync (30 minutes, async-first with Slack threads). Schema Registry enforces backward/forward compatibility — uncoordinated changes are rejected. Contract tests run in CI — broken contracts block merges. Platform Engineering owns the "integration test suite" that verifies all 10 event flows end-to-end nightly. |
| R9 | **Data migration / historical data**: Some analytics features (anomaly detection baselines, H-Score trends) require historical data that may not exist at launch. | MEDIUM | HIGH | AI Analysis Engine cannot establish baselines for 30-60 days. H-Score trends require 90+ days of data. Dashboards show limited historical context. | Backfill strategy: request historical data exports from Cinnamon, Swell, Dental Intel, and PMS systems during Phase 1. Load 12 months of historical data into ClickHouse. For systems without export APIs, accept a "cold start" period with clearly communicated "data since" labels on dashboards. |
| R10 | **Compliance / BAA procurement delays**: Some vendors (particularly Anthropic and NexHealth) may have lengthy BAA negotiation processes that delay production deployment. | HIGH | MEDIUM | Cannot process PHI in production until BAAs are executed. AI Service and PMS Integration cannot go live. | Start BAA procurement in Phase 0 (Week 1). Identify backup vendors for each critical service. Use synthetic (non-PHI) data in dev/staging until BAAs are complete. Do not deploy to production with real patient data until all BAAs are signed. |

---

## Appendix A: Service Dependency Matrix

| Service | Depends On | Depended On By |
|---------|-----------|----------------|
| Practice Data Hub | PostgreSQL RDS, Auth | ALL workstreams, PMS Integration, Communications, Compliance |
| PMS Integration | Practice Data Hub, Kafka, NexHealth, Sikka ONE | WS1 (Funnel, Automation, Concierge), WS2 (Capture Bot) |
| Communications | Practice Data Hub, Auth, Kafka, Twilio, AWS SES, Redis | WS1 (Automation, Concierge, Voice AI), WS2 (Capture Bot) |
| AI Service | Redis, Claude API, OpenAI Whisper | WS1 (AI Analysis, Concierge, Voice AI), WS2 (AI Processing, Captions), WS3 (Trust Transfer NLP) |
| Asset Registry | S3, CloudFront, PostgreSQL, Cloudinary | WS2 (Content Pipeline, Distribution), WS1 (Storefront), WS3 (Recognition) |
| Event Bus (Kafka) | AWS MSK, Schema Registry | ALL cross-workstream events |
| Auth & Identity | AWS Cognito, Redis | ALL user-facing services |
| Compliance Middleware | PostgreSQL (audit log), AWS KMS | ALL services (embedded) |
| Workflow Engine (Temporal) | Temporal Cloud / self-hosted | PMS batch sync, Content pipeline, Comms cadences, H-Score calc |
| Design Tool Abstraction | Canva Connect API | WS2 (template generation) |
| Non-PMS Data Sources | Practice Data Hub, Kafka, Redis, AWS Secrets Manager, vendor APIs (RingCentral, 8x8, Weave, Google Ads, Meta, Sierra, Agnes, iSolved) | WS1 (funnel attribution, missed call detection), WS2 (engagement metrics), WS3 (phone analytics for H-Score, staff roster) |
| Breach Notification / IR | Compliance Middleware, AWS GuardDuty, PagerDuty, PostgreSQL, S3 | Security Lead, Legal, Platform Engineering |
| Disaster Recovery | RDS replication, MSK MirrorMaker, S3 CRR, Route 53, Secrets Manager | ALL services (failover target) |
| Shared Design System | npm registry, Storybook | WS1 (Practice Ops + Network Intel frontends), WS2 (Content Dashboard), WS3 (Culture Dashboard) |

---

## Appendix B: Technology Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Cloud | AWS (primary), Vercel (frontend), Cloudflare (edge/CDN) | Infrastructure |
| Compute | AWS ECS (Fargate) | Service hosting |
| Database (operational) | PostgreSQL 16 (AWS RDS Multi-AZ) | Practice data, consent, audit, assets metadata |
| Database (analytics) | ClickHouse (self-managed on EC2 or ClickHouse Cloud) | Funnel events, comms metrics, H-Score history |
| Cache | Redis (AWS ElastiCache) | Sessions, rate limiting, agent context, response cache |
| Object Storage | AWS S3 + CloudFront | Media assets, audit archives, cold data |
| Message Bus | Apache Kafka (AWS MSK Serverless) | Cross-workstream events |
| Workflow Orchestration | Temporal | Long-running multi-step workflows |
| Job Queue | BullMQ (Redis-backed) | Intra-service task queues |
| API Framework | Fastify (Node.js/TypeScript) | All API services |
| Frontend (web) | Next.js 14 + Tailwind CSS | Dashboards, admin UIs |
| Frontend (mobile) | React Native | Content capture app |
| AI (text) | Claude API (Haiku/Sonnet/Opus) | All text generation, analysis, classification |
| AI (audio) | OpenAI Whisper | Audio transcription |
| SMS/Voice | Twilio (HIPAA tier) | Patient comms, content capture |
| Email | AWS SES | Transactional patient email |
| Internal comms | Microsoft 365 Graph API | Teams bot, studio email intake |
| Design | Canva Connect API (behind abstraction) | Template generation, brand asset management |
| Image/Video | Cloudinary | Image transforms, video processing |
| Identity | AWS Cognito | Authentication |
| Monitoring | Datadog (or CloudWatch + X-Ray) | Logging, metrics, tracing |
| CI/CD | GitHub Actions → AWS ECS | Build, test, deploy |
| Feature Flags | LaunchDarkly (or Unleash) | Gradual rollout, practice-level toggles |
| Schema Registry | Confluent Schema Registry (or AWS Glue) | Kafka event schema enforcement |
| Phone Systems | RingCentral, 8x8, Weave APIs | Practice phone call data, missed call detection |
| Ad Platforms | Google Ads API, Meta Marketing API | Ad spend and conversion attribution |
| AI Phone Agents | Sierra, Agnes APIs | AI agent call outcomes and booking confirmations |
| HR System | iSolved API | Staff roster sync, onboarding status |
| Incident Response | PagerDuty, AWS GuardDuty | Breach detection, alerting, incident management |
| DR / Failover | Route 53, cross-region RDS/MSK/S3 replication | Disaster recovery and business continuity |
| Design System | React + Storybook + Tailwind tokens (@sgadental/design-system) | Shared UI component library across all frontends |

---

*This document should be reviewed alongside the consolidation feature map (`consolidation-feature-map.md`) and QC-1 validation (`qc-1-pain-point-validation.md`) for full context on feature requirements and pain point coverage.*

*Tech Lead Agent | 2026-03-14*
