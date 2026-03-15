# QC-3: Technical Architecture Quality Gate Review

**Gate**: QC-3 — QA Engineer Review of Technical Architecture
**Date**: 2026-03-14
**Reviewer**: QA Engineer Agent
**Document Under Review**: `resources/problem-sprints/technical-architecture.md`
**Scope**: Validate architecture coverage, event contract completeness, schema quality, HIPAA compliance, build sequence feasibility, and ADR quality against acceptance criteria from all 6 original sprint dev packages.

---

## Verdict: REVISE

**Zero critical findings. Three major findings. Seven minor findings. Four enhancements.**

The architecture must address the 3 major findings before proceeding to build. All are resolvable without fundamental redesign.

---

## Review Area 1: Coverage Check — Shared Services vs. Original Sprint Requirements

### Methodology
Mapped each original sprint's infrastructure requirements to the 10 shared services defined in Sections 1.1-1.10 (note: the document titles "8 shared services" but actually defines 10, including the Workflow Engine and Design Tool Abstraction Layer).

### Coverage Matrix

| Original Sprint Infrastructure Need | Covered By | Coverage Status |
|---|---|---|
| Practice Data Hub (Sprint 3) | 1.1 Practice Data Hub | FULL |
| Data Harvester / Playwright scraping (Sprint 3) | Not a shared service (WS1-internal) | ACCEPTABLE — workstream-level, not shared |
| Template Engine for websites (Sprint 3) | Not a shared service (WS1-internal) | ACCEPTABLE |
| Channel Sync Engine: GMB, Meta, directories (Sprint 3) | Not a shared service (WS1-internal) | ACCEPTABLE |
| PMS Integration: NexHealth (Sprint 5), Sikka ONE (Sprint 6), Airbyte (Sprint 4) | 1.2 PMS Integration Service | FULL |
| Twilio SMS/Voice (Sprint 5), Twilio for capture prompts (Sprint 6) | 1.3 Communications Service | FULL |
| Claude API (Sprints 1, 3, 4, 5, 6), OpenAI Whisper (Sprint 2, 6) | 1.4 AI Service | FULL |
| Asset storage: S3, CDN, Cloudinary (Sprints 1, 6) | 1.5 Asset Registry | FULL |
| Event bus for cross-workstream data flow (all sprints) | 1.6 Event Bus (Kafka) | FULL |
| Authentication and RBAC (all sprints) | 1.7 Auth & Identity | FULL |
| HIPAA compliance / audit logging (Sprint 5 Compliance Auditor) | 1.8 Compliance & Audit Layer | FULL |
| Temporal workflow orchestration (Sprint 4, others) | 1.9 Workflow Engine | FULL |
| Canva Connect API abstraction (Sprint 1) | 1.10 Design Tool Abstraction | FULL |
| **Phone system integrations: RingCentral, 8x8, Weave** (Sprint 4) | **NOT COVERED** | **GAP** |
| **Ad platform integrations: Google Ads, Meta Ads, LSAs** (Sprint 4) | **NOT COVERED** | **GAP** |
| **AI phone agent integrations: Sierra, Agent Agnes** (Sprint 4) | **NOT COVERED** | **GAP** |
| **HR system integration: iSolved** (Sprint 3, Sprint 4) | **NOT COVERED** | **GAP** |
| **Print fulfillment: Gelato API** (Sprint 1) | **NOT COVERED** | **MINOR GAP** |
| **Weave API for email campaigns / TV slides** (Sprint 1) | **NOT COVERED** | **MINOR GAP** |
| **Social platform APIs: Instagram, Facebook, YouTube, LinkedIn, TikTok, GBP** (Sprint 6) | **NOT COVERED** | **GAP** |
| **Cinnamon, Swell, Dental Intel external API integrations** (Sprint 2) | Not shared — WS3-internal | ACCEPTABLE |

### Findings

