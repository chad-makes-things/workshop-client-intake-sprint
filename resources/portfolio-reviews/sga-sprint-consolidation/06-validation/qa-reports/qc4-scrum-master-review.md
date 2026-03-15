# QC-4: Scrum Master Review --- Capacity & Feasibility Assessment

**Date**: 2026-03-14
**Reviewer**: Scrum Master Agent
**Artifacts Reviewed**: consolidated-epic.md, milestone-definitions.md, build-map.md, technical-architecture.md, component-map.md

---

## Verdict: Feasible with Caveats

---

## Executive Summary

The roadmap is well-structured with sound sequencing logic: infrastructure before features, WS3 as an early-start validator, and parallel workstream tracks once contracts are hardened. The 18-week critical path to M4 is achievable with the proposed 20-person team, but the plan operates on razor-thin capacity margins (~10% buffer by the PM's own admission). Three specific risks require mitigation before build start: (1) the 4-week infrastructure sprint is aggressive for a newly-formed team that will suffer a -30% velocity penalty, (2) the M3 milestone packs too much feature scope into 4 weeks across 4 parallel teams with no integration slack, and (3) the design system delivery plan overlaps with shared infrastructure work, creating a resource conflict on Platform Engineering in Weeks 1-4.

---

## Milestone Feasibility Assessment

### M0: Pre-Build Complete (Weeks 1-2) -- FEASIBLE

Straightforward procurement and environment setup. Two concerns:

- **BAA execution in 2 weeks is optimistic.** Healthcare vendor BAAs (NexHealth, Anthropic) routinely take 4-6 weeks. The risk-adjusted timeline acknowledges this but the mitigation ("use synthetic data") only partially addresses the problem -- Communications Service and PMS Integration cannot be fully tested with synthetic data.
- **Staffing 15 of 20 engineers in 2 weeks** assumes an existing hiring pipeline or internal reallocation. If this is a net-new team, M0 will slip.

**Assessment**: Feasible if team allocation is from an existing pool, not net-new hiring.

### M1: Shared Infrastructure Live (Weeks 3-6) -- FEASIBLE WITH RISK

10 shared services in 4 weeks with a 5-person Platform Engineering team. The build map shows these services can be built in parallel (Group 1), which is correct -- they share PostgreSQL/AWS but not each other.

**Concerns**:
- The -30% new-team velocity modifier (acknowledged in build-map.md Section 8) reduces effective Platform Engineering velocity from 25 pts/week to ~17.5 pts/week in Weeks 3-6. That gives 70 effective points over 4 weeks, not 100. The demand is 180 points over 8 weeks, but the first 4 weeks carry disproportionate load since the second batch of services (PMS, Comms, AI, Asset Registry, Workflow Engine, Design Tool) depends on Practice Data Hub being live by Week 4.
- **Design System delivery plan (component-map.md Section 8) allocates Weeks 1-4 for tokens, primitives, patterns, and layouts.** This is a major deliverable that the build-map.md does not account for in Platform Engineering's allocation. The technical architecture adds `@sgadental/design-system` to Phase 1 exit criteria but does not identify who builds it. If Platform Engineering owns it, their capacity is further strained. If a dedicated UX engineer builds it, that person is not in the 20-person team roster.

**Assessment**: The service count is manageable if services are kept to MVP scope. The design system delivery is an unresourced dependency -- see Impediment #1.

### M1.5: Partial Value Delivery (Weeks 6-7) -- FEASIBLE

Smart milestone. WS3's dependencies (Practice Data Hub, Auth, Kafka) are all available by Week 4-5. The 20-practice pilot is a controlled scope. Soft gate means it does not block M2.

**One concern**: WS3 team (3 people) starts in Week 5 per the build map, meaning they have only 2-3 weeks to deliver H-Score Engine + Morning Huddle OS + pilot onboarding by Week 7. This is tight but achievable because these features have well-defined external API integrations (Cinnamon, Swell, Dental Intel) rather than novel system design.

**Assessment**: Feasible. The soft gate is the right call.

### M2: Event Contracts Hardened + WS3 MVP (Weeks 7-10) -- FEASIBLE

Platform Engineering shifts to contract hardening while WS3 continues building. No resource conflict -- these are different teams.

**Concerns**:
- CCA Academy Bronze (M2.6) requires 10 video modules. Content production is not engineering work. Who creates the training content? This is not addressed in the build plan. If it falls to the WS3 team, it competes with engineering time.
- Credly API integration is a third-party dependency. Lead time for Credly organization setup should be initiated in M0.

**Assessment**: Feasible, contingent on CCA content being pre-produced by SGA's L&D team.

### M3: WS1 + WS2 Tier 1 Features (Weeks 11-14) -- HIGHEST RISK

This is the most overloaded milestone. Four teams building in parallel (WS1-A, WS1-B, WS2, WS3 continuing) with 19 deliverables in 4 weeks.

**Story count analysis**:
- WS1-A (4 people, 20 pts/week): 7 deliverables (M3.1-M3.7). 80 capacity points. The Nerve Center, Concierge Agent, Data Harvesting, Enrollment, Storefront Generation, Automation Engine v1, and Graduated Autonomy span 18 Tier 1 stories. This is the tightest fit in the plan.
- WS1-B (3 people, 15 pts/week): 4 deliverables (M3.8-M3.11). 60 capacity points for Funnel Data Lake, Executive Dashboard, AI Analysis Engine, and 2 data connectors. 10 Tier 1 stories. Achievable but no slack.
- WS2 (4 people, 20 pts/week): 5 deliverables (M3.12-M3.16). 80 capacity points for Content Capture Bot, Studio Agent, Production Pipeline, Approval Workflow, Distribution Engine. 18 Tier 1 stories. Similar to WS1-A.
- WS3 (3 people, 15 pts/week): 3 deliverables (M3.17-M3.19) including cross-workstream integration. 60 capacity points. Should be manageable since WS3 core is already built by M2.

**Critical issue**: The -15% cross-team dependency velocity modifier kicks in during Weeks 11-14 (per build-map.md Section 8), reducing all teams' effective velocity. Combined with the fact that these are the first sprints where 4 teams are simultaneously consuming shared infrastructure, expect Platform Engineering to be pulled into support/debugging. The build map acknowledges this (40% build / 60% support in Weeks 11-14), but the milestone deliverables do not account for the reduced Platform Engineering output.

**Assessment**: Achievable only if M1 and M2 are delivered on time and shared infrastructure is genuinely stable. Any slip in M1 or M2 cascades directly into M3. Recommend adding 2 weeks of buffer (move M3 gate to Week 16 instead of Week 14).

### M4: Integrated Platform Launch (Weeks 15-18) -- FEASIBLE IF M3 DELIVERS

Integration testing and hardening phase. 50+ practices across all workstreams. External HIPAA compliance review.

**Concern**: External HIPAA compliance review requires 4-6 weeks of lead time for scheduling. Must be initiated no later than Week 12.

**Assessment**: Feasible if M3 is on track. The 4-week integration window is appropriate.

### M5-M7: Expansion, Rollout, Optimization (Weeks 19-36) -- FEASIBLE

Tier 2 and Tier 3 features with soft gates. Practice adoption scaling. These later milestones have appropriate scope and realistic timelines. The gradual ramp from 50 to 150 to 260 practices is sensible.

**Assessment**: Feasible. Lower risk than M1-M4 because the platform foundation exists.

---

## Dependency Sequencing Analysis

### Correctly Sequenced

1. **Infrastructure before features** (Track 1 before Tracks 4-6). This is the most important sequencing decision and it is correct. The build-once-serve-many architecture justifies the upfront investment.

2. **WS3 early start** (Track 2 starting Week 5). WS3's minimal dependency on PMS and Communications Service is correctly identified. This is a smart de-risking move that validates Kafka event flow before WS1 and WS2 come online.

3. **Event contract hardening before feature development** (Track 3 before Tracks 4-6). Contract testing at M2 prevents integration failures at M3/M4. Well-sequenced.

4. **Sequential Chain 2 (Data Flow)**: Practice Data Hub -> PMS Integration -> Funnel Data Lake -> AI Analysis -> Automation Engine. Correctly ordered; each layer depends on the one below.

5. **Sequential Chain 3 (Content Flow)**: Communications Service -> Capture Bot -> AI Pipeline -> Approval -> Distribution. Correct.

### Potential Sequencing Issues

1. **Design System vs. Infrastructure timing conflict.** The component-map.md specifies a 4-week design system delivery plan (tokens in Week 1, primitives in Week 2, patterns in Week 3, layouts in Week 4). The technical architecture places `@sgadental/design-system` in Phase 1 exit criteria. But the build-map.md Section 6 does not allocate any team to design system work in Weeks 1-6. Platform Engineering is 100% on shared services. WS teams have not started yet. This is an unresolved dependency.

2. **Storefront Generation (M3.5, Weeks 13-14) depends on Design Tool Abstraction (Canva adapter).** The build-map shows Design Tool Abstraction as a WS2 deliverable in Week 9, but the consolidated-epic.md lists it as shared infrastructure (Epic S-10). If WS2 builds it, they consume capacity from their own feature work. If Platform Engineering builds it, it must happen in Weeks 7-10 alongside contract hardening.

3. **Cross-workstream events #7 and #8 are verified at M4 (Week 18) but not tested until then.** Events `ws1.funnel-event` and `ws2.content-engagement-updated` are the last two verified. Consider adding contract test stubs at M2 even if the producers don't exist yet.

4. **Champion Network (M2.7) requires "6 weeks of huddle data" per Sequential Chain 4.** If WS3 starts in Week 5, huddle data collection begins ~Week 6 (after M1.5). Six weeks from Week 6 is Week 12, but Champion Network v1 is scheduled for Weeks 9-10 in M2. This is a sequencing contradiction. The Champion CRUD and event publishing can ship at Week 9-10, but the champion designation algorithm (which requires huddle history) cannot be fully operational until Week 12.

---

## Impediments & Risks

### Impediment #1: Design System Ownership Gap (BLOCKING)

The component-map.md defines a 4-week design system delivery plan. The technical architecture includes it in Phase 1 exit criteria. But the build-map.md allocates zero dedicated capacity. No team is assigned to build `@sgadental/ui` in Weeks 1-4.

**Impact**: If unresolved, workstream teams starting in Weeks 5-11 will have no shared component library and will build bespoke frontends, undermining the entire design consistency strategy.

**Resolution required**: Either (a) add a dedicated frontend/design engineer to the team (person #21), or (b) explicitly assign design system work to Platform Engineering and reduce their shared services scope in Phase 1, or (c) delay the design system to Weeks 5-8 and accept that WS3's early frontend work will be refactored later.

### Impediment #2: CCA Academy Content Production

The CCA Academy Bronze track (M2.6) requires 10 video training modules with scenario assessments. This is instructional design work, not engineering. The build plan treats it as an engineering deliverable assigned to the WS3 team (3 engineers).

**Impact**: If the WS3 team must produce training content, their engineering velocity drops significantly during M2.

**Resolution required**: Confirm that SGA's Learning & Development team (or an external vendor) will produce the 10 modules by Week 8. The WS3 engineering team builds the delivery platform; content comes from elsewhere.

### Impediment #3: Thin Capacity Buffers

The build-map.md Section 8 acknowledges ~10% buffer across all teams. After applying velocity modifiers (-30% new team, -15% cross-team dependencies, -10% production incidents), effective capacity shrinks further:

| Team | Raw Capacity (Tier 1) | After Modifiers | Demand | Effective Buffer |
|---|---|---|---|---|
| Platform Eng (5) | 200 pts | ~150 pts (new team penalty Weeks 3-6) | 180 pts | **-30 pts (deficit)** |
| WS3 (3) | 150 pts | ~120 pts | 120 pts | **0 pts** |
| WS1-A (4) | 160 pts | ~130 pts (cross-team penalty Weeks 11-14) | 150 pts | **-20 pts (deficit)** |
| WS1-B (3) | 120 pts | ~100 pts | 110 pts | **-10 pts (deficit)** |
| WS2 (4) | 160 pts | ~130 pts | 140 pts | **-10 pts (deficit)** |

When velocity modifiers are applied, every team is at or over capacity. The raw numbers work; the risk-adjusted numbers do not.

### Risk #1: BAA Procurement Delays

NexHealth and Anthropic BAAs are critical-path blockers. Healthcare BAAs commonly take 4-8 weeks. If these slip past Week 2, PMS Integration and AI Service timelines cascade.

**Probability**: Medium-High
**Mitigation**: Start BAA negotiations immediately (pre-M0). Use mock APIs for development until BAAs close.

### Risk #2: 10DLC Registration Delays

Twilio 10DLC campaign registration currently takes 2-6 weeks. This is an external dependency with no acceleration path.

**Probability**: Medium
**Mitigation**: Submit campaigns in M0. Use Toll-Free numbers as interim (noted in milestone-definitions.md).

### Risk #3: ClickHouse Operational Complexity

ClickHouse is powerful but operationally demanding. The WS1-B team (3 people, including 2 engineers) must stand up ClickHouse, design the funnel schema, build dbt views, and build the Executive Dashboard. If ClickHouse tuning absorbs more time than planned, the dashboard timeline slips.

**Probability**: Low-Medium
**Mitigation**: Use ClickHouse Cloud (managed service) as the risk-adjusted timeline suggests.

### Risk #4: Pilot Practice Engagement

M1.5 and M2 depend on 20 pilot practices actively participating. Dental practices are operationally busy; engagement is not guaranteed.

**Probability**: Medium
**Mitigation**: SGA Operations assigns a dedicated champion per pilot cohort (noted in milestone-definitions.md). Select practices with existing SGA relationship depth.

---

## Capacity Analysis

### Team Size vs. Work Volume

**160 discrete build items** (86 Tier 1 + 56 Tier 2 + 15 Tier 3 + 3 deferred) for a 20-person team over 36 weeks.

**Tier 1 (launch-critical) analysis**:
- 86 Tier 1 stories must be delivered by Week 18 (M4 gate).
- 20 people, but only 15 are engineers (5 are leads).
- Leads presumably code at ~50% capacity, giving an effective engineering headcount of ~17.5.
- At an assumed average of 2-3 story points per story, Tier 1 represents ~170-260 story points.
- 17.5 engineers at ~5 points/week/engineer = 87.5 points/week network-wide.
- Weeks 3-18 (16 weeks of active build) = ~1,400 total available points.
- Tier 1 demand of ~260 points is well within 1,400 available points.

**However**: Not all 17.5 engineers are building features simultaneously. In Weeks 3-10, only Platform Engineering (5) and WS3 (3) are active. In Weeks 11-18, all teams are active but Platform Engineering shifts to 40-80% support. The temporal distribution of work is front-loaded on Platform Engineering and back-loaded on workstream teams.

**Conclusion**: The aggregate capacity is sufficient. The constraint is temporal distribution -- too much depends on the first 10 weeks going to plan.

### Missing Roles

The 20-person team structure does not include:
1. **UX/Frontend design engineer** for the shared design system (Impediment #1)
2. **Instructional designer** for CCA Academy content (Impediment #2)
3. **DevOps/SRE** beyond 2 QA engineers -- who manages infrastructure in production (Weeks 15+)?
4. **Product Manager / Scrum Master** -- who runs ceremonies, manages the backlog, unblocks teams day-to-day?

These are support functions that consume budget and coordination overhead even if they are not counted in the 20-person engineering team.

---

## Recommendations

1. **Add 2-week buffer to M3.** Move the M3 gate from Week 14 to Week 16, and compress M4 integration from 4 weeks to 2 weeks (Weeks 17-18). M3 is the highest-risk milestone with the most parallel work and cross-team dependencies. A 2-week buffer absorbs one slip without cascading to M4.

2. **Resolve design system ownership before build start.** Assign a dedicated frontend engineer (add to team or reallocate from a workstream) to own `@sgadental/ui` delivery in Weeks 1-4. Alternatively, shift design system delivery to Weeks 5-8 and accept temporary UI inconsistency in WS3's early build.

3. **Confirm CCA content production ownership.** Before M0, secure commitment from SGA's L&D team or an external vendor to deliver 10 training video modules by Week 8. Document this as an M0 exit criterion.

4. **Initiate BAA negotiations immediately (pre-M0).** Do not wait for formal M0 kickoff. Every week of BAA lead time reduces the risk of M1 delays.

5. **Add Credly organization setup to M0 deliverables.** Credly API onboarding takes 2-4 weeks. Include it in the M0 procurement checklist alongside Twilio and NexHealth.

6. **Clarify Design Tool Abstraction (Canva adapter) ownership.** The consolidated-epic.md lists it as shared infrastructure (Epic S-10) while the build-map.md implies WS2 builds it. Assign explicitly to Platform Engineering in Weeks 7-8 (alongside contract hardening) since it is architecturally a shared service.

7. **Split Champion Network delivery.** Ship Champion CRUD + event publishing at M2 (Weeks 9-10) as planned. Defer the champion designation algorithm (which requires 6 weeks of huddle data) to M3 (Week 12+). This resolves the sequencing contradiction in Sequential Chain 4.

8. **Add contract test stubs for events #7 and #8 at M2.** Even though the producers (WS1 funnel events, WS2 content engagement) won't exist until M3, define the schemas and write consumer contract test stubs at M2. This prevents late-breaking schema mismatches at M4.

9. **Use ClickHouse Cloud (managed) from the start.** Do not self-host ClickHouse with a 3-person team. The operational overhead is not justified at 260-practice scale. Managed service eliminates tuning risk.

10. **Schedule external HIPAA compliance review by Week 12.** M4 gate requires it to pass. Lead time for healthcare compliance auditors is 4-6 weeks. Initiate procurement in M2.

---

## Items Requiring PM/Tech Lead Revision

The verdict is **Feasible with Caveats**, not "Not Feasible." The plan is fundamentally sound. The following items should be addressed before build start but do not require a full roadmap rewrite:

| # | Item | Owner | Action |
|---|---|---|---|
| 1 | Design system ownership gap | PM + Tech Lead | Add frontend engineer to team or reassign scope. Update build-map.md Section 6 with explicit allocation. |
| 2 | CCA Academy content production | PM + SGA Operations | Confirm L&D team commitment. Add as M0 exit criterion in milestone-definitions.md. |
| 3 | M3 buffer | PM | Consider shifting M3 gate from Week 14 to Week 16 or reducing M3 scope to create slack. |
| 4 | Design Tool Abstraction ownership | Tech Lead | Assign to Platform Engineering (Weeks 7-8) or explicitly to WS2 (Week 11). Update consolidated-epic.md and build-map.md to be consistent. |
| 5 | Champion Network sequencing | PM | Split M2.7 into CRUD (M2) + designation algorithm (M3). Update milestone-definitions.md. |
| 6 | Velocity modifiers applied to capacity table | PM | Re-run capacity analysis with modifiers applied (as shown in Impediment #3). Acknowledge that adjusted capacity shows deficits and document which stories are at risk of slipping to Tier 2 if velocity is below plan. |

---

*This review assesses the consolidated roadmap as fundamentally achievable by a 20-person team over 36 weeks. The architecture decisions (infrastructure-first, WS3 early start, 3-tier phasing) are sound. The primary risks are temporal: thin buffers in M1 and M3, unresourced design system work, and external procurement dependencies. Addressing the 6 revision items above converts this plan from "feasible with caveats" to "feasible."*
