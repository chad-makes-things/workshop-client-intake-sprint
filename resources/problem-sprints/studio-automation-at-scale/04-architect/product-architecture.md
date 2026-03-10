# Product Architecture

**Sprint:** Studio Automation at Scale
**Agent:** Prototype Builder
**Date:** 2026-03-10
**Solution:** A — Conversational Studio Agent (multi-channel: Teams, Slack, Email)

---

## Architecture Overview

The SGA Studio Agent is a multi-channel AI-powered creative workflow system. It sits between practice managers (who have creative needs) and the existing production infrastructure (Canva, print vendors, digital channels). The agent handles intake, intent classification, asset verification, design generation, approval routing, and fulfillment — across three intake channels — without requiring the studio design team to touch templated requests.

---

## Component Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTAKE CHANNELS                               │
│  [Teams Bot]  [Slack Bot]  [Email: studio@sgadental.com]        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              STUDIO AGENT CORE (Claude API)                      │
│  • Intent classification                                         │
│  • Conversational intake (clarifying questions)                  │
│  • Medium challenge logic                                        │
│  • Brief assembly + validation                                   │
│  • Routing decision (automated / human / digital-only)           │
└───────┬───────────────────────────────┬────────────────────────-┘
        │                               │
        ▼                               ▼
┌──────────────────┐          ┌──────────────────────────┐
│  PRACTICE        │          │  HUMAN DESIGNER QUEUE     │
│  PROFILE + DAM   │          │  (custom / edge-case)     │
│  (Canva Brand    │          │  Project management tool  │
│  Kit per         │          │  (Asana / Monday / etc.)  │
│  practice)       │          └──────────────────────────┘
└──────┬───────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────────┐
│              DESIGN GENERATION ENGINE (Canva API)                │
│  • Template selection by collateral type                         │
│  • Variable population (offer text, dates, CTA, photo)          │
│  • Brand-locked output (colors, fonts, logo from brand kit)      │
│  • Preview image generation                                      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              APPROVAL WORKFLOW                                    │
│  • Practice preview (agent sends previews via original channel)  │
│  • Revision handling (variable swaps only, <2 min turnaround)    │
│  • CD mobile review (push notification, one-tap approve/reject)  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
             ┌─────────────┴──────────────┐
             ▼                            ▼
┌────────────────────────┐   ┌────────────────────────────────────┐
│  DIGITAL DEPLOYMENT    │   │  PRINT FULFILLMENT                 │
│  • Patient email →     │   │  • Nominated vendor API (Gelato)   │
│    Weave / Podium      │   │  • Auto-submit with correct specs  │
│  • TV slide → display  │   │  • Tracking confirmation to PM     │
│    queue               │   │  • Studio never touches order      │
│  • Social post →       │   └────────────────────────────────────┘
│    scheduling tool     │
└────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              STUDIO ANALYTICS DASHBOARD                          │
│  • Request volume by channel, practice, collateral type          │
│  • Medium split (digital vs. print trend)                        │
│  • Turnaround time tracking                                      │
│  • Brand kit health per practice                                 │
│  • CD review time and approval rate                              │
│  • Practices with no activity in 90+ days                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Components

### Component 1 — Intake Channel Adapters
**Responsibility:** Normalize inbound messages from Teams, Slack, and email into a standard request object the Studio Agent Core can process.

**Interfaces:**
- **Teams:** Microsoft Bot Framework SDK. Registers as a Teams app. Receives messages from @StudioAgent mention or direct message.
- **Slack:** Slack Bolt SDK. Registered as a Slack app in the SGA workspace. DM or channel mention.
- **Email:** Dedicated mailbox `studio@sgadental.com`. Email parser reads inbound messages (sender, subject, body, attachments). Replies sent via the same address. Microsoft 365 or Google Workspace API.

**Key behaviors:**
- All three channels produce the same request schema: `{sender_email, practice_id, raw_message, timestamp, channel}`
- Practice identification: sender email matched to practice profile in the database
- Attachment handling: if a practice sends a logo or photo, the adapter uploads it to their Canva brand kit and confirms receipt before continuing

**ADR 1:** Email chosen as the third channel (rather than SMS or a standalone app) because every practice manager already has email, it's channel-agnostic across SGA/Gen4/MODIS platforms, and it creates a searchable paper trail of every request without additional infrastructure.

---

### Component 2 — Studio Agent Core (Claude API)
**Responsibility:** The intelligence layer. Understands what the practice is trying to accomplish, runs the conversational intake, classifies the request, challenges the medium, assembles a complete brief, and routes to the correct downstream workflow.

