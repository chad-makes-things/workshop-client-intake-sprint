# Storyboard

**Sprint:** Studio Automation at Scale
**Agent:** Decision Architect
**Date:** 2026-03-10
**Selected Direction:** A — Conversational Studio Agent (with email channel)

---

## Storyboard Overview

**User:** Ashley, office manager at Towne Lake Dental (Gen4 practice, 2 dentists, 3,200 active patients)
**Entry point:** Ashley wants to promote a teeth whitening special running next month. She's done this before by texting the designer — who took 4 days to respond and then Staples got the size wrong on the flyers.
**System:** The SGA Studio Agent — reachable via Teams, Slack, or email at `studio@sgadental.com`

---

## Step 1 — Ashley Sends a Request

**Scene:** Ashley is at the front desk between patients. She opens her email and types to `studio@sgadental.com`:
> "Hi — we're running a teeth whitening special in April, $299 for Zoom whitening. Can you make us some materials?"

**Action:** Email sent to the studio agent inbox. Subject line auto-detected: "Whitening Special — Marketing Materials."

**System response:** The agent reads the email within seconds. It recognizes: a promotional request, a specific offer, a specific service. It checks Ashley's practice profile — Towne Lake Dental, Gen4 platform, Canva brand kit status: ✅ complete (logo, brand colors, fonts, 6 photos on file).

**Agent reply (via email):**
> "Hi Ashley! Got your request for the whitening special. A few quick questions so I can get this right:
> — Who's the primary audience? (existing patients, new patients, or both?)
> — Do you have a deadline for when materials need to be ready?
> — Any specific doctor you want featured, or keep it practice-branded?
> Also — before I put together a flyer, I want to show you something that might work even better. Can I send you a preview of a patient email + in-office TV slide? No printing required, and I can have them ready today. Happy to do both if you still want print too."

**User reaction:** Ashley thinks, "Oh interesting — I've been meaning to get something on the TV anyway." She replies with her answers and says yes to seeing the digital option.

**Critical detail:** The medium challenge happens here — in the first reply, not as an afterthought. The agent doesn't wait to be asked about digital alternatives.

---

## Step 2 — Intent Confirmed, Assets Verified

**Scene:** Ashley's reply comes in. Agent reads it:
> "Existing patients mainly, but new patients too if possible. Deadline is March 28 so we have it for April 1. No specific doctor. Yes, show me the digital option — and yes let's do print too just to have something at the desk."

