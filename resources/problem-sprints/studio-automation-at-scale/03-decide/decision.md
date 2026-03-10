# Decision Record

**Sprint:** Studio Automation at Scale
**Agent:** Decision Architect
**Date:** 2026-03-10
**Status:** Awaiting user selection (Phase 5 gate)

---

## Evaluation Framework

Sprint questions weighted by strategic importance:
- **SQ1** (practice manager self-serves): Weight 3 — table stakes for the whole system
- **SQ3** (reduce print 30-40%): Weight 3 — the strategic opportunity the expert most wanted
- **SQ2** (AI brand quality): Weight 2 — matters but is a technical solvable; doesn't differentiate solutions
- **SQ5** (vendor consolidation): Weight 2 — important but downstream; any solution can route to a nominated vendor
- **SQ4** (DAM foundation): Weight 1 — prerequisite to all solutions equally; doesn't differentiate

Additional criteria:
- **Speed to pilot** (Weight 2): SGA has a small team; can't afford an 18-month build before learning
- **Practice manager experience** (Weight 3): Adoption fails if the experience is worse than texting the designer
- **Strategic fit with proactive shift** (Weight 2): Does this move the studio toward proactive brand stewardship?

---

## Solution Evaluation

### Solution A — The Conversational Studio Agent
*Chatbot in Teams/Slack handles intake, challenges medium, generates Canva assets, routes to approval + vendor*

| Sprint Question / Criterion | Score (1-5) | Rationale |
|-----------------------------|-------------|-----------|
| SQ1: Self-serve | 5 | Natural language intake is the lowest-friction possible — meets practice managers where they are |
| SQ3: Reduce print | 5 | Medium challenge is built into the conversation — the agent asks "why print?" before generating a flyer |
| SQ2: AI brand quality | 4 | Canva brand kit + locked templates mean AI generation is constrained to on-brand outputs |
| SQ5: Vendor consolidation | 4 | API print vendor connection is straightforward once assets are approved |
| SQ4: DAM foundation | 3 | Agent checks the brand kit but doesn't solve the upstream asset gap — pre-work still required |
| Speed to pilot | 3 | 10-14 weeks for v1; requires brand kit pre-work before launching |
| PM experience | 5 | Best-in-class UX — feels like texting a teammate, not filing a ticket |
| Proactive shift | 4 | Agent can proactively push content suggestions into Teams channels; proactive capability exists |

**Weighted Score: (5×3)+(5×3)+(4×2)+(4×2)+(3×1)+(3×2)+(5×3)+(4×2) = 15+15+8+8+3+6+15+8 = 78**

**Verdict: Recommended. Highest practice manager experience. Best medium-challenge mechanism. The strategic long-game.**

---

### Solution B — The Self-Service Brand Portal
*Web portal with template library, form-based intake, generation, approval workflow, print ordering*

| Sprint Question / Criterion | Score (1-5) | Rationale |
|-----------------------------|-------------|-----------|
| SQ1: Self-serve | 4 | Portal works, but requires a new login and deliberate navigation — friction vs. texting |
| SQ3: Reduce print | 3 | Requires practice manager to choose "TV Slide" over "Flyer" — doesn't challenge the medium |
| SQ2: AI brand quality | 3 | Template selection is deliberate, reducing AI misinterpretation; quality is consistent if not AI-generated |
| SQ5: Vendor consolidation | 4 | One-click print ordering from approved vendor is a portal feature |
| SQ4: DAM foundation | 4 | Brand health dashboard is a governance tool; studio can push asset updates from admin side |
| Speed to pilot | 4 | 8-12 weeks; no AI dependency for v1 — straightforward web development |
| PM experience | 3 | Usable but not delightful — practice managers will still sometimes email the designer |
| Proactive shift | 2 | Purely reactive — practice manager must initiate every request |

**Weighted Score: (4×3)+(3×3)+(3×2)+(4×2)+(4×1)+(4×2)+(3×3)+(2×2) = 12+9+6+8+4+8+9+4 = 60**

**Verdict: Viable. Fastest non-platform path. Best for SGA if AI build risk feels too high. But doesn't challenge the medium — misses the strategic print-reduction opportunity.**

