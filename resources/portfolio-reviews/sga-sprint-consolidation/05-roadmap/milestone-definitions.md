# SGA Dental Partners — Milestone Definitions

**Date**: 2026-03-14
**Author**: Product Manager Agent (Phase 6)
**Purpose**: Define milestones with success criteria, gate conditions, and dependencies for the consolidated 3-workstream build.

---

## Milestone Overview

| # | Milestone | Weeks | Key Deliverable | Gate Type |
|---|---|---|---|---|
| M0 | Pre-Build Complete | 1-2 | Team staffed, vendors contracted, environments provisioned | Hard gate |
| M1 | Shared Infrastructure Live | 3-6 | All 10 shared services operational with integration tests | Hard gate |
| **M1.5** | **Partial Value Delivery** | **6-7** | **WS3 H-Score + Huddle live in 20 pilot practices** | **Soft gate** |
| M2 | Event Contracts Hardened + WS3 MVP | 7-10 | All 10 event contracts tested; WS3 CCA + Champions live | Hard gate |
| M3 | WS1 + WS2 Tier 1 Features | 11-14 | Practice Nerve Center, Concierge, Content Capture live | Hard gate |
| M4 | Integrated Platform Launch | 15-18 | All 3 workstreams operational; cross-workstream events flowing | Hard gate |
| M5 | Tier 2 Expansion | 19-24 | Automation engine, distribution engine, analytics expanded | Soft gate |
| M6 | Full Network Rollout | 25-30 | 260+ practices active across all workstreams | Hard gate |
| M7 | Tier 3 + Optimization | 31-36 | Learning loops, predictive analytics, full tooling | Soft gate |

---

## Milestone M0: Pre-Build Complete

**Target**: Weeks 1-2
**Owner**: Engineering Manager + Tech Lead + Legal

### Deliverables

| # | Deliverable | Owner | Exit Criteria |
|---|---|---|---|
| M0.1 | Team staffing | Engineering Manager | All 5 team leads hired/allocated. 15 of 20 engineers confirmed. |
| M0.2 | BAA execution | Legal + Tech Lead | BAAs signed with: AWS, Twilio, NexHealth, Sikka ONE, Anthropic, OpenAI |
| M0.3 | AWS environment provisioning | Platform Engineering | Dev, staging, prod accounts created. IAM configured. VPC networking. |
| M0.4 | CI/CD pipeline | Platform Engineering | GitHub Actions pipeline deploys to dev ECS cluster. |
| M0.5 | Temporal cluster | Platform Engineering | Temporal Cloud or self-hosted accessible from dev. |
| M0.6 | Kafka cluster (dev) | Platform Engineering | MSK dev cluster operational. Topics creatable. |
| M0.7 | Twilio HIPAA account | Platform Engineering | Account active. First 10DLC campaign submitted. Test numbers provisioned. |
| M0.8 | Monitoring setup | Platform Engineering | Datadog (or CloudWatch) dashboards + basic alerts operational. |

### Success Metrics
- 100% of critical vendor BAAs signed (NexHealth, Twilio, Anthropic are blockers)
- CI/CD pipeline successfully deploys a hello-world service to ECS
- All team leads have onboarded and reviewed architecture documentation

### Gate Criteria
- **Must be true to proceed to M1**: All BAAs signed. At least 12 of 20 engineers available. Dev environment operational.
- **Risk escalation**: If any BAA is unsigned after Week 2, escalate to CTO with backup vendor plan.

### Dependencies
- Legal team availability for BAA negotiations
- HR/recruiting pipeline for engineering hires
- AWS account approval (if new organization)

---

## Milestone M1: Shared Infrastructure Live

**Target**: Weeks 3-6
**Owner**: Platform Engineering Lead

### Deliverables

| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M1.1 | Practice Data Hub | 3-4 | CRUD API live. 260 practices seeded. Read replica configured. |
| M1.2 | Auth & Identity Service | 3-4 | Cognito user pool. JWT middleware npm package. RBAC service deployed. Test users for all 8 roles. |
| M1.3 | Compliance Middleware | 4-5 | Fastify plugin npm package published. Audit log operational. PHI detection + encryption helpers. |
| M1.4 | Event Bus (Kafka) | 4-5 | MSK operational. Schema Registry deployed. All 10 event schemas registered. Producer/consumer SDK published. |
| M1.5 | PMS Integration Service | 5-6 | NexHealth adapter for Dentrix + Eaglesoft + Open Dental. Webhook receiver. Canonical events emitting to Kafka. |
| M1.6 | Communications Service | 5-6 | Twilio SMS send API. Consent registry. Message routing rules (quiet hours, rate limits). 10 test practice numbers provisioned. |
| M1.7 | AI Service | 6 | Claude API integration. PHI filter operational. Prompt registry seeded. Token accounting. Whisper for audio. |
| M1.8 | Asset Registry | 6 | S3 buckets + CloudFront. Upload/download API. Metadata catalog. |
| M1.9 | Workflow Engine | 5-6 | Temporal cluster operational. Practice enrollment workflow template. |
| M1.10 | Design Tool Abstraction | 6 | Provider interface defined. Canva adapter operational. Template listing + generation working. |

### Success Metrics
- All 10 shared services return 200 on health check endpoints
- End-to-end integration test passes: authenticate (JWT) -> call Practice Data Hub -> publish event to Kafka -> consumer receives -> response logged to audit
- PMS Integration Service receives real NexHealth webhook and emits canonical event
- Communications Service sends real SMS via Twilio with consent check
- AI Service processes request with PHI filter (tokenize -> Claude -> detokenize)

### Gate Criteria (ALL must pass)
1. Practice Data Hub API returns data for 260 practices
2. JWT authentication + RBAC enforcement validated end-to-end
3. Compliance middleware logs PHI access event to audit log
4. Kafka producer/consumer test passes with schema validation for all 10 event types
5. PMS Integration emits `pms.appointment-created` from real NexHealth webhook
6. Communications Service sends SMS with consent check and message log entry
7. AI Service PHI filter round-trip (tokenize -> LLM -> detokenize) returns correct result
8. Asset Registry accepts upload, stores in S3, returns CDN URL
9. All services pass HIPAA-relevant security checklist (encryption at rest, TLS in transit, audit logging)
10. Load test: 260 concurrent practice-scoped API calls complete within SLA (< 500ms p95)

### Dependencies
- M0 complete (environments, BAAs, team)
- NexHealth sandbox/test account active
- Twilio HIPAA account approved
- Anthropic API access confirmed

---

## Milestone M1.5: Partial Value Delivery

**Target**: Weeks 6-7
**Owner**: WS3 Lead + Platform Engineering Lead
**Per Strategist carry-forward recommendation**

### Purpose
Deliver visible, measurable value to 20 pilot practices before the full platform launches. This proves the architecture works end-to-end and gives leadership something tangible to show stakeholders while WS1 and WS2 are still building.

### Deliverables

| # | Deliverable | Exit Criteria |
|---|---|---|
| M1.5.1 | H-Score Engine v1 | Cinnamon + Swell API connected. Composite score calculating daily for 20 pilot practices. Practice dashboard showing score + 3 components + trend. |
| M1.5.2 | Morning Huddle OS v1 | Dental Intel integration live. Content packages auto-loading. First 12 weeks of content available. Completion tracking + streak indicator working. |
| M1.5.3 | Pilot practice onboarding | 20 practices selected, enrolled, champions identified, baseline H-Scores calculated from 90-day historical data. |
| M1.5.4 | H-Score -> Kafka event | `ws3.hscore-calculated` event publishing to Kafka daily. At least one consumer (even if just a log consumer) validating the contract. |

### Success Metrics
- 20 pilot practices have active H-Scores with daily updates
- 70%+ of pilot practices complete at least 1 morning huddle in the first week
- H-Score baseline calculated for all 20 practices using historical Cinnamon/Swell data
- `ws3.hscore-calculated` Kafka events verified by contract test
- Zero PHI compliance violations in pilot

