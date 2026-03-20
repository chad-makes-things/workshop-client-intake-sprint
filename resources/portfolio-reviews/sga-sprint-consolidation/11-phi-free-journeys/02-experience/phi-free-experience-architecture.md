# PHI-Free Experience Architecture — SGA Dental Partners

**Date**: 2026-03-20
**Author**: Product Designer Agent (Phase A Redesign)
**Scope**: All 3 workstreams, 36 interfaces (42 original minus 6 patient-facing)
**Constraint**: Zero patient health information in Phase A
**Companion Documents**: [PHI Data Flow Analysis](../../10-phi-analysis/phi-data-flow-analysis.md) | [Original Experience Architecture](../../03-experience/experience-architecture.md)

---

## 1. Design Principles for the PHI-Free Phase

### 1.1 Original Principles — Retained

The five original design principles remain in force:

1. **One user, one surface.** No actor type navigates between product surfaces.
2. **Progressive disclosure over feature density.** Show three things that matter now.
3. **The system initiates; the human decides.** AI generates; humans approve.
4. **Mobile is the default for practice staff; desktop is the default for HQ.** Design for the device where the work happens.
5. **Workstream boundaries are invisible to users.** Users see "my work."

### 1.2 New Principles — Phase A Specific

These six principles govern every Phase A design decision and ensure Phase B integration is clean:

6. **Design for the aggregate, not the individual.** Phase A works with practice-level scores, staff rosters, brand metrics, and enrollment status. No individual patient data appears anywhere. When a metric would normally show "Mrs. Patterson's implant consult," Phase A shows "3 treatment consults scheduled this week."

7. **Empty states are Phase B invitations, not broken features.** Every UI location where PHI features will eventually appear must show a purposeful empty state — not an error, not a hidden element. These states communicate: "This feature requires PMS integration, coming in Phase B." They are designed to be replaced by live data, not redesigned.

8. **No phantom navigation.** If a feature is not available in Phase A, its navigation item does not appear. Users never see a grayed-out "Patients" tab they cannot click. The navigation structure is complete and functional for Phase A — smaller, but fully usable.

9. **Aggregate data tells the operational story.** Phase A proves that practice-level aggregate metrics (H-Score from Cinnamon/Swell/Dental Intel, brand health from Content Engine, enrollment status from the pipeline) are sufficient for executive decision-making and practice culture management. Patient-level granularity enriches the story in Phase B but is not required for the story to begin.

10. **Architecture for addition, not subtraction.** Every Phase A component, API contract, navigation slot, and data model is designed so that Phase B features are added — new tabs appear, new widgets populate, new data sources light up. Nothing in Phase A is removed or restructured when PHI arrives. This means: tab containers support dynamic tab injection, dashboard grids have reserved slots, and component props accept optional PHI data sources.

11. **Staff data is not PHI.** Employee names, roles, certifications, training progress, performance scores, and HR data from iSolved are NOT protected health information. Phase A can freely display staff rosters, certification status, trust transfer scores, champion leaderboards, and team management features.

### 1.3 What Counts as PHI (Exclusion Boundary)

| Data Category | PHI? | Phase A Status |
|---------------|------|----------------|
| Patient names, DOB, SSN, insurance IDs | Yes | Excluded |
| Individual appointment data from PMS | Yes | Excluded |
| Call recordings / transcripts with patient references | Yes | Excluded |
| Patient photos (facial images = biometric identifier) | Yes | Excluded |
| Patient communication history (SMS, email) | Yes | Excluded |
| Patient funnel stage (individual) | Yes | Excluded |
| Practice-level aggregate scores (H-Score) | No | Included |
| Staff/employee data (iSolved HR) | No | Included |
| Brand assets, marketing content, social metrics | No | Included |
| Practice profile data (name, address, providers, hours) | No | Included |
| Enrollment pipeline status | No | Included |
| Content pipeline metrics (counts, not patient content) | No | Included |
| Aggregate production / new patient counts (practice-level, no patient identity) | Borderline | Included if sourced from practice-reported totals, not PMS record-level data |

---

## 2. Redesigned Interface Inventory — Phase A

### 2A. Practice Operations (WS1 Surface A) — Phase A

**Original**: 9 interfaces (PO-1 through PO-9)
**Phase A**: 5 interfaces survive; 4 are deferred to Phase B

| # | Interface | Phase A Status | Actor Types | Platform | Primary Use Cases (Phase A) | Data Sources (Phase A) | What Changed |
|---|-----------|---------------|-------------|----------|----------------------------|----------------------|--------------|
| PO-1 | **Practice Daily Agenda** | **Modified** | Office Managers | PWA, Web | View practice profile, H-Score trend, content capture brief, enrollment status updates. No patient-specific action cards. | Practice Data Hub (profile), Culture OS (H-Score via Kafka), Content Engine (capture brief), Enrollment Pipeline | **Major change**: Morning Briefing loses patient-driven action cards (confirmations, follow-ups, rebooking). Replaced with practice operations cards: H-Score focus, content capture opportunities, storefront health, enrollment milestones. End-of-Day Scorecard replaced with Practice Activity Summary (content captures, huddle completion, storefront updates). |
| PO-5 | **Automation Status Board** | **Deferred (Phase B)** | — | — | — | — | Automations are patient-communication sequences. No patient data = no automation targets. Entire interface deferred. |
| PO-6 | **Storefront Manager** | **Unchanged** | Office Managers, HQ Ops | Web | View/edit practice website, GMB listing, directory listings; pull approved assets from Asset Registry; trigger publication | Practice Data Hub, Asset Registry (WS2), Channel Sync Engine | **No change needed.** Storefront management uses practice profile data and brand assets — zero PHI. Digital Health Score works entirely from public data (website crawl, GMB listing status, directory sync). |
| PO-7 | **Content Capture Prompt** | **Modified** | Office Managers, Clinical Staff | SMS, PWA push | Receive daily capture brief for non-patient content: team photos, office tours, provider introductions, brand moments. No patient before/after photos. | Content Engine (capture briefs), Practice Data Hub (schedule data — non-PHI practice schedule, not patient appointments) | **Moderate change**: Capture briefs shift from procedure-driven ("2 implant cases today") to brand/culture-driven ("Capture a team photo for the website" or "Film a 30-second office tour"). Patient consent photography deferred to Phase B. |
| PO-8 | **Morning Huddle Screen** | **Unchanged** | Office Managers, Clinical Staff, Front Desk | TV display, PWA | View 5-minute huddle agenda: yesterday's H-Score, today's focus behavior, recognition shoutout, practice wins | Culture OS (H-Score, behaviors, recognition via Kafka), Practice Data Hub (practice profile) | **No change needed.** Huddle content is already PHI-free. H-Score is aggregate. Focus behaviors are generic culture guidance. Recognition is staff-to-staff. Impact stories shift from funnel metrics to culture/brand metrics in Phase A. |
| PO-9 | **Playbook Viewer** | **Unchanged** | Office Managers, Front Desk, Clinical Staff | PWA, Web | Access current playbooks (phone scripts, hospitality protocols, onboarding checklists), view version history | Playbook Engine (HQ-managed content) | **No change needed.** Playbooks are reference documents — they contain no patient data. |
| PO-2 | **Patient Outreach Queue** | **Deferred (Phase B)** | — | — | — | — | Patient outreach requires patient names, contact info, appointment context. Entirely PHI-dependent. Deferred. |
| PO-3 | **Call Dashboard** | **Deferred (Phase B)** | — | — | — | — | Call context cards require patient records from PMS. Call logging requires patient identification. Voice AI call handling is PHI-heavy. Deferred. |
| PO-4 | **Voice AI Monitor** | **Deferred (Phase B)** | — | — | — | — | Voice AI transcripts contain verbal PHI. Call recordings are the highest-risk PHI flow in the platform. Deferred. |

