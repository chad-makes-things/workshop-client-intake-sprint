# SGA Dental Partners -- 90-Day Launch Roadmap

**Prepared:** March 2026
**Scope:** Full go-to-market launch across 4 funnels, 7 channels, and 5 target avatars
**Prerequisite Documents:** Product Brief, Funnel Architecture, Positioning, Social Growth Strategy, A/B Test Playbook
**Asset Inventory:** 55+ marketing assets ready for deployment across landing pages, ads, emails, social, SEO, lead magnets, scorecard, VSL, webinar, chatbot, competitor pages, case studies, and sales enablement

---

## 1. Executive Summary

### Launch Philosophy

This is not a consumer product launch. SGA Dental Partners is consolidating three entities (SGA, Gen4, MODIS) under one brand while simultaneously running four distinct funnels targeting audiences with fundamentally different buying timelines. A practice owner making a once-in-a-career $2.4M decision over 6-18 months cannot be launched at the same way as an implant patient booking a consultation in 2-8 weeks.

The launch philosophy is **staged deployment with controlled ignition**:

1. **Build the engine before turning the key.** Weeks 1-2 install tracking, configure automation, QA all assets, and align the team. Nothing goes live until the measurement infrastructure is proven.
2. **Light one fire at a time.** Launch week activates channels in a deliberate sequence -- organic before paid, owned before rented, high-intent before awareness. This prevents a flood of traffic hitting unfinished pages or untested automations.
3. **Optimize before scaling.** Weeks 4-12 run the A/B test program, refine based on data, and only increase paid spend on channels that prove unit economics. Scaling a broken funnel wastes budget. Scaling a proven funnel compounds returns.

The V1 launch (Weeks 1-3) activates the core funnel assets: landing pages, email sequences, paid ads, SEO foundation, and social organic. V2 assets (scorecard, webinar, YouTube series, chatbot, competitor pages, story emails) deploy on a staggered schedule through Weeks 4-12 to sustain momentum, add funnel layers, and prevent audience fatigue.

### Key Milestones

| Milestone | Target Date | Success Criteria |
|-----------|-------------|------------------|
| Infrastructure complete | End of Week 2 | All tracking pixels, UTMs, email automation, and CRM integrations verified |
| V1 launch complete | End of Week 3 | All 5 landing pages live, email sequences active, paid ads running on Google + LinkedIn + Meta, social posting cadence established |
| First A/B test results | End of Week 5 | LP-1 (headline) and AD-1 (hook type) winners declared |
| First discovery calls from marketing | End of Week 6 | Minimum 5 discovery calls booked through marketing-generated leads |
| Scorecard funnel live | End of Week 6 | Partnership Readiness Scorecard deployed with follow-up email sequences |
| First webinar hosted | Week 8 | Minimum 75 registrations, 35% attendance rate |
| YouTube content series launched | Week 5 | First 4 videos from content calendar published |
| Chatbot deployed | Week 7 | Live on main landing page and patient landing page |
| 90-day review complete | End of Week 12 | Full funnel performance review, A/B test cumulative lift calculated, budget reallocation based on data |

### 90-Day Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Practice Partnership Assessment completions | 150+ | CRM + quiz tool |
| Discovery calls booked (PA + SP funnels) | 30-45 | CRM calendar integration |
| Associate dentist applications generated | 50+ | ATS tracking |
| Implant patient consultations booked | 100+ (across network) | Practice management systems |
| Email list size (all audiences combined) | 2,000+ | Email platform |
| Combined social following growth | +5,000 across platforms | Platform analytics |
| Organic search impressions for target keywords | 50,000+ monthly by Week 12 | Google Search Console |
| Cost per qualified lead (PA funnel) | Under $350 | Ad platform + CRM attribution |
| Cost per lead (PT implant funnel) | Under $120 | Ad platform + booking system |
| Funnel-wide conversion lift from A/B testing | 25-40% aggregate improvement | Test results documentation |

---

## 2. Pre-Launch Phase (Weeks 1-2)

### Week 1: Infrastructure and Internal Alignment

#### Day 1-2: Tracking and Analytics Setup

| Task | Detail | Owner |
|------|--------|-------|
| Install Google Tag Manager container | Deploy across sgadental.com and all landing page subdomains. Create a single GTM container with environment-specific configurations. | Marketing Ops |
| Configure GA4 property | Set up conversion events: assessment_start, assessment_complete, discovery_call_booked, guide_download, consultation_booked, application_submitted. Enable enhanced measurement. | Marketing Ops |
| Set up UTM convention | Standard format: `utm_source / utm_medium / utm_campaign / utm_content / utm_term`. Document in shared spreadsheet. All team members trained on convention before any links go live. | Marketing Ops |
| Install Meta Pixel | Deploy across all pages. Configure standard events (Lead, CompleteRegistration, Schedule) and custom events matching GA4 conversion events. Verify with Meta Pixel Helper. | Paid Media |
| Install LinkedIn Insight Tag | Deploy across all pages. Configure conversion tracking for Lead Gen Form completions and landing page conversions. | Paid Media |
| Configure Google Ads conversion tracking | Set up conversion actions: assessment_complete, discovery_call_booked (offline import from CRM), consultation_booked. Enable Enhanced Conversions. | Paid Media |
| Set up heatmapping | Deploy Hotjar or Microsoft Clarity on all landing pages. Configure recording for assessment flow, pricing page, and webinar registration page. | Marketing Ops |

#### Day 2-3: Email Platform Configuration

| Task | Detail | Owner |
|------|--------|-------|
| Configure email sending domain | Authenticate SPF, DKIM, and DMARC for the sending domain. Warm the domain with low-volume sends for 7-10 days before full deployment. | Marketing Ops |
| Build tag architecture | Implement the full tag taxonomy from the Funnel Architecture document (pa_assessment_complete, pa_tier_strong, dr_guide_download, pt_assessment_complete, sp_guide_download, etc.). | Marketing Ops |
| Build automation triggers | Configure conditional logic: tag-based sequence entry, 3+ opens triggering engagement sequences, cross-funnel routing rules. | Marketing Ops |
| Load email sequences | Import all 5 email sequences: PA-Indoctrination (5 emails), PA-Engagement (8 emails), DR-Nurture (4 emails), PT-Implant Nurture (6 emails), SP-Indoctrination (4 emails). QA every link, merge tag, and conditional block. | Content + Marketing Ops |
| Configure re-engagement triggers | Set cold_60d and cold_90d tags to fire automatically based on email engagement data. Build re-engagement sequence (3 emails). | Marketing Ops |
| Test email deliverability | Send test emails to Litmus or Email on Acid. Verify rendering across Gmail, Outlook, Apple Mail (desktop and mobile). Fix any formatting issues before launch. | Marketing Ops |

#### Day 3-4: CRM and Sales Integration

| Task | Detail | Owner |
|------|--------|-------|
| Configure CRM lead routing | Assessment completions with score > 80 route to BD team within 48 hours. Score < 40 routes to educational content track. All leads tagged by source and funnel. | Sales Ops |
| Build BD team dashboard | Real-time view of: new assessment completions, discovery calls booked, lead score distribution, lead source attribution. | Sales Ops |
| Brief BD team on messaging | Walk through the positioning document, objection-handling language, and the messaging hierarchy. BD team must speak the same language as the marketing assets. | Marketing Lead + BD Lead |
| Set up call booking system | Configure Calendly or HubSpot meeting links for discovery calls. Embed in email CTAs and landing page CTAs. Set up confirmation and reminder emails. | Sales Ops |
| Configure lead-to-opportunity handoff | Define the criteria for marketing-qualified lead (MQL) vs. sales-qualified lead (SQL). MQL: completed assessment + opened 3+ emails OR clicked CTA 3+ times. SQL: booked discovery call. | Marketing Lead + Sales Ops |

#### Day 4-5: Landing Page QA and Deployment Preparation

| Task | Detail | Owner |
|------|--------|-------|
| QA main landing page | Test all links, form submissions, mobile responsiveness, page load speed (<3 seconds), and CTA button tracking. Verify assessment embed loads correctly. | Marketing Ops + QA |
| QA associate landing page | Same QA protocol. Verify career guide download delivery, application links, and job listing integration. | Marketing Ops + QA |
| QA patient landing page | Same QA protocol. Verify Smile Assessment embed, consultation booking integration, Dr. Olson video embeds, and financing calculator if applicable. | Marketing Ops + QA |
| QA pricing page | Verify tier section functionality, CTA tracking at each tier level, and value equation table rendering on mobile. | Marketing Ops + QA |
| QA competitor comparison page | Verify all competitor data is current and accurate. Check that no competitor claims could trigger legal issues. | Marketing Lead + Legal |
| Stage all pages in preview/staging | All 5 landing pages staged and accessible via private links. Internal team reviews for copy accuracy, brand alignment, and factual claims. | Marketing Lead |
| Prepare lead magnet delivery | Test Practice Partnership Assessment delivery flow: completion > email capture > instant delivery email > results page. Verify scoring logic across all 3 tiers. | Marketing Ops |

#### Day 5: Internal Team Alignment