### Gate Criteria (Soft gate -- proceed to M2 regardless, but report status)
- H-Score calculating daily for >= 15 of 20 pilot practices
- Morning Huddle content auto-loading in Dental Intel for >= 15 practices
- At least one event contract validated end-to-end through Kafka

### Dependencies
- M1 services: Practice Data Hub, Auth, Kafka, Compliance Middleware
- External: Cinnamon API credentials, Swell API credentials, Dental Intel API access
- Operational: Pilot practice selection and champion identification by SGA Operations team

### Why This Matters
- Provides 6-week proof point for leadership ("the platform is already improving hospitality scores")
- Tests shared infrastructure under real load before WS1 and WS2 come online
- Generates real event data for Kafka contract hardening
- WS3 has the fewest infrastructure dependencies -- it can start earliest

---

## Milestone M2: Event Contracts Hardened + WS3 MVP

**Target**: Weeks 7-10
**Owner**: Platform Engineering Lead (contracts) + WS3 Lead (Culture OS MVP)

### Deliverables

| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M2.1 | Contract testing framework | 7-8 | Pact (or equivalent) deployed. Producer tests for all 10 contracts. Consumer contract tests for all consumers. |
| M2.2 | Kafka load test | 8 | 260 practices emitting events concurrently. Consumer lag < 30 seconds. |
| M2.3 | Dead-letter queue handling | 8-9 | DLQ for each consumer group. Alerts on DLQ depth > 0. |
| M2.4 | Event monitoring dashboard | 9 | Events/sec by topic. Consumer lag by group. Schema validation failures. |
| M2.5 | PMS coverage extension | 8-10 | NexHealth adapters for remaining PMS systems. Sikka ONE batch adapter live for 50 practices. |
| M2.6 | CCA Academy Bronze v1 | 8-9 | 10 modules live on mobile. Scenario assessments. Credly badge issuance. |
| M2.7 | Champion Network v1 | 9-10 | Champion assignment CRUD. `champion-assigned` event publishing. H-Score daily calc workflow in Temporal. |
| M2.8 | Trust Transfer OS v1 | 9-10 | Swell review NLP tagging. Huddle commitment tap. Trust transfer sentiment score visible. |

### Success Metrics
- 10/10 event contracts pass consumer contract tests
- Kafka consumer lag stays < 30 seconds under simulated 260-practice load
- CCA Academy: 80%+ of pilot practice staff enrolled; first Bronze certifications earned
- Champion designation active for >= 10 pilot practices
- `ws3.champion-assigned` event consumed by at least one downstream service

### Gate Criteria (ALL must pass)
1. All 10 event contracts pass Pact consumer tests in CI
2. Kafka load test: 260-practice simulation completes with < 30s consumer lag
3. DLQ monitoring active for all consumer groups
4. PMS Integration covers >= 85% of SGA practices (NexHealth + Sikka ONE combined)
5. CCA Academy Bronze track fully functional with Credly integration
6. WS3 emitting `hscore-calculated`, `champion-assigned`, and `champion-updated` events to Kafka

### Dependencies
- M1 complete and stable
- M1.5 pilot data informing H-Score calibration
- Credly API credentials and organization setup
- Dental Intel API finalized

---

## Milestone M3: WS1 + WS2 Tier 1 Features

**Target**: Weeks 11-14
**Owner**: WS1 Practice Ops Lead + WS1 Network Intel Lead + WS2 Lead

### Deliverables

