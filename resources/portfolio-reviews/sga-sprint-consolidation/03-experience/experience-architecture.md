# SGA Dental Partners — Experience Architecture

**Date**: 2026-03-14
**Author**: Product Designer Agent
**Status**: QC-1 Revision Response
**QC Items Addressed**: #1 (WS1 scope overload), #2 (storefront/content boundary), #6 (mobile strategy)

---

## Design Principles

Before detailing screens and flows, these principles govern every design decision in this document:

1. **One user, one surface.** No actor type should need to navigate between product surfaces to complete their daily workflow. Cross-workstream data comes to them.
2. **Progressive disclosure over feature density.** Show three things that matter now, not thirty things that might matter sometime.
3. **The system initiates; the human decides.** AI generates the agenda, drafts the message, flags the anomaly. Humans approve, redirect, or override.
4. **Mobile is the default for practice staff; desktop is the default for HQ.** Design for the device where the work actually happens.
5. **Workstream boundaries are invisible to users.** Users see "my work." They do not see "Workstream 1" or "Workstream 2."

---

## Section 1: Interface Inventory

### 1A. Practice Operations (WS1 Surface A)

| # | Interface | Actor Types | Platform | Primary Use Cases | Frequency |
|---|-----------|-------------|----------|-------------------|-----------|
| PO-1 | **Practice Daily Agenda** (Practice Nerve Center) | Office Managers | PWA (mobile-first), Web | Review 3 prioritized action cards, execute patient outreach, close end-of-day scorecard | Daily (morning + end-of-day) |
| PO-2 | **Patient Outreach Queue** | Office Managers, Front Desk | PWA, Web | Review AI-drafted messages, approve/edit/send, manage follow-up sequences | Daily |
| PO-3 | **Call Dashboard** | Front Desk | Web (desktop at front desk station) | View incoming call context, see patient history during call, log call outcomes | Daily (continuous) |
| PO-4 | **Voice AI Monitor** | Office Managers | PWA, Web | Review Voice AI call transcripts, listen to recordings, flag for training, manage escalations | Daily |
| PO-5 | **Automation Status Board** | Office Managers | Web | View active automation sequences (confirmations, follow-ups, rebooking), see graduation status per communication type, pause/resume sequences | Daily |
| PO-6 | **Storefront Manager** | Office Managers, HQ Ops | Web | View/edit practice website, GMB listing, directory listings; pull approved assets from Asset Registry; trigger publication | Weekly |
| PO-7 | **Content Capture Prompt** (embedded) | Office Managers, Clinical Staff | SMS, PWA push notification | Receive daily capture brief ("3 implant cases today — capture checklist"), submit photos/video via camera interface | Daily (procedure-triggered) |
| PO-8 | **Morning Huddle Screen** | Office Managers, Clinical Staff, Front Desk | TV display (Dental Intel), PWA | View 5-minute huddle agenda: yesterday's wins, today's focus behaviors, H-Score trend | Daily (morning) |
| PO-9 | **Playbook Viewer** | Office Managers, Front Desk, Clinical Staff | PWA, Web | Access current playbooks (phone scripts, treatment follow-up protocols, onboarding checklists), view version history | On-demand |

### 1B. Network Intelligence (WS1 Surface B)

| # | Interface | Actor Types | Platform | Primary Use Cases | Frequency |
|---|-----------|-------------|----------|-------------------|-----------|
| NI-1 | **Executive Dashboard** | CEO/Executives | Web (desktop, large screen) | View network-wide KPIs, drill from network to region to practice, review AI alerts, assess portfolio health | Daily |
| NI-2 | **Portfolio Heatmap** | VP Ops, Regional Directors | Web | View geographic and color-coded practice performance grid, identify struggling/thriving clusters, compare regions | Daily |
| NI-3 | **Intervention Queue** | VP Ops, Regional Directors | Web | Review AI-ranked practices needing attention (top 14 by revenue impact), read AI summaries, assign interventions, track resolution | Daily |
| NI-4 | **HQ Ops Standup Board** | HQ Operations Team | Web | View prioritized problem queue across all 260+ practices, batch operations panel, enrollment pipeline status, QC queues | Daily |
| NI-5 | **Practice Deep Dive** | VP Ops, HQ Ops, Regional Directors | Web | Single-practice 360 view: funnel metrics, comms performance, digital health score, H-Score, content activity, automation status, staff roster | On-demand |
| NI-6 | **Funnel Analytics Explorer** | HQ Ops, VP Ops | Web | Custom funnel analysis: filter by practice type, region, time period, service type; cohort comparison; anomaly investigation | Weekly |
| NI-7 | **Enrollment Pipeline** | HQ Ops | Web | Track new practice onboarding: data harvesting status, integration connectivity, blitz deploy countdown, go-live readiness | On-demand (acquisition-driven) |
| NI-8 | **HQ Playbook Manager** | HQ Ops | Web | Design, version, and deploy playbook updates across the network; A/B test playbook variants; view adoption metrics | Weekly |
| NI-9 | **Compliance Dashboard** | HQ Ops (Compliance Officer) | Web | HIPAA audit trail, PHI access logs, opt-out compliance rates, consent status by practice, flagged communications | Weekly |
| NI-10 | **Network Learning Report** | VP Ops, CEO | Web | Monthly AI-generated report: what the system learned, top optimization wins, cross-practice insights, recommended strategy changes | Monthly |

### 1C. Content Engine (WS2)

| # | Interface | Actor Types | Platform | Primary Use Cases | Frequency |
|---|-----------|-------------|----------|-------------------|-----------|
| CE-1 | **Content Capture Bot** | Office Managers, Clinical Staff (Champions) | SMS / WhatsApp | Receive daily capture brief, submit photos/video via reply, get instant confirmation and processing status | Daily |
| CE-2 | **Capture App** (Phase 2) | Champions | PWA (iOS/Android) | Camera interface with overlay guides, before/after pairing, video recording with timer, batch upload | Daily |
| CE-3 | **Editorial Queue** | HQ Content Team | Web | Review AI-processed content: approve/reject/edit captions, review photo corrections, schedule distribution, manage content calendar | Daily |
| CE-4 | **Studio Request Portal** | Office Managers, HQ Marketing | Web, Email (studio@sgadental.com), Teams Bot | Submit design requests (flyers, social posts, TV slides), track request status, receive proofs, approve finals | On-demand |
| CE-5 | **Creative Director Review** | Creative Director | Teams (Adaptive Card), Web | 58-second tap-approve flow for brand-controlled collateral, batch review mode, brand deviation flags | Daily |
| CE-6 | **Asset Registry** | HQ Content, HQ Ops, Office Managers (read-only) | Web | Browse all approved assets by practice, brand universe, asset type, date; download originals; view usage history; check brand health score | On-demand |
| CE-7 | **Content Engine Dashboard** | HQ Content Team, VP Marketing | Web | Pipeline metrics: requests received, content captured, assets produced, assets distributed, engagement metrics, brand health scores, champion leaderboard | Daily |
| CE-8 | **Distribution Monitor** | HQ Content Team | Web | View distribution status across 6 social channels + Weave + print; see engagement metrics per post; manage scheduling queue | Daily |
| CE-9 | **Event Mode Console** | HQ Content Team | Web, PWA | Special capture mode for 4Front, Pikos courses, partner events; high-volume intake; real-time processing dashboard | Event-driven |

### 1D. Culture OS (WS3)

| # | Interface | Actor Types | Platform | Primary Use Cases | Frequency |
|---|-----------|-------------|----------|-------------------|-----------|
| CO-1 | **Morning Huddle Display** | All Practice Staff | TV display (Dental Intel integration), PWA | 5-minute daily ritual: yesterday's H-Score, today's focus behavior, recognition shoutout, practice wins | Daily |
| CO-2 | **CCA Academy Portal** | Clinical Staff, Office Managers | Web, PWA | Complete certification modules (Bronze/Silver/Gold), watch training videos, take assessments, view credentials, track progress | Weekly |
| CO-3 | **Champion Hub** | Champions (1 per practice) | PWA, Web | View champion scorecard, streak counter, content capture assignments, culture mission for the week, regional cohort leaderboard | Daily |
| CO-4 | **Recognition Feed** | All Practice Staff | PWA, TV display | View recognition posts, celebrate peers, see leaderboard standings, bonus pool status | Daily (passive viewing) |
| CO-5 | **H-Score Dashboard** | Office Managers, Regional Directors | Web | Detailed H-Score breakdown: Cinnamon phone score (35%), Swell reviews (45%), Dental Intel (20%); trend over time; behavior-level detail | Weekly |
| CO-6 | **Playbook Library** | All Practice Staff | PWA, Web | Access hospitality playbooks by behavior, watch Grant training videos, view practice-type-specific protocols | On-demand |
| CO-7 | **Trust Transfer Tracker** | Office Managers, Clinical Staff | Web | View NLP-verified trust transfer scores per call, identify training opportunities, track improvement trends | Weekly |
| CO-8 | **Culture Admin Console** | HQ Culture Team | Web | Manage CCA curriculum, deploy playbook updates, configure huddle content, manage champion network, set reward budgets | Weekly |

