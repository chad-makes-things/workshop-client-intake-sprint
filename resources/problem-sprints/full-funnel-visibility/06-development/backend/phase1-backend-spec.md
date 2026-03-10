# Phase 1 Backend Specification — Funnel OS

**Sprint**: Full Funnel Visibility
**Phase**: 1 (Month 1)
**Scope**: 3 integration connectors (PMS, Phone, Lead Source) + Funnel Data Lake + Executive Dashboard via Metabase
**Pilot**: 10 practices
**Date**: 2026-03-10

---

## 1. Service Architecture

Phase 1 deploys four backend services, all containerized on AWS ECS. Services communicate via Kafka for event data and synchronous HTTP for admin/query operations.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS ECS Cluster                              │
│                                                                     │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  Connector Svc   │───▶│   AWS MSK         │                      │
│  │  (Airbyte +      │    │   (Kafka)         │                      │
│  │   custom Python)  │    │                   │                      │
│  └──────────────────┘    │  funnel.events.raw│                      │
│                          │  connector.status  │                      │
│                          │  connector.dlq     │                      │
│                          └────────┬──────────┘                      │
│                                   │                                  │
│                                   ▼                                  │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  Event Pipeline  │◀───│   Kafka Consumer  │                      │
│  │  (transforms +   │    │   Groups          │                      │
│  │   writes to CH)  │    └──────────────────┘                      │
│  └────────┬─────────┘                                               │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  ClickHouse      │◀───│  dbt (scheduled)  │                      │
│  │  (hot store)     │    │  transforms       │                      │
│  └────────┬─────────┘    └──────────────────┘                      │
│           │                                                         │
│           ▼                                                         │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  Query Service   │    │  Admin Service    │                      │
│  │  (FastAPI)       │    │  (FastAPI)        │                      │
│  │  :8001           │    │  :8002            │                      │
│  └────────┬─────────┘    └────────┬─────────┘                      │
│           │                       │                                  │
│           ▼                       ▼                                  │
│  ┌──────────────────┐    ┌──────────────────┐                      │
│  │  Metabase        │    │  PostgreSQL (RDS) │                      │
│  │  (dashboard)     │    │  (operational)    │                      │
│  │  :3000           │    │  :5432            │                      │
│  └──────────────────┘    └──────────────────┘                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Service Inventory

| Service | Port | Language | Responsibility |
|---------|------|----------|----------------|
| **Connector Service** | 8003 | Python 3.12 | Manages Airbyte sync jobs + 3 custom Python connectors. Produces raw events to Kafka. Exposes sync-trigger and health-check endpoints. |
| **Event Pipeline** | — (no HTTP) | Python 3.12 | Kafka consumer that validates, deduplicates, enriches, and writes funnel events to ClickHouse. Writes failed events to the dead-letter topic. |
| **Query Service** | 8001 | Python 3.12 (FastAPI) | Serves dashboard-oriented read queries against ClickHouse. Metabase connects here OR directly to ClickHouse (both paths supported). |
| **Admin Service** | 8002 | Python 3.12 (FastAPI) | CRUD for practices, connector configs, users. Reads/writes PostgreSQL. Triggers connector syncs via internal HTTP to Connector Service. |

### Inter-Service Communication

| From | To | Protocol | Purpose |
|------|----|----------|---------|
| Connector Service | Kafka | Kafka Producer | Raw funnel events |
| Event Pipeline | Kafka | Kafka Consumer | Consume raw events |
| Event Pipeline | ClickHouse | Native TCP (port 9000) | Insert validated events |
| Event Pipeline | Kafka | Kafka Producer | Dead-letter queue writes |
| Query Service | ClickHouse | HTTP interface (port 8123) | Read queries |
| Admin Service | PostgreSQL | psycopg3 | Operational data CRUD |
| Admin Service | Connector Service | HTTP (internal) | Trigger sync, get status |
| Metabase | ClickHouse | ClickHouse JDBC driver | Direct analytical queries |

### Shared Libraries

Create a shared Python package `funnel_os_common` installed by all services:

```
funnel_os_common/
  models/
    events.py          # Pydantic models for FunnelEvent, ConnectorStatus
    enums.py           # FunnelStage, EventSource, HandlerType enums
  kafka/
    producer.py        # Configured KafkaProducer with serialization
    consumer.py        # Configured KafkaConsumer with deserialization
  config.py            # Environment-based config (pydantic-settings)
  logging.py           # Structured JSON logging
```

---

## 2. Connector Details

### 2.1 PMS Connector — Open Dental

Open Dental exposes a REST API (CustomerKey + DeveloperKey authentication). This is the most API-friendly PMS in the SGA portfolio and the Phase 1 target.

**Data to Pull**

| Open Dental Endpoint | Data Extracted | Funnel Stage Mapping |
|---------------------|----------------|---------------------|
| `GET /patients` (new, modified since last sync) | New patient records, demographics | `lead_generated` (when source is external referral or web form) |
| `GET /appointments` | Appointment status, scheduled time, provider, procedure codes | `lead_booked` (scheduled), `patient_showed` (completed/checked-in) |
| `GET /procedurelogs` | Completed procedures, fee amounts, provider | `production_completed` (with `value_estimate` = fee) |
| `GET /recalls` | Recall status, due dates | Metadata enrichment for patient lifecycle |
| `GET /referrals` | Referral source tracking | `source` field mapping |

**Polling Configuration**

| Sync Type | Frequency | Strategy |
|-----------|-----------|----------|
| Incremental appointments | Every 5 minutes | `GET /appointments?DateTStamp={last_sync}` — uses server-side modified-since filter |
| Incremental procedures | Every 15 minutes | `GET /procedurelogs?DateTStamp={last_sync}` |
| New patients | Every 15 minutes | `GET /patients?DateTStamp={last_sync}` |
| Full reconciliation | Daily at 02:00 UTC | Full pull of last 48 hours, deduplicate against existing events |

**Data Mapping Logic**

```python
# Appointment status → funnel stage mapping
APPOINTMENT_STATUS_MAP = {
    1: None,                    # Scheduled (no funnel event yet — booked already captured)
    2: "patient_showed",        # Complete
    3: None,                    # UnschedList
    4: None,                    # ASAP list
    5: "patient_showed",        # Broken/Missed → still generates event with metadata.no_show=true
    6: None,                    # Planned
}

def map_appointment_to_funnel_event(apt: dict, practice_id: str) -> FunnelEvent | None:
    """
    Produces up to 2 events per appointment lifecycle:
    1. lead_booked — when appointment first created (AptStatus=1, first seen)
    2. patient_showed — when appointment completed (AptStatus=2)
       OR patient_showed with metadata.no_show=true (AptStatus=5)
    """
    stage = APPOINTMENT_STATUS_MAP.get(apt["AptStatus"])
    if stage is None:
        return None

    return FunnelEvent(
        event_id=generate_deterministic_uuid(practice_id, apt["AptNum"], stage),
        practice_id=practice_id,
        timestamp=parse_opendental_datetime(apt["AptDateTime"]),
        funnel_stage=stage,
        previous_stage="lead_booked" if stage == "patient_showed" else "lead_connected",
        source=resolve_source(apt),  # maps referral source to enum
        handler=resolve_handler(apt),
        patient_id=hash_patient_id(apt["PatNum"]),
        appointment_id=str(apt["AptNum"]),
        value_estimate=Decimal(str(apt.get("ProcFee", 0))),
        metadata={
            "provider": apt.get("ProvNum"),
            "procedure_codes": apt.get("ProcDescript", ""),
            "operatory": apt.get("Op"),
            "no_show": apt["AptStatus"] == 5,
            "confirmation_status": apt.get("Confirmed"),
        },
    )

def map_procedure_to_funnel_event(proc: dict, practice_id: str) -> FunnelEvent:
    """Completed procedure → production_completed event."""
    return FunnelEvent(
        event_id=generate_deterministic_uuid(practice_id, proc["ProcNum"], "production_completed"),
        practice_id=practice_id,
        timestamp=parse_opendental_datetime(proc["ProcDate"]),
        funnel_stage="production_completed",
        previous_stage="patient_showed",
        source="pms_opendental",
        handler="human",
        patient_id=hash_patient_id(proc["PatNum"]),
        appointment_id=str(proc.get("AptNum")),
        value_estimate=Decimal(str(proc["ProcFee"])),
        metadata={
            "procedure_code": proc["CodeNum"],
            "description": proc.get("Descript", ""),
            "provider": proc.get("ProvNum"),
            "tooth_num": proc.get("ToothNum"),
        },
    )
```