**FINDING-01: Phone System, Ad Platform, and Social Platform Integrations Missing from Architecture**
- **Severity**: MAJOR
- **Detail**: Sprint 4 (Funnel OS) explicitly requires connectors for phone systems (RingCentral, 8x8, Weave), ad platforms (Google Ads, Meta Ads, LSAs), and AI phone agents (Sierra, Agent Agnes). These are listed as part of the "15-platform integration middleware" in the Funnel OS architecture and appear in feature 1.13 of the consolidation feature map. The technical architecture's PMS Integration Service only covers NexHealth and Sikka ONE. The remaining data sources (phone, ads, AI agents, HR) are not addressed by any shared service.
- **Impact**: Without phone system integration, the Funnel OS cannot track call answer rates, missed calls, or call-to-booking conversion — critical funnel stage 2 (Lead Connected) metrics. Without ad platform integration, lead source attribution is impossible. These are foundational to Sprint 4's value proposition.
- **Recommendation**: Either (a) extend the PMS Integration Service into a broader "Data Integration Service" covering all external data sources, or (b) define a separate Integration Middleware shared service, or (c) explicitly document these as WS1-internal integrations with their own build phase tasks. Option (c) is acceptable but must appear in the Phase 3 build sequence.

**FINDING-02: Social Platform API Integrations Not Addressed**
- **Severity**: MINOR
- **Detail**: Sprint 6 (Content Concierge) and Sprint 1 (Studio Automation) require distribution to Instagram, Facebook, YouTube, LinkedIn, TikTok, and Google Business Profile via their respective APIs (Meta Graph API, YouTube Data API, LinkedIn API, etc.). The architecture mentions these channels in the event contracts (approved assets, engagement metrics) but does not specify where the social API integration code lives or how API credentials are managed across 260 practices.
- **Impact**: WS2 team will need to build all social platform connectors without shared infrastructure guidance. Credential management for 260 practices x 6 platforms is a significant operational concern.
- **Recommendation**: Add a brief note to the architecture clarifying that social platform API management is WS2-internal, or consider adding it as a lightweight shared integration registry.

**FINDING-03: iSolved HR Integration Unaddressed**
- **Severity**: MINOR
- **Detail**: Sprint 3's Data Harvester and Sprint 4's Integration Middleware both reference iSolved for HR data (staff changes, provider roster updates). The Practice Data Hub has a `practice_staff` table but no integration path for automatically updating it from iSolved.
- **Impact**: Staff data in the Practice Data Hub may become stale, undermining the "single source of truth" guarantee. Manual staff updates defeat the purpose of centralization.
- **Recommendation**: Add iSolved integration as a data source for the Practice Data Hub, or note it as a Phase 2 enhancement.

---

## Review Area 2: Event Contract Completeness

### Methodology
Reviewed all 10 event contracts against the consolidation feature map's integration points (Section 4) and the original sprint data flows.

### Event Contracts Present (10)

| # | Contract | Adequate? |
|---|----------|-----------|
| 1 | Champion Roster to Dashboard (WS3 to WS1, WS2) | YES |
| 2 | H-Score to Executive Dashboard (WS3 to WS1) | YES |
| 3 | Benchmarking Taxonomy to Shared (WS3 to WS1, WS2) | YES |
| 4 | Champion Stories to Content Pipeline (WS3 to WS2) | YES |
| 5 | Patient Data to Content Personalization (Shared to WS2) | YES |
| 6 | Approved Assets to Storefront (WS2 to WS1) | YES |
| 7 | Funnel Events to Content Performance (WS1 to WS2) | YES |
| 8 | Social Engagement to Dashboard (WS2 to WS1) | YES |
| 9 | Training Completion to Readiness (WS3 to WS1) | YES |
| 10 | Onboarding Status to Activation (WS1 to WS3) | YES |

### Missing Event Contracts

**FINDING-04: Missing Event Contract — WS2 Content Assets to WS3 Recognition Engine**
- **Severity**: MINOR
- **Detail**: Consolidation feature map item 3.06 (Recognition & Rewards Engine) and item 3.08 (Content Production Pipeline integration) indicate WS3 "reads brand assets for recognition posts and certificates" (also stated in Asset Registry consumers, Section 1.5). However, there is no event contract for WS3 to be notified when a recognition-worthy asset or content piece is available. WS3 must either poll the Asset Registry or receive a content event.
- **Impact**: Minor — WS3 can query the Asset Registry API directly. But this creates an undocumented coupling pattern outside the event-driven architecture.
- **Recommendation**: Add a lightweight event contract or document the polling pattern explicitly.

