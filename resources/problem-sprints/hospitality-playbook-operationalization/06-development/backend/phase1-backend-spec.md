# Backend Phase 1 Specification: The Hospitality Playbook Platform

**Focus**: Morning Huddle OS + H-Score Engine (Cinnamon, Swell, Dental Intel integrations)
**Stack**: Node.js 20 / TypeScript / Fastify / PostgreSQL / Prisma / BullMQ / Redis

---

## Architecture Overview

```
                    ┌─────────────────────────────────────────┐
                    │          INBOUND DATA SOURCES            │
                    │                                          │
  POST /webhooks/   │  Swell         Cinnamon      Dental Intel│
  swell/review ─────┤  Webhook       Pull/Webhook  Webhook/    │
  cinnamon/daily ───┤  (real-time)   (daily)       Fallback    │
  dental-intel/ ────┤                                          │
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────▼────────────────────────┐
                    │           INGESTION LAYER                │
                    │  WebhookController → Validator →         │
                    │  Queue (BullMQ) → Processor              │
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────▼────────────────────────┐
                    │         PROCESSING SERVICES              │
                    │                                          │
                    │  ReviewNLPService   CinnamonScorer       │
                    │  HScoreCalculator   HuddleSessionSvc     │
                    │  AnomalyDetector    ContentPackageAssembler│
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────▼────────────────────────┐
                    │          PERSISTENCE LAYER               │
                    │    PostgreSQL (via Prisma ORM)           │
                    │    Redis (BullMQ + response cache)       │
                    └────────────────┬────────────────────────┘
                                     │
                    ┌────────────────▼────────────────────────┐
                    │            REST API LAYER                │
                    │  Fastify + JWT auth + role middleware    │
                    └─────────────────────────────────────────┘
```

---

## Huddle Session Lifecycle

### Endpoints

#### GET `/api/v1/huddles/:practiceId/today`
Returns today's assembled content package for the practice.

**Auth**: Practice owner or Champion role for this practice.

**Response**:
```typescript
{
  session_date: string;          // "2026-03-10"
  status: "not_started" | "in_progress" | "complete" | "missed";
  streak_days: number;
  content_package: {
    theme: string;               // "Trust Transfer — How the Whole Team Builds One Experience"
    week_number: number;
    segments: {
      culture_moment: {
        type: "grant_video" | "champion_story" | "network_story";
        video_url: string;       // Mux playback URL
        duration_seconds: number;
        title: string;
      };
      wins: {
        reviews: Review[];       // Last 24hr Swell reviews, trust_transfer flagged first
        h_score_current: number;
        h_score_7day_delta: number;
        h_score_sparkline: number[]; // 7 data points
      };
      focus: {
        behavior_title: string;
        scripted_exercise: string;
        duration_seconds: number;
        practice_script: string; // The exact phrase for role-play
      };
      accountability: {
        trust_transfer_prompt: string;
        certification_spotlights: CertSpotlight[]; // Staff who earned certs since last huddle
        environmental_checklist: string[];  // 3 items
      };
      intention: {
        text: string;            // "Today, every guest leaves knowing their name matters here."
      };
    };
  };
}
```

#### POST `/api/v1/huddles/:practiceId/start`
Creates a HuddleSession record and marks it in_progress.

**Body**: `{ facilitator_id?: string }`

**Logic**:
1. Check if a session already exists for today → return existing if so
2. Create session record with `status: 'in_progress'`, `started_at: now()`
3. Return session ID

#### PATCH `/api/v1/huddles/:sessionId/segment`
Marks a segment as complete. Called as each segment finishes.

**Body**: `{ segment: 'culture_moment' | 'wins' | 'focus' | 'accountability' | 'intention', complete: true }`

**Logic**: Update the corresponding boolean column. No H-Score update triggered here.

#### POST `/api/v1/huddles/:sessionId/complete`
Marks session complete and triggers downstream updates.

**Body**: `{ staff_count?: number }`

**Logic**:
1. Update session: `status: 'complete'`, `completed_at: now()`
2. Calculate `duration_seconds`
3. Enqueue `h-score-partial-update` job — updates only the Dental Intel component score (not full recalculation)
4. Increment `champions.huddles_facilitated` if facilitator is a Champion
5. Update streak tracking (see streak logic below)
6. Return updated session + new streak count

#### POST `/api/v1/huddles/:sessionId/commit-trust`
Logs the trust transfer commitment tap.