**Authentication**

```python
# Per-practice credentials stored in PostgreSQL (encrypted at rest via AWS RDS)
class PMSConnectorConfig:
    practice_id: str
    api_base_url: str           # e.g., "https://api.opendental.com/api/v1"
    customer_key: str           # encrypted in DB, decrypted at runtime
    developer_key: str          # shared across all Open Dental connections
    last_sync_timestamp: datetime
    sync_enabled: bool
    polling_interval_seconds: int  # default 300
```

---

### 2.2 Phone Connector — RingCentral

RingCentral provides a REST API with OAuth 2.0 and webhook support. Phase 1 uses polling (webhooks added in Phase 2 for sub-minute latency).

**Data to Pull**

| RingCentral Endpoint | Data Extracted | Funnel Stage Mapping |
|---------------------|----------------|---------------------|
| `GET /restapi/v1.0/account/{id}/call-log` | Call records: direction, duration, result, from/to numbers | `lead_generated` (inbound answered), `lead_connected` (call resulted in conversation) |
| `GET /restapi/v1.0/account/{id}/call-log/{id}/recording` | Call recording metadata (not content) | Metadata enrichment |
| `GET /restapi/v1.0/account/{id}/extension` | Extension list per practice | Practice-to-number mapping |

**Polling Configuration**

| Sync Type | Frequency | Strategy |
|-----------|-----------|----------|
| Call log incremental | Every 5 minutes | `dateFrom={last_sync}&dateTo=now&perPage=250` with pagination |
| Full reconciliation | Daily at 03:00 UTC | Re-pull last 48 hours |

**Data Mapping Logic**

```python
# Call result → funnel stage mapping
CALL_RESULT_MAP = {
    "Accepted": "lead_connected",
    "Voicemail": "lead_generated",       # lead exists but not connected
    "Missed": "lead_generated",          # lead exists but not connected
    "Hang Up": "lead_generated",
    "No Answer": "lead_generated",
    "Busy": "lead_generated",
    "Reply": "lead_connected",
}

def map_call_to_funnel_event(call: dict, practice_id: str) -> FunnelEvent | None:
    """
    Only maps INBOUND calls. Outbound calls are not lead-generation events.
    Inbound answered → lead_connected (someone engaged with the caller).
    Inbound missed/voicemail → lead_generated (a lead appeared but wasn't connected).
    """
    if call["direction"] != "Inbound":
        return None

    stage = CALL_RESULT_MAP.get(call["result"])
    if stage is None:
        return None

    return FunnelEvent(
        event_id=generate_deterministic_uuid(practice_id, call["id"], stage),
        practice_id=practice_id,
        timestamp=parse_iso_datetime(call["startTime"]),
        funnel_stage=stage,
        previous_stage=None if stage == "lead_generated" else "lead_generated",
        source="phone_inbound",
        handler="human",  # Phase 1 default; Phase 2 cross-refs AI agent logs
        patient_id=hash_phone_number(call["from"]["phoneNumber"]),
        appointment_id=None,  # Linked to PMS appointment in dbt join layer
        value_estimate=Decimal("0"),  # No value estimate at call stage
        metadata={
            "call_duration_seconds": call.get("duration", 0),
            "call_result": call["result"],
            "to_number": call["to"]["phoneNumber"],
            "from_number_masked": mask_phone(call["from"]["phoneNumber"]),
            "has_recording": call.get("recording") is not None,
            "queue_name": call.get("queue", {}).get("name"),
        },
    )
```

**Key Metrics Derived from Phone Data**

| Metric | Calculation | Used In |
|--------|-------------|---------|
| Answer rate | `count(lead_connected) / count(lead_generated) WHERE source='phone_inbound'` | Executive Dashboard, practice health score |
| Avg call duration | `avg(metadata.call_duration_seconds) WHERE funnel_stage='lead_connected'` | Practice detail view |
| Missed call volume | `count(*) WHERE funnel_stage='lead_generated' AND metadata.call_result IN ('Missed','No Answer')` | Alert triggers |
| Peak call times | `hour(timestamp) GROUP BY hour` | Staffing recommendations (Phase 2) |

**Authentication**

```python
class PhoneConnectorConfig:
    practice_id: str
    account_id: str              # RingCentral account ID
    oauth_client_id: str
    oauth_client_secret: str     # encrypted in DB
    oauth_refresh_token: str     # encrypted in DB; auto-refreshed
    jwt_token: str               # service-to-service auth (preferred)
    last_sync_timestamp: datetime
    sync_enabled: bool
    extension_filter: list[str]  # only sync calls to these extensions (practice main lines)
```

---

### 2.3 Lead Source Connector — Google Ads

Google Ads API (v17) provides campaign, ad group, and keyword-level performance data. Phase 1 pulls daily aggregates and click-level conversion data.

**Data to Pull**

| Google Ads Resource | Data Extracted | Funnel Stage Mapping |
|--------------------|----------------|---------------------|
| `campaign` performance | Impressions, clicks, cost, conversions by campaign | `lead_generated` (one event per click with source attribution) |
| `ad_group` performance | Same metrics at ad group level | Metadata enrichment |
| `click_view` (last 90 days) | Individual click records with gclid | Links to website form submissions |
| `conversion_action` | Conversion definitions and values | Value estimation for leads |
| `customer` | Account structure | Practice-to-account mapping |

**Polling Configuration**

| Sync Type | Frequency | Strategy |
|-----------|-----------|----------|
| Campaign daily stats | Every 6 hours | Pull yesterday + today stats (Google Ads data finalizes ~3 hours after midnight) |
| Click-level data | Every 6 hours | `click_view` for last 7 days, deduplicate |
| Full historical backfill | Once at onboarding | Pull last 90 days of campaign data for baseline |

**Data Mapping Logic**

```python
def map_campaign_stats_to_funnel_events(
    stats: dict, practice_id: str, date: str
) -> list[FunnelEvent]:
    """
    Google Ads campaign stats map to aggregated lead_generated events.
    We create one event per click (not per impression) because a click
    represents a real lead entering the funnel.

    For Phase 1, we use daily aggregates rather than individual click events
    when click_view data is not available (some account types).
    """
    events = []

    # One aggregate event per campaign per day
    cost_per_click = (
        Decimal(str(stats["cost_micros"])) / Decimal("1000000") / max(stats["clicks"], 1)
    )

    for i in range(stats["clicks"]):
        events.append(FunnelEvent(
            event_id=generate_deterministic_uuid(
                practice_id, stats["campaign_id"], date, str(i)
            ),
            practice_id=practice_id,
            timestamp=parse_date_to_noon_utc(date),  # daily granularity
            funnel_stage="lead_generated",
            previous_stage=None,
            source="ad_google",
            handler="automation",
            patient_id=None,  # no patient identity at ad click stage
            appointment_id=None,
            value_estimate=cost_per_click,  # cost as negative value? Or estimated lead value
            metadata={
                "campaign_id": stats["campaign_id"],
                "campaign_name": stats["campaign_name"],
                "ad_group_id": stats.get("ad_group_id"),
                "cost_micros": stats["cost_micros"],
                "impressions": stats["impressions"],
                "conversions": stats.get("conversions", 0),
                "conversion_value": stats.get("conversion_value", 0),
                "click_type": "search",  # vs display, video
                "date": date,
            },
        ))

    return events

def map_click_to_funnel_event(click: dict, practice_id: str) -> FunnelEvent:
    """Individual click-level event when click_view data is available."""
    return FunnelEvent(
        event_id=generate_deterministic_uuid(practice_id, click["gclid"]),
        practice_id=practice_id,
        timestamp=parse_iso_datetime(click["click_date_time"]),
        funnel_stage="lead_generated",
        previous_stage=None,
        source="ad_google",
        handler="automation",
        patient_id=None,
        appointment_id=None,
        value_estimate=Decimal(str(click.get("cost_micros", 0))) / Decimal("1000000"),
        metadata={
            "gclid": click["gclid"],
            "campaign_id": click["campaign_id"],
            "ad_group_id": click["ad_group_id"],
            "keyword_text": click.get("keyword_text"),
            "match_type": click.get("match_type"),
            "device": click.get("device"),
            "geo_target": click.get("geo_target_city"),
        },
    )
```

**Key Metrics Derived from Google Ads Data**

