# QC-3: Product Strategist PMF Architecture Assessment

**Date**: 2026-03-14
**Reviewer**: Product Strategist Agent
**Input Document**: Technical Architecture (Tech Lead)
**Reference Baseline**: Consolidation Feature Map, Section 4 (PMF Score: 22/25)

---

## Review Question: Does the Technical Architecture Preserve, Improve, or Degrade the 22/25 PMF Score?

---

## 1. Does the Architecture Support the PMF Thesis?

The consolidation's PMF advantage was concentrated in two characteristics:

- **Scalability (+2, from 2 to 4)**: Eliminating 6-way data synchronization
- **Unfair advantage (+2, from 3 to 5)**: Cross-domain data insights that were siloed

### Scalability Verdict: DELIVERED

The architecture directly eliminates the 6-way synchronization problem through:

- **Practice Data Hub (Section 1.1)**: Single `shared.practices` table with a well-defined API. The consolidation feature map promised "one update propagates everywhere" and the architecture delivers exactly that. When a practice changes its name, one `PUT /api/v1/practices/:id` call emits a `practice.profile.updated` event to Kafka, and all consumers receive it. No multi-system update cascading.
- **Unified PMS Integration (Section 1.2)**: One service, two vendor adapters (NexHealth + Sikka ONE), one canonical event schema. The original 6 sprints designed 4 independent PMS approaches. The architecture collapses this to a single service with a single API surface. The dual-vendor strategy (ADR-01) is pragmatic — NexHealth for real-time read-write, Sikka ONE for coverage breadth.
- **Communications Service (Section 1.3)**: Centralized Twilio with unified consent registry. The architecture explicitly prevents the "two workstreams sending from different numbers to the same patient" failure mode.

The scaling math is also sound: 260 practices, ~13K messages/day, ~260K Kafka events/day, ~1,300 assets/week. None of these numbers stress the chosen technologies. PostgreSQL with read replicas and PgBouncer handles this comfortably. Kafka (MSK) is overspec'd at this volume (designed for millions/second), but MSK Serverless mitigates cost waste and the architecture gains event replay and durability.

**Assessment**: The architecture does not merely "support" scalability — it makes the consolidation's scalability promise structurally enforceable. A workstream physically cannot maintain its own practice database or PMS connector because those APIs do not exist at the workstream level. This is architecture-enforced consolidation, not policy-enforced consolidation. That is stronger than what the PMF assessment assumed.

### Unfair Advantage Verdict: DELIVERED, WITH ACCELERATION MECHANISM

The PMF thesis stated: "Practices with H-Scores above 80 AND active content capture have 34% higher show rates" — an insight requiring culture data + content data + funnel data in the same analytical layer.

The architecture enables this through:

- **Kafka event bus (Section 1.6)**: All 10 cross-workstream integration points are defined as durable, replayable event streams with registered schemas. The event contracts (Section 2) explicitly wire the data flows that create cross-domain insights:
  - Contract #2: H-Score data flows from WS3 to WS1 Executive Dashboard
  - Contract #7: Funnel events flow from WS1 to WS2 for content booking attribution
  - Contract #8: Content engagement flows from WS2 to WS1 dashboards
  - Contract #9: Training completion flows from WS3 to WS1 readiness indicators

- **ClickHouse for analytics**: The Funnel Data Lake uses ClickHouse, which is purpose-built for the kind of cross-domain analytical queries that generate network insights. PostgreSQL stores operational data; ClickHouse handles "show me the correlation between H-Score, content volume, and show rates across 260 practices."

- **AI Service with shared prompt registry (Section 1.4)**: The centralized AI service means that insights discovered in one workstream's analysis (e.g., "practices with warm greetings have 15% higher rebooking") can be encoded as prompts and applied across workstreams. The prompt registry with versioning enables a flywheel: learn from data, encode learning as a prompt, apply to all 260 practices, measure results, iterate.

**Critical addition the PMF assessment did not anticipate**: The architecture introduces a compounding mechanism through Kafka event replay + ClickHouse time-series. Every month of operation adds to a historical dataset that cannot be replicated by a new entrant. The PMF assessment mentioned "each month compounds the data advantage" as an aspiration. The architecture makes it structural — events are retained 7-30 days in Kafka, then permanently in ClickHouse. The Network Learning Engine has an ever-growing corpus.

---

