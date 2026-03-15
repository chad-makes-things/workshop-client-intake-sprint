# SGA Sprint Portfolio — Cross-Sprint Redundancy Report

## Sprint Inventory

| # | Sprint | Slug | Primary Capability | Tech Stack | Cloud | PMS Strategy |
|---|--------|------|--------------------|-----------|-------|-------------|
| 1 | Studio Automation at Scale | studio-automation | AI creative asset production via email intake → Canva generation → approval → deployment | Node.js, TypeScript, Fastify, React 18, PostgreSQL, Redis, BullMQ | Azure | N/A (Phase 3: Dentrix, Eaglesoft, Curve) |
| 2 | The Hospitality Playbook Platform | hospitality-playbook | Culture OS — morning huddles, CCA certification, H-Score measurement | CMS, Dental Intel API, Credly, NLP Pipeline | Unspecified | N/A (uses Cinnamon, Swell, Dental Intel) |
| 3 | The Practice Concierge | digital-storefront | AI concierge for practice onboarding & digital storefront management | Next.js, Astro, Tailwind, Python, Playwright, Node.js, BullMQ, PostgreSQL, Redis | Cloudflare, Vercel, AWS | Custom data harvester (public scraping) |
| 4 | The Funnel OS | funnel-os | Unified patient funnel operating system — real-time visibility from lead to production | Python, ClickHouse, PostgreSQL, Kafka, Temporal, Metabase → React, React Native | AWS | Airbyte + custom Python connectors |
| 5 | The Comms Engine | comms-engine | Autonomous AI patient communications — lead response, treatment follow-up, rebooking, reactivation | Node.js, PostgreSQL, Twilio, Python ML, React | AWS/GCP | NexHealth Synchronizer (20+ PMS) |
| 6 | The Content Concierge | content-concierge | SMS-driven content capture, AI production pipeline, multi-channel distribution | Node.js, Next.js 14, React Native, PostgreSQL, S3, Airtable, Twilio, Cloudinary | AWS | Sikka ONE API (400+ PMS) |

---

## Cross-Sprint Comparison Matrix

### Dimension 1: Data Integration Layer

| Sprint | Approach | Coverage | Conflict Level |
|--------|----------|----------|---------------|
| Practice Concierge | Custom Python + Playwright scraping | Public data only | — |
| Funnel OS | Airbyte + custom Python connectors | 15 platforms | HIGH — different from all others |
| Comms Engine | NexHealth Synchronizer API | 20+ PMS systems | HIGH — different from Funnel OS and Content Concierge |
| Content Concierge | Sikka ONE API | 400+ PMS systems | HIGH — broadest coverage but different from others |
| Studio Automation | None (Phase 3: direct PMS APIs) | Future only | — |
| Hospitality Playbook | Direct API to Cinnamon, Swell, Dental Intel | 3 specific platforms | LOW — unique data sources |

**Finding**: 4 sprints need PMS data. 3 propose incompatible integration strategies. Building all 3 wastes months of engineering.

### Dimension 2: AI Agent Architecture

| Sprint | Agent Model | Scope | Instance Model |
|--------|------------|-------|---------------|
| Practice Concierge | Claude API (conversation, reasoning, tool_use) | Per-practice concierge | One agent per practice |
| Comms Engine | Claude or GPT for personalization | Per-practice comms automation | One agent per practice |
| Studio Automation | Claude Sonnet, Node.js orchestration | Per-request creative production | Shared agent, per-request context |
| Content Concierge | Claude API for captions/descriptions | Per-capture content generation | Shared pipeline |
| Funnel OS | Python ML + Claude API | Network-wide anomaly detection | Centralized engine |
| Hospitality Playbook | OpenAI for NLP sentiment | Review analysis | Centralized pipeline |

**Finding**: Practice Concierge and Comms Engine explicitly state they are THE SAME agent — "Comms Engine agent IS the Practice Concierge agent extended." Building them separately creates two agent instances per practice that must later merge.

### Dimension 3: Dashboards & User Interfaces

| Sprint | Executive View | Practice View | HQ Ops View |
|--------|---------------|---------------|-------------|
| Funnel OS | Executive Dashboard (network → region → practice drill-down) | Practice Nerve Center (daily agenda, action cards, scorecard) | Ops queue (14 practices ranked by revenue impact) |
| Practice Concierge | Network Health Dashboard (health scores, alerts, QC queues) | Per-practice concierge interface | HQ admin dashboard |
| Hospitality Playbook | Network Intelligence Dashboard (regional heatmap, interventions) | Practice H-Score Dashboard (score + huddle + certs) | Regional director portfolio view |
| Comms Engine | — | Weekly Summary (20-min review) | HQ Playbook Manager |
| Content Concierge | — | Champion scorecard (streak counter, engagement) | HQ Editorial Dashboard (Airtable → Next.js) |
| Studio Automation | Analytics Dashboard (request count, turnaround, brand health) | — | — |

**Finding**: 5 separate executive dashboards, 5 practice-level interfaces. An office manager would need to check multiple tools daily — exactly the fragmentation Funnel OS was designed to eliminate.

### Dimension 4: Patient Communication Channels

