# Solution Sketches

## Confirmed Target
**A + C Hybrid**: Network-Wide Digital Modernization Pipeline with Practice-Facing Agent Interface.
- Must handle both new acquisitions AND re-enrollment of all 260+ existing practices.
- Agent interface is the primary touchpoint for practices.
- Stage 2 (365-day curation) builds on top of whatever we design here.

---

## Solution 1: "The Practice Concierge" — Agent-First Orchestration Platform

### Narrative
Every practice gets their own AI concierge — a conversational agent (starts as chat, evolves to voice/video avatar) that is their single point of contact with SGA marketing. The concierge is opinionated, proactive, and backed by SGA's brand standards knowledge base. It handles everything: onboarding intake ("Tell me about your practice"), change management ("Dr. Johnson is leaving"), content approval ("Here's next week's social posts — approve or flag"), and Q&A ("When is 4Front this year?").

Behind the concierge, an orchestration layer routes actions to specialized systems: website template engine, GMB API, social publishing pipeline, CRM, and the executive dashboard. The practice never sees these systems — they just talk to the concierge.

**Enrollment flow for existing practices**: The concierge pre-populates everything it can from public data (website scraping, GMB, social profiles, review sites). Then it initiates a guided enrollment conversation: "Hi, I'm your SGA Marketing Concierge. I've already pulled your current website content, GMB profile, and social handles. Here's what I found — can we verify this together?" The practice confirms/corrects in a 15-minute chat session. The concierge generates a new storefront from the verified data and specialty template. Practice approves. Live in days.

**Enrollment flow for new acquisitions**: Same process, but the concierge is introduced as part of the deal close. Digital asset handoff checklist is a closing requirement. The concierge walks the practice through asset transfer during the first week.

### Strengths
- Single interface solves the "nuisance" problem — practices talk to one thing, not 5 different HQ people
- Agent can be opinionated ("I recommend this layout because periodontist sites convert 23% better with referral-optimized structure")
- Scales infinitely — adding a practice means deploying another concierge instance
- Pre-population via web scraping reduces practice burden dramatically
- Future-proof — chat today, voice tomorrow, video avatar next year

### Risks
- Adoption risk: practices may not engage with a chatbot, especially if the first interaction isn't immediately valuable
- Orchestration complexity: connecting the agent to 5+ backend systems (CMS, GMB API, social APIs, CRM, dashboard) is significant integration work
- The agent needs to be genuinely good — a bad chatbot experience will kill adoption permanently

### Feasibility
**High**. Conversational AI is mature. SGA already has a chatbot flow design. The backend integrations are standard APIs. The main build is the orchestration layer and the practice data model. 30-day MVP is feasible for a core enrollment flow + 1-2 backend integrations.

---

## Solution 2: "The Assembly Line" — Industrialized Pipeline with Human Checkpoints

### Narrative
Rather than an agent-first approach, build a manufacturing-style pipeline where practices are "processed" through a series of automated stations, each with a human quality checkpoint. Think of it like a car assembly line: raw materials (practice data) enter at one end, a finished digital storefront comes out the other end.

**Stations**:
1. **Data Harvesting** — Automated scraping + structured intake form captures all practice data
2. **Brand Generation** — Template engine selects specialty template, applies practice data, generates site + GMB + social profiles
3. **Quality Review** — HQ creative reviews generated assets (human checkpoint, ~30 min per practice)
4. **Deployment** — Automated deployment of site, GMB sync, social handle setup, CRM connection
5. **Practice Approval** — Practice receives a preview link and has 3 weeks to review (if no response, it goes live)
6. **Activation** — Everything goes live, monitoring begins

For existing practices: Run a "network modernization sprint" where batches of 20-30 practices are processed through the pipeline weekly. At 25/week, the full 260+ network is modernized in ~11 weeks.

