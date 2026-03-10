# Implementation Plan: The Funnel OS

**Role**: Tech Lead
**Sprint**: Full Funnel Visibility
**Date**: 2026-03-10
**Source Artifacts**: `04-architect/architecture.md`, `01-frame/problem-map.md`, `05-validate/learnings.md`

---

## 1. Stack Validation

### ClickHouse — Funnel Analytics Store

**Decision**: ClickHouse as the primary analytical store for funnel events.

**Justification**: Funnel events are append-only, time-series, high-volume data. At 260+ practices generating 50-200 funnel events per day each, we're looking at 13K-52K events/day (growing to 100K+ as more data sources connect). ClickHouse is purpose-built for this: columnar storage compresses event data 10-20x, MergeTree engine handles time-series inserts efficiently, and aggregation queries (SUM production by practice by week) execute in <100ms even at millions of rows. Sub-second dashboard queries are the core UX requirement.

**Alternatives considered**: TimescaleDB (PostgreSQL extension — easier to operate but slower at analytical aggregations), BigQuery (serverless but cold-start latency kills dashboard UX), Druid (comparable performance but more operational complexity).

### Apache Kafka — Event Streaming

**Decision**: Kafka (via AWS MSK) for streaming events from connectors to ClickHouse.

**Justification**: Decouples data producers (connectors) from consumers (ClickHouse, AI engine, automation). When we add a new connector or consumer, neither side needs to change. Kafka's durability guarantees mean we never lose an event even if ClickHouse is temporarily down. At our volume (<1K events/second), a 3-broker MSK cluster is sufficient and cost-effective (~$400/month).

**Alternatives considered**: AWS EventBridge (simpler but less flexible for replay/reprocessing), Redis Streams (lighter but less durable), direct inserts to ClickHouse (simpler but couples everything).

**Concern**: Kafka is over-engineered for Stage 1 volume. Mitigation: start with a single-partition setup, scale partitions as volume grows. The architectural benefit of decoupling justifies the operational overhead even at low volume.

### Airbyte — Integration Framework

**Decision**: Airbyte (self-hosted) as the primary integration framework, supplemented by custom Python connectors.

**Justification**: Airbyte has 300+ pre-built connectors including Google Ads, Meta Ads, RingCentral, and various databases. For the dental-specific PMS systems (Dentrix, Eaglesoft, Open Dental), we'll build custom Python connectors using Airbyte's CDK (Connector Development Kit). Self-hosted gives us full control over scheduling, data residency, and connector customization. Cloud Airbyte is an option but costs more and limits custom connector deployment.

**PMS connector strategy**: Open Dental first (open-source, well-documented API), then Dentrix (Dentsply Sirona API, requires partner agreement), then Eaglesoft (Patterson API). These three cover ~70% of SGA's practices (exact distribution TBD from PMS audit).

### PostgreSQL — Operational Store

**Decision**: PostgreSQL 16 (via AWS RDS) for operational data.

**Justification**: Practice configurations, user accounts, connector settings, AI alert history, and action logs are relational data with moderate write volume. PostgreSQL handles this comfortably. Row-level security enables multi-tenant access (practice staff see only their practice). JSONB columns for semi-structured metadata (connector configs, alert details) avoid schema proliferation.

### dbt — Data Transformations

**Decision**: dbt Core for transforming raw funnel events into derived views.

**Justification**: Raw events need transformation into usable views: practice funnel snapshots (current state), conversion rate calculations, week-over-week trends, network aggregates. dbt makes these transformations version-controlled, testable, and documented. dbt models run on ClickHouse (via the dbt-clickhouse adapter). Scheduled via Airflow or cron every 15 minutes for near-real-time derived views.

### FastAPI — API Layer

**Decision**: FastAPI (Python) for the REST API serving dashboard queries, Nerve Center data, and admin operations.

**Justification**: Python aligns with the ML/AI stack (anomaly detection, Claude API integration). FastAPI provides automatic OpenAPI docs, async support for non-blocking ClickHouse queries, and Pydantic models for request/response validation. The team likely has Python experience from the connector work.

### Temporal — Workflow Orchestration

**Decision**: Temporal for automation workflows (confirmation sequences, missed call recovery, no-show re-engagement).

