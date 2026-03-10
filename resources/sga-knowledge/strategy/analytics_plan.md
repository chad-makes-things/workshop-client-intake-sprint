# SGA Dental Partners -- Analytics and Measurement Plan

**Prepared:** March 2026
**Scope:** Full-funnel tracking across 4 funnels, 5 ad platforms, 7 email sequences, 5 social calendars, and all V2 conversion assets
**Audiences:** Practice owners (Dr. Hendricks), associate dentists (Dr. Nair), implant patients (Linda Dawson), specialists (Dr. Whitfield), office managers (Tamara Reeves)
**Integrates with:** Funnel Architecture (Phase 5B), A/B Test Playbook (Phase 5D)

---

## 1. UTM Naming Convention

### Structure

Every tracked link follows this format:

```
?utm_source={source}&utm_medium={medium}&utm_campaign={campaign}&utm_content={content}&utm_term={term}
```

All UTM values are lowercase, use hyphens as separators (never underscores or spaces), and follow the taxonomy below exactly. Inconsistent naming destroys attribution data. One person owns the UTM registry and approves new values.

### Parameter Definitions

| Parameter | Purpose | Format | Examples |
|-----------|---------|--------|----------|
| `utm_source` | Where the traffic comes from (platform/site) | Platform name, lowercase | `google`, `facebook`, `instagram`, `linkedin`, `youtube`, `email`, `podcast`, `dentalpost`, `indeed`, `direct-mail`, `conference`, `referral`, `seo` |
| `utm_medium` | How the traffic arrives (channel type) | Channel category, lowercase | `cpc`, `cpm`, `social-organic`, `social-paid`, `email`, `referral`, `display`, `video`, `lsa`, `content`, `podcast`, `event` |
| `utm_campaign` | Which campaign this belongs to | `{funnel}-{campaign-name}` | `pa-assessment`, `dr-career-guide`, `pt-implant-smile`, `sp-specialist-guide`, `pt-general-lsa` |
| `utm_content` | Which specific asset/variant within a campaign | `{asset-type}-{descriptor}` | `ad-fear-stat-headline`, `email-day3-nurse-story`, `lp-hero-cta`, `video-testimonial-scott`, `blog-dso-comparison` |
| `utm_term` | Keyword or targeting criteria | Keywords or audience segment | `sell-dental-practice`, `dental-implants-cost`, `dso-partnership`, `associate-dentist-jobs` |

### Funnel Prefixes

Every campaign name starts with its funnel prefix to enable instant segmentation:

| Prefix | Funnel | Audience |
|--------|--------|----------|
| `pa-` | Practice Acquisition | Dr. Mark Hendricks (practice owners) |
| `dr-` | Dentist Recruitment | Dr. Priya Nair (associate dentists) |
| `pt-` | Patient Acquisition | Linda Dawson (implant/general patients) |
| `sp-` | Specialist Partnership | Dr. James Whitfield (periodontists, oral surgeons) |
| `brand-` | Brand awareness / cross-funnel | All audiences |
| `re-` | Re-engagement | Lapsed contacts across all funnels |

### Source-Medium Combinations (Approved List)

| Source | Medium | Use Case |
|--------|--------|----------|
| `google` | `cpc` | Google Search Ads (RSAs) |
| `google` | `lsa` | Google Local Services Ads |
| `google` | `display` | Google Display Network |
| `google` | `video` | YouTube Ads (paid) |
| `facebook` | `social-paid` | Facebook Ads |
| `facebook` | `social-organic` | Organic Facebook posts |
| `instagram` | `social-paid` | Instagram Ads |
| `instagram` | `social-organic` | Organic Instagram posts |
| `linkedin` | `social-paid` | LinkedIn Sponsored Content / Lead Gen Forms |
| `linkedin` | `social-organic` | Organic LinkedIn posts |
| `youtube` | `social-organic` | Organic YouTube videos (Dr. Olson channel, SGA channel) |
| `email` | `email` | All email sequences (ESP-generated) |
| `podcast` | `podcast` | MODIS Community Conversations, guest appearances |
| `dentalpost` | `referral` | DentalPost job listings |
| `indeed` | `referral` | Indeed job listings |
| `conference` | `event` | Conference booth, hosted dinners, trade shows |
| `direct-mail` | `direct` | Physical mailers |
| `referral` | `referral` | Partner/patient referral program links |
| `chatbot` | `chat` | Chatbot-initiated links and CTAs |

### UTM Examples by Funnel

**Practice Acquisition (PA):**
```
sgadental.com/partners?utm_source=google&utm_medium=cpc&utm_campaign=pa-assessment&utm_content=ad-fear-stat-headline&utm_term=sell-dental-practice

sgadental.com/partners?utm_source=linkedin&utm_medium=social-paid&utm_campaign=pa-thought-leader&utm_content=ad-lamonte-autonomy-post&utm_term=practice-owners-southeast

sgadental.com/partners?utm_source=email&utm_medium=email&utm_campaign=pa-indoctrination&utm_content=email-day3-7of10-regret
```

**Dentist Recruitment (DR):**
```
sgadental.com/careers?utm_source=linkedin&utm_medium=social-paid&utm_campaign=dr-associate-jobs&utm_content=ad-mentorship-equity&utm_term=associate-dentist-arizona

sgadental.com/careers?utm_source=email&utm_medium=email&utm_campaign=dr-nurture&utm_content=email-day6-benefits-nobody-lists
```

**Patient Acquisition (PT):**
```
sgadental.com/smile-assessment?utm_source=facebook&utm_medium=social-paid&utm_campaign=pt-implant-smile&utm_content=ad-scott-steak-story&utm_term=dental-implants-springfield-mo

sgadental.com/book?utm_source=google&utm_medium=lsa&utm_campaign=pt-general-lsa&utm_content=practice-innovative-dental&utm_term=dentist-near-me
```

**Specialist Partnership (SP):**
```
sgadental.com/specialists?utm_source=linkedin&utm_medium=social-paid&utm_campaign=sp-specialist-guide&utm_content=ad-pikos-endorsement&utm_term=periodontist-dso-partnership
```

### UTM Governance Rules

1. **Never create a new source/medium pair without adding it to the approved list.** Ad hoc values create orphan data.
2. **Campaign names always start with the funnel prefix.** No exceptions.
3. **Content values identify the specific creative or email.** They must be descriptive enough to trace back to the exact asset without looking it up.
4. **Term is optional for non-search channels.** Use it for keyword targeting (search) or audience segment targeting (social/display).
5. **A/B test variants append `-v1`, `-v2`, etc.** Example: `ad-fear-stat-headline-v1` vs `ad-empathy-headline-v2`. This directly integrates with the A/B Test Playbook (Phase 5D).
6. **Use a UTM builder spreadsheet.** Every team member creating links uses the same Google Sheet template. No freehand URL construction. The sheet validates against the approved taxonomy and flags non-conforming values.

---

## 2. Conversion Events -- Full Taxonomy

### Event Naming Convention

All custom events follow the format: `{category}_{action}_{label}`

- Category: The system or asset type (`page`, `form`, `email`, `ad`, `quiz`, `webinar`, `chatbot`, `video`, `cta`)
- Action: What the user did (`view`, `start`, `complete`, `click`, `submit`, `book`, `download`, `play`, `scroll`)
- Label: Specificity as needed (`hero`, `mid`, `footer`, `step1`, `step2`, etc.)

### Funnel Stage: Awareness

These events fire when a user first encounters SGA content or assets.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `page_view` | Any page loads | `page_title`, `page_path`, `page_referrer`, `funnel_prefix`, `audience_type` | All |
| `page_scroll_25` | User scrolls to 25% of page depth | `page_path`, `scroll_depth` | All |
| `page_scroll_50` | User scrolls to 50% of page depth | `page_path`, `scroll_depth` | All |
| `page_scroll_75` | User scrolls to 75% of page depth | `page_path`, `scroll_depth` | All |
| `page_scroll_100` | User scrolls to 100% of page depth | `page_path`, `scroll_depth` | All |
| `page_section_view` | User scrolls a specific landing page section into the viewport | `page_path`, `section_name` (e.g., `hero`, `problem`, `solution`, `social_proof`, `pillars`, `how_it_works`, `faq`, `final_cta`) | PA, PT, SP |
| `ad_impression` | Ad served (platform-side) | `platform`, `campaign_id`, `ad_set`, `creative_id` | All paid |
| `ad_click` | Ad clicked (platform-side) | `platform`, `campaign_id`, `ad_set`, `creative_id`, `cpc` | All paid |
| `blog_view` | Blog post loaded | `post_title`, `post_category`, `post_author`, `funnel_prefix` | SEO |
| `blog_read_complete` | User scrolls to 75%+ of blog content | `post_title`, `read_time_seconds` | SEO |
| `youtube_channel_view` | Organic YouTube video starts playing | `video_id`, `video_title`, `traffic_source` | PT |
| `social_engagement` | Like, comment, share on organic social post | `platform`, `post_id`, `engagement_type` | All |
| `competitor_page_view` | Competitor comparison page loaded | `comparison_type` (e.g., `heartland`, `aspen`, `pds`, `general-dso`) | PA, SP |
| `competitor_page_scroll` | Scroll depth on comparison page | `comparison_type`, `scroll_depth_percent` | PA, SP |
| `competitor_cta_click` | CTA clicked on competitor comparison page | `comparison_type`, `cta_position`, `cta_text` | PA, SP |

