# QC-2: Prototype Lead Review — Testability & Development Readiness

**Gate**: QC-2 — Prototype Lead Assessment
**Date**: 2026-03-14
**Reviewer**: Prototype Lead Agent
**Scope**: Assess testability, development readiness, simplification opportunities, PWA feasibility, and the 42-interface surface area concern for the consolidated Experience Architecture.

**Input Documents Reviewed**:
- `consolidation-feature-map.md` (72 features across 3 workstreams)
- `technical-architecture.md` (10 shared services, 10 event contracts, 7 ADRs, build sequence)
- `qc-1-pain-point-validation.md` (34 pain points across 7 actors, all preserved)
- All 6 original sprint architectures (`04-architect/` in each sprint folder)

---

## Assessment 1: Core Interaction Testability

### Testable Immediately (Static Prototype / Mock Data)

These interactions can be validated with an HTML/React prototype using hardcoded or mock data. No backend infrastructure required.

| # | Interaction | Actor | Surface | What the Prototype Tests |
|---|------------|-------|---------|-------------------------|
| 1 | Practice Nerve Center daily agenda | Office Manager | Practice Ops | Does the "20-minute daily" format work? Are 3 action cards the right number? Is the information hierarchy correct? |
| 2 | Executive Dashboard network funnel view | CEO/VP Ops | Network Intelligence | Can executives find struggling practices in < 2 clicks? Does the funnel visualization communicate clearly? |
| 3 | Regional drill-down from network to practice | Regional Director | Network Intelligence | Does two-click drill-down work? Is the region > practice hierarchy intuitive? |
| 4 | Content capture SMS flow (prompt > photo > confirm) | Champion / Clinical Staff | Content Engine | Is the zero-friction capture flow actually zero-friction? Will staff respond to SMS prompts? |
| 5 | HQ content QC queue (approve/reject/edit) | Content Team | Content Engine | Is the approval workflow fast enough for batch review? |
| 6 | Morning Huddle content display | Office Manager / Clinical | Culture OS | Does 5 minutes of huddle content feel right? Is the Dental Intel integration display clear? |
| 7 | CCA Academy module structure (Bronze/Silver/Gold) | Clinical Staff | Culture OS | Is the certification path motivating? Is the module structure clear? |
| 8 | H-Score practice detail view | Regional Director | Network Intelligence | Do the 3 sub-scores (Cinnamon 35%, Swell 45%, Dental Intel 20%) make sense to users? |
| 9 | HQ Ops queue (14 practices ranked by revenue impact) | HQ Operations | Network Intelligence | Does the prioritization feel right? Are AI summaries useful? |
| 10 | Champion leaderboard and streak counter | Champion | Culture OS | Is the gamification motivating or annoying? |
| 11 | Studio request intake form (email > conversational) | Practice Manager | Content Engine | Does the conversational intake feel natural? |
| 12 | CD review Adaptive Card (58-sec tap-approve) | Creative Director | Content Engine | Can the CD actually review and approve in under 60 seconds? |
| 13 | Practice enrollment wizard steps | HQ Operations | Practice Ops | Is the enrollment flow intuitive? Are the right questions asked at the right time? |
| 14 | End-of-day scorecard | Office Manager | Practice Ops | Does the scorecard capture what matters? |

**Count: 14 interactions testable immediately.**

### Testable With Simulated Backend (API Stubs / Seeded Database)

These require API stubs returning realistic data, but do not need live external integrations.

| # | Interaction | Actor | Requires |
|---|------------|-------|----------|
| 15 | Concierge Agent conversation flow (enrollment, change mgmt) | Office Manager | Claude API stub returning scripted responses |
| 16 | AI Analysis Engine alerts and anomaly cards | VP Ops | Seeded ClickHouse with anomaly data |
| 17 | Automation Engine confirmation sequence preview | Office Manager | Mock cadence engine showing sequence timeline |
| 18 | Playbook Editor (HQ designs cadence sequences) | HQ Operations | Mock playbook data model |
| 19 | Brand health scoring drill-down (0-100) | HQ Operations | Seeded scoring data across practices |
| 20 | Content production pipeline status view | Content Team | Mock pipeline stages with sample media |
| 21 | Graduation Manager trust level display | HQ Operations | Mock trust level data per practice |
| 22 | Channel Router selection explanation | HQ Operations | Mock routing decision tree |
| 23 | Practice-type playbook selection (5 types) | Content Team | Mock playbook library |
| 24 | Weekly summary generator output | Office Manager | Mock summary content |