## 2. Shared Infrastructure as Bottleneck Risk — Blast Radius Analysis

8 shared services (plus Workflow Engine and Design Abstraction) mean all 3 workstreams depend on a common foundation. This is the central tension of the consolidated model: shared infrastructure eliminates duplication but introduces shared failure modes.

### Blast Radius Assessment by Service

| Service | If It Fails... | Workstreams Affected | Blast Radius | User Impact |
|---------|----------------|---------------------|--------------|-------------|
| **Practice Data Hub** | No practice lookups, no config reads | ALL (WS1, WS2, WS3) | CRITICAL | All operations halt. No practice context for any operation. |
| **PMS Integration** | No appointment data, no booking | WS1 (severe), WS2 (moderate), WS3 (low) | HIGH | WS1 cannot track funnel or book. WS2 loses capture triggers. WS3 H-Score still works (uses Cinnamon/Swell/Dental Intel directly, not PMS). |
| **Communications Service** | No SMS/voice/email | WS1 (severe), WS2 (moderate) | HIGH | Patient comms stop. Content capture prompts stop. WS3 unaffected. |
| **AI Service** | No LLM responses | ALL (WS1 severe, WS2 moderate, WS3 moderate) | HIGH | Agent reasoning stops. Caption generation stops. Trust Transfer NLP stops. Basic CRUD operations still work. |
| **Event Bus (Kafka)** | No cross-workstream events | ALL (delayed impact) | MEDIUM-HIGH | Each workstream continues operating on stale data. New events queue at producers. Impact is gradual (minutes to hours) not instant. |
| **Auth & Identity** | No login, no API calls | ALL | CRITICAL | All user-facing interfaces inaccessible. Service-to-service calls fail if JWT validation requires the auth service. |
| **Asset Registry** | No asset upload/download | WS2 (severe), WS1 (moderate) | MEDIUM | Content pipeline halts. Storefronts serve cached assets (CDN). WS3 unaffected. |
| **Compliance Middleware** | Audit logging fails | ALL (compliance risk, not functional) | LOW (functionally) / HIGH (legally) | Operations continue but without audit trail. Must be detected and resolved within hours. |
| **Workflow Engine (Temporal)** | Long-running workflows stall | ALL (moderate) | MEDIUM | Enrollment pipelines pause. Content production pipelines pause. Nightly PMS sync skips. Short-lived operations (API calls) unaffected. |
| **Design Abstraction** | No design generation | WS2 (moderate) | LOW | Content capture still works. Only template-based design generation stops. Manual fallback available. |

### Risk Verdict

Two services are existential: **Practice Data Hub** and **Auth & Identity**. If either fails, everything stops. This is an inherent consequence of consolidation — the PMF gain from shared infrastructure requires shared infrastructure to be reliable.

**Mitigations present in the architecture**:
- Practice Data Hub: Multi-AZ RDS, read replicas, Redis cache (5-min TTL). A complete RDS outage is AWS-SLA-level event. Redis cache provides degraded-mode reads during short outages.
- Auth: JWT validation is stateless at edge (Cloudflare Workers). Cognito is AWS-managed. The auth service itself only needs to be up for login and role changes, not for every request.
- Kafka: Producers buffer locally; consumers process at their own pace. Kafka's durability model means an MSK outage is recoverable (events are not lost, just delayed).
- AI Service: Priority queue ensures patient-facing operations degrade last. Batch workloads (captions, analysis) are the first to be delayed.

**What is NOT mitigated**:
- No circuit-breaker or graceful degradation pattern is described for the PMS Integration Service. If NexHealth is down, WS1's lead response automation and Voice AI booking fail without a described fallback. The risk register mentions "graceful degradation for Sikka-only practices" but not for NexHealth outages affecting NexHealth-connected practices.
- No explicit failover strategy for the AI Service. ADR-07 mentions "AI Service's ability to failover to a backup provider" but no backup provider is configured. If Anthropic has an outage, all three workstreams lose AI capability simultaneously.

**PMF impact of bottleneck risk**: The blast radius analysis shows that the most likely failure modes (PMS connector issues, AI rate limits, Twilio delivery delays) are workstream-specific, not platform-wide. The truly catastrophic failures (RDS down, Cognito down) are AWS-managed services with 99.99% SLAs. The consolidation does NOT introduce new single-points-of-failure that would not exist in the 6-sprint model — each sprint would have independently relied on AWS RDS and an auth system. The difference is that consolidated failure is more visible and more recoverable (one team owns it) than 6 independent failures discovered at different times by different teams.