### Funnel Stage: Consideration

These events fire when a user engages with lead capture or evaluation content.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `quiz_start` | User begins Practice Partnership Assessment or Smile Assessment | `quiz_type` (`practice_assessment` or `smile_assessment`), `traffic_source` | PA, PT |
| `quiz_step_complete` | User completes a step/question in the quiz | `quiz_type`, `step_number`, `step_name`, `answer_value` | PA, PT |
| `quiz_abandon` | User exits quiz without completing | `quiz_type`, `last_step_completed`, `time_on_quiz_seconds` | PA, PT |
| `quiz_complete` | User finishes all quiz questions | `quiz_type`, `score`, `tier` (`strong`, `growth`, `foundation` for PA; `smile_readiness_score` for PT), `completion_time_seconds` | PA, PT |
| `quiz_results_view` | User views their quiz results page | `quiz_type`, `score`, `tier` | PA, PT |
| `quiz_results_cta_click` | User clicks CTA on quiz results page | `quiz_type`, `tier`, `cta_type` (`book_call`, `download_guide`, `subscribe`) | PA, PT |
| `lead_magnet_download` | User downloads a gated asset | `asset_name` (`career_guide`, `partnership_guide`, `specialist_guide`, `financing_guide`), `funnel_prefix` | PA, DR, PT, SP |
| `form_start` | User clicks into first field of any form | `form_name`, `form_location` | All |
| `form_field_complete` | User completes a specific form field | `form_name`, `field_name` | All |
| `form_abandon` | User leaves a form without submitting | `form_name`, `last_field_completed`, `fields_completed_count` | All |
| `form_submit` | Form successfully submitted | `form_name`, `funnel_prefix`, `lead_source` | All |
| `email_signup` | User subscribes to email list (any entry point) | `signup_source`, `funnel_prefix`, `lead_magnet` | All |
| `chatbot_start` | User initiates chatbot conversation | `page_path`, `trigger_type` (`proactive` or `user_initiated`) | All |
| `chatbot_qualify` | Chatbot qualifies lead based on responses | `qualification_score`, `audience_type`, `key_answers` | All |
| `chatbot_handoff` | Chatbot transfers to human or books appointment | `handoff_type` (`phone`, `calendar`, `email`), `audience_type` | All |
| `chatbot_conversion` | Chatbot interaction leads to desired action | `conversion_type` (`assessment_start`, `call_booked`, `guide_download`), `messages_exchanged` | All |

### Funnel Stage: Decision

These events fire when a user takes a high-intent action toward conversion.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `discovery_call_booked` | Practice owner or specialist books a discovery call | `funnel_prefix`, `lead_source`, `assessment_score`, `assessment_tier` | PA, SP |
| `discovery_call_completed` | Discovery call confirmed as completed by BD team | `funnel_prefix`, `call_outcome` (`advance_to_site_visit`, `nurture`, `disqualified`), `call_duration_minutes` | PA, SP |
| `site_visit_scheduled` | Site visit booked with a partner practice | `funnel_prefix`, `practice_name`, `region` | PA, SP |
| `site_visit_completed` | Site visit confirmed as completed | `funnel_prefix`, `visit_outcome` (`advance_to_loi`, `nurture`, `declined`) | PA, SP |
| `consultation_booked` | Patient books a virtual or in-person consultation | `consultation_type` (`virtual`, `in_person`), `practice_name`, `service_type` (`implant`, `general`, `cosmetic`), `lead_source` | PT |
| `consultation_completed` | Consultation confirmed as attended | `practice_name`, `service_type`, `outcome` (`treatment_plan_accepted`, `followup_needed`, `declined`) | PT |
| `application_started` | Associate dentist begins job application | `position_id`, `position_title`, `practice_name`, `lead_source` | DR |
| `application_submitted` | Associate dentist completes job application | `position_id`, `position_title`, `practice_name` | DR |
| `interview_scheduled` | Interview booked with candidate | `position_id`, `interview_type` (`phone_screen`, `clinical`, `practice_visit`) | DR |
| `webinar_pitch_cta_click` | User clicks the pitch CTA during or after webinar | `webinar_id`, `cta_type`, `timing` (`live`, `replay`) | PA, SP |
| `pricing_page_view` | User views the pricing/partnership economics page | `funnel_prefix`, `referrer_page` | PA, SP, PT |
| `pricing_calculator_use` | User interacts with the financing calculator (implant patients) | `monthly_payment_selected`, `treatment_type`, `total_estimate` | PT |

### Funnel Stage: Close / Conversion

These events represent completed conversions -- the business outcomes that matter.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `loi_signed` | Letter of intent signed by practice owner or specialist | `funnel_prefix`, `practice_name`, `practice_revenue_tier`, `days_from_first_touch`, `attribution_source` | PA, SP |
| `deal_closed` | Partnership deal fully executed | `funnel_prefix`, `practice_name`, `deal_value_tier`, `acquisition_cost_total` | PA, SP |
| `treatment_accepted` | Patient accepts treatment plan | `practice_name`, `service_type`, `treatment_value`, `financing_method` | PT |
| `treatment_completed` | Patient completes treatment | `practice_name`, `service_type`, `treatment_value`, `days_to_completion` | PT |
| `offer_accepted` | Associate dentist accepts job offer | `position_id`, `practice_name`, `days_from_application`, `lead_source` | DR |
| `hire_retained_90d` | New hire reaches 90-day retention milestone | `position_id`, `practice_name`, `satisfaction_score` | DR |

### Funnel Stage: Retention / Referral

These events track post-conversion engagement and the viral referral loops defined in the funnel architecture.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `review_requested` | Review request sent to patient post-treatment | `practice_name`, `platform` (`google`, `yelp`, `healthgrades`), `days_post_treatment` | PT |
| `review_submitted` | Patient submits a review | `practice_name`, `platform`, `rating`, `days_post_treatment` | PT |
| `referral_link_shared` | Referral link clicked/shared by existing contact | `referrer_type` (`patient`, `partner`, `associate`), `funnel_prefix` | All |
| `referral_conversion` | Referred contact converts | `referrer_id`, `conversion_type`, `funnel_prefix` | All |
| `testimonial_captured` | Transformation story or partner testimonial collected | `testimonial_type` (`written`, `video`), `funnel_prefix`, `practice_name` | PA, PT, SP |
| `newsletter_open` | Partner Pulse or other retention newsletter opened | `newsletter_name`, `audience_type` | PA, SP, DR |
| `recare_booked` | Patient books recall/recare appointment | `practice_name`, `months_since_last_visit` | PT |
| `ce_event_registered` | Partner/associate registers for CE event | `event_name` (`4front`, `pikos_symposium`, `webinar`), `attendee_type` | PA, SP, DR |
| `re_engagement_responded` | Cold contact re-engages from re-engagement sequence | `funnel_prefix`, `re_engagement_email_number`, `response_type` (`open`, `click`, `reply`) | All |

### Email-Specific Events

These track engagement within the 7 email sequences defined in the funnel architecture.

| Event Name | Trigger | Parameters | Funnel(s) |
|------------|---------|------------|-----------|
| `email_sent` | Email dispatched from ESP | `sequence_name`, `email_number`, `subject_line`, `funnel_prefix` | All |
| `email_delivered` | Email confirmed delivered | `sequence_name`, `email_number` | All |
| `email_opened` | Email opened | `sequence_name`, `email_number`, `open_count` (first open vs. re-open) | All |
| `email_clicked` | Link clicked within email | `sequence_name`, `email_number`, `link_url`, `link_position` | All |
| `email_replied` | Recipient replies to email | `sequence_name`, `email_number` | All |
| `email_unsubscribed` | Recipient unsubscribes | `sequence_name`, `email_number`, `reason` (if captured) | All |
| `email_bounced` | Email bounces (hard or soft) | `sequence_name`, `email_number`, `bounce_type` | All |
| `email_marked_spam` | Recipient marks as spam | `sequence_name`, `email_number` | All |

---

## 3. KPIs per Channel -- Target Benchmarks

### Paid Search (Google Ads)

| KPI | PA Funnel | DR Funnel | PT Funnel (General) | PT Funnel (Implant) | SP Funnel |
|-----|-----------|-----------|---------------------|---------------------|-----------|
| CTR | > 4.0% | > 3.5% | > 5.0% | > 4.0% | > 3.5% |
| CPC | < $25 | < $8 | < $12 | < $18 | < $30 |
| CPL (form fill / assessment start) | < $300 | < $60 | < $75 | < $120 | < $350 |
| Conversion Rate (click to lead) | > 8% | > 12% | > 15% | > 8% | > 7% |
| Quality Score (avg) | > 7 | > 7 | > 6 | > 7 | > 7 |
| ROAS | Track and baseline (long sales cycle) | N/A (cost-per-hire model) | > 5:1 (general), > 8:1 (implant) | > 8:1 | Track and baseline |
| Impression Share | > 60% (branded), > 30% (non-branded) | > 40% | > 50% (local) | > 40% | > 50% (niche terms) |

### Google Local Services Ads (LSAs)