**FINDING-05: Missing Event Contract — Huddle Completion to Practice Nerve Center**
- **Severity**: MINOR
- **Detail**: The topic list in Section 1.6 includes `sgadental.ws3.huddle-completed` but no formal event contract with schema is defined for it. The Practice Nerve Center (WS1) would benefit from knowing whether a practice completed their morning huddle (daily engagement signal).
- **Impact**: Low — the topic exists but the schema is undefined. Could be deferred to implementation.
- **Recommendation**: Add schema definition for `huddle-completed` event, or note it as an internal WS3 event not requiring a cross-workstream contract.

**FINDING-06: No Event for Staff Changes Propagating from Practice Data Hub**
- **Severity**: MINOR
- **Detail**: `sgadental.practice.staff-updated` appears in the topic list (Section 1.6) but has no formal event contract with schema. Staff changes affect WS2 (champion contact info for capture prompts) and WS3 (champion roster). The Practice Data Hub API emits events but the schema is undefined.
- **Impact**: Without a defined schema, consumer teams will implement against undocumented assumptions.
- **Recommendation**: Add a formal schema for `practice.staff-updated` events.

### Assessment
The 10 defined contracts cover the primary cross-workstream data flows well. The missing contracts are lower-priority flows that can be added during implementation. No critical data flows are uncontracted.

---

## Review Area 3: Schema Quality

### Methodology
Reviewed all 10 event payload schemas for field completeness, consumer utility, PHI handling, and data type correctness.

### Findings

**FINDING-07: Event Contract 5 (Appointment Completed) Missing Provider ID**
- **Severity**: MINOR
- **Detail**: The `pms.appointment-completed` schema includes `provider_name` as a string but not `provider_id` (UUID from practice_staff table). WS2 needs to correlate procedures to providers for capture prompt routing — a name string is fragile (name collisions, formatting inconsistencies).
- **Impact**: WS2 must fuzzy-match provider names to staff records instead of using a deterministic ID join.
- **Recommendation**: Add `provider_staff_id: "uuid"` to the appointment schema alongside `provider_name`.

**FINDING-08: No Correlation ID Pattern Across Events**
- **Severity**: ENHANCEMENT
- **Detail**: Events lack a standard `correlation_id` field for tracing related events across workstreams. For example, tracing the chain: appointment completed (EC5) leads to capture prompt sent (WS2 internal) leads to media received (WS2 internal) leads to asset approved (EC6) leads to funnel attribution (EC7). Without correlation, debugging cross-workstream flows requires timestamp/practice_id heuristics.
- **Recommendation**: Add optional `correlation_id: "uuid | null"` to the base event envelope.

### Overall Schema Quality Assessment
Schemas are well-structured with appropriate data types, clear field naming, and appropriate PHI handling (hashed patient IDs, no PII in cross-workstream events). The use of enums for status fields and typed nested objects is good practice. Schemas carry sufficient data for their documented consumers.

---

## Review Area 4: HIPAA Compliance Assessment

### Methodology
Evaluated the Compliance & Audit Layer (Section 1.8) against HIPAA Security Rule (technical safeguards), Privacy Rule (minimum necessary, consent), and Breach Notification Rule requirements.

### Coverage Matrix

