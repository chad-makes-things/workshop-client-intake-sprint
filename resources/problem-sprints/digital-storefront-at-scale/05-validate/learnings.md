# Sprint Learnings: Digital Storefront at Scale

## Sprint Summary
- **Problem**: 260+ dental practices need modern digital storefronts (setup + ongoing curation) managed by a ~10-person HQ team
- **Solution**: The Practice Concierge — an AI agent per practice that handles enrollment, change management, content approval, and Q&A, backed by a data hub, template engine, channel sync, and executive dashboard
- **Decision Path**: Target A+C (Onboarding Pipeline + Agent Interface) → Practice Concierge (30/35) selected over Network OS (27), Smart Wrapper (24), Playbook (21), Assembly Line (20)

## Key Learnings

### 1. The onboarding problem is really a data problem
The biggest insight from framing: the multi-month setup time isn't because building a website is hard. It's because getting the data (passwords, brand assets, team info, service lists) is hard. The Data Harvester + Concierge verification approach attacks this directly — pre-populate everything you can from public data, then verify with the practice in 15 minutes instead of weeks of back-and-forth.

### 2. The agent IS the product, not a feature
Early thinking positioned the agent as one feature of a larger platform. Through the sprint, it became clear the agent should be the ONLY interface practices see. Everything else (templates, sync, dashboard) is backend infrastructure. This simplifies the practice experience to one thing: "talk to your concierge."

### 3. Opinionated by default, customizable by exception
SGA's competitive advantage is expertise. The system should come in with a strong opinion on what every practice's storefront should look like, backed by data and reasoning. The 90% who don't care get a great storefront without lifting a finger. The 10% who care get guardrailed choices. Nobody gets a blank canvas.

### 4. Blitz deploy prevents the adoption trap
If the system requires practice engagement to deliver value, non-engaged practices get nothing — which is the current state. The blitz-deploy safety net (auto-deploy from scraped data after 30 days) ensures 100% network coverage regardless of practice behavior. This reframing was critical: the system must work even if practices never engage.

### 5. Stage 2 (365-day curation) is an extension, not a separate system
The Practice Concierge architecture naturally extends into ongoing curation. The same agent that handles enrollment becomes the ongoing content approval, change management, and proactive outreach channel. This was a key reason for selecting it over the Assembly Line (which solves setup but creates no ongoing relationship).

## Unresolved Questions
1. **CRM selection**: Which CRM will SGA standardize on? The Channel Sync engine needs to integrate with it, but the choice wasn't in scope for this sprint.
2. **Domain/hosting migration strategy**: Many practices have existing websites on various hosts. What's the migration path? Do we keep their domains and point to new hosting, or consolidate?
3. **Photo/brand asset quality**: The Data Harvester can find existing photos, but many practices have low-quality imagery. What's the plan for practices that need professional photography?
4. **Legal/compliance review**: Are there regulatory requirements for dental practice websites (HIPAA, state-specific disclosures) that the templates need to account for?
5. **iSolved integration depth**: How accessible is iSolved's API for detecting staff changes? This is the automated change detection channel and needs validation.

## Risks to Monitor
- **Practice adoption rate**: The behavioral bet. Target: 80%+ engagement within 60 days of invitation. If below 50%, the concierge's value proposition may need reworking.
- **Template quality at scale**: Do 5 templates serve 260+ practices well enough? May need sub-variants (e.g., GP with cosmetic focus vs. GP with family focus).
- **API rate limits**: GMB, Meta, and directory APIs all have rate limits. Channel Sync at 260+ practices simultaneously could hit throttling. Need capacity testing.
- **Data accuracy**: Scraped data will have errors. The verification step is critical — but what about blitz-deployed practices whose data was never verified?

## Next Steps
1. **Development handoff** (optional Phase 10): Map sprint artifacts to development team roles
2. **Development package** (optional Phase 11): Full specs for PM, UX, Tech Lead, Backend, Frontend
3. **Pilot batch**: Identify 20-30 practices for Phase 1 MVP testing
4. **SGA knowledge base integration**: Connect the concierge's knowledge base to SGA's existing brand standards, marketing playbooks, and operational guides
