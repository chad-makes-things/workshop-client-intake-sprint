# PHI-Free-First Technical Architecture Review

**Prepared by**: Tech Lead Agent
**Date**: 2026-03-20
**Classification**: Architecture Validation — CTO Review
**Scope**: Validate the PHI-free-first approach against the shared 12-layer infrastructure defined in [technical-architecture.md](../../04-architecture/technical-architecture.md)
**Companion Documents**: [PHI Scope Analysis](../01-scope/phi-scope-analysis.md) | [Experience Architecture](../02-experience/phi-free-experience-architecture.md) | [Build Plan](../04-build-plan/phi-free-build-plan.md) | [PHI Data Flow Analysis](../../10-phi-analysis/phi-data-flow-analysis.md)

---

## 1. Architecture Feasibility Assessment

### 1.1 Can the Shared Infrastructure Truly Operate Without PHI?

**Verdict: Yes, with caveats on 4 of 10 shared services.**

The original architecture defines 10 shared services (Sections 1.1-1.10 of technical-architecture.md) plus 4 cross-cutting layers (Sections 1.8, 1.11-1.14). The PHI-free-first build plan correctly identifies 6 services that operate without PHI and defers 4 to Phase B0. Here is the service-by-service assessment:

| # | Service | Phase A Viable? | Notes |
|---|---------|----------------|-------|
| 1.1 | Practice Data Hub | Yes | Core schema (`shared.practices`, `shared.practice_staff`, `shared.regions`) contains zero PHI. Staff names/contact info are NOT PHI under HIPAA. PHI tables (`patient_records`, `patient_communications`, `call_transcripts`) exist but remain empty. |
| 1.2 | PMS Integration Service | **Deferred (B0)** | Correct. Every API endpoint (`/pms/:practiceId/patients`, `/pms/:practiceId/appointments`) returns PHI. Every Kafka emission (`pms.appointment.created`, `pms.patient.created`) carries PHI payloads. Cannot operate in a useful capacity without PHI. |
| 1.3 | Communications Service | **Partially deferred** | See Section 1.2 below — the content capture SMS function is PHI-free, but the patient communications function is not. The build plan uses a simple non-HIPAA SMS gateway for Phase A content capture. This is architecturally sound but means the shared Communications Service as designed is a B0 deliverable. |
| 1.4 | AI Service | **Partially viable** | The AI Service's core functions (prompt registry, token accounting, cost-tier routing, response caching) are PHI-independent. The PHI filter is only invoked when `contains_phi: true`. In Phase A, all AI requests set `contains_phi: false`. The service can operate without the PHI filter pipeline, Whisper integration, or Anthropic BAA (non-PHI prompts do not require BAA coverage). **However**: caption generation, brief extraction, anomaly analysis for dashboards — all Phase A AI use cases — work without PHI. This service should be built in A1, not deferred to B0. |
| 1.5 | Asset Registry | Yes | Content-type-agnostic. In Phase A it stores brand/team/office assets. The `before_after` asset type exists in the schema but no patient-mapped photos enter. No PHI in metadata. |
| 1.6 | Event Bus (Kafka) | Yes | Topic architecture supports both PHI-free and PHI-bearing events. Phase A topics (`ws3.hscore-calculated`, `ws2.content-asset-approved`, `ws1.onboarding-status-changed`) carry zero PHI. PHI topics (`pms.appointment-completed`, `ws1.funnel-event`) exist in Schema Registry but no producers emit to them until B0. |
| 1.7 | Auth & Identity | Yes | Cognito user pool, RBAC, JWT — all PHI-independent. User identities are staff/HQ users, not patients. |
| 1.8 | Compliance & Audit Layer | **Deferred (B0)** | Correct. The Compliance Middleware (PHI Detector, Consent Checker, Encryption Guard) has no function when no PHI flows through the system. The audit log infrastructure can be deployed in Phase A for non-PHI access logging, but the PHI-specific components are B0. |
| 1.9 | Workflow Engine (Temporal) | Yes | Temporal is workflow-agnostic. Phase A workflows (practice enrollment, H-Score daily calc, content production) contain no PHI in their payloads. |
| 1.10 | Design Tool Abstraction | Yes | Canva integration is entirely PHI-free. Design templates, brand assets, and generated content contain no patient data. |
| 1.11 | Non-PMS Data Integration | Yes | Cinnamon, Swell, Dental Intel provide aggregate practice-level scores. iSolved provides staff HR data. Social platform APIs provide engagement metrics. None carry individual patient data at the integration level the platform consumes. |

### 1.2 Hidden PHI Dependencies in Phase A Services

**Finding: Two hidden dependencies identified. One is resolvable; one requires a design decision.**

**Hidden Dependency 1: AI Service (Resolvable)**

The build plan defers the entire AI Service to B0 as part of "PHI extensions." This is overly conservative. The AI Service has two distinct layers:

- **Core layer** (PHI-free): Prompt registry, cost-tier routing, token accounting, response cache, Claude API client. Used for: caption generation (WS2), brief extraction (WS2), AI alerts/anomaly analysis (WS1 Executive Dashboard), Network Learning Report (WS1). All Phase A use cases.
- **PHI layer** (B0): PHI filter pipeline, Whisper audio transcription, PHI-bearing prompt handling.

**Recommendation**: Build AI Service core in A1. Add PHI filter and Whisper in B0. The `contains_phi` boolean flag on the API contract already provides the separation mechanism. In Phase A, the PHI filter code path is never triggered — it returns an error if `contains_phi: true` is sent before B0 activation.

**Hidden Dependency 2: Content Capture SMS (Design Decision)**

The build plan correctly identifies that Phase A content capture prompts (SMS to champions) contain no PHI. It proposes using a "simple non-HIPAA SMS gateway" instead of the shared Communications Service. This works but creates a migration burden: when the Communications Service ships in B0, all content capture SMS must migrate from the non-HIPAA gateway to the shared service.

**Options**:
1. **Accept the migration** — Build a lightweight SMS sender in Phase A, swap to Communications Service in B0. Low upfront cost, moderate migration cost.
2. **Build Communications Service in A1 with limited scope** — Deploy only the SMS send API, number management, and delivery tracking. Omit consent registry, HIPAA encryption, and patient-specific routing. This gives WS2 the real Communications Service interface from Day 1.

**Recommendation**: Option 2. The Communications Service data model (`shared.message_log`, `shared.phone_numbers`) is useful even for non-PHI SMS. Building it in A1 means WS2's content capture integration code does not change in Phase B. The consent registry and HIPAA-specific middleware are additive in B0.

### 1.3 Practice Data Hub Schema — Dual-Phase Viability

The current `shared.practices` schema (Section 1.1 of technical-architecture.md) works cleanly for Phase A. The critical question is whether the schema design supports clean Phase B activation.

**Current schema analysis**:

| Table | Phase A Data | Phase B Data | Separation Clean? |
|-------|-------------|-------------|-------------------|
| `shared.practices` | Practice profiles, config, enrollment status | Same + PMS connector activation | Yes — `pms_connector`, `nexhealth_id`, `sikka_practice_id` columns exist but are nullable. Populated when PMS Integration Service connects in B0. |
| `shared.practice_staff` | Staff names, roles, certifications, champion status | Same (staff data is never PHI) | Yes — no changes needed. |
| `shared.regions` | Region hierarchy, director mapping | Same | Yes — no changes needed. |
| Empty PHI tables | `patient_records`, `patient_communications`, `call_transcripts` — exist with RLS, encryption configured, zero rows | Populated by PMS Integration Service, Communications Service, Voice AI | Yes — tables exist in A0 by design. RLS policies are configured before data enters. |

