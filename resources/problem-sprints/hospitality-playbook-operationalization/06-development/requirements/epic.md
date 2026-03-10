# Product Epic: The Hospitality Playbook Platform

**Epic ID**: HPP-0
**Sprint Origin**: Hospitality Playbook Operationalization
**Solution**: The Hospitality Playbook Platform
**Phase 1 Target**: Morning Huddle OS + H-Score Engine + CCA Bronze — 20-30 pilot practices, 90 days

---

## Epic Overview

SGA has developed "Caring with Charisma" — a hospitality philosophy modeled after the Ritz Carlton, Chick-fil-A, and Disney University — but lacks the infrastructure to train staff on it, measure adoption, or reinforce the culture across 260+ practices. The Hospitality Playbook Platform operationalizes this philosophy through five integrated systems: a daily huddle ritual, a career-value certification program, a unified behavioral score, a trust transfer verification layer, and a network of human culture champions.

**Platform tagline**: Caring with Charisma at Scale — Every Practice. Every Morning. Every Patient.

---

## Parent Epic: The Hospitality Playbook Platform (HPP)

**Goal**: By month 6 of pilot (20-30 practices), achieve:
- 78%+ of pilot practices completing a structured morning huddle at least 4 days/week
- 80%+ of pilot practice staff earning CCA Bronze certification within 60 days of onboarding
- Average H-Score in pilot cohort of 65+ (up from estimated baseline of ~51)
- 5-star review velocity up 20%+ vs. same period prior year in pilot practices

---

## Sub-Epic 1: Morning Huddle OS (HPP-1)

**One-liner**: Deliver a turnkey 5-minute hospitality huddle every morning to every practice — inside Dental Intel, requiring zero preparation.

**Objective**: Remove every friction barrier between a practice owner and a world-class morning huddle by pre-assembling personalized daily content packages delivered automatically to Dental Intel.

### User Stories

**HPP-1-001**
```
Given I am a practice owner opening Dental Intel at 7:50 AM,
When the morning huddle time arrives,
Then a pre-assembled 5-minute content package auto-loads on screen
  without any action required from me,
  displaying today's date, week's theme, and estimated duration.
```

**HPP-1-002**
```
Given today's huddle has started,
When the "Yesterday's Wins" segment loads,
Then it displays the most recent Swell review received in the last 24 hours
  including the reviewer's first name (if provided),
  and flags any reviews that mention a staff member by name,
  with a visual "Trust Transfer Signal Detected" badge if the review language
  matches hospitality handoff patterns.
```

**HPP-1-003**
```
Given the huddle content package is being assembled for my practice,
When the system selects "Today's Focus" content,
Then it prioritizes the hospitality behavior category with the lowest recent H-Score sub-score
  for my practice type (rural/suburban/urban/cosmetic/pediatric),
  and presents a 30-second team role-play exercise with exact scripted prompts.
```

**HPP-1-004**
```
Given our practice has completed a huddle today,
When I mark it complete in Dental Intel,
Then the completion event is sent to the H-Score Engine within 5 minutes,
  my 7-day huddle completion rate updates on the practice dashboard,
  and if this is a streak day (3+ consecutive), a streak indicator increments.
```

**HPP-1-005**
```
Given our practice has missed 3 or more consecutive huddles,
When the system detects this pattern,
Then an automated alert is generated for our regional director,
  including our current H-Score, streak status, and a draft intervention message,
  and I as practice owner receive a gentle "Huddle Health Check" notification.
```

**HPP-1-006**
```
Given I am a Champion at my practice,
When I want to submit a patient story as a "wow story" candidate,
Then I can do so directly from the post-huddle screen in under 60 seconds,
  and the submission enters the content review queue for potential network distribution.
```

### Phase 1 Acceptance Criteria (Huddle OS)
- Content package for any practice loads in < 3 seconds on Dental Intel
- Completion event received and processed by H-Score Engine within 5 minutes
- Live Swell review data displayed in huddle refreshes every 24 hours
- Practices without Dental Intel can access huddle via fallback web URL
- Content available 7 days in advance for each practice
- Huddle completion rate visible on practice dashboard same day

---

