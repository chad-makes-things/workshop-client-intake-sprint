# Decision: Solution Evaluation

## Evaluation Criteria (from Sprint Questions)

Each solution scored 1-5 against the five sprint questions, plus two operational criteria.

| Criterion | Weight | Why It Matters |
|-----------|--------|---------------|
| SQ1: Setup speed (months → <2 weeks) | HIGH | The primary goal — if we can't compress setup, nothing else matters |
| SQ2: Quality control at scale (~10 person team) | HIGH | The constraint — system must work with current headcount |
| SQ3: Practice engagement with agent | HIGH | The behavioral bet — adoption is make-or-break |
| SQ4: Proactive change detection | MEDIUM | The ongoing value — catching changes before they rot |
| SQ5: Executive visibility | MEDIUM | The leadership requirement — seeing network health |
| Time to first value | HIGH | Can we show results in 30 days? |
| Stage 2 foundation | MEDIUM | Does this set up the 365-day curation layer? |

---

## Scoring Matrix

### Solution 1: "The Practice Concierge" — Agent-First Orchestration

| Criterion | Score | Rationale |
|-----------|:-----:|-----------|
| SQ1: Setup speed | 4 | Pre-population via scraping + guided conversational intake compresses setup dramatically. Not a perfect 5 because the backend integrations (CMS deployment, GMB API, social setup) still need to be built — the agent is the interface, not the engine. |
| SQ2: Quality at scale | 5 | The agent IS the quality control — it's opinionated by design, trained on SGA brand standards, and every recommendation follows the playbook. Human review only on exceptions. Scales linearly with practice count. |
| SQ3: Practice engagement | 5 | The entire design is built around the agent interaction. It's available 24/7, meets practices in their 5-minute windows, and provides immediate value. The "I already scraped your data — can we verify together?" opening is a strong hook. |
| SQ4: Change detection | 4 | The agent is the detection mechanism — practices tell it about changes, and it cascades. Doesn't solve passive detection (changes nobody reports), but creates a frictionless reporting channel. Can integrate with iSolved for HR-detected changes. |
| SQ5: Executive visibility | 3 | Not native — requires building a dashboard layer that aggregates data from backend systems. The agent generates data (enrollment status, interactions, changes reported), but the dashboard is a separate build. |
| Time to first value | 4 | 30-day MVP is feasible: core enrollment flow + website template integration + basic agent. Won't have all integrations, but can demonstrate the pattern with a batch of practices. |
| Stage 2 foundation | 5 | The agent-first architecture IS the Stage 2 foundation. The same agent that handles onboarding becomes the ongoing curation interface. Conversational relationship established during enrollment carries into day-to-day operations. |
| **Total** | **30/35** | |

### Solution 2: "The Assembly Line" — Industrialized Pipeline

| Criterion | Score | Rationale |
|-----------|:-----:|-----------|
| SQ1: Setup speed | 5 | Fastest setup per practice — it's a manufacturing process. 25 practices/week = entire network in 11 weeks. No practice engagement required, so no delays waiting for responses. |
| SQ2: Quality at scale | 4 | Human QC checkpoint ensures quality, but it IS a bottleneck (130+ hours of review for 260 practices). Works if the template quality is high enough that reviews are quick rubber-stamps. |
| SQ3: Practice engagement | 1 | Zero practice engagement by design. The pipeline processes practices without their input. This is a strength for setup speed but a fatal weakness for ongoing relationship. Practices don't know or care about their digital presence afterward. |
| SQ4: Change detection | 1 | No mechanism for practices to report changes. No agent interface. Changes are discovered whenever someone at HQ notices — which is the current broken state. |
| SQ5: Executive visibility | 3 | Pipeline metrics are easy to track (practices processed, in queue, deployed). But ongoing health monitoring isn't built in — you'd know setup status, not ongoing health. |
| Time to first value | 5 | Fastest to start — could process the first batch within 2 weeks. No complex technology required, mostly process + templates. |
| Stage 2 foundation | 1 | Weak. The pipeline gets storefronts live but establishes no practice relationship, no data feedback loop, and no ongoing engagement channel. Stage 2 would start from scratch on engagement. |
| **Total** | **20/35** | |

