# Product Architecture: The Practice Concierge

## System Overview

The Practice Concierge is an agent-first orchestration platform that manages digital storefronts for 260+ dental practices. The architecture has six core components connected by an event-driven orchestration layer, with three interfaces (practice-facing, HQ-facing, executive-facing).

---

## Component Architecture

### Component 1: Practice Data Hub (Single Source of Truth)
**Responsibility**: Canonical data store for every practice. One record per practice containing all structured data: name, address, team members, services, hours, specialty, brand assets (colors, logo, photos), template selection, account credentials, enrollment status, and health score.

**Interfaces**:
- IN: Data Harvester (initial population), Concierge Agent (practice-reported updates), iSolved integration (HR changes), manual HQ entry
- OUT: Template Engine (site generation), GMB Sync (listing updates), Social Publisher (content localization), Dashboard (metrics), CRM (lead routing)

**Key Design Decision**: Every external system reads FROM this hub. When data changes here, changes propagate outward. No system maintains its own copy of practice data. This eliminates the "Dr. Johnson is still on the website" problem.

**Data Model (per practice)**:
```
Practice {
  id, slug, name, legal_name
  platform: SGA | Gen4 | MODIS
  specialty: GP | Perio | FullArch | Pediatric | Multi
  address, phone, email, hours
  team: [{ name, role, bio, photo_url, start_date, end_date }]
  services: [{ name, description, is_primary }]
  brand: { primary_color, secondary_color, logo_url, photos[] }
  digital: {
    website: { url, template_id, status, last_deployed }
    gmb: { listing_id, status, completeness_score }
    social: { instagram_handle, facebook_url, status }
    crm: { system, connection_status }
  }
  enrollment: { status, invited_date, enrolled_date, method: concierge | blitz }
  health_score: { overall, website, gmb, social, reviews, leads }
  concierge: { last_interaction, interaction_count, satisfaction }
}
```

### Component 2: Data Harvester
**Responsibility**: Automated pre-population of practice profiles from public sources. Runs before any practice interaction to create the "we already know about you" experience.

**Sources**:
- Google My Business API (listing data, reviews, photos)
- Practice website scraping (team pages, service pages, about pages, contact info)
- Social media APIs (Facebook Graph API, Instagram Basic Display API)
- Directory APIs (Healthgrades, Zocdoc, Yelp)
- Internal systems (iSolved for HR data, deal pipeline for acquisition data)

**Process**:
1. Accept practice identifiers (name, address, known URLs)
2. Discover all public digital footprint
3. Extract structured data using LLM-assisted parsing
4. Reconcile conflicts (website says 5 doctors, GMB says 4 — flag discrepancy)
5. Generate Practice Profile Draft with confidence scores per field
6. Store in Practice Data Hub with `source: harvested, confidence: high|medium|low`

**Key Design Decision**: Use LLM-assisted extraction (not rigid scrapers) so the system can parse any website structure. Dental practice websites are wildly inconsistent — there's no standard HTML structure to scrape. An LLM can read any page and extract team members, services, and contact info.

### Component 3: Concierge Agent
**Responsibility**: The practice-facing conversational interface. Handles enrollment, change management, content approvals, Q&A, and proactive outreach. One instance per practice, all sharing the same model + SGA knowledge base, differentiated by practice-specific context.

**Architecture**:
```
Concierge Agent
├── Conversation Engine (LLM with SGA system prompt + practice context)
├── Knowledge Base (SGA brand standards, best practices, FAQs, policies)
├── Action Router (maps conversation intents to backend operations)
├── Practice Context Window (current practice data from Data Hub)
└── Proactive Scheduler (triggers for follow-ups, health checks, content approvals)
```

**Key Capabilities**:
- **Enrollment Flow**: Guided verification of harvested data → template selection → storefront generation → approval
- **Change Management**: Accept change reports → identify cascading impacts → schedule propagation → confirm
- **Content Approval**: Present scheduled social posts → accept/reject/modify → route to publisher
- **Q&A**: Answer practice questions using SGA knowledge base (4Front dates, benefits info, marketing best practices)
- **Proactive Outreach**: "Your website traffic increased 15% this month" / "You have 3 new Google reviews to respond to" / "We detected a staff change in iSolved — can you confirm?"

**Personality**: Sage (knowledgeable, evidence-based) + Caregiver (warm, protective). Opinionated — recommends actions with reasoning. Never bureaucratic.

**Channels**: Web chat (Phase 1) → SMS (Phase 2) → Voice (Phase 3) → Video avatar (future)