**Phase A Practice Operations Summary**: 5 of 9 interfaces ship. The daily workflow shifts from patient-centric operations (outreach, calls, automations) to practice-centric operations (storefront management, brand content, culture rituals, team development). This is a meaningful product on its own — it serves the "practice brand and culture management" use case that SGA needs before any patient touches the system.

---

### 2B. Network Intelligence (WS1 Surface B) — Phase A

**Original**: 10 interfaces (NI-1 through NI-10)
**Phase A**: 8 interfaces survive (2 deferred); several modified

| # | Interface | Phase A Status | Actor Types | Platform | Primary Use Cases (Phase A) | Data Sources (Phase A) | What Changed |
|---|-----------|---------------|-------------|----------|----------------------------|----------------------|--------------|
| NI-1 | **Executive Dashboard** | **Modified** | CEO/Executives | Web (desktop) | View network-wide aggregate KPIs: practice count, avg H-Score, avg brand health, enrollment pipeline status, content activity. Review AI alerts based on aggregate anomalies. | Practice Data Hub (profiles, enrollment), Culture OS (H-Score via Kafka), Content Engine (brand health, pipeline via Kafka), Enrollment Pipeline | **Major change**: KPI bar drops Total Production, New Patients, Show Rate, Treatment Acceptance (all require PMS funnel data). Replaced with: Active Practices, Avg Network H-Score, Avg Brand Health Score, Enrollment Pipeline Active, Content Captures This Week. AI Alerts refocused on culture/brand/enrollment anomalies. |
| NI-2 | **Portfolio Heatmap** | **Modified** | VP Ops, Regional Directors | Web | Geographic view of practices colored by H-Score, brand health, or enrollment status. No funnel performance data. | Practice Data Hub (locations), Culture OS (H-Score), Content Engine (brand health) | **Moderate change**: Color layer selector loses "Composite Health" (requires funnel data), "Funnel Performance," "Show Rate," and "Treatment Acceptance." Retains: H-Score, Brand Health/Content Activity. Gains: Enrollment Status (color by enrollment stage), CCA Certification Level (color by average team certification tier). |
| NI-3 | **Intervention Queue** | **Modified** | VP Ops, Regional Directors | Web | AI-ranked practices needing attention based on H-Score drops, brand health degradation, content capture drought, huddle non-completion. Ranked by culture impact, not revenue impact. | AI Analysis Engine (non-PHI signals), Culture OS, Content Engine, Practice Data Hub | **Moderate change**: Revenue impact ranking replaced with culture/brand impact scoring. Root cause analysis uses non-PHI signals: "H-Score dropped 12 points — Cinnamon phone scores down, no huddles logged in 2 weeks, content champion inactive." Revenue impact column replaced with "Culture Risk Score." |
| NI-4 | **HQ Ops Standup Board** | **Modified** | HQ Operations Team | Web | Prioritized practice queue (culture/brand/enrollment issues), batch operations for storefront updates, enrollment pipeline status, QC queues | Practice Data Hub, Culture OS, Content Engine, Enrollment Pipeline | **Moderate change**: Problem queue shifts from revenue-impact ranking to culture/brand/enrollment ranking. Batch operations panel retains storefront batch ops and playbook deployment. Loses patient-data-dependent batch operations. |
| NI-5 | **Practice Deep Dive** | **Modified** | VP Ops, HQ Ops, Regional Directors | Web | Single-practice view with available tabs: Profile, Culture (H-Score), Content (brand health), Enrollment Status, History. No Performance or Communications tabs. | Practice Data Hub, Culture OS (via Kafka), Content Engine (via Kafka), Enrollment Pipeline | **Major change**: 6-tab structure drops to 4 tabs in Phase A. Performance tab (funnel data) and Communications tab (patient comms) deferred. Digital Presence tab renamed to "Content & Brand" and retains brand health score, content pipeline status, asset counts — loses website health sub-scores that depend on patient booking data. Culture tab fully intact. New "Enrollment" tab for practices in onboarding. |
| NI-6 | **Funnel Analytics Explorer** | **Deferred (Phase B)** | — | — | — | — | Funnel analytics require PMS-sourced patient funnel data (Lead → Connected → Booked → Shows → Production). Entire interface is PHI-dependent. Deferred. |
| NI-7 | **Enrollment Pipeline** | **Unchanged** | HQ Ops | Web | Track new practice onboarding: data harvesting status, integration connectivity, blitz deploy countdown, go-live readiness | Practice Data Hub, Enrollment Service, Integration Middleware (connectivity status) | **No change needed.** Enrollment pipeline tracks practice-level onboarding status — no patient data. Silent data harvesting of public practice data is PHI-free. Integration connectivity checks (API health) are infrastructure-level. |
| NI-8 | **HQ Playbook Manager** | **Unchanged** | HQ Ops | Web | Design, version, and deploy playbook updates; view adoption metrics (which practices downloaded which version) | Playbook Engine, Practice Data Hub | **No change needed.** Playbooks are reference content. Adoption tracking uses practice-level data (did Practice X download Playbook v2.3?), not patient data. |
| NI-9 | **Compliance Dashboard** | **Modified** | HQ Ops (Compliance) | Web | Phase A: Integration health monitoring, consent configuration status, HIPAA readiness checklist (preparing for Phase B). No PHI access logs yet (no PHI to access). | Integration Middleware (health checks), Practice Data Hub (configuration) | **Significant change**: Reframed as "Compliance Readiness Dashboard." Instead of monitoring active PHI flows, it tracks preparation: BAA execution status per vendor, encryption configuration verification, audit logging infrastructure readiness, consent workflow configuration per practice. This becomes the Phase B launch checklist. |
| NI-10 | **Network Learning Report** | **Modified** | VP Ops, CEO | Web | Monthly AI-generated report on culture trends, brand health patterns, enrollment velocity, content effectiveness. No funnel/revenue optimization insights. | AI Analysis Engine (non-PHI signals), Culture OS, Content Engine, Enrollment Pipeline | **Moderate change**: Report content shifts from revenue optimization to culture/brand optimization. "What the system learned" focuses on: H-Score improvement patterns, content capture impact on brand health, playbook adoption correlation with culture scores, enrollment timeline optimization. |

**Phase A Network Intelligence Summary**: 8 of 10 interfaces ship. The strategic view shifts from "revenue performance and patient funnel health" to "culture performance, brand health, enrollment velocity, and operational readiness." Executives can still make meaningful decisions: which practices need culture intervention, where brand health is degrading, how fast new practices are onboarding, and whether the network is ready for Phase B PHI activation.

---

### 2C. Content Engine (WS2) — Phase A

**Original**: 9 interfaces (CE-1 through CE-9)
**Phase A**: 8 interfaces survive (1 deferred); 2 modified

