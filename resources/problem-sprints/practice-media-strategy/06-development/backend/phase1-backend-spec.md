# Phase 1 Backend Specification — Content Concierge

**Sprint**: Practice Media Strategy
**Phase**: 1 — The SMS Coach (Weeks 1–6)
**Scope**: Twilio SMS/WhatsApp capture bot + Sikka ONE API nightly sync + S3 media storage + Airtable review handoff
**Pilot**: 10–15 practices
**Date**: 2026-03-10

---

## 1. Service Architecture

Phase 1 deploys three backend services, all containerized on AWS ECS. The design is intentionally lightweight — human editors handle production, Airtable handles review, and SMS handles the capture interface. No mobile app. No AI pipeline yet.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS ECS Cluster                              │
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  Scheduler Svc   │───▶│  PostgreSQL (RDS) │                      │
│  │  (Node.js)       │    │  :5432            │                      │
│  │  Nightly sync    │    │                   │                      │
│  │  Sikka ONE API   │    │  capture_assignments                     │
│  │  Daily SMS brief │    │  practices                               │
│  │  Post-op follow- │    │  users (champions)                       │
│  │  up scheduler    │    │  playbooks                               │
│  └──────────────────┘    │  consents                                │
│                          └──────────────────┘                      │
│  ┌──────────────────┐                                               │
│  │  Capture Svc     │───▶  AWS S3                                   │
│  │  (Node.js)       │    (media-raw bucket)                         │
│  │  Twilio webhook  │                                               │
│  │  MMS/WhatsApp    │───▶  Airtable API                             │
│  │  ingest          │    (review queue)                             │
│  │  Metadata tag    │                                               │
│  └──────────────────┘                                               │
│                                                                     │
│  ┌──────────────────┐                                               │
│  │  Admin Svc       │───▶  PostgreSQL (RDS)                         │
│  │  (Node.js/       │                                               │
│  │   Express)       │                                               │
│  │  Practice config │                                               │
│  │  Champion mgmt   │                                               │
│  │  Playbook setup  │                                               │
│  └──────────────────┘                                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌──────────────┐   ┌──────────────────┐   ┌──────────────┐
│  Sikka ONE   │   │  Twilio          │   │  Airtable    │
│  API (PMS)   │   │  (SMS/WhatsApp)  │   │  (Review)    │
│  400+ PMS    │   │  Inbound MMS     │   │  Human QC    │
└──────────────┘   │  Outbound SMS    │   └──────────────┘
                   └──────────────────┘
