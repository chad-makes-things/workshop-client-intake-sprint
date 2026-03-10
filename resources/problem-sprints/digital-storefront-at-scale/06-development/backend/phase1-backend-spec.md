# Phase 1 Backend Specification — The Practice Concierge

> AI concierge for 260+ dental practices. Phase 1 targets 20-30 pilot practices through a guided enrollment flow.

---

## 1. API Endpoints (REST)

Base URL: `/api/v1`

### 1.1 Practice CRUD

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/practices` | List all practices (paginated, filterable) | HQ session |
| `GET` | `/practices/:id` | Get practice detail | HQ session |
| `POST` | `/practices` | Create practice record | HQ session |
| `PATCH` | `/practices/:id` | Update practice fields | HQ session / practice token |
| `DELETE` | `/practices/:id` | Soft-delete practice | HQ session |
| `GET` | `/practices/:id/team` | List team members | HQ session |
| `POST` | `/practices/:id/team` | Add team member | HQ session |
| `GET` | `/practices/:id/services` | List services offered | HQ session / practice token |
| `POST` | `/practices/:id/services` | Add/update services | HQ session / practice token |

**Query parameters for `GET /practices`:**
- `page` (int, default 1)
- `limit` (int, default 25, max 100)
- `status` (enum: active, enrolled, pending, inactive)
- `search` (string, fuzzy match on name/location)
- `sortBy` (enum: name, enrollmentDate, healthScore)
- `sortOrder` (enum: asc, desc)

### 1.2 Enrollment Flow

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/enrollment/invite` | Send enrollment invitation to practice | HQ session |
| `POST` | `/enrollment/verify/:token` | Practice verifies via invitation token | Public (token-gated) |
| `GET` | `/enrollment/:id` | Get enrollment status and progress | HQ session / practice token |
| `PATCH` | `/enrollment/:id` | Update enrollment step data | Practice token |
| `POST` | `/enrollment/:id/template-select` | Practice selects template | Practice token |
| `POST` | `/enrollment/:id/approve` | HQ approves storefront for deployment | HQ session |
| `POST` | `/enrollment/:id/deploy` | Trigger storefront deployment | HQ session |
| `GET` | `/enrollment/batch` | List all enrollments with status filters | HQ session |

**Enrollment states:** `invited` -> `verified` -> `data-gathered` -> `template-selected` -> `review` -> `approved` -> `deployed`

### 1.3 Harvester Trigger

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/harvester/trigger` | Trigger scrape for a single practice | API key |
| `POST` | `/harvester/batch` | Trigger scrape for multiple practices | API key |
| `GET` | `/harvester/:practiceId/results` | Get harvested data for a practice | HQ session |
| `POST` | `/harvester/:practiceId/confirm` | Confirm/correct harvested data | HQ session / practice token |

### 1.4 Concierge Conversation

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `POST` | `/concierge/:practiceSlug/messages` | Send message, receive streamed response | Practice token |
| `GET` | `/concierge/:practiceSlug/history` | Get conversation history | Practice token / HQ session |
| `GET` | `/concierge/:practiceSlug/session` | Get or create active session | Practice token |
| `POST` | `/concierge/:practiceSlug/action` | Submit action card response (approve/reject/flag) | Practice token |
| `DELETE` | `/concierge/:practiceSlug/session` | End active session | Practice token |

**Streaming:** The `POST /messages` endpoint returns a `text/event-stream` response using Server-Sent Events (SSE). Events include `delta` (text chunk), `tool_use` (action card trigger), and `done`.

### 1.5 Dashboard Data

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/dashboard/overview` | Aggregate stats (enrollment counts, health averages) | HQ session |
| `GET` | `/dashboard/enrollment-progress` | Enrollment funnel data | HQ session |
| `GET` | `/dashboard/alerts` | Active alerts and flags | HQ session |
| `GET` | `/dashboard/activity` | Recent activity feed | HQ session |

