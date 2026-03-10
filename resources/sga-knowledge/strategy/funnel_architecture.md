# SGA Dental Partners -- Funnel Architecture

**Prepared:** March 2026
**Funnel count:** 4 primary funnels + cross-funnel mechanics
**Audiences:** Practice owners, associate dentists, implant patients, specialist partners

---

## Strategic Overview

SGA operates four distinct funnels, each with different buying psychology, timelines, and conversion economics. A dentist selling a $2.4M practice is making a once-in-a-career decision over 6-18 months. A patient booking an implant consultation is making a high-emotion decision in 2-8 weeks. These cannot be served by the same system.

The architecture below maps each audience from first contact through conversion and into the retention/referral loop. Every stage answers: what does the prospect see, what do they do next, and what triggers the system to respond.

**Naming Convention:** Each funnel is labeled with a prefix (PA = Practice Acquisition, DR = Dentist Recruitment, PT = Patient, SP = Specialist Partnership) so email sequences, tags, and automations stay organized across the platform.

---

## Funnel 1: Practice Acquisition (Dr. Mark Hendricks)

**Business goal:** Generate qualified partnership conversations with practice owners doing $1M+ revenue
**Timeline to close:** 6-18 months from first touch to signed deal
**Funnel type:** Diagnose Then Prescribe (Assessment entry) + High-Trust Nurture
**Expected conversion:** 2-4% of marketing-qualified leads to signed partnership

### Stage Map

```
AWARENESS                    CONSIDERATION                 EVALUATION
Google Search Ads -------->  Practice Valuation     -----> Confidential
LinkedIn Thought Leader      Assessment (gated)            Partnership Guide
Dental Trade Articles        (lead capture)                (deep-dive PDF)
Podcast Appearances                |                             |
Conference Presence                v                             v
SEO Content Hub         Indoctrination Sequence        Engagement Sequence
                        (5 emails, Days 0-7)           (8 emails, Days 8-35)
                                                              |
                                                              v
                                                       CONVERSATION
                                                       Discovery call with
                                                       Biz Dev team
                                                              |
                                                              v
                                                         CLOSE
                                                       Site visit, partner
                                                       introductions, LOI,
                                                       due diligence, signing
```

### Awareness Stage

| Channel | Content Type | CTA | KPI |
|---------|-------------|-----|-----|
| Google Search (non-branded) | RSAs targeting "sell dental practice," "DSO partnership," "dental practice valuation" | "See What Your Practice Is Worth" | CPC < $25, CTR > 4% |
| LinkedIn Thought Leader Ads | Lamonte Jensen and Dr. Ellingson on the Gen4 model, autonomy, peer collaboration | "Read the Full Partnership Guide" | CPL < $150 |
| SEO Content Hub | Pillar pages: "How to Sell Your Dental Practice," "DSO vs Private Practice," "What Is a 4th Generation DSO" | Internal links to assessment | Organic traffic growth 15%+ MoM |
| Trade Publications | Byline articles in Dental Economics, Group Dentistry Now, Becker's | Brand authority, no hard CTA | Impressions, backlinks |
| Conferences | Booth at Dykema, Hinman, state dental meetings. Hosted dinners with 8-12 practice owners | Book a confidential conversation | Conversations started per event |
| Podcasts | Lamonte Jensen or Pete Wong on Shared Practices, Dental Hacks, Dentist Money Show | "Visit sgadental.com/partners" | Referral traffic spike post-episode |

### Lead Magnet: Practice Partnership Assessment

A 12-question self-assessment (not a financial valuation -- that requires real data) that helps practice owners score their readiness across 4 categories:

1. **Operational Readiness** (3 questions) -- How dependent is the practice on the owner for day-to-day operations?
2. **Financial Position** (3 questions) -- Revenue trends, debt load, concentration risk
3. **Growth Potential** (3 questions) -- Technology adoption, service mix, market position
4. **Personal Readiness** (3 questions) -- Timeline, retirement plans, emotional preparedness

**Scoring tiers:**
- **Strong Position (80-100):** "Your practice is well-positioned for a premium partnership. Let's explore what SGA can offer." CTA: Book a confidential discovery call.
- **Growth Opportunity (50-79):** "You have a solid foundation with room to grow. Here's how partnership could accelerate your trajectory." CTA: Download the Partnership Guide.
- **Foundation Building (0-49):** "You're earlier in the journey. Here's what to focus on over the next 12-24 months." CTA: Subscribe for practice growth content.

The assessment serves three purposes: it captures contact information, it segments leads by readiness, and it gives the business development team a conversation starter that is not a cold pitch.

