# Epic: SGA Studio Automation Agent — Phase 1 Pilot

**Product Manager:** Development Team — PM Role
**Sprint source:** Studio Automation at Scale
**Date:** 2026-03-10
**Epic status:** Ready for engineering
**Pilot scope:** 15 Gen4 practices, 8-week measurement period

---

## Epic Overview

Build the SGA Studio Agent — a multi-channel AI-powered creative request system that allows any practice manager to submit creative requests via email (`studio@sgadental.com`) or Microsoft Teams, receive AI-generated on-brand assets within 52 minutes (vs. 4.2-day baseline), and order print fulfillment with one click through a nominated vendor. The Creative Director reviews and approves via mobile notification. Human designers handle only custom/edge-case work.

**Epic goal (from sprint):** In 18 months, 80%+ of templated creative requests from all 160+ SGA locations fulfilled in ≤48 hours without designer involvement. Phase 1 proves the model at 15 Gen4 practices.

---

## Business Context

| Metric | Current State | Phase 1 Target |
|--------|---------------|----------------|
| Avg. turnaround (templated) | 4.2 days | ≤4 hours |
| Designer involvement per request | 100% | ≤20% (custom/edge-case only) |
| Print volume per month | Baseline TBD | -20% vs. baseline |
| Vendor pain (Staples) | High | Eliminated via Gelato API |
| CD review time per day | Unknown (tangled with production) | ≤20 min total across pilot practices |

---

## Sprint Questions → Epic Pillars

| Sprint Question | Epic Pillar | Primary Stories |
|----------------|-------------|-----------------|
| SQ1: Practice manager self-serves via chatbot | Conversational intake across channels | Stories 1.1–1.4 |
| SQ2: AI meets brand standards with 60-sec human gate | Design generation + CD review | Stories 2.1–2.5 |
| SQ3: Reduce print 30-40% via medium challenge | Medium challenge + digital deployment | Stories 3.1–3.3 |
| SQ4: DAM foundation for all practices | Practice profile + brand kit management | Stories 4.1–4.3 |
| SQ5: Vendor consolidation, one-click print | Nominated vendor API integration | Stories 5.1–5.2 |

---

## User Stories

### Pillar 1 — Conversational Intake (SQ1)

**Story 1.1 — Email Intake**
> As a practice manager, I can email `studio@sgadental.com` with a plain-language creative request and receive a reply from the Studio Agent within 60 seconds, so that I don't have to learn a new tool or find a different contact.

**Acceptance Criteria:**
- Given a practice manager sends any email to `studio@sgadental.com`
- When the email is received
- Then the agent responds within 60 seconds with a contextual reply that either (a) asks at most 3 clarifying questions, or (b) confirms the brief is complete and generation is starting
- And the reply is sent from `studio@sgadental.com` (not a no-reply address)
- And the reply identifies the practice by name ("Hi Ashley at Towne Lake Dental")

**Spike:** Verify M365 email API webhook latency vs. polling interval. Must achieve <60s response time.

---

**Story 1.2 — Teams Intake**
> As a practice manager at a Gen4 practice using Microsoft Teams, I can DM the Studio Agent or @mention it in a channel and receive the same conversational intake experience as email.

**Acceptance Criteria:**
- Given a practice manager DMs the Studio Agent bot in Teams
- When they describe a creative need in natural language
- Then the agent responds within 30 seconds with contextual clarifying questions (max 3)
- And the conversation threads correctly (each reply is in the same thread)
- And the final approved assets are delivered back to the same Teams thread

---

**Story 1.3 — Practice Identification**
> As the Studio Agent, I automatically identify which practice a request comes from based on the sender's email address, so that I can load the correct brand kit and address the practice manager by name.

**Acceptance Criteria:**
- Given an email from `ashley@townlakedental.com` arrives
- When the agent processes the request
- Then the agent matches the sender domain to the practice profile in the database
- And loads the correct Canva brand kit for that practice
- And addresses the reply to Ashley (first name from sender)
- If no match found: agent replies asking for the practice name before proceeding

---

**Story 1.4 — Maximum 3 Clarifying Questions**
> As a practice manager, I am never asked more than 3 questions in a single request, so that the intake feels like a quick conversation, not a lengthy form.

**Acceptance Criteria:**
- Given a practice manager submits a creative request
- When the agent determines the brief is incomplete
- Then the agent asks at most 3 questions in a single reply (not one at a time)
- And if the brief is complete (e.g., "I need 50 business cards"), the agent skips all questions and proceeds directly to generation confirmation
- And the agent never asks for information it can infer from the practice profile (practice name, address, phone, logo)

---

### Pillar 2 — Design Generation + CD Review (SQ2)

**Story 2.1 — Template-Based Generation**
> As the Studio Agent, I can generate a brand-correct creative asset in Canva using the practice's brand kit and an approved template, given a complete brief.

