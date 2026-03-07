# Sprint Target — Candidates and Recommendation

## Target Candidates

Based on the problem map, three areas of the journey emerge as viable sprint targets. Each represents a different slice of the problem with different risk/reward profiles.

---

### Target A: Guided Intake and Collection (Steps 2-6)

**Focus:** Design the system that replaces scattered multi-channel collection with a guided, flexible intake experience that works for all client types.

**What this covers:**
- How the client receives the intake request
- What the intake experience looks and feels like
- How materials arrive in one place regardless of source channel
- How clients with minimal materials are guided to provide enough

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| **Risk** | 5 | Highest behavioral risk — requires clients to change how they submit. If they won't use it, everything else fails. Tackling this first de-risks the entire system. |
| **Impact** | 4 | Solves the fragmentation problem and reduces the facilitator's collection burden. Does not address downstream organization or synthesis, but removes the biggest source of chaos. |
| **Sprint Question Alignment** | 5 | Directly addresses Q1 (will clients submit through a system?), Q3 (sophistication spectrum), and Q4 (personal touch). Hits 3 of 5 sprint questions head-on. |
| **Feasibility** | 4 | Can be prototyped as a conversational intake flow with file upload, website scraping, and progressive questioning. No novel technology required. |
| **Learning Potential** | 5 | This is where the biggest unknowns live. We will learn whether clients actually engage with structured intake, what friction points emerge, and how input quality changes. |
| **Total** | **23/25** | |

---

### Target B: Automated Organization and Framework Mapping (Steps 7-10)

**Focus:** Design the system that takes collected materials and automatically organizes them into the prompt framework that generates the market narrative.

**What this covers:**
- Automatic file processing (PDF extraction, transcription, website parsing)
- Categorization of materials against the prompt framework
- Gap detection and follow-up question generation
- Structured output ready for AI narrative generation

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| **Risk** | 4 | High technical risk — the framework mapping is where the facilitator's expertise currently lives. Automating tacit knowledge is hard. But this risk is more tractable than behavioral risk. |
| **Impact** | 5 | This is the biggest time sink per client. Automating the messy middle eliminates the scaling bottleneck entirely. If this works, the facilitator's prep time drops from hours to minutes. |
| **Sprint Question Alignment** | 3 | Primarily addresses Q2 (narrative quality) and Q5 (facilitator adoption). Less direct connection to Q1, Q3, or Q4. |
| **Feasibility** | 3 | Requires deep understanding of the prompt framework structure. Would need the facilitator to articulate their tacit categorization logic. Technically possible but requires more design work. |
| **Learning Potential** | 4 | We learn whether the prompt framework can be made explicit enough for automation, and whether the facilitator trusts automated organization. |
| **Total** | **19/25** | |

---

### Target C: End-to-End Narrative Generation Pipeline (Steps 4-14)

**Focus:** Design the full pipeline from client input to finished market narrative, treating the entire intake-to-narrative flow as one system.

**What this covers:**
- Everything in Target A and Target B
- AI narrative synthesis and quality controls
- Facilitator review and editing workflow
- The complete system architecture

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| **Risk** | 3 | Spreads risk across the entire journey rather than concentrating on the highest-risk area. We learn a little about everything but may not deeply validate any single assumption. |
| **Impact** | 5 | If it works, the entire problem is solved. Maximum potential impact. |
| **Sprint Question Alignment** | 4 | Touches all 5 sprint questions, but may not go deep enough on any one to produce a definitive answer. |
| **Feasibility** | 2 | Too broad for a single sprint. The intake experience, the automation pipeline, and the narrative generation are three distinct design challenges. Attempting all three risks shallow solutions across the board. |
| **Learning Potential** | 3 | Breadth over depth. We get a surface-level read on the full system but may not answer the hard questions (especially Q1 — will clients actually use it?). |
| **Total** | **17/25** | |

---

## Scoring Summary

| Target | Risk | Impact | Q Alignment | Feasibility | Learning | Total |
|--------|:----:|:------:|:-----------:|:-----------:|:--------:|:-----:|
| **A: Guided Intake** | 5 | 4 | 5 | 4 | 5 | **23** |
| **B: Framework Mapping** | 4 | 5 | 3 | 3 | 4 | **19** |
| **C: End-to-End** | 3 | 5 | 4 | 2 | 3 | **17** |

---

## Recommendation: Target A — Guided Intake and Collection

**Why this is the recommended target:**

The intake experience is where the highest-risk assumptions live. The entire system depends on one thing: will clients provide their materials through a structured channel? If they won't, nothing downstream matters. No amount of brilliant automation in the framework mapping or narrative generation will help if the inputs are still arriving as scattered texts, emails, and Dropbox links.

Target A lets us test the hardest behavioral question first. It also has the highest learning potential — we will discover what clients are willing to do, what causes friction, and how input quality changes when the experience is guided rather than ad hoc.

Additionally, Target A has strong feasibility. A guided intake flow — potentially conversational, with file upload, website auto-scraping, and progressive questioning — can be prototyped with existing technology. It does not require the facilitator to fully articulate their tacit framework mapping logic (that comes later, in Target B territory).

**What the sprint would explore:**
- A guided intake experience that feels conversational, not bureaucratic
- Automatic website scraping as a "head start" for every client
- Flexible material submission (upload, link, paste, even voice)
- Progressive disclosure — show clients what's been captured, ask only for what's missing
- A design that works for both the polished-brand-deck client and the "we-just-have-stories" client

**What it explicitly does NOT cover (future sprints):**
- The automation of framework mapping (Target B)
- AI narrative generation and quality tuning
- Facilitator review and editing workflow

---

## Decision Required

**This recommendation is presented for your decision.** Please review the three targets and confirm:

1. **Proceed with Target A** (recommended) — Focus on the guided intake and collection experience
2. **Proceed with Target B** — Focus on automated organization and framework mapping
3. **Proceed with Target C** — Attempt the full end-to-end pipeline
4. **Modify** — Adjust the target scope or combine elements differently

The sprint will halt here until you select a direction.
