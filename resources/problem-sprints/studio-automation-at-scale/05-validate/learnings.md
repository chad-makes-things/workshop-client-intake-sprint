# Sprint Learnings

**Sprint:** Studio Automation at Scale
**Agent:** Facilitator
**Date:** 2026-03-10

---

## What We Learned

### 1. The problem was never "not enough design capacity"

The framing at the start pointed toward capacity as the constraint (1 designer, 160 locations). The sprint revealed the real problem is **friction density** — each request requires too many touches across too many people for too long. The actual request volume is manageable for a two-person team if the workflow is right. This changes the ROI framing: the value of the agent isn't "replacing a designer hire" — it's "making the designer's time available for work that actually requires design talent."

### 2. The medium challenge is the highest-leverage feature — and the most underestimated

Every solution in the sketch phase could automate design production. Only Solution A could challenge the medium before the practice committed to a format. This is where the most strategic value is: a practice manager who would have ordered 200 flyers instead schedules a patient email + TV slide rotation that reaches more people at zero print cost. Over 160 locations, this compounds fast. The medium challenge has to be in the first reply — not after the practice has already said "I need flyers."

### 3. Email as an intake channel is more important than it first appeared

The instinct was to build for Teams and Slack. The user's refinement — add email with a dedicated studio address — is correct for a less obvious reason: **email creates the audit trail**. Every request, every brief, every revision, every approval confirmation is threaded in email. For a multi-location DSO with legal and brand governance needs, that documentation is valuable. It also provides a natural archive the studio can use to identify request patterns and proactively template common needs.

### 4. The asset backfill is the critical path — it's not a nice-to-have

Approximately 30-40% of practices have brand assets inadequate to support automated design generation. The sprint recommends starting the pilot with Gen4 practices because their brand kits are already complete. But for the system to serve all 160 locations, the asset backfill project (building brand foundation packages for underserved practices) must be resourced and scheduled in parallel with the agent build — not after it. The two tracks are: (1) build the agent, (2) build the brand kits. Both need to land at the same time for the system to work at scale.

### 5. The Creative Director review step is a feature, not a bottleneck — if it's designed right

The initial framing of the CD review was as a potential bottleneck ("single point of failure"). The architecture inverts this: by (a) having the practice confirm content accuracy first, (b) making the review mobile and tap-to-approve, and (c) measuring CD review time in the analytics dashboard, the CD review becomes a quality signal and governance mechanism — not a delay. The 60-second target is achievable because the CD isn't redesigning; they're judging.

### 6. Vendor consolidation is a workflow design problem, not a negotiation problem

The instinct is to solve the Staples problem by finding a better vendor. The sprint revealed the deeper solution: make it so easy to use the nominated vendor through the workflow (one-click order, correct specs auto-loaded, tracking delivered directly to practice) that no practice manager would voluntarily go to Staples instead. The nominated vendor wins through convenience, not mandate.

---

## What We'd Do Differently

- **Start the brand kit audit earlier.** The sprint could have produced a brand health scoring rubric as a Phase 1 deliverable — giving the studio team a tool to triage the 50+ practices that need asset work before they can use the automation system.
- **Scope the Canva API more explicitly.** The architecture assumes Canva's Connect API supports full template variable population and programmatic export. This needs verification before the build begins — Canva API capabilities have evolved and some template types may have limitations.
- **Define the TV slide deployment path earlier.** The decision to make TV slide deployment manual in Phase 1 is pragmatic, but it should prompt an immediate inventory of what display systems are in the Gen4 pilot practices — so Phase 2 integration scope can be estimated before Phase 1 goes live.

---

## Unresolved Questions

1. **Canva API scope:** What specific template manipulation and export capabilities are available in the Canva Connect API for the template types in the library? Are there limitations on variable fields, image replacement, or export formats that would require fallback to manual generation?

2. **Weave API access:** Does SGA/Gen4 have an existing Weave API relationship, or does this require a new contract and integration agreement? What's the data privacy model for patient list access from the studio agent?

3. **CD review escalation:** If Marcus is unavailable and the 4-hour auto-escalation fires, who is the backup reviewer? The architecture needs a named backup and a clear notification path.

4. **Practices that resist the system:** What's the policy for practice managers who continue to email designers directly, bypassing the agent? The system needs a soft-redirect response ("Hey, I see this came in directly — I'm going to route it through the Studio Agent so we can track it. Here's how to reach the agent next time.") rather than a hard refusal.

5. **Proactive content calendar (Phase 2):** The analytics dashboard will surface patterns — what practices request most often, what seasonal promotions are common. The next evolution is to use those patterns to push a pre-built monthly content package (Solution C from the sketch phase). When does Phase 2 begin, and who owns the content calendar production?

---

## Sprint Metrics (At Close)

| Metric | Value |
|--------|-------|
| Sprint duration | 1 day (compressed) |
| Phases completed | 8 of 8 (Phases 1-8) |
| Files produced | 10 (goal + map + expert notes + HMW + target + inspiration + sketches + decision + storyboard + architecture + HTML + learnings) |
| Decision points passed | 2/2 (target confirmed, solution selected) |
| User refinements incorporated | 1 (email channel added to intake) |
| HTML visualization | ✅ Complete — opens in browser, no external dependencies |