**Acceptance Criteria:**
- Given a complete brief with: collateral type, offer text, CTA, audience, deadline, format
- When the agent selects the matching template from the 8-template Phase 1 library
- Then the Canva API is called with the practice brand kit + template ID + variable values
- And the generated asset uses the practice's logo, brand colors, and font stack (all locked)
- And a preview image (JPG, min 800px wide) is returned within 90 seconds of generation starting
- If no matching template: brief is routed to the human designer queue with an explanatory message to the practice manager

---

**Story 2.2 — Practice Preview Delivery**
> As a practice manager, I receive a preview of my generated assets via the same channel I used to submit my request, so that I can review and approve without switching tools.

**Acceptance Criteria:**
- Given an asset has been generated
- When the agent sends the preview
- Then the preview is delivered via email (if request came via email) or Teams (if via Teams)
- And the preview includes: one thumbnail per format generated, a one-sentence description of each, and two clear reply options ("Looks good" or "Change [X]")
- And the practice manager can approve with a one-word reply ("Looks good" or "Perfect" or similar affirmative)

---

**Story 2.3 — Text-Only Revision**
> As a practice manager, I can request a text change to a generated asset and receive the revised version within 2 minutes.

**Acceptance Criteria:**
- Given a practice manager replies requesting a text change (headline, CTA, date, offer detail)
- When the agent processes the change request
- Then the agent updates the variable in Canva and regenerates the preview
- And the revised preview is delivered via the original channel within 2 minutes
- And the agent allows maximum 2 automated revision loops before escalating to the human designer queue

**Note:** Layout changes, photo changes, and structural redesigns are NOT handled as revisions — they escalate to the human designer queue.

---

**Story 2.4 — CD Mobile Review**
> As the Creative Director, I receive a mobile push notification when an asset is ready for brand quality review, and can approve or request changes with a single tap.

**Acceptance Criteria:**
- Given a practice manager has confirmed the asset looks correct
- When the agent routes to the CD for final review
- Then the CD receives a Teams push notification with: practice name, collateral type(s), thumbnail image(s), and 3 action buttons: [Approve All] [Request Change] [Reject]
- And the notification is received within 60 seconds of the practice manager's approval
- And tapping "Approve All" triggers asset deployment within 30 seconds
- And tapping "Request Change" opens a text field for the CD to describe the required change

**SLA:** CD review expected within 2 hours during business hours (9am-6pm local time). Reminder notification sent at 2-hour mark. Auto-escalation to named backup at 4-hour mark.

---

**Story 2.5 — Human Designer Escalation**
> As a practice manager with a custom or edge-case creative request, I receive acknowledgment that my request requires a human designer and a clear expectation for when I'll hear back.

**Acceptance Criteria:**
- Given a request that has no matching template, OR a revision request that is structural/layout in nature
- When the agent determines escalation is required
- Then the agent notifies the practice manager via original channel: "This request needs our designer. You'll hear back within 1 business day."
- And the request is logged to the human designer queue with: practice name, full conversation thread, brief summary, priority flag (deadline-driven or standard)
- And the studio admin can see all escalated requests in the analytics dashboard

---

### Pillar 3 — Medium Challenge + Digital Deployment (SQ3)

**Story 3.1 — Medium Challenge**
> As the Studio Agent, I proactively suggest a digital alternative whenever a practice manager requests print collateral, before generating the print version.

**Acceptance Criteria:**
- Given a request that includes the words: flyer, brochure, poster, handout, print, or references a format that is typically print
- When the agent processes the intake
- Then the agent's first reply includes a medium challenge: a specific description of the digital equivalent ("a patient email would reach your 3,200 active patients for free and I can have it ready today")
- And the challenge is a suggestion, not a blocker — the practice can accept both channels or proceed with print only
- And the medium challenge appears in the first reply, not after the brief is complete

---

**Story 3.2 — Patient Email Deployment (Weave)**
> As a practice manager, my approved patient email asset is automatically scheduled for delivery to my patient list through Weave, without me taking any additional action.

**Acceptance Criteria:**
- Given an approved email header asset and a scheduled send date from the brief
- When the agent receives CD approval
- Then the asset is uploaded to Weave via API with the practice's patient list and the specified send date
- And the practice manager receives a confirmation: "Your patient email is scheduled for [date] to [X] active patients"
- And the studio never needs to touch this step

**Spike:** Confirm Weave API access for the 15 pilot practices. Identify which Gen4 practices use Weave vs. alternative patient comms platforms.

---

**Story 3.3 — TV Slide Asset Delivery**
> As a practice manager, my approved TV slide asset is delivered to me as a ready-to-use file (Phase 1 manual upload; Phase 2 auto-deployment).

**Acceptance Criteria (Phase 1):**
- Given an approved TV slide asset (1920×1080 PNG)
- When CD approval is received
- Then the practice manager receives the file as an email attachment or download link
- And the email includes simple upload instructions for the most common display setup at their practice
- And the studio never needs to be involved in delivery

---

### Pillar 4 — Practice Profile + Brand Kit (SQ4)

**Story 4.1 — Practice Profile Database**
> As the Studio Agent, I can look up any practice by the sender's email domain and retrieve their complete profile: name, address, phone, Canva brand kit ID, brand health score, and last request date.

