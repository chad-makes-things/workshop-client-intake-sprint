# Evaluation Matrix: Original Sprints vs. Consolidated Workstreams

**Date**: 2026-03-14
**Author**: Decision Architect Agent (Phase 7)
**Purpose**: Structured cross-workstream validation comparing the original 6-sprint portfolio against the consolidated 3-workstream model across 5 evaluation dimensions.

---

## Executive Summary

The consolidation from 6 independent sprints to 3 workstreams is a significant net improvement. The consolidated model preserves 100% of the original capability inventory (141 stories mapped to 110 deduplicated stories plus 38 shared infrastructure items), eliminates 4 redundant PMS integration designs, 5 competing dashboards, and 3 separate agent architectures, and delivers a coherent platform that users experience as a single system rather than 6 disconnected tools. The consolidation does introduce new coordination risk through increased cross-workstream coupling and a longer critical path before any workstream can deliver independently, but these risks are well-mitigated by the phased build map and early value delivery at Milestone 1.5.

---

## Scoring Methodology

Each dimension is scored on a 1-10 scale for both the original model (6 independent sprints) and the consolidated model (3 workstreams + shared infrastructure). Scoring criteria:

| Score | Meaning |
|---|---|
| 1-2 | Fundamentally broken or missing |
| 3-4 | Significant gaps or contradictions |
| 5-6 | Adequate but with notable weaknesses |
| 7-8 | Strong with minor gaps |
| 9-10 | Excellent with comprehensive coverage |

Scores are evidence-based. Each score references specific artifacts from the sprint portfolio and consolidation deliverables. Where a consolidation score is lower than the original, the rationale explains why and identifies mitigations.

---

## Dimension 1: Capability Coverage

**Question**: Does the consolidated model cover everything the original 6 sprints covered? Are there gaps? Are there additions?

### Original Model (6 Sprints): 8/10

The original 6 sprints covered a wide capability surface:

- **Studio Automation**: Conversational email intake, Canva API generation, CD mobile approval, print fulfillment (Gelato), medium challenge, brand health scoring
- **Hospitality Playbook**: Morning Huddle OS, CCA Academy (Bronze/Silver/Gold), H-Score engine, Trust Transfer OS, Champion Network, recognition/rewards
- **Practice Concierge**: Silent data harvesting, practice enrollment pipeline, 5 specialty templates, storefront generation, blitz deploy, change management
- **Funnel OS**: 15-platform data integration, 5-stage funnel model, AI anomaly detection, Practice Nerve Center, executive dashboard, automation engine
- **Comms Engine**: 4-stage automation (lead response, treatment follow-up, rebooking, reactivation), graduated autonomy, network learning engine, playbook engine, Voice AI
- **Content Concierge**: SMS capture coach, AI photo/video processing, 5 practice-type playbooks, champion scorecard, testimonial pipeline, 6-platform distribution

**Strengths**: Each sprint went deep on its domain. Feature-level detail was thorough with specific user stories, acceptance criteria, and technical specifications.

**Weaknesses**: Capabilities were siloed. No sprint addressed how its features interoperate with other sprints' features. The sprints explicitly documented 9 cross-sprint dependencies but left resolution to "future integration work." Voice AI appeared in Comms Engine with incomplete PMS write-path validation.

### Consolidated Model (3 Workstreams): 9/10

The feature map (`consolidation-feature-map.md`) traces every capability from all 6 sprints into the consolidated model:

- **141 original stories** mapped to **110 consolidated stories** (31 eliminated through deduplication) plus **38 shared infrastructure stories** and **12 cross-cutting items** = 160 total build items
- **Zero capability gaps**: Every feature from every original sprint has a home in the consolidated model. The feature map provides explicit traceability (e.g., "WS1-A2.3: Missed call text-back (< 90 seconds) — Source: CE:LR-2")
- **Additions not present in originals**:
  - Unified Practice Data Hub (replaces 3 independent practice data stores)
  - Cross-workstream event bus with 10 defined event contracts
  - Shared compliance middleware with centralized PHI filtering
  - Design tool abstraction layer (Canva today, pluggable for future alternatives)
  - Unified consent registry (prevents duplicate opt-out handling across WS1/WS2)