**Justification**: Automation workflows are long-running, stateful, and need retry logic. A confirmation sequence spans 48 hours (send text at -48h, -24h, -4h). If a text fails, retry with backoff. If a patient responds, branch the workflow. Temporal handles this natively with durable execution, visibility into running workflows, and automatic retry/timeout handling. Far more robust than cron-based approaches or simple job queues.

### Metabase — Stage 1 Dashboard

**Decision**: Metabase (open-source) as the Stage 1 executive dashboard.

**Justification**: Metabase connects to ClickHouse natively, has a drag-and-drop dashboard builder, supports drill-down and filtering, and can be deployed in <1 day. It's the fastest path to executives seeing real funnel data. We'll build 4-5 dashboards (network view, region view, practice detail, funnel stage deep-dive) in the first week after data starts flowing. Metabase is explicitly disposable — replaced by custom React dashboard in Stage 3.

### Twilio — Patient Communication

**Decision**: Twilio for SMS and voice outreach in the automation engine.

**Justification**: The automation engine sends appointment confirmations, missed call follow-ups, and no-show re-engagement messages. Twilio provides reliable SMS/MMS delivery, programmable voice (for automated callback), and delivery receipts that feed back into the funnel data lake. 10DLC registration handles compliance at scale.

### AWS — Infrastructure

**Decision**: AWS as the cloud platform (ECS for services, MSK for Kafka, RDS for PostgreSQL, S3 for cold storage).

**Justification**: Managed services reduce operational burden. ECS (Fargate) for containerized services eliminates cluster management. MSK for Kafka eliminates broker management. RDS for PostgreSQL handles backups, patching, failover. S3 for cold storage is essentially free at our volume. Total estimated monthly cost: $2,500-4,000 at Stage 1, scaling to $6,000-10,000 at full network deployment.

---

## 2. Database Schema

### ClickHouse — Funnel Events (Analytical Store)

```sql
CREATE TABLE funnel_events (
    event_id        UUID DEFAULT generateUUIDv4(),
    practice_id     String,
    region_id       String,
    timestamp       DateTime64(3, 'UTC'),
    funnel_stage    Enum8(
        'lead_generated' = 1,
        'lead_connected' = 2,
        'lead_booked' = 3,
        'patient_showed' = 4,
        'production_completed' = 5
    ),
    previous_stage  Nullable(Enum8(
        'lead_generated' = 1,
        'lead_connected' = 2,
        'lead_booked' = 3,
        'patient_showed' = 4,
        'production_completed' = 5
    )),
    source          LowCardinality(String),  -- 'phone_inbound', 'form_web', 'ad_google', etc.
    handler         LowCardinality(String),  -- 'human', 'ai_sierra', 'ai_agnes', 'automation'
    patient_hash    String,                   -- SHA-256 hash, not PII
    appointment_id  Nullable(String),
    service_type    LowCardinality(String),  -- 'cleaning', 'new_patient_exam', 'implant_consult', 'full_arch'
    value_estimate  Decimal64(2),
    metadata        String,                   -- JSON string for flexible fields
    ingested_at     DateTime64(3, 'UTC') DEFAULT now64(3)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (practice_id, timestamp, funnel_stage)
TTL timestamp + INTERVAL 2 YEAR
SETTINGS index_granularity = 8192;
```

### ClickHouse — Materialized Views

```sql
-- Practice daily funnel snapshot
CREATE MATERIALIZED VIEW mv_practice_daily_funnel
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(day)
ORDER BY (practice_id, day, funnel_stage)
AS SELECT
    practice_id,
    toDate(timestamp) AS day,
    funnel_stage,
    count() AS event_count,
    sum(value_estimate) AS total_value,
    uniqExact(patient_hash) AS unique_patients
FROM funnel_events
GROUP BY practice_id, day, funnel_stage;

-- Network hourly aggregate
CREATE MATERIALIZED VIEW mv_network_hourly
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, funnel_stage)
AS SELECT
    toStartOfHour(timestamp) AS hour,
    funnel_stage,
    count() AS event_count,
    sum(value_estimate) AS total_value,
    uniqExact(practice_id) AS active_practices
FROM funnel_events
GROUP BY hour, funnel_stage;

-- Conversion rates (stage-to-stage)
CREATE MATERIALIZED VIEW mv_conversion_daily
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(day)
ORDER BY (practice_id, day)
AS SELECT
    practice_id,
    toDate(timestamp) AS day,
    countIf(funnel_stage = 'lead_generated') AS leads,
    countIf(funnel_stage = 'lead_connected') AS connected,
    countIf(funnel_stage = 'lead_booked') AS booked,
    countIf(funnel_stage = 'patient_showed') AS showed,
    countIf(funnel_stage = 'production_completed') AS produced
FROM funnel_events
GROUP BY practice_id, day;
```

