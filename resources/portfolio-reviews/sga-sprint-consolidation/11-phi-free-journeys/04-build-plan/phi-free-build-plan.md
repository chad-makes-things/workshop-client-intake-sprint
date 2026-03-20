# PHI-Free-First Build Plan — SGA Dental Partners

**Date**: 2026-03-20
**Author**: Product Manager Agent (Phase 11 — PHI-Free Restructure)
**Purpose**: Define the phased build plan for the PHI-free-first development approach, with sub-phase definitions, QC gates, team allocation, and risk-adjusted timelines.
**Companion Documents**: [PHI Scope Analysis](../01-scope/phi-scope-analysis.md) | [Experience Architecture](../02-experience/phi-free-experience-architecture.md) | [Original Build Map](../../05-roadmap/build-map.md) | [Original Milestone Definitions](../../05-roadmap/milestone-definitions.md)

---

## 1. Build Philosophy

### How PHI-Free-First Changes Everything

The original build plan treated PHI infrastructure as a prerequisite for all work. BAA procurement was on the critical path from Day 1. Compliance Middleware, PMS Integration Service, and Communications Service were built in Weeks 3-6 alongside PHI-free infrastructure, meaning the entire shared infrastructure phase was gated on legal agreements that SGA does not control.

The PHI-free-first approach removes that gate. Development starts immediately on the 71% of the platform that requires zero patient data. BAA procurement runs in parallel, not on the critical path. The result: time to first value drops from 6-7 weeks (original M1.5) to 5-6 weeks, and the platform delivers a complete operational product before any PHI enters the system.

### Governing Principles

1. **Build what you control first.** Practice profiles, staff data, culture scores, brand assets, and enrollment pipelines are all within SGA's control. Patient data requires third-party BAAs, PMS vendor cooperation, and compliance infrastructure. Build the controllable foundation first.

2. **Infrastructure serves features, not compliance timelines.** In the original plan, Compliance Middleware was built in Weeks 4-5 whether or not any PHI-bearing feature was ready. In the PHI-free plan, Compliance Middleware is built in Phase B0 when PHI features are actually approaching readiness. Infrastructure is built when its consumers are ready.

3. **6 before 10.** The original plan built all 10 shared services in Weeks 3-6. The PHI-free plan builds 6 PHI-free services first (Practice Data Hub, Auth & Identity, Kafka, Asset Registry, Workflow Engine, Design Tool Abstraction), then adds 4 PHI-bearing services in Phase B0 (Compliance Middleware, PMS Integration Service, Communications Service, AI Service PHI extensions). This reduces Week 3-6 scope by 40% and allows faster stabilization.

4. **WS3 and WS2 validate infrastructure; WS1 benefits.** Culture OS (100% PHI-free) and Content Engine (89% PHI-free) are the ideal workstreams to prove shared infrastructure under real load. WS1 Practice Platform splits: its PHI-free components (enrollment, storefronts, executive dashboards with aggregate data) build in Phase A; its patient-facing components (Nerve Center patient cards, Concierge patient SMS, Automation Engine) build in Phase B.

5. **BAA procurement is a background process, not a blocking gate.** Legal starts BAA procurement on Day 1. The build does not wait for completion. If BAAs close early, Phase B starts early. If BAAs are delayed, Phase A is a complete, shippable product.

6. **Phase A is a product, not a prototype.** Phase A delivers practice culture management, brand content pipeline, executive dashboards, practice onboarding, and storefront generation to 260+ practices. It is a production system serving real users, not a placeholder waiting for PHI.

7. **The PHI boundary is a hard line, not a gradient.** No "borderline" PHI data enters the system in Phase A. If there is any question about whether a data point constitutes PHI under HIPAA, it is excluded until Phase B compliance infrastructure is verified. Conservative classification protects the organization.

---

## 2. Sub-Phase Definitions

### Phase A: PHI-Free Foundation

---

### Sub-Phase A0: Pre-Build (Weeks 1-2)

**Purpose**: Environment setup, team onboarding, architecture review, and data model design with PHI hooks for Phase B.

**What changed from original M0**: BAA execution moves to a parallel background track. M0 no longer gates on BAA completion. Twilio HIPAA account provisioning and PMS vendor agreements are tracked but do not block A1. Data model design explicitly includes PHI table schemas (empty in Phase A) to ensure clean Phase B activation.

#### Deliverables

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A0.1 | Team staffing & onboarding | Engineering Manager | 1-2 | All 5 team leads hired/allocated. 15 of 20 engineers confirmed. Architecture documentation reviewed by all leads. |
| A0.2 | AWS environment provisioning | Platform Engineering | 1-2 | Dev, staging, prod accounts created. IAM configured. VPC networking operational. HIPAA-eligible services selected (even though PHI is not flowing yet — infrastructure is Phase B-ready from Day 1). |
| A0.3 | CI/CD pipeline | Platform Engineering | 1-2 | GitHub Actions pipeline deploys to dev ECS cluster. Automated test stage operational. |
| A0.4 | Temporal cluster (dev) | Platform Engineering | 2 | Temporal Cloud or self-hosted accessible from dev environment. |
| A0.5 | Kafka cluster (dev) | Platform Engineering | 2 | MSK dev cluster operational. Topics creatable. Schema Registry deployed. |
| A0.6 | Monitoring setup | Platform Engineering | 2 | Datadog (or CloudWatch) dashboards and basic alerts operational. |
| A0.7 | Data model design with PHI hooks | Platform Engineering + Tech Lead | 1-2 | PostgreSQL schemas for Practice Data Hub include `patient_records`, `patient_communications`, `call_transcripts` tables — all empty, all with row-level security and encryption-at-rest configured. ClickHouse schemas include `funnel_events`, `communication_events`, `call_events` tables — empty but ready. |
| A0.8 | BAA procurement initiated (parallel) | Legal | 1 (start) | Outreach letters sent to AWS, Twilio, NexHealth, Sikka ONE, Anthropic, OpenAI. Tracking spreadsheet maintained. Does NOT gate A1. |
| A0.9 | Feature flag framework | Platform Engineering | 2 | Feature flag service deployed (LaunchDarkly or equivalent). Phase A/B flag taxonomy defined. All teams trained on flag-driven development. |

#### Dependencies
- Engineering Manager identified and available
- AWS account approval (if new organization)
- Legal team availability to initiate BAA outreach

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| Platform Engineering (5) | Environment setup, data model design, CI/CD | 100% |
| All other teams | Onboarding, architecture review, local dev setup | 100% |
| Legal | BAA outreach (background) | 20% |

#### Success Metrics
- CI/CD pipeline deploys a hello-world service to ECS
- All team leads have completed architecture review and can describe the PHI-free/PHI-enabled boundary
- Data model schemas reviewed and approved by Tech Lead and Platform Architect
- Feature flag framework operational with Phase A default flags set
- BAA outreach initiated for all 6 vendors (completion not required)

---

### Sub-Phase A1: Shared Infrastructure — PHI-Free Services (Weeks 3-6)

**Purpose**: Build the 6 shared services that require zero PHI data. These form the operational backbone for all three workstreams.

**What changed from original M1**: Scope reduced from 10 services to 6. Compliance Middleware, PMS Integration Service, Communications Service, and the PHI components of AI Service are deferred to B0. This reduces the Week 3-6 build scope by approximately 40% and eliminates all BAA dependencies from the critical path.

#### Deliverables

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A1.1 | Practice Data Hub | Platform Eng | 3-4 | CRUD API live. 260 practices seeded from SGA master list. Staff roster management API operational. Region hierarchy and director mapping. Practice configuration and feature flags. Read replica configured. `patient_records` table exists but empty (PHI hook). |
| A1.2 | Auth & Identity Service | Platform Eng | 3-4 | Cognito user pool configured. JWT middleware npm package published. RBAC service deployed with 8 roles. Test users for all roles. API key management for service-to-service auth. |
| A1.3 | Event Bus (Kafka) | Platform Eng | 4-5 | MSK operational in staging. Schema Registry with all 10 event schemas registered (including PHI event schemas that will not emit until Phase B). Producer/consumer SDK npm package published. |
| A1.4 | Asset Registry | Platform Eng | 5-6 | S3 buckets configured with CloudFront CDN. Upload/download API. PostgreSQL metadata catalog. Cloudinary integration for image transforms. |
| A1.5 | Workflow Engine (Temporal) | Platform Eng | 5-6 | Temporal cluster operational in staging. Practice enrollment workflow template deployed. H-Score daily calculation workflow template deployed. |
| A1.6 | Design Tool Abstraction | Platform Eng | 6 | Provider interface defined. Canva adapter operational. Template listing and generation working for storefront and content templates. |

