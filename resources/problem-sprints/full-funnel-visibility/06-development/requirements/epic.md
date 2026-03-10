# Epic: The Funnel OS — Unified Patient Funnel Operating System

**Sprint**: Full Funnel Visibility
**Product Manager**: SGA Dental Partners — Technology & Operations
**Date**: 2026-03-10
**Status**: Draft

---

## Epic Description

The Funnel OS is a unified patient funnel operating system for SGA Dental Partners (260+ dental practices). It replaces a fragmented landscape of 75+ disconnected systems with a single platform that integrates 15 core third-party platforms, applies AI-driven analysis, and surfaces actionable intelligence through two interfaces: an Executive Dashboard for leadership and a Practice Nerve Center for practice staff.

The platform tracks every patient through a 5-stage funnel:

```
Leads Generated → Leads Connected With → Leads Booked → Patient Shows Up → Production / In the Chair
```

Three architectural layers work together:

1. **Data Integration Layer** — Connects PMS systems (Dentrix, Eaglesoft, Open Dental), phone systems, AI phone agents (Sierra, Agent Agnes), ad platforms, and HR systems into a unified funnel data lake
2. **Intelligence Layer** — AI anomaly detection with per-practice baselines, root cause correlation across data sources, action generation with revenue impact estimates, and a weekly learning loop
3. **Experience Layer** — Executive Dashboard (network to region to practice drill-down) and Practice Nerve Center (20-minute daily agenda for practice staff, extending the Practice Concierge from the Digital Storefront sprint)

---

## Business Objectives

| # | Objective | Current State | Target State |
|---|-----------|---------------|--------------|
| BO-1 | Reduce business health visibility lag | 30+ days (end-of-month reconciliation) | Real-time (<4 hours) |
| BO-2 | Executive drill-down visibility | Manual assembly from multiple systems | Network → Region → Practice in 2 clicks, <3 sec load |
| BO-3 | Replace fragmented practice tools | 10+ tools, no prioritization, hour+ daily | 1 tool, 20-minute daily agenda |
| BO-4 | Proactive problem identification | Reactive — problems found weeks later | AI identifies issues within hours and generates prioritized interventions |
| BO-5 | Service-type-aware operations | One-size-fits-all communication | Automation differentiates cleaning vs. implant consult vs. full-arch |

---

## Success Metrics

| Metric | Baseline | Target | Measurement Method | Timeline |
|--------|----------|--------|-------------------|----------|
| Visibility lag | 30+ days | <4 hours | Timestamp delta: event occurrence vs. dashboard display | Stage 1 (Month 1) |
| Executive drill-down speed | N/A (manual) | Network → Practice in 2 clicks, <3 sec load | UX testing + performance monitoring | Stage 1 |
| Practice staff agenda completion | 0% (no agenda exists) | >70% daily completion rate | In-app tracking of action card completion | Stage 3 (Month 3-5) |
| AI alert quality | N/A | >80% of alerts result in action taken | Alert-to-action ratio from ops queue logs | Stage 2 (Month 2-3) |
| Show rate improvement | Practice baseline | +5% across pilot practices | Before/after comparison for pilot cohort | Stage 4 (Month 5-8) |
| Data integration coverage | 0 platforms connected | 15 platforms connected | Connector health monitoring | Stage 3 |
| False positive alert rate | N/A | <20% | Weekly review of dismissed/ignored alerts | Stage 2 |

---

## Staged Delivery

| Stage | Timeline | Scope | Practices |
|-------|----------|-------|-----------|
| **Stage 1** | Month 1 | 3 connectors (PMS, Phone, Leads) + Funnel Data Lake + Exec Dashboard via Metabase | 10 pilot |
| **Stage 2** | Month 2-3 | AI anomaly detection + alerts + ops queue | 30 |
| **Stage 3** | Month 3-5 | Practice Nerve Center MVP + custom exec dashboard (replaces Metabase) | 50 |
| **Stage 4** | Month 5-8 | Full automation engine + Nerve Center/Concierge merge + network rollout | 260+ |
| **Stage 5** | Month 8-12 | Learning loop optimization + predictive analytics | 260+ |

---

## Epic 1: Data Integration

> Connect third-party platforms to a standardized funnel event stream.

### DI-1: PMS Connector (Stage 1) — P0, 8 pts

**As** a data engineer,
**I want** a connector that ingests appointment, production, and patient data from Dentrix, Eaglesoft, and Open Dental,
**So that** PMS data flows into the funnel data lake in a standardized format without manual exports.

**Acceptance Criteria:**

- **Given** a practice uses Dentrix, **When** the connector runs on its configured schedule (minimum every 30 minutes), **Then** new/updated appointments, completed production, and patient check-in/check-out events are ingested as funnel events within 30 minutes of occurrence.
- **Given** a practice uses Eaglesoft or Open Dental, **When** the connector runs, **Then** the same event types are captured with the same schema as the Dentrix connector.
- **Given** a PMS API is unavailable or returns errors, **When** the connector encounters a failure, **Then** it retries with exponential backoff, logs the failure, and alerts the DevOps channel if failures persist for >1 hour.
- **Given** ingested PMS data, **When** events are written to the data lake, **Then** each event includes: `practice_id`, `timestamp`, `funnel_stage`, `patient_id` (hashed), `appointment_id`, `service_type`, `production_value`, and `provider_id`.
- **Given** the connector has been running for 24 hours, **When** a data completeness check runs, **Then** >98% of appointments in the PMS are represented in the data lake.

