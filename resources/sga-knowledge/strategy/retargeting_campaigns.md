# SGA Dental Partners -- Retargeting Architecture & Multi-Channel Campaign Operations

**Prepared:** March 2026
**Scope:** Retargeting architecture, multi-channel orchestration, SMS integration, campaign calendar
**Funnels covered:** PA (Practice Acquisition), DR (Dentist Recruitment), PT (Patient Acquisition), SP (Specialist Partnership)
**Decision cycles:** PA 6-18 months | SP 8-24 months | DR 2-8 weeks | PT 1-4 weeks

---

## Strategic Foundation

SGA runs four audiences with fundamentally different psychology and timelines. The retargeting and channel architecture must reflect this. A practice owner who visited the partnership page three weeks ago requires an entirely different follow-up motion than a patient who started a Smile Assessment yesterday. Applying the same retargeting logic across all four groups destroys trust with the groups that need patience and wastes budget on the groups that need urgency.

Three operating principles govern all decisions in this document:

1. **Temperature, not time.** Audience heat is defined by behavioral intent signals -- what they did -- not simply how many days ago they visited. A dentist who downloaded the Practice Partnership Assessment and opened three emails is hotter than someone who visited the homepage yesterday.
2. **Channel discipline.** Each channel has a job. LinkedIn builds professional authority for B2B. Meta drives emotional demand for B2C. Google captures existing intent. SMS delivers time-critical prompts to people who already opted in. Channels should not duplicate each other's message -- they should each advance the conversation one step further.
3. **Confidentiality as a design requirement.** Practice owners and specialist prospects are researching SGA in private. Retargeting that feels intrusive or overly persistent will cause them to disengage permanently. Frequency caps for B2B audiences are mandatory, not optional.

---

## Section 1: Retargeting Architecture

### 1.1 Pixel and Tag Infrastructure

#### Platform Setup

| Platform | Tag/Pixel | Events to Fire | Priority |
|----------|-----------|----------------|----------|
| **Meta Pixel** | Single pixel across all SGA web properties | PageView, ViewContent (per content type), Lead (form fill / assessment start), CompleteRegistration (assessment or guide completion), Schedule (consultation booked), Purchase (patient treatment confirmation) | Critical |
| **Google Tag Manager** | GTM container manages all tags | Google Ads Remarketing Tag, Google Analytics 4, LinkedIn Insight Tag, all custom events. GTM is the single point of truth -- no hard-coded pixels on pages. | Critical |
| **LinkedIn Insight Tag** | All pages (fires on every visit) | Page views segmented by URL pattern in CANS (Campaign Manager). Conversion events: Lead Gen Form completions, contact page visits, guide download confirmations. | Critical (B2B only) |
| **Google Ads Remarketing** | Via GTM | Google Tag for Remarketing fires on all pages. Custom audience lists built in Google Ads: site visitors by URL, video viewers (YouTube), customer list uploads. | Critical |

#### UTM Parameter Architecture

All paid traffic must use consistent UTM parameters to maintain clean attribution. The naming convention:

```
utm_source     = platform (google / linkedin / meta / email / sms)
utm_medium     = media type (cpc / display / social / email / text)
utm_campaign   = funnel prefix + descriptor (pa_partner-assessment / pt_smile-assessment / dr_career-guide / sp_specialist-guide)
utm_content    = creative variant (video-testimonial-godat / carousel-costcalc / image-staffretention)
utm_term       = keyword or audience segment (practice-transition / implant-cost / associate-dso)
```

Example: A LinkedIn Thought Leader Ad for Dr. Ellingson targeting practice owners:
`utm_source=linkedin&utm_medium=social&utm_campaign=pa_ellingson-thought-leader&utm_content=video-autonomy-q1&utm_term=practice-owners-dso`

#### Tag Firing Rules

| Page URL Pattern | Events to Fire | Audience List to Build |
|------------------|----------------|----------------------|
| `/partners` or `/partner-with-us` | ViewContent (content_type: partnership_page) | PA Warm Audience |
| `/assessment` (start) | Lead | PA Hot Assessment Starters |
| `/assessment/results` | CompleteRegistration | PA Hot Assessment Completers |
| `/pricing` or `/valuation` | ViewContent (content_type: pricing_page) | PA Hottest (pricing intent) |
| `/careers` | ViewContent (content_type: careers_page) | DR Warm |
| `/careers/apply` | Lead | DR Hot Applicants |
| `/smile-assessment` (start) | Lead | PT Hot Assessment Starters |
| `/smile-assessment/results` | CompleteRegistration | PT Hot Assessment Completers |
| `/consultation` or `/book` | Schedule | PT Hottest (booking intent) |
| `/modis` or `/specialist-partnership` | ViewContent (content_type: specialist_page) | SP Warm |
| `/specialist-guide` (download confirm) | CompleteRegistration | SP Hot Guide Downloaders |
| `/blog/*` or `/resources/*` | ViewContent (content_type: content) | Cool Audiences (all segments) |
| Thank-you pages (post-conversion) | Purchase (or custom conversion event) | EXCLUDE from retargeting |

---

### 1.2 Audience Segment Definitions

Audiences are organized by funnel first, temperature second. Each has distinct targeting criteria, frequency caps, and messaging mandates.

#### Funnel PA: Practice Acquisition

| Segment Name | Definition | Temperature | Est. Audience Size | Window |
|--------------|------------|-------------|-------------------|--------|
| PA-Hot-1: Assessment Completers | Completed Practice Partnership Assessment | Hottest | 200-500 | 14 days |
| PA-Hot-2: Pricing Intent | Visited `/valuation` or `/pricing` or `/partner` page 2+ times | Very Hot | 100-300 | 7 days |
| PA-Warm-1: Assessment Starters | Started assessment but did not complete | Warm | 500-1,500 | 21 days |
| PA-Warm-2: Partnership Page Visitors | Visited `/partners` or `/partner-with-us` | Warm | 1,000-3,000 | 30 days |
| PA-Warm-3: Email Engagers | Opened 3+ PA emails without booking call | Warm | Varies | 30 days |
| PA-Cool-1: Site Visitors (non-partnership) | Visited any SGA site page; did NOT visit partnership pages | Cool | 5,000-15,000 | 45 days |
| PA-Cool-2: Content Consumers | Read 2+ blog articles on practice transition, DSO topics | Cool | 1,000-5,000 | 60 days |
| PA-Cold: Lapsed Starters | Visited partnership pages 60-120 days ago; no engagement since | Cold | 2,000-8,000 | 90 days |

#### Funnel PT: Patient Acquisition (MODIS Implant Vertical)

| Segment Name | Definition | Temperature | Est. Audience Size | Window |
|--------------|------------|-------------|-------------------|--------|
| PT-Hot-1: Assessment Completers | Completed Smile Assessment, did not book | Hottest | 500-2,000 | 7 days |
| PT-Hot-2: Booking Page Visitors | Visited `/consultation` or `/book` without completing | Very Hot | 300-1,000 | 5 days |
| PT-Warm-1: Assessment Starters | Started Smile Assessment but abandoned | Warm | 1,000-4,000 | 14 days |
| PT-Warm-2: Service Page Visitors | Visited implant or procedure pages | Warm | 3,000-10,000 | 21 days |
| PT-Warm-3: Video Watchers (75%+) | Watched Dr. Olson transformation videos 75%+ | Warm | 5,000-20,000 | 21 days |
| PT-Cool-1: Site Visitors | Visited any patient-facing page | Cool | 20,000-80,000 | 30 days |
| PT-Cool-2: Video Watchers (25-74%) | Engaged with Dr. Olson content, lower completion | Cool | 10,000-40,000 | 45 days |
| PT-Cold: Past Leads (No Conversion) | Submitted lead form 60+ days ago; no appointment booked | Cold | 1,000-5,000 | 90 days |

#### Funnel DR: Dentist Recruitment

| Segment Name | Definition | Temperature | Est. Audience Size | Window |
|--------------|------------|-------------|-------------------|--------|
| DR-Hot: Guide Downloaders | Downloaded Career Decision Guide | Hot | 200-800 | 21 days |
| DR-Warm-1: Careers Page Visitors | Visited `/careers` or `/jobs` | Warm | 1,000-3,000 | 30 days |
| DR-Warm-2: Application Starters | Began application, did not complete | Warm | 100-400 | 14 days |
| DR-Cool: General Site Visitors (Dentist) | Visited site; dentist-interest signals via LinkedIn targeting | Cool | 5,000-20,000 | 45 days |

#### Funnel SP: Specialist Partnership

| Segment Name | Definition | Temperature | Est. Audience Size | Window |
|--------------|------------|-------------|-------------------|--------|
| SP-Hot: Guide Downloaders | Downloaded Specialist's Guide white paper | Hot | 50-200 | 30 days |
| SP-Warm-1: Specialist Pages | Visited `/modis` or `/specialist-partnership` | Warm | 200-700 | 45 days |
| SP-Warm-2: Podcast/Content | Engaged with MODIS Community Conversations content | Warm | 500-2,000 | 45 days |
| SP-Cold: Pikos-Adjacent | Interest-based targeting around Pikos Institute, AAP, AO, implant dentistry | Cold | 10,000-50,000 | 60 days |

