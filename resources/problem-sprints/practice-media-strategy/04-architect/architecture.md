# Architecture — The Content Concierge

## System Overview

The Content Concierge is a media capture, production, and distribution platform that serves as the content arm of SGA's Practice Concierge ecosystem. It connects practice management data to a smart prompt engine, provides a zero-friction capture interface, processes raw content through an AI pipeline, and distributes finished assets across practice and brand channels automatically.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CONTENT CONCIERGE PLATFORM                       │
│                                                                         │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────┐ │
│  │   PROMPT     │   │   CAPTURE    │   │  PRODUCTION  │   │ DISTRIB- │ │
│  │   ENGINE     │   │   LAYER      │   │  PIPELINE    │   │ UTION    │ │
│  │             │   │              │   │              │   │ ENGINE   │ │
│  │ PMS Sync    │──>│ Mobile App   │──>│ AI Photo     │──>│ Practice │ │
│  │ Content Cal │   │ SMS/WhatsApp │   │ AI Video     │   │ Social   │ │
│  │ Playbook    │   │ Testimonial  │   │ AI Captions  │   │ Brand    │ │
│  │ Rules       │   │ Kiosk        │   │ Human QC     │   │ Social   │ │
│  │ Event Mode  │   │ Passive Cam  │   │ Templates    │   │ Digital  │ │
│  │             │   │              │   │              │   │ Store    │ │
│  └──────┬──────┘   └──────┬───────┘   └──────┬───────┘   └────┬─────┘ │
│         │                 │                   │                │       │
│         v                 v                   v                v       │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                    SHARED DATA LAYER                              │  │
│  │  Content DB │ Asset Storage (S3) │ Metadata │ Analytics │ Consent │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│         │                                                     │       │
│         v                                                     v       │
│  ┌──────────────┐                                    ┌──────────────┐ │
│  │  HQ DASHBOARD │                                    │  SCORECARD   │ │
│  │  Review Queue │                                    │  Champion    │ │
│  │  Analytics    │                                    │  Practice    │ │
│  │  Reporting    │                                    │  Network     │ │
│  └──────────────┘                                    └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
         │                                                     │
         v                                                     v
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Sikka ONE   │  │  Social APIs │  │  Digital      │  │  Practice    │
│  API (PMS)   │  │  (IG, FB,    │  │  Storefront   │  │  Concierge   │
│              │  │  YT, LI,     │  │  (prior       │  │  (prior      │
│  400+ PMS    │  │  TikTok)     │  │  sprint)      │  │  sprint)     │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
```

---

## Component Architecture

### 1. Prompt Engine

**Purpose**: Know what content to capture, when, and tell the right person.

**Inputs**:
- **Sikka ONE API** — Nightly batch sync of appointment schedules across all connected practices. Pulls: patient ID, procedure codes, appointment time, provider, location.
- **Content Calendar** — HQ-defined weekly/monthly capture templates per playbook type.
- **Playbook Rules** — Per-practice configuration: which procedure codes trigger capture prompts, frequency targets, content types.
- **Event Calendar** — Manual event entries (4Front, Pikos courses, partner signings) that trigger Event Mode.
- **Post-op Calendar** — Tracks surgery dates and auto-schedules follow-up capture prompts (2 weeks, 30 days, 90 days post-treatment).

**Logic**:
```
NIGHTLY JOB:
  FOR each practice in active_practices:
    appointments = sikka_api.get_tomorrow_schedule(practice_id)
    FOR each appointment in appointments:
      IF appointment.procedure_code IN playbook.high_value_codes:
        IF patient.media_consent == SIGNED:
          CREATE capture_assignment(
            champion: practice.content_champion,
            patient: appointment.patient,
            procedure: appointment.procedure_type,
            capture_type: playbook.get_capture_template(procedure),
            time: appointment.time,
            guidance: playbook.get_capture_guidance(procedure)
          )

    // Also check content calendar for non-PMS captures
    calendar_tasks = content_calendar.get_tasks(practice, tomorrow)
    FOR each task in calendar_tasks:
      CREATE capture_assignment(champion, task)