---

### DI-2: Phone System Connector (Stage 1) — P0, 5 pts

**As** a data engineer,
**I want** a connector that ingests call logs, answer rates, and call outcomes from RingCentral, 8x8, and Weave,
**So that** phone performance data maps to the "Leads Connected With" funnel stage.

**Acceptance Criteria:**

- **Given** a practice uses RingCentral (or 8x8, or Weave), **When** a call event occurs, **Then** the connector captures it within 15 minutes, including: call direction (inbound/outbound), answered (yes/no), duration, caller ID (hashed), timestamp, and practice ID.
- **Given** an inbound call that is answered, **When** the event is written to the data lake, **Then** it maps to `funnel_stage: lead_connected` with `source: phone_inbound` and `handler: human`.
- **Given** a missed call, **When** the event is written, **Then** it maps to a `lead_generated` event with `connection_status: missed` and triggers a missed-call webhook for downstream processing.
- **Given** call data for a practice over a day, **When** aggregated, **Then** the derived answer rate matches the phone system's native reporting within 2% tolerance.

---

### DI-3: Lead Source Connector (Stage 1) — P0, 5 pts

**As** a data engineer,
**I want** a connector that ingests lead data from Google Ads, Meta Ads, and Google Local Service Ads (LSAs),
**So that** the "Leads Generated" funnel stage has attribution data showing volume, cost, and source.

**Acceptance Criteria:**

- **Given** a Google Ads campaign is running for a practice, **When** the connector syncs (daily), **Then** it captures: impressions, clicks, conversions, cost, campaign name, ad group, and practice attribution.
- **Given** a Meta Ads campaign, **When** the connector syncs, **Then** equivalent metrics are captured and normalized to the same schema.
- **Given** a Google LSA lead, **When** the lead event is received, **Then** it is written as a `lead_generated` funnel event with `source: google_lsa`, `lead_type` (call or message), and `practice_id`.
- **Given** lead data from all sources, **When** viewed in aggregate, **Then** the total leads per practice per day are reconcilable with each platform's native reporting within 5% tolerance.

---

### DI-4: AI Phone Agent Connector (Stage 2) — P1, 5 pts

**As** a data engineer,
**I want** a connector that ingests conversation outcomes from Sierra and Agent Agnes AI phone agents,
**So that** AI-handled calls are tracked through the funnel with outcome data (booked, transferred, voicemail).

**Acceptance Criteria:**

- **Given** an AI phone agent handles a call, **When** the call concludes, **Then** a webhook or API poll captures: call ID, practice ID, caller (hashed), duration, outcome (booked/transferred/voicemail/abandoned), and appointment details if booked.
- **Given** the AI agent books an appointment, **When** the event is written, **Then** it maps to both `lead_connected` (with `handler: ai_sierra` or `ai_agnes`) and `lead_booked` funnel events.
- **Given** the AI agent transfers to a human, **When** the event is written, **Then** it maps to `lead_connected` with `handler: ai_transfer` and the transfer reason is captured in metadata.

---

### DI-5: HR System Connector (Stage 2) — P1, 3 pts

**As** a data engineer,
**I want** a connector that ingests staffing data from iSolved,
**So that** the AI analysis engine can correlate staffing changes with funnel performance shifts.

**Acceptance Criteria:**

- **Given** a staffing change event in iSolved (hire, termination, role change, schedule change), **When** the connector syncs (daily), **Then** the event is captured with: practice ID, role affected, change type, effective date.
- **Given** the staffing data, **When** queried by the AI engine, **Then** it can answer: "How many front desk staff does practice X have this week vs. last week?"

---

### DI-6: Ad Platform Connector — Extended (Stage 2) — P1, 3 pts

**As** a data engineer,
**I want** to extend the lead source connector with cost-per-lead calculations and campaign-level attribution,
**So that** executives can see ROI per marketing channel per practice.

**Acceptance Criteria:**

- **Given** ad spend and lead count data, **When** the derived view is computed, **Then** cost-per-lead is calculated per campaign, per platform, per practice, per week.
- **Given** a lead that converts through the funnel to production, **When** the full journey is tracked, **Then** the system can attribute production revenue back to the originating campaign.

---

## Epic 2: Funnel Data Lake

> Unified storage with event schema, tiered storage, and derived analytical views.

### FDL-1: Unified Funnel Event Schema (Stage 1) — P0, 5 pts

**As** a data engineer,
**I want** a standardized event schema that every connector writes to,
**So that** all downstream systems (dashboard, AI, automation) consume a single, consistent data model.

**Acceptance Criteria:**

