# Product Architecture: The Hospitality Playbook Platform

## Platform Overview

**Name**: The Hospitality Playbook Platform
**Tagline**: Operationalizing "Caring with Charisma" across 260+ practices
**Core Thesis**: Culture cannot be mandated — it must be designed, delivered, measured, and reinforced as a living system. The Hospitality Playbook Platform is the infrastructure that makes "Caring with Charisma" as operational as scheduling software and as measurable as production metrics.

**Architecture Philosophy**: The platform is built around a **flywheel**, not a funnel. Each component feeds the next: content trains behavior, behavior is tracked and scored, scores drive motivation, motivation produces better reviews, reviews generate recognition, recognition reinforces the culture that motivates more training. There is no isolated component — all ten systems share data, surface insights, and compound over time.

---

## System Diagram (Conceptual)

```
                        ┌──────────────────────────────────────────┐
                        │         SGA PLAYBOOK PLATFORM             │
                        │                                            │
    ┌─────────────┐     │  ┌─────────────┐    ┌──────────────────┐ │
    │  CINNAMON   │────▶│  │ HOSPITALITY  │    │  NETWORK INTEL   │ │
    │ Voice/Phone │     │  │ SCORE ENGINE │───▶│   DASHBOARD      │ │
    └─────────────┘     │  └──────┬───────┘    └──────────────────┘ │
                        │         │                      │           │
    ┌─────────────┐     │         ▼                      ▼           │
    │    SWELL    │────▶│  ┌─────────────┐    ┌──────────────────┐ │
    │  Reviews/NPS│     │  │  REWARDS &  │    │   PLAYBOOK       │ │
    └─────────────┘     │  │  RECOGNITION│    │  DISTRIBUTION    │ │
                        │  └──────┬───────┘    └──────────────────┘ │
    ┌─────────────┐     │         │                      │           │
    │ DENTAL INTEL│────▶│         ▼                      ▼           │
    │ Morning Hub.│     │  ┌─────────────┐    ┌──────────────────┐ │
    └─────────────┘     │  │  MORNING    │◀───│  PLAYBOOK        │ │
                        │  │  HUDDLE OS  │    │  CONTENT ENGINE  │ │
    ┌─────────────┐     │  └──────┬───────┘    └──────────────────┘ │
    │   VOICE     │     │         │                      ▲           │
    │ RECORDING   │────▶│         ▼                      │           │
    └─────────────┘     │  ┌─────────────┐    ┌──────────────────┐ │
                        │  │   CCA       │    │  CONTENT         │ │
                        │  │  ACADEMY   │───▶│  PRODUCTION      │ │
                        │  └──────┬───────┘    │  PIPELINE        │ │
                        │         │             └──────────────────┘ │
                        │         ▼                      ▲           │
                        │  ┌─────────────┐    ┌──────────────────┐ │
                        │  │   TRUST     │    │  CULTURE OS /    │ │
                        │  │  TRANSFER OS│    │ CHAMPION NETWORK │ │
                        │  └─────────────┘    └──────────────────┘ │
                        └──────────────────────────────────────────┘
```

---

## Component 1: Playbook Content Engine

### Responsibility
The authoritative repository of all "Caring with Charisma" content — videos, scripts, behavioral exercises, huddle segments, certification modules, and the playbook itself as a living document. This is the source of truth for what the culture is and how it is expressed.

### Interfaces
- Feeds: Morning Huddle OS (daily content delivery), CCA Academy (certification modules), Playbook Distribution System (version-controlled updates)
- Receives from: Content Production Pipeline (new videos, scripts), Culture OS (Champion-submitted practice stories)

### Data Produced / Consumed
- Produces: Content packages (video + script + exercise + discussion prompt), versioned playbook documents, topic rotation calendars
- Consumes: Content production outputs, Champion submissions, playbook version history, usage analytics (which segments have highest completion and engagement)

### Key Design Decision
The Playbook Content Engine is built as a **CMS with a behavior taxonomy** — every piece of content is tagged to a specific hospitality behavior (e.g., "warm greeting," "trust transfer — hygienist step," "exit experience"), a certification tier (Bronze/Silver/Gold), and a practice type (rural, urban, cosmetic, pediatric). This taxonomy allows the Morning Huddle OS to assemble personalized daily huddles by filtering content relevant to a given practice's profile and current focus area.

### Content Architecture
```
Content Library Structure:
├── Playbook Core (evergreen — the philosophy, the language system, the behaviors)
│   ├── The Caring with Charisma Story (Origin video — Grant)
│   ├── 5 Hospitality Domains (Phone, Arrival, Clinical, Trust Transfer, Exit)
│   ├── The 12 Hospitality Behaviors (specific, observable, trainable)
│   └── The C&C Language System (glossary: guests, wellness destination, etc.)
│
├── Morning Huddle Library (260 daily segments per year)
│   ├── Week 1-12: Foundation behaviors (rotation across 12 behaviors)
│   ├── Week 13-24: Intermediate (trust transfer, environmental mastery)
│   ├── Week 25-36: Advanced (patient journey mapping, clinical hospitality)
│   └── Week 37-52: Culture deepening + new cohort content
│
├── CCA Certification Modules (10 Bronze + 8 Silver + 6 Gold)
│   ├── Bronze: Foundational behaviors + scenario practice
│   ├── Silver: Advanced behaviors + peer evaluation + behavior data requirements
│   └── Gold: Leadership + facilitation + mentorship + practice culture ownership
│
└── Special Content
    ├── Grant's Weekly Culture Brief (10 min, monthly release)
    ├── Champion Facilitation Guides
    ├── Practice Story Archive (submitted wow stories)
    └── Academic / External Publication Assets
```