**Logic**: Update `trust_transfer_committed = true` on session record. Log to trust_transfer_events table with `signal_type: 'checklist'`.

### Streak Logic

```typescript
// Called after every session completion
async function updateHuddleStreak(practiceId: string, sessionDate: Date): Promise<void> {
  const yesterday = subDays(sessionDate, 1);
  const yesterdaySession = await db.huddleSessions.findFirst({
    where: { practice_id: practiceId, session_date: yesterday, status: 'complete' }
  });

  if (yesterdaySession) {
    // Continuation — fetch current streak from cache or calculate
    const currentStreak = await getStreakCount(practiceId);
    await redis.set(`streak:${practiceId}`, currentStreak + 1);
  } else {
    // Broken — reset streak
    await redis.set(`streak:${practiceId}`, 1);
  }
}
```

---

## H-Score Calculation Service

### Formula

```typescript
interface HScoreComponents {
  swell: {
    star_rating_score: number;      // 0-100, normalized from Swell avg rating (last 7 days)
    velocity_score: number;         // 0-100, reviews per 100 appointments vs. cohort median
    sentiment_score: number;        // 0-100, from NLP hospitality language detection
    weight: 0.45;
  };
  cinnamon: {
    answer_rate_score: number;      // 0-100, % calls answered in 3 rings
    greeting_score: number;         // 0-100, % calls with greeting phrase detected
    hold_compliance_score: number;  // 0-100, % calls with hold < 2 min
    weight: 0.35;
  };
  dental_intel: {
    huddle_completion_score: number; // 0-100, % days with complete huddle in trailing 14 days
    certification_rate_score: number; // 0-100, % staff with active Bronze or higher
    weight: 0.20;
  };
}

function calculateHScore(components: HScoreComponents): number {
  const swellComponent =
    (components.swell.star_rating_score * 0.20 +
     components.swell.velocity_score * 0.15 +
     components.swell.sentiment_score * 0.10) / 0.45 * 100;
  // Note: sub-weights sum to 0.45 of total; normalize to 0-100 for component score

  const swellComponentNormalized =
    (components.swell.star_rating_score * (20/45) +
     components.swell.velocity_score * (15/45) +
     components.swell.sentiment_score * (10/45));

  const cinnamonComponentNormalized =
    (components.cinnamon.answer_rate_score * (15/35) +
     components.cinnamon.greeting_score * (12/35) +
     components.cinnamon.hold_compliance_score * (8/35));

  const diComponentNormalized =
    (components.dental_intel.huddle_completion_score * (12/20) +
     components.dental_intel.certification_rate_score * (8/20));

  return (
    swellComponentNormalized * 0.45 +
    cinnamonComponentNormalized * 0.35 +
    diComponentNormalized * 0.20
  );
}
```

### Sub-score Normalizations

```typescript
// Swell star rating: raw 1-5 → normalized 0-100
function normalizeStarRating(avgRating: number): number {
  // 5.0 = 100, 4.5 = 80, 4.0 = 60, 3.5 = 40, 3.0 = 20, <3.0 = 0-20 range
  return Math.max(0, Math.min(100, (avgRating - 1) / 4 * 100));
}

// Swell velocity: reviews per 100 appointments vs. cohort median
// Score = 100 if at or above cohort 75th percentile
//       = 50 if at cohort median
//       = 0 if at or below cohort 10th percentile
function normalizeVelocity(practiceRate: number, cohortMedian: number, cohortP75: number): number {
  if (practiceRate >= cohortP75) return 100;
  if (practiceRate <= 0) return 0;
  return Math.max(0, Math.min(100, (practiceRate / cohortMedian) * 50));
}

// Cinnamon answer rate: raw % → normalized 0-100
// 95%+ = 100, 80% = 75, 70% = 50, <60% = 0-25
function normalizeAnswerRate(rate: number): number {
  return Math.max(0, Math.min(100, (rate - 0.5) / 0.5 * 100));
}

// Huddle completion: % days complete in trailing 14 days → normalized 0-100
// 100% = 100, 80% = 75, 60% = 50, 40% = 25, <40% = 0-20
function normalizeHuddleCompletion(completionRate: number): number {
  return Math.round(completionRate * 100);
}
```

### Daily Recalculation Job

