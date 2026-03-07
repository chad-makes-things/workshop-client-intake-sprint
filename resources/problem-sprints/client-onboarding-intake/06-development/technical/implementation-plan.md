# Technical Implementation Plan -- Workshop Intake Engine

**Author:** Tech Lead (Sprint Handoff Review)
**Date:** 2026-03-07
**Status:** Ready for engineering review
**Sprint Source:** Client Onboarding Intake Sprint -- "The Interview" + "The Head Start" Hybrid

---

## 1. Architecture Validation

### 1.1 Tech Stack Confirmation

The sprint-recommended stack is **confirmed with adjustments**:

| Layer | Sprint Recommendation | Tech Lead Decision | Rationale |
|-------|----------------------|-------------------|-----------|
| Frontend | Next.js (React) | **Next.js 14+ (App Router)** | Confirmed. App Router provides server components for the dashboard (reduces client bundle), route handlers for API endpoints, and streaming for real-time progress. Eliminates the need for a separate Express API for most endpoints. |
| Backend API | Node.js / Express or Fastify | **Next.js Route Handlers + standalone Fastify worker** | The Next.js API layer handles all request/response endpoints. A separate Fastify worker process runs the background job queue (scraping, file processing, AI calls). This avoids running long-running jobs inside the Next.js process. |
| Database | PostgreSQL | **PostgreSQL 16** | Confirmed. JSONB support is critical for the question tree config and source breakdown metadata. Full-text search for compiled sections. |
| Cache / Queue | Redis + BullMQ | **Redis 7 + BullMQ** | Confirmed. BullMQ for job orchestration (scrape jobs, file processing, transcription, AI analysis). Redis for session cache and real-time progress. |
| Object Storage | AWS S3 | **AWS S3** (or S3-compatible like Cloudflare R2) | Confirmed. R2 is a cost-effective alternative if egress costs become a concern. |
| AI/LLM | OpenAI or Anthropic | **Anthropic Claude API (primary) + OpenAI as fallback** | Claude's longer context window handles large scraped content merging better. Structured output (tool use) for reliable framework mapping. OpenAI Whisper for transcription. |
| ORM | Not specified | **Drizzle ORM** | Type-safe SQL with excellent PostgreSQL support. Lighter than Prisma, better migration story, first-class JSONB handling. |
| Email | SendGrid / Postmark / SES | **Resend** | Modern developer experience, excellent Next.js integration, reasonable pricing for low volume. Postmark as backup for deliverability. |
| SMS | Twilio | **Twilio** | Confirmed. Industry standard for transactional SMS. |
| Speech-to-Text | Whisper / Deepgram / AssemblyAI | **OpenAI Whisper API (primary) + Deepgram (real-time fallback)** | Whisper for batch transcription of uploaded audio. Deepgram if real-time transcription preview is needed during recording. |

### 1.2 Architecture Gaps Identified

**Gap 1: Rate Limiting and Abuse Prevention**
The sprint architecture has no mention of rate limiting on the public-facing intake endpoints. Magic links are essentially bearer tokens -- if a link is shared or leaked, anyone can access and modify the intake. Need: rate limiting per token, IP-based throttle on magic link generation, and anomaly detection (e.g., submissions from unusual geolocations).

**Gap 2: Content Moderation**
No mention of validating or sanitizing client-submitted content (text, files, links). Uploaded files could contain malware. Pasted links could be malicious. Need: file scanning (ClamAV or cloud-based), URL validation, and input sanitization before LLM processing.

**Gap 3: LLM Cost Controls**
The AI Content Processor touches every submission, every scraped page, and every processed file. Without token budgets and caching, costs will scale unpredictably. Need: per-client token budget, response caching for identical content, and tiered processing (use smaller models for classification, larger models for synthesis).

**Gap 4: Data Retention and Privacy**
No data retention policy defined. Client business data, voice recordings, and scraped content are sensitive. Need: configurable retention periods, data export capability (GDPR-style), and secure deletion after workshop completion + grace period.

**Gap 5: Webhook Ingestion for Booking Triggers**
The architecture references webhook integration with the booking system but does not specify the contract. Need: defined webhook payload schema, signature verification, and idempotent processing.

**Gap 6: Error Recovery for Multi-Step Flows**
If the AI Content Processor fails mid-compilation (e.g., LLM timeout during gap analysis), the system needs graceful degradation. The facilitator should see partial results with clear "processing failed" indicators, not a blank dashboard.

### 1.3 ADR Review and Modifications

**ADR-1 (Branching Logic as JSON Config): CONFIRMED -- no changes.**
JSON-based question tree is the right call. Adding a recommendation: include a schema version field in the tree structure and build a migration utility for tree schema changes. Also, build a tree validator CLI tool that can be run in CI to verify all paths reach terminal nodes with complete framework coverage.

**ADR-2 (Two-Tier Web Scraping): CONFIRMED with one addition.**
Add a Tier 0: metadata-only pass. Before crawling any pages, fetch the homepage and extract `<meta>` tags, Open Graph data, structured data (JSON-LD), and sitemap.xml. This takes under 1 second and provides the page list for Tier 1. It also catches sites that are entirely client-side rendered (empty body), which should skip directly to Tier 2 (Playwright).

**ADR-3 (Multi-Modal Input): CONFIRMED -- no changes.**
Async processing with immediate client confirmation is the right pattern. One implementation note: implement a "processing complete" webhook/SSE event so the review screen can update in real-time when a file finishes processing, rather than requiring a page refresh.

**ADR-4 (Magic-Link Sessions): CONFIRMED with security tightening.**
The ADR mentions "single-use per click (token refreshes on each access)." This is correct but needs clarification: the magic link URL stays the same (for bookmarks/reminder emails), but the session token in the cookie rotates on each visit. Concurrent access from two devices should be handled by allowing multiple active cookies per session (not invalidating the first device when the second opens the link). Add: IP logging per access for audit trail.

**ADR-5 (Hybrid Framework Mapping): CONFIRMED -- no changes.**
Structural-first, AI-second is the right approach. Implementation note: store the structural mapping as a `framework_section_key` field on each question node in the tree, and the AI mapping as a separate `ai_mappings` array on the Submission record with confidence scores. This keeps the two mapping strategies cleanly separated in the data model.

---

## 2. Database Schema Design

### 2.1 Full Schema