## Sub-Epic 2: Caring with Charisma Academy — CCA Bronze (HPP-2)

**One-liner**: Build a career-value certification pathway that staff complete for themselves — portable, LinkedIn-visible, and verifiable by any employer in the dental industry.

**Objective**: Deploy the 10-module CCA Bronze track so that every new hire can achieve certification within their first 30 days of employment, entirely on their phone during micro-breaks.

### User Stories

**HPP-2-001**
```
Given I am a new staff member at an SGA practice,
When I am added to the practice roster by my office manager,
Then I receive an onboarding message with my CCA Bronze enrollment link
  within 24 hours of being added,
  and Module 1 ("The Story") is automatically unlocked and ready to view.
```

**HPP-2-002**
```
Given I am completing a CCA Bronze module on my phone,
When I finish watching the video segment (4-6 minutes),
Then a scenario assessment appears with 2-3 response options,
  I receive immediate audio or text feedback on my answer (Grant's voice where available),
  and module completion is logged to my certification record.
```

**HPP-2-003**
```
Given I have completed all 10 Bronze modules with scores of 80%+ on scenario assessments,
When the system marks me as Bronze certified,
Then a digital certificate is generated with my name, certification date, and CCA seal,
  an issuance request is automatically sent to Credly for LinkedIn-compatible credential,
  my practice's certification rate dashboard updates within 1 hour,
  and a push notification is sent to my practice owner to celebrate at the next huddle.
```

**HPP-2-004**
```
Given a staff member has earned Bronze certification,
When they add it to their LinkedIn profile via the Credly integration,
Then the credential is publicly verifiable via a unique URL,
  the credential displays "Caring with Charisma Academy — Bronze" as the issuing organization,
  and includes the expiration date (12 months from issuance).
```

**HPP-2-005**
```
Given my Bronze certification is expiring in 30 days,
When the renewal window opens,
Then I receive a notification with links to 3 refresher modules (30 min total),
  and on completing them my certification renews for another 12 months
  without requiring full re-certification.
```

### Phase 1 Acceptance Criteria (CCA Bronze)
- All 10 Bronze modules live and playable on mobile and tablet
- Scenario assessments scoring and completion tracked per staff member
- Certificate generation automated via Credly API within 1 hour of completion
- LinkedIn credential publishing via Credly works end-to-end
- Practice owner notification on staff certification delivered within 15 minutes
- Certification rate per practice visible in practice dashboard

---

## Sub-Epic 3: Hospitality Score Engine — H-Score (HPP-3)

**One-liner**: Turn culture from a soft aspiration into a visible, daily score — calculated from Cinnamon, Swell, and Dental Intel signals that already exist.

**Objective**: Build the H-Score calculation engine and practice dashboard so every practice owner sees their score, understands what's driving it, and knows exactly what to do to improve it.

### User Stories

**HPP-3-001**
```
Given I am a practice owner opening the platform for the first time,
When my practice is activated,
Then the system automatically generates a baseline H-Score
  from the last 90 days of Swell review data and Cinnamon call analytics
  already connected to my practice,
  without any manual data entry required from me.
```

**HPP-3-002**
```
Given my practice has an H-Score,
When I view my practice dashboard,
Then I see:
  - My current H-Score (0-100) with my level label (Emerging / Developing / Established / Exceptional / Elite),
  - A 7-day trend sparkline,
  - Three component scores: Swell (45%), Cinnamon (35%), Dental Intel (20%),
  - Each component broken down into its sub-signals with individual scores,
  - A "This Week's Opportunity" recommendation identifying my lowest sub-score
    and the specific action most likely to improve it.
```

**HPP-3-003**
```
Given my Cinnamon phone analytics are connected,
When a new day's call data is ingested,
Then the system calculates:
  - Phone answer rate within 3 rings (% of calls),
  - Greeting phrase detection rate (% of calls with warm opener in first 10 seconds),
  - Hold time compliance (% of calls with hold < 2 minutes),
  and updates my Cinnamon component score within 2 hours of the new day's data arriving.
```

