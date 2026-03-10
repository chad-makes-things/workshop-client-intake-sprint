# Development Handoff: The Comms Engine

**Sprint**: Patient Communications Automation
**Solution**: The Comms Engine — Autonomous AI Agent Patient Communications Playbook
**Date**: 2026-03-10

---

## Handoff Summary

This document maps sprint artifacts to development team roles. Each role receives specific sprint outputs as their starting context — they build FROM the sprint, not from scratch.

## Artifact → Role Mapping

### Product Manager
**Receives**: Sprint framing (goal, questions, target) + chosen solution direction + storyboard
**Starting artifacts**:
- `01-frame/goal.md` — Goal statement, 5 sprint questions
- `01-frame/target.md` — Target selection: Full Lifecycle Playbook (all 4 stages)
- `03-decide/decision.md` — Solution evaluation matrix (5 solutions scored) and selection reasoning
- `03-decide/storyboard.md` — 10-step user journey (Maria's lead, James's treatment follow-up, Linda's $22K close, Jennifer's Monday summary, CEO dashboard)
**Creates**: Epic with user stories, acceptance criteria (Given/When/Then), technical spikes, dependency map

### UX Designer
**Receives**: Storyboard (user journey) + HTML prototype + solution narrative
**Starting artifacts**:
- `03-decide/storyboard.md` — 10-step user journey: supervised-mode daily digest (Step 9), weekly summary format (Step 7), office manager's 20-minute touchpoint, CEO network view (Step 10)
- `04-architect/index.html` — Visual prototype showing 5-layer architecture, 6 components, and playbook stages
- `02-sketch/solutions.md` — Solution 5 narrative (The Comms Engine description)
- `04-architect/architecture.md` — Layer 5 (Human Interface): Weekly Summary Generator, Approval Interface, HQ Playbook Manager
**Creates**: Screen wireframes for practice approval interface (mobile PWA) and HQ Playbook Manager (desktop web), interaction patterns, design system (extending Practice Concierge), accessibility spec

### Tech Lead
**Receives**: Architecture document + ADRs + technology recommendations
**Starting artifacts**:
- `04-architect/architecture.md` — 5-layer architecture, 6 components, data flow diagrams, 5 ADRs, tech stack, deployment roadmap
- `01-frame/map.md` — Full problem context, 4 patient journeys (lead, treatment, rebooking, reactivation), 10 failure points
- `02-sketch/inspiration.md` — Technology landscape (TrueLark, Weave, Arini, NexHealth, Twilio patterns)
- `05-validate/learnings.md` — 5 unresolved questions and risks
**Creates**: Implementation plan with stack validation, database schema, API endpoints, engineering tasks, infrastructure plan, risk register

### Backend Engineer
**Receives**: Architecture + data model + API surface from Tech Lead's plan
**Starting artifacts**:
- `04-architect/architecture.md` — Layer 1 (Data Integration), Layer 2 (Comms Engine Core), Layer 3 (Communication Transport), component interfaces, NexHealth + Twilio integration patterns
- Tech Lead's implementation plan (created in Phase 11)
**Creates**: Phase 1 backend spec — API endpoint details, ORM schema, service layer design for Lead Response stage

### Frontend Engineer
**Receives**: UX wireframes + architecture + storyboard
**Starting artifacts**:
- `03-decide/storyboard.md` — Step 7 (weekly summary format), Step 9 (supervised mode daily digest UX), Step 10 (CEO dashboard comms section)
- `04-architect/architecture.md` — Layer 5 (Human Interface): Approval Interface and HQ Playbook Manager specifications
- UX Designer's design specs (created in Phase 11)
**Creates**: Phase 1 frontend spec — component tree, routes, state management, component specs for practice-facing PWA approval interface

---

## Development Sequence

1. **PM** creates epic and user stories (parallel with UX)
2. **UX Designer** creates wireframes and design system extending Practice Concierge (parallel with PM)
3. **Tech Lead** creates implementation plan (after PM + UX for requirements context)
4. **Backend Engineer** creates Phase 1 backend spec (after Tech Lead)
5. **Frontend Engineer** creates Phase 1 frontend spec (after Tech Lead + UX)

## Sprint Questions → Development Validation

| Sprint Question | How Development Validates |
|----------------|--------------------------|
| SQ1: Can we achieve <60-second lead response 24/7 including weekends and after-hours? | End-to-end latency test: inbound call/form webhook → first outreach timestamp; target <60s p95 in load test |
| SQ2: Will AI-generated messages pass the "sounds human" test with dental patients? | Blind review study: dental patients rate message samples without knowing AI vs. human origin; target >80% "sounds natural" |
| SQ3: Can the graduated autonomy model actually change practice staff behavior (adoption)? | 30-day pilot cohort: track edit rate, approval rate, graduation velocity; target <5% edit rate within 30 days at 80% of pilot practices |
| SQ4: Does NexHealth's API coverage actually support real-time PMS read/write across 80% of SGA's systems? | PMS audit: enumerate all PMS systems in SGA's 260+ practices; test NexHealth coverage; document gap practices requiring adapter |
| SQ5: Can the Network Learning Engine detect statistically significant message improvements with 30-day cohorts? | A/B test simulation: backtest learning engine against 6 months of historical data (where available); validate minimum sample size for significance |

## Connection to Prior Sprint Architectures

### Practice Concierge (Digital Storefront sprint)
The Comms Engine agent IS the Practice Concierge agent extended with communication skills. Development teams should:
- Use the same agent runtime and practice context store
- Add communication capabilities as new skills to the existing agent, not a new system
- Share the practice-facing interface (Concierge tabs + new Comms Engine tab = one app, modular)
- Coordinate NexHealth API usage — single client library, shared auth

### Funnel OS (Full Funnel Visibility sprint)
The Comms Engine feeds all communication events and outcomes to the Funnel OS data layer. Development teams should:
- Write all communication events using the Funnel OS unified event schema (`lead_generated`, `lead_connected`, `lead_booked`, `patient_showed`, `production_completed`)
- The Practice Nerve Center weekly summary from Funnel OS = the same interface as the Comms Engine weekly summary — coordinate to avoid two separate weekly digest systems
- Share the Network Learning Engine infrastructure — both systems need outcome attribution
- Coordinate API design to avoid conflicting event schemas

### Convergence Architecture
```
Practice Concierge Agent (shared runtime)
    ├── Skill: Digital Storefront Management (Sprint 2)
    ├── Skill: Patient Communications Playbook (This Sprint)
    └── [Future: Staff Coaching, 5-Star Culture, etc.]

Funnel OS (data infrastructure)
    ├── Unified Event Schema ← Comms Engine writes here
    ├── Executive Dashboard ← includes Comms Performance section
    ├── Practice Nerve Center = Comms Engine weekly summary (merged)
    └── Network Learning Engine ← shared by both systems
```