**Action:** Agent processes the complete brief:
- Service: Zoom whitening special
- Offer: $299
- Audience: Existing + new patients
- Deadline: March 28 (18 days)
- Formats requested: Patient email, TV slide, desk flyer (print)
- Assets: Towne Lake Dental brand kit loaded — logo, colors (#1B4F72 deep blue, #F5F5F5 warm white), fonts (Playfair Display + Lato), photo: "team-smiling-1.jpg" flagged as most relevant

**System response:** Agent confirms intake is complete. No follow-up questions needed. Triggers three parallel generation jobs in Canva:
1. Patient email header (600px wide, practice branded, whitening offer)
2. TV slide (1920×1080, landscape, rotating display format)
3. Desk flyer (4×6 print, landscape, $299 offer CTA)

**Critical detail:** The agent doesn't ask more questions than needed. Three questions in Step 1, full brief established. The practice manager isn't doing design work — they're answering operational questions.

---

## Step 3 — AI Generation in Canva

**Scene:** Behind the scenes. The agent triggers the Canva API with:
- Practice ID: towne-lake-dental
- Brand kit: loaded (colors, fonts, logo)
- Template IDs: [email-header-v3, tv-slide-promo-v2, desk-flyer-4x6-v1]
- Variables: {offer: "$299 Zoom Whitening", cta: "Call to book", photo: "team-smiling-1.jpg", dates: "April 2026"}

**Action:** Canva generates three draft assets in ~45 seconds. All three use locked brand elements — the agent cannot override the logo placement, font stack, or primary color. The offer text, photo, and CTA are populated from the brief variables.

**System response:** Three preview images generated. Agent packages them into a preview email.

**Critical detail:** Brand guardrails are invisible to Ashley — she only sees an output that looks like her practice. The locked template system means the CD's review is a quality check, not a redesign session.

---

## Step 4 — Practice Preview

**Scene:** Ashley receives an email from `studio@sgadental.com`:
> "Hi Ashley — here are your three April whitening special assets. Let me know what you think!
> [Preview: Patient Email] [Preview: TV Slide] [Preview: Desk Flyer]
> Everything looks right to you? Any text changes needed? (If it all looks good, I'll send it to the creative director for final sign-off — usually takes about an hour.)"

**Action:** Ashley opens the previews on her phone during lunch. The email header looks clean — her practice logo, the offer in big type, the team photo. The TV slide is perfect. The flyer — she wants to change "Call to book" to "Ask us today!"

**User reaction:** She replies: "These look great! On the flyer can you change the CTA to 'Ask us today' instead?"

**System response:** Agent makes the one change in Canva. Regenerates the flyer preview. Sends updated version back in under 2 minutes.

**Critical detail:** Revision cycles are text changes, not design sessions. Because the template handles layout, a CTA change is a variable swap — seconds, not hours.

---

## Step 5 — Creative Director Review (60-Second Gate)

**Scene:** The CD, Marcus, gets a push notification on his phone:
> "Studio Agent | Towne Lake Dental | 3 assets for review"
> [Thumbnail: email header] [Thumbnail: TV slide] [Thumbnail: flyer]
> ✅ Approve All | ✏️ Request Change | ❌ Reject

**Action:** Marcus opens the notification. Scans three thumbnails. All look on-brand. The flyer CTA change is visible. He taps "Approve All."

**System response:** Approval recorded with timestamp. All three assets move to deployment queue. Agent notified — triggers Step 6.

**Critical detail:** Marcus's job is a judgment call on a finished output — not a design session. The 60-second gate works because the template system did the brand governance before the output ever reached him.

---

## Step 6 — Digital Assets Deploy

**Scene:** 45 minutes after Ashley sent her first email.

**Action:** On CD approval, the agent triggers:
1. **Patient email:** Asset packaged and handed off to SGA's patient communication platform (Weave, Podium, or equivalent) with Ashley's patient list. Scheduled to send April 1.
2. **TV slide:** Uploaded to Towne Lake Dental's in-office display queue. Goes into rotation immediately — Ashley will see it on the waiting room screen tomorrow morning.

**System response:** Agent emails Ashley:
> "All three assets are approved! Here's what's happening:
> ✅ Patient email: scheduled to send April 1 to your active patient list
> ✅ TV slide: live in your waiting room display now
> ✅ Desk flyer: ready to order — [Order 50 copies from SGA Print] or [Download PDF]"

**User reaction:** Ashley clicks "Order 50 copies from SGA Print." She doesn't have to pick a vendor, find a file, or specify a size. The button pre-loads the correct file with correct specifications.

**Critical detail:** The TV slide going live today — same day the request was made — is the moment Ashley realizes this is different from what she was doing before.

---

## Step 7 — Print Order Fulfillment

**Scene:** Ashley confirms the print order. 50 desk flyers, 4×6, full color, glossy.

**Action:** The agent submits the approved print-ready PDF to the nominated vendor API (Gelato or equivalent) with:
- File: towne-lake-whitening-flyer-approved-20260310.pdf
- Spec: 4×6 in, full bleed, CMYK, 100lb gloss
- Quantity: 50
- Ship to: [Towne Lake Dental practice address on file]
- Estimated delivery: 5 business days

**System response:** Order confirmation returned from vendor. Agent forwards confirmation to Ashley:
> "Your 50 flyers are ordered! Estimated delivery: March 17. Tracking will come from [Vendor] directly.
> Is there anything else you need for the April whitening special?"

**User reaction:** Ashley says no. She notices the whole thing — email, TV slide, and flyer ordered — took less than an hour. She previously spent 4 days chasing this.

**Critical detail:** The vendor API connection means Ashley never had to know who the vendor is, what file format to use, or what size to order. She clicked one button. The studio never touched the print order.

---

## Step 8 — Studio Analytics + Loop Closed

**Scene:** SGA Studio dashboard, end of week. The Creative Director reviews the week's volume.

**Action:** Dashboard shows:
- 23 requests received (Teams: 8, Slack: 6, Email: 9)
- 19 fulfilled via agent automation (no designer involved)
- 4 escalated to human designer queue (custom requests: event signage × 2, new service launch × 1, request for photography session × 1)
- Medium breakdown: 71% digital-only, 22% digital + print, 7% print-only
- Print volume vs. prior month baseline: -34%
- Average turnaround: 52 minutes (vs. 4.2 days prior month)
- CD review time: avg 58 seconds per request

**System response:** The dashboard surfaces an insight: 6 practices haven't submitted a request in 90+ days. The studio admin can push a proactive check-in: "Hey, do you need any materials for [upcoming month]?"

**Critical detail:** The studio's role has shifted. The designer spent this week on 4 custom requests that actually require design talent — event signage, a service launch campaign. The agent handled everything else. The CD spent ~20 minutes total on quality review. The studio produced more output than the prior month with less designer time consumed on templated work.

---

## Storyboard Summary

| Step | What Happens | Who's Involved | Time |
|------|-------------|----------------|------|
| 1 | Ashley emails studio@sgadental.com; agent responds with medium challenge | Ashley + Agent | 2 min |
| 2 | Ashley confirms brief; agent verifies assets and triggers generation | Ashley + Agent | 5 min |
| 3 | Agent generates 3 Canva assets from brand kit + templates | Agent + Canva API | <1 min |
| 4 | Ashley reviews previews; requests one text change; agent updates | Ashley + Agent | 10 min |
| 5 | CD receives mobile notification; taps approve | CD Marcus | 1 min |
| 6 | Digital assets deploy; Ashley gets print order link | Agent + Patient Comms Platform + TV Display | 5 min |
| 7 | Print order submitted to nominated vendor; confirmation sent | Agent + Print Vendor API | 2 min |
| 8 | Studio reviews weekly analytics; proactive outreach to inactive practices | CD + Studio Admin | End of week |

**Total time from first email to digital assets live: ~52 minutes**
**Practice manager effort: ~15 minutes (2 emails + 1 button click)**
**Designer effort: 0 minutes**
**CD effort: 1 minute**