### Solution 3: "The Network Operating System" — Unified Platform

| Criterion | Score | Rationale |
|-----------|:-----:|-----------|
| SQ1: Setup speed | 3 | Once built, setup is instant (add a practice to the platform = storefront exists). But building the platform takes 3-6+ months. The 30-day goal is only achievable for a narrow MVP. |
| SQ2: Quality at scale | 5 | Ultimate quality control — SGA owns every layer. Templates, brand standards, and guardrails are baked into the platform. No third-party limitations. Batch operations are native. |
| SQ3: Practice engagement | 3 | Practice-facing view exists, but it's a dashboard/app, not a conversational agent. Requires practices to log in and navigate a UI. Better than email, worse than an agent that meets them where they are. |
| SQ4: Change detection | 5 | Single source of truth means changes propagate automatically. Staff update in the system = every surface updates. This is the strongest change management architecture. |
| SQ5: Executive visibility | 5 | Built-in. The platform IS the dashboard. Every metric is a query, not a separate build. Leadership has real-time visibility from day one of deployment. |
| Time to first value | 1 | Slowest to deliver. Even an MVP takes 60-90 days. The 30-day goal would only produce a data model and enrollment flow — no live storefronts. |
| Stage 2 foundation | 5 | The platform IS Stage 2. Ongoing curation, content publishing, review management, and change propagation are features of the platform, not separate systems. |
| **Total** | **27/35** | |

### Solution 4: "The Smart Wrapper" — Agent Over Best-of-Breed Tools

| Criterion | Score | Rationale |
|-----------|:-----:|-----------|
| SQ1: Setup speed | 4 | Similar to Concierge — agent handles intake, tools handle deployment. Slightly slower than pure pipeline because of multi-tool coordination, but faster than custom build. |
| SQ2: Quality at scale | 3 | Quality depends on the tools. SOCi handles social/GMB well. CMS templates handle websites. But cross-tool consistency is harder — different tools have different quality standards and capabilities. The wrapper adds a layer but can't control what's underneath. |
| SQ3: Practice engagement | 4 | Agent interface exists and handles practice interactions. But the agent is limited by what the underlying tools can do — if a practice asks for something the CMS doesn't support, the agent has to say no or route to a human. |
| SQ4: Change detection | 3 | Agent can receive change reports, but propagating across multiple disconnected tools is complex. SOCi handles its channels, CMS handles the website, but coordinating across them requires middleware that maintains consistency. |
| SQ5: Executive visibility | 3 | Each tool has its own analytics. Aggregating into a unified dashboard requires pulling from multiple APIs. Doable but not seamless — there will always be data reconciliation issues. |
| Time to first value | 4 | Fast — tools already exist. The custom build is the agent + integration middleware. Could have an agent + one tool integrated in 30 days. |
| Stage 2 foundation | 3 | Moderate. The agent relationship carries into Stage 2, but the tool fragmentation means curation workflows span multiple systems. Adding new capabilities means evaluating/integrating new tools. |
| **Total** | **24/35** | |

### Solution 5: "The Practice Playbook" — Guided Self-Service