- **Given** a funnel event from any source, **When** it is written to the data lake, **Then** it conforms to the schema: `{event_id, practice_id, timestamp, funnel_stage, previous_stage, source, handler, patient_id, appointment_id, value_estimate, service_type, metadata}`.
- **Given** the five funnel stages (`lead_generated`, `lead_connected`, `lead_booked`, `patient_showed`, `production_completed`), **When** an event is written, **Then** `funnel_stage` is one of these five values (enforced by schema validation).
- **Given** events from PMS, phone, and lead sources, **When** they share a patient identifier, **Then** events can be linked into a patient journey across funnel stages.
- **Given** the schema, **When** a new connector is added, **Then** it can map its source data to the schema using a documented mapping specification without schema changes.

---

### FDL-2: Hot Tier — Real-Time Analytical Store (Stage 1) — P0, 8 pts

**As** a backend engineer,
**I want** a ClickHouse-based hot storage tier for the last 30 days of funnel events,
**So that** the executive dashboard and AI engine can query recent data with sub-second response times.

**Acceptance Criteria:**

- **Given** a query for a single practice's funnel over the last 7 days, **When** executed, **Then** it returns in <500ms.
- **Given** a query for network-wide funnel aggregates (260+ practices), **When** executed, **Then** it returns in <2 seconds.
- **Given** a new funnel event is ingested, **When** it reaches the hot tier, **Then** it is queryable within 5 minutes of ingestion.
- **Given** the hot tier stores 30 days of data, **When** data ages past 30 days, **Then** it is automatically migrated to the warm tier.

---

### FDL-3: Warm and Cold Storage Tiers (Stage 2) — P1, 5 pts

**As** a data engineer,
**I want** a warm tier (columnar store, last 12 months) and cold tier (S3/Parquet, full history),
**So that** historical analysis and ML training have access to all data without degrading real-time query performance.

**Acceptance Criteria:**

- **Given** a query for a practice's funnel over the last 6 months, **When** executed against the warm tier, **Then** it returns in <5 seconds.
- **Given** ML training requires 12+ months of historical data, **When** a batch query is run against cold storage, **Then** it returns a complete dataset in Parquet format within 10 minutes.
- **Given** data lifecycle policies, **When** data ages, **Then** it transitions: hot (0-30 days) → warm (30-365 days) → cold (365+ days) automatically.

---

### FDL-4: Derived Analytical Views (Stage 1) — P0, 5 pts

**As** a backend engineer,
**I want** dbt-managed derived views for practice funnel snapshots, network aggregates, trend views, and conversion rates,
**So that** the dashboard and AI engine query pre-computed views instead of running expensive real-time aggregations.

**Acceptance Criteria:**

- **Given** raw funnel events, **When** the dbt transformation runs (every 15 minutes for hot, daily for warm), **Then** the following views are refreshed:
  - `practice_funnel_snapshot`: Current count and conversion rate at each funnel stage, per practice
  - `network_funnel_aggregate`: Roll-up across all practices, segmentable by region
  - `practice_trend_weekly`: Week-over-week delta for each metric, per practice
  - `conversion_rate_view`: Stage-to-stage conversion rates with practice and network benchmarks
- **Given** the `practice_funnel_snapshot` view, **When** queried, **Then** it shows: leads generated (count, trend), leads connected (count, rate, trend), leads booked (count, rate, trend), patients showed (count, rate, trend), production completed (dollar value, trend) — all for a configurable time window (today, this week, this month).
- **Given** a new practice is added, **When** its first day of data is ingested, **Then** it appears in all derived views within the next refresh cycle.

---

### FDL-5: Data Quality Monitoring (Stage 1) — P0, 3 pts

**As** a data engineer,
**I want** automated data quality checks that detect missing data, schema violations, and ingestion gaps,
**So that** the team is alerted before bad data reaches the dashboard or AI engine.

**Acceptance Criteria:**

- **Given** a practice connector has not sent data in 2 hours during business hours, **When** the monitor checks, **Then** an alert is sent to the DevOps channel.
- **Given** an event with a null or invalid `funnel_stage`, **When** ingested, **Then** it is routed to a dead-letter queue and counted in a data quality dashboard.
- **Given** daily data quality metrics, **When** reviewed, **Then** they show: events ingested per source, schema validation failure rate, ingestion latency p50/p95/p99, and practice coverage (% of expected practices reporting).

---

## Epic 3: Executive Dashboard

> Real-time funnel visibility with network-to-practice drill-down and AI alerts.

### ED-1: Network Funnel View (Stage 1) — P0, 8 pts

**As** the CEO of SGA,
**I want** a single screen showing the health of the patient funnel across all 260+ practices,
**So that** I can assess network health in under 60 seconds every morning.

**Acceptance Criteria:**

- **Given** I open the executive dashboard, **When** the network view loads, **Then** I see five horizontal bars representing the funnel stages with: aggregate counts, stage-to-stage conversion rates, trend arrows (up/down/flat vs. prior week), and color coding (green/yellow/red based on deviation from baseline).
- **Given** the network view is displayed, **When** I look at the sidebar, **Then** I see the top 5 AI-generated alerts ranked by estimated revenue impact.
- **Given** the dashboard, **When** the page loads, **Then** it renders completely in <3 seconds.
- **Given** the data, **When** displayed, **Then** it reflects events no older than 4 hours.