| Task | Detail | Owner |
|------|--------|-------|
| Launch kickoff meeting | All stakeholders (marketing, BD, practice ops, leadership) aligned on launch timeline, success metrics, and escalation paths. | Marketing Lead |
| Distribute sales enablement kit | Battle cards, elevator pitches, FAQ document, and one-pager to all BD team members and practice managers. | Marketing Lead |
| Brief practice-level teams | Inform office managers (Tamara Reeves persona) at all practices about the marketing launch. Provide talking points for any patient or staff questions about the SGA brand. | Practice Ops |
| Align leadership on social posting | Confirm Lamonte Jensen, Dr. Ellingson, Pete Wong, and Dr. Olson are briefed on the LinkedIn thought leadership cadence and have approved their first 2 weeks of content. | Content Lead |

### Week 2: Soft Launch and Domain Warming

#### Day 6-7: Domain and Ad Account Warming

| Task | Detail | Owner |
|------|--------|-------|
| Begin email domain warming | Send 50-100 emails per day to known contacts (existing SGA/Gen4/MODIS partner communication lists with confirmed opt-in). Gradually increase volume by 25% daily. | Marketing Ops |
| Set up ad accounts | Create campaigns in draft/paused state across Google Ads, LinkedIn Campaign Manager, Meta Business Suite. Upload all creative assets. Configure audience targeting. | Paid Media |
| Build retargeting audiences | Create website visitor audiences (30-day, 90-day, 180-day) on Meta and Google. Create engagement audiences from existing YouTube viewers and LinkedIn page visitors. | Paid Media |
| Create lookalike audiences | Build 1% lookalike audiences from existing SGA/Gen4/MODIS patient lists (patient funnel) and from dentist contact lists (practice acquisition funnel). Requires uploading hashed email lists to Meta and Google. | Paid Media |

#### Day 8-9: Content Pre-Staging

| Task | Detail | Owner |
|------|--------|-------|
| Schedule LinkedIn posts for Weeks 3-4 | Queue 3-4 posts per week for each leader (Jensen, Ellingson, Olson, Wong) in a scheduling tool. Content drawn from the LinkedIn calendar asset. | Content Lead |
| Schedule Facebook/Instagram posts for Weeks 3-4 | Queue 5-7 posts per week for the central SGA brand pages. Content drawn from the Facebook/Instagram calendar asset. | Content Lead |
| Prepare YouTube video backlog | If Dr. Olson has pre-recorded content, edit and queue first 4 videos per the YouTube calendar. If not, schedule recording sessions for Week 3. | Video Production |
| Build content kit for local practices | Create the first monthly content kit with 12-15 ready-to-post assets (graphics, captions, video clips) that local office managers can customize and post on their practice pages. | Content Lead |
| Stage SEO pillar page | Finalize "How to Sell Your Dental Practice" pillar page. Internal link structure connecting to supporting blog posts. Schema markup implemented per the SEO asset specifications. | SEO Lead |
| Write and schedule first 4 blog posts | Prioritize highest-opportunity keywords from the keyword map: "dental practice valuation," "DSO vs private practice," "what is a DSO," "sell dental practice." | Content Lead |

#### Day 9-10: Final QA and Go/No-Go

| Task | Detail | Owner |
|------|--------|-------|
| End-to-end funnel test (PA) | One team member completes the entire PA funnel: clicks ad > lands on page > starts assessment > completes assessment > receives email > opens/clicks through 3 emails > books discovery call. Verify every step fires correctly in analytics, CRM, and email platform. | Marketing Ops |
| End-to-end funnel test (PT) | Same for patient funnel: clicks ad > lands on page > starts Smile Assessment > completes > receives email > booking CTA works. | Marketing Ops |
| End-to-end funnel test (DR) | Same for recruitment funnel: career page > guide download > email sequence > application link. | Marketing Ops |
| Verify all conversion tracking | Cross-reference GA4 events, Meta Pixel events, LinkedIn Insight Tag events, and CRM data. All systems must show the same conversion count (within 5% margin for platform attribution differences). | Marketing Ops |
| Go/No-Go decision | Marketing Lead confirms all systems are operational. Any unresolved issues get documented with a workaround plan. Launch proceeds only if core tracking and automation are verified. | Marketing Lead |

---

## 3. Launch Week (Week 3)

### Launch Sequence Philosophy

Channels activate in a deliberate order based on three principles:

1. **Organic before paid.** Organic content establishes credibility before paid ads drive cold traffic. A LinkedIn post from Lamonte Jensen that generates engagement provides social proof context for the Thought Leader Ad that runs the following day.
2. **Owned before rented.** Email and website assets go live before ads activate. This ensures every paid click has a functioning destination with proven delivery mechanics.
3. **High-intent before awareness.** Google Search (capturing existing demand) activates before Meta and LinkedIn (creating new demand). This generates early conversions that validate the funnel before scaling spend.

### Day-by-Day Deployment

#### Monday (Day 1): Owned Assets Go Live

| Time | Action | Channel | Funnel | Notes |
|------|--------|---------|--------|-------|
| 9:00 AM | Publish main landing page | Website | PA | Remove staging gate. Verify all tracking fires on first real pageview. |
| 9:00 AM | Publish associate landing page | Website | DR | Same verification protocol. |
| 9:00 AM | Publish patient landing page | Website | PT | Same verification protocol. |
| 9:00 AM | Publish pricing page | Website | PA | Same verification protocol. |
| 9:00 AM | Publish SEO pillar page + first 2 blog posts | Website | PA/DR | Submit sitemap to Google Search Console. Request indexing for priority pages. |
| 10:00 AM | Activate PA-Indoctrination email sequence | Email | PA | Confirm automation trigger is armed. Send test to internal address to verify. |
| 10:00 AM | Activate DR-Nurture email sequence | Email | DR | Same verification. |
| 10:00 AM | Activate PT-Implant Nurture email sequence | Email | PT | Same verification. |
| 11:00 AM | Send internal launch announcement | Internal | All | Notify all SGA partners and office managers that the marketing engine is live. Include talking points and FAQ. |
| 12:00 PM | Post first LinkedIn thought leadership piece | LinkedIn | PA/SP | Lamonte Jensen publishes the "7 out of 10 dentists regret their DSO sale" perspective piece. No paid amplification yet -- let it gain organic traction. |
| 2:00 PM | Post first Facebook/Instagram content | Meta | PT | Dr. Olson patient transformation content on central SGA pages. Warm the page with organic engagement. |

#### Tuesday (Day 2): Search Capture Activates

| Time | Action | Channel | Funnel | Notes |
|------|--------|---------|--------|-------|
| 8:00 AM | Activate Google Search campaigns (PA) | Google Ads | PA | Target keywords: "sell dental practice," "dental practice valuation," "DSO partnership." Start at 50% of planned daily budget. |
| 8:00 AM | Activate Google Search campaigns (PT) | Google Ads | PT | Target keywords: "dental implants cost," "full arch dental implants [city]," "All-on-4 near me." Start at 50% budget. |
| 8:00 AM | Activate Google LSAs for top 10 practices | Google LSAs | PT | Local Services Ads for highest-priority practice locations. Verify phone tracking numbers are routed correctly. |
| 10:00 AM | LinkedIn thought leadership post #2 | LinkedIn | PA | Dr. Ellingson publishes on clinical autonomy and the "last great profession" theme. |
| 12:00 PM | Facebook/Instagram post #2 | Meta | PT/DR | Behind-the-scenes content from an SGA practice. Dual-purpose: patient trust + recruitment awareness. |
| 3:00 PM | Monitor Day 1-2 analytics | All | All | Verify conversion tracking is firing. Check for any broken links, form errors, or delivery failures. Fix immediately. |

#### Wednesday (Day 3): LinkedIn Paid Activates

| Time | Action | Channel | Funnel | Notes |
|------|--------|---------|--------|-------|
| 8:00 AM | Activate LinkedIn Thought Leader Ads | LinkedIn Ads | PA | Amplify Jensen's Monday post as a Thought Leader Ad. Target: practice owners, dental group executives, 10+ years experience, in target geographies. Start at 50% budget. |
| 8:00 AM | Activate LinkedIn Sponsored Jobs | LinkedIn Ads | DR | Promote highest-priority open positions with differentiated copy (mentorship, equity pathway, Pikos Institute access). |
| 10:00 AM | LinkedIn thought leadership post #3 | LinkedIn | SP | Pete Wong publishes on the MODIS specialist platform and the future of implantology practice models. |
| 12:00 PM | Facebook/Instagram post #3 | Meta | PT | Patient education content: "What dental implants actually cost (honest numbers)." |
| 2:00 PM | Distribute content kit to local practices | Internal | PT | First weekly content kit goes to office managers with posting instructions. |

#### Thursday (Day 4): Meta Paid Activates

| Time | Action | Channel | Funnel | Notes |
|------|--------|---------|--------|-------|
| 8:00 AM | Activate Facebook/Instagram ads (PA) | Meta Ads | PA | Launch "Sunday Night" and "Horror Stories" ad variations simultaneously (Test AD-1 begins). Target practice owners via interest + behavior targeting. Start at 50% budget. |
| 8:00 AM | Activate Facebook/Instagram ads (PT) | Meta Ads | PT | Launch implant patient ads: "Life's Moments" emotional hook and "Steak Moment" creative. Target adults 35-65 within 15-mile radius of top MODIS practices. Start at 50% budget. |
| 10:00 AM | LinkedIn thought leadership post #4 | LinkedIn | PA | Dr. Grant Olson publishes clinical innovation content repurposed from YouTube for professional audience. |
| 12:00 PM | Facebook/Instagram post #4 | Meta | PT | Patient transformation reveal content. |
| 3:00 PM | Review first 48 hours of Google Ads data | Google Ads | PA/PT | Check CTR, CPC, conversion tracking. Pause any keywords with CPC above 2x target. Add negative keywords for irrelevant queries. |