---

### 1.3 Retargeting Sequences by Segment

#### PA Retargeting Sequences

**PA-Hot-1: Assessment Completers (Days 0-14)**

Message mandate: They self-identified. Do not re-educate. Close the next step: a confidential discovery call.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-2 | LinkedIn | Single Image | "Ready for the conversation?" + results reference | 3x/week max |
| 0-2 | Meta | Single Image | Dr. Godat or Dr. Cavendish partner testimonial (peer validation) | 3x/week max |
| 3-5 | LinkedIn | Video (30s) | Lamonte Jensen: "What the first call actually looks like. No pressure. No NDAs." | 3x/week max |
| 3-7 | Google Display | Responsive Display | "You took the assessment. Here's the next step." | 3/day max |
| 7-14 | LinkedIn | Document Ad | Confidential Partnership Guide (if not already downloaded) | 2x/week max |
| 7-14 | Meta | Carousel | 3-slide: Your Name Stays / Your Team Stays / Your Equity Grows | 2x/week max |

Creative rotation rule: Swap all creatives at Day 8 if click-through rate drops below 0.4% on LinkedIn or 0.8% on Meta.

**PA-Warm-1: Assessment Starters / Abandoners (Days 0-21)**

Message mandate: Remove the friction. Something stopped them. Address the most likely objection -- fear of judgment -- and make completion feel safe.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-3 | Meta | Single Image | "Most dentists come back to finish this. See why." | 4x/week max |
| 0-3 | Google Display | Responsive | "Where you left off -- your assessment is saved." | 3/day max |
| 4-10 | Meta | Video (15s) | Dr. Ellingson: "This is not a sales process. It is a conversation between peers." | 3x/week max |
| 4-10 | LinkedIn | Single Image | "What happens when you finish? See the 3 tiers." | 3x/week max |
| 11-21 | LinkedIn | Document Ad | "The questions every dentist asks before exploring partnership" (FAQ format) | 2x/week max |
| 11-21 | Meta | Single Image | 85% staff retention stat + "Here's what didn't change after the deal." | 2x/week max |

**PA-Warm-2: Partnership Page Visitors (Days 0-30)**

Message mandate: They are in research mode. Feed them proof. Make each ad teach one thing.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-5 | LinkedIn | Thought Leader | Lamonte Jensen article: "Why I won't call what we do a DSO" | 3x/week max |
| 0-5 | Google Display | Responsive | Thurston Group: 38 years, 8 platforms, $4B+ returned | 2/day max |
| 6-14 | Meta | Video (60s) | Dr. Godat transformation story (before/after partnership life) | 3x/week max |
| 6-14 | LinkedIn | Carousel | "Three things that stay yours: Name / Team / Patients" | 3x/week max |
| 15-25 | Meta | Single Image | "QuickBooks will never call you to the back office again." | 2x/week max |
| 15-30 | LinkedIn | Document Ad | Partnership Guide download | 2x/week max |
| 25-30 | Meta | Single Image | "7 out of 10 dentists regret their DSO sale. Here's what the other 3 did." | 2x/week max |

**PA-Cool: Content Consumers and Site Visitors (Days 0-60)**

Message mandate: Educational. No direct ask. Warm them toward the assessment.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-15 | Meta | Single Image | Content offer: "How to evaluate a DSO partnership (checklist)" | 2x/week max |
| 0-15 | Google Display | Responsive | Brand awareness: SGA tagline + partner faces | 1/day max |
| 16-30 | LinkedIn | Sponsored | Industry insight: "What dental practice values are doing in 2026" | 2x/week max |
| 31-45 | Meta | Video (30s) | Pillar story: "Dentistry is the last great profession. We're keeping it that way." | 1x/week max |
| 45-60 | LinkedIn | Single Image | "When you're ready to explore options, we'll be here." + assessment CTA | 1x/week max |

**PA-Cold: Lapsed Starters (60-120 days)**

Message mandate: Re-establish relevance with something new. A new partner story, a market data point, an event invitation.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| Trigger | Meta | Single Image | "New partner story: [Name] from [State] recently joined SGA." | 1x/week max |
| Trigger | LinkedIn | Single Image | "Practice valuations update: what's happening in Q1 2026." | 1x/week max |
| +14 days | Meta | Video (30s) | "Things are different now. Here's what changed." (new proof point or partner) | 1x/week max |

---

#### PT Retargeting Sequences (Implant Vertical)

**PT-Hot-1: Assessment Completers (Days 0-7)**

Message mandate: They raised their hand. Book the consultation. Every ad makes that the only ask.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-1 | Meta | Single Image | Patient who matches their profile (same situation, same outcome) | 5x/week max |
| 0-1 | Google Display | Responsive | "Your consultation is one click away." | 4/day max |
| 2-3 | Meta | Video (30s) | "This is exactly what your first visit looks like." (process demystification) | 4x/week max |
| 2-4 | Google Display | Responsive | "Financing from $199/month. Book your free consultation." | 3/day max |
| 4-5 | Meta | Carousel | Scott's steak moment / Nurse transformation / Janice at 87 | 3x/week max |
| 5-7 | Meta | Single Image | "Dr. O sees your assessment. He thinks he can help." (personal framing) | 3x/week max |

**PT-Hot-2: Booking Page Abandoners (Days 0-5)**

Message mandate: They were at the door. Remove the last objection (usually fear or logistics). Get them back.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-1 | Meta | Single Image | "Your spot is still available." + financing anchored to monthly payment | 5x/week max |
| 0-1 | Google Display | Responsive | "Free virtual consultation -- no commitment, no obligation." | 4/day max |
| 2-3 | Meta | Video (15s) | "What happens at your consultation (30 seconds)" | 4x/week max |
| 3-5 | Meta | Single Image | 3,390+ five-star reviews badge + one testimonial headline | 3x/week max |

**PT-Warm-1: Assessment Starters / Abandoners (Days 0-14)**

Message mandate: Something stopped them mid-quiz. Show them the value of finishing: a personalized result that matches their situation.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-3 | Meta | Single Image | "Your results are waiting. Takes 90 seconds to finish." | 4x/week max |
| 0-3 | Google Display | Responsive | "See your Smile Readiness Score -- you were close." | 3/day max |
| 4-10 | Meta | Video (30s) | Patient transformation story matching their likely demographics | 3x/week max |
| 10-14 | Meta | Single Image | "People who finished the assessment: here's what they found out." | 2x/week max |

**PT-Warm-2 and PT-Warm-3: Service Page Visitors and Video Watchers (Days 0-21)**

Message mandate: Awareness is there. Create desire. Show the emotional destination.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-7 | Meta | Video (60s) | Dr. Olson transformation story: "I want to give people back life's moments." | 3x/week max |
| 0-7 | Google Display | Responsive | "If you have to think about your teeth, those moments aren't there." | 2/day max |
| 8-14 | Meta | Carousel | Cost transparency carousel: "What implants actually cost -- honest numbers" | 3x/week max |
| 8-14 | YouTube | Pre-roll (15s) | Before/after visual + "Take our free Smile Assessment" | View-based |
| 14-21 | Meta | Single Image | "3,390 patients trusted Dr. O. Here's what they say." (reviews focus) | 2x/week max |

**PT-Cool: General Site Visitors (Days 0-30)**

Message mandate: Plant the seed. Emotional resonance, no hard ask yet.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-15 | Meta | Video (30s) | "She was afraid to smile at her grandkids' birthday party." | 2x/week max |
| 0-15 | YouTube | Pre-roll (15s) | Dr. Olson mission statement clip | View-based |
| 15-30 | Meta | Single Image | Free assessment offer: "Take 3 minutes. See what's possible for your smile." | 2x/week max |

---

#### DR Retargeting Sequences

**DR-Hot: Career Guide Downloaders (Days 0-21)**

Message mandate: They want information. Give them the inside view. Connect to an open position.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-5 | LinkedIn | Single Image | "Associate at SGA: what year 1 actually looks like." | 3x/week max |
| 0-5 | Instagram | Reel (15s) | Day-in-the-life at an SGA practice | 3x/week max |
| 6-14 | LinkedIn | Carousel | Pikos Institute + 4Front Symposium + Class A equity -- 3 slides | 3x/week max |
| 6-14 | Instagram | Single Image | "Class A shareholder from day one. Not a recruiting line." | 2x/week max |
| 14-21 | LinkedIn | Sponsored Job | Geo-targeted open position matching likely specialty/location | 3x/week max |

**DR-Warm: Careers Page Visitors (Days 0-30)**

Message mandate: They are browsing. Show them what others found when they looked closer.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-7 | LinkedIn | Single Image | Associate growth story (from Class IIs to full-arch implants) | 3x/week max |
| 0-7 | Instagram | Reel (30s) | CE event highlights: 4Front symposium footage | 3x/week max |
| 8-20 | LinkedIn | Document Ad | Career guide download (if not already downloaded) | 2x/week max |
| 20-30 | LinkedIn | Sponsored Job | Specific open positions in their region | 2x/week max |

---

#### SP Retargeting Sequences

**SP-Hot: Guide Downloaders (Days 0-30)**