### PostgreSQL — Operational Tables

```sql
-- Practices
CREATE TABLE practices (
    id              TEXT PRIMARY KEY,
    name            TEXT NOT NULL,
    region_id       TEXT NOT NULL,
    platform        TEXT CHECK (platform IN ('SGA', 'Gen4', 'MODIS')),
    specialty       TEXT CHECK (specialty IN ('GP', 'Perio', 'FullArch', 'Pediatric', 'Multi')),
    address_json    JSONB,
    timezone        TEXT DEFAULT 'America/New_York',
    pms_vendor      TEXT,
    phone_vendor    TEXT,
    ai_agent_vendor TEXT,
    status          TEXT DEFAULT 'active',
    onboarded_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Connectors
CREATE TABLE connectors (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id     TEXT REFERENCES practices(id),
    connector_type  TEXT NOT NULL, -- 'pms', 'phone', 'ai_agent', 'ads', 'hr'
    vendor          TEXT NOT NULL, -- 'open_dental', 'ringcentral', 'sierra', etc.
    config_json     JSONB NOT NULL, -- encrypted API keys, endpoints, polling config
    status          TEXT DEFAULT 'pending', -- 'pending', 'active', 'error', 'disabled'
    last_sync_at    TIMESTAMPTZ,
    last_error      TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_connectors_practice ON connectors(practice_id);

-- AI Alerts
CREATE TABLE alerts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id     TEXT REFERENCES practices(id),
    alert_type      TEXT NOT NULL, -- 'phone_answer_rate', 'show_rate', 'production_gap', etc.
    severity        TEXT CHECK (severity IN ('critical', 'warning', 'info')),
    funnel_stage    TEXT,
    title           TEXT NOT NULL,
    description     TEXT NOT NULL,
    root_cause      TEXT,
    recommended_action TEXT,
    estimated_revenue_impact DECIMAL(10,2),
    status          TEXT DEFAULT 'open', -- 'open', 'acknowledged', 'resolved', 'dismissed'
    assigned_to     UUID REFERENCES users(id),
    resolved_at     TIMESTAMPTZ,
    outcome_notes   TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_alerts_practice ON alerts(practice_id);
CREATE INDEX idx_alerts_status ON alerts(status);

-- Actions (Nerve Center)
CREATE TABLE actions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    practice_id     TEXT REFERENCES practices(id),
    action_type     TEXT NOT NULL, -- 'callback', 'confirm', 'follow_up', 'outreach'
    title           TEXT NOT NULL,
    description     TEXT,
    patient_hash    TEXT,
    phone_number    TEXT,
    appointment_id  TEXT,
    script_template TEXT,
    estimated_value DECIMAL(10,2),
    priority        INT DEFAULT 0,
    status          TEXT DEFAULT 'pending', -- 'pending', 'completed', 'skipped', 'expired'
    completed_at    TIMESTAMPTZ,
    outcome         TEXT,
    generated_date  DATE NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_actions_practice_date ON actions(practice_id, generated_date);

-- Users
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email           TEXT UNIQUE NOT NULL,
    name            TEXT NOT NULL,
    role            TEXT CHECK (role IN ('exec', 'ops', 'office_manager', 'front_desk', 'admin')),
    practice_id     TEXT REFERENCES practices(id), -- NULL for HQ users
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Practice Baselines (for AI anomaly detection)
CREATE TABLE practice_baselines (
    practice_id     TEXT REFERENCES practices(id),
    metric_name     TEXT NOT NULL, -- 'phone_answer_rate', 'show_rate', 'daily_production', etc.
    baseline_value  DECIMAL(10,4),
    std_deviation   DECIMAL(10,4),
    sample_size     INT,
    calculated_at   TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (practice_id, metric_name)
);
```

