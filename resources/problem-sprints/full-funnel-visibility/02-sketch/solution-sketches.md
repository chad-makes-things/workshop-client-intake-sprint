# Solution Sketches

## Solution 1: "The Funnel OS" — Full-Stack Unified Platform

### Narrative
Build a custom platform from the ground up that IS the patient funnel operating system for SGA. Every piece of funnel data — leads, calls, bookings, show rates, production — flows through a single system. Executives see a real-time network dashboard. Practice staff see a daily action agenda. AI constantly monitors and generates interventions. Think of it as "Samsara for dental practice operations."

The platform has three layers:
1. **Data Layer**: Integration middleware connects to PMS, phone, ads, forms. All data normalized into a unified funnel schema. A dental data lake.
2. **Intelligence Layer**: AI sits on the data lake. Learns practice baselines. Detects anomalies. Generates prioritized alerts and recommendations. Learns from outcomes.
3. **Experience Layer**: Two interfaces — an executive dashboard (network → region → practice → funnel step drill-down) and a practice nerve center (the 20-minute daily agenda).

### Key Differentiator
Everything is purpose-built for SGA's funnel model. No compromises, no vendor limitations, no "we can't customize that." The funnel schema, the AI logic, the practice interface — all designed for dental DSO operations at 260+ scale.

### Strengths
- Complete control over every layer — data model, AI logic, interfaces
- The data layer serves as the single source of truth for the entire organization
- Can evolve as SGA's needs change without vendor dependency
- The Practice Nerve Center and Executive Dashboard share the same data and AI — consistency from top to bottom

### Risks
- Largest build effort — 3-6 months to full platform, significant engineering team required
- Integration maintenance burden (every PMS API change requires custom updates)
- If the funnel model or data schema is wrong early, everything built on top is wrong
- "Building everything" means longer time to first value

### Feasibility
- 30-day MVP: Integration for 2-3 data sources + basic executive dashboard for 10 pilot practices. No AI, no practice interface yet.
- Full platform: 6-12 months with dedicated engineering team (4-6 engineers + data engineer + designer)
- Annual maintenance: 2-3 engineers for integration upkeep + ongoing AI tuning

---

## Solution 2: "The Dental Command Center" — BI Platform + Custom Integrations

### Narrative
Don't build a platform — buy one. Take an enterprise BI tool (Domo, Looker, Power BI) and build dental-specific integrations and dashboards on top of it. The BI platform handles the visualization, alerting, and drilling. Custom integration scripts (Python ETL jobs, Fivetran connectors, or Airbyte pipelines) pull data from PMS, phone, and ad systems into a data warehouse. AI analysis runs as scheduled jobs on the warehouse data.

This is the "smart wrapper over existing infrastructure" approach. The BI platform is the executive dashboard. The data warehouse is the integration layer. The AI is a separate service that reads the warehouse and pushes alerts.

For the Practice Nerve Center, build a lightweight web app (or mobile app) that queries the same data warehouse and presents the daily agenda. It doesn't live inside the BI platform — it's a separate thin layer designed for the 20-minute use case.

### Key Differentiator
Speed to executive visibility. A BI platform with pre-built connectors can have a working dashboard in 2-4 weeks. You're buying the visualization and drilling infrastructure rather than building it.

### Strengths
- Fastest path to an executive dashboard — BI platforms are designed for exactly this
- Pre-built connectors for many common platforms (Google Ads, phone APIs, some PMS)
- Self-service analytics for HQ team (build new reports without engineering)
- Lower upfront engineering cost — data pipeline + dashboards, not a full platform
- Enterprise BI platforms handle scale (260+ practices, millions of data points) natively

### Risks
- BI platform is for viewing data, not acting on it. The practice interface and automation layer are separate builds.
- "Dashboard fatigue" — if every problem is "look at this chart," HQ team spends all day staring at dashboards instead of acting
- AI analysis is bolted on, not native — requires a separate service, separate maintenance
- Data warehouse + BI platform + custom integrations + AI service + practice web app = 4-5 separate systems to maintain
- PMS connectors may not exist pre-built — custom ETL for the long tail of dental PMS vendors

### Feasibility
- 30-day MVP: BI dashboard connected to 2-3 data sources for 10 pilot practices. Basic executive funnel view.
- Full system: 3-5 months for comprehensive integrations, AI layer, and practice-facing app
- Annual cost: BI platform license ($50K-150K) + data warehouse ($20K-60K) + engineering maintenance (2 engineers)

---

## Solution 3: "The Nerve Center" — Agent-First, Practice-Up Architecture

### Narrative
Flip the approach. Instead of starting from the executive dashboard and working down, start from the practice and work up. Build the Practice Nerve Center first — the single interface that replaces 10 tools — and let the executive dashboard emerge from the data the nerve center collects.

The Nerve Center is an AI agent (similar to the Practice Concierge from the Digital Storefront sprint) that sits at each practice and:
- Monitors phone calls (answer rates, AI agent conversations, missed calls)
- Tracks bookings and show rates from the PMS
- Identifies today's highest-priority actions (follow up on missed calls, confirm at-risk appointments, reach out to unscheduled treatment plans)
- Presents a 20-minute daily agenda to practice staff
- Reports data upstream to HQ automatically (no manual reporting)

The executive dashboard is a byproduct — the Nerve Center agents across 260+ practices collectively generate the network-level funnel data that executives need. It's bottom-up visibility, not top-down.

### Key Differentiator
The practice interface IS the data collection mechanism. You don't have a separate integration layer + separate dashboard + separate practice tool. The agent that helps the practice is the same system that feeds the executive view.

