# QC-1 Quality Gate Review: Product Design Director Assessment

## SGA Dental Partners — Workstream Consolidation Review

**Reviewer:** Product Design Director
**Date:** 2026-03-14
**Scope:** 6 problem sprints consolidated into 3 workstreams + shared infrastructure
**Verdict:** REVISE

---

## 1. Coherence Assessment

### Workstream 1: The Practice Platform — CONCERN

The Practice Platform is doing the most heavy lifting and this is where I have the strongest reservations. It combines three originally separate sprints — Practice Concierge, Funnel OS, and Comms Engine — into a single workstream with 32 features. Let me be direct: this workstream has a coherence problem.

The unifying concept is "one AI agent per practice," which is elegant in theory. But the actual scope spans:

- Patient onboarding workflows (operational)
- Storefront/listing management (marketing-adjacent)
- Patient communications / messaging (engagement)
- Funnel visibility and conversion analytics (business intelligence)
- Network-level executive dashboarding (corporate reporting)

These are held together by the thesis that a single agent should manage everything practice-level. That thesis may be correct architecturally, but from a product perspective, the person configuring storefront listings is not the same person reviewing funnel conversion metrics, and neither of them is the executive drilling into network dashboards. **Three distinct user archetypes are being served by one workstream, and that will create competing priorities during development.**

The "Nerve Center with modular tabs" approach acknowledges this implicitly — tabs are how you split things that don't fully belong together. That's a design smell, not a design solution.

### Workstream 2: The Content Engine — SOLID

This is the cleanest consolidation of the three. Studio Automation (create assets) + Content Concierge (capture and distribute content) share a natural lifecycle: capture → create → approve → distribute. The merge of Studio Agent + Caption Generator into shared Claude infrastructure is sensible. The unified brand asset library across 3 brand universes is the right call.

One minor tension: the inclusion of TV and print distribution alongside social and email. Physical media distribution has fundamentally different workflows, timelines, and vendor dependencies than digital channels. This isn't a dealbreaker, but it needs to be scoped carefully so the digital distribution engine doesn't get bogged down by print production timelines.

### Workstream 3: The Culture OS — SOLID

Standalone was the right call. Culture operationalization is a distinct domain with distinct users (practice managers, team leads, champions). The 13 features are focused. The 4 integration points to other workstreams are well-scoped and directional (data flows out to dashboard and content pipeline, doesn't try to pull in external concerns).

This workstream has the clearest singular purpose of the three.

---

## 2. Boundary Clarity

### Grey Areas Flagged

**Storefront management (WS1) vs. content distribution (WS2):** A practice's Google Business Profile listing is managed in WS1 (Practice Platform), but the photos and branded content displayed on that listing are produced in WS2 (Content Engine). Who owns the moment of publication to a storefront? This will cause a turf war during development unless the interface contract is explicit.

**Patient communications (WS1) vs. SMS-triggered content capture (WS2):** Both workstreams use Twilio. WS1 sends patient-facing messages. WS2 uses SMS to trigger photo/video capture from staff. These are different use cases but the same channel and potentially the same phone numbers. Number management, opt-in/opt-out compliance, and message threading need a single owner. Currently ambiguous.

**H-Score to executive dashboard (WS3 → WS1):** The H-Score is a Culture OS metric, but it surfaces in the Practice Platform dashboard. Who defines the visualization? Who decides what "good" looks like? If the Culture OS team changes the H-Score calculation, does that break the dashboard? This integration needs a clear contract owner.

**Champion Network (WS3) content pipeline to Content Engine (WS2):** Champions generate content ideas and stories. The Content Engine produces and distributes them. The handoff point is under-specified — is it a queue? A shared Kanban? An API call? This needs definition before build.

---

## 3. Strategic Alignment

| Goal | Alignment | Notes |
|------|-----------|-------|
| Scale to 260+ practices | STRONG | Single-agent-per-practice model scales linearly. Shared infrastructure avoids per-practice custom work. |
| Operationalize culture | STRONG | Culture OS as standalone prevents deprioritization behind revenue-generating features. |
| Reduce tool fragmentation | STRONG | Collapsing 3 PMS strategies to 1 is a major win. Shared Claude infrastructure is correct. |
| Reduce costs | MODERATE | Shared infrastructure reduces TCO long-term, but three parallel workstreams in development will be expensive upfront. 72 total features is a large surface area. |

---

## 4. Consolidation Risks

### WS1: Scope creep through the "unified agent" abstraction
Every new practice need will be rationalized as "just add it to the agent." Without a strict capability boundary, this workstream will absorb features from WS2 and WS3 over time.

### WS2: Canva API dependency
Entire creative pipeline depends on Canva API with rate limits, pricing, and deprecation risks outside SGA's control.

### WS3: Adoption, not technology
Most human-dependent workstream. Technology can be perfect and still fail if practice managers don't use it.

---

## 5. Missing Capabilities

- **Analytics and reporting depth** — Funnel OS conversion analytics may be compressed into a "tab"
- **Escalation and exception handling** — Edge case workflows easy to lose in consolidation
- **Staff-facing mobile experience** — No coherent mobile strategy across workstreams
- **Compliance and audit trail** — No explicit HIPAA compliance layer, audit logging, or consent management

---

## 6. Integration Coherence

4 of 10 integration points are explicitly defined (all WS3 outbound). 6 are implicit:
- Practice Platform patient data → Content Engine personalization
- Content Engine asset library → Practice Platform storefront
- Practice Platform funnel events → Content Engine performance feedback
- Content Engine social engagement → Practice Platform dashboard
- Culture OS training completion → Practice Platform readiness indicators
- Practice Platform onboarding status → Culture OS activation triggers

---

## 7. Build Sequence Assessment: PARTIALLY FEASIBLE

**Recommended adjustment:** Infrastructure phase should include defining all cross-workstream event contracts and API interfaces. Extends to Month 1-3. WS3 starts Month 2 (independent). WS1/WS2 start Month 3 with agreed contracts.

---

## Verdict: REVISE

### Required Revisions

| # | Issue | Severity | Recommended Fix |
|---|-------|----------|----------------|
| 1 | WS1 scope overload — 32 features serving 3 distinct user archetypes | HIGH | Sub-divide WS1 into two product surfaces with shared data layer, or define capability ceiling reducing v1 to ~18-20 features |
| 2 | Storefront/content boundary ambiguity between WS1 and WS2 | HIGH | Define explicit ownership: WS2 produces assets, WS1 publishes. Document interface contract |
| 3 | Twilio channel ownership split across WS1 and WS2 | MEDIUM | Assign Twilio to shared infrastructure layer. Both workstreams consume as service |
| 4 | 6 of 10 integration points are implicit | HIGH | Enumerate all 10 as named event contracts with owner, schema, direction |
| 5 | No compliance/audit architecture | HIGH | Add HIPAA compliance layer to shared infrastructure — cannot be retrofitted |
| 6 | No mobile strategy | MEDIUM | Define SMS vs. PWA vs. native approach for staff interactions |
| 7 | Build sequence needs adjustment | MEDIUM | Extend infrastructure phase for contract definition. Stagger WS1/WS2 behind WS3 |
| 8 | Canva single-vendor risk | LOW-MEDIUM | Add abstraction layer in WS2 architecture for design tool swappability |

### What's Working Well
- WS3 as standalone is correct
- NexHealth + Sikka ONE PMS strategy is pragmatic
- Kafka event bus is the right pattern
- Single Claude infrastructure avoids AI vendor sprawl
- PMF improvement from 16 to 22 is credible
- WS3 integration points are well-chosen
