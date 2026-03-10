# Solution Sketches

**Sprint:** Studio Automation at Scale
**Agent:** Solution Designer
**Date:** 2026-03-10
**Target:** The creative request intake and routing moment

---

## Substitution Test

Each solution below must pass the substitution test: could one solution replace another without the user noticing? If yes, they're not distinct enough. These five are architecturally different enough that choosing one meaningfully changes what gets built, who uses it, and what the experience is.

---

## Solution A — The Conversational Studio Agent

**"Talk to the Studio like you text a teammate"**

### Overview

A conversational AI agent (chatbot) embedded in Teams or Slack that practice managers interact with in natural language. They describe what they need. The agent asks clarifying questions. It surfaces the digital alternative before the print option. It checks the practice's Canva brand kit for existing assets. It generates design options from the approved template library. It routes to the CD for a 60-second mobile approval. If print is needed, it submits the order to the nominated vendor automatically.

### How It Works — Step by Step

1. Practice manager opens Teams and types: "Hey Studio, we're running a liner special next month and need materials."
2. Agent responds: "Got it. Quick questions — who's the audience, existing patients or new? What's the offer? Any deadline?"
3. Practice manager answers. Agent says: "For existing patients, the most effective option is a patient email + in-office TV slide. I can generate both from your brand kit right now. Want to see them before we talk about print?"
4. Practice manager says yes. Agent generates two options in Canva (email header + TV slide), shows previews with the practice's logo, colors, and offer text populated.
5. Practice manager picks one. Agent routes to CD: push notification on mobile — "Studio request from [Practice], 60-second review needed." One tap: Approve / Request changes.
6. On approval: email asset sent to patient comms platform. TV slide uploaded to practice's in-office display queue. If print was also requested, file sent to nominated vendor with correct specs.
7. Practice manager gets confirmation: "Your liner special assets are live. Patient email scheduled. TV slide is in rotation."

### Strengths

- Lowest friction for the practice manager — they already text; this is the same
- Medium challenge is built into the conversation flow, not a separate step
- Feels like a personal service, not a portal to navigate
- Works without a new login or new tool for practices
- Conversational memory: the agent knows what each practice has used before, what their brand kit contains, what their last request was

### Risks

- AI misinterpretation of ambiguous requests could generate wrong outputs — requires good fallback to human designer
- Requires clean, current Canva brand kits for all 160 practices before going live (the asset pre-work is the dependency)
- Teams/Slack adoption varies — if a practice manager doesn't live in Teams, they won't find the agent
- Initial training period where agents generate outputs that need significant correction could frustrate early adopters

### Feasibility

- **Technical:** High. Teams/Slack bot + Claude API for NLU + Canva API for design generation + print vendor API for order routing. All components exist. Integration work is the lift.
- **Organizational:** Medium. Requires the pre-work: enterprise Canva setup, brand kits for all practices, nominated vendor agreement.
- **Time to v1:** 10-14 weeks for a pilot with 10-15 Gen4 practices.

---

## Solution B — The Self-Service Brand Portal

**"A menu, not a conversation"**

### Overview

A dedicated web portal (SGA Studio Portal) where practice managers log in, browse a template library organized by collateral type, select a template, fill in customizable fields (practice name is pre-populated, they add offer text and dates), and submit for generation. The system generates the asset from Canva, sends a preview to the practice, sends a parallel 60-second review to the CD, and on approval routes to the practice for download or routes to the vendor for print.

The portal also shows each practice's asset health status — "Your brand kit is complete / You're missing a current photo set / Your logo is outdated" — and allows the studio to push brand foundation updates from the admin side.

### How It Works — Step by Step

1. Practice manager logs in to SGA Studio Portal. Dashboard shows: recent orders, asset library, content calendar (SGA corporate content already loaded), their brand health status.
2. They click "+ New Request." Category selector: Business Card | Appointment Reminder | Promotional Flyer | Social Post | TV Slide | Doctor Referral Card | Review Request | Other.
3. They select "Promotional Flyer." They see 3 template options in their practice's brand colors with their logo pre-populated. They choose one.
4. Editable fields surface: Offer headline, Offer detail, CTA, Dates. They fill in. Live preview updates as they type.
5. They hit "Submit for Review." System generates the Canva output. Preview goes to CD via mobile notification. 60-second tap-to-approve.
6. On approval, portal sends the practice an email with the download link + "Would you like to print? Click here to order from [Vendor]." One click triggers the print order with correct specs.
7. Portal logs the order. Studio admin can see volume, turnaround times, and which practices haven't used the portal in 90+ days.

### Strengths

- Visual and intuitive — practice managers can browse templates, not describe them in words
- No AI misinterpretation risk — template selection is deliberate, not inferred
- The brand health dashboard is a powerful governance tool for the studio
- Adoption is easier to measure and nudge (usage analytics built in)
- Works without AI for the initial version — AI generation can be layered in later for custom requests

### Risks