**Scalability score impact: None. The blast radius is acceptable and arguably better than 6 independent systems.**

---

## 3. Time-to-Value Assessment

### Current Build Sequence Timeline

| Phase | Duration | User-Facing Value Delivered |
|-------|----------|-----------------------------|
| Phase 0 (Pre-Build) | Weeks 1-2 | None |
| Phase 1 (Shared Infrastructure) | Weeks 1-4 | None |
| Phase 2 (Contracts + WS3 Start) | Weeks 5-8 | WS3 components built but not user-facing yet |
| Phase 3 (WS1 + WS2 Start) | Weeks 9-12 | Practice Nerve Center v1, Executive Dashboard v1, Content Capture Bot v1 |

**First user-facing value**: Week 9-10 (Practice Nerve Center v1 + WS3 H-Score Engine)

That is approximately **2.5 months** before any SGA user sees a working screen. For a DSO executive team that has been through 6 sprint cycles of ideation, this is a long time without tangible output.

### Is This Acceptable?

**For the architecture: Yes.** The Phase 1 infrastructure is genuinely prerequisite. You cannot build the Practice Nerve Center without the Practice Data Hub, PMS Integration, and Auth services. The dependency graph is honest — the arrows are real, not artificial.

**For SGA's patience: Borderline.** The risk is not technical but organizational. SGA stakeholders have seen sprint outputs (HTML prototypes) and expect momentum. A 10-week silence before the first working feature could erode sponsorship.

### Minimum Viable Infrastructure to Accelerate

The architecture could deliver earlier value with two adjustments:

1. **WS3 can deliver standalone value in Week 6-7**, not Week 12. The H-Score Engine integrates with Cinnamon, Swell, and Dental Intel — external APIs that do not depend on shared infrastructure. Morning Huddle OS uses Dental Intel, not the PMS Integration Service. If WS3 builds a simple standalone dashboard (H-Score display + Morning Huddle content delivery) with just Practice Data Hub + Auth (available Week 2-3), SGA executives could see H-Scores for 260 practices by Week 7. This is low-risk, high-visibility, and demonstrates the "cross-domain insight" thesis early.

2. **Executive Dashboard "lite" by Week 8.** The Practice Data Hub contains 260 practice profiles by Week 2. A read-only dashboard showing practice roster, onboarding status, and region groupings requires only Practice Data Hub + Auth — no PMS, no ClickHouse, no Kafka. This is not the full Executive Dashboard, but it demonstrates the "single pane of glass" vision and gives SGA executives something to click on.

**Recommendation**: Add a Phase 1.5 milestone at Week 6-7 that delivers WS3 H-Score Dashboard + Executive Dashboard Lite. This does not change the architecture. It changes the delivery narrative from "nothing for 10 weeks" to "initial visibility in 6 weeks, full capability at 12."

### PMF Impact of Timeline

The build sequence does NOT degrade PMF. It is the correct sequence for the architecture. The improvement opportunity is tactical (deliver partial value earlier) not structural (change the architecture). No score change.

---

## 4. Competitive Moat Assessment

### Does the Architecture Create Defensibility?

The PMF assessment scored defensibility at 4/5 (consolidated) vs 3/5 (original), based on "integration density creating a compound moat." The technical architecture needs to deliver two specific moats:

#### Moat 1: Network Learning Engine (Cross-Practice Intelligence)

**Architecture delivers this**: The combination of Kafka event bus + ClickHouse time-series + centralized AI Service creates the structural prerequisite for network learning. Specifically:

- **260 practices generating funnel events** → ClickHouse ingests all events → AI Analysis Engine identifies patterns across the network ("practices that send appointment confirmations within 15 minutes have 12% higher show rates") → patterns become automated playbooks applied to all practices.
- **The data compounds monthly.** After 12 months, the platform has ~95M Kafka events, ~3.4M patient funnel transitions, ~340K content assets, and 260 daily H-Score time series. No new DSO platform entrant can replicate this dataset.

**But**: The architecture does not yet describe the actual Network Learning Engine as a service. The AI Analysis Engine in WS1 does "per-practice baseline calculation" and "basic anomaly detection." The cross-practice intelligence ("what works at the top 10 practices that the bottom 10 do not do") is implied by the data architecture but not specified as a service or workflow. This is a gap — not an architecture failure, but a missing feature definition that should be prioritized in Phase 3 or a Phase 4 roadmap.

