# SGA Dental Partners — Build Map

**Date**: 2026-03-14
**Author**: Product Manager Agent (Phase 6)
**Purpose**: Map parallel vs. sequential build tracks, critical path, shared infrastructure dependencies, and team allocation across the consolidated 3-workstream platform.

---

## Build Philosophy

Three principles govern the build sequence:

1. **Infrastructure before features.** No workstream starts feature development until shared services are stable. The cost of building on unstable infrastructure is 3-5x the cost of waiting.
2. **WS3 starts first.** Culture OS has the fewest infrastructure dependencies (no PMS, no Twilio for core features). It validates shared infrastructure under real load while WS1 and WS2 finish building.
3. **Parallel tracks after foundations.** Once shared infrastructure is proven, WS1 and WS2 build in parallel. They share the same infrastructure layer but have minimal feature-level dependencies on each other.

---

## Section 1: Build Tracks

### Track Overview

| Track | Owner | Duration | Starts | Depends On |
|---|---|---|---|---|
| **Track 0: Pre-Build** | Eng Manager + Legal | Weeks 1-2 | Immediately | Nothing |
| **Track 1: Shared Infrastructure** | Platform Engineering (5) | Weeks 3-6 | After M0 | Track 0 |
| **Track 2: WS3 Culture OS** | WS3 Team (3) | Weeks 5-14 | After Practice Data Hub + Auth + Kafka | Track 1 (partial) |
| **Track 3: Event Contract Hardening** | Platform Engineering (5) | Weeks 7-10 | After Track 1 complete | Track 1 |
| **Track 4: WS1 Practice Operations** | WS1-A Team (4) | Weeks 11-18 | After M2 | Track 1, Track 3 |
| **Track 5: WS1 Network Intelligence** | WS1-B Team (3) | Weeks 11-18 | After M2 | Track 1, Track 3 |
| **Track 6: WS2 Content Engine** | WS2 Team (4) | Weeks 11-18 | After M2 | Track 1, Track 3 |
| **Track 7: Integration & Launch** | All teams | Weeks 15-18 | After Tracks 4-6 Tier 1 | Tracks 4, 5, 6 |
| **Track 8: Tier 2 Expansion** | All teams | Weeks 19-24 | After M4 | Track 7 |
| **Track 9: Full Rollout** | All teams + SGA Ops | Weeks 25-30 | After M5 | Track 8 |
| **Track 10: Optimization** | All teams | Weeks 31-36 | After M6 | Track 9 |

---

## Section 2: Parallel vs. Sequential Constraints

### What Can Build in Parallel

```
PARALLEL GROUP 1 (Weeks 3-6): Shared Infrastructure Services
  These 10 services have no dependencies on each other (all depend only on PostgreSQL/AWS):
  +---------+  +---------+  +----------+  +---------+
  |Practice |  |  Auth & |  |Compliance|  |  Event  |
  |Data Hub |  |Identity |  |Middleware|  |Bus/Kafka|
  +---------+  +---------+  +----------+  +---------+

  After Practice Data Hub is live (Week 4):
  +---------+  +---------+  +---------+  +---------+  +---------+
  |  PMS    |  | Comms   |  |   AI    |  |  Asset  |  |Workflow |
  |Integr.  |  |Service  |  |Service  |  |Registry |  | Engine  |
  +---------+  +---------+  +---------+  +---------+  +---------+
  + Design Tool Abstraction (after Asset Registry)

PARALLEL GROUP 2 (Weeks 5-10): WS3 Early Start + Contract Hardening
  These run simultaneously because they have no feature-level dependencies:
  +-------------------+     +-------------------+
  | WS3: Culture OS   |     | Platform Eng:     |
  | (H-Score, Huddle, |     | Contract testing, |
  |  CCA, Champions)  |     | load testing,     |
  |                   |     | DLQ, monitoring   |
  +-------------------+     +-------------------+

PARALLEL GROUP 3 (Weeks 11-18): WS1-A + WS1-B + WS2 + WS3 continues
  All three workstreams build Tier 1 features simultaneously:
  +-------------------+  +-------------------+  +-------------------+  +-------------------+
  | WS1-A: Practice   |  | WS1-B: Network    |  | WS2: Content      |  | WS3: Trust Xfer,  |
  | Ops (Nerve Center |  | Intelligence      |  | Engine (Capture,  |  | Recognition,      |
  | Concierge, Enroll,|  | (Funnel Lake,     |  | Studio Agent,     |  | Playbook Dist,    |
  | Storefront, Auto) |  | Dashboard, AI)    |  | Pipeline, Distro) |  | Cross-WS integ)   |
  +-------------------+  +-------------------+  +-------------------+  +-------------------+
```

