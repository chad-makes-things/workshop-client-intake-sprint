# Development Handoff: The Hospitality Playbook Platform

**Sprint**: Hospitality Playbook Operationalization
**Solution**: The Hospitality Playbook Platform
**Handoff Date**: 2026-03-10

---

## What This Document Is

This handoff maps every sprint artifact to the development role responsible for acting on it. Each role receives a curated subset of the sprint output — the materials relevant to their specific planning work. This is not a summary; it is a routing document.

---

## For the Product Manager

**Receives**: Goal and questions + all 5 solution component descriptions + decision record + storyboard

**Sprint artifacts**:
- `01-frame/goal-and-questions.md` — 5-year goal, 2-year goal, and 5 sprint questions with risk levels. These map directly to OKRs and epic success criteria.
- `02-sketch/solution-sketches.md` — The 5 solution components with narratives, strengths, risks, and feasibility ratings. Use these for epic scope definition.
- `03-decide/decision.md` — Decision context: why all 5 are unified, verdict for each, the ranking logic.
- `03-decide/storyboard.md` — 9 user journey scenes across 3 personas (Maya, Dr. Vasquez, Jordan). Use these for user story generation and acceptance criteria grounding.
- `04-architect/product-architecture.md` — The 10-component breakdown, data model, and build sequence. Use for dependency mapping and release planning.
- `05-validate/learnings.md` — 3-phase build sequence with Phase 1 success metrics. Use as the MVP definition.

**Primary PM outputs expected**:
- Epic structure (1 parent epic + 5 sub-epics, one per component)
- User stories in Given/When/Then format for Phase 1 scope
- Phase 1 acceptance criteria
- Dependency map across components
- Success metrics dashboard definition

**Reference file**: `06-development/requirements/epic.md`

---

## For the UX Designer

**Receives**: Storyboard + HTML prototype + architecture component responsibilities

**Sprint artifacts**:
- `03-decide/storyboard.md` — 9 scenes with explicit "system response" descriptions and "critical detail" callouts. These are the emotional and functional design requirements in narrative form.
- `04-architect/index.html` — The sprint prototype. Open in any browser. Sections to reference: Daily Ritual structure, Certification Path visualization, H-Score Dashboard mockup, Storyboard cards. This is a design direction reference, not a production spec.
- `04-architect/product-architecture.md` — Component responsibilities section (Components 1-10). Each responsibility describes what the interface must communicate to the user.
- `01-frame/expert-notes.md` — HMW notes contain implicit UX requirements (e.g., "HMW-02: modules under 5 minutes," "HMW-07: feel spontaneous and local even when pre-recorded"). Review these for design constraints.

**Key design requirements from the sprint**:
- The platform must feel like a **hospitality brand**, not an LMS or HR system
- Mobile-first: morning huddle is run on a tablet in a break room; certification modules are completed on phones between patients
- The morning huddle player is the primary UX surface — it must be zero-friction (one tap to start, no navigation required)
- The H-Score must be transparent and legible — every component visible, every movement explainable
- The CCA Bronze certification completion screen is a milestone moment — it should feel like an achievement, not a form confirmation
- The champion designation flow must feel like an honor being conferred, not an administrative action

**Primary UX outputs expected**:
- Design system (colors, typography, component library)
- 8 key screen wireframes (listed in `06-development/design/design-specs.md`)
- Mobile-first layouts for huddle player and certification module viewer
- Animation and interaction specifications
- Accessibility audit plan

**Reference file**: `06-development/design/design-specs.md`

---

## For the Tech Lead

**Receives**: Full product architecture + integration architecture + data model + ADRs + build sequence

**Sprint artifacts**:
- `04-architect/product-architecture.md` — The complete technical specification: 10 components, responsibilities, interfaces, data produced/consumed, key design decisions per component, integration architecture for Cinnamon/Swell/Dental Intel/Voice/LinkedIn, 5 ADRs, and the full data model.
- `05-validate/learnings.md` — Risk register (6 risks with mitigations), 3-phase build sequence with Phase 1 success metrics.
- `03-decide/decision.md` — ADR rationale for why the Huddle OS is the primary container (not a standalone app), why H-Score is transparent, and why the credential is external from day one.

**Critical ADRs for technical planning**:
1. **ADR-01**: Huddle OS lives in Dental Intel, not a new app. Tech Lead must evaluate Dental Intel's API capabilities for content injection and completion event webhooks.
2. **ADR-02**: H-Score algorithm is fully public. The scoring service must expose its formula via a readable API endpoint so the practice dashboard can explain every score movement.
3. **ADR-03**: Practice-type segmentation is required in every scoring calculation. Practice type must be a required field from day 1.
4. **ADR-04**: CCA credential uses Credly or Accredible for external verification — not a custom-built portal. Tech Lead must evaluate Credly API for certificate issuance, LinkedIn publishing, and verification URL generation.
5. **ADR-05**: Playbook is versioned like software. Content must be tagged to playbook version; updates must not break existing certification records.

**Primary Tech Lead outputs expected**:
- Stack recommendation with rationale
- Phase 1 engineering task breakdown (story points, assignable)
- Integration feasibility assessment (Dental Intel API, Cinnamon webhook, Swell webhook)
- Infrastructure plan (hosting, video delivery CDN, real-time scoring service)
- Risk register with engineering mitigations
- Definition of Done for each Phase 1 component

**Reference file**: `06-development/technical/implementation-plan.md`

---

## Phase 1 Scope (MVP — What to Build First)

All development roles should scope Phase 1 around these 5 components only:

| Component | Phase 1 Scope |
|-----------|--------------|
| Morning Huddle OS | Content package delivery via Dental Intel + completion tracking + live Swell review surface in huddle |
| H-Score Engine | Cinnamon + Swell + Dental Intel ingestion + daily score calculation + practice dashboard with score breakdown |
| CCA Academy (Bronze only) | 10-module track + scenario assessment + Bronze certificate issuance via Credly + LinkedIn integration |
| Trust Transfer Module | As part of CCA Bronze (Modules 05 + 06) + daily huddle checklist prompt + review NLP tagging |
| Practice Dashboard | H-Score display + huddle completion tracker + staff certification rates + "This Week's Opportunity" recommendation |

Phase 2 adds: Rewards Engine, Champion Network Layer, Network Intelligence Dashboard, CCA Silver track
Phase 3 adds: CCA Gold, academic export, dental school outreach package, Content Concierge integration

---

## Development Package Index

| File | Role | Description |
|------|------|-------------|
| `06-development/requirements/epic.md` | Product Manager | Full epic with sub-epics, user stories, acceptance criteria, success metrics |
| `06-development/design/design-specs.md` | UX Designer | Design system, 8 screen wireframe descriptions, component library |
| `06-development/technical/implementation-plan.md` | Tech Lead | Stack, build sequence, schema, API endpoints, integration specs |
| `06-development/backend/phase1-backend-spec.md` | Backend Engineer | Phase 1 API specs, H-Score algorithm, integration details, job architecture |
| `06-development/frontend/phase1-frontend-spec.md` | Frontend Engineer | Phase 1 component tree, routes, state management, mobile requirements |