#### Friday (Day 5): Full Activation and Monitoring

| Time | Action | Channel | Funnel | Notes |
|------|--------|---------|--------|-------|
| 8:00 AM | Increase all paid budgets to 75% | All paid | All | If Days 2-4 show acceptable CPC and no tracking issues, scale to 75% of planned budget. |
| 9:00 AM | Publish blog posts #3 and #4 | Website | PA/DR | "What Is a 4th Generation DSO" and "DSO Dentist Salary Guide." Promote via LinkedIn and email. |
| 10:00 AM | Launch PR/outreach wave | PR | PA/SP | Send press release to Dental Economics, Group Dentistry Now, Becker's Dental Review. Position: SGA Dental Partners unifies Gen4 and MODIS under the "partnership that makes dentists better" positioning. |
| 10:00 AM | LinkedIn thought leadership post #5 | LinkedIn | PA | Jensen or Ellingson shares the press release with personal commentary. |
| 12:00 PM | Facebook/Instagram post #5 | Meta | All | Milestone post: "SGA Dental Partners -- bringing together [X] practices in [X] states. Here is what we stand for." |
| 3:00 PM | Distribute first weekly performance snapshot | Internal | All | Impressions, clicks, conversions, CPL by channel. Share with leadership and BD team. |

#### Weekend (Days 6-7): Monitoring and Engagement

| Task | Detail | Owner |
|------|--------|-------|
| Monitor ad spend and pacing | Ensure no campaigns overspend. Pause any ads with anomalous behavior. | Paid Media |
| Respond to all social comments | LinkedIn, Facebook, Instagram. Dr. O or community manager handles within 4 hours on weekdays, 12 hours on weekends. | Community Manager |
| Review email delivery metrics | Open rates on first emails sent Monday-Friday. Flag any deliverability issues (bounce rate > 2%, spam rate > 0.1%). | Marketing Ops |
| Engagement analysis | Which LinkedIn posts gained the most traction? Which ad creative has the highest CTR? Document early signals. | Marketing Lead |

---

## 4. Post-Launch Sustain (Weeks 4-12)

### Weekly Operating Rhythm

Every week follows a consistent cadence to maintain momentum without burning out the team.

| Day | Activity | Owner |
|-----|----------|-------|
| Monday | Weekly planning: review previous week's metrics, prioritize this week's content and tests, address any issues | Marketing Lead |
| Tuesday | Content production: LinkedIn posts drafted, social content created, blog posts written or edited | Content Lead |
| Wednesday | Optimization: A/B test check-ins, ad creative refresh, email sequence performance review | Paid Media + Marketing Ops |
| Thursday | Distribution: schedule content, deploy new ad creative, activate new automation rules | Content Lead + Paid Media |
| Friday | Reporting: weekly performance snapshot distributed to leadership and BD team; document learnings | Marketing Lead |

### Optimization Cadence

| Cadence | Action | Detail |
|---------|--------|--------|
| Daily | Monitor ad spend pacing | Ensure no campaign overspends. Check for anomalous CPC spikes. Pause underperformers. |
| Daily | Respond to all inbound leads | Assessment completions, guide downloads, and form submissions routed to BD within 4 hours during business hours. |
| Daily | Social engagement | Reply to comments, DMs, and mentions within 4 hours (weekdays) / 12 hours (weekends). |
| Weekly | Performance snapshot | Consolidated report: spend, impressions, clicks, conversions, CPL, pipeline by channel. |
| Weekly | Ad creative review | Identify creative fatigue (CTR decline > 20% week-over-week). Rotate in new creative from the ad asset library. |
| Weekly | Email health check | Open rates, click rates, unsubscribe rates, bounce rates per sequence. Flag any sequence with open rate below 25% for subject line review. |
| Biweekly | A/B test review | Evaluate running tests against minimum sample thresholds. Declare winners when statistical significance is reached. Document results. |
| Biweekly | SEO check | Monitor keyword ranking movement, organic traffic trends, and index coverage. Publish next 2 blog posts per the keyword map. |
| Monthly | Full funnel review | End-to-end conversion analysis: awareness > lead > MQL > SQL > discovery call > site visit > LOI. Identify the weakest stage. Allocate next month's optimization budget to that stage. |
| Monthly | Budget reallocation | Shift spend from underperforming channels to overperformers. Minimum 2 weeks of data before making reallocation decisions. |
| Monthly | Content calendar review | Assess what content performed best. Double down on winning formats. Retire underperformers. |

### Paid Scaling Rules

Paid spend follows a disciplined scaling protocol to prevent wasting budget on unproven channels.

| Rule | Threshold | Action |
|------|-----------|--------|
| Scale up | Channel delivers CPL below target for 2 consecutive weeks with 30+ conversions | Increase budget by 25%. Monitor CPL for 1 week. If CPL holds, increase another 25%. |
| Hold | Channel delivers CPL within 10% of target | Maintain current budget. Focus on creative testing to improve efficiency. |
| Scale down | Channel delivers CPL above target for 2 consecutive weeks | Reduce budget by 25%. Diagnose: is it creative fatigue, audience exhaustion, or landing page issues? Fix the root cause before re-scaling. |
| Pause | Channel delivers CPL more than 2x target for 3 consecutive weeks with no improvement trend | Pause the channel. Redirect budget to top performers. Diagnose and rebuild before reactivating. |
| Test budget | New channel or new creative concept | Allocate 10-15% of total monthly budget to testing. Minimum 2-week test duration. Evaluate against same CPL targets. |

### Content Rhythm

| Content Type | Frequency | Channel | Owner |
|--------------|-----------|---------|-------|
| LinkedIn thought leadership (4 leaders) | 3-4 posts per leader per week | LinkedIn | Each leader + Content Lead support |
| Facebook/Instagram central page | 5-7 posts per week | Meta | Content Lead |
| Local practice content kit | Monthly kit, 12-15 assets | Meta (local pages) | Content Lead + Practice Ops |
| Blog posts (SEO) | 2 posts per week | Website | Content Lead + SEO Lead |
| YouTube long-form (Dr. Olson) | 2 videos per week | YouTube | Video Production |
| YouTube Shorts / Reels / TikTok | 4-5 clips per week | YouTube, IG, TikTok, FB | Video Production |
| Email newsletter (Partner Pulse) | Monthly | Email | Content Lead |
| Podcast appearances (Jensen, Wong) | 1-2 per month | External podcasts | Marketing Lead + PR |

### A/B Test Schedule Integration

The A/B test playbook runs in parallel with the launch sustain phase. Tests are synchronized so that upstream tests (headlines, hooks) resolve before downstream tests (email sequences, pricing page layout) begin.

#### Phase 1 Tests (Weeks 3-6) -- Quick Wins

| Test | Start | Expected Winner Declaration | Assets Affected |
|------|-------|----------------------------|-----------------|
| LP-1: Headline Framing (Empathy vs. Fear-Stat) | Week 3, Day 1 (launch day) | Week 6 | Main landing page |
| LP-2: CTA Button Copy (Professional vs. Peer-Framed) | Week 3, Day 1 | Week 6 | Main landing page (all CTA instances) |
| AD-1: Hook Type (Sunday Night vs. Horror Stories) | Week 3, Day 4 (Meta activation) | Week 5 | Facebook ads (PA funnel) |
| EM-1: Subject Line (Curiosity vs. Direct) | Week 3, Day 1 | Week 7-8 | Welcome sequence email 1 |
| LP-6: Exit Intent Popup | Week 4 | Week 6 | Main landing page |
| AD-2: CTA Type (Direct vs. Transitional) | Week 4 | Week 7 | Facebook + LinkedIn ads (PA funnel) |
| LP-7: Specific Numbers in Social Proof Bar | Week 4 | Week 7 | Main landing page |
| QS-3: Follow-Up Timing (Immediate vs. 30-min) | Week 5 | Week 9 | Scorecard follow-up automation |
| AD-5: Emotional vs. Rational Angle | Week 5 | Week 7 | Facebook ads (PT funnel) |

**Phase 1 Checkpoint (End of Week 6):** Review results, implement winners, calculate cumulative lift. Expected: 15-25% improvement in landing page conversion rate, 20-30% improvement in ad CPL.

#### Phase 2 Tests (Weeks 7-10) -- High-Impact

| Test | Start | Expected Winner Declaration | Assets Affected |
|------|-------|----------------------------|-----------------|
| LP-3: Social Proof Format (Plain vs. Facebook-Style) | Week 7 | Week 10 | Main landing page testimonial section |
| LP-5: Section Order (Testimonials above offer) | Week 7 | Week 10 | Main landing page layout |
| WB-1: Webinar Title (Educational vs. Curiosity) | Week 8 (first webinar) | Week 12 (second webinar) | Webinar registration page |
| AD-3: Creative Format (Single Image vs. Carousel) | Week 7 | Week 9 | Facebook ads (PT funnel) |
| EM-2: Send Time (6:30 AM vs. 7:30 PM) | Week 7 | Week 12+ | All PA sequence emails |
| QS-1: Question Order | Week 8 | Week 12 | Partnership Readiness Scorecard |
| QS-2: Results Gating | Week 8 | Week 12 | Partnership Readiness Scorecard |
| PP-1: Pricing Headline | Week 8 | Week 12 | Pricing page |
| AD-4: Thought Leader vs. Company Page | Week 9 | Week 12 | LinkedIn ads (PA funnel) |
| EM-3: Sender Name | Week 9 | Week 12+ | PA welcome sequence |