### What Must Be Sequential

```
SEQUENTIAL CHAIN 1: Infrastructure -> Features
  Shared Infrastructure (M1)
    |
    v
  Event Contract Hardening (M2) --- cannot test contracts before services exist
    |
    v
  WS1 + WS2 Feature Development (M3) --- cannot build features on unproven infrastructure

SEQUENTIAL CHAIN 2: Data Flow
  Practice Data Hub
    |
    v
  PMS Integration Service --- needs practice records to scope PMS operations
    |
    v
  Funnel Data Lake --- needs PMS events as primary data source
    |
    v
  AI Analysis Engine --- needs funnel data for baseline calculations
    |
    v
  Automation Engine --- needs anomaly detection to trigger automated interventions

SEQUENTIAL CHAIN 3: Content Flow
  Communications Service
    |
    v
  Content Capture Bot --- sends capture prompts via Comms Service
    |
    v
  AI Processing Pipeline --- processes captured media
    |
    v
  Approval Workflow --- routes processed content to reviewers
    |
    v
  Distribution Engine --- publishes approved content

SEQUENTIAL CHAIN 4: WS3 Internal
  H-Score Engine --- must exist before huddle can show score
    |
    v
  Morning Huddle OS --- requires H-Score for "Today's Focus" targeting
    |
    v
  CCA Academy --- can start parallel to Huddle, but Champion Network depends on both
    |
    v
  Champion Network --- requires 6 weeks of huddle data + at least 1 Bronze certified staff
```

---

## Section 3: Critical Path

The critical path is the longest sequential dependency chain. Any delay on this path delays the entire platform launch.

```
CRITICAL PATH (determines M4 launch date):

Week 1-2:  BAA Procurement (Legal)
  |
Week 3-4:  Practice Data Hub + Auth & Identity
  |
Week 5-6:  PMS Integration Service + Communications Service
  |
Week 6:    M1 Gate — Shared Infrastructure verification
  |
Week 7-8:  Event contract testing framework + producer/consumer tests
  |
Week 9-10: Kafka load test + PMS coverage extension (Sikka ONE)
  |
Week 10:   M2 Gate — Event contracts verified
  |
Week 11-12: Funnel Data Lake (ClickHouse) + Practice Nerve Center v1
  |
Week 13-14: AI Analysis Engine + Executive Dashboard + Automation Engine v1
  |
Week 14:   M3 Gate — Tier 1 features live
  |
Week 15-16: Cross-workstream integration testing (all 10 event flows)
  |
Week 17-18: Production hardening + security audit
  |
Week 18:   M4 Gate — Integrated Platform Launch
```

**Critical path duration**: 18 weeks (4.5 months)

**Critical path risks**:
| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| BAA delays | +1-2 weeks | Medium | Start Day 1. Use synthetic data. |
| NexHealth PMS coverage gaps | +1-2 weeks at M2 | Medium | Audit in M0. Sikka ONE fallback. |
| ClickHouse setup/tuning | +1 week at M3 | Low | Consider ClickHouse Cloud (managed). |
| Cross-WS integration issues | +1-2 weeks at M4 | Medium | Contract testing catches issues at M2. |

---

## Section 4: Visual Timeline

### Weeks 1-18 (Launch)

```
Week:     1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16   17   18
          |----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
          M0        |         M1        |M1.5|       M2         |          M3          |     M4
          Pre-Build |  Shared Infra     |    |  Contracts+WS3   |  WS1+WS2+WS3 Tier1  |  Launch
          |         |                   |    |                  |                      |
PLATFORM  |=========|==================|====|==================|======================|========|
ENGR (5)  | BAAs    |PracHub Auth Compl | PMS extension       | Support + ops +              |
          | Env     |Kafka PMS Comms    | Contract tests      | integration testing          |
          | Setup   |AI Asset WF Design | Load test, DLQ      |                              |
          |         |                   | Monitoring          |                              |
          |         |                   |                     |                              |
WS3 (3)   |         |                   |====================|==============================|
          |         |                   | H-Score v1          | Trust Transfer  Recognition  |
          |         |                   | Huddle v1           | Playbook Dist   X-WS integ  |
          |         |                   | CCA Bronze          |                              |
          |         |                   | Champions v1        |                              |
          |         |                   |                     |                              |
WS1-A (4) |         |                   |                     |============================|
          |         |                   |                     | Nerve Center  Storefront    |
          |         |                   |                     | Concierge     Enrollment    |
          |         |                   |                     | Automation v1 Grad Autonomy |
          |         |                   |                     |                              |
WS1-B (3) |         |                   |                     |============================|
          |         |                   |                     | Funnel Lake   AI Analysis  |
          |         |                   |                     | Exec Dash     Connectors   |
          |         |                   |                     | Data Quality                |
          |         |                   |                     |                              |
WS2 (4)   |         |                   |                     |============================|
          |         |                   |                     | Capture Bot   Pipeline     |
          |         |                   |                     | Studio Agent  Approval     |
          |         |                   |                     | Production    Distribution |
          |         |                   |                     |                              |
QA (2)    |         |=========================================================|============|
          |         | Test infra setup  | Contract tests     | WS integration | E2E + perf |
          |         |                   | Load tests         | testing        | Security   |
```

