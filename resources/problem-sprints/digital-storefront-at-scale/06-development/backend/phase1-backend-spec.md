# Phase 1 Backend Specification

**Role**: Backend Engineer
**Sprint**: Digital Storefront at Scale
**Date**: 2026-03-10
**Phase 1 Scope**: Enrollment flow end-to-end for 20-30 pilot practices
**Source Artifacts**: `04-architect/product-architecture.md`, `06-development/technical/implementation-plan.md`

---

## 1. API Endpoint Details (Phase 1)

### 1.1 Practice CRUD

#### `GET /api/v1/practices`

List practices with filtering and pagination.

**Query Parameters**:
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 25 | Items per page (max 100) |
| `status` | string | — | Filter by enrollment_status |
| `specialty` | string | — | Filter by specialty enum |
| `platform` | string | — | Filter by platform enum |
| `search` | string | — | Full-text search on name, city |
| `sort` | string | `name` | Sort field |
| `order` | string | `asc` | Sort direction |

**Response** (`200`):
```json
{
  "data": [
    {
      "id": "uuid",
      "slug": "bright-smile-dental",
      "name": "Bright Smile Dental",
      "specialty": "GP",
      "platform": "SGA",
      "city": "Austin",
      "state": "TX",
      "enrollment_status": "enrolled",
      "health_overall": 72,
      "concierge_last_interaction": "2026-03-09T14:30:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 25,
    "total": 267,
    "totalPages": 11
  }
}
```

#### `GET /api/v1/practices/:id`

Full practice profile including nested team members, services, and health scores.

**Response** (`200`):
```json
{
  "id": "uuid",
  "slug": "bright-smile-dental",
  "name": "Bright Smile Dental",
  "legal_name": "Bright Smile Dental LLC",
  "platform": "SGA",
  "specialty": "GP",
  "address": {
    "line1": "123 Main St",
    "line2": null,
    "city": "Austin",
    "state": "TX",
    "zip": "78701"
  },
  "phone": "(512) 555-0100",
  "email": "info@brightsmile.com",
  "hours": {
    "mon": "8:00-17:00",
    "tue": "8:00-17:00",
    "wed": "8:00-17:00",
    "thu": "8:00-17:00",
    "fri": "8:00-14:00",
    "sat": null,
    "sun": null
  },
  "brand": {
    "primary_color": "#2563eb",
    "secondary_color": "#1e40af",
    "logo_url": "https://r2.sga.com/bright-smile/logo.png",
    "photos": ["https://r2.sga.com/bright-smile/exterior.jpg"]
  },
  "digital": {
    "website_url": "https://brightsmile.sgadental.com",
    "website_template_id": "gp-v1",
    "website_status": "active",
    "website_last_deployed": "2026-03-08T10:00:00Z",
    "gmb_listing_id": null,
    "gmb_status": "not_setup",
    "gmb_completeness": 0,
    "social_instagram": null,
    "social_facebook_url": null,
    "social_status": "not_setup"
  },
  "enrollment": {
    "status": "enrolled",
    "invited_date": "2026-03-01T09:00:00Z",
    "enrolled_date": "2026-03-05T16:20:00Z",
    "method": "concierge"
  },
  "health": {
    "overall": 72,
    "website": 85,
    "gmb": 0,
    "social": 0,
    "reviews": 68,
    "leads": 45,
    "updated_at": "2026-03-09T00:00:00Z"
  },
  "concierge": {
    "last_interaction": "2026-03-09T14:30:00Z",
    "interaction_count": 4,
    "satisfaction": 4.5
  },
  "team": [
    {
      "id": "uuid",
      "name": "Dr. Sarah Chen",
      "role": "dentist",
      "title": "Lead Dentist",
      "bio": "Dr. Chen has been practicing...",
      "photo_url": "https://r2.sga.com/bright-smile/chen.jpg",
      "is_active": true
    }
  ],
  "services": [
    {
      "id": "uuid",
      "name": "General Dentistry",
      "description": "Comprehensive dental care...",
      "is_primary": true
    }
  ]
}
```

#### `POST /api/v1/practices`

**Request Body**:
```json
{
  "name": "Bright Smile Dental",
  "platform": "SGA",
  "specialty": "GP",
  "address_line1": "123 Main St",
  "city": "Austin",
  "state": "TX",
  "zip": "78701",
  "phone": "(512) 555-0100"
}
```
- `name`, `platform`, and `specialty` are required.
- `slug` is auto-generated from name (kebab-case, unique).

**Response** (`201`): Full practice object.

#### `PATCH /api/v1/practices/:id`

Partial update. Only include fields being changed. Triggers a `change_event` record for each modified field. Invalidates Redis cache.

**Request Body** (example):
```json
{
  "phone": "(512) 555-0200",
  "hours": { "fri": "8:00-16:00" }
}
```

**Response** (`200`): Updated practice object.

**Side effects**:
1. Insert `change_event` with `{ field: { old, new } }` diff
2. Invalidate `practice:{id}:profile` in Redis
3. Emit `practice.updated` event to BullMQ (consumed by sync workers in Phase 2)

---

### 1.2 Enrollment Flow

#### `POST /api/v1/enrollment/invite`

Send enrollment invitation to one or more practices.

**Request Body**:
```json
{
  "practice_ids": ["uuid1", "uuid2"],
  "message_template": "default"
}
```

**Behavior**:
1. Validate all practice IDs exist and are in `pending` status
2. Generate unique enrollment link per practice (`/enroll/{practice_slug}/{token}`)
3. Send email via Resend (practice email address)
4. Update `enrollment_status` → `invited`, set `enrollment_invited` timestamp
5. Insert `enrollment_event` (type: `invited`)

