# SGA Dental Partners — Consolidated Epic & User Story Inventory

**Date**: 2026-03-14
**Author**: Product Manager Agent (Phase 6)
**Purpose**: Inventory all user stories from 6 original sprints, re-map into consolidated workstreams, apply RICE prioritization, flag deduplicated stories, and group into epics.

---

## Consolidation Summary

| Source Sprint | Original Stories | Consolidated Workstream | Stories After Dedup |
|---|---|---|---|
| Sprint 1: Studio Automation at Scale | 18 stories | WS2: The Content Engine | 14 (4 merged) |
| Sprint 2: Hospitality Playbook Operationalization | 16 stories (5 sub-epics) | WS3: The Culture OS | 16 (standalone) |
| Sprint 3: Digital Storefront at Scale | 19 stories + 4 spikes | WS1: Practice Platform | 15 (4 merged) |
| Sprint 4: Full Funnel Visibility | 34 stories | WS1: Practice Platform | 26 (8 merged) |
| Sprint 5: Patient Comms Automation | 23 stories | WS1: Practice Platform | 17 (6 merged) |
| Sprint 6: Content Concierge | 31 stories | WS2: The Content Engine | 22 (9 merged) |
| **Total** | **141 stories** | **3 workstreams** | **110 consolidated stories** |

**31 stories eliminated through deduplication** (build once, serve many).

---

## Section 1: Deduplicated Stories — Build Once, Serve Many

These stories appeared independently in 2+ sprints. In the consolidated architecture, each is built once as shared infrastructure or a single workstream feature.

| # | Capability | Appeared In | Consolidated To | Impact |
|---|---|---|---|---|
| D-01 | PMS Integration (NexHealth + Sikka ONE) | Sprint 3, 4, 5, 6 | Shared: PMS Integration Service | Eliminated 4 independent connector designs |
| D-02 | Practice Data Hub / Profile | Sprint 3, 4, 5 | Shared: Practice Data Hub | Single canonical record replaces 3 data stores |
| D-03 | Twilio SMS/Voice integration | Sprint 5, 6 | Shared: Communications Service | Unified consent, opt-out, number management |
| D-04 | AI/LLM integration (Claude) | Sprint 1, 3, 4, 5 | Shared: AI Service | Single prompt registry, token accounting, PHI filter |
| D-05 | Executive Dashboard | Sprint 3, 4 | WS1: Network Intelligence | Merged storefront health + funnel dashboard |
| D-06 | Practice Nerve Center / Daily Agenda | Sprint 3, 4 | WS1: Practice Operations | Unified daily tool for office managers |
| D-07 | QC/Approval Queue (HQ) | Sprint 1, 3, 6 | WS2: Content Engine (primary), WS1: Practice Ops (storefront QC) | Single QC workflow with workstream context |
| D-08 | Role-Based Access Control | Sprint 3, 4, 5 | Shared: Auth & Identity Service | One role model across all surfaces |
| D-09 | Content capture SMS prompts | Sprint 1, 6 | WS2: Content Engine | Single capture bot using Communications Service |
| D-10 | Brand asset management | Sprint 1, 3 | Shared: Asset Registry | One repository for all brand and practice assets |
| D-11 | Social media distribution | Sprint 1, 6 | WS2: Content Engine | Single distribution engine |
| D-12 | Compliance/HIPAA layer | Sprint 4, 5 | Shared: Compliance Middleware | Embedded in all services |

---

## Section 2: Consolidated Epics by Workstream

### Shared Infrastructure (Platform Engineering)

All workstreams depend on these. Built in Phase 1 (Weeks 1-4) with hardening in Phase 2 (Weeks 5-8).

#### Epic S-1: Practice Data Hub
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-1.1 | Practice CRUD API with 260-practice seed data | R:260 I:3 C:95% E:2 = **371** | Tier 1 |
| S-1.2 | Staff roster management API | R:260 I:2 C:90% E:1 = **468** | Tier 1 |
| S-1.3 | Region hierarchy and director mapping | R:20 I:2 C:95% E:0.5 = **76** | Tier 1 |
| S-1.4 | Practice configuration and feature flags | R:260 I:2 C:85% E:1 = **442** | Tier 1 |