---

### Solution C — The Proactive Content Studio
*Studio publishes monthly content packages; practices approve or skip; reactive requests become the exception*

| Sprint Question / Criterion | Score (1-5) | Rationale |
|-----------------------------|-------------|-----------|
| SQ1: Self-serve | 5 | One-tap monthly approval is the ultimate low-friction experience |
| SQ3: Reduce print | 5 | Monthly package defaults to digital (email, TV slides, social) — print is opt-in, not default |
| SQ2: AI brand quality | 4 | Studio controls production quality; AI assists, doesn't lead — quality is highest |
| SQ5: Vendor consolidation | 3 | Print ordering still needed for custom requests; vendor consolidation is secondary here |
| SQ4: DAM foundation | 3 | Same pre-work required as other solutions |
| Speed to pilot | 2 | 14-18 weeks; requires content operations infrastructure SGA doesn't have |
| PM experience | 5 | "Your March content is ready. Tap to approve." — easiest UX of any solution |
| Proactive shift | 5 | This IS the proactive shift — it defines the studio's operating model |

**Weighted Score: (5×3)+(5×3)+(4×2)+(3×2)+(3×1)+(2×2)+(5×3)+(5×2) = 15+15+8+6+3+4+15+10 = 76**

**Verdict: Strategically bold. Addresses the expert's "proactive vs. reactive" question directly. But requires the most organizational change and slowest path to pilot. Best as a Phase 2 evolution of whichever solution is chosen first.**

---

### Solution D — The Franchise Platform (Marq/Papirfly/Lytho)
*Deploy a purpose-built franchise brand management SaaS — template portal, DAM, approval workflows, vendor connections*

| Sprint Question / Criterion | Score (1-5) | Rationale |
|-----------------------------|-------------|-----------|
| SQ1: Self-serve | 4 | Portal UX is good; requires login and template navigation — similar to Solution B |
| SQ3: Reduce print | 2 | Platform doesn't challenge the medium; it serves whatever format the PM selects |
| SQ2: AI brand quality | 3 | Template guardrails ensure brand compliance; AI features depend on the platform chosen |
| SQ5: Vendor consolidation | 5 | Built-in vendor integrations are a core feature — best-in-class for print routing |
| SQ4: DAM foundation | 5 | Full DAM with per-location brand kits, version control, asset search — best DAM of any solution |
| Speed to pilot | 5 | Fastest to pilot — 6-10 weeks; configuration, not custom development |
| PM experience | 3 | Standard portal UX — good, not exceptional |
| Proactive shift | 2 | Reactive by design — practices submit requests |

**Weighted Score: (4×3)+(2×3)+(3×2)+(5×2)+(5×1)+(5×2)+(3×3)+(2×2) = 12+6+6+10+5+10+9+4 = 62**

**Verdict: Best infrastructure play. Fastest to deploy. But misses the medium-challenge opportunity and is the least bold solution. Best if SGA wants to solve the DAM and vendor problem first, then layer AI on top.**

---

### Solution E — The Embedded Trigger System
*Integrates with practice operational systems; creative assets auto-generate when triggers fire (patient visit, promotion scheduled, etc.)*

| Sprint Question / Criterion | Score (1-5) | Rationale |
|-----------------------------|-------------|-----------|
| SQ1: Self-serve | 5 | No request at all — the most radical self-serve |
| SQ3: Reduce print | 5 | Triggers route to digital channels by default; print is not a trigger output |
| SQ2: AI brand quality | 4 | Template-driven generation from known data — low misinterpretation risk |
| SQ5: Vendor consolidation | 3 | Vendor still needed for any print outputs; trigger system doesn't directly address this |
| SQ4: DAM foundation | 3 | Same pre-work as all solutions |
| Speed to pilot | 1 | 20-28 weeks minimum; multi-system integration across 160 locations |
| PM experience | 5 | Ultimate experience: "here's what we did for you this week" |
| Proactive shift | 5 | Fully proactive — no reactive intake at all for covered triggers |

**Weighted Score: (5×3)+(5×3)+(4×2)+(3×2)+(3×1)+(1×2)+(5×3)+(5×2) = 15+15+8+6+3+2+15+10 = 74**

