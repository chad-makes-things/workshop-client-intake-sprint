# Sprint: Digital Storefront at Scale

**Date**: 2026-03-10
**Slug**: `digital-storefront-at-scale`
**Status**: Complete — Awaiting Approval
**Solution**: The Practice Concierge — Agent-First Orchestration Platform

---

## Problem Statement

SGA Dental Partners has 260+ practices (growing to 300+) across multiple platforms (SGA, Gen4, MODIS). Every new acquisition requires standing up a complete digital storefront — best-in-class website, Google My Business profile, lead generation funnel, social handles, and CRM integration. This currently takes **months per practice** and just getting access to existing assets, passwords, and brand materials can take days alone.

Beyond the initial setup, there is no system for **365-day-a-year curation** across the entire network. The HQ marketing team is a handful of people responsible for 260+ locations at wildly different stages of digital maturity. There is no executive visibility into the digital health of the network and no proactive change management when practice-level events happen (staff departures, service changes, etc.).

## Solution: The Practice Concierge

Every SGA practice gets an AI concierge — a conversational agent that is their single point of contact with SGA marketing. The concierge handles enrollment (both new acquisitions and existing practices), ongoing change management, content approvals, and Q&A. Behind the concierge, an orchestration layer routes actions to website templates, GMB, social publishing, CRM, and the executive dashboard.

**Key stats**: Setup compressed from months to <2 weeks. 365-day curation automated. Same ~10-person HQ team manages 300+ practices.

## Context

- **Organization**: SGA Dental Partners (PE-backed DSO, Thurston Group)
- **Scale**: 260+ practices, acquiring 20-50 at a time, heading to 300+
- **Platforms**: SGA (Southeast community), Gen4 (national GP/specialty), MODIS (implantology)
- **HQ Team**: ~10 people total across marketing
- **Target**: A+C Hybrid — Rapid Onboarding Pipeline + Practice-Facing Agent (Stage 2: 365-Day Curation)

---

## Sprint Output

### 01-Frame (Monday)
| File | Description |
|------|-------------|
| [goal-and-questions.md](01-frame/goal-and-questions.md) | 30-day goal + 5 sprint questions with risk levels |
| [problem-map.md](01-frame/problem-map.md) | 5 actors, 10 journey steps, critical moments analysis |
| [expert-notes.md](01-frame/expert-notes.md) | 25 HMW notes across 6 themes + 5 expert insights |
| [target.md](01-frame/target.md) | 3 targets scored — A+C Hybrid selected (23/25) |

### 02-Sketch (Tuesday)
| File | Description |
|------|-------------|
| [inspiration.md](02-sketch/inspiration.md) | 8 research entries: SOCi, franchise models, AI receptionists, Copilot Studio, dental platforms |
| [solution-sketches.md](02-sketch/solution-sketches.md) | 5 distinct solutions with narratives, strengths, risks, feasibility |

### 03-Decide (Wednesday)
| File | Description |
|------|-------------|
| [decision.md](03-decide/decision.md) | Full evaluation matrix — Practice Concierge selected (30/35) |
| [storyboard.md](03-decide/storyboard.md) | 10-step user journey: discovery → enrollment → go-live → change management + HQ experience |

### 04-Architect (Thursday)
| File | Description |
|------|-------------|
| [product-architecture.md](04-architect/product-architecture.md) | 6 components, data model, 5 ADRs, tech stack, 4-phase roadmap |
| [index.html](04-architect/index.html) | Interactive HTML visualization — open in browser |

### 05-Validate (Friday)
| File | Description |
|------|-------------|
| [learnings.md](05-validate/learnings.md) | 5 key learnings, unresolved questions, risks, next steps |

---

## Decisions Made

1. **Target**: A+C Hybrid — Network-wide digital modernization pipeline with practice-facing agent. Existing practices enrolled alongside new acquisitions.
2. **Solution**: The Practice Concierge (30/35) — agent-first orchestration with blitz-deploy safety net and guardrailed decision patterns.
3. **Architecture**: 6-component system — Practice Data Hub, Data Harvester, Concierge Agent, Template Engine, Channel Sync, Network Dashboard.
4. **Implementation**: 4-phase roadmap — MVP (30d), Scale (60d), Curation (90d), Intelligence (120d).

## Next Steps

- [ ] Review HTML visualization (open `04-architect/index.html` in browser)
- [ ] Approve sprint for GitHub push (Phase 9)
- [ ] Optional: Development handoff (Phase 10) → full spec package (Phase 11)
- [ ] Optional: Enhanced website deployment to GitHub Pages (Phase 12)