| Criterion | Score | Rationale |
|-----------|:-----:|-----------|
| SQ1: Setup speed | 3 | Depends on practice engagement. If practices complete the Playbook quickly, sites are generated fast. If they don't, the 30-day auto-deploy kicks in — but that's a month of waiting. The bottleneck moves from HQ to the practice. |
| SQ2: Quality at scale | 4 | Guardrailed choices ensure quality. Every option is pre-approved by SGA. But the auto-deployed defaults for non-engaged practices may feel generic — quality is guaranteed but personality isn't. |
| SQ3: Practice engagement | 4 | The guided experience is engaging and gives practices ownership. But it's a web app, not a conversation — less natural than an agent, requires practices to log in and follow steps. The "I chose this" ownership is a real strength though. |
| SQ4: Change detection | 2 | The Playbook is primarily an onboarding tool. For changes, practices would need to "re-enter" the Playbook to update — less natural than telling an agent or updating a dashboard. No proactive detection. |
| SQ5: Executive visibility | 3 | Can track Playbook completion rates and storefront status. But ongoing health monitoring requires a separate system. |
| Time to first value | 3 | The Playbook itself can be built in 30 days, but value is realized only when practices complete it. With the auto-deploy fallback, all practices have storefronts within 60 days. |
| Stage 2 foundation | 2 | Weak for ongoing curation. The Playbook is a one-time enrollment experience. It doesn't create an ongoing engagement channel. Stage 2 would need a different interface. |
| **Total** | **21/35** | |

---

## Rankings

| Rank | Solution | Score | Verdict |
|------|----------|:-----:|---------|
| 1 | **Practice Concierge** | **30/35** | **RECOMMENDED** — Best balance of speed, quality, engagement, and Stage 2 readiness. The agent-first architecture aligns with SGA's vision and creates the foundation for everything that follows. |
| 2 | Network Operating System | 27/35 | STRONGEST LONG-TERM but too slow for 30-day goals. Consider as the Stage 3 evolution — after the concierge proves the model and captures the data, migrate to a unified platform. |
| 3 | Smart Wrapper | 24/35 | PRAGMATIC but fragmented. Good if SGA wants to minimize custom build, but tool dependency and data fragmentation create long-term friction. |
| 4 | Practice Playbook | 21/35 | CREATIVE but one-time. The guardrailed decision concept is valuable — incorporate it INTO the Concierge (the agent walks practices through the same decisions). |
| 5 | Assembly Line | 20/35 | FASTEST SETUP but dead-end. Gets storefronts live quickly but creates no practice relationship and no foundation for Stage 2. Could be used as a "blitz" for the bottom 20% of practices that have nothing — force-deploy defaults, then enroll them in the Concierge later. |

---

## Recommendation

### Primary: Solution 1 — "The Practice Concierge"

The Practice Concierge is the clear winner because it's the only solution that solves the setup problem AND establishes the ongoing relationship that Stage 2 requires. The agent-first architecture matches SGA's stated vision (chat today → voice → video avatar), and the pre-population approach ("I already know about your practice — let's verify together") is the hook that makes the first interaction valuable enough to drive adoption.

### Enhancement: Borrow from Solutions 2 and 5

- **From Solution 2 (Assembly Line)**: For the bottom 20% of practices that have literally nothing — no website, no GMB, no social — run a quick "blitz" pipeline to force-deploy default storefronts from scraped data. Then invite those practices into the Concierge to customize. Don't wait for engagement to get them to baseline.
- **From Solution 5 (Practice Playbook)**: The Concierge should use the "guardrailed decisions" pattern. When it asks about templates, colors, and preferences, the options are pre-curated. The practice feels like they're choosing, but every option is SGA-approved.

### Evolutionary Path

1. **Month 1**: Practice Concierge MVP (enrollment flow + website template engine)
2. **Month 2-3**: Add GMB, social, CRM integrations to the Concierge
3. **Month 3-6**: Build the 365-day curation layer (Stage 2 — Target B) using the Concierge as the ongoing interface
4. **Month 6-12**: Evaluate whether to evolve toward a Network Operating System (Solution 3) as the backend, with the Concierge remaining as the practice-facing interface

---

## Decision Needed

**Which direction do you want to take?**

- **Solution 1: Practice Concierge** (recommended, with elements from 2 and 5)
- **Solution 2: Assembly Line** (fastest to live storefronts, but dead-end)
- **Solution 3: Network OS** (strongest long-term, but 3-6 month timeline)
- **Solution 4: Smart Wrapper** (buy-first, pragmatic)
- **Solution 5: Practice Playbook** (practice-owned, guided self-service)
- **A combination** (specify what you'd like to combine)