---

## Component 2: Morning Huddle OS

### Responsibility
The daily delivery mechanism for hospitality culture. Integrates with Dental Intel to present pre-assembled, personalized 5-minute huddle experiences to every practice every morning. Tracks completion, measures engagement, and feeds data to the H-Score.

### Interfaces
- Integrates with: Dental Intel API (display, completion tracking, staff roster)
- Receives from: Playbook Content Engine (today's content package), Hospitality Score Engine (live H-Score and yesterday's trend), Rewards Engine (leaderboard position, bonus pool status)
- Feeds: Hospitality Score Engine (huddle completion data), CCA Academy (completion of certification-linked huddle segments)

### Data Produced / Consumed
- Produces: Daily huddle completion records (practice ID, date, completion status, duration, segment completion breakdown), engagement metrics (which segments staff skipped vs. completed)
- Consumes: Content packages, practice H-Score, practice leaderboard position, Swell reviews from last 24 hours, current week's CCA certification focus

### Key Design Decision
The Huddle OS assembles **personalized daily packages** rather than broadcasting a single network-wide huddle. A rural 5-person practice gets a different content emphasis than an urban 30-person cosmetic practice — but both receive the same quality experience. Personalization inputs: practice type (rural/suburban/urban/cosmetic/pediatric), current week's H-Score component needing most improvement, staff certification stage, and regional context.

### Daily Huddle Structure
```
5-Minute Huddle Template:
│
├── 00:00–01:00 — CULTURE MOMENT
│   A Grant video or Champion-submitted story. Emotional hook first.
│   (Rotates: Grant direct × 2/week, Champion story × 2/week, Network story × 1/week)
│
├── 01:00–02:00 — YESTERDAY'S WINS
│   Live Swell data: reviews received in last 24 hours, any staff-mention highlights.
│   H-Score trend line: 7-day sparkline. One sentence on what moved it.
│
├── 02:00–03:30 — TODAY'S FOCUS
│   The week's hospitality behavior: specific protocol, scripted phrase, or clinical behavior.
│   30-second team practice exercise: paired role-play or group recitation.
│
├── 03:30–04:30 — ACCOUNTABILITY
│   Trust Transfer Champion of the Day (nominated or rotated).
│   CCA certification spotlight: anyone who completed a module this week.
│   Environmental check: 3-item opening checklist (waiting room, refreshments, ambient).
│
└── 04:30–05:00 — INTENTION
    Shared phrase read by facilitator, repeated by team.
    "Today, every patient leaves knowing their name matters here."
```

---

## Component 3: Caring with Charisma Academy (CCA)

### Responsibility
The certification engine and credentialing institution. Manages staff enrollment, module delivery, scenario assessment, tier progression, credential issuance, and the portable public certification record.

### Interfaces
- Integrates with: Playbook Content Engine (module content), Hospitality Score Engine (behavior data required for Silver/Gold criteria), Rewards Engine (certification milestone triggers)
- External: LinkedIn API (credential publication), public verification portal, dental school curriculum export
- Feeds: Network Intelligence Dashboard (certification rate per practice), Rewards Engine (badge/jacket unlock triggers)

### Data Produced / Consumed
- Produces: Certification records, module completion events, scenario assessment scores, tier progression history, issued credentials
- Consumes: Staff roster (from practice EHR/PMS), module completion events, H-Score behavior data (for Silver/Gold validation), Champion nominations

### Key Design Decision
CCA is designed as an **externally credentialed institution from day one** — not an internal training program that might later become a credential. All content is designed to be "curriculum-grade": academically structured, scenario-verified, and publishable. Credentials are issued with unique verification URLs, expiration dates (annual renewal), and public LinkedIn integration. This architecture decision makes the dental school adoption pathway viable without requiring a platform rebuild.

### Certification Tier Architecture
```
CCA BRONZE — "Certified Hospitality Practitioner"
  Requirements:
  ├── Complete 10 Bronze modules (avg 5 min each = ~50 min total)
  ├── Pass scenario assessments at 80% or higher for each module
  └── Attend or watch 5 morning huddle sessions (tracked via Dental Intel)

  Issued: Digital certificate + LinkedIn badge + staff profile green badge
  Renewal: Annual (re-take 3 refresher modules + current H-Score threshold)
  Time to complete: 2-4 weeks for motivated staff

CCA SILVER — "Senior Hospitality Practitioner"
  Requirements:
  ├── Hold Bronze for 6+ months
  ├── Complete 8 Silver modules (advanced behaviors + peer evaluation)
  ├── Practice H-Score has been 60+ for at least 60 of the past 90 days
  ├── Peer nomination from practice owner or existing C&C Champion
  └── Record and submit one "wow story" from practice

  Issued: Enhanced digital certificate + network staff directory listing + bonus pool multiplier
  Time to achieve: 6-12 months after Bronze

CCA GOLD — "C&C Champion / Elite Hospitality Leader"
  Requirements:
  ├── Hold Silver for 6+ months
  ├── Complete 6 Gold modules (leadership, facilitation, mentorship, culture ownership)
  ├── Practice H-Score has been 75+ for at least 60 of the past 90 days
  ├── Have facilitated morning huddle 100+ times (tracked)
  ├── Have onboarded at least 2 new staff through Bronze certification
  └── Panel review: short video interview with SGA culture team

  Issued: The Jacket (physical, ordered by SGA, delivered to practice), Gold certificate,
          Champion network seat, stipend for huddle facilitation, LinkedIn "Elite" badge
  Time to achieve: 18-36 months from start
```