| KPI | Target |
|-----|--------|
| Cost per booked appointment | < $75 (general), < $130 (implant) |
| Lead response time | < 5 minutes |
| Lead-to-appointment rate | > 30% |
| Review rating (required for LSA eligibility) | > 4.5 stars |
| Budget pacing accuracy | Within 10% of monthly target |

### Facebook / Instagram Ads

| KPI | PA Funnel | DR Funnel | PT Funnel (General) | PT Funnel (Implant) | SP Funnel |
|-----|-----------|-----------|---------------------|---------------------|-----------|
| CTR (link clicks) | > 1.2% | > 1.5% | > 1.8% | > 1.5% | > 1.0% |
| CPM | < $35 | < $25 | < $20 | < $25 | < $45 |
| CPL | < $150 | < $50 | < $65 | < $90 | < $175 |
| Conversion Rate (click to lead) | > 5% | > 8% | > 10% | > 6% | > 4% |
| Frequency (avg over 30 days) | < 4.0 | < 5.0 | < 3.5 | < 4.0 | < 3.0 |
| Relevance Score / Quality Ranking | Above Average | Above Average | Above Average | Above Average | Above Average |
| ROAS | Track and baseline | N/A | > 4:1 | > 6:1 | Track and baseline |
| Video View Rate (video ads) | > 25% (15-sec), > 15% (30-sec) | > 20% | > 30% (testimonial) | > 25% (transformation) | > 20% |

### LinkedIn Ads

| KPI | PA Funnel | DR Funnel | SP Funnel |
|-----|-----------|-----------|-----------|
| CTR | > 0.8% (Sponsored Content), > 3.0% (InMail) | > 0.6% (Sponsored Jobs) | > 0.7% |
| CPL (Lead Gen Form completion) | < $150 | < $80 | < $175 |
| Lead Gen Form completion rate | > 12% | > 15% | > 10% |
| InMail response rate | > 15% | N/A | > 12% |
| Cost per InMail response | < $50 | N/A | < $60 |
| Engagement rate (organic thought leader posts) | > 3% | > 2% | > 3% |

### YouTube (Organic + Paid)

| KPI | Target | Notes |
|-----|--------|-------|
| Subscriber growth (Dr. Olson channel) | > 3% month-over-month | Currently 138K subscribers |
| Average view duration | > 50% of video length | For transformation videos |
| CTR (thumbnail) | > 5% | Organic click-through rate |
| Views per video (first 48 hours) | > 10,000 (transformation), > 5,000 (educational) | Organic reach |
| Video-to-assessment conversion rate | > 2% of viewers | Tracked via UTM on description links |
| Comments per video | > 50 (transformation), > 20 (educational) | Engagement signal |
| Paid video ad: Cost Per View (CPV) | < $0.05 | TrueView in-stream |
| Paid video ad: View-through rate | > 30% | 15-sec+ view |

### Email Sequences

| KPI | PA Sequences | DR Sequences | PT Sequences | SP Sequences | Re-Engagement |
|-----|-------------|-------------|-------------|-------------|---------------|
| Open Rate | > 40% | > 35% | > 45% | > 45% | > 25% |
| Click Rate | > 5% | > 4% | > 5% | > 6% | > 3% |
| Click-to-Open Rate | > 12% | > 11% | > 11% | > 13% | > 12% |
| Unsubscribe Rate | < 0.3% | < 0.3% | < 0.5% | < 0.2% | < 1.0% |
| Bounce Rate | < 1% | < 1% | < 2% | < 0.5% | < 2% |
| Spam Complaint Rate | < 0.05% | < 0.05% | < 0.08% | < 0.03% | < 0.1% |
| Sequence Completion Rate | > 60% | > 55% | > 50% | > 65% | > 40% |
| Revenue per 1,000 subscribers/month | Baseline in first 90 days | N/A | Baseline in first 90 days | Baseline in first 90 days | N/A |

### SEO / Organic Search

| KPI | Target | Measurement |
|-----|--------|-------------|
| Organic traffic growth | > 15% month-over-month (first 6 months) | GA4 organic sessions |
| Keyword rankings (target keywords) | Top 10 for 15+ primary keywords within 6 months | Ahrefs / Semrush |
| Domain Authority / Domain Rating | Increase by 5+ points in 6 months | Ahrefs DR |
| Blog engagement (avg time on page) | > 3 minutes | GA4 engagement time |
| Blog-to-lead conversion rate | > 2% of blog readers complete a lead capture action | Custom event tracking |
| Internal link click-through rate (blog to assessment/landing page) | > 8% | Custom event tracking |
| Backlinks acquired per month | > 10 quality backlinks from DA 30+ sites | Ahrefs |
| Featured snippet captures | > 5 featured snippets for DSO-related queries | Semrush |

### Social Media (Organic)

| KPI | Facebook/Instagram | LinkedIn | YouTube |
|-----|-------------------|----------|---------|
| Engagement rate | > 3% | > 3% | > 5% (likes + comments / views) |
| Follower growth rate | > 5% month-over-month | > 8% month-over-month | > 3% month-over-month |
| Reach per post | > 10% of follower base | > 15% of connections | N/A (view-based) |
| Content-to-site traffic | > 2% of engagers click through to site | > 3% | > 2% click description links |
| Share/save rate | > 1% of impressions | > 2% of impressions | > 0.5% save-to-playlist rate |
| Video completion rate (Reels/Stories) | > 40% (15-sec), > 20% (60-sec) | > 35% | See YouTube section above |

### Referral Program

| KPI | Patient Referral | Partner Referral | Associate Referral |
|-----|-----------------|-----------------|-------------------|
| Referral rate | > 8 per 100 patients per quarter | > 1-2 warm intros per partner per year | > 15-20% of hires from internal referrals |
| Cost per referred conversion | < $40 | $0 (equity-aligned) | < $5,000 (referral bonus) |
| Referral link share rate | > 15% of recipients share the link | N/A (warm intros) | N/A |
| Referred-to-converted rate | > 30% (vs. ~15% for cold leads) | > 40% | > 35% |

---

## 4. Attribution Model

### Recommended Model: Position-Based (U-Shaped) for B2B Funnels, Data-Driven for Patient Funnel

SGA's four funnels have fundamentally different buying journeys. A single attribution model cannot serve all of them. The recommendation is a two-model approach:

#### B2B Funnels (PA, DR, SP): Position-Based (40/20/40)

**How it works:**
- 40% credit to the **first touch** (the channel that created initial awareness)
- 20% distributed across **middle touches** (nurture emails, content, retargeting)
- 40% credit to the **last touch** (the channel that drove the final conversion action)

**Why this model for B2B:**

1. **Long sales cycles (6-24 months for PA/SP, 2-8 weeks for DR).** First-touch and last-touch models each capture only one end of a long journey. A practice owner might first find SGA via a Google search for "sell dental practice," engage with 12 emails and 3 LinkedIn posts over 4 months, and ultimately book a discovery call after a conference dinner. Both the initial search ad and the conference interaction deserve significant credit. The middle nurture touchpoints deserve moderate credit for keeping the prospect warm.

2. **High consideration, high stakes.** A $2.4M partnership decision involves multiple research sessions across channels. Ignoring middle touches would undervalue the email sequences and content marketing that sustain engagement during the long consideration phase.

3. **Small audience, high value.** With only 5-15 discovery calls per month (PA funnel), every data point matters. Position-based gives the most balanced view of what creates pipeline and what closes it.

**Implementation:**
- Configure in GA4 using custom channel groupings
- Supplement with CRM-based attribution (HubSpot or equivalent) that tracks touchpoints from first form fill through deal close
- CRM attribution is the source of truth for revenue attribution; GA4 handles web-behavior attribution

#### Patient Funnel (PT): Data-Driven Attribution

**How it works:**
- Google's machine learning model analyzes all conversion paths and distributes credit based on actual conversion probability at each touchpoint
- Requires 600+ conversions per month and 15,000+ clicks to be reliable (achievable across the SGA practice network)

**Why this model for patients:**

1. **Shorter sales cycle (1-8 weeks).** Patient journeys are compact enough for data-driven models to identify meaningful patterns quickly.
2. **Higher volume.** Hundreds of patient appointments per month across the SGA network provide enough data for the algorithm to learn.
3. **Multi-channel complexity.** Patients see YouTube videos, Facebook ads, Google search results, and email nurture sequences in unpredictable orders. Data-driven attribution adapts to actual behavior rather than imposing a fixed model.

**Fallback:** If the patient funnel does not reach the conversion volume threshold for data-driven attribution, use **time-decay attribution** with a 7-day half-life. This model gives more credit to touchpoints closer to conversion, which aligns with the shorter, higher-urgency patient decision cycle.

### Attribution Implementation Requirements

| Component | Tool | Purpose |
|-----------|------|---------|
| Web attribution | GA4 with custom channel groupings | Website behavior and conversion tracking |
| CRM attribution | HubSpot (or Salesforce) | Full-journey tracking from first touch to closed deal |
| Cross-device tracking | GA4 User-ID + Google Signals | Connect sessions across devices |
| Offline conversion import | Google Ads Offline Conversion Import + Meta CAPI | Feed CRM conversion data back to ad platforms for optimization |
| Call tracking | CallRail or CallTrackingMetrics | Attribute phone calls to marketing touchpoints |
| UTM parsing | CRM auto-parse on form submission | Store UTM parameters as contact properties for attribution analysis |