### 1.6 Health Scores

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/practices/:id/health` | Current health score breakdown | HQ session |
| `GET` | `/practices/:id/health/history` | Health score over time | HQ session |
| `POST` | `/health/recalculate` | Trigger recalculation for one or all practices | API key |

---

## 2. ORM Schema (Prisma)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─── Practice ───────────────────────────────────────────────────────

model Practice {
  id              String    @id @default(cuid())
  name            String
  slug            String    @unique
  phone           String?
  email           String?
  website         String?
  addressLine1    String?
  addressLine2    String?
  city            String?
  state           String?
  zip             String?
  specialty       String    @default("general") // general, pediatric, orthodontic, periodontic, etc.
  status          PracticeStatus @default(PENDING)
  logoUrl         String?
  brandColors     Json?     // { primary: "#hex", secondary: "#hex", accent: "#hex" }
  operatingHours  Json?     // { mon: { open: "08:00", close: "17:00" }, ... }
  metadata        Json?     // Flexible field for additional practice data

  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
  deletedAt       DateTime?

  teamMembers     TeamMember[]
  services        Service[]
  enrollment      Enrollment?
  conversations   ConversationMessage[]
  healthScores    HealthScore[]
  harvestedData   HarvestedData[]
  templates       Template[]

  @@index([status])
  @@index([slug])
}

enum PracticeStatus {
  PENDING
  ENROLLED
  ACTIVE
  INACTIVE
}

// ─── Team Member ────────────────────────────────────────────────────

model TeamMember {
  id          String   @id @default(cuid())
  practiceId  String
  name        String
  role        String   // dentist, hygienist, office-manager, front-desk
  email       String?
  phone       String?
  isPrimary   Boolean  @default(false) // Primary contact for enrollment
  bio         String?
  photoUrl    String?
  credentials String?  // DDS, DMD, RDH, etc.

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  practice    Practice @relation(fields: [practiceId], references: [id])

  @@index([practiceId])
}

// ─── Service ────────────────────────────────────────────────────────

model Service {
  id          String   @id @default(cuid())
  practiceId  String
  name        String   // "Teeth Whitening", "Invisalign", "Root Canal"
  category    String   // preventive, restorative, cosmetic, orthodontic, surgical, emergency
  description String?
  isActive    Boolean  @default(true)
  sortOrder   Int      @default(0)

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  practice    Practice @relation(fields: [practiceId], references: [id])

  @@index([practiceId])
  @@unique([practiceId, name])
}

// ─── Template ───────────────────────────────────────────────────────

model Template {
  id            String   @id @default(cuid())
  practiceId    String?  // null for master templates
  templateKey   String   // "modern-minimal", "warm-family", "clinical-pro"
  name          String
  previewUrl    String?
  configJson    Json     // Layout + content mapping configuration
  isGenerated   Boolean  @default(false) // true if AI-generated for this practice
  deployedUrl   String?
  deployedAt    DateTime?

  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  practice      Practice? @relation(fields: [practiceId], references: [id])

  @@index([practiceId])
  @@index([templateKey])
}

// ─── Enrollment ─────────────────────────────────────────────────────

model Enrollment {
  id              String           @id @default(cuid())
  practiceId      String           @unique
  inviteToken     String           @unique @default(cuid())
  inviteSentAt    DateTime?
  verifiedAt      DateTime?
  status          EnrollmentStatus @default(INVITED)
  currentStep     Int              @default(0)
  stepData        Json?            // Data collected at each step
  selectedTemplate String?
  reviewNotes     String?
  approvedBy      String?
  approvedAt      DateTime?
  deployedAt      DateTime?

  createdAt       DateTime         @default(now())
  updatedAt       DateTime         @updatedAt

  practice        Practice         @relation(fields: [practiceId], references: [id])

  @@index([status])
  @@index([inviteToken])
}

enum EnrollmentStatus {
  INVITED
  VERIFIED
  DATA_GATHERED
  TEMPLATE_SELECTED
  REVIEW
  APPROVED
  DEPLOYED
  REJECTED
}

// ─── Conversation Message ───────────────────────────────────────────

model ConversationMessage {
  id          String      @id @default(cuid())
  practiceId  String
  sessionId   String
  role        MessageRole // USER, ASSISTANT, SYSTEM, TOOL
  content     String
  toolName    String?     // If role=TOOL, which tool was called
  toolInput   Json?       // Tool call input
  toolOutput  Json?       // Tool call result
  metadata    Json?       // Action card state, attachments, etc.

  createdAt   DateTime    @default(now())

  practice    Practice    @relation(fields: [practiceId], references: [id])

  @@index([practiceId, sessionId])
  @@index([createdAt])
}

enum MessageRole {
  USER
  ASSISTANT
  SYSTEM
  TOOL
}

// ─── Health Score ───────────────────────────────────────────────────

model HealthScore {
  id              String   @id @default(cuid())
  practiceId      String
  overallScore    Float    // 0-100
  dataCompleteness Float   // 0-100 — how much practice data is filled in
  contentFreshness Float   // 0-100 — how recently content was updated
  templateFit     Float    // 0-100 — how well template matches practice data
  engagementScore Float    // 0-100 — concierge interaction frequency
  calculatedAt    DateTime @default(now())

  practice        Practice @relation(fields: [practiceId], references: [id])

  @@index([practiceId, calculatedAt])
}

// ─── Harvested Data ─────────────────────────────────────────────────

model HarvestedData {
  id          String          @id @default(cuid())
  practiceId  String
  source      String          // "website", "google-business", "yelp", "healthgrades"
  rawHtml     String?         // Raw scraped content (stored but not indexed)
  extractedData Json          // LLM-structured extraction
  confidence  Float           // 0-1 confidence score from LLM
  status      HarvestStatus   @default(PENDING_REVIEW)
  reviewedBy  String?
  reviewedAt  DateTime?

  createdAt   DateTime        @default(now())
  updatedAt   DateTime        @updatedAt

  practice    Practice        @relation(fields: [practiceId], references: [id])

  @@index([practiceId, source])
  @@index([status])
}

enum HarvestStatus {
  PENDING_REVIEW
  CONFIRMED
  CORRECTED
  REJECTED
}
```