```typescript
// Runs at 6:00 AM per practice timezone via BullMQ cron
async function recalculateDailyHScore(practiceId: string): Promise<void> {
  const practice = await db.practices.findUnique({ where: { id: practiceId } });

  // Fetch trailing data
  const [swellData, cinnamonData, diData] = await Promise.all([
    getSwellMetrics(practiceId, { days: 7 }),
    getCinnamonMetrics(practiceId, { days: 7 }),
    getDentalIntelMetrics(practiceId, { days: 14 }),
  ]);

  // Fetch cohort benchmarks for practice type
  const benchmarks = await getCohortBenchmarks(practice.practice_type);

  // Calculate components
  const components = buildComponents(swellData, cinnamonData, diData, benchmarks);
  const hScore = calculateHScore(components);
  const level = getHScoreLevel(hScore);

  // Upsert today's record
  await db.hScoreRecords.upsert({
    where: { practice_id_recorded_date: { practice_id: practiceId, recorded_date: today() } },
    create: { practice_id: practiceId, recorded_date: today(), h_score: hScore, ...components, ...level },
    update: { h_score: hScore, ...components, ...level }
  });

  // Update practices.h_score_current
  await db.practices.update({
    where: { id: practiceId },
    data: { h_score_current: hScore, h_score_level: level }
  });

  // Check for anomaly (>10 point drop vs. 7-day average)
  await checkAnomaly(practiceId, hScore);
}
```

### Anomaly Detection

```typescript
async function checkAnomaly(practiceId: string, todayScore: number): Promise<void> {
  const sevenDayAvg = await getSevenDayAverageHScore(practiceId);

  if (sevenDayAvg - todayScore >= 10) {
    await db.hScoreRecords.update({
      where: { practice_id_recorded_date: { practice_id: practiceId, recorded_date: today() } },
      data: { anomaly_flag: true, anomaly_type: 'rapid_drop' }
    });

    // Enqueue intervention alert
    await interventionQueue.add('anomaly-alert', {
      practiceId,
      todayScore,
      sevenDayAvg,
      type: 'rapid_drop',
    });
  }
}
```

---

## Cinnamon Integration

### Option A: Webhook Push (preferred)

Register SGA endpoint with Cinnamon for daily analytics push.

**Endpoint**: `POST /webhooks/cinnamon/daily`

**Auth**: HMAC-SHA256 signature verification using shared secret

**Handler**:
```typescript
interface CinnamonDailyPayload {
  account_id: string;           // Maps to practices.cinnamon_account_id
  analytics_date: string;       // "2026-03-10"
  total_inbound_calls: number;
  calls_answered_3ring: number;
  greeting_phrase_detections?: number;  // If Cinnamon NLP supports this
  average_hold_seconds: number;
  calls_exceeding_hold_threshold: number;
}

async function handleCinnamonWebhook(payload: CinnamonDailyPayload): Promise<void> {
  const practice = await db.practices.findFirst({
    where: { cinnamon_account_id: payload.account_id }
  });
  if (!practice) throw new Error(`No practice for Cinnamon account ${payload.account_id}`);

  const answerRate = payload.calls_answered_3ring / payload.total_inbound_calls;
  const greetingRate = payload.greeting_phrase_detections != null
    ? payload.greeting_phrase_detections / payload.total_inbound_calls
    : null; // null = not available, excluded from scoring (graceful degradation)
  const holdCompliance = 1 - (payload.calls_exceeding_hold_threshold / payload.total_inbound_calls);

  await db.cinnamonDailyAnalytics.upsert({
    where: { practice_id_analytics_date: { practice_id: practice.id, analytics_date: payload.analytics_date } },
    create: { practice_id: practice.id, analytics_date: payload.analytics_date, total_calls: payload.total_inbound_calls,
              calls_answered_3ring: payload.calls_answered_3ring, answer_rate: answerRate,
              greeting_rate: greetingRate, hold_compliance_rate: holdCompliance },
    update: { answer_rate: answerRate, greeting_rate: greetingRate, hold_compliance_rate: holdCompliance }
  });

  // Trigger partial H-Score update (Cinnamon component only)
  await hScoreQueue.add('partial-update-cinnamon', { practiceId: practice.id, date: payload.analytics_date });
}
```

### Option B: Scheduled Pull (fallback)

```typescript
// BullMQ cron job: 2:00 AM daily per practice timezone
async function pullCinnamonAnalytics(practiceId: string): Promise<void> {
  const practice = await db.practices.findUnique({ where: { id: practiceId } });
  const yesterday = subDays(new Date(), 1);

  const data = await cinnamonClient.getDailyAnalytics({
    accountId: practice.cinnamon_account_id,
    date: formatDate(yesterday),
  });

  await processCinnamonData(practiceId, data, formatDate(yesterday));
}
```