---

### ED-2: Region Drill-Down View (Stage 1) — P0, 5 pts

**As** a regional VP,
**I want** to drill from the network view into my region and see a heatmap of practice funnel health,
**So that** I can identify which practices in my region need attention.

**Acceptance Criteria:**

- **Given** I am on the network view, **When** I click a region, **Then** the view transitions to show that region's funnel (same 5-stage layout) with a practice heatmap below it.
- **Given** the region view, **When** I see the heatmap, **Then** each practice is color-coded by an overall health score derived from deviation-from-baseline across all funnel stages.
- **Given** the drill-down, **When** I navigate from network to region, **Then** the transition completes in <1 second (no full page reload).

---

### ED-3: Practice Detail View (Stage 1) — P0, 8 pts

**As** an HQ operations team member,
**I want** a detailed view of a single practice's funnel with trends, AI annotations, and intervention history,
**So that** I can diagnose issues and decide on interventions for that specific practice.

**Acceptance Criteria:**

- **Given** I click a practice from the region view or search by name, **When** the practice detail loads, **Then** I see: the 5-stage funnel with actual numbers, week-over-week and month-over-month trends, AI-generated annotations explaining anomalies, and a log of recent interventions (automated and manual).
- **Given** an anomaly is detected for this practice, **When** the AI annotation is displayed, **Then** it includes: what changed, probable cause (correlated from multiple data sources), recommended action, and estimated revenue impact.
- **Given** the practice detail, **When** I look at the staffing context section, **Then** I see current front desk headcount, recent staffing changes, and provider schedule coverage.
- **Given** the practice view, **When** I toggle the time range (this week / this month / last 90 days), **Then** the funnel and trends update within 2 seconds.

---

### ED-4: AI Alerts Sidebar & Ops Queue (Stage 2) — P0, 5 pts

**As** an HQ operations team lead,
**I want** a prioritized queue of AI-generated alerts across all practices, ranked by revenue impact,
**So that** my team works on the highest-impact items first and does not duplicate effort.

**Acceptance Criteria:**

- **Given** the AI engine has generated alerts, **When** I open the ops queue view, **Then** I see a list of alerts ordered by estimated revenue impact, each with: practice name, issue summary, probable cause, recommended action, and revenue at risk.
- **Given** an alert, **When** I assign it to a team member, **Then** the assignment is logged and the alert moves from "unassigned" to "in progress" with the assignee's name.
- **Given** an alert has been addressed (by automation or manual action), **When** I view the queue, **Then** resolved items are moved to a "resolved" tab with the resolution and outcome logged.
- **Given** the ops queue, **When** displayed, **Then** it shows what has already been addressed by automation or practice staff, so HQ does not duplicate effort.

---

### ED-5: Weekly and Monthly Summary Reports (Stage 2) — P1, 5 pts

**As** the CEO,
**I want** AI-generated weekly and monthly summary reports delivered to my inbox,
**So that** I have a structured narrative of network health trends without needing to log in daily.

**Acceptance Criteria:**

- **Given** it is Monday at 7:00 AM, **When** the weekly report generates, **Then** it includes: network funnel performance vs. prior week, top 5 issues and their status, interventions taken and outcomes measured, and practices to watch this week.
- **Given** it is the 1st of the month, **When** the monthly report generates, **Then** it includes: full funnel metrics with month-over-month and year-over-year comparison, production totals, conversion rates by stage, and a narrative summary generated by the AI.
- **Given** a report, **When** delivered via email, **Then** it includes both an inline summary and a link to the full interactive dashboard.

---

### ED-6: Role-Based Access Control (Stage 1) — P0, 3 pts

**As** a system administrator,
**I want** role-based access control for the dashboard,
**So that** the CEO sees the full network, regional VPs see only their region, and practice managers see only their practice.

**Acceptance Criteria:**

- **Given** a user with the "CEO" role, **When** they log in, **Then** they see the full network view with access to all regions and practices.
- **Given** a user with the "Regional VP" role, **When** they log in, **Then** they see only their assigned region's funnel and practices.
- **Given** a user with the "Practice Manager" role, **When** they log in, **Then** they see only their practice's detail view.
- **Given** any user, **When** they attempt to access a resource outside their role scope, **Then** the system returns a 403 and does not display the data.

---

## Epic 4: AI Analysis Engine

> Per-practice baselines, anomaly detection, root cause correlation, and action generation.

### AI-1: Per-Practice Baseline Calculation (Stage 2) — P0, 8 pts

**As** an ML engineer,
**I want** the system to calculate rolling 90-day baselines for each practice and each funnel metric,
**So that** anomaly detection compares each practice to its own "normal," not a network average.

**Acceptance Criteria:**

- **Given** a practice with 90+ days of data, **When** the baseline is calculated, **Then** it accounts for: day-of-week patterns, seasonality, practice size, service mix, and recent significant events (e.g., new provider, staffing change).
- **Given** a new practice (acquisition) with <90 days of data, **When** a baseline is needed, **Then** the system uses a cohort-based baseline (practices of similar size, specialty, and region) as an initial prior.
- **Given** the baseline, **When** refreshed (weekly batch), **Then** it uses a rolling 90-day window that drops the oldest week and adds the newest.
- **Given** a practice metric (e.g., phone answer rate), **When** the baseline is queried, **Then** it returns: mean, standard deviation, day-of-week adjustment factors, and confidence level.