**Response** (`200`):
```json
{
  "invited": 2,
  "skipped": 0,
  "results": [
    { "practice_id": "uuid1", "status": "invited", "link": "https://concierge.sga.com/enroll/bright-smile/abc123" },
    { "practice_id": "uuid2", "status": "invited", "link": "https://concierge.sga.com/enroll/pearl-dental/def456" }
  ]
}
```

#### `GET /api/v1/enrollment/:practiceId/status`

**Response** (`200`):
```json
{
  "practice_id": "uuid",
  "status": "in_progress",
  "steps": {
    "invited": { "completed": true, "at": "2026-03-01T09:00:00Z" },
    "data_verified": { "completed": true, "at": "2026-03-03T11:15:00Z" },
    "template_selected": { "completed": false, "at": null },
    "storefront_approved": { "completed": false, "at": null },
    "deployed": { "completed": false, "at": null }
  },
  "current_step": "template_selected"
}
```

#### `POST /api/v1/enrollment/:practiceId/verify`

Submit verified/corrected practice data from the concierge conversation. Called by the Concierge Agent via tool_use when practice confirms their data.

**Request Body**:
```json
{
  "verified_fields": ["name", "address", "phone", "hours"],
  "corrections": {
    "team": {
      "remove": [{ "name": "Dr. Miller", "reason": "left_practice" }],
      "add": [{ "name": "Sarah Johnson", "role": "hygienist" }]
    },
    "services": {
      "add": [{ "name": "Invisalign", "is_primary": false }]
    }
  }
}
```

**Behavior**:
1. Apply corrections to practice record + team_member/service tables
2. Update data_source → `concierge`, data_confidence → `high` for verified fields
3. Insert `enrollment_event` (type: `verified_data`)
4. Update `enrollment_status` → `in_progress`
5. Invalidate Redis cache

#### `POST /api/v1/enrollment/:practiceId/select-template`

**Request Body**:
```json
{
  "template_id": "gp-v1",
  "customizations": {
    "primary_color": "#2563eb",
    "hero_image": "exterior"
  }
}
```

**Behavior**:
1. Validate template_id exists and matches practice specialty
2. Store selection on practice record (`website_template_id`)
3. Store customizations in `brand` JSONB
4. Generate preview and return preview URL
5. Insert `enrollment_event` (type: `selected_template`)

**Response** (`200`):
```json
{
  "template_id": "gp-v1",
  "preview_url": "https://preview.sga.com/bright-smile-dental/gp-v1",
  "status": "preview_ready"
}
```

#### `POST /api/v1/enrollment/:practiceId/approve`

Practice approves the storefront for deployment.

**Behavior**:
1. Insert `enrollment_event` (type: `approved`)
2. Add QC notification to HQ review queue
3. After QC window (24h or manual approval), trigger deployment
4. Update `enrollment_status` → `enrolled`, set `enrollment_enrolled`

#### `POST /api/v1/enrollment/:practiceId/deploy`

Trigger site generation and deployment. Called after QC approval.

**Behavior**:
1. Enqueue site generation job in BullMQ `generate` queue
2. Job: pull practice data → render Next.js template → build static output → deploy to Cloudflare Pages
3. On success: update `website_status` → `active`, set `website_last_deployed`
4. Insert `enrollment_event` (type: `deployed`)
5. Notify concierge to send "you're live!" message

**Response** (`202`):
```json
{
  "job_id": "uuid",
  "status": "queued",
  "estimated_time": "2-3 minutes"
}
```

---

### 1.3 Harvester Trigger

#### `POST /api/v1/harvester/run`

**Request Body**:
```json
{
  "practice_id": "uuid",
  "sources": ["website", "gmb", "social"],
  "known_urls": {
    "website": "https://www.brightsmile.com",
    "facebook": "https://facebook.com/brightsmile"
  }
}
```

**Behavior**:
1. Enqueue harvest job in BullMQ `harvest` queue
2. Harvest worker (Python) runs Playwright + Claude extraction
3. Results written to Practice Data Hub with confidence scores
4. Returns job ID for status polling

**Response** (`202`):
```json
{
  "job_id": "uuid",
  "status": "queued",
  "practice_id": "uuid"
}
```

#### `GET /api/v1/harvester/:practiceId/results`

**Response** (`200`):
```json
{
  "practice_id": "uuid",
  "harvested_at": "2026-03-01T12:00:00Z",
  "status": "complete",
  "fields": {
    "name": { "value": "Bright Smile Dental", "confidence": "high", "source": "website" },
    "phone": { "value": "(512) 555-0100", "confidence": "high", "source": "gmb" },
    "team": {
      "value": [
        { "name": "Dr. Sarah Chen", "role": "dentist", "confidence": "high" },
        { "name": "Dr. James Miller", "role": "dentist", "confidence": "medium" }
      ],
      "source": "website"
    },
    "services": {
      "value": ["General Dentistry", "Cosmetic Dentistry", "Invisalign"],
      "confidence": "medium",
      "source": "website"
    }
  },
  "discrepancies": [
    {
      "field": "phone",
      "website_value": "(512) 555-0100",
      "gmb_value": "(512) 555-0101",
      "recommended": "(512) 555-0100",
      "reason": "Website value is more recent"
    }
  ]
}
```

---

### 1.4 Template Generation

#### `GET /api/v1/templates`

**Response** (`200`):
```json
{
  "templates": [
    {
      "id": "gp-v1",
      "name": "General Practice",
      "specialty": "GP",
      "description": "Appointment-booking optimized, new patient focused",
      "version": 1,
      "preview_url": "https://templates.sga.com/gp-v1/preview",
      "is_active": true
    },
    {
      "id": "perio-v1",
      "name": "Periodontics",
      "specialty": "Perio",
      "description": "Referral-optimized, specialist credibility",
      "version": 1,
      "preview_url": "https://templates.sga.com/perio-v1/preview",
      "is_active": true
    }
  ]
}
```