**Strengths**: Complete traceability. Every original capability is accounted for. Shared infrastructure adds capabilities that no individual sprint could justify building alone (e.g., centralized PHI filter, unified consent management).

**Weaknesses**: Voice AI is explicitly deferred to v2 — this was already shaky in the original Comms Engine sprint (NexHealth write-path unproven), so the deferral is pragmatic rather than a loss. Three Tier 3 items (CCA renewal, patient self-record kiosks, passive operatory cameras) are pushed to Weeks 31-36, which delays their availability compared to a standalone sprint timeline where they might have been built sooner.

**Delta: +1** — The consolidated model covers everything the originals did, adds cross-cutting capabilities, and makes honest decisions about deferring unproven features.

---

## Dimension 2: Technical Coherence

**Question**: Is the consolidated architecture more coherent than 6 independent architectures? Does it reduce duplication, unify the data model, and establish shared services?

### Original Model (6 Sprints): 3/10

The redundancy report documented severe technical incoherence across the original sprints:

- **4 incompatible PMS integration strategies**: Funnel OS (Airbyte + custom Python), Comms Engine (NexHealth Synchronizer), Content Concierge (Sikka ONE API), Practice Concierge (custom Playwright scraping). All four need appointment and patient data from the same systems. Building four integration layers is pure engineering waste.
- **3 cloud providers**: Azure (Studio Automation), AWS (Funnel OS, Content Concierge, Comms Engine), Cloudflare/Vercel (Practice Concierge). Three security surfaces, three billing relationships, three ops skill sets.
- **3 queue systems**: BullMQ (Studio Automation, Practice Concierge, Hospitality Playbook), Kafka (Funnel OS), unspecified (Comms Engine, Content Concierge). No event interoperability.
- **3 competing per-practice agent designs**: Practice Concierge (Claude for onboarding), Comms Engine (Claude/GPT for communications), Funnel OS (Python ML for anomaly detection). The Comms Engine sprint itself states "Comms Engine agent IS the Practice Concierge agent extended" — yet both sprints design the agent independently.
- **5 executive dashboards, 5 practice-level interfaces**: An office manager would log into multiple systems daily, which directly contradicts Funnel OS's stated goal of eliminating tool fragmentation.
- **No shared data model**: Each sprint designs its own practice record, patient record, and staff roster independently.

**Strengths**: Each sprint's internal architecture was well-designed for its own scope. Funnel OS's ClickHouse + Kafka pipeline for real-time analytics was particularly strong. The Comms Engine's graduated autonomy model was sophisticated.

**Weaknesses**: The portfolio as a whole has no technical coherence. Building all 6 as designed would produce 6 systems that must later be painfully integrated — undoing months of independent build work.

### Consolidated Model (3 Workstreams): 8/10

The technical architecture (`technical-architecture.md`) establishes a coherent platform:

- **10 shared services** owned by Platform Engineering, consumed by all workstreams via well-defined APIs:
  1. Practice Data Hub (single source of truth for all 260 practices)
  2. PMS Integration Service (NexHealth real-time R/W + Sikka ONE batch read-only)
  3. Communications Service (centralized Twilio, consent, routing)
  4. AI Service (centralized Claude API with PHI filter, prompt registry, cost-tier routing)
  5. Asset Registry (S3 + CloudFront + Cloudinary for all brand/content assets)
  6. Event Bus (Kafka MSK with Schema Registry, 10 defined event contracts)
  7. Auth & Identity (Cognito + RBAC with 8 roles)
  8. Compliance Middleware (Fastify plugin, audit log, encryption helpers)
  9. Workflow Engine (Temporal for long-running processes)
  10. Design Tool Abstraction (Canva adapter, pluggable)

- **Unified data model**: PostgreSQL schemas defined with foreign key relationships across services. Practice Data Hub is the canonical record consumed by all workstreams.
- **Single cloud provider**: AWS (primary) for all workstreams. Eliminates multi-cloud ops overhead.
- **Standardized stack**: Node.js/TypeScript/Fastify for services. React for frontends. PostgreSQL + ClickHouse for storage. Kafka for events. Temporal for workflows.
- **10 cross-workstream event contracts** with Avro/JSON Schema enforcement, consumer contract testing (Pact), and dead-letter queue handling.