---

### AI-2: Anomaly Detection (Stage 2) — P0, 8 pts

**As** an HQ operations team member,
**I want** the system to automatically detect when a practice metric deviates significantly from its baseline,
**So that** problems are flagged within hours instead of discovered weeks later in monthly reports.

**Acceptance Criteria:**

- **Given** a practice metric deviates >2 standard deviations from its baseline for >24 hours, **When** the anomaly detection runs (every 15 minutes), **Then** an alert is generated with: practice ID, metric name, current value, baseline value, deviation magnitude, and duration.
- **Given** different funnel stages, **When** sensitivity is configured, **Then** phone answer rate and show rate use high sensitivity (>1.5 sigma), while ad impressions and general lead volume use low sensitivity (>2.5 sigma).
- **Given** a temporary spike that self-corrects within 4 hours, **When** evaluated, **Then** no alert is generated (suppress transient noise).
- **Given** a weekend or holiday, **When** anomaly detection runs, **Then** it uses day-of-week-adjusted baselines, not weekday baselines.

---

### AI-3: Root Cause Correlation (Stage 2) — P0, 8 pts

**As** an HQ operations team member,
**I want** the system to correlate anomalies across data sources to explain WHY a metric changed,
**So that** alerts include actionable explanations, not just "metric dropped."

**Acceptance Criteria:**

- **Given** a phone answer rate drop, **When** correlated with HR data, **Then** the system identifies if a staffing change occurred within the same time window and surfaces it as the probable cause.
- **Given** a show rate drop, **When** correlated with automation data, **Then** the system checks if confirmation text sequences were sent; if not, it surfaces "confirmation automation not running" as probable cause.
- **Given** a lead volume drop with no change in ad spend, **When** correlated with ad platform data, **Then** the system flags potential landing page or GMB issues.
- **Given** a production drop with steady bookings, **When** correlated with appointment data, **Then** the system identifies it as a case acceptance or treatment planning issue (clinical, not operational).
- **Given** a correlation result, **When** displayed, **Then** it includes: confidence level (high/medium/low), supporting data points, and the reasoning chain.

---

### AI-4: Action Generation with Revenue Impact (Stage 2) — P0, 5 pts

**As** an HQ operations team member,
**I want** each detected anomaly to generate a specific recommended action with an estimated revenue impact,
**So that** I know exactly what to do and can prioritize by dollar value.

**Acceptance Criteria:**

- **Given** a detected anomaly with a root cause, **When** the action generator runs, **Then** it produces: a recommended action (specific and executable), who should do it (practice staff, HQ ops, or automation), estimated revenue impact if unaddressed (in dollars per week), and a priority score.
- **Given** a service-type context, **When** generating actions, **Then** the recommended action is appropriate for the service type (e.g., full-arch follow-up scripts differ from cleaning recall scripts).
- **Given** generated actions, **When** ranked, **Then** they are ordered by estimated weekly revenue impact descending.
- **Given** an action recommendation, **When** displayed, **Then** it is explainable: "Recommended because [data point] + [correlation] suggests [root cause], and similar interventions at comparable practices improved [metric] by [X%]."

---

### AI-5: Weekly Learning Loop (Stage 5) — P2, 8 pts

**As** a product manager,
**I want** the AI to learn from intervention outcomes weekly,
**So that** recommendations improve over time and revenue impact estimates become more accurate.

**Acceptance Criteria:**

- **Given** it is the weekly learning cycle, **When** the batch job runs, **Then** it reviews all interventions from the past week: which were executed, which were ignored, and what happened to the target metric after execution.
- **Given** an intervention that improved the target metric, **When** the model updates, **Then** the confidence and revenue impact estimate for that intervention type increases.
- **Given** an intervention that was consistently ignored across multiple practices, **When** the model updates, **Then** that intervention type is flagged for PM review (possibly bad recommendation).
- **Given** updated models, **When** future recommendations are generated, **Then** they reflect the updated confidence scores and revenue estimates.

---

## Epic 5: Practice Nerve Center

> Daily agenda, action cards, scorecard, and role-based views for practice staff.

### PNC-1: Daily Agenda Generation (Stage 3) — P0, 8 pts

**As** a practice office manager,
**I want** a prioritized list of 3-5 action cards generated fresh each morning based on overnight data,
**So that** I know exactly what to do in my first 20 minutes to move the needle on practice performance.

**Acceptance Criteria:**

- **Given** overnight data processing has completed, **When** the office manager opens the Nerve Center at 7:30 AM, **Then** they see 3-5 action cards ranked by estimated revenue impact.
- **Given** an action card, **When** displayed, **Then** it includes: what to do (specific task), why it matters (data-driven reason), estimated impact (dollar value), and a one-tap execution button (call, text, or review).
- **Given** the daily agenda, **When** all items are completed, **Then** the total time to complete is approximately 20 minutes (validated via session timing analytics).
- **Given** the morning agenda, **When** refreshed at midday, **Then** new urgent items (same-day cancellations, missed calls) are added while completed items are removed.

