# SGA Dental Partners -- Conversion Rate Optimization Audit

**Audit Date:** March 7, 2026
**Auditor Role:** Senior CRO Analyst
**Scope:** All landing pages, quiz/scorecard funnel, webinar registration flow, lead magnet funnel, and chatbot flow
**Assets Reviewed:** 12 marketing documents across 5 funnel systems

---

## 1. Executive Summary

### Overall Funnel Health Score: 7.2 / 10

SGA's marketing funnel system is strategically ambitious and copywriting-mature. The messaging is rooted in proven direct response frameworks (PAS, AIDA, Gain/Logic/Fear), the segmentation logic is sophisticated, and the voice-of-customer integration is stronger than 90% of B2B funnel systems in healthcare. The copy itself is the best asset in this funnel -- it is specific, evidence-backed, and emotionally resonant without being manipulative.

The problems are structural, not narrative. This funnel system was built by a strategist, not a conversion engineer. The copy is excellent. The plumbing is where conversions will die.

### Top 3 Conversion Risks

**Risk 1: Form Field Overload on the Lead Magnet Opt-In Page (Estimated Revenue Impact: HIGH)**
The opt-in landing page for the Practice Partnership Assessment asks for 4 required fields: First Name, Email, Practice Name, and State. For a top-of-funnel lead magnet targeting midnight-Googling practice owners who are explicitly told "no sales call required," asking for Practice Name creates an immediate trust violation. The prospect's internal calculus shifts from "free assessment" to "they're going to call my office." Every field beyond email + first name reduces conversion 5-10% per field. Two unnecessary fields could be suppressing opt-in rates by 10-20%. At a $300 cost-per-assessment-completion target, that is $60-$120 per lead lost to form friction -- on a funnel that should be the cheapest entry point.

**Risk 2: Scorecard Length Creates Completion Drop-Off (Estimated Revenue Impact: HIGH)**
The Partnership Readiness Scorecard is 15 questions across 5 categories. Each question presents 5 answer options with paragraph-length descriptions. On mobile, a single question could fill 2-3 screens of scrolling before the prospect can select an answer. The lead capture gate sits after Q15, meaning the prospect must complete all 15 questions before providing their email. The funnel architecture targets a 35% completion rate for the main assessment. With 15 detailed questions and no progressive email capture, completion rates will likely fall in the 15-25% range. The scorecard is doing two jobs that conflict: providing genuine diagnostic depth (which requires length) and capturing leads (which requires brevity). This is the single highest-leverage optimization opportunity in the entire funnel system.

**Risk 3: No Single CTA Dominance on the Main Landing Page (Estimated Revenue Impact: MEDIUM-HIGH)**
The main landing page presents "Schedule a Confidential Conversation" as the primary CTA and "Download the Practice Partnership Assessment" as the transitional CTA. Both appear in the hero section, in the problem section, after the solution section, after each pillar, after social proof, after the how-it-works section, after the post-deal story, and in the final section. The page has 7+ CTA insertion points with dual CTAs at each. The Dr. Hendricks avatar is a 6-18 month decision-maker. He is not scheduling a confidential conversation on his first visit. The transitional CTA (assessment download) is more aligned with his buying stage, but it is visually subordinated to the primary CTA everywhere on the page. The page should invert CTA prominence: make the assessment the hero CTA and make the conversation a secondary option for the 5-10% who are further along.

---

## 2. Landing Page Friction Analysis

### 2.1 Main Landing Page (Practice Owner -- Dr. Hendricks)

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/landing_pages/main_landing_page.md`

**Page Goal:** Drive practice owners to schedule a confidential conversation (primary) or download the assessment (transitional)

**Element Scores:**

| Element | Score | Notes |
|---------|-------|-------|
| Value Proposition Clarity | 9/10 | Exceptionally clear. The subheadline delivers: who (100+ clinicians), backing (Thurston, 38yr, $4B+), what you keep (name, team, autonomy), and what you gain (relief, community, equity). One of the strongest B2B value propositions I have audited. |
| Headline Effectiveness | 8/10 | "You Did Not Build Your Practice to Watch Someone in a Boardroom Run It" -- strong pattern interrupt, speaks to the autonomy fear directly, uses "you" language. Minor concern: it leads with a negative frame (what you do NOT want) rather than a positive outcome. The A/B test variant ("7 Out of 10 Dentists Regret...") would likely outperform because it introduces a curiosity gap. |
| CTA Quality and Placement | 5/10 | This is the page's weakest element. "Schedule a Confidential Conversation" is a high-commitment CTA for a first-time visitor in a 6-18 month decision cycle. The transitional CTA ("Download the Practice Partnership Assessment") is better aligned with buying stage but is visually subordinate. CTA placement is generous (7+ locations) but the primary/transitional hierarchy is inverted relative to the audience's readiness. |
| Visual Hierarchy | 7/10 | Notes specify clean, premium layout with real photos. Social proof bar placement (below hero) is correct. FAQ accordion on mobile is right. However, the page is extremely long -- hero, problem, solution, 3 pillars, social proof, how-it-works, post-deal story, 16 FAQs, final CTA. On mobile, this is likely 15-20 screens of scrolling. Length is not inherently bad for B2B, but most conversion will happen in the first 3 sections or not at all. |
| Social Proof | 9/10 | Outstanding. Named partners with attributed quotes. Specific metrics (16.5% revenue increase, 100% staff retention, EBITA beating plan). The comparison table (SGA vs. Typical Corporate DSO) is evidence-dense. The "100+ Partner Practices / 10+ States / 85% Staff Retention / $4B+ Returned / 38-Year Track Record" proof strip is one of the strongest social proof bars I have seen in any B2B funnel. |
| Objection Handling | 10/10 | 16 FAQs covering every conceivable objection: autonomy, brand preservation, staff, patients, equity model, EBITDA multiples, PE fear, retirement timing, production quotas, timeline, post-close protections, rural markets, advisor involvement, market timing, competitor comparison. This is the most thorough objection handling I have audited in any funnel system. Each answer includes named proof points. |
| Trust Signals | 9/10 | Thurston Group track record repeated 4+ times. Named partner references. Staff retention rate published. Specific competitor comparisons. "Named references, not anonymous testimonials" language. The only gap: no third-party validation (no press logos, no industry awards, no BBB or similar trust badges). |
| Mobile Experience | 6/10 | The notes specify hero CTA visible without scrolling, FAQ accordion, real photos. However, the extreme page length (16 FAQs alone), lack of sticky mobile CTA specification, and dual-CTA structure create scrolling fatigue on mobile. No mention of tap-target sizing, font minimums, or click-to-call functionality. |

**OVERALL PAGE SCORE: 7.9 / 10**

**Specific Friction Points:**

1. **CTA hierarchy is inverted.** For a 6-18 month decision cycle, the assessment download should be primary and the conversation booking secondary. The current structure assumes conversion readiness that does not exist at first touch.

2. **Page length creates mobile abandonment risk.** 16 FAQs is excessive for a landing page. The page is doing the work of 3 pages: awareness (problem/solution), consideration (pillars/comparison), and decision (FAQ/pricing). Consider splitting the FAQ into a separate linked page or reducing to the top 5-6 questions.

3. **No sticky CTA on mobile.** For a page this long, a persistent floating CTA button on mobile is essential. Without it, the prospect reads for 5 minutes, decides they are interested, and has to scroll back up to find the CTA.

4. **No exit-intent implementation detail.** The notes mention an exit-intent popup for the assessment download, which is strategically correct. But there is no specification for mobile exit-intent (back-button trigger), which is where most mobile exits occur.

5. **Video embed placement is buried.** The 60-90 second partner testimonial video is specified for the social proof section, which is 4-5 sections deep. Video should be in or immediately below the hero section. Visitors who watch video convert at 2-3x the rate of non-watchers.

---

### 2.2 Associate Landing Page (Dr. Priya Nair)

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/landing_pages/associate_landing_page.md`

