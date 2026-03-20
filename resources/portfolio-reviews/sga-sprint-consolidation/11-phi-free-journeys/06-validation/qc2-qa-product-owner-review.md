# QC Gate 2: QA Engineer + Product Owner Review — PHI-Free User Journey Redesign

**Reviewers**: QA Engineer Agent + Product Owner Agent
**Date**: 2026-03-20
**Documents Reviewed**: PHI Scope Analysis, PHI-Free Experience Architecture, WS1 Practice Platform Journey, WS2 Content Engine Journey, WS3 Culture OS Journey, PHI-Free Build Plan, PHI-Free Technical Review, QC1 Design Director Review, Consolidated Epic & User Story Inventory
**Gate**: QC2 — Cross-document consistency, acceptance criteria mapping, defect classification, backlog completeness, value delivery assessment, scope boundary validation

---

# Part 1: QA Engineer Review

## 1. Cross-Document Consistency Check

### 1.1 Interface Phase Classification Consistency

I verified the Phase A vs. Phase B classification of every interface across the scope analysis, experience architecture, and journey documents.

**Scope Analysis states**: 19 GREEN, 11 YELLOW, 12 RED (total 42). Then revises in detailed table to 17 GREEN, 13 YELLOW, 12 RED.

**Experience Architecture states**: 30 Phase A interfaces, 12 deferred. Breakdown: PO 5 of 9, NI 8 of 10, CE 8 of 9, CO 8 of 8, PF 1 of 6.

**Discrepancy FOUND — MAJOR (D-1): Scope Analysis vs. Experience Architecture on multiple interfaces.**

| Interface | Scope Analysis Classification | Experience Architecture Classification | Conflict? |
|-----------|-------------------------------|---------------------------------------|-----------|
| PO-6 Storefront Manager | YELLOW (reduced) | **Unchanged** | YES — Scope says YELLOW (missing patient review integration, before/after gallery). Exp arch says "No change needed" and lists it as unchanged. The experience architecture is correct for Phase A (storefront works on practice profile + brand assets); scope analysis over-counted the PHI dependency. |
| PO-7 Content Capture Prompt | GREEN | **Modified** (in exp arch, listed as PO-7 Modified) | MINOR — Scope says fully GREEN; exp arch correctly notes it is modified (brand-driven briefs replace procedure-driven). Classification difference: scope says no change needed, exp arch says moderate change. Exp arch is more precise. |
| PO-8 Morning Huddle Screen | YELLOW | **Unchanged** (exp arch) | YES — Scope says YELLOW (missing funnel impact stories). Exp arch says "No change needed" because Culture OS (CO-1) handles the huddle display, and the experience architecture treats PO-8 as the PWA reference to CO-1 which is "Modified" (not PO-8 itself). This is a labeling inconsistency, not a real conflict. |
| CO-5 H-Score Dashboard | YELLOW (in scope analysis) | **Unchanged** (exp arch) | YES — Scope analysis classifies CO-5 as YELLOW citing concerns about Cinnamon phone scores derived from patient interactions. Experience architecture says "No change needed" because H-Score uses aggregate vendor scores, not individual patient data. The experience architecture's position is correct (confirmed in WS3 journey and tech review). Scope analysis was overly conservative. |
| CO-7 Trust Transfer Tracker | RED (in scope analysis) | **Unchanged** (exp arch) | YES — Scope analysis says RED (cannot function without PHI). Experience architecture says "Unchanged" noting aggregate scores are available; only individual call drill-down is deferred. WS3 journey confirms aggregate scores work in Phase A. Scope analysis was wrong to classify CO-7 as RED. |
| CE-1 Capture Bot | GREEN (scope analysis) | **Modified** (exp arch) | MINOR — Scope says fully GREEN; exp arch says modified (SMS/WhatsApp delivery with brand content). Both agree it ships in Phase A. |
| CE-2 Capture App | GREEN (scope analysis) | **Modified** (exp arch) | MINOR — Scope says GREEN; exp arch says modified (before/after pairing disabled). Both agree it ships. |
| CE-3 Editorial Queue | YELLOW (scope analysis) | **Unchanged** (exp arch) | YES — Scope says YELLOW (missing patient-mapped B/A pairing). Exp arch says unchanged (pipeline is content-type-agnostic). Exp arch is correct. |
| CE-9 Event Mode Console | GREEN (scope analysis) | **Deferred** (exp arch) | YES — Scope analysis classifies CE-9 as GREEN. Experience architecture defers it to Phase B citing clinical photography consent workflows. The exp arch position is more defensible given that events like 4Front courses involve patient case presentations. |
| PF-1 Practice Website | RED (scope analysis) | **Available** (exp arch) | YES — Scope says RED (requires B/A galleries, patient reviews, booking). Exp arch says available (website built from practice data + brand assets; booking links to NexHealth hosted page). Exp arch is correct for Phase A — the website works without patient-specific content. |
| NI-9 Compliance Dashboard | YELLOW (scope analysis) | **Modified/Reframed** (exp arch) | Consistent — both agree it ships as a compliance readiness dashboard. |

**Summary of GREEN/YELLOW/RED discrepancies:**

The scope analysis was written first and was more conservative. The experience architecture refined the classifications with deeper analysis. The net result:

| Document | Phase A Count | Deferred Count |
|----------|--------------|----------------|
| Scope Analysis (exec summary) | 30 (19G + 11Y) | 12 RED |
| Scope Analysis (detailed table) | 30 (17G + 13Y) | 12 RED |
| Experience Architecture | 30 | 12 |

