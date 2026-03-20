# WS1: Practice Platform — PHI-Free User Journey

**Journey Title**: "Practice Enrollment to Executive Network Visibility"
**Date**: 2026-03-20
**Author**: UX Designer Agent (Phase A Journey Specification)
**Phase**: A (PHI-Free)
**Interfaces Covered**: NI-1, NI-2, NI-3, NI-4, NI-5, NI-7, PO-1, PO-6, PO-8, PO-9
**Companion Documents**: [PHI Scope Analysis](../01-scope/phi-scope-analysis.md) | [Experience Architecture](../02-experience/phi-free-experience-architecture.md) | [Original Dashboard Specs](../../03-experience/interactions/unified-dashboard-specs.md) | [Original Practice Interface Specs](../../03-experience/interactions/practice-interface-specs.md)

---

## Actors

| Actor | Role | Primary Surface | Phase A Access |
|-------|------|----------------|----------------|
| Angela Rivera | HQ Operations Specialist | Network Intelligence (Desktop) | Enrollment Pipeline, HQ Standup Board, QC Queues, Practice Deep Dive |
| Dr. Sarah Chen | CEO / Executive | Network Intelligence (Desktop) | Executive Dashboard, Portfolio Heatmap, Practice Deep Dive, Network Learning Report |
| Marcus Williams | VP Operations / Regional Director | Network Intelligence (Desktop) | Intervention Queue, Portfolio Heatmap, Practice Deep Dive, Standup Board |
| Lisa Torres | Office Manager | Practice Operations (PWA + Web) | Today, Storefront, Capture, Huddle, Playbooks |

---

## Flow 1: New Practice Enrollment — Angela Enrolls "Dental Arts Decatur"

### Step 1.1: Enrollment Pipeline — Harvesting Stage

**Time**: Monday morning, 8:30 AM ET

Angela opens Network Intelligence on her desktop. She navigates to **Operations > Enrollment Pipeline**. The pipeline shows all practices in onboarding, organized as a horizontal Kanban board.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Enrollment Pipeline                                    [+ New Practice] │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  HARVESTING (3)     INTEGRATING (2)    TESTING (1)      LIVE (42)       │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                 │
│  │ Dental Arts   │   │ Smile Plus   │   │ Metro Dental │                 │
│  │ Decatur       │   │ Roswell      │   │ Marietta     │                 │
│  │ Day 2 of 14   │   │ Day 8 of 14  │   │ Day 12 of 14│                 │
│  │ ████░░░░░░░░ │   │ ████████░░░░ │   │ ██████████░░ │                 │
│  │ 14% complete  │   │ 57% complete │   │ 86% complete │                 │
│  │               │   │              │   │              │                 │
│  │ Crawl: Done   │   │ PMS: ✓       │   │ Website: ✓   │                 │
│  │ GMB: Done     │   │ Cinnamon: ✓  │   │ GMB: ✓       │                 │
│  │ Directors: ░░ │   │ Swell: ░░    │   │ Phones: ✓    │                 │
│  │ Photos: ░░    │   │ DI: ✓        │   │ Blitz: Sched │                 │
│  │               │   │ iSolved: ░░  │   │              │                 │
│  │ [View Detail] │   │ [View Detail]│   │ [View Detail]│                 │
│  └──────────────┘   └──────────────┘   └──────────────┘                 │
│  ┌──────────────┐   ┌──────────────┐                                     │
│  │ Bright Smiles │   │ Family Dental│                                     │
│  │ Alpharetta    │   │ Sandy Springs│                                     │
│  │ Day 5 of 14   │   │ Day 10 of 14│                                     │
│  └──────────────┘   └──────────────┘                                     │
└──────────────────────────────────────────────────────────────────────────┘
```

Angela clicks **"+ New Practice"** to begin enrolling Dental Arts Decatur.

**What she sees**: A multi-step enrollment form opens as a full-width panel.

**Step 1 of 4 — Practice Profile:**
- Practice name, address, phone, website URL
- Practice type (GP, Specialty, Pedo, Ortho)
- Brand universe assignment (SGA Legacy, Gen4, MODIS)
- Number of providers, operatories, front desk staff
- Current PMS system (Dentrix, Eaglesoft, Open Dental, etc.)
- Designated office manager contact

**What happens behind the scenes:**
- `POST /api/enrollment/practices` creates a new enrollment record
- Practice Data Hub generates a `practice_id` (e.g., `sga-0261`)
- Kafka event: `sgadental.ws1.enrollment-created` with practice metadata
- Silent data harvesting begins automatically:
  - Web crawler fetches the existing practice website for brand asset extraction
  - GMB scraper pulls current Google Business Profile data (hours, photos, reviews)
  - Public directory scanner checks Healthgrades, Zocdoc, Yelp listings

**Data sources**: All data at this stage is entered by Angela or scraped from public sources. Zero PHI.

**Actions Angela can take**: Save draft (return later), submit and advance to Integrating, assign to a colleague.

### Step 1.2: Enrollment Pipeline — Integrating Stage

**Time**: Day 3-8

The practice card moves to the **Integrating** column. Angela clicks into the practice detail view.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Dental Arts Decatur — Integration Checklist           Day 5 of 14      │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  VENDOR CONNECTIONS                                                      │
│  ✓ Cinnamon (phone scoring)          Connected 3/17                     │
│  ✓ Swell (review management)         Connected 3/18                     │
│  ✓ Dental Intel (operational data)   Connected 3/18                     │
│  ○ iSolved (HR/payroll)              Pending — credentials requested    │
│                                                                          │
│  PMS CONNECTOR (Phase A: Health Check Only)                              │
│  ○ NexHealth / Sikka ONE             Credentials received — testing     │
│  ⓘ PMS integration will sync in Phase B. Phase A verifies connectivity. │
│                                                                          │
│  BRAND ASSETS                                                            │
│  ✓ Logo extracted from existing site                                    │
│  ✓ Provider headshots — 3 of 4 received                                │
│  ○ Office photos — scheduled for 3/20                                   │
│                                                                          │
│  [Send Reminder to Practice]  [Update Status]  [Add Note]               │
└──────────────────────────────────────────────────────────────────────────┘
```