**Page Goal:** Drive associates to explore opportunities (primary) or download the career guide (transitional)

**Element Scores:**

| Element | Score | Notes |
|---------|-------|-------|
| Value Proposition Clarity | 9/10 | "Your Career Is Not a Production Number" paired with a subheadline listing mentors, Pikos Institute, and equity pathway. Exceptionally clear for the target audience. |
| Headline Effectiveness | 9/10 | "Your Career Is Not a Production Number" is a direct hit on the Priya Nair pain point. Short, memorable, emotionally resonant. The A/B test variant ("You Did Not Survive Dental School to Be a Tooth Mechanic") is equally strong but more aggressive. Both are excellent. |
| CTA Quality and Placement | 7/10 | "Explore Associate Opportunities" is action-oriented and specific. Better than "Apply Now" (too committal) or "Learn More" (too vague). The transitional CTA (career guide download) is well-positioned. CTA hierarchy is correct for this audience: associates are shorter-cycle decision-makers than practice owners. |
| Visual Hierarchy | 7/10 | The comparison table ("What You Get Now" vs. "What You Get at SGA") is the strongest visual element -- it creates an instant side-by-side that does the selling. The equity pathway timeline (Day 1, Year 1-2, Year 3-5, Year 5+) gives structure to what is usually vague. |
| Social Proof | 6/10 | Named quotes from Dr. Ellingson, Dr. Cavendish, Spencer Pierce, and Lamonte Jensen. However, there are no associate-level testimonials. All quotes are from leadership or senior partners. For a 28-year-old associate, peer proof is more persuasive than authority proof. The "Gen4 Associate (transformation story)" quote is attributed to a nameless associate -- this weakens credibility. |
| Objection Handling | 9/10 | 11 FAQs covering DSO skepticism, equity reality, case complexity, geographic concerns, mentorship specifics, benefits, compensation, partnership pathway, 90-day evaluation, student debt, and patient volume. Thorough and specific. |
| Trust Signals | 7/10 | Compensation ranges published ($160K-$240K + 28-32% production bonus). Signing bonus ranges ($15K-$30K). Benefits specified by provider name. The value calculation table ($37K-$70K above comparable DSO) is a strong anchoring device. Missing: Glassdoor/Indeed rating, named associates who can be contacted, and any third-party employment awards. |
| Mobile Experience | 6/10 | Notes specify filterable career listings module and thumb-accessible hero CTA, which is good. But no mention of sticky mobile CTA, click-to-apply, or ATS mobile optimization. Associates are mobile-first (Instagram, LinkedIn mobile). |

**OVERALL PAGE SCORE: 7.5 / 10**

**Specific Friction Points:**

1. **No peer-level social proof.** Every testimonial is from a C-suite executive or senior partner. For a 28-year-old associate carrying $340K in debt, the question is: "What does someone like ME say about working here?" The page needs 2-3 named associate testimonials with photos, years of experience, and specific outcomes.

2. **Compensation table may attract the wrong traffic.** Publishing $160K-$240K + 28-32% production bonus on the landing page will attract salary shoppers alongside genuine culture-fit candidates. The A/B test recommendation (compensation visible in hero vs. further down) should be prioritized. Leading with compensation attracts volume; leading with culture attracts quality.

3. **No direct link to specific openings from the hero.** "Explore Associate Opportunities" as a button should lead to a searchable/filterable job listing, but there is no specification for what happens after click. If it goes to a generic careers page, conversion will drop. If it goes to a filtered list with location/specialty matching, conversion will hold.

4. **The "Day in the Life" video is recommended but not specified as hero-adjacent.** For associate recruitment, video is the highest-converting content type. Place it immediately below the hero or integrate it as a background element.

---