---

## Swell Integration

### Webhook Handler

**Endpoint**: `POST /webhooks/swell/review`

**Auth**: Swell webhook signature header verification

```typescript
interface SwellReviewPayload {
  event: "review.created" | "review.updated";
  data: {
    id: string;
    location_id: string;    // Maps to practices.swell_account_id
    rating: number;
    review_text: string;
    reviewer_name?: string;
    platform: string;
    created_at: string;
  };
}

async function handleSwellReview(payload: SwellReviewPayload): Promise<void> {
  const practice = await db.practices.findFirst({
    where: { swell_account_id: payload.data.location_id }
  });

  // Store review immediately
  const review = await db.reviews.upsert({
    where: { external_id: payload.data.id },
    create: {
      practice_id: practice.id,
      external_id: payload.data.id,
      source: 'swell',
      received_at: new Date(payload.data.created_at),
      star_rating: payload.data.rating,
      review_text: payload.data.review_text,
      reviewer_first_name: extractFirstName(payload.data.reviewer_name),
      trust_transfer_signal: 'unprocessed',
    },
    update: { star_rating: payload.data.rating, review_text: payload.data.review_text }
  });

  // Queue NLP processing (non-blocking)
  await reviewNlpQueue.add('process-review', { reviewId: review.id }, { delay: 0 });

  // Acknowledge webhook immediately
}
```

### NLP Processing Job

```typescript
async function processReviewNLP(reviewId: string): Promise<void> {
  const review = await db.reviews.findUnique({ where: { id: reviewId } });

  const prompt = `
Analyze this dental practice patient review for hospitality signals.

Review text: "${review.review_text}"

Return a JSON object with:
1. hospitality_sentiment: float 0-1 (0=very negative, 1=very positive for hospitality/warmth signals)
2. trust_transfer_signal: "positive" | "neutral" | "negative"
   - positive: patient mentions whole team, seamless experience, everyone knowing their story, feeling briefed-about
   - negative: patient mentions feeling rushed, confused, no explanation, cold, impersonal
   - neutral: no signal
3. hospitality_phrases: array of phrases found (warm, friendly, welcoming, cared, explained, comfortable, etc.)
4. staff_mention: boolean (does the review name or describe a specific staff member)
5. staff_description: string or null (the description/name if staff_mention is true)

Return only the JSON, no explanation.
  `;

  const response = await openai.chat.completions.create({
    model: 'gpt-4o-mini',
    messages: [{ role: 'user', content: prompt }],
    response_format: { type: 'json_object' }
  });

  const nlpResult = JSON.parse(response.choices[0].message.content);

  // Resolve staff mention to staff_id if possible
  let mentionedStaffId = null;
  if (nlpResult.staff_mention && nlpResult.staff_description) {
    mentionedStaffId = await resolveStaffMention(review.practice_id, nlpResult.staff_description);
  }

  await db.reviews.update({
    where: { id: reviewId },
    data: {
      hospitality_sentiment: nlpResult.hospitality_sentiment,
      trust_transfer_signal: nlpResult.trust_transfer_signal,
      hospitality_phrases_found: nlpResult.hospitality_phrases,
      staff_mention: nlpResult.staff_mention,
      mentioned_staff_id: mentionedStaffId,
      flagged_for_huddle: nlpResult.trust_transfer_signal === 'positive' || nlpResult.staff_mention,
      nlp_processed_at: new Date()
    }
  });

  // Trigger partial H-Score Swell component update
  await hScoreQueue.add('partial-update-swell', { practiceId: review.practice_id });
}
```

---

## Dental Intel Integration

### Content Push (if API supports it)

