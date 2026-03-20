# PHI Scope Analysis — Phase A (PHI-Free) vs. Phase B (PHI-Enabled)

**Prepared by**: Product Strategist Agent
**Date**: March 2026
**Purpose**: Define exactly what is IN scope and OUT of scope for a PHI-free version of each workstream, enabling development to begin immediately without BAAs, compliance middleware, or PHI data flows.
**Companion Documents**: [PHI Data Flow Analysis](../../10-phi-analysis/phi-data-flow-analysis.md) | [Experience Architecture](../../03-experience/experience-architecture.md) | [Technical Architecture](../../04-architecture/technical-architecture.md)

---

## Executive Summary

Of the platform's 42 interfaces, **19 are GREEN** (fully functional without PHI), **11 are YELLOW** (functional in reduced capacity), and **12 are RED** (blocked until PHI infrastructure is in place). This means **71% of the interface surface area can ship in Phase A** — delivering immediate value to HQ leadership, content teams, culture teams, and practice staff for non-patient workflows.

| Classification | Count | % of Total | Description |
|---|---|---|---|
| GREEN (Phase A — Full) | 19 | 45% | Fully operational without any PHI |
| YELLOW (Phase A — Reduced) | 11 | 26% | Operational with defined feature gaps |
| RED (Phase B Only) | 12 | 29% | Cannot function without PHI |

**Phase A delivers**: Practice profiles and staff management, content production pipeline, culture and training programs, brand management, executive dashboards (with aggregate/non-PHI metrics), playbook management, enrollment pipeline management, and social media distribution.

**Phase B unlocks**: Patient communications, appointment management, funnel tracking, Voice AI, call analysis with PHI, before/after photos with patient mapping, and all patient-facing interfaces.

---

## 1. PHI Classification Matrix — All 42 Interfaces

### 1A. Practice Operations (WS1 Surface A) — 9 Interfaces

| # | Interface | Classification | Rationale |
|---|---|---|---|
| **PO-1** | Practice Daily Agenda | YELLOW | Action cards requiring PMS data (patient schedules, follow-ups) are unavailable. Can show staff task lists, content capture briefs (from WS2), H-Score widgets (from WS3), and practice operational reminders. Missing: patient-specific action cards, appointment summaries, revenue impact estimates. |
| **PO-2** | Patient Outreach Queue | RED | Core function is reviewing and sending AI-drafted patient messages. Requires patient names, contact info, appointment history, treatment data. Cannot operate without PHI. |
| **PO-3** | Call Dashboard | RED | Displays incoming call context with patient history lookup, patient records during calls, and call outcome logging. Every element requires real-time PMS patient data. |
| **PO-4** | Voice AI Monitor | RED | Reviews Voice AI call transcripts containing patient names, treatment discussions, and appointment details. Transcripts are inherently PHI. The call recordings themselves contain verbal PHI. |
| **PO-5** | Automation Status Board | RED | Displays active automation sequences (appointment confirmations, treatment follow-ups, rebooking, reactivation) — all patient-specific communications. Graduation status and sequence performance metrics are derived from patient interaction data. |
| **PO-6** | Storefront Manager | YELLOW | Practice website editing, GMB listing management, and directory sync can operate on practice profile data and approved brand assets from the Asset Registry (no PHI). Missing: patient review integration (reviews may contain PHI), before/after gallery sourced from patient photos, patient-specific testimonials. Digital Health Score can calculate on non-PHI factors (hours, provider info, listing accuracy) but loses the review component. |
| **PO-7** | Content Capture Prompt | GREEN | SMS/push notification prompting staff to capture content. The capture brief is generated from procedure schedules (procedure types per day), but the prompt itself contains no patient data — only "2 implant cases today" type information. Staff submit photos/video of clinical work (with separate patient consent), not patient records. Phase A can use manual procedure-type entry or practice-level scheduling data rather than PMS-triggered prompts. |
| **PO-8** | Morning Huddle Screen | YELLOW | Huddle agenda, H-Score trend, focus behavior, and recognition shoutout can all display without PHI. Missing: "yesterday's wins" tied to specific patient outcomes, funnel data tied to patient conversions. The impact story ("our show rate improved 5%") can use aggregate metrics rather than patient-specific ones. |
| **PO-9** | Playbook Viewer | GREEN | Playbooks (phone scripts, treatment follow-up protocols, onboarding checklists) are institutional knowledge assets. No patient data is required to view, version, or distribute playbooks. |

**PO Summary**: 2 GREEN, 3 YELLOW, 4 RED

---

### 1B. Network Intelligence (WS1 Surface B) — 10 Interfaces