MORNING JOB (7:30 AM local time per practice):
  FOR each champion with assignments:
    SEND daily_brief(champion, assignments)
    // SMS in Phase 1, push notification in Phase 2+

BRAND WEEKLY JOB (Monday 8:00 AM):
  FOR each leader in [Jensen, Ellingson, Wong, Olson]:
    topic = brand_calendar.get_weekly_topic(leader)
    SEND topic_prompt(leader, topic, script_outline)
```

**Tech Stack**:
- Sikka ONE API (PMS integration)
- PostgreSQL (assignment database)
- Node.js worker (nightly sync jobs)
- Twilio (Phase 1 SMS) → Firebase Cloud Messaging (Phase 2+ push)
- Redis (job queue)

### 2. Capture Layer

**Purpose**: Make content capture stupidly simple — camera-first, guided, under 60 seconds.

**Interfaces**:

| Interface | Phase | For | How |
|-----------|-------|-----|-----|
| SMS/WhatsApp | Phase 1 | Champions + Leadership | Twilio → receive media via MMS/WhatsApp → S3 |
| Mobile App (React Native) | Phase 2+ | Champions + Leadership | Camera-first UX, guided overlays, in-app consent |
| Testimonial Kiosk | Phase 3 | Patients (self-serve) | iPad app at checkout, 60-sec video prompt |
| Passive Camera | Phase 3 | Operatory capture | Insta360 API → cloud upload → AI mining |

**Mobile App Screens** (Phase 2):
1. **Home / Dashboard**: Today's assignments, streak counter, recent publishes
2. **Capture Mode**: Camera with overlay guide (framing, lighting tips). Toggles between photo/video. Timer for video. Teleprompter mode for doctor topics.
3. **Assignment Detail**: Patient info (from PMS), capture checklist, reference photos (for matching before/after angles), consent status.
4. **Content Feed**: "Your Published Content" — shows champion what their captures became after production.
5. **Scorecard**: Weekly/monthly stats, practice ranking, network leaderboard.

**Capture Metadata** (auto-tagged on every capture):
```json
{
  "capture_id": "uuid",
  "practice_id": "innovative-dental-springfield",
  "champion_id": "jessica-martinez",
  "patient_id": "PMS-12345",  // null for non-patient content
  "content_type": "before_after_before",
  "procedure_type": "full_arch_implant",
  "playbook": "implant_cosmetic",
  "media_type": "photo",
  "consent_id": "consent-uuid",
  "captured_at": "2026-03-11T13:50:00-06:00",
  "device": "iPhone 15 Pro",
  "location": { "lat": 37.19, "lng": -93.28 }
}
```

**Tech Stack**:
- React Native (iOS + Android)
- Expo Camera API
- AWS S3 (media storage)
- Background upload via tus protocol (resumable uploads)
- Digital consent: DocuSign API or custom signature capture

### 3. Production Pipeline

**Purpose**: Transform raw captures into platform-ready content automatically.

**Pipeline Stages**:

```
RAW CAPTURE → INGEST → ENHANCE → FORMAT → GENERATE → REVIEW → APPROVE
     │           │         │          │          │         │         │
     S3       Validate   Color     Resize    Captions   Human    Queue for
     upload   metadata   correct   format    hashtags   review   distribution
              check      crop      per       AI copy    in HQ
              HIPAA      brand     platform  suggest    dashboard
              screen     overlay