### Email Sequences

#### PA-Indoctrination (5 emails, Days 0-7)

**Trigger:** Completes Practice Partnership Assessment
**Goal:** Establish SGA as the informed, trustworthy alternative to the DSO horror stories they have heard

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 0 | "Your assessment results + what they mean" | Deliver results, introduce SGA | Score summary, personalized tier message, brief SGA intro (who we are in 3 sentences), link to full results page |
| 2 | 1 | "The question every practice owner asks at 2 AM" | Empathy and identification | Acknowledge the late-night googling, mirror their exhaustion ("you didn't go to dental school to manage payroll"), introduce the Gen4 philosophy |
| 3 | 3 | "What 7 out of 10 dentists regret after selling" | Address the elephant in the room | Acknowledge DSO horror stories head-on, cite the "7 out of 10 frustrated or regretful" stat, position SGA's model as the structural counter |
| 4 | 5 | "How Thurston Group has done this 8 times" | Trust anchor: PE track record | 37 years, $4B+ returned, 8 dental/healthcare platforms, Smile Doctors as proof point, link to Thurston Group page |
| 5 | 7 | "Dr. Cavendish was bored after 20 years. Then this happened." | Transformation story bridge | Dr. Cavendish case study (revenues up 16.5%, EBITA beating plan, 100% staff retention), transition to engagement sequence |

#### PA-Engagement (8 emails, Days 8-35)

**Trigger:** Completed indoctrination OR opened 3+ indoctrination emails
**Goal:** Move from awareness to active evaluation using Gain/Logic/Fear + extensions

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 8 | GAIN: "What your practice looks like 2 years after partnering" | Paint the post-deal picture | Dr. Godat testimonial, specific outcomes (back to clinical work, staff retained, practice name preserved), "amplification not absorption" |
| 2 | 12 | LOGIC: "The math behind a JV equity partnership" | Rational justification | How the equity model works (conceptual, not deal-specific), compare to outright sale, management services, and staying solo. Anchor against Heartland (cash-out, no upside) and PDS (fees, no platform equity) |
| 3 | 16 | FEAR: "The cost of waiting another 3 years" | Cost of inaction | Technology gap widening, staffing crisis intensifying, practice values in flux, retirement timeline shrinking. Not manufactured urgency -- real market dynamics |
| 4 | 20 | "Your team stays. Your patients stay. Your name stays." | Objection: culture destruction | 85% staff retention rate, specific practices that kept their names (Periodontal Associates of Memphis, Dental Partners of SW Georgia), staff testimonials |
| 5 | 23 | "What Dr. Ellingson means by 'protect the profession'" | Objection: corporatization fear | Dr. Ellingson's full quote about dentistry vs. medicine, the autonomy paradox (joining gives you MORE voice), ADA president speaking at Gen4 event |
| 6 | 26 | "Three questions your financial advisor should ask about any DSO deal" | Objection: financial opacity | Education-forward email on deal evaluation criteria, positions SGA as transparent, invites them to bring their advisor to the conversation |
| 7 | 30 | "The Pikos Institute + 4Front: why our dentists keep getting better" | Differentiation: education moat | CE as the soul of the platform (not a line item), Pikos Institute for implantology, 4Front for peer collaboration, specific examples of clinical growth |
| 8 | 35 | "Ready for a confidential conversation?" | Conversion ask | Direct CTA to book a discovery call with business development. No pressure framing: "No commitments. No NDAs at this stage. Just a conversation between professionals." |

#### PA-Post-Conversation Nurture (for leads not ready to proceed)

**Trigger:** Completed discovery call but did not advance to site visit
**Cadence:** Biweekly for 6 months, then monthly
**Content:** Industry updates, new partner announcements, Pikos/4Front event invitations, quarterly market reports on practice valuations. Keeps SGA top of mind without sales pressure during the 6-18 month decision cycle.

### Success Metrics

| Stage | Metric | Target |
|-------|--------|--------|
| Awareness | Cost per assessment completion | < $300 |
| Consideration | Assessment completion rate | > 35% of landing page visitors |
| Indoctrination | Email open rate (sequence avg) | > 40% |
| Engagement | Click-through rate (sequence avg) | > 5% |
| Evaluation | Discovery calls booked per month | 8-15 |
| Conversation | Call-to-site-visit conversion | > 30% |
| Close | Site-visit-to-LOI conversion | > 40% |

---

## Funnel 2: Dentist Recruitment (Dr. Priya Nair)