**Interfaces:**
- Input: Normalized request object from Channel Adapter
- Output: (a) Follow-up question back to channel, or (b) Completed brief → Design Generation Engine, or (c) Escalation → Human Designer Queue

**Key behaviors:**

*Intent classification:*
- Promotional collateral (special offer, seasonal campaign)
- Operational collateral (business card, appointment reminder, door sign)
- Referral program materials (doctor referral flyer, referral card)
- Patient communication (review request, welcome, follow-up)
- Custom / edge-case (new service launch, event signage, photography request)

*Intake conversation:*
- Maximum 3 clarifying questions per request (never more)
- Questions are generated dynamically based on what's missing from the brief
- If the request is complete in the first message (e.g. "I need 50 business cards"), no questions asked

*Medium challenge:*
- Triggered when: request mentions flyer, brochure, poster, handout, or print
- Challenge message surfaces the digital equivalent with a concrete description ("a patient email reaches all your active patients for free and I can have it ready today")
- Medium challenge is not a blocker — if the practice confirms they want print, the agent proceeds
- Both channels can be selected (digital + print is a valid outcome)

*Brief assembly:*
- Validated brief fields: service/offer, audience, format(s), deadline, doctor/brand featured, CTA text
- Missing field detection: agent only asks for fields that aren't inferable from context
- Brief is stored per request with unique ID for traceability

*Routing logic:*
- Template match found + complete brief → Design Generation Engine (automated)
- No template match OR custom creative required → Human Designer Queue
- Digital-only output (email, TV slide) where practice wants no print → Digital Deployment (bypasses print path)

**ADR 2:** Claude API selected as the LLM backbone because the intake conversation requires nuanced understanding of dental practice context, multi-turn memory within a session, and the ability to infer incomplete briefs without asking unnecessary questions. The model is prompted with SGA brand context and the medium-challenge script.

---

### Component 3 — Practice Profile + DAM (Canva Enterprise)
**Responsibility:** Stores and serves per-practice brand assets. The single source of truth for logo, brand colors, fonts, and approved photography for each of the 160+ practices.

**Interfaces:**
- Read: Design Generation Engine pulls brand kit by practice ID
- Write: Studio admin uploads new assets; Channel Adapter uploads practice-submitted assets
- Check: Agent Core queries brand kit completeness before committing to automated generation

**Key behaviors:**
- Per-practice brand kit: logo (all formats), primary color, secondary color, font stack, approved photos (tagged by subject: team, office, procedure)
- Brand health score: 0-100 based on completeness of logo formats, color definition, font definition, photo count, and recency of last update
- Practices with brand health score < 40 → agent flags for human designer to complete asset package before automated requests can be fulfilled
- Brand kit is read-only to the Design Generation Engine — only Studio Admin can modify master brand elements
- Canva Enterprise brand locking: colors, fonts, and logo placement are locked in all templates; only approved variable fields can be changed

**ADR 3:** Canva Enterprise chosen as the DAM layer (rather than Brandfolder or Bynder) because it doubles as the design production environment, eliminating a file-transfer step between DAM and design tool. The Canva API allows programmatic brand kit management and template variable population. Cost is lower than standalone DAM + separate design tool licensing.

---

### Component 4 — Design Generation Engine (Canva API)
**Responsibility:** Generates print-ready and screen-ready creative assets from approved templates, populated with practice-specific brand elements and brief-specific variable content.

**Interfaces:**
- Input: Completed brief from Agent Core + brand kit from Practice Profile
- Output: Preview images (JPG) sent back to channel; print-ready PDFs or screen assets routed to Approval Workflow

**Key behaviors:**

*Template library (initial build — 12 templates):*
1. Business card (front + back)
2. Appointment reminder card (4×6 print)
3. Promotional flyer (4×6 print)
4. Promotional flyer (8.5×11 print)
5. Patient email header (600px)
6. In-office TV slide (1920×1080)
7. Social post — square (1080×1080)
8. Social post — story (1080×1920)
9. Doctor referral card (3.5×2 print)
10. Review request card (3.5×2 print, QR code field)
11. Door sign / window cling (8.5×11)
12. Welcome packet cover (letter size)

*Variable fields per template (examples):*
- Practice name (auto-populated from profile, locked)
- Logo (auto-populated from brand kit, locked)
- Offer headline (editable)
- Offer detail / body copy (editable)
- CTA text (editable)
- Photo (selected from approved photo library by agent, swappable)
- Date range (editable)
- Phone / URL (auto-populated from practice profile)