### Weeks 19-36 (Scale + Optimize)

```
Week:    19   20   21   22   23   24   25   26   27   28   29   30   31   32   33   34   35   36
         |----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
                    M5                  |            M6             |           M7
              Tier 2 Expansion          |     Full Network Rollout  |     Tier 3 + Optimization
                                        |                          |
ALL TEAMS|==============================|==========================|==========================|
         | WS1: Rebooking, Reactivation | Scale to 260 practices   | A/B testing framework     |
         |       Alerts, Reports        | Self-service onboarding   | Learning loops            |
         |       Change management      | Ops handoff to SGA IT     | Predictive analytics      |
         | WS2: Mobile app, Video,      |                          | Attribution ML            |
         |       Print, LinkedIn        |                          | Voice AI feasibility      |
         | WS3: Alerts, Wow stories,    |                          | CCA renewal               |
         |       Champion library       |                          |                           |
Practices:        100        150                   260+                        260+
```

---

## Section 5: Shared Infrastructure Dependencies

### Service Dependency Heat Map

Which workstream features depend on which shared services:

| Shared Service | WS1 Practice Ops | WS1 Network Intel | WS2 Content Engine | WS3 Culture OS |
|---|---|---|---|---|
| Practice Data Hub | HIGH (all features) | HIGH (all dashboards) | MEDIUM (practice profiles for brand selection) | MEDIUM (practice profiles for benchmarking) |
| PMS Integration | HIGH (automation, concierge, nerve center) | HIGH (funnel events) | LOW (appointment events for capture prompts) | NONE (H-Score uses Cinnamon/Swell, not PMS) |
| Communications Service | HIGH (all patient comms) | NONE | MEDIUM (capture prompts, champion SMS) | NONE (uses Dental Intel, not Twilio) |
| AI Service | HIGH (concierge, message gen, anomaly analysis) | HIGH (AI analysis engine) | HIGH (caption gen, classification, brief extraction) | MEDIUM (trust transfer NLP) |
| Asset Registry | LOW (storefront assets) | NONE | HIGH (all content storage + delivery) | LOW (recognition posts, certificates) |
| Event Bus (Kafka) | HIGH (funnel events, automation triggers) | HIGH (event consumption for dashboards) | HIGH (asset events, capture events) | HIGH (H-Score, champion events) |
| Auth & Identity | HIGH (practice user auth) | HIGH (executive auth) | HIGH (HQ team auth) | HIGH (champion + practice auth) |
| Compliance Middleware | HIGH (PHI in patient comms) | MEDIUM (PHI in patient analytics) | LOW (no PHI in content) | LOW (no PHI in culture data) |
| Workflow Engine (Temporal) | HIGH (comms cadences, enrollment) | MEDIUM (batch analytics) | MEDIUM (content pipeline orchestration) | MEDIUM (H-Score daily calc) |
| Design Tool Abstraction | NONE | NONE | HIGH (all design generation) | NONE |

### Infrastructure Build Priority (by downstream impact)

| Priority | Service | Blocks |
|---|---|---|
| 1 | Practice Data Hub | Everything |
| 2 | Auth & Identity | All user-facing services |
| 3 | Compliance Middleware | All services processing PHI |
| 4 | Event Bus (Kafka) | All cross-workstream data flow |
| 5 | PMS Integration Service | WS1 (most features), WS2 (capture triggers) |
| 6 | Communications Service | WS1 (patient comms), WS2 (capture prompts) |
| 7 | AI Service | WS1 (concierge, analysis), WS2 (captions), WS3 (NLP) |
| 8 | Asset Registry | WS2 (primary), WS1 (storefront assets) |
| 9 | Workflow Engine | Long-running processes across all WS |
| 10 | Design Tool Abstraction | WS2 only |