**Business goal:** Attract qualified associate dentists and fill open positions across the SGA network
**Timeline to close:** 2-8 weeks from application to offer, ongoing passive nurture for future openings
**Funnel type:** Lead Magnet Funnel (career content entry) + Job Board Optimization
**Expected conversion:** 5-8% of applicants to hired

### Stage Map

```
AWARENESS                  INTEREST                    APPLICATION
LinkedIn Sponsored Jobs    Career Guide Download       Specific Job
Instagram Career Content   (lead capture)              Application
DentalPost/Indeed                |                     (ATS intake)
Dental School Outreach           v                          |
SEO Career Content         Nurture Sequence                 v
                          (4 emails, Days 0-10)        INTERVIEW
                                                       Phone screen,
                                                       clinical interview,
                                                       practice visit
                                                            |
                                                            v
                                                       ONBOARDING
                                                       90-day onboarding
                                                       sequence
```

### Awareness Stage

| Channel | Content Type | CTA | KPI |
|---------|-------------|-----|-----|
| LinkedIn Sponsored Jobs | Position-specific listings with mentorship, CE, and equity pathway highlighted | Apply now | Cost per qualified applicant < $80 |
| Instagram | Day-in-the-life content from SGA practices, CE event highlights, office tour reels | "See open positions" | Engagement rate > 3% |
| DentalPost / Indeed | Optimized listings with differentiated copy (not boilerplate "competitive compensation") | Apply | Applications per listing per month |
| Dental School Outreach | Residency program partnerships, GPR site placements at SGA practices | "Learn about SGA careers" | Applicants from partner programs |
| SEO Content | Articles: "Working at a DSO Pros and Cons," "DSO Dentist Salary Guide," "New Dentist Career Paths" | Download career guide | Organic traffic, guide downloads |

### Lead Magnet: "The Associate Dentist's Career Decision Guide"

A 15-page guide covering:
- Solo practice vs. DSO associate vs. partnership track: honest comparison with real numbers
- What to look for in a DSO (red flags and green flags)
- Compensation structures explained (base + production, equity pathways, benefits)
- Questions to ask in any dental job interview
- SGA's mentorship model and clinical development opportunities (final section, soft positioning)

### Email Sequences

#### DR-Nurture (4 emails, Days 0-10)

**Trigger:** Downloads career guide OR visits careers page without applying
**Goal:** Move from passive browsing to active consideration

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 0 | "Your career guide + a question most DSOs won't answer" | Deliver guide, set expectations | Guide link, introduce SGA's "career not a production number" positioning, tease mentorship model |
| 2 | 3 | "How one associate went from Class IIs to full-arch implants" | Clinical growth proof | Story of a Gen4 associate who expanded their clinical skills through the mentorship program, concrete before/after of case complexity |
| 3 | 6 | "The benefits nobody lists on the job posting" | Differentiate beyond salary | CE stipends, Pikos Institute access, 4Front symposium, 401(k) with match, equity participation from day one (Class A shares), peer community |
| 4 | 10 | "Open positions that match your goals" | Conversion to application | Curated list of current openings based on their geography/specialty interests (if captured), direct links to apply |

#### DR-Passive Candidate Nurture (monthly)

**Trigger:** Engaged with career content but did not apply
**Cadence:** Monthly
**Content:** New position announcements, CE event invitations, clinical case highlights from SGA practices, "Meet the Team" profiles. Maintains relationship with dentists who are not ready to move yet but may be in 6-12 months.

#### DR-Onboarding (90-day sequence, post-hire)

**Trigger:** Signed offer letter
**Goal:** Reduce first-90-day turnover, accelerate clinical integration

| Phase | Timing | Content | Owner |
|-------|--------|---------|-------|
| Pre-arrival | Days -14 to 0 | Welcome packet, team introductions, first-week schedule, benefits enrollment guide | HR + People Ops |
| Week 1 | Days 1-7 | Practice orientation, clinical systems training, mentor assignment, "Who to call for what" guide | Practice lead + mentor |
| Weeks 2-4 | Days 8-30 | Weekly check-in emails, first CE opportunity, peer introduction (regional cohort), 30-day feedback survey | People Ops |
| Weeks 5-12 | Days 31-90 | Monthly check-in, clinical goal setting with mentor, 60-day and 90-day surveys, invitation to next 4Front event | Mentor + People Ops |

### Success Metrics

| Stage | Metric | Target |
|-------|--------|--------|
| Awareness | Cost per qualified applicant | < $80 (LinkedIn), < $40 (DentalPost) |
| Interest | Career guide download rate | > 25% of careers page visitors |
| Application | Application completion rate | > 60% of started applications |
| Interview | Phone-screen-to-offer rate | > 25% |
| Onboarding | 90-day retention rate | > 90% |
| Retention | 12-month retention rate | > 85% |