#### Epic S-2: PMS Integration Service
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-2.1 | NexHealth adapter (Dentrix, Eaglesoft, Open Dental) | R:200 I:3 C:80% E:3 = **160** | Tier 1 |
| S-2.2 | NexHealth webhook receiver + canonical event normalization | R:200 I:3 C:85% E:2 = **255** | Tier 1 |
| S-2.3 | Sikka ONE batch adapter (nightly sync via Temporal) | R:60 I:2 C:75% E:2 = **45** | Tier 2 |
| S-2.4 | NexHealth adapters for remaining PMS systems | R:60 I:2 C:70% E:2 = **42** | Tier 2 |
| S-2.5 | Appointment booking write API (NexHealth) | R:260 I:3 C:75% E:2 = **293** | Tier 1 |

#### Epic S-3: Communications Service
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-3.1 | Twilio SMS send API with consent check | R:260 I:3 C:90% E:2 = **351** | Tier 1 |
| S-3.2 | Consent registry (opt-in/opt-out tracking) | R:260 I:3 C:95% E:1.5 = **494** | Tier 1 |
| S-3.3 | Per-practice number provisioning + 10DLC registration | R:260 I:2 C:80% E:2 = **208** | Tier 1 |
| S-3.4 | Message routing rules (quiet hours, rate limits, dedup) | R:260 I:2 C:90% E:1.5 = **312** | Tier 1 |
| S-3.5 | Email send via AWS SES | R:260 I:1 C:90% E:1 = **234** | Tier 2 |
| S-3.6 | WhatsApp channel (content capture) | R:40 I:1 C:70% E:1.5 = **19** | Tier 3 |
| S-3.7 | Voice AI transport (Twilio Voice + Media Streams) | R:260 I:3 C:60% E:3 = **156** | **Deferred to v2** |

#### Epic S-4: AI Service
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-4.1 | Claude API integration with cost-tier routing | R:260 I:3 C:90% E:2 = **351** | Tier 1 |
| S-4.2 | PHI filter (tokenize/detokenize pipeline) | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| S-4.3 | Prompt registry with versioning | R:260 I:2 C:85% E:1 = **442** | Tier 1 |
| S-4.4 | Token accounting per workstream | R:3 I:1 C:90% E:0.5 = **5** | Tier 2 |
| S-4.5 | OpenAI Whisper integration (audio transcription) | R:260 I:2 C:75% E:1.5 = **260** | Tier 2 |

#### Epic S-5: Event Bus (Kafka)
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-5.1 | MSK cluster + topic provisioning for all 10 event contracts | R:3 I:3 C:90% E:2 = **4** | Tier 1 |
| S-5.2 | Schema Registry with all event schemas registered | R:3 I:3 C:85% E:1.5 = **5** | Tier 1 |
| S-5.3 | Producer/consumer SDK (shared npm package) | R:20 I:2 C:90% E:1 = **36** | Tier 1 |
| S-5.4 | Contract testing framework (Pact) | R:20 I:2 C:80% E:1.5 = **21** | Tier 2 |
| S-5.5 | Dead-letter queue handling + monitoring dashboard | R:20 I:1 C:85% E:1 = **17** | Tier 2 |

#### Epic S-6: Auth & Identity
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-6.1 | Cognito user pool + JWT middleware | R:260 I:3 C:95% E:1.5 = **494** | Tier 1 |
| S-6.2 | RBAC service with role model (8 roles) | R:260 I:2 C:90% E:1.5 = **312** | Tier 1 |
| S-6.3 | API key management (service-to-service) | R:20 I:2 C:90% E:0.5 = **72** | Tier 1 |

#### Epic S-7: Compliance & Audit
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-7.1 | Compliance middleware (Fastify plugin) | R:260 I:3 C:85% E:2 = **332** | Tier 1 |
| S-7.2 | Audit log (append-only, immutable) | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| S-7.3 | Encryption helpers (application-level AES-256-GCM) | R:260 I:3 C:80% E:1 = **624** | Tier 1 |

#### Epic S-8: Asset Registry
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-8.1 | S3 upload + CloudFront CDN distribution | R:260 I:2 C:95% E:1 = **494** | Tier 1 |
| S-8.2 | Metadata catalog (PostgreSQL) | R:260 I:2 C:90% E:1 = **468** | Tier 1 |
| S-8.3 | Cloudinary transforms (on-the-fly resize/crop) | R:260 I:1 C:80% E:1 = **208** | Tier 2 |