The total counts agree (30 Phase A, 12 deferred), but the GREEN/YELLOW breakdown and individual interface classifications have 7 discrepancies. The experience architecture is the authoritative document and should be treated as canonical. The scope analysis needs a reconciliation pass to align its per-interface classifications with the experience architecture's final decisions.

**Severity**: MAJOR — Development teams referencing the scope analysis will get different requirements than those referencing the experience architecture.

---

### 1.2 Kafka Event Flow Consistency

I verified Kafka events referenced across all documents.

| Event | Scope Analysis | Exp Arch | WS Journeys | Build Plan | Tech Review | Consistent? |
|-------|---------------|----------|-------------|-----------|-------------|-------------|
| `ws3.hscore-calculated` | Phase A | Phase A (Sec 4.1) | WS3 Step 1.1, WS1 Exec Dashboard | A2 produced, A3 consumed | Contract 2: Phase A | YES |
| `ws3.huddle-completed` | Phase A | Not explicitly listed | WS3 Step 1.2 | Implied in A2 | Not in 10 contracts | MINOR gap — event referenced in journeys but not in the 10 cross-workstream contracts. Appears to be an intra-WS3 event consumed by WS1. |
| `ws3.training-completed` | Phase A | Phase A | WS3 Step 3.3 | A2 produced, A3 consumed | Contract 9: Phase A | YES |
| `ws3.recognition-created` | Not listed | Not in cross-WS events | WS3 Step 4.1 | Not listed | Not listed | OK — intra-WS3 event, not cross-workstream |
| `ws3.champion-assigned` | Phase A | Phase A | WS3 Step 9.2 | A2 produced, A3 consumed | Contract 1: Phase A | YES |
| `ws2.capture-media-received` | Phase A | Phase A (Sec 4.1 KPI) | WS2 Step 2.3 | Implied in A2 | Not in 10 contracts | MINOR gap — referenced as KPI data source but not in the formal 10 contracts |
| `ws2.content-asset-created` | Phase A | Not in cross-WS events | WS2 Step 2.3 | Not listed | Not listed | OK — intra-WS2 event |
| `ws2.content-asset-approved` | Phase A | Phase A | WS2 Step 5.1 | A2 produced, A3 consumed | Contract 6: Phase A | YES |
| `ws2.content-asset-published` | Phase A | Not in cross-WS events | WS2 Step 7.1 | Not listed | Not listed | OK — appears to be intra-WS2 |
| `ws2.content-engagement-updated` | Phase A | Phase A | WS2 Step 1.1 | A2 produced, A3 consumed | Contract 8: Phase A | YES |
| `ws2.capture-brief-generated` | Phase A | Phase A | WS2 Step 1.2, WS3 Panel 4 | A2 | Not in 10 contracts | MINOR gap |
| `ws2.brand-health-calculated` | Phase A | Phase A (Sec 4.1) | Referenced in exp arch | A2 | Not in 10 contracts | MINOR gap |
| `ws1.onboarding-status-changed` | Phase A | Phase A | WS3 Step 9.3 | A3 produced, A2 consumed | Contract 10: Phase A | YES |
| `pms.appointment-completed` | Phase B | Phase B | Not in Phase A journeys | B0 infra, B1 live | Contract 5: Phase B | YES |
| `ws1.funnel-event` | Phase B | Phase B | Not in Phase A journeys | B1 produced, B2 consumed | Contract 7: Phase B | YES |

**Finding**: The 10 formal cross-workstream contracts (in tech review) cover the major flows, but 4 events referenced in journeys and the experience architecture are not in the formal contract list: `ws3.huddle-completed`, `ws2.capture-media-received`, `ws2.capture-brief-generated`, `ws2.brand-health-calculated`. These appear to be events that were added during the PHI-free redesign to support the KPI bar and cross-workstream content flows. They need formal event schemas in the Schema Registry.

**Severity**: MINOR — These events are well-documented in the journeys and experience architecture. They need schema definitions but are not contradictory.

---

### 1.3 CE-1 SMS Availability — The Known Contradiction

**Documents in conflict:**

| Document | Position on CE-1 SMS in Phase A |
|----------|-------------------------------|
| Scope Analysis | GREEN — CE-1 classified as fully operational, SMS/WhatsApp capture bot |
| Experience Architecture (Section 2C) | Modified — CE-1 listed with SMS/WhatsApp delivery |
| WS2 Journey (Flow 1, Step 1.1) | "NOT via SMS capture bot in Phase A. The Communications Service is not active." PWA push only. |
| Build Plan (A2.9) | "Daily SMS capture briefs" using simple non-HIPAA SMS gateway |
| Tech Review (Section 1.2) | Acknowledges non-PHI SMS is viable; recommends building limited Communications Service in A1 |

**Assessment**: Four documents say SMS is available in Phase A. One document (WS2 journey) says it is not. The WS2 journey conflates the HIPAA Communications Service (which IS deferred) with basic SMS capability (which is NOT deferred). The build plan and tech review are explicit: a non-HIPAA SMS gateway handles content capture briefs because they contain zero PHI.

**QC1 flagged this as M1 (Major).** The contradiction remains unresolved in the documents. However, the build plan and tech review positions are authoritative for development. The WS2 journey needs a targeted update to Steps 1.1 and 1.2 and the summary table.

**Resolution status**: DOCUMENTED but NOT FIXED. The WS2 journey still contains the incorrect statement. This must be corrected before development starts.

---

### 1.4 Team Allocation Consistency