---

## Funnel 3: Patient Acquisition (Linda Dawson)

**Business goal:** Drive new patient appointments across SGA practices, with special emphasis on high-value implant cases for the MODIS vertical
**Timeline to close:** 1-8 weeks from first touch to booked appointment (general dentistry: 1-2 weeks; implants: 4-8 weeks)
**Funnel type:** Split funnel -- Local Search Capture (general) + Content-Led Assessment (implants)
**Expected conversion:** 15-25% of leads to booked appointments (general), 8-15% (implants)

### Stage Map: General Dentistry

```
AWARENESS                 RESEARCH                     BOOKING
Google LSAs               Google Business Profile       Online booking
"Dentist near me"         Reviews, photos, Q&A          or phone call
Local Meta Ads            Practice website                    |
Community presence        service pages                       v
Referrals                                              TREATMENT
                                                       First appointment,
                                                       treatment plan
                                                            |
                                                            v
                                                       REFERRAL
                                                       Post-visit review
                                                       request + referral program
```

### Stage Map: Implant Patients (MODIS Vertical)

```
AWARENESS                 RESEARCH                     BOOKING
YouTube (Dr. Olson)       Smile Assessment        ---> Virtual Consultation
Facebook Ads              (online quiz)                 or In-Person Consult
Google Search             Financing Guide                    |
"dental implants cost"    (download)                         v
Patient testimonials           |                       TREATMENT
                               v                       Implant procedure,
                         Nurture Sequence               same-day or staged
                         (6 emails, Days 0-14)               |
                                                             v
                                                        REFERRAL
                                                        Transformation story
                                                        capture + review
                                                        + referral program
```

### Awareness Stage

| Channel | Content Type | Audience | CTA | KPI |
|---------|-------------|----------|-----|-----|
| Google LSAs | Verified practice profiles with high review counts | General patients | Call or book | CPL < $75 general, < $120 implants |
| Meta Local Ads | Before/after transformations (MODIS), community content (general practices) | Geo-targeted patients | "Book a Free Consultation" | CPL < $65 general, < $90 implants |
| YouTube | Dr. Olson transformation videos, procedure explainers, live Q&As | Implant-curious patients nationally | "Take Our Free Smile Assessment" | Views, assessment starts |
| Google Search | RSAs for "dental implants [city]," "full arch dental implants cost," "All-on-4 near me" | High-intent implant patients | Landing page with assessment CTA | CPC, conversion rate |
| Referral Program | In-office cards, post-visit text/email with referral link | Existing patients | "Refer a friend, both receive $50 credit" | Referrals per month per practice |

### Lead Magnet: Smile Assessment (Implant Vertical)

An 8-question online assessment for potential implant patients:

1. Current dental situation (dentures, partial, failing teeth, missing teeth)
2. How long they have been dealing with the issue
3. What bothers them most (eating, appearance, pain, social situations)
4. Previous dental work attempted
5. Awareness of implant options (none, some research, consulted a dentist)
6. Biggest concern (cost, pain, surgery, travel, time)
7. Financing preference (insurance, savings, financing, unsure)
8. Location / willingness to travel

**Output:** Personalized results page with:
- Their "Smile Readiness Score"
- Relevant patient transformation story matching their situation
- Estimated treatment timeline
- Financing overview (monthly payment range, not a fixed quote)
- CTA: Book a free virtual consultation

### Email Sequences

#### PT-Implant Nurture (6 emails, Days 0-14)

**Trigger:** Completes Smile Assessment but does not book consultation
**Goal:** Overcome fear, build trust, move to consultation booking

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 0 | "Your smile assessment results (and what they mean for you)" | Deliver results, introduce Dr. Olson | Personalized results summary, Dr. Olson intro ("I've helped thousands of patients just like you"), link to 2-3 relevant transformation videos |
| 2 | 2 | "The nurse who finally chose herself" | Emotional identification | Nurse transformation story, "putting everyone else first" narrative, before/after photos, Dr. Olson's quote about life's moments |
| 3 | 4 | "What dental implants actually cost (honest numbers)" | Objection: cost | Transparent price ranges by procedure type, financing breakdown ($199-$349/month examples), what insurance may cover, why the investment pays back in avoided future costs |
| 4 | 7 | "Scott's first meal with his new teeth was a New York strip" | Sensory transformation story | The steak moment. Vivid, specific, sensory. "Nicely seasoned, cooked perfectly, medium rare." Then the reframe: "A steak is not just a steak when you haven't been able to bite into one for 10 years." |
| 5 | 10 | "What to expect on your first visit (no surprises)" | Objection: fear of the unknown | Step-by-step walkthrough of the consultation process, what imaging looks like, how treatment planning works, "no commitment at your first visit" reassurance |
| 6 | 14 | "A personal invitation from Dr. Olson" | Final conversion push | Short, personal email (or video email) from Dr. Olson. "I've watched your assessment and I think we can help. Here's what I'd like to do: a free virtual consultation where we look at your situation together. No pressure. No sales pitch. Just an honest conversation about your options." Direct booking link. |

