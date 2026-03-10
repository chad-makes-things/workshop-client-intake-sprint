# Epic: The Practice Concierge — Agent-First Digital Storefront Platform

**Epic ID**: PC-001
**Owner**: Product Manager
**Sprint**: Digital Storefront at Scale
**Created**: 2026-03-10
**Status**: Draft

---

## Epic Description

Build an AI-powered concierge platform that enables SGA Dental Partners to enroll 260+ practices into best-in-class digital storefronts through a conversational agent interface, compress setup time from months to under 2 weeks, and provide ongoing change management and executive visibility — all managed by the existing ~10-person HQ marketing team.

The platform harvests public data before any practice interaction, guides practices through verification and template selection via chat, auto-deploys storefronts for non-engaged practices, and propagates changes across all digital channels in real time when staff or services change.

### Business Objectives

- Reduce practice digital storefront setup from 3+ months to under 2 weeks
- Achieve 100% network coverage (260+ practices) within 60 days
- Maintain opinionated quality control without increasing HQ headcount
- Provide real-time executive visibility into network-wide digital health
- Establish the ongoing engagement channel for 365-day curation (Stage 2)

### Success Metrics

| Metric | Current State | Target (30 days) | Target (60 days) |
|--------|--------------|-------------------|-------------------|
| Avg. setup time per practice | 3+ months | < 2 weeks | < 1 week |
| Practices with live storefront | ~40% functional | +20-30 enrolled | 100% coverage |
| HQ hours per practice setup | 15-20 hrs | < 2 hrs | < 1 hr |
| Practice engagement rate (concierge) | N/A | 60%+ response | 70%+ response |
| Network health score (avg) | ~34/100 | 55/100 | 70/100 |

---

## User Stories

### Epic 1: Data Harvesting & Pre-Population

#### US-1.1: Automated Practice Data Harvesting

**As** the HQ marketing team,
**I want** the system to automatically harvest public data for each practice before any outreach,
**So that** the first practice interaction feels personalized ("you already know us") rather than burdensome ("tell us everything").

**Acceptance Criteria:**

```
Given a practice identifier (name + address + known URLs)
When the Data Harvester runs for that practice
Then the system creates a Practice Profile Draft containing:
  - Practice name, address, phone, hours
  - Team members found (names, roles, photos if available)
  - Services listed on current website
  - Google My Business status (claimed/unclaimed, completeness score, avg rating, review count)
  - Social media handles and last activity date
  - Current website URL and screenshot
  - Identified gaps (missing GMB, no social, outdated website)
And each field has a confidence score (high/medium/low) and source attribution
```

```
Given a practice with conflicting data across sources (e.g., website lists 5 doctors, GMB lists 4)
When the Data Harvester reconciles the data
Then discrepancies are flagged with both values and sources
And the higher-confidence source is used as the default
And the discrepancy is queued for verification during enrollment
```

```
Given a practice with a JavaScript-rendered website
When the Data Harvester processes the site
Then it uses headless browser rendering to extract content
And LLM-assisted parsing handles non-standard HTML structures
```

**Story Points**: 8
**Priority**: P0 (blocks enrollment)

#### US-1.2: Batch Harvest for Network Seeding

**As** the HQ marketing team,
**I want** to trigger a batch harvest across all 260+ practices in the SGA network,
**So that** we have pre-populated profiles for every practice before sending enrollment invitations.

**Acceptance Criteria:**

```
Given the SGA practice master list (name, address, known URLs for each practice)
When a batch harvest is triggered
Then the system queues all practices for harvesting
And processes them respecting API rate limits (GMB, social platforms)
And provides a progress dashboard showing: queued, processing, completed, failed
And completes the full batch within 72 hours
```

```
Given a harvest failure for a specific practice (website down, no GMB listing found)
When the failure occurs
Then the system logs the failure reason
And creates a partial profile with available data
And flags the practice for manual review
And continues processing remaining practices without interruption
```

**Story Points**: 5
**Priority**: P0

---

### Epic 2: Practice Enrollment Invitation

#### US-2.1: Personalized Enrollment Invitation

**As** the HQ marketing team,
**I want** to send personalized enrollment invitations to practices in configurable batches,
**So that** practices receive a compelling, low-friction entry point to the concierge.

**Acceptance Criteria:**