#### Epic S-9: Workflow Engine
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-9.1 | Temporal cluster setup (Cloud or self-hosted) | R:3 I:3 C:80% E:1.5 = **5** | Tier 1 |
| S-9.2 | Practice enrollment workflow definition | R:260 I:2 C:75% E:2 = **195** | Tier 2 |

#### Epic S-10: Design Tool Abstraction
| ID | Story | RICE Score | Tier |
|---|---|---|---|
| S-10.1 | Design Provider interface + Canva adapter | R:260 I:2 C:80% E:1.5 = **277** | Tier 2 |

---

### Workstream 1: The Practice Platform

Split into **Practice Operations** (practice-facing) and **Network Intelligence** (executive-facing).

#### WS1 Surface A: Practice Operations

##### Epic WS1-A1: Practice Nerve Center
*Sources: Sprint 3 (Concierge), Sprint 4 (Practice Nerve Center)*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A1.1 | Daily agenda generation (3-5 AI-prioritized action cards) | FOS:PNC-1 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS1-A1.2 | Action card types (callback, confirm, follow-up, review) | FOS:PNC-2 | R:260 I:3 C:85% E:1.5 = **442** | Tier 1 |
| WS1-A1.3 | End-of-day scorecard | FOS:PNC-3 | R:260 I:2 C:80% E:1 = **416** | Tier 2 |
| WS1-A1.4 | Role-based views (Office Manager vs Front Desk) | FOS:PNC-4 | R:260 I:1 C:85% E:1 = **221** | Tier 2 |
| WS1-A1.5 | Push notifications for urgent items | FOS:PNC-6 | R:260 I:2 C:75% E:1 = **390** | Tier 2 |
| WS1-A1.6 | Offline capability (PWA) | FOS:PNC-7 | R:260 I:1 C:60% E:2 = **78** | Tier 3 |

##### Epic WS1-A2: Concierge Agent
*Sources: Sprint 3 (Concierge Welcome, Personality), Sprint 5 (Lead Response)*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A2.1 | Concierge chat — first interaction (data verification) | DS:US-3.1 | R:260 I:3 C:80% E:3 = **208** | Tier 1 |
| WS1-A2.2 | Concierge personality & tone (brand voice) | DS:US-3.2 | R:260 I:2 C:90% E:1 = **468** | Tier 1 |
| WS1-A2.3 | Missed call text-back (< 90 seconds) | CE:LR-2 | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| WS1-A2.4 | Form fill auto-response (< 30 seconds) | CE:LR-3 | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| WS1-A2.5 | Voice AI inbound answering with real-time PMS booking | CE:LR-1 | R:260 I:3 C:60% E:3 = **156** | **Deferred to v2** |
| WS1-A2.6 | Outbound Voice AI (reactivation calls) | CE:TF-3.6 | R:260 I:2 C:50% E:3 = **87** | **Deferred to v2** |

##### Epic WS1-A3: Data Harvesting & Enrollment
*Source: Sprint 3*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A3.1 | Automated practice data harvesting (web + GMB + social) | DS:US-1.1 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS1-A3.2 | Batch harvest for network seeding (260+ practices) | DS:US-1.2 | R:260 I:2 C:80% E:1.5 = **277** | Tier 1 |
| WS1-A3.3 | Personalized enrollment invitation (email) | DS:US-2.1 | R:260 I:2 C:85% E:1 = **442** | Tier 1 |
| WS1-A3.4 | Blitz deploy for non-engaged practices (30-day auto) | DS:US-7.1 | R:100 I:2 C:75% E:2 = **75** | Tier 2 |

##### Epic WS1-A4: Storefront Generation
*Source: Sprint 3*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A4.1 | Specialty-based template recommendation | DS:US-4.1 | R:260 I:3 C:85% E:2 = **332** | Tier 1 |
| WS1-A4.2 | Site generation engine (static/JAMstack) | DS:US-4.2 | R:260 I:3 C:80% E:3 = **208** | Tier 1 |
| WS1-A4.3 | Practice review & approval flow | DS:US-5.1 | R:260 I:2 C:85% E:2 = **221** | Tier 1 |
| WS1-A4.4 | Automated deployment & activation | DS:US-6.1 | R:260 I:2 C:80% E:1.5 = **277** | Tier 1 |
| WS1-A4.5 | Domain migration support | DS:US-6.2 | R:100 I:1 C:70% E:2 = **35** | Tier 3 |