| Metric | Calculation | Used In |
|--------|-------------|---------|
| Cost per lead | `SUM(metadata.cost_micros) / COUNT(*) WHERE source='ad_google'` | Executive Dashboard, ROI views |
| Lead volume by campaign | `COUNT(*) GROUP BY metadata.campaign_name` | Campaign performance |
| Cost per booked appointment | Joined with downstream `lead_booked` events via patient_id/gclid | Attribution reporting |
| ROAS | `SUM(production_completed.value_estimate) / SUM(ad cost)` for attributed patients | Network-level ROI |

**Authentication**

```python
class LeadSourceConnectorConfig:
    practice_id: str
    google_ads_customer_id: str   # 10-digit, no dashes
    manager_account_id: str       # MCC account for SGA-wide access
    oauth_client_id: str
    oauth_client_secret: str      # encrypted
    oauth_refresh_token: str      # encrypted
    developer_token: str          # Google Ads API developer token
    last_sync_timestamp: datetime
    sync_enabled: bool
```

---

## 3. Kafka Topic Design

### Topics

| Topic | Key | Value | Partitions | Retention |
|-------|-----|-------|------------|-----------|
| `funnel.events.raw` | `practice_id` | JSON-serialized `FunnelEvent` | 10 | 7 days |
| `funnel.events.validated` | `practice_id` | JSON-serialized `FunnelEvent` (post-validation) | 10 | 7 days |
| `funnel.events.dlq` | `practice_id` | JSON with `original_event` + `error_reason` + `failed_at` | 3 | 30 days |
| `connector.status` | `connector_id` | JSON with sync status, records processed, errors | 3 | 3 days |

### Partitioning Strategy

- **Partition key**: `practice_id` for all event topics
- **Why**: Ensures all events for a single practice land on the same partition, maintaining per-practice ordering. This matters because funnel stage transitions must be processed in order (e.g., `lead_booked` must arrive before `patient_showed` for deduplication logic).
- **10 partitions** for `funnel.events.raw` and `funnel.events.validated`:
  - Phase 1 has 10 practices, so each practice gets its own partition
  - Scales to 260+ practices with consistent hashing (partitions can be increased later, but 10 is sufficient through Phase 2)
- **3 partitions** for low-volume topics (DLQ, status)

### Consumer Groups

| Consumer Group | Reads From | Purpose | Instances |
|---------------|------------|---------|-----------|
| `event-pipeline-validator` | `funnel.events.raw` | Validates schema, deduplicates, enriches, writes to `funnel.events.validated` | 2 (for failover) |
| `event-pipeline-writer` | `funnel.events.validated` | Inserts into ClickHouse | 2 |
| `event-pipeline-s3-archiver` | `funnel.events.validated` | Writes to S3 as Parquet (cold storage) | 1 |
| `connector-monitor` | `connector.status` | Updates PostgreSQL with connector health, triggers alerts | 1 |

### Kafka Configuration (AWS MSK)

```properties
# Broker config
num.partitions=10
default.replication.factor=3
min.insync.replicas=2
log.retention.hours=168           # 7 days default
log.retention.bytes=-1            # no size limit

# Producer config (Connector Service)
acks=all                          # durability over throughput
retries=5
retry.backoff.ms=1000
max.in.flight.requests.per.connection=1  # strict ordering
enable.idempotence=true
compression.type=lz4

# Consumer config (Event Pipeline)
auto.offset.reset=earliest
enable.auto.commit=false          # manual commit after processing
max.poll.records=500
max.poll.interval.ms=300000       # 5 min max processing time per batch
session.timeout.ms=30000
```

### Message Schema (Avro-style, JSON in Phase 1)

```json
{
  "topic": "funnel.events.raw",
  "key": "garner-dental-woodstock",
  "value": {
    "event_id": "550e8400-e29b-41d4-a716-446655440000",
    "practice_id": "garner-dental-woodstock",
    "timestamp": "2026-03-10T14:32:00Z",
    "funnel_stage": "lead_connected",
    "previous_stage": "lead_generated",
    "source": "phone_inbound",
    "handler": "human",
    "patient_id": "sha256:a1b2c3d4...",
    "appointment_id": null,
    "value_estimate": 0.00,
    "metadata": {
      "call_duration_seconds": 240,
      "call_result": "Accepted"
    },
    "_connector": "ringcentral",
    "_connector_version": "1.0.0",
    "_ingested_at": "2026-03-10T14:32:05Z"
  },
  "headers": {
    "source_connector": "ringcentral",
    "practice_id": "garner-dental-woodstock",
    "schema_version": "1"
  }
}
```

---

## 4. ClickHouse Schema

### Funnel Events Table (Primary)

```sql
CREATE TABLE funnel_os.funnel_events
(
    event_id         UUID,
    practice_id      LowCardinality(String),
    timestamp        DateTime64(3, 'UTC'),
    funnel_stage     Enum8(
                       'lead_generated' = 1,
                       'lead_connected' = 2,
                       'lead_booked' = 3,
                       'patient_showed' = 4,
                       'production_completed' = 5
                     ),
    previous_stage   Nullable(Enum8(
                       'lead_generated' = 1,
                       'lead_connected' = 2,
                       'lead_booked' = 3,
                       'patient_showed' = 4,
                       'production_completed' = 5
                     )),
    source           LowCardinality(String),
    handler          Enum8(
                       'human' = 1,
                       'ai_sierra' = 2,
                       'ai_agnes' = 3,
                       'automation' = 4,
                       'unknown' = 5
                     ),
    patient_id       Nullable(String),
    appointment_id   Nullable(String),
    value_estimate   Decimal64(2),
    metadata         String,  -- JSON stored as String, queried with JSONExtract*
    _connector       LowCardinality(String),
    _ingested_at     DateTime64(3, 'UTC') DEFAULT now64(3),
    _deduplication_key String MATERIALIZED concat(toString(event_id), '-', practice_id)
)
ENGINE = ReplacingMergeTree(_ingested_at)
PARTITION BY toYYYYMM(timestamp)
ORDER BY (practice_id, timestamp, funnel_stage, event_id)
TTL timestamp + INTERVAL 13 MONTH  -- hot store keeps 13 months; older data in S3
SETTINGS index_granularity = 8192;
```

**Why ReplacingMergeTree**: Handles late-arriving duplicates. If the same `event_id` is inserted twice, ClickHouse keeps only the row with the latest `_ingested_at` after background merges. For real-time queries, we add `FINAL` or use the deduplication window approach (see Data Quality section).

### Materialized Views

#### Practice Daily Snapshot

```sql
CREATE MATERIALIZED VIEW funnel_os.mv_practice_daily_snapshot
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(snapshot_date)
ORDER BY (practice_id, snapshot_date, funnel_stage)
AS
SELECT
    practice_id,
    toDate(timestamp) AS snapshot_date,
    funnel_stage,
    source,
    handler,
    count() AS event_count,
    sum(value_estimate) AS total_value,
    uniqExact(patient_id) AS unique_patients,
    avg(toFloat64(JSONExtractInt(metadata, 'call_duration_seconds'))) AS avg_call_duration
FROM funnel_os.funnel_events
GROUP BY
    practice_id,
    snapshot_date,
    funnel_stage,
    source,
    handler;
```

#### Network Aggregate (Hourly)

```sql
CREATE MATERIALIZED VIEW funnel_os.mv_network_hourly
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, funnel_stage)
AS
SELECT
    toStartOfHour(timestamp) AS hour,
    funnel_stage,
    count() AS event_count,
    sum(value_estimate) AS total_value,
    uniqExact(practice_id) AS active_practices,
    uniqExact(patient_id) AS unique_patients
FROM funnel_os.funnel_events
GROUP BY
    hour,
    funnel_stage;
```

#### Conversion Rate View (Daily per Practice)

```sql
CREATE MATERIALIZED VIEW funnel_os.mv_conversion_rates_daily
ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(snapshot_date)
ORDER BY (practice_id, snapshot_date)
AS
SELECT
    practice_id,
    toDate(timestamp) AS snapshot_date,
    countIfState(funnel_stage = 'lead_generated') AS leads_generated,
    countIfState(funnel_stage = 'lead_connected') AS leads_connected,
    countIfState(funnel_stage = 'lead_booked') AS leads_booked,
    countIfState(funnel_stage = 'patient_showed') AS patients_showed,
    countIfState(funnel_stage = 'production_completed') AS production_completed,
    sumIfState(value_estimate, funnel_stage = 'production_completed') AS total_production
FROM funnel_os.funnel_events
GROUP BY
    practice_id,
    snapshot_date;
```