#### Dependencies
- A0 complete: environments provisioned, CI/CD operational, data model schemas approved
- No external vendor dependencies (no BAAs required)
- Canva API access confirmed (not a BAA — standard SaaS agreement)

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| Platform Engineering (5) | All 6 shared services | 100% |
| WS3 Team (3) | Test infrastructure setup, H-Score spike, Cinnamon/Swell API research | 50% |
| QA (2) | Test infrastructure setup, integration test harness | 100% |
| WS1, WS2 teams | Local dev against service stubs, story grooming | 30% |

#### Success Metrics
- All 6 shared services return 200 on health check endpoints
- End-to-end integration test: authenticate (JWT) -> call Practice Data Hub -> publish event to Kafka -> consumer receives -> audit log entry
- 260 practices seeded in Practice Data Hub with complete profile data
- Asset Registry accepts upload, stores in S3, returns CDN URL
- Workflow Engine executes a practice enrollment workflow end-to-end
- Design Tool Abstraction generates a test storefront from Canva template

---

### Sub-Phase A2: WS3 Culture OS Full Build + WS2 Content Engine Core (Weeks 5-10)

**Purpose**: Build Culture OS to full capability (all 8 interfaces) and Content Engine core pipeline (7 of 8 Phase A interfaces). These two workstreams have the fewest PHI dependencies and validate shared infrastructure under real practice load.

**What changed from original plan**: WS3 starts 2 weeks earlier (Week 5 vs. original Week 7) because it no longer waits for Compliance Middleware and PMS Integration. WS2 core starts at Week 7 (vs. original Week 11) because the Content Engine pipeline is content-type-agnostic and does not need PHI services. Overlap between A1 and A2 is intentional — WS3 begins as soon as Practice Data Hub + Auth + Kafka are operational (Week 5).

#### Deliverables

**WS3 Culture OS (Weeks 5-10)**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A2.1 | H-Score Engine v1 | WS3 | 5-6 | Cinnamon API integration (phone scores). Swell API integration (review scores + NLP tagging). Composite H-Score calculation (daily batch via Temporal). Practice dashboard showing score + 3 components + trend. `ws3.hscore-calculated` event publishing to Kafka. |
| A2.2 | Morning Huddle OS v1 | WS3 | 6-7 | Dental Intel integration. Content packages auto-loading. 12 weeks of content available. Completion tracking + streak indicator. Yesterday's Wins using culture/brand metrics. Today's Focus targeting lowest H-Score sub-score. |
| A2.3 | CCA Academy Bronze v1 | WS3 | 7-8 | 10 modules live on mobile. Scenario assessments. Credly badge issuance. Certificate generation. Mux video hosting integration. |
| A2.4 | Trust Transfer OS v1 | WS3 | 8-9 | Swell review NLP tagging for trust transfer signals. Huddle commitment tap. Trust transfer sentiment score (30-day rolling, aggregate per staff member). |
| A2.5 | Champion Network v1 | WS3 | 8-9 | Champion readiness algorithm. Designation workflow. `champion-assigned` event publishing. Champion scorecard. Regional cohort leaderboard. |
| A2.6 | Recognition & Rewards v1 | WS3 | 9-10 | Recognition feed. Peer celebration interface. Leaderboard. Basic rewards tracking. |
| A2.7 | Playbook Distribution v1 | WS3 | 9-10 | Versioned playbook content. Practice-type filtering. Adoption tracking. Deployment workflow. |
| A2.8 | Culture Admin Console | WS3 | 10 | CCA curriculum management. Playbook deployment. Huddle content configuration. Champion network management. Reward budgets. |

**WS2 Content Engine Core (Weeks 7-10)**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A2.9 | Content Capture Bot v1 | WS2 | 7-8 | Daily SMS capture briefs (brand/culture-driven, not procedure-driven). MMS capture. Confirmation with streak count. Uses SMS gateway (simple — not Twilio HIPAA, since no PHI in content capture messages). |
| A2.10 | Studio Agent v1 | WS2 | 7-8 | Email intake (studio@sgadental.com). Teams intake. Practice identification from sender. Max 3 clarifying questions. Brief extraction. |
| A2.11 | Production Pipeline v1 | WS2 | 8-9 | Template-based generation (Canva API via Design Tool Abstraction). AI photo processing (non-patient content). AI caption generation. Medium challenge. |
| A2.12 | Editorial Queue v1 | WS2 | 9 | HQ QC queue (kanban). Approve/reject/feedback flow. AI caption edit before approval. `content-asset-approved` event to Kafka. |
| A2.13 | Distribution Engine v1 | WS2 | 9-10 | Instagram + Facebook publishing via Meta Graph API. Scheduling queue. |
| A2.14 | Asset Registry Integration | WS2 | 10 | All approved assets stored in Asset Registry. Metadata catalog populated. Brand health score calculation from asset metadata. |
| A2.15 | Champion Weekly SMS Recap | WS2 | 10 | Automated weekly recap to champions: captures submitted, assets published, engagement metrics. |

#### Dependencies
- A1 services operational: Practice Data Hub (Week 4), Auth & Identity (Week 4), Kafka (Week 5), Asset Registry (Week 6), Workflow Engine (Week 6), Design Tool Abstraction (Week 6)
- WS3 starts at Week 5 (needs Practice Data Hub + Auth + Kafka only)
- WS2 starts at Week 7 (needs all 6 A1 services)
- External: Cinnamon API credentials, Swell API credentials, Dental Intel API access, Credly API credentials, Mux account, Meta Graph API access
- SMS gateway for Content Capture Bot (non-HIPAA SMS provider acceptable for Phase A since messages contain zero PHI)

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| WS3 Team (3) | All WS3 deliverables (A2.1-A2.8) | 100% |
| WS2 Team (4) | Content Engine core (A2.9-A2.15) | 100% (starting Week 7) |
| Platform Engineering (5) | A1 completion (Weeks 5-6), then contract hardening + support (Weeks 7-10) | 60% build / 40% support |
| QA (2) | WS3 integration testing, content pipeline E2E testing | 100% |

#### Success Metrics
- 20 pilot practices with active H-Scores updating daily
- 70%+ of pilot practices complete at least 1 morning huddle in first week
- First CCA Bronze certifications earned by pilot practice staff
- Content Capture Bot generating captures from 10+ pilot practices
- Content pipeline E2E: capture -> process -> review -> approve -> distribute to Instagram
- All WS3 Kafka events verified by consumer contract tests
- `ws3.hscore-calculated` consumed by at least one downstream service

---

### Sub-Phase A3: WS1 Practice Platform PHI-Free Components (Weeks 8-14)

**Purpose**: Build WS1's PHI-free features: enrollment pipeline, storefront generation, executive dashboards (with aggregate/culture/brand metrics), HQ operations tools, and practice daily agenda (modified for Phase A).

**What changed from original plan**: WS1 originally started all features at Week 11 after event contract hardening. In the PHI-free plan, WS1's non-PHI features start at Week 8, overlapping with A2. Patient-facing WS1 features (Nerve Center patient cards, Concierge patient SMS, Automation Engine patient sequences) are deferred to Phase B1. Executive dashboards display culture/brand/enrollment KPIs instead of revenue/funnel KPIs.

#### Deliverables

**WS1-A: Practice Operations — PHI-Free (Weeks 8-12)**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A3.1 | Data Harvesting Engine | WS1-A | 8-9 | Automated practice data harvesting (web + GMB + social). Batch harvest for 260+ practices. Practice profiles pre-populated in Practice Data Hub. |
| A3.2 | Enrollment Invitation System | WS1-A | 9-10 | Email invitations to practice cohorts. Tracking (sent/opened/clicked/enrolled). Enrollment workflow via Temporal. |
| A3.3 | Storefront Generation v1 | WS1-A | 10-11 | 2 templates (GP + 1 specialty). Site generation engine (JAMstack). CDN deployment. Practice preview + approval flow. Automated deployment and activation. |
| A3.4 | Practice Daily Agenda (Phase A) | WS1-A | 11-12 | Practice Briefing (profile summary, H-Score trend widget, content capture brief, enrollment status). Phase A Action Cards (storefront health, content capture, playbook update, culture focus). Practice Activity Summary (captures, huddle, storefront updates, playbook views). |
| A3.5 | Playbook Viewer | WS1-A | 11-12 | Access current playbooks. Version history. Practice-type filtering. |
| A3.6 | Graduated Autonomy v1 (PHI-free subset) | WS1-A | 12 | Autonomy level management framework deployed. Weekly summary generator for office managers (content/culture metrics only — patient metrics deferred). |