```
Given a batch of 20-30 practices selected for enrollment
When the enrollment invitation is triggered
Then each practice receives an email containing:
  - Practice name in the greeting
  - A preview of harvested data ("We found your Google listing, Facebook page, and current website")
  - A direct link to their concierge chat session
  - A "15 minutes" time commitment expectation
And an SMS notification is sent with a shortened link
And each invitation is tracked (sent, opened, clicked, enrolled)
```

```
Given a practice that has not clicked the invitation link after 7 days
When the follow-up window triggers
Then the system sends a reminder with a different hook
And the reminder references a specific data point ("Your Google listing is missing 3 key fields — I can fix that in 5 minutes")
```

```
Given a practice that has not engaged after 30 days of invitation
When the blitz-deploy window triggers
Then the practice is queued for auto-deployment (see US-7.1)
And the concierge sends a final "I went ahead and launched your upgrade" notification
```

**Story Points**: 5
**Priority**: P0

---

### Epic 3: Concierge Welcome & Data Verification

#### US-3.1: Concierge Chat — First Interaction

**As** a practice office manager,
**I want** to open the concierge and immediately see what it already knows about my practice,
**So that** I can verify and correct data in a few minutes rather than entering everything from scratch.

**Acceptance Criteria:**

```
Given a practice user opens the concierge chat for the first time
When the chat interface loads
Then the concierge greets them by practice name
And presents the Practice Profile Draft in digestible chunks:
  1. Team members found (with an ask to confirm, add, or remove)
  2. Current website screenshot and status
  3. Google Business Profile status and review summary
  4. Social media handles found
And each chunk has a "Looks right" / "Needs changes" response option
And the entire verification flow completes in under 10 minutes
```

```
Given the practice reports a correction ("Dr. Miller left 6 months ago")
When the correction is submitted via chat
Then the concierge acknowledges the change immediately
And updates the Practice Data Hub record
And identifies all downstream impacts ("I'll remove Dr. Miller from your new website, Google listing, and social bios")
And asks follow-up questions if needed ("Do you have a replacement provider to add?")
```

```
Given the practice provides a new team member photo
When the photo is uploaded via the chat interface
Then the system accepts JPG, PNG, HEIC formats up to 10MB
And the photo is stored and associated with the team member record
And the concierge confirms: "Got it — Sarah's photo is saved. I'll add her to your new site."
```

**Story Points**: 13
**Priority**: P0

#### US-3.2: Concierge Personality & Tone

**As** a practice interacting with the concierge,
**I want** the agent to feel knowledgeable and warm — not robotic or bureaucratic,
**So that** I trust its recommendations and want to continue using it.

**Acceptance Criteria:**

```
Given any concierge interaction
When the agent responds
Then the tone reflects Sage (knowledgeable, evidence-based) + Caregiver (warm, protective) archetypes
And recommendations include reasoning ("I recommend this because...")
And the agent never uses corporate jargon or passive voice
And responses are concise enough to read in under 30 seconds
```

```
Given a practice asks a question the concierge cannot answer
When the question is outside the agent's capability
Then the concierge says so honestly ("That's outside what I can handle directly")
And routes to the appropriate HQ team member with context
And follows up with the practice once the question is resolved
```

**Story Points**: 5
**Priority**: P1

---

### Epic 4: Template Selection & Site Generation

#### US-4.1: Specialty-Based Template Recommendation

**As** a practice going through enrollment,
**I want** the concierge to recommend a website template based on my specialty with clear reasoning,
**So that** I get an optimized site without needing to be a web design expert.

**Acceptance Criteria:**

```
Given a practice with a known specialty (GP, Perio, Full Arch, Pediatric, Multi-Specialty)
When the template selection step is reached
Then the concierge presents 2-3 template previews populated with the practice's real data
And each preview shows both mobile and desktop views
And the concierge recommends one template with specific reasoning
  (e.g., "Periodontist sites convert 23% better with referral-optimized layouts")
And all template options are SGA-approved (guardrailed decisions)
```

```
Given a practice selects a template
When the selection is confirmed
Then the system generates the full site within 5 minutes:
  - Home, About, Services, Team, Contact, Reviews pages
  - SEO optimization (schema markup, meta tags, sitemap)
  - Mobile-responsive layout
  - Practice-specific colors (from SGA-approved palette)
  - All verified team members, services, and contact info populated
And the concierge presents a preview link for the practice to review
```

```
Given a practice wants to customize beyond the template defaults
When they request changes (color, hero image, content emphasis)
Then the concierge presents SGA-approved options for that element
And applies the selection in real time to the preview
And never allows changes that violate SGA brand standards
```