#### `GET /api/v1/templates/:id/preview/:practiceId`

Generates an on-demand preview of the template populated with the practice's data.

**Response** (`200`):
```json
{
  "preview_url": "https://preview.sga.com/bright-smile-dental/gp-v1",
  "expires_at": "2026-03-10T12:00:00Z",
  "pages": ["home", "about", "services", "team", "contact"]
}
```

---

### 1.5 Concierge Conversation

#### `POST /api/v1/concierge/:practiceId/message`

Send a message to the practice's concierge agent. Returns a streaming response (Server-Sent Events).

**Request Body**:
```json
{
  "conversation_id": "uuid-or-null",
  "message": "Dr. Miller left our practice last month",
  "channel": "web"
}
```

**Behavior**:
1. If `conversation_id` is null, create new conversation record
2. Load practice context from Redis cache
3. Build Claude API request (system prompt + conversation history + new message)
4. Stream response via SSE
5. If Claude invokes tools, execute them and continue the conversation loop
6. Persist all messages to `conversation_message` table
7. Update `concierge_last_interaction` and `concierge_interaction_count`

**SSE Response Stream**:
```
event: message_start
data: {"conversation_id": "uuid", "message_id": "uuid"}

event: content_delta
data: {"text": "Got it — I'll "}

event: content_delta
data: {"text": "remove Dr. Miller from everything. "}

event: tool_use
data: {"tool": "remove_team_member", "input": {"name": "Dr. Miller"}, "status": "executing"}

event: tool_result
data: {"tool": "remove_team_member", "result": "success", "details": "Removed from website, marked inactive"}

event: content_delta
data: {"text": "Done! I've removed Dr. Miller from your website team page. "}

event: action_card
data: {"type": "confirmation", "title": "Staff Change", "items": ["Website team page updated", "GMB listing queued for update"], "actions": [{"label": "Looks good", "value": "confirm"}, {"label": "Undo", "value": "undo"}]}

event: message_end
data: {"message_id": "uuid", "tokens": {"input": 1250, "output": 180}}
```

#### `GET /api/v1/concierge/:practiceId/conversations`

**Response** (`200`):
```json
{
  "conversations": [
    {
      "id": "uuid",
      "started_at": "2026-03-05T16:00:00Z",
      "ended_at": "2026-03-05T16:22:00Z",
      "intent": "enrollment",
      "summary": "Completed enrollment — verified data, selected GP template, approved for deployment",
      "message_count": 12,
      "resolved": true
    }
  ]
}
```

---

### 1.6 Basic Dashboard Data

#### `GET /api/v1/dashboard/summary`

**Response** (`200`):
```json
{
  "total_practices": 267,
  "enrollment": {
    "pending": 37,
    "invited": 180,
    "in_progress": 12,
    "enrolled": 28,
    "blitz_deployed": 0,
    "inactive": 10
  },
  "health": {
    "network_average": 42,
    "optimized": 8,
    "baseline": 20,
    "at_risk": 239
  },
  "recent_activity": [
    {
      "practice": "Bright Smile Dental",
      "action": "enrolled",
      "at": "2026-03-09T14:30:00Z"
    }
  ],
  "alerts_count": 3
}
```

#### `GET /api/v1/dashboard/alerts`

**Response** (`200`):
```json
{
  "alerts": [
    {
      "id": "uuid",
      "type": "review_drop",
      "severity": "warning",
      "practice_id": "uuid",
      "practice_name": "Oak Park Dental",
      "message": "Google review rating dropped from 4.5 to 3.9 in 30 days",
      "detected_at": "2026-03-09T06:00:00Z",
      "acknowledged": false
    }
  ]
}
```

---

## 2. ORM Schema (Prisma)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum Platform {
  SGA
  Gen4
  MODIS
}

enum Specialty {
  GP
  Perio
  FullArch
  Pediatric
  Multi
}

enum EnrollmentStatus {
  pending
  invited
  in_progress
  enrolled
  blitz_deployed
  inactive
}

enum EnrollmentMethod {
  concierge
  blitz
}

enum ChannelStatus {
  active
  pending
  disconnected
  not_setup
}

enum TeamRole {
  dentist
  hygienist
  assistant
  office_manager
  specialist
  admin
  other
}

enum ConfidenceLevel {
  high
  medium
  low
}

enum DataSource {
  harvested
  concierge
  manual
  isolved
  api_sync
}

enum ConversationChannel {
  web
  sms
  email
}

enum MessageRole {
  user
  assistant
  system
  tool
}

