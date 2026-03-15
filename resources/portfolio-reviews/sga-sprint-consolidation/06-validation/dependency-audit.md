# Cross-Workstream Dependency Audit

**Date**: 2026-03-14
**Author**: Product Strategist Agent (Phase 7)
**Artifacts Reviewed**: workstream-visions.md, consolidation-feature-map.md, experience-architecture.md, unified-dashboard-specs.md, practice-interface-specs.md, cross-workstream-flows.md, component-map.md, technical-architecture.md, consolidated-epic.md, milestone-definitions.md, build-map.md, qc4-scrum-master-review.md, qc4-product-owner-review.md

---

## Executive Summary

The consolidated 3-workstream model is structurally sound and genuinely simpler than the original 6-sprint architecture. It reduces 141 user stories to 110 (22% elimination through deduplication), collapses 4 independent PMS integrations into 1 shared service, and consolidates 6 separate LLM integrations into a single AI Service. However, this audit identifies 3 areas requiring attention: (1) the shared infrastructure layer carries concentrated risk -- 7 of 10 shared services sit on the critical path, and a failure in any one delays all 3 workstreams; (2) scope creep has introduced 12 capabilities not present in the original sprints; and (3) one new redundancy exists in QC/approval workflows that the consolidation introduced rather than eliminated.

---

## Shared Services Coverage Matrix

10 shared services support 3 workstreams. Coverage assessment based on technical-architecture.md API surfaces and consolidated-epic.md consumer mappings.

| Shared Service | WS1 Practice Ops | WS1 Network Intel | WS2 Content Engine | WS3 Culture OS | Coverage Gaps |
|---|---|---|---|---|---|
| **S-1: Practice Data Hub** | HIGH (all features read/write practice profiles) | HIGH (all dashboards keyed to practice_id) | MEDIUM (reads brand_universe for template selection) | MEDIUM (reads practice profiles for benchmarking) | None. All workstreams served. |
| **S-2: PMS Integration** | HIGH (automation triggers, concierge booking, nerve center agenda) | HIGH (funnel events are primary data source for ClickHouse) | LOW (appointment-completed events drive capture prompts) | NONE (H-Score uses Cinnamon/Swell, not PMS directly) | **Gap**: WS3 has no PMS dependency, which is correct -- but the Dental Intel component of H-Score (20% weight) derives from PMS appointment completion data. The technical architecture does not specify how Dental Intel receives this data. If Dental Intel pulls from PMS independently, this is fine. If it needs appointment data from the platform, WS3 has an undocumented transitive PMS dependency. |
| **S-3: Communications Service** | HIGH (all patient SMS, voice, email) | NONE | MEDIUM (capture prompts via SMS, champion SMS recaps) | NONE | None. WS3 correctly avoids Twilio -- huddle content flows through Dental Intel. |
| **S-4: AI Service** | HIGH (concierge reasoning, message generation, anomaly analysis) | HIGH (AI Analysis Engine uses Claude for root cause, action generation) | HIGH (brief extraction, caption generation, content classification) | MEDIUM (trust transfer NLP via Claude, call analysis via Whisper) | None. All workstreams served with appropriate cost-tier routing. |
| **S-5: Event Bus (Kafka)** | HIGH (funnel events, automation triggers) | HIGH (consumes events from all 3 workstreams for dashboards) | HIGH (asset lifecycle events, capture events) | HIGH (H-Score events, champion events, training events) | None. 10 event contracts defined. All producers and consumers mapped. |
| **S-6: Auth & Identity** | HIGH (practice user auth, role-filtered PWA) | HIGH (executive auth, regional scoping) | HIGH (HQ content team auth) | HIGH (champion auth, practice staff auth) | None. 8-role model covers all actor types across all workstreams. |
| **S-7: Compliance & Audit** | HIGH (PHI in patient comms) | MEDIUM (PHI in analytics -- hashed patient IDs) | LOW (no PHI in content assets -- but see note below) | LOW (no PHI in culture data) | **Gap**: Before/after dental photos processed through Cloudinary may constitute PHI. The technical architecture acknowledges this (Section 1.8, "CRITICAL" note) but the compliance middleware's PHI detection (Section 1.8, item 2) is designed for text-based PHI patterns (SSN, DOB, names). Image-based PHI detection (patient faces in before/after photos) is not addressed in the compliance middleware specification. |
| **S-8: Asset Registry** | LOW (storefront gallery reads approved assets) | NONE | HIGH (all content storage, production pipeline output) | LOW (brand templates for recognition posts, certificate images) | None. Cross-workstream asset flow (WS2 produces, WS1 reads, WS3 caches) is well-defined. |
| **S-9: Workflow Engine (Temporal)** | HIGH (enrollment pipeline, communication cadences) | MEDIUM (batch analytics, nightly PMS sync) | MEDIUM (content production pipeline) | MEDIUM (H-Score daily calculation, weekly brand template sync) | None. Temporal worker pools isolated per workstream. |
| **S-10: Design Tool Abstraction** | NONE | NONE | HIGH (all Canva template generation) | NONE | None. Single-workstream consumer -- appropriately scoped as shared infrastructure for future extensibility (ADR-07 abstraction). |
| **S-11: Non-PMS Data Sources** | MEDIUM (missed call detection, ad attribution) | MEDIUM (ad spend for funnel attribution) | LOW (social engagement metrics) | MEDIUM (Cinnamon phone analytics, Swell reviews for H-Score) | **Gap**: This service was added in QC-2/QC-3 carry-forward (F-01). It consolidates Cinnamon, Swell, RingCentral, 8x8, Weave, Google Ads, Meta Ads, Sierra, Agnes, iSolved, and social platform APIs. However, WS3's H-Score Engine (WS3-1.1, WS3-1.2) lists direct Cinnamon and Swell API integrations as discrete user stories, not as consumers of S-11. The technical architecture (Section 1.11) includes Cinnamon and Swell adapters in the Non-PMS service, but the consolidated epic does not. This creates ambiguity: does the WS3 team build Cinnamon/Swell adapters, or does Platform Engineering build them as part of S-11? |