**Phase 2 Checkpoint (End of Week 10):** Review results, implement winners. Expected: 15-30% additional improvement on Phase 1 gains.

#### Phase 3 Tests (Weeks 11-12+) -- Strategic Experiments

| Test | Start | Expected Winner Declaration | Assets Affected |
|------|-------|----------------------------|-----------------|
| LP-8: Video Testimonial Compilation | Week 11 | Week 14+ | Main landing page (below hero) |
| LP-4: Problem Section Length | Week 11 | Week 14 | Main landing page |
| WB-2: Registration Bullets (7 vs. 3) | Week 12 (second webinar) | Week 16 | Webinar registration page |
| WB-3: Pitch Timing (Trial close) | Week 12 | Week 20 (4 webinar cycles) | Webinar script |
| EM-4: Social Proof Email Format | Week 12 | Week 16+ | PA sales sequence email 3 |
| EM-5: Sequence Length (7 vs. 5 emails) | Week 12 | Week 20+ | PA welcome sequence |

Phase 3 tests extend beyond the 90-day launch window. They are started within the window but may require additional time to reach statistical significance given lower traffic on pricing pages and webinar registration.

---

## 5. Week-by-Week Calendar

### Week 1: Infrastructure Build

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Tracking | GTM, GA4, Meta Pixel, LinkedIn Insight Tag, Google Ads conversions, Hotjar | All digital properties |
| Email | Domain authentication, tag architecture, automation triggers, all 5 sequences loaded and QA'd | Email platform |
| CRM | Lead routing, BD dashboard, MQL/SQL definitions, call booking system | CRM |
| Sales | Battle cards, elevator pitches, FAQ, one-pager distributed to BD team | Internal |
| QA | End-to-end funnel tests for PA, PT, DR funnels | All |

### Week 2: Soft Launch and Warming

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Email | Domain warming (50-100 sends/day, gradually increasing) | Email |
| Ads | Campaign drafts built, audiences configured, creative uploaded (not yet live) | Google, LinkedIn, Meta |
| Content | LinkedIn posts scheduled (Weeks 3-4), Facebook/Instagram posts scheduled (Weeks 3-4) | LinkedIn, Meta |
| SEO | Pillar page staged, first 4 blog posts staged, schema markup implemented | Website |
| Video | First YouTube videos edited and queued (if pre-recorded content exists) | YouTube |
| Local | First content kit prepared for local practice pages | Meta (local) |

### Week 3: Launch Week

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Website | Main LP, associate LP, patient LP, pricing page, competitor comparison page, pillar page, 4 blog posts -- all live | Website |
| Email | PA-Indoctrination, DR-Nurture, PT-Implant Nurture, re-engagement sequences -- all active | Email |
| Ads | Google Search (PA + PT), Google LSAs (top 10 practices), LinkedIn Thought Leader Ads, LinkedIn Sponsored Jobs, Facebook/Instagram ads (PA + PT) | Google, LinkedIn, Meta |
| Social | LinkedIn thought leadership begins (4 leaders posting), central Facebook/Instagram posting begins | LinkedIn, Meta |
| PR | Press release to dental trade publications | Trade media |
| Sales | Lead flow begins to BD team | CRM |
| Tests | LP-1, LP-2, AD-1, EM-1 begin | Landing page, ads, email |

### Week 4: First Optimization Cycle

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Ads | Scale to 100% budget on channels meeting CPL targets. Pause/adjust underperformers. New creative variations rotated in. | Google, LinkedIn, Meta |
| Content | Blog posts #5-6 published per SEO keyword map. Second LinkedIn content wave. | Website, LinkedIn |
| Email | Monitor sequence performance. First deliverability review. | Email |
| Tests | LP-6 (exit intent popup), AD-2 (CTA type), LP-7 (specific numbers) begin | Landing page, ads |
| Social | Content kit #2 distributed to local practices. First YouTube Shorts cross-posted to TikTok and Instagram Reels. | Meta, TikTok, YouTube |
| Lead Magnet | Practice Partnership Assessment fully operational. First assessment completion data analyzed for tier distribution and BD follow-up effectiveness. | Website, CRM |

### Week 5: YouTube and TikTok Launch + Scorecard Preparation

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| YouTube | First 4 videos from YouTube calendar published. Description links optimized with YourSmileDestination.com and Implant Cost Guide PDF. Pinned comments added. | YouTube |
| TikTok | TikTok account created. First 10 videos posted (cross-posted from YouTube Shorts, re-exported without watermark). | TikTok |
| Social | LinkedIn employee advocacy pilot begins (10-15 partners). Content shared with advocacy library. | LinkedIn |
| SEO | Blog posts #7-8. Monitor keyword ranking movement. Submit new URLs for indexing. | Website |
| Tests | AD-5 (emotional vs. rational, PT funnel), QS-3 (follow-up timing) begin | Ads, scorecard |
| Scorecard | Partnership Readiness Scorecard final QA. Scoring logic tested. Results pages built for all 3 tiers. Follow-up email sequences loaded. | Website, email |
| Content | First podcast appearance confirmed for Jensen or Wong (Shared Practices, Dental Hacks, or Dentist Money Show). | Podcast |

### Week 6: Scorecard Funnel Goes Live

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Scorecard | Partnership Readiness Scorecard live. 15-question assessment with 5 scoring categories. 3-tier results (Strong Position, Growth Opportunity, Foundation Building). Follow-up email sequences active. Scorecard promotion plan activated. | Website, email |
| Ads | Scorecard promoted via LinkedIn Document Ads and Facebook Lead Ads as a micro-commitment entry point. Test against direct landing page CTA. | LinkedIn, Meta |
| Content | Scorecard results used as content: "The 5 categories that predict partnership readiness" LinkedIn post from Jensen. | LinkedIn |
| Email | SP-Indoctrination sequence activated (Specialist funnel). Specialist's Guide white paper delivery tested. | Email |
| Tests | Phase 1 checkpoint. LP-1, LP-2, AD-1 results reviewed. Winners declared and implemented. | All |
| PR | Follow-up pitches to trade publications. Offer byline articles from Jensen or Ellingson. | Trade media |

### Week 7: Webinar Preparation + Chatbot Deployment

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Webinar | Registration page built. Title per webinar strategy (Option A or B based on WB-1 test allocation). 7-bullet content description. Speaker bios (Jensen + Ellingson). Registration form with Lead Gen integration. | Website |
| Webinar | Pre-webinar email sequence built: confirmation, 7-day reminder, 1-day reminder, 1-hour reminder, post-webinar replay + CTA. | Email |
| Chatbot | Chatbot flow deployed on main landing page and patient landing page. Routing logic: practice owner questions to BD, patient questions to nearest practice, career questions to HR. 24/7 automated responses with business-hours handoff to human. | Website |
| Ads | Begin webinar registration ads on LinkedIn and Facebook. Target warm audiences first (website visitors, email subscribers). | LinkedIn, Meta |
| Content | Blog posts #9-10. Continue YouTube cadence (2 long-form + 4-5 Shorts per week). | Website, YouTube |
| Tests | LP-3 (social proof format), LP-5 (section order), AD-3 (creative format), EM-2 (send time) begin | Landing page, ads, email |
| Competitor | First competitor comparison page published: "SGA vs. Heartland Dental" (highest search volume competitor comparison). | Website |

### Week 8: First Webinar + Story Emails Activate

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Webinar | First live webinar hosted. "The Dental Partnership Decision" -- Jensen and Ellingson present the 7-question framework. 60 minutes teaching + 15 minutes Q&A + 10 minutes pitch. Target: 75+ registrations, 35%+ attendance. | Webinar platform |
| Webinar | Post-webinar sequence: replay email (Day 0), social proof email (Day 2), urgency/scarcity email (Day 5 -- limited discovery call slots), final CTA email (Day 7). | Email |
| Story Emails | Soap opera sequence activated for the PA funnel. 5-email narrative arc triggered for leads who have completed indoctrination but have not booked a discovery call. Builds emotional connection through partner transformation stories. | Email |
| Content | Webinar recording edited into 5-7 short clips for LinkedIn and YouTube. Best Q&A moments become standalone content pieces. | LinkedIn, YouTube |
| Tests | QS-1 (question order), QS-2 (results gating), PP-1 (pricing headline), WB-1 (webinar title -- first data point) begin | Scorecard, pricing page, webinar |
| Ads | Webinar replay promoted to non-attendees. Google Search campaigns expanded to include new keywords identified from search term reports. | Google, LinkedIn, Meta |

