# Development Handoff — Workshop Intake Engine

**Sprint:** Client Onboarding Intake
**Date:** 2026-03-07
**Status:** Ready for development planning

---

## What Was Decided

A TurboTax-style guided intake flow ("The Interview") with web-scraping pre-population ("The Head Start") that replaces manual, multi-channel client material collection with a structured, scalable system. The system scrapes the client's web presence before they begin, pre-populates framework sections, then walks them through branching questions with multi-modal input (text, voice, file upload, links). The facilitator receives structured output on a dashboard.

---

## Handoff Map

### Product Manager

**Receives:** Sprint framing + chosen solution direction as epic/feature context.

| Sprint Artifact | Use As | File |
|----------------|--------|------|
| Long-term goal | Epic-level objective | [goal-and-questions.md](01-frame/goal-and-questions.md) |
| Sprint questions (5) | Acceptance criteria themes | [goal-and-questions.md](01-frame/goal-and-questions.md) |
| Problem map (15 steps) | User journey reference | [problem-map.md](01-frame/problem-map.md) |
| Solution sketch | Feature description | [solution-sketches.md](02-sketch/solution-sketches.md) (Solution 1) |
| Decision rationale | Prioritization context | [decision.md](03-decide/decision.md) |

**Key context for requirements:**
- The system must handle 4 client sophistication levels (self-segmentation at Step 3)
- Web scraping pre-population is a core feature, not a nice-to-have
- Voice recording input is critical for "talker" clients who won't type detailed answers
- The facilitator's existing prompt framework format is sacred — output must match it exactly
- Completeness scoring is internal (facilitator-facing), never shown to clients as a grade

### UX Designer

**Receives:** 10-step storyboard + HTML prototype as design foundation.

| Sprint Artifact | Use As | File |
|----------------|--------|------|
| Storyboard (10 steps) | Interaction design blueprint | [storyboard.md](03-decide/storyboard.md) |
| HTML prototype | Visual reference + architecture overview | [index.html](04-architect/index.html) |
| Solution narrative | Design intent | [solution-sketches.md](02-sketch/solution-sketches.md) (Solution 1) |

**Key design considerations from the storyboard:**
- Step 2 (Video Kickoff): Must feel personal, not corporate — direct eye contact, conversational tone
- Step 3 (Self-Segmentation): Options must use client language, no framework jargon, no option should feel like a confession
- Step 4 (Pre-Populated Review): Three response modes (confirm / edit / rewrite) must be visually distinct and always available
- Step 5 (Branching Questions): One question at a time, Typeform-style — never show a full form
- Step 6 (Flexible Input): Voice recording must work with single tap on mobile and desktop
- Step 7 (Pause/Return): No "are you sure?" dialogs; auto-save must be invisible
- Step 8 (Review/Submit): Yellow flags are optional suggestions, never submission blockers
- Step 9 (Dashboard): Content organized by framework section, not by submission chronology

### Tech Lead

**Receives:** Architecture document + ADRs as technical planning input.

| Sprint Artifact | Use As | File |
|----------------|--------|------|
| Product architecture (8 components) | System design reference | [product-architecture.md](04-architect/product-architecture.md) |
| ADRs (5) | Technical decision context | [product-architecture.md](04-architect/product-architecture.md) |
| Data model (12 entities) | Schema planning | [product-architecture.md](04-architect/product-architecture.md) |
| Storyboard | Functional requirements | [storyboard.md](03-decide/storyboard.md) |

**Key technical decisions (from ADRs):**
1. **Branching logic as JSON config graph** — not hardcoded; facilitator can evolve the question tree without code changes
2. **Two-tier web scraping** — lightweight first pass (homepage, about, meta) + deeper crawl (blog, team, products) async
3. **Multi-modal input on every question** — text, voice, file upload, and link paste available everywhere; async processing for heavy items
4. **Magic-link session management** — no account creation; token-based URLs that work cross-device
5. **Hybrid framework mapping** — structural mapping from question tree + AI classification for unstructured inputs (uploads, voice)

**Recommended tech stack:** Next.js + Node.js/TypeScript + PostgreSQL + Redis + S3 + LLM APIs

**8 components to plan:**
1. Intake Flow Engine (client-facing interview)
2. Facilitator Dashboard (admin view)
3. AI Content Processor (LLM integration for pre-population and categorization)
4. Session Manager (magic links, auto-save, cross-device)
5. Web Scraping Engine (site crawling and content extraction)
6. File Processing Pipeline (PDF extraction, transcription, link scraping)
7. Notification Service (email, SMS reminders and follow-ups)
8. Framework Template Store (branching logic config, framework schema)

---

## Development Sequence Recommendation

| Phase | Scope | Why First |
|-------|-------|-----------|
| **Phase 1** | Intake Flow Engine + Session Manager (Steps 1-8, no pre-population) | Core interview flow without web scraping; testable with real clients immediately |
| **Phase 2** | Web Scraping Engine + AI Content Processor (Step 4 pre-population) | Adds the "Head Start" feature; highest-delight upgrade |
| **Phase 3** | Facilitator Dashboard (Steps 9-10) | Replaces manual review; enables multi-client scaling |
| **Phase 4** | File Processing Pipeline + voice input (Steps 5-6 enrichment) | Multi-modal capture; improves input quality |
| **Phase 5** | Notification Service + smart reminders (Step 7) | Automates follow-up; reduces facilitator labor further |

---

## Open Questions for Development

1. What is the facilitator's exact prompt framework structure? (Needed to design the question tree and output format)
2. How many concurrent clients is the V1 target? (Affects infrastructure sizing)
3. Is there a preferred hosting environment or existing infrastructure?
4. Should the facilitator be able to edit the question tree themselves, or is that a developer task for now?
5. What is the budget for third-party services (LLM APIs, speech-to-text, web scraping)?

---

## Sprint Artifacts Index

All files live in `resources/problem-sprints/client-onboarding-intake/`:

```
README.md                          -- Sprint overview with all links
01-frame/
  goal-and-questions.md            -- 2-year goal + 5 sprint questions
  problem-map.md                   -- 15-step journey map with 4 actors
  expert-notes.md                  -- 25 HMW notes in 6 themes + expert insights
  target.md                        -- 3 target candidates with scoring
02-sketch/
  inspiration.md                   -- 9 research inspirations
  solution-sketches.md             -- 5 distinct solution approaches
03-decide/
  decision.md                      -- Weighted evaluation matrix + selection record
  storyboard.md                    -- 10-step detailed user journey
04-architect/
  product-architecture.md          -- 8 components, 12 entities, 5 ADRs
  index.html                       -- Interactive HTML visualization (open in browser)
05-validate/
  learnings.md                     -- Key learnings, risks, next steps
handoff-to-dev.md                  -- This file
```