**WS1-B: Network Intelligence — PHI-Free (Weeks 10-14)**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| A3.7 | Executive Dashboard (Phase A) | WS1-B | 10-11 | 5 KPI cards: Active Practices, Network H-Score, Avg Brand Health Score, Enrollment Pipeline, Content Captures MTD. AI Alerts panel (culture/brand/enrollment anomalies). Trend charts. |
| A3.8 | Portfolio Heatmap (Phase A) | WS1-B | 11-12 | Geographic practice display with 5 color layers: H-Score, Brand Health, Content Activity, Enrollment Status, CCA Certification Level. Practice tooltip on hover. Click for side panel. |
| A3.9 | Intervention Queue (Phase A) | WS1-B | 12-13 | AI-ranked practices by Culture Risk Score. Root cause analysis from non-PHI signals. Assign/dismiss/view actions. |
| A3.10 | HQ Ops Standup Board (Phase A) | WS1-B | 12-13 | Prioritized practice queue (culture/brand/enrollment). Batch operations for storefront updates. Enrollment pipeline status. QC queues (website QC, brand health violations). |
| A3.11 | Practice Deep Dive (Phase A) | WS1-B | 13-14 | 4-tab structure: Profile, Culture, Content & Brand, History. Tab injection framework ready for Phase B Performance and Communications tabs. |
| A3.12 | Enrollment Pipeline Dashboard | WS1-B | 13-14 | Track new practice onboarding. Data harvesting status. Integration connectivity. Blitz deploy countdown. Go-live readiness. |
| A3.13 | Compliance Readiness Dashboard | WS1-B | 13-14 | BAA execution status per vendor. Encryption verification. Audit infrastructure status. Consent configuration. Phase B readiness checklist. |
| A3.14 | Network Learning Report v1 | WS1-B | 14 | Monthly AI-generated report: culture trends, brand health patterns, enrollment velocity, content effectiveness. Non-PHI signals only. |

#### Dependencies
- A1 shared services stable (verified by A2 workstream usage)
- A2 WS3 events flowing (H-Score, champion-assigned for executive dashboard)
- A2 WS2 Content Engine events flowing (brand health, capture metrics for dashboard)
- Kafka consumer contract tests passing for all cross-workstream events
- Google Business Profile API access confirmed (storefront health monitoring)

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| WS1-A Team (4) | Practice Operations PHI-free features (A3.1-A3.6) | 100% |
| WS1-B Team (3) | Network Intelligence PHI-free features (A3.7-A3.14) | 100% |
| Platform Engineering (5) | Contract hardening, support, Kafka load testing | 30% build / 70% support |
| WS3 Team (3) | Continued WS3 refinement, cross-WS integration | 50% |
| WS2 Team (4) | Continued WS2 refinement, cross-WS integration | 50% |
| QA (2) | WS1 integration testing, cross-workstream E2E | 100% |

#### Success Metrics
- 260 practices harvested and profiles populated in Practice Data Hub
- 50+ practices enrolled via invitation system
- 10+ storefronts generated and deployed via template engine
- Executive Dashboard loading in < 3 seconds with live H-Score, brand health, enrollment data
- Portfolio Heatmap displaying all 260 practices with H-Score color coding
- Practice Deep Dive showing 4 tabs with live data for all pilot practices
- Compliance Readiness Dashboard showing BAA procurement status for all 6 vendors
- All 10 cross-workstream event flows verified (at least the PHI-free subset — 8 of 10)

---

### Phase B: PHI Layer

---

### Sub-Phase B0: PHI Infrastructure (Weeks 12-15)

**Purpose**: Build the 4 PHI-bearing shared services and the compliance infrastructure required before any patient data enters the system. This is the most critical phase for regulatory compliance.

**What changed from original plan**: These services were originally built in Weeks 3-6 alongside PHI-free infrastructure. Moving them to Weeks 12-15 means they are built with the benefit of: (a) stable PHI-free infrastructure already proven in production, (b) BAAs that have had 12+ weeks to close, (c) a team that has 10 weeks of experience with the platform architecture.

**Critical prerequisite**: B0 cannot begin until ALL required BAAs are executed. If BAAs are not complete by Week 12, B0 start date slides. Phase A continues to deliver value independently.

#### Deliverables

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| B0.1 | BAA execution verification | Legal + Tech Lead | 12 | BAAs signed with: AWS (HIPAA BAA), Twilio (HIPAA-eligible account), NexHealth, Sikka ONE, Anthropic, OpenAI. All BAAs reviewed by HIPAA Compliance Counsel. |
| B0.2 | Compliance Middleware | Platform Eng | 12-13 | Fastify plugin npm package published. PHI detection engine operational. Application-level AES-256-GCM encryption helpers. Append-only immutable audit log. PHI access event logging. All PHI access requires Compliance Middleware — no bypass path. |
| B0.3 | PHI Filter for AI Service | Platform Eng | 13 | Tokenize/detokenize pipeline operational. PHI stripped before any LLM API call. Detokenization on response. Audit log entry for every PHI filter operation. Token accounting per workstream. |
| B0.4 | PMS Integration Service | Platform Eng | 13-14 | NexHealth adapter for Dentrix, Eaglesoft, Open Dental. Webhook receiver. Canonical event normalization. PMS events emitting to Kafka through Compliance Middleware. Appointment booking write API. Sikka ONE batch adapter for non-NexHealth practices. |
| B0.5 | Communications Service | Platform Eng | 14-15 | Twilio HIPAA-eligible account activated. SMS send API with consent check. Consent registry (opt-in/opt-out tracking). Per-practice number provisioning + 10DLC registration. Message routing rules (quiet hours, rate limits, dedup). Email send via AWS SES. All patient communications encrypted and audit-logged. |
| B0.6 | AI Service PHI extensions | Platform Eng | 14-15 | Claude API integration with PHI filter. Cost-tier routing (Haiku for classification, Sonnet for generation). Prompt registry seeded with patient-facing prompts. OpenAI Whisper integration for audio transcription (call recordings). |
| B0.7 | PHI data model activation | Platform Eng | 15 | `patient_records`, `patient_communications`, `call_transcripts` tables in PostgreSQL activated with test data. `funnel_events`, `communication_events`, `call_events` tables in ClickHouse activated. Row-level security verified. Encryption at rest verified. |
| B0.8 | PHI red-team test | QA + Security | 15 | Penetration test targeting PHI data flows. PHI leakage test (verify PHI cannot escape to non-PHI surfaces). Audit log completeness verification. Encryption verification for data at rest and in transit. Access control verification (only authorized roles can access PHI). |

#### Dependencies
- ALL BAAs executed and verified (hard gate — no exceptions)
- A1 infrastructure stable and proven in production
- At least 2 workstreams (WS3 + WS2) running on A1 infrastructure for 6+ weeks
- Twilio HIPAA account approved and 10DLC campaigns registered
- NexHealth sandbox/test account with real PMS test data
- Anthropic API access confirmed with HIPAA-eligible tier

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| Platform Engineering (5) | All B0 deliverables (full-time PHI infrastructure) | 100% |
| QA (2) | PHI security testing, penetration testing, compliance verification | 100% |
| Legal | BAA verification, compliance review | 30% |
| Security Engineer (contract) | PHI red-team test, encryption audit | 100% (2 weeks) |
| WS1, WS2, WS3 teams | Continue Phase A refinement, prepare Phase B feature branches | 70% A maintenance / 30% B preparation |

#### Success Metrics
- All 6 BAAs signed and verified by HIPAA Compliance Counsel
- Compliance Middleware PHI detection: 100% recall on test PHI dataset (zero false negatives)
- AI Service PHI filter: round-trip tokenize/detokenize returns correct result for 100 test cases
- PMS Integration receives real NexHealth webhook and emits canonical event through Compliance Middleware
- Communications Service sends real SMS with consent check, quiet hours enforcement, and full audit trail
- PHI red-team test: zero PHI leakage paths identified (or all identified paths remediated before gate)
- Audit log: every PHI access event captured with timestamp, actor, resource, action, and justification

---

### Sub-Phase B1: Patient-Facing Features (Weeks 15-20)

**Purpose**: Build all patient-facing features that were deferred from Phase A. These features require PMS data, patient communications infrastructure, and compliance middleware.