```sql
-- ============================================================
-- ENUMS
-- ============================================================

CREATE TYPE client_segmentation AS ENUM (
  'documents-ready',
  'scattered-outdated',
  'in-our-heads',
  'early-stage'
);

CREATE TYPE notification_preference AS ENUM ('email', 'sms', 'both');

CREATE TYPE booking_status AS ENUM (
  'booked',
  'scraping',
  'intake-pending',
  'intake-in-progress',
  'intake-complete',
  'follow-up-pending',
  'narrative-ready',
  'workshop-complete',
  'archived'
);

CREATE TYPE scrape_status AS ENUM (
  'queued',
  'metadata',
  'crawling-tier1',
  'crawling-tier2',
  'processing',
  'complete',
  'partial',
  'failed'
);

CREATE TYPE page_type AS ENUM (
  'homepage', 'about', 'services', 'products', 'blog',
  'team', 'linkedin', 'social', 'review-site', 'press',
  'other'
);

CREATE TYPE session_status AS ENUM (
  'not-started', 'in-progress', 'paused', 'submitted', 'expired'
);

CREATE TYPE input_type AS ENUM (
  'text', 'voice', 'file', 'link',
  'pre-populated-confirmed',
  'pre-populated-edited',
  'pre-populated-rewritten'
);

CREATE TYPE processing_status AS ENUM (
  'queued', 'processing', 'complete', 'failed', 'skipped'
);

CREATE TYPE file_type AS ENUM (
  'pdf', 'docx', 'pptx', 'image', 'audio', 'video', 'other'
);

CREATE TYPE follow_up_status AS ENUM (
  'drafted', 'approved', 'sent', 'answered', 'expired'
);

CREATE TYPE notification_type AS ENUM (
  'welcome-email', 'pause-reminder', 'submission-confirmation',
  'facilitator-alert', 'follow-up-delivery', 'follow-up-answer',
  'processing-complete'
);

CREATE TYPE notification_channel AS ENUM ('email', 'sms');

CREATE TYPE recipient_type AS ENUM ('client', 'facilitator');

-- ============================================================
-- TABLES
-- ============================================================

-- 1. Clients
CREATE TABLE clients (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            TEXT NOT NULL,
  email           TEXT NOT NULL,
  phone           TEXT,
  company_name    TEXT NOT NULL,
  company_url     TEXT,
  segmentation    client_segmentation,
  notification_pref notification_preference DEFAULT 'email',
  metadata        JSONB DEFAULT '{}',
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_clients_email ON clients(email);
CREATE INDEX idx_clients_company ON clients(company_name);

-- 2. Workshop Bookings
CREATE TABLE workshop_bookings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id       UUID NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
  workshop_date   DATE NOT NULL,
  booking_date    TIMESTAMPTZ NOT NULL DEFAULT now(),
  status          booking_status NOT NULL DEFAULT 'booked',
  webhook_payload JSONB,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_bookings_client ON workshop_bookings(client_id);
CREATE INDEX idx_bookings_status ON workshop_bookings(status);
CREATE INDEX idx_bookings_workshop_date ON workshop_bookings(workshop_date);

-- 3. Scrape Jobs
CREATE TABLE scrape_jobs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id      UUID NOT NULL REFERENCES workshop_bookings(id) ON DELETE CASCADE,
  target_url      TEXT NOT NULL,
  status          scrape_status NOT NULL DEFAULT 'queued',
  tier            SMALLINT DEFAULT 0,
  pages_discovered INTEGER DEFAULT 0,
  pages_crawled   INTEGER DEFAULT 0,
  pages_failed    INTEGER DEFAULT 0,
  error_log       JSONB DEFAULT '[]',
  started_at      TIMESTAMPTZ,
  completed_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_scrape_jobs_booking ON scrape_jobs(booking_id);
CREATE INDEX idx_scrape_jobs_status ON scrape_jobs(status);

-- 4. Scraped Pages
CREATE TABLE scraped_pages (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  scrape_job_id   UUID NOT NULL REFERENCES scrape_jobs(id) ON DELETE CASCADE,
  url             TEXT NOT NULL,
  page_type       page_type DEFAULT 'other',
  title           TEXT,
  raw_html        TEXT,
  extracted_text  TEXT,
  structured_data JSONB,
  word_count      INTEGER,
  scraped_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_scraped_pages_job ON scraped_pages(scrape_job_id);
CREATE INDEX idx_scraped_pages_type ON scraped_pages(page_type);

-- 5. Framework Sections (Template Store)
CREATE TABLE framework_sections (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  template_version INTEGER NOT NULL DEFAULT 1,
  section_key     TEXT NOT NULL,
  section_name    TEXT NOT NULL,
  description     TEXT,
  min_completeness DECIMAL(3,2) DEFAULT 0.60,
  display_order   INTEGER NOT NULL,
  question_tree   JSONB NOT NULL,
  is_active       BOOLEAN DEFAULT true,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_framework_sections_key_version
  ON framework_sections(section_key, template_version);
CREATE INDEX idx_framework_sections_active
  ON framework_sections(is_active, display_order);

-- 6. Intake Sessions
CREATE TABLE intake_sessions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id       UUID NOT NULL REFERENCES clients(id) ON DELETE CASCADE,
  booking_id      UUID NOT NULL REFERENCES workshop_bookings(id) ON DELETE CASCADE,
  magic_link_token TEXT NOT NULL,
  current_step    INTEGER DEFAULT 0,
  current_section TEXT,
  completion_pct  DECIMAL(5,2) DEFAULT 0.00,
  segmentation_choice client_segmentation,
  framework_version INTEGER NOT NULL DEFAULT 1,
  session_data    JSONB DEFAULT '{}',
  status          session_status NOT NULL DEFAULT 'not-started',
  started_at      TIMESTAMPTZ,
  last_activity   TIMESTAMPTZ,
  submitted_at    TIMESTAMPTZ,
  expires_at      TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_sessions_magic_token ON intake_sessions(magic_link_token);
CREATE INDEX idx_sessions_client ON intake_sessions(client_id);
CREATE INDEX idx_sessions_booking ON intake_sessions(booking_id);
CREATE INDEX idx_sessions_status ON intake_sessions(status);

-- 7. Submissions
CREATE TABLE submissions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id      UUID NOT NULL REFERENCES intake_sessions(id) ON DELETE CASCADE,
  framework_section TEXT NOT NULL,
  question_node_id TEXT,
  input_type      input_type NOT NULL,
  content         TEXT,
  original_content TEXT,
  file_ref        TEXT,
  ai_mappings     JSONB,
  confidence      DECIMAL(3,2),
  display_order   INTEGER,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_submissions_session ON submissions(session_id);
CREATE INDEX idx_submissions_section ON submissions(framework_section);
CREATE INDEX idx_submissions_type ON submissions(input_type);

-- 8. Processed Files
CREATE TABLE processed_files (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  submission_id   UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  original_filename TEXT NOT NULL,
  file_type       file_type NOT NULL,
  mime_type       TEXT,
  file_size_bytes BIGINT,
  storage_key     TEXT NOT NULL,
  storage_bucket  TEXT NOT NULL,
  extracted_text  TEXT,
  extracted_metadata JSONB,
  processing_status processing_status NOT NULL DEFAULT 'queued',
  error_message   TEXT,
  processed_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_processed_files_submission ON processed_files(submission_id);
CREATE INDEX idx_processed_files_status ON processed_files(processing_status);

-- 9. Voice Recordings
CREATE TABLE voice_recordings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  submission_id   UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
  duration_secs   INTEGER,
  storage_key     TEXT NOT NULL,
  storage_bucket  TEXT NOT NULL,
  transcription   TEXT,
  transcription_status processing_status NOT NULL DEFAULT 'queued',
  transcription_model TEXT,
  error_message   TEXT,
  recorded_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  processed_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_voice_recordings_submission ON voice_recordings(submission_id);
CREATE INDEX idx_voice_recordings_status ON voice_recordings(transcription_status);

-- 10. Compiled Sections
CREATE TABLE compiled_sections (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id      UUID NOT NULL REFERENCES workshop_bookings(id) ON DELETE CASCADE,
  framework_section_id UUID NOT NULL REFERENCES framework_sections(id),
  merged_content  TEXT,
  source_breakdown JSONB NOT NULL DEFAULT '{}',
  completeness    DECIMAL(3,2) DEFAULT 0.00,
  gap_detected    BOOLEAN DEFAULT false,
  gap_details     TEXT,
  facilitator_edited BOOLEAN DEFAULT false,
  facilitator_content TEXT,
  compiled_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_compiled_sections_booking_section
  ON compiled_sections(booking_id, framework_section_id);
CREATE INDEX idx_compiled_sections_booking ON compiled_sections(booking_id);

-- 11. Follow-Ups
CREATE TABLE follow_ups (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id      UUID NOT NULL REFERENCES workshop_bookings(id) ON DELETE CASCADE,
  framework_section TEXT NOT NULL,
  question_text   TEXT NOT NULL,
  ai_generated    BOOLEAN DEFAULT true,
  status          follow_up_status NOT NULL DEFAULT 'drafted',
  channel         notification_channel DEFAULT 'email',
  response_content TEXT,
  approved_by     TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  approved_at     TIMESTAMPTZ,
  sent_at         TIMESTAMPTZ,
  answered_at     TIMESTAMPTZ
);

CREATE INDEX idx_follow_ups_booking ON follow_ups(booking_id);
CREATE INDEX idx_follow_ups_status ON follow_ups(status);

-- 12. Notification Log
CREATE TABLE notification_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id    UUID NOT NULL,
  recipient_type  recipient_type NOT NULL,
  notification_type notification_type NOT NULL,
  channel         notification_channel NOT NULL,
  subject         TEXT,
  content_preview TEXT,
  template_id     TEXT,
  metadata        JSONB DEFAULT '{}',
  sent_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
  delivered       BOOLEAN,
  delivery_id     TEXT,
  error_message   TEXT
);

CREATE INDEX idx_notifications_recipient ON notification_log(recipient_id, recipient_type);
CREATE INDEX idx_notifications_type ON notification_log(notification_type);
CREATE INDEX idx_notifications_sent ON notification_log(sent_at);

-- ============================================================
-- ADDITIONAL TABLES (Gap Fills)
-- ============================================================

-- 13. Session Access Log (security audit trail, from Gap 1)
CREATE TABLE session_access_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id      UUID NOT NULL REFERENCES intake_sessions(id) ON DELETE CASCADE,
  ip_address      INET,
  user_agent      TEXT,
  accessed_at     TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_access_log_session ON session_access_log(session_id);
CREATE INDEX idx_access_log_time ON session_access_log(accessed_at);

-- 14. AI Processing Log (cost tracking, from Gap 3)
CREATE TABLE ai_processing_log (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id      UUID REFERENCES workshop_bookings(id),
  operation       TEXT NOT NULL,
  model           TEXT NOT NULL,
  input_tokens    INTEGER,
  output_tokens   INTEGER,
  latency_ms      INTEGER,
  cost_usd        DECIMAL(8,4),
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_ai_log_booking ON ai_processing_log(booking_id);
CREATE INDEX idx_ai_log_created ON ai_processing_log(created_at);
```

### 2.2 Key Schema Design Decisions

**JSONB Fields:**
- `framework_sections.question_tree` -- The entire branching logic graph per section. Validated at the application layer against a JSON Schema before write. Indexed with GIN if query patterns require it.
- `submissions.ai_mappings` -- Array of `{section_key, confidence, snippet}` objects for AI-classified multi-section content.
- `compiled_sections.source_breakdown` -- Object tracking content provenance: `{ai_drafted: 40, client_authored: 30, file_extracted: 20, voice_transcribed: 10}` as percentage breakdown with source references.
- `scrape_jobs.error_log` -- Array of error objects for debugging failed crawls without a separate table.