**Assessment**: The Practice Data Hub schema is correctly designed for dual-phase operation. PHI tables exist as empty, pre-configured shells. The non-PHI tables carry the full data model for Phase A. No schema migrations are required when Phase B activates — only data starts flowing.

### 1.4 Kafka Event Schemas — Dual-Phase Compatibility

**Can Kafka event schemas support both Phase A (no PHI payloads) and Phase B (PHI payloads) without breaking consumers?**

The 10 cross-workstream event contracts (Section 2 of technical-architecture.md) divide cleanly:

**Phase A events (8 of 10 — zero PHI in payload)**:

| # | Event | PHI in Payload? | Phase A Status |
|---|-------|-----------------|----------------|
| 1 | `ws3.champion-assigned/updated` | No (staff data only) | Active |
| 2 | `ws3.hscore-calculated` | No (aggregate scores) | Active |
| 3 | `ws3.benchmark-taxonomy-updated` | No (taxonomy config) | Active |
| 4 | `ws3.champion-story-submitted` | No (text + media URLs) | Active |
| 6 | `ws2.content-asset-approved` | No (asset metadata + CDN URLs) | Active |
| 8 | `ws2.content-engagement-updated` | No (aggregate engagement metrics) | Active |
| 9 | `ws3.training-completed` | No (staff training data) | Active |
| 10 | `ws1.onboarding-status-changed` | No (practice enrollment status) | Active |

**Phase B events (2 of 10 — contain PHI or PHI-derived data)**:

| # | Event | PHI in Payload? | Phase B Activation |
|---|-------|-----------------|-------------------|
| 5 | `pms.appointment-completed` | Yes — `patient_id_hash`, procedure codes, consent status | Emitted only when PMS Integration Service is active (B0+) |
| 7 | `ws1.funnel-event` | Yes — `patient_id_hash`, funnel stages, revenue attribution | Emitted only when Funnel OS is active (B1+) |

**Critical finding**: Event Contract 5 (`pms.appointment-completed`) uses a hashed patient ID, not raw PHI. The architecture document explicitly states: "Patient PII is NOT included. Only a hashed patient ID is transmitted." This is a deliberate design choice — the Content Engine consumer uses procedure category and consent status, not patient identity. A SHA-256 hash of a patient ID may still qualify as PHI under HIPAA (it is a "unique identifying number" per 45 CFR 164.514(b)(2)(i)(O)). This event should be treated as PHI-bearing regardless of the hash.

**Assessment**: The Kafka architecture supports dual-phase operation without breaking consumers. The Schema Registry's forward-compatibility rules mean Phase B schemas can add optional fields to existing events without breaking Phase A consumers. Phase B-only events simply do not exist on the bus until their producers activate. No consumer changes are needed — consumers that subscribe to Phase B topics receive no events until Phase B producers start.

---

## 2. PHI Hook Design

### 2.1 PostgreSQL Schema Design for Dual-Phase Operation

**Recommended approach: Separate PHI tables (already in the architecture) with a migration activation pattern.**

The build plan (A0.7) already specifies that PHI tables (`patient_records`, `patient_communications`, `call_transcripts`) are created in Phase A with:
- RLS (row-level security) configured per practice
- Application-level encryption (AES-256-GCM) for sensitive columns
- Zero rows — tables exist but are never written to

This is the correct approach. The alternatives and rationale:

| Approach | Pros | Cons | Verdict |
|----------|------|------|---------|
| **Separate PHI tables** (current) | Clean separation; PHI tables can have different RLS/encryption policies; easy to audit which tables contain PHI; Phase B activation is "start writing to existing tables" | Requires JOINs to correlate patient data with practice data; slightly more complex queries | **Selected** |
| Nullable PHI columns on existing tables | Simpler queries (no JOINs); all practice + patient data in one place | Mixes PHI and non-PHI in same table; harder to audit; RLS must distinguish between PHI and non-PHI columns within the same row; encryption overhead on entire table | Rejected |
| Column-level encryption only | Fine-grained control; encrypted columns are unreadable without key | Does not prevent PHI from appearing in query results, logs, or error messages; application must decrypt explicitly; index limitations on encrypted columns | Used as defense-in-depth ON TOP of table separation, not as the primary strategy |

**Recommended schema additions for Phase B readiness (built in A0)**:

```sql
-- PHI tables — created in A0, empty until B0
-- patient_records: master patient index per practice
CREATE TABLE phi.patient_records (
  patient_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  pms_patient_id    TEXT,                    -- ID from PMS (via NexHealth/Sikka)
  patient_id_hash   TEXT NOT NULL,           -- SHA-256 hash for cross-service reference
  name_encrypted    BYTEA,                   -- AES-256-GCM encrypted
  dob_encrypted     BYTEA,                   -- AES-256-GCM encrypted
  ssn_encrypted     BYTEA,                   -- AES-256-GCM encrypted (if collected)
  insurance_id_encrypted BYTEA,              -- AES-256-GCM encrypted
  phone_encrypted   BYTEA,                   -- AES-256-GCM encrypted
  email_encrypted   BYTEA,                   -- AES-256-GCM encrypted
  consent_status    JSONB DEFAULT '{}',      -- per-channel consent: {sms: 'opted_in', email: 'opted_out'}
  funnel_stage      TEXT,                    -- current funnel position
  last_visit_date   DATE,
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);

-- RLS: practice-scoped access
ALTER TABLE phi.patient_records ENABLE ROW LEVEL SECURITY;
CREATE POLICY patient_practice_isolation ON phi.patient_records
  USING (practice_id = current_setting('app.current_practice_id')::UUID);

-- patient_communications: message history
CREATE TABLE phi.patient_communications (
  message_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id        UUID REFERENCES phi.patient_records(patient_id),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  direction         TEXT NOT NULL CHECK (direction IN ('inbound', 'outbound')),
  channel           TEXT NOT NULL,
  body_encrypted    BYTEA,                   -- AES-256-GCM encrypted message body
  template_id       TEXT,
  delivery_status   TEXT,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE phi.patient_communications ENABLE ROW LEVEL SECURITY;
CREATE POLICY comms_practice_isolation ON phi.patient_communications
  USING (practice_id = current_setting('app.current_practice_id')::UUID);

-- call_transcripts: Voice AI transcription data
CREATE TABLE phi.call_transcripts (
  transcript_id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  practice_id       UUID REFERENCES shared.practices(practice_id),
  patient_id        UUID REFERENCES phi.patient_records(patient_id),
  call_recording_s3 TEXT,                    -- S3 key for encrypted recording
  transcript_encrypted BYTEA,               -- AES-256-GCM encrypted transcript text
  analysis_result   JSONB,                   -- AI analysis output (post-PHI-filter)
  call_duration_seconds INTEGER,
  call_direction    TEXT,
  created_at        TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE phi.call_transcripts ENABLE ROW LEVEL SECURITY;
CREATE POLICY transcripts_practice_isolation ON phi.call_transcripts
  USING (practice_id = current_setting('app.current_practice_id')::UUID);
```