### 1E. Patient-Facing Interfaces (Cross-Workstream)

| # | Interface | Actor Types | Platform | Primary Use Cases | Frequency |
|---|-----------|-------------|----------|-------------------|-----------|
| PF-1 | **Practice Website** | Patients | Web (responsive) | Find practice info, view services, see before/after galleries, read reviews, book appointments, access forms | On-demand |
| PF-2 | **Voice AI (Phone)** | Patients | Phone (voice) | Call practice, speak with AI that does real-time PMS lookup, book/reschedule/cancel appointments, get after-hours answers | On-demand |
| PF-3 | **SMS Conversations** | Patients | SMS | Receive appointment confirmations, respond to follow-up messages, opt in/out, receive rebooking prompts, reactivation outreach | Triggered by automation |
| PF-4 | **Email Communications** | Patients | Email | Receive appointment confirmations, treatment follow-up education, rebooking reminders, reactivation campaigns | Triggered by automation |
| PF-5 | **Review Request** | Patients | SMS, Email | Receive post-visit review prompt, link to Google/Swell review page | Post-visit |
| PF-6 | **Digital Forms** | Patients | Web (mobile-responsive) | Complete pre-visit intake forms, update medical history, sign consent | Pre-visit |

### Interface Count Summary

| Surface | Interfaces | Primary Actors |
|---------|-----------|----------------|
| Practice Operations | 9 | Office Managers, Front Desk, Clinical Staff |
| Network Intelligence | 10 | CEO, VP Ops, Regional Directors, HQ Ops |
| Content Engine | 9 | Champions, HQ Content Team, Creative Director |
| Culture OS | 8 | All Practice Staff, Champions, HQ Culture Team |
| Patient-Facing | 6 | Patients |
| **Total** | **42** | |

---

## Section 2: Information Architecture

### 2A. Practice Operations — Navigation Structure

Practice Operations is the daily interface for practice-level staff. It is organized around time, not features.

```
Practice Operations (PWA / Web)
├── Today (default landing)
│   ├── Morning Briefing
│   │   ├── 3 Prioritized Action Cards (AI-generated)
│   │   ├── Today's Schedule Summary (from PMS)
│   │   ├── H-Score Trend Widget (from Culture OS)
│   │   └── Content Capture Brief (from Content Engine)
│   ├── Patient Outreach Queue
│   │   ├── Pending Messages (AI-drafted, awaiting approval)
│   │   ├── Active Sequences (confirmations, follow-ups in progress)
│   │   ├── Escalations (messages flagged for human review)
│   │   └── Sent Today (with delivery/response status)
│   └── End-of-Day Scorecard
│       ├── Actions Completed vs. Planned
│       ├── Calls Handled (human vs. Voice AI)
│       ├── Messages Sent & Response Rate
│       └── Revenue Impact Estimate
│
├── Patients
│   ├── Active Patients (with funnel stage indicator)
│   ├── Follow-Up Queue (treatment acceptance outreach)
│   ├── Rebooking Queue (overdue patients)
│   ├── Reactivation Queue (lapsed patients)
│   └── Patient Detail View
│       ├── Contact Info & Preferences
│       ├── Appointment History
│       ├── Communication History
│       ├── Funnel Stage
│       └── Channel Preference (SMS/email/voice)
│
├── Calls
│   ├── Live Call Panel (active calls with context card)
│   ├── Voice AI Transcripts (today's AI-handled calls)
│   ├── Missed Call Recovery Queue
│   └── Call Analytics (volume, duration, outcomes)
│
├── Automations
│   ├── Active Sequences by Type
│   │   ├── Confirmations
│   │   ├── Follow-Ups
│   │   ├── Rebooking
│   │   └── Reactivation
│   ├── Graduation Status (Supervised → Monitored → Autonomous)
│   ├── Sequence Performance (delivery, response, conversion)
│   └── Pause/Resume Controls
│
├── Storefront
│   ├── Website Preview (live site with edit overlay)
│   ├── Google Business Profile Status
│   ├── Directory Listings (sync status per platform)
│   ├── Asset Gallery (approved assets from Content Engine Asset Registry)
│   │   └── "Use on Website" / "Push to GMB" actions
│   └── Digital Health Score (0-100)
│
└── Settings
    ├── Practice Profile
    ├── Provider Roster
    ├── Schedule Patterns
    ├── Communication Preferences
    └── Automation Rules
```

**Data Sources for Practice Operations:**
- **PMS (via NexHealth/Sikka ONE)**: Appointment schedules, patient records, procedure codes, provider schedules
- **Twilio**: SMS delivery status, call logs, Voice AI transcripts
- **Practice Data Hub (PostgreSQL)**: Practice profile, provider roster, automation configurations
- **ClickHouse**: Funnel metrics, communication performance, conversion analytics
- **Culture OS (via Kafka event)**: H-Score (daily), huddle completion status
- **Content Engine (via Kafka event)**: Approved assets in Asset Registry, content capture prompts, champion scorecard

**Key Actions:**
- Approve/edit/send patient messages
- Book/reschedule appointments
- Review and act on Voice AI transcripts
- Update storefront content
- Submit content captures (photos/video)
- Pause/resume automation sequences

---

### 2B. Network Intelligence — Navigation Structure

Network Intelligence is the strategic interface for HQ and leadership. It is organized around scope — from network-wide down to individual practice.

```
Network Intelligence (Web — Desktop)
├── Network Overview (default landing)
│   ├── KPI Bar
│   │   ├── Total Production (MTD, vs. target, vs. prior year)
│   │   ├── New Patients (MTD, trend)
│   │   ├── Show Rate (network avg, trend)
│   │   ├── Treatment Acceptance (network avg, trend)
│   │   └── H-Score (network avg, from Culture OS)
│   ├── AI Alerts Panel (top 5 anomalies requiring attention)
│   ├── Portfolio Heatmap (geographic map with color-coded health)
│   └── Trend Charts (selectable metrics, 30/60/90-day)
│
├── Regions
│   ├── Region Selector (dropdown or map click)
│   ├── Region KPIs (same structure as network, filtered)
│   ├── Region Practice Ranking (sorted by composite score)
│   ├── Region Intervention Queue (flagged practices in this region)
│   └── Regional Director Notes (context, action items)
│
├── Practices
│   ├── Practice Search (name, ID, region, type)
│   ├── Practice Filters (type, size, acquisition date, health band)
│   └── Practice Deep Dive (single-practice 360 view)
│       ├── Performance Tab
│       │   ├── 5-Stage Funnel Visualization
│       │   ├── Conversion Rates by Stage
│       │   ├── Revenue Trend
│       │   └── Benchmarks vs. Practice Type Cohort
│       ├── Communications Tab
│       │   ├── Message Volume & Response Rates
│       │   ├── Automation Performance
│       │   ├── Voice AI Usage & Quality Scores
│       │   └── Channel Mix (SMS vs. email vs. voice)
│       ├── Digital Presence Tab
│       │   ├── Website Health Score
│       │   ├── GMB Listing Status
│       │   ├── Directory Sync Status
│       │   └── Recent Content Published
│       ├── Culture Tab
│       │   ├── H-Score (composite + sub-scores)
│       │   ├── CCA Certification Status (team roster with tiers)
│       │   ├── Huddle Completion Rate
│       │   ├── Trust Transfer Scores
│       │   └── Champion Activity
│       └── History Tab
│           ├── Enrollment Date & Timeline
│           ├── Key Events Log
│           └── Intervention History
│
├── Operations
│   ├── HQ Standup Board
│   │   ├── Top 14 Practices by Revenue Impact (AI-ranked)
│   │   ├── Unresolved Issues Queue
│   │   ├── Today's Batch Operations
│   │   └── Cross-Practice Patterns (AI-identified)
│   ├── Enrollment Pipeline
│   │   ├── Active Enrollments (Kanban: Harvesting → Integrating → Testing → Live)
│   │   ├── Blitz Deploy Countdown (30-day timers)
│   │   └── Enrollment Velocity Metrics
│   ├── QC Queues
│   │   ├── Website QC Queue (flagged pages/listings)
│   │   ├── Comms QC Queue (flagged messages)
│   │   └── Brand Health Violations (from Content Engine)
│   └── Playbook Manager
│       ├── Active Playbooks (with version numbers)
│       ├── Playbook Editor (design new or version existing)
│       ├── Deployment Status (which practices have which version)
│       └── A/B Test Results
│
├── Analytics
│   ├── Funnel Explorer (custom queries with filters)
│   ├── Cohort Comparison (select practice groups to compare)
│   ├── Attribution Analysis (content → leads → bookings)
│   ├── AI Insights (monthly learning report)
│   └── Export Center (scheduled reports, CSV/PDF export)
│
├── Compliance
│   ├── HIPAA Audit Log
│   ├── PHI Access Report
│   ├── Opt-Out Compliance
│   ├── Consent Status by Practice
│   └── Flagged Communications
│
└── Settings
    ├── User Management (roles, permissions)
    ├── Practice Registry
    ├── Integration Status (PMS connectors, API health)
    ├── Feature Flags
    └── Notification Preferences
```

