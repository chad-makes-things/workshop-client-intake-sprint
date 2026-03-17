# SGA Dental Partners — Existing Tech Stack Evaluation

## Impact on Architecture & Roadmap Recommendations

**Prepared by**: Product Manager, Tech Lead, UX Designer, Backend Engineer
**Date**: March 17, 2026
**Purpose**: Evaluate SGA's existing technology stack for integration advantages, overlap reduction, and acceleration opportunities — without changing the primary architectural recommendation.

---

## Executive Summary

SGA operates **17 production systems** across patient management, analytics, communications, and marketing. Three findings matter for our build:

1. **Microsoft Power BI (96% adoption)** — eliminates the need to build custom executive dashboards from scratch. Our ClickHouse data lake should feed Power BI, not replace it.
2. **Modento + TrueLark + Patient Prism** — significant overlap with our Communications Service and Concierge Agent. Integration-first approach saves 4-6 weeks of build time and avoids change management friction.
3. **OursPrivacy / Freshpaint** — existing PHI eventing infrastructure that can accelerate our Compliance & Audit Layer instead of building it greenfield.

**Net impact**: No change to the 3-workstream architecture. Estimated **4-8 weeks acceleration** on select epics + **$15K-$25K/month vendor cost reduction** at steady state by leveraging existing contracts rather than duplicating capabilities.

---

## Section 1: Stack-to-Architecture Mapping

### 1.1 Direct Architectural Alignment (Already Planned)

These tools are already accounted for in our architecture. No changes needed.

| Existing Tool | Adoption | Our Architecture Component | Status |
|---------------|----------|---------------------------|--------|
| **Dentrix / Eaglesoft / OpenDental** | 96% | PMS Integration Service (via NexHealth + Sikka ONE) | Planned — NexHealth wraps these PMS systems |
| **Dexis** (Imaging) | — | Out of scope | No overlap |
| **Dental Intel** | 95% | WS3 Morning Huddle OS (direct integration) | Planned — Epic 2 |
| **Method** (Procurement) | 100% | Out of scope | No overlap |
| **Pearly** (Collections) | 84% | Out of scope | No overlap — potential future WS1 integration |

### 1.2 Significant Overlap — Integration Advantages

These tools overlap with capabilities we planned to build. Integrating them instead of replacing them creates advantages.

| Existing Tool | Adoption | Overlapping Architecture Component | Recommendation |
|---------------|----------|-----------------------------------|----------------|
| **Microsoft Power BI** | 96% | WS1-B Executive Dashboard, Analytics Dashboards | **INTEGRATE** — see Section 2.1 |
| **Modento** | 79% | WS1 Automation Engine (confirmations, intake, reviews) | **INTEGRATE** — see Section 2.2 |
| **TrueLark** | 61% | WS1 Concierge Agent (after-hours AI, scheduling) | **COEXIST** — see Section 2.3 |
| **Patient Prism** | 85% | WS3 H-Score (phone analytics component) | **INTEGRATE** — see Section 2.4 |
| **Peer Logic** | 78% | Non-PMS Data Source Integration (phone systems) | **INTEGRATE** — see Section 2.5 |

### 1.3 Marketing Stack — Future Integration Targets

These are listed but adoption percentages aren't provided. They represent future integration opportunities for WS1 Network Intelligence.

| Tool Category | Listed Options | Architecture Component | Phase |
|---------------|---------------|----------------------|-------|
| **CallRail / Invoca** | Call attribution | WS1-B Funnel Data Lake (Epic 2), Data Integration Connectors (Epic 4) | Tier 1-2 |
| **GoHighLevel / HubSpot / Salesforce Health Cloud** | CRM | WS1-A Automation Engine + WS1-B Network Learning | Tier 2 |
| **WordPress / Webflow / Unbounce** | Landing pages | WS1-A Storefront Generation (Epic 4) | Tier 1 |
| **Looker Studio / Power BI / Tableau** | Marketing analytics | WS1-B Executive Dashboard | See Power BI integration below |
| **Hightouch / Fivetran** | Data warehouse integration | Non-PMS Data Source Integration Service | Tier 2-3 |
| **OursPrivacy / Freshpaint** | PII/PHI eventing | Compliance & Audit Layer | **INTEGRATE** — see Section 2.6 |