- A portal requires a login, which adds friction — practice managers may default to email/text anyway
- Doesn't proactively challenge the medium — the practice manager has to choose the right category, and "Promotional Flyer" will be chosen even when a TV slide is better
- Template navigation puts the burden on the practice manager to know what they need — doesn't diagnose the actual intent
- Custom or edge-case requests (new service launch, event signage) need a separate "custom request" flow that can become the bottleneck

### Feasibility

- **Technical:** High. Well-understood web app pattern. Canva API for generation. No AI dependency for v1.
- **Organizational:** Medium-high. Portal needs to be built and populated with templates. Canva brand kits required. Training needed for 160 practice managers.
- **Time to v1:** 8-12 weeks for a pilot portal with 20-25 templates.

---

## Solution C — The Proactive Content Studio

**"The Studio sends, you approve"**

### Overview

Flip the intake model entirely. Instead of waiting for practices to request materials, the SGA Studio proactively publishes a monthly content package to each practice: a TV slide rotation, a social media calendar, a patient email template, and seasonal promotional assets — all pre-built, on-brand, and practice-specific (logo, colors, office name populated). The practice manager's job is not to request, but to review and approve (or skip) the monthly package. Reactive requests still happen, but become the exception rather than the rule.

### How It Works — Step by Step

1. At the start of each month, SGA Studio (automated) publishes a content package to each practice: 4 TV slides (rotating seasonal/promotional), 8 social posts (2/week), 1 patient email template, 1 appointment reminder.
2. The practice manager receives a notification: "Your March content is ready for review. Approve in one click, or make changes."
3. They open a simple preview on mobile. Three actions: Approve All / Skip This Month / Request a Change.
4. 85% approve. 10% skip. 5% request a change — those go into the studio's human designer queue.
5. On approval, assets automatically deploy: TV slides upload to the practice's display system, social posts schedule in their social tool, patient email sends on the pre-planned date.
6. For practices that want something not in the package (a one-off flyer, event collateral), a simple request form exists — but the monthly package covers 70-80% of needs, so the request volume drops dramatically.

### Strengths

- Eliminates 70-80% of reactive intake requests entirely — replaces them with proactive delivery
- Most direct path to the studio's long-term proactive posture (vs. reactive order-taker)
- The medium question is already answered — TV slides, email, and social are in the package by default; print is not
- Very low friction for the practice manager — approve once a month is easier than any intake flow
- Content quality is higher because the studio controls production, not the practice

### Risks

- Practices may not feel heard — "I don't want these generic templates, I want what I asked for"
- Local promotions and events won't be covered by a corporate content calendar without significant customization
- Requires a content planning operation inside SGA Studio that doesn't currently exist
- "Approve or skip" doesn't serve practices with unique needs (specialty promotions, doctor referral programs, community events)
- Adoption risk: if practices don't engage with the monthly package, the studio has done work that no one uses

### Feasibility

- **Technical:** Medium. Requires content calendar infrastructure, automated distribution to TV/social/email systems, and approval workflow. More integration complexity than a portal.
- **Organizational:** High lift. SGA Studio must build the capacity to produce a monthly content calendar for 160+ practices. Even with automation, the creative production is significant upfront.
- **Time to v1:** 14-18 weeks. Requires content operations infrastructure before first packages can deploy.

---

## Solution D — The Franchise Brand Portal (Dedicated Platform)

**"Marq/Lytho/Papirfly — purpose-built for this exact problem"**

### Overview

Rather than building a custom solution, deploy a purpose-built franchise brand management platform (Marq, Papirfly, or Lytho) that already solves the multi-location brand problem out of the box. These platforms provide: brand-locked template libraries, per-location brand kits, structured intake flows with approval routing, digital asset management, and print vendor integrations — all in a single SaaS product. SGA configures and administers it; practices log in and use it.

### How It Works — Step by Step

1. SGA contracts with Marq (or equivalent). Studio admin sets up 160 practice workspaces, uploads brand kits per practice, builds the template library (business card, flyer, TV slide, etc.).
2. Practice manager logs in to Marq. Sees their practice's branded template library. They select a template. Editable fields unlock (offer text, dates, CTA). Brand elements are locked.
3. They submit. Marq's built-in approval workflow notifies the CD via email or mobile. One click approve.
4. Approved asset: downloadable PDF/PNG, or direct print order through Marq's integrated vendor network (Gelato, etc.).
5. Studio admin sees usage analytics, brand compliance reports, and which practices are active.
6. AI-enhanced features (if using Lytho or newer Marq capabilities): text suggestions for offer headlines, image replacement from the DAM, copy-length fitting.

### Strengths

- Proven at scale — Marq is used by franchises with 1,000+ locations; the platform handles the complexity
- Fastest to market — configuration, not custom development; no engineering team needed
- Built-in DAM, approval workflows, vendor connections — SGA doesn't have to build any of it
- Audit trail and brand compliance reporting for the studio
- Vendor integrations already live — no API work for print ordering

### Risks