Message mandate: Long decision cycle. Build credibility with peer evidence. Never hard sell.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-7 | LinkedIn | Thought Leader | Pete Wong or Marcus Morosin: "What we built MODIS for and who it's for." | 2x/week max |
| 7-14 | LinkedIn | Video (60s) | MODIS partner story: Pensacola Perio or Troy Family Dental podcast clip | 2x/week max |
| 14-21 | LinkedIn | Single Image | Dr. Pikos endorsement + Pikos Institute as clinical anchor | 2x/week max |
| 21-30 | LinkedIn | Document Ad | "The Specialist's Math Problem" (capital investment gap framework) | 1x/week max |

**SP-Warm: Specialist Page Visitors (Days 0-45)**

Message mandate: Awareness, authority, zero sales pressure.

| Day | Platform | Format | Message Theme | Frequency Cap |
|-----|----------|--------|---------------|---------------|
| 0-14 | LinkedIn | Thought Leader | Specialty dental market trends + MODIS positioning | 2x/week max |
| 14-28 | LinkedIn | Video (30s) | MODIS Community Conversations podcast clip (partner story) | 1x/week max |
| 28-45 | LinkedIn | Single Image | Thurston track record: US Endodontics Partners + US Oral Surgery Management precedents | 1x/week max |

---

### 1.4 Exclusion Lists

Exclusions are not an afterthought -- they are a trust protection mechanism. Advertising to existing partners, current employees, or recent converters is wasteful at best and damaging at worst.

| Exclusion Audience | Definition | Applies To |
|--------------------|------------|------------|
| **Current Partners (PA)** | Customer list of signed practice partners. Upload and refresh monthly. | All PA retargeting campaigns |
| **Current Partners (SP)** | Customer list of signed MODIS specialist partners. Upload and refresh monthly. | All SP retargeting campaigns |
| **Active Applicants** | Email list of anyone with `dr_applied` tag in CRM. Suppress from DR retargeting. | All DR retargeting campaigns |
| **Hired Associates** | Email list of anyone with `dr_hired` tag. | All DR retargeting campaigns |
| **Active Patients** | Email list of patients with `pt_treatment_complete` or `pt_consultation_booked` tags. | All PT retargeting campaigns |
| **Employees** | Employee email list. Suppress from all B2B retargeting. Keep separate from patient list. | All PA, SP, DR campaigns |
| **Bouncers / Hard Bounces** | Emails flagged as invalid in CRM. | All email-based custom audiences |
| **Opt-Outs** | Anyone who unsubscribed from email list or replied STOP to SMS. | All retargeting audiences (required by TCPA) |
| **Recent Converters (30 days)** | Anyone who converted in the last 30 days. Prevent double-messaging new partners/patients. | All active retargeting |

**Exclusion implementation:**
- Upload customer lists to Meta Custom Audiences, Google Customer Match, and LinkedIn Matched Audiences monthly. Use hashed email + phone.
- Apply tag-based exclusions in Google Ads via RLSA negative audiences.
- Maintain a master exclusion list in the CRM. Any tag in the list above triggers suppression automatically via CRM-to-ad platform integration.

---

### 1.5 Creative Fatigue Rotation Schedule

Creative fatigue is the silent campaign killer. On Meta, frequency above 4.0 (same person seeing an ad 4+ times in the reporting window) correlates with sharply declining CTR and rising CPL. On LinkedIn, frequency tolerance is lower -- 2-3 impressions per person per week is the ceiling before performance degrades.

**Rotation triggers (mandatory swap):**

| Signal | Action |
|--------|--------|
| Meta frequency > 4.0 in any 7-day window | Swap creative immediately |
| Meta CTR drops below 0.8% for patient audiences or 0.3% for B2B | Test new creative within 48 hours |
| LinkedIn CTR drops below 0.35% | Swap creative within 72 hours |
| Google Display CTR drops below 0.1% | Rotate to new image/headline combination |
| Cost per conversion increases >40% week-over-week | Creative and audience review within 24 hours |

**Rotation schedule by audience temperature:**

| Audience | Rotation Frequency | Minimum Creative Variants Ready |
|----------|-------------------|--------------------------------|
| Hot (0-7 days) | Every 7-10 days | 4 variants minimum |
| Warm (7-30 days) | Every 14-21 days | 3 variants minimum |
| Cool (30-60 days) | Every 21-30 days | 2 variants minimum |
| Cold (60-120 days) | Monthly | 2 variants minimum |

**Creative library cadence:**
- Quarterly creative shoot/production cycles for B2B (partner testimonial videos, thought leader clips)
- Monthly content pulls from Dr. Olson's YouTube channel for PT creative (new testimonials, new procedure explainers)
- Weekly Instagram Reel pulls from practice staff for DR recruitment creative
- Ad creative brief template: always include 1 emotional hook, 1 proof point, 1 CTA, 1 trust signal per execution

---

### 1.6 Lookalike and Similar Audience Strategy

Lookalike audiences extend proven segments to new prospects who share behavioral and demographic characteristics. Source audiences must have minimum 100 matched users (Meta requires 1,000+ for meaningful lookalikes; LinkedIn requires 300+).

| Source Audience | Lookalike Objective | Platform | Expansion Strategy |
|----------------|---------------------|----------|-------------------|
| **PA Assessment Completers** | Find dentists likely to engage with practice partnership content | Meta (1%, 2%, 3% LAL) | Start at 1%. Expand to 2-3% only after validating CPL at 1%. |
| **PA Discovery Call Bookers** | Find dentists most likely to book a BD conversation | LinkedIn (Lookalike from Matched Audience) | Use as top-of-funnel prospecting on LinkedIn. Layer with dentist/practice-owner targeting. |
| **PT Consultation Bookers** | Find patients likely to book implant consultations | Meta (1%, 2%) | Start 1% in core MODIS markets (Springfield MO, FL, MI, NC). |
| **PT Assessment Completers** | Broader patient prospecting | Meta (2%, 3%) | Use for demand generation in markets where PT-Warm audiences are too small. |
| **DR Guide Downloaders** | Find dentists likely to engage with career content | LinkedIn (Lookalike) + Meta (1%) | Layer LinkedIn lookalike with dentist targeting filters. |
| **SP Guide Downloaders** | Find specialist prospects | LinkedIn (Lookalike from Matched Audience) | Combine with periodontist/oral surgeon job title targeting. Very small audience; use carefully. |

**Exclusion rule for lookalikes:** Always exclude all exclusion audiences listed in Section 1.4 from every lookalike campaign. Lookalikes can inadvertently surface current partners or employees.

---

## Section 2: Multi-Channel Campaign Orchestration

### 2.1 Channel Coordination Matrix

The matrix below defines which channels fire, in what order, and with what message for each avatar moving through the funnel. Channels are sequenced -- not simultaneous -- to avoid message overlap and audience fatigue.

#### Dr. Mark Hendricks (Practice Owner) -- PA Funnel

```
PHASE 1: CAPTURE (Before first form fill)
   Channels Active: Google Search (paid) + LinkedIn (paid awareness)
   Google: RSAs targeting "sell dental practice," "DSO partnership," "practice valuation"
   LinkedIn: Thought Leader Ads (Jensen, Ellingson) to practice-owner targeting
   Gate: First form fill (assessment) or LinkedIn Lead Gen Form
   ---
PHASE 2: INDOCTRINATE (Days 0-7 post-capture)
   Channels Active: Email (primary) + LinkedIn retargeting (supporting) + Meta retargeting (supporting)
   Email: PA-Indoctrination sequence (5 emails). Primary workhorse.
   LinkedIn: PA-Warm retargeting. Reinforce email themes. Different creative, same message.
   Meta: Peer testimonial ads (Dr. Godat, Dr. Cavendish). Emotional validation.
   Google Display: OFF. Do not add noise during initial email sequence.
   Rule: LinkedIn and Meta should never use the same exact message as the email on the same day.
   ---
PHASE 3: ENGAGE (Days 8-35)
   Channels Active: Email (primary) + LinkedIn (primary supporting) + Meta + Google Display
   Email: PA-Engagement sequence (8 emails). Gain / Logic / Fear framework.
   LinkedIn: PA-Hot retargeting if assessment complete. PA-Warm if not. Rotate creatives weekly.
   Meta: Deeper proof content: staff retention, preserved practice names, equity model explainers.
   Google Display: Discovery call CTA for PA-Hot-2 (pricing intent visitors).
   Rule: When a hot lead opens email 6 (the financial advisor email), trigger BD team alert. LinkedIn frequency cap: 3x/week.
   ---
PHASE 4: NURTURE (Post-call, pre-LOI -- ongoing for 6-18 months)
   Channels Active: Email (biweekly) + LinkedIn (light touch 1-2x/month) + Conference presence
   Email: PA-Post-Conversation Nurture. Market updates, partner stories, event invites.
   LinkedIn: Light organic follow (Jensen/Ellingson post shares). Paid: 1x/week max.
   Meta: OFF. Do not run paid retargeting to post-call contacts. The relationship is personal now.
   Google Display: OFF. Same reason.
   Conference: Primary channel. Face-to-face is irreplaceable at this stage.
```