---

### PNC-2: Action Card Types (Stage 3) — P0, 5 pts

**As** a front desk staff member,
**I want** action cards that pre-load everything I need to execute — phone number, text template, patient context,
**So that** I can act in one tap instead of switching between multiple systems.

**Acceptance Criteria:**

- **Given** a "Callback" action card, **When** I tap it, **Then** the patient's phone number is pre-loaded, their name and appointment context are displayed, a suggested script is shown, and tapping "Call" initiates the call.
- **Given** a "Confirm" action card, **When** I tap it, **Then** a service-type-appropriate confirmation text is pre-composed, the patient name is inserted, and tapping "Send" dispatches the text via Twilio.
- **Given** a "Follow-Up" action card (treatment plan follow-up), **When** I tap it, **Then** I see: the patient's unscheduled treatment plan, recommended talking points, and a scheduling action to book the appointment.
- **Given** a "Review" action card (scorecard or metric review), **When** I tap it, **Then** it opens a summary view of the relevant metric with trend data.

---

### PNC-3: End-of-Day Scorecard (Stage 3) — P1, 5 pts

**As** a practice office manager,
**I want** an end-of-day summary showing what was accomplished, what was missed, and how the practice performed,
**So that** I can close out the day with a clear picture and ensure nothing critical rolls to tomorrow unaddressed.

**Acceptance Criteria:**

- **Given** it is 5:00 PM, **When** I open the scorecard, **Then** I see: calls answered vs. total, appointments booked vs. target, show rate for the day, production completed vs. daily target, and actions completed vs. assigned.
- **Given** incomplete action items, **When** the scorecard is generated, **Then** they are automatically rolled to the next day's agenda with an "overdue" flag.
- **Given** revenue attributed to completed actions, **When** displayed, **Then** the scorecard shows: "Your follow-up calls today resulted in 2 appointments booked, estimated revenue: $1,200."

---

### PNC-4: Role-Based Views (Stage 3) — P1, 3 pts

**As** a practice administrator,
**I want** the Nerve Center to show different views based on the staff member's role,
**So that** front desk staff see their call queue and confirmations while the office manager sees the full practice performance picture.

**Acceptance Criteria:**

- **Given** a user with the "Office Manager" role, **When** they log in, **Then** they see: full daily agenda, practice scorecard, performance trends, and all action card types.
- **Given** a user with the "Front Desk" role, **When** they log in, **Then** they see: callback queue, confirmation list, and scheduling-related action cards (no financial performance data).
- **Given** role configuration, **When** a practice admin assigns roles, **Then** they can choose from predefined roles (Office Manager, Front Desk) and assign multiple staff.

---

### PNC-5: Practice Concierge Integration (Stage 4) — P1, 5 pts

**As** a practice office manager,
**I want** the Nerve Center and Practice Concierge to be modules within one application,
**So that** I use one tool for both storefront management (Concierge) and funnel operations (Nerve Center).

**Acceptance Criteria:**

- **Given** the unified application, **When** I navigate between the Concierge and Nerve Center, **Then** I switch between tabs/sections within the same app (not separate logins or URLs).
- **Given** shared infrastructure, **When** both modules query data, **Then** they share the same practice profile, user authentication, and design system.
- **Given** the merged experience, **When** staff are trained, **Then** they learn one interface, one login, one notification system.

---

### PNC-6: Push Notifications for Urgent Items (Stage 3) — P1, 3 pts

**As** a front desk staff member,
**I want** push notifications for time-sensitive items (missed calls, same-day cancellations),
**So that** I can respond within minutes instead of checking the app periodically.

**Acceptance Criteria:**

- **Given** a missed call is detected, **When** the Nerve Center processes it, **Then** a push notification is sent to assigned front desk staff within 5 minutes.
- **Given** a same-day cancellation, **When** detected, **Then** a push notification includes: patient name, appointment time, suggested fill-in actions.
- **Given** notification preferences, **When** configured by the office manager, **Then** they can control which roles receive which notification types.

---

### PNC-7: Offline Capability (Stage 4) — P2, 5 pts

**As** practice staff in a location with spotty internet,
**I want** the Nerve Center to work offline with my current agenda and sync when reconnected,
**So that** I can complete my 20-minute agenda regardless of connectivity.

**Acceptance Criteria:**

- **Given** the app is loaded and the agenda is cached, **When** internet connectivity is lost, **Then** I can still view my action cards, see patient context, and mark items as completed.
- **Given** actions taken offline, **When** connectivity is restored, **Then** all actions sync to the server and the agenda refreshes with any new items.

---

## Epic 6: Automation Engine

> Service-type-aware automated interventions for confirmations, missed calls, and no-shows.

### AE-1: Appointment Confirmation Sequences (Stage 4) — P1, 8 pts

**As** a practice,
**I want** automated, service-type-aware confirmation texts sent at 48h, 24h, and 4h before appointments,
**So that** show rates improve without requiring staff to manually call every patient.

