# Goal & Sprint Questions

## Long-Term Goal

**In 30 days, SGA will have a system where any newly acquired practice can go from acquisition close to a fully operational, best-in-class digital storefront in under 2 weeks — and every existing practice in the 260+ network receives automated, opinionated 365-day digital curation with executive visibility into network-wide digital health — managed by the same small HQ team.**

### Goal Validation

| Test | Result |
|------|--------|
| **Ambition** | Compressing months → 2 weeks is a 4-8x improvement. 365-day curation across 260+ locations with a ~10-person team requires fundamental automation. This is ambitious. |
| **Specificity** | Clear metrics: setup time (months → 2 weeks), curation coverage (sporadic → 365/year), visibility (none → dashboard), team size (same ~10 people). |
| **Believability** | Template-based site generation + automated publishing + agent-assisted practice interaction is technically feasible with current tools. The challenge is orchestration, not invention. |
| **Inspiration** | This turns SGA's marketing team from "nuisance that calls practices for passwords" into "the thing that makes every practice look world-class without them lifting a finger." |

---

## Sprint Questions

These are the questions we need to answer — each represents a way this sprint could fail.

### SQ1: Can we reduce practice setup from months to under 2 weeks without sacrificing quality or local brand identity?
- **Risk Level**: HIGH
- **Rationale**: The current bottleneck is multi-layered — asset access (passwords, brand files, photos), custom website design decisions, GMB setup, social handle creation, CRM integration. Compressing this requires both a technical solution (templates, automation) AND a behavioral solution (practices actually providing what's needed quickly). If we can't crack the access/onboarding problem, the rest of the system doesn't matter.

### SQ2: Can a small HQ team maintain opinionated quality control across 300+ practice digital storefronts without becoming a bottleneck?
- **Risk Level**: HIGH
- **Rationale**: The team is ~10 people. At 260+ practices, that's 26+ practices per person. If every update requires human review, the system collapses. The quality control has to be programmatic — templates, brand guardrails, automated checks — with humans only intervening on exceptions. But "opinionated" quality is hard to automate; it requires judgment (e.g., "this photo doesn't feel right for a periodontist site").

### SQ3: Will practices actually engage with an agent/chatbot interface instead of ignoring it like they ignore emails and calls?
- **Risk Level**: HIGH
- **Rationale**: Practices currently view HQ marketing as a nuisance. They have 5-minute windows between patients. The hypothesis is that an always-available agent that answers immediately will get more engagement than scheduled calls and email chains. But if the agent can't resolve issues quickly or feels like "another thing to check," adoption will be zero. This is a behavioral bet.

### SQ4: Can we detect and respond to practice-level changes (staff departures, service changes) proactively rather than reactively?
- **Risk Level**: MEDIUM
- **Rationale**: Currently, when a doctor leaves a practice, nobody updates the website, social profiles, or signage until someone remembers. The cascading effect touches 7+ digital touchpoints. If we can build event-triggered workflows (staff change → website update → social update → signage notification → CRM update), we solve the "stale storefront" problem. The risk is that change signals are hard to detect — they come through random emails, hallway conversations, or not at all.

### SQ5: Can we provide executive visibility into network-wide digital health without creating a reporting burden on the HQ team?
- **Risk Level**: MEDIUM
- **Rationale**: Leadership needs to know: which practices are at baseline, which are optimized, which are falling behind. This dashboard has to be automated — pulling from live data (site performance, GMB metrics, social activity, review scores). If it requires manual data entry or reporting, it won't get maintained and becomes another abandoned initiative.