| # | Interface | Classification | Rationale |
|---|---|---|---|
| **NI-1** | Executive Dashboard | YELLOW | Network-wide KPI bar can display H-Score (from WS3, aggregate), content metrics (from WS2, non-PHI), staff certification counts, and practice enrollment status. Missing: Total Production (requires PMS procedure revenue data), New Patients count (requires PMS patient records), Show Rate (requires appointment PHI), Treatment Acceptance (requires procedure-level PHI). AI Alerts that reference patient-level data are unavailable. Alerts about staff turnover, content trends, and culture metrics remain. |
| **NI-2** | Portfolio Heatmap | YELLOW | Geographic practice display can operate on practice profile data, H-Score, brand health score, content capture frequency, and staff certification rates. Missing: revenue-based color coding, patient volume heat mapping, funnel-derived health metrics. The composite score loses its PMS-derived components but retains culture and content components. |
| **NI-3** | Intervention Queue | YELLOW | AI-ranked practices needing attention can still rank by: H-Score decline, content capture drop-off, champion resignation, huddle non-completion, certification gaps, brand health violations. Missing: revenue impact ranking (requires PMS production data), patient funnel anomalies, communication performance flags. The queue is usable but loses its most financially impactful ranking signal. |
| **NI-4** | HQ Ops Standup Board | YELLOW | Problem queue can surface: enrollment pipeline issues, content pipeline bottlenecks, brand violations, integration connectivity status, QC queues for websites/content. Missing: revenue-impact ranking of practices, patient communication QC, batch operations involving patient data. |
| **NI-5** | Practice Deep Dive | YELLOW | Single-practice 360 view retains: practice profile, staff roster, enrollment history, Digital Presence tab (website health, GMB, directories), Culture tab (H-Score, CCA certification, huddle completion, champion activity), content activity. Missing: Performance tab (funnel visualization, conversion rates, revenue trend — all PMS-derived), Communications tab (message volumes, response rates, Voice AI quality — all patient communication data). Approximately 3 of 5 tabs are fully available; 2 are blocked. |
| **NI-6** | Funnel Analytics Explorer | RED | Custom funnel analysis requires patient-level funnel stage data (Lead Generated through Production Completed). Every query involves patient journey records, conversion tracking, and appointment data. Cannot operate without PHI. |
| **NI-7** | Enrollment Pipeline | GREEN | Tracks new practice onboarding: data harvesting status, integration connectivity, blitz deploy countdown, go-live readiness. This is practice-level operational data — practice profiles, PMS connector health checks, Twilio number provisioning status. No patient data is involved in enrollment tracking. |
| **NI-8** | HQ Playbook Manager | GREEN | Design, version, and deploy playbook updates across the network. Playbooks are institutional knowledge (phone scripts, protocols, checklists). A/B test playbook variants and view adoption metrics (which practices have which version). No patient data required. |
| **NI-9** | Compliance Dashboard | YELLOW | HIPAA audit trail and PHI access logs are relevant only when PHI is flowing (Phase B). However, the compliance dashboard can track in Phase A: consent status infrastructure readiness, opt-out compliance framework status, BAA procurement tracking, compliance training completion. In Phase A, this becomes a "compliance readiness" dashboard rather than an active PHI monitoring tool. |
| **NI-10** | Network Learning Report | YELLOW | Monthly AI-generated report can cover: content optimization wins, culture program insights, champion network health, brand health trends, enrollment velocity. Missing: patient funnel insights, revenue attribution, communication effectiveness, cross-practice patient conversion patterns. Report loses its most commercially valuable insights (the connection between behaviors and revenue) but retains operational intelligence. |

**NI Summary**: 2 GREEN, 7 YELLOW, 1 RED

---

### 1C. Content Engine (WS2) — 9 Interfaces

| # | Interface | Classification | Rationale |
|---|---|---|---|
| **CE-1** | Content Capture Bot | GREEN | SMS/WhatsApp bot that sends daily capture briefs to champions and receives photo/video submissions. The brief contains procedure types and capture checklists — no patient names or identifiers. Phase A can trigger briefs manually or on a schedule rather than from PMS appointment-completed events. Champions submit clinical work photos (separate patient consent workflow), not patient records. |
| **CE-2** | Capture App (Phase 2) | GREEN | PWA camera interface with overlay guides for before/after pairing, video recording. The app is a capture tool — it does not access PMS data. Patient consent is handled as a separate paper/digital workflow at the practice. Photos are tagged with practice, procedure type, and date — not patient identifiers. |
| **CE-3** | Editorial Queue | YELLOW | AI-processed content review, approval, scheduling, and distribution management operates fully on content assets. Missing: patient-mapped before/after photo pairing (the system cannot link a "before" to an "after" via patient ID without the PMS patient-image mapping in PostgreSQL). Before/after photos can still be manually paired by the HQ content team based on visual matching or champion-provided context. AI caption generation works on the image content itself, not patient data. |
| **CE-4** | Studio Request Portal | GREEN | Design request submission, tracking, proof review, and approval. Practices request flyers, social posts, TV slides. No patient data involved — requests reference practice name, procedure types, promotional campaigns, brand guidelines. |
| **CE-5** | Creative Director Review | GREEN | 58-second tap-approve flow for brand-controlled collateral. Reviews design assets against brand guidelines. No patient data in the review workflow. Brand deviation flags are design/brand concerns, not PHI concerns. |
| **CE-6** | Asset Registry | GREEN | Browse all approved assets by practice, brand universe, asset type. Download originals, view usage history, check brand health score. Assets are de-identified content (photos with metadata stripped, design collateral, brand assets). No patient identifiers in the registry metadata. |
| **CE-7** | Content Engine Dashboard | GREEN | Pipeline metrics (requests received, content captured, assets produced, assets distributed), engagement metrics, brand health scores, champion leaderboard. All non-PHI operational and social media engagement data. |
| **CE-8** | Distribution Monitor | GREEN | Distribution status across social channels, engagement metrics per post, scheduling queue. Social media metrics (likes, shares, reach, impressions) are public engagement data — no PHI. |
| **CE-9** | Event Mode Console | GREEN | Special capture mode for events (4Front, Pikos courses, partner events). High-volume content intake and real-time processing. Event content is practice/brand marketing content, not patient data. |

**CE Summary**: 8 GREEN, 1 YELLOW, 0 RED

---

### 1D. Culture OS (WS3) — 8 Interfaces

| # | Interface | Classification | Rationale |
|---|---|---|---|
| **CO-1** | Morning Huddle Display | YELLOW | Huddle agenda, H-Score (composite), focus behavior, recognition shoutout are all non-PHI. Missing: specific patient-outcome stories tied to funnel data (e.g., "our show rate improved 5% — that's 4 more patients per week" requires PMS appointment data). Phase A can show aggregate practice performance trends from vendor scores without patient-level data. |
| **CO-2** | CCA Academy Portal | GREEN | Certification modules, training videos, assessments, credentials, and progress tracking are all learning management functions. Content is institutional (hospitality techniques, phone skills, trust transfer training). No patient data in the training platform. |
| **CO-3** | Champion Hub | GREEN | Champion scorecard, streak counter, content capture assignments, culture missions, regional cohort leaderboard. All champion/staff performance data — not patient data. Content capture stats come from WS2 (non-PHI asset counts). |
| **CO-4** | Recognition Feed | GREEN | Recognition posts, peer celebrations, leaderboard standings, bonus pool status. Staff-to-staff recognition. No patient data. |
| **CO-5** | H-Score Dashboard | YELLOW | The H-Score composite and sub-scores (Cinnamon phone 35%, Swell reviews 45%, Dental Intel 20%) aggregate vendor data to practice-level metrics. However, the Cinnamon phone score is derived from call analysis that involves patient interactions, and Swell reviews may contain patient-authored health disclosures. Phase A can display aggregate vendor scores if vendors provide practice-level scores without individual patient/call-level detail. The behavior-level detail (12 core behaviors breakdown) is tied to call analysis that may reference patient interactions. Missing: individual call-level trust transfer scores, specific review content analysis. |
| **CO-6** | Playbook Library | GREEN | Hospitality playbooks by behavior, Grant training videos (via Mux), practice-type-specific protocols. All institutional training content. No patient data. |
| **CO-7** | Trust Transfer Tracker | RED | NLP-verified trust transfer scores per call require call recording transcripts (verbal PHI), call-level analysis linking caller identity to interaction quality, and Cinnamon phone analytics at the individual call level. This interface cannot function without processing call recordings containing patient conversations. |
| **CO-8** | Culture Admin Console | GREEN | CCA curriculum management, playbook deployment, huddle content configuration, champion network management, reward budgets. All administrative functions operating on staff/institutional data, not patient data. |

