# SGA Dental Partners -- Re-engagement Sequence

**Trigger:** No email opens for 60+ days (B2B audiences), tagged `cold_60d`
**Primary Audience:** Practice owners and associates who previously engaged but went dormant
**Sequence Length:** 5 emails over 21 days
**Funnel Tag:** `pa_reengagement` / `dr_reengagement`
**Voice:** Sage archetype -- respectful, low-pressure, value-forward
**Goal:** Reactivate dormant leads with new information, updated proof, or changed circumstances; clean the list of permanently disengaged contacts to protect deliverability

---

## Email 1: "We Noticed You Went Quiet" -- Respectful Check-In

**Send:** Day 0 (triggered by `cold_60d` tag)

**Subject Line Options:**
1. Still thinking about it? (No judgment. Just a check-in.)
2. A lot has changed since we last talked
3. Quick question, Dr. {{first_name}} (takes 10 seconds)

**Preview Text:** We are still here. And things have moved.

---

**Body:**

Dr. {{first_name}},

It has been a while since we heard from you. That is completely fine.

A decision about your practice's future is not something you rush. The average partnership conversation takes 6-18 months from first interest to signed agreement, and some dentists take longer. The timeline is yours.

But I wanted to check in for a simple reason: the landscape has shifted since we last connected, and some of what has changed might matter to you.

Rather than assume what is on your mind, I have a quick question:

**What is the biggest priority for you right now?**

[ ] I am still evaluating partnership options -- send me what is new
[ ] My situation has changed -- I am not exploring partnerships right now
[ ] I am ready to have a conversation
[ ] I would rather receive less email from SGA

Click the one that fits and I will adjust accordingly. No sales pitch. No follow-up call. Just the right information at the right cadence.

[Click to Answer]

Talk soon,

[Sender name]

P.S. -- If you clicked "ready to have a conversation," here is the direct link to schedule: [Schedule a Confidential Conversation]. If you clicked anything else, I will respect that completely.

---

**CTA Button:** Click to Answer (survey link)
**Link Destination:** Single-question survey page with the four options

**Notes:**
- Respectful re-engagement -- does not assume why they went quiet
- Single-question survey re-segments leads based on current intent
- Option 4 ("less email") protects deliverability and signals respect
- Routing logic:
  - Option 1: Continue re-engagement sequence (Emails 2-5)
  - Option 2: Tag `paused_not_now`, move to quarterly-only cadence
  - Option 3: Immediate BD notification, priority scheduling
  - Option 4: Reduce to monthly digest only, tag `low_frequency`
- A/B test: Subject line 1 (warm + zero pressure) vs. Subject line 3 (micro-commitment)
- Non-openers of this email receive Emails 2-3 before final suppression decision

---

## Email 2: "Here's What Changed" -- New Developments and Proof Points

**Send:** Day 5

**Subject Line Options:**
1. Three things that happened at SGA since you last checked in
2. New partners. New numbers. New reasons to look again.
3. The update you missed (and one number that changed)

**Preview Text:** We have not been standing still. Here is the evidence.

---

**Body:**

Dr. {{first_name}},

While you have been focused on your practice (as you should be), a few things have happened at SGA that are worth knowing about:

**1. The platform grew.**

SGA now supports 100+ partner practices across multiple states. Every new practice added strengthens the network -- shared purchasing power, deeper recruiting pipelines, more clinical perspectives in the peer community, and a broader equity base for every partner.

For you, this means: the platform your equity would be part of is larger and more diversified than it was when we last connected.

**2. The education platform expanded.**

The Pikos Institute, already the gold standard in implantology education, has expanded its course offerings for SGA partners. The 4Front Symposium -- our annual peer-learning event where "senior clinicians brainstorming together naturally elevated the ideas" -- continues to grow. And the Individual Growth Engine has matched more partner-mentor pairs than ever.

For you, this means: the clinical development infrastructure that makes SGA different from every other DSO is getting stronger, not stagnating.

**3. The numbers keep holding.**

- 85% staff retention rate across the SGA network
- The Thurston Group's track record: 38 years, 8 dental and healthcare platforms, $4B+ returned to partners
- Dr. Cavendish's practice continues to outperform: the 16.5% revenue growth was not a one-time spike. It was the beginning of a trajectory.

For you, this means: the proof points are not just marketing numbers. They are sustained outcomes.

If these developments change the equation for you:

[Schedule a Confidential Conversation]

If not, I will continue to share updates as they happen. The door does not close.

Talk soon,

[Sender name]

P.S. -- If you want to talk to one of the new partners directly -- someone who made the decision recently and can tell you what the first 90 days were like -- just reply and I will connect you.

---

