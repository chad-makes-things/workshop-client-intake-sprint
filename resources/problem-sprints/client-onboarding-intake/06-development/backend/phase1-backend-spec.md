# Phase 1 Backend Specification -- Workshop Intake Engine

**Scope:** Intake Flow Engine + Session Manager + Framework Template Store
**Stack:** Next.js 14 App Router, TypeScript, PostgreSQL 16 (Drizzle ORM), Redis 7, AWS S3
**Phase 1 Constraint:** No AI/scraping, no voice recording, no link processing. Text-only guided interview with branching questions, PDF file upload, magic link sessions, auto-save, cross-device resume.

---

## 1. API Endpoint Specification

All client-facing endpoints live under `/api/intake/`. Auth is via HTTP-only JWT cookie set during magic link validation. The JWT contains `sessionId`, `clientId`, and `bookingId`.

All responses follow a consistent envelope:

```typescript
// Success envelope
interface ApiResponse<T> {
  ok: true;
  data: T;
}

// Error envelope
interface ApiError {
  ok: false;
  error: {
    code: string;        // machine-readable, e.g. "SESSION_EXPIRED"
    message: string;     // human-readable
    details?: unknown;   // validation errors, etc.
  };
}
```

### Standard Error Responses (all endpoints unless noted)

| Status | Code | When |
|--------|------|------|
| 401 | `UNAUTHORIZED` | Missing or invalid JWT cookie |
| 403 | `SESSION_EXPIRED` | Token/session has expired |
| 403 | `SESSION_SUBMITTED` | Session already submitted; mutations blocked |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Unhandled server error |

---

### 1.1 Session Creation (Magic Link)

#### `POST /api/webhooks/booking`

Creates a client, booking, and intake session from the external booking system.

**Auth:** Webhook signature verification (HMAC-SHA256 of raw body using shared secret in `X-Webhook-Signature` header).

**Request Body:**

```typescript
interface BookingWebhookPayload {
  client_name: string;           // required
  client_email: string;          // required, valid email
  client_phone?: string;         // E.164 format
  company_name: string;          // required
  company_url?: string;          // valid URL
  workshop_date: string;         // ISO 8601 date (YYYY-MM-DD)
  booking_id_external?: string;  // external system's booking ID for idempotency
}
```

**Response (201 Created):**

```typescript
interface BookingWebhookResponse {
  booking_id: string;       // UUID
  client_id: string;        // UUID
  session_id: string;       // UUID
  intake_url: string;       // full URL: https://{domain}/intake/{magic_link_token}
  magic_link_token: string; // 32-char base62 token
}
```

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 400 | `VALIDATION_ERROR` | Missing required fields or invalid format |
| 401 | `INVALID_SIGNATURE` | HMAC signature mismatch |
| 409 | `DUPLICATE_BOOKING` | `booking_id_external` already processed (returns existing booking data) |

**Side Effects:**
- Upserts client by email (creates if new, updates name/phone/company if existing)
- Creates `workshop_bookings` row with status `intake-pending`
- Creates `intake_sessions` row with generated `magic_link_token`, status `not-started`, `expires_at` = workshop_date + 30 days
- Logs to `session_access_log`

**Idempotency:** If `booking_id_external` is provided and already exists, returns 409 with the existing booking data. The caller can safely retry.

---

### 1.2 Session Validation (Magic Link Resolution)

#### `GET /intake/[token]`

This is a Next.js page route, not an API endpoint. It validates the magic link token and establishes the session cookie.

**Path Params:**

| Param | Type | Description |
|-------|------|-------------|
| `token` | string | 32-char base62 magic link token |

**Behavior:**
1. Looks up `intake_sessions` by `magic_link_token`
2. Validates: token exists, session not expired, session not submitted
3. Sets HTTP-only secure cookie containing a signed JWT (24h expiry) with `{ sessionId, clientId, bookingId }`
4. Logs access to `session_access_log` (IP, user agent, timestamp)
5. Redirects to `/intake/flow`

**Error Handling:**
- Invalid/missing token: renders an error page with message "This link is invalid or has expired. Check your email for the latest link, or contact [facilitator name]."
- Expired session: same error page
- Already submitted: redirects to `/intake/complete` (read-only confirmation page)

**Rate Limiting:** 5 requests per minute per IP address.

---

### 1.3 Session State

#### `GET /api/intake/session`

Returns the full current session state for session restoration on page load.

**Auth:** JWT cookie

**Response (200 OK):**

```typescript
interface SessionStateResponse {
  session_id: string;
  status: SessionStatus;
  current_step: number;
  current_section: string | null;
  completion_pct: number;           // 0.00 - 100.00
  segmentation_choice: SegmentationChoice | null;
  framework_version: number;
  started_at: string | null;        // ISO 8601
  last_activity: string | null;     // ISO 8601
  workshop_date: string;            // ISO 8601 date
  client: {
    name: string;
    company_name: string;
  };
  sections: SectionSummary[];
}

interface SectionSummary {
  key: string;
  name: string;
  display_order: number;
  status: 'not-started' | 'in-progress' | 'complete';
  submission_count: number;
  completion_pct: number;
}

type SessionStatus = 'not-started' | 'in-progress' | 'paused' | 'submitted';

type SegmentationChoice =
  | 'documents-ready'
  | 'scattered-outdated'
  | 'in-our-heads'
  | 'early-stage';
```

---

#### `PATCH /api/intake/session`

Updates session navigation state. Called on every step transition.

**Auth:** JWT cookie

**Request Body:**

```typescript
interface UpdateSessionRequest {
  current_step?: number;
  current_section?: string;
  segmentation_choice?: SegmentationChoice;
  status?: 'in-progress';  // only transition from not-started to in-progress
}
```

**Response (200 OK):**

```typescript
interface UpdateSessionResponse {
  session_id: string;
  current_step: number;
  current_section: string | null;
  segmentation_choice: SegmentationChoice | null;
  completion_pct: number;
  status: SessionStatus;
  last_activity: string;
}
```

**Validation:**
- `segmentation_choice` can only be set once (returns 400 `SEGMENTATION_LOCKED` if already set)
- `current_step` must be >= 0
- `status` can only transition from `not-started` to `in-progress` via this endpoint

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 400 | `SEGMENTATION_LOCKED` | Attempting to change segmentation after it was set |
| 400 | `INVALID_TRANSITION` | Invalid status transition |

---

### 1.4 Question Tree Retrieval

#### `GET /api/intake/framework`

Returns the full framework structure with question trees for the current session's framework version.

**Auth:** JWT cookie

**Response (200 OK):**