### Week 9: Competitor Pages Roll-Out + Second Ad Wave

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Competitor | Second and third competitor comparison pages published: "SGA vs. MB2 Dental" and "SGA vs. Pacific Dental Services." Promoted via targeted Google Search ads on competitor brand keywords. | Website, Google Ads |
| Ads | New ad creative wave deployed across all platforms. Rotate in 3-4 fresh creative concepts per channel to combat creative fatigue. Include testimonial-based ads, data-led ads, and transformation story ads. | Google, LinkedIn, Meta |
| Tests | AD-4 (Thought Leader vs. Company Page), EM-3 (sender name) begin. Phase 2 mid-point check. | LinkedIn ads, email |
| Content | Blog posts #11-12. YouTube continues at 2/week cadence. TikTok scaling to 5-7 posts/week. | Website, YouTube, TikTok |
| Email | PA-Engagement sequence (8 emails, Days 8-35) now has first cohort completing. Review full-sequence performance: open rates, click rates, discovery call bookings per 100 sequence entrants. | Email |
| Social | LinkedIn employee advocacy program: first performance review. Which partners are most active? Which content formats get the most reshares? | LinkedIn |

### Week 10: Second Webinar + YouTube Series Midpoint

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Webinar | Second live webinar. Apply WB-1 learnings (if first webinar had sufficient data, use the better-performing title). Apply WB-3 (trial close at minute 45 vs. 55 -- different approach from first webinar for testing). Target: 100+ registrations (warm + cold audiences). | Webinar platform |
| Competitor | Fourth and fifth competitor comparison pages: "SGA vs. Aspen Dental" and "SGA vs. SALT Dental Partners." | Website |
| YouTube | Midpoint review: 10-12 videos published. Analyze view counts, watch time, description link clicks, and email captures from the Implant Cost Guide. Adjust content mix based on what is performing. | YouTube |
| Tests | PP-2 (tier presentation), PP-3 (value equation display), EM-4 (social proof email format) begin. Phase 2 checkpoint. | Pricing page, email |
| Content | Blog posts #13-14. LinkedIn continues. Instagram Reels scaling. | All |
| Email | Story email ISM sequence (Invisible Selling Machine) review: engagement rates on broadcast emails, revenue per subscriber metrics if applicable. | Email |

### Week 11: Sustained Momentum + Third Competitor Wave

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Competitor | Sixth and seventh competitor comparison pages: "SGA Alternative for Dentists" (catch-all comparison) and "Best DSO Partnerships 2026" (roundup/authority page). Full competitor page SEO suite now complete. | Website |
| Tests | LP-8 (video testimonial compilation), LP-4 (problem section length) begin. These require video production (started in Week 7-8). | Landing page |
| Case Studies | First 2 formal case studies published using case study templates. Dr. Cavendish story (Gen4 transformation) and a MODIS partner story. Distributed to BD team, posted on website, promoted via LinkedIn. | Website, LinkedIn, sales |
| Content | Blog posts #15-16. YouTube maintains cadence. Local practice content kit #3 distributed. | All |
| Ads | Third creative rotation. Budget reallocation based on 8 weeks of data: shift spend to highest-performing channels and creative concepts. | All paid |

### Week 12: 90-Day Review + Phase 3 Handoff

| Category | Assets Deployed | Channels |
|----------|----------------|----------|
| Review | Full 90-day performance review. Compile all metrics against launch targets. Calculate A/B test cumulative lift. Identify top 3 performing channels, top 3 underperformers, and root causes. | All |
| Tests | WB-2 (registration bullets), WB-3 (pitch timing ongoing), EM-5 (sequence length) begin. Phase 3 tests start but extend beyond 90-day window. | Webinar, email |
| Strategy | Build Month 4-6 plan based on 90-day learnings. Adjust budget allocation, content strategy, and funnel architecture based on real performance data. | All |
| Content | Blog posts #17-18. Total SEO content: 18+ blog posts, 1 pillar page, 7 competitor pages. Foundation for sustained organic traffic growth. | Website |
| Social | Community growth assessment: LinkedIn combined followers, YouTube subscribers, Facebook/Instagram followers, TikTok followers. Measure against 90-day targets. | All social |
| Pipeline | Pipeline review with BD team: total leads generated, MQL > SQL conversion rate, discovery calls booked, site visits, LOIs in progress. Calculate marketing-attributed pipeline value. | CRM |

---

## 6. Budget Allocation

### Percentage-Based Channel Distribution

Budget percentages are designed for the first 90 days. Reallocation should happen monthly based on channel performance data.

| Channel | % of Total Budget | Rationale |
|---------|-------------------|-----------|
| **Google Ads (Search + LSAs)** | 30% | Highest-intent channel. Captures existing demand for "sell dental practice," "dental practice valuation," "dental implants [city]." Fastest path to qualified leads. |
| **LinkedIn Ads (Thought Leader + Sponsored Jobs + InMail)** | 25% | Primary B2B channel for practice acquisition and specialist recruitment. Thought Leader Ads for awareness, Lead Gen Forms for conversion, Sponsored Jobs for recruitment. |
| **Meta Ads (Facebook + Instagram)** | 20% | Patient acquisition (implant vertical) and practice owner awareness. Before/after transformation content performs exceptionally on Meta. Also used for webinar registration and scorecard promotion. |
| **Content Production** | 10% | YouTube video production, blog writing, graphic design, social content creation. The content pyramid produces 25-35 pieces per production day -- high ROI on production investment. |
| **Email Platform + Automation** | 5% | Email platform subscription, deliverability tools, A/B testing tools. Small cost, massive leverage -- email sequences do the selling 24/7. |
| **SEO + Technical** | 5% | Technical SEO fixes, schema markup implementation, link building outreach, content optimization tools. Investment in organic compounds over time. |
| **Testing + Tools** | 3% | Heatmapping (Hotjar/Clarity), A/B testing platform, chatbot platform, quiz/scorecard tool, analytics tools. |
| **PR + Events** | 2% | Trade publication outreach, byline article placement, conference sponsorship deposits for upcoming events. Seed investment for 6-12 month authority building. |

### Budget Phase Adjustments

| Phase | Adjustment | Reason |
|-------|------------|--------|
| Weeks 1-2 (Pre-launch) | Spend at 25% of monthly rate | Infrastructure setup, domain warming. No paid traffic yet. |
| Week 3 (Launch) | Ramp to 75% of monthly rate | Controlled ignition. Ads start at 50% budget, scale to 75% by Friday. |
| Weeks 4-6 | Full monthly budget | Full activation. All channels running. Testing begins. |
| Weeks 7-9 | Full monthly budget + 10% test reserve | Webinar promotion and scorecard launch require additional paid support. Draw from test reserve. |
| Weeks 10-12 | Reallocated monthly budget | Shift 10-20% from underperforming channels to top performers based on 8 weeks of data. |

### Channel-Specific CPL Targets

| Channel | Target CPL (PA Funnel) | Target CPL (PT Funnel) | Target CPL (DR Funnel) |
|---------|----------------------|----------------------|----------------------|
| Google Search | < $350 (assessment complete) | < $75 (general lead), < $120 (implant lead) | N/A |
| Google LSAs | N/A | < $75 (booked appointment) | N/A |
| LinkedIn | < $150 (Lead Gen Form) | N/A | < $80 (qualified applicant) |
| Meta (Facebook/Instagram) | < $200 (assessment complete) | < $65 (general lead), < $90 (implant lead) | < $60 (career guide download) |

---

## 7. V2 Asset Launch Timing

V2 assets add depth and breadth to the core funnel. They deploy on a staggered schedule to prevent overwhelming the team, allow V1 assets to establish baseline performance, and sustain marketing momentum through Weeks 4-12.

### Quiz/Scorecard Funnel Launch (Week 6)

| Step | Timing | Detail |
|------|--------|--------|
| Final QA and scoring validation | Week 5 | Test all 15 questions, scoring logic across all 3 tiers (Strong Position 80-100, Growth Opportunity 50-79, Foundation Building 0-49), results page rendering, and lead capture form. |
| Follow-up email sequences loaded | Week 5 | Tier-specific follow-up emails (from scorecard follow-up emails asset) loaded into email platform. Automation triggers configured based on tier assignment. |
| Scorecard goes live | Week 6, Monday | Published as standalone assessment at sgadental.com/partnership-assessment. Linked from main landing page as secondary CTA. |
| Promotion plan activated | Week 6, Wednesday | Per the scorecard promotion plan asset: LinkedIn organic posts from leaders ("We built a tool that helps practice owners see where they stand"), Facebook ads with micro-commitment CTA ("See how your practice compares"), Google Search ads with "dental practice valuation" keywords pointing to scorecard. |
| Integration with PA funnel | Week 6 | Scorecard completions feed into PA-Indoctrination sequence with tier-specific messaging. Score > 80 triggers BD priority alert for outreach within 48 hours. |
| First performance review | Week 8 | Analyze: completion rate (target > 70%), lead capture rate (target > 60% of completers), tier distribution, and downstream discovery call booking rate by tier. |

### Webinar Launch Sequence (Weeks 7-12)