**Story Points**: 13
**Priority**: P0

#### US-4.2: Site Generation Engine

**As** the system,
**I want** to generate static/JAMstack sites from templates + Practice Data Hub records,
**So that** sites are fast, secure, cheap to host, and batch-updatable.

**Acceptance Criteria:**

```
Given a practice record in the Data Hub with verified data and a selected template
When site generation is triggered
Then the Template Engine produces a deployable static site
And the site scores 90+ on Google PageSpeed Insights (mobile)
And the site includes valid schema markup for LocalBusiness + Dentist
And the site is deployed to CDN hosting with SSL within 10 minutes
And a unique URL is assigned (practice-slug.sgadental.com or custom domain)
```

```
Given a template is updated by the HQ team
When a batch regeneration is triggered
Then all sites using that template are regenerated with the updated template
And a preview is generated for each affected site before deployment
And deployment completes within 2 hours for the full network
```

**Story Points**: 13
**Priority**: P0

---

### Epic 5: Full Storefront Review & Approval

#### US-5.1: Practice Review & Approval Flow

**As** a practice completing enrollment,
**I want** to review my complete digital storefront package before it goes live,
**So that** I feel ownership over what represents my practice online.

**Acceptance Criteria:**

```
Given a practice has a generated storefront ready for review
When the concierge presents the review
Then the package includes:
  - Full website preview (all pages, mobile + desktop)
  - Google Business Profile preview (updated listing)
  - Social media profile previews (cover images, bios, first 4 scheduled posts)
  - Lead capture flow preview (contact form, thank-you email)
And the practice can approve all at once or flag specific items for changes
```

```
Given a practice flags a change during review
When the change is submitted
Then the concierge applies the change and regenerates the preview
And the updated preview is available within 2 minutes
And the practice can re-review and re-approve
```

```
Given a practice has approved their storefront
When approval is submitted
Then the HQ creative team receives a QC notification
And the QC team has a 24-hour window to flag issues
And if no flags are raised within 24 hours, deployment proceeds automatically
And if a flag is raised, the concierge notifies the practice of the delay with reasoning
```

**Story Points**: 8
**Priority**: P0

---

### Epic 6: Go-Live & Activation

#### US-6.1: Automated Deployment & Activation

**As** a practice that has approved their storefront,
**I want** everything to go live seamlessly with a confirmation of what was deployed,
**So that** I know my digital presence is active and working.

**Acceptance Criteria:**

```
Given a practice's storefront has passed QC review
When deployment is triggered
Then the following are activated:
  - Website deployed and accessible at the assigned URL
  - Google Business Profile updated and verified
  - Social media profiles refreshed with first week of content scheduled
  - Contact forms active and connected to CRM
  - Auto-response emails enabled
And the concierge sends a confirmation message listing everything that is now live
And the practice record in the Data Hub is updated to status: "live"
```

```
Given a practice has been live for 7 days
When the follow-up window triggers
Then the concierge sends a performance summary:
  - Website visitors this week
  - Appointment requests received
  - Comparison to similar specialty practices in the network
And the message reinforces the ongoing value of the concierge
```

**Story Points**: 5
**Priority**: P0

#### US-6.2: Domain Migration Support

**As** a practice with an existing custom domain,
**I want** my new site to be accessible at my current domain,
**So that** I do not lose search engine rankings or patient bookmarks.

**Acceptance Criteria:**

```
Given a practice has an existing domain (e.g., smithfamilydental.com)
When domain migration is requested
Then the system provides DNS configuration instructions
And sets up proper 301 redirects from old URL paths to new paths
And monitors for crawl errors for 30 days post-migration
And the concierge walks the practice through the process conversationally
```

**Story Points**: 5
**Priority**: P1

---

### Epic 7: Blitz Deploy for Non-Engaged Practices

#### US-7.1: Automated Storefront for Non-Engaged Practices

**As** the HQ marketing team,
**I want** practices that do not engage within 30 days to automatically receive a baseline storefront,
**So that** 100% of the network has a functional digital presence regardless of practice engagement.

**Acceptance Criteria:**

```
Given a practice has not engaged with the concierge 30 days after invitation
When the blitz-deploy trigger fires
Then the system generates a default storefront using:
  - Harvested data (unverified)
  - Best-guess template based on detected specialty
  - SGA brand defaults (colors, layout)
And the site is flagged as "auto-deployed — unverified" in the dashboard
And the concierge sends a notification: "I went ahead and launched your upgrade"
And the notification invites the practice to customize later
```