#### Deliverables

**WS1-A: Practice Operations — Patient Features**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| B1.1 | Practice Nerve Center (patient cards) | WS1-A | 15-16 | Daily agenda with AI-prioritized patient action cards (callback, confirm, follow-up, review). PMS-driven card generation. Cards merge with Phase A practice cards. |
| B1.2 | Concierge Agent v1 | WS1-A | 16-17 | Chat-based data verification. Brand voice personality. Missed call text-back (< 90s). Form fill auto-response (< 30s). All via Communications Service. |
| B1.3 | Patient Outreach Queue | WS1-A | 17-18 | AI-drafted patient messages. Review/edit/send workflow. Scheduled outreach. Consent verification before every send. |
| B1.4 | Automation Engine v1 | WS1-A | 18-19 | Appointment confirmation sequences (48h/24h/4h). Playbook cadence definitions. Compliance rules engine (HIPAA, TCPA, quiet hours). Missed call recovery (< 5 min). Standard 4-touch treatment follow-up. |
| B1.5 | Graduated Autonomy (patient extensions) | WS1-A | 19-20 | Patient interaction metrics added to autonomy scoring. Weekly summary includes patient communication outcomes. |

**WS1-B: Network Intelligence — Patient Analytics**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| B1.6 | Funnel Data Lake activation | WS1-B | 15-16 | ClickHouse `funnel_events` table receiving PMS events. Unified funnel event schema. dbt-managed derived views. Data quality monitoring. |
| B1.7 | Executive Dashboard (Phase B KPIs) | WS1-B | 16-17 | KPI bar expands to 8 cards: adds Total Production, New Patients, Show Rate. AI alerts incorporate revenue impact. |
| B1.8 | Funnel Analytics Explorer | WS1-B | 17-18 | Custom funnel analysis. Cohort comparison. Practice-level and network-level views. |
| B1.9 | AI Analysis Engine v1 | WS1-B | 18-19 | Per-practice funnel baselines (90-day rolling). Anomaly detection. Revenue impact estimation. |
| B1.10 | Practice Deep Dive (Phase B tabs) | WS1-B | 19-20 | Performance tab (5-stage funnel, conversion rates, revenue trend). Communications tab (message volumes, response rates). Tab injection into existing 4-tab framework. |

**WS2: Content Engine — Patient Content**:

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| B1.11 | Patient photo capture pipeline | WS2 | 16-17 | Before/after photo pairing enabled in Capture App. Patient consent workflow integrated. Patient-mapped photos stored with PHI encryption. |
| B1.12 | Procedure-driven capture briefs | WS2 | 17-18 | Content Capture Bot briefs now driven by PMS `appointment-completed` events. Procedure code triggers. "2 implant cases today" style prompts. |
| B1.13 | Patient outreach content | WS2 | 18-19 | Post-treatment testimonial follow-up prompts (30/60/90 day). Patient email deployment via Weave integration. |

#### Dependencies
- B0 complete and QC Gate B0 passed (hard gate — no exceptions)
- PHI red-team test passed with zero critical findings
- All BAAs executed and verified
- PMS Integration Service receiving real webhooks from NexHealth
- Communications Service sending real patient SMS with consent verification
- Compliance Middleware operational and enforcing on all PHI access

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| WS1-A Team (4) | Patient-facing practice operations (B1.1-B1.5) | 100% |
| WS1-B Team (3) | Patient analytics and dashboards (B1.6-B1.10) | 100% |
| WS2 Team (4) | Patient content pipeline (B1.11-B1.13) + Phase A maintenance | 70% B1 / 30% A maintenance |
| WS3 Team (3) | Phase A maintenance + cross-WS integration support | 30% B1 support / 70% A maintenance |
| Platform Engineering (5) | PHI infrastructure support + scaling | 20% build / 80% support |
| QA (2) | Patient feature integration testing, HIPAA compliance verification | 100% |

#### Success Metrics
- 20-30 practices on full Practice Nerve Center with patient action cards
- Concierge: < 90s missed call text-back verified in production
- Automation Engine running confirmation sequences for 20+ practices
- Funnel Data Lake receiving PMS events with < 4-hour lag to ClickHouse views
- Executive Dashboard showing all 8 KPI cards with live data
- Patient photo pipeline operational with consent verification for 5+ practices
- Zero HIPAA compliance violations in first 2 weeks of PHI operation

---

### Sub-Phase B2: Advanced PHI + Network Rollout (Weeks 20-24)

**Purpose**: Build advanced PHI features (Voice AI, call analytics, attribution) and roll out the full platform (Phase A + Phase B) to 260+ practices.

#### Deliverables

| # | Deliverable | Team | Week | Exit Criteria |
|---|-------------|------|------|---------------|
| B2.1 | Voice AI inbound v1 | WS1-A + Platform | 20-22 | Twilio Voice + Media Streams integration. Real-time PMS lookup during calls. Appointment booking via NexHealth write API. Call transcript generation via Whisper. PHI-compliant recording storage. |
| B2.2 | Call Dashboard | WS1-A | 21-22 | Incoming call context with patient records. Call outcome logging. Call transcript viewer. Voice AI quality monitoring. |
| B2.3 | Content-to-consultation attribution | WS2 + WS1-B | 22-23 | Content published -> new patient inquiries -> bookings attribution model. Requires funnel data + content distribution data correlation. |
| B2.4 | Trust Transfer individual analysis | WS3 | 22-23 | Individual call-level trust transfer scores (requires Cinnamon call recording access). Call playback integration. Staff coaching recommendations based on individual call analysis. |
| B2.5 | Advanced automation playbooks | WS1-A | 22-24 | No-show re-engagement. Same-day rebooking. Hygiene recall sequences. Dormant patient reactivation (tiered campaigns). High-value treatment follow-up extensions. |
| B2.6 | Network rollout to 260+ practices | All teams | 20-24 | Rolling activation: 50 practices/week. Per-practice Phase B feature flag activation after BAA + PMS + consent verified. Self-service onboarding for new acquisitions (< 2 hours HQ effort). |
| B2.7 | Full Compliance Dashboard | WS1-B | 22-24 | Live HIPAA audit logs. PHI access reports. Opt-out compliance monitoring. Consent status per practice. Real-time compliance alerts. |
| B2.8 | Network Learning Report (full) | WS1-B | 23-24 | Monthly AI-generated report with funnel insights, revenue attribution, communication effectiveness, cross-practice conversion patterns — all PHI signals incorporated. |

#### Dependencies
- B1 complete and stable for 2+ weeks
- NexHealth write API stability verified (for Voice AI booking)
- Twilio Media Streams for real-time voice processing
- Cinnamon call recording API access (for individual trust transfer analysis)
- Practice rollout pipeline capable of activating 50+ practices/week
- HQ operations team trained on full Compliance Dashboard

#### Team Allocation
| Team | Focus | % Capacity |
|------|-------|-----------|
| WS1-A Team (4) | Voice AI, Call Dashboard, advanced automations | 80% B2 / 20% support |
| WS1-B Team (3) | Attribution, compliance dashboard, learning reports | 80% B2 / 20% support |
| WS2 Team (4) | Attribution model, patient content expansion | 50% B2 / 50% maintenance |
| WS3 Team (3) | Individual trust transfer, maintenance | 40% B2 / 60% maintenance |
| Platform Engineering (5) | Voice AI infrastructure, scaling for 260+ practices | 50% build / 50% ops |
| QA (2) | Voice AI testing, rollout QA, compliance verification | 100% |

#### Success Metrics
- Voice AI handling inbound calls for 20+ practices with real-time PMS booking
- Call Dashboard live for 50+ practices
- Content-to-consultation attribution model producing insights for 30+ practices
- 260+ practices active on Phase A features; 100+ practices active on Phase B features
- Full Compliance Dashboard monitoring all PHI access with zero gaps
- Platform uptime >= 99.5% over trailing 30 days
- Self-service practice onboarding demonstrated in < 2 weeks end-to-end

---

## 3. QC Gates

### Gate A0: Pre-Build Readiness

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 2 |
| **Reviewers** | Platform Architect, Engineering Manager, Scrum Master |
| **Key Question** | Is the team staffed, environment provisioned, and data model approved — ready to build shared services? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] At least 15 of 20 engineers confirmed and onboarded
- [ ] Dev, staging, and prod AWS accounts operational
- [ ] CI/CD pipeline deploys hello-world service to dev ECS
- [ ] PostgreSQL schemas reviewed and approved (including empty PHI tables with RLS)
- [ ] ClickHouse schemas reviewed and approved (including empty PHI tables)
- [ ] Feature flag framework deployed and Phase A defaults set
- [ ] Kafka dev cluster operational
- [ ] All team leads have reviewed architecture documentation

