# Goal & Sprint Questions

## Long-Term Goal

**In 30 days, SGA will have a unified patient funnel visibility system that connects data from its 15 core technology platforms across 260+ practices — enabling HQ executives to see real-time funnel health (leads generated → leads connected → leads booked → patients shown → production completed) at every practice, with an AI analysis layer that proactively identifies problems and generates prioritized action agendas for both HQ and practice staff — replacing the current 30-day-lag, 10-tool-fragmented, reactive operating model.**

### Goal Validation

| Test | Result |
|------|--------|
| **Ambition** | Moving from 30-day-lag reactive reporting across 75 disconnected tools to real-time proactive visibility across a unified layer is a fundamental operational transformation. This is ambitious. |
| **Specificity** | Clear metrics: visibility lag (30 days → real-time), tool consolidation (75 → 15 core, surfaced through 1 interface), practice staff engagement (10 tools × 20 min → 1 interface × 20 min), proactive vs. reactive (0 automated alerts → AI-generated action agendas). |
| **Believability** | The 15 core platforms exist and have APIs or export capabilities. Integration middleware (ETL + data lake + visualization layer) is well-understood technology. AI anomaly detection on structured funnel data is a solved problem. The challenge is orchestration and prioritization, not invention. |
| **Inspiration** | This turns SGA's HQ from "we find out about problems a month after they happened" into "we see problems forming in real-time and intervene before they hit the P&L." Practice staff go from "I have 10 tabs open and no idea what matters" to "here's my 20-minute agenda for today." |

---

## Sprint Questions

These are the questions we need to answer — each represents a way this sprint could fail.

### SQ1: Can we integrate data from 15 core 3rd-party platforms into a unified funnel view without building custom connectors for each one?
- **Risk Level**: HIGH
- **Rationale**: The 15 platforms include practice management software (multiple vendors), phone systems (multiple vendors), AI phone agents (Sierra, Agent Agnes), scheduling tools, advertising platforms, and payment systems. Each has different APIs, data schemas, and export capabilities. If every integration requires custom ETL work, the system takes 6-12 months to build and is fragile to maintain. The bet is that a combination of standard integration middleware (Fivetran, Airbyte, Segment) + practice management API connectors can cover 80%+ of the data sources without custom development.

### SQ2: Can an AI analysis layer reliably identify actionable problems in the funnel data — distinguishing signal from noise across 260+ practices?
- **Risk Level**: HIGH
- **Rationale**: A dip in phone answering at one practice on one Tuesday is noise. A sustained drop in phone answer rates across a region is signal. The AI layer needs to understand practice-level baselines, seasonal patterns, service-type differences (regular cleaning vs. full-arch implant), and practice-specific context to generate alerts that are actually actionable — not a firehose of false positives that gets ignored. If the alert quality is poor, HQ will stop trusting it within a week.

### SQ3: Will practice staff actually engage with a unified funnel interface when they have only ~20 minutes/day for HQ-directed activities?
- **Risk Level**: HIGH
- **Rationale**: Practice staff are already drowning in tools. Adding "one more dashboard" is a death sentence. The interface must replace existing tool engagement, not add to it. It needs to present a prioritized, actionable agenda — not raw data. The 20-minute constraint means the system has to do the thinking for them: "Here are your 3 most important actions today." If it requires training, navigation, or interpretation, adoption will be zero.

### SQ4: Can we achieve real-time (or near-real-time) funnel visibility without requiring practices to change their existing workflows or adopt new software?
- **Risk Level**: MEDIUM
- **Rationale**: Practices already have their phone systems, their PMS, their scheduling. The integration layer must sit behind these existing tools and pull data passively. If the visibility system requires practices to "log calls in a new system" or "tag leads in a new CRM," it won't happen. The question is whether the existing tools expose enough data (call logs, appointment records, production data) through APIs or exports for the integration layer to construct the funnel view without practice-level behavior change.

### SQ5: Can downstream automation (AI-generated actions, agent engagement) actually move the needle on funnel metrics — or will it create more noise than value?
- **Risk Level**: MEDIUM
- **Rationale**: The vision includes not just visibility but automation — AI recommending a phone answering service when answer rates drop, agents handling appointment confirmations, outbound re-engagement for no-shows. But automation at scale across 260+ practices with different cultures, patient populations, and service mixes is dangerous if not contextual. A one-size-fits-all automated text reminder for a full-arch implant patient is very different from one for a cleaning patient. If the automation feels generic or creates patient-experience problems, practices will revolt.

---

## How Might We (HMW) Notes

### Theme 1: Data Integration & Unification
1. HMW connect 15 different technology platforms without building 15 custom integrations?
2. HMW create a standard data model for "patient funnel" that works across different PMS vendors?
3. HMW handle the fact that different practices use different combinations of the 15 core tools?
4. HMW get phone call data (answer rates, conversation quality, outcomes) into the funnel automatically?
5. HMW pull advertising/lead gen data when SGA doesn't control the ad spend directly?

### Theme 2: Executive Visibility & Decision-Making
6. HMW give executives a single view of 260+ practice funnels without information overload?
7. HMW make the dashboard useful at multiple zoom levels — network, region, practice, individual funnel step?
8. HMW reduce the business health assessment cycle from 30+ days to real-time?
9. HMW show revenue flow through the patient funnel lens (not just volume, but dollar value at each step)?
10. HMW enable executives to drill from "there's a problem" to "here's the root cause" in under 60 seconds?

### Theme 3: AI Analysis & Proactive Intervention
11. HMW train an AI layer to distinguish actionable problems from normal variation across 260+ practices?
12. HMW create practice-specific baselines so the AI doesn't apply one standard to a 2-chair rural practice and a 12-chair metro practice?
13. HMW generate prioritized action agendas rather than raw alerts?
14. HMW ensure AI recommendations are contextual to service type (cleaning vs. full-arch)?
15. HMW make the AI learn from outcomes (did the recommended intervention actually work)?

### Theme 4: Practice Staff Experience
16. HMW give practice staff a single 20-minute interface that replaces logging into 10 separate tools?
17. HMW present actions in priority order so staff know exactly what to do first?
18. HMW make the system feel like it's removing work from practices rather than adding more?
19. HMW handle the fact that different roles (office manager, front desk, hygienist) need different views?
20. HMW build trust with practice staff who view HQ technology mandates with suspicion?

### Theme 5: Automation & Agent Engagement
21. HMW automate the low-value repetitive tasks (appointment confirmations, follow-ups) without losing the personal touch?
22. HMW route AI phone agent conversations (Sierra, Agent Agnes) into the same funnel view?
23. HMW create "baseline production" automation that handles the predictable tasks so staff focus on exceptions?
24. HMW ensure automated actions are contextually appropriate for each practice's patient population?
25. HMW measure whether automation is actually improving funnel metrics or just creating busy work?