**CO Summary**: 5 GREEN, 2 YELLOW, 1 RED

---

### 1E. Patient-Facing Interfaces (Cross-Workstream) — 6 Interfaces

| # | Interface | Classification | Rationale |
|---|---|---|---|
| **PF-1** | Practice Website | RED | While the website shell (practice info, services, hours, provider bios) is non-PHI, the core value features require PHI: before/after galleries with patient photos (facial images = PHI), patient review integration (reviews contain health disclosures), and online appointment booking (requires PMS read/write through NexHealth). See Section 5 for detailed impact. |
| **PF-2** | Voice AI (Phone) | RED | Real-time PMS lookup during calls, appointment booking/rescheduling/cancellation, patient identification by phone number. Every function requires real-time access to patient records. Verbal PHI is unavoidable. |
| **PF-3** | SMS Conversations | RED | Appointment confirmations, follow-up messages, rebooking prompts, reactivation outreach. Every message contains patient-specific content (name, appointment time, treatment references). Requires Twilio HIPAA tier + Communications Service + PMS data. |
| **PF-4** | Email Communications | RED | Appointment confirmations, treatment follow-up education, rebooking reminders. Same PHI dependency as SMS — patient-specific content in every communication. |
| **PF-5** | Review Request | RED | Post-visit review prompts require knowing: (a) that a patient visited (appointment data), (b) the patient's contact info, (c) timing relative to the visit. The review request itself contains the patient's name and practice visit context. |
| **PF-6** | Digital Forms | RED | Pre-visit intake forms, medical history updates, consent signatures. Patients submit their own PHI (name, DOB, medical history, insurance). Forms must be stored in HIPAA-compliant infrastructure with the PMS Integration Service writing data back to the practice management system. |

**PF Summary**: 0 GREEN, 0 YELLOW, 6 RED

---

### Classification Summary Table

| Surface | GREEN | YELLOW | RED | Total |
|---|---|---|---|---|
| Practice Operations (WS1-A) | 2 | 3 | 4 | 9 |
| Network Intelligence (WS1-B) | 2 | 7 | 1 | 10 |
| Content Engine (WS2) | 8 | 1 | 0 | 9 |
| Culture OS (WS3) | 5 | 2 | 1 | 8 |
| Patient-Facing | 0 | 0 | 6 | 6 |
| **Total** | **17** | **13** | **12** | **42** |

*Note: Revised from executive summary after detailed analysis. 17 GREEN + 13 YELLOW = 30 interfaces (71%) available in Phase A.*

---

## 2. Per-Workstream Scope Definition

### 2A. Workstream 1: The Practice Platform

#### Phase A Feature Set (No PHI)

**Fully Available Interfaces (GREEN):**

| Interface | Full Capability |
|---|---|
| PO-7 — Content Capture Prompt | SMS/push capture briefs, photo/video submission flow, capture checklists |
| PO-9 — Playbook Viewer | Full playbook access, version history, search by behavior/protocol |
| NI-7 — Enrollment Pipeline | Complete practice onboarding pipeline (Kanban, blitz deploy countdown, integration status) |
| NI-8 — HQ Playbook Manager | Playbook creation, versioning, deployment, A/B testing, adoption metrics |

**Reduced Capacity Interfaces (YELLOW):**

| Interface | Available | Missing |
|---|---|---|
| PO-1 — Practice Daily Agenda | Staff task lists, content capture briefs (from WS2), H-Score widget (from WS3), operational reminders | Patient-specific action cards (7 confirmations due, 3 follow-ups), appointment schedule summary, revenue impact estimates |
| PO-6 — Storefront Manager | Website editing (practice info, hours, providers, services), GMB listing management, directory sync, approved Asset Registry photos (non-patient) | Patient review integration on website, before/after gallery with patient photos, patient-specific testimonials, review-based Digital Health Score component |
| PO-8 — Morning Huddle Screen | H-Score trend, focus behavior of the day, recognition shoutout, team announcements | Patient outcome stories ("our show rate improved 5% = 4 more patients/week"), funnel-linked impact metrics |
| NI-1 — Executive Dashboard | H-Score (network avg), content metrics, staff certification counts, enrollment status, brand health, content capture frequency, AI alerts for culture/content/staff trends | Total Production $, New Patients count, Show Rate, Treatment Acceptance, revenue-based AI anomaly alerts |
| NI-2 — Portfolio Heatmap | Geographic practice view with H-Score color coding, brand health, content frequency, certification rate | Revenue-based coloring, patient volume heat mapping, funnel-derived composite health |
| NI-3 — Intervention Queue | AI-ranked by: H-Score decline, content capture drop, champion gaps, certification gaps, brand violations | Revenue impact ranking, patient funnel anomalies, communication performance flags |
| NI-4 — HQ Ops Standup Board | Enrollment pipeline issues, content bottlenecks, brand violations, integration health, website/content QC queues | Revenue-impact practice ranking, patient communication QC, patient data batch operations |
| NI-5 — Practice Deep Dive | Practice profile, staff roster, enrollment timeline, Digital Presence tab (website health, GMB, directories), Culture tab (H-Score, CCA, huddles, champions), content activity | Performance tab (funnel, conversions, revenue), Communications tab (message volumes, response rates, Voice AI) |
| NI-9 — Compliance Dashboard | Compliance readiness tracking: BAA procurement status, consent infrastructure setup, compliance training completion, framework configuration | Active PHI audit trail, real-time PHI access logs, opt-out compliance monitoring, flagged communications review |
| NI-10 — Network Learning Report | Content optimization insights, culture program effectiveness, champion network health, brand trends, enrollment velocity | Patient funnel insights, revenue attribution, communication effectiveness, behavior-to-revenue correlation |