| HIPAA Requirement | Addressed? | Location | Assessment |
|---|---|---|---|
| Audit logging (access controls, activity logs) | YES | Section 1.8 audit_log schema | Comprehensive — actor, action, resource, timestamp, PHI flag |
| Unique user identification | YES | Section 1.7 Auth & Identity (Cognito) | JWT-based, per-user |
| Automatic logoff | YES | Section 1.7 (Redis session TTL) | 15-minute RBAC cache; session management implicit |
| Encryption at rest | YES | Section 1.8 (AES-256 RDS, S3 SSE-KMS) | Double encryption for PHI fields — exceeds minimum |
| Encryption in transit | YES | Section 1.8 (mTLS, TLS 1.2+) | Service mesh mTLS is strong |
| PHI access controls (minimum necessary) | YES | Section 1.7 RBAC roles, practice-scoped access | Role model is appropriate |
| Consent management | YES | Section 1.3 consent_registry, Section 1.8 Consent Checker | Covers opt-in/out, TCPA compliance |
| Data retention policies | YES | Section 1.8 (7-year PHI, retention table) | Correct — 7 years exceeds HIPAA 6-year minimum |
| BAA tracking | YES | Section 1.8 BAA Requirements table | All vendors identified with status |
| PHI in third-party systems | PARTIAL | Section 1.4 PHI Filter, Section 1.8 Cloudinary note | See findings below |
| **Breach notification procedures** | **NO** | Not addressed | **See FINDING-09** |
| **Incident response plan** | **NO** | Not addressed | **See FINDING-09** |
| **Risk assessment documentation** | **NO** | Not addressed | **See FINDING-10** |
| **Disaster recovery / backup** | **NO** | Not addressed | **See FINDING-10** |
| **Workforce training requirements** | **NO** | Not addressed | Acceptable — operational, not architectural |
| **Physical safeguards** | N/A | AWS handles physical security | Covered by AWS BAA |

### Findings

**FINDING-09: No Breach Notification Procedure or Incident Response Plan**
- **Severity**: MAJOR
- **Detail**: HIPAA Breach Notification Rule (45 CFR 164.400-414) requires covered entities to notify affected individuals within 60 days of discovering a breach, notify HHS, and (for breaches affecting 500+ individuals) notify media. The architecture defines strong preventive controls (encryption, PHI filter, audit logging) but contains zero mention of what happens WHEN a breach occurs — no incident response workflow, no breach notification system, no breach severity assessment process.
- **Impact**: A healthcare platform processing PHI for 260 practices without a documented breach response procedure is a compliance gap. This is not just a policy document — it requires architectural support: breach detection triggers in audit logs, automated PHI exposure assessment, notification workflows, and forensic log preservation.
- **Recommendation**: Add a Section 1.8.x covering: (1) Breach detection triggers (anomalous PHI access patterns in audit log), (2) Automated breach severity assessment workflow (Temporal), (3) Notification pipeline (affected individuals, HHS, state AGs, media if 500+), (4) Forensic log preservation (immutable snapshots). This is architecturally significant because the audit log schema must support breach investigation queries.

**FINDING-10: No Disaster Recovery or Backup Architecture**
- **Severity**: MAJOR
- **Detail**: HIPAA Security Rule (45 CFR 164.308(a)(7)) requires a contingency plan including data backup, disaster recovery, and emergency mode operation. The architecture specifies "Backups: Encrypted with AWS KMS customer-managed key" (Section 1.8) but provides no backup strategy, RPO/RTO targets, multi-region failover plan, or disaster recovery procedures. For a platform managing patient communications across 260 practices, an outage could mean missed appointment confirmations, lost leads, or inaccessible patient records.
- **Impact**: Without RPO/RTO targets, the build team cannot design appropriate backup strategies. Without a DR plan, a region-level AWS failure could take down all patient communications simultaneously.
- **Recommendation**: Add a section covering: (1) RPO targets per data tier (operational DB: 1 hour, analytics: 24 hours, audit logs: zero loss), (2) RTO targets (critical services: 4 hours, non-critical: 24 hours), (3) Multi-AZ (already implied by RDS Multi-AZ) and cross-region strategy, (4) Database backup schedule and retention, (5) Failover procedures for critical-path services (Communications, PMS Integration, AI Service).

**FINDING-11: Cloudinary BAA Classification Incorrect**
- **Severity**: MINOR
- **Detail**: The BAA matrix marks Cloudinary as "No BAA required" with the note "No PHI in design assets." However, the same section immediately notes: "Cloudinary processes before/after photos which may constitute PHI (patient face, dental work)." The mitigation (de-identification) is sensible but the BAA classification is contradictory. Before/after dental photos are PHI under HIPAA even if de-identified in metadata, because the images themselves can identify patients (facial features, distinctive dental work).
- **Impact**: If Cloudinary processes identifiable patient images without a BAA, this is a HIPAA violation regardless of metadata de-identification.
- **Recommendation**: Either (a) require a BAA with Cloudinary, or (b) route all patient-identifiable media through an in-house processing pipeline (AWS Lambda + Sharp) instead of Cloudinary, or (c) ensure all patient media is truly de-identified (faces blurred, no identifying features) before reaching Cloudinary and document this as a technical control.