**Failure Protocol**: If staffing is < 12 engineers, escalate to CTO with 1-week extension and accelerated recruiting. If AWS accounts are blocked, escalate to SGA IT leadership. No partial starts — all criteria must pass.

---

### Gate A1: PHI-Free Infrastructure Verified

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 6 |
| **Reviewers** | Platform Architect, QA Engineer, Scrum Master, Product Owner |
| **Key Question** | Are all 6 PHI-free shared services operational, integrated, and load-tested? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] All 6 shared services return 200 on health check endpoints in staging
- [ ] End-to-end integration test passes: Auth -> Practice Data Hub -> Kafka -> Consumer -> Audit log
- [ ] Practice Data Hub: 260 practices seeded with complete profile data
- [ ] Kafka: all 10 event schemas registered in Schema Registry (including Phase B schemas)
- [ ] Asset Registry: upload/download/CDN URL generation verified
- [ ] Workflow Engine: practice enrollment workflow executes end-to-end
- [ ] Design Tool Abstraction: storefront template generation from Canva verified
- [ ] Load test: 260 concurrent practice-scoped API calls complete within SLA (< 500ms p95)
- [ ] Zero security vulnerabilities in infrastructure security scan

**Failure Protocol**: Identify failing service. 1-week remediation sprint. WS3 start delayed only if Practice Data Hub, Auth, or Kafka fail (the 3 services WS3 depends on). Other service failures do not block WS3.

---

### Gate A2: Culture + Content Validation

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 10 |
| **Reviewers** | Product Owner, Design Director, Customer Researcher, QA Engineer, Scrum Master |
| **Key Question** | Are WS3 Culture OS and WS2 Content Engine core delivering real value to pilot practices? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] H-Score calculating daily for >= 15 of 20 pilot practices
- [ ] Morning Huddle content auto-loading for >= 15 pilot practices
- [ ] 70%+ of pilot practice staff have completed at least 1 huddle
- [ ] CCA Academy: first Bronze certifications earned
- [ ] Content Capture Bot: >= 50% of prompts result in a capture within 48 hours
- [ ] Content pipeline E2E verified: capture -> process -> review -> approve -> distribute
- [ ] At least 5 of 10 cross-workstream Kafka events verified by contract tests
- [ ] Kafka consumer lag < 30 seconds under simulated 260-practice load
- [ ] DLQ monitoring active for all consumer groups
- [ ] Zero PHI data found in any Phase A data flow (compliance spot check)

**Failure Protocol**: If pilot practice engagement is low (< 50% huddle completion), Customer Researcher investigates with 2-week remediation. If Kafka events are failing contract tests, Platform Engineering gets 1-week remediation sprint. WS1 start is delayed only if Kafka event infrastructure is unreliable.

---

### Gate A3: PHI-Free Platform Complete

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 14 |
| **Reviewers** | Product Owner, Design Director, Strategist, Platform Architect, QA Engineer, Security Engineer, Scrum Master |
| **Key Question** | Is the full Phase A platform operational, delivering value to practices, and architecturally ready for Phase B PHI integration? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] 30 interfaces operational (30 of 42 total)
- [ ] 50+ practices active on at least 2 workstreams simultaneously
- [ ] Executive Dashboard displaying 5 KPI cards with live data (< 5-minute freshness)
- [ ] Portfolio Heatmap rendering all 260 practices with accurate H-Score coloring
- [ ] Enrollment Pipeline tracking new practice onboarding
- [ ] 10+ storefronts generated and deployed via template engine
- [ ] Content pipeline producing >= 3 pieces per active practice per week
- [ ] Cross-workstream integration: 8 of 10 event flows verified in production
- [ ] Compliance Readiness Dashboard showing BAA procurement status
- [ ] Performance: all services meet p95 < 500ms latency SLA
- [ ] Security audit passed for Phase A infrastructure
- [ ] Phase B tab injection framework tested (inject fake Performance tab, verify no layout breaks)
- [ ] Feature flag system verified: Phase B flags toggle correctly without affecting Phase A surfaces
- [ ] Zero PHI data anywhere in the system (full audit)

**Failure Protocol**: Any failure triggers a targeted remediation sprint (1-2 weeks). Phase B start is delayed by the remediation duration. Phase A continues serving practices during remediation. If > 3 criteria fail, escalate to CTO for timeline reassessment.

---

### Gate B0: PHI Readiness (CRITICAL GATE)

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 15 |
| **Reviewers** | HIPAA Compliance Counsel, Security Engineer, Platform Architect, QA Engineer, Product Owner, Scrum Master |
| **Key Question** | Is the platform safe to receive, process, store, and transmit patient health information? |
| **Gate Type** | Hard gate — HIGHEST SEVERITY |

This is the most critical gate in the entire build. No patient data flows through the system until this gate passes. A failure here does not delay Phase A (which continues operating). It delays only Phase B patient features.

**Must-Pass Criteria**:

*BAA Verification Checklist*:
- [ ] AWS HIPAA BAA executed and filed
- [ ] Twilio HIPAA BAA executed; HIPAA-eligible account confirmed
- [ ] NexHealth BAA executed and filed
- [ ] Sikka ONE BAA executed and filed
- [ ] Anthropic BAA (or equivalent data processing agreement) executed
- [ ] OpenAI BAA (or equivalent data processing agreement) executed
- [ ] All BAAs reviewed by HIPAA Compliance Counsel and deemed sufficient
- [ ] BAA coverage verified: every service that will process PHI is covered by a BAA

*Encryption & Security Verification*:
- [ ] Encryption at rest: AES-256 for all databases containing PHI tables (PostgreSQL, ClickHouse)
- [ ] Encryption at rest: S3 SSE-KMS for all buckets that will store patient content
- [ ] Encryption in transit: TLS 1.2+ on all service-to-service communication
- [ ] Application-level encryption: AES-256-GCM for PHI fields in PostgreSQL (double encryption)
- [ ] Key management: AWS KMS keys for all encryption; key rotation policy configured
- [ ] Audit log: append-only, immutable, captures every PHI access event
- [ ] Audit log retention: >= 6 years (HIPAA minimum)
- [ ] Access control: RBAC enforcement verified — only authorized roles access PHI data
- [ ] Network isolation: PHI services in private subnets; no public internet exposure
- [ ] Secret management: all credentials in AWS Secrets Manager; no hardcoded secrets

*PHI Red-Team Testing Protocol*:
- [ ] External penetration test by certified security firm (or contract Security Engineer)
- [ ] PHI leakage test: Inject synthetic PHI into PMS Integration -> verify it does NOT appear in any Phase A surface, any non-PHI API response, any log file, any error message, any Kafka topic not designated for PHI
- [ ] PHI filter test: Send 100 synthetic patient records through AI Service -> verify PHI is tokenized before LLM API call and detokenized on return -> verify no PHI in API request logs
- [ ] Compliance Middleware bypass test: Attempt to access PHI data without going through Compliance Middleware -> verify all bypass attempts are blocked
- [ ] Cross-service PHI boundary test: Verify PHI data from PMS Integration Service is only accessible to services with explicit PHI authorization (Communications Service, AI Service, Funnel Data Lake) and NOT to services without authorization (Asset Registry, Design Tool Abstraction, Playbook Engine)
- [ ] Audit log completeness test: Perform 50 PHI operations -> verify all 50 are captured in audit log with correct metadata
- [ ] Consent verification test: Attempt to send patient SMS without consent record -> verify blocked
- [ ] Data deletion test: Request patient data deletion -> verify cascading deletion across all PHI tables and audit log retention of deletion event

*Infrastructure Verification*:
- [ ] Compliance Middleware health check passing in production
- [ ] PHI Filter for AI Service: 100% recall on test PHI dataset (zero false negatives allowed)
- [ ] PMS Integration Service receiving and processing real NexHealth webhooks
- [ ] Communications Service sends test SMS with full consent check, quiet hours, rate limiting
- [ ] ClickHouse PHI tables receiving test events with correct encryption
- [ ] DLQ monitoring active for all PHI-bearing Kafka topics

**Failure Protocol**: ANY failure on the BAA, encryption, or red-team checklists blocks Phase B entirely. There is no partial pass. The team remediates all findings, and the gate is re-run. HIPAA Compliance Counsel has veto authority — if they are not satisfied, the gate does not pass regardless of technical criteria. Estimated remediation: 1-2 weeks per finding. Multiple findings may be remediated in parallel.