| Sprint | SMS | Voice | Email | Social | Print |
|--------|-----|-------|-------|--------|-------|
| Comms Engine | Twilio SMS/MMS | Twilio Voice + LLM | SendGrid/AWS SES | — | — |
| Content Concierge | Twilio SMS/WhatsApp | — | — | 6 platform APIs | — |
| Studio Automation | — | — | Weave email campaigns | — | Gelato print |
| Practice Concierge | — | — | — | Meta, directories | — |
| Funnel OS | Twilio SMS | Twilio Voice | — | — | — |

**Finding**: Twilio appears in 3 sprints independently. Weave email in Studio Automation overlaps with SendGrid/SES in Comms Engine. Social posting in Content Concierge and directory management in Practice Concierge both push to Meta.

### Dimension 5: Cloud Infrastructure

| Sprint | Primary Cloud | Database | Queue System | CDN/Hosting |
|--------|--------------|----------|-------------|-------------|
| Studio Automation | Azure | PostgreSQL (Azure) | BullMQ + Redis | Azure Static Web Apps |
| Practice Concierge | AWS + Cloudflare + Vercel | PostgreSQL + Redis | BullMQ | Cloudflare, Vercel |
| Funnel OS | AWS | ClickHouse + PostgreSQL | Kafka + Temporal | AWS (ECS, RDS, S3) |
| Comms Engine | AWS/GCP | PostgreSQL | Unspecified | Unspecified |
| Content Concierge | AWS | PostgreSQL (RDS) + S3 | Unspecified | CloudFront |
| Hospitality Playbook | Unspecified | Unspecified | BullMQ | Unspecified |

**Finding**: 3 different cloud providers, 3 different queue systems. Standardization would reduce ops complexity, security surface, and hiring requirements.

### Dimension 6: Explicit Cross-Sprint Dependencies Stated in Sprints

| From Sprint | To Sprint | Dependency |
|------------|-----------|-----------|
| Comms Engine | Practice Concierge | "Comms Engine agent IS the Practice Concierge agent extended" — same agent instance |
| Comms Engine | Funnel OS | "Communication events feed into centralized data platform" — Funnel OS is the data spine |
| Funnel OS | Practice Concierge | "Nerve Center extends Practice Concierge — one app with modular tabs" |
| Hospitality Playbook | Practice Concierge | H-Score feeds "Certified C&C Practice" badge on digital storefront |
| Hospitality Playbook | Funnel OS | Swell component maps to reputation layer |
| Hospitality Playbook | Comms Engine | Trust transfer signals trigger referral campaigns |
| Hospitality Playbook | Content Concierge | Champion wow stories feed testimonial pipeline |
| Content Concierge | Practice Concierge | Content updates practice digital storefront gallery |
| Studio Automation | Practice Concierge | Brand assets deploy to practice storefronts |

**Finding**: The sprints themselves document 9 cross-sprint dependencies. Practice Concierge is the most connected node (5 inbound dependencies). This confirms it should be the integration hub.

---

## Redundancy Findings

### CRITICAL: Three sprints designing one system (Practice Concierge + Comms Engine + Funnel OS)
- All three explicitly acknowledge they converge into a single per-practice platform
- Each independently designs: data integration, dashboards, AI agent architecture, cloud infrastructure
- Building separately creates 3 competing databases, 3 dashboards, 3 agent instances — then massive rework to merge

### HIGH: Four different PMS integration strategies
- Funnel OS (Airbyte), Comms Engine (NexHealth), Content Concierge (Sikka ONE), Practice Concierge (custom scraping)
- All need appointment and patient data from the same systems
- Building 4 integration layers is pure waste

### HIGH: Five dashboards for the same users
- Office managers would check multiple tools daily
- Contradicts the stated goal of eliminating tool fragmentation

### MEDIUM: Parallel content production pipelines (Studio Automation + Content Concierge)
- Both produce marketing content, both need brand assets, both have approval workflows, both distribute to channels
- Different enough to remain separate sprints, but should share infrastructure

### MEDIUM: Technology stack fragmentation
- 3 cloud providers, 3 queue systems, 2 AI providers
- Creates operational overhead, security surface area, hiring complexity

### LOW-MEDIUM: Hospitality Playbook H-Score duplicates Funnel OS data aggregation
- H-Score aggregates Cinnamon + Swell + Dental Intel — a mini data integration layer
- Should be a module within Funnel OS's data layer, not a separate integration

---

## Genuinely Independent Capabilities

These capabilities are unique to their sprint with no meaningful overlap:

| Capability | Sprint | Why Independent |
|-----------|--------|----------------|
| Canva API template generation | Studio Automation | Unique design production tool — no other sprint generates designed assets |
| Morning Huddle OS | Hospitality Playbook | Unique daily ritual delivery — no other sprint touches in-practice team meetings |
| CCA Academy (Bronze/Silver/Gold certification) | Hospitality Playbook | Unique training and credentialing system |
| Champion Network | Hospitality Playbook | Unique human infrastructure layer |
| SMS-triggered photo/video capture | Content Concierge | Unique behavior-change system for content creation |
| AI photo/video processing pipeline | Content Concierge | Unique media enhancement (color correction, smart crop, stabilization) |
| Gelato print fulfillment | Studio Automation | Unique print-on-demand integration |
| Trust Transfer NLP verification | Hospitality Playbook | Unique clinical handoff behavior tracking |