model Practice {
  id        String   @id @default(uuid())
  slug      String   @unique
  name      String
  legalName String?  @map("legal_name")
  platform  Platform @default(SGA)
  specialty Specialty @default(GP)

  // Location
  addressLine1 String?  @map("address_line1")
  addressLine2 String?  @map("address_line2")
  city         String?
  state        String?
  zip          String?
  phone        String?
  email        String?
  hours        Json?
  timezone     String   @default("America/Chicago")

  // Brand
  brand Json @default("{}")

  // Digital Presence
  websiteUrl          String?       @map("website_url")
  websiteTemplateId   String?       @map("website_template_id")
  websiteStatus       ChannelStatus @default(not_setup) @map("website_status")
  websiteLastDeployed DateTime?     @map("website_last_deployed")
  gmbListingId        String?       @map("gmb_listing_id")
  gmbStatus           ChannelStatus @default(not_setup) @map("gmb_status")
  gmbCompleteness     Int           @default(0) @map("gmb_completeness")
  socialInstagram     String?       @map("social_instagram")
  socialFacebookUrl   String?       @map("social_facebook_url")
  socialStatus        ChannelStatus @default(not_setup) @map("social_status")
  crmSystem           String?       @map("crm_system")
  crmStatus           ChannelStatus @default(not_setup) @map("crm_status")

  // Enrollment
  enrollmentStatus   EnrollmentStatus @default(pending) @map("enrollment_status")
  enrollmentInvited  DateTime?        @map("enrollment_invited")
  enrollmentEnrolled DateTime?        @map("enrollment_enrolled")
  enrollmentMethod   EnrollmentMethod? @map("enrollment_method")

  // Health Scores
  healthOverall   Int       @default(0) @map("health_overall")
  healthWebsite   Int       @default(0) @map("health_website")
  healthGmb       Int       @default(0) @map("health_gmb")
  healthSocial    Int       @default(0) @map("health_social")
  healthReviews   Int       @default(0) @map("health_reviews")
  healthLeads     Int       @default(0) @map("health_leads")
  healthUpdatedAt DateTime? @map("health_updated_at")

  // Concierge
  conciergeLastInteraction  DateTime? @map("concierge_last_interaction")
  conciergeInteractionCount Int       @default(0) @map("concierge_interaction_count")
  conciergeSatisfaction     Decimal?  @map("concierge_satisfaction") @db.Decimal(3, 2)

  // Metadata
  dataSource     DataSource      @default(manual) @map("data_source")
  dataConfidence ConfidenceLevel @default(low) @map("data_confidence")
  createdAt      DateTime        @default(now()) @map("created_at")
  updatedAt      DateTime        @updatedAt @map("updated_at")

  // Relations
  teamMembers      TeamMember[]
  services         Service[]
  enrollmentEvents EnrollmentEvent[]
  conversations    Conversation[]
  healthSnapshots  HealthScoreSnapshot[]
  syncLogs         SyncLog[]
  changeEvents     ChangeEvent[]

  @@index([platform])
  @@index([specialty])
  @@index([enrollmentStatus])
  @@index([healthOverall])
  @@map("practice")
}