**Soft Deletes:**
Not implemented in V1. Data is retained until explicit cleanup (per retention policy). If needed, add `deleted_at TIMESTAMPTZ` to `clients`, `workshop_bookings`, and `intake_sessions`.

**Multi-Tenancy:**
Not needed for V1. This is a single-facilitator system. If multi-facilitator support is added later, add a `facilitator_id` FK to `workshop_bookings`, `framework_sections`, and `compiled_sections`.

### 2.3 Migration Strategy

| Phase | Tables Created | Rationale |
|-------|---------------|-----------|
| Phase 1 | `clients`, `workshop_bookings`, `framework_sections`, `intake_sessions`, `submissions`, `session_access_log` | Core interview flow and session management. Everything needed for a client to complete intake with text-only input. |
| Phase 2 | `scrape_jobs`, `scraped_pages`, `ai_processing_log` | Web scraping engine and AI content processor. Adds pre-population capability. |
| Phase 3 | `compiled_sections`, `follow_ups` | Facilitator dashboard needs compiled views and follow-up management. |
| Phase 4 | `processed_files`, `voice_recordings` | File upload and voice recording support. These depend on `submissions` (Phase 1) but are not needed until multi-modal input ships. |
| Phase 5 | `notification_log` | Notification service tracking. The service itself can start in Phase 1 with simple email sends, but the log table comes with the full notification engine. |

Each migration is additive -- no schema-breaking changes across phases. Use Drizzle Kit for migration generation and version control.

### 2.4 Performance Considerations

**Indexes beyond the basics:**
- Composite index on `submissions(session_id, framework_section)` for the review screen query (all submissions grouped by section).
- Partial index on `intake_sessions(status) WHERE status = 'in-progress'` for the notification service reminder query.
- Partial index on `follow_ups(status) WHERE status IN ('drafted', 'sent')` for the dashboard pending items query.

**Data Volume Estimates (at 10 concurrent clients):**
- ~10 active intake sessions, ~200-400 submissions, ~50-100 scraped pages, ~20-40 processed files. This is well within PostgreSQL's comfort zone on a single instance. No sharding or read replicas needed.

**Hot Paths:**
- Session lookup by magic link token: single index scan on `intake_sessions.magic_link_token`. Cache in Redis with 5-minute TTL.
- Auto-save (submission upsert): single row INSERT or UPDATE. No locking concerns at this volume.
- Dashboard load (compiled sections by booking): composite index on `compiled_sections(booking_id, framework_section_id)`.

---

## 3. API Design

### 3.1 Authentication Strategy

```
Booking webhook --> creates Client + Booking + IntakeSession
                --> generates magic_link_token (crypto.randomUUID + base62 encoding)
                --> sends welcome email with URL: /intake/{magic_link_token}

Client clicks link --> GET /intake/{token}
                   --> validates token (exists, not expired, not submitted)
                   --> sets HTTP-only session cookie (short-lived JWT, 24h)
                   --> redirects to /intake/flow

Subsequent requests --> JWT in cookie (auto-refreshed)
                    --> JWT contains: session_id, client_id, booking_id
                    --> Falls back to token re-validation if cookie expired

Facilitator dashboard --> separate auth (email/password or OAuth)
                      --> standard JWT with role: 'facilitator'
```

**Token Security Rules:**
- Magic link tokens are 32-character base62 strings (entropy: ~190 bits)
- Tokens expire 30 days after workshop date
- Rate limit: max 5 token validations per minute per IP
- Token does not rotate on access (would break bookmarked URLs and reminder emails)
- Session cookie rotates every 24 hours

### 3.2 API Endpoints

#### Webhook Endpoints

```
POST /api/webhooks/booking
  Description: Receives new booking from external booking system
  Auth: Webhook signature verification (HMAC-SHA256)
  Body: {
    client_name: string,
    client_email: string,
    client_phone?: string,
    company_name: string,
    company_url?: string,
    workshop_date: string (ISO 8601),
    booking_id_external?: string
  }
  Response: 201 { booking_id, intake_url }
  Side Effects:
    - Creates Client (or upserts by email)
    - Creates WorkshopBooking
    - Creates IntakeSession with magic_link_token
    - Enqueues ScrapeJob (if company_url provided)
    - Schedules welcome email (after scrape completes or 10-min timeout)
```

#### Intake Session Endpoints

```
GET /api/intake/session
  Description: Get current session state (position, progress, status)
  Auth: Session cookie (JWT)
  Response: 200 {
    session_id, status, current_step, current_section,
    completion_pct, segmentation_choice, framework_version,
    sections: [{ key, name, status, submission_count }]
  }

PATCH /api/intake/session
  Description: Update session state (step, section, segmentation)
  Auth: Session cookie
  Body: { current_step?, current_section?, segmentation_choice? }
  Response: 200 { updated session state }

POST /api/intake/session/submit
  Description: Mark session as submitted
  Auth: Session cookie
  Body: {} (all data already saved via auto-save)
  Response: 200 { submitted_at, completeness_score }
  Side Effects:
    - Updates session status to 'submitted'
    - Triggers compilation job
    - Sends submission confirmation to client
    - Sends facilitator notification
```

#### Question Tree Endpoints

```
GET /api/intake/framework
  Description: Get the full framework structure for current session
  Auth: Session cookie
  Response: 200 {
    version: number,
    sections: [{
      key, name, description, display_order,
      question_tree: { nodes, edges, entry_node_id }
    }]
  }

GET /api/intake/section/:sectionKey/next
  Description: Get the next question node based on current position and answers
  Auth: Session cookie
  Query: { current_node_id?, answers?: JSON }
  Response: 200 {
    node: {
      id, question_text, question_type, input_hint,
      pre_populated_content?, response_options?,
      allows_voice, allows_file, allows_link
    },
    progress: { section_pct, overall_pct },
    is_terminal: boolean
  }
```

#### Submission Endpoints

```
POST /api/intake/submissions
  Description: Save a client answer (auto-save on every interaction)
  Auth: Session cookie
  Body: {
    framework_section: string,
    question_node_id?: string,
    input_type: InputType,
    content?: string,
    original_content?: string
  }
  Response: 201 { submission_id, updated_progress }

PUT /api/intake/submissions/:id
  Description: Update an existing submission (edit pre-populated content)
  Auth: Session cookie
  Body: { content: string, input_type: InputType }
  Response: 200 { submission_id }

POST /api/intake/submissions/:id/file
  Description: Upload a file attached to a submission
  Auth: Session cookie
  Body: multipart/form-data (file)
  Response: 202 {
    file_id, processing_status: 'queued',
    message: 'File received and processing'
  }
  Side Effects: Enqueues file processing job

POST /api/intake/submissions/:id/voice
  Description: Upload a voice recording
  Auth: Session cookie
  Body: multipart/form-data (audio blob, duration_secs)
  Response: 202 {
    recording_id, transcription_status: 'queued',
    message: 'Recording received'
  }
  Side Effects: Enqueues transcription job

POST /api/intake/submissions/:id/link
  Description: Submit a URL for scraping
  Auth: Session cookie
  Body: { url: string }
  Response: 202 {
    message: 'Link received and processing'
  }
  Side Effects: Enqueues link scraping job
```

#### Pre-Population Endpoints

```
GET /api/intake/prepopulated/:sectionKey
  Description: Get AI-generated pre-populated content for a section
  Auth: Session cookie
  Response: 200 {
    content: string,
    confidence: number,
    sources: [{ url, page_type, snippet }],
    available: boolean
  }
  Note: Returns available: false if scrape incomplete or no content for section
```

#### Processing Status Endpoints

```
GET /api/intake/processing-status
  Description: Get status of all pending async processing for this session
  Auth: Session cookie
  Response: 200 {
    files: [{ id, filename, status, pct_complete? }],
    voice: [{ id, duration, status }],
    links: [{ id, url, status }]
  }

GET /api/intake/events
  Description: SSE stream for real-time processing updates
  Auth: Session cookie
  Response: text/event-stream
  Events: processing-complete, compilation-update
```

#### Facilitator Dashboard Endpoints

```
GET /api/dashboard/bookings
  Description: List all bookings with intake status
  Auth: Facilitator JWT
  Query: { status?, sort?, page?, limit? }
  Response: 200 {
    bookings: [{
      id, client_name, company_name, workshop_date,
      status, completion_pct, submitted_at,
      pending_files, pending_follow_ups
    }],
    pagination: { total, page, limit }
  }

GET /api/dashboard/bookings/:id
  Description: Get full intake detail for a booking
  Auth: Facilitator JWT
  Response: 200 {
    booking: { ... },
    client: { ... },
    sections: [{
      key, name, completeness, gap_detected,
      merged_content, source_breakdown,
      facilitator_edited,
      submissions: [{
        id, input_type, content, confidence, created_at
      }]
    }],
    follow_ups: [{ ... }],
    timeline: [{ event, timestamp, detail }]
  }

PATCH /api/dashboard/sections/:compiledSectionId
  Description: Facilitator edits a compiled section
  Auth: Facilitator JWT
  Body: { facilitator_content: string }
  Response: 200 { updated section }

POST /api/dashboard/follow-ups
  Description: Create or approve a follow-up question
  Auth: Facilitator JWT
  Body: {
    booking_id: string,
    framework_section: string,
    question_text: string,
    channel: 'email' | 'sms',
    auto_generated?: boolean
  }
  Response: 201 { follow_up_id }

PATCH /api/dashboard/follow-ups/:id/approve
  Description: Approve a drafted follow-up for sending
  Auth: Facilitator JWT
  Body: { question_text?: string } (optional edit before approve)
  Response: 200 { status: 'approved' }
  Side Effects: Enqueues notification send

POST /api/dashboard/bookings/:id/compile
  Description: Trigger framework compilation
  Auth: Facilitator JWT
  Response: 202 { message: 'Compilation started' }
  Side Effects: Enqueues full compilation job

POST /api/dashboard/bookings/:id/generate-narrative
  Description: Export compiled framework as narrative prompt package
  Auth: Facilitator JWT
  Response: 200 {
    framework_package: {
      sections: [{ key, content, sources, confidence }],
      metadata: { client, workshop_date, completeness }
    }
  }
```