### Coverage Summary

- **7 of 10** shared services are consumed by 3+ workstreams (Practice Data Hub, AI Service, Kafka, Auth, Compliance, Workflow Engine, Non-PMS Data Sources)
- **2 of 10** are consumed by 2 workstreams (Communications Service: WS1+WS2; Asset Registry: WS2+WS1+WS3)
- **1 of 10** is consumed by 1 workstream (Design Tool Abstraction: WS2 only)
- **3 gaps identified** requiring clarification (Dental Intel PMS dependency, image-based PHI detection, Cinnamon/Swell ownership)

---

## Scope Creep Assessment

I compared every feature in the consolidated architecture against the 141 original sprint stories. Features that appear in the consolidation but were NOT in any original sprint represent scope creep -- either justified additions or unjustified expansion.

### New Capabilities Not in Original Sprints

| # | New Capability | Where It Appears | Justified? | Rationale |
|---|---|---|---|---|
| SC-1 | Google Business Profile posting (WS2-6.9) | consolidated-epic.md | **Yes** | Was implicit in Sprint 3's Channel Sync Engine and Sprint 6's distribution rules but never a discrete story. Correctly elevated to explicit scope. QC-4 Product Owner confirmed this as a "good addition." |
| SC-2 | Content Pipeline Health widget on Executive Dashboard | unified-dashboard-specs.md, Section 1.4 | **Yes** | Design Director carry-forward item. Executives need visibility into content pipeline bottlenecks. Adds a WS2-to-WS1 touchpoint but serves a real user need. |
| SC-3 | Non-PMS Data Source Integration Service (S-11, 11 adapters) | technical-architecture.md, Section 1.11 | **Partially** | QC-2/QC-3 carry-forward. The service itself is justified (centralizes vendor integrations). But the scope is large: 11 vendor adapters across 5 categories (phone systems, ad platforms, AI phone agents, HR systems, social platforms). The original sprints specified only Cinnamon, Swell, and NexHealth as integrations. Adding RingCentral, 8x8, Weave, Google Ads, Meta Ads, Sierra, Agnes, iSolved, Instagram Graph, Facebook Graph, and Google Business Profile API represents significant new scope. **6 of these 11 adapters have no corresponding user story in the consolidated epic.** |
| SC-4 | Breach Notification & Incident Response architecture (S-12) | technical-architecture.md, Section 1.12 | **Yes** | QC-2/QC-3 carry-forward (F-09). HIPAA requirement. Not optional. |
| SC-5 | Disaster Recovery & Backup architecture (S-13) | technical-architecture.md, Section 1.13 | **Yes** | QC-2/QC-3 carry-forward (F-10). Operational necessity. |
| SC-6 | PWA role-based screen filtering (4 role profiles) | practice-interface-specs.md, Section 1 | **Yes** | Prototype Lead carry-forward (#4). Original sprints designed a single 7-screen PWA. Filtering by role is a UX improvement, not scope expansion. |
| SC-7 | Offline behavior + service worker + IndexedDB sync | practice-interface-specs.md, Section 7 | **Partially** | Sprint 4 mentioned offline as WS1-A1.6 (Tier 3). The interaction specs elevate this to a comprehensive offline architecture with per-screen capability matrices, sync patterns, and PHI caching restrictions. The granularity exceeds the original Tier 3 story scope significantly. |
| SC-8 | Voice AI TTY/TDD support (ADA compliance) | practice-interface-specs.md, Section 9.1 | **Yes** | WCAG carry-forward. ADA requirement for healthcare phone services. |
| SC-9 | Content Engine champion integration (unified champion scorecard) | cross-workstream-flows.md, Journey 3.3 | **Yes** | Directly addresses the WS2-WS3 champion overlap identified in the feature map (3.05). |
| SC-10 | Brand template weekly batch sync (T-16 fix) | cross-workstream-flows.md, Section 2.2 | **Yes** | Design Director carry-forward. Decouples WS3 recognition posts from real-time WS2 dependency. Good architectural decision. |
| SC-11 | Inline brand deviation fix in QC Queue (T-10 fix) | cross-workstream-flows.md, Section 2.1 | **Yes** | Design Director carry-forward. Embeds fix inline instead of linking to external surface. |
| SC-12 | Shared `@sgadental/ui` component library with 10 shared + 19 workstream-specific components | component-map.md | **Partially** | Prototype Lead carry-forward (#3). A shared design system is architecturally necessary. But the specification is extremely detailed (29 components, full state matrices, WCAG testing requirements) for a Phase 1 deliverable. As the Scrum Master review notes, no team is allocated to build it. |

### Scope Creep Verdict

- **8 of 12** new items are justified (carry-forwards, compliance requirements, or correctly elevated implicit capabilities)
- **3 of 12** are partially justified (Non-PMS integrations, offline architecture, design system) -- the capability is needed but the scope exceeds what was committed in original sprints
- **1 of 12** is a net-new capability with no original sprint basis (Google Business Profile posting -- but correctly added)

**Net scope impact**: The 12 additions collectively represent approximately 15-20 additional user stories worth of work that were not in the original 141. The justified items (compliance, WCAG, carry-forwards) account for most of this. The Non-PMS Data Source Integration Service (SC-3) is the largest single scope expansion, adding 11 vendor adapters where the originals specified 3.

### Recommendation

Reduce S-11 (Non-PMS Data Sources) to the 5 adapters that have corresponding user stories: Cinnamon (WS3-1.1), Swell (WS3-1.2), RingCentral/phone systems (WS1-B4.1), Google Ads/Meta Ads (WS1-B4.2), and iSolved HR (WS1-B4.4). The remaining 6 adapters (8x8, Weave, Sierra, Agnes, Instagram Graph, Facebook Graph) should be documented as Tier 2 or Tier 3 extensions, not Phase 1 shared infrastructure.

---

## Redundancy Check

### Did consolidation eliminate redundancies?

**Yes, significantly.** The deduplication eliminated 31 stories across 12 categories. The most impactful eliminations:

| Category | Before Consolidation | After | Reduction |
|---|---|---|---|
| PMS integrations | 4 independent connector designs | 1 shared service (dual-layer: NexHealth + Sikka ONE) | 75% |
| Practice data stores | 3 independent profile stores | 1 Practice Data Hub | 67% |
| LLM integrations | 4 independent Claude API integrations | 1 AI Service with shared prompt registry | 75% |
| SMS/Twilio integrations | 2 independent Twilio setups | 1 Communications Service with unified consent | 50% |
| Executive dashboards | 2 independent dashboard designs | 1 unified Network Intelligence surface | 50% |
| QC/approval queues | 3 independent approval workflows | 1 unified QC system with workstream context | 67% |

### Did consolidation introduce NEW redundancies?

**One partial redundancy identified.**

| # | Redundancy | Workstreams | Severity | Resolution |
|---|---|---|---|---|
| R-1 | **QC/approval split between WS1 and WS2** | WS1 (storefront QC in Network Intelligence, WS1-B1.7) + WS2 (content editorial QC, WS2-5.1) | LOW | The deduplication category D-07 merged QC queues and called it "one QC workflow with workstream context." But the consolidated epic creates two separate stories: WS1-B1.7 (HQ QC queue for brand review) and WS2-5.1 (HQ QC queue kanban with approve/reject). The experience architecture places storefront QC in Network Intelligence and content QC in the Content Engine dashboard. While they use the same `@sgadental/ui` components, they are functionally separate queues with separate review criteria, separate reviewers, and separate backend services. This is not a true code-level redundancy -- the shared component library prevents UI duplication -- but it is an organizational redundancy where two HQ teams manage similar "review and approve" workflows in different surfaces. **This may be intentional** (different review expertise), but it contradicts the D-07 deduplication claim of "one QC system." |
| R-2 | **Champion tracking in WS2 and WS3** | WS2 (champion scorecard, WS2-7.1) + WS3 (champion network, WS3-5.x) | LOW | Addressed by the champion integration touchpoint. WS3 owns the champion roster and culture scoring. WS2 owns content capture scoring. The combined scorecard is rendered in the PWA Champion Hub. Data flows via Kafka events (`ws2.content-engagement-updated` and `ws3.champion-assigned`). Not a redundancy -- it is a deliberate separation of concerns with an integration point. **Verdict: Not redundant.** |
| R-3 | **Practice health scoring** | WS1 (network health score, WS1-B1.4 -- storefront digital health) + WS2 (practice content health score, WS2-7.2) + WS3 (H-Score, WS3-1.3) | LOW | Three workstreams each compute a "health score" for a practice. WS1: digital presence health. WS2: content capture/quality health. WS3: hospitality behavior health. These are distinct metrics measuring different things, surfaced on different dashboards. The Executive Dashboard shows all three. **Not redundant -- complementary metrics.** |

### Redundancy Verdict

The consolidation eliminated far more redundancy than it introduced. The one partial redundancy (R-1, QC queue split) is a minor organizational concern, not a technical one. The shared component library ensures UI consistency regardless.

---

## Simplification Assessment

### Is the consolidated model genuinely simpler?

**Quantitative comparison:**

| Metric | Before (6 Sprints) | After (3 Workstreams) | Change |
|---|---|---|---|
| User stories | 141 | 110 | -22% |
| Independent data stores | 6+ (each sprint designed its own) | 3 (PostgreSQL shared, ClickHouse analytics, Redis cache) | -50%+ |
| PMS connectors | 4 independent designs | 1 shared service, 2 adapters | -75% |
| LLM integrations | 4-6 independent | 1 shared AI Service | -75%+ |
| Twilio integrations | 2 independent | 1 Communications Service | -50% |
| Event bus implementations | 0 explicit, ad-hoc | 1 Kafka cluster, 10 contracts | Centralized (net new but necessary) |
| User-facing surfaces | 6+ (one per sprint) | 5 (Practice Ops, Network Intel, Content Engine, Culture OS, Patient-Facing) | -17% |
| Shared component libraries | 0 (each sprint had its own UI) | 1 (`@sgadental/ui`) | Centralized |
| Authentication systems | 3-6 (each sprint designed its own RBAC) | 1 (Cognito + RBAC service) | -67%+ |
| Deployment targets | 6+ independent | 3 workstreams + shared infra on ECS | -50%+ |

### Complexity Trade-offs

The consolidation reduces feature-level complexity but introduces architectural complexity:

| Simplification Gain | Complexity Cost |
|---|---|
| Single Practice Data Hub eliminates data sync issues | All 3 workstreams depend on one PostgreSQL instance (single point of failure mitigated by Multi-AZ + read replicas) |
| Kafka event bus decouples workstreams | 10 event contracts require schema governance, consumer lag monitoring, and DLQ handling -- none of which existed before |
| Shared AI Service prevents prompt duplication | Priority queue and cost-tier routing add operational complexity |
| Unified Auth eliminates per-sprint login | 8-role model is more complex than any single sprint's auth design |
| Single Communications Service prevents consent conflicts | Message routing rules (7-step pipeline) are more complex than a direct Twilio call |

### Simplification Verdict

**The consolidated model is meaningfully simpler at the feature level and at the user experience level**, but it shifts complexity from "duplicated simple systems" to "shared sophisticated systems." This is the correct trade-off for a 260-practice platform -- the operational cost of maintaining 6 independent systems would have been far higher than managing 10 well-designed shared services.

**Key metric**: The consolidation reduces the number of team-to-team coordination points from O(n^2) across 6 sprints (potentially 15 bilateral integration pairs) to a hub-and-spoke model where all workstreams communicate through 10 shared services and 10 Kafka event contracts. This is architecturally cleaner.

---

## Cross-Workstream Dependency Map

### Which workstream features depend on other workstreams?

| Dependent Feature | Source Workstream | Dependency Workstream | Dependency Type | Health Assessment |
|---|---|---|---|---|
| H-Score widget on Today screen (T-1) | WS1 Practice Ops | WS3 Culture OS | Kafka event `ws3.hscore-calculated` consumed daily | **Healthy**: Async, batch, graceful degradation (shows cached value if stale). No runtime dependency. |
| Content capture action card (T-2) | WS1 Practice Ops | WS2 Content Engine | Kafka event triggers capture brief; camera overlay embedded in PWA | **Healthy**: Capture UI is embedded, not linked. Upload goes to WS2 backend but user stays in WS1 surface. |
| Asset Gallery in Storefront Manager (T-3) | WS1 Practice Ops | WS2 Content Engine | API read from Asset Registry for approved assets | **Moderate risk**: This is a synchronous API dependency. If Asset Registry is down, Storefront Manager's Asset Gallery shows cached thumbnails (per offline spec). But "Push to GMB" and "Use on Website" actions require live API calls to Asset Registry. |
| Culture Tab in Practice Deep Dive (T-4) | WS1 Network Intel | WS3 Culture OS | Kafka events `ws3.hscore-calculated`, `ws3.training-completed` consumed into ClickHouse | **Healthy**: Async ingestion into ClickHouse. Dashboard queries ClickHouse, not WS3 directly. Full decoupling. |
| Content Pipeline Health widget (T-5) | WS1 Network Intel | WS2 Content Engine | Kafka events `ws2.capture-media-received`, `ws2.content-asset-created`, `ws2.content-asset-approved`, `ws2.content-asset-published` | **Healthy**: Async event aggregation. Widget shows stage counts from ClickHouse. 30-minute refresh cycle. |
| Champion content scoring in Champion Hub | WS3 Culture OS | WS2 Content Engine | Kafka event `ws2.content-engagement-updated` | **Healthy**: Async batch update. Champion Hub shows combined culture + content scores. |
| Content capture prompts driven by appointments | WS2 Content Engine | Shared PMS Integration (via WS1 events) | Kafka event `pms.appointment-completed` | **Healthy**: Standard event consumption. If PMS events are delayed, capture prompts are delayed but not broken. |
| Brand templates for recognition posts | WS3 Culture OS | WS2 Content Engine | Weekly batch sync from Asset Registry | **Healthy**: Pre-cached, no runtime dependency. T-16 fix explicitly decoupled this. |
| Content-to-booking attribution | WS2 Content Engine | WS1 Practice Ops | Kafka event `ws1.funnel-event` correlated with `ws2.content-asset-published` | **Healthy**: ClickHouse query joining two event streams. No runtime cross-workstream call. |
| Practice enrollment triggers culture onboarding | WS3 Culture OS | WS1 Practice Ops | Kafka event `ws1.onboarding-status-changed` | **Healthy**: Async trigger. WS3 creates practice entry and begins culture onboarding automatically. |
| Brand deviation QC (inline fix) | WS1 Network Intel | WS2 Content Engine | Kafka event with deviation flag + replacement asset ID; API call to Asset Registry for swap | **Moderate risk**: "Swap to Fix" action requires a write API call to Asset Registry. If WS2 backend is down, the swap cannot execute. Mitigation: queue the swap for retry. |

### Dependency Health Summary

- **9 of 11** cross-workstream dependencies are **healthy** (async, event-driven, gracefully degrading)
- **2 of 11** carry **moderate risk** (synchronous API calls to Asset Registry for storefront actions and brand deviation swaps)
- **0** are **fragile** (no hard runtime dependencies between workstreams for core user journeys)

The event-driven architecture (Kafka) is the primary reason the dependency health is strong. Cross-workstream data flows through events, not synchronous API calls. The two moderate-risk items both involve the Asset Registry, which is a shared service -- not a cross-workstream dependency per se, but a shared infrastructure dependency that affects WS1 features consuming WS2-produced data.

---

## Risk Assessment

### Top Risks to the Consolidated Model

| # | Risk | Probability | Impact | Affected Workstreams | Mitigation Status |
|---|---|---|---|---|---|
| R1 | **Shared infrastructure single point of failure** | Medium | Critical | All | Multi-AZ PostgreSQL, read replicas, Kafka replication factor 3 -- mitigated at infrastructure level. But an operational incident (bad schema migration, misconfigured IAM policy) can take down all 3 workstreams simultaneously. **No workstream-level isolation exists** beyond Kafka consumer group independence. |
| R2 | **Platform Engineering team is the bottleneck** | High | High | All | Acknowledged by Scrum Master review. 5-person team builds all 10 shared services, then shifts to support mode. If M1 slips, all downstream milestones cascade. The -30% new-team velocity penalty makes this worse. **Mitigation status: Acknowledged but not resolved.** |
| R3 | **Design system ownership gap** | High | Medium | All (UI consistency) | Identified by Scrum Master (Impediment #1). No team allocated to build `@sgadental/ui`. **Unresolved.** |
| R4 | **Kafka schema governance overhead** | Medium | Medium | All | 10 event contracts require forward-compatible evolution rules. Schema Registry enforces this technically, but organizational discipline (who approves schema changes?) is not defined. A breaking schema change by one workstream could disrupt consumers in other workstreams. |
| R5 | **Non-PMS Data Sources service scope creep** | Medium | Low-Medium | WS1, WS3 | S-11 specifies 11 vendor adapters. Only 5 have corresponding user stories. The unbounded scope could absorb Platform Engineering capacity that should go to core services. |
| R6 | **CCA Academy content production** | High | Medium | WS3 | Identified by Scrum Master (Impediment #2). 10 video training modules are not engineering work but are assigned to the WS3 engineering team. If SGA L&D does not produce content, WS3 engineering velocity drops. **Unresolved.** |
| R7 | **Capacity deficit after velocity modifiers** | High | High | All | Scrum Master analysis shows every team operates at or over capacity when -30% (new team) and -15% (cross-team) velocity modifiers are applied. Platform Engineering shows a -30 point deficit; WS1-A shows -20. **Any slip compounds.** |
| R8 | **Image-based PHI detection gap** | Low | High | WS2, Compliance | Before/after dental photos may contain patient faces (PHI). The compliance middleware detects text-based PHI patterns but not image-based PHI. A breach involving patient photos without proper de-identification could trigger HIPAA notification obligations. |
| R9 | **Cloudinary BAA gap** | Medium | High | WS2 | Technical architecture (Section 1.8) notes Cloudinary processes before/after photos that may be PHI, but lists Cloudinary as "No BAA required." The mitigation (de-identify patient-media mapping) is a process control, not an architectural safeguard. If a Cloudinary breach exposes photos with metadata linking to patients, the lack of BAA creates legal exposure. |
| R10 | **M3 milestone overload** | High | High | WS1, WS2 | 19 deliverables in 4 weeks across 4 parallel teams. Scrum Master recommends adding 2-week buffer. **Not yet adopted.** |

---

## Recommendations

1. **Resolve the design system ownership gap before build start.** Add a dedicated frontend engineer (person #21) or explicitly reallocate Platform Engineering capacity in Weeks 1-4. The `@sgadental/ui` library is a dependency for all 5 user-facing surfaces. Without it, workstream teams will build inconsistent UIs that require expensive refactoring. This is the single most impactful unresolved item from the QC-4 reviews.

2. **Narrow Non-PMS Data Sources (S-11) to 5 adapters in Phase 1.** Build only the adapters with corresponding user stories: Cinnamon, Swell, phone systems (RingCentral as primary), ad platforms (Google Ads + Meta), and iSolved HR. Defer 8x8, Weave, Sierra, Agnes, Instagram Graph, and Facebook Graph to Tier 2. This reduces Platform Engineering's Phase 1 load by approximately 6 adapter builds.

3. **Clarify Cinnamon/Swell adapter ownership.** The consolidated epic assigns Cinnamon integration (WS3-1.1) and Swell integration (WS3-1.2) to the WS3 team, but the technical architecture places these adapters in S-11 (Platform Engineering). Assign ownership explicitly: either WS3 builds them as workstream-specific integrations, or Platform Engineering builds them as S-11 adapters and WS3 consumes them via the unified query API. The latter is architecturally cleaner but adds to Platform Engineering's already-strained M1 scope.

4. **Add a Kafka schema governance process to M0 exit criteria.** Define who can approve schema changes for each of the 10 event contracts. Assign a "contract owner" (producing workstream lead) and require consumer impact assessment before any schema modification. This prevents breaking changes during parallel development in M3.

5. **Address the image-based PHI gap in Compliance Middleware.** Add a requirement to S-7 (Compliance & Audit) for image metadata stripping before upload to Cloudinary. At minimum: strip EXIF data, remove patient identifiers from filenames, and ensure the Cloudinary metadata catalog contains no patient-linkable data. Consider whether a Cloudinary BAA is actually required given that de-identified photos may still be recognizable.

6. **Adopt the Scrum Master's 2-week M3 buffer recommendation.** Move the M3 gate from Week 14 to Week 16. This absorbs the velocity deficit identified in the capacity analysis without changing M4's Week 18 target (compress M4 integration from 4 weeks to 2 weeks, which is feasible if event contracts are hardened at M2 as designed).

7. **Confirm CCA Academy content production ownership in M0.** Add an M0 exit criterion: "SGA Learning & Development team (or contracted vendor) has committed to delivering 10 CCA Bronze training video modules by Week 8." The WS3 engineering team builds the delivery platform; content production is not their responsibility.

8. **Add Asset Registry resilience for cross-workstream actions.** The two moderate-risk dependencies (Storefront Manager "Use on Website" / "Push to GMB" and QC Queue "Swap to Fix") both require live Asset Registry API calls. Add client-side retry logic with offline queuing for these actions, consistent with the PWA offline patterns already specified. If the Asset Registry is temporarily unavailable, queue the action and execute on recovery.

9. **Document the QC queue split as an intentional architectural decision.** The D-07 deduplication claims "one QC system" but the consolidated architecture implements two separate QC surfaces (WS1 storefront QC in Network Intelligence, WS2 editorial QC in Content Engine). If this is intentional (different reviewers, different expertise), document it as an ADR. If it is unintentional, evaluate whether a single unified QC queue with workstream-context filters would serve both use cases.

10. **Track the 12 scope additions against the original sprint commitments.** Present the scope creep assessment to stakeholders explicitly. The 8 justified additions (compliance, WCAG, carry-forwards) should be formally accepted. The 3 partially justified additions should be reviewed for scope reduction. This prevents downstream disputes about "why is this taking longer than 6 sprints estimated."

---

## Appendix: Event Contract Dependency Matrix

For reference, the 10 cross-workstream event flows and their verification schedule:

| # | Event | Producer | Consumer(s) | First Live | Verified At | Dependency Direction |
|---|---|---|---|---|---|---|
| 1 | `ws3.champion-assigned` | WS3 | WS1 Exec Dash, WS2 Capture Bot | Week 9 | M2 | WS3 --> WS1, WS2 |
| 2 | `ws3.hscore-calculated` | WS3 | WS1 Exec Dash, WS1 Practice Ops | Week 7 | M1.5 | WS3 --> WS1 |
| 3 | `ws3.benchmark-taxonomy-updated` | WS3 | WS1, WS2 | Week 10 | M2 | WS3 --> WS1, WS2 |
| 4 | `ws3.champion-story-submitted` | WS3 | WS2 Content Pipeline | Week 14 | M3 | WS3 --> WS2 |
| 5 | `pms.appointment-completed` | Shared PMS | WS2 Capture Bot | Week 12 | M3 | Shared --> WS2 |
| 6 | `ws2.content-asset-approved` | WS2 | WS1 Storefront | Week 14 | M3 | WS2 --> WS1 |
| 7 | `ws1.funnel-event` | WS1 | WS2 Attribution | Week 15 | M4 | WS1 --> WS2 |
| 8 | `ws2.content-engagement-updated` | WS2 | WS1 Exec Dash | Week 15 | M4 | WS2 --> WS1 |
| 9 | `ws3.training-completed` | WS3 | WS1 Exec Dash | Week 12 | M3 | WS3 --> WS1 |
| 10 | `ws1.onboarding-status-changed` | WS1 | WS3 Culture Activation | Week 14 | M3 | WS1 --> WS3 |

**Observation**: WS3 is the heaviest event producer in early milestones (events 1, 2, 3 at M1.5/M2), which aligns with the WS3-early-start strategy. WS1 and WS2 become producers later (events 6, 7, 8, 10 at M3/M4). This sequencing is correct -- WS3's events validate the Kafka infrastructure before WS1 and WS2 come online as producers.

**Observation**: Event #10 (`ws1.onboarding-status-changed` --> WS3) is the only event where WS1 produces and WS3 consumes. This means WS3 can build and test independently of WS1 for its first 3 milestones (M1.5, M2, most of M3), only needing WS1's enrollment event for auto-provisioning culture onboarding in Week 14. This validates the WS3-early-start decision.

---

*This audit validates that the consolidated 3-workstream model is structurally sound, with healthy cross-workstream dependencies and genuine simplification over the original 6-sprint design. The 10 recommendations address specific gaps in shared service coverage, scope management, capacity planning, and compliance. None require a fundamental redesign -- all are refinements to an already-solid architecture.*