#### PT-Post-Treatment Referral Loop

**Trigger:** Treatment completed
**Cadence:** Day 1, Day 7, Day 30, Day 90 post-treatment

| # | Timing | Content | Goal |
|---|--------|---------|------|
| 1 | Day 1 | "How are you feeling? Here's your recovery guide" | Care delivery, open support line |
| 2 | Day 7 | "You're one week in -- here's what to expect next" + review request (Google) | Generate 5-star review while satisfaction is peak |
| 3 | Day 30 | "30 days of your new smile -- would you share your story?" | Capture transformation testimonial (written or video) |
| 4 | Day 90 | "Know someone who deserves to smile again?" + referral incentive | Activate referral. Offer: "Share this with someone. If they book, you both receive a $100 credit toward any service." |

### Success Metrics

| Stage | Metric | Target |
|-------|--------|--------|
| Awareness | Cost per lead (general) | < $75 |
| Awareness | Cost per lead (implant) | < $120 |
| Research | Smile Assessment completion rate | > 40% of starters |
| Nurture | Email open rate (implant sequence) | > 45% |
| Booking | Lead-to-consultation rate | > 15% (implants), > 25% (general) |
| Treatment | Consultation-to-case-acceptance | > 50% (implants), > 70% (general) |
| Referral | Reviews generated per 100 patients | > 20 |
| Referral | Referrals per 100 patients per quarter | > 8 |

---

## Funnel 4: Specialist Partnership (Dr. James Whitfield)

**Business goal:** Attract periodontists, oral surgeons, and implantologists to the MODIS specialty vertical
**Timeline to close:** 8-24 months (longer than general practice due to higher complexity and smaller prospect pool)
**Funnel type:** Relationship-Led Nurture (conference/peer-network entry) + Content Drip
**Expected conversion:** 3-5% of identified prospects to signed partnership

### Stage Map

```
AWARENESS                   CONSIDERATION                EVALUATION
Pikos Institute Network     Specialist Partnership  ---> MODIS Community
MODIS Community             White Paper (gated)          Conversations
  Conversations Podcast     "The Specialist's Guide      Podcast (deep dive)
LinkedIn (Dr. Pikos,        to DSO Partnership"               |
  Marcus Morosin)                |                            v
Conference Networking            v                       CONVERSATION
  (AAP, AO, Pikos           Indoctrination Sequence     Discovery call with
  Symposium)                (4 emails, Days 0-7)        Marcus Morosin (BD)
Peer Referrals                   |                            |
                                 v                            v
                           Engagement Sequence            CLOSE
                           (6 emails, Days 8-28)         Site visit, partner
                                                         introductions, LOI
```

### Awareness Stage

The specialist audience is small (estimated 12,000 periodontists, 7,500 oral surgeons, plus implantology-focused GPs in the US) and high-value. Mass marketing is wasteful. This funnel relies on precision targeting and peer-network credibility.

| Channel | Content Type | CTA | KPI |
|---------|-------------|-----|-----|
| Pikos Institute Network | Direct outreach to Pikos course alumni, co-branded content with Dr. Pikos | "Explore MODIS Partnership" | Conversations per quarter |
| MODIS Community Conversations Podcast | Episodes featuring current MODIS partners (Pensacola Perio, Troy Family Dental, Coastal Jaw Surgery) telling their story | "Download the Specialist's Guide" | Downloads per episode |
| LinkedIn | Marcus Morosin and Pete Wong content: specialty practice growth, implant market trends, PE in specialty dentistry | "See if MODIS fits your practice" | InMail response rate, connection requests |
| Conferences | Booth and hosted dinners at AAP Annual Meeting, AO Annual Meeting, Pikos Symposium | Book a confidential conversation | Conversations per event |
| Peer Referrals | Existing MODIS partners introduce prospects from their network | Warm introduction | Referral pipeline |

### Lead Magnet: "The Specialist's Guide to DSO Partnership"