#### Source Attribution View

```sql
CREATE MATERIALIZED VIEW funnel_os.mv_source_attribution
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(snapshot_date)
ORDER BY (practice_id, snapshot_date, source, funnel_stage)
AS
SELECT
    practice_id,
    toDate(timestamp) AS snapshot_date,
    source,
    funnel_stage,
    count() AS event_count,
    sum(value_estimate) AS total_value,
    uniqExact(patient_id) AS unique_patients
FROM funnel_os.funnel_events
GROUP BY
    practice_id,
    snapshot_date,
    source,
    funnel_stage;
```

### PostgreSQL Schema (Operational)

```sql
-- Practices
CREATE TABLE practices (
    id              VARCHAR(100) PRIMARY KEY,  -- slug: "garner-dental-woodstock"
    name            VARCHAR(255) NOT NULL,
    region          VARCHAR(100),
    state           VARCHAR(2),
    city            VARCHAR(100),
    specialty       VARCHAR(100) DEFAULT 'general',
    chair_count     INTEGER,
    pms_type        VARCHAR(50),               -- 'opendental', 'dentrix', 'eaglesoft'
    timezone        VARCHAR(50) NOT NULL,       -- 'America/New_York'
    onboarded_at    TIMESTAMP WITH TIME ZONE,
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Connector configurations (one per practice per source)
CREATE TABLE connector_configs (
    id              SERIAL PRIMARY KEY,
    practice_id     VARCHAR(100) REFERENCES practices(id),
    connector_type  VARCHAR(50) NOT NULL,      -- 'opendental', 'ringcentral', 'google_ads'
    config_encrypted JSONB NOT NULL,           -- encrypted credentials + settings
    sync_enabled    BOOLEAN DEFAULT true,
    polling_interval_seconds INTEGER DEFAULT 300,
    last_sync_at    TIMESTAMP WITH TIME ZONE,
    last_sync_status VARCHAR(20),              -- 'success', 'partial', 'failed'
    last_sync_records INTEGER DEFAULT 0,
    last_error      TEXT,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(practice_id, connector_type)
);

-- Sync history (audit trail)
CREATE TABLE sync_history (
    id              SERIAL PRIMARY KEY,
    connector_config_id INTEGER REFERENCES connector_configs(id),
    practice_id     VARCHAR(100) REFERENCES practices(id),
    connector_type  VARCHAR(50) NOT NULL,
    started_at      TIMESTAMP WITH TIME ZONE NOT NULL,
    completed_at    TIMESTAMP WITH TIME ZONE,
    status          VARCHAR(20) NOT NULL,      -- 'running', 'success', 'partial', 'failed'
    records_fetched INTEGER DEFAULT 0,
    records_produced INTEGER DEFAULT 0,       -- events sent to Kafka
    records_failed  INTEGER DEFAULT 0,
    error_summary   TEXT,
    duration_ms     INTEGER
);
CREATE INDEX idx_sync_history_practice ON sync_history(practice_id, started_at DESC);

-- Users (Phase 1: HQ admins only)
CREATE TABLE users (
    id              SERIAL PRIMARY KEY,
    email           VARCHAR(255) UNIQUE NOT NULL,
    name            VARCHAR(255) NOT NULL,
    role            VARCHAR(50) NOT NULL,      -- 'admin', 'executive', 'ops'
    is_active       BOOLEAN DEFAULT true,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Practice health scores (computed daily by dbt, stored here for API)
CREATE TABLE practice_health_scores (
    practice_id     VARCHAR(100) REFERENCES practices(id),
    score_date      DATE NOT NULL,
    overall_score   DECIMAL(5,2),              -- 0-100
    phone_answer_rate DECIMAL(5,4),
    booking_rate    DECIMAL(5,4),
    show_rate       DECIMAL(5,4),
    production_vs_target DECIMAL(5,4),
    lead_volume_trend VARCHAR(10),             -- 'up', 'down', 'stable'
    computed_at     TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (practice_id, score_date)
);
```

---

## 5. dbt Models

### Project Structure

```
dbt_funnel_os/
  models/
    staging/
      stg_funnel_events.sql
      stg_funnel_events.yml
      stg_practices.sql
    intermediate/
      int_daily_practice_funnel.sql
      int_conversion_rates.sql
      int_source_attribution.sql
      int_phone_metrics.sql
    marts/
      mrt_executive_dashboard.sql
      mrt_practice_detail.sql
      mrt_network_trends.sql
      mrt_practice_health_scores.sql
      mrt_source_roi.sql
  macros/
    funnel_stage_order.sql
    safe_divide.sql
  seeds/
    funnel_stages.csv
    source_categories.csv
```

### Staging Layer

**`stg_funnel_events`** — Cleans raw ClickHouse events, applies type casting, filters test data.

```sql
-- stg_funnel_events.sql
-- Materialization: view (no additional storage, reads from ClickHouse funnel_events)
-- Refresh: real-time (it's a view)

SELECT
    event_id,
    practice_id,
    timestamp,
    funnel_stage,
    previous_stage,
    source,
    handler,
    patient_id,
    appointment_id,
    value_estimate,
    JSONExtractString(metadata, 'call_duration_seconds') AS call_duration_seconds,
    JSONExtractString(metadata, 'call_result') AS call_result,
    JSONExtractString(metadata, 'campaign_name') AS campaign_name,
    JSONExtractString(metadata, 'procedure_code') AS procedure_code,
    JSONExtractBool(metadata, 'no_show') AS is_no_show,
    _connector,
    _ingested_at,
    toDate(timestamp) AS event_date,
    toStartOfWeek(timestamp) AS event_week,
    toStartOfMonth(timestamp) AS event_month
FROM funnel_os.funnel_events
WHERE practice_id NOT LIKE 'test-%'
```

**`stg_practices`** — Reads practice dimension from PostgreSQL (via ClickHouse external dictionary or dbt source).

```sql
-- stg_practices.sql
-- Source: PostgreSQL practices table (via dbt source or ClickHouse JDBC dictionary)
-- Materialization: table, refreshed daily

SELECT
    id AS practice_id,
    name AS practice_name,
    region,
    state,
    city,
    specialty,
    chair_count,
    pms_type,
    timezone,
    is_active
FROM {{ source('postgres', 'practices') }}
WHERE is_active = true
```

### Intermediate Layer

**`int_daily_practice_funnel`** — Daily funnel counts per practice, per stage.

```sql
-- int_daily_practice_funnel.sql
-- Materialization: table (ClickHouse MergeTree)
-- Refresh: hourly via dbt Cloud / Airflow

SELECT
    practice_id,
    event_date,
    funnel_stage,
    COUNT(*) AS event_count,
    COUNT(DISTINCT patient_id) AS unique_patients,
    SUM(value_estimate) AS total_value
FROM {{ ref('stg_funnel_events') }}
GROUP BY practice_id, event_date, funnel_stage
```

**`int_conversion_rates`** — Stage-to-stage conversion rates per practice per week.

```sql
-- int_conversion_rates.sql
-- Materialization: table
-- Refresh: hourly

WITH stage_counts AS (
    SELECT
        practice_id,
        event_week,
        countIf(funnel_stage = 'lead_generated') AS leads_generated,
        countIf(funnel_stage = 'lead_connected') AS leads_connected,
        countIf(funnel_stage = 'lead_booked') AS leads_booked,
        countIf(funnel_stage = 'patient_showed') AS patients_showed,
        countIf(funnel_stage = 'production_completed') AS production_completed,
        sumIf(value_estimate, funnel_stage = 'production_completed') AS total_production
    FROM {{ ref('stg_funnel_events') }}
    GROUP BY practice_id, event_week
)
SELECT
    *,
    {{ safe_divide('leads_connected', 'leads_generated') }} AS connect_rate,
    {{ safe_divide('leads_booked', 'leads_connected') }} AS booking_rate,
    {{ safe_divide('patients_showed', 'leads_booked') }} AS show_rate,
    {{ safe_divide('production_completed', 'patients_showed') }} AS production_rate,
    {{ safe_divide('leads_booked', 'leads_generated') }} AS lead_to_book_rate
FROM stage_counts
```

**`int_source_attribution`** — Funnel performance broken down by lead source.