#### Dr. Priya Nair (Associate Dentist) -- DR Funnel

```
PHASE 1: CAPTURE
   Channels Active: LinkedIn Sponsored Jobs + Instagram Recruitment Ads + DentalPost
   LinkedIn: Position-specific sponsored listings highlighting mentorship, CE, equity.
   Instagram: Day-in-the-life Reels, office tour content, CE event highlights.
   DentalPost/Indeed: Optimized listings.
   Gate: Career guide download or job application start
   ---
PHASE 2: NURTURE (Days 0-10 post-capture)
   Channels Active: Email (primary) + LinkedIn retargeting + Instagram retargeting
   Email: DR-Nurture sequence (4 emails).
   LinkedIn: DR-Hot retargeting if guide downloaded. Position-specific ads.
   Instagram: Culture content, peer stories from SGA associates.
   Rule: LinkedIn and Instagram ads should feature real associates, not stock imagery.
   ---
PHASE 3: APPLICATION
   Channels Active: Email (CTA-direct) + LinkedIn (job-specific) + Retargeting (application completers excluded)
   Email: Email 4 of DR-Nurture: curated job list + direct application links.
   LinkedIn: Geo-targeted sponsored job post for their likely region.
   Instagram: OFF once they click to apply (suppress application starters from ads within 24 hours).
   ---
PHASE 4: PASSIVE NURTURE (Monthly, for those not ready to apply)
   Channels Active: Email (monthly) + LinkedIn (organic) + Instagram (organic)
   Email: Monthly new position announcements, CE invites, clinical case highlights.
   LinkedIn: Follow SGA's company page content.
   Instagram: SGA practice accounts' organic content keeps awareness alive.
```

#### Linda Dawson (Implant Patient) -- PT Funnel

```
PHASE 1: AWARENESS
   Channels Active: YouTube (Dr. Olson content) + Meta (Facebook primary, Instagram secondary) + Google Search
   YouTube: Organic content (138K subscribers). Paid: YouTube pre-roll for local market targeting.
   Meta/Facebook: Before/after transformation ads. Demographic: Women 45-70, household income $50K+.
   Google Search: RSAs for "dental implants cost," "All-on-4 near me," "full arch dental implants."
   Gate: Smile Assessment start
   ---
PHASE 2: ASSESSMENT TO BOOKING (Days 0-14)
   Channels Active: Email (primary) + Meta retargeting (heavy) + Google Display
   Email: PT-Implant Nurture (6 emails). Do not send email and SMS on same day.
   Meta: PT-Hot retargeting (assessment completers). Transformation stories.
   Google Display: Booking CTA for PT-Hot-2 (booking page abandoners).
   SMS: If opted in at assessment, use SMS for Day 4 and Day 10 nudges (not Day 0-3 -- let email lead).
   Rule: Frequency cap on Meta: max 5x/week for hot audiences. Drop to 3x after Day 7.
   ---
PHASE 3: PRE-APPOINTMENT
   Channels Active: Email (confirmation + reminders) + SMS (day-before + 1-hour reminders)
   Email: Confirmation email, pre-appointment FAQ, what to bring.
   SMS: Day-before reminder + 1-hour day-of reminder. Email is OFF the day SMS fires.
   Meta: OFF (no need to advertise to booked patients).
   ---
PHASE 4: POST-TREATMENT REFERRAL
   Channels Active: Email (referral sequence) + SMS (review request) + Meta (referral ads targeting their zip code)
   Email: PT-Post-Treatment Referral Loop (Day 1, 7, 30, 90).
   SMS: Day 7 review request (when satisfaction peaks).
   Meta: Geo-targeted "Share your story" ad to their neighborhood -- only fire at Day 30+.
```

#### Dr. James Whitfield (Specialist) -- SP Funnel

```
PHASE 1: AWARENESS
   Channels Active: LinkedIn (primary) + MODIS Community Conversations podcast + Conference
   LinkedIn: Marcus Morosin and Pete Wong content. Implant market trend content.
   Podcast: MODIS Community Conversations. No paid promotion initially -- organic audience reach.
   Conference: AAP Annual Meeting, AO Annual Meeting, Pikos Symposium. Primary channel.
   Gate: Specialist Guide download or conference conversation
   ---
PHASE 2: INDOCTRINATE (Days 0-7)
   Channels Active: Email (primary) + LinkedIn retargeting (supporting)
   Email: SP-Indoctrination (4 emails). Peer validation, Thurston track record, Pikos anchor.
   LinkedIn: SP-Hot retargeting. Pete Wong / Dr. Pikos thought leader content. Max 2x/week.
   Meta: OFF for SP segment. Specialists do not find their DSO partners on Facebook.
   ---
PHASE 3: ENGAGE (Days 8-28)
   Channels Active: Email (primary) + LinkedIn (supporting)
   Email: SP-Engagement (6 emails). GAIN/LOGIC/FEAR + objection handling.
   LinkedIn: SP-Warm retargeting. Rotate: Pensacola Perio story / MODIS capital investment / PE myth-busting.
   Rule: At Day 28 (email 6 -- Marcus Morosin invitation), trigger alert to Marcus in CRM. He should follow up personally via LinkedIn InMail within 48 hours.
   ---
PHASE 4: RELATIONSHIP (Post-call, pre-LOI -- 8-24 months)
   Channels Active: Email (biweekly) + LinkedIn (organic) + Podcast features + Pikos events
   Email: Biweekly nurture. Clinical content, research updates, Pikos CE calendar.
   LinkedIn: Organic engagement from Marcus Morosin and Pete Wong. No paid retargeting.
   Podcast: Invite warm SP prospects to be featured on MODIS Community Conversations episode.
   Events: Pikos Symposium invitation is the highest-value move at this stage.
```

---

### 2.2 Cross-Channel Messaging Consistency Rules

These are standing rules that apply across all campaigns, all channels, all audiences.

**Rule 1: One message per channel per day per person.**
If a prospect receives an email on Tuesday, no LinkedIn or Meta ad should echo the exact same message that day. Different angle, same pillar. Example: Email covers staff retention (Pillar 4). LinkedIn ad that day covers the education platform (Pillar 1). The themes are consistent. The message is not duplicated.

**Rule 2: Platform-native tone.**
- LinkedIn: Professional, peer-to-peer, evidence-backed. Lamonte Jensen's voice. No consumer language.
- Meta: Warmer. More emotional. Storytelling-forward. Acceptable for B2B only as awareness (not conversion).
- Google: Functional. Keyword-relevant. Benefits-led, not story-led. Clear CTA.
- Email: The deepest engagement. Long-form permitted. First-person voice from a named person (Jensen, Dr. Ellingson, Dr. Olson, Marcus Morosin) depending on funnel.
- SMS: The most human. Short. Direct. From a named person, not a brand. Never promotional language for B2B.

**Rule 3: Proof specificity is non-negotiable.**
Every claim in every channel must be backed by a name, a number, or both. "Clinical autonomy" requires: "Dr. Godat kept his name, his team, and his schedule." "Strong track record" requires: "38 years. 8 platforms. $4B+ returned." This applies to ads, emails, and SMS equally.

**Rule 4: Confidentiality language in B2B.**
Every practice acquisition and specialist partnership ad must include a confidentiality signal. Language options: "confidential discovery call," "no NDAs at this stage," "discreet exploration." The word "confidential" should appear in at least one B2B touchpoint per sequence phase.

**Rule 5: No channel isolation.**
Every channel should know what the other channels are doing. Email sequences must be loaded before retargeting ads launch. Retargeting should reference the same proof points as the emails. If a retargeting creative references Dr. Cavendish's story, that story should be in the email sequence and on the landing page the ad points to.

**Rule 6: Patient vs. practice owner language never crosses.**
B2C patient messaging (emotional, sensory, transformation-forward) should never appear in B2B channels. B2B acquisition messaging (equity, valuation, administrative relief) should never appear in patient-facing channels. Brand architecture gap: the MODIS website and Innovative Dental website are patient-facing. The SGA and Gen4 websites are partner-facing. Retargeting audiences built from patient site visits should not receive partner acquisition ads, and vice versa.

---

### 2.3 Attribution Windows by Channel

| Channel | Click Attribution | View-Through Attribution | Rationale |
|---------|------------------|--------------------------|-----------|
| Google Search (B2B) | 30 days | None | High-intent click. View-through not meaningful for search. |
| Google Search (B2C) | 14 days | None | Shorter purchase cycle for patients. |
| Google Display (B2B) | 30 days | 7 days | Display supports the sequence; view-through captures assisted influence. |
| Google Display (B2C) | 7 days | 1 day | Faster decision cycle. View-through is minimal signal for patients. |
| Meta (B2C -- Patient) | 7-day click | 1-day view | Standard Meta recommendation. Implant cycle is 1-4 weeks so 7-day click is appropriate. |
| Meta (B2B -- Awareness) | 7-day click | 1-day view | B2B Meta is awareness only. Lower attribution weight in reporting. |
| LinkedIn (B2B) | 30-day click | 7-day view | 30-day is appropriate for 6-18 month decision cycles. View-through acknowledges LinkedIn's influence at research stage. |
| Email | Opens + clicks attributed to email source | N/A | Email gets last-click credit when a prospect clicks from email to book. Multi-touch credit given to the preceding paid channel that drove the email open. |
| SMS | Click attributed directly | N/A | SMS is a direct-response channel. Track link clicks through UTM + SMS platform reporting. |

