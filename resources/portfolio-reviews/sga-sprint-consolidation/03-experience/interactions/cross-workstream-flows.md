# SGA Dental Partners — Cross-Workstream Flows

**Date**: 2026-03-14
**Author**: UX Designer Agent (Phase 6)
**Purpose**: Define user journeys that cross workstream boundaries, fix forced touchpoints, specify transition patterns and data continuity.
**Carry-Forward Items Addressed**: Fix 2 potentially forced cross-workstream touchpoints T-10 and T-16 (Design Director advisory)

---

## 1. Design Principle: Invisible Boundaries

Users do not know workstreams exist. They see their work. Cross-workstream data arrives via the Kafka event bus and is rendered using the same component library and visual patterns as native data. The user never clicks a link that opens a different product surface.

**Two permitted exceptions** (administrative, not daily workflow):
1. Culture OS Admin Console linked from Network Intelligence for H-Score settings management
2. Content Engine Pipeline view linked from Network Intelligence for pipeline diagnostics

Both open in a new browser tab, are used by HQ staff only, and are infrequent administrative actions.

---

## 2. Forced Touchpoint Fixes

### 2.1 T-10: Brand Deviation Flags in QC Queues — FIXED

**Original Problem**: The QC Queue in Network Intelligence shows brand deviation flags from the Content Engine. The original design included a "direct link to the replacement asset in the Asset Registry," which breaks the "no cross-linking" seam principle. An HQ ops person resolving a brand deviation flag needs to understand what "brand deviation" means and how to fix it, but should not be sent to a different application.

**Fix: Embed the replacement inline instead of linking out.**

**Redesigned QC Queue Item for Brand Deviations**:

```
┌──────────────────────────────────────────────────────────────────┐
│  ⚠ BRAND DEVIATION                              Flagged 2d ago  │
│                                                                  │
│  Smile Center Buckhead — Practice Website                        │
│  Issue: Team photo uses non-brand font overlay ("Comic Sans")    │
│                                                                  │
│  ┌────────────────────┐    ┌────────────────────┐               │
│  │  CURRENT            │    │  SUGGESTED FIX      │               │
│  │  [thumbnail of      │    │  [thumbnail of      │               │
│  │   offending asset]  │    │   compliant asset]  │               │
│  │                     │    │                     │               │
│  │  Source: Manual     │    │  Source: Content     │               │
│  │  upload 3/2/2026   │    │  Engine auto-fix    │               │
│  └────────────────────┘    └────────────────────┘               │
│                                                                  │
│  [Preview Full Size]  [Swap to Fix ✓]  [Dismiss]  [Escalate]    │
└──────────────────────────────────────────────────────────────────┘
```

**How it works**:
1. The Content Engine's brand health scoring system flags assets that violate brand guidelines
2. When a brand deviation is detected, the Content Engine checks if a compliant replacement exists in the Asset Registry. If so, it attaches the replacement asset reference to the Kafka event
3. The QC Queue in Network Intelligence renders both the offending asset thumbnail and the suggested replacement thumbnail **inline, side by side**
4. The HQ ops person can see the problem and the fix without leaving Network Intelligence
5. "Preview Full Size" opens a modal within Network Intelligence showing both images at full resolution
6. "Swap to Fix" triggers an API call to the Asset Registry to replace the offending asset with the compliant one. The change cascades through Channel Sync (website, GMB, directories)
7. "Escalate" creates an intervention ticket for the Content Engine team (rare cases where the auto-fix is wrong or no replacement exists)

**Data Flow**:
```
Content Engine (WS2)                          Network Intelligence (WS1)
┌─────────────────────┐                       ┌─────────────────────────┐
│ Brand Health Scoring │                       │ QC Queue                │
│ detects deviation    │──Kafka event──────────▸│ renders inline preview │
│ + generates/finds    │  with both asset IDs   │ of both current and    │
│ compliant replacement│                       │ replacement assets      │
└─────────────────────┘                       │                         │
                                              │ "Swap to Fix" calls     │
                                              │ Asset Registry API      │
                                              │ (no navigation to WS2)  │
                                              └─────────────────────────┘
```