---

## Review Area 5: Build Sequence Feasibility

### Methodology
Evaluated Phase 0-3 timeline against service dependencies, team capacity, and parallelization opportunities.

### Phase-by-Phase Assessment

| Phase | Duration | Services/Tasks | Feasible? | Risk Level |
|---|---|---|---|---|
| Phase 0: Pre-Build | Weeks 1-2 | Team formation, BAAs, AWS provisioning, CI/CD, Kafka, Twilio, monitoring | TIGHT | HIGH — BAA procurement is 2-8 weeks, not 2 weeks |
| Phase 1: Shared Infra | Weeks 1-4 | 8 services built by 5-person Platform Engineering team | FEASIBLE with caveats | MEDIUM |
| Phase 2: Contracts + WS3 | Weeks 5-8 | Contract testing + WS3 early start + PMS extension | FEASIBLE | LOW |
| Phase 3: WS1 + WS2 | Weeks 9-12 | Both workstreams begin feature work | FEASIBLE | MEDIUM |

### Findings

**FINDING-12: Phase 0 BAA Procurement Timeline Unrealistic**
- **Severity**: MINOR
- **Detail**: Phase 0 allocates 2 weeks for BAA execution with NexHealth, Sikka ONE, Anthropic, and OpenAI. Risk register item R10 acknowledges BAA procurement may take longer. Enterprise BAA negotiations with AI vendors (Anthropic, OpenAI) are new territory — neither company has a long track record of healthcare BAAs. Realistic timeline: 4-8 weeks.
- **Impact**: If BAAs are not signed by Phase 1 start, development proceeds with synthetic data only. This is acceptable for infrastructure services but blocks PMS Integration Service testing with real appointment data and AI Service testing with real patient-context prompts.
- **Recommendation**: Start BAA procurement immediately (before Phase 0 officially begins). Accept that BAAs may not close until mid-Phase 1. Design Phase 1 exit criteria to distinguish "functional with synthetic data" vs. "functional with production data." Production data testing can overlap with Phase 2.

**FINDING-13: Phase 1 Service Count is Actually 10, Not 8**
- **Severity**: MINOR
- **Detail**: The architecture defines 10 shared services (1.1-1.10) but Phase 1 build sequence only lists 8, omitting the Workflow Engine (Temporal) and Design Tool Abstraction from explicit Week-by-week scheduling. Temporal is listed in Phase 0 ("Set up Temporal Cloud or self-hosted cluster") as provisioning only, but the actual Temporal worker framework, workflow definitions, and SDK setup are not scheduled. The Design Tool Abstraction is scheduled in Phase 3 Week 9 under WS2.
- **Impact**: Temporal is a dependency for PMS batch sync (Week 3-4), Communication cadences (Phase 3), and H-Score calculation (Phase 2 Week 8). If Temporal worker framework is not built in Phase 1, these features are blocked.
- **Recommendation**: Add "Temporal SDK framework + first workflow (nightly PMS sync)" to Phase 1 Week 3-4 deliverables, alongside PMS Integration Service work that already depends on it.

**FINDING-14: Platform Engineering Team Capacity in Phase 1**
- **Severity**: ENHANCEMENT
- **Detail**: 5 people (4 engineers + 1 lead) building 8-10 services in 4 weeks. Each service requires: database schema, API endpoints, tests, documentation, deployment config. Some services are straightforward (Practice Data Hub is CRUD + PostgreSQL), others are complex (PMS Integration requires NexHealth adapter + webhook receiver + event normalization + Kafka producer). The schedule appears achievable if services are built incrementally (MVP first, hardening in Phase 2), but leaves no buffer for unexpected issues.
- **Recommendation**: Identify which Phase 1 services are MVP (minimal viable) vs. production-grade. Practice Data Hub and Auth can be production-grade in Phase 1. PMS Integration and AI Service should target MVP with hardening in Phase 2.