**Acceptance Criteria:**

- **Given** a booked appointment, **When** the 48-hour window is reached, **Then** a confirmation text is sent with tone and content appropriate to the service type (casual for cleanings, reassuring and detailed for procedures).
- **Given** the patient confirms via text reply, **When** the response is received, **Then** the appointment is marked as confirmed in the data lake and no further reminders are sent.
- **Given** no confirmation response after the 24-hour text, **When** the 4-hour window is reached, **Then** the appointment is flagged in the Nerve Center for a manual phone call by front desk staff.
- **Given** a full-arch or implant appointment (high-value), **When** confirmation is sent, **Then** the message includes: what to expect, preparation instructions, and a direct line to call with questions.

---

### AE-2: Missed Call Recovery (Stage 4) — P0, 5 pts

**As** a practice,
**I want** missed calls to be automatically followed up within 5 minutes — either by an AI phone agent or queued for human callback,
**So that** potential patients are not lost due to unanswered calls.

**Acceptance Criteria:**

- **Given** a missed inbound call, **When** detected by the phone connector within 5 minutes, **Then** the automation engine either: (a) routes to the AI phone agent (Sierra/Agnes) for immediate callback, or (b) queues in the Nerve Center for human callback within 1 hour — based on practice configuration.
- **Given** the AI agent handles the callback, **When** the conversation concludes, **Then** the outcome (booked, not interested, voicemail) is logged as a funnel event.
- **Given** the callback is queued for human follow-up, **When** 1 hour passes without action, **Then** an escalation notification is sent to the office manager.

---

### AE-3: No-Show Re-Engagement (Stage 4) — P1, 5 pts

**As** a practice,
**I want** no-shows to be automatically detected and re-engaged with a reschedule text within 30 minutes,
**So that** patients are recovered before they forget or find another provider.

**Acceptance Criteria:**

- **Given** an appointment time has passed and no check-in event is received from the PMS, **When** 15 minutes past the appointment time, **Then** the system marks it as a no-show and triggers the re-engagement sequence.
- **Given** a no-show event, **When** the sequence starts, **Then** a text is sent within 30 minutes: "We missed you today! Would you like to reschedule?" (tone adjusted by service type).
- **Given** no response to the text within 24 hours, **When** the escalation timer fires, **Then** a callback action card is added to the next day's Nerve Center agenda.
- **Given** the patient responds to reschedule, **When** a new appointment is booked, **Then** a `lead_booked` funnel event is created with `source: no_show_recovery`.

---

### AE-4: Production Gap Alerts (Stage 4) — P1, 3 pts

**As** an HQ operations team member,
**I want** the system to flag practices that are >10% behind their weekly production target by Wednesday,
**So that** there is time for same-week intervention (filling open slots, treatment plan follow-ups, recall outreach).

**Acceptance Criteria:**

- **Given** a practice's weekly production target and actual production through Wednesday, **When** actual is >10% behind target, **Then** an alert is generated in the ops queue with: gap amount in dollars, suggested actions (open slot list, unscheduled treatment plans, recall-eligible patients), and estimated recoverable revenue.
- **Given** the alert, **When** an HQ team member views it, **Then** they see specific patients/slots to act on, not just a metric.

---

### AE-5: Treatment Plan Follow-Up Sequences (Stage 4) — P2, 5 pts

**As** a practice,
**I want** unscheduled treatment plans to trigger automated follow-up sequences after the consultation,
**So that** patients who need treatment but have not booked are systematically re-engaged.

**Acceptance Criteria:**

- **Given** a treatment plan is created in the PMS but no follow-up appointment is booked within 7 days, **When** the 7-day timer fires, **Then** a follow-up text or email is sent with: treatment summary, importance statement, and a scheduling link.
- **Given** no response after the initial follow-up, **When** 14 days have passed, **Then** a follow-up action card is added to the Nerve Center agenda for a personal phone call.
- **Given** the follow-up results in a booked appointment, **When** tracked, **Then** the system attributes the booking to the follow-up sequence for learning loop analysis.

---

## Dependency Map

The following diagram shows blocking dependencies between stories. Stories within the same tier can be developed in parallel.