```sql
-- int_source_attribution.sql
-- Materialization: table
-- Refresh: hourly

SELECT
    practice_id,
    event_week,
    source,
    funnel_stage,
    COUNT(*) AS event_count,
    SUM(value_estimate) AS total_value,
    COUNT(DISTINCT patient_id) AS unique_patients
FROM {{ ref('stg_funnel_events') }}
GROUP BY practice_id, event_week, source, funnel_stage
```

**`int_phone_metrics`** — Phone-specific metrics (answer rate, call duration, missed calls).

```sql
-- int_phone_metrics.sql
-- Materialization: table
-- Refresh: hourly

SELECT
    practice_id,
    event_date,
    COUNT(*) AS total_inbound_calls,
    countIf(funnel_stage = 'lead_connected') AS answered_calls,
    countIf(funnel_stage = 'lead_generated' AND call_result IN ('Missed', 'No Answer')) AS missed_calls,
    countIf(funnel_stage = 'lead_generated' AND call_result = 'Voicemail') AS voicemail_calls,
    {{ safe_divide('answered_calls', 'total_inbound_calls') }} AS answer_rate,
    avgIf(call_duration_seconds, funnel_stage = 'lead_connected') AS avg_call_duration_seconds
FROM {{ ref('stg_funnel_events') }}
WHERE source = 'phone_inbound'
GROUP BY practice_id, event_date
```

### Mart Layer

**`mrt_executive_dashboard`** — Pre-aggregated data powering the Metabase executive view.

```sql
-- mrt_executive_dashboard.sql
-- Materialization: table
-- Refresh: every 30 minutes

SELECT
    p.practice_id,
    p.practice_name,
    p.region,
    p.state,
    f.event_week,
    f.leads_generated,
    f.leads_connected,
    f.leads_booked,
    f.patients_showed,
    f.production_completed,
    f.total_production,
    f.connect_rate,
    f.booking_rate,
    f.show_rate,
    f.production_rate,
    f.lead_to_book_rate,
    -- Week-over-week deltas
    f.leads_generated - LAG(f.leads_generated) OVER (
        PARTITION BY f.practice_id ORDER BY f.event_week
    ) AS leads_generated_wow_delta,
    f.show_rate - LAG(f.show_rate) OVER (
        PARTITION BY f.practice_id ORDER BY f.event_week
    ) AS show_rate_wow_delta,
    f.total_production - LAG(f.total_production) OVER (
        PARTITION BY f.practice_id ORDER BY f.event_week
    ) AS production_wow_delta
FROM {{ ref('int_conversion_rates') }} f
JOIN {{ ref('stg_practices') }} p ON f.practice_id = p.practice_id
```

**`mrt_practice_detail`** — Full practice detail with all metrics for drill-down.

```sql
-- mrt_practice_detail.sql
-- Materialization: table
-- Refresh: every 30 minutes

SELECT
    p.practice_id,
    p.practice_name,
    p.region,
    p.specialty,
    p.chair_count,
    f.event_date,
    f.funnel_stage,
    f.event_count,
    f.unique_patients,
    f.total_value,
    ph.answer_rate,
    ph.avg_call_duration_seconds,
    ph.missed_calls
FROM {{ ref('int_daily_practice_funnel') }} f
JOIN {{ ref('stg_practices') }} p ON f.practice_id = p.practice_id
LEFT JOIN {{ ref('int_phone_metrics') }} ph
    ON f.practice_id = ph.practice_id AND f.event_date = ph.event_date
```

**`mrt_network_trends`** — Network-wide weekly trends for executive summary.

```sql
-- mrt_network_trends.sql
-- Materialization: table
-- Refresh: every 30 minutes

SELECT
    event_week,
    SUM(leads_generated) AS network_leads_generated,
    SUM(leads_connected) AS network_leads_connected,
    SUM(leads_booked) AS network_leads_booked,
    SUM(patients_showed) AS network_patients_showed,
    SUM(production_completed) AS network_production_completed,
    SUM(total_production) AS network_total_production,
    {{ safe_divide('SUM(leads_connected)', 'SUM(leads_generated)') }} AS network_connect_rate,
    {{ safe_divide('SUM(leads_booked)', 'SUM(leads_connected)') }} AS network_booking_rate,
    {{ safe_divide('SUM(patients_showed)', 'SUM(leads_booked)') }} AS network_show_rate,
    COUNT(DISTINCT practice_id) AS active_practices
FROM {{ ref('int_conversion_rates') }}
GROUP BY event_week
ORDER BY event_week DESC
```

**`mrt_practice_health_scores`** — Composite health score per practice (written to PostgreSQL for API).

```sql
-- mrt_practice_health_scores.sql
-- Materialization: table (synced to PostgreSQL via post-hook)
-- Refresh: daily at 06:00 UTC

WITH recent AS (
    SELECT * FROM {{ ref('int_conversion_rates') }}
    WHERE event_week >= toStartOfWeek(now() - INTERVAL 4 WEEK)
),
phone AS (
    SELECT
        practice_id,
        AVG(answer_rate) AS avg_answer_rate
    FROM {{ ref('int_phone_metrics') }}
    WHERE event_date >= today() - 28
    GROUP BY practice_id
),
scores AS (
    SELECT
        r.practice_id,
        today() AS score_date,
        AVG(r.connect_rate) AS phone_answer_rate,
        AVG(r.booking_rate) AS booking_rate,
        AVG(r.show_rate) AS show_rate,
        -- Production vs trailing 4-week average
        CASE
            WHEN SUM(r.total_production) > 0 THEN 1.0
            ELSE 0.0
        END AS production_vs_target,
        CASE
            WHEN last_value(r.leads_generated) OVER (
                PARTITION BY r.practice_id ORDER BY r.event_week
            ) > first_value(r.leads_generated) OVER (
                PARTITION BY r.practice_id ORDER BY r.event_week
            ) THEN 'up'
            WHEN last_value(r.leads_generated) OVER (
                PARTITION BY r.practice_id ORDER BY r.event_week
            ) < first_value(r.leads_generated) OVER (
                PARTITION BY r.practice_id ORDER BY r.event_week
            ) THEN 'down'
            ELSE 'stable'
        END AS lead_volume_trend
    FROM recent r
    LEFT JOIN phone p ON r.practice_id = p.practice_id
    GROUP BY r.practice_id
)
SELECT
    practice_id,
    score_date,
    -- Weighted composite: phone 25%, booking 25%, show 30%, production 20%
    (phone_answer_rate * 25 + booking_rate * 25 + show_rate * 30 + production_vs_target * 20) AS overall_score,
    phone_answer_rate,
    booking_rate,
    show_rate,
    production_vs_target,
    lead_volume_trend
FROM scores
```

**`mrt_source_roi`** — Lead source ROI for attribution reporting.

```sql
-- mrt_source_roi.sql
-- Materialization: table
-- Refresh: daily

SELECT
    practice_id,
    event_week,
    source,
    countIf(funnel_stage = 'lead_generated') AS leads,
    countIf(funnel_stage = 'lead_booked') AS bookings,
    countIf(funnel_stage = 'production_completed') AS completions,
    sumIf(value_estimate, funnel_stage = 'lead_generated') AS total_ad_cost,
    sumIf(value_estimate, funnel_stage = 'production_completed') AS total_production,
    {{ safe_divide(
        "sumIf(value_estimate, funnel_stage = 'production_completed')",
        "sumIf(value_estimate, funnel_stage = 'lead_generated')"
    ) }} AS roas
FROM {{ ref('stg_funnel_events') }}
GROUP BY practice_id, event_week, source
```

### dbt Macros

```sql
-- macros/safe_divide.sql
{% macro safe_divide(numerator, denominator) %}
    CASE WHEN {{ denominator }} > 0
         THEN ROUND(CAST({{ numerator }} AS Float64) / {{ denominator }}, 4)
         ELSE 0
    END
{% endmacro %}
```

### Refresh Schedule

| Layer | Frequency | Trigger |
|-------|-----------|---------|
| Staging (views) | Real-time | N/A (views) |
| Intermediate | Every 60 minutes | Airflow DAG / dbt Cloud scheduled run |
| Marts (dashboard) | Every 30 minutes | Airflow DAG / dbt Cloud scheduled run |
| Marts (health scores) | Daily 06:00 UTC | Airflow DAG |

---

## 6. API Endpoints (Phase 1)

All endpoints served by the **Query Service** (FastAPI on port 8001) unless noted as Admin Service (port 8002).