**Key design decisions**:
- **Separate `phi` schema**: All PHI tables live under `CREATE SCHEMA phi;` — distinct from `shared` schema. This allows PostgreSQL GRANT policies at the schema level: Phase A services have `USAGE` on `shared` only. Phase B services get `USAGE` on both `shared` and `phi`.
- **BYTEA columns for encrypted fields**: Application-level encryption stores ciphertext as binary. This prevents accidental plaintext exposure in query results, logs, or pg_dump output.
- **`patient_id_hash` for cross-service reference**: Services that need to reference a patient without accessing PHI (e.g., Content Engine checking consent status) use the hash, never the encrypted fields.

### 2.2 Kafka Event Schema Evolution (Phase A to Phase B)

**Recommended approach: Additive schema evolution with optional fields + separate PHI topics for high-sensitivity events.**

The Schema Registry (Confluent or AWS Glue) enforces forward-compatibility rules: new fields must be optional with defaults; existing fields cannot be removed or renamed. This means Phase B can add PHI-related fields to existing events without breaking Phase A consumers.

**Strategy by event type**:

| Event Category | Evolution Strategy | Rationale |
|---------------|-------------------|-----------|
| Events that gain optional PHI context (Contracts 1, 2, 4, 6, 8, 9, 10) | Additive fields with `null` defaults | These events are PHI-free by nature. Phase B might add optional context (e.g., `patient_count` to H-Score event) but the core payload is unchanged. Existing consumers ignore new fields. |
| Events that are entirely PHI-bearing (Contract 5: `pms.appointment-completed`, Contract 7: `ws1.funnel-event`) | **Separate topics** — already in separate Kafka topics | These events do not exist in Phase A. Their topics are registered in Schema Registry but no producer writes to them. When B0 activates the PMS Integration Service, it starts producing to `sgadental.pms.appointment-completed`. Consumer groups for these topics are only deployed in B1. |
| New Phase B events (patient comms, call recordings, automation triggers) | **New topics** | Events that have no Phase A equivalent are created as new Kafka topics in B0. Schema registered at creation time. No backward-compatibility concern. |

**Schema versioning approach**:

```
Schema version header (included in every event):
{
  "schema_version": "1.0",    // Phase A
  "schema_version": "1.1",    // Phase B additions (optional fields)
  "schema_version": "2.0",    // Breaking changes (major version — reserved, not planned)
}
```

Phase A consumers are coded to handle `schema_version: "1.0"`. When Phase B adds optional fields (version `"1.1"`), Phase A consumers safely ignore unknown fields. No consumer redeployment required.

### 2.3 Feature Flag Architecture for Progressive PHI Enablement

**Recommended approach: Hierarchical flags — global phase flag + per-practice activation flags + per-feature flags.**

```
Flag Hierarchy:
├── global.phase_b_enabled: boolean (false until Gate B0 passes)
│   ├── practice.{practice_id}.phi_enabled: boolean (per-practice activation)
│   │   ├── feature.pms_integration: boolean
│   │   ├── feature.patient_communications: boolean
│   │   ├── feature.voice_ai: boolean
│   │   ├── feature.funnel_analytics: boolean
│   │   ├── feature.patient_photos: boolean
│   │   └── feature.automation_engine: boolean
│   └── practice.{practice_id}.baa_verified: boolean
└── global.phase_a_only: boolean (true — safety kill switch)
```

**Flag evaluation logic**:

```typescript
function isFeatureEnabled(practiceId: string, feature: string): boolean {
  // Safety: global kill switch
  if (flags.get('global.phase_a_only') === true) return false;

  // Phase B must be globally enabled
  if (flags.get('global.phase_b_enabled') !== true) return false;

  // Practice must have PHI enabled AND BAA verified
  if (flags.get(`practice.${practiceId}.phi_enabled`) !== true) return false;
  if (flags.get(`practice.${practiceId}.baa_verified`) !== true) return false;

  // Specific feature flag
  return flags.get(`feature.${feature}`) === true;
}
```

**Implementation**: LaunchDarkly (or equivalent) with server-side SDKs embedded in every service. Flags are evaluated at runtime, not build-time. This allows:
- Rolling Phase B activation per practice (50 practices/week as specified in B2.6)
- Emergency kill switch (`global.phase_a_only = true`) that instantly disables all PHI features network-wide
- Granular feature activation (e.g., enable PMS integration for a practice before enabling Voice AI)
- BAA verification gate — a practice cannot have `phi_enabled: true` until `baa_verified: true` is set by the compliance workflow

**Build-time vs. runtime**: All flags are runtime-evaluated. Build-time flags (e.g., compile-time feature removal) are not used because they would require separate builds for Phase A and Phase B deployments. A single codebase with runtime flags is simpler to maintain, test, and deploy.

---

## 3. Shared Service Phasing Validation

### Service-by-Service Assessment

#### 3.1 Practice Data Hub (Section 1.1)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. 260 practices seeded. Staff roster management. Region hierarchy. Practice configuration. Feature flags. |
| **PHI-free mode needed?** | No. The service is inherently PHI-free in its current schema. PHI tables exist in the `phi` schema, not the `shared` schema. |
| **Phase B changes** | PMS Integration Service starts writing to `phi.patient_records`. Practice Data Hub API gains new endpoints for patient queries (behind feature flags). Read replica workload increases with patient data queries. |
| **Hidden PHI risk** | **Low**. The `shared.practice_staff` table has `contact_phone` and `contact_email` — these are staff PII, not patient PHI. Staff data is NOT protected under HIPAA unless the staff member is also a patient at the same practice. This edge case is acceptable. |

#### 3.2 PMS Integration Service (Section 1.2)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Cannot operate. Every endpoint returns patient data. Every Kafka emission contains PHI. |
| **PHI-free mode needed?** | No — entire service is deferred to B0. |
| **Phase B changes** | Service is built from scratch in B0 (Weeks 13-14). |
| **Hidden PHI risk** | None — service does not exist in Phase A. |

#### 3.3 Communications Service (Section 1.3)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Patient communications cannot operate. Content capture SMS (non-PHI) can operate through a simplified SMS sender. See Section 1.2 recommendation. |
| **PHI-free mode needed?** | Yes, if building in A1. The service needs a mode where `contains_phi` must be `false` for all requests. The consent registry, HIPAA encryption, and patient-specific routing are disabled. Only `purpose: 'content_capture'` messages are allowed. |
| **Phase B changes** | Twilio HIPAA account activation. Consent registry population from PMS patient data. Patient communication routing rules. `contains_phi: true` requests accepted. Full message body encryption. Audit logging of PHI in messages. |
| **Hidden PHI risk** | **Medium**. If champions reply to capture prompts with patient information in the message body (e.g., "Here's a photo of Mrs. Patterson's implant"), that inbound message contains PHI even though the outbound prompt did not. **Mitigation**: Inbound message handler in Phase A should NOT store message bodies — only log receipt confirmation. Message body storage activates in Phase B with encryption. |

#### 3.4 AI Service (Section 1.4)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Core functions operational: prompt registry, cost-tier routing, token accounting, Claude API calls for caption generation, brief extraction, anomaly analysis. PHI filter disabled. Whisper integration disabled. |
| **PHI-free mode needed?** | Yes. The service must reject any request with `contains_phi: true` in Phase A. The PHI filter code path returns a 403 error until B0 activates it. |
| **Phase B changes** | PHI filter pipeline activated. Whisper integration for audio transcription. Redis mapping store for tokenize/detokenize. Anthropic BAA covers PHI-bearing prompts. |
| **Hidden PHI risk** | **Low**. Phase A prompts (caption generation, anomaly analysis) use practice-level aggregate data, not patient data. The risk is a developer accidentally sending patient data in a prompt without setting `contains_phi: true`. **Mitigation**: AI Service input validation scans all prompts for PHI patterns (SSN regex, DOB format, common name + medical term co-occurrence) and rejects requests that appear to contain PHI but are not flagged as such. This "defensive PHI detection" runs in Phase A as a safety net. |

