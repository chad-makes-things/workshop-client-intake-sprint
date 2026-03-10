# Development Handoff: The Funnel OS

**Sprint**: Full Funnel Visibility
**Solution**: The Funnel OS — Full-Stack Unified Patient Funnel Operating System
**Date**: 2026-03-10

---

## Handoff Summary

This document maps sprint artifacts to development team roles. Each role receives specific sprint outputs as their starting context — they build FROM the sprint, not from scratch.

## Artifact → Role Mapping

### Product Manager
**Receives**: Sprint framing (goal, questions, target) + chosen solution direction + storyboard
**Starting artifacts**:
- `01-frame/goal-and-questions.md` — Goal statement, 5 sprint questions, 25 HMW notes
- `01-frame/target.md` — Target selection with scoring rationale (all 3 targets combined)
- `03-decide/decision.md` — Solution evaluation matrix (5 solutions scored) and selection reasoning
- `03-decide/storyboard.md` — 10-step user journey (CEO, HQ ops, practice staff perspectives)
**Creates**: Epic with user stories, acceptance criteria (Given/When/Then), technical spikes, dependency map

### UX Designer
**Receives**: Storyboard (user journey) + HTML prototype + solution narrative
**Starting artifacts**:
- `03-decide/storyboard.md` — 10-step user journey with scenes, actions, system responses, user reactions, critical details
- `04-architect/index.html` — Visual prototype showing architecture, funnel model, and journey
- `02-sketch/solution-sketches.md` — Solution 1 narrative (Funnel OS description)
- `04-architect/architecture.md` — Component 5 (Executive Dashboard) and Component 6 (Practice Nerve Center) specifications
**Creates**: Screen wireframes, interaction patterns, design system (colors, typography, spacing, components), accessibility spec

### Tech Lead
**Receives**: Architecture document + ADRs + technology recommendations
**Starting artifacts**:
- `04-architect/architecture.md` — 6-component architecture, data model, 5 ADRs, tech stack, staged delivery, team requirements
- `01-frame/problem-map.md` — Full problem context, 7 actors, 10 journey steps, critical moments
- `01-frame/expert-notes.md` — Technology landscape (15 platform categories), integration patterns
- `05-validate/learnings.md` — 5 unresolved questions and risks
**Creates**: Implementation plan with stack validation, database schema, API endpoints, engineering tasks, infrastructure plan, risk register

### Backend Engineer
**Receives**: Architecture + data model + API surface from Tech Lead's plan
**Starting artifacts**:
- `04-architect/architecture.md` — Funnel event schema, component interfaces, integration middleware design
- Tech Lead's implementation plan (created in Phase 11)
**Creates**: Phase 1 backend spec — API endpoint details, ORM schema, service layer design

### Frontend Engineer
**Receives**: UX wireframes + architecture + storyboard
**Starting artifacts**:
- `03-decide/storyboard.md` — User journey steps that map to screens/views
- `04-architect/architecture.md` — Executive Dashboard (Component 5) and Practice Nerve Center (Component 6) specifications
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
| SQ1: Can we integrate 15 platforms? | PMS connector test with 3 vendors, API coverage audit, data completeness metrics |
| SQ2: Can AI distinguish signal from noise? | Baseline calculation accuracy tests, false positive rate tracking, alert-to-action ratio |
| SQ3: Will practice staff use the Nerve Center? | 20-minute session timing tests, task completion rates, adoption metrics per pilot practice |
| SQ4: Real-time without workflow changes? | Integration passivity audit (no practice-side actions required), data latency measurements |
| SQ5: Can automation move the needle? | A/B test: practices with vs. without automation, show rate and production delta |

## Connection to Digital Storefront Sprint

The Practice Nerve Center (this sprint) extends the Practice Concierge (Storefront sprint). Development teams should:
- Use the same design system and component library
- Share the practice-facing interface (Concierge + Nerve Center = one app, modular tabs)
- Share the data infrastructure where applicable (practice profiles, configurations)
- Coordinate API design to avoid conflicting schemas