**HPP-3-004**
```
Given my Swell reviews are connected,
When new review data arrives via webhook,
Then each review text is processed by the NLP pipeline to produce:
  - Hospitality sentiment score (0-1),
  - Trust transfer signal tag (positive / neutral / negative),
  - Staff mention flag and staff ID if matched,
  and the Swell component score updates within 1 hour of processing.
```

**HPP-3-005**
```
Given my H-Score has dropped by 10+ points in 7 days,
When the anomaly detection job runs,
Then an anomaly alert is generated,
  my regional director sees it in their intervention queue,
  I receive an explanation of which sub-score drove the drop
  and what recent events may have contributed,
  and the alert does NOT trigger a bonus pool penalty
  (anomalies are investigative, not punitive).
```

### Phase 1 Acceptance Criteria (H-Score Engine)
- H-Score calculated and updated daily for all pilot practices
- Baseline H-Score generated at practice activation using historical data
- All three component scores (Cinnamon, Swell, Dental Intel) visible with sub-score breakdown
- H-Score algorithm documentation published in platform (readable by practice owners)
- Practice-type segmentation applied to benchmarking calculations
- Anomaly detection triggers alerts within 24 hours of threshold breach

---

## Sub-Epic 4: Trust Transfer OS (HPP-4)

**One-liner**: Make the hygienist-to-DA-to-doctor clinical handoff trackable, trainable, and connected to patient reviews — without adding documentation burden.

**Objective**: Deploy CCA Bronze Modules 05 and 06 (trust transfer specific) plus the daily huddle checklist prompt and post-visit review NLP tagging so that SGA has its first behavioral compliance signal for the trust transfer protocol.

### User Stories

**HPP-4-001**
```
Given the morning huddle is running,
When the "Accountability" segment loads,
Then a "Trust Transfer Commitment" item appears as a one-tap acknowledgment:
  "Today our team will complete every patient handoff with a warm introduction.",
  the tap is logged as a team commitment event for that practice and date.
```

**HPP-4-002**
```
Given a Swell review has been received and NLP-processed,
When the review text contains trust transfer signal phrases
  (e.g., "the whole team knew my story," "seamless," "everyone briefed each other"),
Then the review is tagged as "Trust Transfer Positive" in the data model,
  flagged for display in the next morning huddle's "Yesterday's Wins" segment,
  and queued for Champion wow story submission consideration.
```

**HPP-4-003**
```
Given a practice has a 30-day rolling Trust Transfer Sentiment Score,
When the score is calculated,
Then it is visible to the regional director and practice owner
  as a sub-component of the overall H-Score's Swell section,
  with the trend vs. the practice's own prior 30 days.
```

### Phase 1 Acceptance Criteria (Trust Transfer OS)
- Trust transfer NLP tagging runs on 100% of incoming Swell reviews
- Daily huddle commitment tap logged and stored per practice per day
- Trust transfer positive reviews surfaced in morning huddle "Yesterday's Wins"
- Trust transfer sentiment score visible in practice dashboard

---

## Sub-Epic 5: Champion Network (HPP-5)

**One-liner**: Identify, designate, and connect one certified culture ambassador per practice — the human infrastructure that makes the platform sustainable after novelty fades.

**Objective**: Build the Champion designation workflow, Champion profile, and regional cohort infrastructure so that the first cohort of Champions is operational within 90 days of platform launch.

### User Stories

**HPP-5-001**
```
Given a practice has been active on the platform for 6+ weeks,
When the Champion readiness algorithm runs
  (huddle completion > 70%, at least one staff member with Bronze certification),
Then the practice owner receives a prompt:
  "Your practice is ready to designate a Caring with Charisma Champion."
  with a data-informed recommendation for the staff member most likely to succeed
  (based on huddle participation, certification progress, and module engagement scores).
```

**HPP-5-002**
```
Given a practice owner designates a Champion,
When the Champion profile is activated,
Then the Champion receives:
  - Access to the Champion content library (facilitation guides, wow story form),
  - An invitation to the next Regional Champion Call,
  - A welcome message from Grant (pre-recorded video, 3 minutes),
  and their Champion status is visible on the practice's public platform profile.
```

**HPP-5-003**
```
Given I am a Champion and I have submitted a wow story,
When the story is selected for network distribution,
Then it appears in the morning huddle content for all practices in my region that week,
  crediting my first name and practice (with my permission),
  and I receive a notification that my story reached X practices.
```