#### 3.5 Asset Registry (Section 1.5)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. Stores brand/team/office content. |
| **PHI-free mode needed?** | No. Content-type-agnostic by design. |
| **Phase B changes** | `before_after` asset type starts receiving patient-mapped photos. Patient-asset mapping stored in `phi` schema (not in Asset Registry metadata). Cloudinary BAA required for patient photo transforms, or switch to self-hosted S3 + Lambda pipeline. |
| **Hidden PHI risk** | **Medium**. Champions may upload photos containing patient faces during Phase A despite guidance to capture intraoral/clinical shots only. **Mitigation**: AI-based image classification at upload time (detects human faces in dental clinical photos). Flagged uploads require HQ review before entering the pipeline. This filter can use the AI Service (Claude vision) with a non-PHI prompt: "Does this image contain a recognizable human face?" |

#### 3.6 Event Bus — Kafka (Section 1.6)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. 8 of 10 event contracts active. PHI topics registered but empty. |
| **PHI-free mode needed?** | No. Kafka is topic-agnostic. Phase B topics simply receive no events until producers activate. |
| **Phase B changes** | PMS Integration Service starts producing to `sgadental.pms.*` topics. WS1 Funnel OS starts producing to `sgadental.ws1.funnel-event`. New consumer groups deployed for B1 features. |
| **Hidden PHI risk** | **Low**. The only risk is a Phase A producer accidentally publishing PHI to a non-PHI topic. **Mitigation**: Schema Registry validation catches payload violations. Additionally, a Kafka consumer-side "PHI sentinel" can monitor all Phase A topics for PHI patterns (regex-based scan of event payloads). Alerts on any detection. |

#### 3.7 Auth & Identity Service (Section 1.7)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. 8 roles defined. JWT middleware. RBAC service. |
| **PHI-free mode needed?** | No. |
| **Phase B changes** | New permissions added for PHI access: `patients:read`, `patients:write`, `comms:send_phi`, `transcripts:read`. These permissions are registered in A0 but not assignable until B0. RBAC policies enforce that PHI-bearing API endpoints require PHI-specific permissions. |
| **Hidden PHI risk** | None. |

#### 3.8 Compliance & Audit Layer (Section 1.8)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Minimal. The audit log table (`shared.audit_log`) can capture non-PHI access events. PHI Detector, Consent Checker, Encryption Guard have no function. |
| **PHI-free mode needed?** | The middleware should be deployed in Phase A in "monitoring mode" — it logs access events but does not enforce PHI-specific controls. This provides: (a) audit trail for Phase A operations, (b) validated infrastructure before PHI arrives, (c) the Compliance Readiness Dashboard (NI-9) has real data to display. |
| **Phase B changes** | Full activation: PHI detection, consent enforcement, encryption guards, breach detection rules. |
| **Hidden PHI risk** | None — the layer is a control, not a data source. |

#### 3.9 Workflow Engine — Temporal (Section 1.9)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. Workflows: practice enrollment, content production, H-Score daily calc. |
| **PHI-free mode needed?** | No. Workflow payloads in Phase A contain practice IDs, staff IDs, and configuration — no PHI. |
| **Phase B changes** | New workflows: communication cadences (multi-day patient follow-up sequences), nightly PMS sync. These workflows carry patient IDs in their payloads. **Critical**: Temporal workflow payloads must be encrypted at application level before submission to Temporal Cloud (per PHI data flow analysis recommendation). |
| **Hidden PHI risk** | **Low**. Phase A workflows do not carry patient data. The risk emerges in B0 when `communication-cadence` and `nightly-pms-sync` workflows activate. |

#### 3.10 Design Tool Abstraction (Section 1.10)

| Aspect | Assessment |
|--------|-----------|
| **Phase A operation** | Fully operational. Canva template generation for storefronts and content. |
| **PHI-free mode needed?** | No. Design tools never process patient data. |
| **Phase B changes** | None. |
| **Hidden PHI risk** | None. |

### Summary: Service Phasing Matrix

| Service | Phase A Build | PHI-Free Mode? | Phase B Delta | Risk |
|---------|--------------|---------------|---------------|------|
| Practice Data Hub | A1 (Weeks 3-4) | Not needed | Patient query endpoints | Low |
| PMS Integration | B0 (Weeks 13-14) | N/A | Full build | None |
| Communications Service | A1 (limited) or B0 (full) | Yes — content capture only | Patient comms, consent, HIPAA encryption | Medium |
| AI Service | A1 (core) + B0 (PHI) | Yes — reject PHI requests | PHI filter, Whisper | Low |
| Asset Registry | A1 (Weeks 5-6) | Not needed | Patient photo mapping | Medium |
| Event Bus (Kafka) | A1 (Weeks 4-5) | Not needed | PHI topic producers activate | Low |
| Auth & Identity | A1 (Weeks 3-4) | Not needed | PHI permissions | None |
| Compliance & Audit | A1 (monitoring) + B0 (enforcement) | Yes — monitoring mode | Full PHI enforcement | None |
| Workflow Engine | A1 (Weeks 5-6) | Not needed | PHI workflow payloads (encrypted) | Low |
| Design Tool Abstraction | A1 (Week 6) | Not needed | None | None |

---

## 4. Event Contract Analysis

### Per-Contract Phase A vs. Phase B Payload Assessment

#### Contract 1: `ws3.champion-assigned/updated`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Staff ID, name, role, champion types, engagement score, streak, certification tier, contact info |
| **Phase B payload** | Identical — champion data is staff data, not patient data |
| **Consumer compatibility** | No change needed. Same payload, same consumers. |
| **Schema evolution** | None required |

#### Contract 2: `ws3.hscore-calculated`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, composite H-Score, 3 component scores (Cinnamon, Swell, Dental Intel), trend, percentile, certification breakdown |
| **Phase B payload** | Same + optional `patient_metrics` field: `{ show_rate, rebooking_rate, treatment_acceptance }` — aggregate practice-level metrics derived from PMS data |
| **Consumer compatibility** | Phase A consumers ignore `patient_metrics` (optional field, defaults to `null`). Phase B consumers use it for enriched dashboard views. |
| **Schema evolution** | Additive. Add optional `patient_metrics` object in version 1.1. Forward-compatible. |

#### Contract 3: `ws3.benchmark-taxonomy-updated`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice type taxonomy, benchmark targets including `target_show_rate` and `target_rebooking_rate` |
| **Phase B payload** | Identical — benchmark targets are configuration, not patient data |
| **Consumer compatibility** | No change. Note: `target_show_rate` and `target_rebooking_rate` benchmarks are in the Phase A schema but only become meaningful when Phase B provides actual show rate and rebooking rate data. |
| **Schema evolution** | None required |

#### Contract 4: `ws3.champion-story-submitted`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, champion staff ID, story text, media URLs, behavior tags, suggested channels, `patient_consent_obtained` flag |
| **Phase B payload** | Identical — stories are champion-authored. The `patient_consent_obtained` field exists in Phase A but is primarily relevant when stories reference patient cases. |
| **Consumer compatibility** | No change. WS2 Content Engine processes the story regardless of phase. |
| **Schema evolution** | None required |
| **Hidden PHI risk** | **Medium**. `text_content` field may contain patient references if a champion describes a specific patient interaction ("Mrs. Patterson's trust transfer was amazing"). **Mitigation**: WS2's content pipeline should run a PHI detection scan on story text before publishing. In Phase A, flag stories with potential PHI for human review. |