A 20-page white paper addressing specialist-specific concerns:
- Why specialty practices are valued differently than general practices
- The capital investment gap: what it costs to build a competitive implant practice alone vs. with a platform
- How MODIS handles scope preservation (you practice full perio, not just implants)
- The Thurston Group track record across dental specialties (US Endodontics Partners, US Oral Surgery Management precedents)
- Case studies: Pensacola Perio, Troy Family Dental, Jacksonville Dental Specialists
- The Pikos Institute as a clinical development engine (not available at any other DSO)

### Email Sequences

#### SP-Indoctrination (4 emails, Days 0-7)

**Trigger:** Downloads Specialist's Guide or requests information at conference
**Goal:** Establish MODIS as the only specialty-focused DSO with clinical credibility

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 0 | "Your guide + why we built MODIS for specialists" | Deliver guide, introduce MODIS | Guide link, Pete Wong intro, the gap MODIS fills (specialists have no DSO built for them) |
| 2 | 2 | "What Thurston learned from US Endodontics Partners and US Oral Surgery Management" | Trust anchor: specialty PE experience | Thurston has built specialty dental platforms before, not just general dentistry. Specific precedents. |
| 3 | 4 | "Dr. Pikos on the future of implantology (and why he chose MODIS)" | Clinical credibility anchor | Dr. Pikos quote and endorsement, the Pikos Institute connection, what this means for partners |
| 4 | 7 | "How [Pensacola Perio / Coastal Jaw Surgery] grew after partnering" | Transformation proof | Specific partner story with outcomes: capital invested, cases expanded, recruiting solved |

#### SP-Engagement (6 emails, Days 8-28)

**Trigger:** Completed indoctrination OR opened 3+ indoctrination emails

| # | Day | Subject Line Direction | Goal | Key Elements |
|---|-----|----------------------|------|--------------|
| 1 | 8 | GAIN: "What your practice looks like with capital, marketing, and a lab behind it" | Aspirational future state | Paint the picture of a fully equipped specialist practice with MODIS resources |
| 2 | 12 | LOGIC: "The solo specialist's math problem" | Rational case for partnership | Cost of CT scanner, surgical suite, in-house lab, marketing, recruiting -- totaled vs. MODIS providing it |
| 3 | 16 | FEAR: "The recruiting crisis is hitting specialists hardest" | Cost of inaction | 18-month associate search stat, inability to compete with DSOs on benefits/equity, succession risk |
| 4 | 20 | "60% of your revenue is traditional perio. That's fine." | Objection: scope narrowing | Explicit reassurance that MODIS supports full-scope specialty practice, not just implants |
| 5 | 24 | "PE exit timelines: what Thurston's 37-year track record actually shows" | Objection: PE fear | Address the "load up debt, flip in 3 years" fear directly with historical evidence |
| 6 | 28 | "A conversation with Marcus Morosin (no NDAs, no pressure)" | Conversion ask | Warm, direct invitation to connect with VP of Business Development |

### Success Metrics

| Stage | Metric | Target |
|-------|--------|--------|
| Awareness | Identified specialist prospects per quarter | 50-100 |
| Consideration | Guide downloads per quarter | 25-40 |
| Indoctrination | Email open rate (sequence avg) | > 45% |
| Engagement | Click-through rate (sequence avg) | > 6% |
| Conversation | Discovery calls per quarter | 5-10 |
| Close | Call-to-LOI conversion | > 25% |

---

## Cross-Funnel Mechanics

### Referral/Viral Loops

Each audience has a distinct referral dynamic. The funnels are designed to activate these loops post-conversion.

| Audience | Referral Mechanism | Incentive | Expected Yield |
|----------|-------------------|-----------|----------------|
| Practice Owners (PA) | Partner introduces a peer practice owner in their network | Equity appreciation (aligned incentive -- more partners = larger platform = higher equity value) | 1-2 warm introductions per partner per year |
| Associate Dentists (DR) | Hired associate refers a classmate or former colleague | Referral bonus ($2,000-$5,000 for successful hire that stays 6+ months) | 15-20% of hires from internal referrals |
| Patients (PT) | Post-treatment referral program with dual incentive | $50-$100 credit for both referrer and referred | 8+ referrals per 100 patients per quarter |
| Specialists (SP) | MODIS partner introduces a specialist colleague through Pikos network | Equity appreciation + clinical collaboration opportunity | 1-2 warm introductions per partner per year |

### Cross-Funnel Interconnections

The four funnels are not isolated. Converted leads in one funnel generate pipeline for others.