---

## 3. Service Layer

### 3.1 HarvesterService

Scrapes public practice data and uses LLM extraction to structure it.

```typescript
class HarvesterService {
  // Scrape a practice's website and public listings
  async harvestPractice(practiceId: string): Promise<HarvestedData[]>

  // Scrape a single source
  async scrapeSource(url: string, source: SourceType): Promise<RawScrapeResult>

  // Extract structured data from raw HTML using Claude
  async extractWithLLM(raw: RawScrapeResult, practiceContext: PracticeContext): Promise<ExtractedData>

  // Compare harvested data against existing practice record, flag differences
  async diffAgainstRecord(practiceId: string, extracted: ExtractedData): Promise<DataDiff[]>

  // Batch harvest for multiple practices
  async batchHarvest(practiceIds: string[]): Promise<BatchHarvestResult>
}
```

**LLM extraction prompt pattern:**
1. Pass raw HTML with instructions to extract: practice name, address, phone, hours, services, team members, insurance accepted, specialties, patient reviews summary.
2. Return structured JSON matching the `Practice`, `TeamMember`, and `Service` schemas.
3. Include confidence score per field.

**Rate limiting:** Max 5 concurrent scrapes. 2-second delay between requests to same domain. Retry with exponential backoff on 429/503.

### 3.2 EnrollmentService

State machine managing the enrollment lifecycle.

```typescript
class EnrollmentService {
  // Create invitation and send email
  async createInvite(practiceId: string, contactEmail: string): Promise<Enrollment>

  // Verify practice via token (transitions INVITED -> VERIFIED)
  async verify(token: string): Promise<{ enrollment: Enrollment; practiceToken: string }>

  // Update step data (validates against current step requirements)
  async updateStep(enrollmentId: string, stepData: StepPayload): Promise<Enrollment>

  // Advance to next step (validates current step is complete)
  async advanceStep(enrollmentId: string): Promise<Enrollment>

  // Select template (transitions to TEMPLATE_SELECTED)
  async selectTemplate(enrollmentId: string, templateKey: string): Promise<Enrollment>

  // Submit for review (transitions to REVIEW)
  async submitForReview(enrollmentId: string): Promise<Enrollment>

  // HQ approves (transitions to APPROVED)
  async approve(enrollmentId: string, approvedBy: string, notes?: string): Promise<Enrollment>

  // Trigger deployment (transitions to DEPLOYED)
  async deploy(enrollmentId: string): Promise<Enrollment>

  // Reject with reason
  async reject(enrollmentId: string, reason: string): Promise<Enrollment>

  // Get enrollment progress summary
  async getProgress(enrollmentId: string): Promise<EnrollmentProgress>
}
```

**State machine transitions:**

```
INVITED --[verify]--> VERIFIED
VERIFIED --[gather data via concierge]--> DATA_GATHERED
DATA_GATHERED --[select template]--> TEMPLATE_SELECTED
TEMPLATE_SELECTED --[submit]--> REVIEW
REVIEW --[approve]--> APPROVED
REVIEW --[reject]--> REJECTED
APPROVED --[deploy]--> DEPLOYED
```

Each transition validates preconditions. Invalid transitions throw `InvalidStateTransitionError`.

**Enrollment steps (within DATA_GATHERED):**
1. Confirm practice identity (name, address, phone)
2. Review and correct harvested data
3. Add/verify team members
4. Confirm services offered
5. Upload logo and select brand colors
6. Review operating hours

