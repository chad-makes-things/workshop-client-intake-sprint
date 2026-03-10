# Sprint Target Recommendation

## Target Candidates

We evaluated three target areas from the problem map. Each represents a different "where do we focus the sprint?"

---

### Target A: Rapid Onboarding Pipeline (Steps 1-6)
**Focus**: Compress the acquisition-to-live-storefront journey from months to under 2 weeks.

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| Risk | 5 | This is the current #1 pain point. If we can't solve setup speed, the 365-day system doesn't matter — there's nothing to curate. |
| Impact | 5 | Directly affects every future acquisition (20-50 at a time). Reduces HQ team burden per practice from weeks to days. |
| Sprint Question Alignment | 5 | Tests SQ1 (setup speed), SQ2 (quality at scale), and partially SQ3 (practice engagement during intake). |
| Feasibility | 4 | Template-based generation is proven in other industries. Conversational intake is started. Asset access remains hard. |
| Learning Potential | 4 | We'll learn whether practices cooperate with a structured intake, whether templates maintain quality, and whether automation can handle the edge cases. |
| **Total** | **23/25** | |

---

### Target B: 365-Day Automated Curation Engine (Steps 7-9)
**Focus**: Build the system that keeps every practice's digital presence alive and current year-round.

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| Risk | 4 | High risk — this is where the scale breaks with a 10-person team. But it's a slower-burning problem than onboarding. |
| Impact | 5 | Affects all 260+ existing practices. The difference between "we set it up" and "it stays great." |
| Sprint Question Alignment | 4 | Tests SQ2 (quality control), SQ4 (change detection), SQ5 (visibility). Partially tests SQ3. |
| Feasibility | 3 | Automated content publishing is doable. Change detection is harder. Requires integrations with multiple systems (CMS, GMB API, social APIs, CRM). |
| Learning Potential | 4 | We'll learn if automation-first curation is good enough or if practices need more human touch than expected. |
| **Total** | **20/25** | |

---

### Target C: Practice-Facing Agent Interface (Steps 7-8, cross-cutting)
**Focus**: Build the 24/7 conversational agent that practices interact with for all digital storefront needs.

| Criterion | Score (1-5) | Rationale |
|-----------|:-----------:|-----------|
| Risk | 5 | The behavioral bet — will practices actually use it? If they don't, the whole system loses its practice-level input channel. |
| Impact | 4 | High impact for engagement, but the agent is a channel, not the core system. Without the underlying systems (templates, curation, dashboards), the agent has nothing to orchestrate. |
| Sprint Question Alignment | 4 | Directly tests SQ3 (will practices engage?). Touches SQ4 (change detection via agent). |
| Feasibility | 4 | Conversational AI is mature. SGA already has chatbot flow design. The challenge is connecting it to backend systems. |
| Learning Potential | 5 | We'll learn the fundamental behavioral question: do practices want to talk to an agent? This unlocks or kills a major part of the vision. |
| **Total** | **22/25** | |

---

## Recommendation: Target A — Rapid Onboarding Pipeline

**Why Target A?**

1. **It's the foundation.** You can't curate what doesn't exist. The 365-day system (Target B) and the agent interface (Target C) both assume that practices have a digital storefront to manage. Getting practices to a best-in-class baseline is the prerequisite for everything else.

2. **It tests the hardest operational question first.** SQ1 asks whether we can compress months to weeks. If the answer is no — if asset access, brand capture, and site deployment can't be systematized — then the automation-first vision has a structural problem that no amount of curation or agents can fix.

3. **It delivers immediate, visible value.** Every practice that gets onboarded faster is a practice that starts generating patient leads sooner. This is directly tied to revenue. The exec team can point to "we used to take 3 months, now we take 10 days" as a concrete win.

4. **It naturally extends into Targets B and C.** The onboarding pipeline creates the data layer, templates, and practice engagement patterns that the curation engine and agent interface build on top of. It's the foundation layer, not an isolated solution.

**What about Target C (the agent)?** Target C scores nearly as high (22 vs 23) and tests a critical behavioral question. My recommendation is that the agent interface becomes a key component *within* the onboarding pipeline — the conversational intake tool and the practice's first touchpoint with the system. This way, we test the agent concept (SQ3) while solving the onboarding problem (SQ1).

---

## Decision Needed

**Do you confirm Target A (Rapid Onboarding Pipeline) as the sprint focus?**

Alternatively:
- **Target B** if you believe the 260 existing practices' ongoing curation is more urgent than new acquisition onboarding
- **Target C** if you believe the agent/behavioral question is the biggest unknown and should be tested first
- **A + C hybrid** if you want the onboarding pipeline with the agent as the primary interface (this is my implicit recommendation)