*Generation process:*
1. Template selected by collateral type from brief
2. Canva API called with brand kit ID + template ID + variable values
3. Asset generated (typically 30-60 seconds)
4. Preview image (JPG 800px wide) returned for practice review
5. On approval: final export triggered (PDF/X-1a for print, PNG/JPG for digital)

**ADR 4:** Template library starts at 12. Adding templates is a studio admin function, not an engineering function — Canva's template system is configured by a designer, not a developer. The 12 initial templates cover ~85% of request volume based on the expert conversation (business cards, appointment reminders, flyers, TV slides, social posts, referral cards, review cards). Edge cases route to the human designer queue.

---

### Component 5 — Approval Workflow
**Responsibility:** Routes completed assets to the practice for preview, handles revision requests, and routes to the Creative Director for final sign-off.

**Interfaces:**
- Input: Generated preview images from Design Generation Engine
- Output: Approved assets routed to Digital Deployment or Print Fulfillment

**Key behaviors:**

*Practice preview:*
- Agent sends preview via the same channel as the original request (if request came via email, preview goes to email; Teams → Teams; Slack → Slack)
- Preview message includes: asset thumbnails, all generated formats, one-sentence description of what was created
- Practice has two options: "Looks good" or "Change [X]"

*Revision handling:*
- Text changes (CTA, headline, date): agent calls Canva API to update the variable, regenerates preview in <2 minutes
- Layout or structural changes: escalated to human designer queue (agent explains: "That change needs our designer — I'll add it to their queue. You'll hear back within 1 business day.")
- Maximum 2 automated revision loops; third revision always escalates to human

*CD review:*
- Push notification sent to CD's mobile device via notification service (Pushover, or native Teams/Slack notification if CD is on those platforms)
- Notification shows: practice name, collateral types, thumbnail previews
- Three actions: Approve All | Request Change (with text field) | Reject (with text field)
- SLA: CD review expected within 2 hours during business hours; agent sends reminder at 2-hour mark
- If no response in 4 hours: auto-escalation to backup reviewer or studio admin

**ADR 5:** Practice preview happens before CD review (not after) so the practice has already confirmed the content is correct before the CD spends time reviewing it. This keeps CD review focused on brand quality, not content accuracy — reducing both the likelihood of change requests from the CD and the total time spent in review.

---

### Component 6 — Digital Deployment
**Responsibility:** Routes approved digital assets to the appropriate distribution channel — patient communication platform, in-office display system, or social scheduling tool.

**Interfaces:**
- Patient email: API integration with Weave, Podium, or equivalent patient communication platform
- TV slide: API integration with in-office display system (to be confirmed per practice — may be Apple TV + a display management app, or a dental-specific system)
- Social post: API integration with a social scheduling tool (Buffer, Later, or equivalent)

**Key behaviors:**
- Patient email: asset uploaded + scheduling data provided (send date from brief); confirmation returned
- TV slide: asset uploaded to practice's display queue; agent confirms "your slide is live in rotation"
- Social post: asset uploaded + caption drafted by agent from brief content; practice confirms caption before scheduling
- All deployments logged to request record in analytics dashboard

**ADR 6:** Digital deployment integrations are the longest lead-time component because they depend on which tools each practice uses. Phase 1 pilot (Gen4 practices) should select 1-2 patient comms platforms that Gen4 already uses and integrate those first. TV slide integration can be manual in Phase 1 (agent sends the asset, practice manager uploads to their display) and automated in Phase 2.

---

### Component 7 — Print Fulfillment (Nominated Vendor API)
**Responsibility:** Submits print-ready files to the nominated vendor with correct specifications; returns tracking confirmation to the practice.

**Interfaces:**
- Input: Approved print-ready PDF from Design Generation Engine
- Output: Order confirmation + tracking number to practice via channel