**Verdict: Most ambitious long-term vision. Highest score on proactive shift. But 20-28 week runway, integration complexity, and practice resistance to system access make this a Phase 3 aspiration, not a sprint target.**

---

## Recommendation

**Primary recommendation: Solution A — The Conversational Studio Agent (score: 78)**

**Secondary recommendation (if AI build risk is a concern): Solution D — The Franchise Platform, as a foundation, with Solution A layered on top in Phase 2**

### Reasoning for A

Solution A scores highest because it uniquely answers the sprint question the expert cared most about: **getting practices to choose the right medium before committing to print.** A chatbot can ask "why print?" before generating a flyer. A portal, a content push, or a platform can't do that — the practice manager has already decided the format before they pick a template.

Additionally, Solution A has the best practice manager experience of any option, and adoption is the single biggest risk to any studio automation system. If practice managers don't use it, nothing else matters.

The weakest point for A is speed and pre-work dependency: brand kits for all 160 practices must exist before the agent is useful. This argues for running a **parallel track**: start building the Canva brand kit infrastructure (partial Solution D territory) immediately, while building the conversational agent for a 20-practice Gen4 pilot.

### Why Not B (Portal)

B is viable and should not be abandoned — it's the right fallback if AI build risk feels high. But it doesn't challenge the medium, and it requires a new login. The medium challenge is the expert's most-requested insight, and a portal can't deliver it without redesigning the portal into a wizard that's basically a non-conversational version of A.

### Why Not C (Proactive Push)

C is the right **Phase 2 evolution** once A is running. Once the agent has 6 months of request history, it can predict what each practice will need each month and push a pre-built package proactively. But C requires content operations infrastructure SGA doesn't have — it's too heavy for a sprint target.

### Why Not D (Franchise Platform)

D is the fastest to pilot and best for DAM infrastructure. **The recommendation is to use D's infrastructure playbook (Canva Enterprise brand kits, template library, nominated vendor) as the foundation layer — but build A's conversational intake on top of it** rather than letting the platform's native portal be the primary interface.

### Why Not E (Embedded Triggers)

E is the long-term vision. It is the right answer to "what does this look like in 5 years?" But the integration complexity and 28-week runway make it the wrong sprint target. Build A first. When it's running, map the high-frequency trigger types and start building E for those cases.

---

## User Selection Required

**The sprint HALTS here. You must select a solution direction before Phase 6 (Storyboard) begins.**

Your options:

| Option | Direction | What You're Choosing |
|--------|-----------|---------------------|
| **A** | Conversational Studio Agent | AI chatbot in Teams/Slack. Best PM experience. Challenges the medium. Needs brand kit pre-work. 10-14 weeks to pilot. |
| **B** | Self-Service Brand Portal | Web portal with template library. Fastest custom build. No medium challenge. Lower AI risk. 8-12 weeks. |
| **C** | Proactive Content Studio | Studio pushes monthly content packages. Boldest operating model shift. Slowest. Best for Phase 2. |
| **D** | Franchise Platform (Marq/etc.) | Deploy proven SaaS. Fastest to pilot. Best DAM + vendor. No medium challenge. Limited AI. 6-10 weeks. |
| **E** | Embedded Triggers | Integrates with practice ops systems. Most automated. Longest build. Best in 3-5 years. |

**What is your selection?**

---

| Field | Value |
|-------|-------|
| Selected solution | **A — Conversational Studio Agent** |
| Selection date | 2026-03-10 |
| Selected by | User |
| User refinement | The agent must also field email requests via a dedicated studio email address (e.g. `studio@sgadental.com`). Intake is multi-channel: Teams, Slack, **and email**. The agent reads inbound emails, responds conversationally, and processes requests identically to chat-based intake. |

### Why the Email Channel Matters

Practice managers at legacy SGA and MODIS practices are less likely to be in Teams or Slack. Email is the universal fallback — every practice manager has it. A dedicated studio email address (`studio@sgadental.com`) gives the agent an always-on inbox that works for every location, regardless of their collaboration tool. The agent reads inbound email, replies via the same address, and runs the same intake + routing workflow as the chat path. No request falls through because someone used the wrong channel.