### Riskiest Item in Build Sequence
**PMS Integration Service (Weeks 3-4)**. This service has the most external dependencies (NexHealth API, Sikka ONE API, both requiring BAAs and vendor onboarding), the most complex integration logic (webhook handling, event normalization, two-adapter pattern), and is on the critical path for both WS1 and WS2 in Phase 3. If NexHealth onboarding is delayed, the entire Phase 3 timeline shifts.

---

## Review Area 6: ADR Quality

### Methodology
Evaluated each ADR for: clear context, well-reasoned decision, honest consequences (both positive and negative), genuine alternatives considered (not strawmen), and traceability to original sprint decisions.

| ADR | Title | Quality | Issues |
|---|---|---|---|
| ADR-01 | NexHealth + Sikka ONE Dual-PMS | STRONG | Well-reasoned. Four alternatives genuinely considered and rejected with evidence. Correctly identifies the coverage gap risk. |
| ADR-02 | Shared Twilio | STRONG | Excellent motivation (the duplicate opt-out scenario is concrete and compelling). Consequences are balanced. |
| ADR-03 | HIPAA Foundational | STRONG | Correctly rejects "defer to post-MVP." Good cost comparison (3-5x retrofit cost). |
| ADR-04 | Kafka Event Bus | GOOD | Honestly acknowledges Kafka may be overkill at this scale. MSK Serverless mitigates over-provisioning. Could have considered AWS EventBridge more thoroughly. |
| ADR-05 | Design Tool Abstraction | GOOD | Proportionate response to the risk. Lightweight abstraction is the right call. |
| ADR-06 | WS1 Split | STRONG | User-group analysis is clear. Frontend-only split (shared backend) is pragmatic. |
| ADR-07 | Claude as Primary AI | GOOD | See finding below. |

### Findings

**FINDING-15: ADR-07 Does Not Address Claude BAA Feasibility**
- **Severity**: ENHANCEMENT
- **Detail**: ADR-07 selects Claude as the primary AI provider. The consequences note "vendor concentration risk" but do not discuss BAA feasibility. The risk register (R10) mentions Anthropic BAA as a procurement risk. ADR-07 should acknowledge that the entire AI strategy depends on Anthropic offering a HIPAA BAA — if Anthropic declines, the PHI filter becomes the only control layer, which may be insufficient for audit purposes.
- **Impact**: If Anthropic cannot or will not sign a BAA, the architecture must either (a) guarantee the PHI filter has zero leakage (unrealistic) or (b) switch to a provider that offers BAAs (OpenAI has HIPAA offerings). This contingency is not documented.
- **Recommendation**: Add a sentence to ADR-07: "Contingency: If Anthropic BAA is unavailable, the AI Service abstraction layer enables migration to OpenAI GPT-4 (which offers HIPAA-eligible endpoints) with prompt library re-engineering estimated at 2-4 weeks."

**FINDING-16: ADR-04 Dismisses AWS EventBridge Too Quickly**
- **Severity**: ENHANCEMENT
- **Detail**: ADR-04 rejects EventBridge with "limited replay (24 hours), no partition-based ordering, higher per-event cost at volume." However, EventBridge Archive now supports replay up to indefinite retention, and at 260K events/day, EventBridge cost would be approximately $0.26/day ($1 per million events). The ordering concern is valid but only matters for events requiring per-practice ordering (which Kafka partitions provide). Most of the 10 contracts are idempotent and do not require strict ordering. EventBridge + SQS would have significantly lower operational overhead than MSK, even in serverless mode.
- **Impact**: The Kafka decision is defensible (schema registry, replay, ordering are genuine advantages), but the EventBridge rejection rationale contains factual inaccuracies that weaken the ADR.
- **Recommendation**: Update the EventBridge alternative to reflect current capabilities. The decision to use Kafka can stand on schema registry + ordering merits alone.

---

## Findings Summary