### Attribution Reporting Views

Build three standard attribution reports:

1. **Channel Comparison Report:** Side-by-side view of first-touch, last-touch, and position-based attribution for each channel. Identifies channels that start conversations (first-touch heavy) vs. channels that close them (last-touch heavy).

2. **Path Analysis Report:** Most common conversion paths by funnel. Shows which channel sequences produce the highest conversion rates. Example output: "Google CPC -> Email Nurture -> LinkedIn Retargeting -> Conference -> Discovery Call" converts at 4.2% vs. the 2.8% funnel average.

3. **Time-to-Conversion Report:** Days from first touch to each conversion milestone (lead, MQL, SQL, opportunity, close) by funnel. Benchmarks the sales cycle and identifies stalls.

---

## 5. Dashboard Structure

### Dashboard 1: Executive Overview (Weekly Review)

**Audience:** CEO, CMO, VP Marketing, VP Business Development
**Tool:** Looker Studio (connected to GA4 + CRM + ad platform APIs)
**Refresh:** Daily auto-refresh, reviewed weekly

| Section | Metrics | Visualization |
|---------|---------|---------------|
| Funnel Health Summary | Leads, MQLs, SQLs, opportunities, conversions -- by funnel (PA, DR, PT, SP) | 4-column funnel bars with week-over-week trend arrows |
| Revenue Attribution | Revenue attributed to marketing by channel (position-based) | Stacked bar chart, monthly trend |
| Blended CAC | Total marketing spend / total conversions, by funnel | Single number with trend line |
| Pipeline Value | Open opportunity value by funnel and stage | Horizontal funnel chart |
| Top Performing Channels | Top 5 channels by CPL, conversion rate, and volume | Ranked table with conditional formatting |
| Budget vs. Actual | Spend by channel vs. budget allocation | Bar chart with budget line overlay |
| A/B Test Scorecard | Active tests, status, results summary | Status table (green/yellow/red) |

### Dashboard 2: Paid Media Performance (Daily Monitoring)

**Audience:** Media buyer, marketing manager
**Tool:** Looker Studio (connected to Google Ads, Meta Ads Manager, LinkedIn Campaign Manager APIs)
**Refresh:** Daily

| Section | Metrics | Visualization |
|---------|---------|---------------|
| Google Ads | Spend, impressions, clicks, CTR, CPC, conversions, CPL, ROAS -- by campaign | Campaign-level table + trend sparklines |
| Meta Ads | Spend, impressions, reach, frequency, clicks, CTR, CPL, ROAS -- by ad set | Ad set table + creative preview thumbnails |
| LinkedIn Ads | Spend, impressions, clicks, CTR, CPL, Lead Gen Form fills -- by campaign | Campaign table |
| YouTube Ads | Spend, views, CPV, view-through rate, conversions -- by video | Video table with thumbnails |
| LSAs | Spend, leads, cost per lead, lead quality rating -- by practice | Practice-level table |
| Budget Pacing | Daily spend vs. daily budget, projected monthly spend vs. budget | Line chart with budget ceiling |
| Audience Insights | Top performing audiences by demographic, geo, device | Demographic breakdown charts |
| Creative Performance | CTR and CPL by ad creative variant | Creative comparison table (integrates with A/B test tracking) |

### Dashboard 3: Email and Sequence Performance (Weekly Review)

**Audience:** Email marketing manager, content strategist
**Tool:** Looker Studio (connected to ESP API -- ActiveCampaign, HubSpot, or Mailchimp)
**Refresh:** Daily

| Section | Metrics | Visualization |
|---------|---------|---------------|
| Sequence Health | Open rate, click rate, unsubscribe rate, bounce rate -- by sequence | Sequence comparison table with health thresholds (green/yellow/red) |
| Email-Level Performance | Open rate, click rate, click-to-open rate -- by individual email | Heatmap grid (sequences as rows, emails as columns) |
| List Health | Total active subscribers, growth rate, deliverability, spam rate -- by funnel | Trend lines with threshold indicators |
| Sequence Completion | Percentage of contacts who complete each sequence | Horizontal funnel chart per sequence |
| Re-Engagement | Cold contacts identified, re-engagement sequence response rate, list cleaned | Trend chart |
| Revenue Attribution | Conversions attributed to email sequences (last-click and position-based) | Bar chart by sequence |

### Dashboard 4: SEO and Content Performance (Monthly Review)

**Audience:** Content strategist, SEO specialist
**Tool:** Looker Studio (connected to GA4 + Search Console + Ahrefs/Semrush API)
**Refresh:** Weekly

| Section | Metrics | Visualization |
|---------|---------|---------------|
| Organic Traffic | Sessions, users, new users -- by landing page cluster | Trend line with month-over-month comparison |
| Keyword Rankings | Position tracking for target keywords (top 50) | Ranking distribution chart + movement table |
| Content Performance | Page views, avg engagement time, scroll depth, conversion rate -- by blog post | Post-level table sorted by conversion contribution |
| Backlink Profile | New backlinks, referring domains, domain rating trend | Trend line + new link table |
| Technical Health | Core Web Vitals (LCP, FID, CLS), crawl errors, index coverage | Scorecard with pass/fail indicators |
| Competitor Comparison Page Performance | Views, scroll depth, CTA clicks, conversion rate -- by comparison page | Comparison page table |
| Content Gap Analysis | Keyword opportunities identified vs. content produced | Progress tracker |

### Dashboard 5: Funnel-Specific Deep Dives (Monthly Review)

**Audience:** Funnel owner, marketing manager
**Tool:** Looker Studio (one tab per funnel)
**Refresh:** Daily

**PA (Practice Acquisition) Tab:**
- Assessment completion funnel: visitors > starts > step completions > finishes, with drop-off at each step
- Score distribution: histogram of assessment scores by tier
- Email sequence engagement waterfall: opens and clicks by email number
- Discovery call pipeline: calls booked, completed, advanced, by lead source
- Time-to-conversion analysis: days from first touch to each stage

**DR (Dentist Recruitment) Tab:**
- Career page traffic by source
- Guide download rate and application conversion
- Application funnel: started > completed > phone screen > interview > offer > hire
- 90-day retention tracking
- Source-of-hire analysis

**PT (Patient Acquisition) Tab:**
- Smile Assessment funnel: starts > step completions > finishes > consultation booked
- Practice-level performance: new patients, CPL, conversion rate by practice
- Treatment acceptance rates by practice and service type
- Review generation metrics
- Referral program performance

**SP (Specialist Partnership) Tab:**
- Specialist guide downloads by source
- Email sequence engagement waterfall
- Discovery call pipeline
- Conference ROI by event
- Pikos Institute conversion funnel (course attendee > engaged prospect > partnership conversation)

### Dashboard 6: V2 Asset Performance (Weekly Review)

**Audience:** Marketing manager, growth team
**Tool:** Looker Studio
**Refresh:** Daily

| Section | Metrics | Visualization |
|---------|---------|---------------|
| Quiz/Scorecard | Start rate, step-by-step completion, abandon points, score distribution, results page CTA clicks | Funnel chart + abandonment heatmap |
| Webinar | Registration rate, attendance rate, replay views, pitch CTA clicks, post-webinar conversions | Funnel chart with reg > attend > convert |
| Chatbot | Conversations started, qualification rate, handoff rate, conversion rate, avg messages to conversion | Flow chart + conversation metrics |
| VSL | Play rate, 25%/50%/75%/100% completion, CTA click rate, post-VSL conversions | Retention curve + CTA performance |
| YouTube Content | Views, retention curve, subscriber conversion, description link clicks | Per-video retention charts |
| A/B Tests | Active tests, sample progress, statistical significance, winner/loser status | Test status board |

### Recommended Tool Stack

| Function | Primary Tool | Backup/Alternative |
|----------|-------------|-------------------|
| Web analytics | GA4 | Mixpanel (for product analytics on quiz/assessment) |
| Tag management | Google Tag Manager (GTM) | -- |
| Dashboards | Looker Studio | Databox (for real-time TV dashboards) |
| CRM | HubSpot Marketing Hub + Sales Hub | Salesforce |
| Email marketing | HubSpot (if CRM) or ActiveCampaign | Mailchimp |
| Call tracking | CallRail | CallTrackingMetrics |
| Heatmaps / session replay | Microsoft Clarity (free) or Hotjar | -- |
| SEO tracking | Ahrefs + Google Search Console | Semrush |
| Ad platform connectors | Supermetrics or Funnel.io | -- |
| A/B testing | Google Optimize (sunset -- use VWO or Convert) | Unbounce (for landing page tests) |
| Chatbot analytics | Drift or Intercom (built-in) | -- |
| Video analytics | YouTube Studio + Wistia (for VSL) | Vidyard |

---

## 6. V2 Tracking Requirements

### 6A. Quiz/Scorecard Completion Tracking

Two quizzes operate in the SGA ecosystem: the Practice Partnership Assessment (PA funnel, 12 questions across 4 categories) and the Smile Assessment (PT funnel, 8 questions).

