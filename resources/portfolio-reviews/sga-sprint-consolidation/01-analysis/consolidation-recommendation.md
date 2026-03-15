# SGA Sprint Portfolio — Consolidation Recommendation

## Recommendation: 6 Sprints → 3 Workstreams

Based on the redundancy analysis, we recommend consolidating 6 independent sprints into 3 unified workstreams. This preserves all product capabilities while eliminating duplicative build effort in data integration, agent architecture, dashboards, and infrastructure.

---

## Proposed Workstreams

### Workstream 1: The Practice Platform
**Combines**: Practice Concierge + Funnel OS + Comms Engine

**Rationale**: These three sprints explicitly state they converge into a single system. The Comms Engine sprint says "Comms Engine agent IS the Practice Concierge agent extended." Funnel OS says "Nerve Center extends Practice Concierge." Building them as one system from the start avoids months of rework.

**What this workstream owns**:
- One AI agent per practice (handles onboarding, storefront management, patient communications, and funnel data)
- Unified data integration layer (one PMS connector strategy for all)
- Single practice-level interface (Nerve Center with modular tabs)
- Single executive dashboard (network → region → practice drill-down)
- Patient funnel visibility (5-stage funnel, real-time analytics)
- Automated patient communications (lead response, treatment follow-up, rebooking, reactivation)
- Practice enrollment and digital storefront management
- Channel sync engine (GMB, Meta, directories, CRM, review platforms)

**Original sprint capabilities preserved**:
- From Practice Concierge: silent data harvesting, enrollment pipeline, 5 specialty templates, blitz deploy, change management, network health dashboard
- From Funnel OS: 15-platform integration, AI anomaly detection, executive dashboard, Practice Nerve Center, automation engine, per-practice baselines
- From Comms Engine: 4-stage automation (lead response, treatment follow-up, rebooking, reactivation), graduated autonomy, network learning engine, voice AI, playbook engine

### Workstream 2: The Content Engine
**Combines**: Studio Automation + Content Concierge

**Rationale**: Both sprints produce marketing content for practices, both need brand asset access, both have approval workflows, both distribute to patient-facing channels. While the production methods differ (Canva design generation vs. photo/video capture + AI processing), they share infrastructure needs and should present as a unified content system to practices.

**What this workstream owns**:
- Creative asset production (email intake → Canva generation → CD approval)
- Content capture system (SMS-triggered photo/video → AI processing pipeline)
- Shared brand asset management (one DAM, one set of brand kits)
- Unified approval workflow engine (practice submits → HQ reviews → auto-deploy)
- Multi-channel distribution (6 social platforms, email campaigns, TV slides, print)
- Content analytics and attribution
- Proactive content triggers (PMS-driven seasonal content, procedure-matched briefs)

**Original sprint capabilities preserved**:
- From Studio Automation: conversational email intake, Canva API template generation, medium challenge (print→digital), CD mobile review via Teams Adaptive Cards, Gelato print fulfillment, brand health scoring
- From Content Concierge: SMS coach with procedure-matched briefs, AI photo/video processing pipeline, 5 practice-type playbooks, champion scorecard, testimonial capture, platform-specific caption generation, paid media asset library

### Workstream 3: The Culture OS
**Keeps**: Hospitality Playbook (standalone, with integration points)

**Rationale**: The Hospitality Playbook is the most independent sprint — it addresses culture, training, and certification, which are fundamentally different from the data/automation/content concerns of the other sprints. However, its H-Score engine should plug into the Practice Platform's data layer rather than building a separate integration.

**What this workstream owns**:
- Morning Huddle OS (daily 5-minute ritual via Dental Intel)
- CCA Academy (Bronze → Silver → Gold certification with portable credentials)
- H-Score Engine (Cinnamon 35% + Swell 45% + Dental Intel 20%)
- Trust Transfer Verification (NLP + behavioral tracking)
- Champion Network (260 champions, peer community, regional cohorts)
- Recognition & Rewards Engine (leaderboard, bonus pool)
- Playbook Distribution System (versioned living document)
- Content Production Pipeline (Grant batch recording)

**Integration points with other workstreams**:
- H-Score badge → Practice Platform (storefront badge)
- Swell review data → Practice Platform (reputation layer in funnel)
- Trust transfer signals → Practice Platform (trigger referral campaigns via Comms Engine)
- Champion stories → Content Engine (feed testimonial pipeline)
- Gold staff → Content Engine (credible brand voices for content)