**Attribution model:** Use data-driven attribution in Google Ads where available. Use linear multi-touch attribution in CRM reporting to credit every channel that touched a converted lead. Do not use last-click exclusively for B2B -- the 6-18 month cycle means the last touch rarely represents where the journey started.

**CRM is the source of truth.** Ad platform attribution will be double-counted. Use CRM data (first-touch, multi-touch, and assisted conversion reports) for budget allocation decisions. Ad platform data for creative optimization decisions.

---

### 2.4 Budget Pacing Rules

#### Daily and Weekly Caps

| Campaign Type | Daily Budget Cap | Weekly Budget Cap | Monthly Budget Cap |
|---------------|-----------------|------------------|-------------------|
| PA Google Search | $200/day | $1,400/week | $5,000-$6,000 |
| PA LinkedIn | $150/day | $1,000/week | $4,000 |
| SP LinkedIn | $100/day | $700/week | $2,500 |
| DR LinkedIn | $100/day | $700/week | $2,500 |
| PT Google LSAs (per practice cluster) | $100-$150/day | $700-$1,050/week | $2,500-$4,000 |
| PT Meta (per practice cluster) | $75-$150/day | $525-$1,050/week | $2,000-$3,500 |
| Retargeting (all B2B) | $100/day total | $700/week | $2,500 |
| Retargeting (B2C patient) | $150/day total | $1,050/week | $3,500 |

Note: Caps above are per campaign cluster. Total monthly ad spend target: $25,000-$40,000 per channel analysis recommendation.

#### Scaling Triggers

Scale a campaign's budget by 25-30% when the following thresholds are sustained for 14+ consecutive days:

| Channel | Scale Trigger | Scale Action |
|---------|--------------|-------------|
| Google Search (PA) | CPL < $300, lead quality > 50% qualified per BD team | +25% daily budget |
| LinkedIn (PA) | CPL < $120 via Lead Gen Forms, OR 1 lead converts to partnership conversation | +25% daily budget |
| LinkedIn (DR) | Cost per qualified applicant < $80 | +25% daily budget |
| Google LSAs (PT) | Practice-level booking rate > 15% of leads; CPL < $100 | +20% per-practice budget |
| Meta (PT) | CPL < $70, show rate > 60% | +25% daily budget |
| Retargeting (all) | Hot audience CTR sustaining above baseline for 14 days | Extend window by 7 days; add creative variant |

#### Pause Triggers

Pause a campaign within 72 hours if:

| Channel | Pause Trigger |
|---------|--------------|
| Google Search (PA) | CPL exceeds $500 for 7+ consecutive days with no qualified leads |
| LinkedIn (PA/SP) | CPL exceeds $200 with zero qualified conversations after 45 days |
| Meta (B2B awareness) | CPL exceeds $150 with no measurable downstream impact on email opens or assessment completions |
| Meta (PT) | CPL exceeds $110; patient show rate drops below 40% |
| Any channel | Ad account flagged or suspended -- immediate halt, escalate to platform support |

#### ROAS Thresholds

B2C (Patient acquisition): Target blended ROAS of 8:1 on first-year patient value. Implant patient LTV = $15,000-$25,000 average case value. If CPL is $100 and show rate is 60%, cost per consultation is $167. Case acceptance at 50% means cost per case is $334. At $15,000 average case value, ROAS is 44:1. If ROAS drops below 10:1, creative and targeting audit required.

B2B (Practice acquisition): ROAS tracking on practice acquisition campaigns is indirect -- the conversion is a discovery call, not a purchase. Track cost-per-discovery-call as the primary B2B paid metric. Target: < $800 per qualified discovery call. (Each signed partnership represents $1M+ in deal value and long-term platform equity growth.)

---

### 2.5 Campaign Naming Conventions

Consistent naming is the infrastructure of clean reporting. Every campaign name must encode: platform + funnel + objective + audience + period.

**Campaign naming format:**
`[Platform] | [Funnel] | [Objective] | [Audience] | [Quarter-Year]`

**Examples:**

| Campaign | Name |
|----------|------|
| LinkedIn Thought Leader Ads for practice owners | `LI | PA | Awareness | Practice-Owners-B2B | Q2-2026` |
| Meta retargeting for assessment completers | `META | PA | Retarget | Assessment-Completers | Q2-2026` |
| Google Search for practice transition keywords | `GGL-S | PA | Search | Practice-Transition-KW | Q2-2026` |
| LinkedIn sponsored jobs for associates | `LI | DR | Jobs | Associate-Dentists | Q2-2026` |
| Meta patient acquisition for implant vertical | `META | PT | Acquisition | Implant-MODIS | Q2-2026` |
| Google LSAs for general dentistry practices | `GGL-LSA | PT | Appointments | General-Dentistry | Q2-2026` |
| LinkedIn retargeting for specialist guide downloaders | `LI | SP | Retarget | Guide-Downloaders | Q2-2026` |
| YouTube pre-roll for patient awareness | `YT | PT | Awareness | Video-Viewers | Q2-2026` |

**Ad Set / Ad Group naming format:**
`[Audience Segment] | [Creative Theme] | [Variant]`

Example: `Assessment-Completers | Testimonial-Godat | V1` or `Pricing-Intent | Discovery-Call-CTA | V2`

**UTM parameters must match campaign names.** If the campaign is `META | PA | Retarget | Assessment-Completers | Q2-2026`, the UTM campaign value should be `pa_retarget-assessment-completers-q2-2026`.

---

## Section 3: SMS Marketing Integration

### 3.1 Compliance Foundation

SMS marketing in the dental and healthcare space carries regulatory obligations that are non-negotiable. All SMS programs must be designed with compliance as the architecture, not as an add-on.

#### TCPA Compliance Checklist

Before any SMS message is sent:

- [ ] Explicit, written opt-in collected separately from email consent (SMS and email are legally distinct)
- [ ] Opt-in language includes: what types of messages they will receive, approximate frequency, that message and data rates may apply
- [ ] "Reply STOP to opt out" (or equivalent) included in every outbound message
- [ ] "Reply HELP for help" available and routed to a live response
- [ ] Quiet hours respected: no messages before 8:00 AM or after 9:00 PM recipient's local time
- [ ] Consent records stored: timestamp, IP address, opt-in language presented, source (which form/touchpoint)
- [ ] Do Not Call registry checked for all business landlines in DR/PA/SP outreach (NOTE: TCPA governs cell phones; DNC governs landlines)
- [ ] Privacy policy updated to include SMS data handling
- [ ] SMS program registered with The Campaign Registry (10DLC registration required for all US A2P SMS campaigns)
- [ ] Carrier spam filtering: avoid words like "FREE," "URGENT," excessive punctuation, and all-caps

#### 10DLC Registration Requirements

All SGA SMS campaigns must be sent through 10-digit long code (10DLC) numbers registered with The Campaign Registry. Requirements:
- Business entity registration (EIN, legal name, business type)
- Campaign use case registration (patient appointment reminders, B2B relationship, transactional, etc.)
- Sample messages provided to carrier for approval
- Opt-in / opt-out flows documented
- Expected message volume declared

10DLC registration takes 2-6 weeks. This must be completed before any mass SMS program launches.

---

### 3.2 Opt-In Collection Strategy

Opt-in must be explicit and contextual. The moment and the reason for collecting a phone number determine the type of SMS messages permissible.

| Collection Point | Opt-In Language | SMS Types Permitted | CRM Tag Applied |
|-----------------|-----------------|---------------------|-----------------|
| **Webinar / Event Registration (B2B)** | "Add my phone number to receive event reminders and follow-up resources via text. [Checkbox -- unchecked by default]" | Event reminders, post-event follow-up, exclusive content alerts. NO sales or deal pitches. | `sms_opted_in_webinar` |
| **Practice Assessment (PA)** | "Would you like text reminders when we publish new partner stories or host live events? [Checkbox -- unchecked by default]" | Content alerts, event invitations, one-touch "book a call" nudges. NOT deal terms or financial discussion. | `sms_opted_in_pa` |
| **Smile Assessment (PT)** | "Get your consultation reminder and follow-up information via text. [Checkbox -- checked by default, with opt-out easy to find]" | Consultation reminders, follow-up care instructions, review requests, referral offers. | `sms_opted_in_pt` |
| **Career Guide Download (DR)** | "Get notified when new positions open in your area. [Checkbox -- unchecked by default]" | New position alerts, application status updates, event invites. | `sms_opted_in_dr` |
| **Consultation Booking (PT)** | "Confirm this appointment via text and receive reminders. [Required for text confirmation]" | Appointment confirmations, reminders only. | `sms_opted_in_pt_appt` |
| **Post-Treatment Follow-Up (PT)** | Consent collected at checkout or in post-treatment intake | Recovery care, review request, referral offer | `sms_opted_in_pt_post` |

**Never collect phone numbers without checkbox.** Pre-checked boxes are illegal for SMS opt-in under TCPA. Every SMS opt-in must be an affirmative action by the user.

