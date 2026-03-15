# QC-2 Quality Gate Review: Product Design Director Assessment

## SGA Dental Partners — Experience Architecture Review

**Reviewer:** Product Design Director
**Date:** 2026-03-14
**Scope:** Experience Architecture (42 interfaces, 5 surfaces, 16 cross-workstream touchpoints)
**Prior Gate:** QC-1 — REVISE (8 items issued, 4 assigned to Product Designer)
**Verdict:** PASS (with 3 advisory notes)

---

## 1. QC-1 Item Resolution Assessment

### Item #1 (HIGH): WS1 Scope Overload — RESOLVED

**Original concern:** 32 features serving 3 distinct user archetypes in one workstream. Tabs as a design smell.

**What was done:** WS1 split into Practice Operations (18 features, practice staff) and Network Intelligence (12 features, HQ/leadership) with 5 spanning features. Shared data layer with RBAC at the API layer.

**Assessment:** This is the correct structural fix. The split follows the natural fault line I identified: practice staff execute, HQ staff analyze. The two surfaces share a data layer but have entirely different navigation structures — Practice Operations organized by time (Today, Patients, Calls, Automations, Storefront), Network Intelligence organized by scope (Network, Regions, Practices, Operations, Analytics). This is not a cosmetic split. The information architecture reflects genuinely different mental models. The 5 spanning features (Practice Data Hub, funnel model, Automation Engine, learning loop, Playbook Engine) are well-justified — each feature legitimately serves both audiences in different modes (practice staff consume, HQ staff configure).

The tab smell is gone. These are two distinct applications sharing infrastructure.

**Status: ACCEPTED.**

---

### Item #2 (HIGH): Storefront/Content Boundary — RESOLVED

**Original concern:** Who owns the moment of publication to a storefront? WS1 manages listings, WS2 produces content — turf war risk during development.

**What was done:** Asset Registry contract defined in Section 7. WS2 writes approved assets. WS1 reads and triggers storefront publication. Social distribution stays with WS2. Kafka event notification on asset approval.

**Assessment:** The contract is explicit and the ownership split is clean. The critical distinction — social posts (ephemeral, WS2 owns) vs. storefront content (persistent, WS1 owns) — is the right line to draw. The lifecycle diagram in Section 7 is clear enough to hand to an engineering team without ambiguity. The one-way write direction (WS2 writes, WS1 reads) eliminates the bidirectional confusion I was worried about.

The Asset Registry metadata schema (practice_id, brand_universe, asset_type, platform_variants, approval metadata) is sufficiently specified. Staleness tracking powering the Digital Health Score is a smart downstream use.

**Status: ACCEPTED.**

---

### Item #3 (MEDIUM): Twilio Channel Ownership — RESOLVED

**Original concern:** Both WS1 and WS2 use Twilio. Number management, opt-in/opt-out compliance, and message threading need a single owner.

**What was done:** Shared Twilio Gateway Service in the infrastructure layer. Separate phone numbers — one per-practice number for patient comms (WS1), one HQ-level number for staff capture briefs (WS2). Centralized opt-in/opt-out compliance. Inbound message classification and routing.

**Assessment:** This is well-architected. The separate-number strategy eliminates the threading collision problem entirely. Patients and staff will never see each other's messages. Centralizing compliance in the gateway is the only defensible approach — splitting TCPA/HIPAA compliance across workstream teams would have been a liability risk.

The routing rules are clear: inbound classification determines which workstream handler receives the message. This means a new message type in the future has a defined path (add a classification rule in the gateway, route to the appropriate handler).

**Status: ACCEPTED.**

---

### Item #6 (MEDIUM): Mobile Strategy — RESOLVED

**Original concern:** No coherent mobile strategy across workstreams.

**What was done:** Comprehensive mobile strategy in Section 6. SMS for brief async interactions (patient comms, capture briefs, champion nudges). Single PWA with 7 screens for sustained mobile workflows (Today, Capture, Huddle, Academy, Champion Hub, Playbooks, Calls). No native app in v1 with explicit triggers for when to reconsider.