| Document | Team Structure |
|----------|---------------|
| Build Plan | Platform Eng (5), WS3 (3), WS2 (4), WS1-A (4), WS1-B (3), QA (2) = 21 engineers |
| Tech Review | References same structure implicitly; discusses Platform Eng and WS teams |

**Finding**: Consistent. Build plan defines team allocation per sub-phase. Tech review does not contradict.

---

### 1.5 Timeline Consistency

| Milestone | Build Plan | Tech Review ADRs |
|-----------|-----------|-----------------|
| A0 Pre-Build | Weeks 1-2 | N/A |
| A1 PHI-Free Infrastructure | Weeks 3-6 | ADRs dated 2026-03-20, reference A1 services |
| A2 Culture + Content | Weeks 5-10 | N/A |
| A3 WS1 PHI-Free | Weeks 8-14 | N/A |
| B0 PHI Infrastructure | Weeks 12-15 | ADRs reference B0 activation |
| B1 Patient Features | Weeks 15-20 | N/A |
| B2 Advanced + Rollout | Weeks 20-24 | N/A |

**Finding**: Consistent. The tech review references build plan phases and timelines correctly. ADRs align with the build plan's phasing.

**One note**: The tech review recommends building AI Service core in A1 (Weeks 5-6) and Communications Service (limited) in A1. The build plan does NOT include these — it defers AI Service entirely and uses a simple SMS gateway. This is a recommendation gap, not a contradiction. The tech review is suggesting improvements to the build plan that have not been incorporated.

**Severity**: MINOR — The tech review's recommendations are sound but represent unresolved optimization suggestions, not conflicts.

---

## 2. Acceptance Criteria Mapping

I traced acceptance criteria from the original consolidated epic through the PHI-free redesign.

### 2.1 Shared Infrastructure (Epics S-1 through S-10)

| Original Epic | Stories | Phase A Status | PHI-Free Coverage |
|---------------|---------|---------------|-------------------|
| S-1 Practice Data Hub | S-1.1 through S-1.4 | Build Plan A1.1 | Fully preserved. 260 practices seeded. |
| S-2 PMS Integration | S-2.1 through S-2.5 | Build Plan B0.4 | Explicitly deferred to Phase B. All stories preserved for B0. |
| S-3 Communications Service | S-3.1 through S-3.7 | Build Plan B0.5 (full) or A1 (limited per tech review) | S-3.7 Voice AI deferred to v2. Patient comms deferred to B0. Content capture SMS available via simplified gateway. |
| S-4 AI Service | S-4.1 through S-4.5 | Build Plan implies B0; Tech Review says A1 (core) | Gap: Build plan does not include AI Service in A1, but Phase A features (caption generation, anomaly analysis) need it. Tech review identifies this. |
| S-5 Event Bus | S-5.1 through S-5.5 | Build Plan A1.3 | Fully preserved. All 10 schemas registered. |
| S-6 Auth & Identity | S-6.1 through S-6.3 | Build Plan A1.2 | Fully preserved. |
| S-7 Compliance & Audit | S-7.1 through S-7.3 | Build Plan B0.2 (full) or A1 monitoring (per tech review) | Tech review suggests A1 monitoring mode. Build plan defers to B0. |
| S-8 Asset Registry | S-8.1 through S-8.3 | Build Plan A1.4 | Fully preserved. |
| S-9 Workflow Engine | S-9.1 through S-9.2 | Build Plan A1.5 | Fully preserved. |
| S-10 Design Tool Abstraction | S-10.1 | Build Plan A1.6 | Fully preserved. |

### 2.2 WS1: Practice Platform Stories

| Original Epic | Key Stories | Phase A Status | Notes |
|---------------|------------|---------------|-------|
| WS1-A1 Practice Nerve Center | WS1-A1.1 (daily agenda), A1.2 (action cards) | Build Plan A3.4 (Practice Daily Agenda Phase A) | Modified: patient action cards replaced with practice operation cards. Acceptance criteria adapted for PHI-free context. |
| WS1-A1 | WS1-A1.3 (end-of-day scorecard) | Modified to "Practice Activity Summary" | Explicitly modified in experience architecture. |
| WS1-A2 Concierge Agent | WS1-A2.1 through A2.6 | Phase B (B1.2) | Fully deferred. All stories preserved for Phase B. Voice AI stories deferred to v2. |
| WS1-A3 Data Harvesting | WS1-A3.1 through A3.4 | Build Plan A3.1, A3.2 | Fully preserved. Zero PHI dependency. |
| WS1-A4 Storefront Generation | WS1-A4.1 through A4.5 | Build Plan A3.3 | Fully preserved. Storefronts use practice data + brand assets. |
| WS1-A5 Automation Engine | WS1-A5.1 through A5.13 | Phase B (B1.4, B2.5) | Fully deferred. All patient communication automation requires PHI. |
| WS1-A6 Graduated Autonomy | WS1-A6.1 through A6.3 | Build Plan A3.6 (PHI-free subset) | Modified: weekly summary uses culture/content metrics only. Patient metrics deferred to B1.5. |
| WS1-A7 Change Management | WS1-A7.1, A7.2 | Implicit in Practice Data Hub | Covered by A1.1 Practice Data Hub CRUD. Not explicitly called out in build plan. |
| WS1-B1 Executive Dashboard | WS1-B1.1 through B1.8 | Build Plan A3.7 | Modified: 5 Phase A KPIs replace 8 original KPIs. Revenue/funnel KPIs deferred. AI alerts refocused. Acceptance criteria adapted. |
| WS1-B2 Funnel Data Lake | WS1-B2.1 through B2.5 | Phase B (B1.6) | Fully deferred. ClickHouse PHI tables exist empty in Phase A. |
| WS1-B3 AI Analysis Engine | WS1-B3.1 through B3.5 | Phase B (B1.9) for patient data. Phase A has culture/brand anomaly detection (A3.7 AI Alerts). | Partially covered in Phase A for non-PHI signals. Revenue impact estimation deferred. |
| WS1-B4 Data Integration | WS1-B4.1 through B4.5 | Mixed | Phone/lead source connectors require patient data (Phase B). iSolved connector is Phase A (staff data). |
| WS1-B5 Network Learning | WS1-B5.1 through B5.4 | Build Plan A3.14 (modified) | Communication outcome tracking deferred. Monthly playbook optimization and learning reports work with culture/brand data in Phase A. |