```
Given a blitz-deployed practice later engages with the concierge
When the practice initiates a conversation
Then the concierge presents the current auto-deployed storefront
And walks them through verification and customization
And the practice status upgrades from "auto-deployed" to "enrolled" upon completion
```

**Story Points**: 8
**Priority**: P1 (Phase 2 delivery, but design in Phase 1)

---

### Epic 8: Change Management

#### US-8.1: Staff Change via Concierge

**As** a practice office manager,
**I want** to report a staff change to the concierge and have it propagate everywhere automatically,
**So that** I do not have to remember and manually update 7+ digital touchpoints.

**Acceptance Criteria:**

```
Given a practice reports "Dr. Johnson is leaving at the end of the month"
When the change is submitted via concierge chat
Then the concierge responds with a complete impact list:
  - Website team page and provider bios
  - Google Business Profile providers list
  - Social media bios
  - Active ad campaigns featuring the provider
  - Draft social post welcoming replacement (when ready)
  - Reminder list for local actions (in-office signage, referring practices)
And the concierge asks for a scheduled date for the changes
And on the scheduled date, all digital updates execute automatically
And the concierge confirms completion: "All done — Dr. Johnson has been removed from everything"
```

```
Given an iSolved HR system integration is active
When a staff termination or new hire is detected in iSolved
Then the system creates a change event in the Data Hub
And the concierge proactively reaches out to the practice:
  "I noticed a staff change in your HR system — can you confirm the details?"
And waits for practice confirmation before propagating changes
```

**Story Points**: 13
**Priority**: P1 (Phase 2, but architecture designed in Phase 1)

#### US-8.2: Hours & Service Changes

**As** a practice office manager,
**I want** to report changes to hours or services and have them update everywhere,
**So that** patients always see accurate information.

**Acceptance Criteria:**

```
Given a practice reports updated office hours via the concierge
When the change is confirmed
Then the system updates:
  - Website contact page and footer
  - Google Business Profile hours
  - Directory listings (Healthgrades, Zocdoc, Yelp)
And the concierge confirms all updates within 15 minutes
```

**Story Points**: 5
**Priority**: P1

---

### Epic 9: HQ Dashboard & Batch Operations

#### US-9.1: Network Health Dashboard

**As** the VP of Marketing,
**I want** a real-time dashboard showing the digital health of all 260+ practices,
**So that** I can identify problems, track progress, and report to leadership without manual data collection.

**Acceptance Criteria:**

```
Given the executive dashboard is loaded
When the user views the network overview
Then the dashboard displays:
  - Enrollment progress (enrolled / blitz-deployed / pending / total)
  - Network average health score with trend
  - Tier breakdown (Optimized 85+ / Baseline 60-84 / Auto-deployed <60)
  - Alert feed (review drops, staff changes detected, website issues, content flags)
  - Content pipeline status (posts scheduled / flagged for QC / published this week)
And all data is pulled from live sources — no manual entry
And the dashboard refreshes every 5 minutes
```

```
Given the user clicks on a specific practice
When the practice detail view loads
Then it displays the Practice Health Scorecard:
  - Website: speed, mobile score, SEO score, content freshness
  - GMB: completeness score, post frequency, response rate
  - Social: post frequency, engagement rate, profile completeness
  - Reviews: avg rating, volume, response rate, sentiment trend
  - Leads: form conversion rate, response time, booking rate
And the overall score (0-100) is calculated as:
  Website (25%) + GMB (25%) + Social (20%) + Reviews (20%) + Leads (10%)
```

**Story Points**: 13
**Priority**: P0

#### US-9.2: HQ QC Queue

**As** an HQ creative team member,
**I want** a queue of items requiring quality review (new sites, content, flagged changes),
**So that** I can maintain brand standards efficiently across the network.

**Acceptance Criteria:**

```
Given new storefronts or content are awaiting QC
When the HQ team member opens the QC queue
Then items are listed by priority (new site > content flag > routine review)
And each item shows a preview with the practice name, template, and flagged concerns
And the reviewer can approve, reject with notes, or request changes
And approved items automatically proceed to deployment
And rejected items notify the concierge to inform the practice
```

```
Given the QC queue has items older than 24 hours
When the aging threshold is reached
Then an alert is sent to the QC team lead
And items without critical flags auto-approve after 48 hours
```