### 3.3 ConversationService

Manages Claude API integration for the concierge chat.

```typescript
class ConversationService {
  // Start or resume a conversation session
  async getOrCreateSession(practiceSlug: string): Promise<ConversationSession>

  // Send a message and stream the response
  async sendMessage(
    sessionId: string,
    userMessage: string
  ): AsyncGenerator<StreamEvent>

  // Process action card response (approve/reject/flag)
  async handleAction(
    sessionId: string,
    actionId: string,
    response: ActionResponse
  ): AsyncGenerator<StreamEvent>

  // Build message history for Claude API (with context window management)
  async buildMessages(sessionId: string): Promise<ClaudeMessage[]>

  // Get conversation history for HQ review
  async getHistory(practiceSlug: string, options?: PaginationOptions): Promise<ConversationMessage[]>

  // End session
  async endSession(sessionId: string): Promise<void>
}
```

**Context window management:**
- Keep system prompt + last 20 messages in context
- Summarize older messages if conversation exceeds 50 turns
- Always include the full practice context in system prompt

### 3.4 TemplateService

Generates and deploys practice storefronts from templates.

```typescript
class TemplateService {
  // List available master templates
  async listTemplates(): Promise<Template[]>

  // Generate a practice-specific template preview
  async generatePreview(practiceId: string, templateKey: string): Promise<PreviewResult>

  // Get AI-recommended template based on practice data
  async recommendTemplate(practiceId: string): Promise<TemplateRecommendation>

  // Deploy storefront (generates static site, uploads to CDN)
  async deploy(practiceId: string, templateKey: string): Promise<DeployResult>

  // Update deployed storefront with new practice data
  async updateDeployment(practiceId: string): Promise<DeployResult>
}
```

### 3.5 DashboardService

Aggregates data for HQ and executive dashboards.

```typescript
class DashboardService {
  // Overview stats
  async getOverview(): Promise<DashboardOverview>
  // { totalPractices, enrolled, deployed, avgHealthScore, activeConversations }

  // Enrollment funnel
  async getEnrollmentProgress(): Promise<EnrollmentFunnel>
  // { invited: 30, verified: 25, dataGathered: 20, templateSelected: 18, deployed: 12 }

  // Active alerts (low health scores, stale data, enrollment stalls)
  async getAlerts(): Promise<Alert[]>

  // Recent activity feed
  async getActivity(options?: PaginationOptions): Promise<ActivityEvent[]>

  // Practice-level analytics
  async getPracticeAnalytics(practiceId: string): Promise<PracticeAnalytics>
}
```

---

## 4. Claude API Integration

### 4.1 System Prompt Structure

The concierge system prompt is assembled dynamically per practice:

```
[IDENTITY]
You are the Practice Concierge for {practice.name}, an AI assistant helping
dental practices set up and maintain their digital storefront. You represent
SGA (Smile Generation Associates) and follow SGA brand standards.

[SGA BRAND STANDARDS]
- Voice: Professional, warm, efficient. Never salesy.
- Tone: Helpful guide, not a chatbot. Speak as a knowledgeable colleague.
- Language: Use dental industry terminology appropriately.
- Never make clinical claims. Never provide medical advice.
- Always confirm before making changes to practice data.

[PRACTICE CONTEXT]
Name: {practice.name}
Specialty: {practice.specialty}
Location: {practice.city}, {practice.state}
Current enrollment status: {enrollment.status}
Current enrollment step: {enrollment.currentStep}
Services: {services as comma-separated list}
Team: {team members with roles}
Health score: {healthScore.overallScore}/100

[ENROLLMENT CONTEXT — if in enrollment]
You are guiding this practice through enrollment step {currentStep}.
Step objective: {stepObjective}
Data collected so far: {stepData summary}
Data still needed: {remaining fields}

[SPECIALTY BEST PRACTICES]
{Loaded from specialty-specific prompt fragments}
- General: Focus on family-friendly messaging, preventive care emphasis
- Pediatric: Child-friendly language, parent-focused features, fun visual themes
- Orthodontic: Before/after focus, treatment timeline, financing options
- Periodontic: Clinical credibility, referral-friendly, procedure education
- Cosmetic: Visual portfolio, smile gallery, luxury positioning

[BEHAVIORAL RULES]
1. Guide conversation toward completing the current enrollment step
2. Present data for verification using action cards, not plain text
3. When practice confirms data, call updatePracticeData immediately
4. Never skip steps — complete current step before moving on
5. If practice asks off-topic questions, answer briefly then redirect
6. Celebrate progress — acknowledge completed steps
7. If you detect inconsistencies in data, flag them with reportChange
```