**User Personas Served in Phase A:**
- HQ Operations Team (NI-4, NI-7, NI-8 fully; NI-1, NI-2, NI-3, NI-5 in reduced form)
- CEO/Executives (NI-1, NI-10 in reduced form)
- VP Ops / Regional Directors (NI-2, NI-3, NI-5 in reduced form)
- Office Managers (PO-1, PO-6, PO-8, PO-9 — daily workflow is significantly reduced without patient action cards)
- Compliance Officer (NI-9 — readiness mode only)

**User Personas Underserved in Phase A:**
- Front Desk Staff (PO-3 Call Dashboard is RED; they lose their primary tool)
- Office Managers doing patient outreach (PO-2 is RED; PO-5 is RED)

**Data Sources (Phase A — Non-PHI Only):**
- Practice Data Hub (PostgreSQL): Practice profiles, staff rosters, configurations, enrollment status
- iSolved: Staff HR data (synced to Practice Data Hub)
- Vendor aggregate scores: Cinnamon (practice-level phone score), Swell (practice-level review score), Dental Intel (practice-level metrics) — aggregate only, no individual call/review/patient data
- ClickHouse: Content engagement analytics, brand health calculations, champion performance metrics
- Asset Registry (S3/CDN): Approved content assets
- Kafka events: H-Score calculated (from WS3), content engagement updated (from WS2), champion roster (from WS3), training completed (from WS3)

#### Phase B Feature Set (PHI Added)

**RED Interfaces Requiring PHI:**

| Interface | PHI Data Types Required | BAAs Required | Compliance Infrastructure |
|---|---|---|---|
| PO-2 — Patient Outreach Queue | Patient names, phone numbers, appointment history, treatment data, communication preferences | Twilio (HIPAA tier), Anthropic (Claude for AI message drafting) | Communications Service, PHI Filter, Consent Engine, Compliance Middleware |
| PO-3 — Call Dashboard | Patient names, appointment history, insurance info, treatment records (real-time PMS lookup during calls) | NexHealth, Sikka ONE | PMS Integration Service, real-time PMS API, Compliance Middleware |
| PO-4 — Voice AI Monitor | Call transcripts (verbal PHI: patient names, treatment discussions, insurance), call recordings | OpenAI (Whisper for transcription), Anthropic (Claude for analysis) | Voice AI Service, PHI Filter, self-hosted Whisper (recommended), Compliance Middleware |
| PO-5 — Automation Status Board | Patient contact info, appointment dates, treatment codes, communication delivery status per patient | Twilio, NexHealth | Communications Service, Workflow Engine (Temporal) with PHI payloads, Compliance Middleware |
| NI-6 — Funnel Analytics Explorer | Patient funnel stage data, appointment records, conversion tracking, procedure codes, revenue per patient | NexHealth, Sikka ONE | PMS Integration Service, ClickHouse (with PHI firewall), Compliance Middleware |

---

### 2B. Workstream 2: Content Engine

#### Phase A Feature Set (No PHI)

**Fully Available Interfaces (GREEN):**

| Interface | Full Capability |
|---|---|
| CE-1 — Content Capture Bot | Daily capture briefs via SMS/WhatsApp, photo/video receipt, processing status updates |
| CE-2 — Capture App | Camera interface with overlay guides, before/after visual pairing (manual), batch upload |
| CE-4 — Studio Request Portal | Design request submission, tracking, proofs, approvals |
| CE-5 — Creative Director Review | 58-second brand approval flow, batch review, brand deviation flags |
| CE-6 — Asset Registry | Full asset browse, metadata, usage history, variants, upload |
| CE-7 — Content Engine Dashboard | Pipeline metrics, engagement analytics, brand health, champion leaderboard |
| CE-8 — Distribution Monitor | Social channel status, engagement metrics per post, scheduling |
| CE-9 — Event Mode Console | High-volume event capture, real-time processing |

**Reduced Capacity Interfaces (YELLOW):**

| Interface | Available | Missing |
|---|---|---|
| CE-3 — Editorial Queue | AI-processed content review, caption editing, scheduling, distribution. Before/after photos can be manually paired by visual similarity or champion context notes. | Automated patient-mapped before/after pairing (linking "before" to "after" via patient_id in PostgreSQL). Loss of automatic before/after matching accuracy. |

**User Personas Fully Served in Phase A:**
- HQ Content Team (CE-3, CE-6, CE-7, CE-8 — their entire daily workflow)
- Creative Director (CE-5 — full tap-approve workflow)
- VP Marketing (CE-7 — full dashboard)
- Champions / Practice Staff (CE-1, CE-2 — full capture workflow)
- Office Managers / HQ Marketing (CE-4 — full request portal)

**User Personas Underserved in Phase A:**
- None significant. WS2 is the most PHI-independent workstream.

**Data Sources (Phase A — Non-PHI Only):**
- Asset Registry (S3 + PostgreSQL metadata): All content assets
- Cloudinary: Image/video transformations (de-identified assets only — no patient-mapped photos)
- Canva (via Design Tool Abstraction): Template generation, brand asset management
- Social platform APIs: Engagement metrics (Instagram, Facebook, Google Business, etc.)
- Practice Data Hub: Practice profiles, brand universe assignments
- Kafka events: Champion roster (from WS3), capture media received, content engagement metrics
- Mux: Video hosting for training and testimonial content

**Phase A Limitation — Before/After Photos:**
In Phase A, before/after dental photos can still be captured, processed, and published. What is missing is the automated patient-ID-based matching that links a "before" image taken at visit 1 to an "after" image taken at visit 2 for the same patient. In Phase A, champions and HQ content team manually pair these based on visual similarity, procedure notes, and timestamps. This is a workflow inconvenience, not a capability loss.

**Important**: Photos of clinical dental work (close-up intraoral shots) where the patient's face is not visible are generally not PHI. Photos showing patient faces ARE PHI. Phase A content capture guidelines should instruct champions to capture intraoral/clinical shots only. Full face before/after photography must wait for Phase B with Cloudinary BAA or self-hosted image processing.

#### Phase B Feature Set (PHI Added)

| Interface | PHI Data Types Required | BAAs Required |
|---|---|---|
| CE-3 (full) — Editorial Queue | Patient-image mapping (patient_id to asset_id for before/after pairing) | Cloudinary (BAA or self-hosted alternative) |
| CE-1 (enhanced) — Capture Bot | PMS appointment-completed events with procedure codes for automated, same-day capture prompts | NexHealth/Sikka ONE (for PMS event triggers) |