**Story Points**: 8
**Priority**: P1

#### US-9.3: Batch Operations

**As** the HQ web developer,
**I want** to make network-wide changes (template updates, brand changes, compliance updates) from a single interface,
**So that** I do not need to manually update 260+ individual sites.

**Acceptance Criteria:**

```
Given a template or brand element is updated in the admin interface
When the batch operation is initiated
Then the system generates previews for all affected sites
And shows which sites are affected and what changes on each
And flags any sites where the change may break layout
And the admin can confirm or exclude specific sites
```

```
Given a batch operation is confirmed
When deployment begins
Then all affected sites are regenerated and deployed
And the process completes within 2 hours for the full network
And the concierge sends a notification to each affected practice
And the dashboard shows batch operation status in real time
```

**Story Points**: 8
**Priority**: P1

---

## Technical Spikes

Spikes are time-boxed investigations to reduce uncertainty before committing to implementation.

### Spike 1: CRM Selection & Integration

**Question**: Which CRM should serve as the lead routing destination for practice contact forms?
**Time Box**: 3 days
**Investigation**:
- Evaluate HubSpot, Salesforce, and GoHighLevel against SGA requirements
- Key criteria: API quality, per-practice lead routing, form integration, cost at 260+ practices, existing SGA tooling
- Assess whether SGA already uses a CRM that should be extended vs. replaced
- Determine if a CRM is needed in Phase 1 or if a lightweight form-to-email solution suffices for MVP
**Output**: CRM recommendation with integration approach document

### Spike 2: iSolved API for Staff Change Detection

**Question**: Can iSolved provide real-time or near-real-time staff change events via API or webhook?
**Time Box**: 2 days
**Investigation**:
- Review iSolved API documentation for employee termination/hire events
- Determine if webhooks are available or if polling is required
- Assess data mapping: iSolved employee record to Practice Data Hub team member record
- Identify authentication and rate limit constraints
- Determine if iSolved covers all SGA platforms (SGA, Gen4, MODIS) or only some
**Output**: Integration feasibility assessment with recommended approach

### Spike 3: Google My Business API Rate Limits & Capabilities

**Question**: Can we manage 260+ GMB listings programmatically within API rate limits?
**Time Box**: 2 days
**Investigation**:
- Review GMB API (Google Business Profile API) rate limits for bulk operations
- Test: listing reads, listing updates, photo uploads, post publishing, review reading
- Determine if SGA qualifies for elevated API access (agency/partner programs)
- Map required operations to API capabilities (some GMB features are not API-accessible)
- Assess Google's verification requirements for bulk listing management
**Output**: GMB API capability matrix with rate limit mitigation strategy

### Spike 4: Domain Migration at Scale

**Question**: How do we handle DNS migration for practices with existing custom domains?
**Time Box**: 2 days
**Investigation**:
- Inventory current domain registrars and hosting providers across the network
- Determine which practices have custom domains vs. subdirectories vs. no web presence
- Design a DNS migration playbook (CNAME/A record changes, SSL provisioning, redirect strategy)
- Assess impact on existing SEO rankings and Google Search Console properties
- Determine whether DNS migration should be self-service (guided by concierge) or HQ-managed
**Output**: Domain migration playbook with risk assessment per migration type

---

## Dependency Map

```
US-1.1 (Data Harvesting)
  |
  +--> US-1.2 (Batch Harvest) -----> US-2.1 (Enrollment Invitation)
  |                                       |
  |                                       +--> US-3.1 (Concierge Welcome & Verification)
  |                                       |         |
  |                                       |         +--> US-4.1 (Template Selection)
  |                                       |                   |
  |                                       |                   +--> US-4.2 (Site Generation)
  |                                       |                             |
  |                                       |                             +--> US-5.1 (Review & Approval)
  |                                       |                                       |
  |                                       |                                       +--> US-6.1 (Go-Live)
  |                                       |
  |                                       +--> US-7.1 (Blitz Deploy) [after 30-day timer]
  |
  +--> US-9.1 (Dashboard) [can begin in parallel once Data Hub exists]

US-3.2 (Concierge Personality) --- parallel, informs all concierge interactions ---

US-6.2 (Domain Migration) --- parallel track, not blocking ---

US-8.1 (Staff Change) --- depends on US-6.1 (practices must be live first) ---
  |
  +--> US-8.2 (Hours/Service Changes) --- same dependency ---

US-9.2 (QC Queue) --- depends on US-5.1 (needs items to review) ---
US-9.3 (Batch Operations) --- depends on US-4.2 (needs Template Engine) ---

Spike 1 (CRM) --- informs US-6.1 (lead routing destination) ---
Spike 2 (iSolved) --- informs US-8.1 (staff change detection source) ---
Spike 3 (GMB API) --- informs US-1.1 (harvest capability) + US-8.2 (sync capability) ---
Spike 4 (Domain Migration) --- informs US-6.2 ---
```