#### Admin / Framework Management

```
GET /api/admin/framework
  Description: Get current framework template
  Auth: Facilitator JWT
  Response: 200 { version, sections: [...] }

PUT /api/admin/framework/sections/:id
  Description: Update a framework section (question tree, thresholds)
  Auth: Facilitator JWT
  Body: { question_tree?, min_completeness?, section_name? }
  Response: 200 { updated section }
  Note: Creates new version; does not modify in-progress sessions

POST /api/admin/framework/validate
  Description: Validate a question tree configuration
  Auth: Facilitator JWT
  Body: { question_tree: JSON }
  Response: 200 {
    valid: boolean,
    errors: [{ path, message }],
    coverage: { sections_reached: [...], dead_ends: [...] }
  }
```

---

## 4. Engineering Task Breakdown

### Phase 1: Intake Flow Engine + Session Manager (8-10 weeks)

**The core interview experience. A client can receive a magic link, open the flow, answer text-based questions through a branching tree, auto-save progress, pause, resume cross-device, review, and submit.**

#### P1-01: Project Scaffolding and Infrastructure
- **Description:** Initialize Next.js 14 project with App Router, TypeScript, Drizzle ORM, PostgreSQL connection, Redis connection, project structure (monorepo or single app with clear module boundaries), ESLint, Prettier, and basic CI.
- **Effort:** M (3-5 days)
- **Components:** All
- **Dependencies:** None
- **Testing:** Verify build, lint, DB connection, Redis connection. Smoke test deploy to staging.

#### P1-02: Database Schema -- Phase 1 Tables
- **Description:** Create Drizzle schema definitions and migrations for: `clients`, `workshop_bookings`, `framework_sections`, `intake_sessions`, `submissions`, `session_access_log`. Include all enums, indexes, and constraints from Section 2.
- **Effort:** S (2-3 days)
- **Components:** All (foundational)
- **Dependencies:** P1-01
- **Testing:** Migration up/down cycle. Seed script with test data.

#### P1-03: Webhook Endpoint -- Booking Ingestion
- **Description:** Build `POST /api/webhooks/booking` endpoint. HMAC signature verification. Upsert client by email. Create booking and intake session. Generate magic link token. Return intake URL. Idempotent on external booking ID.
- **Effort:** M (3-4 days)
- **Components:** Session Manager, Framework Template Store
- **Dependencies:** P1-02
- **Testing:** Unit tests for signature verification, idempotency. Integration test for full webhook-to-session creation flow. Edge cases: duplicate webhooks, missing fields, invalid signatures.

#### P1-04: Magic Link Authentication Flow
- **Description:** Build `GET /intake/{token}` route. Token validation (exists, not expired, not submitted). JWT cookie generation. Cookie refresh middleware. Session restoration. Rate limiting (5 per minute per IP).
- **Effort:** M (3-4 days)
- **Components:** Session Manager
- **Dependencies:** P1-02, P1-03
- **Testing:** Unit tests for token validation logic. Integration tests for cookie flow. Security tests: expired tokens, invalid tokens, rate limit enforcement. Cross-device simulation (two browsers, same token).

#### P1-05: Framework Template Store -- Data Model and Seed
- **Description:** Define the question tree JSON schema (nodes, edges, conditions, framework section tags). Build JSON Schema validator. Create seed data for initial framework sections with question trees for all 4 segmentation paths. Build the `GET /api/admin/framework` and `POST /api/admin/framework/validate` endpoints.
- **Effort:** L (5-8 days)
- **Components:** Framework Template Store
- **Dependencies:** P1-02
- **Testing:** JSON Schema validation for valid and invalid trees. Path traversal tests: verify all 4 segmentation paths reach all framework sections. Dead-end detection. Snapshot tests for seed data.

#### P1-06: Question Tree Traversal Engine
- **Description:** Build the runtime engine that traverses the question tree graph. Given a segmentation choice and an answer history, resolve the next question node. Handle conditional edges (answer-value-based branching). Compute section progress and overall progress percentages. Build `GET /api/intake/section/:sectionKey/next` and `GET /api/intake/framework`.
- **Effort:** L (5-8 days)
- **Components:** Intake Flow Engine, Framework Template Store
- **Dependencies:** P1-05
- **Testing:** Unit tests for graph traversal with all 4 paths. Property-based tests: any sequence of valid answers should reach a terminal node. Performance test: traversal under 10ms for trees with 200+ nodes.

#### P1-07: Submission API -- Auto-Save
- **Description:** Build `POST /api/intake/submissions` and `PUT /api/intake/submissions/:id`. Upsert logic (same session + question_node_id = update, not duplicate). Update session progress on each save. Build `GET /api/intake/session` for state restoration.
- **Effort:** M (3-5 days)
- **Components:** Intake Flow Engine, Session Manager
- **Dependencies:** P1-04, P1-06
- **Testing:** Concurrency test: two rapid saves for the same question. Progress calculation accuracy. Session state restoration matches last saved state.

#### P1-08: Intake Flow UI -- Core Interview
- **Description:** Build the client-facing intake flow as a Next.js page. Single-question-at-a-time Typeform-style layout. Progress bar. Section transitions. Text input field. Pre-populated content display with confirm/edit/rewrite options. Self-segmentation screen (Step 3). Responsive design (mobile + desktop).
- **Effort:** XL (8-12 days)
- **Components:** Intake Flow Engine
- **Dependencies:** P1-06, P1-07
- **Testing:** Visual regression tests (Playwright screenshots). Accessibility audit (WCAG 2.1 AA). Mobile responsiveness check (375px, 768px, 1280px). User flow test: complete intake on all 4 segmentation paths.

#### P1-09: Video Kickoff Page
- **Description:** Build the landing page with video player (Step 2). Video embed (YouTube/Vimeo or self-hosted in S3). "Let's Go" button that initiates the session. Minimal, branded design per storyboard.
- **Effort:** S (2-3 days)
- **Components:** Intake Flow Engine
- **Dependencies:** P1-08
- **Testing:** Video playback on mobile Safari, Chrome, Firefox. Page load performance (LCP < 2.5s).

#### P1-10: Review and Submit Screen
- **Description:** Build the summary review screen (Step 8). Display all framework sections with status indicators (green check, yellow flag). Section previews with edit capability. Submit button. Submission confirmation page. Triggers session status update and facilitator notification (email for now, full notification service in Phase 5).
- **Effort:** M (4-5 days)
- **Components:** Intake Flow Engine, Session Manager
- **Dependencies:** P1-07, P1-08
- **Testing:** Review screen accuracy: all submissions displayed, correct status indicators. Submit flow: session status transitions correctly. Edge case: submit with 0% completion (should be allowed).

#### P1-11: Session Pause and Resume
- **Description:** Auto-save on every interaction (debounced 500ms). Browser close detection (beforeunload -- save any pending). Cross-device resume: same URL restores exact position. Session access logging for audit trail.
- **Effort:** M (3-4 days)
- **Components:** Session Manager
- **Dependencies:** P1-07, P1-08
- **Testing:** Close browser mid-question, reopen: state matches. Open on mobile after desktop session: state matches. Concurrent access from two devices: no data corruption (last-write-wins).

#### P1-12: Phase 1 Integration Testing and QA
- **Description:** End-to-end testing of the full Phase 1 flow: webhook fires, magic link generated, client opens link, watches video, self-segments, answers questions across all 4 paths, pauses and resumes, reviews, submits. Load testing for 10 concurrent sessions.
- **Effort:** M (3-5 days)
- **Components:** All Phase 1
- **Dependencies:** P1-01 through P1-11
- **Testing:** Playwright E2E suite covering all 4 segmentation paths. Performance baseline: page loads < 1s, auto-save < 200ms, session restore < 500ms.

**Phase 1 Total Estimated Effort: 44-66 days (8-10 weeks for a 2-person team)**

---

### Phase 2: Web Scraping Engine + AI Content Processor (6-8 weeks)

**The "Head Start" -- scrape client websites, generate pre-populated content, and integrate into the intake flow.**