```typescript
interface FrameworkResponse {
  version: number;
  sections: FrameworkSectionDTO[];
}

interface FrameworkSectionDTO {
  key: string;
  name: string;
  description: string;
  display_order: number;
  min_completeness: number;
  question_tree: QuestionTree;
}

interface QuestionTree {
  schema_version: number;
  entry_nodes: Record<SegmentationChoice, string>;  // segmentation -> entry node ID
  nodes: Record<string, QuestionNode>;
}

interface QuestionNode {
  id: string;
  type: 'question' | 'pre-populated-review' | 'section-intro' | 'section-complete';
  framework_section: string;
  question_text: string;
  question_subtext?: string;
  input_config: InputConfig;
  edges: ConditionalEdge[];
  metadata?: {
    estimated_time_seconds?: number;
    skip_if_answered?: boolean;
  };
}

interface InputConfig {
  type: 'text' | 'textarea' | 'single-select' | 'multi-select' | 'pre-populated';
  placeholder?: string;
  options?: SelectOption[];                      // for single-select / multi-select
  pre_populated_content?: string;                // for pre-populated review nodes
  allows_file_upload: boolean;                   // Phase 1: PDF only
  max_length?: number;
  required: boolean;
}

interface SelectOption {
  value: string;
  label: string;
  description?: string;
}

interface ConditionalEdge {
  target_node_id: string;
  condition: EdgeCondition | null;  // null = default/fallback edge
}

type EdgeCondition =
  | { type: 'equals'; field: 'answer_value'; value: string }
  | { type: 'contains'; field: 'answer_value'; value: string }
  | { type: 'not_empty'; field: 'answer_value' }
  | { type: 'file_uploaded' }
  | { type: 'always' };
```

**Caching:** Response is cached in Redis for 1 hour keyed by `framework:v{version}`. Cache is invalidated when framework sections are updated.

---

#### `GET /api/intake/section/[sectionKey]/next`

Resolves the next question node in the tree based on current position and submitted answers.

**Auth:** JWT cookie

**Path Params:**

| Param | Type | Description |
|-------|------|-------------|
| `sectionKey` | string | Framework section key (e.g., `market-context`) |

**Query Params:**

| Param | Type | Description |
|-------|------|-------------|
| `current_node_id` | string? | Current node ID. Omit to get the entry node for this section. |

**Response (200 OK):**

```typescript
interface NextQuestionResponse {
  node: QuestionNode;
  progress: {
    section_pct: number;
    overall_pct: number;
    questions_remaining_estimate: number;
  };
  is_terminal: boolean;         // true if this is the last node in the section
  existing_answer?: {           // if the client previously answered this question
    submission_id: string;
    content: string;
    input_type: string;
  };
}
```

**Behavior:**
1. If `current_node_id` is omitted, returns the entry node for the section based on `segmentation_choice`
2. If `current_node_id` is provided, evaluates all outgoing edges against the client's submitted answers for that node
3. The first edge whose condition evaluates to `true` determines the next node
4. If no conditional edge matches, the fallback edge (condition = `null`) is used
5. If the client has already answered the resolved node, includes `existing_answer`

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 400 | `NO_SEGMENTATION` | Segmentation choice not yet set; cannot resolve entry node |
| 404 | `SECTION_NOT_FOUND` | Invalid section key |
| 404 | `NODE_NOT_FOUND` | Invalid `current_node_id` |

---

### 1.5 Answer Submission

#### `POST /api/intake/submissions`

Saves a client answer. Called on every answer (auto-save). Uses upsert logic: if a submission already exists for this session + question_node_id, it updates rather than creating a duplicate.

**Auth:** JWT cookie

**Request Body:**

```typescript
interface CreateSubmissionRequest {
  framework_section: string;        // e.g., "competitive-landscape"
  question_node_id: string;         // node ID from the question tree
  input_type: InputType;
  content: string;                  // the client's answer text
  original_content?: string;        // for pre-populated-edited/rewritten: the original AI draft
}

type InputType =
  | 'text'
  | 'pre-populated-confirmed'
  | 'pre-populated-edited'
  | 'pre-populated-rewritten';
```

**Response (201 Created / 200 OK on upsert):**

```typescript
interface SubmissionResponse {
  submission_id: string;
  created: boolean;                // true if new, false if updated
  progress: {
    section_pct: number;
    overall_pct: number;
  };
}
```

**Validation:**
- `framework_section` must match a valid section key in the current framework version
- `question_node_id` must exist in the question tree for the given section
- `content` must not exceed `max_length` defined in the node's `input_config` (if set)
- `content` is sanitized (HTML stripped, trimmed)

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 400 | `VALIDATION_ERROR` | Invalid section, node ID, or content too long |
| 400 | `CONTENT_REQUIRED` | `input_type` is `text` but content is empty, and node is `required` |

**Side Effects:**
- Updates `intake_sessions.last_activity` and `completion_pct`
- Updates `intake_sessions.current_step` and `current_section`

---

#### `PUT /api/intake/submissions/[id]`

Updates an existing submission. Used when the client edits a previously saved answer.

**Auth:** JWT cookie

**Path Params:**

| Param | Type | Description |
|-------|------|-------------|
| `id` | string | Submission UUID |

**Request Body:**

```typescript
interface UpdateSubmissionRequest {
  content: string;
  input_type?: InputType;  // allows changing type, e.g., confirmed -> edited
}
```

**Response (200 OK):**

```typescript
interface UpdateSubmissionResponse {
  submission_id: string;
  updated_at: string;
}
```

**Validation:**
- Submission must belong to the current session (verified via JWT)
- Same content validation as POST

**Error Responses:**

| Status | Code | When |
|--------|------|------|
| 404 | `SUBMISSION_NOT_FOUND` | Invalid ID or does not belong to current session |

---

### 1.6 Progress Tracking

#### `GET /api/intake/progress`

Returns detailed progress breakdown across all sections.

**Auth:** JWT cookie

**Response (200 OK):**

```typescript
interface ProgressResponse {
  overall_pct: number;
  sections: SectionProgress[];
  estimated_minutes_remaining: number;
}

interface SectionProgress {
  key: string;
  name: string;
  display_order: number;
  status: 'not-started' | 'in-progress' | 'complete';
  completion_pct: number;
  questions_total: number;       // total questions in this section for the client's segmentation path
  questions_answered: number;
  has_file_uploads: boolean;
}
```

**Calculation Logic:**
- Each section's completion is `questions_answered / questions_total` for the client's segmentation path
- Overall completion is a weighted average across sections (each section weighted equally)
- `estimated_minutes_remaining` assumes 45 seconds per unanswered question

---

### 1.7 File Upload (PDF Only -- Phase 1)

#### `POST /api/intake/submissions/[id]/file`

Uploads a PDF file and attaches it to an existing submission.

**Auth:** JWT cookie

**Path Params:**