**Acceptance Criteria:**
- Given a sender email `@[practicedomain].com`
- When the agent queries the practice profile database
- Then the response includes: practice name, primary contact name, Canva brand kit ID, brand health score, and last 3 request records
- And the lookup completes in <500ms
- And if no match: the agent gracefully requests the practice name before continuing

---

**Story 4.2 — Brand Health Score**
> As a studio admin, I can see a brand health score (0-100) for each practice that tells me whether their brand kit is complete enough to support automated design generation.

**Acceptance Criteria:**
- Given a practice profile
- When the brand health score is calculated
- Then the score reflects: logo completeness (all formats present: SVG, PNG, stacked, horizontal) = 30 pts, color definition (primary + secondary) = 20 pts, font stack defined = 20 pts, approved photos (≥3 tagged) = 30 pts
- And practices scoring <40 are flagged in the studio admin dashboard with a "Needs asset lift" indicator
- And the agent routes requests from <40 practices to the human designer queue with a flag: "Brand kit incomplete — asset lift required before automation can serve this practice"

---

**Story 4.3 — Studio Admin Brand Kit Management**
> As a studio admin, I can view and update any practice's brand kit data from a studio admin interface, including uploading new assets and editing the practice profile.

**Acceptance Criteria:**
- Given a studio admin logs into the admin interface
- When they navigate to a practice profile
- Then they can see: brand health score, all uploaded assets (logo files, photos), Canva brand kit link, and last request history
- And they can upload new logo files, photos, and update practice contact info
- And uploaded assets are automatically synced to the practice's Canva brand kit via the Canva API

---

### Pillar 5 — Nominated Vendor (SQ5)

**Story 5.1 — One-Click Print Order**
> As a practice manager, I can order print collateral with a single click after my asset is approved, with no need to specify a vendor, file format, or print specifications.

**Acceptance Criteria:**
- Given an approved print-ready asset (PDF/X-1a)
- When the agent delivers the CD-approved asset
- Then the practice manager's confirmation message includes a single "Order [X] copies" button (pre-loaded with recommended quantity based on practice size)
- And clicking that button triggers the Gelato API order with: correct file, correct print spec for the collateral type, practice's ship-to address from profile, standard quantity
- And the practice manager receives a Gelato order confirmation + estimated delivery date within 60 seconds of clicking

---

**Story 5.2 — Print Order Tracking (Studio-Invisible)**
> As a studio admin, I can see all print orders placed through the system in the analytics dashboard, without being involved in any individual order.

**Acceptance Criteria:**
- Given a print order has been placed via Gelato API
- When the order is confirmed
- Then the order record is logged in the analytics dashboard: practice name, collateral type, quantity, order date, estimated delivery, status
- And Gelato tracking notifications go directly to the practice manager's email — the studio is NOT copied
- And the studio admin can see total print order volume, average order value, and month-over-month print volume trend

---

## Technical Spikes (Required Before Story Implementation)

| Spike | Question | Owner | Due |
|-------|----------|-------|-----|
| SP-1 | Canva Connect API: what template variable types are supported? Can image fields be programmatically swapped? Is PDF/X-1a export available? | Tech Lead | Sprint week 1 |
| SP-2 | M365 email API: what is the latency from email received to webhook fire? Can we achieve <60s response time? | Backend | Sprint week 1 |
| SP-3 | Weave API: does SGA/Gen4 have existing API access? What is the data model for patient list send? | Backend | Sprint week 1 |
| SP-4 | Gelato API: onboarding requirements, per-item pricing model, ship-to address handling, minimum order quantities | Backend | Sprint week 2 |
| SP-5 | Teams Bot Framework: can the bot thread replies in an existing conversation correctly? Can it attach image thumbnails inline? | Backend | Sprint week 1 |

---

## Dependency Map

```
[Practice Profile DB] ──── required by ──── [Intake: Practice ID lookup]
[Canva Brand Kit Setup] ── required by ──── [Design Generation Engine]
[Template Library (8)] ─── required by ──── [Design Generation Engine]
[Weave API access] ──────── required by ──── [Digital Deployment: Patient email]
[Gelato API account] ────── required by ──── [Print Fulfillment]
[M365 email setup] ─────── required by ──── [Email Intake Channel]
[Teams Bot registration] ── required by ──── [Teams Intake Channel]
[CD review notification] ── required by ──── [Approval Workflow]
```

**Critical path:** Practice Profile DB + Canva Brand Kit Setup must be complete before any story in Pillars 1-2 can be tested end-to-end.

---

## Out of Scope — Phase 1

- Slack intake channel (Phase 2)
- TV slide auto-deployment via display system API (Phase 2)
- Social post scheduling integration (Phase 2)
- Proactive content calendar push (Phase 2 / Solution C evolution)
- Embedded trigger integrations with practice management software (Phase 3)
- Brand kit backfill for non-Gen4 practices (parallel track, not this epic)
- Custom creative requests beyond 8-template library (human designer queue handles these)