### Authentication

Phase 1 uses API key authentication (header: `X-API-Key`). Metabase is configured with a dedicated service key. Admin endpoints require admin-role keys.

### 6.1 GET /api/funnel/network

Network-wide funnel snapshot across all active practices.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `period` | string | `current_week` | `current_week`, `last_week`, `last_30_days`, `custom` |
| `start_date` | date | — | Required if period=custom |
| `end_date` | date | — | Required if period=custom |

**Response 200**

```json
{
  "period": {
    "start": "2026-03-02",
    "end": "2026-03-08",
    "label": "Week of Mar 2, 2026"
  },
  "active_practices": 10,
  "funnel": {
    "lead_generated": {
      "count": 1842,
      "unique_patients": 1654,
      "wow_delta": 127,
      "wow_delta_pct": 7.4
    },
    "lead_connected": {
      "count": 1290,
      "unique_patients": 1180,
      "wow_delta": 85,
      "wow_delta_pct": 7.1
    },
    "lead_booked": {
      "count": 812,
      "unique_patients": 780,
      "wow_delta": -23,
      "wow_delta_pct": -2.8
    },
    "patient_showed": {
      "count": 698,
      "unique_patients": 672,
      "wow_delta": 15,
      "wow_delta_pct": 2.2
    },
    "production_completed": {
      "count": 645,
      "unique_patients": 620,
      "total_value": 487250.00,
      "wow_delta": 32,
      "wow_delta_pct": 5.2
    }
  },
  "conversion_rates": {
    "lead_to_connected": 0.7003,
    "connected_to_booked": 0.6295,
    "booked_to_showed": 0.8596,
    "showed_to_produced": 0.9241,
    "lead_to_booked": 0.4409
  },
  "top_sources": [
    {"source": "phone_inbound", "count": 923, "pct": 50.1},
    {"source": "ad_google", "count": 412, "pct": 22.4},
    {"source": "form_web", "count": 287, "pct": 15.6},
    {"source": "referral", "count": 220, "pct": 11.9}
  ]
}
```

### 6.2 GET /api/funnel/practice/{practice_id}

Detailed funnel view for a single practice.

**Path Parameters**

| Param | Type | Description |
|-------|------|-------------|
| `practice_id` | string | Practice slug, e.g. `garner-dental-woodstock` |

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `period` | string | `current_week` | Same as network endpoint |
| `include_daily` | bool | `false` | Include daily breakdown within the period |

**Response 200**

```json
{
  "practice": {
    "id": "garner-dental-woodstock",
    "name": "Garner Dental - Woodstock",
    "region": "Southeast",
    "state": "GA",
    "specialty": "general",
    "chair_count": 6
  },
  "period": {
    "start": "2026-03-02",
    "end": "2026-03-08",
    "label": "Week of Mar 2, 2026"
  },
  "funnel": {
    "lead_generated": {"count": 187, "wow_delta": 12, "wow_delta_pct": 6.9},
    "lead_connected": {"count": 142, "wow_delta": 8, "wow_delta_pct": 6.0},
    "lead_booked": {"count": 89, "wow_delta": -3, "wow_delta_pct": -3.3},
    "patient_showed": {"count": 78, "wow_delta": 5, "wow_delta_pct": 6.8},
    "production_completed": {
      "count": 72,
      "total_value": 54320.00,
      "wow_delta": 4,
      "wow_delta_pct": 5.9
    }
  },
  "conversion_rates": {
    "lead_to_connected": 0.7594,
    "connected_to_booked": 0.6268,
    "booked_to_showed": 0.8764,
    "showed_to_produced": 0.9231
  },
  "phone_metrics": {
    "answer_rate": 0.7594,
    "avg_call_duration_seconds": 195,
    "missed_calls": 45,
    "peak_hour": 10
  },
  "source_breakdown": [
    {"source": "phone_inbound", "leads": 98, "booked": 52, "showed": 47},
    {"source": "ad_google", "leads": 45, "booked": 18, "showed": 14},
    {"source": "form_web", "leads": 28, "booked": 12, "showed": 11},
    {"source": "referral", "leads": 16, "booked": 7, "showed": 6}
  ],
  "health_score": {
    "overall": 78.5,
    "phone_answer_rate": 0.7594,
    "booking_rate": 0.6268,
    "show_rate": 0.8764,
    "production_vs_target": 0.92,
    "lead_volume_trend": "up"
  },
  "connectors": [
    {"type": "opendental", "last_sync": "2026-03-08T14:30:00Z", "status": "success"},
    {"type": "ringcentral", "last_sync": "2026-03-08T14:28:00Z", "status": "success"},
    {"type": "google_ads", "last_sync": "2026-03-08T12:00:00Z", "status": "success"}
  ],
  "daily_breakdown": null
}
```

### 6.3 GET /api/funnel/trends

Week-over-week trends for the network or a specific practice.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `practice_id` | string | null | Optional. Omit for network-wide. |
| `weeks` | int | 8 | Number of weeks to return |
| `metric` | string | `all` | `all`, `lead_generated`, `show_rate`, `production`, `answer_rate` |

**Response 200**

```json
{
  "scope": "network",
  "weeks_requested": 8,
  "trends": [
    {
      "week_start": "2026-03-02",
      "leads_generated": 1842,
      "leads_connected": 1290,
      "leads_booked": 812,
      "patients_showed": 698,
      "production_completed": 645,
      "total_production": 487250.00,
      "connect_rate": 0.7003,
      "booking_rate": 0.6295,
      "show_rate": 0.8596,
      "production_rate": 0.9241,
      "active_practices": 10
    },
    {
      "week_start": "2026-02-23",
      "leads_generated": 1715,
      "leads_connected": 1205,
      "leads_booked": 835,
      "patients_showed": 683,
      "production_completed": 613,
      "total_production": 462100.00,
      "connect_rate": 0.7026,
      "booking_rate": 0.6929,
      "show_rate": 0.8180,
      "production_rate": 0.8975,
      "active_practices": 10
    }
  ]
}
```

### 6.4 GET /api/practices

**Service**: Admin Service (port 8002)

Practice list with health scores and connector status.

**Query Parameters**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `region` | string | null | Filter by region |
| `sort_by` | string | `name` | `name`, `health_score`, `production`, `show_rate` |
| `sort_dir` | string | `asc` | `asc`, `desc` |

**Response 200**

```json
{
  "practices": [
    {
      "id": "garner-dental-woodstock",
      "name": "Garner Dental - Woodstock",
      "region": "Southeast",
      "state": "GA",
      "specialty": "general",
      "chair_count": 6,
      "health_score": 78.5,
      "health_trend": "up",
      "connectors": {
        "opendental": {"status": "success", "last_sync": "2026-03-08T14:30:00Z"},
        "ringcentral": {"status": "success", "last_sync": "2026-03-08T14:28:00Z"},
        "google_ads": {"status": "success", "last_sync": "2026-03-08T12:00:00Z"}
      },
      "this_week": {
        "leads_generated": 187,
        "show_rate": 0.8764,
        "total_production": 54320.00
      }
    }
  ],
  "total": 10,
  "filters_applied": {}
}
```

### 6.5 POST /api/connectors/{connector_config_id}/sync

**Service**: Admin Service (port 8002)

Trigger a manual sync for a specific connector configuration.

**Path Parameters**

| Param | Type | Description |
|-------|------|-------------|
| `connector_config_id` | int | ID from connector_configs table |

**Request Body**

```json
{
  "sync_type": "incremental",
  "lookback_hours": null
}
```

`sync_type` options: `incremental` (from last checkpoint), `full` (re-pull configurable lookback window).
`lookback_hours`: optional override for full sync (default 48).

**Response 202 (Accepted)**

```json
{
  "sync_id": 1234,
  "connector_config_id": 42,
  "practice_id": "garner-dental-woodstock",
  "connector_type": "opendental",
  "sync_type": "incremental",
  "status": "running",
  "started_at": "2026-03-08T15:00:00Z",
  "message": "Sync initiated. Poll GET /api/connectors/42/sync/1234 for status."
}
```

**Response 409 (Conflict)**

```json
{
  "error": "sync_in_progress",
  "message": "A sync is already running for this connector. Started at 2026-03-08T14:58:00Z.",
  "current_sync_id": 1233
}
```

### 6.6 GET /api/connectors/{connector_config_id}/sync/{sync_id}

**Service**: Admin Service (port 8002)