#### WS1: Practice Operations
| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M3.1 | Practice Nerve Center v1 | 11-12 | Daily agenda (3-5 AI-prioritized action cards). Action card types (callback, confirm, follow-up). PWA. |
| M3.2 | Concierge Agent v1 | 11-12 | Chat-based data verification. Brand voice personality. Missed call text-back (< 90s). Form fill auto-response (< 30s). |
| M3.3 | Data Harvesting | 12-13 | Batch harvest for top 50 practices. Practice profiles pre-populated. |
| M3.4 | Enrollment Invitation | 13 | Email invitations to first cohort. Tracking (sent/opened/clicked/enrolled). |
| M3.5 | Storefront Generation v1 | 13-14 | 2 templates (GP + 1 specialty). Site generation engine. CDN deployment. Practice preview + approval flow. |
| M3.6 | Automation Engine v1 | 13-14 | Appointment confirmation sequences. Compliance rules engine. Playbook cadence definitions. |
| M3.7 | Graduated Autonomy v1 | 14 | Autonomy level management. Weekly summary generator. |

#### WS1: Network Intelligence
| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M3.8 | Funnel Data Lake | 11-12 | ClickHouse schema. Funnel event ingestion from Kafka. Derived analytical views (dbt). Data quality monitoring. |
| M3.9 | Executive Dashboard v1 | 12-13 | Network funnel view. Region drill-down. Practice detail view. Network health score. |
| M3.10 | AI Analysis Engine v1 | 13-14 | Per-practice baselines. Basic anomaly detection. |
| M3.11 | Data Connectors v1 | 12-14 | Phone system connector. Lead source connector. |

#### WS2: Content Engine
| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M3.12 | Content Capture Bot v1 | 11-12 | Daily SMS prompts via Communications Service. MMS capture. Confirmation with streak count. |
| M3.13 | Studio Agent v1 | 12-13 | Email intake (studio@sgadental.com). Practice identification. Brief extraction. Max 3 clarifying questions. |
| M3.14 | Production Pipeline v1 | 13 | Template-based generation (Canva). AI photo processing. AI caption generation. Medium challenge. |
| M3.15 | Approval Workflow v1 | 13-14 | HQ QC queue. CD mobile review. Caption editing. `content-asset-approved` event. |
| M3.16 | Distribution Engine v1 | 14 | Instagram + Facebook publishing. |

#### WS3: Culture OS (continues)
| # | Deliverable | Week | Exit Criteria |
|---|---|---|---|
| M3.17 | Recognition & Rewards v1 | 11-12 | Leaderboard. Basic rewards tracking. |
| M3.18 | Playbook Distribution v1 | 12-13 | Versioned playbook content. Practice-type filtering. |
| M3.19 | Cross-workstream integration | 13-14 | H-Score -> WS1 Executive Dashboard. Champion roster -> WS2 Content Engine. |

### Success Metrics
- 20-30 practices enrolled and active on Practice Nerve Center
- Concierge: < 90s missed call text-back confirmed in production
- 10+ practices generating storefronts via template engine
- Executive Dashboard loads in < 3 seconds with live funnel data
- Content capture active at 10+ practices with daily SMS prompts
- H-Score visible on Executive Dashboard via Kafka event integration
- WS2 publishing at least 1 content piece per pilot practice per week

### Gate Criteria (ALL must pass)
1. Practice Nerve Center serving daily agendas to >= 20 practices
2. Concierge Agent responding to missed calls within 90 seconds (verified by monitoring)
3. Executive Dashboard displaying live data from ClickHouse with < 4-hour lag
4. Content Capture Bot: >= 50% of prompts resulting in a capture within 48 hours (pilot practices)
5. At least 3 of 10 cross-workstream event flows verified in production
6. Zero HIPAA compliance violations
7. All services meet p95 < 500ms latency SLA

### Dependencies
- M2 complete (event contracts hardened, PMS coverage extended)
- Pilot practices identified and onboarded for each workstream
- Google Business Profile API access confirmed (for storefront health monitoring)

---

## Milestone M4: Integrated Platform Launch

**Target**: Weeks 15-18
**Owner**: All team leads + Engineering Manager

### Deliverables