| # | Interface | Phase A Status | Actor Types | Platform | Primary Use Cases (Phase A) | Data Sources (Phase A) | What Changed |
|---|-----------|---------------|-------------|----------|----------------------------|----------------------|--------------|
| CE-1 | **Content Capture Bot** | **Modified** | Office Managers, Clinical Staff (Champions) | SMS / WhatsApp | Receive daily capture brief for non-patient content: team photos, office environment, provider introductions, event coverage, community involvement. Submit via reply. | Content Engine Prompt Engine, Practice Data Hub (practice schedule — non-PHI) | **Moderate change**: Capture briefs no longer reference patient procedures ("2 implant cases today"). Instead: "Your office just got new signage — capture a photo!" or "It's Team Appreciation Week — get a group photo before huddle." No patient before/after photos. No procedure-driven prompts. |
| CE-2 | **Capture App** | **Modified** | Champions | PWA | Camera interface with overlay guides for team photos, office tours, provider headshots, event coverage. Before/after pairing disabled (requires patient consent workflow from Phase B). Video recording for office tours, testimonials from willing staff (not patients). | Content Engine (capture config), Practice Data Hub | **Moderate change**: Before/after photo pairing feature hidden (it requires patient consent and is inherently PHI-adjacent). Camera overlay guides adjusted for non-clinical content: headshot framing, office tour composition, event coverage. Video recording fully functional for non-patient content. |
| CE-3 | **Editorial Queue** | **Unchanged** | HQ Content Team | Web | Review AI-processed content: approve/reject/edit captions, review photo corrections, schedule distribution, manage content calendar | Content Engine pipeline (AI processing), Asset Registry | **No change needed.** The editorial queue processes whatever content enters the pipeline. In Phase A, that content is brand/team/office content instead of clinical before/after. The review workflow is identical. |
| CE-4 | **Studio Request Portal** | **Unchanged** | Office Managers, HQ Marketing | Web, Email, Teams Bot | Submit design requests (flyers, social posts, TV slides), track status, receive proofs, approve finals | Content Engine (request tracking) | **No change needed.** Studio requests are for marketing collateral — inherently PHI-free. |
| CE-5 | **Creative Director Review** | **Unchanged** | Creative Director | Teams, Web | 58-second tap-approve flow for brand-controlled collateral, batch review, brand deviation flags | Content Engine (approval queue) | **No change needed.** Brand review is PHI-free. |
| CE-6 | **Asset Registry** | **Unchanged** | HQ Content, HQ Ops, Office Managers | Web | Browse all approved assets by practice, brand universe, asset type, date; download originals; view usage history; check brand health score | Asset Registry (S3 + PostgreSQL), Content Engine | **No change needed.** The registry stores whatever assets are approved. In Phase A, it stores brand/team/office assets. The data model, search, filtering, and usage tracking are identical. Patient before/after assets simply don't enter the registry until Phase B. |
| CE-7 | **Content Engine Dashboard** | **Modified** | HQ Content Team, VP Marketing | Web | Pipeline metrics: requests received, content captured, assets produced, assets distributed, engagement metrics, brand health scores, champion leaderboard. All metrics report on non-patient content volume. | Content Engine analytics, Social platform APIs, Champion data | **Minor change**: Metrics are the same; the content being measured is different. "Assets captured" counts team photos instead of clinical before/afters. "Engagement" still tracks social media interactions. Champion leaderboard still works — it measures capture frequency regardless of content type. One metric deferred: "content → booking attribution" (requires patient funnel data from WS1). |
| CE-8 | **Distribution Monitor** | **Unchanged** | HQ Content Team | Web | View distribution status across social channels + Weave + print; engagement metrics per post; scheduling queue | Distribution Engine, Social platform APIs | **No change needed.** Distribution publishes to social channels regardless of content type. Engagement metrics come from platform APIs (Instagram, Facebook, etc.) — no PHI. |
| CE-9 | **Event Mode Console** | **Deferred (Phase B)** | — | — | — | — | Event mode for 4Front, Pikos courses, and partner events may involve patient case presentations and clinical photography requiring consent workflows. Deferred until PHI-safe capture pipeline exists. |

**Phase A Content Engine Summary**: 8 of 9 interfaces ship. The Content Engine is nearly fully functional in Phase A. The key difference is the type of content flowing through the pipeline: brand content, team photography, office environments, and marketing collateral instead of clinical before/after photography. The pipeline infrastructure, editorial workflow, distribution engine, and analytics are all content-type-agnostic and work identically.

---

### 2D. Culture OS (WS3) — Phase A

**Original**: 8 interfaces (CO-1 through CO-8)
**Phase A**: All 8 interfaces survive; 1 modified

| # | Interface | Phase A Status | Actor Types | Platform | Primary Use Cases (Phase A) | Data Sources (Phase A) | What Changed |
|---|-----------|---------------|-------------|----------|----------------------------|----------------------|--------------|
| CO-1 | **Morning Huddle Display** | **Modified** | All Practice Staff | TV display, PWA | 5-minute daily ritual: H-Score, focus behavior, recognition, practice wins. Impact story uses culture/brand metrics instead of funnel metrics. | Culture OS (H-Score, behaviors, recognition), Content Engine (brand metrics via Kafka) | **Minor change**: Impact Story segment loses funnel data ("Our show rate improved 5%"). Replaced with culture/brand stories: "Our H-Score improved 8 points since we started warm handoff training" or "Our Google rating went from 4.2 to 4.6 since the review request focus." |
| CO-2 | **CCA Academy Portal** | **Unchanged** | Clinical Staff, Office Managers | Web, PWA | Complete certification modules, watch training videos, take assessments, view credentials, track progress | Culture OS (curriculum, progress, credentials), Mux (video hosting) | **No change needed.** Training content is educational — no PHI. Certification tracking uses staff data (iSolved integration). |
| CO-3 | **Champion Hub** | **Unchanged** | Champions | PWA, Web | View champion scorecard, streak counter, content capture assignments, culture mission, regional leaderboard | Culture OS (champion data), Content Engine (capture stats via Kafka) | **No change needed.** Champion scorecards track staff activity (captures submitted, huddles led, behaviors modeled). Content capture stats from WS2 count submissions regardless of content type. |
| CO-4 | **Recognition Feed** | **Unchanged** | All Practice Staff | PWA, TV display | View recognition posts, celebrate peers, see leaderboard, bonus pool status | Culture OS (recognition data, leaderboard), Content Engine (brand templates via Kafka) | **No change needed.** Recognition is staff-to-staff. Templates are brand assets. Leaderboard tracks staff performance metrics. |
| CO-5 | **H-Score Dashboard** | **Unchanged** | Office Managers, Regional Directors | Web | Detailed H-Score breakdown: Cinnamon phone score (35%), Swell reviews (45%), Dental Intel (20%); trend over time; behavior-level detail | Cinnamon (phone scores), Swell (review scores), Dental Intel (operational scores), Culture OS (composite calculation) | **No change needed.** H-Score is computed from three vendor aggregate scores at the practice level. Cinnamon provides call quality scores (aggregate, not individual call recordings). Swell provides review volume and sentiment scores. Dental Intel provides operational compliance scores. None contain individual patient data. |
| CO-6 | **Playbook Library** | **Unchanged** | All Practice Staff | PWA, Web | Access hospitality playbooks, Grant training videos, practice-type protocols | Culture OS (playbook content), Mux (video hosting) | **No change needed.** Playbooks are reference content. |
| CO-7 | **Trust Transfer Tracker** | **Unchanged** | Office Managers, Clinical Staff | Web | View NLP-verified trust transfer scores per call, identify training opportunities, track improvement trends | Cinnamon (call quality analysis — aggregate scores per staff member, not individual patient call recordings) | **No change needed in Phase A.** Trust Transfer scores come from Cinnamon's aggregate analysis and are reported at the staff-member level, not the patient level. Cinnamon's own platform handles the PHI-bearing call recordings; what surfaces in the SGA platform is the derived score per team member. The SGA platform never stores or plays back the underlying call recordings in Phase A. |
| CO-8 | **Culture Admin Console** | **Unchanged** | HQ Culture Team | Web | Manage CCA curriculum, deploy playbook updates, configure huddle content, manage champion network, set reward budgets | Culture OS (admin data), Practice Data Hub (practice/staff profiles) | **No change needed.** Administration of culture programs uses staff/practice data, not patient data. |