---

### 3.3 SMS Sequence Design

#### B2B SMS: Practice Acquisition (Dr. Mark Hendricks)

Operating principle: This is a $500K-$2M+ decision. SMS in the B2B practice acquisition context is a relationship tool, not a sales tool. Messages are brief, respectful of their time, and always offer something of value. Never reference deal terms, equity percentages, or financial specifics via SMS. Those conversations belong in email and in person.

Trigger: `sms_opted_in_pa` tag applied after assessment completion.

| # | Trigger / Timing | Message | Character Count | Notes |
|---|-----------------|---------|-----------------|-------|
| 1 | 24 hours after assessment completion | "Hi [First Name], it's [Biz Dev Rep Name] at SGA Dental. Your assessment results are in your inbox. Happy to walk through them together -- just reply with a good time to chat. Reply STOP to opt out." | 185 chars | Personalized from named BD rep, not brand. Feels like a real person. |
| 2 | 7 days after assessment (if no email open) | "[First Name], we're hosting a live partner roundtable on April 15th -- GPs and specialists sharing what Year 2 looks like inside SGA. Would this interest you? Reply YES for details or STOP to opt out." | 200 chars | Event-triggered. Provides value, not a sales push. |
| 3 | When a new partner from their state signs (triggered event) | "[First Name], a [State] periodontist just joined the SGA network. Their story might resonate -- can I send you a 3-minute read? Reply YES or STOP to opt out." | 170 chars | Hyper-relevant new information trigger. Consent to send further. |
| 4 | 2 weeks before a relevant conference (if SGA is attending) | "[First Name], SGA will be at [Conference] in [City] on [Date]. We're hosting a small private dinner -- 8 practice owners, no pitch. Would you like a seat? Reply YES or STOP to opt out." | 198 chars | Event invite. High signal of relationship intent. |
| 5 | 30 days of no email engagement (re-engagement) | "[First Name], I know timing for a conversation like this is everything. Just want to make sure you have what you need. Anything I can send that would help? Reply STOP to opt out anytime." | 190 chars | Re-engagement. Open-ended question. No pressure. |

**Frequency cap: Maximum 2 messages per month for PA segment.**
**Quiet hours: No messages before 8 AM or after 7 PM local time for B2B (more conservative than legal minimum).**

---

#### B2B SMS: Associate Recruitment (Dr. Priya Nair)

Operating principle: Career decisions are personal and sometimes made on short notice when the right opportunity appears. SMS is appropriate for time-sensitive position alerts and interview logistics.

Trigger: `sms_opted_in_dr` tag applied after career guide download.

| # | Trigger / Timing | Message | Notes |
|---|-----------------|---------|-------|
| 1 | 3 days after guide download | "Hi [First Name], SGA Dental here. We have an opening at [Location] that might match your background. Interested in a quick 15-minute call? Reply YES or STOP to opt out." | Position-specific. Only send if a matching role exists. |
| 2 | When a new position opens in their target geography | "[First Name], a new associate position opened at [Practice Name] in [City, State] -- [key differentiator, e.g., mentorship from Dr. Pikos-trained surgeon]. Reply YES if you want details. Reply STOP to opt out." | New position trigger. One message per opening max. |
| 3 | 24 hours before a scheduled phone screen | "Reminder: Your call with [Interviewer Name] from SGA is tomorrow at [Time] [Timezone]. Call-in: [number]. Questions? Reply here or email [address]. Reply STOP to opt out." | Operational. Reduces no-shows. |
| 4 | 4Front Symposium or Pikos Institute event announcement | "[First Name], SGA is hosting a free associate day at the [Event] in [City] on [Date]. Spots are limited. Reply YES for details. Reply STOP to opt out." | CE event invite. High-value offer creates genuine interest. |

**Frequency cap: Maximum 3 messages per month for DR segment.**

---

#### B2B SMS: Specialist Partnership (Dr. James Whitfield)

Operating principle: Specialists are high-value, small-pool prospects with long decision cycles. SMS must be used with the most restraint of any audience. Only 3 SMS touchpoint scenarios are appropriate.

Trigger: `sms_opted_in_webinar` or conference opt-in only. Do not collect SMS opt-in from cold SP prospects.

| # | Trigger / Timing | Message | Notes |
|---|-----------------|---------|-------|
| 1 | Pikos Institute symposium registration confirmed | "Hi [First Name], looking forward to seeing you at the Pikos Symposium. Marcus Morosin from MODIS will be there -- happy to make an introduction if useful. Reply STOP to opt out." | Warm event context. Personal connection offer. |
| 2 | MODIS Community Conversations episode featuring a peer | "[First Name], Pete Wong just released an episode featuring [Peer Name] from [Practice]. 28 minutes. Might be worth your time: [link] Reply STOP to opt out." | Value delivery. Peer-relevant. |
| 3 | 2 weeks before discovery call (if one is scheduled) | "Looking forward to our conversation on [Date/Time], [First Name]. If anything comes to mind before then, feel free to text back. Reply STOP to opt out." | Pre-call warmth. Confirms the appointment. |

**Frequency cap: Maximum 1-2 messages per month for SP segment. Less is more.**

---

#### B2C SMS: Implant Patient (Linda Dawson)

Operating principle: Patients are navigating fear, financial concern, and hope simultaneously. SMS for patients should feel like a caring, attentive practice -- not a marketing machine. Tone is warm, human, and personal.

Trigger: `sms_opted_in_pt` or `sms_opted_in_pt_appt` applied at assessment or booking.

| # | Trigger / Timing | Message | Notes |
|---|-----------------|---------|-------|
| 1 | 15 minutes after Smile Assessment completion (if no email open within 1 hour) | "Hi [First Name]! Your Smile Assessment results are ready in your inbox. Have a minute to review? Dr. O's team is here if you have questions. Reply STOP to opt out." | Soft nudge. Drives to email. Does not duplicate email content. |
| 2 | Day 4 of PT-Implant Nurture (email 3 was sent Day 4) | "Hi [First Name], we shared some honest numbers about implant costs in your email today. Financing starts at $199/mo for most patients. Book a free call: [link] Reply STOP to opt out." | Coordinated with email 3 (cost objection). SMS reinforces, does not duplicate. |
| 3 | Day 10 of PT-Implant Nurture (after email 5) | "One more thing from Dr. O's team -- we'd love to take 20 minutes and look at your situation together. Free virtual consult: [link] Reply STOP to opt out." | Pre-final-push SMS. Gentle. |
| 4 | 24 hours before consultation appointment | "Reminder: Your consultation with [Practice Name] is tomorrow at [Time]. Location: [Address] or virtual link: [link]. Questions? Reply here. Reply STOP to opt out." | Appointment reminder. Reduces no-shows. |
| 5 | 1 hour before consultation appointment | "You're scheduled in 1 hour! [Practice Name] at [Time]. [Parking / virtual link]. We're looking forward to meeting you. Reply STOP to opt out." | Day-of reminder. Warm and human. |
| 6 | Day 7 post-treatment | "Hi [First Name], it's been a week -- how are you feeling? We'd love to hear your story. If you have 30 seconds, a Google review means the world to us: [link] Thank you. Reply STOP to opt out." | Review request at peak satisfaction. |
| 7 | Day 30 post-treatment | "[First Name], one month in! We hope every meal feels like a celebration. Know someone who deserves to smile again? Share this with them: [referral link] -- you both get $100 credit. Reply STOP to opt out." | Referral activation. Sensory framing ("every meal feels like a celebration"). |
| 8 | Seasonal offer (January, September) | "[First Name], new year is the perfect time to help someone you love. If you know someone struggling with their smile, share Dr. O's free assessment: [link] Reply STOP to opt out." | Annual re-engagement. Not a discount offer. Altruistic framing. |

**Frequency cap: Maximum 4 messages per month during active nurture (pre-consultation). Maximum 2 per month post-treatment.**

---

### 3.4 SMS + Email Coordination Rules

The core rule: SMS and email should not fire simultaneously on the same day. They are meant to work in sequence, not in parallel. SMS acts as either a lead or a follow-up to email.

| Scenario | Email Role | SMS Role | Timing Rule |
|----------|-----------|---------|-------------|
| Assessment completion | Day 0 lead (results delivery) | Day 1 nudge (if email not opened) | SMS fires only if no email open within 24 hours |
| Webinar reminder | 24 hours before (full details) | 15 minutes before (join link only) | SMS supplements email; never replaces |
| Appointment reminder | 48 hours before (confirmation + instructions) | 24 hours before + 1 hour before | Email handles details; SMS handles timing |
| Flash content delivery | Email first (full content) | SMS second (teaser + link) | SMS fires 4-6 hours after email, only if email unopened |
| Post-treatment follow-up | Day 1 care guide (email) | Day 7 review request (SMS) | Different timing prevents overlap |
| Re-engagement | Email first (full re-engagement sequence) | SMS only after email sequence exhausted | SMS is the last-resort re-engagement channel, not the first |

**"Never both simultaneously" rule:** If an email was sent today, no SMS today (unless it is a time-critical appointment reminder). If an SMS was sent today, no email today (to avoid the same person feeling bombarded from two directions within hours).