### 2.3 Patient Landing Page (Linda Dawson -- Implant)

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/landing_pages/patient_landing_page.md`

**Page Goal:** Drive implant patients to book a free virtual consultation (primary) or take the Smile Assessment (transitional)

**Element Scores:**

| Element | Score | Notes |
|---------|-------|-------|
| Value Proposition Clarity | 8/10 | Headline leads with emotional resonance ("If You Have to Think About Your Teeth, Those Moments Are Not There"). Subheadline delivers: Dr. Olson, 3,390+ patients, same-day, free virtual consultation. Clear and compelling. Minor gap: it does not immediately convey what geographic area is served, which matters for a patient deciding if this is relevant to them. |
| Headline Effectiveness | 9/10 | This is Dr. Olson's own language, which makes it authentic. It passes the 3-second test: you immediately understand the emotional proposition. The A/B test variant ("New Teeth in One Visit. 3,390 Patients Trust Dr. O.") trades emotion for specificity -- both are strong. |
| CTA Quality and Placement | 8/10 | "Book Your Free Virtual Consultation" is clear, low-commitment (virtual, free), and action-oriented. The transitional CTA ("Take the Free Smile Assessment") is well-positioned as a lower-barrier entry point. The final CTA section adds a trust reassurance line ("Your consultation is free. Your information is private. No obligation. No sales pitch. Just answers.") that addresses the primary B2C objection directly. |
| Visual Hierarchy | 8/10 | Notes specify warm, spa-like aesthetic with before/after photos. Large readable text for 55-75 audience. Soft colors. This matches the Innovative Dental brand and the patient's emotional state. The pricing table is prominently placed, which is correct -- Linda is price-anxious and hiding pricing creates distrust. |
| Social Proof | 10/10 | Six named patient transformation stories (Scott, Juan, Louis, Maria, Janice, the nurse). Each story has a distinct emotional angle. The "3,390+ Five-Star Reviews" metric is repeated 5 times across the page. YouTube subscriber count (138K) and video view count (23M+) provide third-party validation. Dr. Olson's CEREC Mentor credential (1 of 300 worldwide) adds clinical authority. This is the strongest social proof deployment in the entire funnel system. |
| Objection Handling | 9/10 | 12 FAQs covering cost, same-day reality, surgery fear, bone loss candidacy, fixed income financing, travel logistics, All-on-4 vs snap-in, implant longevity, natural appearance, provider trust, Pikos Institute significance, and recovery expectations. The cost FAQ leads with published price ranges -- this is the right approach for a transparency-positioned brand. |
| Trust Signals | 9/10 | Published pricing ranges. Named patient stories. Specific credentials (CEREC Mentor, patent holder, Pikos Institute). YouTube as external validation. "Patients fly from Florida, Texas" as geographic social proof. The only gap: no guarantee language (lifetime warranty on implant hardware is mentioned in the FAQ but not in the trust signals section). |
| Mobile Experience | 7/10 | Notes specify large tappable CTAs, click-to-call phone number, 16px minimum font, and no horizontal scrolling. This audience accesses primarily through Facebook on mobile, which is correctly identified. Missing: specification for how the pricing table renders on mobile (tables are notoriously bad on small screens). |

**OVERALL PAGE SCORE: 8.5 / 10**

**Specific Friction Points:**

1. **No click-to-call in the hero section.** This audience (55-75, accessing via Facebook mobile) has a strong preference for phone calls over online forms. A prominent click-to-call phone number in the hero section could capture 10-20% of conversions that the form-based CTA misses. The current CTA ("Book Your Free Virtual Consultation") assumes comfort with online scheduling that this demographic may not have.

2. **Pricing table mobile rendering.** The 4-column pricing table (Procedure / Price Range / Monthly Financing) will likely break on mobile screens under 375px width. Specify a stacked card layout for mobile or a swipeable carousel.

3. **Before/after photos are referenced but not placement-specified.** The notes say "before/after patient photos prominently featured" but do not specify where in the page hierarchy. Before/after photos should be immediately below the hero or integrated into the social proof bar. They are the single most persuasive visual element for implant patients and should not be buried.

4. **No urgency mechanism.** The "cost of NOT treating" section in the pricing area provides logical urgency (bone loss, increasing complexity), but there is no time-based or availability-based urgency. For a patient who has been procrastinating for years, adding a consultation availability constraint ("Dr. Olson's next available virtual consultation slot is [date]") could accelerate booking.

---

### 2.4 Pricing / Partnership Tiers Page

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/landing_pages/pricing_page.md`

**Page Goal:** Educate qualified practice owners on deal structure, drive to conversation booking

**Element Scores:**

| Element | Score | Notes |
|---------|-------|-------|
| Value Proposition Clarity | 8/10 | "Every DSO Can Write a Check. The Question Is What Happens After." -- positions SGA as the post-deal value play, not the highest bidder. Subheadline quantifies value: $297K-$575K annual operational value. |
| Headline Effectiveness | 8/10 | Strong reframe from transaction to transformation. The A/B test variant ("7-9x EBITDA + Equity in a $350M Platform") trades narrative for specificity -- worth testing with bottom-funnel traffic. |
| CTA Quality and Placement | 7/10 | Same CTA as main page ("Schedule a Confidential Conversation"). By the time a prospect reaches the pricing page, they are further along in the funnel. Consider a more action-forward CTA: "Get Your Confidential Valuation Range" or "Request Your Partnership Briefing." |
| Social Proof | 7/10 | Risk reversal table mapping fears to protections is excellent. "The Second Bite of the Apple" section with Scenario A vs. B comparison is strong economic storytelling. But the page lacks fresh testimonials -- it reuses the same Godat/Cavendish quotes from the main page. A pricing page needs financial proof: a partner who can speak to actual equity appreciation. |
| Objection Handling | 9/10 | 13 FAQs covering multiples, equity splits, appreciation mechanics, second bite reality, management fees, retirement timing, negotiation, timeline, staff benefits, first-year dissatisfaction, MB2 comparison, EBITDA manipulation concerns, and commercial leases. This is a masterclass in financial objection handling. |
| Trust Signals | 8/10 | Thurston track record repeated 3+ times. Named partner references. EBITDA disclaimer included. Legal review flagged. Market context table with industry benchmarks establishes SGA as an informed participant, not a desperate buyer. |

**OVERALL PAGE SCORE: 7.8 / 10**

**Specific Friction Points:**

1. **Gate consideration is flagged but not resolved.** The notes suggest gating detailed tier information behind the Practice Partnership Assessment. This is the right instinct but the wrong implementation. Gating pricing creates friction. Instead, show the tiers openly (builds trust with the Sage archetype) but gate the personalized valuation range behind a conversation booking. The pricing page should be the best ungated resource in the market -- it positions SGA as transparent while competitors hide their terms.

2. **The value equation table ($297K-$575K) appears twice.** Once in Tier 1 and once in a separate section. This is redundant and dilutes impact. Present it once, prominently, and let it do its work.

3. **No interactive element.** This page is dense with numbers. An interactive "Partnership Value Calculator" where the prospect inputs practice revenue and sees an estimated value range, equity split, and operational savings would dramatically increase engagement and provide a natural lead capture point.

---