**Key behaviors:**
- Vendor API called with: file URL, print spec (size, finish, quantity, ship-to address)
- Print specs pre-configured per template type (the 4×6 flyer template always outputs at the correct DPI, bleed, and color profile for the vendor)
- Ship-to address pulled from practice profile (studio admin manages, practice can't change)
- Order confirmation forwarded to practice manager via original channel
- Vendor tracking emails go to practice manager directly; studio is not CC'd
- Studio admin can view all print orders in the analytics dashboard but does not touch individual orders

**Vendor selection:** Gelato (recommended) — global print-on-demand network, API-native, white-label ordering portal available, 140+ production partners, consistent quality standards. Alternative: MOO for premium business cards specifically.

**ADR 7:** The nominated vendor model means practices can only order through one channel — the agent. They cannot order from Staples, a local printer, or any other vendor and expect the studio to support it. This is enforced through the workflow: the only "Order Print" button in the system routes to the nominated vendor. Practices that want to use local printers can download the print-ready PDF and manage it themselves, but they get no studio support for that path.

---

### Component 8 — Studio Analytics Dashboard
**Responsibility:** Gives the SGA Studio team visibility into request volume, turnaround, medium split, brand kit health, and practice engagement — without touching individual orders.

**Key metrics surfaced:**
- Requests by channel (Teams / Slack / email) — weekly and monthly
- Requests by collateral type — which templates are used most
- Automated fulfillment rate vs. human escalation rate
- Medium split: digital-only / digital + print / print-only — track the trend over time
- Average turnaround time (first message → asset delivered)
- CD average review time per request
- Brand kit health score by practice — flag practices below 40
- Practice engagement: last request date per practice; inactive >90 days flagged for proactive outreach
- Print vendor order volume and average order size

---

## Architectural Decision Records (ADRs)

| # | Decision | Rationale |
|---|----------|-----------|
| ADR 1 | Email as third intake channel (`studio@sgadental.com`) | Universal access across all SGA/Gen4/MODIS platforms; creates searchable audit trail; reaches practice managers who aren't on Teams/Slack |
| ADR 2 | Claude API for the agent core | Nuanced dental-context understanding; multi-turn session memory; medium-challenge scripting; avoids rigid decision trees that break on edge cases |
| ADR 3 | Canva Enterprise as DAM + design tool | Eliminates file transfer between DAM and design tool; API-programmatic brand kit management; lower cost than standalone DAM + separate design tool |
| ADR 4 | Template library starts at 12, studio-admin expandable | 12 templates cover ~85% of request volume; adding templates is a design task, not an engineering task; prevents scope creep in Phase 1 |
| ADR 5 | Practice preview before CD review | CD's review is a brand quality check, not a content accuracy check; eliminates a revision loop after CD approval |
| ADR 6 | Digital deployment integrations phased | Canva → patient comms platform in Phase 1; TV slide manual with asset delivery; full display automation in Phase 2 |
| ADR 7 | Nominated vendor enforced through workflow | Eliminates vendor sprawl (Staples, local printers); consistent specs; no studio involvement per order; Gelato API-native |

---

## Phase 1 Pilot Scope (Gen4, 15 Practices)

| Component | Phase 1 State |
|-----------|---------------|
| Intake channels | Email + Teams (Slack in Phase 2) |
| Template library | 8 templates (business card, appointment reminder, 4×6 promo flyer, 8.5×11 flyer, patient email header, TV slide, review request card, doctor referral card) |
| DAM | Canva brand kits for 15 Gen4 pilot practices (already have good asset bases) |
| Digital deployment | Patient email (Weave integration if Gen4 uses Weave); TV slide: manual delivery |
| Print vendor | Gelato API for business cards, flyers, referral cards |
| Analytics | Basic dashboard: request volume, turnaround, medium split |
| CD review | Mobile push via Teams notification |

**Success criteria for Phase 1 pilot (8 weeks):**
- [ ] ≥80% of templated requests fulfilled without designer involvement
- [ ] Average turnaround ≤4 hours (vs. 4.2 days baseline)
- [ ] ≥1 practice spontaneously chooses digital instead of print after medium challenge
- [ ] CD review time ≤5 minutes per day total across all pilot practices
- [ ] Zero brand compliance violations in approved outputs
- [ ] Net Promoter Score from practice managers: ≥7/10

---

## Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| Agent intelligence | Claude API (claude-sonnet-4-6) | Best-in-class conversational understanding; dental/brand context aware |
| Teams integration | Microsoft Bot Framework SDK | Native Teams bot registration; DM + channel support |
| Email integration | Microsoft 365 API or Google Workspace API | Reads/sends from `studio@sgadental.com`; attachment parsing |
| Design generation | Canva Connect API | Brand kit management; template variable population; export |
| DAM | Canva Enterprise brand kits | Per-practice brand control; locked templates |
| Print vendor | Gelato API | Print-on-demand; API-native; 140+ production partners |
| Patient comms | Weave API (Phase 1) | Most common patient comms platform in Gen4 network |
| CD review notifications | Microsoft Teams + mobile push | CD is already in Teams; no new app required |
| Analytics | Posthog or Metabase on top of request log DB | Event tracking per request; simple dashboard |
| Hosting | Azure (SGA likely on Microsoft ecosystem) | Consistent with Teams + M365 integration |