##### Epic WS1-A5: Automation Engine
*Sources: Sprint 4 (Automation Engine), Sprint 5 (Playbook Engine)*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A5.1 | Appointment confirmation sequences (48h/24h/4h) | FOS:AE-1 | R:260 I:3 C:85% E:2 = **332** | Tier 1 |
| WS1-A5.2 | Playbook cadence definitions (configurable multi-touch) | CE:PE-1 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS1-A5.3 | Compliance rules engine (HIPAA, TCPA, quiet hours) | CE:PE-2 | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| WS1-A5.4 | Missed call recovery (< 5 min auto follow-up) | FOS:AE-2 | R:260 I:3 C:85% E:1.5 = **442** | Tier 1 |
| WS1-A5.5 | No-show re-engagement (same-day text) | FOS:AE-3 | R:260 I:2 C:80% E:1.5 = **277** | Tier 2 |
| WS1-A5.6 | Standard 4-touch treatment follow-up cadence | CE:TF-2 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS1-A5.7 | High-value 5-touch extension ($15K+ plans) | CE:TF-3 | R:40 I:3 C:75% E:1.5 = **60** | Tier 2 |
| WS1-A5.8 | Same-day rebooking detection | CE:RR-1 | R:260 I:3 C:85% E:2 = **332** | Tier 2 |
| WS1-A5.9 | Hygiene recall sequences | CE:RR-2 | R:260 I:2 C:80% E:1.5 = **277** | Tier 2 |
| WS1-A5.10 | Dormant patient reactivation (tiered campaigns) | CE:RR-3 | R:260 I:3 C:75% E:2 = **293** | Tier 2 |
| WS1-A5.11 | Treatment plan follow-up sequences | FOS:AE-5 | R:260 I:2 C:70% E:1.5 = **242** | Tier 2 |
| WS1-A5.12 | Production gap alerts (mid-week flags) | FOS:AE-4 | R:260 I:2 C:80% E:1 = **416** | Tier 2 |
| WS1-A5.13 | A/B testing framework for message templates | CE:PE-3 | R:260 I:2 C:65% E:2 = **169** | Tier 3 |

##### Epic WS1-A6: Graduated Autonomy
*Source: Sprint 5*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A6.1 | Autonomy level management (supervised/monitored/autonomous) | CE:GA-1 | R:260 I:2 C:85% E:2 = **221** | Tier 1 |
| WS1-A6.2 | Graduation criteria & tracking | CE:GA-2 | R:260 I:2 C:80% E:1 = **416** | Tier 2 |
| WS1-A6.3 | Weekly summary generator (20-min office manager digest) | CE:GA-3 | R:260 I:2 C:90% E:2 = **234** | Tier 1 |

##### Epic WS1-A7: Change Management
*Source: Sprint 3*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-A7.1 | Staff change via concierge (cascading updates) | DS:US-8.1 | R:260 I:2 C:75% E:2 = **195** | Tier 2 |
| WS1-A7.2 | Hours & service changes propagation | DS:US-8.2 | R:260 I:1 C:80% E:1 = **208** | Tier 2 |

---

#### WS1 Surface B: Network Intelligence

##### Epic WS1-B1: Executive Dashboard
*Sources: Sprint 3 (Network Health), Sprint 4 (Executive Dashboard)*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-B1.1 | Network funnel view (5-stage horizontal bars) | FOS:ED-1 | R:20 I:3 C:85% E:2 = **26** | Tier 1 |
| WS1-B1.2 | Region drill-down view (heatmap) | FOS:ED-2 | R:20 I:3 C:85% E:1.5 = **34** | Tier 1 |
| WS1-B1.3 | Practice detail view (funnel + AI annotations) | FOS:ED-3 | R:20 I:3 C:80% E:2 = **24** | Tier 1 |
| WS1-B1.4 | Network health score (storefront digital health) | DS:US-9.1 | R:20 I:3 C:80% E:2 = **24** | Tier 1 |
| WS1-B1.5 | AI alerts sidebar & ops queue | FOS:ED-4 | R:20 I:3 C:75% E:1.5 = **30** | Tier 2 |
| WS1-B1.6 | Weekly/monthly AI-generated summary reports | FOS:ED-5 | R:20 I:2 C:80% E:1.5 = **21** | Tier 2 |
| WS1-B1.7 | HQ QC queue (brand review workflow) | DS:US-9.2 | R:10 I:2 C:85% E:2 = **9** | Tier 2 |
| WS1-B1.8 | Batch operations (network-wide template/brand updates) | DS:US-9.3 | R:10 I:2 C:75% E:2 = **8** | Tier 3 |