### 4.2 Tool Definitions

Tools provided to Claude for the concierge:

```typescript
const conciergeTools = [
  {
    name: "updatePracticeData",
    description: "Update verified practice information. Call after practice confirms data.",
    input_schema: {
      type: "object",
      properties: {
        practiceId: { type: "string" },
        field: {
          type: "string",
          enum: [
            "name", "phone", "email", "address",
            "hours", "services", "team", "brandColors", "logo"
          ]
        },
        value: { type: "any" },
        confirmedBy: { type: "string", description: "Who confirmed this data" }
      },
      required: ["practiceId", "field", "value"]
    }
  },
  {
    name: "selectTemplate",
    description: "Record the practice's template selection. Show preview first.",
    input_schema: {
      type: "object",
      properties: {
        practiceId: { type: "string" },
        templateKey: { type: "string" },
        reason: { type: "string", description: "Why this template fits the practice" }
      },
      required: ["practiceId", "templateKey"]
    }
  },
  {
    name: "approveStorefront",
    description: "Mark storefront as ready for HQ review after practice approves preview.",
    input_schema: {
      type: "object",
      properties: {
        practiceId: { type: "string" },
        practiceApproverName: { type: "string" },
        notes: { type: "string" }
      },
      required: ["practiceId", "practiceApproverName"]
    }
  },
  {
    name: "reportChange",
    description: "Flag a data inconsistency or change detected during conversation.",
    input_schema: {
      type: "object",
      properties: {
        practiceId: { type: "string" },
        changeType: {
          type: "string",
          enum: ["inconsistency", "update-request", "concern", "compliment"]
        },
        description: { type: "string" },
        severity: { type: "string", enum: ["low", "medium", "high"] }
      },
      required: ["practiceId", "changeType", "description"]
    }
  },
  {
    name: "getHealthScore",
    description: "Retrieve the current health score breakdown for the practice.",
    input_schema: {
      type: "object",
      properties: {
        practiceId: { type: "string" }
      },
      required: ["practiceId"]
    }
  }
]
```

### 4.3 Conversation Session Management

```typescript
interface ConversationSession {
  id: string
  practiceSlug: string
  practiceId: string
  startedAt: Date
  lastMessageAt: Date
  messageCount: number
  enrollmentStepAtStart: number
  isActive: boolean
}
```

**Session lifecycle:**
1. `getOrCreateSession` checks for an active session (last message < 30 min). If none, creates a new one.
2. Each message updates `lastMessageAt` and increments `messageCount`.
3. Sessions auto-expire after 30 minutes of inactivity.
4. On new session creation, the full practice context is refreshed from the database.
5. Claude API calls use `max_tokens: 1024` for responses, `temperature: 0.3` for consistency.

**API call pattern:**
```typescript
const response = await anthropic.messages.create({
  model: "claude-sonnet-4-20250514",
  max_tokens: 1024,
  temperature: 0.3,
  system: buildSystemPrompt(practice, enrollment),
  tools: conciergeTools,
  messages: await buildMessages(sessionId),
  stream: true
})
```

---

## 5. Authentication & Authorization

### 5.1 API Keys (Internal Services)

- Used by: Harvester triggers, health score recalculation, inter-service calls
- Format: `sga_sk_{random_32_chars}`
- Stored in environment variables
- Validated via middleware: `x-api-key` header
- Rate limit: 1000 requests/minute per key

### 5.2 Session-Based Auth (HQ Dashboard)

- Used by: HQ staff accessing the admin dashboard
- Flow: Email/password login -> JWT stored in httpOnly cookie
- JWT payload: `{ userId, email, role, iat, exp }`
- Roles: `admin`, `manager`, `viewer`
- Session expiry: 8 hours
- Refresh token rotation on each request within the last hour

### 5.3 Practice Tokens (Concierge Chat)

- Used by: Practice staff accessing the concierge chat
- Flow: Practice receives enrollment invite link with token -> token exchanged for short-lived practice JWT
- JWT payload: `{ practiceId, practiceSlug, enrollmentId, role: "practice", iat, exp }`
- Session expiry: 2 hours (auto-refresh while active)
- No password required — token-gated access scoped to their own practice data

### 5.4 Authorization Matrix