| # | Deliverable | Exit Criteria |
|---|---|---|
| M4.1 | All 10 cross-workstream event flows live in production | Every producer emitting. Every consumer processing. DLQ depth = 0. |
| M4.2 | 50+ practices active on Practice Operations | Nerve Center + Concierge daily usage. Automation sequences running. |
| M4.3 | 50+ practices active on Content Engine | Capture prompts, captures, processing, approval, distribution — full pipeline. |
| M4.4 | 50+ practices active on Culture OS | H-Score, Huddle, CCA Academy, Champion Network all operational. |
| M4.5 | Executive Dashboard unified view | Funnel metrics, H-Score, content health — all visible in one dashboard. |
| M4.6 | Performance + security audit | All services meet latency SLAs. Security audit pass. HIPAA compliance verified by external reviewer. |

### Success Metrics
- 50+ practices active across all 3 workstreams
- All 10 event contracts flowing in production with < 5-minute latency
- Executive Dashboard shows unified view of funnel + culture + content health
- Average H-Score in pilot cohort >= 60 (up from ~51 baseline)
- Lead response time < 2 minutes (p95) for practices on Automation Engine
- Content pipeline producing >= 3 pieces per active practice per week

### Gate Criteria (ALL must pass)
1. >= 50 practices active on at least 2 workstreams simultaneously
2. All 10 cross-workstream events verified flowing in production (monitoring dashboard)
3. Executive Dashboard unified view live with < 5-minute data freshness
4. External HIPAA compliance review passed
5. Load test: system handles 260-practice concurrent load with all services within SLA
6. Incident response runbook documented for all critical services
7. Rollback plan documented and tested for each workstream

### Dependencies
- M3 complete
- Practice rollout pipeline capable of onboarding 10+ practices per week
- HQ operations team trained on Executive Dashboard and QC workflows

---

## Milestone M5: Tier 2 Expansion

**Target**: Weeks 19-24
**Owner**: All team leads

### Deliverables

| Category | Features |
|---|---|
| WS1 Practice Ops | No-show re-engagement. Same-day rebooking. Hygiene recall sequences. Dormant patient reactivation. Treatment follow-up extensions. Change management (staff/hours). Graduation criteria. |
| WS1 Network Intel | AI alerts sidebar + ops queue. Root cause correlation. Action generation with revenue impact. Weekly/monthly summary reports. Warm/cold storage tiers. Communication outcome tracking. |
| WS2 Content Engine | WhatsApp capture. Mobile app with guided overlays. Video processing. Practice branding overlay. Optimal timing scheduler. Storefront gallery routing. Print fulfillment. LinkedIn content. Google Business posting. |
| WS3 Culture OS | H-Score anomaly alerts. Wow story distribution. Champion content library. Trust transfer sentiment score. Missed huddle alerts. |

### Success Metrics
- 150+ practices active across all workstreams
- Automation Engine covering 4+ playbook types (confirmation, follow-up, rebooking, reactivation)
- Lead-to-appointment conversion improved >= 5% vs. pre-platform baseline
- Content pipeline producing >= 5 pieces per active practice per week
- H-Score average in active practices >= 65

### Gate Criteria (Soft gate)
- >= 100 practices active
- At least 3 automation playbooks live
- Tier 2 features deployed with no critical regressions

### Dependencies
- M4 complete and stable for 2+ weeks
- Practice adoption pipeline scaling
- A/B testing framework ready for playbook optimization

---

## Milestone M6: Full Network Rollout

**Target**: Weeks 25-30
**Owner**: Engineering Manager + SGA Operations

### Deliverables

| # | Deliverable | Exit Criteria |
|---|---|---|
| M6.1 | 260+ practices active on Practice Platform | All practices with Nerve Center access. Storefronts deployed. Automation running. |
| M6.2 | 260+ practices active on Content Engine | All eligible practices receiving capture prompts. Content flowing through pipeline. |
| M6.3 | 260+ practices active on Culture OS | H-Score calculating daily. Huddle content available. CCA enrollment open. |
| M6.4 | Self-service onboarding | New practice acquisitions can be enrolled with < 2 hours HQ effort. |
| M6.5 | Operational runbook | Complete operational documentation for SGA IT team. |