**Data Sources for Network Intelligence:**
- **ClickHouse**: All analytics — funnel metrics, communication performance, content engagement, trend calculations, anomaly detection
- **Practice Data Hub (PostgreSQL)**: Practice profiles, enrollment status, configuration, staff roster
- **AI Analysis Engine**: Anomaly alerts, root cause analysis, ranked intervention queues, cross-practice pattern detection
- **Culture OS (via Kafka)**: H-Score per practice (daily batch), CCA certification counts, huddle completion rates
- **Content Engine (via Kafka)**: Brand health scores, content capture frequency, asset counts, engagement metrics
- **Compliance Auditor**: PHI access logs, opt-out status, consent records

**Key Actions:**
- Drill from network to region to practice (two clicks)
- Assign interventions to practices
- Approve/deploy playbook updates
- Manage enrollment pipeline
- Run custom analytics queries
- Export reports
- Manage QC queues

**Cross-Workstream Data Surfacing:**

| Data Element | Source Workstream | How It Appears in Network Intelligence |
|---|---|---|
| H-Score | Culture OS (WS3) | Widget on Network Overview KPI bar; dedicated tab in Practice Deep Dive; color layer on Portfolio Heatmap |
| CCA Certification counts | Culture OS (WS3) | Certification status section in Practice Deep Dive Culture Tab |
| Brand Health Score | Content Engine (WS2) | Merged into Digital Presence Tab in Practice Deep Dive; flagged in QC Queues |
| Content Capture Frequency | Content Engine (WS2) | Metric in Practice Deep Dive Digital Presence Tab; sortable column in practice rankings |
| Champion Activity | Culture OS (WS3) + Content Engine (WS2) | Combined view in Practice Deep Dive Culture Tab showing both culture and content champion metrics |
| Content Engagement | Content Engine (WS2) | Attribution analysis in Analytics section; engagement trend in Practice Deep Dive |

---

### 2C. Content Engine — Navigation Structure

```
Content Engine (Web — Desktop)
├── Pipeline (default landing)
│   ├── Capture Inbox
│   │   ├── New Submissions (photos/videos from SMS bot and Capture App)
│   │   ├── AI Processing Queue (color correction, cropping, caption generation in progress)
│   │   └── Ready for Review (AI-processed, awaiting human QC)
│   ├── Editorial Queue
│   │   ├── Pending Approval (AI-processed content with draft captions)
│   │   ├── Approved / Scheduled (ready for distribution)
│   │   ├── Published (live content with engagement tracking)
│   │   └── Rejected (with rejection reason for champion feedback)
│   └── Studio Requests
│       ├── Open Requests (design collateral requests)
│       ├── In Production (assigned to template or human designer)
│       ├── Awaiting Approval (proofs sent to requestor)
│       └── Completed
│
├── Asset Registry
│   ├── Browse (filterable by practice, brand universe, asset type, date, status)
│   ├── Asset Detail View
│   │   ├── Preview (full-size image/video)
│   │   ├── Metadata (practice, brand, type, creation date, platforms published)
│   │   ├── Usage History (where this asset has been published)
│   │   └── Variants (different sizes/crops for different platforms)
│   └── Upload (manual asset upload for legacy/external content)
│
├── Distribution
│   ├── Calendar View (scheduled posts across all practices and channels)
│   ├── Channel Status (per-platform API health and posting status)
│   ├── Distribution Rules (which content types go to which channels)
│   └── Engagement Feed (real-time engagement metrics per distributed post)
│
├── Champions
│   ├── Champion Roster (synced from Culture OS)
│   ├── Leaderboard (ranked by capture frequency, quality, streak)
│   ├── Champion Detail (individual activity, scorecard, communication log)
│   └── Brief Configuration (per-practice capture playbook settings)
│
├── Brand
│   ├── Brand Universes (SGA Legacy, Gen4, MODIS)
│   │   ├── Brand Guidelines (colors, fonts, logo usage)
│   │   ├── Template Library (Canva templates per brand)
│   │   └── Brand Health Scores (per practice within brand universe)
│   └── Brand Deviation Queue (flagged assets that may violate brand guidelines)
│
├── Analytics
│   ├── Pipeline Metrics (volume, velocity, quality at each stage)
│   ├── Champion Performance (capture rates, quality trends)
│   ├── Engagement Analytics (per channel, per content type, per practice)
│   ├── Attribution (content → booking correlation, from Practice Platform)
│   └── Brand Health Trends
│
└── Settings
    ├── Channel Configurations (API keys, posting accounts per practice)
    ├── Template Management
    ├── AI Processing Rules
    ├── Distribution Rules
    └── Practice Content Playbooks (per practice-type capture strategies)
```

---

### 2D. Culture OS — Navigation Structure

```
Culture OS (Web + PWA)
├── Network View (HQ Culture Team default)
│   ├── H-Score Network Map (geographic view with practice H-Scores)
│   ├── Network Trends (H-Score over time, certification progress)
│   ├── Top Practices / Bottom Practices
│   └── AI Insights (behavior patterns, training recommendations)
│
├── Practice View (Office Manager / Champion default)
│   ├── Today's Huddle
│   │   ├── Huddle Agenda (auto-generated, 5 minutes)
│   │   ├── Yesterday's H-Score & Trend
│   │   ├── Focus Behavior of the Day
│   │   ├── Recognition Moment (peer shoutout)
│   │   └── Impact Story (funnel data tied to culture, from WS1)
│   ├── H-Score Detail
│   │   ├── Composite Score (0-100)
│   │   ├── Cinnamon Phone Score (35% weight)
│   │   ├── Swell Review Score (45% weight)
│   │   ├── Dental Intel Score (20% weight)
│   │   └── 12 Core Behaviors Breakdown
│   ├── Team
│   │   ├── Certification Status (per team member: Bronze/Silver/Gold/None)
│   │   ├── Training Progress (modules completed, upcoming)
│   │   └── Trust Transfer Scores (per team member, from call analysis)
│   └── Recognition
│       ├── Recognition Feed (recent recognition posts)
│       ├── Leaderboard (practice-level and regional)
│       └── Bonus Pool Status
│
├── Academy
│   ├── My Progress (individual learner view)
│   │   ├── Current Tier & Progress to Next
│   │   ├── Active Modules
│   │   ├── Completed Modules & Credentials
│   │   └── Credly/LinkedIn Badges
│   ├── Module Library
│   │   ├── Bronze Track (foundational hospitality)
│   │   ├── Silver Track (advanced service skills)
│   │   ├── Gold Track (leadership & coaching)
│   │   └── Doctor-Specific Modules
│   └── Assessments
│       ├── Upcoming Assessments
│       ├── Completed (with scores)
│       └── Certification Exam
│
├── Champions
│   ├── My Dashboard (champion's personal view)
│   │   ├── Culture Scorecard
│   │   ├── Content Capture Stats (from Content Engine)
│   │   ├── Weekly Mission
│   │   └── Streak Counter
│   ├── Regional Cohort
│   │   ├── Cohort Leaderboard
│   │   ├── Shared Best Practices
│   │   └── Upcoming Cohort Activities
│   └── Champion Resources (playbooks, talking points, templates)
│
├── Playbooks
│   ├── Active Playbooks (current version per practice type)
│   ├── Behavior Guides (12 core behaviors with examples)
│   ├── Training Videos (Grant recordings via Mux)
│   └── Version History
│
└── Admin (HQ Culture Team only)
    ├── Curriculum Editor
    ├── Playbook Deployment
    ├── Huddle Content Manager
    ├── Champion Network Manager
    ├── Reward Budget Configuration
    └── Benchmarking Configuration
```

---

## Section 3: Consolidated User Journeys

### Journey 1: CEO / Executive — Monday Morning (15 minutes)

**Persona**: Dr. Sarah Chen, CEO of SGA Dental Partners. Checks the platform from her office on a 27" monitor before the 9 AM leadership call.

**8:35 AM — Opens Network Intelligence**
- Lands on **Network Overview**. The KPI bar loads in <3 seconds.
- Sees: Total Production $4.2M MTD (6% above target), Network H-Score 73 (up from 68 last month — data from Culture OS), 847 New Patients this month.
- The **AI Alerts Panel** shows 3 items:
  1. "Southeast region show rate dropped 4.2% week-over-week. Root cause: 3 practices had front desk turnover simultaneously."
  2. "Full-arch practices outperforming GP practices by 23% on treatment acceptance after playbook update v2.3."
  3. "Content capture frequency up 31% since champion training cohort 4 launched — 12 practices now at >5 captures/week."
- She clicks Alert #1. The **Practice Deep Dive** opens for the Southeast region, filtered to the 3 affected practices.

**8:40 AM — Drills into struggling practices**
- In the **Practice Deep Dive** for "Smile Center Buckhead":
  - **Performance Tab**: Show rate dropped from 82% to 71% over 3 weeks. The funnel visualization highlights the "Booked → Shows Up" stage in red.
  - **Communications Tab**: Confirmation sequence response rate normal. The AI notes: "New front desk staff member started 2 weeks ago. Call handling scores via Cinnamon are 22% below practice average."
  - **Culture Tab** (data from Culture OS): H-Score dropped from 78 to 64. Cinnamon phone sub-score is the primary driver. Trust Transfer scores show the new hire is missing warm handoffs.