### Strengths
- Immediate practice-level value — staff get a useful tool from day one, not another reporting burden
- Data quality is higher because the practice is engaging with the data (confirming, acting on it), not just having it scraped
- Aligns with the Practice Concierge architecture from the Digital Storefront sprint — potential to merge or share infrastructure
- Adoption-first: if practices use it, data flows; if they don't, you learn immediately

### Risks
- Executive visibility depends on practice adoption — if practices don't use the Nerve Center, HQ doesn't get data
- Slower path to network-wide coverage — need to onboard practices onto the Nerve Center one by one
- The AI agent is more complex to build than a BI dashboard (conversational interface + action engine + PMS integration)
- If the 20-minute agenda isn't good enough, practices abandon it and the whole system collapses
- Doesn't solve the data integration problem architecturally — each Nerve Center instance has to connect to that practice's specific tools

### Feasibility
- 30-day MVP: Nerve Center agent connected to phone + PMS for 5 pilot practices. Daily agenda + basic reporting.
- Full system: 4-8 months for 260+ practice rollout + executive dashboard layer
- Synergy with Practice Concierge: If the Concierge from the Digital Storefront sprint is already being built, the Nerve Center could extend it rather than start from scratch.

---

## Solution 4: "The Data Mesh" — Distributed Integration with Centralized Intelligence

### Narrative
Accept that the 15 core platforms aren't going away and shouldn't be replaced. Instead, build a "data mesh" — a distributed integration architecture where each data source pushes events into a central event stream (like a Kafka pipeline or cloud event bus). Each practice's PMS, phone system, and ad platforms emit standardized events: "new lead captured," "call answered," "appointment booked," "patient checked in," "production recorded."

The central intelligence layer consumes these events in real-time. It builds and maintains the funnel view for every practice. It runs AI analysis on the event stream (not batch — real-time). When it detects an anomaly ("Practice 47 hasn't logged a new patient call in 3 hours on a Tuesday"), it generates an alert.

The executive dashboard is a real-time view of the event stream. The practice interface is a filtered view of events + AI-generated actions for that specific practice.

### Key Differentiator
True real-time. Not "updated every 15 minutes" or "refreshed hourly" — actual event-driven, sub-second visibility. And because it's event-based, adding a new data source is just "emit events in our schema" — not a new integration pipeline.

### Strengths
- Truly real-time — events flow the moment they happen
- Scalable architecture — adding practice #261 or data source #16 is just another event emitter
- Clean separation: data sources emit, intelligence layer consumes, experience layer displays
- Event-based architecture naturally supports automation (event triggers action, not batch job)
- Industry-standard patterns (event streaming, CQRS) with mature tooling

### Risks
- Most complex architecture — event streaming infrastructure requires DevOps expertise
- Many dental platforms don't have webhook or event APIs — you'd need to build pollers that simulate events from batch data
- Over-engineered for the current state? If the 15 platforms are updating data hourly (not in real-time), a batch pipeline every 15 minutes gives you 99% of the value at 30% of the complexity
- Requires practice-side integration at each location (event emitters need to run somewhere)
- Operational complexity — monitoring an event pipeline across 260+ practices is its own challenge

### Feasibility
- 30-day MVP: Event pipeline + basic dashboard for 3-5 practices with 2 data sources. Proves the event-driven pattern.
- Full system: 6-9 months. Requires dedicated platform engineering.
- Annual cost: Cloud infrastructure ($30K-80K) + platform engineering team (2-3 engineers)

---

## Solution 5: "The SGA Intranet" — Portal-First with Embedded Analytics

### Narrative
Build a single portal — call it the SGA Intranet — that is the ONE place every person in the organization goes to do their work. It's not a dashboard. It's not an analytics tool. It's the operating environment.

Executives log in and see network health. HQ ops team logs in and sees practice-level dashboards with action queues. Practice office managers log in and see their 20-minute agenda. Front desk logs in and sees today's confirmation list and call queue. Everyone sees the same data through role-appropriate views.

The intranet embeds existing tools where possible (PMS access, phone controls, scheduling) so staff don't leave the portal. Where it can't embed, it deep-links. Data flows from the embedded/linked tools into the funnel view through backend integrations.

The AI layer is the "homepage" — everyone's landing page is AI-generated based on their role, their practice's current state, and the network's current priorities.

### Key Differentiator
It's not "another tool." It replaces the concept of "tools." The intranet IS the tool. Staff don't open 10 things — they open one thing and everything is there. This directly addresses the "20 minutes, 10 tools" problem at a fundamental level.

### Strengths
- Most complete solution to the "too many tools" problem — replaces the experience of 10 tools with one
- Role-based views mean every stakeholder gets exactly what they need
- AI-generated homepage means the system does the thinking, not the user
- "Where do I go to do my job?" has one answer: the SGA Intranet
- Natural home for the Practice Concierge (Storefront sprint) — it could live inside the Intranet

### Risks
- Largest UX challenge — building an intranet that's actually good (most intranets are where productivity goes to die)
- If it tries to do everything, it does nothing well
- Embedding external tools is technically fragile (iframes break, SSO fails, mobile doesn't work)
- Practice adoption requires changing deeply ingrained habits — "I go to [PMS] to do my scheduling" → "I go to the SGA Intranet"
- The intranet becomes a bottleneck — if it's down, nobody can work
- Longest time to full value across all three layers

### Feasibility
- 30-day MVP: Portal with executive dashboard view + basic practice view for 10 pilot practices. Links (not embeds) to existing tools.
- Full system: 9-12 months for role-based views, embedded tools, AI homepage, 260+ practice rollout
- Annual cost: Platform hosting ($40K-100K) + engineering team (4-6 engineers) + design team (2 designers)
