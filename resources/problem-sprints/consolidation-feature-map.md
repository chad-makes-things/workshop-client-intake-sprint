# SGA Dental Partners — Consolidation Feature Map

**Date**: 2026-03-14
**Author**: Product Strategist Agent
**Purpose**: Map every capability from 6 original sprints into 3 consolidated workstreams, resolve technical conflicts, define integration points, and assess product-market fit.

---

## Consolidation Summary

| Original Sprint | Consolidated Workstream |
|---|---|
| Sprint 1: Studio Automation at Scale | **Workstream 2: The Content Engine** |
| Sprint 2: Hospitality Playbook Platform | **Workstream 3: The Culture OS** |
| Sprint 3: The Practice Concierge (Digital Storefront) | **Workstream 1: The Practice Platform** |
| Sprint 4: The Funnel OS | **Workstream 1: The Practice Platform** |
| Sprint 5: The Comms Engine | **Workstream 1: The Practice Platform** |
| Sprint 6: The Content Concierge | **Workstream 2: The Content Engine** |

---

## Section 1: Complete Feature Map

### Workstream 1: The Practice Platform
*Consolidates: Practice Concierge (Sprint 3) + Funnel OS (Sprint 4) + Comms Engine (Sprint 5)*

| # | Feature / Capability | Original Sprint | Standalone or Merge | Merges With | Merge Rationale / Notes |
|---|---|---|---|---|---|
| **1.01** | Practice Data Hub (single source of truth per practice) | Sprint 3 | **Merge** | Funnel Data Lake (Sprint 4), Practice Agent Instance context (Sprint 5) | The Practice Data Hub becomes the canonical record for all three systems. The Funnel OS data lake adds funnel-stage event data as a time-series layer on top of the practice record. The Comms Engine's per-practice context (provider roster, specialties, schedule patterns) merges into the same hub rather than maintaining a separate copy. This eliminates the data duplication problem that would have plagued three independent systems. |
| **1.02** | Silent public data harvesting (Playwright + Claude) | Sprint 3 | Standalone | — | Remains as the enrollment and data enrichment engine for the Practice Data Hub. No overlap with other sprints. |
| **1.03** | Practice enrollment pipeline (<2 weeks) | Sprint 3 | Standalone | — | Enrollment workflow stays intact. Now enrolls practices into the unified platform rather than just the digital storefront. |
| **1.04** | 5 specialty website templates (GP, Perio, Full Arch, Pediatric, Multi) | Sprint 3 | Standalone | — | Website generation remains a distinct capability within the platform. Templates now benefit from Funnel OS conversion data to optimize layouts. |
| **1.05** | Concierge Agent per practice (conversational AI) | Sprint 3 | **Merge** | Practice Agent Instance (Sprint 5) | This is the most significant merge in the consolidation. Sprint 3's Concierge Agent (enrollment, change management, Q&A, proactive outreach) and Sprint 5's Practice Agent Instance (lead response, treatment follow-up, rebooking, reactivation) become a single unified agent per practice. The Concierge Agent gains communication automation capabilities; the Comms Engine gains the conversational enrollment and change management flows. The merged agent covers the full practice lifecycle: enrollment, digital presence management, patient communications, and funnel optimization. |
| **1.06** | Channel sync engine (GMB, Meta, directories, CRM, review platforms) | Sprint 3 | Standalone | — | Digital presence management stays as a module within the platform. Feeds health score data to the dashboard layer. |
| **1.07** | Blitz deploy safety net (auto-deploy after 30 days non-engagement) | Sprint 3 | Standalone | — | Remains as an enrollment accelerator. No overlap. |
| **1.08** | Change management (one change cascades everywhere) | Sprint 3 | Standalone | — | Core architecture pattern. Now cascading changes also update Comms Engine templates and Funnel OS practice profiles. |
| **1.09** | Network health dashboard with automated scoring | Sprint 3 | **Merge** | Executive Dashboard (Sprint 4), HQ Ops queue (Sprint 4) | Sprint 3's network health dashboard (digital presence scoring) merges with Sprint 4's executive dashboard (funnel metrics). The unified dashboard shows both digital health and funnel health in a single view. Two-click drill-down from network to region to practice. |
| **1.10** | QC queues and batch operations | Sprint 3 | Standalone | — | Batch operations for website and digital presence management. Now extended to cover batch comms operations. |
| **1.11** | Opinionated agent with reasoning | Sprint 3 | **Merge** | AI Analysis Engine recommendations (Sprint 4) | The agent's opinionated stance ("perio templates convert 23% better") now draws from Funnel OS analytics data, making recommendations evidence-backed rather than heuristic-based. |
| **1.12** | 5-stage patient funnel model | Sprint 4 | Standalone | — | The funnel model (Leads Generated, Connected, Booked, Shows Up, Production) becomes the analytical backbone of the Practice Platform. |
| **1.13** | 15-platform integration middleware (Airbyte + Python connectors) | Sprint 4 | **Merge** | NexHealth Synchronizer (Sprint 5), Channel sync engine (Sprint 3) | All external system connectors consolidate into a single Integration Middleware layer. Airbyte handles bulk data sync. NexHealth provides the universal PMS API. Channel sync handles digital presence APIs. One connector framework, one credential vault, one monitoring system. |
| **1.14** | Funnel Data Lake (ClickHouse + PostgreSQL + S3) | Sprint 4 | **Merge** | Practice Data Hub (Sprint 3) | ClickHouse remains the hot-tier analytics engine for funnel events. PostgreSQL is shared as the operational database. S3 handles cold storage. The data lake sits alongside the Practice Data Hub — operational data in PostgreSQL, analytical data in ClickHouse, both keyed to the same practice_id. |
| **1.15** | AI Analysis Engine (anomaly detection, root cause correlation, baselines) | Sprint 4 | **Merge** | Network Learning Engine (Sprint 5) | The Funnel OS AI engine (anomaly detection, root cause correlation, per-practice baselines) merges with the Comms Engine's Network Learning Engine (cross-practice communication optimization). The combined engine analyzes both funnel performance AND communication effectiveness, enabling root cause analysis that spans the full pipeline: "Practice X's show rate dropped because the confirmation sequence changed AND a front desk staff member left." |
| **1.16** | Automation Engine (confirmation sequences, missed call recovery, no-show prevention) | Sprint 4 | **Merge** | Playbook Engine (Sprint 5), 4 automation stages (Sprint 5) | Sprint 4's Automation Engine defined WHAT automations to run (confirmations, missed calls, no-shows). Sprint 5's Playbook Engine defined HOW to run them (cadences, practice-type variants, compliance). They merge into a single Automation + Playbook layer that handles all patient communication automation with sophisticated cadence logic. |
| **1.17** | Executive Dashboard (network-wide, region, practice, AI alerts, <3s load) | Sprint 4 | **Merge** | Network health dashboard (Sprint 3) | See 1.09 above. The executive dashboard becomes the unified command center. |
| **1.18** | Practice Nerve Center (daily 20-min agenda, 3 action cards, end-of-day scorecard) | Sprint 4 | **Merge** | Weekly Summary Generator (Sprint 5) | The Practice Nerve Center (Sprint 4's daily agenda for practice managers) absorbs Sprint 5's Weekly Summary Generator. Practice managers get a unified daily view covering both funnel actions and communication outcomes, plus a weekly rollup. The "20-minute daily" model scales down to practice size. |
| **1.19** | HQ Ops queue (14 practices ranked by revenue impact, AI summaries) | Sprint 4 | Standalone | — | HQ-facing prioritization tool. Now includes both funnel and comms data for ranking. |
| **1.20** | Service-type-aware automation (cleaning vs. implant vs. full-arch) | Sprint 4 | **Merge** | Practice-type variants (Sprint 5) | Both sprints independently designed for service/practice type differentiation. They merge into a single taxonomy: practice type (GP, Perio, Cosmetic, Pediatric) x service type (preventive, restorative, implant, full-arch, cosmetic). This taxonomy drives template selection, cadence timing, message tone, and funnel benchmarks. |
| **1.21** | Learning loop (weekly refinement) | Sprint 4 | **Merge** | Network Learning Engine (Sprint 5) | See 1.15 above. Single learning loop covers both funnel and comms optimization. |
| **1.22** | Practice Agent Instance (one per practice, extends Concierge) | Sprint 5 | **Merge** | Concierge Agent (Sprint 3) | See 1.05 above. |
| **1.23** | Playbook Engine (cadence execution, practice-type variants, HIPAA compliance) | Sprint 5 | **Merge** | Automation Engine (Sprint 4) | See 1.16 above. |
| **1.24** | Message Generator (LLM personalization, warm tone, PHI validation) | Sprint 5 | Standalone | — | Core capability of the merged comms layer. PHI validation is critical for HIPAA compliance. |
| **1.25** | Voice AI Agent (natural voice, real-time PMS lookup, direct booking) | Sprint 5 | Standalone | — | Unique capability. No overlap with other sprints. Becomes a channel option within the unified agent. |
| **1.26** | Network Learning Engine (cross-practice optimization, monthly cycles) | Sprint 5 | **Merge** | AI Analysis Engine (Sprint 4), Learning loop (Sprint 4) | See 1.15 above. |
| **1.27** | Graduation Manager (Supervised, Monitored, Autonomous per communication type) | Sprint 5 | Standalone | — | Trust management system for autonomous comms. No overlap. Applies to the merged agent's communication capabilities. |
| **1.28** | NexHealth Synchronizer (universal PMS API, 20+ systems) | Sprint 5 | **Merge** | Integration middleware (Sprint 4), PMS triggers (Sprint 1 Phase 3) | See 1.13 above. NexHealth becomes the primary PMS connector within the unified integration middleware. |
| **1.29** | Channel Router (SMS, voice, email selection based on demographics and history) | Sprint 5 | Standalone | — | Unique intelligence layer for communication channel selection. |
| **1.30** | Escalation Manager (routes to human when confidence is low) | Sprint 5 | Standalone | — | Critical safety system for autonomous operations. |
| **1.31** | Compliance Auditor (HIPAA audit trail, PHI monitoring, opt-out compliance) | Sprint 5 | Standalone | — | Non-negotiable compliance infrastructure. |
| **1.32** | HQ Playbook Manager (web interface for designing and deploying playbook updates) | Sprint 5 | Standalone | — | Admin tool for managing communication playbooks network-wide. |

---

### Workstream 2: The Content Engine
*Consolidates: Studio Automation (Sprint 1) + Content Concierge (Sprint 6)*

| # | Feature / Capability | Original Sprint | Standalone or Merge | Merges With | Merge Rationale / Notes |
|---|---|---|---|---|---|
| **2.01** | Email intake at studio@sgadental.com with AI brief extraction | Sprint 1 | Standalone | — | Request-driven intake for designed collateral. Distinct from content capture (Sprint 6). |
| **2.02** | Multi-channel intake adapters (Teams Bot, Slack Bot, Email) | Sprint 1 | Standalone | — | Studio request channels. Not overlapping with Sprint 6's SMS/WhatsApp capture. Different intent: requesting designed assets vs. submitting raw content. |
| **2.03** | Studio Agent Core (Claude API — intent classification, conversational intake, brief assembly) | Sprint 1 | **Merge** | AI caption generation (Sprint 6) | Both systems use Claude API for content intelligence. The Studio Agent Core expands to handle two request types: (1) design collateral requests (original Sprint 1 flow) and (2) content processing requests (Sprint 6 captured media needing AI enhancement). Shared LLM infrastructure, shared brand context, shared practice profiles. |
| **2.04** | Medium challenge (conversational nudge from print to digital) | Sprint 1 | Standalone | — | Behavior change logic unique to the Studio workflow. No overlap. |
| **2.05** | Canva Connect API template generation (12 templates Phase 1, 30+ Phase 2) | Sprint 1 | **Merge** | Content templates/playbooks (Sprint 6) | Sprint 1's Canva templates (marketing collateral: flyers, social posts, TV slides) and Sprint 6's content templates (social post formats per platform, before/after layouts) merge into a unified template library. Canva remains the design engine, but templates now serve both request-driven collateral AND content-pipeline social posts. |
| **2.06** | Practice approval via email reply | Sprint 1 | **Merge** | Human QC queue (Sprint 6) | Both sprints have approval workflows. Sprint 1: practice manager approves designed collateral. Sprint 6: HQ reviews AI-processed content before distribution. These merge into a unified approval system with two tracks: practice-facing approvals (collateral) and HQ-facing approvals (content pipeline output). |
| **2.07** | CD review via Teams Adaptive Card (58-sec mobile tap-approve) | Sprint 1 | Standalone | — | Creative Director review workflow. Specific to brand-controlled collateral. |
| **2.08** | Digital deployment: Weave API (email campaigns, TV slides) | Sprint 1 | **Merge** | Auto-distribution to 6 channels (Sprint 6) | Sprint 1 deploys to Weave (patient email, TV). Sprint 6 deploys to social platforms (Instagram, Facebook, YouTube, LinkedIn, TikTok, Google Business Profile). They merge into a unified Distribution Engine covering all output channels: patient-facing (Weave), social (platform APIs), print (Gelato), and digital storefront (Practice Platform integration). |
| **2.09** | Print fulfillment: Gelato API | Sprint 1 | Standalone | — | Physical fulfillment. No overlap with Sprint 6. |
| **2.10** | Brand health scoring (0-100 per practice) | Sprint 1 | **Merge** | Champion scorecard with streak counter (Sprint 6) | Brand health (Sprint 1: are practices on-brand?) and content health (Sprint 6: are practices capturing content?) merge into a unified Content & Brand Health Score per practice. Sub-scores: brand compliance, content capture frequency, content quality, distribution coverage. |
| **2.11** | Analytics dashboard (request count, auto-fulfill rate, turnaround, print volume) | Sprint 1 | **Merge** | HQ dashboard (Sprint 6 Phase 2) | The Studio analytics dashboard and the Content Concierge HQ dashboard merge into a single Content Engine Dashboard. Metrics span the full pipeline: requests received, content captured, assets produced, assets distributed, engagement metrics. |
| **2.12** | 3 brand universes (SGA legacy, Gen4, MODIS — separate Canva workspaces) | Sprint 1 | Standalone | — | Brand governance structure. Content Engine must respect all three brand universes when generating assets. |
| **2.13** | Human designer queue (custom / edge-case routing) | Sprint 1 | Standalone | — | Escalation path for requests that exceed template capabilities. |
| **2.14** | SMS/WhatsApp capture bot with daily procedure-matched briefs | Sprint 6 | Standalone | — | The zero-friction content capture interface. Unique to Sprint 6. |
| **2.15** | Nightly appointment sync via Sikka ONE API (400+ PMS systems) | Sprint 6 | **Merge** | PMS triggers (Sprint 1 Phase 3), NexHealth Synchronizer (Sprint 5 via Practice Platform) | Sprint 6 uses Sikka ONE for appointment data to drive content capture prompts. Sprint 1 Phase 3 planned PMS triggers for automated collateral. Sprint 5 uses NexHealth. Resolution: The Content Engine uses the Practice Platform's unified PMS integration layer (see Shared Infrastructure) rather than maintaining its own Sikka ONE connection. Capture prompts are driven by appointment events from the shared event bus. |
| **2.16** | AI photo processing (color correction, smart crop, before/after pairing via Cloudinary) | Sprint 6 | Standalone | — | Unique media processing capability. No overlap. |
| **2.17** | AI video processing (stabilization, Whisper transcription, OpusClip highlights) | Sprint 6 | Standalone | — | Unique media processing capability. No overlap. |
| **2.18** | AI caption generation (Claude API, platform-specific for 6 channels) | Sprint 6 | **Merge** | Studio Agent Core (Sprint 1) | See 2.03 above. |
| **2.19** | 5 practice-type playbooks (Implant/Cosmetic, Specialty, Growth GP, Standard GP, SGA Brand) | Sprint 6 | Standalone | — | Content strategy playbooks that define capture targets and content types per practice profile. |
| **2.20** | Champion scorecard with streak counter | Sprint 6 | **Merge** | Brand health scoring (Sprint 1) | See 2.10 above. |
| **2.21** | Airtable editorial queue (Phase 1) → Next.js HQ dashboard (Phase 2) | Sprint 6 | **Merge** | Analytics dashboard (Sprint 1) | See 2.11 above. |
| **2.22** | React Native iOS/Android app (Phase 2) | Sprint 6 | Standalone | — | Mobile content capture app. No overlap. |
| **2.23** | Auto-distribution to Instagram, Facebook, YouTube, LinkedIn, TikTok, Google Business Profile | Sprint 6 | **Merge** | Digital deployment (Sprint 1) | See 2.08 above. |
| **2.24** | Testimonial kiosks + Insta360 cameras (Phase 3) | Sprint 6 | Standalone | — | Hardware-based passive capture. No overlap. |
| **2.25** | ML booking attribution (Phase 4) | Sprint 6 | **Merge** | AI Analysis Engine (Sprint 4 via Practice Platform) | Content-to-booking attribution analysis uses the Practice Platform's funnel data. The Content Engine sends content distribution events; the Practice Platform correlates with booking events. Attribution logic lives in the shared intelligence layer. |
| **2.26** | Content Production Pipeline (batch processing, asset management) | Sprint 6 | Standalone | — | The automated pipeline from raw capture to finished asset. |
| **2.27** | Event Mode (4Front, Pikos courses, partner signings) | Sprint 6 | Standalone | — | Special capture mode for events. No overlap. |

---

### Workstream 3: The Culture OS
*Contains: Hospitality Playbook Platform (Sprint 2) — standalone with integration points*

| # | Feature / Capability | Original Sprint | Standalone or Merge | Merges With | Merge Rationale / Notes |
|---|---|---|---|---|---|
| **3.01** | Morning Huddle OS (5-minute daily ritual via Dental Intel) | Sprint 2 | Standalone | — | Core culture delivery mechanism. Distinct from any other sprint. |
| **3.02** | CCA Academy (Bronze/Silver/Gold with Credly + LinkedIn credentials) | Sprint 2 | Standalone | — | Certification and training platform. No overlap. |
| **3.03** | H-Score Engine (Cinnamon 35% + Swell 45% + Dental Intel 20%) | Sprint 2 | Standalone | — | Composite hospitality score. Feeds data to Practice Platform dashboard. |
| **3.04** | Trust Transfer NLP verification | Sprint 2 | Standalone | — | Cinnamon call analysis for verifying trust transfer behaviors. Unique. |
| **3.05** | Champion Network (260 champions, regional cohorts) | Sprint 2 | **Integration point** | Content champions (Sprint 6) | The Champion Network for culture overlaps with content champions from Sprint 6. A single champion per practice may serve both roles. Integration: the Culture OS champion roster syncs with the Content Engine champion assignments, enabling a unified champion experience and combined scorecards. |
| **3.06** | Recognition & Rewards Engine (leaderboard, bonus pool) | Sprint 2 | Standalone | — | Gamification and rewards system. Could extend to content capture rewards in future. |
| **3.07** | Playbook Distribution System (versioned) | Sprint 2 | Standalone | — | Versioned playbook content delivery. Distinct from patient communication playbooks (Sprint 5). Culture playbooks teach behaviors; comms playbooks execute automated messaging. |
| **3.08** | Content Production Pipeline (Grant batch recording via Mux) | Sprint 2 | **Integration point** | Content Production Pipeline (Sprint 6) | Sprint 2 produces training videos (Grant recordings). Sprint 6 produces marketing content. Different content, different purpose, but shared infrastructure opportunity: both need video hosting, transcription, and distribution. Integration: Culture OS can use Content Engine's video processing pipeline (Whisper, OpusClip) for training video production while maintaining its own content library and taxonomy. |
| **3.09** | 12 core behaviors defined and measured | Sprint 2 | Standalone | — | Foundational behavioral framework. No technical overlap. |
| **3.10** | Network Intelligence Dashboard (regional director portfolio view) | Sprint 2 | **Integration point** | Executive Dashboard (Sprint 4 via Practice Platform) | Culture metrics can surface in the Practice Platform's executive dashboard as a tab or section rather than requiring a separate dashboard. Integration: H-Score data feeds into the Practice Platform dashboard; the Culture OS maintains its own detailed views for culture-specific drill-downs. |
| **3.11** | Practice-type segmented benchmarking (rural, suburban, urban, cosmetic, pediatric) | Sprint 2 | **Integration point** | Service-type-aware taxonomy (Sprints 4+5 via Practice Platform) | The Culture OS uses the same practice-type taxonomy as the Practice Platform, enabling consistent segmentation across funnel, comms, and culture metrics. |
| **3.12** | Playbook Content Engine (CMS with behavior taxonomy) | Sprint 2 | Standalone | — | Content management for hospitality training materials. Distinct from marketing content. |
| **3.13** | Trust Transfer OS (tracking handoff behaviors between team members) | Sprint 2 | Standalone | — | Behavioral measurement system unique to hospitality. |

---

## Section 2: Shared Infrastructure Resolution

### Conflict 1: PMS Integration Approach

| Dimension | Sprint 4 (Funnel OS) | Sprint 5 (Comms Engine) | Sprint 6 (Content Concierge) | Sprint 1 (Studio Auto, Phase 3) |
|---|---|---|---|---|
| **Approach** | Direct connectors (Dentrix, Eaglesoft, Open Dental) via Airbyte | NexHealth Synchronizer (universal API, 20+ systems) | Sikka ONE API (400+ PMS systems) | Planned PMS triggers (Dentrix, Eaglesoft, Curve) |
| **Coverage** | 3 specific PMS systems | 20+ systems | 400+ systems | 3 specific PMS systems |
| **Data direction** | Read-only (funnel events) | Read-write (appointment booking) | Read-only (appointment schedules) | Read-only (procedure triggers) |

**Resolution: Dual-layer PMS integration**
- **Primary (real-time)**: NexHealth Synchronizer for all read-write operations (appointment booking, real-time event streams). NexHealth covers 20+ systems representing the majority of SGA practices and offers the write API needed for the Comms Engine's Voice AI to book directly.
- **Secondary (batch/coverage)**: Sikka ONE for read-only appointment data where NexHealth lacks a connector. Sikka ONE's 400+ system coverage ensures no practice is excluded from content capture prompts.
- **Eliminated**: Direct Airbyte PMS connectors (Sprint 4) and Sprint 1's planned direct PMS triggers. These are replaced by NexHealth + Sikka ONE, reducing connector maintenance from 4 approaches to 2.

| Sprint | Gains | Loses |
|---|---|---|
| Sprint 4 (Funnel OS) | Broader PMS coverage (20+ vs. 3), real-time event streams via NexHealth webhooks | Must adapt Airbyte connector framework to consume NexHealth/Sikka APIs instead of direct PMS connections |
| Sprint 5 (Comms Engine) | No change — NexHealth was the original choice | Nothing |
| Sprint 6 (Content Concierge) | NexHealth real-time events supplement Sikka ONE batch sync | Must consume events from shared bus rather than direct Sikka connection |
| Sprint 1 (Studio Auto) | PMS triggers delivered via shared event bus without custom development | Phase 3 PMS work becomes a configuration task rather than a build |

**Migration path**: Build the NexHealth integration first (Sprint 5's design). Add Sikka ONE as a batch adapter for gap coverage. Expose all PMS events on a shared event bus (Kafka or Redis Streams) that all workstreams consume.

---

### Conflict 2: Application Runtime & Language

| Dimension | Sprint 1 | Sprint 3 | Sprint 4 | Sprint 5 | Sprint 6 |
|---|---|---|---|---|---|
| **Backend** | Node.js 20, TypeScript, Fastify | Node.js, Next.js | Python, Temporal | Python | Node.js, Next.js 14 |
| **Frontend** | React 18 | Next.js/Astro + Tailwind | React + Recharts → React Native/PWA | React | Next.js 14, React Native |
| **Workflow orchestration** | BullMQ | BullMQ | Temporal + Kafka | BullMQ | BullMQ |

**Resolution: Node.js/TypeScript primary, Python for data/ML, Temporal for orchestration**
- **API services**: Node.js 20 + TypeScript + Fastify. Three of five sprints already chose this stack. TypeScript provides type safety across the full stack.
- **Frontend**: Next.js 14 + Tailwind for all web dashboards. React Native for mobile apps. Eliminates Astro (Sprint 3 option) to reduce framework count.
- **Data pipelines and ML**: Python. Sprint 4's data engineering (dbt, ClickHouse queries, anomaly detection) and Sprint 5's ML optimization stay in Python. Data scientists and ML engineers expect Python.
- **Workflow orchestration**: Temporal replaces BullMQ for cross-service workflows. BullMQ is excellent for single-service job queues but cannot orchestrate multi-step, multi-service workflows with retries, compensation, and visibility. Temporal handles long-running workflows (e.g., enrollment pipeline, content production pipeline, communication cadences) with built-in durability. BullMQ remains for intra-service task queues where Temporal would be overkill.
- **Data streaming**: Kafka replaces ad-hoc Redis pub/sub for cross-workstream events. Redis remains for caching and ephemeral state.

| Sprint | Gains | Loses |
|---|---|---|
| Sprint 1 | Fastify choice validated; gains Temporal for complex approval workflows | Must adopt Temporal for cross-service flows (moderate learning curve) |
| Sprint 3 | Clear framework choice (Next.js); gains Temporal for enrollment pipeline | Loses Astro option (minor — Astro was listed as alternative, not primary) |
| Sprint 4 | Python data stack preserved; gains Temporal (replaces custom Kafka consumers for workflow logic) | Must wrap Temporal workers in Python SDK instead of pure Kafka consumers |
| Sprint 5 | Python ML preserved; Node.js API layer aligns with other services | Must split service: Node.js API + Python ML workers (already the implicit design) |
| Sprint 6 | Next.js 14 choice validated; React Native preserved | Minor: must integrate with Temporal for pipeline orchestration |

---

### Conflict 3: LLM Provider

| Sprint | LLM Choice | Usage |
|---|---|---|
| Sprint 1 | Claude Sonnet | Brief extraction, intent classification, medium challenge |
| Sprint 2 | OpenAI NLP | Trust Transfer verification, call analysis |
| Sprint 3 | Claude API | Data harvesting extraction, Concierge Agent |
| Sprint 4 | Claude API | Anomaly root cause analysis, action generation |
| Sprint 5 | Claude/GPT (both listed) | Message generation, personalization |
| Sprint 6 | Claude API + OpenAI Whisper | Caption generation (Claude), audio transcription (Whisper) |

**Resolution: Claude as primary LLM, OpenAI for specialized audio/speech**
- **Primary LLM (Claude API)**: All text generation, classification, analysis, and agent reasoning. Claude is already the dominant choice (4 of 6 sprints). Standardizing reduces prompt engineering effort, simplifies billing, and enables consistent behavior.
- **Audio/Speech (OpenAI Whisper)**: Audio transcription stays on Whisper. No Claude equivalent with Whisper's accuracy and language coverage. Sprint 2's Cinnamon call analysis also benefits from Whisper for pre-processing before Claude analysis.
- **Eliminated**: Sprint 2's direct OpenAI NLP for text analysis. Trust Transfer NLP verification migrates to Claude, which handles nuanced behavioral analysis well.

| Sprint | Gains | Loses |
|---|---|---|
| Sprint 2 | Consistent analysis model, shared prompt patterns | Must re-engineer Trust Transfer prompts for Claude (one-time effort) |
| Sprint 5 | Clear model choice, no ambiguity | Loses optionality of model switching (mitigated by Claude's capabilities) |
| All | Unified token accounting, single vendor relationship, shared prompt library | Vendor concentration risk (mitigated by LLM abstraction layer) |

**Migration path**: Build an LLM abstraction layer (provider-agnostic interface) so model swaps are configuration changes, not code changes. Start with Claude for all text workloads. Keep Whisper for audio. Re-evaluate periodically.

---

### Conflict 4: Database Architecture

| Sprint | Primary DB | Analytics DB | Cache |
|---|---|---|---|
| Sprint 1 | PostgreSQL | — | Redis |
| Sprint 2 | CMS (unspecified) | — | — |
| Sprint 3 | PostgreSQL | — | Redis |
| Sprint 4 | PostgreSQL (operational) | ClickHouse (hot), S3 (cold) | Redis |
| Sprint 5 | PostgreSQL | — | — |
| Sprint 6 | PostgreSQL RDS | — | — |

**Resolution: PostgreSQL (operational) + ClickHouse (analytics) + Redis (cache) + S3 (storage)**
- **PostgreSQL (AWS RDS)**: Single operational database cluster for all three workstreams. Schema separation by workstream (schemas: `platform`, `content`, `culture`). Shared tables for cross-cutting entities (practices, users, permissions).
- **ClickHouse**: Analytics database for time-series funnel events, communication metrics, content performance, and H-Score history. All workstreams write events; all workstreams query for dashboards and ML.
- **Redis**: Shared caching layer. Session state, rate limiting, real-time leaderboards (Culture OS), agent context windows.
- **S3**: Asset storage (content media, generated designs, video files), data lake cold tier, audit logs.

No sprint loses its original choice — PostgreSQL and Redis were nearly universal. ClickHouse was only in Sprint 4 but benefits all workstreams. Sprint 2's "CMS" resolves to PostgreSQL + a headless CMS layer (content models in PostgreSQL, media in S3).

---

### Conflict 5: Cloud Provider

| Sprint | Cloud Choice |
|---|---|
| Sprint 1 | Azure |
| Sprint 3 | Cloudflare + Vercel + AWS |
| Sprint 4 | AWS (ECS, RDS, S3) |
| Sprint 5 | AWS/GCP (listed as options) |
| Sprint 6 | AWS + CloudFront |

**Resolution: AWS primary, Vercel for frontend, Cloudflare for edge/CDN**
- **AWS**: Primary cloud. ECS (or EKS) for services, RDS for PostgreSQL, S3 for storage, SQS/MSK for messaging. Four of five sprints already use AWS.
- **Vercel**: Frontend hosting for Next.js dashboards. Superior Next.js deployment experience and edge rendering.
- **Cloudflare**: CDN and edge caching for practice websites (Sprint 3). Workers for edge logic where latency matters.
- **Eliminated**: Azure as primary cloud. Sprint 1's Azure choice was driven by Microsoft Teams Bot Framework dependency. Resolution: Teams Bot Framework SDK works from any cloud — it communicates with Azure Bot Service via HTTPS. The bot service registration stays in Azure; the bot application runs on AWS.

| Sprint | Gains | Loses |
|---|---|---|
| Sprint 1 | Unified cloud ops, shared infrastructure | Must configure Teams Bot Framework for AWS hosting (well-documented pattern) |
| Sprint 3 | Cloudflare + Vercel preserved for frontend; AWS backend unified | Nothing — this was already a multi-cloud design |
| Sprint 4 | No change — AWS was the primary choice | Nothing |

**Migration path for Sprint 1**: Register Bot Framework app in Azure. Host bot application on AWS ECS alongside other services. Use Azure Bot Service as a relay (this is Microsoft's supported architecture for non-Azure hosting).

---

### Conflict 6: Communication Transport

| Sprint | SMS/Voice | Email |
|---|---|---|
| Sprint 1 | — | M365 Graph API (existing) |
| Sprint 5 | Twilio (HIPAA/BAA) | SendGrid or AWS SES |
| Sprint 6 | Twilio (SMS/WhatsApp) | — |

**Resolution: Twilio for all patient-facing communications, M365 for internal/studio communications**
- **Twilio (HIPAA BAA)**: All patient-facing SMS, MMS, voice, and WhatsApp. Single vendor with HIPAA compliance, BAA, and comprehensive channel coverage. Sprint 5 and Sprint 6 already chose Twilio.
- **M365 Graph API**: Internal communications (studio@sgadental.com intake, Teams bot, internal notifications). Sprint 1's existing M365 infrastructure preserved.
- **Email (patient-facing)**: AWS SES for transactional patient email (appointment confirmations, treatment follow-ups). Cost-effective at scale and already within the AWS ecosystem.
- **Eliminated**: SendGrid as an option. AWS SES provides the same HIPAA-eligible transactional email at lower cost within the existing AWS infrastructure.

---

## Section 3: Integration Points Between Workstreams

### Integration 1: Practice Profile Sync

| Dimension | Detail |
|---|---|
| **Source** | Practice Platform (Workstream 1) |
| **Target** | Content Engine (Workstream 2) |
| **Data** | Practice profile: name, specialty, brand universe (SGA/Gen4/MODIS), brand assets (logo, colors), provider roster, content champion assignment, template preferences |
| **Trigger** | Event-driven: `practice.profile.updated` event on shared event bus |
| **Latency** | < 30 seconds |
| **Why** | Content Engine needs current practice data to select correct brand universe, apply correct templates, and route content to the right champion. Without this sync, content could be generated with stale branding or wrong provider names. |

### Integration 2: Practice Profile to Culture OS

| Dimension | Detail |
|---|---|
| **Source** | Practice Platform (Workstream 1) |
| **Target** | Culture OS (Workstream 3) |
| **Data** | Practice profile: name, type (rural/suburban/urban), specialty, region, regional director, staff roster, Morning Huddle configuration |
| **Trigger** | Event-driven: `practice.profile.updated` event on shared event bus |
| **Latency** | < 60 seconds |
| **Why** | Culture OS needs practice segmentation data for benchmarking and huddle content personalization. |

### Integration 3: Appointment Events to Content Engine

| Dimension | Detail |
|---|---|
| **Source** | Practice Platform (Workstream 1) — via shared PMS integration layer |
| **Target** | Content Engine (Workstream 2) |
| **Data** | Tomorrow's appointment schedule: patient ID (anonymized), procedure codes, provider, appointment time, media consent status |
| **Trigger** | Scheduled: Nightly batch at 10 PM local per practice timezone |
| **Latency** | Batch — delivered by 6 AM next day |
| **Why** | Content Engine's Prompt Engine needs appointment data to generate daily capture briefs for content champions. The SMS brief ("Tomorrow you have 3 implant cases — here's your capture checklist") depends on this data. |

### Integration 4: Content Assets to Practice Platform

| Dimension | Detail |
|---|---|
| **Source** | Content Engine (Workstream 2) |
| **Target** | Practice Platform (Workstream 1) |
| **Data** | Finished content assets: social post images, video clips, before/after galleries, testimonial videos with metadata (practice_id, content_type, platforms_published, publish_date) |
| **Trigger** | Event-driven: `content.asset.published` event on shared event bus |
| **Latency** | < 5 minutes |
| **Why** | Practice Platform's digital storefront can feature recent content (latest before/after, recent testimonial). Channel sync engine can push content to Google Business Profile. Funnel OS can correlate content publication with lead generation for attribution analysis. |

### Integration 5: Content Distribution Events to Funnel OS

| Dimension | Detail |
|---|---|
| **Source** | Content Engine (Workstream 2) |
| **Target** | Practice Platform (Workstream 1) — Funnel OS intelligence layer |
| **Data** | Content distribution events: content_id, practice_id, channel, publish_timestamp, engagement_metrics (views, likes, shares, clicks) |
| **Trigger** | Event-driven: `content.engagement.updated` (polled from social APIs every 6 hours, emitted as events) |
| **Latency** | 6 hours (engagement data is inherently delayed) |
| **Why** | Enables ML booking attribution (Sprint 6, Phase 4). The Funnel OS correlates content engagement spikes with lead generation spikes to measure content ROI. |

### Integration 6: H-Score to Practice Platform Dashboard

| Dimension | Detail |
|---|---|
| **Source** | Culture OS (Workstream 3) |
| **Target** | Practice Platform (Workstream 1) — Executive Dashboard |
| **Data** | H-Score per practice: composite score (0-100), sub-scores (Cinnamon phone 35%, Swell reviews 45%, Dental Intel 20%), certification tier breakdown (x Bronze, y Silver, z Gold), trend (improving/stable/declining) |
| **Trigger** | Scheduled: Daily recalculation at 6 AM ET |
| **Latency** | Daily batch |
| **Why** | Executive dashboard shows holistic practice health: funnel performance + digital presence + hospitality culture. H-Score is the culture dimension. Regional directors need all three views without switching dashboards. |

### Integration 7: Funnel Performance to Culture OS

| Dimension | Detail |
|---|---|
| **Source** | Practice Platform (Workstream 1) — Funnel OS |
| **Target** | Culture OS (Workstream 3) |
| **Data** | Funnel conversion metrics per practice: show rate, rebooking rate, treatment acceptance rate, patient satisfaction signals |
| **Trigger** | Scheduled: Weekly batch (Monday 5 AM ET) |
| **Latency** | Weekly |
| **Why** | Culture OS uses funnel data to close the loop between hospitality behaviors and business outcomes. If a practice's show rate improves after hospitality training focus, that correlation reinforces the culture flywheel. Also feeds the Morning Huddle's "impact story" segment. |

### Integration 8: Champion Roster Sync

| Dimension | Detail |
|---|---|
| **Source** | Culture OS (Workstream 3) |
| **Target** | Content Engine (Workstream 2) |
| **Data** | Champion assignments per practice: champion name, contact info, role, engagement score, streak status |
| **Trigger** | Event-driven: `champion.assigned` or `champion.updated` events |
| **Latency** | < 5 minutes |
| **Why** | Content champions (who capture photos/video) are often the same people as culture champions (who lead huddles). The Content Engine needs to know who to send capture briefs to. If a champion changes in the Culture OS, the Content Engine must update its routing. |

### Integration 9: Content Engine Brand Assets to Culture OS

| Dimension | Detail |
|---|---|
| **Source** | Content Engine (Workstream 2) |
| **Target** | Culture OS (Workstream 3) |
| **Data** | Brand-approved visual assets: practice photos, team photos, branded templates for recognition posts |
| **Trigger** | On-demand: Culture OS requests assets via API |
| **Latency** | < 2 seconds (API call to asset storage) |
| **Why** | Recognition & Rewards Engine generates social recognition posts and certificates. These need brand-compliant templates and current practice photos. Rather than maintaining its own asset library, the Culture OS pulls from the Content Engine's asset store. |

### Integration 10: Comms Engine Events to Content Engine

| Dimension | Detail |
|---|---|
| **Source** | Practice Platform (Workstream 1) — Comms Engine |
| **Target** | Content Engine (Workstream 2) |
| **Data** | Patient communication events relevant to content: successful reactivation (potential testimonial candidate), treatment completion (capture opportunity), positive review response |
| **Trigger** | Event-driven: `comms.treatment_completed`, `comms.reactivation_success` events |
| **Latency** | < 1 hour |
| **Why** | When the Comms Engine successfully reactivates a patient who then completes a high-value treatment, the Content Engine can prompt a capture opportunity. The Comms Engine identifies satisfied patients; the Content Engine captures their stories. |

### Integration Architecture Summary

```
┌──────────────────────────────────────────────────────────────────┐
│                     SHARED EVENT BUS (Kafka)                      │
│                                                                    │
│  Topics:                                                           │
│    practice.profile.updated                                        │
│    practice.appointment.scheduled                                  │
│    content.asset.published                                         │
│    content.engagement.updated                                      │
│    culture.hscore.calculated                                       │
│    culture.champion.updated                                        │
│    comms.treatment_completed                                       │
│    comms.reactivation_success                                      │
│    funnel.metrics.weekly                                           │
└──────────┬───────────────────┬──────────────────┬────────────────┘
           │                   │                  │
           ▼                   ▼                  ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  PRACTICE        │ │  CONTENT         │ │  CULTURE         │
│  PLATFORM        │ │  ENGINE          │ │  OS              │
│  (Workstream 1)  │ │  (Workstream 2)  │ │  (Workstream 3)  │
│                  │ │                  │ │                  │
│  Publishes:      │ │  Publishes:      │ │  Publishes:      │
│  - profile.*     │ │  - content.*     │ │  - culture.*     │
│  - appointment.* │ │                  │ │                  │
│  - comms.*       │ │  Consumes:       │ │  Consumes:       │
│  - funnel.*      │ │  - practice.*    │ │  - practice.*    │
│                  │ │  - comms.*       │ │  - funnel.*      │
│  Consumes:       │ │  - culture.*     │ │  - content.*     │
│  - content.*     │ │                  │ │                  │
│  - culture.*     │ │                  │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### Shared Infrastructure Layer

All three workstreams share these cross-cutting services:

| Service | Purpose | Technology |
|---|---|---|
| **Identity & Auth** | Practice users, HQ users, API keys, role-based access | Auth0 or AWS Cognito |
| **PMS Integration Hub** | Unified PMS connectivity for all workstreams | NexHealth (primary) + Sikka ONE (batch/coverage) |
| **Event Bus** | Cross-workstream event streaming | Kafka (AWS MSK) |
| **LLM Gateway** | Rate limiting, prompt routing, token accounting, model abstraction | Custom Node.js service wrapping Claude API + Whisper |
| **Asset Storage** | Media files, generated designs, documents | AWS S3 + CloudFront CDN |
| **Practice Registry** | Canonical practice data consumed by all workstreams | PostgreSQL shared schema |
| **Notification Service** | Email, push, SMS routing for internal notifications | AWS SNS + SES + Twilio |
| **Observability** | Logging, metrics, tracing across all services | Datadog or AWS CloudWatch + X-Ray |
| **Workflow Engine** | Long-running workflow orchestration | Temporal |
| **Feature Flags** | Gradual rollout, A/B testing, practice-level feature toggles | LaunchDarkly or Unleash |

---

## Section 4: Five Characteristics PMF Assessment

### Scoring Scale
1 = Weak / Not present
2 = Emerging / Partial
3 = Moderate / Functional
4 = Strong / Compelling
5 = Exceptional / Category-defining

---

### Characteristic 1: Does it solve a real problem?

| Model | Score | Rationale |
|---|---|---|
| **6 Sprints (Original)** | 5 | Each sprint was born from an observed pain point at SGA. Studio automation addresses a real bottleneck (designers overwhelmed). Hospitality operationalization addresses a real culture gap. Digital storefronts address a real absence. Funnel visibility addresses real blindness. Comms automation addresses real missed revenue. Content capture addresses a real content desert. Every single problem is real, validated, and painful. |
| **3 Workstreams (Consolidated)** | 5 | Consolidation does not dilute the problems — it groups related problems into coherent solutions. The Practice Platform solves the "practice operations" meta-problem. The Content Engine solves the "content pipeline" meta-problem. The Culture OS solves the "culture operationalization" problem. All original pain points are still addressed. |

**Score change: 5 → 5 (no change)**. The problems are equally real regardless of packaging. Consolidation does not invent new problems or abandon existing ones.

---

### Characteristic 2: Is the timing right?

| Model | Score | Rationale |
|---|---|---|
| **6 Sprints (Original)** | 3 | The timing for each individual capability is right (AI-powered workflows, practice management automation, content capture), but launching 6 separate platforms simultaneously is organizationally wrong. SGA would need 6 parallel implementation streams, 6 vendor relationships, 6 training programs. The market timing is good; the execution timing creates a capacity crisis. |
| **3 Workstreams (Consolidated)** | 4 | Consolidation fixes the execution timing problem. Three workstreams can be sequenced: Practice Platform first (highest revenue impact, most integration points), Content Engine second (builds on Practice Platform data), Culture OS third (standalone, can run in parallel). The AI infrastructure moment (Claude API maturity, NexHealth coverage, Twilio HIPAA) is ripe. Three streams are manageable; six were not. |

**Score change: 3 → 4 (+1)**. Consolidation converts good market timing into viable execution timing.

---

### Characteristic 3: Is the approach defensible?

| Model | Score | Rationale |
|---|---|---|
| **6 Sprints (Original)** | 3 | Individual sprints have moderate defensibility. Any single capability (chatbot, content pipeline, dashboard) can be replicated. Sprint 2's H-Score and Sprint 4's dental-specific funnel model have some moats, but most features use commodity APIs (Canva, Twilio, Claude). The defensibility comes from SGA-specific domain knowledge embedded in the system, not from the technology itself. |
| **3 Workstreams (Consolidated)** | 4 | Consolidation significantly increases defensibility through integration density. A competitor could replicate a content capture bot. They cannot easily replicate a system where content capture is triggered by PMS appointment data, processed through a brand-aware AI pipeline, distributed to practice storefronts, correlated with patient funnel data, and attributed to booking revenue — all coordinated through a single practice profile. The 10 integration points between workstreams create a compound moat. Each integration adds value that is invisible to single-product competitors. |

**Score change: 3 → 4 (+1)**. Integration density creates defensibility that isolated products cannot match.

---

### Characteristic 4: Can it scale?

| Model | Score | Rationale |
|---|---|---|
| **6 Sprints (Original)** | 2 | Six independent systems scaling to 260+ practices means 6 separate databases tracking practice data, 6 separate PMS integrations, 6 separate onboarding flows, and 6 separate support surfaces. When a practice changes its name or a provider leaves, someone must update 6 systems. The data synchronization problem alone makes this architecture fragile at scale. Individual sprint architectures were well-designed for their scope but did not account for the full ecosystem. |
| **3 Workstreams (Consolidated)** | 4 | Shared Practice Registry eliminates the "update 6 systems" problem. Shared PMS integration means one connector per PMS system, not six. Shared event bus means new workstreams can subscribe to existing events without integration rework. The Practice Platform handles 260+ practices with a single per-practice agent pattern. The Content Engine processes media through a pipeline that scales horizontally. The main scaling risk is the unified PostgreSQL database (mitigated by read replicas and ClickHouse offloading analytics queries). |

**Score change: 2 → 4 (+2)**. This is the largest improvement from consolidation. Shared infrastructure dramatically improves scalability by eliminating redundant integrations and data synchronization challenges.

---

### Characteristic 5: Does it create unfair advantage?

| Model | Score | Rationale |
|---|---|---|
| **6 Sprints (Original)** | 3 | The unfair advantage is SGA's proprietary operational data: 260 practices, 400K+ patients, real funnel data, real call recordings, real treatment outcomes. But with 6 separate systems, this data stays siloed. Sprint 4 has funnel data but does not know about content performance. Sprint 6 has content data but cannot correlate with bookings. Sprint 2 has culture data but cannot prove its impact on revenue. The raw material for unfair advantage exists but is not activated. |
| **3 Workstreams (Consolidated)** | 5 | Consolidation activates the data advantage. The cross-workstream integrations enable insights no competitor can generate: "Practices with H-Scores above 80 AND active content capture have 34% higher show rates and 2.3x the rebooking rate." This is not a feature — it is an ecosystem insight that requires culture data + content data + funnel data in the same analytical layer. The Network Learning Engine (merged from Sprints 4+5) can now optimize across all dimensions simultaneously. Each month of operation compounds the data advantage. No new DSO entrant can replicate 12 months of cross-domain operational data from 260 practices. |

**Score change: 3 → 5 (+2)**. The single largest PMF improvement. Data that was siloed across 6 systems becomes a connected intelligence layer that creates genuine unfair advantage.

---

### PMF Summary Scorecard

| Characteristic | 6 Sprints | 3 Workstreams | Delta |
|---|---|---|---|
| Solves a real problem | 5 | 5 | 0 |
| Timing is right | 3 | 4 | +1 |
| Approach is defensible | 3 | 4 | +1 |
| Can scale | 2 | 4 | +2 |
| Creates unfair advantage | 3 | 5 | +2 |
| **Total** | **16/25** | **22/25** | **+6** |

**Assessment**: The consolidated 3-workstream model scores 37.5% higher on PMF characteristics than the original 6-sprint model. The improvement is concentrated in scalability and unfair advantage — the two characteristics most critical for a 260-practice network that intends to grow through acquisition. The original sprints identified the right problems and designed strong individual solutions. Consolidation preserves those solutions while resolving the architectural fragmentation that would have undermined execution at scale.

---

## Appendix: Feature Count Summary

| Category | Count |
|---|---|
| Total features mapped | 72 |
| Standalone features (no merge needed) | 40 |
| Merged features (two or more sprints combined) | 28 (from 14 merge groups) |
| Integration points (cross-workstream) | 4 (Culture OS features that integrate without merging) |
| Technical conflicts resolved | 6 |
| Cross-workstream data flows defined | 10 |
| Shared infrastructure services | 10 |

---

*This document should be reviewed alongside the individual sprint architecture documents in `resources/problem-sprints/*/04-architect/` for full technical context on any specific capability.*