#### Contract 5: `pms.appointment-completed` (Phase B only)

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | N/A — event not emitted |
| **Phase B payload** | Practice ID, appointment ID, provider name, procedure codes, procedure category, `is_high_value`, `patient_id_hash`, `media_consent_status` |
| **Consumer compatibility** | WS2 Content Engine subscribes to this topic in B1. Consumer group deployed only after B0 gate passes. Phase A has no consumer for this event. |
| **Schema evolution** | Initial schema created in B0. No Phase A baseline to evolve from. |
| **PHI classification** | Despite using `patient_id_hash` instead of raw patient ID, this event should be classified as PHI-bearing. The hash is a unique identifying number derived from a patient record. Topic should be in the PHI-designated partition set with encryption at rest and restricted consumer group authorization. |

#### Contract 6: `ws2.content-asset-approved`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, asset metadata, CDN URLs, captions, tags, `storefront_eligible` flag |
| **Phase B payload** | Same + optional `patient_association` field: `{ patient_id_hash, consent_status, is_before_after }` for patient-mapped content |
| **Consumer compatibility** | Phase A consumers (WS1 storefront) ignore `patient_association`. Phase B consumers use it for patient content workflows. |
| **Schema evolution** | Additive. Add optional `patient_association` object in version 1.1. Forward-compatible. |

#### Contract 7: `ws1.funnel-event` (Phase B only)

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | N/A — event not emitted |
| **Phase B payload** | Practice ID, `patient_id_hash`, funnel stage transition, source, service type, estimated value, attribution |
| **Consumer compatibility** | WS2 Content Engine subscribes in B1 for booking attribution. No Phase A consumer. |
| **Schema evolution** | Initial schema created in B1. |
| **PHI classification** | PHI-bearing. `patient_id_hash` + `estimated_value` + `service_type` could be combined to re-identify patients. Restricted topic. |

#### Contract 8: `ws2.content-engagement-updated`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, engagement metrics by channel (posts, reach, impressions, engagements), brand health score, content health score |
| **Phase B payload** | Same + optional `attribution` field: `{ bookings_attributed, revenue_attributed }` linking content to patient outcomes |
| **Consumer compatibility** | Phase A consumers ignore `attribution`. |
| **Schema evolution** | Additive. Forward-compatible. |

#### Contract 9: `ws3.training-completed`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, staff ID, module details, certification tier, completion score, practice readiness metrics |
| **Phase B payload** | Identical — training data is staff data, not patient data |
| **Consumer compatibility** | No change needed. |
| **Schema evolution** | None required |

#### Contract 10: `ws1.onboarding-status-changed`

| Aspect | Detail |
|--------|--------|
| **Phase A payload** | Practice ID, previous/new status, milestone, completed/pending steps |
| **Phase B payload** | Same + additional milestones in `completed_steps`: `pms_connected`, `twilio_provisioned`, `agent_configured` — these steps only occur in Phase B onboarding |
| **Consumer compatibility** | WS3 Culture OS consumes `new_status: 'active'` to trigger culture onboarding. This trigger is identical in both phases. Phase B adds more granular milestones but does not change the activation trigger. |
| **Schema evolution** | Additive. New enum values in `milestone` field. Forward-compatible (unknown enum values are safely ignored by consumers). |

### Event Contract Evolution Summary

| Contract | Phase A → B Strategy | Consumer Redeployment Needed? | Risk |
|----------|---------------------|------------------------------|------|
| 1: Champion Roster | No change | No | None |
| 2: H-Score | Additive optional field | No | None |
| 3: Benchmark Taxonomy | No change | No | None |
| 4: Champion Stories | No change | No | Medium (PHI in text) |
| 5: Appointment Completed | New topic (B0) | New consumer deployed in B1 | Low |
| 6: Asset Approved | Additive optional field | No | None |
| 7: Funnel Event | New topic (B1) | New consumer deployed in B1 | Low |
| 8: Engagement Updated | Additive optional field | No | None |
| 9: Training Completed | No change | No | None |
| 10: Onboarding Status | Additive enum values | No | None |

---

## 5. Architecture Decision Records

### ADR-PHI-001: PHI-Free Data Model Strategy

**Status**: Proposed
**Date**: 2026-03-20
**Deciders**: Tech Lead, Platform Architect

**Context**:
The PHI-free-first approach requires the PostgreSQL data model to support two operational phases: Phase A (no patient data) and Phase B (full patient data). The schema must be designed in A0 so that Phase B activation does not require schema migrations, only data flow activation. Three approaches were considered: nullable PHI columns on existing tables, separate PHI tables in a dedicated schema, and column-level encryption as the primary separation mechanism.

**Decision**:
Adopt a **separate `phi` schema** for all patient-related tables. The `shared` schema contains practice, staff, configuration, and operational data (PHI-free). The `phi` schema contains `patient_records`, `patient_communications`, and `call_transcripts` with application-level AES-256-GCM encryption on all sensitive columns. Both schemas are created in A0. Phase A services have `USAGE` privileges on `shared` only. Phase B services get `USAGE` on both schemas. Foreign keys from `phi` tables reference `shared.practices(practice_id)` for practice scoping.

**Consequences**:
- (+) Clean audit boundary: `SELECT * FROM information_schema.tables WHERE table_schema = 'phi'` lists every table containing patient data
- (+) Schema-level PostgreSQL GRANT policies enforce that Phase A services cannot access PHI tables even if application code is buggy
- (+) RLS policies are scoped to the `phi` schema; `shared` schema tables do not pay the RLS performance cost
- (+) Phase B activation is "grant USAGE on phi schema to service X" + "start writing data" — no schema migration
- (+) Backup and encryption policies can target the `phi` schema independently (e.g., more frequent backups, different KMS keys)
- (-) JOINs across `shared` and `phi` schemas are required for queries that combine practice and patient data (e.g., "patients at practice X") — minor performance impact, resolved by proper indexing
- (-) Two schemas add operational complexity (two sets of migration scripts, two sets of seed data)
- (-) Developers must understand which schema to use for each table — mitigated by naming convention and code review

**Alternatives Considered**:
1. **Nullable PHI columns**: Add `patient_name`, `patient_dob`, etc. as nullable columns to `shared.practices` or a single `shared.patients` table. Rejected — mixes PHI and non-PHI data in the same table, making audit, encryption, and access control more complex.
2. **Column-level encryption only**: Encrypt PHI columns and use decryption-key-based access control. Rejected as primary strategy — encrypted columns prevent indexing, complicate queries, and do not prevent data from appearing in `pg_dump` backups. Used as defense-in-depth on top of schema separation.
3. **Separate database instance**: Run a dedicated RDS instance for PHI data. Rejected — excessive operational overhead for 260 practices. Schema-level separation within the same RDS instance provides sufficient isolation with lower cost.

---

### ADR-PHI-002: Event Schema Evolution for Phase A to Phase B

**Status**: Proposed
**Date**: 2026-03-20
**Deciders**: Tech Lead, Platform Architect

**Context**:
The 10 cross-workstream event contracts must support Phase A (no PHI payloads) and Phase B (PHI payloads) without breaking existing consumers. The Schema Registry enforces compatibility rules. The architecture must accommodate three scenarios: (a) existing events gaining optional PHI-related fields, (b) entirely new PHI-bearing events, and (c) events whose PHI status is ambiguous (e.g., hashed patient IDs).