**Opt-out cascade:** If someone opts out of SMS (replies STOP), immediately check their email status. If they are engaged email subscribers, do not suppress email. If they opt out of both, move to `cold_unresponsive` tag and quarterly-only cadence. Respect the signal -- they are telling you to reduce contact, not eliminate the relationship.

---

### 3.5 SMS Platform Requirements

| Requirement | Specification |
|-------------|--------------|
| Platform | Dedicated SMS platform with CRM integration (e.g., Attentive, Klaviyo SMS, Podium for B2C; Salesmsg or Twilio for B2B) |
| 10DLC Registration | Required for all business SMS. Register before launch. |
| CRM Integration | Two-way: SMS platform writes opt-in/opt-out status back to CRM in real time. CRM triggers control send timing. |
| Sender Name | B2B: Named individual (e.g., "Chase from SGA Dental") for personal feel. B2C: Practice name (e.g., "Innovative Dental") for brand recognition. |
| Link Tracking | All SMS links use UTM-tagged shortened URLs (branded domain preferred: e.g., sga.dental/link) to maintain deliverability. |
| Opt-Out Handling | STOP keyword immediately suppresses contact from all SMS sends. System logs opt-out with timestamp. CRM tag applied: `sms_opted_out`. |
| Compliance Logs | All sends, opt-ins, and opt-outs logged with timestamp, message content, and recipient identifier. Retain for 4 years (TCPA statute of limitations). |

---

## Section 4: Campaign Calendar Template

### 4.1 Monthly Campaign Rhythm by Vertical

The calendar below defines the standing monthly rhythm for each vertical. This is the minimum operating cadence -- event-triggered campaigns layer on top of this foundation.

#### Practice Acquisition (PA) -- Monthly Rhythm

| Week | Channel | Activity |
|------|---------|----------|
| Week 1 | Google Search | Check search term reports. Add new negative keywords. Adjust bids based on CPL from prior month. |
| Week 1 | LinkedIn | Launch new Thought Leader Ad for the month (Jensen or Ellingson). Topic rotates monthly per editorial calendar. |
| Week 1 | Email | Send PA-Post-Conversation Nurture biweekly email to post-call leads. Content: new partner announcement, market data, or event invitation. |
| Week 2 | Meta | Refresh creative for PA-Warm audience (new testimonial or pillar visual). |
| Week 2 | LinkedIn | Mid-month check: frequency review. If any audience above 3.0 weekly frequency, swap creative. |
| Week 3 | Email | Send PA-Post-Conversation Nurture email 2 of the month. |
| Week 3 | Google Display | Review PA-Hot retargeting performance. Pause underperforming placements. |
| Week 4 | LinkedIn | Review Lead Gen Form performance. Update lead routing to BD team if new completions. |
| Week 4 | All | Monthly performance review: CPL by channel, discovery calls booked, pipeline stage movement. Budget allocation decision for following month. |
| Month-end | CRM | Refresh exclusion lists in Meta, Google, and LinkedIn. Upload any new partners or converted leads to exclusion audiences. |

#### Patient Acquisition -- PT Implant (Monthly Rhythm)

| Week | Channel | Activity |
|------|---------|----------|
| Week 1 | Meta | Launch new creative for the month. Rotate: patient transformation story, cost transparency, Dr. Olson live content clip. |
| Week 1 | Google LSAs | Review and respond to all unread lead notifications from prior week. Dispute any invalid leads. |
| Week 2 | Email | Review PT-Implant Nurture sequence performance. A/B test subject line on email 3 (cost email) -- this one drives the most consult bookings. |
| Week 2 | YouTube | Post new Dr. Olson content to channel (organic). Boost top-performing organic video as paid pre-roll. |
| Week 3 | Meta | Frequency check on all PT audiences. Swap any creative with frequency > 4.0 weekly. |
| Week 3 | SMS | Review SMS delivery rates, opt-out rates, and link click rates from prior 30 days. Flag any spike in opt-outs. |
| Week 4 | Google LSAs | Bid adjustment review. Increase bids for services with best lead quality. |
| Week 4 | All | Monthly patient acquisition review: cost per consultation booked, consultation-to-case-acceptance rate, referrals generated per 100 patients. |
| Month-end | Reviews | Pull Google review count per practice. Identify practices below 100 reviews for review velocity campaign support. |

#### Dentist Recruitment (DR) -- Monthly Rhythm

| Week | Channel | Activity |
|------|---------|----------|
| Week 1 | LinkedIn | Refresh sponsored job posts for all active openings. Update any job descriptions with current team photos or recent CE events. |
| Week 1 | Email | Send DR-Passive Candidate Nurture email (monthly cadence). Content: new position openings, CE event highlight, clinical case story. |
| Week 2 | Instagram | Post 2-3 organic recruitment-focused Reels (office tours, CE events, associate testimonials). |
| Week 2 | DentalPost | Review listing performance. Update as needed. |
| Week 3 | LinkedIn | Check DR-Hot retargeting frequency. Swap creative if CTR declining. |
| Week 3 | ATS | Review applicant pipeline. Identify drop-off points in application flow. |
| Week 4 | LinkedIn | Review cost per qualified applicant for the month. Compare against $80 target. Adjust targeting or creative if above threshold. |
| Month-end | HR + Marketing | Sync: which positions closed? Which remain open? Update all active campaigns accordingly. |

#### Specialist Partnership (SP) -- Monthly Rhythm

| Week | Channel | Activity |
|------|---------|----------|
| Week 1 | LinkedIn | Marcus Morosin posts one organic piece of content (specialty market insight, MODIS partner story). |
| Week 2 | Email | Send SP-Post-Conversation biweekly nurture email. Content: Pikos CE calendar, clinical research update, new MODIS partner announcement. |
| Week 2 | Podcast | MODIS Community Conversations: confirm next episode guest, record if scheduled. |
| Week 3 | LinkedIn | SP-Warm retargeting creative review. |
| Week 3 | Conference | Confirm event calendar for next quarter. Reserve sponsorships and speaking slots. |
| Week 4 | BD Team | Marcus Morosin pipeline review: which SP prospects are in active conversation? Which need a personal LinkedIn message or call? |
| Month-end | All | Refresh SP exclusion audience with any newly signed partners. |

---

### 4.2 Seasonal Considerations for Dental

The dental industry has predictable seasonal patterns. Campaign strategy must account for them.