**Phase A Culture OS Summary**: All 8 interfaces ship. Culture OS is the cleanest workstream for Phase A — it was designed around aggregate vendor scores (Cinnamon, Swell, Dental Intel) and staff development data, not patient records. The only modification is the Morning Huddle's Impact Story, which swaps funnel-derived stories for culture/brand-derived stories.

---

### 2E. Patient-Facing Interfaces — Phase A

**Original**: 6 interfaces (PF-1 through PF-6)
**Phase A**: 1 interface survives; 5 deferred

| # | Interface | Phase A Status | Rationale |
|---|-----------|---------------|-----------|
| PF-1 | **Practice Website** | **Available** | Practice websites are built from public practice data, brand assets from Asset Registry, and static content. No PMS integration required. Online booking button can link to NexHealth's hosted booking page (PHI handled by NexHealth, not our platform). |
| PF-2 | **Voice AI (Phone)** | **Deferred (Phase B)** | Voice AI performs real-time PMS lookups, handles appointment booking, and generates transcripts containing verbal PHI. Entirely PHI-dependent. |
| PF-3 | **SMS Conversations** | **Deferred (Phase B)** | Patient SMS requires patient contact data from PMS, HIPAA-compliant messaging infrastructure, consent management. |
| PF-4 | **Email Communications** | **Deferred (Phase B)** | Same as PF-3. Patient emails require patient records and consent. |
| PF-5 | **Review Request** | **Deferred (Phase B)** | Post-visit review requests are triggered by PMS appointment completion data. |
| PF-6 | **Digital Forms** | **Deferred (Phase B)** | Patient intake forms collect PHI directly. |

---

### Phase A Interface Count Summary

| Surface | Original | Phase A | Deferred to Phase B | Modified |
|---------|----------|---------|---------------------|----------|
| Practice Operations | 9 | 5 | 4 (PO-2, PO-3, PO-4, PO-5) | 2 (PO-1, PO-7) |
| Network Intelligence | 10 | 8 | 2 (NI-6, NI-9*) | 5 (NI-1, NI-2, NI-3, NI-4, NI-5) |
| Content Engine | 9 | 8 | 1 (CE-9) | 3 (CE-1, CE-2, CE-7) |
| Culture OS | 8 | 8 | 0 | 1 (CO-1) |
| Patient-Facing | 6 | 1 | 5 (PF-2 through PF-6) | 0 |
| **Total** | **42** | **30** | **12** | **11** |

*NI-9 Compliance Dashboard is reframed, not deferred — it ships as "Compliance Readiness Dashboard."

**71% of interfaces ship in Phase A.** The 12 deferred interfaces are concentrated in patient-facing features (5) and patient-data-dependent Practice Operations features (4).

---

## 3. Information Architecture Changes

### 3A. Practice Operations — Phase A Navigation

The navigation structure contracts to reflect available features. No phantom items.

```
Practice Operations (PWA / Web) — PHASE A
├── Today (default landing)
│   ├── Practice Briefing
│   │   ├── Practice Profile Summary
│   │   ├── H-Score Trend Widget (from Culture OS)
│   │   ├── Content Capture Brief (from Content Engine — non-patient content)
│   │   └── Enrollment Status (if practice is in onboarding)
│   ├── Action Cards (Phase A set)
│   │   ├── Storefront Health Alert ("Provider photo is 8 months old")
│   │   ├── Content Capture Opportunity ("Team photo day — capture the team before huddle")
│   │   ├── Playbook Update ("New phone script v2.4 available — review changes")
│   │   └── Culture Focus ("H-Score focus this week: warm handoffs")
│   └── Practice Activity Summary (replaces End-of-Day Scorecard)
│       ├── Content Captures Submitted Today
│       ├── Huddle Completed (Yes/No)
│       ├── Storefront Updates Made
│       └── Playbook Sections Viewed
│
├── Storefront
│   ├── Website Preview (live site with edit overlay)
│   ├── Google Business Profile Status
│   ├── Directory Listings (sync status per platform)
│   ├── Asset Gallery (approved brand/team assets from Content Engine)
│   │   └── "Use on Website" / "Push to GMB" actions
│   └── Digital Health Score (0-100)
│
├── Capture (Content Engine mobile interface)
│   ├── Today's Brief (non-patient capture assignments)
│   ├── Camera Interface (overlay guides for team/office content)
│   ├── Pending Uploads (offline queue)
│   └── My Captures (submitted content history)
│
├── Huddle
│   ├── Today's Huddle Agenda (from Culture OS)
│   ├── H-Score Detail (expandable)
│   └── Mark Huddle Complete
│
└── Settings
    ├── Practice Profile
    ├── Provider Roster
    ├── Playbooks (accessible from all roles)
    ├── Help & Support
    └── Sign Out
```

**Key structural changes:**
- "Patients" section removed entirely (PHI-dependent)
- "Calls" section removed (PHI-dependent)
- "Automations" section removed (PHI-dependent)
- "Capture" elevated to primary navigation (was embedded in action cards)
- "Huddle" elevated to primary navigation (was shared with PO-8)
- "Storefront" retained as-is

### 3B. Practice Operations — Phase A PWA Bottom Navigation

Role-based filtering still applies, but with a simplified screen set:

| Role | Visible Screens | Bottom Nav Items |
|------|----------------|-----------------|
| **Office Manager** | Today, Storefront, Capture, Huddle | 4 items |
| **Front Desk** | Today, Huddle | 2 items + Playbooks in Settings |
| **Clinical Staff / Champion** | Today, Capture, Huddle | 3 items |
| **Champion (dual role)** | Today, Capture, Huddle, Champion Hub | 4 items |

```
Office Manager (Phase A):
┌───────────────────────────────────────────┐
│  [ Today ]  [ Storefront ]  [ Capture ]  [ Huddle ]  │
└───────────────────────────────────────────┘

Front Desk (Phase A):
┌───────────────────────────────────────────┐
│          [ Today ]          [ Huddle ]             │
└───────────────────────────────────────────┘

Clinical Staff / Champion (Phase A):
┌───────────────────────────────────────────┐
│     [ Today ]     [ Capture ]    [ Huddle ]       │
└───────────────────────────────────────────┘

Champion — dual role (Phase A):
┌───────────────────────────────────────────┐
│  [ Today ]  [ Capture ]  [ Huddle ]  [ Hub ]  │
└───────────────────────────────────────────┘
```

**Phase B additions**: When PHI features activate, "Patients" and "Calls" screens appear in the bottom nav for Office Managers and Front Desk. "Automations" appears for Office Managers. The navigation framework already supports dynamic screen injection based on feature flags.