Check status of a sync job.

**Response 200**

```json
{
  "sync_id": 1234,
  "connector_config_id": 42,
  "practice_id": "garner-dental-woodstock",
  "connector_type": "opendental",
  "status": "success",
  "started_at": "2026-03-08T15:00:00Z",
  "completed_at": "2026-03-08T15:00:47Z",
  "duration_ms": 47200,
  "records_fetched": 142,
  "records_produced": 38,
  "records_failed": 0,
  "error_summary": null
}
```

---

## 7. Data Quality

### 7.1 Duplicate Events

**Problem**: Connectors may produce the same event twice (retries, overlapping sync windows, Kafka redelivery).

**Strategy**: Deterministic event IDs + ClickHouse ReplacingMergeTree.

```python
import hashlib
import uuid

def generate_deterministic_uuid(*components: str) -> str:
    """
    Generate a UUID5-style deterministic ID from input components.
    Same inputs always produce the same event_id, so re-ingesting
    the same source record produces an identical event_id.
    """
    composite = "|".join(str(c) for c in components if c is not None)
    return str(uuid.UUID(hashlib.md5(composite.encode()).hexdigest()))
```

- **PMS events**: `deterministic_uuid(practice_id, appointment_num, funnel_stage)` — same appointment + stage always yields the same event_id.
- **Phone events**: `deterministic_uuid(practice_id, ringcentral_call_id, funnel_stage)` — same call always yields the same event_id.
- **Ad events**: `deterministic_uuid(practice_id, gclid)` or `deterministic_uuid(practice_id, campaign_id, date, click_index)` for aggregate mode.

**ClickHouse handling**: `ReplacingMergeTree(_ingested_at)` deduplicates on the full ORDER BY key during background merges. For real-time queries that need dedup before merge, add `FINAL` to critical queries (acceptable at Phase 1 scale of 10 practices).

**Kafka handling**: Producer idempotence is enabled (`enable.idempotence=true`). Consumer commits offsets manually after successful ClickHouse insert, preventing redelivery on consumer restart.

### 7.2 Late-Arriving Data

**Problem**: PMS data may update hours after the event (e.g., appointment status changes from Scheduled to Complete after the patient visit). Google Ads data finalizes 3+ hours after midnight.

**Strategy**:

1. **Daily reconciliation syncs**: Each connector runs a full re-pull of the last 48 hours at a scheduled off-peak time. This catches any events that were missed or updated since the incremental sync.

2. **Event versioning**: The `_ingested_at` field in ClickHouse tracks when the event was last written. `ReplacingMergeTree` keeps the latest version. If a procedure fee is updated in the PMS, the next sync produces the same `event_id` with an updated `value_estimate` and a newer `_ingested_at`.

3. **dbt model tolerance**: Mart models that calculate weekly aggregates use a 48-hour buffer before considering a week "final." The executive dashboard labels the current week as "In Progress" and the previous week as "Final" only after Wednesday (allowing for weekend data to settle).

4. **Google Ads lag**: Campaign stats are pulled with a 6-hour delay. The connector always re-pulls yesterday + today, and the dbt model treats the current day's ad data as preliminary.

### 7.3 Missing Fields

**Problem**: Source data may have null or empty fields. Patient may not have a referral source. Appointment may not have a procedure code.

**Strategy**:

```python
# Validation in the Event Pipeline consumer (before ClickHouse insert)

from pydantic import BaseModel, validator
from typing import Optional
from decimal import Decimal
from datetime import datetime

class FunnelEventValidated(BaseModel):
    event_id: str                          # REQUIRED — reject if missing
    practice_id: str                       # REQUIRED — reject if missing
    timestamp: datetime                    # REQUIRED — reject if missing
    funnel_stage: FunnelStage              # REQUIRED — reject if missing
    previous_stage: Optional[FunnelStage]  # NULLABLE — ok if missing
    source: str                            # REQUIRED — default to 'unknown' if missing
    handler: HandlerType                   # REQUIRED — default to 'unknown' if missing
    patient_id: Optional[str]              # NULLABLE — ok for ad clicks
    appointment_id: Optional[str]          # NULLABLE — ok for phone/ad events
    value_estimate: Decimal                # REQUIRED — default to 0.00
    metadata: dict                         # REQUIRED — default to {}

    @validator('source', pre=True, always=True)
    def default_source(cls, v):
        return v or 'unknown'

    @validator('handler', pre=True, always=True)
    def default_handler(cls, v):
        return v or 'unknown'

    @validator('value_estimate', pre=True, always=True)
    def default_value(cls, v):
        return v if v is not None else Decimal('0.00')
```

**Rejection vs. defaulting rules**:

| Field | If Missing | Action |
|-------|------------|--------|
| `event_id` | Always | **Reject** to DLQ — cannot track without ID |
| `practice_id` | Always | **Reject** to DLQ — cannot attribute without practice |
| `timestamp` | Always | **Reject** to DLQ — cannot place in time |
| `funnel_stage` | Always | **Reject** to DLQ — core schema |
| `previous_stage` | Often null | **Accept** as null — first-touch events have no previous stage |
| `source` | Rare | **Default** to `'unknown'`, emit warning metric |
| `handler` | Rare | **Default** to `'unknown'`, emit warning metric |
| `patient_id` | Ad clicks | **Accept** as null — not all events have patient identity |
| `appointment_id` | Phone/ads | **Accept** as null — only PMS events have this |
| `value_estimate` | Common | **Default** to `0.00` — value often unknown at early funnel stages |
| `metadata` | Never | **Default** to `{}` |

**DLQ monitoring**: The `connector.dlq` Kafka topic receives all rejected events. A daily Airflow task counts DLQ events by connector and practice, and alerts if the rejection rate exceeds 5% of total events for any connector.

### 7.4 Timezone Normalization

**Problem**: Open Dental stores timestamps in the practice's local timezone (often without timezone info). RingCentral returns UTC. Google Ads returns in the account's timezone.

**Strategy**:

1. **All events stored in UTC** in ClickHouse. This is non-negotiable.

2. **Practice timezone is stored in PostgreSQL** (`practices.timezone`, e.g., `'America/New_York'`).

3. **Connector-level conversion**:

```python
from zoneinfo import ZoneInfo
from datetime import datetime

def normalize_to_utc(
    raw_timestamp: str | datetime,
    source_timezone: str,  # from practice config or connector config
    assume_local: bool = False
) -> datetime:
    """
    Normalize any timestamp to UTC.

    - If the timestamp has timezone info, convert directly to UTC.
    - If the timestamp is naive (no tz info) and assume_local=True,
      treat it as the source_timezone and convert to UTC.
    - If assume_local=False and timestamp is naive, raise ValueError.
    """
    if isinstance(raw_timestamp, str):
        raw_timestamp = datetime.fromisoformat(raw_timestamp)

    if raw_timestamp.tzinfo is not None:
        return raw_timestamp.astimezone(ZoneInfo("UTC")).replace(tzinfo=None)

    if assume_local:
        local_tz = ZoneInfo(source_timezone)
        localized = raw_timestamp.replace(tzinfo=local_tz)
        return localized.astimezone(ZoneInfo("UTC")).replace(tzinfo=None)

    raise ValueError(
        f"Naive timestamp {raw_timestamp} received without timezone. "
        f"Set assume_local=True if this should be treated as {source_timezone}."
    )
```

4. **Per-connector rules**:

| Connector | Input Timezone | `assume_local` | Notes |
|-----------|---------------|-----------------|-------|
| Open Dental | Practice local (naive) | `True` | API returns naive datetimes in practice local time |
| RingCentral | UTC (ISO 8601 with Z) | `False` | Already UTC, direct parse |
| Google Ads | Account timezone | `True` | Date-only fields treated as noon in account timezone |

5. **Dashboard display**: The Query Service accepts an optional `timezone` parameter for display purposes, converting UTC results to the requested timezone. Default is the practice's configured timezone for practice-level views and `America/New_York` (SGA HQ) for network views.

### 7.5 PMS Data Inconsistencies

**Problem**: Open Dental data has known quirks — appointment statuses that don't follow the expected lifecycle, procedures logged without appointments, deleted records that reappear, and fee amounts that change after the fact.

**Strategy**:

1. **Appointment lifecycle validation**: The Event Pipeline validates that funnel stage transitions make sense. If a `patient_showed` event arrives for a patient with no prior `lead_booked` event, the pipeline creates a synthetic `lead_booked` event with the same timestamp and `source='pms_backfill'`.