- Platform cost: Marq enterprise licensing is significant for 160 locations (typically $50-150/location/year range)
- Less flexibility: out-of-the-box platforms don't do custom conversational AI intake or the "challenge the medium" logic SGA wants
- Still requires a login — same adoption risk as Solution B
- Doesn't solve the proactive content calendar problem — it's reactive by design
- Lock-in risk: migrating away from a platform after 2-3 years of template/asset investment is painful

### Feasibility

- **Technical:** Very high. No custom development. Configuration and migration project.
- **Organizational:** Medium. Vendor selection, contract negotiation, template migration, 160-location onboarding.
- **Time to v1:** 6-10 weeks (fastest option). Pilot 20 practices in month 2.

---

## Solution E — The Embedded Trigger System

**"Creative requests shouldn't start with a request"**

### Overview

The most radical departure: eliminate the intake step entirely for a large class of creative needs by building direct integrations between the practice's existing operational systems (practice management software, patient communication tools, appointment scheduling) and the SGA Studio workflow. When a patient hits their 3rd visit, it automatically triggers a review request follow-up. When a liner special is scheduled in the practice's ops system, it auto-triggers an in-office TV slide and patient email. When a new doctor referral relationship is created in the CRM, it auto-generates a referral introduction email. Creative happens because of what practices do, not because of what they request.

### How It Works — Step by Step

1. SGA Studio maps the 10 most common creative triggers (patient review ask, procedure promotion, seasonal event, new service launch, holiday hours, appointment reminder, doctor referral introduction, online review response, new patient welcome, renewal follow-up) to automated workflows.
2. Each trigger is connected to the practice's operational system via API (Dentrix, Eaglesoft, Weave, Podium, or similar).
3. When a trigger fires, the system checks the practice's Canva brand kit, selects the appropriate template, populates it with the specific data (patient name, procedure, dates), and generates the asset.
4. Asset goes to the CD queue for a 60-second review (or is auto-approved if it's a standard template that has already been approved for that practice).
5. Asset deploys automatically to the right channel — patient text, email, TV slide, social post — without the practice manager touching it.
6. Practice manager gets a weekly digest: "Here's what your Studio generated for you this week. 12 review requests sent. 1 promotional TV slide deployed. 4 new patient welcome emails sent."

### Strengths

- Eliminates intake almost entirely — creative happens as a byproduct of operations
- The medium question is already answered — the trigger determines the channel
- Review requests, appointment reminders, and referral comms go digital by default — no print
- Highest ROI over time: once built, it runs without ongoing practice manager involvement
- Addresses the "reactive vs. proactive" question most directly — the studio is now generating content based on what's happening in the practice, not waiting to be asked

### Risks

- Highest technical complexity: requires integrations with multiple practice management systems (Dentrix, Eaglesoft, Curve, etc.) across 160 locations — a significant engineering investment
- Practices are protective of their operational system access — some will resist API integrations
- Doesn't cover custom or one-off creative needs (event signage, campaign creative, new service launches) — a separate intake still required for those
- If a trigger fires incorrectly (wrong data, wrong patient), an automated asset goes out that should not have — quality risk is harder to catch when humans are out of the loop
- Requires the broadest organizational readiness: 160 practices must connect their systems, which requires IT support per location

### Feasibility

- **Technical:** Low-medium. Multiple integration points, API dependencies per practice management system, significant QA required.
- **Organizational:** Low. Requires practice-level system access and IT coordination — high friction to deploy.
- **Time to v1:** 20-28 weeks minimum, and only for a narrow set of triggers.

---

## Solution Comparison Matrix

| Criterion | A: Conversational Agent | B: Self-Service Portal | C: Proactive Push | D: Franchise Platform | E: Embedded Triggers |
|-----------|------------------------|----------------------|-------------------|----------------------|---------------------|
| Addresses SQ1 (self-serve) | ★★★★★ | ★★★★ | ★★★★★ | ★★★★ | ★★★★★ |
| Addresses SQ2 (AI brand quality) | ★★★★ | ★★★ | ★★★★ | ★★★ | ★★★★ |
| Addresses SQ3 (reduce print 30-40%) | ★★★★★ | ★★★ | ★★★★★ | ★★ | ★★★★★ |
| Addresses SQ4 (DAM foundation) | ★★★ | ★★★★ | ★★★ | ★★★★★ | ★★★ |
| Addresses SQ5 (vendor consolidation) | ★★★★ | ★★★★ | ★★★ | ★★★★★ | ★★★ |
| Feasibility (build complexity) | ★★★ | ★★★★★ | ★★★ | ★★★★★ | ★★ |
| Speed to pilot | ★★★ | ★★★★ | ★★ | ★★★★★ | ★ |
| Practice manager experience | ★★★★★ | ★★★ | ★★★★★ | ★★★ | ★★★★★ |
| Studio team empowerment | ★★★★ | ★★★ | ★★★★★ | ★★★ | ★★★★★ |
| Strategic boldness | ★★★★ | ★★★ | ★★★★★ | ★★ | ★★★★★ |

**Totals:** A: 42 | B: 36 | C: 41 | D: 39 | E: 38

*(Note: totals alone don't determine the recommendation — tradeoffs and sprint question alignment matter more.)*