#### Events to Track

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `quiz_impression` | Quiz widget or page loads in viewport | `quiz_type`, `page_path`, `traffic_source` | Measures exposure-to-start rate |
| `quiz_start` | User clicks to begin quiz | `quiz_type`, `traffic_source`, `referrer` | Entry rate tracking |
| `quiz_step_view` | Each question loads | `quiz_type`, `step_number`, `step_name` | Identify which questions cause hesitation |
| `quiz_step_answer` | User selects/submits an answer | `quiz_type`, `step_number`, `answer_value`, `time_on_step_seconds` | Scoring data + time analysis |
| `quiz_step_back` | User navigates backward | `quiz_type`, `from_step`, `to_step` | Identifies confusing questions |
| `quiz_abandon` | User leaves without completing | `quiz_type`, `last_step_completed`, `total_time_seconds`, `answers_given` | Drop-off analysis |
| `quiz_complete` | Final question answered | `quiz_type`, `total_score`, `tier`, `completion_time_seconds`, `category_scores` | Completion and scoring |
| `quiz_results_view` | Results page loads | `quiz_type`, `score`, `tier`, `recommended_cta` | Results page engagement |
| `quiz_results_cta_click` | User clicks CTA on results page | `quiz_type`, `tier`, `cta_type` (`book_call`, `download_guide`, `subscribe_content`) | Post-quiz conversion |
| `quiz_results_share` | User shares results (if sharing is enabled) | `quiz_type`, `share_platform` | Viral coefficient tracking |

#### Scoring Data to Capture

For the **Practice Partnership Assessment**, capture and store:

```
{
  "quiz_type": "practice_assessment",
  "total_score": 78,
  "tier": "growth",
  "category_scores": {
    "operational_readiness": 22,
    "financial_position": 18,
    "growth_potential": 24,
    "personal_readiness": 14
  },
  "completion_time_seconds": 234,
  "traffic_source": "google_cpc",
  "utm_campaign": "pa-assessment"
}
```

For the **Smile Assessment**, capture and store:

```
{
  "quiz_type": "smile_assessment",
  "smile_readiness_score": 72,
  "current_situation": "failing_teeth",
  "primary_concern": "cost",
  "financing_preference": "monthly_payments",
  "willingness_to_travel": "within_state",
  "completion_time_seconds": 156,
  "traffic_source": "facebook_paid",
  "utm_campaign": "pt-implant-smile"
}
```

#### KPIs for Quiz Performance

| Metric | Practice Assessment Target | Smile Assessment Target |
|--------|---------------------------|------------------------|
| Impression-to-start rate | > 50% | > 60% |
| Start-to-complete rate | > 35% (from funnel architecture target) | > 40% (from funnel architecture target) |
| Average completion time | 2-4 minutes (12 questions) | 1-3 minutes (8 questions) |
| Results page CTA click rate | > 40% (Strong tier), > 25% (Growth tier), > 10% (Foundation tier) | > 35% |
| Highest drop-off step | Identify and optimize | Identify and optimize |

---

### 6B. Webinar Registration, Attendance, and Replay Tracking

#### Events to Track

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `webinar_page_view` | Registration page loads | `webinar_id`, `webinar_title`, `funnel_prefix`, `traffic_source` | Registration page traffic |
| `webinar_reg_start` | User clicks into registration form | `webinar_id`, `traffic_source` | Form engagement rate |
| `webinar_reg_complete` | Registration form submitted | `webinar_id`, `registrant_email`, `registrant_type` (`practice_owner`, `associate`, `specialist`, `patient`), `traffic_source` | Registration conversion |
| `webinar_reminder_sent` | Reminder email dispatched | `webinar_id`, `reminder_number` (`24h`, `1h`, `starting_now`) | Reminder delivery |
| `webinar_reminder_opened` | Reminder email opened | `webinar_id`, `reminder_number` | Reminder engagement |
| `webinar_join_live` | Attendee joins live session | `webinar_id`, `join_time_relative` (`on_time`, `5min_late`, `10min_late`, `15min_plus_late`) | Attendance timing |
| `webinar_attendance_duration` | Periodic check (every 5 minutes) while attendee is present | `webinar_id`, `minutes_attended`, `total_duration` | Retention tracking |
| `webinar_poll_response` | Attendee responds to in-webinar poll | `webinar_id`, `poll_question`, `poll_answer` | Engagement and qualification data |
| `webinar_chat_message` | Attendee sends chat message | `webinar_id`, `message_count` | Engagement scoring |
| `webinar_question_submitted` | Attendee submits a Q&A question | `webinar_id`, `question_text` | High-intent signal |
| `webinar_pitch_view` | Pitch/offer slide or segment is shown | `webinar_id`, `attendees_present_at_pitch` | Pitch audience size |
| `webinar_pitch_cta_click` | Attendee clicks the pitch CTA (book call, assessment, etc.) | `webinar_id`, `cta_type`, `timing` (`during_live`, `post_live`) | Pitch conversion |
| `webinar_exit` | Attendee leaves the live session | `webinar_id`, `exit_time_minutes`, `attended_through_pitch` (`yes`/`no`) | Drop-off analysis |
| `webinar_replay_view` | Replay page loads | `webinar_id`, `days_after_live`, `traffic_source` | Replay traffic |
| `webinar_replay_play` | Replay video starts playing | `webinar_id`, `viewer_type` (`registered_no_show`, `registered_attendee`, `new_viewer`) | Replay engagement |
| `webinar_replay_progress` | Replay reaches 25%, 50%, 75%, 100% | `webinar_id`, `progress_percent`, `viewer_type` | Replay retention |
| `webinar_replay_pitch_cta_click` | Replay viewer clicks the pitch CTA | `webinar_id`, `cta_type`, `viewer_type` | Replay conversion |

#### Webinar Lead Scoring

Assign engagement scores that feed into the CRM lead score:

| Action | Points |
|--------|--------|
| Registered | +5 |
| Attended live | +15 |
| Attended through pitch | +25 |
| Asked a Q&A question | +10 |
| Clicked pitch CTA (live) | +30 |
| Watched replay (100%) | +20 |
| Clicked pitch CTA (replay) | +25 |
| No-show (registered, never watched) | +0 (registered points only) |

#### KPIs for Webinar Performance

| Metric | Target |
|--------|--------|
| Registration page conversion rate | > 35% |
| Registration-to-attendance rate | > 45% (target from A/B test playbook: 48-58% post-optimization) |
| Avg attendance duration | > 70% of total webinar length |
| Attendees remaining at pitch | > 65% of peak attendance |
| Live pitch CTA click rate | > 12% |
| Replay view rate (of no-shows) | > 25% |
| Replay pitch CTA click rate | > 8% |
| Webinar-to-discovery-call conversion (within 14 days) | > 8% (PA/SP), > 15% (PT) |

---

### 6C. Chatbot Conversation Tracking and Lead Scoring

#### Events to Track

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `chatbot_widget_view` | Chatbot widget loads on page | `page_path`, `funnel_prefix`, `visitor_type` (`new`, `returning`) | Widget exposure |
| `chatbot_proactive_trigger` | Chatbot proactively initiates (time-on-page, scroll, exit-intent) | `page_path`, `trigger_type`, `trigger_threshold` | Proactive engagement rate |
| `chatbot_start` | User sends first message or clicks to open | `page_path`, `trigger_type` (`proactive`, `user_initiated`, `cta_click`), `funnel_prefix` | Conversation initiation |
| `chatbot_message_sent` | User sends a message | `conversation_id`, `message_number`, `message_length` | Engagement depth |
| `chatbot_message_received` | Bot sends a response | `conversation_id`, `message_number`, `response_type` (`automated`, `human`) | Response tracking |
| `chatbot_qualify_question` | Bot asks a qualification question | `conversation_id`, `question_type` (`role`, `practice_size`, `timeline`, `location`, `dental_issue`) | Qualification flow |
| `chatbot_qualify_answer` | User answers a qualification question | `conversation_id`, `question_type`, `answer_value` | Qualification data capture |
| `chatbot_qualified` | Bot determines lead is qualified | `conversation_id`, `qualification_score`, `audience_type` (`practice_owner`, `associate`, `patient`, `specialist`), `recommended_action` | Qualification completion |
| `chatbot_handoff_offered` | Bot offers handoff to human or action | `conversation_id`, `handoff_type` (`phone`, `calendar`, `email`, `live_agent`) | Handoff rate |
| `chatbot_handoff_accepted` | User accepts handoff | `conversation_id`, `handoff_type` | Handoff conversion |
| `chatbot_handoff_declined` | User declines handoff | `conversation_id`, `handoff_type`, `reason` (if captured) | Handoff friction analysis |
| `chatbot_conversion` | User completes desired action via chatbot | `conversation_id`, `conversion_type` (`assessment_start`, `call_booked`, `guide_download`, `appointment_booked`), `messages_to_conversion` | Chatbot ROI |
| `chatbot_end` | Conversation ends (user closes, inactivity timeout) | `conversation_id`, `end_type` (`user_closed`, `timeout`, `handoff_complete`, `conversion_complete`), `total_messages`, `duration_seconds` | Conversation completion |
| `chatbot_feedback` | User rates the chatbot experience | `conversation_id`, `rating`, `feedback_text` | CSAT tracking |