model TeamMember {
  id         String          @id @default(uuid())
  practiceId String          @map("practice_id")
  name       String
  role       TeamRole
  title      String?
  bio        String?
  photoUrl   String?         @map("photo_url")
  startDate  DateTime?       @map("start_date") @db.Date
  endDate    DateTime?       @map("end_date") @db.Date
  isActive   Boolean         @default(true) @map("is_active")
  sortOrder  Int             @default(0) @map("sort_order")
  source     DataSource      @default(manual)
  confidence ConfidenceLevel @default(medium)
  createdAt  DateTime        @default(now()) @map("created_at")
  updatedAt  DateTime        @updatedAt @map("updated_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId])
  @@index([practiceId, isActive])
  @@map("team_member")
}

model Service {
  id         String     @id @default(uuid())
  practiceId String     @map("practice_id")
  name       String
  description String?
  isPrimary  Boolean    @default(false) @map("is_primary")
  sortOrder  Int        @default(0) @map("sort_order")
  source     DataSource @default(manual)
  createdAt  DateTime   @default(now()) @map("created_at")
  updatedAt  DateTime   @updatedAt @map("updated_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId])
  @@map("service")
}

model Template {
  id          String    @id
  name        String
  specialty   Specialty
  description String?
  version     Int       @default(1)
  isActive    Boolean   @default(true) @map("is_active")
  config      Json      @default("{}")
  previewUrl  String?   @map("preview_url")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")

  @@map("template")
}

model EnrollmentEvent {
  id         String   @id @default(uuid())
  practiceId String   @map("practice_id")
  eventType  String   @map("event_type")
  metadata   Json     @default("{}")
  createdAt  DateTime @default(now()) @map("created_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId])
  @@index([eventType])
  @@map("enrollment_event")
}

model Conversation {
  id         String              @id @default(uuid())
  practiceId String              @map("practice_id")
  channel    ConversationChannel @default(web)
  startedAt  DateTime            @default(now()) @map("started_at")
  endedAt    DateTime?           @map("ended_at")
  summary    String?
  intent     String?
  resolved   Boolean             @default(false)
  metadata   Json                @default("{}")

  practice Practice              @relation(fields: [practiceId], references: [id], onDelete: Cascade)
  messages ConversationMessage[]

  @@index([practiceId])
  @@map("conversation")
}

model ConversationMessage {
  id             String      @id @default(uuid())
  conversationId String      @map("conversation_id")
  role           MessageRole
  content        String
  toolCalls      Json?       @map("tool_calls")
  tokensInput    Int?        @map("tokens_input")
  tokensOutput   Int?        @map("tokens_output")
  createdAt      DateTime    @default(now()) @map("created_at")

  conversation Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)

  @@index([conversationId])
  @@map("conversation_message")
}

model HealthScoreSnapshot {
  id         String   @id @default(uuid())
  practiceId String   @map("practice_id")
  overall    Int
  website    Int
  gmb        Int
  social     Int
  reviews    Int
  leads      Int
  capturedAt DateTime @default(now()) @map("captured_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId, capturedAt])
  @@map("health_score_snapshot")
}

model SyncLog {
  id          String    @id @default(uuid())
  practiceId  String    @map("practice_id")
  channel     String
  action      String
  status      String
  payload     Json?
  error       String?
  attempts    Int       @default(1)
  createdAt   DateTime  @default(now()) @map("created_at")
  completedAt DateTime? @map("completed_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId])
  @@index([status])
  @@map("sync_log")
}

model ChangeEvent {
  id          String     @id @default(uuid())
  practiceId  String     @map("practice_id")
  entityType  String     @map("entity_type")
  entityId    String?    @map("entity_id")
  action      String
  changes     Json
  triggeredBy DataSource @map("triggered_by")
  syncStatus  String     @default("pending") @map("sync_status")
  createdAt   DateTime   @default(now()) @map("created_at")

  practice Practice @relation(fields: [practiceId], references: [id], onDelete: Cascade)

  @@index([practiceId])
  @@index([syncStatus])
  @@map("change_event")
}
```

---

## 3. Service Layer Design

### 3.1 Service Architecture

```
src/
├── services/
│   ├── harvester.service.ts      # Data harvesting orchestration
│   ├── enrollment.service.ts     # Enrollment flow state machine
│   ├── conversation.service.ts   # Concierge agent conversation management
│   ├── template.service.ts       # Template rendering and site generation
│   ├── sync.service.ts           # Change event propagation
│   ├── practice.service.ts       # Practice CRUD with cache management
│   └── dashboard.service.ts      # Aggregated dashboard queries
├── queues/
│   ├── harvest.queue.ts          # Harvest job definitions
│   ├── generate.queue.ts         # Site generation job definitions
│   └── sync.queue.ts             # Channel sync job definitions
├── lib/
│   ├── claude.ts                 # Claude API client wrapper
│   ├── cache.ts                  # Redis cache abstraction
│   ├── events.ts                 # Change event emitter
│   └── auth.ts                   # API key + session auth
└── routes/
    ├── practices.ts
    ├── enrollment.ts
    ├── harvester.ts
    ├── templates.ts
    ├── concierge.ts
    └── dashboard.ts
```

### 3.2 HarvesterService

Orchestrates data collection from public sources and writes results to Practice Data Hub.

```typescript
class HarvesterService {
  /**
   * Trigger a harvest for a single practice. Enqueues a job
   * that runs the Python harvester subprocess.
   */
  async harvest(practiceId: string, options: {
    sources: ('website' | 'gmb' | 'social')[];
    knownUrls?: Record<string, string>;
  }): Promise<{ jobId: string }>;

  /**
   * Process harvest results from the Python worker.
   * Writes fields to practice record with confidence scores.
   * Flags discrepancies between sources.
   */
  async processResults(practiceId: string, results: HarvestResult): Promise<void>;

  /**
   * Get the latest harvest results for a practice,
   * including per-field confidence and discrepancies.
   */
  async getResults(practiceId: string): Promise<HarvestResult | null>;
}
```

**Python Harvester Worker** (separate process, communicates via BullMQ):
1. Receives job with practice name, address, known URLs
2. Launches Playwright browser
3. Scrapes practice website (team page, services page, about page, contact page)
4. Sends raw HTML to Claude API with extraction prompt
5. Queries Google Places API for listing data
6. Reconciles data from multiple sources
7. Returns structured `HarvestResult` with confidence scores

### 3.3 EnrollmentService

Manages the enrollment state machine for each practice.

```typescript
class EnrollmentService {
  /**
   * State machine transitions:
   * pending → invited → in_progress → enrolled → [deployed]
   * pending → blitz_deployed (after 30-day timeout)
   */

  /** Send invitation email and update status */
  async invite(practiceIds: string[]): Promise<InviteResult[]>;

  /** Get current enrollment status with step completion */
  async getStatus(practiceId: string): Promise<EnrollmentStatus>;

  /** Process verified data from concierge conversation */
  async verifyData(practiceId: string, verification: DataVerification): Promise<void>;

  /** Record template selection */
  async selectTemplate(practiceId: string, templateId: string, customizations?: Json): Promise<{
    previewUrl: string;
  }>;

  /** Practice approves storefront — enters QC queue */
  async approve(practiceId: string): Promise<void>;

  /** Trigger deployment after QC */
  async deploy(practiceId: string): Promise<{ jobId: string }>;

  /** Blitz deploy for non-engaged practices */
  async blitzDeploy(practiceIds: string[]): Promise<BlitzResult[]>;

  /** Check for practices past 30-day invitation window */
  async checkBlitzEligible(): Promise<string[]>;
}
```

### 3.4 ConversationService

Manages concierge agent conversations, Claude API calls, and tool execution.

```typescript
class ConversationService {
  /**
   * Process a user message and stream the agent response.
   * Handles tool execution loop internally.
   */
  async processMessage(
    practiceId: string,
    conversationId: string | null,
    message: string,
    channel: ConversationChannel
  ): AsyncGenerator<ConversationEvent>;

  /** Build the full Claude API request with context */
  private buildRequest(
    practice: PracticeWithRelations,
    conversationHistory: ConversationMessage[],
    newMessage: string
  ): ClaudeRequest;

  /** Execute a tool call from Claude and return the result */
  private executeTool(
    practiceId: string,
    toolName: string,
    toolInput: Record<string, unknown>
  ): Promise<ToolResult>;

  /** List conversations for a practice */
  async listConversations(practiceId: string): Promise<ConversationSummary[]>;

  /** Get full conversation with messages */
  async getConversation(conversationId: string): Promise<ConversationWithMessages>;
}
```

**Conversation event types** (yielded by the async generator):

```typescript
type ConversationEvent =
  | { type: 'message_start'; conversationId: string; messageId: string }
  | { type: 'content_delta'; text: string }
  | { type: 'tool_use'; tool: string; input: Record<string, unknown>; status: string }
  | { type: 'tool_result'; tool: string; result: unknown }
  | { type: 'action_card'; card: ActionCard }
  | { type: 'message_end'; messageId: string; tokens: { input: number; output: number } };
```

### 3.5 TemplateService

Handles template management, preview generation, and site builds.

```typescript
class TemplateService {
  /** List available templates, optionally filtered by specialty */
  async listTemplates(specialty?: Specialty): Promise<Template[]>;

  /** Generate a preview site with practice data */
  async generatePreview(practiceId: string, templateId: string): Promise<{
    previewUrl: string;
    expiresAt: Date;
  }>;

  /** Build and deploy a practice site */
  async buildAndDeploy(practiceId: string): Promise<{
    jobId: string;
    estimatedTime: number;
  }>;

  /** Batch regenerate all sites using a given template */
  async batchRegenerate(templateId: string): Promise<{
    jobId: string;
    practiceCount: number;
  }>;

  /** Get deployment status for a practice site */
  async getDeploymentStatus(practiceId: string): Promise<DeploymentStatus>;
}
```

### 3.6 SyncService

Processes change events and propagates updates to external channels. Phase 1 handles website regeneration only; GMB and social sync come in Phase 2.

```typescript
class SyncService {
  /** Record a change event and queue sync jobs */
  async recordChange(change: {
    practiceId: string;
    entityType: string;
    entityId?: string;
    action: 'created' | 'updated' | 'deleted';
    changes: Record<string, { old: unknown; new: unknown }>;
    triggeredBy: DataSource;
  }): Promise<void>;

  /** Get impact analysis for a pending change */
  async analyzeImpact(changeId: string): Promise<ImpactAnalysis>;

  /** Process a sync job (called by BullMQ worker) */
  async processSync(job: SyncJob): Promise<SyncResult>;

  /** Run nightly reconciliation */
  async reconcile(): Promise<ReconciliationReport>;
}
```

---

## 4. Claude API Integration Pattern

### 4.1 System Prompt Structure

The concierge system prompt is composed of three layers, assembled per-practice on each conversation.

```typescript
function buildSystemPrompt(practice: PracticeWithRelations): string {
  return [
    BASE_PROMPT,
    getSpecialtyContext(practice.specialty),
    getPracticeContext(practice),
  ].join('\n\n---\n\n');
}
```

**Layer 1 — BASE_PROMPT** (shared across all practices):

```
You are the SGA Marketing Concierge — a knowledgeable, warm, and opinionated AI assistant
for dental practices in the SGA Dental Partners network.

## Personality
- Sage: You provide expert guidance backed by data and dental marketing best practices
- Caregiver: You are warm, protective, and genuinely invested in each practice's success
- Opinionated: You always make a recommendation with clear reasoning. Never present options
  without a recommendation.
- Efficient: Practice staff have 5-minute windows between patients. Be concise.
- Never bureaucratic: You handle things. You don't create tickets or redirect to other teams.

## Capabilities
You can help practices with:
- Enrollment: verify practice data, select website template, approve storefront
- Change management: update team members, services, hours, contact info
- Questions: answer questions about SGA programs, marketing best practices, digital presence
- Status: report on website performance, review scores, digital health

## Guardrails
- All template and color options are SGA-approved. Present them as choices, not limitations.
- Never make up data. If you don't know something, say so and offer to find out.
- When suggesting changes, always explain what will be updated and where.
- For staff removals, always confirm the effective date before processing.
```

**Layer 2 — SPECIALTY_CONTEXT** (one per specialty):

```typescript
const SPECIALTY_CONTEXTS: Record<Specialty, string> = {
  GP: `
## Specialty Context: General Practice
- Focus: new patient acquisition, appointment booking optimization
- Key metrics: new patient calls, website form submissions, Google review volume
- Template recommendation: GP template (appointment-booking optimized)
- Common services: cleanings, fillings, crowns, root canals, cosmetic, Invisalign
- Marketing priorities: local SEO, Google Business Profile completeness, review generation
  `,
  Perio: `
## Specialty Context: Periodontics
- Focus: referral optimization, specialist credibility
- Key metrics: referral source tracking, case acceptance rate
- Template recommendation: Perio template (referral-optimized)
- Referring GP practices in network: consider cross-promotion opportunities
- Marketing priorities: referring doctor relationships, clinical credibility content
  `,
  // ... FullArch, Pediatric, Multi
};
```

**Layer 3 — PRACTICE_CONTEXT** (unique per practice, loaded from Redis):

```typescript
function getPracticeContext(practice: PracticeWithRelations): string {
  return `
## Your Practice: ${practice.name}
- Location: ${practice.city}, ${practice.state}
- Platform: ${practice.platform}
- Specialty: ${practice.specialty}
- Enrollment Status: ${practice.enrollmentStatus}

## Team (${practice.teamMembers.filter(t => t.isActive).length} active)
${practice.teamMembers.filter(t => t.isActive).map(t =>
  `- ${t.name} (${t.role}${t.title ? ', ' + t.title : ''})`
).join('\n')}

## Services (${practice.services.length} listed)
${practice.services.map(s =>
  `- ${s.name}${s.isPrimary ? ' [PRIMARY]' : ''}`
).join('\n')}

## Digital Presence
- Website: ${practice.websiteUrl || 'Not set up'} (${practice.websiteStatus})
- GMB: ${practice.gmbStatus} (completeness: ${practice.gmbCompleteness}%)
- Social: ${practice.socialStatus}

## Health Scores
- Overall: ${practice.healthOverall}/100
- Website: ${practice.healthWebsite} | GMB: ${practice.healthGmb} | Social: ${practice.healthSocial}
- Reviews: ${practice.healthReviews} | Leads: ${practice.healthLeads}

## Recent Interactions
- Last interaction: ${practice.conciergeLastInteraction || 'Never'}
- Total interactions: ${practice.conciergeInteractionCount}
  `;
}
```

### 4.2 Tool Definitions

Tools the concierge can invoke during conversations. Defined using Claude's tool_use format.

```typescript
const CONCIERGE_TOOLS: Tool[] = [
  {
    name: "verify_practice_data",
    description: "Mark practice data fields as verified by the practice. Call this when the practice confirms their information is correct.",
    input_schema: {
      type: "object",
      properties: {
        verified_fields: {
          type: "array",
          items: { type: "string" },
          description: "List of field names confirmed correct (e.g., 'name', 'phone', 'address', 'hours')"
        }
      },
      required: ["verified_fields"]
    }
  },
  {
    name: "update_team_member",
    description: "Add, update, or remove a team member. Use 'remove' action to mark someone as no longer active.",
    input_schema: {
      type: "object",
      properties: {
        action: { type: "string", enum: ["add", "update", "remove"] },
        name: { type: "string" },
        role: { type: "string", enum: ["dentist", "hygienist", "assistant", "office_manager", "specialist", "admin", "other"] },
        title: { type: "string" },
        effective_date: { type: "string", description: "ISO date for when the change takes effect" }
      },
      required: ["action", "name"]
    }
  },
  {
    name: "update_service",
    description: "Add or remove a service from the practice listing.",
    input_schema: {
      type: "object",
      properties: {
        action: { type: "string", enum: ["add", "remove"] },
        name: { type: "string" },
        description: { type: "string" },
        is_primary: { type: "boolean" }
      },
      required: ["action", "name"]
    }
  },
  {
    name: "select_template",
    description: "Record the practice's template selection. Always recommend a template first with reasoning before calling this.",
    input_schema: {
      type: "object",
      properties: {
        template_id: { type: "string" },
        customizations: {
          type: "object",
          properties: {
            primary_color: { type: "string" },
            hero_image: { type: "string" }
          }
        }
      },
      required: ["template_id"]
    }
  },
  {
    name: "approve_storefront",
    description: "Record that the practice has approved their storefront for deployment. Only call this after showing them the preview and getting explicit approval.",
    input_schema: {
      type: "object",
      properties: {
        confirmation: { type: "boolean" },
        notes: { type: "string" }
      },
      required: ["confirmation"]
    }
  },
  {
    name: "get_practice_health",
    description: "Retrieve current health scores and key metrics for the practice.",
    input_schema: {
      type: "object",
      properties: {}
    }
  },
  {
    name: "show_template_preview",
    description: "Generate and display a template preview populated with the practice's data.",
    input_schema: {
      type: "object",
      properties: {
        template_id: { type: "string" }
      },
      required: ["template_id"]
    }
  }
];
```

### 4.3 Conversation Management

```typescript
async function* processMessage(
  practiceId: string,
  conversationId: string | null,
  userMessage: string
): AsyncGenerator<ConversationEvent> {
  // 1. Load or create conversation
  const conversation = conversationId
    ? await db.conversation.findUnique({ where: { id: conversationId }, include: { messages: true } })
    : await db.conversation.create({ data: { practiceId, channel: 'web' } });

  // 2. Load practice context from Redis (fallback to Postgres)
  const practice = await cache.getPracticeProfile(practiceId);

  // 3. Persist user message
  await db.conversationMessage.create({
    data: { conversationId: conversation.id, role: 'user', content: userMessage }
  });

  yield { type: 'message_start', conversationId: conversation.id, messageId: '' };

  // 4. Build Claude request
  const systemPrompt = buildSystemPrompt(practice);
  const messages = buildMessageHistory(conversation.messages, userMessage);

  // 5. Tool execution loop
  let continueLoop = true;
  while (continueLoop) {
    const stream = await claude.messages.stream({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1024,
      system: systemPrompt,
      messages,
      tools: CONCIERGE_TOOLS,
    });

    let fullResponse = '';
    let toolUses: ToolUseBlock[] = [];

    for await (const event of stream) {
      if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
        fullResponse += event.delta.text;
        yield { type: 'content_delta', text: event.delta.text };
      }
      if (event.type === 'content_block_start' && event.content_block.type === 'tool_use') {
        toolUses.push(event.content_block);
      }
    }

    // 6. If no tool calls, we are done
    if (toolUses.length === 0) {
      continueLoop = false;
    } else {
      // 7. Execute each tool and feed results back
      for (const toolUse of toolUses) {
        yield { type: 'tool_use', tool: toolUse.name, input: toolUse.input, status: 'executing' };
        const result = await executeTool(practiceId, toolUse.name, toolUse.input);
        yield { type: 'tool_result', tool: toolUse.name, result };

        // Add tool use and result to messages for next iteration
        messages.push({ role: 'assistant', content: [{ type: 'tool_use', id: toolUse.id, name: toolUse.name, input: toolUse.input }] });
        messages.push({ role: 'user', content: [{ type: 'tool_result', tool_use_id: toolUse.id, content: JSON.stringify(result) }] });
      }
    }
  }

  // 8. Persist assistant response
  await db.conversationMessage.create({
    data: { conversationId: conversation.id, role: 'assistant', content: fullResponse }
  });

  // 9. Update practice interaction metadata
  await db.practice.update({
    where: { id: practiceId },
    data: {
      conciergeLastInteraction: new Date(),
      conciergeInteractionCount: { increment: 1 }
    }
  });

  yield { type: 'message_end', messageId: '', tokens: { input: 0, output: 0 } };
}
```

---

## 5. Authentication

### 5.1 API Key Authentication (Internal Services)

For service-to-service communication (harvester worker → API, sync worker → API, dashboard backend → API).

```typescript
// Middleware
async function apiKeyAuth(req: Request, res: Response, next: NextFunction) {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey) return res.status(401).json({ error: 'API key required' });

  const hashedKey = hashApiKey(apiKey as string);
  const validKey = await db.apiKey.findUnique({ where: { hash: hashedKey } });

  if (!validKey || !validKey.isActive) {
    return res.status(401).json({ error: 'Invalid API key' });
  }

  req.context = { type: 'service', keyId: validKey.id, scopes: validKey.scopes };
  next();
}
```

**API key scopes**: `practices:read`, `practices:write`, `enrollment:manage`, `harvest:trigger`, `dashboard:read`, `admin:all`.

### 5.2 Session-Based Authentication (HQ Dashboard)

For HQ team members accessing the dashboard.

```typescript
// Using iron-session for encrypted session cookies
const sessionConfig = {
  cookieName: 'sga_session',
  password: process.env.SESSION_SECRET,
  cookieOptions: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'lax' as const,
    maxAge: 60 * 60 * 8, // 8 hours
  },
};
```

**Phase 1 auth flow**: Simple email + password login for HQ team (5-10 users). No SSO integration until Phase 2. User records stored in a `hq_user` table with bcrypt-hashed passwords.

### 5.3 Practice Session (Concierge Chat)

Practices access the concierge via a tokenized enrollment link. No username/password.

```typescript
// Practice access via signed URL
// https://concierge.sga.com/enroll/{slug}/{token}
// Token is a JWT with: { practiceId, slug, exp: 90 days }

async function practiceAuth(req: Request, res: Response, next: NextFunction) {
  const { slug, token } = req.params;
  try {
    const payload = jwt.verify(token, process.env.PRACTICE_TOKEN_SECRET);
    const practice = await cache.getPracticeProfile(payload.practiceId);
    if (!practice || practice.slug !== slug) throw new Error('Mismatch');
    req.context = { type: 'practice', practiceId: practice.id };
    next();
  } catch {
    return res.status(401).json({ error: 'Invalid or expired link' });
  }
}
```

---

## 6. Error Handling Patterns

### 6.1 Application Error Classes

```typescript
class AppError extends Error {
  constructor(
    public statusCode: number,
    public code: string,
    message: string,
    public details?: Record<string, unknown>
  ) {
    super(message);
  }
}

class NotFoundError extends AppError {
  constructor(entity: string, id: string) {
    super(404, 'NOT_FOUND', `${entity} not found: ${id}`);
  }
}

class ValidationError extends AppError {
  constructor(message: string, details?: Record<string, unknown>) {
    super(400, 'VALIDATION_ERROR', message, details);
  }
}

class ConflictError extends AppError {
  constructor(message: string) {
    super(409, 'CONFLICT', message);
  }
}

class ExternalServiceError extends AppError {
  constructor(service: string, message: string) {
    super(502, 'EXTERNAL_SERVICE_ERROR', `${service}: ${message}`);
  }
}
```

### 6.2 Global Error Handler

```typescript
function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  // Log to Sentry
  if (!(err instanceof AppError) || err.statusCode >= 500) {
    Sentry.captureException(err);
  }

  // Structured logging
  logger.error({
    error: err.message,
    code: err instanceof AppError ? err.code : 'INTERNAL_ERROR',
    path: req.path,
    method: req.method,
    practiceId: req.context?.practiceId,
  });

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { code: err.code, message: err.message, details: err.details }
    });
  }

  return res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' }
  });
}
```

### 6.3 Claude API Error Handling

```typescript
async function callClaude(request: ClaudeRequest, retries = 3): Promise<ClaudeResponse> {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      return await claude.messages.create(request);
    } catch (error) {
      if (error.status === 429) {
        // Rate limited — exponential backoff
        const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
        await sleep(delay);
        continue;
      }
      if (error.status === 529) {
        // Overloaded — back off more aggressively
        const delay = Math.min(5000 * Math.pow(2, attempt), 60000);
        await sleep(delay);
        continue;
      }
      if (error.status >= 500) {
        // Server error — retry
        if (attempt < retries) continue;
      }
      throw new ExternalServiceError('Claude API', error.message);
    }
  }
  throw new ExternalServiceError('Claude API', 'Max retries exceeded');
}
```

### 6.4 Queue Job Error Handling

```typescript
// BullMQ worker with built-in retry
const harvestWorker = new Worker('harvest', processHarvestJob, {
  connection: redis,
  concurrency: 5,
  limiter: { max: 10, duration: 60000 }, // 10 jobs per minute
});