```typescript
// Assembles today's content package and pushes to Dental Intel
async function pushHuddleContent(practiceId: string, date: Date): Promise<void> {
  const package = await assembleContentPackage(practiceId, date);

  // Dental Intel API call (structure TBD from spike)
  await dentalIntelClient.updateHuddleContent({
    locationId: practice.dental_intel_id,
    date: formatDate(date),
    content: {
      videoUrl: package.segments.culture_moment.video_url,
      slides: buildSlideData(package),
      exerciseScript: package.segments.focus.practice_script,
      intentionText: package.segments.intention.text,
    }
  });
}

// Handles completion webhook from Dental Intel
async function handleDentalIntelCompletion(payload: DICompletionPayload): Promise<void> {
  const practice = await db.practices.findFirst({
    where: { dental_intel_id: payload.location_id }
  });

  await db.huddleSessions.upsert({
    where: { practice_id_session_date: { practice_id: practice.id, session_date: payload.date } },
    create: { practice_id: practice.id, session_date: payload.date, status: 'complete',
              completed_at: payload.completed_at, duration_seconds: payload.duration_seconds,
              culture_moment_done: true, wins_segment_done: true, focus_segment_done: true,
              accountability_done: true, intention_done: true },
    update: { status: 'complete', completed_at: payload.completed_at }
  });

  await hScoreQueue.add('partial-update-di', { practiceId: practice.id, date: payload.date });
}
```

### Fallback Web URL (if Dental Intel API is insufficient)

The fallback serves the huddle content at `https://app.playbook.sga.com/huddle/:practiceId` — a mobile-optimized webpage practice owners open on any screen. Completion is tracked server-side via session analytics.

---

## ORM Schema (Prisma)

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Practice {
  id                  String    @id @default(uuid())
  orgId               String    @map("org_id")
  name                String
  practiceType        String    @map("practice_type")
  region              String?
  dentalIntelId       String?   @map("dental_intel_id")
  cinnamonAccountId   String?   @map("cinnamon_account_id")
  swellAccountId      String?   @map("swell_account_id")
  hScoreCurrent       Decimal   @default(0) @map("h_score_current")
  hScoreLevel         String    @default("emerging") @map("h_score_level")
  championStaffId     String?   @map("champion_staff_id")
  playbookVersion     String    @default("v1.0") @map("playbook_version")
  activatedAt         DateTime? @map("activated_at")
  createdAt           DateTime  @default(now()) @map("created_at")

  staffMembers        StaffMember[]
  hScoreRecords       HScoreRecord[]
  huddleSessions      HuddleSession[]
  reviews             Review[]
  cinnamonAnalytics   CinnamonDailyAnalytics[]
  champions           Champion[]

  @@map("practices")
}

model StaffMember {
  id                  String    @id @default(uuid())
  practiceId          String    @map("practice_id")
  fullName            String    @map("full_name")
  email               String?
  role                String
  hireDate            DateTime? @map("hire_date")
  ccaTier             String    @default("none") @map("cca_tier")
  bronzeCompletedAt   DateTime? @map("bronze_completed_at")
  silverCompletedAt   DateTime? @map("silver_completed_at")
  goldCompletedAt     DateTime? @map("gold_completed_at")
  isChampion          Boolean   @default(false) @map("is_champion")
  championTier        String?   @map("champion_tier")
  credlyCredentialId  String?   @map("credly_credential_id")
  bonusMultiplier     Decimal   @default(1.0) @map("bonus_multiplier")
  createdAt           DateTime  @default(now()) @map("created_at")

  practice            Practice  @relation(fields: [practiceId], references: [id])
  moduleCompletions   ModuleCompletion[]
  certifications      Certification[]
  champion            Champion?

  @@map("staff_members")
}

model HScoreRecord {
  id                    String   @id @default(uuid())
  practiceId            String   @map("practice_id")
  recordedDate          DateTime @map("recorded_date") @db.Date
  hScore                Decimal  @map("h_score")
  swellComponent        Decimal  @map("swell_component")
  cinnamonComponent     Decimal  @map("cinnamon_component")
  dentalIntelComponent  Decimal  @map("dental_intel_component")
  swellStarRating       Decimal? @map("swell_star_rating")
  swellVelocity         Decimal? @map("swell_velocity")
  swellSentiment        Decimal? @map("swell_sentiment")
  cinnamonAnswerRate    Decimal? @map("cinnamon_answer_rate")
  cinnamonGreeting      Decimal? @map("cinnamon_greeting")
  cinnamonHold          Decimal? @map("cinnamon_hold")
  diHuddleCompletion    Decimal? @map("di_huddle_completion")
  diCertificationRate   Decimal? @map("di_certification_rate")
  anomalyFlag           Boolean  @default(false) @map("anomaly_flag")
  anomalyType           String?  @map("anomaly_type")
  practiceTypeCohort    String?  @map("practice_type_cohort")
  cohortPercentile      Decimal? @map("cohort_percentile")
  createdAt             DateTime @default(now()) @map("created_at")

  practice              Practice @relation(fields: [practiceId], references: [id])

  @@unique([practiceId, recordedDate])
  @@index([practiceId, recordedDate(sort: Desc)])
  @@map("h_score_records")
}