- She sees the connection: new hire, no hospitality training yet, calls are degrading, patients aren't showing up.

**8:45 AM — Takes action**
- From the Practice Deep Dive, clicks "Assign Intervention."
- Types note: "New front desk hire needs accelerated CCA Bronze onboarding. Regional director should schedule 1:1 coaching this week."
- The intervention is routed to the Intervention Queue where the VP Ops for Southeast will see it with her context.
- She also notices the practice has no content captures in 2 weeks — the champion scorecard (from Content Engine) shows the content champion is the staff member who left.

**8:48 AM — Checks a bright spot**
- Clicks on the full-arch practices referenced in Alert #2.
- Opens the **Cohort Comparison** in Analytics. Selects "Full Arch practices" vs. "GP practices" for the last 90 days.
- Treatment acceptance: Full Arch 47% vs. GP 31%. The AI attribution note says: "Playbook v2.3 introduced treatment presentation scripting for full-arch. Practices using the playbook show 18% higher acceptance."
- She screenshots this for the leadership call.

**8:52 AM — Prepares for the call**
- Opens the **Network Learning Report** (monthly). Reviews the AI summary: top 3 optimization wins, top 3 areas needing attention, recommended strategy changes.
- Total time: 15 minutes. She never left Network Intelligence. Culture OS data and Content Engine data came to her.

---

### Journey 2: VP Operations / Regional Director — Daily Workflow

**Persona**: Marcus Williams, VP Ops for the Southeast region. Manages 45 practices. Works from a regional office.