```

### Service Inventory

| Service | Port | Language | Responsibility |
|---------|------|----------|----------------|
| **Scheduler Service** | 8001 | Node.js 20 | Nightly Sikka ONE sync, creates capture assignments, sends daily SMS briefs at 7:30 AM local time per practice, manages post-op follow-up schedule |
| **Capture Service** | 8002 | Node.js 20 | Receives Twilio webhooks (inbound MMS + WhatsApp), uploads media to S3, tags metadata, writes record to Airtable review queue, sends confirmation SMS back to champion |
| **Admin Service** | 8003 | Node.js 20 (Express) | CRUD for practices, champions, playbook configs, consent records. Internal use only. |

### Inter-Service Communication

| From | To | Protocol | Purpose |
|------|----|----------|---------|
| Scheduler Service | Sikka ONE API | HTTPS REST | Nightly appointment data pull |
| Scheduler Service | PostgreSQL | pg driver | Write capture assignments |
| Scheduler Service | Twilio | HTTPS REST | Send daily brief SMS to champions |
| Capture Service | Twilio | Webhook (HTTP POST) | Receive inbound MMS/WhatsApp |
| Capture Service | AWS S3 | AWS SDK | Upload raw media |
| Capture Service | Airtable API | HTTPS REST | Create review record |
| Capture Service | PostgreSQL | pg driver | Log capture, update assignment status |
| Admin Service | PostgreSQL | pg driver | Practice/champion CRUD |

---

## 2. Database Schema (PostgreSQL)

### `practices`

```sql
CREATE TABLE practices (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                  TEXT NOT NULL,
  slug                  TEXT UNIQUE NOT NULL,           -- e.g. 'innovative-dental-springfield'
  pms_system            TEXT,                           -- 'open_dental' | 'dentrix' | 'eaglesoft'
  sikka_practice_id     TEXT,                           -- Sikka ONE practice identifier
  timezone              TEXT NOT NULL DEFAULT 'America/Chicago',
  playbook_type         TEXT NOT NULL,                  -- 'implant_cosmetic' | 'specialty' | 'growth_gp' | 'standard_gp'
  capture_target_per_week INT NOT NULL DEFAULT 3,
  active                BOOLEAN NOT NULL DEFAULT true,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### `users` (champions + leadership)

```sql
CREATE TABLE users (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id           UUID REFERENCES practices(id),  -- NULL for HQ/leadership
  name                  TEXT NOT NULL,
  role                  TEXT NOT NULL,                  -- 'champion' | 'doctor' | 'leader' | 'hq_admin'
  phone                 TEXT NOT NULL UNIQUE,            -- E.164 format: +15555551234
  whatsapp_opted_in     BOOLEAN NOT NULL DEFAULT false,
  streak_count          INT NOT NULL DEFAULT 0,
  total_captures        INT NOT NULL DEFAULT 0,
  active                BOOLEAN NOT NULL DEFAULT true,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### `playbooks`

```sql
CREATE TABLE playbooks (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name                  TEXT NOT NULL,
  type                  TEXT NOT NULL UNIQUE,
  capture_frequency_target INT NOT NULL DEFAULT 3,      -- per week
  high_value_procedure_codes TEXT[] NOT NULL DEFAULT '{}',
  content_types         TEXT[] NOT NULL,                -- 'before_after' | 'testimonial' | 'topic_video'
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE playbook_guidance (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  playbook_id           UUID REFERENCES playbooks(id),
  procedure_code        TEXT NOT NULL,
  content_type          TEXT NOT NULL,
  guidance_text         TEXT NOT NULL,
  example_caption       TEXT,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### `capture_assignments`

```sql
CREATE TABLE capture_assignments (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id           UUID NOT NULL REFERENCES practices(id),
  champion_id           UUID NOT NULL REFERENCES users(id),
  patient_id            TEXT,                           -- PMS patient ID, nullable for non-patient content
  procedure_code        TEXT,
  procedure_description TEXT,
  content_type          TEXT NOT NULL,                  -- 'before_after_before' | 'testimonial' | 'topic_video'
  guidance_text         TEXT NOT NULL,
  due_date              DATE NOT NULL,
  status                TEXT NOT NULL DEFAULT 'pending', -- 'pending' | 'captured' | 'expired' | 'skipped'
  source                TEXT NOT NULL DEFAULT 'pms',    -- 'pms' | 'content_calendar' | 'post_op' | 'event'
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_assignments_champion_due ON capture_assignments(champion_id, due_date);
CREATE INDEX idx_assignments_practice_status ON capture_assignments(practice_id, status);
```

### `captures`

```sql
CREATE TABLE captures (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id         UUID REFERENCES capture_assignments(id),  -- nullable for unmatched captures
  practice_id           UUID NOT NULL REFERENCES practices(id),
  champion_id           UUID NOT NULL REFERENCES users(id),
  media_type            TEXT NOT NULL,                  -- 'photo' | 'video'
  s3_key                TEXT NOT NULL UNIQUE,
  s3_bucket             TEXT NOT NULL,
  file_size_bytes       BIGINT,
  mime_type             TEXT,
  content_type          TEXT,                           -- derived from assignment or AI classification
  airtable_record_id    TEXT,                           -- once pushed to review queue
  twilio_message_sid    TEXT,
  captured_at           TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_captures_practice ON captures(practice_id, captured_at DESC);
CREATE INDEX idx_captures_champion ON captures(champion_id, captured_at DESC);
```

### `consents`

```sql
CREATE TABLE consents (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id            TEXT NOT NULL,
  practice_id           UUID NOT NULL REFERENCES practices(id),
  consent_type          TEXT NOT NULL,                  -- 'photo' | 'video' | 'both'
  signed_at             TIMESTAMPTZ NOT NULL,
  expires_at            TIMESTAMPTZ,                    -- NULL = no expiry
  status                TEXT NOT NULL DEFAULT 'active', -- 'active' | 'expired' | 'revoked'
  notes                 TEXT,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX idx_consents_patient_practice ON consents(patient_id, practice_id);
```

---

## 3. Scheduler Service (Node.js)

### 3.1 Nightly Sikka ONE Sync

Runs at **11:00 PM UTC** (covers all US timezones for "tomorrow's" appointments).

```
JOB: nightly-assignment-sync
Schedule: 0 23 * * * (UTC)

FOR each practice WHERE active = true AND sikka_practice_id IS NOT NULL:
  1. Call Sikka ONE API: GET /appointments?practice={id}&date={tomorrow}
  2. FOR each appointment:
       IF procedure_code IN playbook.high_value_procedure_codes:
         IF consents table has active consent for patient_id:
           guidance = playbook_guidance.get(procedure_code, content_type)
           INSERT capture_assignment(
             practice_id, champion_id=practice.content_champion,
             patient_id, procedure_code,
             content_type, guidance_text,
             due_date=tomorrow, source='pms'
           )
     IF appointment has NO prior capture in last 90 days AND
        appointment.procedure_code is high_value:
       ALSO create 'before_after_before' assignment with guidance
  3. Log sync result (success/fail/skipped count) to sys.log
```

**Sikka ONE API Integration Notes**:
- Auth: API key in `Authorization: Bearer {key}` header
- Rate limit: 100 req/min per practice; serialize sync across practices with 700ms delay
- Error handling: if Sikka returns 5xx, retry 3× with exponential backoff; log failure and skip that practice — do NOT retry failed practices on next night's job without manual review

### 3.2 Daily Brief SMS (7:30 AM local time)

Runs at **7:30 AM per practice timezone** (separate scheduled job per timezone offset, or time-shifted single job).

```
JOB: daily-brief-sms
Schedule: Triggered by scheduler per-practice at 7:30 AM local time

FOR each champion with pending assignments WHERE due_date = today:
  assignments = SELECT * FROM capture_assignments
                WHERE champion_id = ? AND due_date = today AND status = 'pending'

  IF assignments.length == 0: SKIP

  message = format_daily_brief(champion.name, assignments)
  twilio.messages.create({
    to: champion.phone,
    from: TWILIO_NUMBER,
    body: message
  })
```

**Daily Brief SMS Format**:
```
Good morning {first_name}! 📸 {count} capture(s) today:

1. Before photo — {procedure} at {time} (Pt: {initials})
   💡 {guidance_text_short}

2. Testimonial — {patient_name} finishing {procedure}
   💡 Ask at checkout: "Would you be willing to share your experience?"

Reply with your photos/video to this number.
Questions? Text HELP.
```

**Character budget**: Keep under 320 characters per segment (MMS-aware).

### 3.3 Post-Op Follow-Up Scheduler

Runs nightly alongside the sync job. Checks for appointments from 14, 30, and 90 days ago.

```
JOB: post-op-followup
FOR each completed appointment at [14, 30, 90] days ago:
  IF procedure_type IN ['full_arch_implant', 'all_on_4', 'smile_makeover']:
    IF no existing testimonial capture for this patient:
      INSERT capture_assignment(
        content_type='testimonial',
        source='post_op',
        guidance_text='Patient is 30 days post full arch. Perfect time for a smile reveal video.',
        due_date=today
      )
```

---

## 4. Capture Service (Node.js)

### 4.1 Inbound MMS Webhook

**Endpoint**: `POST /webhooks/twilio/sms`

Twilio calls this endpoint when any inbound message arrives at the registered number. Validated with Twilio signature.

```javascript
// Validate Twilio webhook signature
// Extract: From (champion phone), Body (text), MediaUrl0..N (media URLs)
// Resolve champion from phone number
// For each media attachment:
//   1. Download from Twilio CDN (temporary URL, valid 4 hours)
//   2. Upload to S3: s3://content-concierge-raw/{practice_slug}/{date}/{uuid}.{ext}
//   3. Resolve active capture assignment for champion (earliest pending today)
//   4. INSERT captures record
//   5. Push to Airtable review queue
//   6. Reply SMS: confirmation with emoji + count
```

**S3 Key Pattern**:
```
content-concierge-raw/
  {practice_slug}/
    {YYYY-MM-DD}/
      {capture_uuid}.{jpg|mp4|mov}
```

**Airtable Record Push** (per capture):
```json
{
  "Practice": "{practice_name}",
  "Champion": "{champion_name}",
  "Content Type": "{content_type}",
  "Procedure": "{procedure_description}",
  "Patient Initials": "{initials}",
  "Media URL": "{s3_presigned_url_7days}",
  "Media Type": "Photo | Video",
  "Captured At": "{ISO timestamp}",
  "Status": "Needs Review",
  "Guidance": "{guidance_text}",
  "Capture ID": "{uuid}"
}
```

**Response SMS to champion**:
```
Got it! ✅ {count} photo(s) received. Our team will review and publish within 24 hours.
This week: {streak_count} day streak 🔥
```

### 4.2 Inbound WhatsApp Webhook

**Endpoint**: `POST /webhooks/twilio/whatsapp`

Same logic as MMS webhook. WhatsApp-specific:
- Messages arrive via `whatsapp:+15555551234` format
- Twilio Content API used for WhatsApp-specific templating (opt-in flow at onboarding)
- Media downloads same pattern as MMS

### 4.3 HELP / STOP / STATUS Commands

Any inbound text (no media) is parsed for commands:

| Keyword | Response |
|---------|----------|
| `HELP` | "To send content: reply with photos/videos. For support: contact your SGA rep." |
| `STOP` | Unsubscribe from daily briefs (update `users.active = false`). Twilio STOP compliance. |
| `STATUS` | "This week: {captures_count} captures, {streak_count}-day streak. Playbook target: {target}/week." |
| `SKIP` | Mark today's pending assignments as `skipped`. Log reason (no patients matched, etc.) |
| Anything else | If no media: "To send content, attach a photo or video to your message." |

---

## 5. Admin Service (Node.js/Express)

Internal-only REST API. No public exposure. Auth: static API key for Phase 1 (JWT in Phase 2).

### Endpoints

```
Practice Management
  POST   /practices                — Create practice
  GET    /practices                — List all active practices
  GET    /practices/:id            — Get practice details
  PATCH  /practices/:id            — Update practice config
  DELETE /practices/:id            — Soft-delete (active = false)

Champion Management
  POST   /practices/:id/champions  — Add champion to practice
  GET    /practices/:id/champions  — List practice champions
  PATCH  /champions/:id            — Update champion (phone, role, etc.)

Playbook Management
  GET    /playbooks                — List playbooks
  GET    /playbooks/:type          — Get playbook with guidance
  PATCH  /playbooks/:id            — Update guidance text

Consent Management
  POST   /consents                 — Record signed consent
  GET    /consents?practice=&patient= — Check consent status
  PATCH  /consents/:id/revoke      — Revoke consent

Capture Monitoring
  GET    /captures?practice=&date= — View captures for review
  GET    /assignments?champion=&date= — View assignments
  GET    /health/sync              — Last Sikka sync status per practice
```

---

## 6. AWS Infrastructure

### S3 Buckets

| Bucket | Purpose | Retention | Access |
|--------|---------|-----------|--------|
| `content-concierge-raw` | Raw MMS/WhatsApp uploads | 7 years (HIPAA) | Private; presigned URLs only |
| `content-concierge-processed` | (Phase 2) AI-processed assets | Indefinite | CloudFront CDN |

**Bucket Policy**: Block all public access. Server-side encryption (SSE-KMS). Access logging enabled.

### ECS Task Definitions

| Service | CPU | Memory | Desired Count |
|---------|-----|--------|---------------|
| Scheduler Service | 256 | 512 MB | 1 (singleton) |
| Capture Service | 512 | 1 GB | 2 (for failover) |
| Admin Service | 256 | 512 MB | 1 |

### RDS

- Engine: PostgreSQL 15
- Instance: `db.t3.medium` (2 vCPU, 4 GB) — sufficient for Phase 1 volume
- Multi-AZ: No (Phase 1; enable in Phase 2)
- Backup: 7-day automated backups
- Encryption: at rest (AES-256)

### Secrets Manager

All credentials stored in AWS Secrets Manager. Never in environment variables directly.

| Secret | Values |
|--------|--------|
| `content-concierge/twilio` | account_sid, auth_token, sms_number, whatsapp_number |
| `content-concierge/sikka` | api_key, base_url |
| `content-concierge/airtable` | api_key, base_id, table_id |
| `content-concierge/database` | host, port, database, username, password |

---

## 7. Twilio Configuration

### Phone Numbers

| Number | Use | Configured Webhook |
|--------|-----|-------------------|
| `+1 (XXX) XXX-XXXX` | SMS captures (all practices) | POST /webhooks/twilio/sms |
| `whatsapp:+1 (XXX) XXX-XXXX` | WhatsApp captures | POST /webhooks/twilio/whatsapp |

### WhatsApp Business Account Setup
1. Register Twilio WhatsApp Sender via Twilio Console
2. Submit for Meta WhatsApp Business API approval (~5 business days)
3. Pre-approve opt-in template: "SGA Content Concierge: reply YES to receive daily photo prompts from your SGA team"
4. Champions opt in during onboarding call

### Twilio Webhook Validation
All inbound webhooks validated with `twilio-node` signature validation middleware:
```javascript
import { validateExpressRequest } from 'twilio/lib/webhooks/webhooks';
// 401 if signature invalid
```

---

## 8. Airtable Review Queue

Phase 1 uses Airtable as the production review interface for human editors. Base structure:

### Table: `Content Review Queue`

| Field | Type | Values |
|-------|------|--------|
| Practice | Text | Practice name |
| Champion | Text | Champion name |
| Content Type | Single Select | Before Photo, After Photo, Testimonial, Topic Video |
| Procedure | Text | Procedure description |
| Patient Initials | Text | For reference |
| Media URL | URL | Presigned S3 URL (7-day expiry) |
| Media Type | Single Select | Photo, Video |
| Captured At | DateTime | ISO timestamp |
| Status | Single Select | Needs Review, In Production, Approved, Rejected |
| Editor Notes | Long Text | Editor feedback |
| Published To | Multiple Select | Instagram, Facebook, YouTube, etc. |
| Guidance | Long Text | Original capture guidance from playbook |
| Capture ID | Text | UUID reference back to PostgreSQL |

### Airtable Automation (built-in, no code)
- **When Status → Approved**: Notify HQ Slack channel `#content-ready`
- **When Status → Rejected**: Trigger webhook to Capture Service → send SMS to champion with feedback

### View Configuration
- **Default View**: Kanban grouped by Status
- **Practice View**: Filtered by Practice, sorted by Captured At
- **Today's Queue**: Filtered by Captured At = today

---

## 9. Sikka ONE API Integration

### Authentication
```
Authorization: Bearer {api_key}
Content-Type: application/json
X-Practice-ID: {sikka_practice_id}
```

### Key Endpoints Used

| Endpoint | Purpose | Call Frequency |
|----------|---------|----------------|
| `GET /appointments?date={YYYY-MM-DD}` | Tomorrow's appointments for a practice | Nightly |
| `GET /patients/{patient_id}` | Patient details (for consent check cross-reference) | On-demand during sync |
| `GET /procedures?codes={list}` | Procedure code descriptions | One-time on setup |

### Data Mapping (Sikka → PostgreSQL)

```
sikka.appointment.patient_id      → capture_assignments.patient_id
sikka.appointment.procedure_code  → capture_assignments.procedure_code
sikka.appointment.provider_name   → capture_assignments.guidance_text (context)
sikka.appointment.appt_time       → daily brief SMS appointment time display
```

### PMS Coverage for Phase 1 Pilot
Start with practices on **Open Dental** (best API access, no partner agreement needed). Dentrix and Eaglesoft require Dentsply/Patterson API partner agreements — initiate paperwork immediately, expect 4–6 week approval cycle.

---

## 10. Error Handling & Monitoring

### Critical Failure Paths

| Failure | Impact | Response |
|---------|--------|---------|
| Sikka API down at sync time | No assignments created for tomorrow | Retry 3× with 5-min backoff. Alert HQ Slack. Champions still receive brief with yesterday's guidance if no assignments found. |
| Twilio webhook delivery failure | Capture lost | Twilio retries automatically up to 3× over 30 min. S3 upload attempted before Airtable push. |
| Airtable API rate limit | Capture data not in review queue | Queue Airtable writes with exponential backoff. 10,000 records/month is generous for Phase 1 volume. |
| S3 upload failure | Media lost permanently | Twilio media URLs valid for 4 hours. If S3 upload fails, retry immediately once. If second failure, SMS champion: "Your capture couldn't be saved — please resend." |
| Database connection failure | Service degraded | Connection pool with max retry. Alert PagerDuty. |

### Monitoring (Phase 1)
- **CloudWatch Logs**: All service logs shipped to CloudWatch
- **CloudWatch Alarms**: Capture webhook response time >5s, S3 upload failure rate >1%, scheduler job failure
- **Slack Alerts** (`#content-ops`): Daily sync summary (practices synced, assignments created), failed captures, Airtable push errors

---

## 11. Security & HIPAA Compliance

| Requirement | Implementation |
|-------------|----------------|
| Patient data encryption at rest | RDS: AES-256. S3: SSE-KMS. |
| Patient data encryption in transit | TLS 1.3 everywhere. Twilio uses HTTPS. |
| PHI in SMS messages | Patient initials only in SMS (not full name). Procedure type generic ("dental procedure") not specific in SMS. |
| Consent verification before capture | `capture_assignments` only created when `consents` has active record for patient_id + practice_id |
| BAA required | AWS (signed), Twilio (sign HIPAA BAA via Twilio console), Airtable (Enterprise tier has BAA — confirm before Phase 1 launch) |
| Access logging | S3 access logging. RDS audit logging. CloudTrail for all AWS API calls. |
| Media URL expiry | S3 presigned URLs for Airtable set to 7-day expiry. Do not use public S3 URLs. |

---

## 12. Phase 1 Deployment Checklist

### Pre-Launch (Week 1)
- [ ] AWS ECS cluster provisioned in `us-east-1`
- [ ] RDS PostgreSQL instance created and schema migrated
- [ ] S3 buckets created with encryption + access logging
- [ ] Twilio account configured; SMS + WhatsApp numbers purchased
- [ ] WhatsApp Business API approval submitted
- [ ] Sikka ONE API credentials obtained; test connection
- [ ] Airtable base created with table structure above
- [ ] Secrets loaded to AWS Secrets Manager
- [ ] Services deployed to ECS and health checks passing
- [ ] Webhook URL registered with Twilio
- [ ] Twilio signature validation confirmed working

### Practice Onboarding (Week 2–3, per practice)
- [ ] Practice record created in Admin Service
- [ ] Content champion user created with phone number
- [ ] Playbook type assigned
- [ ] Sikka practice ID confirmed and test sync run
- [ ] Champion briefed: send test SMS, champion replies with test photo
- [ ] Capture shows up in Airtable within 30 seconds

### Go-Live (Week 4)
- [ ] 10–15 practices active
- [ ] Daily brief SMS firing at correct 7:30 AM local time per practice
- [ ] Airtable editorial workflow running (2 human editors)
- [ ] Slack `#content-ops` alerts configured
- [ ] Week 1 daily volume baseline established (expected: 30–50 captures/day across 10 practices)