---

### Gate B1: Patient Features Operational

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 20 |
| **Reviewers** | Product Owner, Design Director, QA Engineer, HIPAA Compliance Counsel, Customer Researcher, Scrum Master |
| **Key Question** | Are patient-facing features delivering value to practices with full HIPAA compliance? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] 30+ practices on full Practice Nerve Center (Phase A + Phase B cards)
- [ ] Concierge Agent: < 90s missed call text-back verified by monitoring (95th percentile)
- [ ] Automation Engine: confirmation sequences running for 20+ practices
- [ ] Patient Outreach Queue: AI-drafted messages reviewed and sent with consent verification
- [ ] Funnel Data Lake: PMS events flowing with < 4-hour lag to ClickHouse views
- [ ] Executive Dashboard: all 8 KPI cards live with accurate data
- [ ] Funnel Analytics Explorer: custom funnel queries returning correct results
- [ ] Patient photo pipeline: consent verified, PHI encryption confirmed for 5+ practices
- [ ] Zero HIPAA compliance violations since Phase B activation
- [ ] Audit log review: all PHI access events properly captured
- [ ] Patient opt-out processed within 24 hours across all communication channels

**Failure Protocol**: Patient-facing feature failures trigger immediate investigation. If HIPAA compliance violation is detected, affected feature is immediately disabled via feature flag until remediated. Customer Researcher conducts practice satisfaction survey to verify user experience quality.

---

### Gate B2: Full Platform Launch

| Attribute | Detail |
|-----------|--------|
| **When** | End of Week 24 |
| **Reviewers** | Product Owner, Strategist, Platform Architect, HIPAA Compliance Counsel, QA Engineer, Security Engineer, Scrum Master |
| **Key Question** | Is the full platform ready for 260+ practice network rollout with all features active? |
| **Gate Type** | Hard gate |

**Must-Pass Criteria**:
- [ ] 100+ practices active on both Phase A and Phase B features
- [ ] Voice AI handling inbound calls for 20+ practices with < 3% error rate
- [ ] All 42 interfaces operational
- [ ] All 10 cross-workstream event flows verified in production
- [ ] Executive Dashboard unified view with all KPI cards and live data
- [ ] Content-to-consultation attribution model producing insights
- [ ] Full Compliance Dashboard monitoring all PHI access
- [ ] Load test: system handles 260-practice concurrent load across all services within SLA
- [ ] Self-service onboarding demonstrated: new practice activated in < 2 weeks
- [ ] Incident response runbook documented and tested for all critical services
- [ ] Rollback plan documented and tested for each workstream
- [ ] External HIPAA compliance review passed
- [ ] Platform uptime >= 99.5% over trailing 14 days

**Failure Protocol**: Any compliance failure blocks rollout expansion. Performance failures trigger scaling remediation. Feature failures allow rollout to continue for practices using working features while specific features are remediated.

---

## 4. Milestone Timeline

```
PHASE A: PHI-FREE FOUNDATION
═══════════════════════════════════════════════════════════════════════════════════════════

Week:     1    2    3    4    5    6    7    8    9   10   11   12   13   14
          |----|----|----|----|----|----|----|----|----|----|----|----|----|----|
          GA0       |         GA1       |              GA2              |    GA3
          Pre-Build | PHI-Free Infra    |  Culture+Content+WS1 Build   | Full A
          |         |                   |                              |
PLATFORM  |=========|==================|==============================|
ENGR (5)  | Env     |PracHub Auth      | Contract hardening           |
          | CI/CD   |Kafka Asset        | Kafka load test              |
          | Schemas |WF Eng Design      | Support WS teams             |
          |         |                   |                              |
WS3 (3)   |         |                   |===========|==================|
          | Onboard |  Spike/research   | H-Score Huddle CCA           | Recog Playbook
          |         |                   | Trust Champion               | Admin Cross-WS
          |         |                   |                              |
WS2 (4)   |         |                   |     |====|==================|
          | Onboard |  Story grooming   |     | CaptureBot Studio    | Distro Asset
          |         |                   |     | Pipeline Editorial   | Champion Recap
          |         |                   |                              |
WS1-A (4) |         |                   |          |===================|
          | Onboard |  Story grooming   |          | Harvest Enroll   | Agenda Playbook
          |         |                   |          | Storefront       | Grad Autonomy
          |         |                   |                              |
WS1-B (3) |         |                   |               |=============|
          | Onboard |  Story grooming   |               | ExecDash   | Heatmap Queue
          |         |                   |               |            | DeepDive Enroll
          |         |                   |               |            | Compliance Learn
          |         |                   |                              |
QA (2)    |         |==========================================|======|
          |         | Test infra  | Contract tests | WS integ  | E2E + perf
          |         |             | Load tests     | testing    |
          |         |                   |                              |
LEGAL     |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ |
          | BAA outreach (parallel background track — does NOT block) |
          |                                                            |
Practices:                        20 pilot                    50+ enrolled      260 harvested
                                  (WS3 H-Score)               (multi-WS)       (profiles)


PHASE B: PHI LAYER
═══════════════════════════════════════════════════════════════════════════════════════════

Week:    12   13   14   15   16   17   18   19   20   21   22   23   24
         |----|----|----|----|----|----|----|----|----|----|----|----|----|----|
                        GB0       |              GB1              |    GB2
              PHI Infrastructure  | Patient-Facing Features      | Advanced + Rollout
              |                   |                              |
PLATFORM |========================|                              |
ENGR (5) | Compliance Middleware  | Support + scaling            | Voice AI infra
         | PHI Filter  PMS Integ |                              | 260+ scaling
         | Comms Svc  AI PHI ext |                              |
         | Red-team test         |                              |
         |                       |                              |
WS1-A(4) |                       |============================|==================|
         | Prep feature branches | Nerve Center  Concierge    | Voice AI  Call Dash
         |                       | Outreach Queue Automation  | Adv Automations
         |                       | Grad Autonomy extensions   |
         |                       |                              |
WS1-B(3) |                       |============================|==================|
         | Prep feature branches | Funnel Lake  ExecDash B    | Attribution Compliance
         |                       | Funnel Explorer AI Engine  | Learning Report full
         |                       | Deep Dive B tabs           |
         |                       |                              |
WS2 (4)  |                       |     |=======================|==========|
         | A maintenance         |     | Patient photos  Procedure briefs | Attribution
         |                       |     | Patient outreach content         |
         |                       |                              |
WS3 (3)  |                       |                              |==========|
         | A maintenance         | Cross-WS support            | Trust Transfer
         |                       |                              | individual analysis
         |                       |                              |
QA (2)   |========================|============================|==================|
         | PHI security testing  | Patient feature integration | Voice AI testing
         | Penetration testing   | HIPAA compliance verify    | Rollout QA
         | Compliance verify     |                              |
         |                       |                              |
Practices:                       30+ Phase B pilot    100+ Phase A+B     260+ rollout
                                                                         (50/week)


OVERLAP: Phase A3 (Weeks 8-14) and Phase B0 (Weeks 12-15) overlap by 3 weeks.
This is intentional: WS1 PHI-free features complete while PHI infrastructure builds.
Teams do NOT overlap — Platform Engineering shifts to B0 while WS1 teams finish A3.
```

---

## 5. Cross-Workstream Integration Schedule

### Event Flow Activation by Sub-Phase