**Assessment:** The three-question framework (native capability needed? brief async? sustained session?) is a sound decision model. The SMS-first philosophy for champions and patients is correct — asking a hygienist to open an app between patients is a 60-second tax on a 10-second interaction.

The PWA screen list is right-sized. 7 screens is manageable for a v1 mobile experience. The bottom navigation grouping by task rather than by workstream reinforces the "invisible workstream boundaries" design principle.

The explicit "when to reconsider native" triggers (RAW photo processing, Bluetooth hardware, iOS Web Push reliability) prevent the perennial "but shouldn't we have an app?" conversation from derailing development. These are concrete, measurable triggers.

One note: the PWA's offline capabilities (agenda, playbooks, scorecards cached; captures stored locally; videos require connection) are specified with appropriate granularity. This tells engineering exactly what to cache and what to defer.

**Status: ACCEPTED.**

---

## 2. Experience Coherence: Does the WS1 Split Feel Like One Product?

**Verdict: Yes — with one caveat.**

The two surfaces are designed for different people at different times on different devices. An office manager will never need to navigate to Network Intelligence. A VP Ops will rarely need Practice Operations (only during site visits, where the iPad-optimized Practice Deep Dive serves them within Network Intelligence). The shared data layer ensures that when an office manager updates a practice profile, the VP Ops sees the change in their next dashboard refresh. This is coherence through data, not through UI.

The design principle "users never click links between workstreams" is maintained. Cross-workstream data appears as embedded widgets (H-Score in Network Intelligence, Asset Gallery in Practice Operations). The one documented exception — admin settings linking to Culture OS Admin Console in a new tab — is correctly scoped to infrequent admin actions.

**The caveat:** The Playbook Engine spans both surfaces. Practice staff consume playbooks. HQ staff design and deploy playbooks. The experience architecture describes this correctly, but the moment an HQ staff member deploys a playbook update and wants to see how it renders in the practice view, they will need to context-switch. This is acceptable — playbook deployment is a weekly/monthly activity, not a daily workflow. But the Playbook Manager in Network Intelligence should include a "Preview as Practice Staff" mode. This is not a blocker; it is a UX detail for wireframing.

---

## 3. Cross-Workstream Seam Analysis

### The 16 Touchpoints: Natural or Forced?

I reviewed each of the 16 touchpoints against two criteria: (a) does the user have a natural reason to see this data at this moment, and (b) does the design treatment avoid requiring the user to understand the source system.

**Natural (13 of 16):**

- T-1 (H-Score in Morning Briefing): Office manager starts the day, sees practice health. Natural.
- T-2 (Capture brief as Action Card): Office manager sees "today's opportunities." Content capture is one of those. Natural.
- T-3 (Asset Gallery in Storefront Manager): Office manager managing their website sees available photos. Natural.
- T-4 (Impact Story in Huddle): Huddle is motivational. Funnel data as a story is motivational context. Natural.
- T-5 (Capture count in Scorecard): End-of-day productivity summary. Natural.
- T-6 (H-Score in Network KPI Bar): Executive KPIs include culture health. Natural.
- T-7 (H-Score as heatmap layer): Executive exploring portfolio health. Natural.
- T-8 (Culture Tab in Practice Deep Dive): Executive drilling into a practice wants the full picture. Natural.
- T-9 (Brand health in Practice Deep Dive): Same as T-8 for content metrics. Natural.
- T-11 (Champion roster in Content Engine): Content team managing champions needs to see who they are. Natural.
- T-12 (Appointment data driving capture briefs): Invisible to users — infrastructure. Natural.
- T-14 (Funnel data in Huddle): Same as T-4. Natural.
- T-15 (Content stats in Champion Hub): Champion sees their unified scorecard. Natural.

**Potentially Forced (2 of 16):**