**Key Design Decision**: The agent uses a system prompt with SGA brand standards, specialty-specific best practices, and the practice's current data. This means every recommendation is contextual: a periodontist gets referral-optimization advice, a GP gets new-patient-acquisition advice. The agent doesn't just answer questions — it pushes practices toward better outcomes.

### Component 4: Template Engine
**Responsibility**: Generate and deploy practice websites from structured data + specialty templates. Handle batch operations for network-wide updates.

**Templates**:
- GP Template (appointment-booking optimized, new patient focused)
- Perio Template (referral-optimized, specialist credibility)
- Full Arch Template (transformation stories, financing, Dr. Olson content)
- Pediatric Template (family-friendly, comfort-focused)
- Multi-Specialty Template (service navigation, provider matching)

**Process**:
1. Receive practice data from Data Hub
2. Select template based on specialty
3. Populate with practice-specific content (team, services, photos, colors)
4. Generate all pages (home, about, services, team, contact, reviews)
5. Apply SEO optimization (schema markup, meta tags, sitemap)
6. Deploy to hosting infrastructure
7. Configure SSL, domain, analytics tracking

**Batch Operations**:
- Template update → regenerate all sites using that template
- Brand change → propagate across all sites
- New feature → deploy to all sites simultaneously

**Key Design Decision**: Sites are generated, not hand-built. The template is the opinion. When the template changes, all sites using it change. This is how a 10-person team manages 260+ websites — they manage 5 templates, not 260 sites.

### Component 5: Channel Sync Engine
**Responsibility**: Keep all external channels (GMB, social, directories, CRM) synchronized with the Practice Data Hub. Handle both one-time setup and ongoing synchronization.

**Channels**:
- **Google My Business**: Listing creation/update via API, weekly posts, review monitoring, photo management
- **Social Media**: Facebook and Instagram profile management, content publishing, comment monitoring
- **Directories**: Healthgrades, Zocdoc, Yelp — listing accuracy sync
- **CRM**: Lead routing, form submission handling, appointment request automation
- **Review Aggregation**: Pull reviews from all platforms, surface for response, track sentiment trends

**Sync Pattern**:
- Practice Data Hub change → Channel Sync evaluates which channels are affected → updates each channel via API → confirms sync → logs result
- External change detected (new review, competitor mention, listing edit) → Channel Sync ingests → updates Data Hub → notifies Concierge if practice action needed

**Key Design Decision**: Channel Sync is event-driven, not batch. When a doctor is removed from the Data Hub, the sync fires immediately — not on a nightly batch. This means the "Dr. Johnson left" scenario is handled in minutes, not days.

### Component 6: Network Dashboard
**Responsibility**: Executive and HQ visibility into network-wide digital health. Automated scoring, alerting, and reporting.

**Views**:
- **Executive View**: Network health score, enrollment progress, tier distribution, ROI metrics, alerts
- **HQ Marketing View**: QC queue (content/sites to review), batch operation controls, practice drill-down, content pipeline status
- **Practice Health Detail**: Per-practice scorecard with website performance, GMB metrics, social activity, review health, lead conversion

**Health Score Calculation**:
```
Practice Health Score (0-100) =
  Website (25%):  speed + mobile + SEO + freshness
  GMB (25%):      completeness + post frequency + response rate
  Social (20%):   post frequency + engagement + profile completeness
  Reviews (20%):  avg rating + volume + response rate + sentiment trend
  Leads (10%):    form conversion + response time + booking rate
```

**Alerts**:
- Practice health score drops below threshold
- Review score drops >0.5 stars in 30 days
- Staff change detected in iSolved not reflected in storefront
- Website downtime or performance degradation
- Content flagged by automated quality check