**Decision**:
Adopt a **three-tier schema evolution strategy**:

1. **Additive fields for enrichment**: Events that are inherently PHI-free but gain optional PHI-derived context in Phase B (Contracts 2, 6, 8, 10) use forward-compatible additive fields. New fields are optional with `null` defaults. Schema version bumps from `1.0` to `1.1`. No consumer redeployment required.

2. **Separate topics for PHI-bearing events**: Events that are entirely PHI-dependent (Contracts 5, 7 — `pms.appointment-completed`, `ws1.funnel-event`) are registered as separate Kafka topics in Phase A but receive no events until Phase B producers activate. Consumer groups for these topics are deployed only in Phase B.

3. **PHI sentinel monitoring**: A dedicated consumer (`phi-sentinel`) subscribes to ALL Phase A topics and runs regex-based PHI pattern detection on every event payload. Any event containing potential PHI (SSN patterns, name + DOB co-occurrence, unmasked phone numbers) triggers an alert. This catches accidental PHI leakage into non-PHI topics.

**Consequences**:
- (+) Zero consumer redeployment for existing Phase A events when Phase B activates
- (+) Clear PHI boundary at the topic level — security teams can audit which topics carry PHI by checking topic metadata
- (+) PHI sentinel provides runtime protection against accidental PHI leakage
- (-) PHI sentinel adds a consumer to every topic — small compute cost (~$50/month for a lightweight consumer)
- (-) Additive fields mean Phase A consumers carry unused `null` fields after Phase B — minor payload bloat

**Alternatives Considered**:
1. **Version headers with consumer routing**: Include a `phi_version` header in events; consumers filter by version. Rejected — adds complexity to every consumer and does not leverage Schema Registry's built-in compatibility.
2. **Separate Kafka clusters for PHI vs. non-PHI**: Run two MSK clusters. Rejected — excessive infrastructure cost and operational complexity. Topic-level separation within a single encrypted cluster is sufficient.
3. **Event payload encryption for PHI fields**: Encrypt PHI fields within the event payload; only authorized consumers have the decryption key. Considered for future enhancement but rejected for initial implementation — adds significant complexity to producer/consumer logic and prevents Schema Registry from validating encrypted fields.

---

### ADR-PHI-003: Feature Flag Architecture for Progressive PHI Enablement

**Status**: Proposed
**Date**: 2026-03-20
**Deciders**: Tech Lead, Engineering Manager

**Context**:
Phase B activates PHI features progressively: per-practice, per-feature, with BAA verification as a prerequisite. The build plan specifies rolling activation at 50 practices/week (B2.6). A practice cannot receive PHI features until its BAA chain is verified. An emergency kill switch must disable all PHI features network-wide within seconds.

**Decision**:
Implement a **three-level hierarchical feature flag system** using LaunchDarkly (or equivalent server-side flag service):

- **Level 1 (Global)**: `phase_b_enabled` (boolean) — master switch for all PHI features. `phase_a_only` (boolean) — emergency kill switch that overrides all other flags.
- **Level 2 (Practice)**: `{practice_id}.phi_enabled` (boolean) — per-practice PHI activation. `{practice_id}.baa_verified` (boolean) — set by compliance workflow after all BAAs for this practice's vendor chain are verified.
- **Level 3 (Feature)**: `{feature}.enabled` (boolean) — per-feature flags (PMS integration, patient comms, Voice AI, funnel analytics, patient photos, automation engine).

A PHI feature is enabled for a practice if and only if ALL three levels evaluate to `true`. The evaluation is: `!phase_a_only AND phase_b_enabled AND practice.phi_enabled AND practice.baa_verified AND feature.enabled`.

All flags are **runtime-evaluated** (server-side SDK check on every request). No build-time flags.

**Consequences**:
- (+) Rolling activation: individual practices can be activated independently
- (+) Emergency disable: `phase_a_only = true` disables all PHI features in < 1 second (LaunchDarkly propagation SLA)
- (+) BAA enforcement: compliance team controls the `baa_verified` flag; engineering cannot bypass
- (+) Granular rollback: if Voice AI has a PHI leak, disable `feature.voice_ai` without affecting patient communications
- (-) Flag evaluation on every request adds ~2-5ms latency (LaunchDarkly SDK caches flags locally, evaluates in-memory)
- (-) Flag management complexity — 260 practices x 6 features = 1,560 practice-feature flag combinations. Requires automation (Temporal workflow for batch activation)
- (-) LaunchDarkly cost (~$500-1,000/month at this scale)

**Alternatives Considered**:
1. **Build-time feature flags**: Compile separate Phase A and Phase B builds. Rejected — requires maintaining two deployment pipelines, cannot do per-practice activation, cannot do emergency disable without a new deploy.
2. **Database-driven flags**: Store flags in Practice Data Hub `config` JSONB column. Rejected — no real-time propagation, requires API call + cache invalidation to update, no built-in audit trail of flag changes.
3. **AWS AppConfig**: Native AWS solution for feature flags. Viable alternative to LaunchDarkly with lower cost but less mature targeting rules and no built-in percentage rollout.

---

### ADR-PHI-004: AI Service Operation Without PHI Filter in Phase A

**Status**: Proposed
**Date**: 2026-03-20
**Deciders**: Tech Lead, Platform Architect

**Context**:
The AI Service (Section 1.4 of technical architecture) includes a PHI filter pipeline that tokenizes PHI before sending prompts to Claude API and detokenizes PHI in responses. In Phase A, no prompts contain PHI. The build plan defers the entire AI Service to B0 as a "PHI extension." However, Phase A has legitimate AI use cases: caption generation (WS2), brief extraction (WS2), AI alerts and anomaly analysis (WS1 Executive Dashboard), and Network Learning Reports (WS1). These use cases require Claude API access without PHI.

**Decision**:
Build the AI Service core in Phase A1 with the PHI filter **disabled but code-present**. The service operates in "PHI-free mode":

1. The `contains_phi` field on the `/api/v1/ai/complete` API is validated. If `true`, the service returns HTTP 403 with `{"error": "PHI processing not enabled. Requires Phase B activation."}`.
2. A defensive PHI detection scan runs on ALL prompts (even those marked `contains_phi: false`). This scan uses regex patterns for SSN, DOB, phone numbers, and a basic NER model for person names co-occurring with medical terms. If potential PHI is detected in a non-PHI request, the request is rejected with HTTP 422 and an alert is sent to the platform engineering team.
3. The PHI filter code is deployed but dormant — activated by feature flag `feature.ai_phi_filter` (Level 3 flag from ADR-PHI-003).
4. Whisper integration is not deployed until B0 (audio transcription is inherently PHI-bearing for dental calls).

**Consequences**:
- (+) Phase A AI use cases are unblocked (caption generation, anomaly analysis, learning reports)
- (+) AI Service infrastructure (prompt registry, token accounting, cost-tier routing, response cache) is validated under real load before Phase B adds PHI complexity
- (+) Defensive PHI detection provides a safety net against accidental PHI in prompts
- (+) Anthropic BAA is NOT required for Phase A — non-PHI prompts do not trigger BAA obligations
- (-) Defensive PHI detection may produce false positives (e.g., a practice named after a person: "Dr. Smith Family Dentistry" flagged as a patient name). Tuning required.
- (-) AI Service team must clearly document which prompts are Phase A vs. Phase B in the prompt registry
- (-) Developers may incorrectly assume the AI Service "handles PHI" because it exists — training required to clarify that Phase A mode rejects PHI