```
TIER 0 — Foundations (no dependencies, start immediately)
├── FDL-1  Unified Funnel Event Schema
├── FDL-5  Data Quality Monitoring
└── ED-6   Role-Based Access Control

TIER 1 — Data Pipeline (depends on schema)
├── FDL-1 ──→ DI-1   PMS Connector
├── FDL-1 ──→ DI-2   Phone System Connector
├── FDL-1 ──→ DI-3   Lead Source Connector
└── FDL-1 ──→ FDL-2  Hot Tier (ClickHouse)

TIER 2 — Views & Dashboard (depends on connectors + hot tier)
├── DI-1 + DI-2 + DI-3 + FDL-2 ──→ FDL-4  Derived Analytical Views
├── FDL-4 + ED-6 ──→ ED-1  Network Funnel View
├── FDL-4 + ED-6 ──→ ED-2  Region Drill-Down
└── FDL-4 + ED-6 ──→ ED-3  Practice Detail View

                ┌──── STAGE 1 MILESTONE: Exec Dashboard Live (10 Practices) ────┐

TIER 3 — Extended Connectors + AI Foundations (depends on data lake)
├── FDL-1 ──→ DI-4   AI Phone Agent Connector
├── FDL-1 ──→ DI-5   HR System Connector
├── DI-3  ──→ DI-6   Ad Platform Connector (Extended)
├── FDL-2 ──→ FDL-3  Warm & Cold Storage
├── FDL-2 + DI-1..3 ──→ AI-1  Per-Practice Baselines
└── AI-1  ──→ AI-2   Anomaly Detection

TIER 4 — AI Analysis + Ops Queue (depends on anomaly detection)
├── AI-2 + DI-5 ──→ AI-3   Root Cause Correlation
├── AI-3 ──→ AI-4   Action Generation
├── AI-4 ──→ ED-4   AI Alerts Sidebar & Ops Queue
└── ED-1 ──→ ED-5   Weekly/Monthly Summary Reports

                ┌──── STAGE 2 MILESTONE: AI Alerts + Ops Queue (30 Practices) ────┐

TIER 5 — Practice Nerve Center (depends on AI actions)
├── AI-4  ──→ PNC-1  Daily Agenda Generation
├── PNC-1 ──→ PNC-2  Action Card Types
├── PNC-1 ──→ PNC-3  End-of-Day Scorecard
├── PNC-1 ──→ PNC-4  Role-Based Views
├── PNC-1 ──→ PNC-6  Push Notifications
└── ED-1  ──→ ED-1*  Custom Exec Dashboard (replaces Metabase)

                ┌──── STAGE 3 MILESTONE: Nerve Center MVP (50 Practices) ────┐

TIER 6 — Automation + Integration (depends on Nerve Center + connectors)
├── DI-2 + AI-4 ──→ AE-2  Missed Call Recovery
├── DI-1 + AI-4 ──→ AE-1  Appointment Confirmation Sequences
├── DI-1 + AI-4 ──→ AE-3  No-Show Re-Engagement
├── FDL-4 ──→ AE-4  Production Gap Alerts
├── DI-1 ──→ AE-5   Treatment Plan Follow-Up Sequences
├── PNC-2 ──→ PNC-5  Practice Concierge Integration
└── PNC-2 ──→ PNC-7  Offline Capability

                ┌──── STAGE 4 MILESTONE: Full Automation (260+ Practices) ────┐

TIER 7 — Learning & Optimization
└── AI-4 + AE-1..5 ──→ AI-5  Weekly Learning Loop

                ┌──── STAGE 5 MILESTONE: Learning Loop + Predictive (260+) ────┐
```

---

## Story Point Summary

| Epic | Stories | Total Points | P0 Points | P1 Points | P2 Points |
|------|:-------:|:------------:|:---------:|:---------:|:---------:|
| Epic 1: Data Integration | 6 | 29 | 18 | 11 | 0 |
| Epic 2: Funnel Data Lake | 5 | 26 | 21 | 5 | 0 |
| Epic 3: Executive Dashboard | 6 | 34 | 24 | 5 | 0 |*
| Epic 4: AI Analysis Engine | 5 | 37 | 29 | 0 | 8 |
| Epic 5: Practice Nerve Center | 7 | 34 | 13 | 16 | 5 |
| Epic 6: Automation Engine | 5 | 26 | 5 | 16 | 5 |
| **Total** | **34** | **186** | **110** | **53** | **18** |

*Includes 5 pts for ED-5 (P1) counted under P1.

**P0 stories (110 pts)** represent the minimum viable Funnel OS through Stage 2.
**P0 + P1 stories (163 pts)** deliver the full product through Stage 4.
**P2 stories (18 pts)** are Stage 5 optimization and resilience enhancements.

---

## Open Questions & Technical Spikes

| # | Question | Spike Required | Owner | Stage |
|---|----------|---------------|-------|-------|
| 1 | What is the actual API coverage for Dentrix, Eaglesoft, and Open Dental? Which data points require workarounds? | Yes — 1-week API audit per PMS vendor | Backend Eng | Stage 1 |
| 2 | Can ClickHouse handle the query patterns for 260+ practices with sub-second response? | Yes — load test with synthetic data at full scale | Data Eng | Stage 1 |
| 3 | What is the false positive rate for statistical anomaly detection with only 90 days of baseline? | Yes — backtest against historical data | ML Eng | Stage 2 |
| 4 | Will Twilio's SMS deliverability work for dental appointment confirmations at scale (opt-in, A2P compliance)? | Yes — compliance review + pilot test | Backend Eng | Stage 3 |
| 5 | How will the Nerve Center and Practice Concierge share a codebase without coupling their release cycles? | Architecture review — module boundaries, shared component library | Tech Lead | Stage 3 |
| 6 | What is the data latency from PMS event to dashboard display end-to-end? Can we hit <4 hours consistently? | Yes — end-to-end latency measurement with pilot practices | Data Eng | Stage 1 |

---

*This epic is derived from the Full Funnel Visibility sprint (2026-03-10). Source artifacts: `01-frame/`, `03-decide/`, `04-architect/architecture.md`.*