**Count: 10 interactions testable with stubs.**

### Require Live Infrastructure (Cannot Prototype Without Backend)

| # | Interaction | Actor | Why It Needs Infrastructure |
|---|------------|-------|-----------------------------|
| 25 | Voice AI Agent phone conversation | Patient / Front Desk | Requires Twilio Voice + Media Streams + Claude real-time |
| 26 | PMS appointment data flowing into Nerve Center | Office Manager | Requires NexHealth webhook integration |
| 27 | SMS send/receive with consent checking | Patient | Requires Twilio + consent registry |
| 28 | Content auto-distribution to Instagram/Facebook | Content Team | Requires Meta Graph API + publishing permissions |
| 29 | H-Score real-time calculation from Cinnamon/Swell/Dental Intel | Culture Team | Requires 3 external API integrations |
| 30 | Cross-workstream event flow (e.g., content published > storefront update) | All | Requires Kafka + multiple services |
| 31 | PHI filter round-trip through AI Service | All | Requires Claude API + Redis PHI mapping |
| 32 | Nightly PMS sync via Sikka ONE | Content Team | Requires Sikka ONE API + Temporal workflow |

**Count: 8 interactions require live infrastructure.**

### Testability Verdict

**24 of 32 core interactions (75%) are testable without live infrastructure.** This is a strong ratio. The prototype phase can validate the majority of user-facing value before committing to backend development.

---

## Assessment 2: Quality Gate Criteria — Surface Graduation

Each surface must pass these criteria before moving from prototype to development investment.

### Surface 1: Practice Operations (18 features)

| Gate | Criterion | Pass/Fail Test |
|------|-----------|----------------|
| G1.1 | Office Manager validates daily workflow | 3 office managers complete the Practice Nerve Center daily flow in < 20 minutes using mock data and rate it "useful" or "essential" (not "confusing" or "irrelevant") |
| G1.2 | Action card prioritization makes sense | In 5 test scenarios with seeded anomaly data, office managers agree with the top-3 action card ranking at least 4 out of 5 times |
| G1.3 | Automation sequence is understandable | Office manager can explain what the confirmation/follow-up sequence will do to a patient without reading documentation |
| G1.4 | Concierge Agent conversation passes Turing test for helpfulness | 3 office managers interact with the agent (Claude stub) and rate it more helpful than calling HQ for the same question at least 70% of the time |
| G1.5 | Enrollment flow < 30 minutes for new practice | HQ operations team can complete enrollment wizard for a test practice in under 30 minutes |

### Surface 2: Network Intelligence (12 features)

| Gate | Criterion | Pass/Fail Test |
|------|-----------|----------------|
| G2.1 | Executive finds struggling practice in < 3 clicks | CEO/VP Ops can navigate from network overview to a specific struggling practice's detail view in 3 or fewer clicks |
| G2.2 | AI alert is actionable | 5 sample AI alerts are shown to VP Ops. At least 4 out of 5 generate a clear next action (not "I don't know what to do with this") |
| G2.3 | HQ Ops queue reduces triage time | HQ operations team estimates the queue would save > 30 minutes per day vs. current manual process |
| G2.4 | H-Score is understood | Regional director can explain what the H-Score means and what would improve it without reading documentation |
| G2.5 | Cross-workstream data makes sense | Executive viewing a practice detail page with funnel + content + culture data finds the combined view more useful than separate views |

### Surface 3: Content Engine (12 features)

| Gate | Criterion | Pass/Fail Test |
|------|-----------|----------------|
| G3.1 | SMS capture flow < 60 seconds end-to-end | Champion receives SMS prompt, opens camera, takes photo, sends via SMS, receives confirmation — all within 60 seconds |
| G3.2 | AI-processed content is publication-ready | 7 out of 10 AI-processed content pieces (photo enhanced + caption generated) are approved by HQ with no edits |
| G3.3 | CD review flow < 60 seconds | Creative Director can review and approve/reject a design via Teams Adaptive Card in under 60 seconds |
| G3.4 | Brand universe separation is clear | Content generated for SGA Legacy, Gen4, and MODIS practices is visually distinguishable and brand-appropriate |
| G3.5 | Distribution pipeline is traceable | Content team can track an asset from capture to publication across all channels without losing visibility |