| Step | Timing | Detail |
|------|--------|--------|
| Registration page built | Week 7 | Webinar strategy document provides 2 title options for A/B testing (WB-1). Build registration page with 7-bullet content description, speaker bios, and Lead Gen Form integration. |
| Pre-webinar email sequence built | Week 7 | 5-email sequence: Confirmation (Day 0), "What to prepare" (Day -7), "Tomorrow" reminder (Day -1), "Starting in 1 hour" (Day 0, -60 min), Post-webinar replay (Day 0, +2 hours). |
| Registration ads launched | Week 7, Thursday | LinkedIn ads targeting practice owners and specialists. Meta ads targeting warm audiences (website visitors, email subscribers). Target: 75+ registrations for first webinar. |
| First webinar hosted | Week 8, Thursday at 12 PM ET | 60 minutes teaching (7-question framework) + 15 minutes Q&A + 10 minutes pitch (per webinar pitch script asset). Jensen + Ellingson present. |
| Post-webinar replay sequence | Week 8-9 | 4-email post-webinar sequence: replay link (Day 0), social proof from attendees (Day 2), urgency on discovery call slots (Day 5), final CTA (Day 7). |
| Webinar content repurposing | Week 9 | Edit recording into 5-7 short clips for LinkedIn and YouTube. Best Q&A moments become standalone LinkedIn posts. Webinar slides become downloadable PDF lead magnet. |
| Second webinar hosted | Week 10 or 12 | Apply WB-1 test learnings (alternate title). Expand audience targeting to include cold traffic. Target: 100+ registrations. |
| Evergreen webinar consideration | Week 12 | Evaluate whether to convert the best-performing webinar into an on-demand evergreen asset (automated webinar funnel). Decision based on attendance-to-conversion data from live events. |

### YouTube Content Publishing Schedule (Weeks 5-12)

| Week | Videos Published | Content Focus |
|------|-----------------|---------------|
| Week 5 | 2 long-form + 4 Shorts | Launch week: Patient transformation reveal, dental education explainer. Establish publishing cadence. |
| Week 6 | 2 long-form + 4 Shorts | Transformation reveal, behind-the-scenes office tour (serves PT + DR audiences). |
| Week 7 | 2 long-form + 5 Shorts | Education ("How much do dental implants cost in 2026"), live Q&A session #1. |
| Week 8 | 2 long-form + 5 Shorts | Transformation reveal, dentist-to-dentist case presentation (serves SP + DR). |
| Week 9 | 2 long-form + 5 Shorts | Education, behind-the-scenes technology showcase (CEREC, guided implant surgery). |
| Week 10 | 2 long-form + 5 Shorts | Live Q&A session #2, transformation reveal. Midpoint analytics review. |
| Week 11 | 2 long-form + 5 Shorts | "Smiles in the Wild" podcast-style episode featuring MODIS partner, dental myth-busting. |
| Week 12 | 2 long-form + 5 Shorts | Transformation reveal, year-in-review or practice spotlight content. 90-day YouTube performance review. |

**Cross-platform distribution:** Every YouTube Short is re-exported without watermark and cross-posted to TikTok (same day), Instagram Reels (same day), and Facebook Reels (next day). One production day yields 25-35 platform-specific pieces.

**Conversion optimization:** Every video description includes YourSmileDestination.com in line 1, Implant Cost Guide PDF link in line 2, and "Explore SGA Partnership" link in line 3. Pinned comments with consultation booking link on every transformation video.

### Chatbot Deployment Timeline (Week 7)

| Step | Timing | Detail |
|------|--------|--------|
| Chatbot flow QA | Week 6 | Test all routing paths from the chatbot flow asset. Verify: practice owner path leads to assessment/discovery call booking, patient path leads to nearest practice finder/consultation booking, career path leads to open positions/guide download. |
| Deploy on main landing page | Week 7, Monday | Bottom-right widget. Trigger: 30-second delay on page or scroll to 50%. Welcome message: "Have a question about dental partnership? I can help." |
| Deploy on patient landing page | Week 7, Monday | Bottom-right widget. Trigger: 30-second delay. Welcome message: "Curious about dental implants? Ask me anything." |
| Deploy on associate landing page | Week 7, Wednesday | Bottom-right widget. Welcome message: "Looking for your next career move in dentistry? Let me help." |
| Monitor and refine | Weeks 7-12 | Track: engagement rate (% of visitors who interact), routing accuracy (% correctly classified), handoff rate (% escalated to human), and conversion attribution (leads/bookings sourced from chatbot). |
| Add to pricing page | Week 9 | Deploy chatbot on pricing page with specialized flow for deal-structure questions. Route to BD team for sensitive financial discussions. |

### Competitor Comparison Page Publishing Cadence (Weeks 7-12)

Published in order of search volume and competitive threat.

| Week | Page | Target Keyword | Priority |
|------|------|---------------|----------|
| Week 7 | SGA vs. Heartland Dental | "heartland dental reviews," "heartland dental partnership" | Highest -- largest competitor, most search volume |
| Week 9 | SGA vs. MB2 Dental | "MB2 dental reviews," "MB2 dental partnership" | High -- direct positioning competitor ("DPO" framing) |
| Week 9 | SGA vs. Pacific Dental Services | "Pacific Dental Services careers," "PDS partnership" | High -- owner-dentist model competitor |
| Week 10 | SGA vs. Aspen Dental | "Aspen Dental franchise," "Aspen Dental reviews" | Medium -- consumer brand competitor, different model |
| Week 10 | SGA vs. SALT Dental Partners | "SALT Dental Partners," "specialty DSO" | Medium -- specialty competitor, relevant for SP funnel |
| Week 11 | SGA Alternative for Dentists | "best DSO to work for," "DSO alternative" | Medium -- catch-all comparison for broad search intent |
| Week 11 | Best DSO Partnerships 2026 | "best DSO partnerships," "top DSOs for dentists" | Medium -- authority/roundup page targeting informational queries |

**SEO integration:** Each competitor page uses the on-page specs and schema markup from the SEO assets. Internal links from blog posts and the pillar page point to relevant competitor pages. Google Search ads target competitor brand keywords, directing to the comparison pages.

### Story Email Sequence Activation Triggers (Week 8+)

| Sequence | Activation Trigger | Audience | Purpose |
|----------|-------------------|----------|---------|
| Soap Opera Sequence (5 emails) | Lead completes PA-Indoctrination (5 emails) + has NOT booked discovery call + has opened 3+ indoctrination emails | Practice owners (PA funnel) | Emotional bridge between indoctrination and engagement. Uses narrative storytelling (partner transformation arcs) to build emotional connection before the rational engagement sequence begins. |
| ISM Broadcast Sequence | Lead is on the active PA or SP email list + has NOT converted to customer | Practice owners, Specialists | Weekly broadcast emails following the Invisible Selling Machine methodology: Lifestyle, Opportunity, Authority, Proof rotation. Keeps SGA top of mind during the long 6-18 month decision cycle without sales pressure. |
| Post-Conversation Story Drip | Lead completed a discovery call + did NOT advance to site visit | Practice owners (PA funnel) | Biweekly narrative emails featuring different partner stories. Each email profiles a partner who was in a similar situation to the prospect. Maintains relationship during the extended decision-making period. |

**Activation protocol:** Story emails do not activate until the core sequences (indoctrination + engagement) have been running for 4+ weeks and have established baseline performance. This ensures the foundation is solid before adding complexity.

---

## 8. Risk Mitigation

### Technical Risks

| Risk | Probability | Impact | Mitigation | Contingency |
|------|-------------|--------|------------|-------------|
| Email deliverability issues (spam filtering, low inbox placement) | Medium | High | Domain warming protocol in Week 2. SPF/DKIM/DMARC authentication. Gradual volume increase. Clean list hygiene. | If bounce rate exceeds 2% or spam complaints exceed 0.1%, immediately pause all sends. Investigate domain reputation. Consider switching sending subdomain. |
| Tracking pixel failures (GTM misconfiguration, conversion events not firing) | Medium | High | End-to-end funnel tests in Week 2 (Day 9-10). Cross-reference GA4, Meta, LinkedIn, and CRM data before launch. | If discovered post-launch, pause paid spend on affected channels until tracking is verified. Use manual UTM-based tracking as a fallback. |
| Landing page downtime or slow load times | Low | High | Staging environment tested under load. CDN configured. Mobile performance verified (<3 second load time). | Maintain a static backup page with core messaging and a contact form. Redirect traffic to backup if primary page goes down during critical periods (launch week, webinar day). |
| CRM integration breaks (leads not routing to BD team) | Medium | Medium | Integration tested in Week 1. Backup notification: email alert to BD team on every assessment completion, independent of CRM routing. | Manual lead monitoring via email platform dashboard. Daily check of assessment completions vs. CRM entries. |

### Market Risks

| Risk | Probability | Impact | Mitigation | Contingency |
|------|-------------|--------|------------|-------------|
| Competitor launches counter-campaign during SGA launch window | Medium | Medium | Monitor competitor activity weekly. Google Alerts for competitor brand names + "partnership" / "new" / "launch." | If a competitor launches a major campaign, accelerate the competitor comparison page for that specific competitor. Increase ad spend on brand-defense keywords. |
| Negative press or social media criticism about DSOs in general | Medium | Medium | The positioning already addresses DSO skepticism head-on ("7 out of 10 regret"). The messaging is built to withstand anti-DSO sentiment because it acknowledges and channels it. | If specific criticism targets SGA, respond with named partner testimonials and verifiable data points. Do not get defensive. Use the Sage archetype: "Here are the facts. Here are the names. Call them yourself." |
| Practice owner market shifts (recession fears, practice value declines) | Low | High | Monitor dental market publications (Dental Economics, ADA Health Policy Institute reports). Adjust messaging to emphasize stability and risk reduction if market anxiety increases. | Shift messaging from "growth opportunity" to "protection and stability." Emphasize Thurston Group's 38-year track record through multiple economic cycles. Increase emphasis on the equity diversification benefit. |

### Operational Risks