#### P2-01: Database Schema -- Phase 2 Tables
- **Description:** Create migrations for `scrape_jobs`, `scraped_pages`, `ai_processing_log`.
- **Effort:** S (1-2 days)
- **Components:** Web Scraping Engine, AI Content Processor
- **Dependencies:** P1-02
- **Testing:** Migration up/down. Verify FK constraints.

#### P2-02: BullMQ Job Queue Infrastructure
- **Description:** Set up BullMQ with Redis. Define job types: `scrape-metadata`, `scrape-tier1`, `scrape-tier2`, `ai-analyze`, `ai-prepopulate`, `ai-compile`. Build the standalone Fastify worker process that consumes jobs. Dashboard for job monitoring (Bull Board or similar). Retry policies, dead letter queues, concurrency limits.
- **Effort:** M (4-5 days)
- **Components:** Web Scraping Engine, AI Content Processor, File Processing Pipeline
- **Dependencies:** P1-01
- **Testing:** Job enqueue/dequeue cycle. Retry behavior on failure. Concurrency limit enforcement. Dead letter queue population.

#### P2-03: Web Scraper -- Tier 0 (Metadata)
- **Description:** Given a URL, fetch the homepage and extract: `<title>`, meta description, Open Graph tags, JSON-LD structured data, favicon, sitemap.xml links. Classify site type (static HTML vs. SPA). Determine priority page list from sitemap + common paths (/about, /team, /services, /products, /blog).
- **Effort:** M (3-4 days)
- **Components:** Web Scraping Engine
- **Dependencies:** P2-02
- **Testing:** Test against 15-20 real websites (varied: static, SPA, minimal, complex). Verify metadata extraction accuracy. Timeout handling (5s per request).

#### P2-04: Web Scraper -- Tier 1 (HTTP/Cheerio)
- **Description:** Crawl priority pages using HTTP client + Cheerio. Extract clean text (strip nav, footer, boilerplate). Identify page type (about, services, blog, team, etc.). Respect robots.txt. Rate limiting (max 2 concurrent requests per domain). Cap at 25 pages. Store raw HTML and extracted text in `scraped_pages`.
- **Effort:** L (5-7 days)
- **Components:** Web Scraping Engine
- **Dependencies:** P2-03
- **Testing:** Test against diverse real websites. Content extraction quality assessment. robots.txt compliance. Rate limit verification. Graceful handling of 404s, 500s, timeouts.

#### P2-05: Web Scraper -- Tier 2 (Playwright Fallback)
- **Description:** For pages where Tier 1 extracted empty/minimal content (SPA detection), re-crawl with Playwright headless browser. Wait for JS rendering (networkidle or custom selectors). Same content extraction pipeline as Tier 1. Resource-limited: max 2 concurrent Playwright instances.
- **Effort:** M (4-5 days)
- **Components:** Web Scraping Engine
- **Dependencies:** P2-04
- **Testing:** Test against known SPA sites. Memory/CPU usage monitoring. Timeout handling (30s max per page). Graceful fallback if Playwright fails.