#### Moat 2: Cross-Domain Data Correlation

**Architecture delivers this fully.** The 10 event contracts explicitly wire the three data domains:

- Culture data (H-Score, champion engagement, training completion) → Executive Dashboard
- Content data (asset production, engagement, booking attribution) → Practice Platform
- Operations data (funnel events, appointment completions, onboarding status) → Culture OS and Content Engine

The event contracts are not abstract — they have payload schemas with the exact fields needed for cross-domain correlation. Event Contract #7 (funnel events with `last_content_id` attribution) explicitly enables "which content assets drive bookings?" Event Contract #2 (H-Score with component breakdown) explicitly enables "what is the relationship between phone scores and patient outcomes?"

**Defensibility verdict**: The architecture delivers the structural moat. A competitor would need to replicate not just the individual capabilities (achievable with off-the-shelf tools) but also the 10 integration points with matching schemas, the shared data layer, and the historical dataset. This is a 12-18 month head start that compounds.

### PMF Impact on Defensibility

Defensibility score MAINTAINED at 4/5. The architecture delivers the integration density the PMF assessment assumed. The missing Network Learning Engine service definition is a Phase 4 item, not an architectural gap.

---

## 5. Cost-to-Operate at Scale

### Major Cost Drivers at 260 Practices

| Cost Category | Monthly Estimate | Per-Practice | Notes |
|---------------|-----------------|--------------|-------|
| **AWS Infrastructure** | | | |
| RDS PostgreSQL (Multi-AZ, read replicas) | $2,500 | $9.62 | db.r6g.xlarge primary + 1 read replica |
| MSK Kafka (Serverless) | $1,500 | $5.77 | ~260K events/day, 3 AZ replication |
| ECS/Fargate (8 services) | $3,200 | $12.31 | 8 services x avg 2 tasks x $200/task |
| ClickHouse (self-managed on EC2) | $1,200 | $4.62 | 2x r6g.xlarge for analytics |
| S3 + CloudFront | $400 | $1.54 | ~70K assets/year, CDN distribution |
| Redis (ElastiCache) | $500 | $1.92 | Session, cache, rate limiting |
| **SaaS Vendors** | | | |
| Claude API (Anthropic) | $4,000-8,000 | $15.38-30.77 | Biggest variable. Depends on Haiku/Sonnet/Opus mix. 70% Haiku keeps costs low. |
| Twilio (SMS + Voice) | $3,000-5,000 | $11.54-19.23 | 260 numbers ($260/mo) + ~13K msgs/day ($0.0079/msg) + Voice AI minutes |
| NexHealth | $2,000-4,000 | $7.69-15.38 | Per-practice pricing, volume discount expected |
| Sikka ONE | $500-1,000 | $1.92-3.85 | Batch-only, lower tier |
| OpenAI Whisper | $300-600 | $1.15-2.31 | Audio transcription for call analysis |
| Canva (Connect API) | $500-1,000 | $1.92-3.85 | Enterprise plan with API access |
| Cloudinary | $300-500 | $1.15-1.92 | Image transformations |
| Temporal Cloud | $400-800 | $1.54-3.08 | If using managed; less if self-hosted |
| Cognito | $200 | $0.77 | ~1,000 users, well within free tier boundary |
| Datadog / Monitoring | $800-1,200 | $3.08-4.62 | 8 services, custom metrics |
| **People** | | | |
| 20-person team (loaded) | ~$300,000/mo | $1,153.85 | This is the dominant cost |
| **Total Infrastructure + SaaS** | **$21,300-34,500** | **$81.92-132.69** | Excluding people |

### Is $47/practice/month Achievable?

**No, not with this architecture at 260 practices.** The infrastructure + SaaS floor is approximately $82-133/practice/month, nearly 2-3x the $47 target.

However, context matters:

1. **The $47/practice/month target was for the Comms Engine alone** (Sprint 5). The consolidated platform delivers vastly more value than just communications — it delivers practice operations, content pipeline, culture operationalization, executive intelligence, and cross-domain analytics. Comparing the full platform cost to a single-sprint pricing target is an apples-to-oranges comparison.