**CTA Button:** Schedule a Confidential Conversation
**Link Destination:** Calendar booking page

**Notes:**
- Value-based re-engagement -- provides genuinely new information
- Three updates structure is easy to scan (most re-engagement emails need to be readable in 30 seconds)
- "For you, this means" framing makes each update personally relevant
- This email should be updated with fresh data each quarter
- A/B test: Subject line 1 (specific number) vs. Subject line 2 (proof-forward)
- Internal note: Marketing team should maintain a "latest developments" doc for quarterly email refresh

---

## Email 3: New Partner Stories -- Fresh Social Proof

**Send:** Day 10

**Subject Line Options:**
1. A partner who joined last quarter just told us this
2. "I wish I had done this three years ago." (New partner, real quote.)
3. The newest SGA partner's first 90 days -- in their words

**Preview Text:** A fresh perspective from someone who was in your shoes recently.

---

**Body:**

Dr. {{first_name}},

The best evidence that a partnership works is not in the pitch deck. It is in the words of someone who made the decision recently -- while the memory of the before-state is still fresh.

Here is what one of our newest partners shared about their first 90 days:

[NOTE: Insert most recent partner testimonial here. Template below.]

---

**Before partnering:**

"[Quote about their before-state -- administrative burden, exhaustion, isolation, financial concentration risk]."

They had been practicing for [X] years. Revenue of $[X]. A team of [X]. By every external measure, successful. Privately, carrying the weight of everything.

**Why they chose SGA:**

"[Quote about what made SGA different -- Thurston track record, education platform, peer community, clinical autonomy guarantee]."

**90 days in:**

"[Quote about the tangible changes -- operational relief, first mentor meeting, team's response to transition, the moment it felt real]."

[Practice name] is still [practice name]. The team is still there. The patients still see their doctor first.

What changed: [specific operational improvements, specific relief moments, specific outcomes].

---

The reason I share this: when you were first exploring SGA, these stories may not have existed yet. The proof base has grown. The partners are newer. The relevance to your situation is sharper.

If hearing from someone who was in your position recently would help:

[Schedule a Confidential Conversation]

Talk soon,

[Sender name]

P.S. -- We can connect you with this partner directly if you would like to hear the unvarnished version. Just reply and ask.

---

**CTA Button:** Schedule a Confidential Conversation
**Link Destination:** Calendar booking page

**Notes:**
- TEMPLATE EMAIL: The bracketed sections should be filled with the most recent partner testimonial
- This email should be refreshed quarterly with the newest partner story
- The power of this email depends on recency -- "someone who just did this" is more persuasive than a two-year-old case study
- Follows the Customer Transformation Story Template from brand_story.md
- A/B test: Subject line 1 (recency + curiosity) vs. Subject line 2 (VoC quote + specificity)
- Offer to connect directly with the new partner -- maximum trust signal

---

## Email 4: Market Update -- What Has Shifted in the Industry

**Send:** Day 15

**Subject Line Options:**
1. The dental acquisition market just shifted. Here is how.
2. What happened to practice valuations in the last quarter
3. Market intelligence for practice owners (Q[X] 2026 update)

**Preview Text:** Fresh data from the field. No spin.

---

**Body:**

Dr. {{first_name}},

Quick market update for practice owners who are keeping an eye on the landscape:

**Valuation trends:**

[NOTE: Insert most recent quarterly data. Template below.]

- Mid-market practice multiples (1-3 doctors, $1M-$3M revenue): [X-Xx EBITDA]
- Group practice multiples (4+ doctors, $3M+ revenue): [X-Xx EBITDA]
- Year-over-year trend: [up/flat/down by X%]
- Key driver: [PE capital deployment / interest rates / market saturation / buyer competition]

**Staffing market:**

- Hygienist recruiting difficulty: [X%] of practices reporting "extremely challenging" (ADA data)
- Associate availability: [current state]
- Impact: Practices with institutional recruiting support (benefits, career paths, infrastructure) continue to attract talent that solo practices cannot access

**Consolidation pace:**

- Estimated DSO-affiliated practice growth: [X]% year-over-year
- Number of qualified IDSO buyers in market: fewer than 100 (per Group Dentistry Now)
- Trend: [current observation about buyer selectivity, deal quality, or market dynamics]

**What this means for you:**

