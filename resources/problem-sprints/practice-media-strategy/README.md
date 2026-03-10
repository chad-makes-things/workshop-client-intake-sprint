# Sprint: Practice-Level + SGA Brand Media Strategy

**The Content Concierge**

| Field | Value |
|-------|-------|
| **Sprint Date** | 2026-03-10 |
| **Problem** | 260+ practices never capture content (before/afters, testimonials, video) despite having strategy and channels ready. Broken human behavior starves the entire content-to-patient pipeline. Same gap exists at SGA brand level for B2B content. |
| **Solution** | The Content Concierge — phased hybrid system: SMS prompts → mobile app → AI pipeline → passive hardware. Extends Practice Concierge ecosystem. |
| **Target** | The capture gap at implant/cosmetic practices (practice-level) and SGA leadership content (brand-level) |
| **Decision** | Solution 5: Hybrid phased approach — test behavior with SMS before investing in full platform |

---

## Sprint Artifacts

### 01-frame/
- [goal.md](01-frame/goal.md) — Sprint goal, why it matters, success criteria
- [map.md](01-frame/map.md) — Problem map showing 5 break points in the content pipeline
- [hmw.md](01-frame/hmw.md) — 30 How Might We questions across capture, behavior, production, distribution, scale
- [target.md](01-frame/target.md) — Sprint target: the content capture gap + SGA brand pillar

### 02-sketch/
- [research-inspiration.md](02-sketch/research-inspiration.md) — Market analysis, competitive gaps, design inspiration
- [solutions.md](02-sketch/solutions.md) — 5 solution approaches with comparison matrix

### 03-decide/
- [decision-record.md](03-decide/decision-record.md) — Why Solution 5 was selected
- [storyboard.md](03-decide/storyboard.md) — 3 detailed scenarios: practice capture, brand content, event mode

### 04-architect/
- [architecture.md](04-architect/architecture.md) — Full system architecture: 5 components, data model, tech stack, phased delivery, cost estimates
- [prototype.html](04-architect/prototype.html) — Interactive HTML prototype with 3 views: HQ Dashboard, Mobile App, Pipeline

### 05-validate/
- [learnings.md](05-validate/learnings.md) — Validation, research findings, risks, costs, next steps

### 06-development/
- [requirements/epic.md](06-development/requirements/epic.md) — Full product epic: business objectives, user stories, acceptance criteria
- [technical/implementation-plan.md](06-development/technical/implementation-plan.md) — Tech lead plan: stack decisions, 6-week sprint plan, monorepo structure, CI/CD
- [backend/phase1-backend-spec.md](06-development/backend/phase1-backend-spec.md) — Phase 1 backend: Scheduler, Capture, Admin services; DB schema; Twilio/Sikka/Airtable integration
- [frontend/phase1-frontend-spec.md](06-development/frontend/phase1-frontend-spec.md) — Phase 1 admin dashboard spec (Next.js) + SMS UX conversation design
- [design/design-specs.md](06-development/design/design-specs.md) — UX design spec: admin dashboard screens, SMS tone guidelines, Phase 2 mobile app direction

---

## Key Numbers

| Metric | Value |
|--------|-------|
| Practices in network | 260+ |
| High-value practices (primary target) | 65+ (25 implant/cosmetic + 40 specialty) |
| PMS coverage via Sikka ONE API | ~90% (400+ systems) |
| Content multiplication factor | 1 capture → 15-20 platform-specific pieces |
| Phase 1 cost (SMS pilot) | ~$15K setup + $5K/mo |
| Full platform cost | ~$183K build + $20K/mo steady state |
| Time to first content (Phase 1) | 2-3 weeks |
| Time to full platform (Phase 4) | 6-9 months |

---

## Phased Delivery

| Phase | Timeline | What | Cost |
|-------|----------|------|------|
| 1: SMS Coach | Weeks 1-6 | SMS prompts, 10-15 pilot practices, human editors | $15K + $5K/mo |
| 2: Mobile App | Months 2-4 | React Native app, AI pipeline, HQ dashboard, 65 practices | $80K + $10K/mo |
| 3: Full Network | Months 4-6 | 260+ practices, hardware kits, passive capture pilot | $88K + $18K/mo |
| 4: Intelligence | Months 6-12 | ML optimization, predictive prompts, self-optimizing | $30K + $20K/mo |