---

## Section 6: Team Allocation

### Team Structure (20 people total)

| Team | Size | Primary Track | Secondary Track |
|---|---|---|---|
| Platform Engineering | 4 engineers + 1 lead | Track 1, Track 3 (Weeks 1-10) | Track 7 support (Weeks 11-18) |
| WS1: Practice Operations | 3 engineers + 1 lead | Track 4 (Weeks 11-18) | Track 8 expansion (Weeks 19+) |
| WS1: Network Intelligence | 2 engineers + 1 lead | Track 5 (Weeks 11-18) | Track 8 analytics expansion |
| WS2: Content Engine | 3 engineers + 1 lead | Track 6 (Weeks 11-18) | Track 8 distribution expansion |
| WS3: Culture OS | 2 engineers + 1 lead | Track 2 (Weeks 5-18) | Track 8 culture expansion |
| QA / DevOps | 2 engineers | All tracks (continuous) | Performance + security testing |

### Team Allocation by Phase

```
         Weeks 1-6        Weeks 7-10       Weeks 11-18      Weeks 19-30       Weeks 31-36
         (Infra)          (Contracts+WS3)  (All WS Tier 1)  (Expand+Rollout)  (Optimize)
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
Platform | Build all  |   | Contract  |    | Support + |    | Maintain  |    | Voice AI  |
Eng (5)  | shared svcs|   | hardening |    | integ test|    | + scale   |    | research  |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
WS3 (3)  | (not yet)  |   | H-Score   |    | Trust Xfer|    | Tier 2    |    | Tier 3    |
         |            |   | Huddle    |    | Recog     |    | features  |    | features  |
         |            |   | CCA, Champ|    | Playbook  |    |           |    |           |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
WS1-A(4) | (not yet)  |   | (not yet) |    | Nerve Ctr |    | Rebooking |    | A/B test  |
         |            |   |           |    | Concierge |    | Reactivat |    | Learning  |
         |            |   |           |    | Storefront|    | Change Mgmt    | Voice AI  |
         |            |   |           |    | Automation|    |           |    |           |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
WS1-B(3) | (not yet)  |   | (not yet) |    | Funnel DL |    | Alerts    |    | Predictive|
         |            |   |           |    | Exec Dash |    | Reports   |    | Analytics |
         |            |   |           |    | AI Engine |    | Outcome   |    | HQ Tools  |
         |            |   |           |    | Connectors|    | Tracking  |    |           |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
WS2 (4)  | (not yet)  |   | (not yet) |    | Capture   |    | Mobile App|    | Attribution|
         |            |   |           |    | Studio Agt|    | Video     |    | Auto-Approv|
         |            |   |           |    | Pipeline  |    | Print     |    | Kiosks    |
         |            |   |           |    | Approval  |    | LinkedIn  |    |           |
         |            |   |           |    | Distro v1 |    | GBP       |    |           |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
QA (2)   | Test infra |   | Contract  |    | WS integ  |    | Rollout   |    | Perf opt  |
         |            |   | + load    |    | + E2E     |    | QA        |    |           |
         +-----------+    +-----------+    +-----------+    +-----------+    +-----------+
```

### Platform Engineering Transition Plan

Platform Engineering is the critical-path team. After Weeks 1-10 (infrastructure + contracts), they transition:

| Phase | Platform Eng Focus | % of Time |
|---|---|---|
| Weeks 1-6 | Building shared services | 100% |
| Weeks 7-10 | Contract hardening + PMS extension | 80% build, 20% support |
| Weeks 11-14 | Support workstream integration, fix infrastructure issues | 40% build, 60% support |
| Weeks 15-18 | Integration testing, performance tuning, security audit | 20% build, 80% ops |
| Weeks 19+ | Ongoing maintenance, scaling, new shared features | 30% build, 70% ops |

---

## Section 7: Cross-Workstream Integration Milestones

These are the 10 event flows that must be verified at specific points:

| # | Event Flow | Producer | Consumer | First Live | Verified At |
|---|---|---|---|---|---|
| 1 | `ws3.champion-assigned` -> WS1, WS2 | WS3 | WS1 Exec Dash, WS2 Capture Bot | Week 9 | M2 |
| 2 | `ws3.hscore-calculated` -> WS1 | WS3 | WS1 Exec Dash | Week 7 | M1.5 |
| 3 | `ws3.benchmark-taxonomy-updated` -> WS1, WS2 | WS3 | WS1, WS2 | Week 10 | M2 |
| 4 | `ws3.champion-story-submitted` -> WS2 | WS3 | WS2 Content Pipeline | Week 14 | M3 |
| 5 | `pms.appointment-completed` -> WS2 | Shared PMS | WS2 Capture Bot | Week 12 | M3 |
| 6 | `ws2.content-asset-approved` -> WS1 | WS2 | WS1 Storefront | Week 14 | M3 |
| 7 | `ws1.funnel-event` -> WS2 | WS1 | WS2 Attribution | Week 15 | M4 |
| 8 | `ws2.content-engagement-updated` -> WS1 | WS2 | WS1 Exec Dash | Week 15 | M4 |
| 9 | `ws3.training-completed` -> WS1 | WS3 | WS1 Exec Dash | Week 12 | M3 |
| 10 | `ws1.onboarding-status-changed` -> WS3 | WS1 | WS3 Culture Activation | Week 14 | M3 |

### Integration Verification Schedule

```
M1.5 (Week 7):   Event #2 (H-Score -> Dashboard) — first cross-WS event in production
M2 (Week 10):    Events #1, #3 — WS3 -> WS1/WS2 events verified via contract tests
M3 (Week 14):    Events #4, #5, #6, #9, #10 — WS1/WS2/WS3 cross-flows live
M4 (Week 18):    Events #7, #8 — all 10/10 events flowing in production
```

---

## Section 8: Capacity & Velocity Assumptions

### Story Point Capacity

| Team | Velocity (pts/week) | Weeks Available (Tier 1) | Tier 1 Capacity | Tier 1 Demand |
|---|---|---|---|---|
| Platform Eng (5) | 25 | 8 (Weeks 3-10) | 200 pts | ~180 pts (shared infra) |
| WS3 (3) | 15 | 10 (Weeks 5-14) | 150 pts | ~120 pts (WS3 Tier 1) |
| WS1-A (4) | 20 | 8 (Weeks 11-18) | 160 pts | ~150 pts (Practice Ops Tier 1) |
| WS1-B (3) | 15 | 8 (Weeks 11-18) | 120 pts | ~110 pts (Network Intel Tier 1) |
| WS2 (4) | 20 | 8 (Weeks 11-18) | 160 pts | ~140 pts (Content Engine Tier 1) |

**Assessment**: Tier 1 delivery fits within capacity with ~10% buffer. Buffer is thin -- schedule risk is real if infrastructure takes longer than planned.

### Velocity Modifiers

| Factor | Impact on Velocity | Applied When |
|---|---|---|
| New team (first 2 sprints) | -30% | Weeks 3-6 for all teams |
| Cross-team dependencies | -15% | Weeks 11-14 (integration overhead) |
| Production incidents | -10% | Weeks 15+ (operational support) |
| Holiday weeks | -100% | Plan around known holidays |

---

## Section 9: Decision Log

| # | Decision | Rationale | Alternatives Considered |
|---|---|---|---|
| D1 | WS3 starts 4 weeks before WS1/WS2 | Fewest infrastructure dependencies. Validates Kafka under real load. Delivers early value (M1.5). | All WS start simultaneously (rejected: shared infrastructure not ready). |
| D2 | WS1 split into two sub-teams | 32 features serving two distinct user groups. Independent deployment cycles. | Single WS1 team (rejected: too many context switches between practice-facing and executive-facing). |
| D3 | Platform Engineering builds ALL shared services before workstream work starts | Prevents duplicate builds, ensures consistent APIs, avoids workstream-specific hacks on infrastructure. | Each workstream builds its own infra (rejected: the entire consolidation rationale is shared infrastructure). |
| D4 | Voice AI deferred to v2 | NexHealth write-path needs production stability proof. Text-first approach has lower risk. Voice AI adds 13+ story points of high-complexity work. | Include Voice AI in Tier 1 (rejected: risk of NexHealth write instability, Twilio Media Streams latency uncertainty). |
| D5 | 42 interfaces phased into 3 tiers | 42 simultaneous interfaces would overwhelm teams. Tier 1 (16) delivers the core user journeys. Tiers 2-3 expand once core is stable. | Build all 42 in Tier 1 (rejected: impossible within team capacity and timeline). |
| D6 | Phase 1.5 milestone at Week 6-7 | Delivers tangible value to 20 practices before full platform launch. Tests infrastructure. Gives leadership proof points. | No early value delivery (rejected: 18-week wait for any user-facing output is too long for stakeholder confidence). |

---

*This build map provides the full parallel/sequential analysis, critical path, team allocation, and integration schedule for QC-4 review. All decisions incorporate carry-forward items from QC-1 through QC-3.*
