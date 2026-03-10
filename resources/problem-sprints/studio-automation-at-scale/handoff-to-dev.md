# Development Handoff Manifest

**Sprint:** Studio Automation at Scale
**Agent:** Facilitator
**Date:** 2026-03-10
**Solution:** A — Conversational Studio Agent (multi-channel: Teams, Slack, Email)

---

## Handoff Summary

The sprint is complete. This manifest maps all sprint artifacts to the development team inputs. The Product Manager, UX Designer, Tech Lead, Backend Engineer, and Frontend Engineer each receive a specific subset of sprint artifacts as their starting context — they do not start from scratch.

---

## Product Manager → Epic Creation

**Receives:**
- [Goal & Sprint Questions](01-frame/goal-and-questions.md) — 18-month goal + 5 testable sprint questions become the epic's acceptance criteria framework
- [Problem Map](01-frame/problem-map.md) — 10-step journey + actor map defines the user personas and pain points driving each story
- [Decision Record](03-decide/decision.md) — Solution A selection + scoring provides the "why we're building this" context for stakeholder communications
- [Expert Notes](01-frame/expert-notes.md) — 8 key constraints (capacity, brand kit gaps, Staples pain, Canva workflow) shape story acceptance criteria

**PM's job:** Translate the sprint framing into a structured epic with user stories, Given/When/Then acceptance criteria, technical spike callouts, and a dependency map. The sprint goal becomes the epic goal. The 5 sprint questions become the 5 epic pillars.

**Key constraints from sprint to carry into the epic:**
- Phase 1 pilot is 15 Gen4 practices — scope stories to this boundary
- Brand kit pre-work (asset backfill for underserved practices) is a parallel track, not a story inside this epic
- Template library starts at 12 — expanding templates is a studio admin function, not a story
- CD review SLA is 2 hours during business hours with named backup

---

## UX Designer → Design Specifications

**Receives:**
- [Storyboard](03-decide/storyboard.md) — 8-step user journey (Ashley at Towne Lake Dental) is the design brief. Each step maps to one or more screens.
- [Solution Sketches](02-sketch/solution-sketches.md) — Solution A narrative describes the interaction model; Solutions B-D describe approaches considered and rejected (design should not regress toward these)
- [HTML Visualization](04-architect/index.html) — Open in browser for the before/after framing, component relationships, and storyboard visual

**UX's job:** Build screen wireframes for every interaction in the 8-step storyboard: email intake UX (what does the agent's reply email look like?), preview delivery format, CD mobile review notification, practice dashboard (if any), and the print order confirmation. Also define the design system: the studio-facing admin dashboard, the CD mobile review experience, and the analytics dashboard.

**Key UX constraints from sprint:**
- The practice manager's primary interface is email — the "app" is the agent's reply email. Design the email format as a first-class product surface.
- CD review must be mobile-first and tap-to-approve. No scrolling to approve. Thumbnail + 3 buttons visible without scrolling on a standard phone.
- Medium challenge message must be non-blocking — it surfaces an option, not an objection. Tone: "here's something that might work even better" not "are you sure you want print?"
- Maximum 3 clarifying questions per intake conversation. UX should never design a flow that requires more.

---

## Tech Lead → Implementation Plan

**Receives:**
- [Product Architecture](04-architect/product-architecture.md) — 8 components with responsibilities, interfaces, and technology recommendations
- [Decision Record](03-decide/decision.md) — 7 ADRs with rationale; Tech Lead may revise but must document why
- [Solution Sketches](02-sketch/solution-sketches.md) — Sprint questions SQ1-SQ5 define the technical success criteria

**Tech Lead's job:** Validate the proposed stack (Claude API, Canva Connect API, Gelato API, Microsoft Bot Framework, M365 email integration). Produce database schema, API endpoint definitions, infrastructure plan (Azure recommended per ADR 7's Microsoft ecosystem alignment), engineering task breakdown, and risk register. Pay particular attention to: Canva Connect API capability verification (ADR 3 notes this needs validation), Weave API access requirements (Phase 1 digital deployment), and multi-channel adapter architecture (Teams + Email in Phase 1, Slack in Phase 2).

**Key technical risks from sprint to flag in implementation plan:**
1. Canva Connect API template variable limits — verify before committing to 12-template library
2. Weave API patient list access — data privacy model and consent requirements
3. CD review escalation path — named backup and notification logic must be defined before build
4. Multi-practice-management-system diversity (Dentrix, Eaglesoft, Curve) — affects future Phase 2 trigger integrations; document now even if not building yet

---

## Backend Engineer → Phase 1 Backend Spec

**Receives:**
- [Product Architecture](04-architect/product-architecture.md) — Components 1-3 and 7-8 are primarily backend: channel adapters, agent core, practice profile/DAM management, print fulfillment, analytics
- [Storyboard](03-decide/storyboard.md) — Steps 1-3 and 6-8 are backend-heavy: intake normalization, Canva API calls, Gelato API calls, digital deployment triggers

**Backend's job:** Phase 1 backend spec covering: email ingestion service (M365 API polling or webhook), practice profile database schema, Claude API integration layer (prompt management, conversation threading per request), Canva API client (brand kit read, template generation, export), Gelato API client (order submission, confirmation handling), analytics event logging, and CD review notification service.

---

## Frontend Engineer → Phase 1 Frontend Spec

**Receives:**
- [Storyboard](03-decide/storyboard.md) — The practice manager's primary UI is email (no frontend app needed for their flow). The CD review notification is mobile (Teams push). The studio admin dashboard and analytics are the primary frontend surfaces.
- [Product Architecture](04-architect/product-architecture.md) — Component 8 (Studio Analytics Dashboard) is the primary frontend deliverable for Phase 1

**Frontend's job:** Phase 1 frontend spec covering: Studio Analytics Dashboard (request volume, turnaround, medium split trend, brand kit health, CD review time, 90-day inactive flag), CD review experience (mobile-optimized push notification with thumbnail + approve/reject), and studio admin interface (brand kit management, template library view, practice profile editor). Note: the practice manager's primary interface is the agent's reply email — this is NOT a web app for practice managers in Phase 1.

---

## Shared Context

This handoff is registered in `.context/artifacts.json`. All development output should be written to:

```
resources/problem-sprints/studio-automation-at-scale/06-development/
  requirements/epic.md              ← Product Manager
  design/design-specs.md            ← UX Designer
  technical/implementation-plan.md  ← Tech Lead
  backend/phase1-backend-spec.md    ← Backend Engineer
  frontend/phase1-frontend-spec.md  ← Frontend Engineer
```

Sprint README will be updated to include the 06-development section with links to all specs when the development package is complete.