### 2.3 WS2: Content Engine Stories

| Original Epic | Key Stories | Phase A Status | Notes |
|---------------|------------|---------------|-------|
| WS2-1 Prompt Engine | WS2-1.1 (daily SMS brief) | Build Plan A2.9 | Modified: briefs are brand-driven, not procedure-driven. SMS delivery via non-HIPAA gateway. |
| WS2-1 | WS2-1.2 (procedure code triggers) | Phase B (B1.12) | Deferred. Procedure triggers require PMS data. |
| WS2-2 Capture Layer | WS2-2.1 (SMS/MMS capture) | Build Plan A2.9 | Preserved for brand content. Patient content capture deferred. |
| WS2-2 | WS2-2.4 (guided camera overlays) | WS2 Journey Flow 2 | Modified: clinical overlays replaced with brand content overlays. Before/after mode hidden. |
| WS2-3 Studio Agent | WS2-3.1 through 3.4 | Build Plan A2.10 | Fully preserved. Studio requests are PHI-free. |
| WS2-4 Production Pipeline | WS2-4.1 through 4.8 | Build Plan A2.11 | Fully preserved. Pipeline is content-type-agnostic. |
| WS2-5 Review & Approval | WS2-5.1 through 5.5 | Build Plan A2.12 | Fully preserved. Editorial workflow unchanged. |
| WS2-6 Distribution Engine | WS2-6.1 through 6.9 | Build Plan A2.13 | Mostly preserved. Patient email deployment (6.7) deferred. |
| WS2-7 Scorecard & Analytics | WS2-7.1 through 7.5 | Build Plan A2.15 (champion recap) | WS2-7.4 Content-to-consultation attribution explicitly deferred (CE-7 empty state). |
| WS2-8 Human Escalation | WS2-8.1 | Implied in A2.10 | Preserved within Studio Agent flow. |

### 2.4 WS3: Culture OS Stories

| Original Epic | Key Stories | Phase A Status | Notes |
|---------------|------------|---------------|-------|
| WS3-1 H-Score Engine | WS3-1.1 through 1.5 | Build Plan A2.1 | Fully preserved. H-Score is PHI-free. |
| WS3-2 Morning Huddle OS | WS3-2.1 through 2.6 | Build Plan A2.2 | Preserved. Impact story data source changes (culture/brand vs. funnel). |
| WS3-3 CCA Academy | WS3-3.1 through 3.3 | Build Plan A2.3 | Fully preserved. Training is PHI-free. |
| WS3-4 Trust Transfer OS | WS3-4.1 through 4.3 | Build Plan A2.4 | Preserved at aggregate level. Individual call drill-down deferred. |
| WS3-5 Champion Network | WS3-5.1 through 5.3 | Build Plan A2.5 | Fully preserved. |

### 2.5 Orphaned Acceptance Criteria

**Stories with NO coverage in Phase A OR explicit Phase B deferral:**

| Story ID | Description | Status |
|----------|-------------|--------|
| WS1-A7.1, A7.2 | Change Management (staff change cascading, hours propagation) | NOT explicitly addressed in build plan. Implicitly covered by Practice Data Hub CRUD but no specific build plan deliverable or journey flow. |
| WS2-2.5 | Patient self-record testimonial kiosk | Not mentioned in PHI-free documents. Originally Tier 3. Presumably Phase B but not explicitly stated. |
| WS2-2.6 | Passive operatory cameras | Not mentioned. Originally Tier 3. Not addressed. |
| WS2-6.5 | LinkedIn leadership content scheduling | Not mentioned in PHI-free journeys. Originally Tier 2. Should be Phase A (no PHI dependency) but not called out. |
| WS2-6.6 | Print fulfillment (Gelato API) | Not mentioned in PHI-free journeys. Originally Tier 2. Should be Phase A but not called out. |
| WS2-6.8 | TV slide asset delivery | Not mentioned in PHI-free journeys. Originally Tier 2. Should be Phase A but not called out. |
| WS1-B4.1 through B4.5 | Data Integration connectors (phone system, lead source, AI phone agent, HR, ad platform) | Only iSolved (HR) is implicitly Phase A. Others not explicitly classified. |

**Severity**: MINOR for most (Tier 2/3 stories). The WS1-A7 change management gap and the WS2 distribution channel gaps (LinkedIn, print, TV slide) are MINOR because they are straightforward Phase A features that were likely omitted from journeys because they are low-complexity, not because they were intentionally excluded.

---

## 3. QC Gate 1 Issue Resolution

### 3.1 CE-1 Capture Bot SMS Contradiction (QC1 Issue M1)

**QC1 Finding**: WS2 journey says "NOT via SMS" in Phase A. Build plan and experience architecture say SMS is available via non-HIPAA gateway.

**Current Status**: UNRESOLVED. The WS2 journey document has not been updated. The contradiction persists.