```python
async def validate_stage_transition(
    event: FunnelEventValidated,
    recent_events_cache: dict  # practice_id+patient_id → last known stage
) -> list[FunnelEventValidated]:
    """
    Returns a list of events to insert. Usually just the input event,
    but may prepend synthetic events to fill gaps in the funnel.
    """
    result = []
    cache_key = f"{event.practice_id}:{event.patient_id}"
    last_stage = recent_events_cache.get(cache_key)

    expected_previous = STAGE_ORDER.get(event.funnel_stage)
    # expected_previous maps: lead_connected→lead_generated, lead_booked→lead_connected, etc.

    if last_stage is None and event.funnel_stage != "lead_generated":
        # Patient appeared mid-funnel — backfill missing stages
        for missing_stage in get_missing_stages(None, event.funnel_stage):
            result.append(create_synthetic_event(event, missing_stage))

    elif last_stage and STAGE_ORDINAL[event.funnel_stage] <= STAGE_ORDINAL[last_stage]:
        # Regression (e.g., booked→generated) — likely a new episode, allow it
        pass

    result.append(event)
    recent_events_cache[cache_key] = event.funnel_stage
    return result
```

2. **Fee reconciliation**: If a procedure's fee changes in the PMS (common with insurance adjustments), the reconciliation sync picks up the update. The deterministic event_id stays the same, but the `value_estimate` changes, and `ReplacingMergeTree` keeps the latest version.

3. **Deleted records handling**: The daily reconciliation compares the set of known event_ids for the last 48 hours against the current PMS data. If an appointment was deleted in the PMS (rare but happens), the event is soft-deleted by inserting a version with `metadata.deleted=true` and `value_estimate=0`. Queries filter these out.

4. **Procedure-without-appointment**: Some procedures in Open Dental (walk-ins, emergency) have `AptNum=0`. These are valid `production_completed` events with `appointment_id=null`. The pipeline accepts them and maps `source='walk_in'`.

5. **Data quality metrics**: A daily dbt model computes data quality scores per practice:

```sql
-- Tracked in a dbt model, surfaced in Admin API
SELECT
    practice_id,
    today() AS check_date,
    -- Events with 'unknown' source
    countIf(source = 'unknown') * 100.0 / count() AS pct_unknown_source,
    -- Events with synthetic backfill
    countIf(JSONExtractString(metadata, 'synthetic') = 'true') * 100.0 / count() AS pct_synthetic,
    -- Events with zero value at production stage
    countIf(funnel_stage = 'production_completed' AND value_estimate = 0) * 100.0
        / countIf(funnel_stage = 'production_completed') AS pct_zero_value_production,
    -- Show rate sanity check (should be 50-100%)
    countIf(funnel_stage = 'patient_showed') * 1.0
        / GREATEST(countIf(funnel_stage = 'lead_booked'), 1) AS show_rate_check
FROM funnel_os.funnel_events
WHERE toDate(timestamp) >= today() - 7
GROUP BY practice_id
```

Alert thresholds:
- `pct_unknown_source > 10%` — warning, investigate connector mapping
- `pct_synthetic > 20%` — warning, PMS integration may be missing data
- `pct_zero_value_production > 15%` — warning, procedure fees not flowing
- `show_rate_check > 1.05` — error, more shows than bookings (data integrity issue)

---

## Appendix A: Environment Variables

```bash
# Shared
ENVIRONMENT=production              # production, staging, development
LOG_LEVEL=INFO
AWS_REGION=us-east-1

# Kafka (MSK)
KAFKA_BOOTSTRAP_SERVERS=b-1.funnel-os.xxx.kafka.us-east-1.amazonaws.com:9096
KAFKA_SECURITY_PROTOCOL=SASL_SSL
KAFKA_SASL_MECHANISM=AWS_MSK_IAM

# ClickHouse
CLICKHOUSE_HOST=clickhouse.funnel-os.internal
CLICKHOUSE_PORT=9000
CLICKHOUSE_HTTP_PORT=8123
CLICKHOUSE_DATABASE=funnel_os
CLICKHOUSE_USER=funnel_writer       # event-pipeline-writer
CLICKHOUSE_PASSWORD=<secret>
CLICKHOUSE_READER_USER=funnel_reader  # query-service, metabase
CLICKHOUSE_READER_PASSWORD=<secret>

# PostgreSQL (RDS)
POSTGRES_HOST=funnel-os-ops.xxx.us-east-1.rds.amazonaws.com
POSTGRES_PORT=5432
POSTGRES_DATABASE=funnel_os_ops
POSTGRES_USER=funnel_admin
POSTGRES_PASSWORD=<secret>

# Connector encryption
CONNECTOR_ENCRYPTION_KEY=<AWS KMS key ARN>

# Services
CONNECTOR_SERVICE_URL=http://connector-service.funnel-os.internal:8003
QUERY_SERVICE_URL=http://query-service.funnel-os.internal:8001
ADMIN_SERVICE_URL=http://admin-service.funnel-os.internal:8002
```

## Appendix B: Repository Structure

```
funnel-os/
  services/
    connector-service/
      connectors/
        opendental/
          client.py          # Open Dental API client
          mapper.py          # Raw → FunnelEvent mapping
          sync.py            # Incremental + full sync logic
        ringcentral/
          client.py
          mapper.py
          sync.py
        google_ads/
          client.py
          mapper.py
          sync.py
      scheduler.py           # APScheduler for polling intervals
      main.py                # FastAPI app (health check + manual sync trigger)
      Dockerfile
    event-pipeline/
      consumers/
        validator.py         # Schema validation + dedup check
        writer.py            # ClickHouse batch insert
        archiver.py          # S3 Parquet writer
      pipeline.py            # Consumer group orchestration
      Dockerfile
    query-service/
      routes/
        funnel.py            # /api/funnel/* endpoints
      queries/
        network.py           # ClickHouse query builders
        practice.py
        trends.py
      main.py                # FastAPI app
      Dockerfile
    admin-service/
      routes/
        practices.py         # /api/practices/*
        connectors.py        # /api/connectors/*
      crud/
        practices.py
        connector_configs.py
        sync_history.py
      main.py                # FastAPI app
      Dockerfile
  packages/
    funnel-os-common/
      funnel_os_common/
        models/
          events.py
          enums.py
        kafka/
          producer.py
          consumer.py
        config.py
        logging.py
      pyproject.toml
  dbt/
    dbt_funnel_os/
      models/
        staging/
        intermediate/
        marts/
      macros/
      seeds/
      dbt_project.yml
      profiles.yml
  infra/
    terraform/
      ecs.tf
      msk.tf
      rds.tf
      clickhouse.tf
      s3.tf
      networking.tf
    docker-compose.yml       # Local development
  tests/
    integration/
      test_opendental_connector.py
      test_ringcentral_connector.py
      test_google_ads_connector.py
      test_event_pipeline.py
      test_query_service.py
    unit/
      test_mappers.py
      test_validators.py
      test_deduplication.py
  README.md
  pyproject.toml             # Workspace root (monorepo)
```

## Appendix C: Phase 1 Milestones

| Week | Milestone | Definition of Done |
|------|-----------|-------------------|
| 1 | Infrastructure provisioned | MSK cluster, ClickHouse, RDS, ECS cluster running. Kafka topics created. ClickHouse schema deployed. |
| 1 | Common package published | `funnel_os_common` with event models, Kafka helpers, config. Unit tests passing. |
| 2 | Open Dental connector live | Pulling appointments + procedures from 1 test practice. Events flowing through Kafka to ClickHouse. |
| 2 | Event Pipeline operational | Validator + writer consumers running. DLQ handling verified. Deduplication confirmed. |
| 3 | RingCentral connector live | Call logs flowing for 1 test practice. Phone metrics visible in ClickHouse. |
| 3 | Google Ads connector live | Campaign data flowing for 1 test practice. Ad spend + clicks in ClickHouse. |
| 3 | dbt models deployed | All staging, intermediate, and mart models running on schedule. |
| 4 | Query Service + Admin Service live | All 6 API endpoints returning data. Metabase connected to ClickHouse. |
| 4 | Metabase dashboards built | Network view, practice detail, trends, source breakdown. Shared with stakeholders. |
| 4 | 10 practices onboarded | All 3 connectors configured and syncing for 10 pilot practices. Data quality checks passing. |