##### Epic WS1-B2: Funnel Data Lake
*Source: Sprint 4*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-B2.1 | Unified funnel event schema | FOS:FDL-1 | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| WS1-B2.2 | ClickHouse hot tier (30-day real-time) | FOS:FDL-2 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS1-B2.3 | dbt-managed derived analytical views | FOS:FDL-4 | R:20 I:3 C:85% E:1.5 = **34** | Tier 1 |
| WS1-B2.4 | Warm/cold storage tiers | FOS:FDL-3 | R:260 I:1 C:75% E:1.5 = **130** | Tier 2 |
| WS1-B2.5 | Data quality monitoring | FOS:FDL-5 | R:260 I:2 C:85% E:1 = **442** | Tier 1 |

##### Epic WS1-B3: AI Analysis Engine
*Source: Sprint 4*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-B3.1 | Per-practice baseline calculation (90-day rolling) | FOS:AI-1 | R:260 I:3 C:75% E:2 = **293** | Tier 1 |
| WS1-B3.2 | Anomaly detection (multi-sensitivity per metric) | FOS:AI-2 | R:260 I:3 C:75% E:2 = **293** | Tier 1 |
| WS1-B3.3 | Root cause correlation (cross-data-source) | FOS:AI-3 | R:260 I:3 C:65% E:2 = **254** | Tier 2 |
| WS1-B3.4 | Action generation with revenue impact estimates | FOS:AI-4 | R:260 I:3 C:70% E:1.5 = **364** | Tier 2 |
| WS1-B3.5 | Weekly learning loop | FOS:AI-5 | R:260 I:2 C:50% E:2 = **130** | Tier 3 |

##### Epic WS1-B4: Data Integration (Connectors)
*Source: Sprint 4*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-B4.1 | Phone system connector (RingCentral, 8x8, Weave) | FOS:DI-2 | R:260 I:2 C:80% E:1.5 = **277** | Tier 1 |
| WS1-B4.2 | Lead source connector (Google Ads, Meta, LSA) | FOS:DI-3 | R:260 I:2 C:80% E:1.5 = **277** | Tier 1 |
| WS1-B4.3 | AI phone agent connector (Sierra, Agent Agnes) | FOS:DI-4 | R:260 I:2 C:70% E:1.5 = **242** | Tier 2 |
| WS1-B4.4 | HR system connector (iSolved) | FOS:DI-5 | R:260 I:1 C:70% E:1 = **182** | Tier 2 |
| WS1-B4.5 | Ad platform connector extended (ROI attribution) | FOS:DI-6 | R:260 I:1 C:70% E:1 = **182** | Tier 3 |

##### Epic WS1-B5: Network Learning
*Source: Sprint 5*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS1-B5.1 | Communication outcome tracking (attribution) | CE:NL-1 | R:260 I:2 C:80% E:1.5 = **277** | Tier 2 |
| WS1-B5.2 | Monthly playbook optimization | CE:NL-2 | R:260 I:2 C:60% E:2 = **156** | Tier 3 |
| WS1-B5.3 | HQ Visual Cadence Designer | CE:HQ-1 | R:10 I:2 C:65% E:2 = **7** | Tier 3 |
| WS1-B5.4 | HQ Template Editor | CE:HQ-2 | R:10 I:1 C:65% E:1.5 = **4** | Tier 3 |

---

### Workstream 2: The Content Engine

*Sources: Sprint 1 (Studio Automation), Sprint 6 (Content Concierge)*

