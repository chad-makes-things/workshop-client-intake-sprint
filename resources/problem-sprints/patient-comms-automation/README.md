# Sprint: Patient Communications Automation

## Problem Statement
SGA Dental's 260+ practices have fragmented, manual patient communication across the entire patient lifecycle — from initial lead response through treatment follow-up to rebooking and database reactivation. No standardized playbook exists. No CRM captures the workflow. Staff capacity and skill gaps mean leads go unworked, treatment plans go unfollowed, and patients fall out of the funnel at every stage. The desired outcome is an automated agent system that handles the vast majority of patient communications autonomously, leaving only ~20 minutes/week of human touchpoints for things only humans can do.

## Sprint Status: Development-Ready

**Solution**: The Comms Engine — Autonomous AI agent per practice executing the full patient communications playbook: lead response in <60 seconds, treatment follow-up on 3/7/14/30-day cadences, same-day rebooking recovery, and monthly database reactivation. Graduated autonomy (Supervised → Monitored → Autonomous) builds staff trust over 30 days. Network Learning Engine optimizes the playbook monthly from 260-practice outcome data. $47/practice/month vs. $189–299 for vendor platforms.

## Key Context
- **Client**: SGA Dental Partners (PE-backed DSO, 260+ practices)
- **Sprint Date**: 2026-03-10
- **Related Sprints**:
  - [Digital Storefront at Scale](../digital-storefront-at-scale/) — Practice Concierge agent (Comms Engine extends this agent)
  - [Full Funnel Visibility](../full-funnel-visibility/) — Funnel OS data layer (Comms Engine feeds this)
- **Domain Knowledge**: `resources/sga-knowledge/` (151 files, full marketing/sales/strategy library)

## Key Numbers
- **< 60s** lead response vs. 47-hour industry average
- **50%+ more** treatment plan closings from automated follow-up
- **$47/mo** per practice vs. $189–299 vendor platforms ($440K–$785K annual savings at 260 practices)
- **20 min/week** human time per practice
- **$24M+** quarterly pipeline from reactivation (projected at scale)

---

## Sprint Artifacts

### 01 — Frame
- [Goal & Sprint Questions](01-frame/goal.md)
- [Problem Map](01-frame/map.md)
- [HMW Notes](01-frame/hmw-notes.md)
- [Target Recommendation](01-frame/target.md)

### 02 — Sketch
- [Research Inspiration](02-sketch/inspiration.md)
- [Solution Approaches](02-sketch/solutions.md)

### 03 — Decide
- [Decision Record](03-decide/decision.md)
- [Storyboard](03-decide/storyboard.md)

### 04 — Architect
- [Architecture Document](04-architect/architecture.md)
- [HTML Visualization](04-architect/index.html)

### 05 — Validate
- [Learnings](05-validate/learnings.md)

---

## Development Package

### Phase 10 — Development Handoff
- [Handoff Manifest](handoff-to-dev.md) — Sprint artifact → dev role mapping

### Phase 11 — Development Specifications
- [Epic & User Stories](06-development/requirements/epic.md) — 23 stories across 8 epics, 158 pts
- [UX Design Specs](06-development/design/design-specs.md) — Practice PWA + HQ Playbook Manager wireframes
- [Tech Lead Implementation Plan](06-development/technical/implementation-plan.md) — Stack validation, schema, API endpoints, infrastructure
- [Phase 1 Backend Spec](06-development/backend/phase1-backend-spec.md) — Lead Response services, webhooks, Voice AI
- [Phase 1 Frontend Spec](06-development/frontend/phase1-frontend-spec.md) — Supervised mode approval interface (Practice Concierge PWA module)

---

## Architecture Summary

```
Layer 1: Data Integration
  NexHealth Synchronizer → Event Stream → Practice PMS data (20+ systems)
  Phone System Integration → Twilio inbound/missed call events
  Web Form Webhooks → Form fill lead events
  AI Note-Taker Bridge → Treatment context enrichment (Stage 2)

Layer 2: Comms Engine Core
  Playbook Engine → Rule-based cadence execution
  Agent Orchestrator → 260+ practice agent instances
  Message Generator → Claude LLM personalization
  Channel Router → SMS / Voice / Email selection
  Escalation Manager → Weekly summary escalations

Layer 3: Communication Transport
  Twilio (HIPAA BAA) → SMS, MMS, Voice, Email
  Voice AI Engine → Claude-powered conversational voice

Layer 4: Monitoring & Learning
  Outcome Tracker → Revenue attribution per message
  Network Learning Engine → Monthly playbook optimization
  Compliance Auditor → HIPAA audit trail
  Graduation Manager → Supervised → Monitored → Autonomous advancement

Layer 5: Human Interface (20 min/week)
  Weekly Summary → Monday AM SMS to office manager
  Supervised Approval Digest → Daily message review (PWA)
  HQ Playbook Manager → Cadence design, templates, A/B tests (Stage 5)
```

## Deployment Roadmap

| Stage | Timeline | Scope | Practices |
|-------|----------|-------|-----------|
| Stage 1 | Month 1–2 | Lead Response (voice AI, missed call, form fill) | 5 pilot |
| Stage 2 | Month 2–3 | Treatment Follow-Up (4/5-touch cadences) | 20 |
| Stage 3 | Month 3–4 | Rebooking (same-day recovery, recall) | 50 |
| Stage 4 | Month 4–6 | Reactivation + Network Learning Engine | 100+ |
| Stage 5 | Month 6–12 | Full network + HQ Playbook Manager | 260+ |

## Tech Stack
| Component | Technology |
|-----------|-----------|
| PMS Integration | NexHealth Synchronizer API |
| SMS / Voice / Email | Twilio (HIPAA BAA) |
| LLM | Claude API (claude-sonnet-4-6 / claude-opus-4-6) |
| Workflow Orchestration | Temporal |
| API Layer | FastAPI (Python 3.12) |
| Database | PostgreSQL 16 (AWS RDS) |
| Event Streaming | Apache Kafka (AWS MSK) |
| Frontend | React 18 + PWA (extends Practice Concierge) |
| Infrastructure | AWS (ECS Fargate, HIPAA-eligible) |