```

**Stage Details**:

| Stage | What Happens | Tech |
|-------|-------------|------|
| **Ingest** | Validate metadata, check consent exists, screen for HIPAA (no visible patient identifiers in non-consented content) | Custom Lambda, AWS Rekognition (face detection for HIPAA screening) |
| **Enhance (Photos)** | Auto color correction, exposure adjustment, dental-specific smart crop (focus on smile area), before/after side-by-side generation | Cloudinary API (transformation pipeline) |
| **Enhance (Video)** | Stabilization, color grade, AI transcription, auto-captioning (burned-in, bold, high-contrast) | FFmpeg (processing), Whisper API (transcription), custom caption renderer |
| **Format** | Resize/reformat for each target platform | Cloudinary (photos), FFmpeg (video) |
| **Generate** | AI-written captions, hashtags, descriptions per platform. Alt text for accessibility. | Claude API or GPT-4o (text generation with dental context prompt) |
| **AI Video Cutting** | For videos >60 sec: detect highlight moments, cut into 15s/30s/60s clips, generate thumbnail options | OpusClip API or custom model trained on dental content engagement patterns |
| **Template Application** | Apply practice-specific branding (logo, colors, fonts) and SGA brand templates | Cloudinary overlays, custom template engine |

**Output per capture**:

| Input | Outputs Generated |
|-------|------------------|
| 1 before photo | 4 formatted versions (IG square, IG story, FB, storefront) + before/after placeholder |
| 1 after photo | 4 formatted versions + completed before/after side-by-side (4 formats) = 8 pieces |
| 1 testimonial video (60s) | Full-length (3 formats: vertical, horizontal, square) + highlight clip (15s, 3 formats) + quote graphic + AI caption set = 10 pieces |
| 1 doctor topic video (4 min) | Full LinkedIn video + 2-3 short clips (15-60s) + quote graphic + blog excerpt draft = 7-9 pieces |

**Tech Stack**:
- AWS Lambda (orchestration)
- Cloudinary (photo processing)
- FFmpeg on ECS (video processing)
- OpenAI Whisper (transcription)
- Claude API (caption/copy generation)
- OpusClip API (video highlight detection)
- S3 (processed asset storage)

### 4. Review & Approval Queue

**Purpose**: Human quality gate between AI production and public distribution.

**HQ Dashboard** (Web app):

**Views**:
- **Review Queue**: Cards showing each content piece. Thumbnail, AI-generated caption, target platform, source practice. Batch select → approve/reject.
- **Network Pipeline**: Kanban view showing content flowing through stages (Captured → Processing → Review → Scheduled → Published).
- **Practice Health**: Grid of all practices with content health scores (green/yellow/red). Click into any practice for detailed stats.
- **Brand Content**: Separate tab for SGA brand-level content (leadership videos, partner stories, event content).
- **Analytics**: Content performance tied to business metrics (not just vanity). Views, engagement, link clicks, consultation bookings attributed.

**Approval Workflow**:
1. Content appears in queue with AI confidence score (high confidence = safe to auto-approve in future)
2. Reviewer can: Approve, Edit Caption, Reject (with feedback to champion), Star (escalate to premium treatment)
3. Approved content enters distribution queue immediately
4. Over time, content types with >95% approval rate can be auto-approved (human spot-checks only)

**Tech Stack**:
- Next.js web application
- PostgreSQL (content metadata, workflow state)
- Real-time updates via WebSocket
- Role-based access (HQ admin, practice manager view-only, champion mobile-only)

### 5. Distribution Engine

**Purpose**: Route approved content to the right channels at the right times, automatically.

**Routing Rules** (per playbook):

```yaml
playbook_implant_cosmetic:
  before_after:
    - platform: instagram_feed
      account: practice_instagram
      schedule: next_optimal_slot  # AI-determined best time
      format: square_side_by_side
    - platform: facebook_feed
      account: practice_facebook
      schedule: +2_hours_after_instagram
      format: landscape
    - platform: digital_storefront
      account: practice_website
      section: implant_gallery
      schedule: immediate
    - platform: content_library
      tag: paid_media_eligible
      schedule: immediate

  testimonial_video:
    - platform: instagram_reels
      account: practice_instagram
      schedule: next_optimal_slot
      format: vertical_15s  # highlight clip
    - platform: youtube_shorts
      account: practice_youtube  # or Dr. Olson pipeline queue
      schedule: queued_for_review
      format: vertical_60s
    - platform: facebook_reels
      account: practice_facebook
      schedule: +1_day
      format: vertical_30s
    - platform: google_business
      account: practice_gbp
      schedule: immediate
      format: video_post

playbook_sga_brand:
  leadership_video:
    - platform: linkedin_personal
      account: speaker_linkedin  # Jensen, Ellingson, etc.
      schedule: tuesday_8am
      format: horizontal_full
    - platform: linkedin_company
      account: sga_dental_linkedin
      schedule: +1_day
      format: clip_60s
    - platform: youtube
      account: sga_dental_youtube
      schedule: thursday
      format: horizontal_full
    - platform: sales_enablement
      tag: bd_outreach
      schedule: immediate