---

## 3. API Endpoints

### Dashboard Queries

```
GET /api/v1/funnel/network
  → Returns network-wide funnel snapshot (5 stages, totals, conversion rates)
  → Query params: period=7d|30d|mtd, compare=wow|mom

GET /api/v1/funnel/region/{region_id}
  → Returns region funnel + practice heatmap
  → Query params: period, metric=answer_rate|show_rate|production

GET /api/v1/funnel/practice/{practice_id}
  → Returns practice funnel detail with all 5 stages, trends, intervention history
  → Query params: period

GET /api/v1/funnel/stage/{stage}
  → Returns deep-dive on one funnel stage across all practices (e.g., phone answer rates ranked)
  → Query params: sort=asc|desc, limit

GET /api/v1/alerts
  → Returns open alerts ranked by revenue impact
  → Query params: status=open|acknowledged|resolved, practice_id, severity

PATCH /api/v1/alerts/{alert_id}
  → Update alert status (acknowledge, resolve, dismiss)
  → Body: { status, outcome_notes, assigned_to }

GET /api/v1/summary/weekly
  → Returns AI-generated weekly summary (auto-generated, cached)

GET /api/v1/summary/daily
  → Returns AI-generated daily summary
```

### Nerve Center

```
GET /api/v1/nerve-center/{practice_id}/agenda
  → Returns today's prioritized action cards (3-5 items)
  → Response: { actions: [{ id, type, title, description, patient_phone, estimated_value, script }] }

POST /api/v1/nerve-center/{practice_id}/actions/{action_id}/complete
  → Mark an action as completed with outcome
  → Body: { outcome: "booked" | "no_answer" | "rescheduled" | "declined" }

GET /api/v1/nerve-center/{practice_id}/scorecard
  → Returns end-of-day scorecard
  → Response: { answer_rate, new_patients_booked, show_rate, production, target_pct, actions_completed }
```

### Admin

```
GET /api/v1/practices
  → List all practices with health scores and connector status

POST /api/v1/practices
  → Create a new practice

GET /api/v1/practices/{id}/connectors
  → List connectors for a practice with sync status

POST /api/v1/connectors/{id}/sync
  → Trigger manual sync for a connector

GET /api/v1/system/health
  → System health: Kafka lag, ClickHouse status, connector errors
```

---

## 4. Engineering Tasks

### Stage 1: Foundation (Month 1)

| # | Task | Description | Days | Dependencies | Role |
|---|------|------------|:----:|-------------|------|
| 1.1 | Infrastructure setup | AWS VPC, ECS cluster, RDS, MSK, S3 buckets, CI/CD pipeline | 3 | — | DevOps |
| 1.2 | ClickHouse deployment | Deploy ClickHouse on ECS, create schema + materialized views | 2 | 1.1 | Data Eng |
| 1.3 | PostgreSQL schema | Create operational tables, seed practice data for 10 pilots | 2 | 1.1 | Backend |
| 1.4 | Kafka topic setup | Create topics, configure retention, deploy consumer framework | 2 | 1.1 | Backend |
| 1.5 | Open Dental connector | Build Airbyte custom connector for Open Dental API | 5 | 1.4 | Backend |
| 1.6 | RingCentral connector | Build connector for call logs, answer rates | 4 | 1.4 | Backend |
| 1.7 | Google Ads connector | Configure Airbyte pre-built Google Ads connector | 2 | 1.4 | Data Eng |
| 1.8 | dbt models | Staging + mart models for practice snapshots, conversion rates | 3 | 1.2 | Data Eng |
| 1.9 | FastAPI scaffold | API framework, auth, ClickHouse + PostgreSQL clients | 3 | 1.2, 1.3 | Backend |
| 1.10 | Dashboard endpoints | Network, region, practice, stage query endpoints | 3 | 1.9, 1.8 | Backend |
| 1.11 | Metabase setup | Deploy Metabase, connect to ClickHouse, build 4 dashboards | 3 | 1.8 | Data Eng |
| 1.12 | Admin interface | Next.js app for practice + connector management | 4 | 1.9 | Frontend |
| 1.13 | Pilot onboarding | Configure connectors for 10 pilot practices, validate data | 3 | 1.5-1.7 | Backend + PM |
| 1.14 | Data quality framework | Dedup logic, late-arrival handling, timezone normalization | 3 | 1.8 | Data Eng |

