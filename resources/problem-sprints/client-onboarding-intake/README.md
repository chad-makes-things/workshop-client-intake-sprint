# Sprint: Client Onboarding Intake

**Date**: 2026-03-07
**Slug**: client-onboarding-intake
**Status**: complete

## Problem Statement
Pre-workshop client material collection and organization is manual, fragmented, and unscalable. The business runs AI-enabled multi-day workshops that produce dashboards and deliverables based on client insights. While the workshop experience itself is streamlined and loved by clients, the onboarding process prior to the workshop is a bottleneck: materials arrive via text, email, Dropbox, and secure file services in the form of PDFs, anecdotes, website snapshots, and ad-hoc conversations. All of this must be manually collected, organized, and structured into a framework of prompts that generate a market narrative — the foundation of the first workshop session. As client volume scales, this manual process cannot keep up.

## Solution: "The Interview" + "The Head Start" (Workshop Intake Engine)

A TurboTax-style conversational guided intake flow that walks each client through branching questions mapped to the workshop's prompt framework. Before the client begins, the system scrapes their web presence and pre-populates framework sections. The client reviews pre-filled content and fills gaps through guided questions, file uploads, voice recording, and link pasting. A facilitator video kickoff preserves the personal touch. The facilitator receives structured, framework-ready output on a dashboard with gap detection and one-click follow-up.

### Why This Direction
- Structured guidance solves the root cause of client non-submission (ambiguity, not laziness)
- Web scraping pre-population creates a "we did our homework" moment that elevates the brand
- Scales to unlimited concurrent clients without adding facilitator labor
- Self-segmentation handles the full spectrum of client digital sophistication
- Facilitator expertise is embedded in the question tree, not replaced

## Sprint Decisions

| Decision Point | Choice | Reasoning |
|---------------|--------|-----------|
| Sprint Target | Target A: Guided Intake & Collection (Steps 2-6) | Highest-risk behavioral assumptions; tests whether clients will use a structured system |
| Solution Direction | Solution 1: The Interview + Solution 3: Head Start integration | Strongest scalable solution (53/65); addresses root cause through structure |

## Sprint Output

| Phase | Folder | Files | Summary |
|-------|--------|-------|---------|
| Frame | [01-frame/](01-frame/) | [goal-and-questions.md](01-frame/goal-and-questions.md), [problem-map.md](01-frame/problem-map.md), [expert-notes.md](01-frame/expert-notes.md), [target.md](01-frame/target.md) | 2-year goal, 5 sprint questions, 15-step problem map with 4 actors, 25 HMW notes in 6 themes, 3 target candidates |
| Sketch | [02-sketch/](02-sketch/) | [inspiration.md](02-sketch/inspiration.md), [solution-sketches.md](02-sketch/solution-sketches.md) | 9 research inspirations, 5 distinct solution approaches |
| Decide | [03-decide/](03-decide/) | [decision.md](03-decide/decision.md), [storyboard.md](03-decide/storyboard.md) | Weighted evaluation matrix, 10-step storyboard from welcome email to framework-ready output |
| Architect | [04-architect/](04-architect/) | [product-architecture.md](04-architect/product-architecture.md), [index.html](04-architect/index.html) | 8 components, 12 entities, 5 ADRs, interactive HTML visualization |
| Validate | [05-validate/](05-validate/) | [learnings.md](05-validate/learnings.md) | 6 key learnings, unresolved risks, recommended next steps |

## Quick Links

- **Visual prototype**: Open [04-architect/index.html](04-architect/index.html) in any browser
- **Architecture**: [04-architect/product-architecture.md](04-architect/product-architecture.md)
- **Storyboard**: [03-decide/storyboard.md](03-decide/storyboard.md)
- **All 5 solutions**: [02-sketch/solution-sketches.md](02-sketch/solution-sketches.md)
- **Learnings & next steps**: [05-validate/learnings.md](05-validate/learnings.md)

## Context
- **Business model**: AI-enabled multi-day workshop producing dashboards and deliverables
- **Users**: Workshop facilitator (you) + workshop clients (businesses)
- **Current state**: Manual collection over text, email, Dropbox, secure file services
- **Pain point**: Organizing fragmented inputs into a structured framework for market narrative generation
- **Trigger**: Scaling client volume makes manual process unsustainable