##### Epic WS2-1: Prompt Engine
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-1.1 | Daily SMS capture brief (appointment-matched) | CC:PE-1 | R:260 I:3 C:85% E:1.5 = **442** | Tier 1 |
| WS2-1.2 | Procedure code trigger configuration | CC:PE-2 | R:10 I:2 C:85% E:1 = **17** | Tier 1 |
| WS2-1.3 | Post-treatment testimonial follow-up prompts (30/60/90 day) | CC:PE-3 | R:260 I:2 C:75% E:1 = **390** | Tier 2 |
| WS2-1.4 | SGA brand leadership LinkedIn prompts | CC:PE-4 | R:5 I:2 C:70% E:1 = **7** | Tier 2 |
| WS2-1.5 | Event mode prompts (4Front, Pikos) | CC:PE-5 | R:30 I:1 C:60% E:1 = **18** | Tier 3 |

##### Epic WS2-2: Capture Layer
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-2.1 | SMS/MMS capture (reply to prompt with photo/video) | CC:CL-1 | R:260 I:3 C:90% E:1.5 = **468** | Tier 1 |
| WS2-2.2 | WhatsApp capture channel | CC:CL-2 | R:40 I:1 C:70% E:1.5 = **19** | Tier 2 |
| WS2-2.3 | Immediate capture confirmation with streak/count | CC:CL-3 + CL-4 | R:260 I:1 C:90% E:0.5 = **468** | Tier 1 |
| WS2-2.4 | Mobile app with guided camera overlays | CC:CL-5 | R:260 I:2 C:65% E:3 = **113** | Tier 2 |
| WS2-2.5 | Patient self-record testimonial kiosk | CC:CL-6 | R:50 I:2 C:50% E:2 = **25** | Tier 3 |
| WS2-2.6 | Passive operatory cameras | CC:CL-7 | R:50 I:1 C:30% E:3 = **5** | Tier 3 |

##### Epic WS2-3: Studio Agent (Conversational Intake)
*Source: Sprint 1*

| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-3.1 | Email intake (studio@sgadental.com) | SA:1.1 | R:160 I:3 C:85% E:2 = **204** | Tier 1 |
| WS2-3.2 | Teams intake (DM/channel) | SA:1.2 | R:160 I:2 C:80% E:1.5 = **171** | Tier 1 |
| WS2-3.3 | Practice identification from sender | SA:1.3 | R:160 I:2 C:90% E:0.5 = **576** | Tier 1 |
| WS2-3.4 | Maximum 3 clarifying questions | SA:1.4 | R:160 I:1 C:90% E:0.5 = **288** | Tier 1 |

##### Epic WS2-4: Production Pipeline
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-4.1 | Template-based generation (Canva API) | SA:2.1 | R:160 I:3 C:80% E:2 = **192** | Tier 1 |
| WS2-4.2 | AI photo/video processing (Cloudinary + Claude captions) | CC:PP-3 + PP-4 | R:260 I:3 C:75% E:2 = **293** | Tier 1 |
| WS2-4.3 | Practice preview delivery (same channel) | SA:2.2 | R:160 I:2 C:85% E:1 = **272** | Tier 1 |
| WS2-4.4 | Text-only revision (< 2 min) | SA:2.3 | R:160 I:2 C:80% E:1 = **256** | Tier 1 |
| WS2-4.5 | Dental-specific color correction + smart crop | CC:PP-5 | R:260 I:1 C:70% E:1.5 = **121** | Tier 2 |
| WS2-4.6 | Video auto-caption + highlight clip generation | CC:PP-6 + PP-7 | R:100 I:2 C:60% E:2 = **60** | Tier 2 |
| WS2-4.7 | Practice branding overlay (logo, colors) | CC:PP-8 | R:260 I:2 C:80% E:1 = **416** | Tier 2 |
| WS2-4.8 | Medium challenge (suggest digital alternatives to print) | SA:3.1 | R:160 I:2 C:85% E:0.5 = **544** | Tier 1 |

##### Epic WS2-5: Review & Approval
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-5.1 | HQ QC queue (kanban) with approve/reject/feedback | CC:RV-3 + RV-2 | R:10 I:3 C:85% E:2 = **13** | Tier 1 |
| WS2-5.2 | CD mobile review (push notification + 1-tap approve) | SA:2.4 | R:10 I:3 C:85% E:1.5 = **17** | Tier 1 |
| WS2-5.3 | AI caption edit before approval | CC:RV-4 | R:10 I:2 C:85% E:0.5 = **34** | Tier 1 |
| WS2-5.4 | Star high-quality content for premium treatment | CC:RV-5 | R:10 I:1 C:85% E:0.5 = **17** | Tier 2 |
| WS2-5.5 | Auto-approval for >95% approval rate content types | CC:PP-9 | R:260 I:2 C:60% E:1.5 = **208** | Tier 3 |