### 3C. Network Intelligence — Phase A Navigation

```
Network Intelligence (Web — Desktop) — PHASE A
├── Network Overview (default landing)
│   ├── KPI Bar (Phase A metrics — see Section 4)
│   ├── AI Alerts Panel (culture/brand/enrollment anomalies)
│   ├── Portfolio Heatmap (H-Score and brand health color layers)
│   └── Trend Charts (H-Score trend, brand health trend, enrollment velocity)
│
├── Regions
│   ├── Region Selector
│   ├── Region KPIs (aggregate culture/brand metrics)
│   ├── Region Practice Ranking (by H-Score, brand health, enrollment status)
│   └── Regional Director Notes
│
├── Practices
│   ├── Practice Search
│   ├── Practice Filters (type, acquisition date, H-Score band, enrollment status)
│   └── Practice Deep Dive (Phase A tabs — see Section 4)
│       ├── Profile Tab
│       ├── Culture Tab (H-Score + CCA + Huddle + Trust Transfer)
│       ├── Content & Brand Tab (brand health + pipeline + assets)
│       └── History Tab (enrollment timeline + events + interventions)
│
├── Operations
│   ├── HQ Standup Board (culture/brand/enrollment ranking)
│   ├── Enrollment Pipeline (unchanged)
│   ├── QC Queues
│   │   ├── Website QC Queue
│   │   └── Brand Health Violations
│   └── Playbook Manager (unchanged)
│
├── Compliance Readiness  ← renamed from "Compliance"
│   ├── BAA Execution Status (per vendor)
│   ├── Encryption Verification
│   ├── Audit Infrastructure Status
│   ├── Consent Configuration
│   └── Phase B Readiness Checklist
│
└── Settings
    ├── User Management
    ├── Practice Registry
    ├── Integration Status
    ├── Feature Flags (Phase A/B toggle visibility)
    └── Notification Preferences
```

**Key structural changes:**
- "Analytics" section removed (Funnel Explorer is PHI-dependent; remaining analytics folded into Network Overview trend charts)
- "Compliance" renamed to "Compliance Readiness" with Phase B preparation focus
- Practice Deep Dive tabs reduced from 6 to 4 (Performance and Communications tabs deferred)
- QC Queues loses "Comms QC Queue" (patient communications deferred)

### 3D. Content Engine — Phase A Navigation

```
Content Engine (Web — Desktop) — PHASE A
├── Pipeline (default landing)
│   ├── Capture Inbox (brand/team/office content submissions)
│   ├── Editorial Queue (AI-processed, human QC)
│   └── Studio Requests (design collateral)
│
├── Asset Registry (unchanged)
│
├── Distribution (unchanged)
│
├── Champions (unchanged)
│
├── Brand (unchanged)
│
├── Analytics
│   ├── Pipeline Metrics
│   ├── Champion Performance
│   ├── Engagement Analytics
│   ├── Brand Health Trends
│   └── [Phase B: Attribution — content → booking correlation]  ← empty state
│
└── Settings (unchanged)
```

**Key structural change:** Analytics section retains all subsections except "Attribution" (requires patient funnel data from WS1). Attribution shows a purposeful empty state: "Content attribution analysis requires PMS integration. Available in Phase B."

### 3E. Culture OS — Phase A Navigation

**No changes.** The entire Culture OS navigation structure ships as-is. The only content change is in the Morning Huddle Impact Story (culture/brand metrics replace funnel metrics).

---

## 4. Dashboard Redesign — Phase A

### 4.1 Executive Dashboard (NI-1) — Phase A KPI Bar

The KPI bar is the most visible change. It replaces revenue/funnel KPIs with culture/brand/operational KPIs.

**Phase A KPI Cards:**

| Position | Metric | Source | Update Frequency | Rationale |
|----------|--------|--------|-----------------|-----------|
| 1 | **Active Practices** | Practice Data Hub (enrollment status) | Real-time | Replaces "Total Production." Executives need to know how many practices are live, onboarding, or pending. |
| 2 | **Network H-Score (avg)** | Culture OS via Kafka `sgadental.ws3.hscore-calculated` | Daily batch (6 AM ET) | Retained from original. Already PHI-free. Same visual treatment, same drill-down behavior. |
| 3 | **Avg Brand Health Score** | Content Engine via Kafka `sgadental.ws2.brand-health-calculated` | Daily batch | Replaces "Show Rate." Brand health measures practice digital presence quality across website, GMB, directories, social engagement. Sourced from Content Engine. |
| 4 | **Enrollment Pipeline** | Practice Data Hub (enrollment records) | Real-time | Replaces "Treatment Acceptance." Shows: X practices onboarding, Y days avg to go-live, Z at risk of delayed launch. Drill-down opens Enrollment Pipeline view. |
| 5 | **Content Captures (MTD)** | Content Engine via Kafka `sgadental.ws2.capture-media-received` | Real-time (15-min refresh) | Replaces "New Patients." Network-wide content capture volume — a leading indicator of champion engagement and content pipeline health. |

**KPI Card interactions are unchanged** from the original spec: hover for elevation change, click to drill down, comparison toggle (vs. Target / vs. Prior Month / vs. Prior Year), shimmer loading, stale data indicators.

**Phase B KPI Bar additions:** When PHI features activate, the KPI bar expands to 7-8 cards. Total Production, New Patients, Show Rate, and Treatment Acceptance are added alongside the Phase A metrics. The grid layout supports dynamic card count (5-card row wraps gracefully to 4+4 on smaller viewports).

### 4.2 AI Alerts Panel — Phase A

AI alerts are generated by the AI Analysis Engine using non-PHI signals:

**Phase A Alert Types:**

| Alert Type | Source Signals | Example |
|------------|---------------|---------|
| H-Score Drop | Cinnamon + Swell + Dental Intel aggregate scores | "Southeast region H-Score dropped 6 points week-over-week. Root cause: 3 practices had no morning huddles logged this week." |
| Brand Health Degradation | Content Engine brand health scoring | "4 practices in Metro Atlanta have brand health scores below 50. Provider photos are >12 months old and no content published in 30 days." |
| Content Pipeline Bottleneck | Content Engine pipeline metrics | "Content pipeline bottleneck: 23 assets awaiting editorial review for 4+ days across 12 practices." |
| Enrollment Velocity Anomaly | Enrollment Pipeline data | "Dental Arts Decatur enrollment stalled at 'Integrating' for 8 days. PMS connector not responding." |
| Champion Inactivity | Content Engine + Culture OS champion data | "5 content champions have zero captures in the last 14 days. 3 are in the same region — possible coordinator issue." |
| CCA Certification Gap | Culture OS certification data | "12 practices have zero Gold-certified staff. Network average is 2.3 per practice." |
| Playbook Adoption Lag | Playbook Engine + Practice Data Hub | "Phone script v2.4 deployed 14 days ago. 28 practices have not accessed the update." |

**Severity ranking** in Phase A uses "Culture Impact Score" instead of "Revenue Impact":
- **High Impact**: H-Score below 50, brand health below 40, enrollment stalled >10 days
- **Medium Impact**: H-Score declining >5 points/week, content pipeline >20 assets stuck, champion inactive >14 days
- **Advisory**: Playbook adoption lagging, certification gaps, minor brand deviations

### 4.3 Portfolio Heatmap (NI-2) — Phase A

**Color Layer Selector (Phase A):**