#### P2-06: AI Content Processor -- Framework Mapping
- **Description:** Given extracted text from scraped pages, use Claude API to classify content into framework sections with confidence scores. Structured output (tool use / JSON mode) for reliable parsing. Prompt engineering: system prompt includes framework section definitions and example mappings. Token budget management. Batch processing for efficiency.
- **Effort:** L (5-8 days)
- **Components:** AI Content Processor
- **Dependencies:** P2-04, P1-05
- **Testing:** Classification accuracy against manually labeled test set (target: 80%+ agreement with facilitator's judgment on 20 test cases). Token usage logging. Cost projection at scale. Edge cases: very short content, non-English content, irrelevant content.

#### P2-07: AI Content Processor -- Pre-Population Draft Generation
- **Description:** For each framework section with sufficient scraped content (confidence > 0.6), generate a human-readable draft paragraph. Tone: factual, editable, clearly sourced. Store as pre-populated content linked to the framework section. Build `GET /api/intake/prepopulated/:sectionKey`.
- **Effort:** M (4-5 days)
- **Components:** AI Content Processor
- **Dependencies:** P2-06
- **Testing:** Draft quality review on 10 test cases. Verify tone is neutral/factual, not marketing copy. Verify source attribution accuracy. Test with thin-content sections (should return `available: false`).

#### P2-08: Scrape-to-Welcome-Email Pipeline
- **Description:** Orchestrate the full booking flow: webhook received, scrape job enqueued, Tier 0+1 run (Tier 2 if needed), AI mapping + draft generation, welcome email sent with magic link. Timeout: if scrape takes > 10 minutes, send email with adjusted copy ("We've pulled together some initial thoughts"). Track end-to-end latency.
- **Effort:** M (3-4 days)
- **Components:** Web Scraping Engine, AI Content Processor, Notification Service (basic email)
- **Dependencies:** P2-07, P1-03
- **Testing:** End-to-end: booking webhook to email received. Timeout behavior: verify email sends even if scrape fails. Email copy adaptation based on scrape richness.

#### P2-09: Intake Flow Integration -- Pre-Populated Mode
- **Description:** Update the intake flow UI to display pre-populated content when available. Seamless transition between review mode (pre-populated) and question mode (no pre-populated content). Confirm/edit/rewrite UI with inline editor. Delta tracking (save original vs. edited content).
- **Effort:** M (4-5 days)
- **Components:** Intake Flow Engine, AI Content Processor
- **Dependencies:** P2-07, P1-08
- **Testing:** UI test: sections with pre-populated content show review mode. Sections without show question mode. Edit tracking: verify `original_content` is preserved. All 4 segmentation paths work with and without pre-populated content.

#### P2-10: Phase 2 Integration Testing
- **Description:** End-to-end: booking webhook, scrape runs, AI generates drafts, client opens flow, sees pre-populated content, edits/confirms, completes intake. Test with 5 real company URLs.
- **Effort:** M (3-4 days)
- **Components:** All Phase 2
- **Dependencies:** P2-01 through P2-09
- **Testing:** Full flow with real URLs. Scrape failure recovery. AI draft quality spot check. Performance: scrape + AI pipeline completes in < 5 minutes for typical sites.

**Phase 2 Total Estimated Effort: 36-49 days (6-8 weeks for a 2-person team)**

---

### Phase 3: Facilitator Dashboard (5-7 weeks)

**Admin view of all client intakes with framework-organized content, gap detection, follow-up management, and compilation.**

#### P3-01: Database Schema -- Phase 3 Tables
- **Description:** Create migrations for `compiled_sections`, `follow_ups`.
- **Effort:** S (1-2 days)
- **Components:** Facilitator Dashboard
- **Dependencies:** P1-02
- **Testing:** Migration up/down. FK constraints.

#### P3-02: Facilitator Authentication
- **Description:** Email/password authentication for the facilitator. JWT-based session. Password hashing (argon2). Login page. Protected route middleware. Single facilitator for V1 (no user management UI needed -- seed the account).
- **Effort:** M (3-4 days)
- **Components:** Facilitator Dashboard
- **Dependencies:** P1-01
- **Testing:** Login flow. Invalid credentials. JWT expiry and refresh. Protected route enforcement.

#### P3-03: Compilation Engine
- **Description:** Given a booking ID, compile all submissions into framework sections. Merge content from multiple sources (pre-populated + client edits + direct answers + file extracts + voice transcripts) using AI. Calculate completeness scores per section. Detect gaps. Store in `compiled_sections`. Build `POST /api/dashboard/bookings/:id/compile`.
- **Effort:** L (5-8 days)
- **Components:** AI Content Processor, Facilitator Dashboard
- **Dependencies:** P3-01, P2-06
- **Testing:** Compilation accuracy on test data. Completeness score validation. Gap detection accuracy (compared to facilitator's manual assessment). Idempotent re-compilation.

#### P3-04: Dashboard -- Booking Pipeline View
- **Description:** Build the main dashboard page. List all bookings with: client name, company, workshop date, intake status, completion percentage, submission date, pending items count. Sortable and filterable. Build `GET /api/dashboard/bookings`.
- **Effort:** M (3-4 days)
- **Components:** Facilitator Dashboard
- **Dependencies:** P3-02
- **Testing:** Sorting and filtering correctness. Empty state. Performance with 50 bookings (future-proofing).

#### P3-05: Dashboard -- Client Detail View
- **Description:** Build the per-client detail view. Framework sections as tabs or cards. Each section shows: merged content, source breakdown visualization (pie chart or bar), completeness indicator, gap flag. Content provenance labels (AI-drafted, client-authored, file-extracted, voice-transcribed). Build `GET /api/dashboard/bookings/:id`.
- **Effort:** L (5-7 days)
- **Components:** Facilitator Dashboard
- **Dependencies:** P3-03, P3-04
- **Testing:** Content accuracy: verify merged content matches source submissions. Source breakdown accuracy. Section navigation. Empty sections display correctly.

#### P3-06: Dashboard -- Facilitator Editing
- **Description:** Allow facilitator to edit compiled section content directly. Save as `facilitator_content` (preserving the original merged version). Track `facilitator_edited` flag. Build `PATCH /api/dashboard/sections/:compiledSectionId`.
- **Effort:** S (2-3 days)
- **Components:** Facilitator Dashboard
- **Dependencies:** P3-05
- **Testing:** Edit save/load cycle. Original content preservation. Edited flag set correctly.

#### P3-07: Follow-Up System
- **Description:** AI generates follow-up questions for gap sections. Facilitator reviews draft, edits if needed, approves. Approved follow-up triggers email to client with magic link to a follow-up response form. Client response saved and merged into compiled section. Build follow-up CRUD endpoints and UI.
- **Effort:** L (5-7 days)
- **Components:** Facilitator Dashboard, AI Content Processor, Notification Service
- **Dependencies:** P3-05, P2-06
- **Testing:** Follow-up lifecycle: draft, edit, approve, send, client responds, content merged. AI question quality. Email delivery. Client response form UX.

#### P3-08: Narrative Package Export
- **Description:** Build `POST /api/dashboard/bookings/:id/generate-narrative`. Compile all sections into the facilitator's prompt framework format. Include metadata, sources, confidence levels. Export as JSON and formatted markdown. This is the bridge to the downstream narrative generation system.
- **Effort:** M (3-4 days)
- **Components:** Facilitator Dashboard, AI Content Processor
- **Dependencies:** P3-06
- **Testing:** Output format matches facilitator's existing prompt framework. All sections included. Source attribution accurate. Export with partial data (some sections incomplete).

#### P3-09: Phase 3 Integration Testing
- **Description:** Full facilitator workflow: login, view pipeline, click into client, review sections, edit content, generate follow-up, approve, export narrative package.
- **Effort:** S (2-3 days)
- **Components:** All Phase 3
- **Dependencies:** P3-01 through P3-08
- **Testing:** E2E facilitator flow. Cross-reference with client intake data. Follow-up round-trip test.

**Phase 3 Total Estimated Effort: 29-42 days (5-7 weeks for a 2-person team)**

---

### Phase 4: File Processing Pipeline + Voice Input (4-6 weeks)

**Multi-modal input: file uploads (PDF, DOCX, PPTX, images), voice recording in-browser, and link scraping within the intake flow.**

#### P4-01: Database Schema -- Phase 4 Tables
- **Description:** Create migrations for `processed_files`, `voice_recordings`.
- **Effort:** S (1 day)
- **Components:** File Processing Pipeline
- **Dependencies:** P1-02
- **Testing:** Migration up/down.

#### P4-02: S3 File Upload Infrastructure
- **Description:** S3 bucket setup with lifecycle policies. Presigned URL generation for direct-to-S3 uploads from the browser (bypasses API server for large files). Organize by `bookings/{booking_id}/files/` and `bookings/{booking_id}/voice/`. File size limits (50MB for documents, 10MB for audio). MIME type validation.
- **Effort:** M (3-4 days)
- **Components:** File Processing Pipeline
- **Dependencies:** P1-01
- **Testing:** Upload via presigned URL. File size limit enforcement. MIME type validation. Lifecycle policy verification.

#### P4-03: File Processing -- PDF Extraction
- **Description:** BullMQ job: download PDF from S3, extract text using `pdf-parse` (for text-based PDFs) and cloud OCR (for scanned PDFs). Store extracted text in `processed_files.extracted_text`. Handle multi-page documents. Extract metadata (page count, author, title).
- **Effort:** M (3-4 days)
- **Components:** File Processing Pipeline
- **Dependencies:** P4-02, P2-02
- **Testing:** Text extraction accuracy on 10 varied PDFs (text-based, scanned, mixed). OCR quality assessment. Multi-page handling. Corrupted PDF handling.

#### P4-04: File Processing -- DOCX/PPTX Extraction
- **Description:** BullMQ job: extract text from DOCX (using `mammoth`) and PPTX (using `pptx-parser` or `officegen`). Preserve document structure (headings, bullet points). Extract slide text in order for presentations.
- **Effort:** S (2-3 days)
- **Components:** File Processing Pipeline
- **Dependencies:** P4-02, P2-02
- **Testing:** Extraction accuracy on varied documents. Structure preservation. Corrupted file handling.

#### P4-05: File Processing -- AI Content Mapping
- **Description:** After text extraction, send to AI Content Processor for framework mapping. Uploaded files often span multiple framework sections (e.g., a pitch deck covers market, competition, and positioning). Multi-section mapping with confidence scores. Update submissions with `ai_mappings`. Trigger re-compilation if dashboard compilation already exists.
- **Effort:** M (3-4 days)
- **Components:** File Processing Pipeline, AI Content Processor
- **Dependencies:** P4-03, P4-04, P2-06
- **Testing:** Multi-section mapping accuracy on test pitch deck. Confidence score calibration. Re-compilation trigger verification.

#### P4-06: Voice Recording -- Browser Capture
- **Description:** Build in-browser voice recording using Web Audio API / MediaRecorder. Single-tap start/stop. Waveform visualization during recording. Recording preview with playback. Upload to S3 via presigned URL. Duration tracking. Mobile and desktop support.
- **Effort:** L (5-7 days)
- **Components:** Intake Flow Engine, File Processing Pipeline
- **Dependencies:** P4-02
- **Testing:** Recording quality on Chrome, Firefox, Safari (desktop + mobile). iOS Safari MediaRecorder support (known issues -- may need polyfill). Microphone permission UX. Recording under poor network (upload retry).

#### P4-07: Voice Recording -- Transcription Pipeline
- **Description:** BullMQ job: download audio from S3, send to Whisper API for transcription. Store transcription in `voice_recordings.transcription`. Map transcription to framework section via AI Content Processor. Update submission content with transcription.
- **Effort:** M (3-4 days)
- **Components:** File Processing Pipeline, AI Content Processor
- **Dependencies:** P4-06, P2-02
- **Testing:** Transcription accuracy assessment (5 test recordings). Latency measurement (target: < 15s for 90-second recording). Error handling for unintelligible audio.

#### P4-08: Link Scraping Within Intake
- **Description:** When a client pastes a URL in the intake flow, enqueue a scraping job (reuse Tier 1/2 scraper from Phase 2). Extract article/blog/podcast page content. Map to framework section. Display processing status in the intake flow. Build `POST /api/intake/submissions/:id/link`.
- **Effort:** M (3-4 days)
- **Components:** File Processing Pipeline, Web Scraping Engine
- **Dependencies:** P2-04, P2-05
- **Testing:** Link scraping for blog posts, podcast pages, articles. Processing status UI updates. Invalid URL handling.

#### P4-09: Intake Flow UI -- Multi-Modal Input
- **Description:** Update the intake flow UI to show voice, file upload, and link input options on every question. Processing status indicators (spinner, checkmark, error). Inline file list for uploaded documents. Voice playback and transcription preview. SSE integration for real-time processing status updates.
- **Effort:** L (5-7 days)
- **Components:** Intake Flow Engine
- **Dependencies:** P4-06, P4-08, P1-08
- **Testing:** All input types work on all questions. Processing status updates in real-time. Multiple files per question. Mobile responsiveness for all input types.

#### P4-10: Phase 4 Integration Testing
- **Description:** Full flow with multi-modal input: answer questions with text, record voice, upload PDF, paste link. Verify all processing completes. Verify compiled sections include all input types with source attribution.
- **Effort:** S (2-3 days)
- **Components:** All Phase 4
- **Dependencies:** P4-01 through P4-09
- **Testing:** E2E with all input types in a single session. Processing pipeline under load (5 concurrent files). Source attribution accuracy in compiled output.

**Phase 4 Total Estimated Effort: 28-41 days (4-6 weeks for a 2-person team)**

---

### Phase 5: Notification Service + Smart Reminders (3-4 weeks)

**Automated communications: welcome emails, pause reminders, submission confirmations, follow-up delivery, and facilitator alerts.**

#### P5-01: Database Schema -- Phase 5 Tables
- **Description:** Create migration for `notification_log` (if not already created with basic email sends in earlier phases).
- **Effort:** S (1 day)
- **Components:** Notification Service
- **Dependencies:** P1-02
- **Testing:** Migration up/down.

#### P5-02: Email Template System
- **Description:** Build email templates using React Email (renders to HTML). Templates: welcome email (with/without strong scrape results), pause reminder (gentle and direct variants), submission confirmation, facilitator alert, follow-up delivery, follow-up answer confirmation. All templates use the facilitator's name and voice. Dynamic content injection.
- **Effort:** M (4-5 days)
- **Components:** Notification Service
- **Dependencies:** P1-01
- **Testing:** Template rendering accuracy. Dynamic content injection. Email client compatibility (Gmail, Outlook, Apple Mail). Dark mode. Mobile rendering.

#### P5-03: SMS Integration
- **Description:** Twilio integration for SMS sends. Message templates (shorter versions of email content). Magic link URLs in SMS. Opt-in tracking. Delivery status webhooks from Twilio.
- **Effort:** M (3-4 days)
- **Components:** Notification Service
- **Dependencies:** P5-02
- **Testing:** SMS delivery verification. Link formatting. Character limit compliance (160 char segments). Delivery status tracking.

#### P5-04: Smart Reminder Engine
- **Description:** Scheduled job (cron via BullMQ repeatable jobs) that scans for paused sessions. Calculate reminder timing based on: (a) time since last activity, (b) days until workshop. Gentle reminders when workshop is distant (> 7 days), direct when close (< 3 days). Max one reminder per 24 hours per client. Channel selection based on client preference. Include progress summary and magic link.
- **Effort:** M (4-5 days)
- **Components:** Notification Service, Session Manager
- **Dependencies:** P5-02, P5-03
- **Testing:** Reminder timing accuracy. Frequency capping enforcement. Tone adaptation based on workshop proximity. Opt-out handling. No reminders for submitted sessions.

#### P5-05: Event-Driven Notification Triggers
- **Description:** Wire up notification triggers to existing events: booking created (welcome email), scrape complete (welcome email send), session submitted (client confirmation + facilitator alert), follow-up approved (client delivery), follow-up answered (facilitator notification), processing complete (optional client notification). All notifications logged in `notification_log`.
- **Effort:** M (3-4 days)
- **Components:** Notification Service
- **Dependencies:** P5-02, P5-03
- **Testing:** Each trigger fires correct notification. Notification log accuracy. No duplicate sends.

#### P5-06: Facilitator Notification Preferences
- **Description:** Facilitator can configure: which events trigger notifications, preferred channel (email/SMS/both), quiet hours, digest mode (batch notifications every N hours vs. real-time).
- **Effort:** S (2-3 days)
- **Components:** Notification Service, Facilitator Dashboard
- **Dependencies:** P5-05
- **Testing:** Preference persistence. Quiet hours enforcement. Digest mode batching.

#### P5-07: Phase 5 Integration Testing
- **Description:** Full notification lifecycle. Verify all triggers, templates, delivery, and logging.
- **Effort:** S (2-3 days)
- **Components:** All Phase 5
- **Dependencies:** P5-01 through P5-06
- **Testing:** E2E: booking to welcome email to reminder to submission confirmation. SMS delivery verification. Notification log completeness.

**Phase 5 Total Estimated Effort: 19-25 days (3-4 weeks for a 2-person team)**

---

### Summary: Total Effort Across All Phases

| Phase | Scope | Estimated Effort | Calendar (2-person team) |
|-------|-------|-----------------|-------------------------|
| Phase 1 | Intake Flow + Session Manager | 44-66 days | 8-10 weeks |
| Phase 2 | Web Scraping + AI Processor | 36-49 days | 6-8 weeks |
| Phase 3 | Facilitator Dashboard | 29-42 days | 5-7 weeks |
| Phase 4 | File Processing + Voice | 28-41 days | 4-6 weeks |
| Phase 5 | Notifications + Reminders | 19-25 days | 3-4 weeks |
| **Total** | **Full system** | **156-223 days** | **26-35 weeks** |

**Note:** Phases 2-5 have some parallelization potential. Phase 2 and Phase 4 can run concurrently if team size increases to 3-4 engineers. Realistic delivery for 2 engineers: 7-9 months. For 3 engineers: 5-7 months.

---

## 5. Infrastructure and DevOps

### 5.1 Hosting Recommendation

**Primary: Vercel (frontend + API) + Railway or Render (worker process)**

| Component | Host | Rationale |
|-----------|------|-----------|
| Next.js Application | Vercel | Native Next.js hosting. Serverless functions for API routes. Edge middleware for rate limiting. Automatic preview deployments for PRs. |
| BullMQ Worker Process | Railway or Render | Persistent process (not serverless -- BullMQ workers need to stay alive). Docker-based deployment. Auto-scaling based on queue depth. |
| PostgreSQL | Neon (serverless) or Supabase | Neon: serverless scaling, branching for preview environments. Supabase: adds auth and realtime if needed. |
| Redis | Upstash | Serverless Redis. Pay-per-request. BullMQ compatible. Global replication. |
| S3 | AWS S3 or Cloudflare R2 | R2 for zero egress costs. S3 if tighter AWS integration is needed. |
| Playwright (Tier 2 scraping) | Browserless.io or self-hosted on Railway | Cloud browser service avoids managing Playwright infrastructure. Fallback: Docker container on Railway. |

**Alternative: Full AWS**
If cost optimization or AWS-specific requirements arise: Fargate (application + worker), RDS PostgreSQL, ElastiCache Redis, S3, CloudFront. More ops overhead but full control.

### 5.2 CI/CD Pipeline

```
GitHub Repository
  |
  |--> PR Created / Updated
  |     |--> GitHub Actions:
  |     |     1. Lint (ESLint + Prettier check)
  |     |     2. Type check (tsc --noEmit)
  |     |     3. Unit tests (Vitest)
  |     |     4. Integration tests (Vitest + testcontainers for PG/Redis)
  |     |     5. Question tree validation (custom CLI tool)
  |     |     6. Database migration dry-run
  |     |
  |     |--> Vercel Preview Deployment (automatic)
  |     |     - Preview URL per PR
  |     |     - Connected to staging database branch (Neon)
  |     |
  |     |--> Playwright E2E tests against preview URL
  |
  |--> Merge to main
  |     |--> Vercel Production Deployment (automatic)
  |     |--> Railway Worker Deployment (via Dockerfile)
  |     |--> Database migration (automated via deploy hook)
  |     |--> Smoke tests against production
  |     |--> Sentry release tracking
  |
  |--> Release tag (v1.x.x)
        |--> GitHub Release with changelog
        |--> Notification to stakeholders
```

### 5.3 Environment Strategy

| Environment | Purpose | Database | External Services |
|-------------|---------|----------|-------------------|
| **Local** | Development | Docker Compose (PG + Redis) | LLM APIs with test keys (low-rate-limit), mock email/SMS |
| **Preview** | PR review | Neon branch (auto-created per PR) | LLM APIs (test mode), email to test inbox (Resend test mode) |
| **Staging** | Pre-production validation | Neon staging branch | Full external services with staging keys, SMS to test numbers |
| **Production** | Live | Neon production | Full external services with production keys |

### 5.4 Environment Variables and Secrets

```bash
# ============================================================
# Application
# ============================================================
NODE_ENV=                       # development | staging | production
NEXT_PUBLIC_APP_URL=            # https://intake.workshopname.com
NEXT_PUBLIC_SITE_NAME=          # Workshop Intake Engine

# ============================================================
# Database
# ============================================================
DATABASE_URL=                   # postgresql://user:pass@host:5432/dbname
DATABASE_URL_UNPOOLED=          # Direct connection for migrations

# ============================================================
# Redis
# ============================================================
REDIS_URL=                      # redis://user:pass@host:6379

# ============================================================
# Authentication
# ============================================================
JWT_SECRET=                     # 256-bit random string
MAGIC_LINK_SECRET=              # Separate secret for token generation
WEBHOOK_SIGNING_SECRET=         # HMAC secret for booking webhook verification
FACILITATOR_EMAIL=              # Seeded facilitator account email
FACILITATOR_PASSWORD_HASH=      # Argon2 hash of initial password

# ============================================================
# AI / LLM
# ============================================================
ANTHROPIC_API_KEY=              # Claude API
OPENAI_API_KEY=                 # Whisper transcription + fallback LLM
AI_MODEL_PRIMARY=               # claude-sonnet-4-20250514
AI_MODEL_CLASSIFICATION=        # claude-haiku-3-20250414 (cheaper for classification)
AI_TOKEN_BUDGET_PER_CLIENT=     # Max tokens per client (cost control)

# ============================================================
# Storage
# ============================================================
S3_BUCKET=                      # intake-files-{env}
S3_REGION=                      # us-east-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_PRESIGNED_URL_EXPIRY=        # 3600 (seconds)

# ============================================================
# Email
# ============================================================
RESEND_API_KEY=
EMAIL_FROM_NAME=                # Facilitator's name
EMAIL_FROM_ADDRESS=             # facilitator@workshopname.com
EMAIL_REPLY_TO=                 # facilitator's real email

# ============================================================
# SMS
# ============================================================
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=

# ============================================================
# Web Scraping
# ============================================================
BROWSERLESS_API_KEY=            # For Playwright cloud (Tier 2)
SCRAPE_MAX_PAGES=               # 25
SCRAPE_TIMEOUT_MS=              # 600000 (10 minutes)
SCRAPE_RATE_LIMIT_PER_DOMAIN=   # 2 (concurrent requests)

# ============================================================
# Monitoring
# ============================================================
SENTRY_DSN=
SENTRY_AUTH_TOKEN=              # For source maps

# ============================================================
# Feature Flags (optional, for phased rollout)
# ============================================================
FF_VOICE_INPUT=                 # true | false
FF_FILE_UPLOAD=                 # true | false
FF_WEB_SCRAPING=                # true | false
FF_SMS_NOTIFICATIONS=           # true | false
```

---

## 6. Risk Register

| # | Risk | Severity | Likelihood | Impact | Mitigation |
|---|------|----------|-----------|--------|------------|
| R1 | **Web scraping produces poor/inaccurate pre-population** -- scraped content is wrong, outdated, or too thin, damaging the "we did our homework" impression | High | Medium | Client trust is damaged at the first touchpoint; facilitator loses confidence in the system | Tier 0 metadata check estimates content richness before committing to the "homework" messaging. Adaptive email copy for thin-web-presence clients. Clear AI-generated labels on all pre-populated content. Easy edit/rewrite UX. Test against 15+ real client websites before launch. |
| R2 | **LLM costs exceed budget at scale** -- every submission, scrape, file, and voice recording triggers AI processing | High | Medium | Unit economics break at 10+ concurrent clients | Implement per-client token budgets. Use cheaper models (Haiku) for classification, expensive models (Sonnet) only for synthesis. Cache identical content mappings. Batch processing where possible. Track costs per client in `ai_processing_log`. Set alerts at 80% of budget thresholds. |
| R3 | **Client abandonment mid-intake** -- clients start but never finish, despite pause/resume and reminders | High | Medium | The system collects partial data that is unusable for narrative generation | Monitor completion rates from day 1. A/B test flow length (shorter flow vs. more questions). Track drop-off by question (identify friction points). Allow submission at any completion level. Smart reminders with progress reinforcement. Consider a "quick mode" (fewer questions, lower quality output) for resistant clients. |
| R4 | **Question tree complexity becomes unmanageable** -- branching logic for 4 segmentation paths with 20-40 questions each produces a configuration that is hard to test and maintain | Medium | Medium | Bugs in branching logic cause dead ends, skipped questions, or incorrect framework mapping | JSON Schema validation on every tree update. Automated path coverage testing in CI. Tree visualization tool for the facilitator. Version control for tree changes. Integration tests that walk all 4 paths end-to-end. |
| R5 | **Voice recording fails on certain devices/browsers** -- Web Audio API and MediaRecorder have inconsistent support, especially on iOS Safari | Medium | High | "Talker" clients (the primary voice input audience) cannot use the feature; fall back to typing which produces thinner answers | Test on all major browser/OS combinations before Phase 4 launch. Use a polyfill library (e.g., `audio-recorder-polyfill`). Provide a fallback: "Upload a voice memo from your phone's recorder app" if in-browser recording fails. Feature flag to disable voice recording per device if critical bugs are found. |
| R6 | **Magic link security incident** -- a magic link is shared, forwarded, or intercepted, granting unauthorized access to client business data | Medium | Low | Confidential business information exposed to unauthorized parties | Token entropy is high (190 bits). Session cookie rotation on each visit. IP logging for audit trail. Rate limiting on token validation. Anomaly detection (access from unusual geolocations). Token expiry 30 days after workshop date. Option to invalidate and regenerate token. |
| R7 | **Facilitator does not trust AI-compiled output** -- the facilitator manually reviews every section and effectively does the same work with an extra step | Medium | Medium | No time savings achieved; system adds overhead rather than removing it | Build confidence gradually: start with source attribution and provenance tracking so the facilitator can verify. Track facilitator edit rates -- if they change < 10% of compiled content, confidence is building. Calibrate AI mapping prompts with the facilitator during development. Allow full override without friction. |
| R8 | **File processing pipeline creates a backlog** -- large PDFs, long audio files, or many concurrent uploads cause processing delays that block the review screen | Medium | Medium | Client experience degrades; review screen shows stale or incomplete data | Async processing with immediate confirmation (ADR-3). SSE for real-time status updates. Timeout limits per file (60s for documents, 120s for audio). Queue prioritization (active sessions > completed sessions). Scale worker processes based on queue depth. |
| R9 | **Booking system webhook integration is fragile** -- the external booking system changes its payload format or webhook delivery is unreliable | Low | Medium | New bookings do not trigger intake creation; clients never receive welcome emails | Webhook signature verification prevents unauthorized triggers. Idempotent processing prevents duplicates. Webhook payload validation with clear error logging. Manual booking creation fallback in the facilitator dashboard. Webhook retry handling (accept replays gracefully). |
| R10 | **Database migration fails during phased rollout** -- schema changes between phases conflict with existing data or running sessions | Low | Low | Downtime during deployment; potential data corruption | All migrations are additive (new tables, new columns with defaults). No column drops or type changes in production. Test all migrations against production-like data in staging. Use Neon branching for pre-migration testing. Maintain rollback scripts for every migration. |

---

## Appendix A: Dependency Graph (Simplified)

```
Phase 1
  P1-01 Scaffolding
    |
    P1-02 Schema -----> P1-03 Webhook -----> P1-04 Auth
    |                                          |
    P1-05 Framework Store                     |
    |                                          |
    P1-06 Tree Engine -----> P1-07 Submissions -----> P1-08 Intake UI
                                |                       |
                                P1-11 Pause/Resume      P1-09 Video Page
                                                        |
                                                        P1-10 Review/Submit
                                                        |
                                                        P1-12 Integration Tests

Phase 2 (can start P2-01 and P2-02 during Phase 1)
  P2-02 BullMQ Setup (parallel with Phase 1)
    |
    P2-03 Tier 0 -----> P2-04 Tier 1 -----> P2-05 Tier 2
                          |
                          P2-06 AI Mapping -----> P2-07 Draft Gen -----> P2-08 Pipeline
                                                                          |
                                                                          P2-09 UI Integration
                                                                          |
                                                                          P2-10 Integration Tests

Phase 3 (starts after Phase 2)
  P3-02 Auth -----> P3-04 Pipeline View
                      |
  P3-03 Compiler -----> P3-05 Detail View -----> P3-06 Editing
                          |                        |
                          P3-07 Follow-Ups         P3-08 Export
                                                    |
                                                    P3-09 Integration Tests

Phase 4 (can run parallel with Phase 3)
  P4-02 S3 Setup
    |
    P4-03 PDF -----> P4-05 AI Mapping
    P4-04 DOCX ----/
    |
    P4-06 Voice Recording -----> P4-07 Transcription
    |
    P4-08 Link Scraping
    |
    P4-09 UI Updates -----> P4-10 Integration Tests

Phase 5 (starts after Phase 3)
  P5-02 Email Templates -----> P5-04 Smart Reminders
  P5-03 SMS Integration ----/   |
                                 P5-05 Event Triggers -----> P5-06 Preferences
                                                              |
                                                              P5-07 Integration Tests
```

---

## Appendix B: Question Tree JSON Schema (Reference)

This is the expected shape of the `question_tree` JSONB field in `framework_sections`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "version": { "type": "integer" },
    "entry_node_id": { "type": "string" },
    "nodes": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "required": ["id", "type"],
        "properties": {
          "id": { "type": "string" },
          "type": {
            "type": "string",
            "enum": ["question", "review", "info", "terminal"]
          },
          "question_text": { "type": "string" },
          "question_hint": { "type": "string" },
          "framework_section_key": { "type": "string" },
          "input_types": {
            "type": "array",
            "items": {
              "type": "string",
              "enum": ["text", "voice", "file", "link"]
            }
          },
          "pre_populated_section": { "type": "string" },
          "response_options": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "value": { "type": "string" },
                "label": { "type": "string" },
                "next_node_id": { "type": "string" }
              }
            }
          },
          "default_next_node_id": { "type": "string" },
          "conditions": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "field": { "type": "string" },
                "operator": {
                  "type": "string",
                  "enum": ["eq", "neq", "contains", "exists"]
                },
                "value": {},
                "next_node_id": { "type": "string" }
              }
            }
          },
          "segmentation_filter": {
            "type": "array",
            "items": { "type": "string" }
          }
        }
      }
    }
  },
  "required": ["version", "entry_node_id", "nodes"]
}
```

**Node types:**
- `question` -- Presents a question, collects input, branches based on answer or default
- `review` -- Presents pre-populated content for confirm/edit/rewrite
- `info` -- Displays informational content (e.g., section intro), no input collected
- `terminal` -- End of a section path; triggers transition to next section or review screen

---

## Appendix C: Key Technical Decisions Log

| Decision | Choice | Alternatives Considered | Rationale |
|----------|--------|------------------------|-----------|
| ORM | Drizzle | Prisma, Kysely, raw SQL | Type-safe, lightweight, first-class JSONB, better migration control than Prisma |
| Email service | Resend | SendGrid, Postmark, SES | Modern DX, React Email integration, simple pricing, good deliverability |
| Worker process | Standalone Fastify + BullMQ | Next.js API routes with edge functions, AWS Lambda | BullMQ workers need persistent connections; serverless functions timeout too quickly for scraping and AI processing |
| AI provider (primary) | Anthropic Claude | OpenAI GPT-4, Gemini | Longer context window for large content merging; structured tool use for reliable JSON output; competitive pricing |
| Transcription | OpenAI Whisper API | Deepgram, AssemblyAI | Best accuracy for general speech; simple API; already using OpenAI key |
| Database hosting | Neon | Supabase, AWS RDS, PlanetScale | Serverless scaling, branch-per-PR for preview environments, PostgreSQL-native |
| Frontend deployment | Vercel | AWS Amplify, Netlify, Cloudflare Pages | Native Next.js support, preview deployments, edge middleware |
| Headless browser | Browserless.io | Self-hosted Playwright, Puppeteer | Managed infrastructure avoids memory/CPU management; pay-per-session pricing |