##### Epic WS2-6: Distribution Engine
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-6.1 | Auto-distribute to Instagram + Facebook (Meta Graph API) | CC:DI-2 | R:260 I:3 C:75% E:2 = **293** | Tier 1 |
| WS2-6.2 | Optimal timing scheduler (AI-determined) | CC:DI-3 | R:260 I:1 C:65% E:1 = **169** | Tier 2 |
| WS2-6.3 | Route before/afters to storefront gallery | CC:DI-4 | R:260 I:2 C:75% E:1 = **390** | Tier 2 |
| WS2-6.4 | Tag paid-media-eligible content for library | CC:DI-5 | R:260 I:1 C:80% E:0.5 = **416** | Tier 2 |
| WS2-6.5 | LinkedIn leadership content scheduling | CC:DI-6 | R:5 I:2 C:70% E:1 = **7** | Tier 2 |
| WS2-6.6 | Print fulfillment (Gelato API one-click order) | SA:5.1 + 5.2 | R:160 I:2 C:70% E:1.5 = **149** | Tier 2 |
| WS2-6.7 | Patient email deployment (Weave integration) | SA:3.2 | R:160 I:2 C:65% E:1.5 = **139** | Tier 2 |
| WS2-6.8 | TV slide asset delivery | SA:3.3 | R:160 I:1 C:80% E:0.5 = **256** | Tier 2 |
| WS2-6.9 | Google Business Profile posting | New | R:260 I:2 C:70% E:1 = **364** | Tier 2 |

##### Epic WS2-7: Scorecard & Analytics
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-7.1 | Champion weekly SMS recap | CC:SC-2 | R:260 I:1 C:85% E:0.5 = **442** | Tier 1 |
| WS2-7.2 | Practice content health score (HQ dashboard) | CC:SC-4 | R:10 I:2 C:80% E:1.5 = **11** | Tier 2 |
| WS2-7.3 | Network-wide content analytics tied to outcomes | CC:SC-5 | R:10 I:2 C:60% E:2 = **6** | Tier 3 |
| WS2-7.4 | Content-to-consultation attribution | CC:SC-6 | R:260 I:3 C:40% E:2 = **156** | Tier 3 |
| WS2-7.5 | Studio analytics dashboard (turnaround, volume, escalation) | SA:4.1-4.3 | R:10 I:2 C:80% E:1.5 = **11** | Tier 2 |

##### Epic WS2-8: Human Escalation
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS2-8.1 | Human designer escalation queue | SA:2.5 | R:160 I:2 C:85% E:1 = **272** | Tier 1 |

---

### Workstream 3: The Culture OS

*Source: Sprint 2 (Hospitality Playbook Operationalization)*

##### Epic WS3-1: H-Score Engine
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS3-1.1 | Cinnamon API integration (phone score) | HPP-3-003 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS3-1.2 | Swell API integration (review score + NLP) | HPP-3-004 | R:260 I:3 C:80% E:2 = **312** | Tier 1 |
| WS3-1.3 | Composite H-Score calculation (daily batch) | HPP-3-001 + HPP-3-002 | R:260 I:3 C:85% E:1.5 = **442** | Tier 1 |
| WS3-1.4 | Practice dashboard (score + components + trend) | HPP-3-002 | R:260 I:2 C:85% E:1.5 = **295** | Tier 1 |
| WS3-1.5 | H-Score anomaly detection (10+ point drop alert) | HPP-3-005 | R:260 I:2 C:75% E:1 = **390** | Tier 2 |

##### Epic WS3-2: Morning Huddle OS
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS3-2.1 | Dental Intel integration + content package auto-load | HPP-1-001 | R:260 I:3 C:75% E:2 = **293** | Tier 1 |
| WS3-2.2 | Yesterday's Wins (Swell review display + trust transfer badge) | HPP-1-002 | R:260 I:2 C:80% E:1 = **416** | Tier 1 |
| WS3-2.3 | Today's Focus (lowest H-Score sub-score targeting) | HPP-1-003 | R:260 I:2 C:80% E:1 = **416** | Tier 1 |
| WS3-2.4 | Huddle completion tracking + streak indicator | HPP-1-004 | R:260 I:2 C:90% E:0.5 = **936** | Tier 1 |
| WS3-2.5 | Missed huddle alerts (3+ consecutive) | HPP-1-005 | R:20 I:2 C:85% E:0.5 = **68** | Tier 2 |
| WS3-2.6 | Wow story submission from post-huddle screen | HPP-1-006 | R:260 I:1 C:80% E:1 = **208** | Tier 2 |