```
Color by: [ H-Score ▾ ]
Options:
  - H-Score (default)
  - Brand Health Score
  - Content Activity (captures per week)
  - Enrollment Status
  - CCA Certification Level
```

Removed from Phase A: Composite Health (requires funnel data), Funnel Performance, Show Rate, Treatment Acceptance.

**Map interactions unchanged**: hover for tooltip (practice name, selected metric, trend), click for side panel Practice Deep Dive summary, cluster zoom, filter bar (Region, Practice Type, Location Type, Acquisition Date).

**Phase B additions**: Color layers for Composite Health, Funnel Performance, Show Rate, and Treatment Acceptance are added. The dropdown selector dynamically includes options based on active feature flags.

### 4.4 Practice Deep Dive (NI-5) — Phase A Tabs

**Phase A Tab Structure:**

| Tab | Data Sources | Key Content |
|-----|-------------|-------------|
| **Profile** | Practice Data Hub | Practice name, address, type, providers, hours, acquisition date, PMS system, integration status. Contact info for practice manager. Staff roster (from iSolved). |
| **Culture** | Culture OS via Kafka | H-Score composite + 3 sub-scores (Cinnamon phone 35%, Swell reviews 45%, Dental Intel 20%). 12-behavior breakdown. CCA certification roster (per team member: None/Bronze/Silver/Gold). Huddle completion rate (calendar view). Trust transfer scores (per staff member, aggregate from Cinnamon). Champion activity log. |
| **Content & Brand** | Content Engine via Kafka, Asset Registry | Brand health score (0-100 gauge). Content pipeline: assets in each stage for this practice. Content capture frequency (sparkline, weekly). Recent published assets (gallery preview). Champion content scorecard. Website health score (based on public crawl data: freshness, completeness, brand compliance). |
| **History** | Practice Data Hub, Enrollment Pipeline | Enrollment date and onboarding timeline. Key events log (go-live, playbook deployments, champion assignments, intervention history). Notes from VP Ops / Regional Directors. |

**Tabs deferred to Phase B:**
- **Performance Tab**: 5-stage funnel visualization, conversion rates, revenue trend, cohort benchmarks — all require PMS patient data.
- **Communications Tab**: Message volume, response rates, automation performance, Voice AI usage, channel mix — all require patient communication data.

**Tab injection design**: The `<PracticeDeepDive>` component accepts a `tabs` prop that is dynamically populated based on feature flags. In Phase A, it renders 4 tabs. When Phase B feature flags activate, Performance and Communications tabs are injected at positions 1 and 2, pushing Culture to position 3 and Content & Brand to position 4. The History tab always anchors at the end. No UI redesign required.

### 4.5 Content Pipeline Health Widget — Phase A

The Content Engine pipeline health widget on the Executive Dashboard is **unchanged** from the original spec. It displays:

```
┌──────────────────────────────────────────────────────────────┐
│  Content Pipeline Health                    [Last 7 days ▾]  │
├──────────────────────────────────────────────────────────────┤
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐     │
│  │  47  │──▸│  38  │──▸│  12  │──▸│  31  │──▸│  28  │     │
│  │Captured│  │Processing│ │In Review│ │Approved│ │Published│ │
│  │  ▲12% │  │  ●OK  │  │  ▲3d avg│ │  ●OK  │  │  ▲8%  │  │
│  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘     │
│                                                              │
│  ⚠ 4 assets stuck in review >5 days  [View in Content Engine]│
└──────────────────────────────────────────────────────────────┘
```

This widget is entirely PHI-free — it counts assets in pipeline stages, not patient data. In Phase A, the assets counted are brand/team content; in Phase B, patient clinical content joins the count. The widget is content-type-agnostic.

### 4.6 Intervention Queue (NI-3) — Phase A Redesign

**Phase A Card Anatomy:**

```
┌──────────────────────────────────────────────────────────────────┐
│  #1  Smile Center Buckhead              Southeast | GP | Active  │
│                                                                  │
│  Primary Issue: H-Score dropped 12 points in 2 weeks            │
│  Root Cause (AI): Cinnamon phone scores down 18% — new front    │
│  desk hire, no CCA Bronze enrollment started. Huddles missed     │
│  3 of last 5 days.                                               │
│                                                                  │
│  Culture Risk: HIGH    Days Flagged: 5                           │
│  Assigned: Unassigned                                            │
│                                                                  │
│  [Assign to Me]  [View Practice]  [Dismiss with Note]            │
└──────────────────────────────────────────────────────────────────┘
```

**Key difference from original**: "Est. Weekly Impact: -$3,200" replaced with "Culture Risk: HIGH." Revenue impact requires patient funnel data. Culture risk is derived from H-Score trajectory, huddle compliance, certification gaps, and brand health — all non-PHI signals. Ranking still produces a meaningful priority order for executive action.

---

## 5. Component Reuse Analysis

### 5.1 Components Reused Unchanged

These components from `@sgadental/ui` ship in Phase A with zero modification:

| Component | Category | Used In (Phase A) |
|-----------|----------|-------------------|
| `Button` (all variants) | Primitive | All surfaces |
| `Card` (all variants) | Pattern | All surfaces |
| `DataTable` | Pattern | NI (practice rankings, enrollment pipeline), CE (editorial queue, asset registry), CO (certification roster) |
| `LineChart` | Pattern | NI (H-Score trends, brand health trends), CO (H-Score dashboard) |
| `BarChart` | Pattern | NI (region comparison), CE (champion performance) |
| `Sparkline` | Pattern | NI (KPI cards), PO (H-Score widget) |
| `GaugeChart` | Pattern | NI (Practice Deep Dive brand health), CO (H-Score detail) |
| `PipelineChart` | Pattern | NI (enrollment pipeline stages, content pipeline), CE (capture pipeline) |
| `Badge / Tag` (all variants) | Primitive | All surfaces |
| `SideNav` | Layout | NI, CE, CO |
| `BottomNav` | Layout | PO (PWA) |
| `TopBar` | Layout | All surfaces |
| `Breadcrumb` | Layout | NI (drill-down navigation) |
| `Alert / Banner` (all variants) | Pattern | All surfaces |
| `Modal / Dialog` | Pattern | All surfaces |
| `EmptyState` | Pattern | All surfaces (especially Phase B placeholder states) |
| `Skeleton Loader` | Pattern | All surfaces |
| `HuddleDisplay` | CO-specific | CO (TV display), PO (Huddle screen) |
| `AcademyModule` | CO-specific | CO (CCA Academy) |
| `ChampionScorecard` | CO-specific | CO (Champion Hub), PO (Champion Hub in PWA) |
| `RecognitionPost` | CO-specific | CO (Recognition Feed) |
| `EditorialCard` | CE-specific | CE (Editorial Queue) |
| `DistributionTimeline` | CE-specific | CE (Distribution Monitor) |
| `BrandHealthGauge` | CE-specific | CE (Brand section), NI (Practice Deep Dive Content tab) |

### 5.2 Components Modified for Phase A