**Strengths**: The architecture directly addresses every "CRITICAL" and "HIGH" finding from the redundancy report. PMS integration is build-once. Dashboards are unified. The agent architecture is a single instance per practice with capability modules. Event contracts make cross-workstream dependencies explicit and testable.

**Weaknesses**: The architecture is complex. 10 shared services is a lot of infrastructure to build and maintain before any workstream feature ships. The 18-week critical path to integrated launch is long. Kafka adds operational complexity that BullMQ alone would not require — but Kafka is necessary for durable, replayable cross-workstream events.

**Delta: +5** — This is the largest improvement in the entire evaluation. Moving from 4 PMS integrations to 1, from 5 dashboards to 1, and from 3 cloud providers to 1 is a dramatic coherence gain.

---

## Dimension 3: User Experience Quality

**Question**: Do users get a better experience from unified interfaces vs. 6 separate products?

### Original Model (6 Sprints): 5/10

Each sprint designed its own user interfaces without awareness of the others:

- **Office Managers** would interact with: Practice Concierge (storefront management), Funnel OS Nerve Center (daily agenda), Comms Engine (weekly summary review), potentially Content Concierge (champion scorecard), and Hospitality Playbook (huddle and H-Score). That is 5 separate tools for one role.
- **Executives** would check: Funnel OS Executive Dashboard, Practice Concierge Network Health Dashboard, Hospitality Playbook Network Intelligence Dashboard, and Studio Automation Analytics Dashboard. 4 dashboards for one role.
- **Content Champions** would use: Content Concierge (SMS capture) and Studio Automation (request submission). 2 separate content systems.
- **No single sign-on** — each sprint designs its own auth approach.
- **No consistent design language** — 6 independent UI designs, 6 navigation patterns, 6 notification systems.

**Strengths**: Individual sprint UIs were well-designed for their isolated scope. The Funnel OS Nerve Center and Comms Engine weekly summary were particularly thoughtful about reducing office manager cognitive load — within their own boundaries.

**Weaknesses**: The aggregate experience across 6 products is fragmented. This is ironic: the stated goal of several sprints was to reduce tool fragmentation for practices, yet the portfolio itself would create a new layer of fragmentation.

### Consolidated Model (3 Workstreams): 8/10

The experience architecture (`experience-architecture.md`) addresses fragmentation directly:

- **Design Principle 1**: "One user, one surface. No actor type should need to navigate between product surfaces to complete their daily workflow."
- **Design Principle 5**: "Workstream boundaries are invisible to users. Users see 'my work.' They do not see 'Workstream 1' or 'Workstream 2.'"
- **Office Managers** get one surface: the Practice Daily Agenda (Nerve Center), which integrates action cards from WS1 (patient outreach, funnel alerts), WS2 (content capture prompts via widget), and WS3 (H-Score, huddle). Single login, single daily workflow.
- **Executives** get one surface: the Network Intelligence Dashboard, which shows funnel metrics (from WS1), content health (from WS2), and H-Scores (from WS3) in a unified view with region/practice drill-down.
- **Content Champions** get one workflow: SMS prompt arrives, they reply with photo/video, Content Engine handles processing/approval/distribution. Studio requests come through the same content pipeline.
- **Shared design system** (per Prototype Lead #3 carry-forward): consistent component library, typography, spacing, and interaction patterns across all surfaces.
- **42 interfaces** phased into 3 tiers (16 at launch, 14 in expansion, 12 at full platform) — a realistic scope management approach.

**Strengths**: The unified interface approach is a genuine improvement for every actor type. Cross-workstream data flows (H-Score appearing on the executive dashboard, content assets flowing to storefronts) are designed as native features rather than bolted-on integrations.

**Weaknesses**: The scope of WS1 is large — combining Practice Concierge + Funnel OS + Comms Engine into one workstream creates a dense product surface. The experience architecture addresses this by splitting WS1 into two sub-surfaces (Practice Operations and Network Intelligence), but the Practice Operations surface still carries a lot: daily agenda, patient outreach, concierge chat, storefront management, and automation configuration. Progressive disclosure principles help, but there is a risk of cognitive overload as Tier 2 and Tier 3 features are layered in.

**Delta: +3** — Unified surfaces and invisible workstream boundaries are a substantial UX improvement. The risk of WS1 scope overload is acknowledged and partially mitigated but not fully eliminated.

---

## Dimension 4: Build Efficiency

**Question**: Is the consolidated build more efficient? Does it reduce duplication, share infrastructure, and optimize team utilization?

### Original Model (6 Sprints): 4/10

Building 6 sprints independently would incur massive duplication:

- **4 PMS integration builds**: ~3-4 months of engineering duplicated across Funnel OS, Comms Engine, Content Concierge, and Practice Concierge
- **5 dashboard builds**: ~2-3 months of frontend engineering for 5 executive dashboards and 5 practice interfaces serving overlapping user groups
- **3 agent architecture builds**: ~2 months designing 3 competing per-practice agent designs that must later merge (as the Comms Engine sprint itself acknowledges)
- **3 approval workflow engines**: Studio Automation, Content Concierge, and Practice Concierge each design their own approval queue — ~1 month duplicated
- **2 brand asset management systems**: Studio Automation DAM + Content Concierge pipeline — duplicated storage, duplicated metadata models
- **Independent hiring for 3 cloud platforms**: Azure + AWS + GCP ops expertise required
- **No shared infrastructure**: Each sprint builds its own auth, compliance, and data access patterns from scratch

The consolidated epic inventory estimates **141 original stories** across 6 sprints. Without deduplication, each duplicated story would be built, tested, and maintained independently.

**Strengths**: Independent sprints can move faster initially because they have no coordination overhead. Each team owns its full stack and can ship without waiting for shared infrastructure.

**Weaknesses**: The efficiency advantage of independence is temporary. The 9 documented cross-sprint dependencies guarantee that integration work will eventually be required — and integration after independent builds is 3-5x more expensive than building integrated from the start.

### Consolidated Model (3 Workstreams): 8/10

The consolidation achieves significant build efficiency:

- **31 stories eliminated through deduplication** (build once, serve many): D-01 through D-12 in the consolidated epic document detail each deduplicated capability
- **Shared infrastructure built once**: 10 shared services (38 stories) serve all 3 workstreams. Without consolidation, each workstream would build partial versions of these capabilities independently.
- **Team structure optimized**: 20 engineers organized into 6 teams (Platform Engineering 5, WS1-A 4, WS1-B 3, WS2 4, WS3 3, QA 2) with clear ownership boundaries and phased allocation
- **Build sequence prevents rework**: Infrastructure first (Weeks 1-6), then WS3 early start (Weeks 5-10, validating infrastructure), then WS1+WS2 in parallel (Weeks 11-18)
- **Velocity assumptions are realistic**: ~10% capacity buffer across all teams at Tier 1 delivery. Velocity modifiers account for new team ramp-up (-30%), cross-team dependencies (-15%), and production incidents (-10%).

**Quantified savings** (from consolidation recommendation):
| Area | Before | After | Savings |
|---|---|---|---|
| PMS integration builds | 4 | 1 | ~3-4 months engineering |
| Dashboard builds | 10 (5 exec + 5 practice) | 2 (1 exec + 1 practice with modular tabs) | ~2-3 months frontend |
| Agent architectures | 3 | 1 (with capability modules) | ~2 months architecture + rework avoidance |
| Cloud providers | 3 | 1 | Ongoing ops cost reduction |
| Queue systems | 3 | 2 (BullMQ + Temporal, Kafka for events) | Simplified infrastructure |
| Approval workflows | 3 | 1 shared engine | ~1 month engineering |

**Strengths**: The build map shows a well-thought-out parallel/sequential plan with a clear critical path (18 weeks). WS3 starting first is smart — it has the fewest dependencies and validates infrastructure under real load.

**Weaknesses**: The shared infrastructure phase (Weeks 1-6) delays all workstream feature development. In the original model, any single sprint could begin shipping user-facing features immediately. The consolidation trades early velocity for long-term efficiency — this is the right tradeoff, but it means 6 weeks of "invisible progress" before any user sees anything. The M1.5 milestone (WS3 pilot at Week 6-7) mitigates this by delivering early value, but WS1 and WS2 users wait until Week 11-14 for their first features.

**Delta: +4** — The efficiency gains from deduplication and shared infrastructure are substantial. The 31 eliminated stories alone represent months of avoided engineering effort.

---

## Dimension 5: Risk Profile

**Question**: Is the consolidated model lower risk? Are there single points of failure, blast radius concerns, or rollback complexity?

### Original Model (6 Sprints): 6/10

Independent sprints have a naturally favorable risk profile in some dimensions:

- **Blast radius is contained**: If Studio Automation fails, it does not affect Funnel OS. Each sprint is isolated.
- **Independent rollback**: Each sprint can be rolled back without affecting others.
- **Smaller teams, less coordination**: Each sprint team can make decisions autonomously.
- **Incremental delivery**: Each sprint can ship value independently on its own timeline.

But the original model carries significant hidden risks:

- **Integration risk is deferred, not eliminated**: The 9 documented cross-sprint dependencies will eventually force integration. Deferred integration is higher risk than planned integration — the systems will have hardened around incompatible assumptions.
- **Data consistency risk**: 4 PMS integrations pulling the same data through different connectors will produce inconsistent patient/appointment records across systems.
- **Compliance risk**: Each sprint independently manages HIPAA compliance, PHI handling, and audit logging. Without coordination, one sprint's compliance gap exposes the entire portfolio.
- **Vendor risk**: 3 cloud providers, 3 PMS connectors, 2 AI providers — each requires separate BAAs, security reviews, and incident response plans.
- **Talent risk**: Hiring for Azure + AWS + GCP + Python + Node.js + multiple queue systems requires broader (and rarer) talent than a standardized stack.

### Consolidated Model (3 Workstreams): 7/10

The consolidated model trades some types of risk for others:

**Risks reduced**:
- **Integration risk**: Eliminated. Cross-workstream dependencies are designed in from Day 1 with explicit event contracts, schema validation, and consumer contract testing (Pact).
- **Data consistency**: Eliminated. One Practice Data Hub, one PMS Integration Service, one canonical event schema.
- **Compliance risk**: Significantly reduced. Centralized Compliance Middleware (Fastify plugin) with audit logging, PHI filter, and encryption helpers applied uniformly across all services.
- **Vendor risk**: Reduced from 3 cloud providers to 1, 4 PMS connectors to 2 (NexHealth + Sikka ONE), 2 AI providers to 1 primary (Claude) + 1 specialized (Whisper for transcription).
- **Talent risk**: Reduced. Standardized Node.js/TypeScript/Fastify stack. One cloud provider (AWS). One queue system for events (Kafka). Smaller hiring surface.

**Risks introduced or increased**:
- **Shared infrastructure as single point of failure**: If the Practice Data Hub goes down, all three workstreams are affected. If Kafka is unavailable, cross-workstream events stop flowing. Mitigation: Multi-AZ deployment, read replicas, circuit breakers, graceful degradation (workstreams should function in read-only mode without Kafka).
- **Blast radius**: A broken deployment to a shared service (e.g., Compliance Middleware) could impact all workstreams simultaneously. Mitigation: Blue-green deployments, canary releases, comprehensive integration testing at M2.
- **Coordination overhead**: 6 teams (Platform Engineering, WS1-A, WS1-B, WS2, WS3, QA) must coordinate on shared API contracts, event schemas, and deployment windows. Mitigation: Schema Registry enforcement, Pact contract testing in CI, weekly integration test suite.
- **Critical path length**: 18-week critical path means a delay in shared infrastructure delays everything. The original model had no single critical path — each sprint could proceed independently. Mitigation: Buffer built into velocity assumptions; synthetic data allows workstreams to begin development before PMS BAAs close.
- **Rollback complexity**: Rolling back a shared service version requires coordinating with all consuming workstreams. Mitigation: API versioning, backward-compatible schema evolution, feature flags.

**New risk not present in originals**:
- **Kafka operational complexity**: Kafka (via MSK) is more operationally demanding than BullMQ. Consumer lag, partition rebalancing, schema evolution, and dead-letter queue management require specialized expertise. The build map budgets 1 week for team training and uses MSK Serverless to reduce ops burden. This is a manageable but real risk.

**Delta: +1** — The consolidated model is marginally lower risk overall. It eliminates the severe hidden risks of the original model (deferred integration, data inconsistency, compliance gaps) while introducing new risks that are addressable through standard platform engineering practices. The risk profile is different in character rather than dramatically better — concentrated and manageable vs. distributed and unpredictable.

---

## Summary Scorecard

| Dimension | Original (6 Sprints) | Consolidated (3 Workstreams) | Delta | Rationale |
|---|---|---|---|---|
| **Capability Coverage** | 8 | 9 | **+1** | 100% coverage preserved, 31 stories deduplicated, cross-cutting capabilities added, Voice AI honestly deferred |
| **Technical Coherence** | 3 | 8 | **+5** | 4 PMS integrations -> 1, 5 dashboards -> 1, 3 cloud providers -> 1, unified data model, 10 shared services |
| **User Experience Quality** | 5 | 8 | **+3** | One surface per role, invisible workstream boundaries, shared design system; WS1 scope density is a concern |
| **Build Efficiency** | 4 | 8 | **+4** | 31 stories eliminated, shared infrastructure built once, phased team allocation; 6-week infrastructure investment delays early feature delivery |
| **Risk Profile** | 6 | 7 | **+1** | Hidden integration/consistency/compliance risks eliminated; new shared-infrastructure SPOF and coordination risks introduced but mitigated |
| **Total** | **26/50** | **40/50** | **+14** | |

---

## Overall Assessment

**The consolidation is justified. It produces a net +14 point improvement across all five dimensions, with the strongest gains in technical coherence (+5) and build efficiency (+4).**

The original 6-sprint portfolio was well-conceived at the individual sprint level but fundamentally incoherent at the portfolio level. Four incompatible PMS integrations, five competing dashboards, three independent agent architectures, and three cloud providers would have produced an expensive, fragmented product suite that contradicts its own stated goals. The sprints themselves documented 9 cross-sprint dependencies and explicitly acknowledged convergence (Comms Engine: "IS the Practice Concierge agent extended"), making the case that consolidation is not an external imposition but an organic conclusion.

The consolidated model is not without tradeoffs:

1. **Infrastructure-first delays visible output** — but M1.5 (WS3 pilot at Week 6-7) delivers early value to 20 practices.
2. **Shared services create single points of failure** — but Multi-AZ deployments, read replicas, and circuit breakers are standard mitigations.
3. **Coordination overhead increases** — but event contracts, schema enforcement, and contract testing make dependencies explicit and testable rather than implicit and fragile.
4. **WS1 scope is dense** — but splitting into Practice Operations (A) and Network Intelligence (B) sub-teams with separate deployment cycles manages the complexity.

The consolidation does not degrade any dimension. Every score improved or held steady. The only dimension where the gain is modest (+1) is risk profile, where the nature of risk shifts from distributed-and-hidden to concentrated-and-visible. Concentrated, visible risk is preferable — it can be monitored, tested, and mitigated.

---

## Concerns & Mitigations

### Concern 1: Shared Infrastructure as Bottleneck (Severity: HIGH)

**Risk**: All three workstreams depend on Platform Engineering completing 10 shared services by Week 6. If infrastructure delivery slips, everything slips.

**Evidence**: The build map shows a 10% capacity buffer for Platform Engineering in Weeks 3-6 (200 pts capacity vs. ~180 pts demand). This is thin.

**Mitigation**:
- M1.5 milestone at Week 6-7 provides an early integration test via WS3 pilot
- Synthetic data and mock services allow workstream teams to begin development before infrastructure is fully live
- The build map explicitly deprioritizes less critical infrastructure (Design Tool Abstraction is last, at Week 6) to protect the critical path
- If infrastructure slips 1-2 weeks, WS3 start can absorb the delay (WS3 has a 10-week window, only needs 8 weeks of build)

### Concern 2: WS1 Scope Overload (Severity: MEDIUM-HIGH)

**Risk**: Workstream 1 combines Practice Concierge (19 stories + 4 spikes), Funnel OS (34 stories), and Comms Engine (23 stories) — the three largest sprints — into one workstream. Even after deduplication (58 consolidated stories), WS1 carries the most feature weight and serves the most actor types.

**Evidence**: WS1 has 61 stories (37 Practice Ops + 24 Network Intelligence) vs. WS2 at 40 and WS3 at 21. WS1 is split into two sub-teams (WS1-A at 4 engineers, WS1-B at 3), but coordination overhead between the sub-teams is not budgeted explicitly.

**Mitigation**:
- WS1-A and WS1-B have independent deployment cycles and serve different actor types (practice staff vs. executives)
- The experience architecture splits WS1 into two distinct surfaces with separate navigation and information architecture
- RICE prioritization ensures the most impactful stories ship first; Tier 3 stories (5 in WS1) are deferred to Weeks 31-36
- If WS1-A falls behind, WS1-B can continue independently (their only coupling is through Kafka events)

### Concern 3: Kafka Operational Complexity (Severity: MEDIUM)

**Risk**: Kafka (via AWS MSK) adds operational complexity that may exceed the team's current expertise. Consumer lag management, partition rebalancing, schema evolution, and dead-letter queue handling require specialized knowledge.

**Evidence**: The original sprints used BullMQ (simpler, Node.js-native) in 3 of 6 cases. Kafka is a step up in complexity.

**Mitigation**:
- AWS MSK Serverless reduces infrastructure management burden
- 1 week of team training budgeted in the build map
- Schema Registry enforces contract compliance at the infrastructure level, reducing the surface for human error
- Contract testing (Pact) in CI catches breaking changes before deployment
- Dead-letter queue monitoring with alerts on DLQ depth > 0

### Concern 4: Cross-Workstream Event Coupling (Severity: MEDIUM)

**Risk**: 10 defined event contracts create coupling between workstreams. A breaking change in one producer's event schema could cascade to multiple consumers.

**Evidence**: The event verification schedule shows events flowing as early as M1.5 (Week 7) with all 10 live by M4 (Week 18). That is 10 integration surfaces to monitor and maintain.

**Mitigation**:
- Schema Registry with backward-compatible evolution policy (fields can be added but not removed or renamed)
- Pact consumer contract tests run in CI — a producer cannot deploy if it breaks a consumer contract
- Event versioning strategy (v1 -> v2 with deprecation window)
- Each workstream should gracefully degrade if an expected event is unavailable (eventual consistency, not hard dependency)

### Concern 5: Delayed Time-to-First-Value for WS1/WS2 Users (Severity: LOW-MEDIUM)

**Risk**: WS1 and WS2 users do not see any features until Week 11-14. In the original model, any sprint could have shipped an MVP sooner.

**Evidence**: The build map shows WS1-A, WS1-B, and WS2 teams idle from Weeks 1-10 (during infrastructure build + contract hardening). This is by design but delays value delivery.

**Mitigation**:
- M1.5 delivers WS3 value (H-Score + Huddles) to 20 practices by Week 6-7, providing leadership with a tangible proof point
- WS1/WS2 teams can work on design, user research, and content (playbook definitions, brand kits, template design) during Weeks 1-10 even if they cannot build features
- The infrastructure-first approach means WS1/WS2 features ship faster once they start, because shared services are proven and stable
- An individual sprint's "early MVP" would lack PMS data, shared auth, and compliance middleware — it would be a demo, not a production system

---

*This evaluation matrix provides the Phase 7 cross-workstream validation for the SGA Sprint Portfolio Review. All scores are evidence-based, referencing specific findings from the redundancy report, consolidation recommendation, feature map, experience architecture, technical architecture, consolidated epic, milestone definitions, and build map. The overall assessment is that the consolidation is a clear improvement, with the strongest gains in technical coherence and build efficiency, and manageable tradeoffs in risk profile and time-to-first-value.*