| Param | Type | Description |
|-------|------|-------------|
| `id` | string | Submission UUID |

**Request:** `multipart/form-data`

| Field | Type | Constraints |
|-------|------|-------------|
| `file` | File | PDF only (`.pdf`, `application/pdf`). Max 25 MB. |

**Response (202 Accepted):**

```typescript
interface FileUploadResponse {
  file_id: string;             // UUID
  original_filename: string;
  file_size_bytes: number;
  processing_status: 'queued';
  message: string;             // "File received. It will be available for review shortly."
}
```

**Validation:**

| Status | Code | When |
|--------|------|------|
| 400 | `INVALID_FILE_TYPE` | Not a PDF |
| 400 | `FILE_TOO_LARGE` | Exceeds 25 MB |
| 404 | `SUBMISSION_NOT_FOUND` | Invalid submission ID or not owned by session |
| 413 | `PAYLOAD_TOO_LARGE` | Request body exceeds limit (caught by middleware) |

**Side Effects:**
- Uploads file to S3 at key: `intake/{booking_id}/{session_id}/{file_id}/{original_filename}`
- Creates `processed_files` row with `processing_status: 'queued'`
- In Phase 1, file processing is deferred -- the file is stored but text extraction does not run until Phase 2's AI pipeline is built. The `processed_files` row tracks it for future processing.

---

### 1.8 Review / Summary

#### `GET /api/intake/review`

Returns the complete review summary for the submission screen (Storyboard Step 8).

**Auth:** JWT cookie

**Response (200 OK):**

```typescript
interface ReviewResponse {
  session_id: string;
  client_name: string;
  company_name: string;
  completion_pct: number;
  sections: ReviewSection[];
  can_submit: boolean;         // always true -- submission is never blocked
}

interface ReviewSection {
  key: string;
  name: string;
  display_order: number;
  status: 'complete' | 'partial' | 'empty';
  completion_pct: number;
  submissions: ReviewSubmission[];
  files: ReviewFile[];
}

interface ReviewSubmission {
  id: string;
  question_text: string;         // from the question tree node
  input_type: InputType;
  content: string;
  created_at: string;
  updated_at: string;
}

interface ReviewFile {
  id: string;
  original_filename: string;
  file_size_bytes: number;
  processing_status: string;
  uploaded_at: string;
}
```

---

### 1.9 Final Submission

#### `POST /api/intake/session/submit`

Marks the session as submitted. This is a one-way transition.

**Auth:** JWT cookie

**Request Body:** Empty (`{}`)

**Response (200 OK):**

```typescript
interface SubmitResponse {
  submitted_at: string;           // ISO 8601
  completion_pct: number;         // final completeness score
  section_summary: {
    key: string;
    status: 'complete' | 'partial' | 'empty';
    submission_count: number;
    file_count: number;
  }[];
}
```

**Validation:**
- Session must be in status `in-progress` or `paused`
- Session must not already be submitted

| Status | Code | When |
|--------|------|------|
| 409 | `ALREADY_SUBMITTED` | Session already in `submitted` status |

**Side Effects:**
- Sets `intake_sessions.status` to `submitted` and `submitted_at` to now
- Sets `workshop_bookings.status` to `intake-complete`
- Sends submission confirmation email to client (basic email via Resend; full notification service comes later)
- Sends facilitator notification email with completeness score, file count, and dashboard link

---

## 2. Database Schema (Phase 1 Tables -- Drizzle ORM)

