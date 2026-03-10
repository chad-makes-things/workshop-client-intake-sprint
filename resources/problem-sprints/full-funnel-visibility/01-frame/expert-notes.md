# Expert Notes

## Domain Context: SGA Dental Partners

SGA Dental Partners is a Thurston Group-backed DSO operating 260+ dental practices across multiple states. The organization is in an active consolidation phase, merging three previously independent platforms (SGA original, Gen4 Dental Partners, MODIS Dental Partners) under a single umbrella.

### The Technology Landscape

**Current State: 75 individual pieces of 3rd-party technology**

These fall into approximately 15 core categories:

| Category | Examples | Data It Holds |
|----------|----------|---------------|
| Practice Management Software (PMS) | Multiple vendors across practices | Appointments, production, patient records, treatment plans |
| Phone Systems | Multiple standard vendors | Call logs, answer rates, call duration |
| AI Phone Agents | Sierra, Agent Agnes | Conversation transcripts, intent detection, automated actions |
| Advertising Platforms | Google Ads, Meta, LSAs | Ad spend, impressions, clicks, leads generated |
| Website / Forms | Various CMS, form tools | Form submissions, website traffic |
| Scheduling / Booking | Often part of PMS, sometimes separate | Appointment bookings, confirmations, cancellations |
| CRM / Lead Capture | **MISSING — biggest gap** | Should hold: lead records, contact history, lifecycle stage |
| Patient Communication | Text/email platforms | Appointment reminders, confirmations, follow-ups |
| Conversation Intelligence | AI call analysis tools | Call scoring, conversion tracking, script adherence |
| Revenue / Billing | Part of PMS + separate financial systems | Production, collections, A/R, insurance claims |
| HR / Staffing | iSolved, others | Staff records, scheduling, payroll |
| Review Management | Reputation.com, others | Google reviews, patient satisfaction |
| Social Media | Various platforms | Social engagement, content performance |
| Reporting / BI | Excel, various dashboards | Manual reports, fragmented metrics |
| File Storage / Communication | Email, Slack, Dropbox, etc. | Practice-HQ communication |

### Key Expert Insights (from problem statement)

**1. "The biggest problem is getting visibility into that full system"**
- Systems aren't connected
- Except for a CRM solution (which doesn't exist yet), the rest of the tooling is fairly standardized
- The CRM/lead capture solution is the single biggest missing standard piece

**2. "We need to have visibility in live time into those steps in that funnel"**
- Not just a dashboard — need to see revenue flows through the patient funnel lens
- Executive level visibility to make live decisions
- Downstream automation triggered by funnel insights

**3. "If you're looking at a patient funnel and you're not tracking to production goals, it's not hard to say 'well, you've had a dip in phone answering'"**
- The analysis layer could be relatively straightforward IF the data is connected
- The hard part is data integration, not data analysis
- AI sitting on top of connected data could identify problems before humans notice them

**4. "20 minutes a day in 10 tools — it's not time I'll spend"**
- Practice staff capacity is the binding constraint
- Any solution must REDUCE tool count, not add to it
- The unified view must be the thing that drives action and redirects prioritization

**5. "We have AI answering systems right now that we're using"**
- AI phone agents are already adopted (not theoretical)
- Agent Agnes plugs into practice software and triggers actions based on conversations
- This is an existing capability that needs to be surfaced in the funnel view, not built from scratch

**6. "30 days post month-end is still a very difficult behavior"**
- The current reporting cycle means problems are discovered a month after they happen
- By the time leadership knows a practice is struggling, the damage is done
- Real-time visibility is not a nice-to-have — it's the difference between proactive and reactive management

**7. "How do you even expose that in a similar mirror without having another 10 tools to integrate with?"**
- The concern is that solving the visibility problem creates MORE tool fragmentation
- The solution must expose data through the existing ecosystem, not alongside it
- Integration-first, not portal-first

### Related Sprint: Digital Storefront at Scale

The [Digital Storefront sprint](../digital-storefront-at-scale/) addressed the marketing/digital presence layer — how practices appear online, attract patients, and manage their brand. The solution was "The Practice Concierge" — an agent-first orchestration platform.

**Connection to this sprint**: The Practice Concierge handles the TOP of the funnel (attracting patients). This sprint handles EVERYTHING FROM LEAD CAPTURE THROUGH PRODUCTION — the operational funnel that turns marketing leads into revenue.

**Potential synergy**: If both sprints produce solutions, they share the same data layer — lead source attribution from the storefront sprint feeds directly into the funnel visibility system. The Practice Concierge's agent interface could be extended to surface funnel data, or vice versa.

### Industry Context: DSO Funnel Management

The DSO industry broadly struggles with the same fragmentation problem:
- Most practices were independent businesses that chose their own tech stacks
- Post-acquisition, DSOs inherit dozens of different PMS, phone, and marketing systems
- Platform standardization takes years and creates massive change management resistance
- The "middleware" approach (integration layer over existing tools) is increasingly preferred over "rip and replace"

**Key pattern from successful DSO tech consolidation:**
1. Start with a data lake that aggregates from existing systems (don't replace them)
2. Build the visibility layer on top of the data lake (dashboards, alerts)
3. Add the intelligence layer (AI analysis, anomaly detection, recommendations)
4. Add the action layer (automated workflows, agent engagement)
5. Over time, selectively replace underperforming tools as the data layer proves which ones matter

**Agent Agnes as a reference model**: The mention of Agent Agnes is significant — it's a phone call agent that integrates with practice management software and triggers actions based on conversations. This is exactly the pattern the solution needs: AI agents that sit on top of existing systems, understand context, and take actions. The question is whether to build or buy each piece.