**7:30 AM — Reviews Intervention Queue**
- Opens **Network Intelligence > Intervention Queue**.
- Sees 6 practices flagged, ranked by revenue impact:
  1. Smile Center Buckhead — show rate drop, new hire issue (CEO's intervention note is already here)
  2. Family Dental Marietta — reactivation campaign underperforming, AI suspects outdated patient contact data
  3. Peachtree Pediatric — H-Score dropped below 60 (from Culture OS), morning huddles not happening
- For each, he reads the AI summary, views the Practice Deep Dive, and either assigns to an HQ ops team member or takes a direct action.

**8:00 AM — Portfolio Heatmap review**
- Switches to **Portfolio Heatmap**. His 45 practices are displayed on a map.
- Color coding: green (composite score >80), yellow (60-80), red (<60).
- He notices a cluster of 3 yellow practices in suburban Atlanta that were green last month. Clicks the cluster.
- The drill-down shows all 3 had a content champion resign in the last 30 days. Content capture dropped to zero. He creates a note: "Need champion replacement plan for suburban Atlanta cluster."

**10:00 AM — Checks enrollment**
- A new acquisition closed last week. He opens **Enrollment Pipeline**.
- "Dental Arts Decatur" is in the "Harvesting" stage. Silent data harvesting is 60% complete. Integration connectors are pending (PMS is Open Dental, covered by NexHealth).
- He sees the projected go-live date: 11 days from now. Blitz deploy safety net is armed for day 30 if the practice manager doesn't engage.

**Weekly — Cohort analysis**
- Every Thursday, he opens **Funnel Analytics Explorer** and runs his standard query: all Southeast practices, grouped by practice type, last 30 days vs. prior 30 days.
- He exports a PDF for the regional leadership meeting.

---

### Journey 3: HQ Operations Team Member — Daily Standup

**Persona**: Angela Rivera, HQ Operations Analyst. One of ~10 people managing 260+ practices from Atlanta HQ.

**7:45 AM — HQ Standup Board**
- Opens **Network Intelligence > Operations > HQ Standup Board**.
- The board shows the **Top 14 Practices by Revenue Impact** — the 14 practices where intervention would recover the most revenue, ranked by the AI Analysis Engine.
- Each card shows: practice name, region, primary issue, estimated weekly revenue impact, days since flagged, assigned owner.
- 3 practices are unassigned. She claims "Family Dental Marietta" and opens its Practice Deep Dive.

**8:00 AM — Problem investigation**
- In Practice Deep Dive for Family Dental Marietta:
  - **Communications Tab**: Reactivation campaign sent 340 messages last month. Response rate: 3% (network average: 12%). The AI notes: "47% of patient phone numbers returned delivery errors. Suspect stale contact data from pre-acquisition period."
  - She clicks into the patient list, filtered to delivery errors. Confirms the pattern: patients acquired more than 2 years ago have outdated numbers.
- She takes two actions:
  1. Opens the **Playbook Manager**, creates a "data refresh" playbook variant that sends an email to patients with undeliverable phone numbers asking them to update their contact info.
  2. Opens a batch operation in the **QC Queue** to flag all practices acquired before 2024 for contact data freshness audit.

**9:00 AM — QC Queue rotation**
- She rotates to the **Website QC Queue**. There are 8 flagged pages across the network:
  - 3 have stale provider photos (provider left, photo still on website)
  - 2 have outdated hours
  - 3 have brand deviation flags (from Content Engine brand health scoring)
- For the stale provider photos, she uses batch operations to pull the providers from the websites. The change cascades to GMB and directory listings automatically (feature 1.08).
- For brand deviations, she sees the Content Engine has already generated compliant replacement assets in the **Asset Registry**. She approves the swap.

**Ongoing — Enrollment pipeline**
- She manages 3 active enrollments. Each morning she checks the pipeline status for her assigned practices, unblocking integration issues and following up with practice managers who haven't provided credentials.

---

### Journey 4: Office Manager — Full Day

**Persona**: Lisa Torres, Office Manager at a GP practice in suburban Atlanta. Has an iPhone and an iPad at the front desk.

**7:15 AM — Morning Huddle (5 minutes)**
- Opens the **Morning Huddle Display** on the break room TV (via Dental Intel integration, data from Culture OS).
- The huddle shows:
  - Yesterday's H-Score: 76 (up 2 from last week)
  - Focus Behavior Today: "Warm handoff — introduce patients to the hygienist by name and mention something personal"
  - Recognition: "Shoutout to Maria for getting 3 five-star reviews this week!"
  - Impact Story (from Practice Platform funnel data): "Our show rate improved 5% since we started the confirmation text sequence. That's approximately 4 more patients per week."
- Huddle takes 4 minutes. The team disperses to their stations.

**7:20 AM — Daily Agenda (Practice Operations PWA, 10 minutes)**
- Opens Practice Operations on her iPad. Lands on **Today**.
- The **Morning Briefing** shows 3 Action Cards:
  1. "7 patients need confirmation for tomorrow. AI has drafted messages — review and approve." (from Automation Engine)
  2. "3 treatment follow-ups due today. Mrs. Patterson's implant consultation was 6 days ago — AI suggests a check-in call." (from Comms Engine)
  3. "Content opportunity: You have 2 implant cases and 1 full-arch case today. Capture checklist ready." (from Content Engine via integration)
- She taps Action Card #1. The **Patient Outreach Queue** opens, showing 7 confirmation messages with AI-drafted text personalized per patient. She scans them, edits one ("Add that we have a new hygienist — introduce them"), approves all 7. Sent.

**7:30 AM — Content capture prompt**
- Her phone buzzes with an SMS from the Content Capture Bot (Content Engine): "Good morning Lisa! Today's capture brief: 2 implant cases (Dr. Patel, 10am and 2pm) and 1 full-arch consult (Dr. Kim, 11am). Before/after photos for the implant cases would be great. Reply SKIP to pass today."
- She texts back a thumbs-up emoji. She'll grab photos between appointments.

**10:15 AM — Content capture**
- After Dr. Patel's 10 AM implant case, Lisa opens the **Capture App** (PWA on her iPhone).
- She takes a before/after photo pair using the overlay guide. The app auto-tags it with the practice, procedure type, and date.
- She uploads. The Content Engine processes it: color correction, smart crop, and generates draft captions for Instagram, Facebook, and Google Business Profile.
- She goes back to her day. The content flows to the HQ Editorial Queue for approval.

**12:30 PM — Quick queue check**
- Between patients, she opens Practice Operations. The **Patient Outreach Queue** shows:
  - 2 patients responded to confirmation texts with "Yes, see you tomorrow"
  - 1 patient asked to reschedule — the AI flagged this for her to handle manually
  - Mrs. Patterson's follow-up: the AI drafted a warm message. Lisa approves it.

**2:00 PM — Storefront check (weekly)**
- Once a week, Lisa opens the **Storefront** section.
- The Digital Health Score is 88. One flag: "Provider photo for Dr. Kim is 8 months old."
- She sees the **Asset Gallery** has new photos from last week's content captures (approved by HQ Content Team and deposited in the Asset Registry).
- She selects two before/after photos and clicks "Use on Website." The photos appear on the practice website's gallery. She clicks "Push to GMB" — the photos are synced to the Google Business Profile.
- This is the WS2 → WS1 handoff in action: Content Engine produced the assets, deposited them in the Asset Registry, and Practice Operations' Storefront Manager pulled them for publication.

**4:45 PM — End of Day**
- Opens Practice Operations. The **End-of-Day Scorecard** shows:
  - 3/3 action cards completed
  - 22 calls handled (18 human, 4 Voice AI)
  - 12 messages sent, 8 responses received
  - 2 content captures submitted
  - Estimated revenue impact: $3,200 (from follow-ups and confirmations that converted)

---

### Journey 5: Clinical Staff (Hygienist) — Daily Touchpoints

**Persona**: Maria Santos, Lead Hygienist and Culture Champion at the same GP practice. Also serves as the content champion.

**7:15 AM — Morning Huddle**
- Attends the huddle in the break room. Sees her recognition shoutout on the display. Sees today's focus behavior: warm handoffs.

**7:25 AM — Champion Hub check (PWA, 2 minutes)**
- Opens **Culture OS > Champions > My Dashboard**.
- Culture scorecard: She led 4 huddles this week (streak: 12 consecutive weeks). Her team's H-Score trend is positive.
- Content capture stats (from Content Engine): She captured 6 pieces of content last week. Her streak counter shows 8 consecutive weeks of 5+ captures. She's ranked #3 in her regional cohort.
- This week's culture mission: "Focus on trust transfers between hygienist and front desk during checkout."
- This week's content mission (from Content Engine brief): "Capture at least 1 patient testimonial video — the practice hasn't had one in 3 weeks."

**During the day — Content capture moments**
- Between patients, she captures a quick 30-second testimonial video from a happy patient (with consent). She submits it via the Capture App.
- The Content Engine processes it: stabilization, Whisper transcription, OpusClip highlight extraction, draft captions.

**Weekly — CCA Academy (20 minutes)**
- On her lunch break on Tuesdays, she opens **CCA Academy** on her phone.
- She's working on Silver Track Module 3: "Advanced trust transfer techniques."
- She watches a 5-minute Grant training video, completes a knowledge check, and sees her progress bar advance to 78%.
- Her Credly badge for Bronze certification is already on her LinkedIn profile.

**Weekly — Trust Transfer review**
- Her office manager shares the **Trust Transfer Tracker** scores from this week. Maria's warm handoff scores from Cinnamon call analysis are above practice average. One area to improve: she sometimes forgets to mention the patient's upcoming procedure during the handoff to the front desk at checkout.

---

### Journey 6: Front Desk Staff — Daily Workflow

**Persona**: Jennifer Park, Front Desk Coordinator. Works at the front desk station with a desktop computer and a practice phone.

**Throughout the day — Call handling**
- Her primary interface is the **Call Dashboard** (Practice Operations, web on desktop).
- When a call comes in, the dashboard shows a **context card**: caller ID matched to patient record, last visit date, upcoming appointments, preferred name, communication history.
- For known patients, Jennifer greets them by name and can see relevant context instantly.
- For new patient calls, the system flags it as "New Lead" and starts the funnel tracking.
- When she's on another call or away from the desk, the **Voice AI** picks up. It answers in a natural voice, does real-time PMS lookup, and can book appointments directly.

**Between calls — Queue management**
- She checks the **Patient Outreach Queue** for her assigned items:
  - Appointment confirmations that need phone follow-up (patients who didn't respond to automated text)
  - Rebooking calls for patients overdue for cleanings
- The AI has drafted talking points for each call. She uses them as guides.

**End of day — Missed call recovery**
- The **Missed Call Recovery Queue** shows 3 calls that went to voicemail (not answered by Voice AI due to line capacity).
- The Automation Engine has already sent a "Sorry we missed your call" text to each. Two responded with booking requests. Jennifer completes the bookings.

**Weekly — Playbook review**
- Her office manager mentions the new phone script from HQ. Jennifer opens the **Playbook Viewer** in Practice Operations and reads the updated script for handling insurance questions. The playbook is versioned — she can see what changed from the previous version.

---

### Journey 7: Patient — Appointment Lifecycle

**Persona**: David Kim, age 42. Hasn't been to the dentist in 3 years. Finds SGA practice via Google.

**Discovery**
- Searches "dentist near me" on Google. Finds "Smile Center Buckhead" on Google Maps.
- The **Practice Website** (generated by Practice Platform, feature 1.04) loads. Clean, specialty-appropriate template. Recent before/after photos (from Content Engine Asset Registry, published via Storefront Manager). 4.7 star rating (from Swell, surfaced on site). Online booking button prominent.
- He clicks "Book Appointment." The form pre-fills with basic info from his Google profile.

**Pre-visit**
- 3 days before his appointment, he receives an **SMS confirmation** (from Automation Engine): "Hi David, this is Smile Center Buckhead. Looking forward to seeing you Thursday at 10am. Reply YES to confirm or RESCHEDULE to pick a new time."
- He replies YES.
- 1 day before, he receives an **email** with a link to **Digital Forms**. He completes intake on his phone in 8 minutes.

**Day of visit**
- He arrives. Jennifer at the front desk greets him by name (she saw the context card showing "New patient, first visit in 3 years, confirmed via text yesterday").
- Maria the hygienist introduces herself warmly, mentions his dog (from the intake form's "tell us about yourself" field — a CCA hospitality technique from Culture OS training).
- After the cleaning, Maria does a warm handoff to Jennifer at the front desk: "David, Jennifer is going to help you get your next cleaning scheduled. Jennifer, David was great today — Dr. Patel wants to see him back in 6 months for a follow-up on that upper right area." (Trust Transfer behavior, scored by Cinnamon.)

**Post-visit**
- 2 hours after checkout, David receives an **SMS**: "Thanks for visiting us today, David! How was your experience? We'd love your feedback." Links to a Google review.
- He leaves a 5-star review. The system logs it, Swell captures it, the H-Score updates.
- 5 days later, he receives an **email** from Dr. Patel (AI-generated, warm tone, PHI-validated): treatment recommendations with a clear call to action to schedule the follow-up.
- All of David's interactions are tracked in the 5-stage funnel: Lead Generated (Google search) → Connected (booked online) → Booked (confirmed via SMS) → Shows Up (checked in) → Production (treatment pending — follow-up sequence active).

---

## Section 4: The WS1 Split — Practice Operations vs. Network Intelligence

### Why Two Surfaces, Not Two Products

Practice Operations and Network Intelligence are not separate products — they are two views of the same data. They share:
- The same Practice Data Hub (PostgreSQL)
- The same Funnel Data Lake (ClickHouse)
- The same AI Analysis Engine
- The same Integration Middleware
- The same authentication and permission model

They are separated because the users, tasks, devices, and cadences are fundamentally different:

| Dimension | Practice Operations | Network Intelligence |
|-----------|-------------------|---------------------|
| Users | Office managers, front desk, clinical staff | CEO, VP Ops, regional directors, HQ ops |
| Device | iPad/phone (PWA), front desk desktop | Desktop (large screen) |
| Cadence | Real-time, throughout the day | Morning review, weekly analysis |
| Task type | Execute (send this message, capture this photo, answer this call) | Analyze and decide (which practices need help, what patterns exist, what to change) |
| Data direction | Write-heavy (creating messages, capturing content, logging calls) | Read-heavy (dashboards, reports, drill-downs) |
| Scope | Single practice | 1-260+ practices |

### Feature Allocation

#### Practice Operations Features (18 features)

| # | Feature | Rationale for Practice Operations |
|---|---------|----------------------------------|
| 1.01 | Practice Data Hub | Practice staff read and update their own practice's data |
| 1.04 | 5 specialty website templates | Office manager manages their practice's website |
| 1.05 | Concierge Agent per practice | The agent serves the practice directly |
| 1.06 | Channel sync engine | Storefront publication is a practice-level action |
| 1.08 | Change management (cascading) | Practice-initiated changes cascade from here |
| 1.16 | Automation Engine | Office managers manage their practice's automations |
| 1.18 | Practice Nerve Center | The daily agenda IS Practice Operations |
| 1.20 | Service-type-aware automation | Automation configuration per practice |
| 1.22 | Practice Agent Instance | The per-practice agent lives here |
| 1.24 | Message Generator | Message drafting for practice-level communications |
| 1.25 | Voice AI Agent | Handles calls at the practice level |
| 1.27 | Graduation Manager | Office managers control trust levels for their practice |
| 1.28 | NexHealth Synchronizer | PMS data flows into the practice view |
| 1.29 | Channel Router | Channel selection per patient at the practice level |
| 1.30 | Escalation Manager | Escalations surface to the practice's office manager |
| 1.31 | Compliance Auditor | Practice-level compliance view (read-only) |
| 1.12 | 5-stage funnel model | The funnel model powers the daily agenda's action cards |
| 1.23 | Playbook Engine | Practice staff consume playbooks (read + execute) |

#### Network Intelligence Features (12 features)

| # | Feature | Rationale for Network Intelligence |
|---|---------|-----------------------------------|
| 1.02 | Silent public data harvesting | Enrollment function, HQ-initiated |
| 1.03 | Practice enrollment pipeline | HQ manages new practice onboarding |
| 1.07 | Blitz deploy safety net | HQ-level enrollment policy |
| 1.09 | Network health dashboard | Network-wide view, executive audience |
| 1.10 | QC queues and batch operations | HQ batch actions across practices |
| 1.11 | Opinionated agent with reasoning | AI recommendations for HQ decision-making |
| 1.13 | 15-platform integration middleware | Infrastructure managed by HQ ops |
| 1.14 | Funnel Data Lake | Analytics infrastructure, serves dashboards |
| 1.15 | AI Analysis Engine | Powers anomaly detection, root cause, rankings |
| 1.17 | Executive Dashboard | The core of Network Intelligence |
| 1.19 | HQ Ops queue | HQ prioritization tool |
| 1.32 | HQ Playbook Manager | HQ designs and deploys playbooks |

#### Features That Span Both (5 features)

| # | Feature | Practice Operations Role | Network Intelligence Role |
|---|---------|-------------------------|--------------------------|
| 1.01 | Practice Data Hub | Practice staff edit their practice profile, view their data | HQ views and manages all practice profiles, handles enrollment data entry |
| 1.12 | 5-stage funnel model | Powers the daily action cards ("these patients are at stage 3, follow up") | Powers the funnel visualization, cohort analysis, and conversion metrics |
| 1.16 | Automation Engine | Office managers see their practice's active sequences, approve/pause | HQ designs automation playbooks, views network-wide automation performance |
| 1.21 | Learning loop | Practice-level optimization ("your confirmation texts work better at 4pm") | Network-level learning ("practices under 50 patients/month benefit from different cadence") |
| 1.23 | Playbook Engine | Practice staff consume and execute playbooks | HQ staff design, version, test, and deploy playbooks |

### Shared Data Layer Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    SHARED DATA LAYER                     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Practice Data Hub (PostgreSQL)                     │  │
│  │  - practice_profiles                                │  │
│  │  - provider_roster                                  │  │
│  │  - patient_records (PHI-protected)                  │  │
│  │  - automation_configurations                        │  │
│  │  - playbook_assignments                             │  │
│  │  - enrollment_status                                │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Funnel Data Lake (ClickHouse)                      │  │
│  │  - funnel_events (timestamped stage transitions)    │  │
│  │  - communication_events (sends, deliveries, responses)│
│  │  - call_events (calls, durations, outcomes)         │  │
│  │  - automation_performance (per-sequence metrics)    │  │
│  │  - content_events (captures, publications, engagement)│
│  │  - h_score_history (daily snapshots from Culture OS)│  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  AI Analysis Engine (Python services)               │  │
│  │  - anomaly_detection (per-practice baselines)       │  │
│  │  - root_cause_analysis (cross-signal correlation)   │  │
│  │  - ranking_engine (revenue impact scoring)          │  │
│  │  - learning_loop (weekly optimization cycles)       │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Integration Middleware (Node.js services)          │  │
│  │  - NexHealth connector (real-time PMS)              │  │
│  │  - Sikka ONE connector (batch PMS)                  │  │
│  │  - Channel sync (GMB, directories, review platforms)│  │
│  │  - Twilio gateway (SMS, voice, WhatsApp)            │  │
│  └────────────────────────────────────────────────────┘  │
└──────────┬──────────────────────────────┬────────────────┘
           │                              │
           ▼                              ▼
┌─────────────────────┐      ┌─────────────────────────┐
│  PRACTICE           │      │  NETWORK                 │
│  OPERATIONS          │      │  INTELLIGENCE            │
│                     │      │                          │
│  Scope: 1 practice  │      │  Scope: 1-260+ practices │
│  Users: Practice    │      │  Users: HQ + Leadership  │
│  staff              │      │                          │
│  Device: Mobile/    │      │  Device: Desktop         │
│  tablet first       │      │  (large screen)          │
│                     │      │                          │
│  Read own data      │      │  Read all data           │
│  Write own data     │      │  Write config/playbooks  │
│  Execute actions    │      │  Analyze & decide        │
└─────────────────────┘      └─────────────────────────┘
```

**Permission model**: Role-Based Access Control (RBAC) enforced at the API layer.
- Practice staff authenticate with practice-scoped tokens. They can only read/write data for their assigned practice.
- HQ/leadership staff authenticate with network-scoped tokens. Read access to all practices; write access scoped by role (VP Ops can assign interventions but not modify playbooks; HQ Ops can modify playbooks but not change enrollment policy).

---

## Section 5: Cross-Workstream Touchpoints

These are the moments where data or actions from one workstream appear inside another workstream's interface. Each must feel native, not bolted on.

### Touchpoint Map

| # | Where It Appears | What the User Sees | Source Workstream | Target Surface | Design Treatment |
|---|-----------------|--------------------|--------------------|---------------|-----------------|
| T-1 | Practice Ops: Morning Briefing | H-Score trend widget (small card showing score + 7-day sparkline + focus behavior) | Culture OS (WS3) | Practice Operations | Embedded card in the Today view. Not a link to another app. Shows score, trend, and today's focus behavior in 3 lines. Tapping it opens a modal with H-Score detail — still within Practice Operations. |
| T-2 | Practice Ops: Morning Briefing | Content capture brief as Action Card #3 | Content Engine (WS2) | Practice Operations | Action Card with camera icon. "2 implant cases today — capture checklist ready." Tapping opens the Capture App PWA overlay. The user never leaves Practice Operations visually. |
| T-3 | Practice Ops: Storefront Manager | Asset Gallery showing approved assets from Content Engine | Content Engine (WS2) | Practice Operations | Gallery grid within Storefront section. Assets are pre-filtered to this practice and approved status. "Use on Website" and "Push to GMB" buttons. The user does not know these assets came from a different workstream. |
| T-4 | Practice Ops: Morning Huddle | Impact Story showing funnel data tied to culture behaviors | Practice Platform (WS1) | Culture OS display | A section of the huddle display that shows: "Our show rate improved 5% since focusing on confirmation calls." Data comes from the Funnel Data Lake. The Culture OS formats it as a motivational story. |
| T-5 | Practice Ops: End-of-Day Scorecard | Content captures submitted count | Content Engine (WS2) | Practice Operations | Line item in the scorecard: "2 content captures submitted." Simple count, no complexity. |
| T-6 | Network Intelligence: Network Overview KPI Bar | Network H-Score average | Culture OS (WS3) | Network Intelligence | Fifth KPI in the bar, alongside Production, New Patients, Show Rate, Treatment Acceptance. Same visual weight. Same interaction (click to drill down). |
| T-7 | Network Intelligence: Portfolio Heatmap | H-Score as a selectable color layer | Culture OS (WS3) | Network Intelligence | Toggle in the heatmap control panel: "Color by: Composite Health / Funnel Performance / H-Score / Content Activity." When H-Score is selected, the map recolors based on Culture OS data. |
| T-8 | Network Intelligence: Practice Deep Dive | Culture Tab with full H-Score breakdown, CCA status, huddle completion, trust transfer | Culture OS (WS3) | Network Intelligence | A full tab in the practice deep dive. Data is read from the Culture OS via the shared data layer. The tab looks and feels identical to other tabs — same typography, same chart styles, same interaction patterns. |
| T-9 | Network Intelligence: Practice Deep Dive | Digital Presence Tab with brand health score and content capture frequency | Content Engine (WS2) | Network Intelligence | Two metrics within the Digital Presence tab. Brand health score appears as a gauge (0-100). Content capture frequency appears as a sparkline with weekly counts. |
| T-10 | Network Intelligence: QC Queues | Brand deviation flags from Content Engine | Content Engine (WS2) | Network Intelligence | Queue items tagged with "Brand" category. The flag includes the violation detail and a direct link to the replacement asset in the Asset Registry. |
| T-11 | Content Engine: Champions section | Champion roster and engagement scores from Culture OS | Culture OS (WS3) | Content Engine | The Champion Roster in the Content Engine is populated from Culture OS champion data. Engagement scores show both content capture stats (native to Content Engine) and culture champion stats (from Culture OS) side by side. |
| T-12 | Content Engine: Distribution | Appointment data driving daily capture briefs | Practice Platform (WS1) | Content Engine | The Prompt Engine reads tomorrow's appointment schedule from the shared PMS integration layer. This data is invisible to Content Engine users — it's infrastructure that drives the capture brief content. |
| T-13 | Content Engine: Analytics | Attribution analysis (content → booking correlation) | Practice Platform (WS1) | Content Engine | A section in Analytics showing: "Content published on [date] correlated with [X]% increase in new patient inquiries for [practice]." The booking/funnel data comes from the Practice Platform. |
| T-14 | Culture OS: Morning Huddle | Impact Story using funnel data | Practice Platform (WS1) | Culture OS | Same as T-4. The huddle's impact story segment reads funnel metrics from the Practice Platform's weekly batch (Integration 7). |
| T-15 | Culture OS: Champion Hub | Content capture stats alongside culture scorecard | Content Engine (WS2) | Culture OS | The Champion Dashboard shows two scorecards side by side: "Culture Score" (huddles led, behaviors modeled) and "Content Score" (captures submitted, quality rating, streak). Both feel like one unified champion experience. |
| T-16 | Culture OS: Recognition Feed | Brand-compliant recognition post templates | Content Engine (WS2) | Culture OS | Recognition posts use templates and brand assets from the Content Engine Asset Registry (Integration 9). The user creates a recognition post; the system auto-applies brand-correct formatting. |

### Seam Design Principles

1. **Data flows via the Kafka event bus; UI never links between workstreams.** Users do not click a link in Practice Operations that opens a Content Engine page. Instead, Content Engine data is embedded within Practice Operations views.

2. **Stale data indicators.** When cross-workstream data is older than expected (e.g., H-Score hasn't updated in 48 hours), the widget shows a subtle "Last updated 2 days ago" label. No blocking errors — just transparency.

3. **Graceful degradation.** If the Culture OS is temporarily unavailable, the H-Score widget in Network Intelligence shows the last known value with a "Last updated" timestamp. It does not show an error. It does not break the dashboard layout.

4. **Cross-workstream data is read-only in the consuming surface.** You cannot edit H-Score configuration from Network Intelligence. You can view it. If you need to change it, the system provides a clear path: "Manage H-Score settings" links to Culture OS Admin Console (opening in a new tab — this is the one exception to the "no cross-linking" rule, and it's an admin action, not a daily workflow).

---

## Section 6: Mobile Strategy

### Design Decision Framework

The mobile strategy is determined by three questions for each interaction:
1. **Does it require a native device capability?** (Camera, push notifications, background sync) → Native/PWA
2. **Is it a brief, asynchronous interaction?** (Confirm something, submit something, read a notification) → SMS
3. **Is it a sustained work session?** (Data analysis, content editing, pipeline management) → Desktop web only

### SMS-Only Interactions

| Interaction | Why SMS | Content |
|---|---|---|
| Daily content capture brief | Zero-friction delivery to champions who may not open an app. Reply-to-capture workflow. Champions are clinical staff — they have seconds between patients, not minutes. | "Good morning! Today's capture brief: 2 implant cases (Dr. Patel, 10am + 2pm). Before/after photos ideal. Reply SKIP to pass." |
| Patient appointment confirmations | Patients respond to SMS at 3-5x the rate of email. HIPAA-compliant via Twilio BAA. Minimal PHI in message. | "Hi David, this is Smile Center Buckhead. Confirming your appointment Thu 10am with Dr. Patel. Reply YES to confirm or RESCHEDULE." |
| Patient follow-up messages | Same channel as confirmation — maintains conversation thread. Patients don't download apps for their dentist. | "Hi David, Dr. Patel wanted to check in after your visit last week. How are you feeling? We'd love to help schedule your next step." |
| Patient reactivation outreach | Reaching lapsed patients who haven't visited in 6+ months. SMS has the highest open rate for re-engagement. | "Hi David, it's been a while since your last visit at Smile Center Buckhead. We'd love to see you again. Reply BOOK to schedule." |
| Patient review requests | Post-visit prompt while the experience is fresh. SMS links to Google/Swell review page. | "Thanks for visiting us today, David! We'd love your feedback: [link]. It takes 30 seconds." |
| Missed call recovery | Automated follow-up within minutes of a missed call. Patient is still in the moment. | "Sorry we missed your call! We're here to help. Reply CALLBACK for us to call you back, or BOOK to schedule online: [link]." |
| Champion streak reminders | Nudge to maintain capture streak. Brief, motivational. | "Hey Maria! You're on a 12-week capture streak. Don't break it today — you've got 1 implant case at 2pm." |

**Why not an app for these?** These interactions are 10-30 seconds each. Asking a hygienist to open an app, authenticate, navigate to a screen, and tap a button adds 45-60 seconds of friction. SMS eliminates that friction entirely. The patient interactions are even more clear-cut — patients will not download an app for their dental office.

### PWA Interactions

A single PWA (Progressive Web App) serves as the mobile interface for practice staff across all three workstreams. It is installed from the web browser and lives on the home screen like a native app.

**Why PWA instead of native:**
- Staff use a mix of iOS and Android personal devices. One codebase covers both.
- No App Store approval delays for updates. Deploy daily if needed.
- Offline capability via service workers for core functions (viewing today's agenda, playbook access).
- Push notifications via Web Push API (supported on both iOS 16.4+ and Android).
- Camera access via the MediaDevices API (sufficient for photo/video capture).
- No download barrier — staff visit a URL and "Add to Home Screen."

**PWA Screens:**

| Screen | Primary Users | What It Does | Offline Capable? |
|---|---|---|---|
| **Today** (Practice Operations mobile) | Office Managers, Front Desk | Morning briefing, 3 action cards, patient outreach queue (approve/reject messages), end-of-day scorecard | Partial — agenda loads offline, message sending requires connection |
| **Capture** (Content Engine mobile) | Champions, Office Managers | Camera interface with overlay guides, before/after pairing, video recording, batch upload to Content Engine | Yes — captures stored locally, uploaded when online |
| **Huddle** (Culture OS mobile) | All Practice Staff | View today's huddle agenda on personal device (supplement to TV display), mark huddle as completed | Yes — agenda cached nightly |
| **Academy** (Culture OS mobile) | Clinical Staff, Office Managers | Watch training videos, complete modules, view progress, take assessments | Partial — videos require connection, progress syncs when online |
| **Champion Hub** (Culture OS + Content Engine mobile) | Champions | View combined scorecard, current missions, streak counter, regional leaderboard | Yes — scorecard cached, leaderboard updates on connection |
| **Playbooks** (Practice Operations + Culture OS mobile) | All Practice Staff | Read current playbooks, phone scripts, behavior guides | Yes — playbooks cached for offline access |
| **Calls** (Practice Operations mobile) | Office Managers | Review Voice AI transcripts, listen to recordings, manage escalations (not primary call handling — that's desktop) | No — requires connection |

**PWA Architecture:**

```
┌─────────────────────────────────────────┐
│           SGA Practice PWA               │
│                                          │
│  Bottom Navigation:                      │
│  [ Today ] [ Capture ] [ Huddle ]        │
│  [ Academy ] [ More... ]                 │
│                                          │
│  "More" expands to:                      │
│  - Champion Hub                          │
│  - Playbooks                             │
│  - Calls                                 │
│  - Storefront                            │
│  - Settings                              │
└─────────────────────────────────────────┘
```

The PWA presents a unified experience. Practice staff do not know they are using features from 3 different workstreams. The bottom navigation groups by task, not by system.

**PWA Technical Requirements:**
- Service Worker for offline caching (agenda, playbooks, scorecards)
- IndexedDB for local capture storage before upload
- Web Push API for notifications (capture prompts, escalation alerts, huddle reminders)
- MediaDevices API for camera access (captures)
- Background Sync API for deferred uploads when connectivity returns

### Native App: Not Recommended for v1

**Why no native app at launch:**
- PWA covers 100% of mobile use cases identified in user journeys.
- Native app development doubles the mobile engineering effort (iOS + Android codebases, or React Native with native module maintenance).
- App Store approval creates a 1-2 week deployment lag that conflicts with the rapid iteration needed in the first year.
- The primary native-only capability that PWA lacks — background location tracking — is not needed for any use case.
- Camera quality via the MediaDevices API is sufficient for the content capture use case (dental photography is not medical imaging — it's social media content).

**When to reconsider native (Phase 2+):**
- If content capture quality requirements escalate (RAW photo processing, advanced camera controls)
- If offline-first workflows expand significantly beyond current scope
- If push notification reliability on iOS PWA proves insufficient (currently stable on iOS 16.4+, but Apple's commitment to Web Push is evolving)
- If testimonial kiosk hardware integration (Insta360 cameras, feature 2.24) requires native Bluetooth APIs

The Content Engine's React Native app (feature 2.22, originally planned for Phase 2) becomes the candidate for native development if/when these triggers are hit. It would be a content-capture-specific native app, not a full platform native app.

### Mobile vs. Desktop Matrix

| Actor Type | Primary Device | Primary Interface | Mobile Interactions | Desktop Interactions |
|---|---|---|---|---|
| CEO/Executives | Desktop (large screen) | Network Intelligence | None — dashboards are desktop-first. If mobile access needed, Network Intelligence is responsive but optimized for desktop. | All Network Intelligence screens |
| VP Ops / Regional Directors | Desktop + iPad | Network Intelligence | Portfolio Heatmap on iPad during site visits. Practice Deep Dive on iPad for in-practice conversations. | Full Network Intelligence, Analytics Explorer |
| HQ Operations Team | Desktop (dual monitor) | Network Intelligence | None — HQ ops work is desktop-intensive | HQ Standup Board, QC Queues, Enrollment Pipeline, Playbook Manager |
| Office Managers | iPad (practice) + iPhone (personal) | Practice Operations PWA | Today view, Patient Outreach Queue (approve messages), Content Capture, Huddle view | Storefront Manager, Automation Status Board, detailed patient records |
| Clinical Staff | iPhone (personal) | PWA + SMS | Content Capture, Champion Hub, Academy modules, Huddle view, SMS capture prompts | CCA Academy (longer training sessions) |
| Front Desk Staff | Desktop (front desk station) | Practice Operations Web | Playbook Viewer (quick reference on phone) | Call Dashboard, Patient Outreach Queue, scheduling |
| Patients | Personal smartphone | SMS + Web | SMS conversations (confirmations, follow-ups), Practice Website (mobile-responsive), Digital Forms, Online Booking | None expected |

### Twilio Channel Ownership Resolution (QC-1 Item #3, addressed here)

All Twilio usage is managed by a **shared Twilio Gateway Service** in the infrastructure layer. This service owns:
- Phone number provisioning and assignment per practice
- Message routing (inbound messages are classified and routed to the correct workstream handler)
- Opt-in/opt-out compliance (centralized, not per-workstream)
- Rate limiting and cost monitoring

**Routing rules:**
- Inbound patient messages (appointment-related, general inquiries) → Practice Platform (WS1) handler
- Inbound staff messages (photo/video submissions, capture responses) → Content Engine (WS2) handler
- Outbound patient messages (confirmations, follow-ups) → Practice Platform (WS1) composes, Twilio Gateway sends
- Outbound staff messages (capture briefs, champion nudges) → Content Engine (WS2) composes, Twilio Gateway sends

**Phone number strategy:**
- Each practice has 1 primary Twilio number for patient communications (owned by WS1)
- Content capture briefs go to champions' personal phones from a separate HQ-level number (owned by WS2)
- This separation prevents patient-facing and staff-facing messages from colliding on the same thread

---

## Section 7: The Asset Registry — WS1/WS2 Handoff Contract (QC-1 Item #2)

This section defines the explicit interface between the Content Engine (WS2) and Practice Operations' Storefront Manager (WS1).

### The Asset Registry

The Asset Registry is a shared data store (S3 + PostgreSQL metadata) that serves as the handoff point between content production and content publication.

**Ownership:** Content Engine (WS2) owns the Asset Registry. It writes assets into it. Practice Platform (WS1) reads from it.

### Asset Lifecycle

```
Content Engine (WS2) domain          │  Shared Asset Registry  │  Practice Operations (WS1) domain
                                     │                         │
Champion captures photo ──────────→  │                         │
AI processes (crop, color) ────────→ │                         │
HQ Content reviews ────────────────→ │                         │
                                     │                         │
Approved? ─── YES ──→ Writes asset   │  ┌──────────────────┐  │
              │       to registry ──→│  │ Asset Record:     │  │
              │                      │  │ - asset_id        │  │
              NO                     │  │ - practice_id     │  │
              │                      │  │ - brand_universe   │  │
              ▼                      │  │ - asset_type      │  │
         Rejected                    │  │ - status: approved│  │
         (feedback to champion)      │  │ - s3_url          │  │
                                     │  │ - variants[]      │  │
                                     │  │ - metadata{}      │  │
                                     │  │ - created_at      │  │
                                     │  │ - approved_by     │  │
                                     │  └──────┬───────────┘  │
                                     │         │              │
                                     │         ▼              │
                                     │  Kafka event:          │  Office Manager opens Storefront
                                     │  content.asset.        │  Manager, sees new asset in
                                     │  published ──────────→ │  Asset Gallery.
                                     │                        │
                                     │                        │  Clicks "Use on Website" or
                                     │                        │  "Push to GMB" ──→ Publication
                                     │                        │  triggered by WS1.
                                     │                        │
                                     │                        │  WS1 Channel Sync Engine
                                     │                        │  publishes to destination.
```

### Contract Rules

1. **Content Engine writes; Practice Platform reads.** WS2 is the sole writer to the Asset Registry. WS1 never writes new assets — it only reads approved assets and triggers publication to external channels.

2. **Publication trigger is WS1's responsibility.** When a Content Engine asset is approved and deposited in the registry, it is NOT automatically published to the practice website or GMB. An office manager (or HQ ops via batch) must explicitly select assets for publication. This gives practice-level control over what appears on their digital presence.

3. **Exception: Social distribution is WS2's responsibility.** Content Engine distributes directly to social channels (Instagram, Facebook, LinkedIn, TikTok, YouTube, Google Business Profile posts). These are ephemeral content (social posts). Practice website and GMB listing content (persistent, storefront-defining) requires WS1 publication.

4. **Asset metadata is standardized.** Every asset in the registry includes: practice_id, brand_universe (SGA/Gen4/MODIS), asset_type (before-after, testimonial, team-photo, social-post, flyer, tv-slide), platform_variants (sizes/crops per platform), and approval metadata.

5. **Staleness tracking.** The Practice Platform can query the Asset Registry to determine if a practice's storefront is using stale assets (e.g., provider photo older than 6 months while newer approved photos exist). This powers the Digital Health Score's "content freshness" sub-score.

---

## Appendix A: Interface-to-Feature Traceability

This maps every interface in Section 1 to the feature numbers from the consolidation feature map.

| Interface | Features |
|-----------|----------|
| PO-1: Practice Daily Agenda | 1.18, 1.12, 1.15 |
| PO-2: Patient Outreach Queue | 1.24, 1.16, 1.29, 1.30 |
| PO-3: Call Dashboard | 1.25, 1.05, 1.28 |
| PO-4: Voice AI Monitor | 1.25, 1.30 |
| PO-5: Automation Status Board | 1.16, 1.23, 1.27, 1.20 |
| PO-6: Storefront Manager | 1.04, 1.06, 1.08, Asset Registry (WS2) |
| PO-7: Content Capture Prompt | 2.14, 2.15 (via integration) |
| PO-8: Morning Huddle Screen | 3.01 (via integration) |
| PO-9: Playbook Viewer | 1.23, 3.07 (via integration) |
| NI-1: Executive Dashboard | 1.17, 1.09, 1.15 |
| NI-2: Portfolio Heatmap | 1.09, 3.10 (via integration) |
| NI-3: Intervention Queue | 1.19, 1.15 |
| NI-4: HQ Ops Standup Board | 1.19, 1.10 |
| NI-5: Practice Deep Dive | 1.01, 1.12, 1.14, 1.15, 3.03, 3.10, 2.10 (via integrations) |
| NI-6: Funnel Analytics Explorer | 1.14, 1.15, 1.12 |
| NI-7: Enrollment Pipeline | 1.02, 1.03, 1.07 |
| NI-8: HQ Playbook Manager | 1.32, 1.23 |
| NI-9: Compliance Dashboard | 1.31 |
| NI-10: Network Learning Report | 1.15, 1.21, 1.26 |
| CE-1: Content Capture Bot | 2.14 |
| CE-2: Capture App | 2.22, 2.16 |
| CE-3: Editorial Queue | 2.06, 2.18, 2.26 |
| CE-4: Studio Request Portal | 2.01, 2.02, 2.03 |
| CE-5: Creative Director Review | 2.07 |
| CE-6: Asset Registry | 2.05, 2.12, 2.10, 2.26 |
| CE-7: Content Engine Dashboard | 2.11, 2.10 |
| CE-8: Distribution Monitor | 2.08, 2.23, 2.09 |
| CE-9: Event Mode Console | 2.27 |
| CO-1: Morning Huddle Display | 3.01 |
| CO-2: CCA Academy Portal | 3.02 |
| CO-3: Champion Hub | 3.05, 3.06 |
| CO-4: Recognition Feed | 3.06 |
| CO-5: H-Score Dashboard | 3.03, 3.09 |
| CO-6: Playbook Library | 3.07, 3.12, 3.08 |
| CO-7: Trust Transfer Tracker | 3.04, 3.13 |
| CO-8: Culture Admin Console | 3.07, 3.12, 3.05, 3.06, 3.11 |
| PF-1: Practice Website | 1.04 |
| PF-2: Voice AI (Phone) | 1.25 |
| PF-3: SMS Conversations | 1.16, 1.24, 1.29 |
| PF-4: Email Communications | 1.16, 1.24 |
| PF-5: Review Request | 1.16 (automation trigger) |
| PF-6: Digital Forms | 1.05 (enrollment workflow) |

---

## Appendix B: QC-1 Revision Traceability

| QC-1 Item | Severity | Section Addressed | Summary of Fix |
|-----------|----------|-------------------|----------------|
| #1 | HIGH | Section 4 | WS1 split into Practice Operations (18 features, practice staff) and Network Intelligence (12 features, HQ/leadership) with 5 spanning features and a shared data layer. No tabs — two distinct surfaces with distinct navigation, device targets, and user mental models. |
| #2 | HIGH | Section 7 | Explicit Asset Registry contract: WS2 writes approved assets, WS1 reads and triggers publication. Social distribution owned by WS2; storefront publication owned by WS1. Kafka event notification on asset approval. |
| #3 | MEDIUM | Section 6 (Twilio subsection) | Shared Twilio Gateway Service in infrastructure layer. Separate phone numbers for patient comms (WS1) vs. staff capture briefs (WS2). Centralized opt-in/opt-out compliance. Inbound routing by message classification. |
| #6 | MEDIUM | Section 6 | Comprehensive mobile strategy: SMS for brief async interactions (captures, patient comms), PWA for sustained mobile workflows (daily agenda, capture app, academy, champion hub), no native app v1 with clear triggers for when to reconsider. |

---

*Experience Architecture designed by Product Designer Agent | 2026-03-14*
*For UX wireframing handoff: every interface in Section 1 has a navigation structure in Section 2, usage context in Section 3, and cross-workstream data sources in Section 5.*