### Stage 2: Intelligence (Month 2-3)

| # | Task | Description | Days | Dependencies | Role |
|---|------|------------|:----:|-------------|------|
| 2.1 | Baseline calculator | Python service: 90-day rolling baselines per practice per metric | 5 | Stage 1 | ML Eng |
| 2.2 | Anomaly detector | Statistical process control: flag >2σ deviations sustained >24h | 5 | 2.1 | ML Eng |
| 2.3 | Root cause correlator | Cross-source correlation rules (phone+HR, booking+PMS) | 5 | 2.2 | ML Eng |
| 2.4 | Action generator | Claude API integration for NL alert descriptions + recommended actions | 4 | 2.3 | ML Eng |
| 2.5 | Alert API + ops queue | Endpoints for alert management, assignment, resolution | 3 | 2.4 | Backend |
| 2.6 | Additional connectors | Dentrix, 8x8, Sierra/Agnes API, Meta Ads, iSolved | 10 | Stage 1 | Backend |
| 2.7 | Expand to 30 practices | Onboard 20 additional practices, validate data quality | 5 | 2.6 | PM + Backend |

### Stage 3: Nerve Center (Month 3-5)

| # | Task | Description | Days | Dependencies | Role |
|---|------|------------|:----:|-------------|------|
| 3.1 | Action generation service | Daily agenda builder: pull missed calls, unconfirmed appts, unscheduled TX | 5 | Stage 2 | Backend |
| 3.2 | Nerve Center API | Agenda, action completion, scorecard endpoints | 4 | 3.1 | Backend |
| 3.3 | Nerve Center UI | React Native/PWA: action cards, one-tap execution, scorecard | 10 | 3.2 | Frontend |
| 3.4 | Custom exec dashboard | Replace Metabase: React + Recharts + WebSocket real-time | 12 | Stage 2 | Frontend |
| 3.5 | Design system | Shared component library (Concierge + Nerve Center) | 5 | — | Frontend + Design |
| 3.6 | Role-based views | Office manager vs. front desk vs. exec view logic | 3 | 3.3 | Frontend |
| 3.7 | Expand to 50 practices | Onboard 20 more, iterate on Nerve Center based on feedback | 5 | 3.3 | PM |

### Stage 4: Automation (Month 5-8)

| # | Task | Description | Days | Dependencies | Role |
|---|------|------------|:----:|-------------|------|
| 4.1 | Temporal setup | Deploy Temporal server, define workflow patterns | 3 | — | DevOps |
| 4.2 | Confirmation workflows | Service-type-aware confirmation sequences via Twilio | 8 | 4.1 | Backend |
| 4.3 | Missed call recovery | Detect + route to AI agent or practice queue within 5 min | 5 | 4.1 | Backend |
| 4.4 | No-show re-engagement | Auto-detect + text within 30 min + phone follow-up queue | 5 | 4.1 | Backend |
| 4.5 | Learning loop | Weekly batch: measure intervention outcomes, update models | 5 | 4.2-4.4 | ML Eng |
| 4.6 | Concierge merge | Integrate Nerve Center into Practice Concierge app | 8 | 3.3 | Frontend |
| 4.7 | Network rollout | Onboard remaining 210+ practices | 15 | All above | PM + Backend |

---

## 5. Infrastructure Plan

### AWS Architecture