| Season / Period | Pattern | Campaign Implication |
|----------------|---------|---------------------|
| **January -- New Year** | Patient demand spike (New Year resolutions, resetting deductibles). Practice owner reflection (year-end review of business performance triggers partnership interest). | Launch PT "New Year, New Smile" patient campaigns January 2. Launch PA content with year-end financial reflection framing. |
| **February -- Pre-Spring** | Practice owners begin evaluating year-ahead strategies. Low competition in dental B2B paid search. | Strong month for PA Google Search investment. Valentine's Day: patient-facing "gift yourself a smile" angle for implant vertical (careful -- not a discount, a permission-giving moment). |
| **March-April -- Spring** | Associate dentists graduating from residency programs begin job searches. Patient demand steady. | DR LinkedIn spending peaks here. Target dental school graduates by graduation year. |
| **May-June -- Pre-Summer** | Conference season begins (Dykema DSO in May/June). Practice owners attend meetings. Insurance deductibles partially consumed -- some patient hesitation. | Conference campaign coordination. Pair conference presence with retargeting of attendee-segment LinkedIn audiences. B2C: emphasize financing (deductibles don't apply to implants). |
| **July-August -- Summer** | Practice owner activity slows (vacations). Patient demand dips. | Reduce PA/SP ad spend 15-20%. Use this period for creative production and audience building rather than heavy conversion spending. Launch DR campaigns targeting associates considering a move before fall. |
| **September -- Fall Ramp** | B2B decision-makers return from summer. Dental conference season picks back up. Patients return. Insurance "use it or lose it" begins (deductibles resetting). | Full PA/SP budget restoration. Launch PA "use your year-end review to explore options" messaging. Strong month for PT general dentistry ("use your dental benefits before year-end"). |
| **October -- Use It or Lose It Peak** | Strongest month for patient acquisition. "Use your insurance before December 31" is the most effective patient ad copy in dental. | PT campaigns at maximum budget. Launch "use your benefits" urgency messaging across Google LSAs, Meta, and email. Implant patients: financing pitch + timing urgency. |
| **November -- Pre-Holiday Slowdown** | Business decisions slow. Patients rush before holidays. | PA/SP: light touch. PT: sustain through mid-November. Pull back after Thanksgiving. |
| **December -- Quiet Period + Planning** | Lowest business activity. Year-end reflection for practice owners. | PA: Educational long-form content ("What practice owners are evaluating for 2027"). No direct asks. PT: Holiday messaging in early December. Budget conservation month. Plan Q1 campaigns. |

---

### 4.3 Event-Triggered Campaigns

Event-triggered campaigns activate automatically based on specific user behaviors. These are the highest-ROI activities in the system because they are perfectly timed to the prospect's intent signal.

#### Trigger 1: Assessment Completion (PA or PT)

| Trigger | Automatic Actions |
|---------|------------------|
| Practice Partnership Assessment completed | 1. Send PA-Indoctrination email sequence (starts immediately). 2. Add to PA-Hot-1 Meta retargeting audience (fires within 24 hours). 3. Add to PA-Hot-1 LinkedIn retargeting audience. 4. Alert BD team in CRM if score > 80 (priority outreach within 48 hours). 5. If `sms_opted_in_pa`: send SMS 1 at 24-hour mark. |
| Smile Assessment completed (no booking) | 1. Send PT-Implant Nurture email sequence (email 1 immediately). 2. Add to PT-Hot-1 Meta retargeting audience. 3. Add to PT-Hot-1 Google Display retargeting audience. 4. If `sms_opted_in_pt`: flag for SMS 1 at 24-hour mark (only if email 1 unopened). |

#### Trigger 2: Quiz or Guide Abandonment

| Trigger | Automatic Actions |
|---------|------------------|
| Assessment started, not completed (PA) | 1. Add to PA-Warm-1 Meta retargeting audience. 2. Google Display: "Where you left off -- assessment is saved" retargeting. 3. No email (they did not give permission). 4. Google Display only. |
| Smile Assessment started, not completed (PT) | 1. Add to PT-Warm-1 Meta retargeting audience. 2. Google Display retargeting. 3. No email (no permission yet). |
| Career guide download page visited (not completed) | 1. Add to DR-Warm-1 LinkedIn retargeting. 2. Instagram retargeting. |

#### Trigger 3: Webinar or Event Registration

| Trigger | Automatic Actions |
|---------|------------------|
| B2B webinar registered (PA/SP) | 1. Confirmation email (immediate). 2. Reminder email 24 hours before. 3. If `sms_opted_in_webinar`: SMS 15 minutes before event. 4. Add to warm retargeting audience for 30 days post-event. 5. Post-event: replay email within 2 hours of session end. |
| PT patient event registered (free webinar, Q&A with Dr. Olson) | 1. Confirmation email (immediate) + calendar link. 2. Reminder email 24 hours before. 3. SMS 1 hour before. 4. Post-event: PT-Implant Nurture sequence if not already running. |

#### Trigger 4: Pricing / Valuation Page Visit

| Trigger | Automatic Actions |
|---------|------------------|
| PA prospect visits `/valuation` or `/pricing` page | 1. Add to PA-Hot-2 audience (pricing intent). 2. Priority alert to BD team via CRM notification (visit logged, page URL included). 3. Google Display retargeting: discovery call CTA only. 4. LinkedIn: discovery call CTA ad. 5. If in email sequence: accelerate to email 8 (the discovery call ask) if Day 12+ in sequence. |

#### Trigger 5: Email Engagement Threshold

| Trigger | Automatic Actions |
|---------|------------------|
| PA lead opens 3+ emails (PA-Indoctrination) | 1. Tag: `pa_indoctrinated`. 2. Start PA-Engagement sequence. 3. Upgrade from PA-Warm to PA-Hot in retargeting audiences. |
| PA/SP lead opens email 6+ times without booking call | 1. Tag: `hot_lead`. 2. Alert BD team for personal outreach (phone or LinkedIn InMail). 3. Escalate from standard retargeting to discovery call CTA retargeting. |
| PT patient email opened 4+ times (PT-Implant Nurture) | 1. Tag: `pt_hot_lead`. 2. Alert front desk for optional personal outreach (practice decision). 3. Intensify Meta retargeting: PT-Hot-1 sequence. |

#### Trigger 6: No Engagement (Re-Engagement)

| Trigger | Automatic Actions |
|---------|------------------|
| No email opens for 60 days (B2B) | 1. Tag: `cold_60d`. 2. Start 3-email re-engagement sequence. 3. Shift retargeting from warm to cold audience window. 4. Reduce ad frequency to 1-2x per week. |
| No email opens for 90 days (patients) | 1. Tag: `cold_90d`. 2. Start patient re-engagement sequence. 3. Shift to PT-Cold retargeting audience. |
| No engagement after re-engagement sequence | 1. Tag: `cold_unresponsive`. 2. Suppress from active email and SMS sequences. 3. Move to quarterly-only cold email touchpoint. 4. Remove from paid retargeting audiences (no more ad spend on confirmed non-responders). |

#### Trigger 7: New Partner Signed (Cross-Funnel)

| Trigger | Automatic Actions |
|---------|------------------|
| New practice partner signed (`pa_customer` tag) | 1. Remove from ALL PA prospect sequences immediately. 2. Add to Partner Pulse newsletter. 3. Automatically trigger DR sequence notification to BD team: "New practice in [Location] -- fill associate positions." 4. Add new practice location to PT ad targeting (Google LSAs, Meta geo-targeting). 5. Route partnership story to content team for use in PA/SP awareness campaigns. |
| New specialist partner signed (`sp_customer` tag) | 1. Remove from all SP sequences. 2. Add to MODIS Partner newsletter. 3. Invite to MODIS Community Conversations as a future guest. 4. Route partnership story to content team. |

---

## Appendix A: Pre-Launch Checklist

### Infrastructure Readiness

- [ ] Meta Pixel installed and firing correctly on all SGA web properties (verify via Meta Pixel Helper)
- [ ] Google Tag Manager container live with correct GTM ID on all pages
- [ ] LinkedIn Insight Tag verified on all SGA, Gen4, and MODIS web properties
- [ ] All custom event triggers configured and tested (ViewContent, Lead, CompleteRegistration, Schedule)
- [ ] UTM parameter guide distributed to all campaign managers and email team
- [ ] 10DLC registration submitted for all SMS campaigns (allow 2-6 weeks)
- [ ] SMS platform integrated with CRM (two-way opt-in/opt-out sync)
- [ ] Exclusion audience lists uploaded to Meta, Google, and LinkedIn
- [ ] Campaign naming convention document distributed and enforced

### Audience Architecture Readiness

- [ ] All retargeting audiences created in Meta Custom Audiences (8 PA segments, 6 PT segments, 3 DR segments, 3 SP segments)
- [ ] All retargeting audiences created in Google Ads RLSA
- [ ] LinkedIn Matched Audiences uploaded (email lists for each warm segment)
- [ ] Lookalike audiences created from assessment completers (PA and PT) -- must have 1,000+ matched users before activation
- [ ] Exclusion audiences applied to all relevant campaigns

### Sequence Readiness

- [ ] All email sequences loaded in email platform with correct triggers and tags
- [ ] SMS sequences programmed with correct triggers, timing rules, and opt-out handling
- [ ] CRM automation rules tested end-to-end (tag triggers, BD team alerts, cross-funnel routing)
- [ ] Re-engagement sequences tested with test contact records
- [ ] BD team trained on CRM hot lead alerts and required response SLA (48 hours for PA/SP hot leads)

### Compliance Readiness

- [ ] TCPA opt-in language reviewed by legal counsel
- [ ] SMS opt-in checkboxes on all collection forms (unchecked by default -- PA, DR, SP; checked by default with easy opt-out -- PT appointment booking)
- [ ] STOP / HELP keywords configured in SMS platform
- [ ] Privacy policy updated to include SMS program description
- [ ] Consent logs verified to capture timestamp, IP, and opt-in language presented
- [ ] Quiet hours configured in SMS platform (8 AM - 9 PM recipient local time for PT; 8 AM - 7 PM for B2B)

---

## Appendix B: Campaign Performance Dashboard

### Weekly Monitoring Metrics

| Metric | PA | DR | PT | SP |
|--------|----|----|----|----|
| Cost per lead (paid) | < $300 target | < $80 target | < $100 target | < $400 target |
| Assessment / guide completion rate | > 35% | > 25% | > 40% | > 60% of page visitors |
| Email sequence open rate | > 40% | > 35% | > 45% | > 45% |
| Retargeting CTR (Meta) | > 0.5% | > 0.8% | > 1.2% | > 0.3% |
| Retargeting CTR (LinkedIn) | > 0.4% | > 0.5% | N/A | > 0.35% |
| SMS delivery rate | > 98% | > 98% | > 98% | > 98% |
| SMS opt-out rate | < 0.5% | < 0.5% | < 1% | < 0.3% |
| Discovery calls / consultations booked | 8-15/month | N/A | 20-50/month | 5-10/quarter |

### Monthly Decision Framework

At the end of each month, answer these four questions:

1. **Which channel delivered the lowest cost per qualified outcome?** Increase that channel's budget by 20% next month.
2. **Which channel's CPL exceeded target by more than 40%?** Pause or reduce to minimum test budget.
3. **Which creative has frequency > 4.0 on Meta or > 3.0 average weekly impressions per person on LinkedIn?** Replace it before the next campaign month begins.
4. **Are any exclusion lists outdated?** Refresh uploads to all platforms within 72 hours of month-end review.

---

*Document total: ~11,500 words*
*Prepared for SGA Dental Partners -- March 2026*
*Integrates with: product_brief.md, funnel_architecture.md, positioning.md, channel_analysis.md*
*For copy execution: coordinate with marketing-copywriter and marketing-email-specialist*
*For ad buying and optimization: coordinate with marketing-paid-ads-specialist*