```

**Integration Points**:

| Platform | Integration Method | Notes |
|----------|-------------------|-------|
| Instagram | Meta Graph API | Business account required per practice |
| Facebook | Meta Graph API | Pages API for practice pages |
| YouTube | YouTube Data API v3 | Upload + schedule |
| LinkedIn | LinkedIn Marketing API | Company pages; personal posts via scheduled publishing |
| TikTok | TikTok Content Posting API | Direct upload |
| Google Business | Google Business Profile API | Posts + photos |
| Digital Storefront | Practice Concierge API (internal) | Push to practice websites |
| Email Newsletter | SendGrid / Mailchimp API | Excerpt insertion |
| Sales Enablement | Internal API → CRM (HubSpot/Salesforce) | Tag for BD team use |

**Scheduling Intelligence**:
- Start with platform best-practice defaults (IG: Tue-Thu 11AM-1PM, LinkedIn: Tue-Thu 7-8AM)
- Over time, learn optimal posting times per practice based on engagement data
- Avoid posting conflicts (don't post to same practice's IG twice in 2 hours)

**Tech Stack**:
- Node.js distribution service
- Platform APIs (Meta, YouTube, LinkedIn, TikTok, Google)
- Bull queue (scheduled jobs)
- PostgreSQL (distribution log, scheduling)

### 6. Scorecard & Analytics

**Purpose**: Drive adoption through visibility, gamification, and business impact tracking.

**Champion Scorecard** (in-app):
- Captures this week/month
- Streak (consecutive days with at least 1 capture)
- Content published count
- Top-performing piece (by engagement)
- Network ranking

**Practice Health Score** (HQ dashboard):
- Content captures / week vs. playbook target
- Content quality score (approval rate)
- Channel coverage (are all assigned channels receiving content?)
- Patient consent capture rate
- Trend (improving, stable, declining)

**Network Analytics** (HQ dashboard):
- Total content volume across network
- Content → engagement → conversion attribution
- Before/after impact: practices with active content vs. inactive — patient booking comparison
- ROI: content cost per practice vs. incremental patient revenue attributed
- Brand content metrics: LinkedIn engagement, recruitment inquiries, partnership inquiries

---

## Data Model (Core Entities)

```
Practice
├── id, name, location, playbook_type, pms_system
├── content_champion (User)
├── social_accounts[] (platform, handle, api_credentials)
└── hardware_kit (type, installed_date)

User (Champion / Leader)
├── id, name, role, practice_id (null for HQ/leadership)
├── phone, notification_preference (sms/push)
├── streak_count, total_captures, score
└── playbook_role (champion / doctor / leader)

CaptureAssignment
├── id, champion_id, practice_id
├── patient_id (nullable), procedure_code
├── content_type, playbook, due_date
├── status (pending / captured / expired / skipped)
├── guidance_text, reference_images[]
└── consent_id

Capture
├── id, assignment_id, champion_id, practice_id
├── media_type (photo/video), media_url (S3)
├── metadata (device, location, duration)
├── content_type, procedure_type
├── patient_id, consent_id
└── captured_at

ProcessedAsset
├── id, source_capture_id
├── asset_type (before_after, reel, short, quote_graphic, etc.)
├── platform_format (ig_square, ig_story, fb_landscape, yt_short, etc.)
├── media_url, thumbnail_url
├── ai_caption, ai_hashtags, ai_description
├── review_status (pending / approved / rejected / auto_approved)
├── reviewer_id, reviewed_at
└── quality_score (AI confidence)

Distribution
├── id, processed_asset_id
├── platform, account_id
├── scheduled_at, published_at
├── post_url, post_id (platform-specific)
└── engagement_metrics (views, likes, comments, shares, clicks)

Consent
├── id, patient_id, practice_id
├── consent_type (photo / video / both)
├── signed_at, expires_at
├── signature_url, consent_form_version
└── status (active / expired / revoked)