### 2.5 Competitor Comparison Page

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/landing_pages/competitor_comparison.md`

**Page Goal:** Position SGA as structurally superior through honest comparison

**Element Scores:**

| Element | Score | Notes |
|---------|-------|-------|
| Value Proposition Clarity | 8/10 | "Not All DSOs Are the Same. Most Dentists Find That Out Too Late." -- effective pattern interrupt that validates the prospect's research behavior. |
| Headline Effectiveness | 8/10 | Creates urgency through implied regret without being manipulative. The A/B variant ("SGA vs. Corporate DSOs: An Honest Comparison") trades drama for utility -- likely better for organic search traffic. |
| Trust Signals | 10/10 | The "Where Competitors May Be Better" section is the most trust-building element in the entire funnel system. Acknowledging that Heartland has more scale, Aspen has more brand recognition, and PDS offers 100% ownership is remarkably honest for a comparison page. This is Sage archetype execution at its best. It will disarm skepticism more effectively than any claim of superiority. |
| Social Proof | 8/10 | PE sponsor comparison table is devastating (Thurston's 38yr track record vs. KKR's lack of dental narrative vs. MB2's undisclosed backers). Education platform comparison (SGA with Pikos Institute vs. every competitor with "None") is a knockout. The Aspen 1.2-star Sitejabber reference is aggressive but effective if legally reviewed. |

**OVERALL PAGE SCORE: 8.2 / 10**

**Specific Friction Points:**

1. **Comparison table mobile rendering.** A 6-column comparison table (SGA + 5 competitors) will not render legibly on mobile. Specify a tabbed interface or swipeable card format for mobile. The notes mention this as an A/B test option -- it should be the default for mobile.

2. **No embedded competitor reviews.** The page references Heartland's Indeed reviews and Aspen's Sitejabber score but does not embed screenshots or direct quotes. Adding 2-3 actual competitor employee review excerpts (sourced from public platforms) would make the comparison viscerally real.

3. **Missing CTA after the "Where Competitors May Be Better" section.** After the honesty section, the prospect's trust is at peak. There is no CTA at this emotional high point. Insert a CTA immediately after: "Now that you have seen where others may be stronger, here is what no other DSO offers..." bridging to the differentiators section.

---

## 3. Quiz / Scorecard Funnel Audit

**Files:**
- `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/quiz_scorecard/partnership_readiness_scorecard.md`
- `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/quiz_scorecard/results_categories.md`
- `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/quiz_scorecard/followup_emails.md`

### Question Count Optimization

**Current:** 15 questions, 5 answers per question, estimated 4-5 minutes
**Recommended:** 8-10 questions with streamlined answer options

**Analysis:** The scorecard is strategically brilliant but operationally bloated. Each question presents 5 paragraph-length answer options. On mobile, Q2 alone (EBITDA margin confidence) presents 5 options ranging from 25-50 words each. A prospect must scroll through approximately 200 words of answer text per question before selecting. Across 15 questions, that is 3,000+ words of answer text alone -- the length of a long-form blog post.

The Daniel Priestley scorecard methodology works because it is quick and frictionless. The power is in the segmentation and personalized results, not in question depth. This scorecard has traded completion rate for diagnostic precision. The fix: reduce to 8-10 questions by combining categories and shortening answer options to single sentences.

**Specific reductions:**
- Category 1 (Practice Performance): Keep Q1 (revenue trajectory), cut Q2 (EBITDA margins -- too technical for top-of-funnel), keep Q3 (patient retention)
- Category 2 (Operational Independence): Keep Q4 (owner dependency -- the "two-week vacation" question is brilliant), cut Q5 (staff stability -- partially redundant with Q4), keep Q6 (systems documentation)
- Category 3 (Clinical Growth): Cut Q7 (CE investment -- weak discriminator), keep Q8 (case mix evolution), cut Q9 (technology adoption -- redundant with Q8)
- Category 4 (Market Positioning): Keep Q10 (competitive awareness), cut Q11 (referral network -- nice to have, not essential for segmentation), keep Q12 (growth ceiling)
- Category 5 (Personal Alignment): Keep Q13 (Monday morning energy), cut Q14 (financial timeline -- too sensitive for a quiz), keep Q15 (legacy vision)

**Optimized scorecard: 8 questions (Q1, Q3, Q4, Q6, Q8, Q10, Q12, Q13, Q15 from the current set minus Q13). Score range: 8-40. Tier thresholds adjusted proportionally.**

### Progress Indication

**Current:** Progress bar showing 95% complete appears only at the lead capture screen.
**Recommended:** Persistent progress bar from Q1 onward, showing category labels ("Practice Performance - Question 1 of 3").

The absence of a visible progress bar from the start is a significant drop-off risk. Prospects who see "Question 1" with no indication of total length assume the worst. A progress bar that shows "Question 1 of 15" with 5 category labels provides structure and sets expectations. Even better: "Category 1 of 5: Practice Performance -- Question 1 of 3" gives granular progress without overwhelming.

### Results Gating Analysis

**Current:** Email capture occurs after Q15 (100% completion), before results are revealed.
**Issue:** This is the highest-friction lead capture pattern for an interactive tool. The prospect has invested 4-5 minutes answering questions and now faces a gate between them and the payoff. Drop-off at this point is typically 20-40%.

**Recommended:** Progressive capture. After Q3 (completing Category 1), show a partial result: "Your Practice Performance Score: X/15." Then offer: "Enter your email to continue to Category 2 and see your full results." This captures the email at 20% completion rather than 100%, dramatically increasing lead capture while still providing value progression.

**Alternative:** If progressive capture feels too aggressive for the Sage archetype, capture email after Q8 (roughly 50% through the quiz) with: "You're halfway through. Enter your email so we can save your progress and deliver your full PDF report when you finish." This frames email capture as a service (save progress), not a gate (pay to see results).

### Follow-Up Sequence Timing

**Current:** 3 emails over 7 days per tier, then routing to main sequences
**Assessment:** The timing is well-calibrated.

- Day 0 (immediate): Results delivery -- correct, capitalizes on peak engagement
- Day 3: Weakest category deep dive -- correct spacing, provides value without pressure
- Day 7: Social proof story + CTA -- correct escalation to conversion ask

The dynamic content blocks based on weakest category are sophisticated and will require significant ESP configuration (20 content blocks total). Implementation risk: if the dynamic content fails to render, Email 2 becomes generic and loses its personalization power. Include a fallback content block for each tier.

**One gap:** No re-engagement for scorecard abandonment. If a prospect starts the scorecard but does not complete it, there is no retargeting or email recovery sequence. This requires capturing email early (progressive capture above) or using retargeting pixels to serve "You started your scorecard -- finish it here" ads.

---

## 4. Webinar Registration Audit

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/webinar/webinar_strategy.md`

### Registration Form Friction

**Current:** 2 fields (Email + First Name)
**Assessment:** This is correct and industry-best-practice for webinar registration. No friction issues.

The explicit note "No last name, no phone, no practice details at this stage" demonstrates conversion awareness. The 25-35% registration rate target (cold traffic) and 40-55% (warm traffic) are achievable with this field count.

### Urgency Elements

**Current:** Countdown timer to live event. Live-only bonus (Partnership Evaluation Scorecard). 72-hour replay window. 48-hour partner conversation booking window.

**Assessment:** The urgency mechanics are well-constructed and -- critically -- authentic. The document explicitly states: "No fake countdown timers. No 'only X spots left' when there is no limit. No secret extensions after publicly closing." This matches the Sage archetype and builds credibility for future events. The 48-hour partner conversation window is justified by a real constraint (partner dentists batch availability). This is how urgency should work.

