# Research Inspiration

## 1. SOCi Genius Agents — The Agentic Workforce Model
**Source**: SOCi (multi-location marketing platform)
**What they did**: Deployed 150,000+ AI "Genius Agents" across multi-location brands. Each agent is trained on brand guidelines, then cloned and deployed per location. Three agent types: Search (listing optimization), Reputation (review management), Social (content publishing). Results: 90% of locations ranking in local 3-pack, estimated 750K hours of manual work saved.
**What's relevant**: The "train once, clone per location" architecture. SGA could train an agent on SGA brand standards + practice-specific data, then deploy one per practice. The per-location agent handles ongoing curation, the central orchestrator ensures brand compliance.
**Gap**: SOCi is marketing-only — no website generation, no CRM, no lead gen funnels, and no practice-facing conversational interface. It's a tool the HQ team uses, not a tool practices interact with.

## 2. Franchise Digital Playbooks — McDonald's, Marriott, RE/MAX
**Source**: Multi-industry franchise operations
**What they did**: Franchises solved this decades ago with rigid template systems. McDonald's doesn't let franchisees design their own signage. Marriott doesn't let properties build custom websites. RE/MAX provides agent sites from structured data (name, photo, listings, market area). The pattern: central brand system → structured data input → auto-generated storefront → limited local customization within guardrails.
**What's relevant**: The "opinionated template" approach. SGA's 90/10 split (90% of practices don't care, 10% care deeply) maps perfectly to the franchise model — the default is a great storefront that requires zero practice input. The 10% get a "premium customization" path with guardrails.
**Gap**: Franchise models are rigid. They don't preserve local brand identity (which SGA explicitly values — practices keep their names). Need a hybrid: franchise-level consistency with local brand preservation.

## 3. AI Receptionist DSO Case Study — 120 Locations
**Source**: Industry case study (Savvy Agents / TrueLark)
**What they did**: A 120-location DSO deployed AI receptionists that handle patient calls, scheduling, insurance questions 24/7. Result: $13.9K additional revenue per month per location, 68% booking conversion rate. The AI agents integrated with existing PMS systems and handled the "5-minute window" problem — patients get immediate responses instead of voicemail.
**What's relevant**: Proves the "always-available agent" model works in dental. Practices adopted it because it solved a problem they care about (missed calls = missed revenue). The adoption pattern: it worked, it made them money, they kept using it. The SGA practice-facing agent needs the same "obvious value" proposition.
**Gap**: These are patient-facing. The practice-facing agent for HQ operations (managing digital storefront, reporting changes, approving content) doesn't exist in the market yet. SGA would be pioneering this.

## 4. Microsoft Copilot Studio — Enterprise Agent Builder
**Source**: Microsoft
**What they did**: Natural-language creation of AI agents for employee-facing scenarios. Dow Chemical used it to transform freight invoicing across global operations. The pattern: define the agent's skills, connect it to enterprise systems (CRM, ERP, etc.), deploy to employees who interact conversationally. The agent routes actions to the right backend system.
**What's relevant**: The architectural pattern — the agent is an abstraction layer over multiple systems. Instead of training practice managers on 5 different tools (CMS, GMB, social scheduler, CRM, analytics), they talk to one agent that orchestrates all of them. This is exactly what SGA needs.
**Gap**: Copilot Studio is horizontal. Would need significant customization for dental/DSO workflows.

## 5. Webflow/Dentex — Template Dental Sites
**Source**: Dental website template market
**What they did**: Pre-designed website templates for dental practices with pages for services, team bios, testimonials, FAQs, scheduling. A practice provides their name, photos, and colors → site is generated. Webflow enables visual customization within guardrails.
**What's relevant**: Template-based site generation is a solved UX problem. The technology exists. The innovation isn't "can we build templates" — it's "can we populate them automatically from structured data captured via conversational AI, at the moment of acquisition, without the practice doing any design work."
**Gap**: These are one-at-a-time, practice-initiated tools. Nobody has built "bulk enrollment" — take 260 practices and migrate them all into a template system with auto-populated data.

## 6. CareStack / OS Dental — Unified DSO Operating Systems
**Source**: Dental-specific platforms
**What they did**: Cloud-based practice management systems designed for multi-location dental operations. CareStack: single login, multi-location management, centralized reporting. OS Dental positions as "unified enterprise operating system." Both eliminate multi-vendor complexity.
**What's relevant**: The "single system" aspiration. SGA's practices currently use fragmented tools. A unified platform that handles PMS + marketing + digital presence would be the dream state. The dashboard/visibility layer that executives want is built into these systems.
**Gap**: None of these include website generation, social media management, or AI agent interfaces. They're clinical/operational, not marketing/digital. SGA needs the marketing equivalent.

## 7. Yext — Structured Data as Source of Truth
**Source**: Yext (digital presence management)
**What they did**: Acts as a "single source of truth" for location data, syncing across 200+ platforms (Google, Apple Maps, Yelp, etc.). When you update data in Yext, it propagates everywhere. The architectural insight: one canonical data record per location → automated distribution to all channels.
**What's relevant**: The "single source of truth" pattern solves the cascading update problem (doctor leaves → update everywhere). If SGA has one structured data record per practice, changes propagate to website, GMB, social, directories, etc. automatically. This is the core of the change management solution.
**Gap**: Yext is listings/data focused. Doesn't handle website generation, content publishing, or conversational interfaces.

## 8. Hub-and-Spoke Content Model (SGA's Own Strategy)
**Source**: SGA's social growth strategy document
**What they did**: Designed a content pyramid where one central production (e.g., Dr. Olson YouTube video) yields 25-35 pieces across platforms. Central SGA creates assets; local practices customize and post. Posting cadence: 2 long-form/week, 4-5 Shorts/week, 3-4 posts/platform/week.
**What's relevant**: SGA already has the content strategy designed. The missing piece is the distribution and localization automation. If the content pipeline can auto-generate localized versions (swap in practice name, local photos, local offers) and auto-publish, the content creation isn't the bottleneck — the last-mile customization and publishing is.
**Gap**: Strategy exists on paper but isn't operationalized. No automation connects central content to local practice social channels.

---

## Inspiration Synthesis

The winning solution likely combines:
- **SOCi's "train once, clone per location" agent architecture** (Inspiration 1)
- **Franchise template-first approach** with local brand preservation (Inspiration 2)
- **The "obvious value" adoption pattern** from AI receptionists (Inspiration 3)
- **Agent as abstraction layer** over multiple systems (Inspiration 4)
- **Single source of truth** data model with automated propagation (Inspiration 7)
- **SGA's own hub-and-spoke content model**, operationalized (Inspiration 8)