model HuddleSession {
  id                      String    @id @default(uuid())
  practiceId              String    @map("practice_id")
  sessionDate             DateTime  @map("session_date") @db.Date
  contentPackageId        String?   @map("content_package_id")
  status                  String    @default("not_started")
  facilitatorId           String?   @map("facilitator_id")
  startedAt               DateTime? @map("started_at")
  completedAt             DateTime? @map("completed_at")
  durationSeconds         Int?      @map("duration_seconds")
  staffCount              Int?      @map("staff_count")
  cultureMomentDone       Boolean   @default(false) @map("culture_moment_done")
  winsSegmentDone         Boolean   @default(false) @map("wins_segment_done")
  focusSegmentDone        Boolean   @default(false) @map("focus_segment_done")
  accountabilityDone      Boolean   @default(false) @map("accountability_done")
  intentionDone           Boolean   @default(false) @map("intention_done")
  trustTransferCommitted  Boolean   @default(false) @map("trust_transfer_committed")
  createdAt               DateTime  @default(now()) @map("created_at")

  practice                Practice  @relation(fields: [practiceId], references: [id])

  @@unique([practiceId, sessionDate])
  @@map("huddle_sessions")
}

model Review {
  id                      String    @id @default(uuid())
  practiceId              String    @map("practice_id")
  externalId              String?   @unique @map("external_id")
  source                  String    @default("swell")
  receivedAt              DateTime  @map("received_at")
  starRating              Int       @map("star_rating")
  reviewText              String?   @map("review_text")
  reviewerFirstName       String?   @map("reviewer_first_name")
  hospitalitySentiment    Decimal?  @map("hospitality_sentiment")
  trustTransferSignal     String    @default("unprocessed") @map("trust_transfer_signal")
  hospitalityPhrasesFound String[]  @map("hospitality_phrases_found")
  staffMention            Boolean   @default(false) @map("staff_mention")
  mentionedStaffId        String?   @map("mentioned_staff_id")
  flaggedForHuddle        Boolean   @default(false) @map("flagged_for_huddle")
  wowStoryCandidate       Boolean   @default(false) @map("wow_story_candidate")
  nlpProcessedAt          DateTime? @map("nlp_processed_at")
  createdAt               DateTime  @default(now()) @map("created_at")

  practice                Practice  @relation(fields: [practiceId], references: [id])

  @@map("reviews")
}
```

---

## Authentication and Multi-Practice Tenancy

### JWT Structure
```typescript
interface SgaJWTPayload {
  sub: string;          // staff_member.id or service account ID
  practice_id: string;  // current practice context
  org_id: string;       // SGA org ID
  role: 'staff' | 'champion' | 'practice_owner' | 'regional_director' | 'sga_admin';
  exp: number;
}
```

### Role Middleware

```typescript
// Fastify preHandler middleware
function requireRole(...roles: string[]) {
  return async (request: FastifyRequest) => {
    const jwt = await request.jwtVerify<SgaJWTPayload>();
    if (!roles.includes(jwt.role)) {
      throw new UnauthorizedError(`Requires role: ${roles.join(' or ')}`);
    }
    // Attach to request for downstream use
    request.user = jwt;
  };
}

// Practice scoping — ensures staff can only access their practice's data
function requirePracticeAccess() {
  return async (request: FastifyRequest) => {
    const { practiceId } = request.params as { practiceId: string };
    const jwt = request.user as SgaJWTPayload;

    if (jwt.role === 'sga_admin' || jwt.role === 'regional_director') return; // Full access

    if (jwt.practice_id !== practiceId) {
      throw new ForbiddenError('Access denied to this practice');
    }
  };
}
```

---

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/playbook_platform

# Redis (BullMQ)
REDIS_URL=redis://localhost:6379

# External Services
CINNAMON_WEBHOOK_SECRET=...
SWELL_WEBHOOK_SECRET=...
DENTAL_INTEL_API_KEY=...
OPENAI_API_KEY=...
MUX_TOKEN_ID=...
MUX_TOKEN_SECRET=...
CREDLY_API_KEY=...
CREDLY_ORG_ID=...

# Auth
JWT_SECRET=...
JWT_EXPIRY=86400  # 24 hours

# Application
NODE_ENV=production
PORT=3000
APP_URL=https://api.playbook.sga.com
```