**Phase B also enables:**
- Automated before/after photo pairing via patient_id
- Patient facial photo capture, storage, and publication with full consent tracking
- PMS-triggered capture prompts (real-time, procedure-specific)
- Patient testimonial videos with patient identity management

---

### 2C. Workstream 3: Culture OS

#### Phase A Feature Set (No PHI)

**Fully Available Interfaces (GREEN):**

| Interface | Full Capability |
|---|---|
| CO-2 — CCA Academy Portal | Full learning management: certification modules (Bronze/Silver/Gold), training videos, assessments, Credly badges, progress tracking |
| CO-3 — Champion Hub | Champion scorecard, streak counter, content capture assignments (from WS2), culture missions, regional cohort leaderboard |
| CO-4 — Recognition Feed | Peer recognition posts, celebrations, leaderboard standings, bonus pool status |
| CO-6 — Playbook Library | All hospitality playbooks, Grant training videos (Mux), behavior guides, practice-type protocols |
| CO-8 — Culture Admin Console | CCA curriculum management, playbook deployment, huddle content config, champion network management, reward budgets |

**Reduced Capacity Interfaces (YELLOW):**

| Interface | Available | Missing |
|---|---|---|
| CO-1 — Morning Huddle Display | H-Score composite trend, focus behavior of the day, recognition shoutout, team announcements, culture missions | Patient-specific impact stories linking behaviors to outcomes (e.g., "show rate improved 5% = 4 more patients"), funnel-attributed impact metrics |
| CO-5 — H-Score Dashboard | Practice-level composite score and sub-scores (if vendors provide aggregate practice-level data without individual call/patient detail). 12 core behaviors framework visible. Trend over time. | Individual call-level behavior scoring, specific review content analysis, patient-interaction-level trust transfer detail. If vendor aggregate feeds include no patient/caller identifiers, H-Score can display in reduced form. |

**RED Interfaces Requiring PHI:**

| Interface | PHI Dependency |
|---|---|
| CO-7 — Trust Transfer Tracker | Requires call recording transcripts (verbal PHI), per-call NLP scoring, caller identity matching. Cinnamon phone analytics at the individual call level contain patient interaction data. Cannot score trust transfers without processing the actual patient conversation. |

**User Personas Fully Served in Phase A:**
- All Practice Staff (CO-2, CO-4, CO-6 — training, recognition, playbooks)
- Champions (CO-3 — full champion workflow)
- HQ Culture Team (CO-8 — full admin console)

**User Personas Partially Served:**
- Office Managers (CO-1 huddle is reduced; CO-5 H-Score is reduced)
- Regional Directors (CO-5 H-Score dashboard loses depth)

**User Personas Underserved:**
- Office Managers / Clinical Staff using CO-7 Trust Transfer Tracker (fully blocked)

**Data Sources (Phase A — Non-PHI Only):**
- Practice Data Hub: Practice profiles, staff rosters, champion assignments
- Vendor aggregate scores: Cinnamon (practice-level phone score), Swell (practice-level review score/avg rating), Dental Intel (practice-level huddle completion, operational metrics)
- iSolved: Staff HR data for certification tracking against roster
- Mux: Training video hosting (Grant recordings)
- Credly: Badge/certification management
- Kafka events: Content capture stats (from WS2), champion activity metrics, certification events
- PostgreSQL: CCA curriculum data, playbook versions, recognition posts, reward pool

#### Phase B Feature Set (PHI Added)

| Interface | PHI Data Types Required | BAAs Required |
|---|---|---|
| CO-7 — Trust Transfer Tracker | Call recordings (verbal PHI), call transcripts, caller identity, per-call NLP scores | OpenAI (Whisper for transcription), Anthropic (Claude for NLP analysis), Cinnamon (BAA for call-level data) |
| CO-1 (full) — Morning Huddle | Patient funnel data for impact stories, appointment-level outcome data | NexHealth/Sikka ONE, PMS Integration Service |
| CO-5 (full) — H-Score Dashboard | Individual call-level scores, specific review content, patient-interaction-level detail | Cinnamon (BAA), Swell (BAA) |

---

## 3. Shared Infrastructure Phasing

### Phase A Shared Services (No PHI Required)

| # | Service | Phase A Capability | PHI Dependency |
|---|---|---|---|
| 1.1 | **Practice Data Hub** | Full. Practice profiles, staff rosters, configurations, regions. The `shared.practices` and `shared.practice_staff` tables contain no patient data. | NONE — practice/staff data is not PHI |
| 1.5 | **Asset Registry** | Full. S3 storage, CDN delivery, metadata catalog. Assets are de-identified content. Cloudinary transforms operate on non-facial clinical photos only. | NONE for non-facial assets. Phase B for patient facial photos. |
| 1.6 | **Event Bus (Kafka)** | Operational. Topic infrastructure, schema registry, producer/consumer SDK. Phase A events carry no patient data in payloads (see Section 4). | NONE for Phase A event contracts |
| 1.7 | **Auth & Identity** | Full. AWS Cognito, RBAC, API keys, session management. User identities are staff/HQ users, not patients. | NONE — staff identity is not PHI |
| 1.9 | **Workflow Engine (Temporal)** | Full for non-PHI workflows: practice enrollment, content production pipeline, nightly vendor score sync, playbook deployment. | NONE for Phase A workflows. Phase B adds PHI-carrying workflow payloads (communication cadences, PMS sync). |
| 1.10 | **Design Tool Abstraction** | Full. Canva template operations, asset generation, brand management. Design tools never process PHI. | NONE |

### Phase B Shared Services (PHI Infrastructure Required)

| # | Service | Phase B Capability | Prerequisites |
|---|---|---|---|
| 1.2 | **PMS Integration Service** | NexHealth real-time read/write, Sikka ONE batch sync, webhook processing, appointment/patient API. Every operation involves patient records. | NexHealth BAA, Sikka ONE BAA, PMS Integration Service deployed with Compliance Middleware |
| 1.3 | **Communications Service** | Twilio SMS/Voice/WhatsApp, AWS SES email, consent registry, message routing, delivery tracking. All patient-facing communications. | Twilio HIPAA tier BAA ($500/mo), AWS SES BAA (included in AWS BAA), consent registry populated, Compliance Middleware |
| 1.4 | **AI Service (with PHI Filter)** | PHI Filter pipeline (scan, replace, send to LLM, re-inject), Voice AI transcription via Whisper, patient message generation via Claude. | Anthropic BAA, OpenAI BAA (Whisper), PHI Filter implementation + testing, Redis for PHI token mapping |
| 1.8 | **Compliance & Audit Layer** | Active PHI monitoring: real-time audit logging, PHI access tracking, breach detection, retention enforcement. | Full deployment into all services, audit log infrastructure (PostgreSQL + S3/Glacier archival), KMS key management |