### Surface 4: Culture OS (8 features)

| Gate | Criterion | Pass/Fail Test |
|------|-----------|----------------|
| G4.1 | Morning Huddle < 5 minutes | Practice team can consume the huddle content in under 5 minutes |
| G4.2 | CCA Academy is motivating | 3 clinical staff members rate the Bronze tier module as "worth doing" (not "corporate waste of time") |
| G4.3 | H-Score is trusted | Regional directors believe the H-Score reflects actual hospitality quality when compared to their subjective assessment of practices they know well |
| G4.4 | Champion role is not overwhelming | Champions confirm the combined culture + content role is manageable within their existing workload |

### Surface 5: Patient-Facing (5 features)

| Gate | Criterion | Pass/Fail Test |
|------|-----------|----------------|
| G5.1 | Website template loads < 2 seconds | All 5 specialty templates achieve < 2s LCP on mobile |
| G5.2 | Concierge Agent (patient-facing chat) resolves common queries | Agent can answer hours, insurance, directions, and service questions with > 90% accuracy |
| G5.3 | Voice AI Agent is not uncanny valley | 5 test callers rate the voice interaction as "natural" or "acceptable" (not "robotic" or "creepy") |
| G5.4 | Automated messages feel personal, not spammy | 10 sample patient messages are rated "would read" by at least 7 of 10 test reviewers |

---

## Assessment 3: Simplification Opportunities

### Over-Designed / Defer to v2

| # | Feature | Current State | Recommendation | Rationale |
|---|---------|--------------|----------------|-----------|
| S1 | **Voice AI Agent (1.25)** | Full conversational voice with real-time PMS booking | **DEFER booking to v2.** v1: voicemail capture + SMS callback within 15 minutes. | Voice AI with real-time PMS write is the single highest-risk, highest-complexity feature. It requires Twilio Voice + Media Streams + Claude real-time + NexHealth write API all working in concert with sub-second latency. The core pain point (47-hour response time) is solved by fast SMS callback, not by a robot booking the appointment. Ship the hard version in v2 after the PMS integration layer is proven. |
| S2 | **ML booking attribution (2.25)** | Full ML model correlating content to bookings | **DEFER entirely to v2.** | Requires months of training data to be statistically meaningful. Cannot deliver value in v1 because there is no historical content-to-booking data. Build the event pipeline (capture content distribution events + funnel events), defer the ML model. |
| S3 | **React Native app for content capture (2.22)** | Native iOS/Android app | **DEFER to v2. SMS capture is v1.** | The SMS/WhatsApp capture flow (2.14) is the v1 interface. It requires zero app installation, zero training, zero app store review. A native app adds distribution friction. Build it only if SMS capture adoption exceeds 60% and users request richer capture tools. |
| S4 | **Testimonial kiosks + Insta360 cameras (2.24)** | Hardware-based passive capture | **DEFER to v3.** | Hardware procurement, installation, and maintenance for 260+ practices is a logistics project, not a software feature. The core content capture value is delivered by SMS-prompted human capture. |
| S5 | **Event Mode for 4Front/Pikos courses (2.27)** | Special capture mode for events | **DEFER to v2.** | Events happen a few times per year. Manual capture with post-event processing is fine for v1. |
| S6 | **AI Note-Taker Bridge (Comms Engine Layer 1)** | Ingests in-room AI note-taker transcriptions | **DEFER to v2.** | Depends on practices having AI note-takers deployed. Adoption is low. The comms automation works without this signal. |
| S7 | **Network Learning Engine monthly optimization cycles (1.26)** | Cross-practice ML optimization | **Simplify to v1: manual pattern reports.** | The learning loop requires 3+ months of data across 260 practices to produce statistically significant insights. v1 should generate reports that HQ reviews manually. Automated optimization is v2. |
| S8 | **3 brand universes with separate Canva workspaces (2.12)** | Full brand separation in Canva | **Simplify: start with Gen4 only.** | Gen4 is the active brand. SGA Legacy practices are being rebranded. MODIS is a small subset. Build templates for Gen4 first. Add legacy/MODIS templates when the pipeline is proven. |