**Build Block Assessment**: Does NOT block the build. The build plan (A2.9) is the authoritative development specification. Developers building CE-1 will follow the build plan, which says "Daily SMS capture briefs via non-HIPAA SMS gateway." The WS2 journey is a UX reference document, not a build specification.

**Required Action**: Update WS2 journey Flow 1 (Steps 1.1 and 1.2) and the summary table before development starts. This is a documentation fix, not a design decision — the decision is already made in the build plan.

### 3.2 Front Desk Persona Adoption Risk (QC1 Issue M2)

**QC1 Finding**: Front Desk staff have only 2 bottom nav items (Today, Huddle). Their primary workflow (patient phone calls, scheduling) is absent in Phase A. Risk of low adoption.

**Current Status**: DOCUMENTED in QC1 but NOT incorporated into the experience architecture or build plan as a formal "Known Adoption Risk."

**Build Block Assessment**: Does NOT block the build. This is an inherent consequence of the PHI boundary. Phase A genuinely cannot serve Front Desk patient-facing workflows.

**Required Action**: Add a "Known Adoption Risks" section to the experience architecture (as QC1 recommended). Include adoption metrics that measure Front Desk by CCA Academy engagement and huddle participation, not platform login frequency.

---

## 4. Defect Classification

### CRITICAL (blocks build)

None identified.

### MAJOR (must resolve before development starts)

| # | Issue | Location | Description | Impact |
|---|-------|----------|-------------|--------|
| D-1 | Scope analysis / experience architecture classification discrepancies | Scope Analysis vs. Experience Architecture | 7 interfaces have different GREEN/YELLOW/RED classifications between the two documents. Experience architecture is correct; scope analysis needs reconciliation. | Developers referencing scope analysis will receive incorrect requirements for PO-6, PO-8, CO-5, CO-7, CE-3, CE-9, PF-1. |
| D-2 | CE-1 SMS contradiction unresolved | WS2 Journey Flow 1 | WS2 journey states "NOT via SMS" in Phase A; build plan says SMS is available. Four of five documents agree SMS works in Phase A. Journey needs update. | Development team for WS2 capture bot will receive contradictory UX spec vs. build spec. |
| D-3 | AI Service not in A1 build plan | Build Plan A1 deliverables | Build plan defers AI Service entirely. Tech review identifies that Phase A AI use cases (caption generation, anomaly analysis, learning reports) require AI Service core in A1. Build plan has no A1 deliverable for AI Service. | WS2 content pipeline (A2.11 AI photo processing, caption generation) and WS1 executive dashboard (AI Alerts, Network Learning Report A3.14) have no AI service to call. |
| D-4 | Front Desk adoption risk undocumented | Experience Architecture | QC1 flagged Front Desk as highest-risk persona (~40% functionality). No formal "Known Adoption Risks" section exists in any document. | Stakeholders may misinterpret low Front Desk engagement as a platform failure rather than an expected Phase A limitation. |

### MINOR (address during development)

| # | Issue | Location | Description |
|---|-------|----------|-------------|
| d-1 | Missing journey flows for NI-8, NI-9, NI-10 | WS1 Journey | Three Phase A interfaces lack narrative journey coverage. Interfaces are well-specified in experience architecture. (Carried from QC1 m1, m2) |
| d-2 | 4 Kafka events referenced in journeys not in formal contract list | WS3 Journey, WS2 Journey, Exp Arch | `ws3.huddle-completed`, `ws2.capture-media-received`, `ws2.capture-brief-generated`, `ws2.brand-health-calculated` need Schema Registry definitions. |
| d-3 | Orphaned Tier 2/3 distribution stories | Consolidated Epic vs. PHI-Free Docs | WS2-6.5 (LinkedIn), WS2-6.6 (print), WS2-6.8 (TV slides) are Phase A-eligible but not addressed. |
| d-4 | WS1-A7 Change Management stories not in build plan | Build Plan | Staff change cascading and hours propagation not explicitly planned. Implicitly covered by Practice Data Hub CRUD. |
| d-5 | Scope analysis GREEN count inconsistency | Scope Analysis | Executive summary says 19 GREEN; detailed table says 17 GREEN. Internal inconsistency within the document. |
| d-6 | Brand template sync failure scenario | WS3 Journey Flow 4 | (Carried from QC1 m3) No mention of degradation behavior when weekly sync from Asset Registry fails. |
| d-7 | Tech review recommendations not incorporated into build plan | Build Plan vs. Tech Review | Three tech review recommendations (AI Service A1, Comms Service limited A1, Compliance monitoring A1) are not reflected in the build plan. |

---

## 5. QA Verdict: PASS

**Rationale**: No CRITICAL defects. The 4 MAJOR issues are all resolvable through documentation updates and build plan adjustments — none require design rework. The core architecture, journey flows, and Phase A/B boundaries are sound. Cross-document consistency is strong for the authoritative documents (experience architecture, build plan, tech review) with the scope analysis being the outlier that needs reconciliation.

**Conditions for PASS**:
1. D-1: Reconcile scope analysis classifications to match experience architecture (or add an explicit note that experience architecture supersedes scope analysis for interface-level classification)
2. D-2: Update WS2 journey Flow 1 to reflect SMS availability in Phase A
3. D-3: Either add AI Service core to A1 build plan deliverables or document how WS2 caption generation and WS1 AI Alerts will function without it
4. D-4: Add "Known Adoption Risks" section to experience architecture

---

# Part 2: Product Owner Review

## 1. Backlog Completeness

### 1.1 User Story Traceability