| Component | Modification | Reason |
|-----------|-------------|--------|
| `KPICard` | Props updated to accept Phase A metric types (Active Practices, Brand Health, Content Captures) alongside original types. Metric type is a prop; rendering logic unchanged. | New KPI content; component is metric-type-agnostic. |
| `AlertCard` | Severity criteria updated: Culture Risk replaces Revenue Impact. Badge colors and icons unchanged — just the label text and threshold logic. | AI alerts use different ranking signals. |
| `InterventionCard` | "Est. Weekly Impact" field replaced with "Culture Risk" field. Same card layout, same action buttons, same interaction patterns. | Revenue impact data unavailable without funnel data. |
| `PracticeDeepDive` | `tabs` prop becomes dynamic (injected from feature flag config). Phase A: 4 tabs. Component handles variable tab count. | Tab count changes between phases. |
| `ActionCard` (PO-specific) | Card content types expanded: storefront alerts, content capture prompts, playbook updates, culture focus. Same swipe gestures, same completion state, same visual treatment. | Patient-specific action cards (confirmations, follow-ups) deferred. |
| `HeatmapView` | Color layer selector options list reduced. Same map rendering, same marker behavior, same accessibility alternatives. | Funnel-based color layers deferred. |
| `CaptureCamera` | Before/after pairing mode disabled via feature flag. Overlay guides adjusted for non-clinical content (headshot framing, office tour composition). | Patient before/after photography deferred. |
| `PipelineHealthWidget` | No code change. Content type flowing through pipeline changes (brand content vs. clinical content). Widget is content-type-agnostic. | Pipeline metrics are agnostic to content type. |

### 5.3 Components Deferred to Phase B

| Component | Reason | Phase B Activation |
|-----------|--------|-------------------|
| `FunnelChart` | 5-stage patient funnel visualization requires PMS patient data | Activates with Practice Deep Dive Performance tab |
| `MessagePreview` | Patient outreach message cards require patient names, contact info, message content | Activates with Patient Outreach Queue (PO-2) |
| `CallContextCard` | Incoming call context requires patient record lookup from PMS | Activates with Call Dashboard (PO-3) |
| `AutomationCard` | Automation sequences target individual patients | Activates with Automation Status Board (PO-5) |
| `ScorecardWidget` (end-of-day version) | Original scorecard tracks patient-specific metrics (calls handled, messages sent, revenue impact) | Replaced by Practice Activity Summary in Phase A; original activates in Phase B |
| `TrustTransferScore` (individual call view) | Viewing individual call scores with playback requires call recording access (PHI) | Individual call drill-down activates in Phase B; aggregate scores per staff member available in Phase A |

### 5.4 Shared Hooks

| Hook | Phase A Status | Notes |
|------|---------------|-------|
| `useOfflineStatus` | Unchanged | Offline detection and banner logic is PHI-independent |
| `useRole` | Unchanged | Role-based screen filtering applies in both phases |
| `useStaleData` | Unchanged | Stale data detection for cross-workstream data applies identically |
| `useFeatureFlag` | **New** | Controls Phase A/B feature visibility. Components check feature flags to conditionally render Phase B content. |

---

## 6. Phase B Transition Design

### 6.1 Design Strategy: Progressive Revelation

Phase B does not "replace" Phase A. It adds to it. The transition strategy is **progressive revelation**: features appear as they become available, without disrupting existing workflows.

### 6.2 Navigation Transitions

**Practice Operations PWA:**
- Phase A bottom nav: 2-4 items depending on role (Today, Storefront, Capture, Huddle)
- Phase B: "Patients" and "Calls" tabs slide into the nav. Animation: 300ms spring, existing items shift to accommodate. For roles with >5 items, overflow moves to the "More" menu.
- Users see a one-time onboarding tooltip: "New: Patient Management is now available. Tap to explore."

**Network Intelligence sidebar:**
- Phase A: 5 primary nav items (Network Overview, Regions, Practices, Operations, Compliance Readiness)
- Phase B: "Analytics" section re-appears between Operations and Compliance. Compliance Readiness transitions to full "Compliance" with live PHI audit logs.
- Sidebar items animate in with a 200ms fade-in. No reorganization of existing items.

### 6.3 Dashboard Transitions

**Executive Dashboard KPI Bar:**
- Phase A: 5 KPI cards (Active Practices, H-Score, Brand Health, Enrollment Pipeline, Content Captures)
- Phase B: Expands to 8 cards. New cards (Total Production, New Patients, Show Rate) slide in with staggered 150ms animations. Phase A cards shift right. Active Practices may be demoted to a secondary position or merged into a dashboard header stat.
- The comparison toggle ("vs. Target / vs. Prior Month / vs. Prior Year") works identically for both Phase A and Phase B metrics.

**Portfolio Heatmap color layers:**
- Phase A: 5 options (H-Score, Brand Health, Content Activity, Enrollment Status, CCA Certification)
- Phase B: 4 additional options appear in the dropdown (Composite Health, Funnel Performance, Show Rate, Treatment Acceptance). Dropdown dynamically renders available options from a feature flag config array. No UI change needed — the dropdown just has more items.

### 6.4 Practice Deep Dive Tab Injection

This is the most visible Phase B transition and must be handled carefully:

**Phase A state:**
```
[ Profile ] [ Culture ] [ Content & Brand ] [ History ]
```

**Phase B state:**
```
[ Profile ] [ Performance ] [ Communications ] [ Culture ] [ Content & Brand ] [ History ]
```

**Transition behavior:**
1. When Phase B feature flags activate for a practice (may be rolling activation, not all-at-once):
   - Performance and Communications tabs appear with a subtle "NEW" badge (blue pill badge, auto-dismisses after 3 views)
   - Existing tabs shift right. Tab underline animation follows the active tab smoothly.
   - URL structure supports both: `/practices/:id/culture` continues to work. `/practices/:id/performance` becomes available.
2. If a user is on the Practice Deep Dive when tabs are injected (unlikely but possible during a real-time deployment):
   - Their current tab remains active and in focus
   - New tabs appear to the left but do not steal focus
   - No page reload required — React state update adds tabs to the rendered list

### 6.5 Empty State to Live Data Transitions

Several Phase A screens contain elements that will gain richer data in Phase B:

| Location | Phase A Empty State | Phase B Transition |
|----------|--------------------|--------------------|
| Practice Daily Agenda — Action Cards | 3-4 practice-focused cards (storefront, content, culture, playbook) | Patient-driven cards (confirmations, follow-ups, rebooking) join the AI-prioritized card set. Card count may grow to 5-6; pagination dots accommodate. |
| Content Engine Analytics — Attribution | Empty state panel: "Content attribution analysis requires PMS integration. Available in Phase B." with a subtle illustration | Panel populates with attribution chart: content published → new patient inquiries → bookings. Same panel container, same chart component family. |
| Network Intelligence — AI Alerts | Alerts based on culture/brand/enrollment signals | Patient funnel alerts join the mix (show rate drops, conversion anomalies). Same `AlertCard` component, same severity system. Alert volume increases. |
| Practice Deep Dive — missing tabs | 4-tab view; no visual indication of missing tabs | 6-tab view; tabs simply exist. Users who saw 4 tabs now see 6. The "NEW" badge calls attention. |
| Compliance Readiness Dashboard | Phase B readiness checklist, BAA tracker | Transforms into live Compliance Dashboard with HIPAA audit logs, PHI access reports, opt-out compliance, consent status. Same page URL, new content populated by Compliance Auditor service. |
| Intervention Queue — Culture Risk | "Culture Risk: HIGH" ranking metric | "Est. Weekly Revenue Impact: -$3,200" appears alongside culture risk. Dual ranking available. |
| Morning Huddle — Impact Story | Culture/brand impact stories | Funnel-derived impact stories return: "Our show rate improved 5% since we started confirmation texts." |
| Content Capture — Before/After | Before/after pairing hidden | Before/after pairing mode activates with a feature flag. Camera overlay guides switch to clinical content mode when a procedure-driven capture brief is received. |