### What These Deferrals Save

| Metric | Before Deferrals | After Deferrals | Reduction |
|--------|-----------------|-----------------|-----------|
| Interfaces | 42 | ~34 | -19% |
| External API integrations | 15+ | 10-12 | -20-30% |
| Hardware dependencies | 2 (kiosks, cameras) | 0 | -100% |
| Highest-risk feature (Voice AI booking) | In v1 critical path | Degraded gracefully | Risk reduced |
| Native app development | React Native iOS+Android | None (SMS only) | -1 codebase |

### What Must NOT Be Simplified

| Feature | Why It Stays |
|---------|-------------|
| Practice Nerve Center (1.18) | Core daily value prop for office managers. The "20-minute daily" is the adoption hook. |
| Executive Dashboard (1.17) | Executives need visibility on day one. This is what makes the sale. |
| SMS content capture (2.14) | Zero-friction capture is the Content Engine's core innovation. |
| H-Score Engine (3.03) | Culture OS has no value without measurement. |
| Morning Huddle OS (3.01) | The daily ritual IS the culture platform. |
| Shared PMS Integration (1.13/1.28) | Everything depends on PMS data. This is the keystone. |
| Communications Service (shared) | Patient comms + content capture both need Twilio. Centralize from day one. |
| Compliance/Audit Layer (1.8) | HIPAA is not optional. Building it in later is 3-5x more expensive (per ADR-03). |

---

## Assessment 4: PWA Feasibility

### The Proposal

A single PWA with 7 screens serving staff across all 3 workstreams:
1. Today (Practice Nerve Center daily agenda)
2. Patients (action cards, follow-up queue)
3. Calls (missed calls, Voice AI status)
4. Automations (sequence status, approval queue)
5. Storefront (website health, content preview)
6. Huddle (morning huddle content)
7. Capture (content capture prompt + camera)

### Feasibility Assessment

**Technically feasible: YES.** A PWA can render 7 screens. The technology is not the concern.

**Adoption risk: MEDIUM-HIGH.** Here is why:

| Risk Factor | Assessment |
|-------------|-----------|
| **Screen count** | 7 screens is borderline. Mobile apps with > 5 nav items suffer discoverability problems. Users will gravitate to 2-3 screens and ignore the rest. |
| **Cross-workstream confusion** | "Today" (WS1) and "Huddle" (WS3) and "Capture" (WS2) serve different purposes from different teams. A front desk person does not need "Capture." A champion does not need "Calls." The PWA shows features irrelevant to most roles. |
| **Notification overload** | Three workstreams sending push notifications to one app creates noise. Users will disable notifications entirely. |
| **Offline capability** | PWAs on iOS still have limited push notification support and no background sync. Content capture (taking a photo, uploading) may fail in poor-connectivity dental offices. |
| **Install friction** | PWAs require "Add to Home Screen" which most users do not know how to do. Adoption will be lower than a native app, but the SMS-first strategy mitigates this. |

### Recommendation

**Split the PWA into 2 focused apps, or use role-based screen filtering:**

**Option A (Preferred): Role-filtered PWA.** Keep a single PWA codebase but show only the screens relevant to each user's role:
- Office Manager sees: Today, Patients, Calls, Automations (4 screens)
- Champion sees: Huddle, Capture, Today (3 screens)
- Front Desk sees: Today, Patients, Calls (3 screens)
- HQ never uses the PWA (they use the full web dashboard)

This is technically simple (role from Auth & Identity determines visible screens) and eliminates the "7 screens for everyone" problem.

**Option B: SMS-first, PWA-later.** For v1, deliver everything via SMS:
- Daily agenda summary via SMS
- Action cards via SMS with tap-to-call/tap-to-text
- Content capture prompts via SMS
- Huddle reminders via SMS with link to web content

Build the PWA in v1.5 only after validating that staff engage with SMS-delivered workflows. This avoids building a PWA that nobody installs.

### PWA Verdict

The PWA as described (7 screens, all roles) is **not recommended in its current form.** Role-filtered PWA (Option A) or SMS-first (Option B) are both viable alternatives that reduce adoption risk without losing functionality.