**Why this feels natural**: The HQ ops person is doing QC. They see a problem, they see the fix, they approve the swap. The entire interaction stays within their QC workflow in Network Intelligence. They do not need to understand that the replacement came from a different system.

---

### 2.2 T-16: Brand-Compliant Recognition Templates — FIXED

**Original Problem**: Recognition posts in Culture OS (WS3) auto-apply brand formatting from Content Engine (WS2) templates and assets. The Design Director flagged this as a thin, cosmetic touchpoint that adds a WS2-WS3 engineering dependency for minimal user value.

**Fix: Decouple from real-time Content Engine dependency. Pre-cache brand templates at enrollment.**

**Redesigned Approach**:

Instead of Culture OS querying the Content Engine's Asset Registry at recognition-post-creation time (which creates a runtime dependency), the brand templates and assets are pre-cached in Culture OS during practice enrollment and updated on a weekly schedule.

**Implementation**:

1. **At enrollment**: When a practice is activated (Kafka event `sgadental.ws1.onboarding-status-changed` with `new_status: active`), Culture OS receives the practice's `brand_universe` (SGA Legacy, Gen4, MODIS) from the Practice Data Hub

2. **Weekly sync**: A Temporal workflow runs weekly:
   - Fetches the latest brand template set (colors, fonts, logo URLs, recognition card templates) from the Asset Registry for each brand universe
   - Stores them in Culture OS's local template cache (PostgreSQL table: `ws3.brand_templates`)
   - This is a **batch read**, not a real-time dependency

3. **At recognition-post creation**: When a champion or manager creates a recognition post, Culture OS applies the locally cached brand template. No real-time call to Content Engine

4. **If cache is stale**: If the weekly sync fails, Culture OS uses the last known template. Recognition posts are cosmetically slightly out of date (wrong logo version, for example) but functionally correct. This is acceptable because brand template changes are rare (quarterly at most)

**Why this feels natural**: The user creates a recognition post. It looks on-brand. They did not click anything that felt like it came from another system. The brand compliance is automatic and invisible.

**Engineering Benefit**: Culture OS has zero runtime dependency on Content Engine for recognition posts. The weekly batch sync is a lightweight background job that can fail without breaking any user-facing workflow.

---

## 3. Cross-Workstream User Journeys

### 3.1 Journey: Content Capture to Storefront Publication

**Actor**: Lisa Torres (Office Manager) + HQ Content Team
**Workstreams Crossed**: Content Engine (WS2) --> Practice Operations (WS1)
**Duration**: Hours to days (async pipeline)

```
STEP 1: Capture (Content Engine context, Practice Operations surface)
─────────────────────────────────────────────────────────────────────
Lisa sees Action Card #3 in her Today view: "2 implant cases today."
She taps "Open Camera." The Capture screen opens as a full-screen
overlay within the PWA — she stays in Practice Operations visually.

She takes a before/after photo pair using the overlay guides.
The photo is tagged with practice ID, procedure type, and date.
She taps "Upload." Success toast: "Photo uploaded. Processing..."

    What happens behind the scenes:
    - Photo saved to IndexedDB (offline-safe)
    - On connection: uploaded to S3 via Content Engine upload API
    - Kafka event: sgadental.ws2.capture-media-received
    - Content Engine AI pipeline: color correction, smart crop,
      caption generation (Claude Haiku), variant creation

STEP 2: Review (Content Engine surface, HQ Content Team)
─────────────────────────────────────────────────────────
HQ Content Team member opens the Editorial Queue in the Content
Engine dashboard. Lisa's photo appears in "Ready for Review."

They see: AI-corrected photo, draft captions for Instagram/Facebook/
Google Business, brand compliance check (auto-pass for Gen4 template).

They approve with one edit to the Instagram caption.

    Kafka event: sgadental.ws2.content-asset-approved
    Asset written to Asset Registry with status: approved

STEP 3: Publication (Practice Operations surface, Lisa)
───────────────────────────────────────────────────────
Next time Lisa opens the Storefront section in Practice Operations,
the Asset Gallery shows her new approved photo.

She selects it and taps "Use on Website." The photo appears on
her practice website gallery.

She also taps "Push to GMB." The photo queues for Google Business.

    Lisa's experience: she took a photo, and a few hours later
    it showed up as available for her website. She does not know
    about the Content Engine, the Editorial Queue, or Kafka events.
```