2. **The relevant question is total cost vs. total value created.** At $133/practice/month ($34,500 total), the platform costs ~$414K/year in infrastructure. If it drives even a 2% improvement in production per practice (a conservative estimate given funnel optimization, show rate improvement, and reactivation), and average practice production is $1.5M/year, that is $30K additional production per practice, or $7.8M across the network. The ROI is approximately 19:1.

3. **Costs improve with scale.** At 500 practices (SGA's acquisition trajectory), most infrastructure costs are fixed or sub-linear. RDS, Kafka, ClickHouse, ECS, and monitoring costs stay roughly the same. Only Twilio, Claude API, and NexHealth scale linearly. The per-practice cost drops to approximately $55-80/practice at 500 practices.

4. **The Claude API cost is the wild card.** At 70% Haiku routing (as specified in ADR-07), costs stay manageable. If real-world usage shifts toward Sonnet/Opus (e.g., Voice AI requires more complex reasoning than expected), Claude API could double. The AI Service's token accounting and cost-tier routing are the right controls, but budget monitoring must be tight.

### PMF Impact of Cost

Cost does not degrade the PMF score. The architecture is not cheap, but it is not unreasonable for the value it delivers. The $47/practice target needs to be retired and replaced with a full-platform unit economics model. The dominant cost is people ($300K/month), not infrastructure — which means the consolidation from 6 independent teams to 5 coordinated teams (20 people vs. potentially 30-40 for 6 independent products) is itself a cost optimization.

---

## VERDICT

### PMF Score: MAINTAINED at 22/25

The technical architecture faithfully delivers the structural advantages that earned the consolidation its 22/25 score. Point by point:

| Characteristic | Consolidated Score | Architecture Impact | Post-Architecture Score |
|---|---|---|---|
| Solves a real problem | 5 | No change. Architecture does not add or remove problems. | **5** |
| Timing is right | 4 | Slight risk from 10-week infrastructure phase, but mitigable with Phase 1.5 deliverables. | **4** |
| Approach is defensible | 4 | Fully delivered. 10 event contracts with schemas, Kafka + ClickHouse enable the integration density moat. | **4** |
| Can scale | 4 | Strengthened. Architecture-enforced consolidation (workstreams physically cannot bypass shared services) is stronger than the policy-level consolidation the PMF assessment assumed. | **4** (could argue 4.5) |
| Creates unfair advantage | 5 | Delivered. Cross-domain data correlation is structurally enabled. Network Learning Engine needs explicit service definition in a future phase but the data plumbing is in place. | **5** |
| **Total** | **22/25** | | **22/25** |

### Items That Could IMPROVE the Score (Potential 23/25)

1. **Scalability could reach 4.5-5** if the architecture adds explicit circuit breakers and graceful degradation patterns for PMS Integration and AI Service failures. The current architecture assumes these services are available; describing fallback behavior would strengthen the scalability claim.

2. **Network Learning Engine as an explicit service** (not just implied by the data architecture) would solidify the unfair advantage score at 5 and potentially push defensibility to 5.

### Items to Watch (Could DEGRADE if Mismanaged)

1. **AI Service vendor concentration**: All three workstreams on Claude with no configured backup provider. If Anthropic has a multi-hour outage during business hours, all AI-powered features fail simultaneously. This is a operational risk, not an architectural one — the AI Service abstraction supports failover; it just needs a second provider configured.

2. **Phase 1 delivery execution**: If shared infrastructure takes 8 weeks instead of 4, the entire build sequence shifts. The architecture is correct but optimistic about Phase 1 timeline given the breadth (8 services in 4 weeks with a new team). Recommend a 6-week Phase 1 contingency plan.

3. **Claude API costs at scale**: The 70/25/5 Haiku/Sonnet/Opus routing assumption needs validation against real workloads. Voice AI and agent reasoning may demand more Opus than projected.

---

### QC-3 Gate: PASS

The technical architecture preserves the consolidation's PMF thesis. No score degradation. The architecture is sound, the event contracts are well-defined, the shared infrastructure eliminates the duplication that dragged the original 6-sprint model to 16/25, and the data architecture enables the cross-domain intelligence that justified the 5/5 unfair advantage score.

Proceed to implementation planning with the two recommendations:
1. Add Phase 1.5 milestone (H-Score Dashboard + Executive Dashboard Lite at Week 6-7)
2. Define the Network Learning Engine as an explicit service in Phase 4 roadmap