- T-10 (Brand deviation flags in QC Queues): The QC Queue concept is well-designed, but "brand deviation" is a Content Engine judgment surfacing in Network Intelligence. The HQ ops person resolving this flag needs to understand what "brand deviation" means and how to fix it. The document says "the flag includes the violation detail and a direct link to the replacement asset in the Asset Registry." This is workable, but the "direct link to the Asset Registry" breaks the "no cross-linking" principle stated in the Seam Design Principles. This should be resolved: either embed a preview of the replacement asset directly in the QC Queue item (no link needed), or acknowledge this as a second permitted exception alongside the admin settings link.

- T-16 (Brand-compliant recognition templates in Culture OS): This is the thinnest touchpoint. Auto-applying brand formatting to recognition posts is a nice-to-have, not a pain point driver. It adds a dependency between Culture OS and Content Engine for a cosmetic feature. Not harmful, but it should be deferred to a later phase if it creates engineering complexity.

**Missing Touchpoint (1):**

There is no touchpoint for Network Intelligence receiving Content Engine pipeline health data (how many assets are in production, how many are stuck in review, what is the network-wide content velocity). An executive looking at a practice's Digital Presence Tab sees content capture frequency and brand health score, but not whether the Content Engine pipeline itself is backed up. If content is stuck in the Editorial Queue for 2 weeks, the practice's storefront stagnates and the executive has no visibility into why. This is not critical for v1 but should be on the v2 touchpoint roadmap.

---

## 4. Surface Area Assessment: Are 42 Interfaces Appropriate for v1?

**Verdict: The count is appropriate. The sequencing matters more than the count.**

42 interfaces across 5 surfaces sounds high, but the breakdown reveals that several are lightweight:

- 6 patient-facing interfaces (PF-1 through PF-6) are largely templated outputs, not interactive applications. A practice website, an SMS conversation, and a digital form are well-understood patterns.
- The PWA's 7 screens are mobile adaptations of web interfaces already counted in the surface tallies. They are not 7 additional interfaces — they are responsive views.
- Several Network Intelligence interfaces (Compliance Dashboard, Network Learning Report) are read-only dashboards that can ship with minimal interactivity and be enhanced over time.

The real question is not "is 42 too many" but "in what order do we build them." The user journeys in Section 3 answer this implicitly: the Office Manager's daily workflow (Journey 4) touches PO-1, PO-2, PO-6, and the Capture App. That is 4 interfaces. Build those first for practice-level value. The CEO's 15-minute Monday morning (Journey 1) touches NI-1, NI-5, and NI-6. Build those for executive value. The Content Engine pipeline (CE-3, CE-6) and Culture OS huddle (CO-1, CO-5) round out the core.

A viable v1 could ship with approximately 18-22 of the 42 interfaces and cover the highest-frequency, highest-intensity pain points. The remaining interfaces are enhancements, not foundations.

**Advisory:** The roadmap/scope phase that follows this gate should define a "launch surface" (the 18-22 interfaces that constitute a usable product) and a "full surface" (all 42). Do not attempt to ship all 42 simultaneously.

---

## 5. Accessibility Assessment (WCAG 2.1 AA)

### Strengths

- The PWA's 7-screen constraint naturally limits cognitive load on mobile.
- The Network Intelligence heatmap specifying "color-coded health" with a toggle for different data layers is a good interaction pattern, provided color is not the sole indicator.
- The Morning Briefing's "3 prioritized action cards" pattern is a strong progressive disclosure model.

### Concerns

1. **Heatmap color reliance (NI-2, NI-7):** The Portfolio Heatmap uses color coding (green/yellow/red) as the primary health indicator. WCAG 1.4.1 requires that color is not the sole means of conveying information. The architecture should specify that each practice marker also displays a text label or icon indicator (checkmark, warning, alert) so that color-blind users can distinguish practice health bands. This is a wireframing-level fix, not an architecture-level problem.

2. **SMS interaction accessibility:** SMS-based workflows (capture briefs, patient comms) have limited accessibility control — no font sizing, no high contrast mode, no screen reader optimization beyond the device's native capabilities. This is inherent to SMS as a channel and not fixable at the architecture level. However, the SMS content should follow plain language guidelines (short sentences, clear calls to action, no complex formatting) and the system should support voice call fallback for patients who cannot use SMS.