**One gap:** The "live-only bonus" is given to no-shows in the No-Show Email 1 with the note "I am including it here for replay viewers too, because the framework is too useful to gatekeep." This undermines the live-only urgency for future events. If no-shows learn that they get the bonus anyway, show-up rates will decline over time. Either keep the bonus truly live-only, or do not position it as exclusive.

### Social Proof Placement

**Current:** Below the registration form: partner testimonial quote + dynamic registrant count.
**Assessment:** Correct placement. Social proof should appear immediately adjacent to the conversion action. The dynamic registrant count ("Already registered: [dynamic count] practice owners and specialists") creates social momentum. Ensure the count is seeded with a credible minimum (e.g., start displaying at 50+ registrants, not 3).

### Confirmation Flow

**Current:** Confirmation page with 4 elements (thank-you video, event ticket, immediate resource offer, share prompt) + confirmation email.
**Assessment:** This is textbook-correct confirmation flow design. The thank-you video from Dr. Ellingson (in a clinical setting, not a boardroom) matches the brand archetype. The immediate resource offer ("5 Deal Terms That Matter Most" cheat sheet) provides value between registration and event, reducing no-show rates. The share/refer prompt with unique referral links targets a 0.15-0.25 viral coefficient -- realistic for B2B.

**Friction point:** The confirmation email comes from "Dr. Mitch Ellingson" but the reminder emails alternate between Dr. Ellingson and Lamonte Jensen. For deliverability and recognition, the sender should be consistent. Use Dr. Ellingson for all pre-webinar emails (he is the clinical authority the audience identifies with) and Lamonte Jensen only for the post-webinar follow-up where CEO-level authority adds weight.

### Show-Up Rate Optimization

**Current target:** 45-55% (industry average 25-40%)
**Assessment:** The 45-55% target is ambitious but achievable with the specified tactics (SMS reminders, live-only bonus, short registration gap, calendar integration). The 3-5 day registration window for warm traffic is correct -- longer windows cause attendance decay.

**Missing element:** No mention of a "bring a colleague" incentive. Practice owners making partnership decisions often discuss with a spouse, business partner, or office manager. Explicitly inviting them to attend together increases show-up rates (social commitment) and decision velocity (both parties hear the same message simultaneously).

---

## 5. Lead Magnet Funnel Audit

### Opt-In Landing Page

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/lead_magnet/opt_in_landing_page.md`

**Form Fields:**

| Field | Required | Friction Level | Keep / Cut |
|-------|----------|---------------|------------|
| First Name | Yes | Low | KEEP -- enables email personalization |
| Email Address | Yes | Low | KEEP -- required for delivery |
| Practice Name | Yes | HIGH | CUT -- signals "we're going to contact your practice," which violates the "no sales call" promise |
| State | Yes | Medium | MOVE -- capture on the thank-you page or in the first email, not on the form |

**Recommendation:** Reduce to 2 fields (First Name + Email). Capture State via IP geolocation on the backend. Capture Practice Name in the first follow-up email or on the thank-you page ("Want us to customize your results? Tell us your practice name and state."). This reframes data collection as a value exchange rather than a toll gate.

**Opt-In Page Copy Assessment:**

The headline ("It's 11 PM and You're Googling 'Should I Sell My Dental Practice'") is exceptional. It meets the prospect exactly where they are -- literally -- and creates instant identification. This is Schwartz Level 4 messaging at its best: it does not tell them they have a problem, it mirrors the behavior they are already exhibiting.

The problem identification copy is strong but slightly long for a squeeze page. The page has 6 sections before the form. For a lead magnet opt-in page, the ideal structure is: headline, 3-4 bullet points of what they will learn, social proof line, form. The current structure (headline, problem section, "what you'll learn" section, social proof, form, trust signals, closing section) adds 3 sections that may not be needed. The prospect is already motivated (they clicked an ad or a content link). Excess copy before the form creates scrolling friction on mobile.

**Trust signals are well-placed below the form** -- Thurston Group backing, privacy assurance, and "you are in control" language. The "This assessment is a diagnostic tool, not a sales funnel" line is a direct counter to the prospect's skepticism and should stay.

### Thank-You Page

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/lead_magnet/thank_you_page.md`

**Conversion Opportunity Assessment:**

The thank-you page offers three paths forward:

1. Subscribe to newsletter (low commitment)
2. Request a peer conversation (medium commitment)
3. Schedule a confidential conversation (high commitment)

This is correct thank-you page architecture. The three paths map to three readiness levels, allowing the prospect to self-select their next step. The visual specification (distinct cards with icons) ensures the paths are scannable, not buried in text.

**One significant gap:** No email delivery troubleshooting for the primary deliverable. The page says "Check your inbox... if you do not see it within 5 minutes: check spam, add to contacts, search inbox." This is good but incomplete. The single biggest lead magnet conversion killer is non-delivery. Add: "Still nothing after 5 minutes? [Click here to download directly]" with a direct download link that does not require email. This backup captures the conversion even when email delivery fails.

**This gap is partially addressed:** The page includes "[Download Now] (direct download link as backup -- does not require email)." This is the correct implementation. Ensure this button is visually prominent, not subordinate to the email delivery instructions.

**Additional thank-you page opportunity:** The page does not mention the Partnership Readiness Scorecard. After downloading the PDF assessment, the most natural next step is the interactive scorecard: "Want a more detailed, personalized evaluation? Take the 5-Minute Partnership Readiness Scorecard." This creates a progressive engagement funnel: opt-in (low barrier) --> scorecard (medium engagement) --> conversation booking (high commitment).

---

## 6. Chatbot Flow Audit

**File:** `/Users/chasebowers/Documents/AI Workflows/sgadental/marketing/assets/chatbot/chatbot_flow.md`

### Conversation Length

**Current B2B flow:** 5 questions (role confirmation, practice size, biggest challenge, partnership awareness, timeline) before routing to a CTA.
**Assessment:** 5 questions is the upper limit for a chatbot qualification flow. Each additional question after 3 drops completion rates by approximately 15-20%. However, the practice owner flow correctly uses the questions to personalize the response (different scripts per challenge type), which justifies the length.

**Recommended optimization:** Make Q1 (role confirmation) implicit when the bot is triggered from a page-specific context. If the prospect is on /partners, skip "are you a practice owner?" -- they are. This saves one interaction and reduces the flow to 4 questions on the most important B2B pages.

### Qualification Efficiency

**Current:** The routing logic after Q5 (timeline) is well-structured:
- "Ready now" --> immediate BD handoff
- "6-12 months" + 4+ ops --> Partnership Guide + call booking
- "Just exploring" --> Assessment + email nurture