| # | Finding | Severity | Category | Recommendation |
|---|---------|----------|----------|----------------|
| F-01 | Phone, Ad, Social, HR integrations missing from shared infra | MAJOR | Coverage | Define integration strategy for non-PMS data sources |
| F-09 | No breach notification or incident response architecture | MAJOR | HIPAA | Add breach detection, assessment, notification pipeline |
| F-10 | No disaster recovery or backup architecture | MAJOR | HIPAA | Add RPO/RTO targets, backup strategy, DR plan |
| F-02 | Social platform API management undefined | MINOR | Coverage | Clarify ownership (WS2-internal or shared) |
| F-03 | iSolved HR integration unaddressed | MINOR | Coverage | Add to Practice Data Hub data sources |
| F-04 | Missing event: WS2 assets to WS3 recognition | MINOR | Events | Add lightweight contract or document polling |
| F-05 | Missing schema: huddle-completed event | MINOR | Events | Define schema or mark as WS3-internal |
| F-06 | Missing schema: practice.staff-updated event | MINOR | Events | Add formal schema definition |
| F-07 | Appointment completed schema missing provider_id | MINOR | Schema | Add provider_staff_id UUID field |
| F-11 | Cloudinary BAA classification contradictory | MINOR | HIPAA | Require BAA or ensure true de-identification |
| F-12 | BAA procurement timeline unrealistic | MINOR | Build | Start procurement pre-Phase 0 |
| F-13 | Phase 1 omits Temporal framework build | MINOR | Build | Add Temporal SDK to Phase 1 Week 3-4 |
| F-08 | No correlation_id in event envelope | ENHANCEMENT | Schema | Add optional correlation_id field |
| F-14 | Phase 1 team capacity has no buffer | ENHANCEMENT | Build | Define MVP vs. production-grade per service |
| F-15 | ADR-07 missing Claude BAA contingency | ENHANCEMENT | ADR | Add fallback to OpenAI if BAA unavailable |
| F-16 | ADR-04 EventBridge rejection contains inaccuracies | ENHANCEMENT | ADR | Update with current EventBridge capabilities |

---

## Severity Counts

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| MAJOR | 3 |
| MINOR | 10 |
| ENHANCEMENT | 4 |
| **Total** | **17** |

---

## Verdict: REVISE

**Conditions for PASS**:
1. **Address F-01**: Define where phone system, ad platform, AI phone agent, and social platform API integrations live in the architecture. These do not need to be shared services — they can be documented as WS1-internal (phone, ads) and WS2-internal (social) — but they must appear in the build sequence with dependencies and team ownership.
2. **Address F-09**: Add a breach notification and incident response section to the Compliance & Audit Layer. This requires architectural support (detection triggers, notification workflows, forensic preservation), not just a policy document.
3. **Address F-10**: Add disaster recovery and backup architecture with RPO/RTO targets for each data tier and critical service failover procedures.

**Items that can be deferred to implementation**: All MINOR and ENHANCEMENT findings. These are real issues but none block the build from starting once the 3 MAJOR items are resolved.

---

## Positive Observations

The following aspects of the architecture are well-executed:

1. **Shared service decomposition** is clean and well-motivated. Each service has clear ownership, rationale, and consumer documentation.
2. **Event contract definitions** are thorough — JSON schemas with trigger conditions, latency SLAs, and explicit contract owners are rare in architecture documents at this stage.
3. **PHI handling** is thoughtful throughout. The PHI filter in the AI Service, hashed patient IDs in cross-workstream events, and double encryption for PHI fields show genuine security engineering.
4. **ADRs are honest**. Negative consequences are documented alongside positives. Alternatives are genuinely considered rather than strawmanned. This is better than most enterprise architecture documents.
5. **Build sequence dependency analysis** is clear and the dependency graph is well-constructed. WS3 early start is a smart scheduling decision.
6. **The Compliance & Audit Layer as cross-cutting middleware** (Fastify plugin pattern) rather than a standalone proxy service is the correct architectural choice for performance-sensitive PHI operations.
7. **Schema quality** is high — consistent naming, appropriate use of enums, typed nested objects, and clear separation between identifiers and display values.