### 6.6 Feature Flag Architecture

All Phase A/B transitions are controlled by a feature flag system in the Network Intelligence Settings:

```
Feature Flags (Settings > Feature Flags)
├── PMS Integration Active          [toggle per practice]
├── Patient Communications Active   [toggle per practice]
├── Voice AI Active                 [toggle per practice]
├── Patient Content Capture Active  [toggle per practice]
├── Funnel Analytics Active         [toggle — network-wide]
├── Full Compliance Dashboard       [toggle — network-wide]
└── Revenue Impact Scoring          [toggle — network-wide]
```

**Granularity**: Per-practice flags allow rolling Phase B activation. A practice that has completed BAA procurement and PMS integration can activate PHI features while other practices remain in Phase A mode. Network-wide flags control features that don't make sense at the practice level (analytics, compliance dashboard).

**Component behavior**: Every component that behaves differently in Phase A vs. Phase B reads from the `useFeatureFlag` hook. Components are never conditionally rendered at the page level — they render themselves and internally decide what to show based on flags. This prevents layout shifts when flags change.

### 6.7 Data Layer Transition

The shared data layer architecture (Practice Data Hub, Funnel Data Lake, AI Analysis Engine) is deployed in Phase A with all tables and schemas ready for Phase B data. The difference is that PHI tables are empty — no data flows in until BAAs are executed and PMS integrations are activated.

| Data Source | Phase A State | Phase B Activation |
|-------------|--------------|-------------------|
| Practice Data Hub (PostgreSQL) | Practice profiles, provider rosters, enrollment status, configurations populated. `patient_records` table exists but is empty. | PMS integration activates; `patient_records` begins receiving data. Row-level security and encryption already configured. |
| Funnel Data Lake (ClickHouse) | `h_score_history` and `content_events` tables populated. `funnel_events`, `communication_events`, `call_events` tables exist but are empty. | PMS + Twilio integrations activate; funnel and communication events begin flowing. Schemas are pre-deployed. |
| AI Analysis Engine | Generates alerts from H-Score, brand health, enrollment, content signals only. Anomaly detection models for funnel data are pre-trained on synthetic data. | Real patient funnel data activates funnel anomaly detection. Models transition from synthetic to real data. Alert ranking incorporates revenue impact. |
| Integration Middleware | NexHealth/Sikka ONE connectors deployed and authenticated but not actively syncing PHI. Connectivity health checks running. | Sync activated per practice as BAA + consent are confirmed. Data begins flowing through existing connectors. |

---

## 7. Cross-Workstream Touchpoints — Phase A Status

| # | Touchpoint | Phase A Status | Notes |
|---|-----------|---------------|-------|
| T-1 | H-Score in Practice Ops Morning Briefing | **Active** | H-Score is aggregate vendor data — fully PHI-free |
| T-2 | Content Capture Brief as Action Card | **Modified** | Brief content shifts from procedure-driven to brand/culture-driven |
| T-3 | Asset Gallery in Storefront Manager | **Active** | Asset type changes (brand/team content); gallery mechanics identical |
| T-4 | Impact Story in Morning Huddle (funnel data) | **Modified** | Story source shifts from funnel metrics to culture/brand metrics |
| T-5 | Content Captures in Scorecard | **Active** | Capture count still reported in Practice Activity Summary |
| T-6 | Network H-Score in NI KPI Bar | **Active** | Already PHI-free |
| T-7 | H-Score as Heatmap color layer | **Active** | Already PHI-free |
| T-8 | Culture Tab in Practice Deep Dive | **Active** | Fully intact — all Culture OS data is non-PHI |
| T-9 | Brand health in Practice Deep Dive | **Active** | Brand health and content metrics are non-PHI |
| T-10 | Brand deviation flags in QC Queues | **Active** | Brand compliance checking uses asset metadata, not PHI |
| T-11 | Champion roster in Content Engine | **Active** | Champion data is staff data, not PHI |
| T-12 | Appointment data driving capture briefs | **Deferred** | PMS appointment data is PHI; capture briefs use non-patient triggers in Phase A |
| T-13 | Attribution analysis (content → booking) | **Deferred** | Requires patient funnel data from WS1 |
| T-14 | Impact Story in Huddle (funnel data) | **Modified** | Same as T-4 |
| T-15 | Content capture stats in Champion Hub | **Active** | Capture stats count submissions regardless of content type |
| T-16 | Brand templates in Recognition Feed | **Active** | Brand assets are non-PHI |

**14 of 16 touchpoints are active or modified in Phase A.** Only T-12 (PMS-driven capture briefs) and T-13 (content → booking attribution) are fully deferred.

---

## 8. Summary: Phase A Platform at a Glance

### What Ships

| Capability | Interfaces | Value Delivered |
|------------|-----------|-----------------|
| **Practice Culture Management** | CO-1 through CO-8, PO-1 (modified), PO-8 | Daily huddles, H-Score tracking, CCA certification, champion programs, recognition, trust transfer scoring, playbook management |
| **Brand Content Pipeline** | CE-1 through CE-8, PO-6, PO-7 (modified) | Content capture (brand/team), AI processing, editorial review, multi-channel distribution, asset registry, brand health scoring, storefront management |
| **Network Visibility** | NI-1 through NI-5, NI-7, NI-8, NI-10 (all modified) | Executive dashboards with aggregate KPIs, portfolio heatmap, intervention queue (culture-driven), enrollment pipeline, playbook management, practice deep dive (4 tabs) |
| **Practice Onboarding** | NI-7, NI-9 (reframed) | Enrollment pipeline, silent data harvesting, integration connectivity, compliance readiness tracking |
| **Practice Websites** | PF-1, PO-6 | Brand-compliant practice websites from template + asset registry, GMB/directory sync |

### What Waits for Phase B

| Capability | Interfaces | Dependency |
|------------|-----------|------------|
| Patient Communications | PO-2, PO-3, PO-4, PO-5, PF-3, PF-4, PF-5 | PMS integration + Twilio BAA + patient consent |
| Patient Funnel Analytics | NI-6, NI-5 (Performance + Comms tabs) | PMS data flowing to ClickHouse |
| Voice AI | PF-2, PO-4 | Call recording pipeline + Whisper BAA |
| Patient Photo Capture | CE-2 (before/after mode), CE-9 | Patient consent workflow + clinical photo guidelines |
| Patient Forms | PF-6 | Direct PHI collection + storage |
| Revenue-Based Ranking | NI-1 (original KPIs), NI-3 (revenue impact) | Funnel data in ClickHouse |

### The Phase A Thesis

Phase A proves that a dental support organization can run its **culture program, brand content pipeline, practice onboarding, and network visibility** without touching a single patient record. This is not a reduced product — it is the operational foundation that must exist before patient-facing features have any context to operate in. When Phase B activates PMS integrations, the patient data flows into a platform that already has trained champions, established huddle rituals, functioning brand content pipelines, and executives who know how to read their dashboards.

---

*PHI-Free Experience Architecture designed by Product Designer Agent | 2026-03-20*
*Companion to: Original Experience Architecture (03-experience/experience-architecture.md)*
*Next deliverable: Detailed PHI-free user journeys for each actor type (11-phi-free-journeys/03-journeys/)*