```typescript
// src/db/schema.ts

import {
  pgTable,
  uuid,
  text,
  timestamp,
  integer,
  decimal,
  boolean,
  jsonb,
  pgEnum,
  uniqueIndex,
  index,
  bigint,
  inet,
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// ============================================================
// ENUMS
// ============================================================

export const clientSegmentationEnum = pgEnum('client_segmentation', [
  'documents-ready',
  'scattered-outdated',
  'in-our-heads',
  'early-stage',
]);

export const notificationPreferenceEnum = pgEnum('notification_preference', [
  'email',
  'sms',
  'both',
]);

export const bookingStatusEnum = pgEnum('booking_status', [
  'booked',
  'scraping',
  'intake-pending',
  'intake-in-progress',
  'intake-complete',
  'follow-up-pending',
  'narrative-ready',
  'workshop-complete',
  'archived',
]);

export const sessionStatusEnum = pgEnum('session_status', [
  'not-started',
  'in-progress',
  'paused',
  'submitted',
  'expired',
]);

export const inputTypeEnum = pgEnum('input_type', [
  'text',
  'voice',
  'file',
  'link',
  'pre-populated-confirmed',
  'pre-populated-edited',
  'pre-populated-rewritten',
]);

export const processingStatusEnum = pgEnum('processing_status', [
  'queued',
  'processing',
  'complete',
  'failed',
  'skipped',
]);

export const fileTypeEnum = pgEnum('file_type', [
  'pdf',
  'docx',
  'pptx',
  'image',
  'audio',
  'video',
  'other',
]);

// ============================================================
// TABLES
// ============================================================

// --- clients ---
export const clients = pgTable('clients', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: text('name').notNull(),
  email: text('email').notNull(),
  phone: text('phone'),
  companyName: text('company_name').notNull(),
  companyUrl: text('company_url'),
  segmentation: clientSegmentationEnum('segmentation'),
  notificationPref: notificationPreferenceEnum('notification_pref').default('email'),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  emailIdx: uniqueIndex('idx_clients_email').on(table.email),
  companyIdx: index('idx_clients_company').on(table.companyName),
}));

// --- workshop_bookings ---
export const workshopBookings = pgTable('workshop_bookings', {
  id: uuid('id').primaryKey().defaultRandom(),
  clientId: uuid('client_id').notNull().references(() => clients.id, { onDelete: 'cascade' }),
  workshopDate: timestamp('workshop_date', { mode: 'date' }).notNull(),
  bookingDate: timestamp('booking_date', { withTimezone: true }).notNull().defaultNow(),
  status: bookingStatusEnum('status').notNull().default('booked'),
  externalBookingId: text('external_booking_id'),
  webhookPayload: jsonb('webhook_payload'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  clientIdx: index('idx_bookings_client').on(table.clientId),
  statusIdx: index('idx_bookings_status').on(table.status),
  workshopDateIdx: index('idx_bookings_workshop_date').on(table.workshopDate),
  externalIdIdx: uniqueIndex('idx_bookings_external_id').on(table.externalBookingId),
}));

// --- framework_sections ---
export const frameworkSections = pgTable('framework_sections', {
  id: uuid('id').primaryKey().defaultRandom(),
  templateVersion: integer('template_version').notNull().default(1),
  sectionKey: text('section_key').notNull(),
  sectionName: text('section_name').notNull(),
  description: text('description'),
  minCompleteness: decimal('min_completeness', { precision: 3, scale: 2 }).default('0.60'),
  displayOrder: integer('display_order').notNull(),
  questionTree: jsonb('question_tree').notNull(),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  keyVersionIdx: uniqueIndex('idx_framework_sections_key_version')
    .on(table.sectionKey, table.templateVersion),
  activeOrderIdx: index('idx_framework_sections_active')
    .on(table.isActive, table.displayOrder),
}));

// --- intake_sessions ---
export const intakeSessions = pgTable('intake_sessions', {
  id: uuid('id').primaryKey().defaultRandom(),
  clientId: uuid('client_id').notNull().references(() => clients.id, { onDelete: 'cascade' }),
  bookingId: uuid('booking_id').notNull().references(() => workshopBookings.id, { onDelete: 'cascade' }),
  magicLinkToken: text('magic_link_token').notNull(),
  currentStep: integer('current_step').default(0),
  currentSection: text('current_section'),
  completionPct: decimal('completion_pct', { precision: 5, scale: 2 }).default('0.00'),
  segmentationChoice: clientSegmentationEnum('segmentation_choice'),
  frameworkVersion: integer('framework_version').notNull().default(1),
  sessionData: jsonb('session_data').default({}),
  status: sessionStatusEnum('status').notNull().default('not-started'),
  startedAt: timestamp('started_at', { withTimezone: true }),
  lastActivity: timestamp('last_activity', { withTimezone: true }),
  submittedAt: timestamp('submitted_at', { withTimezone: true }),
  expiresAt: timestamp('expires_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  magicTokenIdx: uniqueIndex('idx_sessions_magic_token').on(table.magicLinkToken),
  clientIdx: index('idx_sessions_client').on(table.clientId),
  bookingIdx: index('idx_sessions_booking').on(table.bookingId),
  statusIdx: index('idx_sessions_status').on(table.status),
}));

// --- submissions ---
export const submissions = pgTable('submissions', {
  id: uuid('id').primaryKey().defaultRandom(),
  sessionId: uuid('session_id').notNull().references(() => intakeSessions.id, { onDelete: 'cascade' }),
  frameworkSection: text('framework_section').notNull(),
  questionNodeId: text('question_node_id'),
  inputType: inputTypeEnum('input_type').notNull(),
  content: text('content'),
  originalContent: text('original_content'),
  fileRef: text('file_ref'),
  aiMappings: jsonb('ai_mappings'),
  confidence: decimal('confidence', { precision: 3, scale: 2 }),
  displayOrder: integer('display_order'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  sessionIdx: index('idx_submissions_session').on(table.sessionId),
  sectionIdx: index('idx_submissions_section').on(table.frameworkSection),
  typeIdx: index('idx_submissions_type').on(table.inputType),
  sessionSectionIdx: index('idx_submissions_session_section')
    .on(table.sessionId, table.frameworkSection),
  upsertIdx: uniqueIndex('idx_submissions_session_node')
    .on(table.sessionId, table.questionNodeId),
}));

// --- processed_files ---
export const processedFiles = pgTable('processed_files', {
  id: uuid('id').primaryKey().defaultRandom(),
  submissionId: uuid('submission_id').notNull().references(() => submissions.id, { onDelete: 'cascade' }),
  originalFilename: text('original_filename').notNull(),
  fileType: fileTypeEnum('file_type').notNull(),
  mimeType: text('mime_type'),
  fileSizeBytes: bigint('file_size_bytes', { mode: 'number' }),
  storageKey: text('storage_key').notNull(),
  storageBucket: text('storage_bucket').notNull(),
  extractedText: text('extracted_text'),
  extractedMetadata: jsonb('extracted_metadata'),
  processingStatus: processingStatusEnum('processing_status').notNull().default('queued'),
  errorMessage: text('error_message'),
  processedAt: timestamp('processed_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  submissionIdx: index('idx_processed_files_submission').on(table.submissionId),
  statusIdx: index('idx_processed_files_status').on(table.processingStatus),
}));

// --- session_access_log ---
export const sessionAccessLog = pgTable('session_access_log', {
  id: uuid('id').primaryKey().defaultRandom(),
  sessionId: uuid('session_id').notNull().references(() => intakeSessions.id, { onDelete: 'cascade' }),
  ipAddress: inet('ip_address'),
  userAgent: text('user_agent'),
  accessedAt: timestamp('accessed_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  sessionIdx: index('idx_access_log_session').on(table.sessionId),
  timeIdx: index('idx_access_log_time').on(table.accessedAt),
}));

// ============================================================
// RELATIONS
// ============================================================

export const clientsRelations = relations(clients, ({ many }) => ({
  bookings: many(workshopBookings),
  sessions: many(intakeSessions),
}));

export const workshopBookingsRelations = relations(workshopBookings, ({ one, many }) => ({
  client: one(clients, {
    fields: [workshopBookings.clientId],
    references: [clients.id],
  }),
  sessions: many(intakeSessions),
}));

export const intakeSessionsRelations = relations(intakeSessions, ({ one, many }) => ({
  client: one(clients, {
    fields: [intakeSessions.clientId],
    references: [clients.id],
  }),
  booking: one(workshopBookings, {
    fields: [intakeSessions.bookingId],
    references: [workshopBookings.id],
  }),
  submissions: many(submissions),
  accessLog: many(sessionAccessLog),
}));

export const submissionsRelations = relations(submissions, ({ one, many }) => ({
  session: one(intakeSessions, {
    fields: [submissions.sessionId],
    references: [intakeSessions.id],
  }),
  files: many(processedFiles),
}));

export const processedFilesRelations = relations(processedFiles, ({ one }) => ({
  submission: one(submissions, {
    fields: [processedFiles.submissionId],
    references: [submissions.id],
  }),
}));

export const sessionAccessLogRelations = relations(sessionAccessLog, ({ one }) => ({
  session: one(intakeSessions, {
    fields: [sessionAccessLog.sessionId],
    references: [intakeSessions.id],
  }),
}));
```

---

## 3. Question Tree Data Structure

### 3.1 JSON Schema

The question tree is stored in `framework_sections.question_tree` as JSONB. Every tree conforms to this structure:

```typescript
interface QuestionTree {
  schema_version: number;                              // currently 1
  entry_nodes: Record<SegmentationChoice, string>;     // maps each segmentation path to its starting node ID
  nodes: Record<string, QuestionNode>;                 // all nodes keyed by ID
}

interface QuestionNode {
  id: string;                                          // unique within the tree, e.g. "cl-q1", "cl-q2a"
  type: 'question' | 'pre-populated-review' | 'section-intro' | 'section-complete';
  framework_section: string;                           // the section this node belongs to
  question_text: string;                               // displayed to the client
  question_subtext?: string;                           // helper text below the question
  input_config: InputConfig;
  edges: ConditionalEdge[];                            // ordered; first match wins
  metadata?: {
    estimated_time_seconds?: number;
    skip_if_answered?: boolean;                        // skip node if a submission already exists
    weight?: number;                                   // contribution to section completion (default 1)
  };
}

interface InputConfig {
  type: 'text' | 'textarea' | 'single-select' | 'multi-select' | 'pre-populated';
  placeholder?: string;
  options?: { value: string; label: string; description?: string }[];
  pre_populated_content?: string;
  allows_file_upload: boolean;
  max_length?: number;
  required: boolean;
}

interface ConditionalEdge {
  target_node_id: string;
  condition: EdgeCondition | null;                     // null = default fallback edge
}

type EdgeCondition =
  | { type: 'equals'; field: 'answer_value'; value: string }
  | { type: 'contains'; field: 'answer_value'; value: string }
  | { type: 'not_empty'; field: 'answer_value' }
  | { type: 'file_uploaded' }
  | { type: 'always' };
```

### 3.2 Example Question Tree Fragment

This shows a portion of the Competitive Landscape section with branching logic:

```json
{
  "schema_version": 1,
  "entry_nodes": {
    "documents-ready": "cl-intro",
    "scattered-outdated": "cl-intro",
    "in-our-heads": "cl-intro",
    "early-stage": "cl-early-q1"
  },
  "nodes": {
    "cl-intro": {
      "id": "cl-intro",
      "type": "section-intro",
      "framework_section": "competitive-landscape",
      "question_text": "Now let's talk about your competition.",
      "question_subtext": "Understanding who else is in your space helps us position your story.",
      "input_config": { "type": "text", "required": false, "allows_file_upload": false },
      "edges": [
        { "target_node_id": "cl-q1", "condition": { "type": "always" } }
      ]
    },
    "cl-q1": {
      "id": "cl-q1",
      "type": "question",
      "framework_section": "competitive-landscape",
      "question_text": "Who are the 2-3 companies your customers would consider instead of you?",
      "question_subtext": "Names are great, but even descriptions like 'the cheap alternative' or 'the enterprise player' work.",
      "input_config": {
        "type": "textarea",
        "placeholder": "e.g., Competitor A focuses on enterprise, Competitor B is the budget option...",
        "allows_file_upload": true,
        "max_length": 2000,
        "required": true
      },
      "edges": [
        {
          "target_node_id": "cl-q2-upload",
          "condition": { "type": "file_uploaded" }
        },
        {
          "target_node_id": "cl-q2",
          "condition": { "type": "not_empty", "field": "answer_value" }
        }
      ],
      "metadata": { "estimated_time_seconds": 60, "weight": 2 }
    },
    "cl-q2": {
      "id": "cl-q2",
      "type": "question",
      "framework_section": "competitive-landscape",
      "question_text": "What makes you different from those competitors?",
      "question_subtext": "What would a customer say is the reason they chose you over the alternatives?",
      "input_config": {
        "type": "textarea",
        "placeholder": "Think about what your best customers say about why they chose you...",
        "allows_file_upload": false,
        "max_length": 2000,
        "required": true
      },
      "edges": [
        { "target_node_id": "cl-q3", "condition": { "type": "not_empty", "field": "answer_value" } }
      ],
      "metadata": { "estimated_time_seconds": 45 }
    },
    "cl-q2-upload": {
      "id": "cl-q2-upload",
      "type": "question",
      "framework_section": "competitive-landscape",
      "question_text": "Great, we'll review that document. Anything you'd add about what makes you different?",
      "question_subtext": "The document covers the details -- we're looking for the story behind it.",
      "input_config": {
        "type": "textarea",
        "placeholder": "Optional -- skip if the document covers it",
        "allows_file_upload": false,
        "max_length": 2000,
        "required": false
      },
      "edges": [
        { "target_node_id": "cl-q3", "condition": null }
      ],
      "metadata": { "estimated_time_seconds": 30, "skip_if_answered": true }
    },
    "cl-q3": {
      "id": "cl-q3",
      "type": "question",
      "framework_section": "competitive-landscape",
      "question_text": "Is there anything a competitor does better than you right now?",
      "question_subtext": "Honest answers here help us build a credible narrative, not just a polished one.",
      "input_config": {
        "type": "textarea",
        "placeholder": "e.g., Their onboarding is faster, they have stronger brand recognition...",
        "allows_file_upload": false,
        "max_length": 2000,
        "required": false
      },
      "edges": [
        { "target_node_id": "cl-complete", "condition": null }
      ],
      "metadata": { "estimated_time_seconds": 45 }
    },
    "cl-early-q1": {
      "id": "cl-early-q1",
      "type": "question",
      "framework_section": "competitive-landscape",
      "question_text": "Who were your customers using before they found you?",
      "question_subtext": "Even if you don't think of them as competitors, the alternatives your customers considered are valuable context.",
      "input_config": {
        "type": "textarea",
        "placeholder": "Could be other products, DIY solutions, or doing nothing at all...",
        "allows_file_upload": false,
        "max_length": 2000,
        "required": true
      },
      "edges": [
        { "target_node_id": "cl-q3", "condition": { "type": "not_empty", "field": "answer_value" } }
      ],
      "metadata": { "estimated_time_seconds": 60 }
    },
    "cl-complete": {
      "id": "cl-complete",
      "type": "section-complete",
      "framework_section": "competitive-landscape",
      "question_text": "That covers the competitive landscape. Nice work.",
      "input_config": { "type": "text", "required": false, "allows_file_upload": false },
      "edges": []
    }
  }
}
```

### 3.3 Condition Evaluation

The traversal engine evaluates edges in array order. The first edge whose condition returns `true` is followed. A `null` condition is the fallback (always true, used as the last edge).

```typescript
// src/lib/intake/tree-traversal.ts

function evaluateCondition(
  condition: EdgeCondition | null,
  answer: SubmissionRecord | null,
  files: ProcessedFileRecord[]
): boolean {
  if (condition === null) return true; // fallback edge

  switch (condition.type) {
    case 'equals':
      return answer?.content?.trim() === condition.value;

    case 'contains':
      return answer?.content?.toLowerCase().includes(condition.value.toLowerCase()) ?? false;

    case 'not_empty':
      return (answer?.content?.trim().length ?? 0) > 0;

    case 'file_uploaded':
      return files.length > 0;

    case 'always':
      return true;

    default:
      return false; // unknown condition type: skip edge
  }
}
```

### 3.4 Framework Section Mapping