##### Epic WS3-3: CCA Academy (Bronze)
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS3-3.1 | 10-module Bronze track (mobile-first video + scenarios) | HPP-2-001 + HPP-2-002 | R:2000 I:3 C:80% E:3 = **1600** | Tier 1 |
| WS3-3.2 | Certificate generation + Credly badge issuance | HPP-2-003 + HPP-2-004 | R:2000 I:2 C:75% E:1.5 = **2000** | Tier 1 |
| WS3-3.3 | Renewal flow (30-day window, 3 refresher modules) | HPP-2-005 | R:2000 I:1 C:75% E:1 = **1500** | Tier 3 |

##### Epic WS3-4: Trust Transfer OS
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS3-4.1 | Trust Transfer commitment tap in huddle | HPP-4-001 | R:260 I:2 C:90% E:0.5 = **936** | Tier 1 |
| WS3-4.2 | Swell review NLP tagging (trust transfer signals) | HPP-4-002 | R:260 I:2 C:70% E:1.5 = **242** | Tier 1 |
| WS3-4.3 | Trust transfer sentiment score (30-day rolling) | HPP-4-003 | R:260 I:1 C:70% E:1 = **182** | Tier 2 |

##### Epic WS3-5: Champion Network
| ID | Story | Original | RICE | Tier |
|---|---|---|---|---|
| WS3-5.1 | Champion readiness algorithm + designation workflow | HPP-5-001 + HPP-5-002 | R:260 I:2 C:75% E:1.5 = **260** | Tier 1 |
| WS3-5.2 | Champion content library + regional cohort | HPP-5-002 | R:260 I:1 C:75% E:1 = **195** | Tier 2 |
| WS3-5.3 | Wow story distribution to regional huddles | HPP-5-003 | R:260 I:1 C:70% E:1 = **182** | Tier 2 |

---

## Section 3: Tier Summary

### Interface Tier Mapping (per Prototype Lead #2 carry-forward)

| Tier | Interfaces | Stories | Target |
|---|---|---|---|
| **Tier 1 (Launch)** | 16 interfaces | 67 stories | MVP — Week 12 exit |
| **Tier 2 (Expansion)** | 14 interfaces | 31 stories | Weeks 13-18 |
| **Tier 3 (Full Platform)** | 12 interfaces | 12 stories | Weeks 19-24+ |

### Story Count by Priority and Workstream

| Workstream | Tier 1 | Tier 2 | Tier 3 | Deferred v2 | Total |
|---|---|---|---|---|---|
| Shared Infrastructure | 27 | 10 | 0 | 1 | 38 |
| WS1: Practice Operations | 18 | 14 | 3 | 2 | 37 |
| WS1: Network Intelligence | 10 | 9 | 5 | 0 | 24 |
| WS2: Content Engine | 18 | 16 | 6 | 0 | 40 |
| WS3: Culture OS | 13 | 7 | 1 | 0 | 21 |
| **Total** | **86** | **56** | **15** | **3** | **160** |

Note: Story count includes shared infrastructure stories which support all workstreams. The 110 workstream-specific stories + 38 shared infrastructure + 12 cross-cutting = 160 total discrete build items.

### Deferred to v2

Per carry-forward from Prototype Lead #1:

| Story | Reason |
|---|---|
| S-3.7: Voice AI transport | Real-time PMS booking via Voice AI requires proven NexHealth write-path stability |
| WS1-A2.5: Voice AI inbound answering | Same dependency; build text-first, prove, then layer voice |
| WS1-A2.6: Outbound Voice AI calls | Requires inbound Voice AI to be proven first |

---

*This document provides the complete user story inventory for QC-4 review. All stories are traceable to their original sprint source. RICE scores use: Reach (number of users/practices affected), Impact (1-3 scale), Confidence (percentage), Effort (person-months). Deferred items follow Prototype Lead #1 guidance.*
