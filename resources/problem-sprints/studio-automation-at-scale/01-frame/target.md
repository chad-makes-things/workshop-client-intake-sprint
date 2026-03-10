# Target Recommendation

**Sprint:** Studio Automation at Scale
**Agent:** Problem Mapper
**Date:** 2026-03-10

---

## The Problem Map, Annotated

```
1. Notice Need
2. Submit Request  ◄━━━━ [TARGET ZONE STARTS HERE]
3. Studio Interprets
4. Asset Check     ◄━━━━ [UPSTREAM FAILURE POINT]
5. Design Production
6. Internal Review
7. Practice Approval
8. Route to Vendor  ◄━━━━ [TARGET ZONE ENDS HERE]
9. Print Production
10. Usage at Practice
```

---

## Recommended Target

**The intake and routing moment — Steps 2 through 4 on the problem map.**

Specifically: **the moment a practice manager submits a creative request and the system determines what they actually need, whether print is the right medium, whether the assets exist to fulfill the request, and what workflow to trigger.**

---

## Why This Target

### 1. It's the highest-leverage intervention point

Everything downstream — design quality, vendor routing, digital vs. print decision, turnaround time — is determined by what happens in this moment. A bad intake produces a bad brief, which produces a bad design, which produces a revision loop, which delays the vendor order, which arrives wrong. Fix the intake and you fix the cascade.

### 2. It's where the biggest opportunity is untouched

Every other part of the system has at least a partial solution: Canva templates exist, a merch vendor exists, a review process exists. But **there is no structured intake**. Requests come in via text, email, Slack, and hallway conversation — and then a human designer has to parse them. This is the biggest gap, and it's fully greenfield.

### 3. It's where AI has the most natural role

The intake is a conversation. AI is designed for conversations. A smart chatbot can:
- Understand what the practice is trying to accomplish (not just what format they asked for)
- Challenge the medium ("Why print — here's a digital option that does the same thing for free")
- Check the DAM for existing assets and flag gaps before design starts
- Route the request to the right workflow: automated Canva generation for templates, human designer for custom work, digital channel for things that shouldn't be print

### 4. It immediately validates or invalidates the most critical sprint questions

- **SQ1** (can a non-designer self-serve?) — the intake is where this is proven or broken
- **SQ3** (can we reduce print 30-40%?) — the intake is where the medium challenge lives
- **SQ4** (does the DAM exist and work?) — the intake is the first place a missing asset is discovered

### 5. It's a forcing function for everything else

Choosing this target forces the sprint to answer: What template library exists? What does the DAM look like? What are the 10 most-requested collateral types? How does the quality gate work? Solving the intake correctly *requires* solving the surrounding system.

---

## Why Not Other Targets

| Alternative Target | Why Not |
|-------------------|---------|
| **The design production layer (Step 5)** | Already partially solved with Canva + agent workflow. The bigger problem is upstream — the intake is too chaotic and assets are missing. Automating design before fixing intake is adding speed to a broken process. |
| **The vendor procurement layer (Step 8-9)** | Real pain point, but it's a downstream problem. Even with a perfect vendor, if the intake is broken, wrong files arrive at the wrong vendor with the wrong specs. Fix intake first. |
| **The digital channel migration (Step 10)** | This is important strategically, but it's a behavior-change problem, not a systems problem. The best way to drive digital adoption is through the intake — not by pushing digital channel education separately. |
| **The DAM / asset foundation (Step 4)** | Critical, but it's a prerequisite, not the target. The DAM must be built alongside the intake system, but it's not the thing we're designing the solution around. |

---

## Target Scoring

| Criterion | Score (1-5) | Notes |
|-----------|------------|-------|
| Addresses highest-risk sprint question (SQ1) | 5 | Intake is where SQ1 lives or dies |
| Has the most direct path to goal achievement | 5 | Every other improvement flows from a working intake |
| Is feasible to prototype and test | 4 | Chatbot + Canva generation is buildable in weeks |
| Is most likely to surprise and delight the user (practice manager) | 5 | Going from "email the designer and wait 5 days" to "done in 48 hours" is dramatic |
| Represents the least-solved part of the current system | 5 | Fully greenfield; no current intake structure exists |

**Total: 24/25**

---

## What the Sprint Will Design

An **AI-powered creative intake and routing system** that:

1. **Understands intent** — "We want to promote our liner special" before "We need a flyer"
2. **Challenges the medium** — "Here's what a patient email would look like instead. Want to see it?"
3. **Audits assets** — "Your practice brand kit is in Canva. You're missing a photography set — want to use a stock option for now?"
4. **Triggers the right workflow** — Automated Canva generation for templates / human designer queue for custom / digital channel deployment for non-print
5. **Routes to approval** — Sends the CD a mobile-first preview with one-tap approve
6. **Closes the loop** — Sends approved asset to practice + vendor (if print) with correct specs

---

## User Confirmation Required

**Before the sprint proceeds to Phase 4 (Sketch), you must confirm or redirect the recommended target.**

> **Recommended target:** The creative request intake and routing moment (Steps 2-4 of the problem map) — building a smart chatbot-driven intake that understands what the practice needs, challenges the medium, checks for missing assets, and triggers the right downstream workflow.

Please confirm: Does this feel like the right place to focus? Or would you redirect to a different part of the map?