### Success Metrics
- 260+ practices active (100% coverage)
- New practice onboarding completed within 2 weeks of acquisition
- Platform uptime >= 99.5% over trailing 30 days
- HQ operations team manages platform with existing headcount (no new hires required)

### Gate Criteria (ALL must pass)
1. >= 250 practices active on at least Practice Platform + one other workstream
2. New practice onboarding demonstrated end-to-end in < 2 weeks
3. Operational handoff complete to SGA IT/Ops team
4. Platform uptime >= 99.5% for trailing 30 days

### Dependencies
- M5 complete
- SGA Operations team trained and assuming daily operations
- All NexHealth + Sikka ONE PMS coverage verified for every practice

---

## Milestone M7: Tier 3 + Optimization

**Target**: Weeks 31-36
**Owner**: All team leads

### Deliverables

| Category | Features |
|---|---|
| WS1 | A/B testing framework. Monthly playbook optimization. HQ Visual Cadence Designer. HQ Template Editor. Weekly learning loop. Offline PWA capability. Ad platform extended ROI attribution. Domain migration at scale. Batch operations. |
| WS2 | Auto-approval for high-confidence content. Content-to-consultation attribution ML. Patient self-record kiosks. Event mode prompts. Network-wide content analytics. |
| WS3 | CCA renewal flow. |
| Platform | Predictive analytics. Voice AI inbound (v2 investigation). |

### Success Metrics
- Treatment acceptance rate improved >= 10% vs. pre-platform baseline
- Show rate improved >= 5% across active practices
- Content -> consultation attribution model producing actionable insights
- H-Score network average >= 70
- AI recommendations improving over time (measured by action-taken rate trend)

### Gate Criteria (Soft gate -- ongoing optimization)
- Core platform stable with 260+ practices
- At least 2 Tier 3 features delivered per workstream
- Voice AI feasibility study complete with go/no-go recommendation

### Dependencies
- M6 complete (full network rollout)
- 6+ months of production data for ML/optimization features
- Anthropic Voice AI capabilities assessment

---

## Milestone Timeline Summary

```
Week:  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25-30 31-36
       |-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|---------|----------|
       | M0  |      M1       |M1.5|    M2        |       M3          |      M4       |   M5   |   M6    | M7
       |Pre- |  Shared Infra  |Part|  Contracts   |  WS1+WS2+WS3     |  Integrated   | Tier 2 |  Full   | Tier 3
       |Build|               |Val |  + WS3 MVP   |  Tier 1 Features  |  Launch       | Expand | Rollout | Optimize
       |     |               |    |              |                   |               |        |         |
Practices:                   20                  50                  50+             150+     260+
                             |                   |                   |               |        |
                         WS3 Pilot         First enrolled      Cross-WS         Scale up  Network
                         H-Score live      practices live     events live       all features coverage
```

---

## Risk-Adjusted Timeline

| Risk | Impact on Timeline | Mitigation |
|---|---|---|
| BAA delays (NexHealth, Anthropic) | M0 slips 1-2 weeks | Start procurement Day 1. Use synthetic data until BAAs close. |
| 10DLC registration delays | M1.6 (Comms Service) slips | Register campaigns in M0. Use Toll-Free as interim. |
| Kafka expertise gap | M1.4, M2.1-M2.4 slower | Use MSK Serverless. Budget 1 week for team training. |
| Pilot practice disengagement | M1.5 under-delivers | Select highly motivated practices. SGA Operations champion assigned per pilot cohort. |
| NexHealth PMS coverage gaps | M2.5 incomplete | Audit all 260 practices' PMS systems in M0. Sikka ONE as fallback for unsupported systems. |
| Cross-workstream integration bugs | M4 delayed | Contract testing in M2 catches issues early. Weekly integration test suite. |

---

*This document defines all milestones for QC-4 review. The Phase 1.5 milestone at Weeks 6-7 is included per Strategist recommendation. All gate criteria are measurable and verifiable. Deferred Voice AI items follow Prototype Lead #1 guidance.*