### Strengths
- Predictable throughput — you can calculate exactly how many practices per week
- Human QC at the checkpoint ensures quality without requiring practice input
- The "goes live if no response in 3 weeks" approach solves the 90% who don't care
- Simple to explain to leadership: "we process 25 practices per week, entire network done by Q2"
- Less technology risk — no complex agent system to build

### Risks
- Doesn't solve the ongoing engagement problem — practices still have no direct interface
- Human checkpoint is a bottleneck — 30 min × 260 practices = 130 hours of review work
- Doesn't handle the 10% who want deep customization — they clog the pipeline
- No built-in change management — when something changes, there's no mechanism for the practice to report it
- Sets up Stage 2 (curation) for failure because there's no practice relationship established

### Feasibility
**Very High**. This is the lowest-technology solution. Mostly process design + templates + basic automation. Could start processing practices within 2 weeks of building the pipeline.

---

## Solution 3: "The Network Operating System" — Unified Digital Platform

### Narrative
Build a custom, unified platform that IS the digital storefront for every practice. Not a system that generates WordPress sites — a system that IS the website, IS the GMB manager, IS the social publisher, IS the CRM. Every practice's digital presence runs on the same platform, like Shopify runs every merchant's store.

**The platform has three interfaces**:
- **Practice view**: Simplified dashboard showing their storefront status, pending approvals, upcoming content, and a chat interface for requests/changes
- **HQ view**: Network-wide management — batch operations, template management, content pipeline, QC queue
- **Executive view**: Dashboard showing digital health scores, practice maturity tiers, KPIs, alerts

Every practice's data lives in one canonical data model. Change anything in the data model → it propagates to every surface (website, GMB, social, directories, marketing materials). Staff change in the system = automatic website update + GMB update + social update + signage notification.

### Strengths
- Single source of truth eliminates the cascading update problem entirely
- One platform means one codebase, one team to maintain, one system to learn
- Batch operations are native — "update all GP practice sites to new template" is one click
- Executive visibility is built-in, not bolted on
- Most control over quality, since SGA owns every layer of the stack

### Risks
- Build vs. buy: this is a significant custom software project (months, not weeks)
- You're building a SaaS company inside a dental DSO — do you have the engineering capacity?
- Practices might resist moving off their existing platforms (especially the 10% who care)
- Migration of 260+ practices onto a new platform is a massive project
- If the platform has a bug or goes down, every practice's digital presence is affected

### Feasibility
**Medium**. This is the most ambitious solution. A full build could take 6-12 months. However, a Phase 1 MVP (website template engine + practice data model + basic dashboard) is achievable in 60-90 days. The 30-day goal would only cover the data model and enrollment flow.

---

## Solution 4: "The Smart Wrapper" — Agent Layer Over Best-of-Breed Tools

### Narrative
Don't build the underlying systems — buy them. Use SOCi (or similar) for social/GMB/reputation management. Use a template CMS (Webflow, WordPress Multisite) for websites. Use an existing CRM. Then build a thin but smart orchestration layer + practice-facing agent on top.

The agent is the "smart wrapper" that talks to practices and routes their requests to the right tool. The HQ team manages the underlying tools directly. The agent reduces practice friction and provides the 24/7 availability, but the heavy lifting is done by proven platforms.

**Enrollment flow**: Agent scrapes public data → populates the CMS template + SOCi profile + CRM record → presents to practice for confirmation → HQ reviews → deploys across all platforms via their native APIs.

### Strengths
- Fastest to market — leveraging existing platforms means less custom development
- Each tool is best-in-class at its specific job (SOCi for social, Webflow for sites, etc.)
- Lower maintenance burden — tool vendors handle updates, bugs, infrastructure
- The agent layer is a focused build — conversational AI + API integrations
- Can evolve underlying tools without changing the practice experience

### Risks
- Vendor dependency — if SOCi changes their API or pricing, you're exposed
- Integration complexity — connecting 4-5 tools through a unified agent requires significant middleware
- Data lives in multiple systems — the "single source of truth" benefit is weaker
- Cost: SOCi + CMS + CRM + agent platform = significant SaaS spend × 260+ locations
- The "wrapper" can only be as good as the tools underneath — limitations cascade