### Critical Path (Phase 1)

```
Spike 3 (GMB) --> US-1.1 (Harvest) --> US-1.2 (Batch) --> US-2.1 (Invite)
  --> US-3.1 (Welcome) --> US-4.1 (Template) --> US-4.2 (Generate)
  --> US-5.1 (Review) --> US-6.1 (Go-Live)
```

This is the minimum path to get the first practices enrolled and live.

---

## MVP Scope: Phase 1 (Days 1-30)

### In Scope

| Story | Scope Notes |
|-------|------------|
| US-1.1 | Full harvest capability: web scraping + GMB API + social API reads |
| US-1.2 | Batch harvest for top 50 practices (not full 260+) |
| US-2.1 | Email invitations only (SMS in Phase 2). Batches of 20-30. |
| US-3.1 | Core verification flow: team, website, GMB status. Web chat only. |
| US-3.2 | System prompt with SGA brand voice. Basic personality. |
| US-4.1 | 2 templates: GP + 1 specialty (Perio or Full Arch). Concierge recommends with reasoning. |
| US-4.2 | Static site generation + CDN deployment. Subdomain hosting (practice.sgadental.com). |
| US-5.1 | Practice approval via chat. HQ QC notification (manual process, not full queue UI). |
| US-6.1 | Website deployment only. GMB update manual in Phase 1. Social setup manual. |
| US-9.1 | Basic dashboard: enrollment progress list, practice status, no health scores yet. |
| Spike 1 | CRM selection completed |
| Spike 3 | GMB API assessment completed |

### Phase 1 Milestone

20-30 practices enrolled and live through the concierge. End-to-end flow validated from harvest through go-live.

### Out of Scope (Phase 2+)

| Story | Phase | Notes |
|-------|-------|-------|
| US-2.1 (SMS) | Phase 2 | Add SMS channel for invitations |
| US-4.1 (remaining templates) | Phase 2 | Pediatric, Full Arch, Multi-Specialty templates |
| US-6.2 (Domain Migration) | Phase 2 | Custom domain support |
| US-7.1 (Blitz Deploy) | Phase 2 | Auto-deploy for non-engaged after 30 days |
| US-8.1 (Staff Change Management) | Phase 2 | Cascading change propagation |
| US-8.2 (Hours/Service Changes) | Phase 2 | Multi-channel sync |
| US-9.2 (QC Queue UI) | Phase 2 | Full QC workflow interface |
| US-9.3 (Batch Operations) | Phase 2 | Network-wide template/brand updates |
| Spike 2 (iSolved) | Phase 2 | HR integration for change detection |
| Spike 4 (Domain Migration) | Phase 2 | DNS migration playbook |
| Channel Sync (GMB, Social, Directories) | Phase 2 | Automated multi-channel propagation |
| Health Scores & Alerts | Phase 2 | Automated scoring and alert system |
| Content approval flows | Phase 3 | Social post approval via concierge |
| Review management | Phase 3 | Review response automation |
| Voice channel | Phase 4 | Concierge available via phone |
| A/B testing framework | Phase 4 | Template performance optimization |

---

## Sprint Question Traceability

| Sprint Question | Stories That Validate |
|----------------|---------------------|
| SQ1: Setup < 2 weeks | US-1.1, US-1.2, US-2.1, US-3.1, US-4.1, US-4.2, US-5.1, US-6.1 (full enrollment pipeline) |
| SQ2: Quality at scale | US-3.2, US-4.1, US-5.1, US-9.2, US-9.3 (opinionated agent + guardrailed templates + QC) |
| SQ3: Practice engagement | US-3.1, US-3.2, US-8.1 (conversational interface adoption and ongoing use) |
| SQ4: Proactive change detection | US-8.1, US-8.2, Spike 2 (staff change flow + iSolved integration) |
| SQ5: Executive visibility | US-9.1 (automated dashboard with no manual entry) |