Playbook
├── id, name, type (implant_cosmetic / specialty / growth_gp / standard_gp / sga_brand)
├── capture_frequency_target (per week)
├── high_value_procedure_codes[]
├── content_types[] (before_after, testimonial, topic_video, etc.)
├── distribution_rules[] (platform, format, schedule)
├── hardware_kit_type
└── guidance_templates[]
```

---

## Infrastructure & Deployment

| Component | Service | Why |
|-----------|---------|-----|
| Mobile App | React Native + Expo | Cross-platform, fast iteration, OTA updates |
| API Backend | Node.js on AWS ECS (Fargate) | Serverless containers, auto-scaling |
| Database | AWS RDS PostgreSQL | Relational data, strong consistency for workflows |
| Media Storage | AWS S3 + CloudFront CDN | Scalable blob storage, global delivery |
| Job Queue | AWS SQS + Lambda | Async processing, decoupled pipeline stages |
| Video Processing | FFmpeg on ECS tasks | Compute-intensive, scales horizontally |
| Photo Processing | Cloudinary | Managed service, dental-specific transformations |
| AI Text Generation | Claude API | Caption, hashtag, description generation |
| AI Transcription | Whisper API | Video → text for captions |
| AI Video Cutting | OpusClip API | Highlight detection, short-form generation |
| SMS/WhatsApp | Twilio | Phase 1 interface, ongoing notifications |
| Push Notifications | Firebase Cloud Messaging | Phase 2+ mobile notifications |
| HQ Dashboard | Next.js on Vercel | Fast web app, real-time updates |
| PMS Integration | Sikka ONE API | 400+ dental PMS systems |
| Monitoring | Datadog | Application + infrastructure monitoring |
| Auth | Auth0 | SSO, role-based access, HIPAA-compliant |

---

## Security & Compliance

| Concern | Approach |
|---------|----------|
| **HIPAA** | All patient data encrypted at rest (AES-256) and in transit (TLS 1.3). BAA with AWS, Cloudinary, and all processors. Patient media stored in HIPAA-compliant S3 buckets. Access logging. |
| **Patient Consent** | Digital consent captured before any media. Consent linked to each capture. Expired/revoked consent blocks distribution. Consent audit trail. |
| **Data Retention** | Raw media retained 7 years (HIPAA). Published content retained indefinitely. Consent records retained 10 years. |
| **Access Control** | Role-based: Champions see only their practice's data. HQ sees all. Leaders see brand content only. Auth0 RBAC. |
| **PHI Screening** | AI screening (AWS Rekognition) checks captures for visible PHI (patient charts, screens with data) before processing. Flagged content held for manual review. |

---

## Phased Delivery

### Phase 1: The SMS Coach (Weeks 1-6)
- Twilio SMS/WhatsApp bot
- Sikka ONE API integration (batch nightly sync, 10-15 pilot practices)
- S3 media upload via MMS/WhatsApp
- Airtable-based review queue (manual, lightweight)
- 2 human editors for production
- Manual distribution via Sprout Social / Later
- **Cost**: ~$15K setup + $5K/month (Twilio + editors + tools)

### Phase 2: The Mobile App + AI Pipeline (Months 2-4)
- React Native app launch
- AI photo processing (Cloudinary)
- AI video processing (FFmpeg + Whisper + OpusClip)
- AI caption generation (Claude API)
- Next.js HQ dashboard with review queue
- Automated distribution engine (platform APIs)
- Content scorecard in-app
- Roll to 40-65 high-value practices
- Brand playbook live for leadership
- **Cost**: ~$80K build + $12K/month (cloud + APIs + processing)

### Phase 3: Full Network + Hardware (Months 4-6)
- Roll to 260+ practices (Playbooks 3-4)
- Testimonial kiosks at top 20 practices
- Passive Insta360 cameras at top 10 implant practices
- AI auto-approval for high-confidence content
- Advanced analytics (content → booking attribution)
- **Cost**: ~$40K build + $60K hardware + $18K/month

### Phase 4: Content Intelligence (Months 6-12)
- ML model: which content types drive bookings (not just views)
- Predictive prompt optimization
- Auto-adjusted posting schedules per practice
- Self-service practice onboarding
- **Cost**: ~$30K build + $20K/month