**What Angela sees**: A checklist of integrations with real-time status indicators. Each vendor connection shows a green checkmark (connected), amber circle (pending), or red alert (failed).

**What happens behind the scenes:**
- Integration Middleware runs connectivity health checks every 15 minutes
- Kafka events: `sgadental.ws1.integration-connectivity-checked` per vendor
- Staff roster begins populating from iSolved once connected
- Cinnamon, Swell, and Dental Intel begin collecting baseline scores (these will become the practice's first H-Score after 7 days of data)

**What's explicitly MISSING compared to original:**
- PMS data sync is not active. In the original design, Integrating stage would begin syncing patient records. In Phase A, the PMS connector is tested for connectivity only — verified it can authenticate and reach the API — but no patient data flows.
- The integration checklist shows an info banner: "PMS integration will sync in Phase B. Phase A verifies connectivity."

### Step 1.3: Enrollment Pipeline — Testing Stage

**Time**: Day 9-12

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Dental Arts Decatur — Testing Checklist               Day 11 of 14     │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  WEBSITE                                                                 │
│  ✓ Template applied (Gen4 Dental Standard)                              │
│  ✓ Content populated (practice profile, providers, hours)               │
│  ✓ Brand assets deployed (logo, provider photos, office photos)         │
│  ✓ CDN deployed (CloudFront edge cached)                                │
│  ✓ Mobile responsive check passed                                       │
│  ○ QC review pending (in QC Queue)                                      │
│                                                                          │
│  GOOGLE BUSINESS PROFILE                                                 │
│  ✓ Profile claimed / verified                                           │
│  ✓ Photos uploaded (6 of 6)                                             │
│  ✓ Hours synchronized                                                   │
│  ○ Category optimization pending                                        │
│                                                                          │
│  VENDOR SCORES (Baseline)                                                │
│  Cinnamon phone score: 68 (7-day baseline)                              │
│  Swell review score: 4.1 stars (142 reviews)                            │
│  Dental Intel score: 72                                                  │
│  Projected H-Score: 67                                                   │
│                                                                          │
│  CULTURE OS READINESS                                                    │
│  ✓ Practice created in Culture OS (auto-triggered)                      │
│  ✓ Staff roster synced (16 team members)                                │
│  ✓ Default huddle template assigned (GP Standard)                       │
│  ○ Champion designation — awaiting office manager confirmation          │
│  ○ CCA Bronze invitations — sent, 0 started                            │
│                                                                          │
│  [Schedule Blitz Deploy]  [Approve for Go-Live]                         │
└──────────────────────────────────────────────────────────────────────────┘
```

**What happens behind the scenes:**
- Website generation pipeline: Template Engine selects the Gen4 template, populates with practice data from Practice Data Hub, deploys to CloudFront CDN
- Kafka event: `sgadental.ws1.onboarding-status-changed` with `new_status: testing`
- Culture OS receives the event and automatically creates the practice entry, assigns default huddle content, and generates CCA Bronze enrollment invitations for all 16 staff members
- Content Engine receives the event and adds the practice to the content champion assignment pool

### Step 1.4: Enrollment Pipeline — Go-Live

**Time**: Day 14

Angela clicks **"Approve for Go-Live."** A confirmation dialog appears:

```
┌────────────────────────────────────────────────┐
│  Approve Go-Live: Dental Arts Decatur?         │
│                                                │
│  Checklist status: 18 of 20 items complete     │
│  ⚠ 2 items incomplete:                        │
│    - Champion designation (non-blocking)        │
│    - CCA Bronze enrollment (non-blocking)       │
│                                                │
│  Go-live will:                                 │
│  • Activate the practice website               │
│  • Publish GMB profile updates                 │
│  • Begin H-Score tracking                      │
│  • Activate morning huddle delivery            │
│  • Add practice to Executive Dashboard         │
│                                                │
│  [Cancel]  [Approve Go-Live]                   │
└────────────────────────────────────────────────┘
```

**What happens behind the scenes:**
- Kafka event: `sgadental.ws1.onboarding-status-changed` with `new_status: active`, `milestone: fully_active`
- Practice status changes to "Live" across all systems
- Practice Data Hub updates the active practice count (network KPI)
- H-Score tracking begins (first official score calculated at 6 AM the following day)
- Practice appears on the Portfolio Heatmap with a "New" badge for 14 days
- Morning huddle content delivery activates for the next business day
- Content Engine activates capture brief delivery for the practice

---

## Flow 2: Storefront Generation — Website + GMB + CDN

### Step 2.1: Website Template Selection

During the Testing stage (Step 1.3), the website generation is largely automated. Angela can review and override defaults.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Website Configuration — Dental Arts Decatur                             │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  TEMPLATE: Gen4 Dental Standard                   [Change Template ▾]   │
│                                                                          │
│  ┌─────────────────────────────────┐  ┌──────────────────────────────┐  │
│  │                                 │  │  BRAND CONFIGURATION          │  │
│  │    [Live Preview]               │  │                               │  │
│  │    Desktop / Mobile toggle      │  │  Primary Color: #2563EB       │  │
│  │                                 │  │  Secondary Color: #1E40AF     │  │
│  │    Dental Arts Decatur          │  │  Logo: ✓ uploaded             │  │
│  │    "Your smile, our passion"    │  │  Tagline: [editable]          │  │
│  │                                 │  │  Hours: Mon-Fri 8-5, Sat 9-1 │  │
│  │    [Provider Grid]              │  │                               │  │
│  │    [Services List]              │  │  PROVIDERS                    │  │
│  │    [Gallery]                    │  │  Dr. Patel — headshot ✓       │  │
│  │    [Contact / Map]              │  │  Dr. Kim — headshot ✓         │  │
│  │                                 │  │  Dr. Rodriguez — headshot ✓   │  │
│  │                                 │  │  Dr. Nguyen — headshot ○      │  │
│  └─────────────────────────────────┘  └──────────────────────────────┘  │
│                                                                          │
│  [Preview Full Site]  [Deploy to CDN]  [Schedule GMB Sync]              │
└──────────────────────────────────────────────────────────────────────────┘
```

**Data sources**: Practice Data Hub (profile, providers, hours), Asset Registry (logos, headshots, office photos), Channel Sync Engine (deployment).

**Actions**: Angela can edit the tagline, swap template variants, reorder provider display, select which photos appear in the gallery, and trigger CDN deployment.

**What happens behind the scenes on "Deploy to CDN":**
1. Template Engine renders final HTML/CSS/JS from template + practice data
2. Static assets (images, fonts) packaged and uploaded to S3
3. CloudFront distribution created/updated with the new site bundle
4. SSL certificate provisioned via ACM (if custom domain)
5. DNS record updated (if managed domain) or CNAME instructions generated
6. Kafka event: `sgadental.ws1.storefront-deployed`

### Step 2.2: GMB Optimization

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Google Business Profile — Dental Arts Decatur                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  STATUS: Verified ✓     Last sync: 2 hours ago                          │
│                                                                          │
│  Business Name: Dental Arts Decatur                    ✓ Matches        │
│  Address: 285 Church St, Decatur, GA 30030             ✓ Matches        │
│  Phone: (404) 555-0192                                 ✓ Matches        │
│  Website: dentalartsdecatur.com                        ✓ Linked         │
│  Hours: Mon-Fri 8-5, Sat 9-1                          ✓ Synced         │
│  Categories: Dentist, Cosmetic Dentist                 ⚠ Recommend add  │
│              "Emergency Dental Service"                                  │
│  Photos: 6 uploaded                                    ✓ Synced         │
│  Reviews: 142 (4.1 avg)                               Via Swell         │
│                                                                          │
│  [Sync Now]  [Edit Categories]  [Upload More Photos]                    │
└──────────────────────────────────────────────────────────────────────────┘
```

**What's explicitly MISSING compared to original:**
- No "Booking Link" integration with PMS. In the original design, the GMB profile would include a direct booking button linked to NexHealth's hosted booking page. In Phase A, the booking button links to a simple contact form or phone number instead. Phase B adds the NexHealth booking widget.
- No review request automation. Swell collects reviews independently, but the platform does not trigger post-visit review requests (that requires appointment completion data from PMS).

---

## Flow 3: Executive Dashboard Daily Use — Dr. Sarah Chen

### Step 3.1: Morning Dashboard Review

**Time**: 7:15 AM ET, Dr. Chen opens Network Intelligence on her laptop.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  SGA Dental Partners | Network Intelligence        Dr. Chen ▾  [?]  🔔 │
├──────────────────────────────────────────────────────────────────────────┤
│  │                                                                       │
│  │ Network    ┌──────────────────────────────────────────────────────┐   │
│  │ Overview   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │   │
│  │            │  │  43     │ │  74     │ │  76     │ │   5     │  │   │
│  │ Regions    │  │ Active  │ │ Avg     │ │ Avg     │ │ Enrolling│ │   │
│  │            │  │ Prac-   │ │ H-Score │ │ Brand   │ │ Practices│ │   │
│  │ Practices  │  │ tices   │ │         │ │ Health  │ │         │  │   │
│  │            │  │ ▲1 new  │ │ ▲2 WoW │ │ ▼1 WoW │ │ 18d avg │  │   │
│  │ Operations │  │ ── ──  │ │ ── ──  │ │ ── ──  │ │ ── ──  │  │   │
│  │            │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │   │
│  │ Compliance │  ┌─────────┐                                       │   │
│  │ Readiness  │  │  287    │  vs. Target | vs. Prior Mo | vs. Prior Yr │
│  │            │  │ Content │                                       │   │
│  │ Settings   │  │ Captures│                                       │   │
│  │            │  │ MTD     │                                       │   │
│  │            │  │ ▲12%   │                                       │   │
│  │            │  └─────────┘                                       │   │
│  │            ├──────────────────────────────────────────────────────┤   │
│  │            │  ┌─────────────────┐ ┌────────────────────────────┐ │   │
│  │            │  │ AI ALERTS (3)   │ │ PORTFOLIO HEATMAP          │ │   │
│  │            │  │                 │ │ Color by: [H-Score ▾]      │ │   │
│  │            │  │ ● HIGH          │ │                            │ │   │
│  │            │  │ Southeast H-    │ │    [Map of GA with         │ │   │
│  │            │  │ Score dropped   │ │     practice markers       │ │   │
│  │            │  │ 6 pts WoW.     │ │     colored by H-Score]    │ │   │
│  │            │  │ Root cause: 3   │ │                            │ │   │
│  │            │  │ practices no    │ │  ● High (>80)  43%        │ │   │
│  │            │  │ huddles this wk │ │  ● Moderate    41%        │ │   │
│  │            │  │ [View] [Dismiss]│ │  ● Low (<60)   16%        │ │   │
│  │            │  │                 │ │                            │ │   │
│  │            │  │ ▲ MEDIUM       │ │                            │ │   │
│  │            │  │ 5 champions     │ │                            │ │   │
│  │            │  │ inactive >14d.  │ │                            │ │   │
│  │            │  │ Same region.    │ │                            │ │   │
│  │            │  │ [View] [Dismiss]│ │                            │ │   │
│  │            │  │                 │ │                            │ │   │
│  │            │  │ ⓘ ADVISORY     │ │                            │ │   │
│  │            │  │ Phone script    │ │                            │ │   │
│  │            │  │ v2.4 — 28 prac │ │                            │ │   │
│  │            │  │ not accessed.   │ │                            │ │   │
│  │            │  │ [View] [Dismiss]│ │                            │ │   │
│  │            │  └─────────────────┘ └────────────────────────────┘ │   │
│  │            ├──────────────────────────────────────────────────────┤   │
│  │            │  Trend Charts    [H-Score ▾]  [30d | 60d | 90d]    │   │
│  │            │  ── network trend line with regional overlays ──   │   │
│  │            ├──────────────────────────────────────────────────────┤   │
│  │            │  Content Pipeline Health              [Last 7 days ▾]│  │
│  │            │  ┌──────┐──▸┌──────┐──▸┌──────┐──▸┌──────┐──▸┌────┐│  │
│  │            │  │  47  │   │  38  │   │  12  │   │  31  │   │ 28 ││  │
│  │            │  │Captured│ │Process│  │Review│   │Approved│  │Pub ││  │
│  │            │  │ ▲12% │   │ ●OK  │   │▲3d avg│  │ ●OK  │   │▲8% ││  │
│  │            │  └──────┘   └──────┘   └──────┘   └──────┘   └────┘│  │
│  │            └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
```

**Phase A KPI Cards (5 total):**

| Position | Metric | Value | Source | What it replaces from original |
|----------|--------|-------|--------|-------------------------------|
| 1 | Active Practices | 43 | Practice Data Hub (enrollment status) | Replaces "Total Production ($4.2M)" |
| 2 | Avg Network H-Score | 74 | Culture OS via Kafka | Retained from original (already PHI-free) |
| 3 | Avg Brand Health Score | 76 | Content Engine via Kafka | Replaces "Show Rate (92%)" |
| 4 | Enrolling Practices | 5 | Practice Data Hub (enrollment records) | Replaces "Treatment Acceptance (68%)" |
| 5 | Content Captures MTD | 287 | Content Engine via Kafka | Replaces "New Patients (412)" |

**What's explicitly MISSING compared to original:**
- **Total Production, New Patients, Show Rate, Treatment Acceptance** KPIs are all absent. These require PMS patient funnel data flowing through ClickHouse.
- **Revenue impact ranking** on AI Alerts is replaced with "Culture Impact Score."
- **Funnel-based heatmap layers** (Composite Health, Funnel Performance) are unavailable. The layer selector shows 5 options instead of 9.

**How the UI handles the absence:** No empty slots, no "coming soon" badges on the KPI bar. The 5 Phase A KPIs fill the bar completely. The UI is fully designed for 5 cards. When Phase B activates, the grid expands to 7-8 cards with staggered animation.

### Step 3.2: AI Alerts Investigation

Dr. Chen clicks **"View Practices"** on the HIGH alert about Southeast H-Score dropping.

**What she sees**: The Portfolio Heatmap zooms to the Southeast region, and 3 practice markers flash with `status-critical` red borders. A filter chip appears: "Alert: Southeast H-Score Drop."

She clicks the marker for **Smile Center Buckhead**.

### Step 3.3: Practice Deep Dive — Smile Center Buckhead

A side panel slides in from the right (480px width) showing the Practice Deep Dive summary. She clicks "Open Full View" to see the 4-tab detail.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  ← Network > Southeast > Smile Center Buckhead                          │
├──────────────────────────────────────────────────────────────────────────┤
│  [ Profile ]  [ Culture ]  [ Content & Brand ]  [ History ]             │
├──────────────────────────────────────────────────────────────────────────┤
```

**Tab: Profile**
```
│  PRACTICE PROFILE                                                       │
│  Name: Smile Center Buckhead                                            │
│  Address: 3280 Peachtree Rd NE, Atlanta, GA 30305                      │
│  Type: General Practice | Brand: Gen4                                   │
│  Acquired: 2024-06-15 | Live since: 2024-07-02                         │
│  PMS: Dentrix | Providers: 3 | Operatories: 6 | Staff: 16             │
│                                                                         │
│  STAFF ROSTER (from iSolved)                                            │
│  ┌────────────┬───────────────┬──────────┬────────────────┐             │
│  │ Name       │ Role          │ CCA Tier │ Start Date     │             │
│  │ Dr. Patel  │ Lead Dentist  │ Gold     │ 2024-06-15     │             │
│  │ Maria S.   │ Hygienist/Chmp│ Silver   │ 2024-06-15     │             │
│  │ Lisa T.    │ Office Mgr    │ Bronze   │ 2024-06-15     │             │
│  │ Jamie L.   │ Front Desk    │ None     │ 2026-03-03     │ ← NEW HIRE │
│  │ ...12 more │               │          │                │             │
│  └────────────┴───────────────┴──────────┴────────────────┘             │
│                                                                         │
│  OFFICE MANAGER CONTACT                                                 │
│  Lisa Torres | lisa.torres@smilecenter.com | (404) 555-0134            │
```

**Tab: Culture** (Dr. Chen clicks this tab to investigate the alert)
```
│  CULTURE OVERVIEW                                                       │
│                                                                         │
│  H-SCORE: 64            ▼14 from 78 (2 weeks ago)                     │
│                                                                         │
│  ┌── 30-day sparkline showing steep decline ──┐                        │
│                                                                         │
│  SUB-SCORES                                                             │
│  Cinnamon Phone (35%)   ████████████░░░░░░░░  58  ▼20                  │
│  Swell Reviews (45%)    ██████████████░░░░░░  71  ▼3                   │
│  Dental Intel (20%)     ██████████████░░░░░░  74  ▼2                   │
│                                                                         │
│  PRIMARY DRIVER: Cinnamon phone score down 20 pts                      │
│  AI Analysis: "New front desk hire (Jamie L., started 3/3)             │
│  has call handling scores 22% below practice average.                  │
│  Trust transfer warm handoff scores are missing."                      │
│                                                                         │
│  CCA CERTIFICATION STATUS                                               │
│  Gold: 1 (Dr. Patel) | Silver: 3 | Bronze: 8 | None: 4               │
│  ⚠ Jamie L. — no certification started (17 days since hire)           │
│                                                                         │
│  HUDDLE COMPLETION (Last 14 days)                                       │
│  ┌─ M T W T F M T W T F M T W T ─┐                                    │
│  │  ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✗ ✗ ✓ ✗ ✗ ✗ ✓ │                                    │
│  └────────────────────────────────┘                                    │
│  Completion rate: 64% (9/14) — ▼ from 93% prior period                │
│                                                                         │
│  TRUST TRANSFER SCORES (Practice Aggregate)                             │
│  Network avg: 78 | This practice: 61 ▼12                              │
│  Breakdown by staff: Dr. Patel 91, Maria 85, Lisa 74, Jamie 34        │
│                                                                         │
│  CHAMPION ACTIVITY                                                      │
│  Maria Santos — Last culture mission: 3/18 (2 days ago) ✓              │
│  12-week huddle streak | 8-week content capture streak                 │
│                                                                         │
│  [Assign Intervention]                                                  │
```

**Data sources for this tab:**
- H-Score composite + sub-scores: Culture OS via Kafka `sgadental.ws3.hscore-calculated`
- CCA certification: Culture OS via Kafka `sgadental.ws3.training-completed`
- Huddle completion: Culture OS via Kafka `sgadental.ws3.huddle-completed`
- Trust transfer scores: Cinnamon aggregate scores (per staff member)
- Staff roster + start dates: iSolved via Practice Data Hub
- Champion activity: Culture OS + Content Engine via Kafka

**What's explicitly MISSING compared to original:**
- **Performance Tab**: The entire 5-stage funnel visualization (Lead, Connected, Booked, Shows, Production), conversion rates, revenue trend, and cohort benchmarks are absent. This tab does not appear in the navigation.
- **Communications Tab**: Message volume, response rates, automation performance, Voice AI usage, and channel mix are absent. This tab does not appear in the navigation.
- **Within the Culture Tab**: Individual call transcript playback is not available. Trust transfer scores are shown as aggregate per staff member, but the "Listen to call" drill-down is grayed out with tooltip: "Individual call analysis available in Phase B."

**How the UI handles the absence:** The tab bar shows exactly 4 tabs. There are no grayed-out "Performance" or "Communications" tabs. Users who have never seen the original design experience a complete 4-tab interface. When Phase B activates, Performance and Communications tabs are injected at positions 1 and 2 with a "NEW" badge.

**Tab: Content & Brand**
```
│  BRAND HEALTH SCORE                                                     │
│  ┌────────────────┐                                                    │
│  │  78 / 100      │  ▼2 from last month                               │
│  │  ◐ Gauge       │                                                    │
│  └────────────────┘                                                    │
│                                                                         │
│  Website: 82 | GMB: 75 | Directories: 71 | Social: 84                 │
│                                                                         │
│  CONTENT PIPELINE (This Practice)                                       │
│  Captured: 4 | Processing: 1 | In Review: 0 | Published: 8 (MTD)      │
│  Capture frequency: 3.2/week (network avg: 2.8)                       │
│                                                                         │
│  RECENT PUBLISHED ASSETS                                                │
│  [Team photo 3/18] [Office tour 3/15] [Provider intro 3/12]           │
│                                                                         │
│  CHAMPION CONTENT SCORECARD                                             │
│  Maria Santos: 6 captures last week | 8-week streak                   │
│                                                                         │
│  ⚠ Provider photo for Dr. Nguyen is 8 months old                      │
```

**Tab: History**
```
│  ENROLLMENT TIMELINE                                                    │
│  2024-06-15 — Acquisition completed                                    │
│  2024-06-18 — Enrollment started (Harvesting)                          │
│  2024-06-25 — Integrations connected                                   │
│  2024-07-02 — Go-Live                                                  │
│                                                                         │
│  KEY EVENTS                                                             │
│  2026-03-03 — New front desk hire: Jamie L.                            │
│  2026-03-10 — Playbook v2.4 deployed                                   │
│  2026-03-15 — H-Score alert triggered (dropped below 70)              │
│                                                                         │
│  INTERVENTION HISTORY                                                   │
│  2025-11-08 — "Brand deviation on website" — Resolved (Angela R.)     │
│  2026-03-15 — "H-Score degradation" — OPEN (Unassigned)               │
│                                                                         │
│  VP OPS NOTES                                                           │
│  (none for this practice)                                              │
```

### Step 3.4: Assigning an Intervention

Dr. Chen clicks **"Assign Intervention"** from the Culture Tab.

```
┌────────────────────────────────────────────────────────────────┐
│  Create Intervention — Smile Center Buckhead                    │
│                                                                │
│  Issue Type: [H-Score Degradation ▾]                           │
│  Priority: [High ▾]                                            │
│  Assign to: [Marcus Williams (VP Ops, Southeast) ▾]            │
│                                                                │
│  Note:                                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ New front desk hire Jamie L. needs accelerated CCA       │  │
│  │ Bronze onboarding. Regional director should schedule     │  │
│  │ 1:1 coaching. Also investigate why huddle completion     │  │
│  │ dropped — may be related to the new hire adjusting.      │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                │
│  [Cancel]  [Create Intervention]                               │
└────────────────────────────────────────────────────────────────┘
```

**What happens behind the scenes:**
- Intervention record created in Practice Data Hub
- Kafka event: `sgadental.ws1.intervention-created`
- Marcus Williams receives a notification (in-app + email)
- The intervention appears in the Intervention Queue (NI-3) and in Marcus's Standup Board

---

## Flow 4: HQ Ops Standup Board — Angela's Daily Standup

### Step 4.1: Morning Standup

**Time**: 9:00 AM ET, Angela opens the HQ Standup Board.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  HQ Ops Standup Board                           Thursday, March 20       │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  TOP 14 PRACTICES NEEDING ATTENTION (Culture/Brand/Enrollment Ranked)   │
│  ┌────┬─────────────────────┬──────────┬──────────────┬───────────────┐ │
│  │ #  │ Practice            │ Issue    │ Culture Risk │ Days Flagged  │ │
│  ├────┼─────────────────────┼──────────┼──────────────┼───────────────┤ │
│  │ 1  │ Smile Ctr Buckhead  │ H-Score  │ HIGH         │ 5             │ │
│  │ 2  │ Metro Dental Midtown│ Brand    │ HIGH         │ 3             │ │
│  │ 3  │ Bright Smiles Decatu│ Enrollment│ MEDIUM      │ 8             │ │
│  │ ...│                     │          │              │               │ │
│  └────┴─────────────────────┴──────────┴──────────────┴───────────────┘ │
│                                                                          │
│  ENROLLMENT PIPELINE SUMMARY                                             │
│  Harvesting: 3 | Integrating: 2 | Testing: 1 | Avg time to live: 18d  │
│  ⚠ Bright Smiles Decatur stalled at Integrating for 8 days            │
│                                                                          │
│  QC QUEUES                                                               │
│  Website QC: 4 pending | Brand Deviations: 2 pending                   │
│  [Open Website QC]  [Open Brand Violations]                             │
│                                                                          │
│  BATCH OPERATIONS                                                        │
│  [Deploy Playbook Update]  [Batch Storefront Refresh]                   │
│  [Send Champion Reminders]                                               │
│                                                                          │
│  TODAY'S COMPLETED                                                       │
│  ✓ 2 website QC reviews completed                                      │
│  ✓ 1 enrollment advanced to Testing                                    │
│  ✓ Playbook v2.4 deployed to 15 practices                             │
└──────────────────────────────────────────────────────────────────────────┘
```

**What's explicitly MISSING compared to original:**
- **Revenue-impact ranking**: The original Standup Board ranked practices by estimated weekly revenue impact. Phase A uses "Culture Risk Score" derived from H-Score trajectory + huddle compliance + certification gaps + brand health.
- **Patient-data batch operations**: Original included batch patient outreach, automation resets, and call routing changes. Phase A retains only storefront and playbook batch operations.

**Data sources:**
- Top 14 ranking: AI Analysis Engine (non-PHI signals from Culture OS + Content Engine + Enrollment Pipeline)
- Enrollment pipeline: Practice Data Hub
- QC queues: Content Engine (brand deviations) + Channel Sync Engine (website QC)
- Batch operations: Playbook Engine + Channel Sync Engine

---

## Flow 5: Intervention Queue — Marcus Williams (VP Ops)

### Step 5.1: Culture-Risk-Based Interventions

**Time**: 9:30 AM, Marcus opens the Intervention Queue.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Intervention Queue                    [Region: Southeast ▾]  [All ▾]   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  #1  Smile Center Buckhead              Southeast | GP | Active    │  │
│  │                                                                    │  │
│  │  Primary Issue: H-Score dropped 14 points in 2 weeks              │  │
│  │  Root Cause (AI): Cinnamon phone scores down 20% — new front      │  │
│  │  desk hire (Jamie L.), no CCA Bronze started. Huddles missed      │  │
│  │  5 of last 14 days.                                                │  │
│  │                                                                    │  │
│  │  Culture Risk: HIGH    Days Flagged: 5                             │  │
│  │  Assigned: Marcus Williams (by Dr. Chen, 7:32 AM)                 │  │
│  │                                                                    │  │
│  │  Note from Dr. Chen: "New front desk hire Jamie L. needs          │  │
│  │  accelerated CCA Bronze onboarding. Schedule 1:1 coaching."       │  │
│  │                                                                    │  │
│  │  [View Practice]  [Mark In Progress]  [Resolve with Note]          │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  #2  Family Dental Sandy Springs        Southeast | GP | Active    │  │
│  │                                                                    │  │
│  │  Primary Issue: Content champion inactive for 21 days             │  │
│  │  Root Cause (AI): Champion (Sarah M.) on leave per iSolved data.  │  │
│  │  No replacement champion assigned. Content capture frequency at 0.│  │
│  │                                                                    │  │
│  │  Culture Risk: MEDIUM    Days Flagged: 21                          │  │
│  │  Assigned: Unassigned                                              │  │
│  │                                                                    │  │
│  │  [Assign to Me]  [View Practice]  [Dismiss with Note]              │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

**Key difference from original design:**
- **"Est. Weekly Impact: -$3,200"** is replaced with **"Culture Risk: HIGH"**
- Revenue impact requires patient funnel data from ClickHouse. Culture risk is derived entirely from non-PHI signals: H-Score trajectory, huddle compliance, certification gaps, content capture frequency, brand health.
- The ranking still produces a meaningful priority order. HIGH/MEDIUM/LOW maps to urgency thresholds: H-Score below 50 = HIGH, declining >5 pts/week = HIGH, champion inactive >14d = MEDIUM, brand health below 50 = MEDIUM.

**Phase B Addition:** When funnel data becomes available, the intervention card will show both "Culture Risk: HIGH" AND "Est. Weekly Revenue Impact: -$3,200." Dual ranking becomes possible — executives can sort by culture risk or revenue impact.

---

## Flow 6: Office Manager Limited View — Lisa Torres

### Step 6.1: Lisa's Morning — Practice Operations PWA

**Time**: 7:45 AM, Lisa opens the PWA on her phone.

**Bottom navigation (Office Manager, Phase A):**
```
┌───────────────────────────────────────────────┐
│  [ Today ]  [ Storefront ]  [ Capture ]  [ Huddle ]  │
└───────────────────────────────────────────────┘
```

**Today Screen:**

```
┌──────────────────────────────┐
│  SGA ◆ Smile Center Buckhead │
│  Thursday, March 20          │
├──────────────────────────────┤
│                              │
│  PRACTICE BRIEFING           │
│  ┌──────────────────────┐    │
│  │  H-SCORE: 64         │    │
│  │  ▼14 from 2 weeks ago│    │
│  │  Focus: Warm Handoffs│    │
│  │  [Tap for detail]    │    │
│  └──────────────────────┘    │
│                              │
│  ACTION CARDS                │
│  ┌──────────────────────┐    │
│  │  ⚠ STOREFRONT ALERT  │    │
│  │  Dr. Nguyen headshot │    │
│  │  is 8 months old.    │    │
│  │  Schedule new photo.  │    │
│  │  [Open Storefront →]  │    │
│  └──────────────────────┘    │
│  ● ○ ○                      │
│  ┌──────────────────────┐    │
│  │  📷 CONTENT CAPTURE  │    │
│  │  Team photo day —     │    │
│  │  capture the team     │    │
│  │  before morning huddle│    │
│  │  [Open Camera →]      │    │
│  └──────────────────────┘    │
│  ○ ● ○                      │
│  ┌──────────────────────┐    │
│  │  📋 CULTURE FOCUS    │    │
│  │  This week: Warm      │    │
│  │  Handoffs. "Introduce │    │
│  │  every patient to     │    │
│  │  their next caregiver │    │
│  │  by name."            │    │
│  │  [View Playbook →]    │    │
│  └──────────────────────┘    │
│  ○ ○ ●                      │
│                              │
│  ENROLLMENT STATUS           │
│  (not shown — practice is    │
│   already live)              │
│                              │
│  ┌──────────────────────┐    │
│  │  PRACTICE ACTIVITY    │    │
│  │  (expands after 4 PM) │    │
│  │  ▼ Tap to preview     │    │
│  └──────────────────────┘    │
│                              │
├──────────────────────────────┤
│ [Today] [Storefront] [Capture] [Huddle] │
└──────────────────────────────┘
```

**What Lisa sees that's different from the original design:**

| Original Design (Full PHI) | Phase A (PHI-Free) |
|---|---|
| Action Card: "7 confirmations due — Review & Send" | Not present. Patient confirmations require patient names, appointment data. |
| Action Card: "3 treatment follow-ups due today" | Not present. Follow-ups require patient treatment records. |
| Action Card: "5 reactivation candidates — messages drafted" | Not present. Reactivation requires patient visit history. |
| Patients screen in bottom nav | Not present. Entire screen deferred. |
| Calls screen in bottom nav | Not present. Entire screen deferred. |
| Auto screen (Automations) in bottom nav | Not present. Entire screen deferred. |
| End-of-Day Scorecard: calls handled, messages sent, revenue impact | Replaced with Practice Activity Summary: captures submitted, huddle completed, storefront updates, playbook sections viewed. |

**How the UI handles the absence:**
- The bottom nav shows exactly 4 items — no empty slots, no grayed-out icons.
- Action cards are AI-generated from available Phase A signals. The AI selects the top 3 most relevant from: storefront alerts, content capture opportunities, playbook updates, culture focus, enrollment milestones. The card count and swipe UX are identical to the original; only the card content changes.
- The Today screen feels complete. A user who has never seen the original design would not know anything is missing.

### Step 6.2: Huddle Screen

Lisa navigates to the **Huddle** tab to prepare for the morning huddle.

```
┌──────────────────────────────┐
│  Morning Huddle               │
│  Thursday, March 20           │
├──────────────────────────────┤
│                              │
│  H-SCORE                     │
│  64 (▼14 from 2 weeks ago)  │
│  Phone: 58 | Reviews: 71    │
│  DI: 74                     │
│                              │
│  ─────────────────────────── │
│                              │
│  FOCUS TODAY                 │
│  Warm Handoffs               │
│  "Introduce every patient to │
│  their next caregiver by     │
│  name. Say: 'Dr. Patel is   │
│  going to take great care of │
│  you today.'"                │
│                              │
│  ─────────────────────────── │
│                              │
│  RECOGNITION                 │
│  Maria Santos — 12-week      │
│  huddle streak! Longest in   │
│  the Southeast region.       │
│                              │
│  ─────────────────────────── │
│                              │
│  IMPACT STORY                │
│  "Our Google rating improved │
│  from 4.1 to 4.3 since we   │
│  focused on the review       │
│  request script in huddles.  │
│  That's 18 new reviews in    │
│  30 days."                   │
│                              │
│  [Mark Huddle Complete ✓]    │
│                              │
├──────────────────────────────┤
│ [Today] [Storefront] [Capture] [Huddle] │
└──────────────────────────────┘
```

**Impact Story change**: Original design used funnel metrics ("Our show rate improved 5% — that's 4 more patients/week"). Phase A uses culture/brand metrics ("Our Google rating improved from 4.1 to 4.3"). This is the ONLY content change in the entire Morning Huddle interface.

### Step 6.3: What Lisa CANNOT See

Lisa does not have access to:

| Missing Capability | Why | Phase B Restoration |
|---|---|---|
| **Practice Nerve Center** (patient flow real-time view) | Requires PMS appointment data showing patient names, times, providers | PO-1 Today screen gains patient-driven action cards |
| **Patient Outreach** (confirmation, follow-up, reactivation messaging) | Requires patient contact data, appointment context, communication history | PO-2 Patient Outreach Queue appears in nav |
| **Automation Status** (monitoring autonomous messaging sequences) | Automations target individual patients with personalized messages | PO-5 Automations screen appears in nav |
| **Call Dashboard** (incoming call context, Voice AI status) | Call context cards require PMS patient record lookup | PO-3 Calls screen appears in nav |
| **End-of-Day Scorecard** (calls handled, messages sent, revenue impact) | Revenue impact requires funnel conversion data | Scorecard regains patient-centric metrics |

Lisa's Phase A experience is focused on **practice brand and culture management**: keeping the storefront healthy, capturing content, participating in huddles, following playbooks. This is genuinely useful work — it builds the operational foundation that patient-facing workflows will depend on in Phase B.

---

## Cross-Flow Data Summary

### Data That Crosses Workstream Boundaries in Phase A

| Data Element | Source | Consumer | Mechanism | PHI Status |
|---|---|---|---|---|
| H-Score (composite + sub-scores) | Culture OS (WS3) | Network Intelligence KPI bar, Practice Deep Dive Culture tab, PO Today screen | Kafka `sgadental.ws3.hscore-calculated` | Non-PHI (aggregate vendor scores) |
| Brand health score | Content Engine (WS2) | Network Intelligence KPI bar, Practice Deep Dive Content tab | Kafka `sgadental.ws2.brand-health-calculated` | Non-PHI |
| Content pipeline metrics | Content Engine (WS2) | Network Intelligence Pipeline Health widget | Kafka events (capture, process, approve, publish) | Non-PHI (counts, not content) |
| Content capture stats | Content Engine (WS2) | Network Intelligence KPI bar, Champion Hub scorecard | Kafka `sgadental.ws2.content-engagement-updated` | Non-PHI |
| CCA certification status | Culture OS (WS3) | Network Intelligence Practice Deep Dive Culture tab | Kafka `sgadental.ws3.training-completed` | Non-PHI (staff data) |
| Huddle completion | Culture OS (WS3) | Network Intelligence Practice Deep Dive Culture tab | Kafka `sgadental.ws3.huddle-completed` | Non-PHI |
| Trust transfer scores (aggregate) | Cinnamon via Culture OS (WS3) | Network Intelligence Practice Deep Dive Culture tab | Kafka `sgadental.ws3.hscore-calculated` | Non-PHI (aggregate per staff) |
| Brand deviation flags | Content Engine (WS2) | Network Intelligence QC Queues | Kafka event with asset IDs | Non-PHI |
| Enrollment status | Practice Data Hub (shared) | All surfaces | Kafka `sgadental.ws1.onboarding-status-changed` | Non-PHI |
| Staff roster | iSolved via Practice Data Hub (shared) | Culture OS, Network Intelligence | PostgreSQL shared table | Non-PHI (employee data) |
| Approved brand assets | Content Engine Asset Registry (WS2) | Practice Operations Storefront Gallery | API read + Kafka `sgadental.ws2.content-asset-approved` | Non-PHI |

### Data That Does NOT Flow in Phase A

| Data Element | Would Flow From | Blocked By | Phase B Requirement |
|---|---|---|---|
| Patient funnel events | PMS via ClickHouse | No PMS sync active | BAA + PMS integration activation |
| Patient communications | Twilio / Communications Service | No patient data to message | BAA + patient consent |
| Call recordings / transcripts | Twilio / Cinnamon | PHI in recordings | Twilio BAA + Whisper BAA |
| Individual appointment data | PMS | PHI | BAA + PMS integration |
| Revenue / production data | PMS | PHI (derived from patient records) | BAA + PMS integration |
| Patient before/after photos | Content capture | Patient consent required | Consent workflow + clinical photo pipeline |

---

## Phase B Addition Callouts

| Phase A State | Phase B Addition | Trigger |
|---|---|---|
| KPI bar: 5 cards (practices, H-Score, brand health, enrollment, captures) | +3 cards: Total Production, New Patients, Show Rate. Grid expands with staggered animation. | `revenue_impact_scoring` feature flag enabled |
| Practice Deep Dive: 4 tabs (Profile, Culture, Content & Brand, History) | +2 tabs: Performance (position 2), Communications (position 3). Existing tabs shift right. "NEW" badge on injected tabs. | `pms_integration_active` flag per practice |
| AI Alerts: culture/brand/enrollment anomalies only | +funnel alerts: show rate drops, conversion anomalies, revenue impact. Same AlertCard component. | `funnel_analytics_active` flag |
| Intervention Queue: Culture Risk ranking | +Revenue Impact column alongside Culture Risk. Dual sort available. | `revenue_impact_scoring` flag |
| Heatmap: 5 color layers | +4 layers: Composite Health, Funnel Performance, Show Rate, Treatment Acceptance. Dropdown gains options. | `funnel_analytics_active` flag |
| Office Manager nav: 4 items (Today, Storefront, Capture, Huddle) | +Patients, +Calls, +Automations. Nav expands. Onboarding tooltip for new tabs. | Per-practice feature flags |
| Morning Huddle Impact Story: culture/brand metrics | +funnel stories: "Show rate improved 5% since confirmation texts." | `pms_integration_active` flag |
| Standup Board: culture/brand/enrollment ranking | +revenue-impact ranking as alternative sort. Both available. | `revenue_impact_scoring` flag |

---

*This journey specification covers all WS1 Practice Platform flows in Phase A (PHI-free) configuration. Every screen, data source, and interaction is functional without patient health information. The 30% of interfaces that require PHI are cleanly deferred, with no phantom navigation, no broken states, and no user-visible gaps. The Phase A platform delivers practice culture management, brand content operations, network visibility, and enrollment pipeline — the operational foundation required before patient-facing features activate in Phase B.*