The original consolidated epic contains 110 workstream-specific stories + 38 shared infrastructure stories = 160 total build items.

**Phase A coverage**:
- Shared Infrastructure: 27 of 38 Tier 1 stories fully in A1. 4 services (PMS, Comms, Compliance, AI PHI) deferred to B0.
- WS3 Culture OS: 13 of 13 Tier 1 stories in A2. 7 Tier 2 stories in A2. All 21 stories addressable in Phase A.
- WS2 Content Engine: 18 of 18 Tier 1 stories in A2 (most preserved; capture and prompt stories modified for brand content). 10+ of 16 Tier 2 stories addressable. 2 stories (procedure code triggers, patient testimonials) explicitly Phase B.
- WS1 Practice Platform: 18 of 28 Tier 1 WS1-A stories split — data harvesting, storefront, daily agenda (modified) in A3; patient-facing stories (concierge, automation) to B1. Network Intelligence Tier 1 stories modified for Phase A in A3. Funnel Data Lake and AI Analysis Engine deferred to B1.

**Traceability verdict**: GOOD. The mapping from original stories through PHI-free redesign is traceable. Every original story is either: (a) preserved in Phase A, (b) modified for PHI-free context, or (c) explicitly deferred to Phase B with the deferral rationale documented. The 7 orphaned stories identified in Part 1 (Section 2.5) are edge cases, not systematic gaps.

### 1.2 Orphaned Requirements

**From original sprints not addressed in Phase A or Phase B:**
- WS2-2.5 (patient self-record kiosk) — Tier 3, not mentioned. Acceptable omission.
- WS2-2.6 (passive operatory cameras) — Tier 3, not mentioned. Acceptable omission.
- WS2-6.5, 6.6, 6.8 (LinkedIn, print fulfillment, TV slides) — Tier 2, should be Phase A but not called out in journeys. These are distribution channels that the Content Engine architecture supports inherently. They should be added to the build plan.
- WS1-A7 Change Management — Tier 2. Needs explicit placement in A3 or confirmation that Practice Data Hub CRUD covers it.

**Assessment**: No critical orphans. The omissions are low-priority features (Tier 2/3) that were likely not included in the PHI-free journeys because they are straightforward implementations with zero PHI dependency. They should be added to the Phase A build backlog during sprint grooming.

### 1.3 Phase B Backlog Definition

The Phase B backlog is clearly defined across multiple documents:

- **Build Plan**: B0 (PHI infrastructure), B1 (patient features with specific deliverables), B2 (advanced features + rollout)
- **Experience Architecture Section 6**: Phase B transitions for every surface area
- **Each Journey Document**: "Phase B Addition Callouts" tables at the end
- **Tech Review**: ADRs for Phase B schema evolution, feature flag architecture

**Assessment**: COMPLETE. Phase B is not just a vague "add PHI later" — it has defined deliverables, timelines, team allocations, QC gates, and technical architecture. The phase boundary is well-documented.

---

## 2. Value Delivery Assessment

### 2.1 Phase A Standalone ROI

**Does Phase A deliver enough value for SGA to see ROI?**

YES, with nuance. Phase A delivers:

| Value Category | Concrete Deliverable | Business Impact |
|---------------|---------------------|-----------------|
| Culture Transformation | Daily huddles, H-Score tracking, CCA certification, champion program for 260+ practices | Measurable hospitality improvement across the network. This is SGA's stated #1 priority per the Sprint 2 problem statement. |
| Brand Content Pipeline | End-to-end capture-process-review-approve-distribute pipeline | Eliminates manual content creation bottleneck. Standardizes brand across 260+ practices. |
| Executive Visibility | Dashboards with culture, brand, enrollment KPIs for CEO and VP Ops | First-ever network-wide operational visibility (even without revenue data). |
| Practice Onboarding | Automated enrollment pipeline, storefront generation, data harvesting | Reduces HQ effort per practice onboarding from weeks to days. |
| Storefront Quality | Template-based websites, GMB optimization, directory sync | Immediate digital presence improvement for all practices. |

**The key question: Is this worth the investment before patient features arrive?**

Yes. SGA's original problem (Sprint 2) was that their practices lack consistent hospitality culture. Phase A is the direct solution to that problem. Patient-facing automation (Phase B) amplifies the culture but does not create it. Phase A creates the foundation that makes Phase B valuable.

### 2.2 Value Distribution by Actor Type

| Actor Type | Phase A Value | Rating |
|-----------|--------------|--------|
| HQ Culture Team | 100% — full Culture Admin Console, full CCA program | HIGH |
| Creative Director | 100% — unchanged brand review workflow | HIGH |
| HQ Content Team | ~90% — full content pipeline, missing only attribution | HIGH |
| VP Marketing | ~90% — full dashboard minus one panel | HIGH |
| Champions (Maria Santos) | ~85% — full scorecard, missions, leaderboards | HIGH |
| HQ Operations (Angela) | ~70% — full enrollment, reduced standup board | MEDIUM-HIGH |
| Clinical Staff | ~70% — capture, huddle, CCA certification | MEDIUM-HIGH |
| CEO / Executives | ~60% — culture/brand dashboards but no revenue data | MEDIUM |
| VP Ops / Regional Directors | ~60% — culture-based intervention but no revenue ranking | MEDIUM |
| Office Managers | ~55% — practice brand/culture management but no patient operations | MEDIUM |
| Front Desk Staff | ~40% — huddle + CCA only; primary workflow absent | LOW |

**Highest value**: HQ-level users and content/culture teams get near-full functionality.
**Lowest value**: Front Desk staff. This is the persona most likely to question "why do I need this platform?"