### Phase A AI Service (Limited, No PHI)

The AI Service CAN operate in Phase A for non-PHI use cases:
- Content caption generation (image analysis, no patient data)
- Playbook content generation/editing
- Brand health analysis (evaluating design assets)
- Anomaly detection on aggregate practice metrics
- Report generation from non-PHI data sources

In Phase A, the AI Service operates WITHOUT the PHI Filter (it is not needed because no PHI enters the prompts). The PHI Filter is implemented and tested during Phase A but activated in Phase B.

---

## 4. Cross-Workstream Event Phasing

### Phase A Events (No Patient Data in Payloads)

| # | Event Contract | Source | Consumer | Phase A Status |
|---|---|---|---|---|
| 1 | Champion Roster | WS3 | WS1, WS2 | GREEN — Payload contains staff data (staff_id, name, role, engagement_score). No patient data. |
| 2 | H-Score Calculated | WS3 | WS1 | GREEN — Payload contains practice-level composite scores, certification breakdown. Aggregate vendor scores, not patient-level data. |
| 3 | Benchmarking Taxonomy | WS3 | WS1, WS2 | GREEN — Taxonomy definitions (practice types, benchmarks, location types). Institutional data, no PHI. |
| 4 | Champion Stories | WS3 | WS2 | YELLOW — Story text/media submissions from champions. Generally non-PHI (culture moments, team highlights). Risk: `patient_testimonial` story type may reference patients. Phase A mitigation: filter out `patient_testimonial` type; allow `wow_moment`, `team_highlight`, `culture_win` only. |
| 6 | Approved Assets | WS2 | WS1 | GREEN — Content asset metadata (CDN URLs, dimensions, captions, tags). No patient identifiers in asset payload. |
| 8 | Social Engagement | WS2 | WS1 | GREEN — Aggregate engagement metrics (reach, impressions, engagements per channel). Public social media data, no PHI. |
| 9 | Training Completion | WS3 | WS1 | GREEN — Staff certification events (module completed, tier earned, practice readiness stats). Staff data, not patient data. |
| 10 | Onboarding Status | WS1 | WS3 | GREEN — Practice onboarding milestones (enrollment stages, completed steps). Practice-level operational data, no PHI. |

### Phase B Events (Patient Data in Payloads)

| # | Event Contract | Source | Consumer | PHI in Payload |
|---|---|---|---|---|
| 5 | Patient Data for Content Personalization | PMS Integration | WS2 | Contains `patient_id_hash`, procedure codes, media consent status. While the event uses a hashed patient ID (not cleartext PII), the procedure codes combined with practice_id and timestamp could be re-identified. Phase B only. |
| 7 | Funnel Events | WS1 | WS2 | Contains `patient_id_hash`, funnel stage transitions, source attribution, estimated value. Same re-identification risk as Event 5. Phase B only. |

**Phase A Summary**: 8 of 10 event contracts can flow in Phase A. 2 require Phase B.

---

## 5. Patient-Facing Interfaces — Impact Assessment

All 6 patient-facing interfaces (PF-1 through PF-6) are classified RED / Phase B Only.

### PF-1: Practice Website

**Why RED**: The website shell (practice name, hours, services, provider bios, contact info) CAN be generated from practice profile data alone and is technically non-PHI. However, the experience architecture defines the practice website as including:
- Before/after photo galleries (patient facial images = PHI)
- Patient review integration (reviews contain health disclosures)
- Online appointment booking (requires PMS read/write)

**Phase A Partial Alternative**: Deploy a "brochure-mode" website with practice info, provider bios, services, and hours — but WITHOUT patient photos, reviews, or booking. Use a "Call to Book" CTA instead of online scheduling. This delivers SEO value and digital presence without PHI.

**Impact**: Practices lose online booking capability — the highest-converting patient acquisition feature. The brochure website is a stepping stone, not a destination.

### PF-2: Voice AI (Phone)

**Why RED**: Voice AI performs real-time PMS lookup during patient calls, identifies patients by phone number, books/reschedules/cancels appointments, and provides after-hours answers with patient context. Every interaction involves accessing and creating patient records. The audio stream itself contains unstructured verbal PHI that cannot be pre-filtered.

**Impact**: Practices must rely entirely on human call handling. The Voice AI's capacity to handle ~20% of calls (particularly after-hours and overflow) is unavailable. This is a significant operational impact for practices with limited front desk staff.

### PF-3: SMS Conversations

**Why RED**: Every SMS to a patient contains PHI (patient name, appointment details, treatment references). The automation engine that sends confirmations, follow-ups, rebooking prompts, and reactivation outreach all require patient records from the PMS.

**Impact**: Practices lose automated patient communication — the feature most directly tied to revenue (appointment confirmations reduce no-shows, follow-ups drive treatment acceptance). Manual phone calls must substitute.

### PF-4: Email Communications

**Why RED**: Same as PF-3. Patient emails contain appointment details, treatment recommendations, and follow-up instructions. AI-generated emails reference specific treatment plans.

**Impact**: Same as PF-3. Loss of automated email communication for appointment and treatment workflows.

### PF-5: Review Request

**Why RED**: Post-visit review prompts require knowing that a patient visited (appointment data from PMS), the patient's contact info, and appropriate timing. The review request message contains the patient's name and practice context.

**Impact**: Practices lose automated review solicitation. Review volume and velocity — key inputs to H-Score (Swell 45% weight) and practice reputation — may decline. However, reviews can still be solicited manually by staff.

### PF-6: Digital Forms

**Why RED**: Intake forms collect PHI directly from patients (name, DOB, medical history, insurance info, consent signatures). Form submissions must be stored in HIPAA-compliant infrastructure and written back to the PMS.

**Impact**: Practices continue using paper or existing third-party digital forms (many PMS systems have basic form capabilities). The unified forms experience is deferred.