**Alternatives Considered**:
1. **Defer entire AI Service to B0**: Blocks all Phase A AI use cases. WS2 caption generation and WS1 anomaly analysis would require direct Claude API calls from each workstream — defeating the centralization benefit. Rejected.
2. **Build AI Service without any PHI-related code**: Remove PHI filter code entirely from Phase A build; add it in B0. Rejected — having the code present (but dormant) enables faster B0 activation and allows integration testing of the filter pipeline in staging before B0 gate.
3. **Allow PHI without filter in Phase A** (rely on Anthropic's data handling): Rejected — violates the conservative PHI boundary principle. Even if Anthropic's enterprise tier handles data securely, the platform must not transmit PHI until BAAs are executed and the PHI filter is verified.

---

### ADR-PHI-005: Practice Data Hub Schema Design for Dual-Phase Operation

**Status**: Proposed
**Date**: 2026-03-20
**Deciders**: Tech Lead, Platform Architect

**Context**:
The Practice Data Hub is the central data store for the platform. It must serve Phase A (practice/staff operational data) and Phase B (patient records, communications, transcripts) from the same PostgreSQL instance. The schema design must ensure: (a) Phase A services cannot access patient data even accidentally, (b) Phase B activation does not require schema migrations, (c) PHI data has independent encryption, backup, and retention policies.

**Decision**:
Adopt a **dual-schema architecture** within a single RDS Multi-AZ PostgreSQL instance:

- `shared` schema: Practice profiles, staff rosters, regions, assets, message logs, audit logs, consent registry, phone numbers, security incidents. Created and populated in A0/A1.
- `phi` schema: Patient records, patient communications, call transcripts. Created in A0 (empty tables with RLS and encryption configured). Populated starting in B0.

**Schema-level access control**:
```sql
-- Phase A service roles
GRANT USAGE ON SCHEMA shared TO phase_a_service_role;
REVOKE ALL ON SCHEMA phi FROM phase_a_service_role;

-- Phase B service roles (activated in B0)
GRANT USAGE ON SCHEMA shared TO phase_b_service_role;
GRANT USAGE ON SCHEMA phi TO phase_b_service_role;
```

**Cross-schema foreign keys**: `phi.patient_records.practice_id` references `shared.practices.practice_id`. This allows patient queries to be scoped to a practice without duplicating practice data in the PHI schema.

**Encryption layers**:
1. RDS instance encryption (AES-256, AWS managed) — covers both schemas
2. Application-level encryption (AES-256-GCM) on `phi` schema sensitive columns — separate KMS key from `shared` schema encryption
3. S3 SSE-KMS for PHI media (call recordings, patient photos) — dedicated KMS key with annual rotation

**Consequences**:
- (+) PostgreSQL GRANT policies provide database-level enforcement of the PHI boundary — even SQL injection in a Phase A service cannot read PHI tables
- (+) Single RDS instance simplifies operations (one connection string, one backup policy, one monitoring dashboard) while maintaining logical separation
- (+) Phase B activation is a GRANT operation + application deployment, not a schema migration
- (+) Independent KMS keys for PHI schema enable key rotation and access revocation specific to patient data
- (-) Cross-schema JOINs require both schema privileges — only Phase B services can perform patient-practice JOINs
- (-) Database migration tooling (e.g., Prisma, Knex) must be configured to manage two schemas
- (-) Connection pooling (PgBouncer) must route Phase A and Phase B service roles to appropriate connection pools

**Alternatives Considered**:
1. **Single schema with RLS only**: All tables in `shared` schema; RLS policies restrict patient table access to Phase B roles. Rejected — RLS is per-table, not per-schema; easier to accidentally misconfigure. Schema-level GRANT is a stronger boundary.
2. **Separate RDS instances**: Dedicated RDS for PHI data. Rejected — doubles RDS cost ($400-800/month), complicates cross-database queries, and adds operational burden (two monitoring dashboards, two backup policies, two failover procedures).
3. **DynamoDB for PHI data**: Use DynamoDB for patient records (key-value access pattern, fine-grained IAM policies). Rejected — the platform is standardized on PostgreSQL; adding DynamoDB introduces a new data store with different query patterns, backup procedures, and team skill requirements.

---

## 6. Risk Assessment

### 6.1 Technical Debt from Dual-Phase Approach

| Debt Category | Severity | Description | Mitigation |
|---------------|----------|-------------|------------|
| **Temporary SMS gateway** | Medium | If the build plan uses a non-HIPAA SMS gateway for Phase A content capture (instead of building Communications Service in A1), the migration to the shared Communications Service in B0 requires re-integrating WS2's capture prompt code. | Build Communications Service in A1 with limited scope (see Section 1.2 recommendation). Eliminates this debt entirely. |
| **AI Service dual-mode code** | Low | The AI Service carries PHI filter code that is dormant in Phase A. Developers must maintain awareness of which code paths are active. | Feature flag on the PHI filter. Clear code comments and architectural documentation. PHI filter code is in a separate module (`/phi-filter/`) not imported by Phase A handlers. |
| **Dashboard placeholder slots** | Low | Executive Dashboard, Portfolio Heatmap, Practice Deep Dive all have reserved slots for Phase B widgets (empty states saying "Available in Phase B"). These are UI debt until Phase B fills them. | Designed as intentional "Phase B invitations" per experience architecture principle #7. Not debt — deliberate product design. |
| **Schema forward-compatibility constraints** | Low | Phase A event schemas must not include fields that conflict with planned Phase B fields. Schema planning in A0 must anticipate Phase B field names and types. | Phase B schema fields are defined in A0 (registered in Schema Registry as future optional fields) even though no producer emits them. This locks field names and types before any consumer is built. |
| **Compliance Middleware in monitoring mode** | Low | Phase A Compliance Middleware logs access events but does not enforce PHI controls. A code change is required in B0 to switch from monitoring to enforcement. | Feature flag: `compliance.enforcement_mode` (values: `monitor`, `enforce`). Default `monitor` in Phase A. Switched to `enforce` in B0. No code change — configuration-driven. |

### 6.2 Migration Risks When Enabling Phase B

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **PHI leaking into Phase A data paths after B0 activation** | Medium | High | PHI sentinel consumer monitors all non-PHI topics. Schema Registry validation rejects non-conforming payloads. Compliance Middleware in enforcement mode blocks unauthorized PHI access. Triple defense-in-depth. |
| **Practice Data Hub performance degradation when PHI tables are populated** | Low | Medium | PHI queries run against the `phi` schema with its own indexes. Phase A queries against `shared` schema are unaffected. Read replicas absorb dashboard query load. At 260 practices, even with 500 patients per practice (130,000 patient records), PostgreSQL handles this easily. |
| **Kafka consumer group rebalancing when Phase B consumers deploy** | Low | Low | Phase B consumers are deployed to NEW consumer groups (`ws2-content-engine-phi`, distinct from `ws2-content-engine`). No rebalancing of existing Phase A consumer groups. |
| **Feature flag misconfiguration enabling PHI for a practice without BAA** | Medium | High | The `baa_verified` flag is set ONLY by the compliance workflow (Temporal), not by engineering. The workflow checks BAA status against a verified registry before setting the flag. Manual override requires `sga_admin` role + audit log entry. |
| **Temporal workflow payload encryption not applied to Phase B workflows** | Medium | High | Application-level payload encryption is enforced by a Temporal interceptor (middleware) deployed in B0. The interceptor rejects any workflow execution that contains PHI patterns in unencrypted payloads. Cannot be bypassed without removing the interceptor (requires deployment). |
| **PMS Integration Service flooding Kafka with patient events on first sync** | Medium | Medium | Nightly PMS sync (Sikka ONE) processes practices in batches of 50 with rate limiting. NexHealth webhook receiver has a per-practice backpressure mechanism. Initial sync is throttled to prevent Kafka partition hotspots. Consumer lag monitoring alerts at 60-second lag. |

### 6.3 Performance Implications of PHI Layer Addition

| Component | Phase A Performance | Phase B Delta | Mitigation |
|-----------|-------------------|---------------|------------|
| **PostgreSQL query latency** | < 10ms (practice queries against `shared` schema) | +5-15ms for cross-schema JOINs with `phi` tables; +10-20ms for application-level decryption of encrypted columns | Index `phi.patient_records(practice_id)`. Use materialized views for common patient-practice JOINs. Decrypt PHI fields only when rendering to authorized users — never in background processing. |
| **Kafka throughput** | ~3 events/sec average (Phase A: 8 topics, ~260K events/day) | +50-100% event volume from PMS events (~260K PMS events/day at 260 practices x 50 appointments/day x 2 events/appointment) | MSK handles 500K events/day trivially. 16 partitions per topic supports up to 16 concurrent consumers. No capacity concern. |
| **AI Service latency** | ~200-500ms (Claude Haiku/Sonnet, non-PHI prompts) | +50-80ms for PHI filter tokenization/detokenization; +100-300ms for Whisper audio transcription API call | PHI filter adds fixed overhead. Whisper latency is dominated by audio length, not infrastructure. Cost-tier routing sends PHI-bearing prompts to appropriate models. |
| **API gateway latency** | ~5ms (JWT validation, RBAC check) | +5-10ms for Compliance Middleware PHI detection and consent verification on PHI-bearing requests | Compliance Middleware runs as Fastify plugin (in-process, no network hop). PHI detection is regex-based (< 2ms). Consent check hits Redis cache (< 3ms). |

### 6.4 Testing Strategy for Phase B Enablement

**Problem**: How do you test PHI features in staging without using real patient data?

**Strategy: Synthetic PHI + PHI-Realistic Test Fixtures**

| Test Layer | Approach | Detail |
|-----------|---------|--------|
| **Unit tests** | Synthetic PHI factory | A `@sgadental/test-fixtures` npm package generates realistic but fake patient records (names from Faker.js, synthetic SSNs, fake insurance IDs). Every PHI-bearing service uses this factory in unit tests. |
| **Integration tests** | Staging PMS simulator | A mock NexHealth/Sikka ONE service (`pms-simulator`) generates realistic appointment events, patient records, and webhook callbacks. Deployed in staging only. Produces to the same Kafka topics as the real PMS Integration Service. |
| **E2E tests** | Synthetic practice with synthetic patients | A dedicated "test practice" (SGA-TEST-001) in staging has 50 synthetic patients with full lifecycle: appointments, communications, call transcripts. All data is clearly marked as synthetic (patient names prefixed with "TEST-"). |
| **PHI red-team test** | Controlled PHI injection | During Gate B0, the security team injects synthetic PHI into the system and verifies: (a) PHI filter strips it from AI prompts, (b) Compliance Middleware logs the access, (c) PHI does not appear in non-PHI topics, (d) PHI does not appear in application logs. |
| **Load test** | Synthetic PMS event firehose | Simulate 260 practices each generating 50 appointments/day. Verify Kafka throughput, consumer lag, PostgreSQL write performance, and ClickHouse materialized view refresh under PHI-bearing load. |
| **Consent verification test** | Synthetic opt-out scenarios | Inject opt-out events for test patients. Verify Communications Service blocks all subsequent messages. Verify opt-out propagates across all channels within SLA (< 24 hours). |

**Critical rule**: Real patient data NEVER enters staging or development environments. All PHI testing uses synthetic data generated by the test fixture library. The `pms-simulator` is the only source of "patient" data in non-production environments.

---

## 7. Verdict

### Is the PHI-free-first approach technically sound?

**Yes. The approach is architecturally sound, with three required adjustments.**

The PHI-free-first strategy correctly identifies that 6 of 10 shared services operate without modification in Phase A. The data model design (separate `phi` schema), event architecture (forward-compatible schemas + separate PHI topics), and feature flag system (hierarchical, runtime-evaluated) provide clean Phase B activation without requiring Phase A rework.

### Required Adjustments

1. **Build AI Service core in A1, not B0.** The build plan over-defers the AI Service. Phase A has real AI use cases (caption generation, anomaly analysis, learning reports) that are blocked without it. The PHI filter is a separate module activated by feature flag in B0. The core service (prompt registry, routing, caching, Claude API client) is PHI-free and should ship in A1 Weeks 5-6.

2. **Build Communications Service in A1 with limited scope.** Rather than a throwaway non-HIPAA SMS gateway for Phase A content capture, build the real Communications Service with its PHI-independent components (SMS send API, number management, delivery tracking, message log). Omit the consent registry, HIPAA encryption, and patient routing until B0. This eliminates a migration debt and gives WS2 the production API interface from Day 1.

3. **Deploy Compliance Middleware in monitoring mode in A1.** The audit log infrastructure and access logging should be operational from Phase A. This validates the middleware under real load, populates the Compliance Readiness Dashboard (NI-9) with real data, and reduces B0 activation risk. Switch from `monitor` to `enforce` mode via feature flag in B0.

### No Blockers Identified

There are no architectural blockers to the PHI-free-first approach. The three adjustments above are optimizations, not blockers. The approach can proceed as-designed; these adjustments reduce Phase B transition risk and eliminate a migration debt.

### Confidence Level

| Area | Confidence | Rationale |
|------|-----------|-----------|
| Data model (PostgreSQL dual-schema) | **High** | Separate schemas with GRANT-level isolation is a well-established PostgreSQL pattern. Empty PHI tables with pre-configured RLS is clean. |
| Event architecture (Kafka) | **High** | Forward-compatible schema evolution is Kafka's core strength. Separate topics for PHI-bearing events is the right call. PHI sentinel provides runtime safety. |
| Feature flag system | **High** | Hierarchical flags with BAA verification gate and emergency kill switch cover all activation scenarios. LaunchDarkly is battle-tested for this. |
| Service phasing (6+4 split) | **Medium-High** | 6 services are clearly PHI-free. The remaining 4 are correctly deferred to B0. The two recommendations (AI Service and Communications Service partial A1 build) improve the split without adding risk. |
| Phase B migration | **Medium** | The design is sound but unproven. The primary risk is accidental PHI leakage when B0 activates — mitigated by triple defense-in-depth (PHI sentinel, Schema Registry, Compliance Middleware). Real confidence comes from the B0 red-team test. |
| Performance at scale | **Medium-High** | 260 practices is modest scale. PostgreSQL, Kafka, and the AWS stack handle this without concern. Performance risk is localized to PHI-specific operations (encryption overhead, PHI filter latency) and is within acceptable bounds (+50-80ms per PHI-bearing request). |

---

*Prepared by Tech Lead Agent | Citizen & Partners | bilt.studio*
*This review supplements the PHI-free-first build plan and should be validated by the Platform Architect and HIPAA Compliance Counsel before finalizing the Phase A build sequence.*