Each `QuestionNode` carries a `framework_section` field that directly maps it to a `framework_sections.section_key`. When a submission is saved, the `framework_section` on the submission record comes from the node definition, not from client input. This is the "structural mapping" described in ADR-5 -- every answer is tagged to its framework slot with 100% certainty based on which question produced it.

The question tree for each section is stored in the `framework_sections.question_tree` column for that section. The section key ties everything together:

```
framework_sections.section_key = "competitive-landscape"
  -> question_tree contains nodes cl-q1, cl-q2, cl-q3, etc.
  -> each node has framework_section = "competitive-landscape"
  -> submissions created from these nodes have framework_section = "competitive-landscape"
```

---

## 4. Session Management Flow

### 4.1 Magic Link Generation and Delivery

**Generation (triggered by booking webhook):**

```typescript
// src/lib/session/magic-link.ts

import { randomBytes } from 'crypto';
import { base62Encode } from '@/lib/utils/encoding';

function generateMagicLinkToken(): string {
  // 24 random bytes -> base62 encoded -> ~32 characters
  // Entropy: 192 bits (far exceeds the 128-bit minimum for bearer tokens)
  const bytes = randomBytes(24);
  return base62Encode(bytes);  // e.g., "7kX9mNpQ2rTvWxYz4aBcDeFgHjKL"
}

function buildIntakeUrl(token: string): string {
  return `${process.env.NEXT_PUBLIC_BASE_URL}/intake/${token}`;
}
```

**Delivery:** The intake URL is returned in the webhook response. In Phase 1, the booking system or the facilitator sends the welcome email manually. Automated email delivery (Resend integration) is implemented as a simple transactional send triggered by the webhook handler but is not the full Notification Service (that comes in Phase 5).

### 4.2 Token Structure and Validation

**Magic Link Token:**
- 32-character base62 string stored in `intake_sessions.magic_link_token`
- Not a JWT -- it is an opaque lookup key. All session data lives server-side.
- Does not rotate on access (preserving bookmarked URLs and reminder email links)

**Session Cookie (JWT):**

```typescript
// JWT payload
interface SessionJWT {
  sessionId: string;    // intake_sessions.id
  clientId: string;     // clients.id
  bookingId: string;    // workshop_bookings.id
  iat: number;          // issued at
  exp: number;          // expires: iat + 24 hours
}
```

- Signed with `HS256` using `SESSION_JWT_SECRET` from environment
- HTTP-only, Secure, SameSite=Lax
- 24-hour expiry; automatically refreshed on each API request via middleware
- If the cookie expires and the client returns via the magic link URL, a new cookie is issued after token validation

**Validation Middleware:**

```typescript
// src/middleware/auth.ts (applied to all /api/intake/* routes)

async function validateSession(request: NextRequest) {
  const token = request.cookies.get('intake_session')?.value;
  if (!token) throw new UnauthorizedError();

  const payload = verifyJwt<SessionJWT>(token, process.env.SESSION_JWT_SECRET);

  // Check session still valid in DB (not expired, not submitted for mutation endpoints)
  const session = await redis.get(`session:${payload.sessionId}`);
  if (!session) {
    // Cache miss: check database
    const dbSession = await db.query.intakeSessions.findFirst({
      where: eq(intakeSessions.id, payload.sessionId),
    });
    if (!dbSession || dbSession.status === 'expired') {
      throw new SessionExpiredError();
    }
    // Populate cache
    await redis.set(`session:${payload.sessionId}`, JSON.stringify(dbSession), 'EX', 300);
  }

  // Refresh cookie if within 1 hour of expiry
  if (payload.exp - Date.now() / 1000 < 3600) {
    const newToken = signJwt({ ...payload, iat: undefined, exp: undefined });
    // Set refreshed cookie in response
  }

  return payload;
}
```

### 4.3 Auto-Save Implementation

Auto-save fires on the client side with debouncing. The server treats every `POST /api/intake/submissions` as an upsert.

**Client-Side Debounce:**

```typescript
// Frontend: src/hooks/useAutoSave.ts (specification for frontend implementation)

// Debounce: 500ms after last keystroke for text inputs
// Immediate: on selection change (single-select, multi-select)
// Immediate: on file upload completion
// On blur: save immediately when input loses focus
// On beforeunload: flush any pending save (navigator.sendBeacon as fallback)
```

**Server-Side Upsert Logic:**

```typescript
// src/services/submission.service.ts

async function upsertSubmission(
  sessionId: string,
  data: CreateSubmissionRequest
): Promise<SubmissionResponse> {
  // Use the unique index on (session_id, question_node_id) for upsert
  const result = await db
    .insert(submissions)
    .values({
      sessionId,
      frameworkSection: data.framework_section,
      questionNodeId: data.question_node_id,
      inputType: data.input_type,
      content: sanitizeHtml(data.content),
      originalContent: data.original_content,
    })
    .onConflictDoUpdate({
      target: [submissions.sessionId, submissions.questionNodeId],
      set: {
        content: sanitizeHtml(data.content),
        inputType: data.input_type,
        updatedAt: new Date(),
      },
    })
    .returning();

  // Update session progress
  const progress = await recalculateProgress(sessionId);

  // Update session last_activity
  await db
    .update(intakeSessions)
    .set({
      lastActivity: new Date(),
      completionPct: progress.overall_pct.toString(),
      currentSection: data.framework_section,
      updatedAt: new Date(),
    })
    .where(eq(intakeSessions.id, sessionId));

  // Invalidate Redis session cache
  await redis.del(`session:${sessionId}`);

  return {
    submission_id: result[0].id,
    created: /* check if insert or update */,
    progress,
  };
}
```

**Progress Recalculation:**

```typescript
async function recalculateProgress(sessionId: string): Promise<ProgressData> {
  const session = await db.query.intakeSessions.findFirst({
    where: eq(intakeSessions.id, sessionId),
  });

  // Get all framework sections for this version
  const sections = await db.query.frameworkSections.findMany({
    where: and(
      eq(frameworkSections.templateVersion, session.frameworkVersion),
      eq(frameworkSections.isActive, true),
    ),
  });

  // Get all submissions for this session
  const allSubmissions = await db.query.submissions.findMany({
    where: eq(submissions.sessionId, sessionId),
  });

  // For each section, count answerable nodes on this client's segmentation path
  // and compare to submitted answers
  const sectionProgress = sections.map((section) => {
    const tree = section.questionTree as QuestionTree;
    const entryNodeId = tree.entry_nodes[session.segmentationChoice];
    const reachableNodes = getReachableQuestionNodes(tree, entryNodeId);
    const answeredNodes = allSubmissions
      .filter((s) => s.frameworkSection === section.sectionKey)
      .map((s) => s.questionNodeId);

    const totalWeight = reachableNodes.reduce((sum, n) => sum + (n.metadata?.weight ?? 1), 0);
    const answeredWeight = reachableNodes
      .filter((n) => answeredNodes.includes(n.id))
      .reduce((sum, n) => sum + (n.metadata?.weight ?? 1), 0);

    return {
      key: section.sectionKey,
      completion_pct: totalWeight > 0 ? (answeredWeight / totalWeight) * 100 : 0,
      total: reachableNodes.length,
      answered: answeredNodes.length,
    };
  });

  const overallPct = sectionProgress.reduce((sum, s) => sum + s.completion_pct, 0) / sectionProgress.length;

  return { overall_pct: overallPct, sections: sectionProgress };
}
```