harvestWorker.on('failed', (job, error) => {
  logger.error({ jobId: job.id, error: error.message, attempts: job.attemptsMade });
  if (job.attemptsMade >= job.opts.attempts) {
    // Final failure — mark harvest as failed, alert HQ
    markHarvestFailed(job.data.practiceId, error.message);
  }
});

// Job options for different queues
const QUEUE_CONFIGS = {
  harvest: { attempts: 3, backoff: { type: 'exponential', delay: 5000 } },
  generate: { attempts: 2, backoff: { type: 'fixed', delay: 3000 } },
  sync: { attempts: 5, backoff: { type: 'exponential', delay: 2000 } },
};
```

---

## 7. Phase 1 Scope Summary

### In Scope

| Feature | Details |
|---------|---------|
| Practice CRUD | Full CRUD with Redis caching and change events |
| Data Harvester | Website scraping + Google Places API for pilot practices |
| Enrollment flow | invite → verify → select template → approve → deploy |
| Concierge chat | Web-based chat with streaming responses and tool execution |
| 2 templates | GP and Perio, static site generation, Cloudflare Pages deploy |
| HQ Dashboard | Practice list, enrollment progress, basic drill-down |
| Auth | API keys (internal), session (HQ), token links (practices) |

### Out of Scope (Phase 2)

| Feature | Reason |
|---------|--------|
| GMB write access | Requires business verification setup with Google |
| Social media sync | Facebook/Instagram API integration |
| Blitz deploy | Depends on having templates + harvester proven first |
| Health score engine | Needs channel integrations for metric sources |
| iSolved integration | HR webhook setup with SGA operations |
| SMS/voice channels | Web chat proves the pattern first |
| Review management | Requires review aggregation infrastructure |
| Batch template operations | Needs more than 2 templates first |

### Phase 1 Milestones

| Week | Milestone |
|------|-----------|
| 1 | Database + API scaffold + practice CRUD operational |
| 2 | Harvester running on pilot practices + enrollment API complete |
| 3 | Concierge chat functional + 2 templates rendering + site deploy pipeline |
| 4 | End-to-end enrollment tested with 5 practices, then open to 20-30 pilot |