| # | Event Flow | Producer | Consumer | Activation Sub-Phase | Week First Live | Notes |
|---|-----------|----------|----------|---------------------|-----------------|-------|
| 1 | `ws3.hscore-calculated` -> WS1 | WS3 H-Score Engine | WS1 Executive Dashboard | **A2** -> **A3** | Week 7 (produced) / Week 11 (consumed by dashboard) | First cross-WS event. H-Score is PHI-free. |
| 2 | `ws3.champion-assigned` -> WS1, WS2 | WS3 Champion Network | WS1 Executive Dashboard, WS2 Capture Bot | **A2** -> **A3** | Week 9 (produced) / Week 11 (consumed) | Champion roster drives content capture assignments. |
| 3 | `ws3.benchmark-taxonomy-updated` -> WS1, WS2 | WS3 Admin Console | WS1 Practice Deep Dive, WS2 Pipeline | **A2** -> **A3** | Week 10 (produced) / Week 13 (consumed) | Taxonomy changes propagate to all workstreams. |
| 4 | `ws3.champion-story-submitted` -> WS2 | WS3 Champion Hub | WS2 Content Pipeline | **A2** -> **A2** | Week 9 (both) | Champion stories feed content pipeline. PHI-free. |
| 5 | `pms.appointment-completed` -> WS2 | PMS Integration Service | WS2 Capture Bot | **B0** -> **B1** | Week 17 | PHI event. Triggers procedure-driven capture briefs. Blocked until B0 passes. |
| 6 | `ws2.content-asset-approved` -> WS1 | WS2 Editorial Queue | WS1 Storefront, Asset Gallery | **A2** -> **A3** | Week 10 (produced) / Week 11 (consumed) | Approved assets available in storefront gallery. PHI-free. |
| 7 | `ws1.funnel-event` -> WS2 | WS1 Funnel Data Lake | WS2 Attribution Engine | **B1** -> **B2** | Week 22 | PHI event. Content-to-consultation attribution. Requires funnel data. |
| 8 | `ws2.content-engagement-updated` -> WS1 | WS2 Distribution Engine | WS1 Executive Dashboard | **A2** -> **A3** | Week 10 (produced) / Week 11 (consumed) | Social engagement metrics. PHI-free. |
| 9 | `ws3.training-completed` -> WS1 | WS3 CCA Academy | WS1 Executive Dashboard | **A2** -> **A3** | Week 8 (produced) / Week 11 (consumed) | CCA completion events drive dashboard metrics. PHI-free. |
| 10 | `ws1.onboarding-status-changed` -> WS3 | WS1 Enrollment Pipeline | WS3 Culture Activation | **A3** -> **A2** | Week 10 (produced) / Week 10 (consumed) | New practice onboarding triggers culture program setup. PHI-free. |

### Summary

| Sub-Phase | Events Activated | PHI Status |
|-----------|-----------------|------------|
| A2 (Weeks 5-10) | Events 1, 2, 3, 4, 6, 8, 9, 10 (produced) | All PHI-free |
| A3 (Weeks 8-14) | Events 1, 2, 3, 6, 8, 9 (consumed by WS1) | All PHI-free |
| B0 (Weeks 12-15) | Event 5 infrastructure ready | PHI — requires B0 gate pass |
| B1 (Weeks 15-20) | Event 5 (live), Event 7 (produced) | PHI |
| B2 (Weeks 20-24) | Event 7 (consumed — attribution) | PHI |

**8 of 10 event flows are fully operational in Phase A.** Only events 5 (PMS appointment -> capture) and 7 (funnel -> attribution) require PHI and activate in Phase B.

---

## 6. Comparison to Original Plan

| Dimension | Original Plan | PHI-Free-First Plan | Advantage |
|-----------|--------------|--------------------|-----------|
| **Time to first value** | Week 6-7 (M1.5: H-Score + Huddle for 20 practices) | Week 6-7 (same — H-Score + Huddle for 20 practices, but with faster infrastructure completion) | Neutral. Both deliver WS3 pilot at similar timing. |
| **Time to multi-workstream value** | Week 14 (M3: all WS Tier 1 features including PHI-dependent features) | Week 10-12 (A2+A3: WS3 full + WS2 core + WS1 PHI-free features) | **PHI-free-first is 2-4 weeks faster** to multi-WS value because PHI infrastructure does not block WS2 or WS1 PHI-free features. |
| **BAA dependency on critical path** | Yes — BAAs must close by Week 2 (M0 gate). BAA delay = entire build delay. | No — BAAs run in parallel. BAA delay = Phase B delay only. Phase A is a complete product. | **PHI-free-first eliminates BAA from critical path.** |
| **Infrastructure complexity (Weeks 3-6)** | 10 services built simultaneously (including Compliance Middleware, PMS Integration, Communications Service requiring BAAs) | 6 services built (only PHI-free services). 40% reduction in Week 3-6 scope. | **Reduced scope = faster stabilization, lower risk of infrastructure delays.** |
| **Risk profile** | High — any BAA delay, PMS integration issue, or compliance middleware problem delays entire platform. | Low for Phase A (zero regulatory risk). Risk concentrated in Phase B where it can be managed independently. | **Risk isolation.** Phase A risk is purely technical. Phase B risk includes regulatory + technical, but affects only Phase B timeline. |
| **Team utilization (Weeks 1-10)** | WS1-A (4), WS1-B (3), WS2 (4) idle from Weeks 1-10 (waiting for M2 event contracts). 11 engineers idle for 10 weeks = 110 person-weeks wasted. | WS2 starts Week 7, WS1 starts Week 8-10. Only 3-5 weeks of reduced utilization during early A1 (onboarding/research time). ~30 person-weeks saved. | **Significantly better team utilization.** |
| **Compliance risk during build** | PHI flows through system from Week 5-6 (as PMS Integration and Communications Service go live). Compliance incidents possible during build phase. | Zero PHI in system until Week 15+ (after B0 gate pass). No compliance risk during 14 weeks of Phase A build. | **14 weeks of compliance-risk-free development.** |
| **Total timeline to full platform** | Week 18 (M4: integrated launch with all WS Tier 1 features) | Week 20-24 (B1-B2: all patient features + network rollout) | **Original is 2-6 weeks faster to full feature set.** This is the tradeoff: PHI-free-first trades total timeline for de-risking and earlier partial value. |
| **Practice rollout** | 20 (M1.5) -> 50 (M4) -> 150 (M5) -> 260 (M6) | 20 (A2) -> 50 (A3) -> 260 Phase A (A3) -> 100 Phase B (B1) -> 260 Phase B (B2) | **PHI-free-first reaches 260 practices on Phase A features faster** (Week 14 vs. Week 30 for full rollout in original). Full PHI rollout is comparable. |
| **Shippable product if BAAs fail** | No standalone product. Entire platform depends on PHI infrastructure. | Phase A is a complete product: culture management, content pipeline, executive dashboards, storefronts, enrollment — all without PHI. | **PHI-free-first has a fallback product.** If BAAs are delayed indefinitely, SGA still has a functioning platform. |

---

## 7. Risk-Adjusted Timeline

### Scenario 1: BAAs Take Longer Than Expected

| Scenario | BAA Completion | Impact on Phase A | Impact on Phase B | Mitigation |
|----------|---------------|-------------------|-------------------|------------|
| **On-time** | Week 10-12 | None | B0 starts Week 12 as planned | — |
| **2-week delay** | Week 14 | None | B0 starts Week 14. B1 starts Week 17. B2 starts Week 22. Total: +2 weeks. | Phase A continues delivering value. No team idle time — WS teams refine Phase A features. |
| **4-week delay** | Week 16 | None | B0 starts Week 16. B1 starts Week 19. B2 starts Week 23. Total: +4 weeks. | Phase A is fully operational for 260 practices. Use delay to harden Phase A, expand storefront templates, deepen content pipeline. |
| **8-week delay** | Week 20 | None | B0 starts Week 20. B1 starts Week 23. B2 starts Week 27. Total: +8 weeks. | Significant delay to patient features. Evaluate: (a) escalate to vendor leadership, (b) identify alternative vendors with faster BAA processes, (c) consider phased BAAs (AWS first, then others). Phase A serves as production platform during delay. |
| **BAAs fail** | Never | None — Phase A is a complete product | Phase B cancelled or redesigned | Phase A delivers culture management, brand content, executive dashboards, storefronts, enrollment for 260+ practices indefinitely. This is the "worst case" and it is still a functioning product. |

### Scenario 2: Phase A Infrastructure Takes Longer

| Scenario | A1 Completion | Impact | Mitigation |
|----------|--------------|--------|------------|
| **On-time** | Week 6 | Nominal | — |
| **1-week delay** | Week 7 | A2 starts 1 week late. Cascading 1-week delay through A3. B0 start unchanged (BAA-gated, not A1-gated). | Compress A2 by parallelizing WS3 H-Score and Huddle builds (originally sequential). |
| **2-week delay** | Week 8 | A2 and A3 each compress by 1 week. Total timeline +1 week (some compression absorbs delay). | WS3 team starts H-Score spike against service stubs while A1 completes. WS2 starts capture bot against mock APIs. |
| **4-week delay** | Week 10 | Significant restructuring needed. A2 and A3 merge into a single compressed phase. | Escalate to CTO. Evaluate: (a) bring in additional Platform Engineering contractors, (b) reduce A1 scope to 4 critical services (Practice Data Hub, Auth, Kafka, Asset Registry), (c) defer Workflow Engine and Design Tool Abstraction to later. |