3. **PWA offline indicators:** When the PWA is offline, cached content should display a clear, non-color-only indicator (e.g., a banner with text "You're offline — showing cached data") per WCAG 2.1 AA status message guidance.

4. **Voice AI (PF-2):** The Voice AI phone interface is inherently accessible to users who cannot use visual interfaces. However, the architecture should specify that the Voice AI supports TTY/TDD relay services for deaf/hard-of-hearing callers, which is a legal requirement for healthcare-adjacent phone services under ADA.

**None of these are architectural blockers.** Items 1, 3, and 4 are wireframing/implementation details. Item 2 is a channel limitation. All should be documented as requirements for the design specs phase.

---

## 6. Verdict: PASS

The experience architecture is ready to proceed to roadmap and scope definition.

### Resolution Summary

| QC-1 Item | Original Severity | QC-2 Status |
|-----------|-------------------|-------------|
| #1 — WS1 scope overload | HIGH | RESOLVED — Clean split with correct structural rationale |
| #2 — Storefront/content boundary | HIGH | RESOLVED — Explicit contract with clear ownership |
| #3 — Twilio ownership | MEDIUM | RESOLVED — Shared gateway with separate numbers |
| #6 — Mobile strategy | MEDIUM | RESOLVED — SMS + PWA + no native v1, well-reasoned |

### Advisory Notes (Non-Blocking)

These do not prevent the architecture from proceeding. They should be addressed during wireframing or roadmap definition.

| # | Advisory | Phase to Address |
|---|----------|-----------------|
| A-1 | T-10 (brand deviation QC flag) uses a cross-link to Asset Registry, contradicting the "no cross-linking" seam principle. Either embed the replacement asset preview inline or document this as a second permitted exception. | Wireframing |
| A-2 | T-16 (brand-compliant recognition templates) is a thin, cosmetic touchpoint that adds a WS2-WS3 dependency. Consider deferring to post-v1 if it creates engineering overhead. | Roadmap/Scope |
| A-3 | Playbook Manager in Network Intelligence should include a "Preview as Practice Staff" mode so HQ staff can verify how playbooks render in the practice view. | Wireframing |

### What Stands Out as Excellent

1. **The user journeys are the best part of this document.** Seven concrete journeys with named personas, specific times, and realistic workflows. These are wireframe-ready. Any designer picking up Journey 4 (Office Manager daily workflow) can start sketching screens immediately. This is what an experience architecture should look like.

2. **The seam design principles are mature.** "Data flows via Kafka; UI never links between workstreams" is a principle that will prevent years of cross-team integration debt. The stale data indicators and graceful degradation specifications show systems thinking, not just screen thinking.

3. **The mobile strategy decision framework is reusable.** The three-question model (native capability? brief async? sustained session?) is not SGA-specific — it is a generalizable framework. The explicit "when to reconsider native" triggers prevent scope creep without closing the door permanently.

4. **The Asset Registry contract is production-grade.** It specifies ownership, write direction, lifecycle, metadata schema, event notification, and staleness tracking. This is not a hand-wave — it is an implementable interface contract.

5. **The RBAC model is correctly scoped.** Practice-scoped tokens vs. network-scoped tokens with role-based write restrictions is the right security posture for a multi-tenant healthcare platform. This was not explicitly called out in QC-1 but the Designer added it proactively.

### What the Next Phase Must Do

The roadmap/scope phase must:
- Define the "launch surface" (~18-22 interfaces) vs. "full surface" (42 interfaces)
- Sequence the build by user journey coverage, not by workstream
- Address QC-1 Items #4 (integration contracts), #5 (compliance architecture), #7 (build sequence), and #8 (Canva vendor risk), which were assigned to the Technical Architect, not the Product Designer
- Incorporate the 4 WCAG advisory notes into the design specification requirements

---

*QC-2 reviewed by Product Design Director | 2026-03-14*
*Prior gate: QC-1 REVISE (2026-03-14) | This gate: QC-2 PASS*