#### Chatbot Lead Scoring

| Signal | Points | Notes |
|--------|--------|-------|
| Started conversation | +3 | Basic engagement signal |
| Answered 1+ qualification question | +5 | Showed willingness to share info |
| Identified as practice owner (PA) | +15 | High-value audience |
| Identified as specialist (SP) | +15 | High-value audience |
| Identified as patient seeking implants (PT) | +10 | High-value procedure |
| Accepted handoff to phone/calendar | +20 | Strong conversion intent |
| Completed conversion action via chatbot | +25 | Converted |
| Declined handoff | -5 | Lower immediate intent |
| Conversation < 2 messages and closed | -3 | Low engagement |

#### KPIs for Chatbot Performance

| Metric | Target |
|--------|--------|
| Widget-to-conversation rate | > 3% of page visitors |
| Proactive trigger engagement rate | > 8% of triggered visitors |
| Average messages per conversation | 4-8 (optimal qualification range) |
| Qualification completion rate | > 60% of started conversations |
| Handoff acceptance rate | > 50% of offered handoffs |
| Chatbot-to-conversion rate | > 12% of qualified conversations |
| Average time to conversion | < 5 minutes |
| CSAT score | > 4.0 / 5.0 |

---

### 6D. Video View and Retention Tracking (YouTube + VSL)

#### YouTube Content Events

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `yt_video_impression` | Video appears in search/suggested/home feed | `video_id`, `video_title`, `impression_source` | Discoverability |
| `yt_video_click` | Thumbnail clicked to start watching | `video_id`, `video_title`, `click_source` | Thumbnail CTR |
| `yt_video_view` | Video starts playing (30 seconds or full video if < 30 sec) | `video_id`, `video_title`, `traffic_source`, `viewer_type` (`subscriber`, `non_subscriber`) | View tracking |
| `yt_retention_25` | Viewer reaches 25% of video | `video_id`, `video_length_seconds` | Early retention |
| `yt_retention_50` | Viewer reaches 50% of video | `video_id`, `video_length_seconds` | Mid retention |
| `yt_retention_75` | Viewer reaches 75% of video | `video_id`, `video_length_seconds` | Late retention |
| `yt_retention_100` | Viewer reaches 100% of video | `video_id`, `video_length_seconds` | Full view |
| `yt_subscribe` | Viewer subscribes from this video | `video_id` | Subscriber conversion |
| `yt_like` | Viewer likes the video | `video_id` | Engagement signal |
| `yt_comment` | Viewer comments on the video | `video_id`, `comment_length` | Engagement signal |
| `yt_share` | Viewer shares the video | `video_id`, `share_platform` | Viral coefficient |
| `yt_card_click` | Viewer clicks an info card or end screen element | `video_id`, `card_type`, `card_destination` | Cross-promotion tracking |
| `yt_description_link_click` | Viewer clicks a link in the description | `video_id`, `link_url`, `link_position` | Off-platform conversion tracking (via UTM) |

**Note:** Most YouTube events are tracked natively in YouTube Studio analytics. The description link clicks require UTM tracking on all description URLs. For cross-platform attribution, use the UTM convention: `utm_source=youtube&utm_medium=social-organic&utm_campaign=pt-implant-smile&utm_content=video-{video-slug}`.

#### VSL (Video Sales Letter) Events

VSLs are self-hosted (Wistia, Vimeo, or custom player) and require custom event tracking via GTM.

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `vsl_page_view` | VSL page loads | `vsl_id`, `vsl_title`, `funnel_prefix`, `traffic_source` | Page traffic |
| `vsl_play` | Play button clicked | `vsl_id`, `auto_play` (`true`/`false`), `traffic_source` | Play rate |
| `vsl_progress_25` | Video reaches 25% | `vsl_id`, `video_duration_seconds` | Early retention |
| `vsl_progress_50` | Video reaches 50% | `vsl_id`, `video_duration_seconds` | Mid retention |
| `vsl_progress_75` | Video reaches 75% | `vsl_id`, `video_duration_seconds` | Late retention |
| `vsl_progress_100` | Video reaches 100% | `vsl_id`, `video_duration_seconds` | Full view |
| `vsl_pause` | User pauses video | `vsl_id`, `pause_time_seconds`, `pause_count` | Engagement pattern |
| `vsl_seek` | User seeks forward or backward | `vsl_id`, `seek_from_seconds`, `seek_to_seconds`, `seek_direction` | Content interest mapping |
| `vsl_mute` | User mutes/unmutes | `vsl_id`, `muted` (`true`/`false`), `mute_time_seconds` | Audio engagement |
| `vsl_cta_reveal` | CTA button appears (typically at a timed trigger) | `vsl_id`, `reveal_time_seconds`, `viewers_at_reveal` | CTA exposure |
| `vsl_cta_click` | User clicks the CTA | `vsl_id`, `cta_type`, `click_time_seconds`, `video_progress_at_click` | CTA conversion |
| `vsl_exit` | User leaves the page | `vsl_id`, `exit_time_seconds`, `video_progress_at_exit`, `cta_clicked` (`true`/`false`) | Drop-off analysis |

#### Video KPIs

| Metric | YouTube (Dr. Olson Channel) | VSL |
|--------|---------------------------|-----|
| Play rate | N/A (auto-plays on YouTube) | > 65% of page visitors |
| 25% retention | > 70% | > 60% |
| 50% retention | > 55% | > 45% |
| 75% retention | > 40% | > 30% |
| 100% retention | > 30% | > 20% |
| CTA click rate | > 2% (description links) | > 8% of those who reach the CTA reveal point |
| Post-video conversion rate | > 1.5% (video view to assessment start) | > 5% (VSL view to desired action) |

---

### 6E. A/B Test Tracking Setup (Integrating with Phase 5D Playbook)

The A/B Test Playbook (Phase 5D) defines 20+ tests across landing pages, emails, ads, scorecard, pricing page, and webinar. This section specifies the tracking infrastructure to measure those tests.

#### Test Event Taxonomy

| Event | Trigger | Parameters | Purpose |
|-------|---------|------------|---------|
| `ab_test_impression` | User is assigned to a test variant | `test_id`, `test_name`, `variant` (`control`, `variant_a`, `variant_b`), `page_path`, `user_segment` | Variant exposure tracking |
| `ab_test_conversion` | User completes the primary metric action for this test | `test_id`, `test_name`, `variant`, `conversion_type`, `conversion_value` | Primary metric measurement |
| `ab_test_secondary_metric` | User completes a secondary metric action | `test_id`, `test_name`, `variant`, `metric_name`, `metric_value` | Secondary analysis |

#### Integration with A/B Test Playbook Tests

Each test from the playbook maps to specific conversion events:

| Test ID | Test Name | Primary Metric Event | Secondary Metric Events |
|---------|-----------|---------------------|------------------------|
| LP-1 | Headline: Empathy vs. Fear-Stat | `quiz_start` (assessment start rate) | `page_scroll_50`, `page_scroll_75`, `page_time_on_page` |
| LP-2 | CTA: Professional vs. Peer-Framed | `cta_click` (CTA button click rate) | `discovery_call_booked`, `page_bounce` |
| LP-3 | Social Proof: Plain vs. Facebook-Style | `cta_click` (post-social-proof CTA) | `page_section_view` (social proof section), `page_scroll_75` |
| LP-4 | Problem Section: Long vs. Short | `page_section_view` (solution section reached) | `quiz_start`, `page_time_on_page` |
| LP-5 | Page Order: Testimonials Above Offer | `quiz_start` (assessment start rate) | `page_scroll_75`, `cta_click` |
| EM-* | Email subject line and format tests | `email_opened`, `email_clicked` | `quiz_start`, `discovery_call_booked`, `consultation_booked` |
| AD-* | Ad creative and copy tests | `ad_click` (CTR), `form_submit` (CPL) | `quiz_start`, `lead_magnet_download` |
| SC-* | Scorecard/quiz flow tests | `quiz_complete` (completion rate) | `quiz_results_cta_click`, `discovery_call_booked` |
| PR-* | Pricing page tests | `pricing_page_cta_click` | `consultation_booked`, `discovery_call_booked` |
| WB-* | Webinar tests | `webinar_reg_complete` (registration rate) | `webinar_join_live`, `webinar_pitch_cta_click` |

#### Statistical Rigor Requirements

| Parameter | Requirement |
|-----------|-------------|
| Minimum sample size per variant | As specified in each test (typically 400-500 unique visitors for landing page tests, 1,000+ emails for email tests) |
| Statistical significance threshold | 95% confidence (p < 0.05) |
| Minimum test duration | 2 full business weeks (to account for day-of-week variance) |
| Maximum test duration | 8 weeks (if significance not reached, the test is inconclusive -- do not keep running indefinitely) |
| Segmentation | All tests must be analyzable by traffic source, device type, and audience segment |
| SRM check | Sample Ratio Mismatch check at 48 hours and weekly thereafter. If variant allocation deviates > 2% from target split, investigate before trusting results. |
| Multiple test isolation | Do not run overlapping tests on the same page element. Run one test per page section at a time. Sequential, not parallel, on the same audience. |

#### Test Result Logging

Every completed test is logged to a persistent test results database (Google Sheet or Airtable) with:

```
{
  "test_id": "LP-1",
  "test_name": "Headline: Empathy vs. Fear-Stat",
  "start_date": "2026-03-15",
  "end_date": "2026-04-05",
  "total_visitors": 1124,
  "control_visitors": 558,
  "variant_visitors": 566,
  "control_conversions": 187,
  "variant_conversions": 248,
  "control_rate": "33.5%",
  "variant_rate": "43.8%",
  "lift": "+30.7%",
  "confidence": "97.2%",
  "result": "winner_variant",
  "implementation_date": "2026-04-06",
  "notes": "Fear-stat headline significantly outperformed empathy. Rolled out to all PA landing pages."
}
```

---

## 7. Data Layer Specification

### GTM Data Layer Architecture

All custom tracking flows through the Google Tag Manager data layer. Every page push begins with a base data layer object, and events push additional variables as they fire.

#### Base Data Layer (Fires on Every Page Load)

```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  'event': 'page_data_ready',

  // Page Information
  'page_type': 'landing_page',           // landing_page, blog, assessment, careers, pricing, webinar_reg, comparison, homepage
  'page_funnel': 'pa',                   // pa, dr, pt, sp, brand, none
  'page_audience': 'practice_owner',     // practice_owner, associate_dentist, patient_general, patient_implant, specialist, all
  'page_practice': '',                   // Practice name if practice-specific page (e.g., 'innovative_dental')
  'page_region': 'southeast',            // geographic region for practice pages

  // User Information
  'user_id': '',                         // CRM contact ID if known (from cookie or login)
  'user_logged_in': false,               // true if authenticated
  'user_type': 'anonymous',              // anonymous, lead, mql, sql, customer, partner
  'user_funnel_stage': 'awareness',      // awareness, consideration, decision, customer, churned

  // UTM Passthrough (parsed from URL on landing)
  'utm_source': '',
  'utm_medium': '',
  'utm_campaign': '',
  'utm_content': '',
  'utm_term': '',

  // Session Information
  'session_landing_page': '',            // First page in session
  'session_referrer': '',                // document.referrer
  'session_device': 'desktop',           // desktop, mobile, tablet
  'session_new_visitor': true            // First visit or returning
});
```

#### Quiz/Assessment Data Layer Events

```javascript
// Quiz Start
window.dataLayer.push({
  'event': 'quiz_start',
  'quiz_type': 'practice_assessment',     // practice_assessment, smile_assessment
  'quiz_traffic_source': 'google_cpc',
  'quiz_page_path': '/partners/assessment'
});

// Quiz Step Completion
window.dataLayer.push({
  'event': 'quiz_step_complete',
  'quiz_type': 'practice_assessment',
  'quiz_step_number': 3,
  'quiz_step_name': 'financial_position_revenue_trends',
  'quiz_step_answer': 'growing_5_plus_percent',     // Sanitized answer value
  'quiz_step_time_seconds': 18
});

// Quiz Completion
window.dataLayer.push({
  'event': 'quiz_complete',
  'quiz_type': 'practice_assessment',
  'quiz_total_score': 78,
  'quiz_tier': 'growth',
  'quiz_category_scores': {
    'operational_readiness': 22,
    'financial_position': 18,
    'growth_potential': 24,
    'personal_readiness': 14
  },
  'quiz_completion_time_seconds': 234
});

// Quiz Results CTA Click
window.dataLayer.push({
  'event': 'quiz_results_cta_click',
  'quiz_type': 'practice_assessment',
  'quiz_tier': 'growth',
  'quiz_cta_type': 'download_guide'
});
```

#### Form Data Layer Events

```javascript
// Form Start
window.dataLayer.push({
  'event': 'form_start',
  'form_name': 'discovery_call_booking',
  'form_location': 'assessment_results_page',
  'form_funnel': 'pa'
});

// Form Submission
window.dataLayer.push({
  'event': 'form_submit',
  'form_name': 'discovery_call_booking',
  'form_funnel': 'pa',
  'form_lead_source': 'assessment_strong_tier',
  'form_fields_completed': 5
});
```

#### Video Data Layer Events (VSL)

```javascript
// VSL Play
window.dataLayer.push({
  'event': 'vsl_play',
  'vsl_id': 'pa_partnership_overview',
  'vsl_auto_play': false,
  'vsl_page_path': '/partners'
});

// VSL Progress Milestone
window.dataLayer.push({
  'event': 'vsl_progress',
  'vsl_id': 'pa_partnership_overview',
  'vsl_progress_percent': 50,
  'vsl_video_duration_seconds': 420
});

// VSL CTA Click
window.dataLayer.push({
  'event': 'vsl_cta_click',
  'vsl_id': 'pa_partnership_overview',
  'vsl_cta_type': 'book_discovery_call',
  'vsl_click_time_seconds': 380,
  'vsl_progress_at_click': 90
});
```

#### Chatbot Data Layer Events

```javascript
// Chatbot Start
window.dataLayer.push({
  'event': 'chatbot_start',
  'chatbot_page_path': '/partners',
  'chatbot_trigger_type': 'user_initiated',
  'chatbot_funnel': 'pa'
});

// Chatbot Qualification
window.dataLayer.push({
  'event': 'chatbot_qualified',
  'chatbot_conversation_id': 'conv_12345',
  'chatbot_qualification_score': 75,
  'chatbot_audience_type': 'practice_owner',
  'chatbot_recommended_action': 'book_discovery_call'
});

// Chatbot Conversion
window.dataLayer.push({
  'event': 'chatbot_conversion',
  'chatbot_conversation_id': 'conv_12345',
  'chatbot_conversion_type': 'assessment_start',
  'chatbot_messages_to_conversion': 6,
  'chatbot_duration_seconds': 180
});
```

#### A/B Test Data Layer Events

```javascript
// Test Variant Assignment
window.dataLayer.push({
  'event': 'ab_test_impression',
  'ab_test_id': 'LP-1',
  'ab_test_name': 'headline_empathy_vs_fear_stat',
  'ab_test_variant': 'variant_a',
  'ab_test_page_path': '/partners'
});

// Test Conversion
window.dataLayer.push({
  'event': 'ab_test_conversion',
  'ab_test_id': 'LP-1',
  'ab_test_name': 'headline_empathy_vs_fear_stat',
  'ab_test_variant': 'variant_a',
  'ab_test_conversion_type': 'assessment_start',
  'ab_test_conversion_value': 1
});
```

#### E-commerce / Transaction Data Layer (Patient Treatments)

```javascript
// Treatment Accepted (for revenue tracking)
window.dataLayer.push({
  'event': 'treatment_accepted',
  'ecommerce': {
    'transaction_id': 'TRT-2026-00451',
    'value': 28500,
    'currency': 'USD',
    'items': [{
      'item_name': 'Full Arch Implant Restoration',
      'item_category': 'implant',
      'item_brand': 'innovative_dental',
      'price': 28500,
      'quantity': 1
    }]
  },
  'treatment_practice': 'innovative_dental',
  'treatment_financing': 'monthly_payments',
  'treatment_lead_source': 'facebook_paid'
});
```

#### Scroll Tracking Data Layer

```javascript
// Section View (landing page sections scroll into viewport)
window.dataLayer.push({
  'event': 'page_section_view',
  'section_name': 'social_proof',         // hero, problem, solution, social_proof, pillars, how_it_works, faq, final_cta
  'section_position': 4,                  // Ordinal position on page
  'page_path': '/partners',
  'time_to_section_seconds': 28           // Time from page load to section view
});
```

#### Competitor Comparison Page Data Layer

```javascript
// Comparison Page Engagement
window.dataLayer.push({
  'event': 'competitor_page_engagement',
  'comparison_type': 'heartland',          // heartland, aspen, pds, general_dso
  'engagement_type': 'scroll_depth',
  'scroll_depth_percent': 75,
  'time_on_page_seconds': 145
});

// Comparison Page CTA Click
window.dataLayer.push({
  'event': 'competitor_cta_click',
  'comparison_type': 'heartland',
  'cta_position': 'mid_page',
  'cta_text': 'See How SGA Is Different'
});
```

### GTM Container Structure

Organize GTM with the following folder structure:

| Folder | Contains | Purpose |
|--------|----------|---------|
| `01 - Page Tracking` | GA4 page_view tag, base data layer variables | Core page analytics |
| `02 - Scroll and Section Tracking` | Scroll depth triggers, section visibility triggers | Engagement depth |
| `03 - Quiz and Assessment` | Quiz start/step/complete/abandon tags | Quiz funnel tracking |
| `04 - Forms` | Form start/submit/abandon tags | Lead capture tracking |
| `05 - Video (VSL)` | VSL play/progress/CTA tags | Video engagement |
| `06 - Chatbot` | Chatbot event tags | Chatbot tracking |
| `07 - Webinar` | Webinar registration/attendance/replay tags | Webinar funnel |
| `08 - Email` | Email link click decorators, UTM passthrough | Email attribution |
| `09 - A/B Tests` | Test impression/conversion tags | Test measurement |
| `10 - Conversions` | GA4 conversion events, Meta CAPI, Google Ads conversion tags | Cross-platform conversion firing |
| `11 - Advertising` | Google Ads remarketing, Meta Pixel, LinkedIn Insight Tag | Ad platform pixels |
| `12 - Third Party` | Hotjar, Clarity, CallRail, Drift/Intercom | Third-party tool tags |