### Module Architecture (Bronze Track)
```
Module 01: The Story — Why This Practice Is Different (Origin + philosophy)
Module 02: The Language System — How We Talk Here (Glossary + framing)
Module 03: The Warm Welcome — Arrival and First Impression
Module 04: The Phone as Hospitality — Scripted Phone Protocol
Module 05: The Trust Transfer — Part 1: Front Desk → Hygienist
Module 06: The Trust Transfer — Part 2: Hygienist → DA → Doctor
Module 07: The Environment — What Every Patient Sees Before They See You
Module 08: Handling Anxiety — When the Patient Is Afraid
Module 09: The Exit Experience — Closing the Circle
Module 10: The Score — How We Know It's Working (H-Score, Swell, Cinnamon)
```

---

## Component 4: Hospitality Score Engine (H-Score)

### Responsibility
Aggregates behavioral signals from Cinnamon, Swell, and Dental Intel into a single per-practice "H-Score" updated daily. Generates trend analysis, anomaly alerts, component breakdowns, and network benchmarking. This is the accountability spine of the entire platform.

### Interfaces
- Receives from: Cinnamon API (phone analytics), Swell API (reviews + NPS), Dental Intel API (huddle completion + certification data)
- Feeds: Morning Huddle OS (yesterday's score for daily display), Network Intelligence Dashboard (portfolio view), Rewards Engine (bonus pool trigger), CCA Academy (behavior data for Silver/Gold criteria)

### Data Produced / Consumed
- Produces: Daily H-Score per practice, component scores, trend lines, anomaly flags, network percentile rankings, component attribution reports
- Consumes: Cinnamon call data (graded per call), Swell review data (star rating + sentiment analysis), Dental Intel completion data

### Key Design Decision
The H-Score is designed to be **transparent, interpretable, and fair** — each component's contribution is visible and explainable, not a black box. Staff can see exactly which sub-score is lowest and what specific behaviors affect it. This transparency is critical for the score to be motivating rather than perceived as a surveillance instrument. The score is also **practice-type adjusted** — a rural practice with fewer patient touchpoints per day is scored against rural peer benchmarks, not the full network average.

### H-Score Algorithm
```
H-SCORE COMPONENTS AND WEIGHTS:

SWELL COMPONENT (45% weight)
├── 7-day star rating average: 20%
│   └── Score: Raw Swell rating normalized to 0-100
├── Review velocity (reviews per 100 patients): 15%
│   └── Score: Practice rate vs. practice-type cohort benchmark
└── Sentiment quality (hospitality language in reviews): 10%
    └── Score: NLP analysis detecting warm-greeting, trust, comfort language

CINNAMON COMPONENT (35% weight)
├── Phone answer rate within 3 rings: 15%
│   └── Score: % calls answered within 3 rings in trailing 7 days
├── Hospitality phrase detection: 12%
│   └── Score: % calls containing warm greeting phrase in first 10 seconds
└── Hold time compliance: 8%
    └── Score: % calls with hold time < 2 minutes

DENTAL INTEL COMPONENT (20% weight)
├── Huddle completion rate: 12%
│   └── Score: % days with completed huddle in trailing 14 days
└── Certification progress: 8%
    └── Score: % staff with active Bronze or higher certification

H-SCORE LEVELS:
├── 0-39:  Emerging    (red)    — Intervention triggered
├── 40-59: Developing  (orange) — Improvement plan activated
├── 60-74: Established (yellow) — On track; identify weakest component
├── 75-89: Exceptional (green)  — Bonus pool eligible
└── 90+:   Elite       (gold)   — Network recognition + case study candidate
```

### Anomaly Detection
The H-Score engine runs anomaly detection for:
- Sudden drops (>10 points in 7 days) → Practice owner alert + Jordan's intervention queue
- Stagnation (< 2 point movement in 30 days) → Content adjustment + targeted intervention
- Score gaming signals (review velocity spike without corresponding phone/huddle improvement) → Flag for manual review

---

## Component 5: Trust Transfer Verification System

### Responsibility
Lightweight behavioral verification for the trust transfer protocol — the hygienist-to-DA-to-doctor patient handoff sequence. Combines a structured daily checklist, voice sentiment correlation, and post-visit review signal analysis to detect whether the handoff is being executed.

### Interfaces
- Integrates with: Voice recording infrastructure (operatory ambient audio, if available), Swell API (post-visit review language), Dental Intel (daily checklist prompt within huddle opening)
- Feeds: H-Score Engine (trust transfer compliance signal), CCA Academy (Silver certification evidence), Content Engine (which practices need trust transfer content reinforcement)

### Data Produced / Consumed
- Produces: Trust transfer checklist completion events, voice sentiment scores (where audio available), review-based trust transfer proxy scores
- Consumes: Voice recordings (optional; where operatory mics deployed), Swell review text, daily checklist completions, patient appointment data

### Key Design Decision
**Verification is proxy-based, not surveillance-based.** The system never creates a "someone is watching you" dynamic in the operatory. Instead, it uses two lightweight signals: (1) the 30-second daily checklist within the morning huddle ("did the team commit to the trust transfer protocol today?") and (2) post-visit review language analysis from Swell (NLP tagging reviews for language consistent with successful vs. failed trust transfer). Where voice recording is deployed in common areas, it supplements but does not replace these signals.

### Trust Transfer Signal Architecture
```
SIGNAL 1: Morning Commitment Check (daily, team-level)
  └── During huddle, team verbally commits to trust transfer execution
      → Logged as "team intention" (binary: yes/no)

SIGNAL 2: Post-Visit Review NLP (daily, practice-level)
  └── Swell reviews analyzed for language patterns:
      High-signal positive: "everyone knew my story," "the whole team,"
                            "I felt like they briefed each other," "seamless"
      High-signal negative: "felt rushed," "no one explained," "confused about
                            my treatment," "the doctor seemed surprised"
      → Tagged as Trust Transfer Positive / Neutral / Negative per review
      → 30-day rolling Trust Transfer Sentiment Score per practice

SIGNAL 3: Voice Sentiment (where available, common areas only)
  └── Cinnamon-equivalent analysis on check-in / reception audio
      → Detecting scripted greeting phrases, patient name usage frequency
      → NOT deployed in operatories without staff consent and policy review

SIGNAL 4: Champion Self-Report (weekly)
  └── Champion submits a weekly trust transfer pulse check (3 questions, 2 min)
      → Captures in-practice qualitative signal not available in system data
```

---

## Component 6: Champion Network Layer

### Responsibility
The people infrastructure of the platform. Manages Champion identification, onboarding, peer community, content contribution, regional cohort coordination, and Summit logistics. Champions are the human delivery mechanism for culture — the platform serves them, not the other way around.

### Interfaces
- Integrates with: CCA Academy (Gold certification track), Content Engine (Champion-submitted wow stories), Morning Huddle OS (Champion-led content substitution for experienced practices), Rewards Engine (Champion stipend and recognition)
- External: Video conferencing (Regional Champion Calls), Summit registration

### Data Produced / Consumed
- Produces: Champion profiles, wow story submissions, Champion call attendance, practice culture health self-assessments, peer mentorship connections
- Consumes: Staff certification data (to identify Champion candidates), H-Score trends (to surface Champions at high-performing practices), practice owner nominations

### Key Design Decision
The Champion Network is designed to **grow the playbook from the bottom up** — Champions are contributors to the playbook, not just recipients of it. Their submitted practice stories become part of the content library. Their facilitation innovations get shared with the network. Their questions and adaptations feed the Content Production Pipeline for the next content cycle. Over time, the playbook becomes a living document that reflects the wisdom of 260 practices, not just SGA HQ's original vision.

### Champion Architecture
```
CHAMPION TIERS:
├── CHAMPION CANDIDATE — Practice has designated but not yet Gold-certified
│   └── Access: Champion content library, regional call observer
│
├── CERTIFIED CHAMPION (CCA Gold) — Full Champion status
│   └── Access: Network Champion directory, wow story submission,
│             regional call participant, Summit invitation,
│             facilitation guide authorship, mentorship pairing
│   └── Stipend: Monthly facilitation stipend (amount TBD by SGA)
│
└── MASTER CHAMPION — Top 5% by huddle quality and network contribution
    └── Access: Co-creates new playbook content with Grant,
              invited to present at industry events,
              credentials submitted for dental school partnership discussions

CHAMPION NETWORK CALENDAR:
├── Weekly: Champion Content Drop (Culture Card + digital one-pager)
├── Monthly: Regional Champion Call (15-20 practices, SGA-facilitated)
├── Monthly: Grant's Culture Brief video (10 min, distributed to all)
├── Quarterly: Champion Showcase (network-wide virtual event, story sharing)
└── Annual: Champion Summit (in-person, travel stipend for Top Champions)
```

---

## Component 7: Recognition and Rewards Engine

### Responsibility
Manages the motivation architecture — leaderboard, point accumulation, bonus pool triggers, physical reward fulfillment (jackets), and public recognition moments. Connects behavior outcomes to tangible rewards in a way that is motivating, transparent, and fair.

### Interfaces
- Receives from: H-Score Engine (score thresholds for bonus triggers), CCA Academy (certification milestone events), Morning Huddle OS (huddle streak milestones), Trust Transfer Verification (behavior compliance signals)
- Feeds: Morning Huddle OS (leaderboard for daily display), Network Intelligence Dashboard (recognition highlights), CCA Academy (reward unlock information per staff profile)

### Data Produced / Consumed
- Produces: Leaderboard rankings, bonus pool accumulation records, reward fulfillment requests (jackets), point transaction histories, recognition event logs
- Consumes: H-Score records, certification milestones, huddle completion streaks, Champion designations

### Key Design Decision
The rewards system operates at **three levels simultaneously** — individual staff, practice, and region — to prevent the scenario where a high-performing individual feels penalized by a low-performing practice (or vice versa). Individual rewards (LinkedIn badge, jacket, personal recognition) are tied to individual certification progress. Practice rewards (bonus pool contribution, leaderboard visibility) are tied to H-Score. Regional rewards (Summit invitations, network spotlight) are tied to regional Champion performance.

### Rewards Architecture
```
INDIVIDUAL STAFF REWARDS:
├── CCA Bronze earned: Digital certificate + LinkedIn badge + huddle recognition
├── CCA Silver earned: Enhanced certificate + network directory listing + bonus multiplier
├── CCA Gold earned: THE JACKET + Gold certificate + Champion stipend unlock
└── Champion of the Month: Name in network newsletter + Summit invitation

PRACTICE REWARDS (H-Score based):
├── Exceptional (75+) for 1 month: Bonus pool contribution ($X per staff member)
├── Exceptional (75+) for 3 consecutive months: Featured in Network Spotlight
├── Elite (90+): Case study publication candidate + regional director recognition
└── Most Improved (month over month): "Rising Star" network recognition

LEADERBOARD MECHANICS:
├── Network Leaderboard: Top 25 practices by H-Score (displayed in huddle monthly)
├── Regional Leaderboard: Top 10 in each region (displayed in Regional Champion Call)
├── Huddle Streak Board: Top 10 practices by consecutive days with completed huddle
└── Certification Board: Top 10 practices by % staff certified

BONUS POOL MECHANICS:
├── SGA contributes to practice bonus pool when H-Score crosses 75+ threshold
├── Pool accumulates monthly; distributed quarterly to all staff at the practice
├── Distribution: Equal split among all staff (not performance-weighted within practice)
├── Transparency: Each staff member can see the current pool balance in their profile
└── Champion multiplier: C&C Gold Champions receive 1.5x standard pool share
```

---

## Component 8: Playbook Distribution System

### Responsibility
Manages how the living "Caring with Charisma" playbook gets updated, versioned, and distributed to all 260 practices without requiring retraining. Think of this as the "firmware update" system for the culture OS — when the playbook evolves, practices are notified, content is updated, and new behaviors are introduced through the existing huddle ritual rather than a separate training event.

### Interfaces
- Receives from: Content Production Pipeline (new content), SGA leadership approval workflow (version gates), Champion Network (field feedback and practice stories)
- Feeds: Playbook Content Engine (versioned content updates), CCA Academy (updated module versions), Morning Huddle OS (new content packages), all practices (notification of playbook updates)

### Data Produced / Consumed
- Produces: Versioned playbook documents, update release notes, practice-level adoption tracking for new content
- Consumes: Content production outputs, Champion feedback submissions, SGA editorial review records

### Key Design Decision
The playbook is versioned like software — each major version has a changelog, a release date, and a structured rollout plan. **Major updates** (new behavior protocols, new certification requirements) require a "Playbook Update Huddle" — a special huddle segment that introduces the change, explains why it's happening, and gives staff a 30-day adoption window. **Minor updates** (new wow stories, new video content, refreshed scripts) roll automatically into the next content cycle. This versioning model makes the playbook genuinely "living" — capable of evolving without requiring a system-wide relaunch.

### Distribution Channels
```
PLAYBOOK DISTRIBUTION CHANNELS:
├── Dental Intel Integration: Huddle OS updates automatically on content publish
├── CCA Academy: Module updates versioned with clear changelog visible to learners
├── Champion Network: Champions notified of updates 2 weeks before practice rollout
│                    (preview + facilitation notes for introducing the change)
├── Mobile Push: Staff with app installed receive playbook update notification
├── Physical: Major version releases ship an updated "Caring with Charisma" desk card
│            to every practice (the physical artifact that sits at every station)
└── Academic Export: Each major version generates a structured academic-format
                     document suitable for dental school curriculum submission
```

---

## Component 9: Network Intelligence Dashboard

### Responsibility
The SGA HQ and regional director command center. Provides portfolio-level visibility into H-Score trends, certification rates, huddle completion, leaderboard standings, intervention alerts, and network-wide impact reports. Transforms the oversight of 260 practices from a manual, lagging process into a real-time, automated, prescriptive system.

### Interfaces
- Receives from: H-Score Engine (all practice scores), CCA Academy (certification rates), Morning Huddle OS (completion rates), Rewards Engine (leaderboard, bonus events), Trust Transfer Verification (compliance signals)
- External outputs: PDF/export for board presentations, academic publication assets, PR/communications materials

### Data Produced / Consumed
- Produces: Practice portfolios, intervention alerts, weekly digest reports, monthly network impact reports, quarterly executive summaries, academic publication data packages
- Consumes: All platform signals from all 260 practices

### Key Design Decision
The dashboard is built for **three distinct user personas** with different views: (1) Regional Directors (80-practice portfolio, heatmap + intervention queue), (2) SGA Executive Team (network-wide trends, ROI data, strategic metrics), (3) Practice Owners (their own practice plus peer benchmarking). Each persona sees only the data relevant to their decision-making scope. Practice owners cannot see individual competitor H-Scores — only their own rank and the top 25 anonymized leaderboard.

### Dashboard Views
```
REGIONAL DIRECTOR VIEW:
├── Practice Heatmap: H-Score color-coded across portfolio
├── Intervention Queue: Auto-triggered alerts with recommended actions
├── Certification Progress: % staff certified per practice
├── Huddle Streak Tracker: Consecutive days per practice
└── Champion Status: Designated vs. active vs. Gold-certified by practice

SGA EXECUTIVE VIEW:
├── Network Health Score: Average H-Score across all practices + trend
├── Program Adoption: % practices with active huddle, % staff certified
├── Revenue Correlation: H-Score quartile vs. review velocity vs. new patient acquisition
├── ROI Dashboard: Platform investment vs. measurable outcomes
└── Academic Publication Asset: Data export for dental journal submission

PRACTICE OWNER VIEW:
├── My H-Score: Current score + trend + component breakdown
├── My Staff: Certification status per staff member
├── My Huddle: Completion streak + engagement metrics
├── My Leaderboard Position: Regional rank + network rank (anonymized competitors)
└── My Bonus Pool: Current balance + next milestone to unlock
```

---

## Component 10: Content Production Pipeline

### Responsibility
The operational workflow for creating all platform content — Grant videos, Champion story edits, module updates, huddle segments, and playbook revisions. Manages the intake, production, review, and publishing cycle that keeps the content engine running without requiring SGA to improvise.

### Interfaces
- Receives from: Champion Network (submitted practice stories + video recordings), SGA executive team (Grant availability, strategic priorities), Content Engine (gap analysis — which topics need new content)
- Feeds: Playbook Content Engine (published content ready for delivery)

### Data Produced / Consumed
- Produces: Published video modules, edited wow stories, huddle content packages, playbook revision drafts
- Consumes: Raw video submissions, Champion stories, Grant's recording sessions, SGA editorial guidelines

### Key Design Decision
The pipeline is designed as a **two-tier system**: Grant's content (high production, quarterly batch recording — 30-40 videos per session, covering 3-4 months of content) and Champion/practice content (lower production bar, monthly cycle, edited by SGA culture team). Grant records in batches to minimize executive time commitment while maintaining content freshness. The pipeline also generates the **academic content layer** — formatted versions of the playbook suitable for dental school curriculum submission.

### Production Cadence
```
GRANT CONTENT (Tier 1):
├── Quarterly batch recording (1-2 day shoot): 30-40 short segments
│   └── Weekly Culture Moment videos (52/year)
│   └── CCA module anchor videos (one per module per tier = 24 total)
│   └── Playbook update introduction videos (as needed)
└── Monthly Culture Brief: 10-minute in-depth video (1 shoot/month)

CHAMPION/PRACTICE CONTENT (Tier 2):
├── Weekly Champion story submissions reviewed and edited
├── Monthly top 3 stories selected for network distribution
├── Quarterly Showcase recording (edited highlights)
└── Annual Summit documentation (keynotes, breakouts, case studies)

PLAYBOOK PUBLISHING CYCLE:
├── Minor update: 2-week cycle (story swap, script refresh, data update)
├── Major update: 6-week cycle (new behavior protocol, new module, network announcement)
└── Annual Version Release: Full playbook document refresh with academic export
```

---

## Integration Architecture

### Cinnamon Integration (Phone Analytics → H-Score)
```
Data Flow:
Cinnamon captures all inbound/outbound calls across practices
  → Cinnamon API pushes daily analytics to H-Score Engine:
    - Answer rate (calls answered within 3 rings)
    - Hold duration (average and % over 2 minutes)
    - Call sentiment (positive/neutral/negative per call)
    - Phrase detection (warm greeting phrases in first 10 seconds)
  → H-Score Engine calculates Cinnamon component score (35% weight)
  → Anomalies (answer rate drop >15%) trigger automatic alert to practice owner
  → CCA Academy: phone behavior gaps trigger assignment of relevant Bronze module
    (e.g., answer rate below 70% → Module 04 "The Phone as Hospitality" auto-assigned)
```

### Swell Integration (Reviews + NPS → H-Score + Rewards)
```
Data Flow:
Swell captures 5-star review requests, responses, NPS surveys
  → Swell API pushes daily review data to H-Score Engine:
    - New reviews (star rating, text, date, patient pseudonym)
    - NPS scores (practice-level rolling average)
    - Review velocity (reviews per 100 appointments)
  → NLP pipeline runs on review text:
    - Hospitality language detection (warm, friendly, welcoming, explained, etc.)
    - Trust transfer signal detection (whole team, briefed, knew my story, etc.)
    - Negative signal detection (rushed, confused, cold, nobody explained, etc.)
  → Swell component score calculated (45% weight in H-Score)
  → High-signal positive reviews flagged for huddle "Yesterday's Wins" segment
  → Staff-mention reviews flagged for Recognition Engine (personal celebration)
  → Trust transfer positive reviews added to Trust Transfer Sentiment Score
```

### Dental Intel Integration (Huddle → Completion Tracking)
```
Data Flow:
Dental Intel hosts the morning huddle interface for Gen 4 offices
  → Dental Intel API reports to H-Score Engine:
    - Huddle completion events (practice ID, date, duration)
    - Staff check-in data (which staff participated in huddle)
    - Patient schedule for the day (used to personalize today's focus content)
  → Huddle OS pushes daily content package TO Dental Intel display
    (content assembled by Playbook Content Engine, filtered by practice profile)
  → Certification-linked segments trigger CCA Academy completion events
    when marked complete in Dental Intel
  → Dental Intel component score calculated (20% weight in H-Score)

Note on non-Gen-4 practices: For practices not using Dental Intel,
the Huddle OS is accessible via a standalone web URL (mobile-friendly)
that practice owners can open on any screen. Completion is self-reported
and verified via session duration and segment completion events.
```

### Voice Recording Integration (Trust Transfer Verification)
```
Data Flow:
Voice recording deployed in reception/common areas (not operatories)
  → Audio analyzed by NLP pipeline (same vendor as Cinnamon or equivalent):
    - Patient name usage frequency at reception
    - Warm greeting phrase detection
    - "I'll walk you back" and proactive escort behavior signals
  → Results feed Trust Transfer Verification System as Signal 3
  → NOT connected to H-Score directly — used as supplementary signal
    for Champion self-report validation and clinical behavior coaching

Policy note: Operatory recording requires state-specific consent protocols.
Architecture supports it where legally permissible; defaults to proxy signals
(review sentiment + daily checklist) in all other environments.
```

### Playbook Living Document System (Versioning + Distribution)
```
Playbook Version Control:
├── Playbook stored as structured document (JSON + Markdown)
├── Each behavior tagged with version number (e.g., "trust-transfer-v2.1")
├── Changes tracked with rationale, author, and effective date
├── Practice-level adoption of each behavior version tracked
├── Champion feedback tagged to specific playbook sections
└── Academic export auto-generates from versioned structure:
    → Each behavior maps to a learning objective
    → Each learning objective maps to a CCA module
    → Module maps to an assessment scenario
    → This structure matches dental school curriculum format
```

---

## Architecture Decision Records (ADRs)

### ADR-01: The Huddle OS is the Primary Content Delivery Container — Not a Standalone App or New LMS

**Decision**: All training content, certification prompts, H-Score displays, and cultural rituals are delivered through the morning huddle experience inside Dental Intel — not through a standalone mobile app, a dedicated LMS, or a separate web portal.

**Rationale**: The morning huddle is the only moment in the day when the entire practice team is together and not yet in patient-facing mode. It is a captive attention window that already exists. Building a new app or LMS creates an adoption problem: something new to download, log into, remember, and use. Extending the morning huddle — which already happens (imperfectly) — eliminates the adoption barrier entirely. The platform makes an existing behavior better rather than creating a new behavior.

**Trade-offs accepted**: Dental Intel dependency means practices not on Dental Intel need a parallel delivery mechanism (standalone web URL). Non-Dental-Intel practices get a slightly degraded experience until migration is complete. This is an acceptable trade-off given that Gen 4 offices are the primary rollout target.

**Consequence**: Every platform decision must ask "can this surface in the morning huddle?" before creating a separate interface. The huddle is the UX.

---

### ADR-02: H-Score Weights Are Transparent and Documented — No Black Box

**Decision**: The H-Score algorithm is fully documented and visible to practice owners and staff. Every component, its weight, and its calculation method is published in the platform.

**Rationale**: In healthcare and high-trust service environments, opaque scoring systems create resentment and distrust. If staff believe the score measures things outside their control (or doesn't measure what they actually do), they will game it or ignore it. Transparency about what the score measures and how it is calculated is the only path to the score being trusted as fair — which is the precondition for it being motivating.

**Trade-offs accepted**: Transparency about the algorithm creates some gaming surface (e.g., soliciting Swell reviews at specific times to optimize velocity score). Mitigated by anomaly detection for unusual review patterns and by weighting phone behavior and huddle completion alongside review scores — gaming one component doesn't move the composite score enough to matter.

**Consequence**: All H-Score documentation is written for a practice owner audience — not technical documentation, but plain-language explanations of "here's what moves your score and why."

---

### ADR-03: Practice-Type Adjusted Scoring — Rural Practices Are Not Compared Against Urban Practices

**Decision**: H-Score benchmarking and leaderboard rankings are segmented by practice type: Rural (< 20 patients/day), Suburban (20-50 patients/day), Urban (50-100 patients/day), Cosmetic (any size, high revenue per visit), and Pediatric (any size, youth-focused). Each type benchmarks against its own cohort.

**Rationale**: A rural practice with 12 patients per day generates fundamentally different data volumes than an urban practice with 80. The same hospitality behaviors produce fewer Swell reviews, fewer Cinnamon call analytics, and fewer huddle participation data points. Comparing them on raw metrics is unfair and demoralizing for rural practices — which would systematically appear at the bottom of any network ranking regardless of actual hospitality quality.

**Trade-offs accepted**: Segmented benchmarking makes network-level comparisons more complex and limits the "one universal leaderboard" concept. Mitigated by maintaining a separate network-wide "Most Improved" leaderboard that compares percent-change in H-Score (not absolute score) — allowing any practice type to compete and win.

**Consequence**: All content personalization, leaderboard displays, and intervention thresholds are configured per practice-type segment. The platform's data model includes practice_type as a required field for all scoring calculations.

---

### ADR-04: The CCA Credential Is Portable and Externally Verifiable From Day One

**Decision**: The CCA credential is designed as an external, verifiable credential from day one — not an internal training completion record that might later be made public. Every certificate issued includes a unique verification URL. LinkedIn integration is built into the Bronze issuance flow. All curriculum content is structured to academic standards.

**Rationale**: The difference between a credential and a training completion record is portability and external recognition. If the credential can only be verified by SGA (or only lives in an HR system), it has no career value to staff — and career value is the entire motivation architecture for CCA Bronze engagement. Building the external credentialing infrastructure at launch, rather than as a future upgrade, makes the credential genuinely valuable from the first day it is issued.

**Trade-offs accepted**: External credential infrastructure (verification portal, LinkedIn API integration, academic formatting) requires additional development investment upfront. Partially mitigated by using an off-the-shelf credentialing platform (Credly or Accredible) for the public verification and LinkedIn integration layer — SGA owns the credential design and content, the platform handles the technical issuance.

**Consequence**: All CCA content must meet the documentation standards required for academic submission. This creates a higher quality bar for content production — which is a positive constraint, not a burden.

---

### ADR-05: The Playbook Is a Living Document — Never "Final"

**Decision**: The "Caring with Charisma" playbook is versioned software — never printed as a final document, always capable of evolution without requiring a full relaunch. Major versions (v1.0, v2.0) are significant releases. Minor versions (v1.1, v1.2) are incremental refinements. Every practice knows which version their training is built on.

**Rationale**: The biggest failure mode for culture programs is the "binder on the shelf" problem — a playbook that was written once, is now out of date, and sits unread in every practice's break room. By designing the playbook as a living document with a clear versioning and distribution system, SGA can evolve the culture standard as the network grows, as practices contribute insights, and as external research (dental school partnerships, academic publications) generates new best practices.

**Trade-offs accepted**: Living document versioning requires an editorial discipline that is organizationally unusual for a dental DSO. SGA needs a dedicated "Playbook Editor" role (or function within the culture team) to manage the update cycle. This is a real ongoing operational cost.

**Consequence**: All platform content is tagged to a playbook version. When a version update is released, affected content is flagged for review in the CCA Academy and Huddle OS. Staff who completed Bronze under v1.0 are given a "What's Changed" micro-module when v2.0 launches — not a full recertification.

---

## Data Model

```
CORE ENTITIES:

Practice
├── practice_id (unique)
├── practice_name
├── practice_type (rural/suburban/urban/cosmetic/pediatric)
├── region_id
├── dental_intel_id (integration key)
├── cinnamon_account_id (integration key)
├── swell_account_id (integration key)
├── h_score_current (float, 0-100)
├── h_score_level (enum: emerging/developing/established/exceptional/elite)
├── champion_id (FK → StaffMember, nullable)
└── playbook_version (FK → PlaybookVersion)

StaffMember
├── staff_id (unique)
├── practice_id (FK → Practice)
├── full_name
├── role (enum: front_desk/hygienist/da/doctor/office_manager)
├── hire_date
├── cca_tier (enum: none/bronze/silver/gold)
├── cca_bronze_completed_date
├── cca_silver_completed_date
├── cca_gold_completed_date
├── is_champion (boolean)
├── champion_tier (enum: candidate/certified/master, nullable)
├── linkedin_credential_id (nullable, from Credly/Accredible)
└── bonus_pool_multiplier (float, default 1.0)

CertificationRecord
├── record_id (unique)
├── staff_id (FK → StaffMember)
├── module_id (FK → ContentModule)
├── completion_date
├── scenario_score (int, 0-100)
├── passed (boolean)
└── version_completed (FK → PlaybookVersion)

HScore Record
├── record_id (unique)
├── practice_id (FK → Practice)
├── date
├── h_score (float, 0-100)
├── swell_component (float)
├── cinnamon_component (float)
├── dental_intel_component (float)
├── anomaly_flag (boolean)
└── anomaly_type (nullable string)

HuddleSession
├── session_id (unique)
├── practice_id (FK → Practice)
├── date
├── completion_status (enum: complete/partial/skipped)
├── facilitator_id (FK → StaffMember, nullable)
├── content_package_id (FK → ContentPackage)
├── staff_attended (count)
└── duration_seconds

TrustTransferEvent
├── event_id (unique)
├── practice_id (FK → Practice)
├── date
├── signal_type (enum: checklist/review_nlp/voice_sentiment/champion_report)
├── signal_value (float, 0-1)
├── review_id (FK → Review, nullable)
└── notes (nullable text)

Review
├── review_id (unique)
├── practice_id (FK → Practice)
├── source (enum: swell/google/yelp)
├── date_received
├── star_rating (int, 1-5)
├── review_text (text)
├── hospitality_sentiment_score (float, 0-1, from NLP)
├── trust_transfer_signal (enum: positive/neutral/negative)
├── staff_mention (boolean)
├── staff_mentioned_id (FK → StaffMember, nullable)
└── flagged_for_huddle (boolean)

Champion
├── champion_id (unique)
├── staff_id (FK → StaffMember)
├── practice_id (FK → Practice)
├── designation_date
├── tier (enum: candidate/certified/master)
├── wow_stories_submitted (int)
├── huddles_facilitated (int)
├── staff_onboarded (int)
└── regional_cohort_id (FK → RegionalCohort)

Reward
├── reward_id (unique)
├── recipient_type (enum: staff/practice/region)
├── recipient_id (FK → StaffMember or Practice)
├── reward_type (enum: bonus_pool/jacket/certificate/recognition/leaderboard)
├── trigger_event (string)
├── amount (nullable float, for bonus pool)
├── fulfillment_status (enum: pending/processing/delivered)
└── awarded_date

PlaybookVersion
├── version_id (unique)
├── version_number (string, e.g., "v1.2")
├── release_date
├── major_changes (text)
├── behaviors_added (array)
├── behaviors_modified (array)
├── academic_export_url (nullable)
└── is_current (boolean)
```

---

## Connection to SGA Ecosystem

The Hospitality Playbook Platform is the culture layer of the broader SGA technology ecosystem. It connects to:

**Practice Concierge**: The Practice Concierge manages per-practice digital presence and content operations. The Hospitality Playbook Platform feeds the Practice Concierge with review velocity signals (from H-Score Swell component) and staff certification status (for "Certified C&C Practice" badge on practice websites and Google Business Profiles). The C&C certification can become a marketing asset — visible on the practice's digital storefront.

**Funnel OS / Practice Nerve Center**: The Funnel OS unifies patient acquisition and conversion data. The H-Score's Swell component (review velocity and quality) is a direct input to the Funnel OS's "Reputation" layer. Practices with higher H-Scores generate more organic new patient inquiries. The Funnel OS closes the attribution loop: hospitality investment → better reviews → more new patients → measurable revenue delta.

**Comms Engine**: The patient communications automation layer (Comms Engine) handles post-visit follow-up messages. The Hospitality Playbook Platform can flag patients who had exceptional trust transfer experiences (based on review signals) for targeted referral program outreach — "you had a great experience, tell a friend."

**Content Concierge**: The Content Concierge captures practice-level media (before/afters, testimonials, video). Champion-submitted wow stories from the Culture OS are a natural input to the Content Concierge's testimonial pipeline. Staff who are CCA Gold certified are the most credible voices for practice testimonial videos. The two platforms share a content taxonomy — both tag content to practice type, patient experience theme, and distribution channel.

---

## Platform Summary

| Component | Primary User | Core Output | Connects To |
|-----------|-------------|-------------|-------------|
| Playbook Content Engine | SGA Culture Team | Living content library | All delivery components |
| Morning Huddle OS | Practice Team | Daily ritual experience | Dental Intel, H-Score, CCA |
| CCA Academy | Staff Members | Tiered certification + credential | Rewards, H-Score, Champion Network |
| Hospitality Score Engine | Practice Owners, SGA HQ | Daily H-Score per practice | All input sources, Dashboard |
| Trust Transfer Verification | Champions, Clinical Staff | Handoff compliance signal | H-Score, CCA, Content Engine |
| Champion Network Layer | C&C Champions | Peer community + content contribution | CCA, Content Engine, Rewards |
| Recognition & Rewards Engine | All Users | Leaderboard, bonus pool, jacket | H-Score, CCA, Huddle OS |
| Playbook Distribution System | SGA Culture Team | Versioned playbook updates | Content Engine, CCA, Huddle OS |
| Network Intelligence Dashboard | Regional Directors, SGA Exec | Portfolio view + intervention triggers | All components |
| Content Production Pipeline | SGA Culture Team + Champions | Published content and videos | Content Engine |