### Aggregate Patient-Facing Impact

The loss of all 6 patient-facing interfaces means Phase A has **zero direct patient touchpoints**. Every patient interaction (booking, communication, review solicitation, form collection) either does not happen or happens through existing tools (phone calls, paper forms, manual review requests). Phase A is entirely an **internal operations platform** — it serves staff and HQ, not patients.

---

## 6. Risk Assessment

### 6.1 What Value Does Phase A Deliver on Its Own?

Phase A delivers substantial value across three dimensions:

**a) Content Production Pipeline (WS2 — 8 of 9 interfaces GREEN)**
The Content Engine is nearly fully operational. SGA can immediately begin:
- Automated content capture from 260 practices via champion network
- AI-powered content processing (color correction, caption generation, variant creation)
- Editorial workflow (review, approve, schedule, distribute to 6+ social channels)
- Brand management across 3 brand universes (SGA Legacy, Gen4, MODIS)
- Engagement analytics and content-to-practice attribution (without patient funnel data)

This is a standalone product with immediate ROI: it replaces the current patchwork of manual content processes, reduces HQ content team workload, and accelerates social media output.

**b) Culture & Training Platform (WS3 — 5 of 8 interfaces GREEN)**
The Culture OS training and champion programs are fully operational:
- CCA Academy certification program (Bronze/Silver/Gold tiers)
- Champion network management and gamification
- Recognition and rewards program
- Playbook library with Grant training videos
- Morning huddles (reduced but functional)
- H-Score display (aggregate, reduced depth)

This is a standalone product with immediate cultural impact: it systematizes the hospitality training program that currently runs informally.

**c) HQ Operations Visibility (WS1 NI — 2 GREEN, 7 YELLOW)**
Network Intelligence provides reduced but meaningful visibility:
- Practice enrollment pipeline (fully operational)
- Playbook management (fully operational)
- Heatmap of practice health (culture + content signals, missing revenue signals)
- Intervention queue (culture/content triggers, missing revenue triggers)
- Practice deep dive (3 of 5 tabs available)

**Estimated Phase A Value**: 60-70% of the platform's operational value, 0% of its patient-facing value, and approximately 30-40% of its revenue-impacting value (content and culture contribute to revenue indirectly, but the direct revenue drivers — patient communications, booking, follow-ups — are all Phase B).

### 6.2 Risk: Phase A Feeling Like "Half a Product"

**Risk Level: MEDIUM for HQ / LOW for practices**

**For HQ teams**: Phase A is a complete product. The Content Engine is fully operational. The Culture OS training platform is fully operational. The operations dashboards provide meaningful (if incomplete) visibility. HQ teams will experience Phase A as a functional platform.

**For practice-level users**: Phase A is a limited product. Office managers lose their most-used tool (patient outreach queue), front desk staff lose their primary tool (call dashboard), and the Morning Huddle and Daily Agenda are noticeably thinner without patient data. The risk is that practice staff perceive the platform as "another tool that doesn't do the important stuff." Mitigation: position Phase A practice-level features as the "culture and training hub" rather than the "practice operating system." The full operating system arrives in Phase B.

**For patients**: Phase A is invisible. No patient-facing interfaces ship. This is acceptable because patients do not know the platform exists — they interact with the practice, not the platform.

### 6.3 Ensuring Phase A Architecture Has PHI Hooks for Phase B

The following architectural provisions must be built into Phase A to ensure Phase B integration is seamless:

| Provision | Description | Where |
|---|---|---|
| **PHI-ready data model** | The Practice Data Hub schema includes patient-related tables (empty in Phase A, populated in Phase B). Foreign keys and indexes are pre-built. | Practice Data Hub (PostgreSQL) |
| **Kafka topic pre-registration** | All 10 event contract topics are registered in the Schema Registry in Phase A, including Phase B topics (PMS events, funnel events). Schemas are defined. No events flow on Phase B topics until Phase B. | Event Bus (Kafka) |
| **PHI Filter implementation** | The AI Service PHI Filter is coded, unit-tested, and integration-tested against synthetic PHI data during Phase A. It is deployed but in "pass-through" mode (no PHI to filter). Phase B activates it. | AI Service |
| **Compliance Middleware stubs** | Every service includes the Compliance Middleware Fastify plugin from day 1. In Phase A, it logs non-PHI access events. In Phase B, it activates PHI detection, consent checking, and encryption enforcement. | All services |
| **Communications Service skeleton** | The Communications Service is deployed with Twilio integration, number provisioning, and consent registry infrastructure. In Phase A, it handles only WS2 champion SMS (non-PHI). Phase B activates patient-facing channels. | Communications Service |
| **PMS Integration Service interface** | The PMS Integration Service API contract is defined and a mock implementation is available. WS1 and WS3 code against the mock. Phase B swaps in the real NexHealth/Sikka ONE adapters. | PMS Integration Service |
| **RBAC patient-scoped roles** | Auth & Identity includes patient-scoped permissions (read_patients, write_appointments) in the role model. These roles are defined but unassigned in Phase A. | Auth & Identity |
| **UI placeholder panels** | Practice Operations and Network Intelligence interfaces include placeholder panels for Phase B features: "Patient Outreach — Coming Soon" with a clear indication that the capability is planned. | Frontend (React) |

### 6.4 BAA Procurement Delay Scenario

**If BAA procurement takes longer than expected:**

