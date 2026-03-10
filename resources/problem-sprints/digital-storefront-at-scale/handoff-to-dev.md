# Development Handoff: The Practice Concierge

**Sprint**: Digital Storefront at Scale
**Solution**: The Practice Concierge — Agent-First Orchestration Platform
**Date**: 2026-03-10

---

## Handoff Summary

This document maps sprint artifacts to development team roles. Each role receives specific sprint outputs as their starting context — they build FROM the sprint, not from scratch.

## Artifact → Role Mapping

### Product Manager
**Receives**: Sprint framing (goal, questions, target) + chosen solution direction + storyboard
**Starting artifacts**:
- `01-frame/goal-and-questions.md` — Goal statement and 5 sprint questions to validate
- `01-frame/target.md` — Target selection with scoring rationale
- `03-decide/decision.md` — Solution evaluation matrix and selection reasoning
- `03-decide/storyboard.md` — 10-step user journey (both practice and HQ perspectives)
**Creates**: Epic with user stories, acceptance criteria (Given/When/Then), technical spikes, dependency map

### UX Designer
**Receives**: Storyboard (user journey) + HTML prototype + solution narrative
**Starting artifacts**:
- `03-decide/storyboard.md` — 10-step user journey with scenes, actions, system responses, user reactions
- `04-architect/index.html` — Visual prototype showing architecture and journey
- `02-sketch/solution-sketches.md` — Solution 1 narrative (Practice Concierge description)
**Creates**: Screen wireframes, interaction patterns, design system (colors, typography, spacing, components), accessibility spec

### Tech Lead
**Receives**: Architecture document + ADRs + technology recommendations
**Starting artifacts**:
- `04-architect/product-architecture.md` — 6-component architecture, data model, 5 ADRs, tech stack, 4-phase roadmap
- `01-frame/problem-map.md` — Full problem context and critical moments
- `05-validate/learnings.md` — Unresolved questions and risks
**Creates**: Implementation plan with stack validation, database schema, API endpoints, engineering tasks, infrastructure plan, risk register

### Backend Engineer
**Receives**: Architecture + data model + API surface from Tech Lead's plan
**Starting artifacts**:
- `04-architect/product-architecture.md` — Practice data model, component interfaces, technology recommendations
- Tech Lead's implementation plan (created in Phase 11)
**Creates**: Phase 1 backend spec — API endpoint details, ORM schema, service layer design

### Frontend Engineer
**Receives**: UX wireframes + architecture + storyboard
**Starting artifacts**:
- `03-decide/storyboard.md` — User journey steps that map to screens/views
- `04-architect/product-architecture.md` — Three interfaces (practice, HQ, executive)
- UX Designer's design specs (created in Phase 11)
**Creates**: Phase 1 frontend spec — component tree, routes, state management, component specs

---

## Development Sequence

1. **PM** creates epic and user stories (parallel with UX)
2. **UX Designer** creates wireframes and design system (parallel with PM)
3. **Tech Lead** creates implementation plan (after PM + UX for requirements context)
4. **Backend Engineer** creates Phase 1 backend spec (after Tech Lead)
5. **Frontend Engineer** creates Phase 1 frontend spec (after Tech Lead + UX)

## Sprint Questions → Development Validation

| Sprint Question | How Development Validates |
|----------------|--------------------------|
| SQ1: Setup < 2 weeks | End-to-end enrollment flow timing test with pilot batch |
| SQ2: Quality at scale | Template QA coverage, automated brand compliance checks |
| SQ3: Practice engagement | Concierge interaction analytics, adoption rate tracking |
| SQ4: Proactive change detection | Event propagation latency tests, iSolved webhook integration |
| SQ5: Executive visibility | Dashboard load tests, automated metric accuracy validation |
