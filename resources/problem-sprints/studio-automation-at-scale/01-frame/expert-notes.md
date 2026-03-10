# Expert Notes

**Sprint:** Studio Automation at Scale
**Agent:** Problem Mapper
**Date:** 2026-03-10
**Source:** Direct conversation with internal expert (Creative/Brand lead, SGA)

---

## Key Insights from the Expert Session

### 1. The Asset Baseline Problem Is the Pre-Problem

Before you can automate design, you need assets to automate with. The spectrum of asset quality across 160 practices is extreme:

- **A few (10-15%):** Strong visual identity — logo in all formats, brand colors defined, photo sets, video, social media library. These practices could use an automated system today.
- **Many (50-60%):** "Decent" — some logo files, maybe a photo or two. Functional but limited. Templates would work, but photography gaps would show.
- **Some (25-35%):** Minimal — "a color and a logo." Not enough to build anything that looks good.
- **A few:** No assets at all. No documentation.

**Implication for the sprint:** Any solution must either (a) accommodate thin-asset practices with a graceful fallback, or (b) include an upfront lift to create brand foundation packages for underserved practices before the system goes live. The expert confirmed: "There's probably an upfront lift for asset creation, and then it's super repeatable."

---

### 2. Gen4 Has Done the Onboarding Asset Package Well

When a practice joins Gen4, they receive a standard asset set: reminder cards, business cards, logo (with all file formats — stacked, horizontal), brand guidelines. This is a playbook that works and should be templated for all incoming SGA practices.

**Implication:** The Gen4 onboarding asset model is the template. Extend it to legacy SGA practices and MODIS partners who were never given a proper brand foundation. This is a one-time backfill project, not an ongoing burden.

---

### 3. Design Capacity Is the Core Constraint — But Requests Are Fewer Than You'd Think

Current studio team:
- 1 web developer (primarily handles web for practices)
- 1 designer (handles all creative production)
- 1 part-time contractor (~20 hours/week)

The expert noted: "There's not a ton of requests. You'd probably be fine with one backup person, especially if you have templates they can choose from. You're only ordering business cards once a year. And they're always the same."

**Critical insight:** The volume isn't the primary problem. The *friction* and *waste* per request is the problem. A request that should take 10 minutes takes 5 days because of intake chaos, asset hunting, vendor pain, and approval bottlenecks. Fix the workflow, and a small team can serve 160 locations.

---

### 4. Canva + Agent Team Is the Existing Nucleus

A small Canva team is already creating templates. A separate "agent team" uses the Canva CI to generate assets from those templates, with a "shorthanded review process." This is the seed of the automated studio — it just hasn't been systematized or scaled.

**Key Canva insight:** Canva can serve as both the DAM (brand assets, colors, fonts locked at the brand level) and the design production tool. Enterprise Canva accounts allow brand kits, template libraries, and team access. The expert noted Canva's ability to go from template to print-ready digital output.

**Implication:** Don't rebuild what exists. Extend and formalize: enterprise Canva account with per-practice brand kits → structured template library → agent-powered generation → human quality gate → direct vendor routing.

---

### 5. Staples Is a Known Failure State

"Utilizing even enterprise-level at Staples has been the most painful thing I've ever dealt with in my career." — Direct quote from expert.

Gen4 currently routes practices back to local printers because Staples is so bad. SGA legacy has a decent merch vendor (for gifts, branded items) but it's "one-off, not a nominated system." There is appetite for a curated, locked-in vendor structure where practices order directly — the expert confirmed: "We could pick a vendor, yes. The offices would know: hey, this is the only way to print, because we get it in full."

**The desired state:** Practices submit approved designs. Vendor auto-receives the order. Materials arrive. Studio's role = set up vendor relationships and audit quality, not touch each order.

---

### 6. The Medium Question Is the Biggest Strategic Opportunity

The expert's most powerful insight: practices are ordering print collateral that doesn't work, when digital alternatives would be cheaper, faster, and more effective.

Examples:
- **Review request business cards:** Practices print cards to hand patients when asking for a review. Could be a text/QR code instead — more trackable, zero print cost.
- **Doctor referral flyers (PRD program):** Specialty practices hand-deliver printed referral flyers to referring doctors. If a digital referral program (email, patient portal) handles the communication, the PRD doesn't need to carry a brochure.
- **Promotional flyers:** Most dental offices have TVs in waiting rooms. An in-office TV slide rotation (Apple TV, Dental Lifeline Network, practice management software screen) replaces a flyer — and patients actually see it.

**The expert wants discernment built into the intake:** "I almost want there to be a way of talking them out of even doing print — asking questions before the request even becomes a request. What are you going to do with flyers? Where are you going to deliver them? Are you just stacking them on the front desk?"

**The proactive shift:** The expert explicitly asked: "Does capacity change? Does output change? Does requirement or execution change if we're no longer in a reactive posture, but now in a proactive posture?" The answer is yes — if the studio is pushing out a content calendar and communication templates, practices pull less one-off collateral.

---

### 7. SharePoint Exists but Isn't Working as a DAM

Assets "end up in SharePoint" but the system doesn't function as a proper DAM. No taxonomy, no search, no version control, no per-practice organization. "I don't know if there's a Canva account" — the expert wasn't certain whether a formal Canva enterprise account even exists.

**Implication:** The DAM is effectively greenfield. Starting with Canva as the anchor makes sense — it's already in the workflow, it's accessible to non-designers, and it has brand governance features.

---

### 8. SGA Has Three Distinct Brand Universes to Serve

The combined SGA entity brings three different platforms with different brand cultures:

| Platform | Brand Maturity | Asset State | Special Notes |
|----------|---------------|-------------|---------------|
| **Gen4** | Best — structured onboarding, brand kits, templates | Good for most practices | Has existing Canva template workflow |
| **MODIS** | High for flagship (Innovative Dental) — lower for others | Varies widely | Implant-specialty brand; more premium aesthetic required |
| **SGA Legacy** | Mixed — community dentistry DNA | Many practices underserved | SE market; lower tech adoption likely |

The studio automation system must accommodate all three — with different brand kits, different asset quality levels, and potentially different communication channels by platform.

---

## Open Questions the Sprint Must Answer

1. Is Canva the right platform for both DAM and design generation, or does a dedicated DAM (Bynder, Brandfolder, Canto) serve the asset management layer better?
2. What is the print volume baseline? How many print orders per month across 160 locations? (This determines the ROI of vendor consolidation.)
3. Who owns the quality gate? Creative Director alone, or can a tiered review be built where the AI flags the 5% that needs human eyes and auto-approves the 95%?
4. What's the fastest path to brand foundation packages for the ~50 practices with inadequate assets?
5. How does the chatbot intake live in the workflow — is it a standalone tool, a Teams/Slack integration, or inside an existing practice management platform?