### 4.4 Cross-Device Resume Flow

```
1. Client opens magic link on Device A (laptop)
   -> Token validated, JWT cookie set on Device A
   -> Session status: not-started -> in-progress
   -> Client answers questions, each auto-saved

2. Client closes browser on Device A
   -> beforeunload flushes any pending save
   -> No server-side action needed; state is already persisted

3. Client opens magic link on Device B (phone) -- same URL from email
   -> GET /intake/{token}
   -> Token validated again (same token, still valid)
   -> New JWT cookie set on Device B
   -> Device A's cookie remains valid (not invalidated)
   -> Redirect to /intake/flow

4. /intake/flow page loads
   -> GET /api/intake/session returns full state
   -> current_step, current_section, segmentation_choice all restored
   -> GET /api/intake/framework returns question tree
   -> Frontend navigates to the exact question the client was on
   -> Any previously submitted answers are pre-filled via existing_answer in the next-question response

5. Client continues from where they left off
   -> New submissions saved normally
   -> Last-write-wins if both devices are open simultaneously
```

**Concurrent Access Handling:**
- Both devices can have valid cookies simultaneously
- Submissions use the upsert pattern on `(session_id, question_node_id)` -- last write wins
- No optimistic locking in Phase 1 (the chance of true concurrent edits from the same person is negligible)
- `session_access_log` records all access events for audit

### 4.5 Session Expiration and Security

**Expiration Rules:**
- `intake_sessions.expires_at` is set to `workshop_date + 30 days`
- A daily cron job marks sessions past `expires_at` as `status: 'expired'`
- Expired sessions cannot be accessed (magic link returns error page, API returns 403)

**Security Measures:**

| Measure | Implementation |
|---------|----------------|
| Token entropy | 192 bits (24 random bytes, base62 encoded) |
| Token storage | Server-side only; never in URL query params, always in path |
| Cookie security | HTTP-only, Secure, SameSite=Lax, 24h expiry |
| Rate limiting | 5 magic link validations per minute per IP |
| Access logging | Every magic link access logged with IP and user agent |
| Input sanitization | All text content stripped of HTML before storage |
| File validation | MIME type + magic bytes check for PDF uploads |
| Content-length limit | 25 MB max upload, 10 KB max text submission |

**CSRF Protection:** SameSite=Lax cookies + custom header check (`X-Intake-Client: true`) on all mutation endpoints. The Next.js App Router's built-in CSRF protection covers server actions.

---

## 5. Service Layer Architecture

### 5.1 Module Structure

```
src/
  app/
    intake/
      [token]/page.tsx           # Magic link landing (token validation + redirect)
      flow/page.tsx              # Main intake flow UI
      complete/page.tsx          # Post-submission confirmation
    api/
      intake/
        session/route.ts         # GET + PATCH session state
        session/submit/route.ts  # POST submit
        framework/route.ts       # GET framework structure
        section/[sectionKey]/
          next/route.ts          # GET next question
        submissions/route.ts     # POST create submission
        submissions/[id]/
          route.ts               # PUT update submission
          file/route.ts          # POST file upload
        progress/route.ts        # GET progress
        review/route.ts          # GET review summary
      webhooks/
        booking/route.ts         # POST booking webhook
  lib/
    db/
      index.ts                   # Drizzle client
      schema.ts                  # Schema definitions (Section 2)
      migrations/                # Drizzle Kit migrations
    services/
      session.service.ts         # Session CRUD, token generation, validation
      submission.service.ts      # Submission upsert, progress calculation
      framework.service.ts       # Question tree access, validation
      file-upload.service.ts     # S3 upload, processed_files record creation
      email.service.ts           # Basic transactional email (Resend)
    intake/
      tree-traversal.ts          # Question tree traversal engine
      tree-validator.ts          # Validates tree structure (all paths reachable, etc.)
      progress-calculator.ts     # Completion percentage logic
      condition-evaluator.ts     # Edge condition evaluation
    middleware/
      auth.ts                    # JWT validation + refresh
      rate-limit.ts              # Rate limiting (Redis-based sliding window)
      webhook-verify.ts          # HMAC signature verification
    utils/
      encoding.ts                # base62 encoding
      sanitize.ts                # HTML stripping, input sanitization
      jwt.ts                     # Sign/verify JWT helpers
    redis/
      client.ts                  # Redis connection
      keys.ts                    # Key naming conventions
  types/
    api.ts                       # Request/response types (Section 1)
    question-tree.ts             # QuestionTree, QuestionNode, etc. (Section 3)
    session.ts                   # Session-related types
```

### 5.2 Service Responsibilities

**SessionService** (`src/lib/services/session.service.ts`)

| Method | Responsibility |
|--------|----------------|
| `createFromWebhook(payload)` | Creates client (upsert by email), booking, and intake session. Generates magic link token. Returns intake URL. |
| `validateToken(token)` | Looks up session by magic link token. Checks expiry and status. Returns session data or throws. |
| `getState(sessionId)` | Returns full session state with section summaries. Checks Redis cache first, falls back to DB. |
| `updateNavigation(sessionId, data)` | Updates current_step, current_section, segmentation_choice. Validates transition rules. |
| `submit(sessionId)` | Transitions session to submitted. Updates booking status. Triggers confirmation emails. |
| `logAccess(sessionId, ip, userAgent)` | Writes to session_access_log. |
| `expireStale()` | Cron-callable: marks sessions past expires_at as expired. |

**SubmissionService** (`src/lib/services/submission.service.ts`)

| Method | Responsibility |
|--------|----------------|
| `upsert(sessionId, data)` | Inserts or updates submission by (session_id, question_node_id). Sanitizes content. Updates session progress. |
| `update(submissionId, sessionId, data)` | Updates existing submission. Verifies ownership. |
| `getBySession(sessionId)` | Returns all submissions for a session, grouped by framework section. |
| `getReviewSummary(sessionId)` | Builds the review screen data structure with section status and submission details. |

**FrameworkService** (`src/lib/services/framework.service.ts`)