```
RECRUITED DENTIST (DR) -----> refers a practice owner colleague -----> PA FUNNEL
                       -----> generates patients through clinical work -> PT FUNNEL

ACQUIRED PRACTICE (PA) -----> needs associate dentists filled ---------> DR FUNNEL
                       -----> brings existing patient base to network --> PT FUNNEL
                       -----> owner refers a specialist peer -----------> SP FUNNEL

IMPLANT PATIENT (PT)   -----> refers family/friends for general care --> PT FUNNEL
                       -----> transformation story becomes content -----> PA + SP AWARENESS

SPECIALIST PARTNER (SP) ----> refers general practice peers ------------> PA FUNNEL
                       -----> Pikos network generates clinical referrals -> PT FUNNEL
```

**Automation triggers for cross-funnel routing:**
- When a new partner signs (PA close), automatically trigger DR sequence for open positions at that practice
- When a recruited dentist starts (DR close), add their practice location to patient acquisition ad campaigns
- When a patient transformation story is captured (PT post-treatment), route to content team for use in PA and SP awareness campaigns
- When a specialist partner refers a colleague (SP referral), tag as "warm intro" and route to Marcus Morosin with context

### Retention Mechanics

| Audience | Retention System | Cadence | Content |
|----------|-----------------|---------|---------|
| Partner Dentists | "Partner Pulse" newsletter + quarterly partner calls + annual 4Front symposium | Monthly newsletter, quarterly calls, annual event | Network growth updates, clinical innovation highlights, peer success stories, market intelligence |
| Associate Dentists | 90-day onboarding sequence + mentor check-ins + CE calendar | Weekly (first 90 days), monthly (ongoing) | Clinical development milestones, CE opportunities, career pathing, peer cohort connections |
| Patients | Recall/recare system + seasonal campaigns + referral program | Biannual recall reminders, quarterly promotions | Appointment reminders, educational content (oral health tips), seasonal offers, referral prompts |
| Specialist Partners | MODIS Community Conversations podcast features + Pikos CE calendar + peer case collaboration | Monthly touchpoints | Advanced CE events, clinical case sharing, research updates, network expansion news |

### Re-Engagement Sequences

**Trigger:** No email opens for 60+ days (B2B audiences) or 90+ days (patients)
**Sequence:** 3 emails over 10 days

| # | Day | Subject Line Direction | Goal |
|---|-----|----------------------|------|
| 1 | 0 | "Still thinking about it? Here's what changed since we last talked" | Re-establish relevance with new information (new partner announcement, market data, event) |
| 2 | 5 | "Quick question (takes 10 seconds)" | Single-question survey to re-engage and re-segment ("What's your biggest priority right now?") |
| 3 | 10 | "Last note from us (unless you say otherwise)" | Give them an explicit opt-out or opt-back-in. Clean the list. Protect deliverability. |

**Outcome logic:**
- Opens/clicks email 1 or 2: Return to active nurture sequence
- Responds to survey (email 2): Re-segment based on response, route to appropriate sequence
- No engagement after email 3: Tag as "cold_unresponsive," suppress from active sequences, move to quarterly-only cadence

---

## Automation Map

### Tag Architecture

| Tag | Applied When | Used For |
|-----|-------------|----------|
| `pa_assessment_complete` | Completes Practice Partnership Assessment | Entry to PA-Indoctrination |
| `pa_tier_strong` / `pa_tier_growth` / `pa_tier_foundation` | Assessment scoring tier | Segment-specific messaging in engagement |
| `pa_indoctrinated` | Opens 3+ emails in PA-Indoctrination | Trigger PA-Engagement |
| `pa_discovery_call` | Books or completes discovery call | Move to post-conversation nurture, alert BD team |
| `pa_customer` | Signs LOI | Remove from all PA prospect sequences |
| `dr_guide_download` | Downloads career guide | Entry to DR-Nurture |
| `dr_applied` | Submits job application | Remove from DR-Nurture, route to ATS |
| `dr_hired` | Signs offer letter | Entry to DR-Onboarding |
| `pt_assessment_complete` | Completes Smile Assessment | Entry to PT-Implant Nurture |
| `pt_consultation_booked` | Books virtual or in-person consult | Remove from PT nurture, add to pre-appointment reminders |
| `pt_treatment_complete` | Completes treatment | Entry to PT-Post-Treatment loop |
| `sp_guide_download` | Downloads Specialist's Guide | Entry to SP-Indoctrination |
| `sp_indoctrinated` | Opens 3+ emails in SP-Indoctrination | Trigger SP-Engagement |
| `sp_customer` | Signs LOI | Remove from all SP prospect sequences |
| `cold_60d` | No email opens for 60 days (B2B) | Trigger re-engagement sequence |
| `cold_90d` | No email opens for 90 days (B2C) | Trigger re-engagement sequence |
| `unresponsive` | No engagement after re-engagement sequence | Suppress from active sequences |