[Current quarter's specific implications for practice owners evaluating partnerships]

I share these updates because your decision should be informed by current data, not by the market as it existed when you first started exploring. The landscape moves. Your assessment should move with it.

If you want to discuss what these trends mean for your specific practice:

[Schedule a Confidential Conversation]

Talk soon,

[Sender name]

P.S. -- Want this data quarterly without the rest of the emails? Reply "market updates only" and I will adjust your preferences.

---

**CTA Button:** Schedule a Confidential Conversation
**Link Destination:** Calendar booking page

**Notes:**
- TEMPLATE EMAIL: Bracketed sections must be updated quarterly with fresh market data
- Pure value play -- positions SGA as the informed market analyst
- PS offers a reduced-frequency option, respecting the reader's preferences
- This email works even if the recipient has no intention of partnering right now -- it builds SGA's authority for the future
- A/B test: Subject line 1 (news hook) vs. Subject line 3 (newsletter-style branding)
- Internal process: Marketing team must provide quarterly data refresh for this email

---

## Email 5: Last Chance -- Before We Stop Emailing

**Send:** Day 21

**Subject Line Options:**
1. Should I keep writing to you? (Honest question.)
2. This is the last email -- unless you say otherwise
3. A final note from SGA, Dr. {{first_name}}

**Preview Text:** Your inbox matters. So does ours. One click tells me what to do.

---

**Body:**

Dr. {{first_name}},

I respect your time. And I respect your inbox.

Over the past few weeks I have shared new developments at SGA, fresh partner stories, and market data. If none of that resonated, that tells me something -- and I would rather ask directly than keep sending emails that do not serve you.

Here is what I need from you:

**One click. That is it.**

[ ] **Keep sending.** I am still interested. The timing is not right yet, but I want to stay informed. (I will move you to a monthly digest -- one email per month with the most important updates.)

[ ] **I am ready to talk.** Schedule me a conversation with your partnership team. (I will send you a direct calendar link within 24 hours.)

[ ] **Please stop.** I am not interested. Remove me from your email list. (Done immediately, no questions asked.)

[Click to Tell Me]

If I do not hear from you, I will make the decision for you: I will move you to a quarterly-only cadence (four emails per year) with the option to unsubscribe at any time. Your contact information stays private. Nothing changes about your relationship with SGA if and when you decide to explore a conversation in the future.

The door never closes. But I will stop knocking.

Respectfully,

[Sender name]
SGA Dental Partners

P.S. -- If you know a colleague who might benefit from the information I have been sharing, feel free to forward any of these emails. Sometimes the right person is not the one on the list.

---

**CTA Button:** Click to Tell Me (survey link)
**Link Destination:** Three-option survey page

**Notes:**
- Clean list management email -- critical for deliverability health
- Three clear options with immediate respect for the choice
- "I will stop knocking" language is direct without being guilt-inducing
- Routing logic:
  - Option 1: Tag `reengaged_monthly`, move to monthly digest
  - Option 2: Immediate BD notification, send calendar link within 24 hours
  - Option 3: Unsubscribe immediately, tag `unsubscribed_reengagement`
  - No response within 7 days: Tag `cold_unresponsive`, move to quarterly cadence
- PS activates referral even from departing contacts
- A/B test: Subject line 1 (direct question) vs. Subject line 2 (transparency)
- Deliverability impact: This email protects sender reputation by removing or reducing frequency for disengaged contacts

---

## Sequence Summary

| Email | Day | Purpose | Key Technique | Expected Outcome |
|-------|-----|---------|--------------|-----------------|
| 1 | 0 | Respectful check-in | Single-question survey | Re-segment by current intent |
| 2 | 5 | New developments | Value-first update | Reignite with fresh proof |
| 3 | 10 | Fresh social proof | Recent partner story | Relevance through recency |
| 4 | 15 | Market intelligence | Quarterly data update | Authority + informed urgency |
| 5 | 21 | Final decision | Three-option survey | Clean list or reactivate |

**Sequence-wide metrics to track:**
- Re-engagement rate: Target 15-25% of dormant contacts re-engaging (open + click)
- Conversion to conversation: Target 3-5% of re-engaged contacts booking a discovery call
- Unsubscribe rate: Expect 10-20% -- this is healthy list hygiene, not a failure signal
- List cleaning: Contacts with zero engagement across all 5 emails should be suppressed from active sequences permanently

**Maintenance requirements:**
- Email 2 (new developments): Refresh quarterly with latest platform data
- Email 3 (partner stories): Refresh quarterly with most recent partner testimonial
- Email 4 (market update): Refresh quarterly with current market data
- Create an internal "re-engagement content calendar" that feeds fresh data into these templates

**Exit routing:**
- Re-engaged contacts: Route back to Sales Sequence (if not previously completed) or to monthly Partner Pulse digest
- "Ready to talk" responses: Immediate BD notification, priority scheduling
- Unsubscribes: Immediate removal, no exceptions
- Non-responders after Email 5: Quarterly cadence for 12 months, then permanent suppression