### Feasibility
**High**. SOCi and similar platforms already support multi-location dental. The agent layer is the custom build. A 30-day MVP could integrate the agent with 1-2 tools (e.g., website template engine + GMB), with additional integrations in subsequent sprints.

---

## Solution 5: "The Practice Playbook" — Guided Self-Service with Automation Backbone

### Narrative
This is the most unconventional approach. Instead of building FOR practices, build a system that guides practices through building their own digital storefront — but with so many guardrails and so much automation that "building it yourself" means making 5-10 decisions while the system does the rest.

Each practice gets a "Digital Storefront Playbook" — a step-by-step guided experience (web app, not a chatbot) that walks them through:
1. Verify your practice info (pre-populated from scraping)
2. Choose your template (2-3 options for your specialty)
3. Pick your color palette (from SGA-approved options)
4. Upload your team photos (or schedule a photo shoot)
5. Confirm your services and hours
6. Review and approve your storefront

Behind the scenes, every decision triggers automation: template selected → site generated. Colors chosen → applied across all assets. Photos uploaded → processed and placed. The practice feels ownership ("I built my digital presence") while SGA maintains control (every option is pre-approved).

For practices that don't engage: after 30 days of no activity, the system auto-generates a storefront from default settings + scraped data and deploys it. "We built this for you. If you'd like to customize it, your Playbook is always available."

### Strengths
- Practice ownership increases engagement and reduces pushback ("you chose this, remember?")
- Guardrailed decisions mean quality is guaranteed regardless of what the practice chooses
- The auto-deploy fallback handles the 90% who won't engage
- Creates a natural onboarding moment that establishes the digital relationship
- The Playbook can expand over time (social preferences, content themes, review response style)

### Risks
- Still relies on practice engagement, which is the core behavioral risk
- The guided web app is a custom build with significant UX requirements
- "5-10 decisions" might still be too many — practices might want zero decisions
- Doesn't naturally solve the ongoing curation/change management problem
- Less agent-like, more tool-like — doesn't match the conversational vision

### Feasibility
**High**. Guided decision flows are straightforward web development. The automation backbone (template engine, GMB API, social setup) is the same regardless of which solution we choose. 30-day MVP is feasible.

---

## Substitution Test

| Solution | If we swapped this out, would the sprint feel different? |
|----------|--------------------------------------------------------|
| 1: Practice Concierge | YES — removes the agent-first, conversational paradigm entirely |
| 2: Assembly Line | YES — removes practice interaction, makes it a manufacturing problem |
| 3: Network OS | YES — changes from integration to platform build, fundamentally different scope |
| 4: Smart Wrapper | YES — changes build philosophy from custom to integration-first |
| 5: Practice Playbook | YES — shifts from agent-driven to practice-driven with automation |

All 5 solutions are genuinely distinct. They solve the same problem through different paradigms.

---

## Quick Comparison

| Dimension | 1: Concierge | 2: Assembly Line | 3: Network OS | 4: Smart Wrapper | 5: Playbook |
|-----------|:------------:|:-----------------:|:-------------:|:-----------------:|:-----------:|
| Practice Engagement | High | None | Medium | Medium | Medium-High |
| Time to First Value | 30 days | 14 days | 90+ days | 30 days | 30 days |
| Custom Build Effort | Medium | Low | Very High | Low-Medium | Medium |
| Ongoing Curation Ready | Yes | No | Yes | Yes | Partial |
| Executive Visibility | Via dashboard | Via reporting | Built-in | Via tools | Via dashboard |
| Scales to 300+ | Yes | Yes (with staff) | Yes | Yes | Yes |
| Stage 2 (365-day) Foundation | Strong | Weak | Very Strong | Medium | Medium |