**Assessment:** This routing is correct and prevents two common chatbot failures: (1) sending early-stage prospects to sales too fast, and (2) letting hot prospects cool off in a nurture sequence. The "Ready to have a real conversation" option triggers immediate BD handoff -- this is the highest-value chatbot interaction and should be treated with priority.

**Gap:** No lead scoring is mentioned for "6-12 months" prospects who select "Admin burden" or "Succession planning" as their biggest challenge. These are higher-urgency pain points than "Growth ceiling" or "Clinical development." The chatbot should fast-track admin burden + succession planning prospects to BD regardless of their stated timeline, because their behavior (engaging with a partnership chatbot) suggests higher readiness than their stated timeline indicates.

### Handoff Points

**Current:** Handoff to humans on: explicit request, frustration detected, high-value lead ready now, complex clinical questions, pricing negotiation.
**Assessment:** These handoff triggers are correct. The "frustration detected" trigger should be defined with specific criteria: repeated question reformulations, negative sentiment keywords ("not helping," "talk to someone," "real person"), or timeout behavior (typing then deleting repeatedly).

**B2C Patient Flow Gap:** The patient chatbot flow collects symptom information before routing to scheduling. For implant patients (Linda Dawson avatar, 55-75 years old), chatbot interaction may feel impersonal. Add a proactive handoff option early in the patient flow: "Would you prefer to chat here, or would it be easier to call our patient coordinator directly? [Click to Call: (xxx) xxx-xxxx]." This demographic preference for phone communication should not be forced through a text-based qualification flow.

### Overall Chatbot Assessment

**Score: 7.5 / 10**

The chatbot is well-designed for qualification and routing. The persistent "skip to scheduling" and "talk to a person" shortcuts are excellent -- they prevent the chatbot from becoming a gatekeeper. The biggest risk is over-engineering: the document specifies entry points across 4 platforms, 6+ page-specific triggers, keyword classifiers, and AI intent routing. Implementation complexity creates fragility. Start with the website widget (B2B partnership pages only) and expand after baseline metrics are established.

---

## 7. Top 5 Conversion Leaks (Ranked by Estimated Revenue Impact)

```
PRIORITY RANKED CONVERSION LEAKS
==========================================

#1  SCORECARD COMPLETION DROP-OFF
    Stage: Quiz/Scorecard Funnel
    Issue: 15 detailed questions with email gate at 100% completion
    Impact: Estimated 40-55% of scorecard starters abandon before email capture
    Revenue Impact: CRITICAL -- the scorecard is the primary segmentation engine
    for the highest-value funnel (Practice Acquisition, $2M+ deal values)
    Fix: Reduce to 8-10 questions. Implement progressive email capture at 
    Q3 or Q5. Shorten answer options to single sentences. Add persistent 
    progress bar from Q1.
    Expected Lift: 30-50% increase in scorecard completion rate
    Effort: 3/10 (copy editing + form restructuring)

#2  OPT-IN FORM FIELD OVERLOAD (LEAD MAGNET)
    Stage: Lead Magnet Funnel (Top of Funnel)
    Issue: 4 required fields (Name, Email, Practice Name, State) on a 
    squeeze page promising "no sales call required"
    Impact: Estimated 10-20% suppression of opt-in rate vs. 2-field form
    Revenue Impact: HIGH -- this is the cheapest lead capture point in the 
    entire system, feeding the PA-Indoctrination sequence
    Fix: Cut to 2 fields (Name + Email). Capture State via IP geolocation.
    Capture Practice Name on thank-you page or in first email.
    Expected Lift: 15-25% increase in opt-in conversion rate
    Effort: 1/10 (form field removal)

#3  CTA HIERARCHY INVERSION ON MAIN LANDING PAGE
    Stage: Main Landing Page (Practice Acquisition)
    Issue: Primary CTA is "Schedule a Confidential Conversation" -- a 
    high-commitment action for a 6-18 month decision cycle at first touch.
    The lower-barrier assessment download is visually subordinated.
    Impact: Estimated 60-70% of first-time visitors are not ready for a 
    conversation but would download the assessment if it were the primary CTA
    Revenue Impact: HIGH -- this page receives the most expensive traffic 
    (Google Search at $25 CPC, LinkedIn at $150 CPL)
    Fix: Make "Download the Practice Partnership Assessment" the primary 
    hero CTA with prominent button. Relegate "Schedule a Confidential 
    Conversation" to a secondary CTA below it or in the how-it-works section.
    Add sticky mobile CTA for the assessment.
    Expected Lift: 20-35% increase in landing page conversion (assessment 
    downloads)
    Effort: 2/10 (CTA button swap + mobile sticky bar)

#4  MISSING CLICK-TO-CALL ON PATIENT LANDING PAGE
    Stage: Patient Landing Page (Implant Vertical)
    Issue: The Linda Dawson avatar (55-75, Facebook mobile) strongly prefers 
    phone contact. The page offers only online booking and assessment CTAs.
    No click-to-call phone number is visible in the hero or CTA sections.
    Impact: Estimated 15-25% of high-intent implant patients prefer to call 
    rather than fill out an online form. These are often the most qualified 
    patients -- they are ready to act NOW.
    Revenue Impact: HIGH -- implant cases are $22K-$55K per patient
    Fix: Add a click-to-call phone number as a co-equal CTA alongside 
    "Book Your Free Virtual Consultation." Format: "Prefer to call? 
    (XXX) XXX-XXXX -- speak to our patient coordinator now." Place in 
    hero section, pricing section, and final CTA section.
    Expected Lift: 10-20% increase in consultation bookings from mobile 
    traffic
    Effort: 1/10 (add phone number element)

#5  NO SCORECARD/ASSESSMENT ABANDONMENT RECOVERY
    Stage: Quiz/Scorecard + Lead Magnet Funnels
    Issue: Prospects who start the scorecard or visit the opt-in page but 
    do not complete have no recovery mechanism. No retargeting sequence, 
    no browser notification, no partial-save with email capture.
    Impact: Estimated 50-65% of scorecard starters and 70-80% of opt-in 
    page visitors leave without converting. These are warm prospects with 
    demonstrated intent who receive zero follow-up.
    Revenue Impact: MEDIUM-HIGH -- these are the closest-to-conversion 
    prospects in the awareness stage
    Fix: (1) Install retargeting pixels on the scorecard and opt-in pages.
    (2) Create dedicated retargeting ad creative: "You started your 
    Partnership Readiness Scorecard -- see your results." 
    (3) Implement progressive email capture at Q3 of the scorecard to 
    enable email recovery for partial completions.
    (4) Add exit-intent on the opt-in page with simplified 1-field form 
    (email only).
    Expected Lift: 10-15% recovery of abandoned scorecard starts; 
    5-10% recovery of abandoned opt-in page visits
    Effort: 4/10 (retargeting setup + creative + progressive capture)
```