---

## Shared Infrastructure Layer

All three workstreams share:

| Service | Recommendation | Rationale |
|---------|---------------|-----------|
| **PMS Integration** | Sikka ONE API | Broadest coverage (400+ PMS systems); single integration point for appointment, patient, and practice data |
| **Cloud Provider** | AWS (primary) | Already used by 4 of 6 sprints; most mature healthcare/HIPAA tooling |
| **Database** | PostgreSQL (RDS) + ClickHouse (analytics) | PostgreSQL for operational data (universal across sprints); ClickHouse for real-time analytics (Funnel OS requirement) |
| **Queue System** | BullMQ (simple jobs) + Temporal (complex workflows) | BullMQ for straightforward async tasks; Temporal for multi-step automation sequences (Comms Engine, Funnel OS) |
| **AI Provider** | Claude API (primary) + OpenAI Whisper (transcription only) | Claude for all conversational, reasoning, and generation tasks; Whisper only for video transcription in Content Engine |
| **Communication** | Twilio (SMS/Voice) + SendGrid (Email) | Already appears in 3 sprints; HIPAA BAA available |
| **Auth** | AWS Cognito or Azure AD (if Microsoft ecosystem required) | Single auth system across all interfaces |

---

## What This Consolidation Saves

| Dimension | Before (6 sprints) | After (3 workstreams) | Savings |
|-----------|--------------------|-----------------------|---------|
| Data integration builds | 4 separate PMS integrations | 1 shared service | ~3-4 months engineering |
| Dashboards | 5 executive + 5 practice interfaces | 1 executive + 1 practice interface (modular tabs) | ~2-3 months frontend |
| Agent architectures | 3 competing per-practice agent designs | 1 unified agent with capability modules | ~2 months architecture + rework avoidance |
| Cloud providers to manage | 3 (Azure, AWS, GCP) | 1 (AWS) | Ongoing ops reduction |
| Queue systems to maintain | 3 (BullMQ, Kafka, Temporal) | 2 (BullMQ + Temporal) | Simplified infrastructure |
| Approval workflow engines | 3 (Studio, Content, Practice) | 1 shared engine | ~1 month engineering |
| Brand asset management | 2 (Studio DAM + Content pipeline) | 1 unified DAM | Data consistency + reduced storage |

---

## What Stays Separate (and Why)

| Capability | Stays In | Reason |
|-----------|----------|--------|
| Canva template generation | Content Engine | Unique design tool integration — no overlap |
| SMS photo/video capture | Content Engine | Unique behavior-change system — no overlap |
| Morning Huddle content | Culture OS | Unique daily ritual — no overlap |
| CCA certification | Culture OS | Unique credentialing system — no overlap |
| Patient comms automation | Practice Platform | Unique communication playbook — no overlap |
| Funnel anomaly detection | Practice Platform | Unique ML/analytics — no overlap |

---

## Build Sequence Recommendation

```
Month 1-2:  Shared Infrastructure (PMS connector, data lake, auth, agent framework)
            ↓
Month 2-4:  Practice Platform MVP (enrollment + storefront + basic funnel dashboard)
            Content Engine Phase 1 (SMS capture + editorial review — no AI pipeline yet)
            Culture OS Phase 1 (Huddle OS + Bronze certification — uses existing Dental Intel)
            ↓
Month 4-6:  Practice Platform + Comms (add lead response + treatment follow-up to agent)
            Content Engine Phase 2 (AI processing pipeline + mobile app + auto-distribution)
            Culture OS Phase 2 (H-Score engine plugged into Practice Platform data layer)
            ↓
Month 6-9:  Practice Platform full (rebooking, reactivation, network learning, full Nerve Center)
            Content Engine Phase 3 (full network rollout, hardware, auto-approval)
            Culture OS Phase 3 (Silver/Gold certification, Champion Network, academic pathway)
            ↓
Month 9-12: Practice Platform optimization (predictive analytics, advanced automation)
            Content Engine Phase 4 (content intelligence, booking attribution)
            Culture OS integration (H-Score badges on storefronts, champion stories in content)
```

**Key principle**: Shared infrastructure FIRST (Month 1-2), then all three workstreams build in parallel on top of it.