### GTM Custom Variables

| Variable Name | Type | Purpose |
|---------------|------|---------|
| `dlv_page_funnel` | Data Layer Variable | Funnel prefix from base data layer |
| `dlv_page_audience` | Data Layer Variable | Audience type from base data layer |
| `dlv_user_type` | Data Layer Variable | User lifecycle stage |
| `dlv_utm_source` | Data Layer Variable | UTM source passthrough |
| `dlv_utm_campaign` | Data Layer Variable | UTM campaign passthrough |
| `dlv_quiz_type` | Data Layer Variable | Quiz identifier |
| `dlv_quiz_score` | Data Layer Variable | Quiz total score |
| `dlv_quiz_tier` | Data Layer Variable | Quiz scoring tier |
| `dlv_ab_test_id` | Data Layer Variable | Active A/B test ID |
| `dlv_ab_test_variant` | Data Layer Variable | Assigned variant |
| `js_scroll_depth` | Custom JavaScript | Calculates scroll depth percentage |
| `js_time_on_page` | Custom JavaScript | Calculates seconds since page load |
| `js_is_returning_visitor` | 1st Party Cookie | Checks for returning visitor cookie |
| `lu_page_type` | Lookup Table | Maps URL patterns to page types |
| `lu_funnel_prefix` | Lookup Table | Maps URL patterns to funnel prefixes |

---

## 8. Reporting Cadence

### Daily (Automated Alerts)

**Who receives:** Marketing manager, media buyer
**Delivery:** Slack channel (#sga-marketing-alerts) + email digest at 8:00 AM ET
**Format:** Automated alert from Looker Studio or Databox

| Check | Trigger Threshold | Action |
|-------|-------------------|--------|
| Ad spend pacing | Spend > 120% or < 80% of daily budget | Investigate campaign pacing, pause overspending ad sets |
| CPL spike | CPL > 150% of 7-day average for any campaign | Check ad fatigue, audience saturation, landing page issues |
| Conversion drop | Conversions < 50% of 7-day daily average | Check tracking (broken pixel? form error?), landing page status, ad delivery |
| Email bounce spike | Bounce rate > 3% on any send | Investigate list hygiene, ESP authentication |
| Website down/error | 5xx error rate > 1% or uptime alert | Immediately notify dev team |
| Lead response time | Any lead not contacted within 5 minutes (LSA) or 1 hour (form) | Alert BD team or practice front desk |

### Weekly (Monday Marketing Meeting)

**Who attends:** CMO, marketing manager, media buyer, content strategist
**Delivery:** Looker Studio dashboard + 15-minute standup
**Format:** Dashboard review with week-over-week comparison

| Report Section | Key Metrics Reviewed |
|----------------|---------------------|
| Funnel summary | Leads, MQLs, SQLs, conversions -- by funnel, WoW change |
| Paid media | Spend, CPL, CTR, conversion rate -- by platform, WoW change |
| Email performance | Sends, opens, clicks, unsubscribes -- by active sequence |
| Content/SEO | Organic sessions, new blog posts published, keyword movements |
| Pipeline | Discovery calls booked (PA/SP), consultations booked (PT), applications (DR) |
| A/B tests | Active test status, sample progress toward significance |
| Budget | WTD spend vs. budget, projected monthly spend |
| Action items | Top 3 actions for the coming week |

### Monthly (First-of-Month Deep Dive)

**Who attends:** CMO, CEO, VP Marketing, VP Business Development, marketing team
**Delivery:** Looker Studio + slide deck summary (5-7 slides)
**Format:** 45-minute review meeting

| Report Section | Key Metrics Reviewed |
|----------------|---------------------|
| Executive summary | Total spend, total leads, total conversions, blended CAC, pipeline value |
| Funnel-by-funnel performance | Full funnel metrics for PA, DR, PT, SP -- MoM trend |
| Channel ROI analysis | CPL, conversion rate, ROAS -- by channel -- ranked by efficiency |
| Attribution analysis | First-touch vs. last-touch vs. position-based comparison, path analysis |
| Email health | Sequence performance, list growth, deliverability, revenue attribution |
| SEO progress | Rankings movement, organic traffic trend, content pipeline |
| Social media | Follower growth, engagement rates, content performance |
| A/B test results | Completed tests, implemented winners, cumulative lift impact |
| Competitive landscape | Competitor ad activity, share of voice, ranking changes |
| Budget performance | Actual vs. planned spend, reallocation recommendations |
| Next month plan | Planned campaigns, tests, content, budget adjustments |

### Quarterly (Strategic Review)

**Who attends:** CEO, CMO, CFO, VP Marketing, VP Business Development, Thurston Group representative (if applicable)
**Delivery:** Formal slide deck (15-20 slides) + supporting data appendix
**Format:** 90-minute strategic review

| Report Section | Key Metrics Reviewed |
|----------------|---------------------|
| Business impact summary | Partnerships signed (PA/SP), positions filled (DR), new patient volume (PT), revenue attributed to marketing |
| Marketing-to-revenue attribution | Full path from marketing spend to closed revenue, by funnel |
| CAC and LTV analysis | Customer acquisition cost trends, estimated lifetime value by segment |
| Channel portfolio review | Which channels to scale, maintain, reduce, or cut -- based on 90 days of data |
| Funnel optimization summary | Cumulative conversion improvements from A/B testing, sequence optimization |
| Competitive positioning | Market share movement, competitive wins/losses, SOV changes |
| SEO authority growth | Domain rating trend, keyword ranking portfolio, organic traffic trajectory |
| Content performance | Top-performing assets, content gaps, production efficiency |
| Technology/tool assessment | Tool stack performance, integration health, data quality audit |
| Budget recommendation | Next quarter budget allocation proposal with ROI projections |
| Strategic initiatives | New channels to test, market expansion plans, partnership opportunities |

### Annual Review

**Timing:** January (reviewing prior year)
**Format:** Comprehensive annual marketing performance report

- Year-over-year comparison on all KPIs
- Marketing contribution to business growth (practices acquired, positions filled, patients gained)
- Full attribution model validation (does the model match reality? adjust if needed)
- Tool stack audit and vendor review
- Budget allocation vs. actual performance -- inform next year's budget
- Competitive landscape evolution
- Strategic plan for next year

---

## Appendix A: Tracking Checklist for Launch

Before any campaign, landing page, email sequence, or asset goes live, verify:

| Check | Status |
|-------|--------|
| UTM parameters follow the naming convention | [ ] |
| UTM parameters are correctly appended to all links | [ ] |
| GA4 page_view firing on the page | [ ] |
| Base data layer pushing correct values | [ ] |
| GTM tags firing correctly (use GTM Preview mode) | [ ] |
| All form submissions triggering `form_submit` event with correct parameters | [ ] |
| Conversion events mapped to GA4 conversions | [ ] |
| Ad platform pixels firing (Meta Pixel, LinkedIn Insight Tag, Google Ads tag) | [ ] |
| Call tracking number installed (for phone-based CTAs) | [ ] |
| CRM integration receiving leads with UTM data preserved | [ ] |
| A/B test variant assignment working correctly (if applicable) | [ ] |
| Cross-domain tracking configured (if user crosses domains) | [ ] |
| Cookie consent banner working and blocking tags appropriately | [ ] |
| Dashboard updated with new campaign/asset data sources | [ ] |

## Appendix B: Tag Naming Convention for GTM

All GTM tags follow: `{type} - {platform} - {event description}`

Examples:
- `GA4 - Event - quiz_start`
- `GA4 - Event - form_submit`
- `GA4 - Event - vsl_progress`
- `Meta - Pixel - Lead`
- `Meta - CAPI - Purchase`
- `Google Ads - Conversion - discovery_call_booked`
- `LinkedIn - Event - lead_gen_form_submit`
- `Hotjar - Trigger - assessment_page`
- `CallRail - Swap - practice_phone_number`

## Appendix C: Privacy and Compliance

| Requirement | Implementation |
|-------------|---------------|
| Cookie consent | CMP (Cookiebot or OneTrust) integrated with GTM consent mode |
| GA4 consent mode | Implemented -- tags fire in restricted mode until consent granted |
| Data retention | GA4 set to 14-month retention (maximum for free tier) |
| PII handling | No PII in data layer values, event parameters, or custom dimensions. Email addresses hashed (SHA-256) for cross-platform matching only. |
| HIPAA consideration | Patient health information (PHI) never enters the analytics stack. Quiz answers about dental conditions are stored in the HIPAA-compliant CRM only, not in GA4. Data layer quiz answer values use sanitized category labels, not free text. |
| Right to deletion | Process documented for removing user data from GA4, CRM, and ad platforms upon request |
| Data processing agreements | DPAs signed with Google, Meta, HubSpot, and any third-party tool that handles user data |

---

*Document total: ~7,800 words*
*Integrates with: Product Brief, Funnel Architecture (Phase 5B), A/B Test Playbook (Phase 5D)*
*Sources: Funnel architecture conversion events, A/B test primary/secondary metrics, channel benchmarks from funnel architecture success metrics, email sequence specifications, quiz/assessment structure from lead magnet definitions*