**Key Design Decision**: The dashboard reads from the Practice Data Hub and channel APIs — it never requires manual data entry. If a metric can't be automated, it doesn't go on the dashboard. This ensures the dashboard stays current without creating reporting burden.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PRACTICE CONCIERGE PLATFORM                  │
│                                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────────────┐  │
│  │ PRACTICE │    │   HQ TEAM    │    │     EXECUTIVE            │  │
│  │ Interface│    │  Interface   │    │     Dashboard            │  │
│  │ (Chat)   │    │ (Admin + QC) │    │  (Health + Metrics)      │  │
│  └────┬─────┘    └──────┬───────┘    └────────────┬─────────────┘  │
│       │                 │                          │                │
│  ┌────▼─────────────────▼──────────────────────────▼─────────────┐ │
│  │              CONCIERGE AGENT (per practice)                    │ │
│  │  ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌────────────────┐  │ │
│  │  │Conversa- │ │ Knowledge │ │ Action   │ │   Proactive    │  │ │
│  │  │tion      │ │ Base      │ │ Router   │ │   Scheduler    │  │ │
│  │  │Engine    │ │ (SGA +    │ │          │ │                │  │ │
│  │  │(LLM)    │ │ Practice) │ │          │ │                │  │ │
│  │  └──────────┘ └───────────┘ └────┬─────┘ └────────────────┘  │ │
│  └──────────────────────────────────┼────────────────────────────┘ │
│                                     │                              │
│  ┌──────────────────────────────────▼────────────────────────────┐ │
│  │                 EVENT-DRIVEN ORCHESTRATION BUS                 │ │
│  └──┬──────────┬──────────────┬───────────────┬─────────────────┘ │
│     │          │              │               │                    │
│  ┌──▼───┐  ┌──▼───────┐  ┌──▼──────────┐ ┌──▼──────────────┐    │
│  │DATA  │  │TEMPLATE  │  │CHANNEL SYNC │ │NETWORK          │    │
│  │HARV- │  │ENGINE    │  │ENGINE       │ │DASHBOARD        │    │
│  │ESTER │  │          │  │             │ │                  │    │
│  │      │  │GP / Perio│  │GMB / Social │ │Health Scores     │    │
│  │Web   │  │FullArch  │  │Directories  │ │Alerts            │    │
│  │Scrape│  │Pediatric │  │CRM / Reviews│ │Reporting         │    │
│  └──┬───┘  └──┬───────┘  └──┬──────────┘ └──┬───────────────┘    │
│     │         │              │                │                    │
│  ┌──▼─────────▼──────────────▼────────────────▼──────────────────┐│
│  │              PRACTICE DATA HUB (Single Source of Truth)        ││
│  │   260+ Practice Records │ Team │ Brand │ Digital │ Health     ││
│  └───────────────────────────────────────────────────────────────┘│
│                                                                    │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │                    EXTERNAL INTEGRATIONS                       ││
│  │  Google APIs │ Meta APIs │ iSolved │ CRM │ Hosting │ CDN      ││
│  └────────────────────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────────────────────┘
```

---

## Architecture Decision Records (ADRs)

### ADR-1: Agent-Per-Practice vs. Shared Agent
**Decision**: Deploy one logical agent instance per practice (shared model, practice-specific context).
**Context**: Could have one agent serving all practices (shared context, practice identified by login) or one dedicated agent per practice (always aware of its practice's data).
**Rationale**: Per-practice agents feel personal ("my concierge knows my practice") and eliminate context-switching errors. The cost is minimal — the model is shared, only the system prompt and context window differ per practice. At 260+ practices, the overhead is context storage, not compute.
**Consequences**: Practice data must be efficiently loadable into the agent's context. Need a practice context service that prepares the prompt for each practice on each interaction.

### ADR-2: Generated Sites vs. CMS-Managed Sites
**Decision**: Generate static/JAMstack sites from templates + data, not CMS-managed dynamic sites.
**Context**: Could use WordPress Multisite, Webflow, or similar CMS for each practice. Or generate sites from a custom template engine.
**Rationale**: Generated sites are faster (no database queries), cheaper to host (static hosting/CDN), more secure (no CMS vulnerabilities), and easier to batch-update (re-generate from template). CMS adds overhead per site — 260 WordPress instances is a maintenance nightmare. The Template Engine owns the templates; the Practice Data Hub owns the data; the output is a deployable site.
**Consequences**: Content updates require re-generation and re-deployment. The pipeline must be fast enough that a change in the Data Hub results in a live site update within minutes. Must build a preview system for practice approval before deployment.

### ADR-3: Event-Driven vs. Batch Synchronization
**Decision**: Event-driven synchronization with batch as fallback.
**Context**: Channel Sync could run on a schedule (nightly batch) or react to events (data change → immediate sync).
**Rationale**: The "Dr. Johnson left" scenario requires immediate propagation. If we wait for a nightly batch, the practice's website shows a departed doctor for up to 24 hours. Event-driven sync handles this in minutes. Batch sync remains as a safety net — a nightly reconciliation that catches anything the event system missed.
**Consequences**: Need a reliable event bus (message queue) that guarantees delivery. Must handle API rate limits gracefully — GMB and social APIs throttle requests. Build retry logic with exponential backoff.

### ADR-4: Blitz Deploy as Safety Net
**Decision**: Auto-deploy default storefronts for non-engaged practices after 30 days.
**Context**: Some practices will never engage with the Concierge. Without a fallback, these practices remain in their current (often broken) digital state indefinitely.
**Rationale**: A clean, generic storefront built from scraped data is better than a broken or nonexistent one. The blitz deploy ensures 100% network coverage regardless of practice engagement. The 30-day window gives practices time to engage, and the notification ("I went ahead and launched your upgrade") is an invitation to customize later.
**Consequences**: Blitz-deployed sites may have inaccuracies (scraped data isn't always current). Must flag these as "auto-deployed — unverified" in the dashboard. The concierge continues to invite these practices to verify and customize.

### ADR-5: Opinionated Agent with Reasoning
**Decision**: The Concierge always provides recommendations WITH reasoning, not just options.
**Context**: Could build the agent as a neutral tool ("what would you like?") or as an opinionated advisor ("I recommend X because Y").
**Rationale**: SGA's competitive advantage is expertise. The brand personality is Sage + Caregiver. An agent that says "here are 5 options, you choose" adds no value — the practice doesn't know which is best. An agent that says "I recommend Template A because periodontist sites convert 23% better with referral-optimized layouts" adds immense value and reduces decision fatigue. This is what SGA means by "forceful opinion."
**Consequences**: The knowledge base must include conversion data, industry benchmarks, and specialty-specific best practices to support recommendations. Recommendations must be accurate — a wrong recommendation erodes trust permanently. Build a recommendation quality feedback loop.

---

## Technology Recommendations

| Component | Recommended Stack | Rationale |
|-----------|------------------|-----------|
| Practice Data Hub | PostgreSQL + Redis cache | Relational data with fast reads. Practice records are structured and relational (team → practice, services → practice). Redis for hot path (agent context loading). |
| Data Harvester | Python + BeautifulSoup/Playwright + Claude API | LLM-assisted extraction handles any website structure. Playwright for JavaScript-rendered pages. Claude for parsing unstructured content into structured data. |
| Concierge Agent | Claude API + custom orchestration layer | Claude for conversation + reasoning. Custom layer for action routing, practice context injection, and proactive scheduling. Could use Claude's tool_use for direct backend actions. |
| Template Engine | Next.js/Astro + Tailwind + MDX | Static site generation with component-based templates. Tailwind for consistent design system. MDX for content blocks. Deploy to Vercel/Netlify/Cloudflare Pages. |
| Channel Sync | Node.js workers + message queue (BullMQ/SQS) | Event-driven workers per channel. Message queue for reliability and retry. API integrations: Google My Business API, Meta Graph API, Yelp Fusion API. |
| Network Dashboard | Next.js + Chart.js/Recharts + real-time subscriptions | Server-rendered dashboard with real-time updates. PostgreSQL views for aggregated metrics. WebSocket for live alerts. |
| Hosting | Cloudflare Pages (sites) + Vercel (dashboard/agent) + AWS (data/sync) | Static site hosting at edge (fast, cheap, scalable). Application hosting on Vercel. Data and worker infrastructure on AWS. |

---

## Implementation Roadmap

### Phase 1: MVP (Days 1-30)
- Practice Data Hub schema + seed with SGA practice list
- Data Harvester for top 50 practices (public web scraping + GMB)
- Concierge Agent core (enrollment conversation flow, data verification)
- 2 website templates (GP + one specialty)
- Template Engine (generate + deploy sites from data)
- Basic dashboard (enrollment progress, practice list)
- **Milestone**: 20-30 practices enrolled and live through Concierge

### Phase 2: Scale (Days 31-60)
- Remaining 3 templates
- Channel Sync: GMB + Facebook + Instagram
- Blitz deploy for non-engaged practices
- Concierge: change management flows
- Dashboard: health scores, alerts
- iSolved integration for staff change detection
- **Milestone**: All 260+ practices at baseline (enrolled or blitz-deployed)

### Phase 3: Curation (Days 61-90)
- Hub-and-spoke social content automation
- Concierge: content approval flows, proactive outreach
- Review management and response automation
- Dashboard: full KPI suite, ROI metrics
- CRM integration for lead routing
- **Milestone**: 365-day curation engine operational (Stage 2 / Target B)

### Phase 4: Intelligence (Days 91-120)
- Concierge: voice channel
- A/B testing framework for templates
- Predictive health scoring (which practices are trending down?)
- Network-wide content performance analytics
- Automated optimization recommendations
- **Milestone**: Self-improving system with data-driven optimization