| Risk | Probability | Impact | Mitigation | Contingency |
|------|-------------|--------|------------|-------------|
| Content production bottleneck (Dr. Olson unavailable for video, leadership too busy for LinkedIn) | High | Medium | Pre-record content buffer: 2 weeks of YouTube content recorded in advance. LinkedIn ghostwriting support for all 4 leaders. Pre-approved content library. | If Dr. Olson is unavailable for extended period, pivot to repurposing existing 200+ YouTube video library. Extract new Shorts and Reels from archival content. Feature other MODIS partners in new videos. |
| BD team overwhelmed by lead volume (unable to follow up within 48 hours) | Medium | High | Set realistic lead volume expectations. Establish priority triage: Score > 80 gets same-day outreach, Score 50-79 gets 48-hour outreach, Score < 50 enters automated nurture only. | If lead volume exceeds BD capacity, temporarily pause paid acquisition spend on the PA funnel. Focus spend on PT (patient) funnel where practice-level teams handle follow-up. Hire or reassign BD resources. |
| Brand architecture confusion (Gen4/MODIS/SGA naming creates prospect confusion) | Medium | Medium | All marketing assets use "SGA Dental Partners" as the primary brand. Gen4 and MODIS appear only as descriptive context ("formerly Gen4 Dental Partners" or "the MODIS implantology vertical"). Consistent brand guidelines applied across all touchpoints. | If confusion persists, add a "Who Is SGA?" FAQ section to every landing page. Create a short "Our Story" video explaining the consolidation narrative. |
| A/B test results are inconclusive due to low traffic | Medium | Low | Traffic volume feasibility checked against minimum sample sizes in the A/B test playbook (Appendix C). Tests on low-traffic pages (pricing, webinar registration) run for longer durations. | Lower significance threshold to 90% for low-traffic tests and treat results as directional. Consider sequential testing (4 weeks control, 4 weeks variant) instead of simultaneous split testing. |

### Legal and Compliance Risks

| Risk | Probability | Impact | Mitigation | Contingency |
|------|-------------|--------|------------|-------------|
| Partner testimonial usage without proper consent | Medium | High | Confirm written approval from Dr. Godat, Dr. Cavendish, Dr. Olson, Dr. Pikos, and all other named partners before any testimonial appears in marketing materials. | Maintain a "testimonial consent" tracker. Any testimonial without documented consent is removed immediately and replaced with an approved alternative. |
| Competitor comparison pages trigger legal concerns | Low | Medium | All competitor claims factually sourced from public information (websites, press releases, SEC filings). No subjective characterizations. Review language with legal before publication. | If a competitor sends a cease-and-desist, immediately review the specific claim in question. Remove or modify any contested statement. Maintain the page with factual, defensible language only. |
| Healthcare advertising regulations (patient testimonials, before/after images) | Medium | Medium | All patient testimonials include appropriate disclaimers ("individual results may vary"). Before/after images comply with Meta and Google health advertising policies. HIPAA consent for any patient-identifiable content. | If a platform rejects ad creative, immediately swap to approved backup creative. Maintain a library of 3-4 approved creative variants per ad set. |
| Financial claims accuracy (Thurston $4B+ returned, 85% retention rate) | Low | High | Verify all financial and statistical claims with Thurston Group legal and SGA finance team before launch. Document sources for every number used in marketing. | If any claim cannot be verified, replace with a softer statement ("billions returned to partners over 38 years") until exact figures are confirmed. |

---

## 9. Team Responsibilities

### Role Definitions

Each role below is a function, not a headcount. In a lean team, one person may hold multiple roles. In a scaled team, each role may have dedicated staff.

| Role | Primary Responsibilities | Key Deliverables |
|------|------------------------|------------------|
| **Marketing Lead** | Overall launch ownership. Strategy, timeline, budget, cross-functional coordination. Weekly reporting to leadership. Go/no-go decisions. | Weekly performance snapshots, monthly funnel reviews, budget reallocation recommendations, A/B test result documentation. |
| **Content Lead** | Content calendar execution. Blog writing, social content creation, LinkedIn ghostwriting, content kit production. Quality control across all written and visual assets. | 2 blog posts/week, LinkedIn posts for 4 leaders, weekly content kits for local practices, social content scheduling. |
| **Paid Media Manager** | Campaign setup, optimization, and scaling across Google, LinkedIn, and Meta. Creative testing. Budget pacing. Audience management. | Campaign performance reports, creative rotation schedules, audience expansion recommendations, CPL tracking by channel. |
| **Marketing Ops / Automation** | Email platform management, CRM integration, tracking infrastructure, A/B test implementation, chatbot configuration, scorecard/quiz tool management. | Email sequence deployment, automation trigger configuration, tracking QA, test setup and monitoring, lead routing verification. |
| **Video Production** | YouTube content production for Dr. Olson's channel. Filming, editing, Short/Reel extraction, cross-platform distribution. Thumbnail design. | 2 long-form videos/week, 4-5 Shorts/week, cross-platform exports, video testimonial compilation for landing pages. |
| **SEO Lead** | Keyword strategy execution, on-page optimization, technical SEO, link building, competitor page SEO, search ranking monitoring. | Blog post optimization, pillar page maintenance, competitor page publishing, schema markup, monthly ranking reports. |
| **Community Manager** | Social media engagement across all platforms. Comment replies, DM management, review monitoring, user-generated content curation. | Daily engagement (4-hour response time weekdays), weekly engagement reports, UGC curation for content team. |
| **BD Team Lead** | Lead follow-up, discovery call hosting, partner relationship management. Feedback loop to marketing on lead quality. | Lead follow-up within triage SLA (same-day for score > 80, 48 hours for score 50-79), discovery call notes in CRM, monthly lead quality feedback to Marketing Lead. |
| **Practice Ops Liaison** | Communication bridge between central marketing and local practices. Content kit distribution, office manager briefing, local page management support. | Monthly content kit distribution, office manager training, local practice feedback collection. |
| **PR / Thought Leadership Coordinator** | Trade publication relationships, byline article placement, podcast booking for leadership, conference planning. | 1-2 podcast appearances/month, quarterly byline articles, conference booth coordination. |

### RACI Matrix for Key Activities

| Activity | Marketing Lead | Content Lead | Paid Media | Marketing Ops | BD Team | Practice Ops |
|----------|:---:|:---:|:---:|:---:|:---:|:---:|
| Launch timeline and milestones | A | I | I | I | I | I |
| Landing page copy and design | A | R | C | C | I | I |
| Email sequence deployment | A | C | I | R | I | I |
| Paid campaign setup and optimization | A | C | R | C | I | I |
| A/B test design and analysis | R | C | C | R | I | I |
| Lead follow-up and discovery calls | I | I | I | C | R/A | I |
| Content calendar execution | A | R | C | I | I | C |
| Local practice content kits | C | R | I | I | I | R/A |
| YouTube production | C | C | I | I | I | I |
| Weekly performance reporting | R/A | C | C | C | C | I |
| Budget management | R/A | I | C | I | I | I |
| Webinar execution | A | C | C | R | C | I |
| Scorecard/quiz deployment | A | C | I | R | C | I |
| Chatbot deployment and monitoring | A | C | I | R | C | C |

**Legend:** R = Responsible (does the work), A = Accountable (owns the outcome), C = Consulted, I = Informed

### Weekly Meeting Cadence

| Meeting | Attendees | Frequency | Duration | Purpose |
|---------|-----------|-----------|----------|---------|
| Launch standup | All marketing roles | Daily (Weeks 1-3), then 3x/week | 15 min | Quick status updates, blockers, priorities for the day |
| Performance review | Marketing Lead, Paid Media, Marketing Ops | Weekly (Thursday) | 30 min | Review metrics against targets, identify issues, adjust tactics |
| Content planning | Content Lead, Video Production, SEO Lead, Marketing Lead | Weekly (Monday) | 30 min | Plan content for the week, review previous week's performance, coordinate cross-platform publishing |
| BD-Marketing sync | Marketing Lead, BD Team Lead | Biweekly (Tuesday) | 30 min | Lead quality feedback, pipeline review, messaging alignment, sales enablement needs |
| Leadership update | Marketing Lead, SGA leadership | Monthly | 30 min | High-level metrics, budget review, strategic recommendations, escalations |

---

## Appendix A: Asset Deployment Checklist

Complete inventory of all assets and their launch timing.

### V1 Assets (Launch Week -- Week 3)