---

## 8. Recommended A/B Tests (Priority Ordered)

### Test 1: Scorecard Question Count

**PRIORITY: 1 (Highest)**

**Hypothesis:** If we reduce the Partnership Readiness Scorecard from 15 questions to 8 questions (with shorter answer options), then scorecard completion rate will increase by 30-50% because cognitive load and time investment are the primary barriers to completion for time-pressed practice owners.

**Control:** 15 questions, 5 paragraph-length answers per question, email gate at Q15
**Variant:** 8 questions, 5 single-sentence answers per question, email gate at Q4

**Expected Impact:** 30-50% increase in completion rate (15-25% baseline to 25-40%)
**Minimum Sample:** 200 scorecard starts per variant (400 total)
**Success Metric:** Scorecard completion rate (email captured / scorecard started)
**Ease of Implementation:** 2/5

---

### Test 2: Opt-In Form Fields (2 vs 4)

**PRIORITY: 2**

**Hypothesis:** If we reduce the Practice Partnership Assessment opt-in form from 4 fields (Name, Email, Practice Name, State) to 2 fields (Name, Email), then opt-in conversion rate will increase by 15-25% because Practice Name creates a trust barrier for prospects who are told "no sales call required."

**Control:** 4 required fields (Name, Email, Practice Name, State)
**Variant:** 2 required fields (Name, Email)

**Expected Impact:** 15-25% increase in opt-in rate
**Minimum Sample:** 500 page visitors per variant (1,000 total)
**Success Metric:** Form submission rate (submissions / unique page visitors)
**Ease of Implementation:** 1/5

---

### Test 3: Main Landing Page CTA Hierarchy

**PRIORITY: 3**

**Hypothesis:** If we make "Download the Practice Partnership Assessment" the primary hero CTA (with "Schedule a Confidential Conversation" as secondary), then landing page conversion rate will increase by 20-35% because first-time visitors in a 6-18 month decision cycle are more likely to take a low-commitment diagnostic action than schedule a high-commitment conversation.

**Control:** Primary CTA = "Schedule a Confidential Conversation" with transitional "Download the Assessment" below
**Variant:** Primary CTA = "Take the Free Practice Assessment (5 min)" with secondary "Or, schedule a confidential conversation" below

**Expected Impact:** 20-35% increase in total page conversions (assessment downloads + conversation bookings combined)
**Minimum Sample:** 1,000 page visitors per variant (2,000 total)
**Success Metric:** Total conversion rate (all CTA clicks / unique visitors)
**Ease of Implementation:** 1/5

---

### Test 4: Patient Page Click-to-Call Addition

**PRIORITY: 4**

**Hypothesis:** If we add a click-to-call phone number as a co-equal CTA alongside the online booking button on the patient landing page, then total consultation bookings will increase by 10-20% because the 55-75 demographic accessing via Facebook mobile has a strong preference for phone contact over online forms.

**Control:** CTA = "Book Your Free Virtual Consultation" (online booking only)
**Variant:** Dual CTA = "Book Your Free Virtual Consultation" + "Or Call Now: (XXX) XXX-XXXX"

**Expected Impact:** 10-20% increase in total consultation bookings
**Minimum Sample:** 500 page visitors per variant (1,000 total)
**Success Metric:** Total consultation bookings (online + phone) / unique page visitors
**Ease of Implementation:** 1/5

---

### Test 5: Main Landing Page Hero Headline

**PRIORITY: 5**

**Hypothesis:** If we change the hero headline from "You Did Not Build Your Practice to Watch Someone in a Boardroom Run It" to "7 Out of 10 Dentists Regret Their DSO Sale. Here Is What the Other 3 Did Differently," then page engagement (scroll depth, time on page) and CTA clicks will increase by 10-15% because the curiosity gap in the variant headline creates stronger pull into the page body.

**Control:** "You Did Not Build Your Practice to Watch Someone in a Boardroom Run It"
**Variant:** "7 Out of 10 Dentists Regret Their DSO Sale. Here Is What the Other 3 Did Differently."

**Expected Impact:** 10-15% increase in CTA click rate
**Minimum Sample:** 1,000 page visitors per variant (2,000 total)
**Success Metric:** CTA click rate (any CTA click / unique visitors)
**Ease of Implementation:** 1/5

---

### Test 6: Webinar Live-Only Bonus Integrity

**PRIORITY: 6**

**Hypothesis:** If we keep the Partnership Evaluation Scorecard truly exclusive to live attendees (not shared with no-shows in the replay email), then live show-up rate will increase by 5-10% because the scarcity is real and perceived, creating genuine incentive to attend live.

**Control:** Scorecard shared with both live attendees and no-show replay viewers
**Variant:** Scorecard shared only with live attendees; no-show sequence offers a different (less valuable) resource

**Expected Impact:** 5-10% increase in live show-up rate
**Minimum Sample:** 3 webinar events with 100+ registrants each
**Success Metric:** Show-up rate (live attendees / total registrants)
**Ease of Implementation:** 2/5

---

### Test 7: Patient Page Headline (Emotional vs. Credential)

**PRIORITY: 7**

**Hypothesis:** If we change the patient landing page headline from the emotional "If You Have to Think About Your Teeth, Those Moments Are Not There" to the credential-led "New Teeth in One Visit. 3,390 Patients Trust Dr. O. Here Is Why," then consultation booking rate will increase by 5-15% because the credential variant delivers immediate proof and specificity that the emotional variant lacks.

**Control:** "If You Have to Think About Your Teeth, Those Moments Are Not There"
**Variant:** "New Teeth in One Visit. 3,390 Patients Trust Dr. O. Here Is Why."

**Expected Impact:** 5-15% increase in consultation booking rate
**Minimum Sample:** 500 page visitors per variant (1,000 total)
**Success Metric:** Consultation booking rate (bookings / unique visitors)
**Ease of Implementation:** 1/5

---

### Test 8: Associate Page Peer Testimonials

**PRIORITY: 8**

**Hypothesis:** If we add 2-3 named associate-level testimonials (with photos, years of experience, and specific outcomes) to the associate landing page, then application rate will increase by 10-20% because early-career dentists identify more with peer proof than with C-suite executive quotes.

