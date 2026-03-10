# Problem Map

**Sprint:** Studio Automation at Scale
**Agent:** Problem Mapper
**Date:** 2026-03-10

---

## Actors

| Actor | Role | Pain Points |
|-------|------|-------------|
| **Practice Manager** | Identifies need, submits request, approves final output, orders/receives print | No standard intake channel; doesn't know if print is the right medium; frustrated by Staples; chases orders |
| **SGA Studio Designer** | Receives requests, designs assets, routes to vendors | Spends time on templated work that should be automated; bad briefs force revision loops; assets often missing for the practice |
| **Studio Creative Director** | Quality gate; brand governance | Single point of failure for approvals; can't scale review if volume increases |
| **SGA Brand Administrator** | Manages DAM, brand standards, vendor relationships | No centralized DAM; asset quality inconsistent across 160 practices; vendor relationships are informal |
| **Print Vendor (Staples / local)** | Produces and ships physical materials | Staples: consistently painful; local printers: quality inconsistent; formats often wrong |
| **Digital Channel (TV, Email, Patient App)** | Delivers digital versions of what print currently does | Underutilized; most practices don't know how to use in-office TV, patient email, or digital frames |
| **Doctor Referral Rep (PRD)** | Hand-delivers printed referral materials to referring doctors | Has to physically carry flyers; digital equivalent would reduce print need |

---

## Problem Map Journey

```
[PRACTICE MANAGER]
       |
       v
1. NOTICE NEED
   "We're running a liner special. We need a flyer."
   [Problem: Reactive, not proactive. Need often isn't right medium.]
       |
       v
2. SUBMIT REQUEST (Current State)
   Email / Slack / text / hallway conversation to studio
   [Problem: No standard intake. Missing info. Wrong format requests.]
       |
       v
3. REQUEST RECEIVED BY STUDIO
   Designer interprets the need, guesses at context
   [Problem: Brief incomplete. Studio has to chase: "What's the offer? What size? When?"]
       |
       v
4. ASSET CHECK
   Designer looks for practice logo, photos, brand colors
   [Problem: ~30-40% of practices have inadequate assets. Some have only a low-res logo.
    For these practices, design can't even start without an upstream asset problem.]
       |
       v
5. DESIGN PRODUCTION
   Designer builds in Canva or Adobe. Most work is templated but done manually.
   [Problem: Manual production doesn't scale. 1 designer + 1 contractor = ~40 design-hours/week.]
       |
       v
6. INTERNAL REVIEW
   Creative Director reviews for brand standards
   [Problem: Single point of failure. CD is a bottleneck. No SLA. Turnaround undefined.]
       |
       v
7. PRACTICE PREVIEW / APPROVAL
   Design sent to practice manager for approval. Revisions loop.
   [Problem: Practices often request changes. Each revision cycle adds 1-3 days.]
       |
       v
8. ROUTE TO VENDOR
   Studio routes approved design to Staples or local printer.
   [Problem: Staples = painful. Wrong sizes, delays, account management issues. "Most painful thing in my career."]
       |
       v
9. PRINT PRODUCTION & DELIVERY
   Printer produces and ships to practice.
   [Problem: No tracking. Wrong format. Delayed. Practice manager chases.]
       |
       v
10. USAGE AT PRACTICE
    Practice receives print materials and uses them.
    [Problem: Sometimes wrong medium. Flyers stack on the front desk. Patients don't take them.
     Could have been a digital frame rotation, a patient email, or a QR code instead.]
```

---

## High-Risk Journey Moments

| # | Moment | Risk Type | Sprint Question Link |
|---|--------|-----------|---------------------|
| **2-3** | Request submission → Studio interprets | **Missing context / wrong medium** | SQ1, SQ3 |
| **4** | Asset check → Asset doesn't exist | **Upstream failure blocks design** | SQ4 |
| **5** | Design production | **Scale ceiling: 1 designer can't serve 160 locations** | SQ1, SQ2 |
| **8-9** | Route to vendor → Delivery | **Vendor pain creates delays and quality failures** | SQ5 |
| **10** | Usage at practice | **Wrong medium; print ROI is poor** | SQ3 |

---

## The Failure Mode We Most Want to Avoid

A practice manager texts the designer: "Hey can you make a flyer for our liner special?" The designer doesn't have their logo. Chases it for 2 days. Makes the flyer manually in Canva. The CD is on a call. Approval takes 3 days. Routes to Staples. Staples screws up the size. Flyers arrive 2 weeks after the promotion ended. The PRD has been hand-delivering a low-res printout from the office printer in the meantime.

**This is the current state for a meaningful percentage of the 160 locations.**

---

## The Success Mode We're Designing Toward

A practice manager opens a chat: "I want to promote our liner special to patients." The system asks: "Who's the audience — existing patients or new ones? When is the promotion? Do you have a photo of the team?" It then says: "For existing patients, we'd recommend a patient email + in-office TV slide instead of a flyer — same reach, no printing cost. Want to do both?" If they say yes, it generates both assets in Canva from approved templates with their logo and practice brand colors, routes to the CD for 60-second quality review, and deploys the email and TV slide same-day. If they still want print, it routes to the approved vendor with correct specs. Done. The designer touched nothing.