**Data Continuity Across the Boundary**:

| Data Element | Created By | Consumed By | How It Crosses |
|-------------|-----------|------------|----------------|
| Photo file | Practice Ops (Capture screen) | Content Engine | S3 upload API |
| Practice ID | Auth & Identity (JWT) | Content Engine (for routing) | Embedded in upload metadata |
| Procedure type | PMS data (today's appointments) | Content Engine (for caption generation) | Kafka event from PMS service |
| Approved asset | Content Engine | Practice Ops (Asset Gallery) | Kafka event `content-asset-approved` + Asset Registry API read |
| Publication status | Practice Ops (Channel Sync) | Content Engine (Distribution Monitor) | Kafka event `content-asset-published` |

**Transition Pattern**: No navigation between surfaces. Content Engine data appears inside Practice Operations as native content (Asset Gallery). The user's mental model is: "I took a photo, HQ approved it, now I can use it on my website."

---

### 3.2 Journey: H-Score Degradation to Intervention

**Actor**: Dr. Sarah Chen (CEO) + Marcus Williams (VP Ops)
**Workstreams Crossed**: Culture OS (WS3) --> Network Intelligence (WS1) --> Practice Operations (WS1)
**Duration**: Minutes (real-time investigation)

```
STEP 1: Detection (Culture OS generates, Network Intelligence surfaces)
────────────────────────────────────────────────────────────────────────
Culture OS runs the daily H-Score calculation at 6 AM.
Smile Center Buckhead's H-Score drops from 78 to 64.
The Cinnamon phone sub-score is the primary driver.

    Kafka event: sgadental.ws3.hscore-calculated
    Consumed by: AI Analysis Engine (ClickHouse ingestion)

The AI Analysis Engine correlates the H-Score drop with:
- PMS data: show rate dropped 4.2% (from ClickHouse funnel events)
- Staff data: new front desk hire started 2 weeks ago (Practice Data Hub)
- Comms data: call handling quality declined (from Cinnamon integration)

The AI generates an alert: "Southeast region show rate dropped 4.2%
WoW. Root cause: 3 practices had front desk turnover simultaneously."

STEP 2: Investigation (Network Intelligence surface)
──────────────────────────────────────────────────────
Dr. Chen opens the Executive Dashboard. The alert appears in the
AI Alerts Panel. She clicks "View Practices."

She lands on the Practice Deep Dive for Smile Center Buckhead.
She clicks the Culture Tab. She sees:
- H-Score: 64 (down from 78)
- Cinnamon phone sub-score: 58 (primary driver)
- Trust Transfer scores: new hire is missing warm handoffs
- CCA certification: new hire has no certification started

    All of this data comes from Culture OS via Kafka events.
    It renders using the same card/chart components as every
    other tab. The user does not know this is "Culture OS data."

She clicks the Communications Tab. She sees:
- Call handling scores for the new hire are 22% below average
- Confirmation sequence response rates are normal (automation works)

STEP 3: Action (Network Intelligence surface)
─────────────────────────────────────────────
Dr. Chen clicks "Assign Intervention."
She types: "New front desk hire needs accelerated CCA Bronze
onboarding. Regional director should schedule 1:1 coaching."

The intervention appears in the Intervention Queue where Marcus
(VP Ops for Southeast) will see it.

    The intervention note is stored in the Practice Data Hub.
    It does not cross into Culture OS — it is an internal
    note within Network Intelligence.

STEP 4: Resolution (happens outside the platform)
─────────────────────────────────────────────────
Marcus calls the practice. The office manager enrolls the new
hire in CCA Bronze track (via Culture OS Academy).

Over the next 2 weeks:
- Culture OS: new hire completes Bronze modules, trust transfer
  scores improve
- Kafka: sgadental.ws3.training-completed events flow
- AI Analysis Engine: H-Score trend reverses upward
- Network Intelligence: the practice's Culture Tab shows improvement
- The intervention item in the queue is resolved
```

**Data Continuity Across the Boundary**:

| Data Element | Source | Consumer | Crossing Mechanism |
|-------------|--------|----------|-------------------|
| H-Score (composite + sub-scores) | Culture OS | Network Intelligence | Kafka `hscore-calculated` --> ClickHouse --> Dashboard query |
| CCA certification status | Culture OS | Network Intelligence | Kafka `training-completed` --> Practice Data Hub staff record |
| Trust transfer scores | Culture OS (via Cinnamon) | Network Intelligence | Kafka `hscore-calculated` (embedded in component scores) |
| Call handling quality | PMS Integration / Cinnamon | Network Intelligence | ClickHouse call events |
| Show rate | PMS Integration | Network Intelligence | ClickHouse funnel events |
| Staff turnover | Practice Data Hub | AI Analysis Engine | PostgreSQL query |

**Transition Pattern**: The entire investigation happens within Network Intelligence. Culture OS data appears in the Culture Tab of the Practice Deep Dive. The user drills down from network alert to practice detail to specific tab without ever leaving their surface.

---

### 3.3 Journey: Champion Content Mission to Culture Recognition

**Actor**: Maria Santos (Hygienist / Champion)
**Workstreams Crossed**: Culture OS (WS3) + Content Engine (WS2) + Practice Operations (WS1)
**Duration**: Throughout the day

```
STEP 1: Morning Mission (Culture OS context, PWA surface)
─────────────────────────────────────────────────────────
Maria opens the PWA. Her role-filtered nav shows: Today, Capture,
Huddle, Champion Hub (4 screens).

She opens Champion Hub. Her scorecard shows:
- Culture Score: Led 4 huddles this week (streak: 12 weeks)
- Content Score: 6 captures last week (streak: 8 weeks of 5+)
- This week's content mission: "Capture 1 patient testimonial video"
- This week's culture mission: "Focus on trust transfers at checkout"

    Culture Score data: from Culture OS (native)
    Content Score data: from Content Engine via Kafka event
      sgadental.ws2.content-engagement-updated
    Both displayed side by side in the same scorecard card.
    Maria sees one unified champion dashboard.

STEP 2: Content Capture (Content Engine context, PWA surface)
─────────────────────────────────────────────────────────────
Between patients, Maria captures a 30-second testimonial video.
She opens the Capture screen from her bottom nav.

The video is uploaded to the Content Engine.

STEP 3: Huddle Recognition (Culture OS context, PWA surface)
────────────────────────────────────────────────────────────
The next morning huddle display includes a recognition shoutout:
"Maria reached an 8-week content capture streak! 🎯"

This recognition post uses a pre-cached brand template (see
Section 2.2 fix). The template was synced from the Asset Registry
last Sunday. Maria's name and achievement are dynamic; the visual
formatting (colors, fonts, logo) comes from the cached template.

    Maria experiences this as: "I do my champion work, and the
    system notices and celebrates me." She does not distinguish
    between culture work and content work — it is all "being
    a champion."
```

**Data Continuity Across the Boundary**:

| Data Element | Source | Consumer | Crossing Mechanism |
|-------------|--------|----------|-------------------|
| Culture scorecard (huddles led, behaviors) | Culture OS | Champion Hub | Native Culture OS data |
| Content scorecard (captures, quality, streak) | Content Engine | Champion Hub | Kafka `content-engagement-updated` |
| Capture media | PWA Capture screen | Content Engine | S3 upload API |
| Content mission for the week | Content Engine | Champion Hub | Kafka event with weekly brief |
| Culture mission for the week | Culture OS | Champion Hub | Native Culture OS data |
| Recognition achievement | Culture OS | Huddle Display | Native Culture OS trigger |
| Brand template for recognition | Content Engine (Asset Registry) | Culture OS | Weekly batch sync (cached locally) |

**Transition Pattern**: Maria uses 3 PWA screens (Champion Hub, Capture, Huddle). Each screen belongs to a different workstream. But the PWA bottom nav groups them by task, not by system. Maria taps between "Hub" and "Capture" and "Huddle" as naturally as tapping between screens in any mobile app. The workstream boundaries are completely invisible.

---

### 3.4 Journey: Practice Enrollment Triggers Culture Onboarding

**Actor**: Angela Rivera (HQ Operations) + HQ Culture Team
**Workstreams Crossed**: Practice Operations (WS1) --> Culture OS (WS3)
**Duration**: Days (async enrollment pipeline)

```
STEP 1: Practice Activation (Practice Operations / Network Intelligence)
────────────────────────────────────────────────────────────────────────
Angela manages the enrollment pipeline for "Dental Arts Decatur."
In Network Intelligence > Enrollment Pipeline, the practice
progresses through: Harvesting --> Integrating --> Testing --> Live.

On day 14, the practice goes live. The enrollment status changes
to "active."

    Kafka event: sgadental.ws1.onboarding-status-changed
    new_status: "active"
    milestone: "fully_active"

STEP 2: Culture Onboarding Triggered (Culture OS, automatic)
────────────────────────────────────────────────────────────
Culture OS subscribes to the onboarding-status-changed event.
When it receives "fully_active":

1. Creates practice entry in Culture OS with practice profile
   data from Practice Data Hub
2. Assigns default huddle content template based on practice type
3. Generates CCA Bronze enrollment invitations for all staff
   (staff roster from Practice Data Hub)
4. Assigns a culture champion (if office manager indicated one
   during enrollment)
5. Schedules the first morning huddle content delivery

    This is entirely automated. Angela does not need to
    "also set up Culture OS." The enrollment event triggers
    everything downstream.

STEP 3: Visibility (Network Intelligence, Angela)
────────────────────────────────────────────────
Angela checks the Practice Deep Dive for Dental Arts Decatur
a few days later. The Culture Tab shows:
- CCA enrollment: 16 staff invited, 4 started Bronze
- Huddle status: first huddle completed yesterday
- H-Score: "Calculating..." (needs 7 days of data)

    Angela sees culture readiness as part of the practice's
    overall health view. She does not open Culture OS.
```

---

### 3.5 Journey: Content Performance Attribution

**Actor**: VP Marketing + HQ Content Team
**Workstreams Crossed**: Content Engine (WS2) <--> Practice Operations (WS1)
**Duration**: Ongoing (analytics)

```
STEP 1: Content Published (Content Engine surface)
──────────────────────────────────────────────────
A before/after implant photo is published to Instagram for
Smile Center Buckhead via the Content Engine Distribution Monitor.

    Kafka event: sgadental.ws2.content-asset-published
    Includes: asset_id, practice_id, channel, publish_timestamp

STEP 2: Engagement Tracked (Content Engine, automatic)
─────────────────────────────────────────────────────
Every 6 hours, the Content Engine polls Instagram API for
engagement metrics. Likes, comments, saves, reach.

    Kafka event: sgadental.ws2.content-engagement-updated
    Consumed by: ClickHouse (for analytics)

STEP 3: Patient Shows Up (Practice Operations, automatic)
──────────────────────────────────────────────────────────
A new patient books an appointment at Smile Center Buckhead.
During intake, the patient mentions they found the practice
on Instagram (source tracking).

    Kafka event: sgadental.ws1.funnel-event
    source: "organic", source_detail: "instagram"
    attribution.last_content_id: asset_id (if trackable)

STEP 4: Attribution Visible (Content Engine Analytics)
──────────────────────────────────────────────────────
In the Content Engine > Analytics > Attribution section,
the Content Team sees:

"Content published on 3/12 (implant B/A, Instagram) correlated
with 3 new patient inquiries at Smile Center Buckhead within
5 days. Estimated booking value: $13,500."

This attribution data comes from Practice Operations funnel
events flowing back to the Content Engine via ClickHouse.

STEP 5: Also Visible in Network Intelligence
────────────────────────────────────────────
In the Practice Deep Dive > Digital Presence Tab, the VP Ops sees:
"Content capture frequency: 5/week. Brand health: 88.
Last published content: Implant B/A (3/12).
Attribution: 3 inquiries within 5 days."

    Same underlying data, two different surfaces, two different
    audiences. Content Team sees it as "our content is working."
    VP Ops sees it as "this practice's digital presence is healthy."
```

---

## 4. Transition Patterns

### 4.1 How Users Move Between Contexts

Users never navigate between workstream surfaces. Cross-workstream data comes to them. But there are moments where the user's context shifts between workstream domains within a single surface:

| Surface | Context Shift | Transition Pattern |
|---------|--------------|-------------------|
| **PWA** | Bottom nav: Today (WS1) --> Capture (WS2) --> Huddle (WS3) | Standard tab transition (200ms crossfade). No visual indicator of workstream change. All screens use the same component library, same header pattern, same spacing |
| **Network Intelligence** | Practice Deep Dive tabs: Performance (WS1) --> Culture (WS3) --> Content Pipeline (WS2) | Horizontal slide transition (200ms). All tabs use identical chart components, card patterns, and typography. No visual distinction between "native" and "cross-workstream" tabs |
| **Network Intelligence** | QC Queue items: Website QC (WS1) --> Brand Deviation (WS2 data in WS1 surface) | Same list component, same card pattern. Brand deviation cards include inline asset previews (no linking out). Queue is sorted by priority, not by source workstream |
| **Content Engine** | Champions section: Content capture stats (native) alongside Culture champion scores (WS3 data) | Side-by-side scorecard cards in the same panel. Both use the same gauge/metric components. No visual distinction |

### 4.2 Anti-Patterns to Avoid

| Anti-Pattern | Why It Is Wrong | Correct Pattern |
|-------------|-----------------|-----------------|
| Link that opens another surface | Breaks mental model. User does not know workstreams exist | Embed the data inline using shared components |
| "Powered by Content Engine" label | Exposes internal architecture to users | No attribution labels. Data just appears |
| Different visual styling for cross-workstream data | Creates a "foreign widget" feeling | Use identical components (same `@sgadental/ui` library) |
| Loading cross-workstream data only on demand | Creates perceived lag, makes the data feel secondary | Pre-fetch cross-workstream data during initial page load. Show skeleton loaders like any other data |
| Error message mentioning source system | "Culture OS is unavailable" means nothing to users | "Culture data is temporarily unavailable. Showing last known values." |

---

## 5. Data Continuity Rules

### 5.1 What Context Carries Across Boundaries

When data crosses from one workstream to another, these elements must be preserved:

| Context Element | Carries From --> To | How |
|----------------|--------------------|----|
| Practice ID | All workstreams | Kafka event key. All events are partitioned by practice_id |
| Practice name | Practice Data Hub --> all surfaces | Consumed from shared API. Never duplicated |
| Practice type | Practice Data Hub --> all surfaces | Used for benchmarking, playbook selection, template selection |
| Brand universe | Practice Data Hub --> WS2, WS3 | Determines visual templates, brand rules |
| Staff ID | Practice Data Hub --> WS3 (champions), WS2 (champion tracking) | Shared identifier in shared.practice_staff table |
| Timestamps | Event source --> consumers | All events include ISO 8601 timestamps in UTC. UI converts to practice local timezone |
| Scores and metrics | Producing workstream --> consuming surface | Numeric values pass without transformation. Display formatting (decimals, rounding) is applied by the consuming UI |

### 5.2 What Does NOT Cross Boundaries

| Data Element | Stays Within | Why |
|-------------|-------------|-----|
| Patient PII (name, DOB, phone) | WS1 Practice Operations | HIPAA. Only hashed patient IDs cross via events |
| Content Engine editorial comments | WS2 Content Engine | Internal workflow. Not relevant to other surfaces |
| Culture OS assessment answers | WS3 Culture OS | Training details. Only completion status crosses |
| Automation sequence configurations | WS1 Practice Operations | Internal. Other workstreams see results, not configs |
| AI prompt templates | AI Service (shared infra) | Infrastructure. No surface exposes prompt details |

### 5.3 Stale Data Handling Across Boundaries

| Data Type | Expected Freshness | Stale Threshold | Stale Treatment |
|-----------|-------------------|-----------------|-----------------|
| H-Score | Daily (6 AM batch) | >48 hours | "Last updated [date]" caption. No error state. Show last known value |
| Content engagement | 6-hour batch | >24 hours | "Last updated [time]" caption |
| Champion roster | Near-real-time | >24 hours | Use last known roster. New champions will appear on next sync |
| Brand templates (for recognition) | Weekly batch | >14 days | Use last known template. Recognition posts will be slightly off-brand until next sync |
| Training completion | Near-real-time | >24 hours | Show last known certification status. Training progress may be slightly behind |
| Content pipeline metrics | 30-minute cycle | >2 hours | "Last updated [time]" caption in pipeline widget |

**Rule**: Stale data is always better than no data. Show the last known value with a timestamp. Never show an error state for stale cross-workstream data unless the producing service has been unavailable for >7 days.

---

## 6. Event-Driven Architecture for Cross-Workstream Flows

### 6.1 Event Flow Map

```
                    ┌────────────────────────┐
                    │     Practice Data Hub    │
                    │     (Shared Infra)       │
                    └────────┬───────────────┘
                             │
                    practice.created
                    practice.profile-updated
                    practice.staff-updated
                    onboarding-status-changed
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  WS1: Practice │  │  WS2: Content  │  │  WS3: Culture  │
│  Platform       │  │  Engine        │  │  OS            │
│                 │  │                │  │                │
│  Produces:      │  │  Produces:     │  │  Produces:     │
│  • funnel-event │  │  • asset-      │  │  • hscore-     │
│  • comms-msg-   │  │    created     │  │    calculated  │
│    sent         │  │  • asset-      │  │  • champion-   │
│  • onboarding-  │  │    approved    │  │    assigned    │
│    status       │  │  • asset-      │  │  • training-   │
│                 │  │    published   │  │    completed   │
│  Consumes:      │  │  • engagement- │  │  • champion-   │
│  • hscore-calc  │  │    updated     │  │    story-      │
│  • asset-       │  │  • capture-    │  │    submitted   │
│    approved     │  │    media-rcvd  │  │                │
│  • training-    │  │               │  │  Consumes:     │
│    completed    │  │  Consumes:     │  │  • onboarding- │
│  • engagement-  │  │  • champion-   │  │    status      │
│    updated      │  │    assigned    │  │  • funnel-     │
│  • champion-    │  │  • champion-   │  │    event (for  │
│    assigned     │  │    story-      │  │    impact      │
│                 │  │    submitted   │  │    stories)    │
│                 │  │  • appt-       │  │                │
│                 │  │    completed   │  │                │
│                 │  │  • funnel-     │  │                │
│                 │  │    event       │  │                │
└────────────────┘  └────────────────┘  └────────────────┘
```

### 6.2 Cross-Workstream Data Latency Budget

| Flow | Source Event | Consumer | Max Acceptable Latency | User Impact if Violated |
|------|-------------|----------|----------------------|-------------------------|
| H-Score to Executive Dashboard | `hscore-calculated` | Network Intelligence KPI bar | 30 minutes after 6 AM batch | CEO sees yesterday's score instead of today's. Low impact |
| Approved asset to Storefront Gallery | `content-asset-approved` | Practice Ops Asset Gallery | 5 minutes | Office manager does not see newest asset immediately. Can refresh |
| Training completion to Practice Deep Dive | `training-completed` | Network Intelligence Culture Tab | 5 minutes | VP Ops sees slightly stale certification count. Low impact |
| Champion assignment to Content Engine | `champion-assigned` | Content Engine Champions section | 5 minutes | New champion does not receive capture briefs for up to 5 minutes |
| Content engagement to dashboard | `content-engagement-updated` | Network Intelligence + Content Engine Analytics | 6 hours (batch) | Acceptable -- engagement is not real-time |
| Funnel event to content attribution | `funnel-event` | Content Engine Analytics | 1 minute | Attribution accuracy degrades. Medium impact |
| Pipeline health to executive widget | Aggregated from multiple WS2 events | Network Intelligence Pipeline Widget | 30 minutes | Executive sees slightly stale pipeline counts. Low impact |

---

## 7. Error Handling Across Workstream Boundaries

### 7.1 Cross-Workstream Component Failure Modes

| Failure Scenario | User-Visible Effect | Recovery |
|-----------------|---------------------|----------|
| Culture OS service down | H-Score widget in Network Intelligence shows last known value + "Last updated 4 hours ago." Culture Tab in Practice Deep Dive shows same. Practice Ops morning briefing H-Score widget shows cached value | No user action needed. Auto-recovers when service restores. Data backfills from Kafka replay |
| Content Engine service down | Asset Gallery in Practice Ops shows cached thumbnails. Pipeline Health widget in Network Intelligence shows "Temporarily unavailable." Capture uploads queue in PWA IndexedDB | Captures safe locally. Upload resumes on recovery. Gallery refreshes on next successful API call |
| Kafka partition lag | Cross-workstream data is delayed (events queued but not consumed) | Consumer lag monitoring alerts Platform Engineering. User sees slightly stale data. No visible error -- just stale timestamps |
| Practice Data Hub down | All workstreams lose practice profile access. Cached profiles serve reads. No new enrollments possible | Critical incident. All surfaces show cached data. Banner in Network Intelligence: "Some practice data may be outdated." |

### 7.2 Graceful Degradation Hierarchy

When cross-workstream data is unavailable, components degrade in this order:

1. **Show cached data with timestamp** (preferred): "76 (updated 3 hours ago)"
2. **Show skeleton with message** (if no cache): "Culture data loading..."
3. **Show empty state with explanation** (if prolonged): "Culture data is temporarily unavailable. This section will update automatically when restored."
4. **Hide the component** (last resort, only for non-essential widgets): The Content Pipeline Health widget in Network Intelligence can be hidden entirely if Content Engine is down for >24 hours, as it is advisory, not critical

Components never show technical error messages ("Error 500", "Kafka consumer lag", "WS3 service unavailable") to users.

---

*This specification defines how user experiences cross workstream boundaries without the user knowing those boundaries exist. The two forced touchpoints (T-10 and T-16) are fixed: T-10 now embeds replacement assets inline instead of linking out, and T-16 decouples from real-time Content Engine dependency via weekly batch sync. All cross-workstream data flows through Kafka events and renders using the shared `@sgadental/ui` component library.*