### 2.3 Shelfware Risk Assessment

**Features at risk of non-use because PHI features are absent:**

| Feature | Risk Level | Rationale |
|---------|-----------|-----------|
| Practice Daily Agenda (PO-1) | MEDIUM | Office managers accustomed to patient-driven morning agendas may find the practice-operations-focused version insufficient for their daily workflow. The 3-4 practice action cards (storefront, content, culture, playbook) may feel low-urgency compared to "7 patient confirmations due." Mitigation: the H-Score widget and content capture brief give daily freshness. |
| Intervention Queue (NI-3) | LOW | Culture Risk scoring is a reasonable proxy for practice health even without revenue impact. VP Ops can still identify and act on struggling practices. |
| Executive Dashboard (NI-1) | LOW | 5 KPI cards tell a coherent operational story. Executives can make decisions. Revenue will be missed but the culture/brand narrative holds. |
| Front Desk "Today" screen | HIGH | With only 2 nav items, Front Desk staff have minimal reason to open the PWA daily. CCA Academy engagement may sustain usage during the certification journey (2-4 weeks per tier) but post-certification, the platform offers limited daily value. |

**Overall shelfware risk**: LOW-MEDIUM. The platform's daily-use features (huddle, capture, champion missions) create habitual engagement loops that do not depend on PHI. The risk concentrates in the Front Desk persona and the Office Manager daily agenda.

---

## 3. Scope Boundaries

### 3.1 Phase A / Phase B Boundary Crispness

**Is the boundary crisp?** YES, with one fuzzy area.

The boundary is defined by a single principle: **no individual patient data crosses into Phase A**. This is a clean, auditable line. The experience architecture's Principle 11 ("Staff data is not PHI") and the PHI classification table (Section 1.3) provide clear guidance.

**The one fuzzy area**: Aggregate practice-level metrics sourced from PMS systems. The scope analysis includes a "Borderline" category: "Aggregate production / new patient counts (practice-level, no patient identity) — Included if sourced from practice-reported totals, not PMS record-level data." The experience architecture resolves this by excluding ALL PMS-sourced data, even aggregates, from Phase A — using only vendor aggregate scores (Cinnamon, Swell, Dental Intel) and platform-native data (content captures, enrollment status). This is the correct conservative approach.

### 3.2 Phase B Features Pullable into Phase A

| Candidate | Feasibility | Recommendation |
|-----------|------------|----------------|
| PF-1 Practice Website basic version | Already pulled into Phase A by experience architecture | Done. Website works with practice data + brand assets. |
| Content-to-booking attribution (CE-7 panel) | Requires patient funnel data from PMS. Cannot pull. | Keep Phase B. |
| Before/after photo capture (CE-2) | Requires patient consent workflow and patient-photo mapping. Cannot pull without PHI. | Keep Phase B. |
| Voice AI (PF-2) | Deep PHI dependency. Cannot pull. | Keep Phase B. |
| Individual call transcript drill-down (CO-7) | Requires call recording access containing verbal PHI. Cannot pull. | Keep Phase B. |
| Compliance Dashboard live monitoring (NI-9) | No PHI to monitor in Phase A. The "readiness" version is the right Phase A approach. | Keep as-is. |

**Assessment**: The experience architecture has already pulled everything pullable into Phase A. The remaining Phase B items have genuine, irreducible PHI dependencies. There are no easy wins left.

### 3.3 Phase A Features That Secretly Need PHI

I reviewed every Phase A interface for hidden PHI dependencies.

| Feature | Hidden PHI Risk | Assessment |
|---------|----------------|------------|
| H-Score (CO-5) | Cinnamon phone scores derived from patient calls | LOW — Cinnamon provides aggregate scores at the staff/practice level. The SGA platform never accesses individual call recordings or patient data. Cinnamon's BAA covers their processing of call content. Tech review confirms this at Section 3.11 (Non-PMS Data Integration). |
| Trust Transfer Tracker (CO-7) aggregate scores | Derived from call analysis that involves patient interactions | LOW — Same as H-Score. Aggregate scores per staff member are the output; patient-level data stays in Cinnamon. |
| AI Alerts (NI-1) | Could AI analysis inadvertently surface patient patterns? | LOW — Phase A AI uses only aggregate metrics (H-Score, brand health, content captures, enrollment). No patient data enters the AI Service. Tech review ADR-PHI-004 adds defensive PHI detection as a safety net. |
| Content Capture photos | Champions could photograph patients during brand content capture | MEDIUM — Addressed in tech review (Section 3.5): AI-based face detection at upload, flagged uploads require HQ review. This is a process risk, not an architectural PHI dependency. |
| Morning Huddle Impact Story | Could auto-generated stories reference individual patients? | LOW — Phase A impact stories use culture/brand metrics only. The AI generation prompt explicitly excludes patient data. |
| Dental Intel integration (DI sub-score in H-Score) | Dental Intel metrics include schedule utilization which could be derived from appointment data | LOW — The WS3 journey notes DI provides "operational compliance scores" at practice level. SGA does not receive patient-level DI data. |

**Verdict**: No Phase A features secretly require PHI. The two medium-risk items (content capture photos, Dental Intel integration) are process risks managed by procedural controls, not architectural PHI dependencies that would cause test failures.

---

## 4. Product Owner Verdict: COMPLETE

**With the following gaps to close:**