### Scenario 3: Pilot Practice Engagement Is Low

| Scenario | Engagement Rate | Impact | Mitigation |
|----------|----------------|--------|------------|
| **Target** | 70%+ huddle completion, 50%+ capture response | Nominal | — |
| **Low** | 30-50% huddle, 20-30% capture | Phase A metrics look weak. Executive confidence drops. | Customer Researcher conducts practice interviews within 1 week. Likely causes: (a) poor champion selection — re-select, (b) unclear value prop — update onboarding materials, (c) technology friction — UX audit and fix. |
| **Very low** | < 30% huddle, < 20% capture | Phase A thesis at risk. | Pause rollout expansion. Deep-dive into 5 lowest-engagement practices. Consider: (a) incentive program for early adopters, (b) in-person training sessions, (c) SGA Operations champion embedded with pilot cohort for 2 weeks. |

### Scenario 4: Cross-Workstream Integration Issues

| Scenario | Impact | Mitigation |
|----------|--------|------------|
| Kafka consumer lag > 60s under load | Dashboard data staleness. Executive trust erosion. | Investigate consumer bottleneck. Options: increase partition count, optimize consumer code, add consumer instances. Budget 1 week. |
| Event schema mismatches between workstreams | Consumer failures, DLQ accumulation | Contract testing should catch this at Gate A2. If missed: 1-week remediation sprint. Add schema validation CI step to prevent recurrence. |
| H-Score calculation drift between WS3 and WS1 dashboard | Executives see different numbers in different places | Root cause: caching, timing, or rounding differences. Fix: single source of truth (WS3 Kafka event is canonical). WS1 dashboard never recalculates — only displays. |

---

## 8. Phase B Readiness Checklist

This checklist must be completed before ANY patient data flows through the system. It is the operational companion to QC Gate B0. Every item is verified by a named reviewer. No exceptions, no partial passes.

### Legal & Contractual

| # | Item | Verified By | Status |
|---|------|-------------|--------|
| L-1 | AWS HIPAA BAA executed and countersigned | HIPAA Compliance Counsel | [ ] |
| L-2 | Twilio HIPAA BAA executed; HIPAA-eligible account confirmed by Twilio | HIPAA Compliance Counsel | [ ] |
| L-3 | NexHealth BAA executed and countersigned | HIPAA Compliance Counsel | [ ] |
| L-4 | Sikka ONE BAA executed and countersigned | HIPAA Compliance Counsel | [ ] |
| L-5 | Anthropic data processing agreement with PHI provisions executed | HIPAA Compliance Counsel | [ ] |
| L-6 | OpenAI data processing agreement with PHI provisions executed | HIPAA Compliance Counsel | [ ] |
| L-7 | All BAAs reviewed for adequacy by HIPAA Compliance Counsel | HIPAA Compliance Counsel | [ ] |
| L-8 | SGA internal HIPAA policies updated to reflect platform operations | HIPAA Compliance Counsel | [ ] |
| L-9 | Breach notification procedures documented and tested | HIPAA Compliance Counsel + Security Engineer | [ ] |
| L-10 | Data retention and destruction policies documented | HIPAA Compliance Counsel | [ ] |

### Technical Infrastructure

| # | Item | Verified By | Status |
|---|------|-------------|--------|
| T-1 | Compliance Middleware deployed to production, health check passing | Platform Architect | [ ] |
| T-2 | PHI Filter for AI Service operational in production | Platform Architect | [ ] |
| T-3 | PMS Integration Service receiving real NexHealth webhooks in production | Platform Architect | [ ] |
| T-4 | Communications Service sending real SMS via Twilio HIPAA account | Platform Architect | [ ] |
| T-5 | All PHI database tables encrypted at rest (AES-256) | Security Engineer | [ ] |
| T-6 | Application-level encryption (AES-256-GCM) on PHI fields verified | Security Engineer | [ ] |
| T-7 | TLS 1.2+ enforced on all service-to-service communication | Security Engineer | [ ] |
| T-8 | AWS KMS key rotation configured | Security Engineer | [ ] |
| T-9 | Audit log operational: append-only, immutable, 6-year retention | Security Engineer | [ ] |
| T-10 | Audit log captures: timestamp, actor, resource, action, justification for every PHI access | QA Engineer | [ ] |
| T-11 | RBAC enforcement: verified that non-authorized roles cannot access PHI endpoints | QA Engineer | [ ] |
| T-12 | PHI services in private subnets with no public internet exposure | Security Engineer | [ ] |
| T-13 | All secrets in AWS Secrets Manager; no hardcoded credentials in code or config | Security Engineer | [ ] |
| T-14 | DLQ monitoring active for all PHI-bearing Kafka topics | Platform Architect | [ ] |
| T-15 | Backup and disaster recovery tested for all PHI databases | Platform Architect | [ ] |

### Security Testing

| # | Item | Verified By | Status |
|---|------|-------------|--------|
| S-1 | External penetration test completed with zero critical/high findings (or all remediated) | Security Engineer | [ ] |
| S-2 | PHI leakage test: synthetic PHI injected -> verified absent from all non-PHI surfaces | QA Engineer | [ ] |
| S-3 | PHI filter test: 100 synthetic records -> tokenized before LLM -> detokenized on return | QA Engineer | [ ] |
| S-4 | Compliance Middleware bypass test: all bypass attempts blocked | Security Engineer | [ ] |
| S-5 | Cross-service PHI boundary test: PHI only accessible to authorized services | Security Engineer | [ ] |
| S-6 | Audit log completeness: 50 operations -> all 50 captured | QA Engineer | [ ] |
| S-7 | Consent verification: SMS without consent -> blocked | QA Engineer | [ ] |
| S-8 | Data deletion: patient deletion cascades correctly across all PHI tables | QA Engineer | [ ] |
| S-9 | Session management: PHI access sessions timeout after 15 minutes of inactivity | Security Engineer | [ ] |
| S-10 | Error handling: no PHI exposed in error messages, stack traces, or log files | QA Engineer | [ ] |

### Operational Readiness

| # | Item | Verified By | Status |
|---|------|-------------|--------|
| O-1 | Incident response runbook for PHI breach documented | Security Engineer + HIPAA Compliance Counsel | [ ] |
| O-2 | On-call rotation established for PHI services | Scrum Master | [ ] |
| O-3 | PHI access monitoring dashboard operational (real-time alerts for anomalous access) | Platform Architect | [ ] |
| O-4 | Patient opt-out workflow tested end-to-end (request -> cascade -> confirmation) | QA Engineer | [ ] |
| O-5 | Staff HIPAA training completed for all engineers with PHI access | HIPAA Compliance Counsel | [ ] |
| O-6 | Compliance Dashboard (NI-9) operational and displaying live metrics | QA Engineer | [ ] |
| O-7 | Feature flag system verified: Phase B features can be instantly disabled per-practice | Platform Architect | [ ] |
| O-8 | Rollback plan tested: PHI features can be deactivated without affecting Phase A surfaces | Platform Architect | [ ] |

### Final Sign-Off

| Reviewer | Signature | Date |
|----------|-----------|------|
| HIPAA Compliance Counsel | _________________ | ___/___/______ |
| Security Engineer | _________________ | ___/___/______ |
| Platform Architect | _________________ | ___/___/______ |
| Product Owner | _________________ | ___/___/______ |
| Engineering Manager | _________________ | ___/___/______ |

**All five signatures required before Phase B activation. No exceptions.**

---

## Appendix: Sub-Phase Dependency Map

```
A0 (Pre-Build)
 |
 v
A1 (PHI-Free Infrastructure)
 |          \
 v           \
A2 (WS3 + WS2 Core)     BAA Procurement (parallel)
 |          \                    |
 v           v                   |
A3 (WS1 PHI-Free)               |
 |                               |
 |   +---------------------------+
 |   |
 v   v
B0 (PHI Infrastructure) <-- REQUIRES: A1 stable + ALL BAAs signed
 |
 v
B1 (Patient Features) <-- REQUIRES: B0 gate passed
 |
 v
B2 (Advanced PHI + Rollout) <-- REQUIRES: B1 stable for 2+ weeks
```

---

*PHI-Free-First Build Plan prepared by Product Manager Agent | 2026-03-20*
*This document supersedes the original build-map.md and milestone-definitions.md for the PHI-free-first approach.*
*For the original plan, see: [Build Map](../../05-roadmap/build-map.md) | [Milestone Definitions](../../05-roadmap/milestone-definitions.md)*