| Resource | Admin | Manager | Viewer | Practice |
|----------|-------|---------|--------|----------|
| All practices (list/read) | Yes | Yes | Yes | No |
| Practice (own data) | Yes | Yes | Yes | Own only |
| Enrollment (approve/deploy) | Yes | Yes | No | No |
| Enrollment (update steps) | Yes | Yes | No | Own only |
| Concierge (conversation) | Yes | Yes | No | Own only |
| Dashboard (executive) | Yes | Yes | Yes | No |
| Harvester (trigger) | Yes | No | No | No |
| Health (recalculate) | Yes | No | No | No |

---

## 6. Error Handling Patterns

### 6.1 Error Response Format

```json
{
  "error": {
    "code": "ENROLLMENT_INVALID_TRANSITION",
    "message": "Cannot transition from INVITED to TEMPLATE_SELECTED. Practice must verify first.",
    "details": {
      "currentStatus": "INVITED",
      "attemptedStatus": "TEMPLATE_SELECTED",
      "allowedTransitions": ["VERIFIED"]
    },
    "requestId": "req_abc123"
  }
}
```

### 6.2 Error Categories

| Code Prefix | HTTP Status | Category |
|-------------|-------------|----------|
| `AUTH_*` | 401/403 | Authentication and authorization |
| `VALIDATION_*` | 400 | Input validation failures |
| `ENROLLMENT_*` | 400/409 | Enrollment state machine violations |
| `HARVESTER_*` | 502/504 | External scraping failures |
| `CONCIERGE_*` | 502/504 | Claude API failures |
| `NOT_FOUND_*` | 404 | Resource not found |
| `RATE_LIMIT_*` | 429 | Rate limit exceeded |
| `INTERNAL_*` | 500 | Unexpected server errors |

### 6.3 Retry Strategy

| Service | Retries | Backoff | Circuit Breaker |
|---------|---------|---------|-----------------|
| Claude API | 3 | Exponential (1s, 2s, 4s) | Open after 5 consecutive failures, half-open after 30s |
| Harvester (scrape) | 2 | Exponential (2s, 4s) | Open after 10 failures per domain |
| Database | 2 | Fixed (100ms) | None (fail fast) |
| Template deploy | 3 | Exponential (1s, 2s, 4s) | None |

### 6.4 Logging

All errors are logged with:
- `requestId` for tracing
- `practiceId` when available
- `userId` or `sessionId` for attribution
- Stack trace for 5xx errors
- Claude API errors include the full request/response (with PII redacted)

---

## 7. Phase 1 Scope

### What's Included

- **20-30 pilot practices** enrolled through the concierge flow
- Full enrollment state machine (invite through deploy)
- Concierge chat with Claude integration (enrollment guidance only)
- Harvester for website scraping (single source: practice website)
- 3 master templates (modern-minimal, warm-family, clinical-pro)
- HQ dashboard with enrollment tracking and basic health scores
- Practice token auth for concierge access
- HQ session auth for dashboard access

### What's Deferred to Phase 2

- Multi-source harvesting (Google Business, Yelp, Healthgrades)
- Post-enrollment concierge (ongoing content updates, change detection)
- Advanced health score algorithms (SEO, performance, accessibility)
- Practice self-service dashboard
- Custom template builder
- Automated re-scraping schedules
- SSO integration for HQ staff
- Webhook notifications for enrollment events
- Analytics and reporting beyond basic dashboard
- Scale testing for 260+ practices

### Infrastructure (Phase 1)

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Runtime | Node.js 20 + Express | Team familiarity, streaming support |
| Database | PostgreSQL 16 | Relational data, JSON support, Prisma compatibility |
| ORM | Prisma | Type-safe, migration management, schema-first |
| AI | Claude API (claude-sonnet-4-20250514) | Best balance of quality and speed for conversational use |
| Hosting | Railway or Render | Simple deploy, managed PostgreSQL, WebSocket support |
| CDN | Cloudflare Pages | Static site hosting for deployed storefronts |
| Queue | BullMQ + Redis | Harvester jobs, template generation |
| Email | Resend | Enrollment invitations |

### Environment Variables

```
DATABASE_URL=postgresql://...
ANTHROPIC_API_KEY=sk-ant-...
SGA_API_KEY=sga_sk_...
JWT_SECRET=...
RESEND_API_KEY=re_...
REDIS_URL=redis://...
CLOUDFLARE_API_TOKEN=...
CLOUDFLARE_ACCOUNT_ID=...
NODE_ENV=production
LOG_LEVEL=info
```