### Conditional Logic

| Condition | Action |
|-----------|--------|
| Tag `pa_customer` or `sp_customer` applied | Immediately remove from ALL prospect sequences; add to Partner Pulse newsletter |
| Tag `dr_hired` applied | Remove from DR-Nurture; start DR-Onboarding; add practice location to PT ad targeting |
| Tag `pt_consultation_booked` applied | Remove from PT-Implant Nurture; start pre-appointment reminder sequence |
| Score > 80 on Practice Assessment | Fast-track: add note to BD team for priority outreach within 48 hours |
| Score < 40 on Practice Assessment | Route to educational content track (SEO articles, market reports), not sales sequences |
| Clicked offer/CTA 3+ times without converting | Tag as `hot_lead`; alert BD team for personal outreach (PA/SP) or front desk for phone call (PT) |

---

## Key Metrics Dashboard

### Funnel-Level KPIs

| Funnel | North Star Metric | Leading Indicators |
|--------|------------------|--------------------|
| PA (Practice Acquisition) | Signed partnerships per quarter | Assessment completions, discovery calls booked, site visits completed |
| DR (Dentist Recruitment) | Positions filled per quarter (with 90-day retention) | Guide downloads, applications submitted, interviews completed |
| PT (Patient Acquisition) | New patient appointments per practice per month | Smile Assessment completions, consultation bookings, ad CPL |
| SP (Specialist Partnership) | Specialist partnerships per year | Guide downloads, podcast listens, discovery calls booked |

### Email Health Metrics (monitored across all sequences)

| Metric | Healthy | Warning | Action Required |
|--------|---------|---------|-----------------|
| Open Rate | > 35% | 25-35% | < 25% -- review subject lines, sender name, send times |
| Click Rate | > 4% | 2-4% | < 2% -- review CTAs, content relevance, link placement |
| Unsubscribe Rate | < 0.3% | 0.3-0.5% | > 0.5% -- review frequency, relevance, list hygiene |
| Bounce Rate | < 1% | 1-2% | > 2% -- clean list, verify capture forms |
| Deliverability | > 95% | 90-95% | < 90% -- investigate domain reputation, authentication |

### Channel-Level Benchmarks

| Channel | Metric | Target |
|---------|--------|--------|
| Google Search (B2B) | Cost per qualified lead | < $350 |
| LinkedIn (B2B) | Cost per Lead Gen Form completion | < $150 |
| Google LSAs (B2C) | Cost per booked appointment | < $75 (general), < $130 (implant) |
| Meta Ads (B2C) | Cost per lead | < $65 (general), < $90 (implant) |
| Referral Program | Cost per referred patient | < $40 |
| Email Sequences | Revenue attributed per 1,000 subscribers per month | Track and baseline in first 90 days |

---

## Implementation Priority

### Phase 1: Weeks 1-4 (Foundation)

1. **PA Funnel:** Build Practice Partnership Assessment, landing page, and PA-Indoctrination sequence (5 emails). Launch Google Search and LinkedIn campaigns.
2. **PT Funnel (General):** Activate Google LSAs for top 10 practices. Implement Google Business Profile optimization protocol.
3. **DR Funnel:** Launch LinkedIn Sponsored Jobs for highest-priority open positions.

### Phase 2: Weeks 5-8 (Expansion)

4. **PA Funnel:** Build PA-Engagement sequence (8 emails). Launch conference presence strategy.
5. **PT Funnel (Implant):** Build Smile Assessment. Launch PT-Implant Nurture sequence (6 emails). Begin Meta ad campaigns using Dr. Olson content.
6. **DR Funnel:** Build and deploy Career Decision Guide. Launch DR-Nurture sequence (4 emails).
7. **SP Funnel:** Build Specialist's Guide white paper. Launch SP-Indoctrination sequence (4 emails).

### Phase 3: Weeks 9-12 (Optimization and Loops)

8. **All Funnels:** Deploy re-engagement sequences. Implement cross-funnel routing automation.
9. **PT Funnel:** Launch post-treatment referral loop. Implement review generation system.
10. **PA + SP Funnels:** Launch post-conversation nurture for leads not ready to proceed.
11. **All Funnels:** First performance review. Optimize sequences based on open/click/conversion data. Scale winning channels.

---

*Document total: ~3,400 words*
*Sources: Product brief, buyer personas, competitive landscape, voice of customer, channel analysis, SEO opportunity report, narrative research*