```
VPC (us-east-1)
├── Public Subnet
│   ├── ALB (Application Load Balancer) → FastAPI + Metabase + Admin UI
│   └── NAT Gateway
├── Private Subnet A
│   ├── ECS Fargate: FastAPI service (2 tasks, auto-scaling)
│   ├── ECS Fargate: Connector workers (1 task per connector type)
│   ├── ECS Fargate: AI Analysis Engine (1 task)
│   ├── ECS Fargate: Metabase (1 task)
│   └── ECS Fargate: Temporal workers (2 tasks)
├── Private Subnet B
│   ├── RDS PostgreSQL (db.r6g.large, Multi-AZ)
│   ├── ClickHouse on ECS (r6i.xlarge equivalent, EBS gp3)
│   └── ElastiCache Redis (cache.r6g.large)
├── MSK (Kafka)
│   └── 3 brokers (kafka.m5.large)
└── S3
    ├── funnel-os-cold-storage (Parquet archives)
    ├── funnel-os-backups
    └── funnel-os-static (frontend assets)
```

### Estimated Monthly Cost

| Service | Stage 1 (10 practices) | Stage 4 (260+ practices) |
|---------|:----------------------:|:------------------------:|
| ECS Fargate (all services) | $600 | $1,800 |
| RDS PostgreSQL | $350 | $700 |
| ClickHouse (ECS + EBS) | $400 | $800 |
| MSK (Kafka) | $400 | $600 |
| ElastiCache Redis | $200 | $200 |
| S3 + data transfer | $50 | $200 |
| ALB | $50 | $100 |
| Twilio (SMS/voice) | $0 | $2,000 |
| Airbyte (self-hosted) | $0 | $0 |
| Metabase (open-source) | $0 | N/A (replaced) |
| **Total** | **~$2,050/month** | **~$6,400/month** |

### CI/CD Pipeline

- GitHub Actions for CI (lint, test, build)
- AWS ECR for container images
- Terraform for infrastructure-as-code
- GitHub → ECR → ECS deploy on merge to main
- Staging environment mirrors production (smaller instances)

### Monitoring

- CloudWatch for infrastructure metrics (CPU, memory, network)
- Grafana + Prometheus for application metrics (API latency, Kafka lag, connector sync times)
- PagerDuty for critical alerts (Kafka consumer lag > 1hr, ClickHouse disk > 80%, connector sync failures)
- Sentry for application error tracking

---

## 6. Risk Register

| Risk | Probability | Impact | Mitigation |
|------|:-----------:|:------:|-----------|
| PMS API access denied (Dentrix requires partner agreement) | Medium | High | Start with Open Dental (open API). Pursue Dentrix partnership in parallel. Fallback: CSV export ingest. |
| Data quality issues across 260+ practices (inconsistent coding, missing fields) | High | High | Data quality framework in Stage 1. Validation rules per connector. Practice-specific mapping configs. Accept imperfection — 80% accuracy is useful, 0% is not. |
| AI false positive rate too high (alert fatigue) | Medium | High | Conservative initial thresholds (3σ, not 2σ). Human review of all alerts in Stage 2. Calibrate thresholds based on feedback before widening to more practices. |
| Practice staff don't adopt Nerve Center | Medium | High | Pilot with 5 most engaged practices first. Iterate on UX before rollout. Make it REDUCE work (missed call list replaces checking voicemail). Track time-to-complete as key metric. |
| ClickHouse operational complexity | Low | Medium | Use ClickHouse Cloud if self-hosted proves too complex. The schema and queries transfer directly. Budget for Cloud pricing ($500-1500/month). |
| Kafka operational overhead at low volume | Low | Low | Acceptable overhead for architectural benefit. If truly problematic, replace with Redis Streams for Stage 1 and migrate to Kafka when volume justifies it. |
| HIPAA compliance concerns with patient data in cloud | Medium | High | No PII in ClickHouse — only hashed patient IDs. PII stays in PMS. Funnel events contain appointment IDs and hashed identifiers, not names/SSNs. AWS BAA covers RDS and S3. |
| Scope creep from 15 planned connectors | High | Medium | Stage 1 is 3 connectors, period. Additional connectors are gated by Stage 2 validation. Each connector must prove data quality before adding the next. |
| Temporal learning curve for team | Medium | Low | Temporal has excellent documentation and Go/Python SDKs. Assign one engineer to become the Temporal expert. Simple workflows first (confirmation text) before complex ones (multi-step recovery). |
| Metabase limitations frustrate executives | Medium | Low | Set expectations: Stage 1 dashboard is a "prototype with real data." Custom dashboard replaces it in Stage 3. Collect executive feedback on Metabase to inform custom dashboard design. |
