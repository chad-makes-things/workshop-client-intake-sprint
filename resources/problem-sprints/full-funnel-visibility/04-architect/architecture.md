# Architecture: The Funnel OS

## Overview

The Funnel OS is a purpose-built patient funnel operating system for SGA Dental Partners. It provides real-time visibility into the 5-stage patient funnel (Leads Generated → Connected → Booked → Showed → Produced) across 260+ practices, powered by an AI analysis layer that proactively identifies problems and generates prioritized action agendas.

**Three native layers:**
1. **Data Integration Layer** — Connects 15 core platforms into a unified funnel data model
2. **Intelligence Layer** — AI that monitors, analyzes, and recommends across all practices
3. **Experience Layer** — Executive Dashboard + Practice Nerve Center

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXPERIENCE LAYER                                     │
│                                                                              │
│  ┌──────────────────────┐          ┌──────────────────────────────────┐      │
│  │  EXECUTIVE DASHBOARD │          │    PRACTICE NERVE CENTER         │      │
│  │                      │          │                                  │      │
│  │  • Network funnel    │          │  • 20-min daily agenda           │      │
│  │  • Region drill-down │          │  • Priority action cards         │      │
│  │  • Practice detail   │          │  • End-of-day scorecard          │      │
│  │  • AI alerts queue   │          │  • One-tap actions (call, text)  │      │
│  │  • Weekly/monthly    │          │  • Treatment plan follow-ups     │      │
│  │    summaries         │          │  • Extends Practice Concierge    │      │
│  └──────────┬───────────┘          └──────────────┬───────────────────┘      │
│             │                                      │                         │
│             └──────────────┬───────────────────────┘                         │
│                            │                                                 │
├────────────────────────────┼─────────────────────────────────────────────────┤
│                            │                                                 │
│                   INTELLIGENCE LAYER                                         │
│                                                                              │
│  ┌─────────────────────────┴────────────────────────────────────────────┐    │
│  │                     AI ANALYSIS ENGINE                                │    │
│  │                                                                       │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │    │
│  │  │ Anomaly      │  │ Root Cause   │  │ Action       │  │ Learning │ │    │
│  │  │ Detection    │  │ Correlation  │  │ Generation   │  │ Loop     │ │    │
│  │  │              │  │              │  │              │  │          │ │    │
│  │  │ Per-practice │  │ Cross-source │  │ Prioritized  │  │ Outcome  │ │    │
│  │  │ baselines,   │  │ correlation  │  │ actions with │  │ tracking │ │    │
│  │  │ deviation    │  │ (phone+HR+   │  │ revenue      │  │ & model  │ │    │
│  │  │ detection,   │  │ PMS+ads)     │  │ impact       │  │ updating │ │    │
│  │  │ trend        │  │ to explain   │  │ estimates,   │  │          │ │    │
│  │  │ analysis     │  │ WHY metrics  │  │ scripts,     │  │          │ │    │
│  │  │              │  │ changed      │  │ templates    │  │          │ │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  └──────────┘ │    │
│  └──────────────────────────┬────────────────────────────────────────────┘    │
│                             │                                                │
│  ┌──────────────────────────┴────────────────────────────────────────────┐    │
│  │                    AUTOMATION ENGINE                                   │    │
│  │                                                                       │    │
│  │  • Confirmation text sequences (service-type-aware)                   │    │
│  │  • Missed call follow-up routing (to AI agent or practice queue)      │    │
│  │  • No-show re-engagement (automated outreach + reschedule)            │    │
│  │  • Production gap alerts (same-week intervention triggers)            │    │
│  │  • Treatment plan follow-up sequences                                 │    │
│  └──────────────────────────┬────────────────────────────────────────────┘    │
│                             │                                                │
├─────────────────────────────┼────────────────────────────────────────────────┤
│                             │                                                │
│                   DATA INTEGRATION LAYER                                     │
│                                                                              │
│  ┌──────────────────────────┴────────────────────────────────────────────┐    │
│  │                    FUNNEL DATA LAKE                                    │    │
│  │                                                                       │    │
│  │  Unified schema: Every record is a patient funnel event               │    │
│  │  {practice_id, timestamp, funnel_stage, source, value, metadata}      │    │
│  │                                                                       │    │
│  │  Stages: lead_generated → lead_connected → lead_booked →              │    │
│  │          patient_showed → production_completed                         │    │
│  └──────────────────────────┬────────────────────────────────────────────┘    │
│                             │                                                │
│  ┌──────────────────────────┴────────────────────────────────────────────┐    │
│  │                 INTEGRATION MIDDLEWARE                                 │    │
│  │                                                                       │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │    │
│  │  │   PMS   │ │  Phone  │ │  AI     │ │   Ads   │ │   HR    │       │    │
│  │  │Connector│ │Connector│ │ Agent   │ │Connector│ │Connector│       │    │
│  │  │         │ │         │ │Connector│ │         │ │         │       │    │
│  │  │Dentrix  │ │RingCntrl│ │Sierra   │ │Google   │ │iSolved  │       │    │
│  │  │Eaglesoft│ │8x8      │ │Agent    │ │Meta     │ │         │       │    │
│  │  │Open     │ │Weave    │ │Agnes    │ │LSAs     │ │         │       │    │
│  │  │Dental   │ │         │ │         │ │         │ │         │       │    │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘       │    │
│  └───────┼──────────┼──────────┼──────────┼──────────┼──────────────────┘    │
│          │          │          │          │          │                        │
└──────────┼──────────┼──────────┼──────────┼──────────┼────────────────────────┘
           │          │          │          │          │
     ┌─────┴─────┐┌───┴───┐┌────┴────┐┌────┴────┐┌───┴────┐
     │  Practice │ │ Phone │ │   AI   │ │   Ad   │ │  HR   │
     │Management│ │Systems│ │ Phone  │ │Platforms│ │Systems│
     │ Software │ │       │ │ Agents │ │        │ │       │
     └──────────┘└───────┘└─────────┘└─────────┘└────────┘
     (per practice)  (per practice)  (per practice)  (network)  (network)