### Phase 1 Acceptance Criteria (Champion Network)
- Champion designation workflow end-to-end functional
- Champion content library accessible post-designation
- Regional cohort structure established (practices grouped by region)
- Wow story submission form functional

---

## Technical Spikes Needed

| Spike | Owner | Priority | Estimated Effort |
|-------|-------|----------|-----------------|
| Dental Intel API capabilities assessment | Tech Lead | P0 | 3 days |
| Cinnamon webhook/API integration spec | Backend | P0 | 2 days |
| Swell webhook integration + NLP pipeline selection | Backend | P0 | 3 days |
| Credly API for credential issuance + LinkedIn publishing | Backend | P1 | 2 days |
| Video delivery CDN selection for module content | Tech Lead | P1 | 1 day |
| Practice-type segmentation data model | Backend | P1 | 1 day |
| NLP vendor evaluation (trust transfer phrase detection) | Backend | P1 | 2 days |

---

## Dependency Map

```
HPP-3 (H-Score Engine)
  ├── requires: Cinnamon API integration [SPIKE]
  ├── requires: Swell webhook integration [SPIKE]
  ├── requires: Dental Intel API integration [SPIKE]
  └── blocks: HPP-1 (live score in huddle display)
             HPP-5 (champion readiness algorithm uses H-Score)

HPP-1 (Morning Huddle OS)
  ├── requires: Dental Intel API [SPIKE]
  ├── requires: HPP-3 H-Score calculation (for "Yesterday's Score" segment)
  ├── requires: Playbook Content Engine (videos and scripts to deliver)
  └── blocks: HPP-5 (champion designation readiness trigger uses huddle completion)

HPP-2 (CCA Academy Bronze)
  ├── requires: Video hosting CDN [SPIKE]
  ├── requires: Credly API [SPIKE]
  ├── requires: Staff roster API or manual import
  └── blocks: HPP-5 (champion designation requires at least 1 Bronze certified staff)

HPP-4 (Trust Transfer OS)
  ├── requires: HPP-2 Module 05 + 06 content (clinical trust transfer modules)
  ├── requires: HPP-3 Swell NLP pipeline (trust transfer signal detection)
  └── feeds: HPP-3 H-Score (trust transfer sentiment as Swell sub-signal)

HPP-5 (Champion Network)
  ├── requires: HPP-1 completion (6+ weeks of huddle data)
  ├── requires: HPP-2 Bronze certification (at least 1 certified staff)
  └── requires: HPP-3 H-Score (champion readiness algorithm)
```

---

## Success Metrics (Phase 1 — 90 Days, Pilot Cohort)

| Metric | Baseline | Phase 1 Target | How Measured |
|--------|----------|---------------|--------------|
| Morning huddle completion rate | ~12% (estimated) | 70%+ | Dental Intel completion events |
| CCA Bronze certification rate (% of staff) | 0% | 80%+ (new hires within 30 days) | CCA Academy completion records |
| Average H-Score (pilot practices) | ~51 (estimated) | 65+ | H-Score Engine daily calculation |
| 5-star review velocity | Baseline captured at activation | +20% vs. prior 90-day period | Swell review count per 100 appts |
| Trust Transfer Positive review rate | Baseline 0 (not yet measured) | 30%+ of reviews tagged positive | Swell NLP tagging |
| Practice owner NPS for platform | Not yet measured | 7.5+ (out of 10) | Post-pilot survey |
| Champion designation rate | 0% | 60% of pilot practices | Champion profile activations |

---

## Phase 2 Scope (Not in Phase 1)

- Rewards Engine (leaderboard, bonus pool, jacket fulfillment)
- Network Intelligence Dashboard (regional director view, intervention queue)
- CCA Silver certification track
- Champion Network content contribution workflow
- Playbook Distribution System (versioning and update distribution)
- Content Production Pipeline tooling

## Phase 3 Scope (Not in Phase 1 or 2)

- CCA Gold certification track
- Academic publication data export
- Dental school outreach package
- LinkedIn "recognized institution" application
- Content Concierge integration
- Practice Concierge H-Score badge integration