| # | Gap | Priority | Resolution |
|---|-----|----------|------------|
| G-1 | Add WS2-6.5 (LinkedIn), WS2-6.6 (print), WS2-6.8 (TV slides) to Phase A backlog | Medium | These are Phase A-eligible distribution channels already supported by the Content Engine architecture. Add as Tier 2 stories in the A2 build plan. |
| G-2 | Clarify WS1-A7 Change Management placement | Low | Confirm that Practice Data Hub CRUD (A1.1) covers staff change cascading and hours propagation, or add explicit A3 deliverable. |
| G-3 | Resolve AI Service placement in build plan | High | The build plan must either add AI Service core to A1 or document an alternative for Phase A AI use cases. This is the same as QA defect D-3. |
| G-4 | Add "Known Adoption Risks" section | Medium | Same as QA defect D-4. Required for stakeholder expectation management. |

---

# Part 3: Combined Recommendation

## Overall Readiness to Proceed to HTML Visualization

**READY — with 4 documentation fixes required first.**

The PHI-free user journey redesign is architecturally sound, internally consistent (with documented exceptions), and delivers a coherent Phase A product. The 9 documents form a comprehensive specification package that covers scope, experience design, user journeys, build plan, and technical architecture. QC Gate 1 passed with 2 Major issues; this QC Gate 2 identifies 4 Major issues (2 carried from QC1, 2 new). All 4 are documentation/planning fixes, not design rework.

## Top 3 Items That MUST Be Addressed

1. **Resolve the AI Service build plan gap (D-3 / G-3).** Phase A features that require AI (WS2 caption generation, WS1 AI Alerts, WS1 Network Learning Report) have no AI Service in the A1 build plan. Either add AI Service core to A1 (as the tech review recommends) or document how these features will function without centralized AI. This is the only issue that could cause a build-time surprise.

2. **Update the WS2 journey to reflect SMS availability (D-2).** The CE-1 Capture Bot SMS contradiction has been flagged since QC1 and remains unfixed. This is a 30-minute documentation update that eliminates developer confusion.

3. **Reconcile scope analysis with experience architecture (D-1).** Add a header note to the scope analysis stating that the experience architecture is the authoritative source for interface-level Phase A/B classification. Alternatively, update the 7 conflicting interface classifications to match.

## Top 3 Strengths of the Approach

1. **The PHI boundary is clean, conservative, and auditable.** The single principle (no individual patient data in Phase A) is consistently applied across all 30 Phase A interfaces. The tech review's dual-schema PostgreSQL design, hierarchical feature flags, and PHI sentinel monitoring provide defense-in-depth. There is no "almost PHI" gray area — every borderline case was resolved conservatively.

2. **Phase A is a real product, not a demo.** The Content Engine pipeline, Culture OS, and executive dashboards deliver genuine daily-use value. The WS2 journey (content capture through multi-channel distribution) and WS3 journey (morning huddle through CCA certification) are complete end-to-end workflows that practice staff will use every day. This is not a stripped-down prototype waiting for Phase B to become useful.

3. **Phase B transition is architecturally invisible.** The "architecture for addition, not subtraction" principle (Principle 10) is meticulously applied. Tab injection, feature-flagged component rendering, reserved dashboard slots, forward-compatible Kafka schemas, and dynamic navigation all ensure Phase B is an expansion, not a migration. No Phase A user will experience disruption when Phase B activates.

## Recommendation for Client Review Presentation

Present Phase A as the **operational foundation** for SGA's hospitality transformation — the culture, brand, and visibility layer that must exist before patient-facing automation has context to operate in. Lead with the value story:

1. **Open with Culture OS** (WS3) — this is the strongest workstream (8/8 interfaces, essentially unchanged) and directly addresses SGA's stated #1 priority (hospitality culture at scale). Show the morning huddle, CCA Academy, and champion program.

2. **Follow with Content Engine** (WS2) — the content pipeline is the most impressive technical demonstration (AI processing, multi-channel distribution, 58-second brand approval). Show Maria's capture-to-Instagram journey.

3. **Show Executive Dashboards** (WS1-B) — demonstrate the 5 KPI cards, portfolio heatmap, and practice deep dive. Be transparent about what Phase B adds (revenue data) but emphasize that culture/brand visibility is the prerequisite for revenue optimization.

4. **Address Phase B directly** — do not hide the PHI boundary. Present it as a strategic advantage: "We build the foundation first, prove it works, then add patient data safely." Show the feature flag architecture and the progressive reveal strategy.

5. **Acknowledge the Front Desk gap** — proactively note that Front Desk staff have a lean Phase A experience focused on training and culture development. Frame this honestly: their patient-facing tools arrive in Phase B.

---

## Summary Table

| Category | Verdict |
|----------|---------|
| **QA Verdict** | PASS (4 Major documentation fixes required) |
| **Product Owner Verdict** | COMPLETE (3 backlog gaps to close) |
| **Critical Defects** | 0 |
| **Major Defects** | 4 (D-1 scope reconciliation, D-2 SMS contradiction, D-3 AI Service gap, D-4 adoption risk) |
| **Minor Defects** | 7 |
| **Orphaned Requirements** | 7 (all Tier 2/3, none blocking) |
| **Phase A Standalone Value** | YES — delivers culture, content, and visibility |
| **Shelfware Risk** | LOW-MEDIUM (concentrated in Front Desk persona) |
| **Phase B Readiness** | STRONG — clean boundary, additive architecture |
| **Ready for HTML Visualization** | YES — after the 4 Major fixes |

---

*Reviewed by: QA Engineer Agent + Product Owner Agent*
*Date: 2026-03-20*
*Gate: QC2 — PHI-Free User Journey Redesign*