---

## Assessment 5: The 42-Interface Concern

### Interface Inventory

Based on the architecture documents, the 42 interfaces break down as follows:

| Surface | Interfaces | Examples |
|---------|-----------|----------|
| **Practice Operations** | ~18 | Practice Nerve Center, action cards, automation status, enrollment wizard, Concierge Agent chat, Voice AI, comms approval, QC queues, batch ops, storefront manager, change management, playbook viewer, escalation queue, scorecard, weekly summary, patient timeline, channel router view, graduation status |
| **Network Intelligence** | ~12 | Executive Dashboard (network/region/practice), AI alerts, HQ Ops queue, funnel drill-down, anomaly detail, H-Score integration, content performance, reporting, practice comparison, regional heatmap, intervention tracker, compliance dashboard |
| **Content Engine** | ~8 | SMS capture flow, studio request intake, AI processing pipeline view, QC approval queue, distribution dashboard, brand health scoring, template manager, analytics |
| **Culture OS** | ~6 | Morning Huddle, CCA Academy, champion leaderboard, H-Score detail, recognition feed, playbook library |
| **Patient-Facing** | ~5 | 5 specialty website templates, chat widget, online booking, review response, content gallery |
| **Shared/Admin** | ~3 | Auth/login, user management, system health |

**Total: ~52 distinct views/screens** (the 42 cited in the prompt likely excludes admin and patient-facing templates).

### Is 42 Manageable?

**No. Not for a single release. Yes, if phased aggressively.**

| Concern | Assessment |
|---------|-----------|
| **Development effort** | At ~2-3 weeks per interface (design + implement + test), 42 interfaces = 84-126 engineer-weeks = ~6-9 months for a 5-person frontend team. This is the entire 12-week Phase 3 timeline and beyond. |
| **Design consistency** | 42 interfaces across 5 teams require a shared design system. Without one, each team will create divergent UI patterns. The technical architecture does not mention a design system. |
| **Testing surface** | 42 interfaces x 7 actor roles = 294 role-interface combinations that need RBAC testing. Many will be "access denied" but all must be verified. |
| **Cognitive load** | Even after the PWA is role-filtered, the web dashboards will have deep navigation trees. Users who switch between Practice Ops and Network Intelligence will encounter different navigation paradigms (time-based vs. scope-based). |

### Phasing Recommendation

**Phase the 42 interfaces into 3 tiers:**

#### Tier 1: Launch Minimum (16 interfaces) — Months 3-4
The interfaces that deliver core daily value. These are the ones that make the sale and drive adoption.

| Surface | Tier 1 Interfaces | Count |
|---------|-------------------|-------|
| Practice Ops | Practice Nerve Center (daily agenda), action cards, automation status, scorecard | 4 |
| Network Intelligence | Executive Dashboard (network + region + practice drill-down), AI alerts, HQ Ops queue | 3 |
| Content Engine | SMS capture flow, QC approval queue, distribution status | 3 |
| Culture OS | Morning Huddle, H-Score detail, champion leaderboard | 3 |
| Patient-Facing | GP website template (1 of 5), chat widget | 2 |
| Shared | Login | 1 |

#### Tier 2: Adoption Expansion (14 interfaces) — Months 5-7
Interfaces that deepen value for users already engaged.

| Surface | Tier 2 Interfaces |
|---------|-------------------|
| Practice Ops | Concierge Agent chat, enrollment wizard, comms approval, Voice AI (degraded: SMS callback), QC queues, batch ops |
| Network Intelligence | Funnel drill-down, anomaly detail, reporting, practice comparison |
| Content Engine | Studio request intake, AI pipeline view, template manager |
| Culture OS | CCA Academy (Bronze tier) |

#### Tier 3: Full Platform (12 interfaces) — Months 8-12
Interfaces for power users and edge cases.

| Surface | Tier 3 Interfaces |
|---------|-------------------|
| Practice Ops | Change management, playbook viewer, escalation queue, patient timeline, weekly summary |
| Network Intelligence | Regional heatmap, intervention tracker, compliance dashboard |
| Content Engine | Brand health drill-down, analytics |
| Culture OS | Recognition feed, playbook library, CCA Silver/Gold tiers |
| Patient-Facing | Remaining 4 specialty templates |