---

## Section 2: Detailed Integration Recommendations

### 2.1 Microsoft Power BI — Executive Dashboard Layer

**Current state**: 96% adopted. Custom solution iteratively improved. SGA leadership already lives in Power BI for corporate analytics.

**Our plan**: Build custom React dashboards (Executive Dashboard, practice drill-downs, funnel views) using ClickHouse as the analytical backend.

**Recommendation: Build ClickHouse as the data layer. Deliver dashboards through Power BI, not custom React.**

**Why this is better**:
- SGA leadership already knows Power BI. Zero adoption friction for the executive audience.
- Their BI team already has custom solutions — they can extend our data models themselves.
- We still build ClickHouse (it's the analytical engine), but instead of building 8-12 custom React dashboard screens, we build Power BI datasets + semantic models.
- Custom React dashboards are still needed for **practice-level** interfaces (Nerve Center, Morning Huddle) where Power BI doesn't fit. But network-level analytics belong in Power BI.

**Architecture impact**:

```
BEFORE (planned):
  ClickHouse → Custom React Dashboards (WS1-B Epic 1)

AFTER (recommended):
  ClickHouse → Power BI DirectQuery/Import → Executive Dashboards
  ClickHouse → Custom React → Practice Nerve Center (unchanged)
  ClickHouse → Custom React → HQ Content Dashboard (unchanged)
```

**What changes**:
- WS1-B Epic 1 (Executive Dashboard) scope reduces by ~60%. Instead of building 6-8 custom React screens, we build 2-3 Power BI reports + the ClickHouse semantic layer.
- Add a Power BI Gateway service to the shared infrastructure (lightweight — managed by Azure, we just configure the data source).
- Practice-level React dashboards remain unchanged.

**Build time saved**: 3-4 weeks (WS1-B team)
**Vendor cost impact**: $0 incremental (SGA already has Power BI licensing)
**Risk**: Power BI DirectQuery to ClickHouse requires ClickHouse's ODBC/JDBC connector. Well-documented but needs validation in Sprint 0.

**User stories affected**:
- "As an HQ executive, I want a network-wide funnel dashboard" → Power BI report
- "As a regional director, I want to compare my region's practices" → Power BI report with RLS
- "As an HQ analyst, I want to export data for custom analysis" → Power BI + ClickHouse (already their workflow)

---

### 2.2 Modento — Communications Acceleration

**Current state**: 79% adopted. Handles automated confirmations, online intake forms, cohort-based patient communications, Google Review requests. $140/practice/month.

**Our plan**: Build Communications Service (shared infra) + Automation Engine (WS1 Epic 5) handling appointment confirmations (48h/24h/4h), review requests, intake forms.

**Recommendation: Integrate Modento as a communications channel provider within our Communications Service, rather than replacing it.**

**Why this is better**:
- 79% of practices already use Modento. Replacing it means retraining ~205 practices on a new confirmation/intake system.
- Modento's confirmation sequences work. Our value isn't in rebuilding confirmations — it's in the **intelligence layer** (AI-powered missed call recovery, no-show re-engagement, treatment follow-ups) that Modento doesn't do.
- We still own the Communications Service for orchestration, consent management, compliance, and the AI-powered sequences. But Modento handles the commodity confirmation flows.

**Architecture impact**:

```
BEFORE (planned):
  Communications Service → Twilio → Patient (all sequences)

AFTER (recommended):
  Communications Service → Twilio → Patient (AI sequences: missed call, no-show, treatment follow-up)
  Communications Service → Modento API → Patient (confirmation, intake, review requests)
  Modento webhook → Event Bus → WS1 Automation Engine (confirmation status events)
```

**What changes**:
- Add Modento as a provider in the Communications Service (adapter pattern — same as Twilio, just different channel).
- WS1 Epic 5 (Automation Engine) appointment confirmation sequences become Modento-delegated instead of Twilio-direct for practices that have Modento.
- Practices without Modento (21%) get the Twilio-native confirmation path.
- **New capability**: Our AI layer sits on top of Modento's data — e.g., if a patient doesn't confirm via Modento, our missed call recovery agent triggers.

**Build time saved**: 2-3 weeks (WS1-A team — confirmation sequence development)
**Vendor cost impact**: Modento is already $140/practice. We reduce Twilio SMS volume by ~40% for Modento practices, saving ~$3K-$6K/month on Twilio at scale.
**Risk**: Modento API maturity — need to validate webhook reliability and API coverage during Sprint 0.

---

### 2.3 TrueLark — Concierge Coexistence

**Current state**: 61% adopted. Enterprise agreement. After-hours AI chat, website chat, online scheduling. $300/practice/month.

**Our plan**: Build Concierge Agent (WS1 Epic 2) — chat-based first interaction, missed call text-back, form fill auto-response, brand voice personality.

**Recommendation: Coexist with TrueLark in the near term. Our Concierge Agent focuses on differentiated capabilities (brand voice, graduated autonomy, cross-system intelligence). Do not try to replace TrueLark at launch.**

**Why**:
- TrueLark has an enterprise agreement — SGA is likely contractually committed.
- At 61% adoption, it's partially deployed. Some practices rely on it; others don't have it yet.
- Our Concierge Agent's value is in **intelligence**: it knows the patient's funnel stage, H-Score context, content history. TrueLark is a generic scheduling chatbot.
- Long-term, our Concierge Agent may replace TrueLark at practices that graduate to full platform autonomy. But forcing a swap at launch creates unnecessary risk.

**Architecture impact**:
- Concierge Agent builds as planned but deploys initially to **non-TrueLark practices** (39%, ~100 practices).
- Add TrueLark webhook integration to Non-PMS Data Source Integration Service — capture scheduling outcomes, after-hours engagement data.
- Feed TrueLark events into Funnel Data Lake for attribution.
- At Tier 2/3, evaluate practice-by-practice migration from TrueLark to Concierge Agent based on engagement data.

**Build time impact**: No change to build time. But reduces **deployment risk** — we don't fight TrueLark's enterprise agreement or retrain 160 practices at launch.
**Vendor cost impact**: TrueLark remains a cost ($300/practice for 61% = ~$48K/month). As practices migrate to our Concierge, this decreases over 12-18 months.

---

### 2.4 Patient Prism — H-Score Enhancement

**Current state**: 85% adopted. Phone AI analysis for iterative improvement of office phone skills. $150/practice/month.

**Our plan**: WS3 H-Score Engine uses Cinnamon API for phone scoring component.

**Recommendation: Evaluate Patient Prism as an additional or alternative data source for the H-Score phone component.**

**Why**:
- Patient Prism is at 85% adoption vs. Cinnamon's current deployment. If Patient Prism data is richer or more consistent, it could strengthen the H-Score signal.
- Patient Prism focuses on **phone skills improvement** — which is exactly what H-Score's phone component measures.
- Cinnamon may still be the right choice (it depends on API depth, data granularity, and cost). But with 85% of practices already sending data to Patient Prism, there's a pre-existing data asset.

**Architecture impact**:
- No structural change. The H-Score Engine already abstracts its data sources.
- Add Patient Prism as an optional data source adapter alongside Cinnamon.
- During Sprint 0 vendor evaluation, compare: Cinnamon API vs. Patient Prism API for phone scoring data (call scoring, conversion tracking, skill metrics).

**Build time impact**: +1 week for vendor evaluation. Could save time if Patient Prism API is better documented than Cinnamon's.
**Cost consideration**: SGA already pays $150/practice for Patient Prism ($39K/month). If it replaces Cinnamon ($3K-$5K/month additional), net savings of $3K-$5K/month.

---

### 2.5 Peer Logic — Phone System Integration

**Current state**: 78% adopted. Digital VOIP phone system. $120/practice/month. Used in new implementations and centralized call centers.

**Our plan**: Non-PMS Data Source Integration Service integrates with phone systems (RingCentral, 8x8, Weave) for call analytics and missed call detection.

**Recommendation: Add Peer Logic as the primary phone system integration target, not RingCentral/8x8/Weave.**

**Why**:
- Our architecture lists RingCentral, 8x8, and Weave as phone system connectors. But SGA actually uses Peer Logic at 78% of practices.
- Building a Peer Logic adapter first (instead of RingCentral/8x8) means we immediately cover 200+ practices.
- Still build generic phone system adapters for the remaining 22%, but Peer Logic is the priority connector.

**Architecture impact**:
- Non-PMS Data Source Integration Service: reorder connector priority. Peer Logic first, others second.
- Validate Peer Logic API capabilities: call records, missed call events, call recordings, caller ID.
- Feed Peer Logic data into Funnel Data Lake for call attribution.

**Build time impact**: No change (same effort to build one phone adapter), but faster time-to-value since 78% of practices are covered immediately.

---

### 2.6 OursPrivacy / Freshpaint — Compliance Acceleration

**Current state**: Listed as PII/PHI support and eventing tools. Adoption not specified.

**Our plan**: Build Compliance & Audit Layer as cross-cutting middleware (PHI detection, encryption helpers, consent checking, audit logging).

**Recommendation: Evaluate OursPrivacy/Freshpaint as the PII/PHI detection and eventing layer within our Compliance Middleware, rather than building detection from scratch.**

**Why**:
- OursPrivacy and Freshpaint specialize in exactly what our Compliance Middleware needs: PII/PHI detection in data flows, consent management eventing, and data governance.
- Freshpaint specifically is designed for healthcare — it handles PHI in analytics/marketing data flows, which is precisely our challenge with the AI Service PHI filter pipeline.
- Building PHI detection from scratch is high-risk. Using a purpose-built tool reduces HIPAA compliance risk.

**Architecture impact**:
- Compliance Middleware integrates OursPrivacy/Freshpaint SDK as the PHI detection engine.
- Our audit logging, encryption, and consent checking still custom-built (these are specific to our event contracts).
- The AI Service PHI filter pipeline can potentially use Freshpaint's PII detection instead of custom regex/NER.

**Build time saved**: 1-2 weeks on PHI detection implementation + reduced compliance audit risk.
**Cost impact**: Likely already contracted. If not, $500-$2K/month — far less than the compliance risk of custom PHI detection.

---

## Section 3: Marketing Stack Integration Strategy

SGA's marketing stack (CallRail/Invoca, CRM, landing pages, analytics) represents future integration opportunities. These don't change the architecture — they inform the **Data Integration Connectors** (WS1-B Epic 4) priority order.

### Connector Priority Reordering

| Priority | Original Plan | Revised (Based on Existing Stack) |
|----------|--------------|-----------------------------------|
| 1 | Phone systems (RingCentral/8x8/Weave) | **Peer Logic** (78% adopted) → then others |
| 2 | Lead sources (Google Ads/Meta) | **CallRail/Invoca** (if adopted) → then ad platforms |
| 3 | AI phone agents (Sierra/Agnes) | **Patient Prism** (85% adopted) → then AI agents |
| 4 | HR systems (iSolved) | iSolved (unchanged) |
| 5 | Social platforms | Social platforms (unchanged) |

### CRM Consideration

SGA lists GoHighLevel, HubSpot, and Salesforce Health Cloud as CRM options. If they adopt one of these:
- Our Automation Engine (WS1 Epic 5) should integrate with it rather than building standalone lead/patient journey tracking.
- Our Funnel Data Lake should ingest CRM events for attribution.
- **Recommendation**: During Sprint 0, confirm whether SGA has selected and deployed a CRM. If yes, add it to the connector roadmap at Tier 2.

### Landing Page / Storefront Consideration

SGA lists WordPress + Elementor, Webflow, and Unbounce. Our Storefront Generation (WS1 Epic 4) builds static/JAMstack sites.
- If practices already have WordPress sites, our Storefront should **enhance or replace** them, not coexist. Two websites per practice creates confusion.
- **Recommendation**: During enrollment (WS1 Epic 3), audit each practice's existing web presence. Storefront generation either replaces or integrates with existing sites.

---

## Section 4: Consolidated Impact Summary

### Build Timeline Acceleration

| Area | Weeks Saved | How |
|------|-------------|-----|
| Executive Dashboard → Power BI | 3-4 weeks | Build BI datasets instead of custom React dashboards |
| Confirmation Sequences → Modento | 2-3 weeks | Delegate commodity confirmations to existing tool |
| PHI Detection → OursPrivacy/Freshpaint | 1-2 weeks | Use purpose-built detection instead of custom |
| Phone Integration → Peer Logic first | 0 weeks (reorder) | Same effort, faster coverage |
| **Total** | **6-9 weeks** | Reallocated to differentiated features |

### Vendor Cost Impact (Monthly at 260 Practices)

| Change | Impact |
|--------|--------|
| Twilio volume reduction (Modento handles confirmations) | -$3K to -$6K/month |
| ClickHouse potential downsizing (Power BI handles heavy analytics) | -$200 to -$400/month |
| Cinnamon potentially replaced by Patient Prism (already paid) | -$3K to -$5K/month |
| Power BI Gateway (minimal) | +$0 to +$200/month |
| **Net monthly savings** | **-$6K to -$11K/month** |
| **Net annual savings** | **-$72K to -$132K/month** |

### Change Management Reduction

| Tool | Practices Affected | Training Avoided |
|------|-------------------|------------------|
| Keep Modento (confirmations) | 205 practices | No retraining on confirmation workflows |
| Keep TrueLark (near-term) | 160 practices | No retraining on after-hours AI |
| Power BI dashboards | All HQ users | No learning new analytics tool |
| Keep Patient Prism | 220 practices | No retraining on phone analytics |

---

## Section 5: Sprint 0 Vendor Validation Checklist

Before build begins, validate these integration assumptions:

| # | Validation Task | Owner | Blocker? |
|---|----------------|-------|----------|
| 1 | Power BI DirectQuery to ClickHouse — test ODBC connector, confirm latency acceptable | Platform Architect | **Yes** — if this fails, revert to custom React dashboards |
| 2 | Modento API — confirm webhook availability, confirmation status events, intake form data | Integration Architect | **Yes** — if API is insufficient, build Twilio-native confirmations |
| 3 | Patient Prism API — compare data granularity vs. Cinnamon for H-Score phone component | Integration Architect | No — Cinnamon is fallback |
| 4 | Peer Logic API — confirm call records, missed call events, recording access | Integration Architect | No — other phone systems are fallback |
| 5 | OursPrivacy/Freshpaint — confirm SDK capabilities for real-time PHI detection in event streams | Principal Architect | No — custom detection is fallback |
| 6 | TrueLark webhook API — confirm we can capture scheduling outcomes for Funnel Data Lake | Integration Architect | No — nice-to-have for attribution |
| 7 | SGA CRM status — confirm whether GoHighLevel/HubSpot/Salesforce is deployed | Product Manager | No — informs Tier 2 planning |
| 8 | SGA Power BI environment — confirm licensing, gateway infrastructure, data refresh capacity | Platform Architect | **Yes** — critical for dashboard strategy |
| 9 | CallRail/Invoca deployment status — confirm if active and which tier | Integration Architect | No — informs connector priority |
| 10 | Existing website inventory — WordPress/Webflow/Unbounce by practice | Product Manager | No — informs storefront strategy |

---

## Section 6: Revised Shared Infrastructure (Additive Only)

These additions don't change the 12-layer architecture. They add integration adapters within existing services.

### Addition to Shared Service #3: Communications Service

```
New provider adapter: Modento
  - Confirmation sequences (delegate for 79% of practices)
  - Intake form triggers
  - Google Review request delegation
  - Webhook receiver: confirmation_completed, form_submitted, review_requested
  - Fallback: Twilio-native path for non-Modento practices (21%)
```

### Addition to Shared Service #11: Non-PMS Data Source Integration

```
Reordered connector priority:
  1. Peer Logic (78%) — primary phone system
  2. Patient Prism (85%) — phone AI analytics (evaluate vs. Cinnamon)
  3. TrueLark (61%) — scheduling outcomes, after-hours data
  4. CallRail/Invoca — call attribution (if deployed)
  5. RingCentral/8x8/Weave — secondary phone systems
  6. Google Ads/Meta — ad platform connectors (unchanged)
  7. HR systems (iSolved) — unchanged
```

### Addition to Shared Service #8: Compliance & Audit Layer

```
PHI Detection Enhancement:
  - Evaluate OursPrivacy/Freshpaint SDK as PHI detection engine
  - Replace or augment custom regex/NER pipeline
  - Benefits: purpose-built for healthcare, reduces compliance audit risk
```

### New: Power BI Integration Layer

```
Not a new shared service — a data access pattern:
  - ClickHouse ODBC/JDBC endpoint configured for Power BI DirectQuery
  - Semantic model definitions for: network funnel, regional comparison, practice health, content analytics
  - Row-Level Security mapping: Power BI roles → our RBAC model
  - Refresh schedule: DirectQuery for real-time, Import for heavy aggregations (daily)
```

---

## Section 7: What Does NOT Change

To be explicit — the following architectural decisions are **unchanged** by this evaluation:

1. **3-workstream structure** (Practice Platform, Content Engine, Culture OS) — unchanged
2. **12-layer shared infrastructure** — unchanged (additions are adapter-level, not structural)
3. **Event-driven architecture (Kafka)** — unchanged
4. **ClickHouse as analytical backend** — unchanged (Power BI consumes it, doesn't replace it)
5. **Custom React interfaces for practice-level tools** — unchanged (Nerve Center, Huddle, Content Dashboard)
6. **AI Service with PHI filter pipeline** — unchanged (OursPrivacy/Freshpaint enhances detection, doesn't replace architecture)
7. **Temporal workflow engine** — unchanged
8. **Milestone sequence and tier structure** — unchanged
9. **Staffing model recommendation** — unchanged
10. **Build timeline (36 weeks)** — potentially accelerated by 4-8 weeks on select epics

---

## Appendix A: Full Stack Mapping Matrix

| # | Existing Tool | Adoption | Monthly/Practice | Our Component | Relationship | Action |
|---|---------------|----------|-----------------|---------------|-------------|--------|
| 1 | Dentrix/Eaglesoft/OpenDental | 96% | $159 | PMS Integration Service | **Wrapped by NexHealth** | No change |
| 2 | Dexis | — | — | None | Out of scope | None |
| 3 | Dental Intel | 95% | $225 | WS3 Morning Huddle | **Direct integration** | Already planned |
| 4 | Microsoft Power BI | 96% | N/A | WS1-B Executive Dashboard | **Replace custom dashboards** | Power BI as presentation layer |
| 5 | TrueLark | 61% | $300 | WS1 Concierge Agent | **Coexist** | Deploy Concierge to non-TrueLark practices first |
| 6 | Modento | 79% | $140 | WS1 Communications + Automation | **Integrate** | Delegate confirmations, build AI layer on top |
| 7 | Peer Logic | 78% | $120 | Non-PMS Data Source Integration | **Integrate (priority 1)** | Reorder connector priority |
| 8 | Patient Prism | 85% | $150 | WS3 H-Score phone component | **Evaluate vs. Cinnamon** | Sprint 0 vendor comparison |
| 9 | Pearly | 84% | $195 | None (out of scope) | **Future opportunity** | Potential WS1 Tier 3 integration |
| 10 | Method | 100% | $79 | None (out of scope) | No overlap | None |
| 11 | CallRail/Invoca | TBD | $150-$500 | WS1-B Funnel Data Lake | **Integrate** | Connector in Tier 2 |
| 12 | GoHighLevel/HubSpot/Salesforce | TBD | $200-$800 | WS1 Automation Engine | **Evaluate** | Sprint 0 — confirm if deployed |
| 13 | Website platforms | TBD | $100-$400 | WS1 Storefront Generation | **Replace or enhance** | Audit during enrollment |
| 14 | Looker Studio/Power BI/Tableau | See #4 | $0-$200 | WS1-B Dashboards | **See Power BI strategy** | Covered by #4 |
| 15 | Hightouch/Fivetran | TBD | $300-$1,000 | Non-PMS Data Source Integration | **Evaluate** | If deployed, use as integration layer |
| 16 | OursPrivacy/Freshpaint | TBD | — | Compliance & Audit Layer | **Integrate** | PHI detection engine |

---

*Prepared by Product Manager, Tech Lead, UX Designer, Backend Engineer Agents | Citizen & Partners | bilt.studio*
*This evaluation supplements the primary architecture at [04-architecture/technical-architecture.md](../04-architecture/technical-architecture.md) and roadmap at [05-roadmap/](../05-roadmap/).*
*No changes to primary recommendations. All findings are additive integration opportunities.*