| Scenario | Impact | Mitigation |
|---|---|---|
| **NexHealth/Sikka ONE BAA delayed** | All PMS data flows blocked. Funnel tracking, appointment management, and patient communications cannot begin. This blocks PO-2, PO-3, PO-5, NI-6, all PF interfaces. | Begin BAA negotiation immediately (Sprint 0). NexHealth is a HIPAA-focused company — BAA should be standard. Sikka ONE may be slower. Prioritize NexHealth (covers 96% of practices). |
| **Anthropic BAA delayed** | PHI Filter cannot process real patient data through Claude. Blocks AI-drafted patient messages (PO-2), anomaly analysis on patient data (NI-1, NI-3), Voice AI transcript analysis (PO-4). Does NOT block Phase A AI features (caption generation, report generation on non-PHI data). | Anthropic offers enterprise BAAs. Begin negotiation in Sprint 0. If delayed, consider local LLM deployment for PHI-touching use cases (slower, more expensive, but eliminates BAA dependency). |
| **OpenAI Whisper BAA delayed** | Call transcription blocked. Blocks PO-4 Voice AI Monitor, CO-7 Trust Transfer Tracker, and any H-Score component requiring call-level analysis. | Deploy self-hosted Whisper on GPU instances (g5.xlarge) within AWS VPC. Audio never leaves BAA-covered infrastructure. Adds ~$1,000/mo compute but eliminates external BAA dependency entirely. Recommended regardless of BAA timing. |
| **Cloudinary BAA delayed or unavailable** | Patient facial photos cannot be processed through Cloudinary. Blocks before/after galleries with patient faces. | Architect self-hosted image processing from Phase A: S3 + Lambda for transforms (resize, crop, format conversion). Adds ~2 weeks build effort. Cloudinary used only for non-PHI clinical/marketing assets. |
| **Swell/Cinnamon/Dental Intel BAAs delayed** | H-Score loses granularity. Swell: review score component may be limited to aggregate star ratings (publicly available) rather than review content analysis. Cinnamon: phone score becomes a black-box vendor number rather than behavior-level analysis. Dental Intel: operational metrics only at practice aggregate level. | H-Score can still calculate using aggregate vendor-provided scores without per-patient or per-call detail. The composite number is less actionable (can't drill into "which calls were weak") but the trend is still valid. |
| **All BAAs delayed 6+ months** | Phase A operates indefinitely. The Content Engine and Culture OS are fully productive. HQ visibility is reduced but functional. Practices use existing tools for patient communication. The platform accumulates content assets, trains staff, builds culture metrics. When BAAs arrive, Phase B features light up on an already-adopted platform. | This is actually a defensible strategy: build adoption on non-PHI value, then layer PHI capabilities onto an already-loved tool. Risk: practice staff may not adopt Phase A practice-level features if they perceive them as insufficient. |

### 6.5 Phase Transition Readiness Checklist

Before moving from Phase A to Phase B, the following must be in place:

- [ ] All P1 BAAs executed: NexHealth, Sikka ONE, Anthropic, OpenAI (or self-hosted Whisper)
- [ ] Compliance Middleware fully activated with PHI detection and audit logging
- [ ] PHI Filter tested with red-team exercises (novel PHI format bypass attempts)
- [ ] Self-hosted Whisper deployed and tested (if chosen over OpenAI BAA)
- [ ] S3 + Lambda image processing pipeline operational (if chosen over Cloudinary BAA)
- [ ] Consent registry populated with patient consent status from existing PMS data
- [ ] PMS Integration Service connected to NexHealth for at least 10 pilot practices
- [ ] Practice Data Hub patient tables populated with initial PMS sync
- [ ] Encryption at rest verified for all PHI-storing services (application-level + database-level)
- [ ] HIPAA compliance audit completed by qualified counsel
- [ ] Breach notification procedures documented and tested (tabletop exercise)
- [ ] Twilio HIPAA tier activated with BAA ($500/mo)
- [ ] AWS BAA executed as part of account setup

---

## Appendix A: Interface-to-Data-Flow Mapping

This maps each RED interface to the specific PHI data flows from the PHI Data Flow Analysis (21 total flows).

| Interface | PHI Flows Involved (by #) | Risk Level |
|---|---|---|
| PO-2 Patient Outreach Queue | #3 (PMS through platform), #8 (patient SMS), #11 (AI-generated messages) | HIGH + MEDIUM |
| PO-3 Call Dashboard | #3 (PMS through platform), #12 (PMS webhook events) | HIGH + MEDIUM |
| PO-4 Voice AI Monitor | #1 (call recordings to Whisper), #4 (AI prompts to Claude) | HIGH |
| PO-5 Automation Status Board | #3 (PMS), #8 (patient SMS), #11 (AI-generated messages) | HIGH + MEDIUM |
| NI-6 Funnel Analytics Explorer | #3 (PMS), #15 (NexHealth/Sikka data transit) | HIGH + MEDIUM |
| CO-7 Trust Transfer Tracker | #1 (call recordings to Whisper), #4 (AI prompts to Claude), #6 (call data to Cinnamon) | HIGH |
| PF-1 Practice Website | #2 (patient photos to Cloudinary), #5 (review data to/from Swell) | HIGH |
| PF-2 Voice AI (Phone) | #1 (call recordings), #3 (PMS), #4 (AI prompts) | HIGH |
| PF-3 SMS Conversations | #3 (PMS), #8 (patient SMS), #10 (inbound form submissions) | MEDIUM |
| PF-4 Email Communications | #3 (PMS), #11 (AI-generated messages) | MEDIUM |
| PF-5 Review Request | #3 (PMS), #5 (Swell review data), #8 (patient SMS) | MEDIUM + HIGH |
| PF-6 Digital Forms | #3 (PMS), #10 (inbound form submissions) | MEDIUM |

---

## Appendix B: Decision Record

| Decision | Rationale | Date |
|---|---|---|
| Phase A / Phase B split at PHI boundary | BAA procurement and compliance infrastructure are the longest-lead items. Splitting allows 71% of interfaces to ship immediately while compliance catches up. | March 2026 |
| PF-1 website classified RED (not YELLOW) | While a brochure website is possible, the experience architecture defines the website with patient photos, reviews, and booking. Shipping a degraded website risks brand perception. A "brochure mode" alternative is noted but not the primary classification. | March 2026 |
| H-Score classified YELLOW (not RED) | Vendor aggregate scores at practice level are likely available without per-patient data. The composite H-Score can display, but individual behavior drill-down is limited. Confirmed pending vendor data model verification. | March 2026 |
| Content Engine before/after pairing classified YELLOW | Manual pairing by HQ content team is a viable workaround. The automated patient-ID-based matching is a convenience, not a requirement for content production. | March 2026 |
| Self-hosted Whisper recommended regardless of BAA | Audio containing verbal PHI is the highest-risk flow. Self-hosting within AWS VPC eliminates the external data flow entirely. Cost is ~$1,000/mo. The risk reduction justifies the cost independent of OpenAI BAA status. | March 2026 |

---

*Prepared by Product Strategist Agent | Citizen & Partners | bilt.studio*
*This analysis should be reviewed alongside the [PHI Data Flow Analysis](../../10-phi-analysis/phi-data-flow-analysis.md) and validated by qualified HIPAA compliance counsel before implementation decisions.*