| Asset | File | Status | Deploy Date |
|-------|------|--------|-------------|
| Main landing page | `assets/landing_pages/main_landing_page.md` | Ready | Week 3, Day 1 |
| Associate landing page | `assets/landing_pages/associate_landing_page.md` | Ready | Week 3, Day 1 |
| Patient landing page | `assets/landing_pages/patient_landing_page.md` | Ready | Week 3, Day 1 |
| Pricing page | `assets/landing_pages/pricing_page.md` | Ready | Week 3, Day 1 |
| Competitor comparison landing page | `assets/landing_pages/competitor_comparison.md` | Ready | Week 3, Day 1 |
| Welcome email sequence (PA) | `assets/emails/welcome_sequence.md` | Ready | Week 3, Day 1 |
| Nurture email sequence (PA) | `assets/emails/nurture_sequence.md` | Ready | Week 3, Day 1 |
| Sales email sequence (PA) | `assets/emails/sales_sequence.md` | Ready | Week 3, Day 1 |
| Associate email sequence (DR) | `assets/emails/associate_sequence.md` | Ready | Week 3, Day 1 |
| Re-engagement email sequence | `assets/emails/reengagement_sequence.md` | Ready | Week 3, Day 1 |
| Practice Partnership Assessment | `assets/lead_magnet/practice_partnership_assessment.md` | Ready | Week 3, Day 1 |
| Assessment opt-in page | `assets/lead_magnet/opt_in_landing_page.md` | Ready | Week 3, Day 1 |
| Assessment delivery email | `assets/lead_magnet/delivery_email.md` | Ready | Week 3, Day 1 |
| Assessment thank-you page | `assets/lead_magnet/thank_you_page.md` | Ready | Week 3, Day 1 |
| Facebook ads (all variations) | `assets/ads/facebook_ads.md` | Ready | Week 3, Day 4 |
| Google ads (all variations) | `assets/ads/google_ads.md` | Ready | Week 3, Day 2 |
| LinkedIn ads (all variations) | `assets/ads/linkedin_ads.md` | Ready | Week 3, Day 3 |
| Instagram ads (all variations) | `assets/ads/instagram_ads.md` | Ready | Week 3, Day 4 |
| Facebook/Instagram content calendar | `assets/social_content/facebook_instagram_calendar.md` | Ready | Week 3, Day 1 |
| LinkedIn content calendar | `assets/social_content/linkedin_calendar.md` | Ready | Week 3, Day 1 |
| SEO keyword map | `assets/seo/keyword_map.md` | Ready | Week 3, Day 1 |
| SEO blog outlines (first 4) | `assets/seo/blog_outlines.md` | Ready | Week 3, Day 1 |
| SEO pillar page | `assets/seo/pillar_page_brief.md` | Ready | Week 3, Day 1 |
| SEO schema markup | `assets/seo/schema_markup.md` | Ready | Week 3, Day 1 |
| SEO on-page specs | `assets/seo/on_page_specs.md` | Ready | Week 3, Day 1 |
| SEO internal linking plan | `assets/seo/internal_linking_plan.md` | Ready | Week 3, Day 1 |
| Battle cards | `assets/sales_enablement/battle_cards.md` | Ready | Week 1, Day 5 |
| Elevator pitches | `assets/sales_enablement/elevator_pitches.md` | Ready | Week 1, Day 5 |
| FAQ document | `assets/sales_enablement/faq_document.md` | Ready | Week 1, Day 5 |
| One pager | `assets/sales_enablement/one_pager.md` | Ready | Week 1, Day 5 |
| Case study template | `assets/sales_enablement/case_study_template.md` | Ready | Week 1, Day 5 |
| Social proof strategy | `assets/case_studies/social_proof_strategy.md` | Ready | Week 1 |
| Case study templates | `assets/case_studies/case_study_templates.md` | Ready | Week 1 |
| Interview question bank | `assets/case_studies/interview_question_bank.md` | Ready | Week 1 |
| Avatar case studies | `assets/case_studies/avatar_case_studies.md` | Ready | Week 3 |
| Content repurposing matrix | `assets/social_content/repurposing_matrix.md` | Ready | Week 2 |
| Long-form content outlines | `assets/social_content/long_form_outlines.md` | Ready | Week 2 |

### V2 Assets (Staggered Deployment -- Weeks 5-12)

| Asset | File | Deploy Week | Dependencies |
|-------|------|-------------|--------------|
| YouTube content calendar | `assets/social_content/youtube_calendar.md` | Week 5 | Video production schedule confirmed, Dr. Olson availability |
| YouTube ad variations | `assets/ads/youtube_ads.md` | Week 6 | YouTube channel optimized, first 4 videos published |
| YouTube scripts | `assets/video_scripts/youtube_scripts.md` | Week 5 | Production schedule set |
| Partnership Readiness Scorecard | `assets/quiz_scorecard/partnership_readiness_scorecard.md` | Week 6 | Scoring logic validated, results pages built |
| Scorecard results categories | `assets/quiz_scorecard/results_categories.md` | Week 6 | Scorecard QA complete |
| Scorecard follow-up emails | `assets/quiz_scorecard/followup_emails.md` | Week 6 | Email automation configured |
| Scorecard promotion plan | `assets/quiz_scorecard/promotion_plan.md` | Week 6 | Scorecard live, ad accounts configured |
| Chatbot flow | `assets/chatbot/chatbot_flow.md` | Week 7 | Chatbot platform configured, routing tested |
| Webinar strategy | `assets/webinar/webinar_strategy.md` | Week 7 | Registration page built, email sequences loaded |
| Webinar pitch script | `assets/webinar/webinar_pitch_script.md` | Week 8 | Webinar rehearsal completed |
| VSL script | `assets/vsl/vsl_script.md` | Week 9 | VSL production scheduled (video recording + editing) |
| SGA vs. Heartland Dental | `assets/competitor_pages/sga_vs_heartland.md` | Week 7 | Legal review of competitor claims |
| SGA vs. MB2 Dental | `assets/competitor_pages/sga_vs_mb2.md` | Week 9 | Legal review |
| SGA vs. Pacific Dental Services | `assets/competitor_pages/sga_vs_pds.md` | Week 9 | Legal review |
| SGA vs. Aspen Dental | `assets/competitor_pages/sga_vs_aspen.md` | Week 10 | Legal review |
| SGA vs. SALT Dental Partners | `assets/competitor_pages/sga_vs_salt.md` | Week 10 | Legal review |
| SGA Alternative for Dentists | `assets/competitor_pages/sga_alternative.md` | Week 11 | Legal review |
| Best DSO Partnerships 2026 | `assets/competitor_pages/best_dso_partnerships_2026.md` | Week 11 | Legal review |
| Soap opera email sequence | `assets/story_emails/soap_opera_sequence.md` | Week 8 | PA-Indoctrination has been running for 4+ weeks, baseline performance established |
| ISM broadcast sequence | `assets/story_emails/ism_sequence.md` | Week 8 | Active subscriber list of 200+ in PA/SP funnels |

---

## Appendix B: Pre-Launch Verification Checklist

Complete this checklist before the go/no-go decision on Day 10 (end of Week 2).

### Tracking and Analytics

- [ ] GTM container deployed on all pages
- [ ] GA4 property configured with all conversion events
- [ ] Meta Pixel installed and verified (Pixel Helper green)
- [ ] LinkedIn Insight Tag installed and verified
- [ ] Google Ads conversion tracking configured
- [ ] Heatmapping tool deployed on landing pages
- [ ] UTM convention documented and shared with all team members
- [ ] Cross-platform conversion data reconciled (GA4 vs. Meta vs. LinkedIn vs. CRM within 5% margin)

### Email Infrastructure

- [ ] Sending domain authenticated (SPF, DKIM, DMARC)
- [ ] Domain warming completed (7+ days, gradually increasing volume)
- [ ] All email sequences loaded and QA'd (links, merge tags, conditional blocks)
- [ ] Tag architecture implemented
- [ ] Automation triggers configured and tested
- [ ] Re-engagement triggers set (cold_60d, cold_90d)
- [ ] Email rendering verified across Gmail, Outlook, Apple Mail (desktop + mobile)

### CRM and Sales

- [ ] Lead routing rules configured
- [ ] BD team dashboard built and accessible
- [ ] Call booking system configured (Calendly/HubSpot meetings)
- [ ] MQL/SQL definitions documented and communicated to BD team
- [ ] Sales enablement kit distributed to all BD team members

### Landing Pages

- [ ] Main landing page QA'd (links, forms, mobile, speed)
- [ ] Associate landing page QA'd
- [ ] Patient landing page QA'd
- [ ] Pricing page QA'd
- [ ] Competitor comparison page QA'd
- [ ] All pages load in under 3 seconds on mobile
- [ ] Assessment/scorecard embed loads correctly
- [ ] Form submissions route to correct CRM pipeline

### Paid Media

- [ ] Ad accounts created and configured
- [ ] All creative uploaded in draft/paused state
- [ ] Audience targeting configured (interest, behavior, lookalike, retargeting)
- [ ] Conversion tracking verified at the ad platform level
- [ ] Daily budget caps set to prevent overspend

### Content

- [ ] LinkedIn posts scheduled for Weeks 3-4 (all 4 leaders)
- [ ] Facebook/Instagram posts scheduled for Weeks 3-4
- [ ] First 4 blog posts written and staged
- [ ] SEO pillar page staged
- [ ] Local practice content kit #1 prepared

### Internal Alignment

- [ ] Leadership aligned on launch timeline and success metrics
- [ ] BD team briefed on messaging and lead handling
- [ ] Practice managers informed about marketing launch
- [ ] Thought leadership leaders (Jensen, Ellingson, Olson, Wong) confirmed for LinkedIn posting

### End-to-End Funnel Tests

- [ ] PA funnel: ad click > landing page > assessment > email > discovery call booking -- all steps verified
- [ ] PT funnel: ad click > landing page > Smile Assessment > email > consultation booking -- all steps verified
- [ ] DR funnel: career page > guide download > email sequence > application link -- all steps verified

---

*Document total: ~7,800 words*
*Covers: 12-week deployment calendar, 55+ assets, 4 funnels, 7 channels, 27 A/B tests, 5 target avatars*
*Source documents: Product Brief, Funnel Architecture, Positioning, Social Growth Strategy, A/B Test Playbook*