### Critical Missing Item: Design System

The technical architecture defines shared backend infrastructure exhaustively (10 services, event bus, compliance layer) but **does not mention a shared frontend design system**. With 5 surfaces built by 4-5 teams, this is a gap. A shared component library (buttons, cards, tables, charts, navigation patterns, color tokens, typography scale) must be established in Phase 1 alongside the backend infrastructure.

**Recommendation**: Add a "Design System" deliverable to Phase 1 (Weeks 1-4). Publish a shared React component library as an npm package. All surfaces consume the same components. This prevents visual fragmentation across 42 interfaces and reduces per-interface build time by ~30%.

---

## Summary Scorecard

| Dimension | Score (1-5) | Notes |
|-----------|-------------|-------|
| **Core interaction testability** | 4 | 75% testable without infrastructure. Strong prototype potential. |
| **Quality gate clarity** | 4 | Gates are specific and measurable. Each surface has clear pass/fail. |
| **Simplification maturity** | 3 | 8 features should be deferred. Voice AI booking is over-scoped for v1. |
| **PWA feasibility** | 2 | 7 screens for all roles is a design problem. Needs role filtering or SMS-first. |
| **42-interface manageability** | 2 | Too many for a single release. Needs aggressive 3-tier phasing. |
| **Design system readiness** | 1 | Not addressed in the architecture. Must be added to Phase 1. |
| **Backend architecture quality** | 5 | Shared infrastructure is thorough, well-documented, and addresses all QC-1 items. |
| **Cross-workstream integration** | 5 | 10 event contracts are fully specified with schemas. Kafka topology is sound. |

**Weighted Average: 3.4 / 5.0**

---

## VERDICT: NEEDS SIMPLIFICATION

The Experience Architecture is **not fundamentally flawed** — the backend infrastructure design is excellent, the pain point coverage is complete, and the cross-workstream integration contracts are well-specified. The QC-1 pass was earned.

However, the frontend surface area is over-scoped for a single development investment. Specifically:

### Must-Fix Before Development Investment (4 items)

1. **Defer Voice AI booking to v2.** Ship v1 with SMS callback (solves the same pain point at 20% of the engineering cost and risk). Voice AI with real-time PMS booking is the single highest-risk feature in the entire architecture.

2. **Phase the 42 interfaces into 3 tiers.** Tier 1 (16 interfaces) is the launch minimum. Tier 2 (14 interfaces) is adoption expansion. Tier 3 (12 interfaces) is full platform. Do not attempt to build all 42 for a single release.

3. **Add a shared design system to Phase 1.** The backend has 10 shared infrastructure services. The frontend has zero shared infrastructure. This asymmetry will produce visual fragmentation across 5 surfaces. A shared React component library must ship in Phase 1 alongside the backend services.

4. **Fix the PWA strategy.** A single 7-screen PWA for all roles will suffer low adoption. Either implement role-based screen filtering (preferred) or go SMS-first for v1 and build the PWA in v1.5.

### Recommended But Not Blocking (4 items)

5. Defer React Native content capture app (2.22) to v2. SMS capture is the v1 interface.
6. Defer ML booking attribution (2.25) to v2. Build the event pipeline, defer the ML model.
7. Defer testimonial kiosks and hardware (2.24) to v3.
8. Start with Gen4 brand universe only. Add SGA Legacy and MODIS templates after pipeline is proven.

### What Passes Without Changes

- Shared infrastructure architecture (all 10 services)
- Event contracts (all 10 fully specified)
- Build sequence and team structure
- HIPAA compliance as foundational layer
- PMS dual-strategy (NexHealth + Sikka ONE)
- LLM strategy (Claude primary, Whisper for audio)
- WS1 split into Practice Operations + Network Intelligence (ADR-06)

### Bottom Line

The architecture is a **strong foundation being asked to carry too much surface area at once**. The 4 must-fix items reduce v1 scope by ~25% while preserving 100% of the core value propositions for all 7 actor types. After these changes, the architecture is ready for development investment.

---

*This QC-2 review should be addressed by the Product Designer before the architecture advances to development specification. The 4 must-fix items are non-negotiable for a PASS verdict on re-review.*