**Control:** Current testimonials (Dr. Ellingson, Dr. Cavendish, Spencer Pierce, Lamonte Jensen -- all leadership)
**Variant:** Add 2-3 named associate testimonials above the leadership quotes

**Expected Impact:** 10-20% increase in application start rate
**Minimum Sample:** 500 page visitors per variant (1,000 total)
**Success Metric:** Application start rate (clicks on "Explore Associate Opportunities" / unique visitors)
**Ease of Implementation:** 3/5 (requires sourcing and permission from actual associates)

---

### Test 9: Pricing Page CTA Copy

**PRIORITY: 9**

**Hypothesis:** If we change the pricing page CTA from "Schedule a Confidential Conversation" to "Get Your Confidential Valuation Range," then CTA click rate will increase by 10-15% because prospects on the pricing page are seeking financial specificity and a valuation-oriented CTA matches their intent better than a generic conversation CTA.

**Control:** "Schedule a Confidential Conversation"
**Variant:** "Get Your Confidential Valuation Range"

**Expected Impact:** 10-15% increase in CTA click rate
**Minimum Sample:** 500 page visitors per variant (1,000 total)
**Success Metric:** CTA click rate
**Ease of Implementation:** 1/5

---

### Test 10: Scorecard Progressive Email Capture Timing

**PRIORITY: 10**

**Hypothesis:** If we capture email at Q3 (after the first category completes) instead of Q15 (after all questions), then email capture rate will increase by 40-60% while scorecard completion rate remains within 10% of baseline, because prospects are willing to provide email early when framed as "save your progress" rather than "pay to see results."

**Control:** Email capture at Q15 (current -- gate before results)
**Variant A:** Email capture at Q3 with framing: "Enter your email to save your progress and continue"
**Variant B:** Email capture at Q5 with framing: "You're one-third through. Enter your email to save your progress and get your results by PDF"

**Expected Impact:** 40-60% increase in email capture rate
**Minimum Sample:** 200 scorecard starts per variant (600 total for 3-way test)
**Success Metric:** Email capture rate (emails captured / scorecard starts)
**Secondary Metric:** Scorecard completion rate (must not drop more than 10%)
**Ease of Implementation:** 3/5

---

## Quick Wins vs Strategic Improvements

```
QUICK WINS (Implement This Week)
---------------------------------
1. Cut opt-in form to 2 fields (Name + Email)
   Expected impact: 15-25% lift in opt-in rate
   Time: 15 minutes

2. Add click-to-call phone number to patient landing page hero
   Expected impact: 10-20% lift in total consultation bookings
   Time: 30 minutes

3. Swap CTA hierarchy on main landing page (assessment primary, 
   conversation secondary)
   Expected impact: 20-35% lift in total page conversions
   Time: 30 minutes

4. Add sticky mobile CTA bar to main landing page and patient 
   landing page
   Expected impact: 5-10% lift in mobile conversion rate
   Time: 1 hour (development)

5. Move partner testimonial video from social proof section to 
   immediately below hero on main landing page
   Expected impact: 5-10% lift in page engagement
   Time: 15 minutes


STRATEGIC IMPROVEMENTS (Plan and Test Over 4-8 Weeks)
------------------------------------------------------
1. Reduce scorecard from 15 to 8 questions + implement progressive 
   email capture
   Requires: Copy editing, form restructuring, scoring recalibration,
   tier threshold adjustment, email sequence updates
   Timeline: 2-3 weeks

2. Build retargeting sequences for scorecard and opt-in page 
   abandonment
   Requires: Pixel installation, creative development, audience 
   segmentation, budget allocation ($500-$1,000/month for retargeting)
   Timeline: 2-4 weeks

3. Add named associate testimonials to associate landing page
   Requires: Identifying 2-3 willing associates, conducting interviews,
   getting photo and quote permissions, legal review
   Timeline: 3-6 weeks

4. Build interactive Partnership Value Calculator for pricing page
   Requires: Development of interactive widget, integration with CRM 
   for lead capture, testing across devices
   Timeline: 4-8 weeks

5. Split main landing page FAQ into separate linked page (reduce page 
   length by 40%)
   Requires: New page creation, internal linking strategy, SEO 
   considerations (FAQ schema on separate page), redirect testing
   Timeline: 1-2 weeks
```

---

## Appendix: Asset-Level Scoring Summary

| Asset | Score | Primary Issue | Priority Fix |
|-------|-------|--------------|-------------|
| Main Landing Page | 7.9/10 | CTA hierarchy inverted for buying stage | Swap primary/transitional CTAs |
| Associate Landing Page | 7.5/10 | No peer-level (associate) social proof | Source 2-3 named associate testimonials |
| Patient Landing Page | 8.5/10 | No click-to-call for phone-preferring demo | Add prominent click-to-call |
| Pricing Page | 7.8/10 | Generic CTA does not match pricing intent | Test "Get Your Valuation Range" CTA |
| Competitor Comparison | 8.2/10 | Table unusable on mobile | Implement tabbed mobile layout |
| Scorecard (15 questions) | 6.5/10 | Too long, email gate too late | Reduce to 8 Qs, progressive capture |
| Scorecard Results | 8.5/10 | Dependent on scorecard completion rate | N/A (quality is strong) |
| Scorecard Follow-Up Emails | 8.0/10 | Dynamic content implementation risk | Build fallback content blocks |
| Webinar Strategy | 8.5/10 | Live-only bonus undermined for no-shows | Keep bonus truly exclusive |
| Opt-In Landing Page | 6.8/10 | 4 form fields, page too long for squeeze | Cut to 2 fields, compress copy |
| Thank-You Page | 8.0/10 | No bridge to scorecard funnel | Add scorecard CTA |
| Chatbot Flow | 7.5/10 | Over-engineered for initial deployment | Launch website B2B only first |

---

## Methodology Notes

This audit was conducted by analyzing copy, structure, and conversion architecture across all provided marketing assets. No live traffic data, heatmaps, or session recordings were available. All conversion rate estimates are based on B2B SaaS and healthcare industry benchmarks, adjusted for the audience sophistication level (Schwartz Level 3-4) and deal value ($1M-$10M+).

Frameworks applied: AIDA, PAS, Gain/Logic/Fear (Ryan Deiss), Daniel Priestley Scorecard methodology, Hook Point 3-second rule (Brendan Kane), Schwartz awareness levels, and John Caples tested advertising principles.

All recommendations are diagnostic, not surgical. Implementation should be prioritized by the Impact vs. Effort matrix in Section 8.

---

*Audit prepared March 7, 2026*
*Total findings: 5 critical conversion leaks, 10 A/B test recommendations, 5 quick wins, 5 strategic improvements*