| Method | Responsibility |
|--------|----------------|
| `getActiveFramework(version)` | Returns all active framework sections for a version. Redis-cached (1h TTL). |
| `getNextNode(sectionKey, version, segmentation, currentNodeId, sessionId)` | Traverses the question tree to resolve the next node. Loads submissions to evaluate edge conditions. |
| `validateTree(tree)` | Validates a question tree: all nodes reachable, all paths reach terminal, no orphaned nodes. Returns validation report. |
| `calculateSectionProgress(sectionKey, tree, segmentation, submissions)` | Calculates completion percentage for a section given current submissions. |

**FileUploadService** (`src/lib/services/file-upload.service.ts`)

| Method | Responsibility |
|--------|----------------|
| `uploadPdf(submissionId, file)` | Validates file (PDF MIME + magic bytes, size < 25MB). Uploads to S3. Creates processed_files record with status `queued`. Returns file metadata. |
| `getUploadsBySession(sessionId)` | Returns all processed_files for a session's submissions. |

**EmailService** (`src/lib/services/email.service.ts`)

| Method | Responsibility |
|--------|----------------|
| `sendSubmissionConfirmation(client, session)` | Sends confirmation email to client via Resend. |
| `sendFacilitatorNotification(session, completeness)` | Sends alert to facilitator with completeness score and dashboard link. |

### 5.3 Error Handling Strategy

**Application Errors** -- Custom error classes with HTTP status codes and machine-readable codes:

```typescript
// src/lib/errors.ts

abstract class AppError extends Error {
  abstract readonly statusCode: number;
  abstract readonly code: string;
  readonly details?: unknown;

  constructor(message: string, details?: unknown) {
    super(message);
    this.details = details;
  }
}

class ValidationError extends AppError {
  readonly statusCode = 400;
  readonly code = 'VALIDATION_ERROR';
}

class UnauthorizedError extends AppError {
  readonly statusCode = 401;
  readonly code = 'UNAUTHORIZED';
}

class SessionExpiredError extends AppError {
  readonly statusCode = 403;
  readonly code = 'SESSION_EXPIRED';
}

class SessionSubmittedError extends AppError {
  readonly statusCode = 403;
  readonly code = 'SESSION_SUBMITTED';
}

class NotFoundError extends AppError {
  readonly statusCode = 404;
  readonly code: string;
  constructor(resource: string) {
    super(`${resource} not found`);
    this.code = `${resource.toUpperCase().replace(/ /g, '_')}_NOT_FOUND`;
  }
}

class ConflictError extends AppError {
  readonly statusCode = 409;
  readonly code: string;
  constructor(code: string, message: string) {
    super(message);
    this.code = code;
  }
}

class RateLimitError extends AppError {
  readonly statusCode = 429;
  readonly code = 'RATE_LIMITED';
}
```

**Global Error Handler** -- Applied to all route handlers:

```typescript
// src/lib/middleware/error-handler.ts

function withErrorHandler(
  handler: (req: NextRequest, ctx: RouteContext) => Promise<NextResponse>
) {
  return async (req: NextRequest, ctx: RouteContext) => {
    try {
      return await handler(req, ctx);
    } catch (error) {
      if (error instanceof AppError) {
        return NextResponse.json(
          { ok: false, error: { code: error.code, message: error.message, details: error.details } },
          { status: error.statusCode }
        );
      }

      // Unexpected error: log full stack, return generic message
      logger.error('Unhandled error', {
        error: error instanceof Error ? error.stack : error,
        path: req.nextUrl.pathname,
        method: req.method,
      });

      return NextResponse.json(
        { ok: false, error: { code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' } },
        { status: 500 }
      );
    }
  };
}
```

**Database Errors:**
- Unique constraint violations (e.g., duplicate booking) are caught and converted to `ConflictError`
- Connection failures trigger a 503 response and log critical alert
- Query timeouts (5s default via Drizzle config) log a warning and return 500

### 5.4 Logging Approach

**Logger:** Structured JSON logging via `pino` (lightweight, fast, compatible with any log aggregator).

```typescript
// src/lib/logger.ts

import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    service: 'intake-engine',
    env: process.env.NODE_ENV,
  },
});
```

**What gets logged:**

| Level | Events |
|-------|--------|
| `info` | Session created, session validated, session submitted, submission saved, file uploaded, email sent |
| `warn` | Rate limit triggered, invalid token attempt, expired session access, validation failure |
| `error` | Database connection failure, S3 upload failure, email send failure, unhandled exceptions |
| `debug` | Tree traversal path (node sequence), condition evaluation results, progress calculation details, Redis cache hits/misses |

**Log Context:** Every log entry includes `sessionId` and `requestId` (generated per request via middleware) for tracing.

```typescript
// Example log output
{
  "level": "info",
  "service": "intake-engine",
  "env": "production",
  "requestId": "abc123",
  "sessionId": "def456",
  "msg": "submission_saved",
  "data": {
    "submissionId": "ghi789",
    "section": "competitive-landscape",
    "nodeId": "cl-q1",
    "inputType": "text",
    "progressPct": 42.5
  },
  "timestamp": "2026-03-15T14:30:00.000Z"
}
```

**No PII in Logs:** Client names, emails, and submission content are never logged. Only IDs, section keys, and metadata are included.

### 5.5 Redis Key Conventions

```
session:{sessionId}              -> cached session state (JSON, TTL 5min)
framework:v{version}             -> cached framework sections (JSON, TTL 1h)
ratelimit:magic:{ip}             -> sliding window counter (TTL 1min)
ratelimit:api:{sessionId}        -> sliding window counter (TTL 1min)
```

---

## Appendix: Phase 1 Implementation Checklist

This spec covers the backend for tasks P1-02 through P1-07 and P1-10 through P1-11 from the implementation plan. Frontend tasks (P1-08, P1-09) consume these APIs. The Phase 1 integration test (P1-12) validates the full flow end-to-end.

| Task | Spec Section | Key Deliverables |
|------|-------------|-----------------|
| P1-02: Database Schema | Section 2 | Drizzle schema + migrations for 6 tables |
| P1-03: Booking Webhook | Section 1.1 | `POST /api/webhooks/booking` with HMAC verification |
| P1-04: Magic Link Auth | Sections 1.2, 4.1-4.2 | Token validation, JWT cookie, rate limiting |
| P1-05: Framework Store | Sections 1.4, 3 | Question tree schema, JSON validation, seed data |
| P1-06: Tree Traversal | Sections 1.4, 3.3 | Edge evaluation, path resolution, progress calculation |
| P1-07: Submission API | Sections 1.5, 4.3 | Upsert logic, auto-save, progress updates |
| P1-10: Review + Submit | Sections 1.8, 1.9 | Review summary endpoint, submission flow |
| P1-11: Pause + Resume | Section 4.4 | Cross-device restore, access logging |