```

---

## Components

### Component 1: Integration Middleware

**Purpose**: Connect the 15 core 3rd-party platforms to the Funnel Data Lake without building 15 custom integrations from scratch.

**Architecture**:
- **Connector framework**: A pluggable connector architecture where each data source has a standardized adapter that translates source-specific data into the unified funnel event schema.
- **Connector types**:
  - **API connectors**: For platforms with REST/GraphQL APIs (most modern platforms). Poll at configurable intervals (5 min to 1 hour depending on source).
  - **Webhook receivers**: For platforms that push events (AI phone agents, some PMS systems). Near-real-time.
  - **File-based ingest**: For platforms that only export CSV/Excel (legacy PMS systems). Scheduled daily pulls.
  - **Database connectors**: For platforms where direct DB access is available (some self-hosted PMS). Read-only connections.
- **Priority connectors (Stage 1)**:
  1. PMS connectors (Dentrix, Eaglesoft, Open Dental) — appointments, production, patient records
  2. Phone system connectors (RingCentral, 8x8, Weave) — call logs, answer rates, duration
  3. AI phone agent connectors (Sierra, Agent Agnes) — conversation outcomes, bookings made
  4. Ad platform connectors (Google Ads, Meta) — lead volume, cost per lead, attribution

**Technology choices**:
- Use Airbyte (open-source) or Fivetran for pre-built connectors where available
- Custom Python connectors for dental-specific platforms (PMS APIs are often poorly documented)
- Apache Kafka or AWS EventBridge for event streaming between connectors and the data lake

**Interfaces**:
- IN: Raw data from 15+ platforms in various formats
- OUT: Standardized funnel events to the Funnel Data Lake

---

### Component 2: Funnel Data Lake

**Purpose**: Store all patient funnel data in a unified schema that supports real-time querying, historical analysis, and AI processing.

**Architecture**:
- **Event schema**: Every record is a funnel stage transition event:
  ```
  {
    event_id: uuid,
    practice_id: "garner-dental-woodstock",
    timestamp: "2026-03-10T14:32:00Z",
    funnel_stage: "lead_connected",
    previous_stage: "lead_generated",
    source: "phone_inbound",
    handler: "human" | "ai_sierra" | "ai_agnes",
    patient_id: "hashed_identifier",
    appointment_id: "pms_ref_12345",
    value_estimate: 350.00,
    metadata: {
      call_duration: 240,
      call_outcome: "appointment_booked",
      service_type: "new_patient_exam",
      source_campaign: "google_lsa_woodstock"
    }
  }
  ```
- **Storage tiers**:
  - **Hot tier**: Last 30 days, sub-second query (PostgreSQL or ClickHouse)
  - **Warm tier**: Last 12 months, <5 second query (columnar store)
  - **Cold tier**: Full history, batch-query (S3/parquet)
- **Derived views**:
  - Practice funnel snapshot (current state of each funnel stage per practice)
  - Network funnel aggregate (roll-up across all practices, regions, specialties)
  - Trend views (week-over-week, month-over-month per practice and network)
  - Conversion rate views (stage-to-stage conversion rates with drill-down)

**Technology choices**:
- ClickHouse for the hot analytical store (purpose-built for real-time analytics over event data)
- PostgreSQL for operational data (practice configs, user accounts, alert history)
- S3 + Parquet for cold storage and ML training data
- dbt for data transformations (raw events → derived views)

**Interfaces**:
- IN: Standardized funnel events from Integration Middleware
- OUT: Query API for Dashboard, Nerve Center, and AI Engine

---

### Component 3: AI Analysis Engine

**Purpose**: Continuously monitor funnel data across all practices, detect anomalies, correlate root causes, and generate prioritized recommendations.

**Architecture**:
- **Anomaly Detection**:
  - Per-practice baselines calculated from rolling 90-day history (not network averages)
  - Adjustments for: day-of-week, seasonality, practice size, service mix, recent events
  - Alert when a metric deviates >2σ from baseline for >24 hours (configurable threshold)
  - Different sensitivity for different funnel stages (phone answer rate is high-sensitivity; ad impressions are low-sensitivity)

- **Root Cause Correlation**:
  - Cross-source correlation engine that links metrics across data sources
  - Example patterns:
    - Phone answer rate drops + iSolved staffing change = understaffing
    - Show rate drops + no confirmation texts sent = automation failure
    - Lead volume drops + ad spend unchanged = landing page or GMB issue
    - Production drops + bookings steady = case acceptance problem (clinical, not operational)
  - Correlation rules: mix of predefined rules (expert knowledge) + learned patterns (ML)

- **Action Generation**:
  - Each detected anomaly generates a recommended action (not just an alert)
  - Actions are ranked by estimated revenue impact
  - Actions include: who should do it (practice staff vs. HQ vs. automation), what specifically to do (scripts, templates, one-tap actions), and expected outcome
  - Action templates are service-type-aware (routine cleaning ≠ implant consult)

- **Learning Loop**:
  - Weekly batch: review all interventions, measure outcomes, update recommendation models
  - Key learning: which interventions were executed? Which improved the target metric? By how much?
  - Model updates: adjust revenue impact estimates, refine which interventions work for which practice profiles
  - Transparency: every recommendation is explainable ("recommended because phone calls at similar practices improved show rates by X%")

**Technology choices**:
- Python-based ML pipeline (scikit-learn for baselines, statistical process control for anomaly detection)
- LLM (Claude API) for generating natural-language action descriptions and follow-up scripts
- Redis for real-time alert state management
- Scheduled jobs (Airflow or Temporal) for the weekly learning cycle

**Interfaces**:
- IN: Funnel Data Lake queries (real-time and batch)
- OUT: Alerts, recommendations, and actions → Dashboard, Nerve Center, Automation Engine

---

### Component 4: Automation Engine

**Purpose**: Execute automated interventions based on AI recommendations and predefined rules, without requiring human action for routine tasks.

**Architecture**:
- **Confirmation workflows**:
  - Automated text/email appointment confirmations at configurable intervals (48h, 24h, 4h)
  - Service-type-aware messaging (casual tone for cleanings, reassuring tone for procedures)
  - Escalation: if no confirmation response, flag for Nerve Center phone call

- **Missed call recovery**:
  - Detect missed calls within 5 minutes
  - Route to AI phone agent (Sierra/Agnes) for immediate callback OR
  - Queue in Nerve Center for human callback within 1 hour
  - Log outcome back to funnel data lake

- **No-show re-engagement**:
  - Auto-detect no-shows from PMS (appointment time passed + no check-in)
  - Send text within 30 minutes: "We missed you today! Would you like to reschedule?"
  - If no response in 24h, queue for phone follow-up in Nerve Center

- **Production gap interventions**:
  - When a practice is >10% behind weekly production target by Wednesday, flag to HQ ops queue
  - Generate suggestions: open appointment slots to fill, unscheduled treatment plans to follow up, recall patients to contact

**Technology choices**:
- Temporal or AWS Step Functions for workflow orchestration
- Twilio for SMS/voice outreach
- Integration with existing AI phone agents (Sierra, Agnes) via their APIs
- All actions logged to Funnel Data Lake for the learning loop

**Interfaces**:
- IN: Triggers from AI Analysis Engine + predefined rules
- OUT: SMS/email/voice actions to patients, notifications to Nerve Center, events to Data Lake

---

### Component 5: Executive Dashboard

**Purpose**: Give SGA leadership real-time, drill-down visibility into patient funnel health across the entire network.

**Architecture**:
- **Views** (hierarchical drill-down):
  1. **Network view**: 5-stage funnel with aggregate numbers, conversion rates, trend arrows. Color-coded health. Top 5 AI alerts sidebar.
  2. **Region view**: Same funnel for a geographic region. Practice-level heatmap (each practice colored by health score).
  3. **Practice view**: Individual practice funnel with all 5 stages, week-over-week trends, intervention history, staffing context.
  4. **Stage view**: Deep dive into one funnel stage across the network (e.g., "Phone Answer Rates: all 260 practices ranked").

- **AI-generated summaries**:
  - Daily: "3 practices need attention. Top issue: phone answer rates in Southeast region."
  - Weekly: Full network summary with trends, interventions taken, outcomes measured.
  - Monthly: Month-end report with financials, funnel conversion rates, year-over-year comparison.

- **Stage 1 implementation**: Use Metabase (open-source BI tool) or Looker connected to ClickHouse. Replace with custom dashboard UI in Stage 3 for tighter integration with the Nerve Center and AI alerts.

**Technology choices**:
- Stage 1: Metabase or Apache Superset (open-source BI) connected to ClickHouse
- Stage 3: React-based custom dashboard with Recharts/D3 for visualizations
- WebSocket for real-time updates (push, not poll)
- Role-based access: CEO sees network, regional VP sees region, ops team sees queue

**Interfaces**:
- IN: Funnel Data Lake queries + AI Analysis Engine alerts
- OUT: Visual dashboard served to web browsers (responsive, works on desktop + tablet)

---

### Component 6: Practice Nerve Center

**Purpose**: Replace 10 tools with 1 interface that gives practice staff a prioritized, actionable 20-minute daily agenda driven by funnel data and AI recommendations.

**Architecture**:
- **Daily Agenda**:
  - 3-5 action cards, ranked by estimated revenue impact
  - Each card has: what to do, why it matters, estimated impact, one-tap execution
  - Card types: callback (phone number pre-loaded), confirm (text template ready), follow-up (script attached), review (scorecard view)
  - Generated fresh each morning by AI Analysis Engine based on overnight data processing

- **End-of-Day Scorecard**:
  - Summary of the day: calls answered, appointments booked, show rate, production
  - Actions completed vs. assigned
  - Revenue attributed to completed actions
  - Incomplete items rolled to tomorrow's agenda

- **Role-based views**:
  - Office Manager: full agenda + scorecard + practice performance
  - Front Desk: call queue + confirmation list + scheduling actions
  - (Future) Hygienist: recall list + treatment plan follow-ups

- **Integration with Practice Concierge**:
  - The Nerve Center extends the Practice Concierge (from the Storefront sprint)
  - Same interface, same agent interaction model
  - Concierge handles: storefront, GMB, social, brand
  - Nerve Center handles: funnel metrics, daily actions, production tracking
  - Staff see one tool, not two

**Technology choices**:
- React Native or Progressive Web App (PWA) for cross-platform (iPad at front desk, phone for office manager)
- Same design system as Practice Concierge (shared components, consistent UX)
- Offline-capable for practices with spotty internet (sync when connected)
- Push notifications for urgent items (missed calls, same-day cancellations)

**Interfaces**:
- IN: AI Analysis Engine (daily agenda, actions) + Funnel Data Lake (scorecard data)
- OUT: Action execution (calls, texts, confirmations logged back to Data Lake)

---

## Architecture Decision Records (ADRs)

### ADR 1: Unified Platform vs. Best-of-Breed Assembly

**Decision**: Build a unified platform (Funnel OS) rather than assembling separate BI tool + data warehouse + AI service + practice app.

**Context**: The Dental Command Center approach (Solution 2) would be faster to first dashboard but fragments into 4-5 separate systems. The Funnel OS keeps all three layers (integration, intelligence, experience) in one coherent platform.

**Rationale**: At 260+ practices, system fragmentation is the CURRENT problem. Building a solution that's itself fragmented perpetuates the problem at a different level. The unified platform costs more upfront but reduces long-term maintenance, ensures data consistency, and enables the AI to operate across all layers natively.

**Tradeoff**: Slower time to first value (mitigated by using an off-the-shelf BI tool as Stage 1 dashboard). Higher initial engineering investment. But: one system to maintain, one data model, one AI that understands everything.

---

### ADR 2: BI Tool as Stage 1 Dashboard (Buy Then Replace)

**Decision**: Use Metabase or Superset as the Stage 1 executive dashboard, then replace with a custom dashboard in Stage 3.

**Context**: Executives need to see data in 30 days. Building a custom dashboard takes 3-5 months. The solution: use an open-source BI tool connected to the data lake immediately, then build the custom experience layer once the data model is validated.

**Rationale**: The BI tool validates the data model and executive needs before investing in custom UI. If the funnel stages, drill-down patterns, or alert formats change based on executive feedback (they will), it's cheaper to iterate in a BI tool than in custom React code. The BI tool is disposable infrastructure — it served its purpose once the custom dashboard is built.

**Tradeoff**: Executives see a "generic BI dashboard" in Stage 1, not the polished Funnel OS experience. But they see real data, in real-time, from day 30. That's the priority.

---

### ADR 3: Per-Practice Baselines (Not Network Averages)

**Decision**: The AI Analysis Engine calculates anomaly baselines per-practice, not from network averages.

**Context**: A 2-chair rural practice in Reidsville, GA and a 12-chair metro practice in Scottsdale, AZ have completely different "normal." A 70% phone answer rate might be excellent for one and terrible for the other.

**Rationale**: Network averages create false positives (flagging healthy small practices) and false negatives (missing problems at large practices where the average is still "fine"). Per-practice baselines trained on 90 days of that practice's own data ensure alerts are meaningful and contextual.

**Tradeoff**: Requires 90 days of data before baselines are reliable. New practices (acquisitions) won't have baselines. Mitigation: use cohort-based baselines (similar size, similar specialty, similar region) as initial priors until practice-specific data accumulates.

---

### ADR 4: Practice Nerve Center Extends Practice Concierge

**Decision**: Build the Practice Nerve Center as an extension of the Practice Concierge (from the Digital Storefront sprint), not as a separate application.

**Context**: Practices already need to engage with the Concierge for storefront management. Adding a SECOND practice-facing tool defeats the "replace 10 tools with 1" objective.

**Rationale**: The Concierge and Nerve Center share: the same users (office managers), the same interaction model (agent-first), the same data infrastructure (practice data hub), and the same adoption challenge (practice staff need to use it). Merging them creates one tool, one interface, one relationship.

**Tradeoff**: More complex application with broader scope. Risk of "everything app" that does nothing well. Mitigation: the Concierge and Nerve Center are separate MODULES within the same app — different screens, different contexts, but shared navigation and shared AI. Think Gmail tabs (Inbox, Social, Promotions) not a single cluttered screen.

---

### ADR 5: Service-Type-Aware Automation

**Decision**: All automation (confirmations, follow-ups, interventions) is parameterized by service type, not one-size-fits-all.

**Context**: A confirmation text for a routine cleaning is casual: "See you tomorrow at 2!" A confirmation text for a $30,000 full-arch procedure is reassuring and detailed: "We're looking forward to seeing you tomorrow. Here's what to expect..." Sending the cleaning text for the implant procedure is tone-deaf. Sending the implant text for the cleaning is over-engineered.

**Rationale**: SGA operates across general dentistry, periodontics, oral surgery, implantology, and cosmetic dentistry. Patient psychology, no-show risk, revenue per visit, and appropriate communication tone vary dramatically by service type. The automation engine must understand these differences.

**Tradeoff**: More complex automation rules. More templates to maintain. But: contextual automation builds patient trust, while generic automation erodes it. And the revenue difference between a $150 cleaning no-show and a $30,000 implant no-show justifies the investment in differentiation.

---

## Staged Delivery Plan

| Stage | Timeline | Components Built | Data Sources | Practices | Key Milestone |
|-------|----------|-----------------|-------------|-----------|---------------|
| **1** | Month 1 | Integration Middleware (3 connectors) + Funnel Data Lake + Executive Dashboard (via BI tool) | PMS, Phone, 1 lead source | 10 pilot | Executives see real-time funnel for 10 practices |
| **2** | Month 2-3 | AI Analysis Engine (anomaly detection + alerts) + additional connectors | + AI phone agents, Ads, HR | 30 pilot | AI generates first alerts; HQ ops queue functional |
| **3** | Month 3-5 | Practice Nerve Center MVP + custom Executive Dashboard (replaces BI tool) | All 15 sources | 50 pilot | Practice staff using 20-min agenda; custom dashboard live |
| **4** | Month 5-8 | Automation Engine + Nerve Center/Concierge merge + network rollout | All sources | 260+ | Full automation; all practices onboarded |
| **5** | Month 8-12 | Learning Loop optimization + advanced analytics + predictive capabilities | All + new sources | 260+ | AI learning from outcomes; predictive show rates, production forecasting |

---

## Technology Stack Summary

| Layer | Technology | Why |
|-------|-----------|-----|
| Integration | Airbyte + custom Python connectors | Open-source, pre-built connectors for common platforms, extensible for dental-specific |
| Event streaming | Apache Kafka or AWS EventBridge | Decouples connectors from consumers, enables real-time flow |
| Hot data store | ClickHouse | Purpose-built for real-time analytics on event data, fast aggregations |
| Operational DB | PostgreSQL | Practice configs, user accounts, alert history, action logs |
| Cold storage | S3 + Parquet | Historical data, ML training data |
| Transformations | dbt | SQL-based transformations from raw events to derived views |
| AI/ML | Python (scikit-learn, statsmodels) + Claude API | Statistical anomaly detection + LLM for natural-language generation |
| Workflow orchestration | Temporal | Long-running workflows, automation sequences, retry logic |
| Stage 1 dashboard | Metabase or Apache Superset | Fast, open-source BI connected to ClickHouse |
| Custom dashboard | React + Recharts/D3 | Custom experience layer, real-time via WebSocket |
| Practice Nerve Center | React Native / PWA | Cross-platform (iPad + phone), offline-capable, shared with Practice Concierge |
| Communication | Twilio | SMS, voice for patient outreach; well-supported APIs |
| Infrastructure | AWS (ECS/EKS, RDS, S3, EventBridge) | Scalable, managed services, cost-effective at scale |

---

## Team Requirements

| Role | Count | Stage Needed | Responsibilities |
|------|:-----:|-------------|-----------------|
| Tech Lead / Architect | 1 | Stage 1 | System design, integration architecture, data model |
| Backend Engineer | 2 | Stage 1 | Connectors, data pipeline, API layer |
| Data Engineer | 1 | Stage 1 | Data lake, transformations (dbt), ClickHouse |
| ML Engineer | 1 | Stage 2 | Anomaly detection, learning loop, recommendation models |
| Frontend Engineer | 2 | Stage 3 | Executive dashboard (custom), Practice Nerve Center |
| Designer | 1 | Stage 3 | Dashboard UX, Nerve Center UX, shared design system |
| DevOps / Platform | 1 | Stage 1 | AWS infrastructure, CI/CD, monitoring |
| Product Manager | 1 | Stage 1 | Priorities, stakeholder alignment, pilot practice management |
| **Total** | **10** | | |
