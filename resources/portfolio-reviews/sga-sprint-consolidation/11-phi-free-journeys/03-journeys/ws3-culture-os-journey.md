# WS3: Culture OS — PHI-Free User Journey

**Journey Title**: "Hospitality Culture Through Daily Rituals and Continuous Development"
**Date**: 2026-03-20
**Author**: UX Designer Agent (Phase A Journey Specification)
**Phase**: A (PHI-Free)
**Interfaces Covered**: CO-1, CO-2, CO-3, CO-4, CO-5, CO-6, CO-7, CO-8
**Companion Documents**: [PHI Scope Analysis](../01-scope/phi-scope-analysis.md) | [Experience Architecture](../02-experience/phi-free-experience-architecture.md) | [Original Cross-Workstream Flows](../../03-experience/interactions/cross-workstream-flows.md)

---

## Actors

| Actor | Role | Primary Surface | Phase A Access |
|-------|------|----------------|----------------|
| All Practice Staff | Morning huddle participants | TV Display + PWA (Huddle screen) | View huddle content, mark attendance |
| Maria Santos | Champion / Hygienist | PWA (Champion Hub) | Scorecard, missions, leaderboard, capture |
| Lisa Torres | Office Manager | PWA + Web (H-Score Dashboard) | H-Score monitoring, huddle oversight, staff certification tracking |
| Marcus Williams | Regional Director / VP Ops | Network Intelligence (Practice Deep Dive Culture tab) | Regional culture oversight, intervention management |
| HQ Culture Team | Culture Program Administrators | Culture Admin Console (Desktop) | Curriculum management, playbook deployment, champion network management |

---

## Why WS3 Is Almost Entirely PHI-Free

Culture OS was designed around three data domains that contain zero patient health information:

1. **Aggregate vendor scores**: Cinnamon (phone quality), Swell (review management), Dental Intel (operational compliance) — all report practice-level or staff-level aggregate scores, never individual patient data
2. **Staff development data**: CCA certification progress, training completion, trust transfer scores per staff member — employee data from iSolved, not patient data
3. **Culture rituals**: Morning huddle content, recognition posts, playbook content, champion missions — all staff-facing, never patient-facing

**Phase A impact**: All 8 of 8 interfaces ship. Only 1 interface has a minor modification (CO-1 Morning Huddle Display — Impact Story content changes from funnel metrics to culture/brand metrics). The remaining 7 interfaces are completely unchanged.

**The one deferred feature**: Individual call transcript drill-down in Trust Transfer Tracker (CO-7). Aggregate trust transfer scores per staff member are available; listening to or reading individual call transcripts requires call recording access, which contains verbal PHI. This is handled with an empty state in the drill-down view.

---

## Flow 1: Morning Huddle Ritual

### Step 1.1: TV Display — Break Room, 7:50 AM

The break room TV at Smile Center Buckhead displays the morning huddle content. The display URL is bookmarked on the TV's browser (or launched via Dental Intel integration).

**Panel 1 of 4 (auto-advances every 30 seconds):**

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│                      SMILE CENTER BUCKHEAD                                │
│                      Thursday Morning Huddle                              │
│                                                                          │
│  ┌───────────────────┐  ┌───────────────────┐  ┌──────────────────────┐ │
│  │  H-SCORE          │  │  FOCUS TODAY       │  │  RECOGNITION         │ │
│  │                   │  │                    │  │                      │ │
│  │     64            │  │  Warm Handoffs     │  │  Maria Santos        │ │
│  │    ▼14 from       │  │                    │  │  12-week huddle      │ │
│  │    2 weeks ago    │  │  "Introduce every  │  │  streak! Longest     │ │
│  │                   │  │  patient to their  │  │  in the Southeast    │ │
│  │  Phone: 58 ▼     │  │  next caregiver    │  │  region.             │ │
│  │  Reviews: 71 ▼   │  │  by name. Say:     │  │                      │ │
│  │  DI: 74 ▼        │  │  'Dr. Patel is     │  │  [Gen4 branded       │ │
│  │                   │  │  going to take     │  │   recognition card   │ │
│  │  Network avg: 74  │  │  great care of     │  │   with Maria's       │ │
│  │                   │  │  you today.'"      │  │   name + achievement]│ │
│  └───────────────────┘  └───────────────────┘  └──────────────────────┘ │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────────┐│
│  │  IMPACT STORY: Our Google rating improved from 4.1 to 4.3 since     ││
│  │  we focused on the review request script in huddles. That's 18      ││
│  │  new 5-star reviews in 30 days.                                     ││
│  └──────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│  [Auto-advances in 30s]  ● ○ ○ ○                                       │
└──────────────────────────────────────────────────────────────────────────┘
```

**Panel 2: Yesterday's Wins**
```
│  YESTERDAY'S PRACTICE WINS                                               │
│                                                                          │
│  ✓ Huddle completed with full attendance                                │
│  ✓ Maria captured 2 team photos (8-week content streak!)               │
│  ✓ 3 new Google reviews received (all 5-star)                          │
│  ✓ Dr. Kim's trust transfer score improved 8 points this week          │
│                                                                          │
│  TEAM CCA PROGRESS                                                       │
│  Gold: 1 | Silver: 3 | Bronze: 8 | In Progress: 4                     │
│  Next milestone: Jamie L. completes Bronze Module 1 (est. 3/25)        │
```

**Panel 3: Today's Playbook Focus**
```
│  PLAYBOOK SPOTLIGHT                                                      │
│                                                                          │
│  Warm Handoff Script — Phone to Treatment Room                          │
│                                                                          │
│  Step 1: "Let me introduce you to [provider name]."                     │
│  Step 2: Share one relevant detail: "I mentioned to Dr. Patel           │
│          that you're interested in the whitening options."               │
│  Step 3: Confirm comfort: "Is there anything else I can help            │
│          with before Dr. Patel gets started?"                            │
│                                                                          │
│  Grant Training Tip:                                                     │
│  "The warm handoff is the single highest-impact moment in the           │
│  patient experience. It takes 10 seconds and transforms how the         │
│  patient feels about the entire visit."                                 │
```

**Panel 4: Content Capture Reminder**
```
│  CONTENT OPPORTUNITY                                                     │
│                                                                          │
│  Champions: Today's content mission is team photography.                │
│  Capture a group photo before patients arrive.                          │
│                                                                          │
│  Last capture: 2 days ago (Maria — team photo)                          │
│  This week's target: 3 captures remaining                               │
│                                                                          │
│  "Great content starts with showing the people behind the smiles."     │
```

**Data sources for the huddle display:**
- H-Score + sub-scores: Culture OS composite calculation from Cinnamon (35%), Swell (45%), Dental Intel (20%)
- Focus behavior: Culture OS (weekly behavior rotation configured by HQ Culture Team)
- Recognition: Culture OS (automated achievement detection + staff nominations)
- Impact Story: Culture OS AI generation using non-PHI signals (review volume, H-Score trends, brand health improvements)
- Practice wins: Culture OS (huddle completion), Content Engine (capture counts via Kafka), Swell (review counts)
- CCA progress: Culture OS (training tracker)
- Playbook content: Culture OS (playbook engine, managed by HQ)
- Content mission: Content Engine (capture brief via Kafka `sgadental.ws2.capture-brief-generated`)

**Impact Story change (the ONLY modification in Phase A):**
| Original Impact Story (Full PHI) | Phase A Impact Story (PHI-Free) |
|---|---|
| "Our show rate improved 5% since we started the confirmation text sequence. That's ~4 more patients per week." | "Our Google rating improved from 4.1 to 4.3 since we focused on the review request script. That's 18 new 5-star reviews in 30 days." |
| "Treatment acceptance is up 3% this month. The warm handoff training is working!" | "Our H-Score improved 8 points since we started the warm handoff training focus. Cinnamon phone scores up 12%." |

The Impact Story sources change; the display format, animation, and timing are identical.

**Auto-advance behavior:** Panels cycle every 30 seconds. Total passive viewing time: ~2 minutes. The remaining 3 minutes of the 5-minute huddle are for team discussion. TV display does not require any user interaction.

### Step 1.2: PWA Huddle Screen — Mobile Reference

Staff who want to reference huddle content on their phones open the **Huddle** tab in the PWA.

```
┌──────────────────────────────┐
│  Morning Huddle               │
│  Thursday, March 20           │
├──────────────────────────────┤
│                              │
│  H-SCORE: 64                │
│  ▼14 from 2 weeks ago       │
│  Phone: 58 | Reviews: 71    │
│  DI: 74                     │
│                              │
│  ─────────────────────────── │
│  FOCUS TODAY                 │
│  Warm Handoffs               │
│  "Introduce every patient to │
│  their next caregiver by     │
│  name."                      │
│  ─────────────────────────── │
│  RECOGNITION                 │
│  Maria Santos — 12-week      │
│  huddle streak!              │
│  ─────────────────────────── │
│  IMPACT STORY                │
│  "Our Google rating improved │
│  from 4.1 to 4.3..."        │
│  ─────────────────────────── │
│  YESTERDAY'S WINS            │
│  ✓ Full attendance huddle   │
│  ✓ 2 content captures       │
│  ✓ 3 new 5-star reviews     │
│  ─────────────────────────── │
│  PLAYBOOK SPOTLIGHT          │
│  Warm Handoff Script...      │
│  [View Full Playbook →]      │
│  ─────────────────────────── │
│                              │
│  [Mark Huddle Complete ✓]    │
│                              │
├──────────────────────────────┤
│ [Today] [Capture] [Huddle] [Hub] │
└──────────────────────────────┘
```

Same content as TV display, vertical scroll instead of auto-cycle. The **"Mark Huddle Complete"** button logs the huddle as completed in Culture OS.

**What happens behind the scenes on "Mark Huddle Complete":**
- Culture OS records huddle completion for this practice and date
- Kafka event: `sgadental.ws3.huddle-completed`
- Consumed by: Network Intelligence (Practice Deep Dive Culture tab shows calendar), AI Analysis Engine (huddle compliance factored into culture risk scoring)
- Champion hub updates huddle streak counter

**Offline behavior:** Huddle content is cached nightly at midnight. The Huddle screen works fully offline. "Mark Huddle Complete" queues locally and syncs on reconnection.

---

## Flow 2: Champion Hub Daily Use

### Step 2.1: Champion Scorecard Deep Dive

Maria taps her scorecard on the Champion Hub for a detailed view:

```
┌──────────────────────────────┐
│  Champion Scorecard           │
│  Maria Santos                 │
├──────────────────────────────┤
│                              │
│  OVERALL RANK: #1            │
│  Southeast Region             │
│  186 points this month       │
│                              │
│  ─── CULTURE ───             │
│  Score: 92 / 100             │
│                              │
│  Huddles Led: 4/5 this week  │
│  Streak: 12 weeks            │
│                              │
│  Behavior Modeling: 4.2/5    │
│  (peer feedback aggregate)   │
│                              │
│  Recognition Given: 3 this wk│
│  Recognition Received: 2     │
│                              │
│  ─── CONTENT ───             │
│  Score: 88 / 100             │
│                              │
│  Captures: 6 last week       │
│  Streak: 8 weeks of 5+      │
│  Quality Score: 4.0/5        │
│  (editorial quality rating)  │
│                              │
│  Published: 4 assets this mo │
│  Engagement: 2.3K total reach│
│                              │
│  ─── POINTS BREAKDOWN ───    │
│  Huddles led:        48 pts  │
│  Captures submitted: 36 pts  │
│  Assets published:   24 pts  │
│  Behaviors modeled:  21 pts  │
│  Recognition given:   9 pts  │
│  Streaks maintained: 48 pts  │
│  ──────────────────────────  │
│  Total:             186 pts  │
│                              │
│  [View Leaderboard]          │
│  [View My History]           │
│                              │
├──────────────────────────────┤
│ [Today] [Capture] [Huddle] [Hub] │
└──────────────────────────────┘
```

**Data flow for the scorecard:**

| Data Element | Source | Mechanism |
|---|---|---|
| Huddles led count | Culture OS (native) | Direct database query |
| Huddle streak | Culture OS (native) | Streak calculator from `huddle-completed` events |
| Behavior modeling score | Culture OS (peer feedback system, native) | Aggregate peer ratings |
| Recognition given/received | Culture OS (Recognition Feed, native) | Recognition event counter |
| Captures submitted | Content Engine (WS2) | Kafka `sgadental.ws2.content-engagement-updated` |
| Capture streak | Content Engine (WS2) | Kafka event includes streak calculation |
| Quality score | Content Engine (WS2, editorial rating) | Kafka event includes quality metrics |
| Assets published | Content Engine (WS2) | Kafka `sgadental.ws2.content-asset-published` |
| Engagement reach | Content Engine (WS2, from social APIs) | Kafka `sgadental.ws2.content-engagement-updated` |
| Points calculation | Culture OS (native) | Weighted formula applied to all inputs |

Maria sees a unified scorecard. She does not know that Culture Score data comes from WS3 natively while Content Score data arrives via Kafka from WS2. Both render with identical card styles and typography from the `@sgadental/ui` component library.

### Step 2.2: Cohort Leaderboard

Maria taps **"View Leaderboard"**:

```
┌──────────────────────────────┐
│  Champion Leaderboard         │
│  [Southeast ▾] [This Month ▾]│
├──────────────────────────────┤
│                              │
│  #1  Maria Santos       186  │
│      Smile Ctr Buckhead      │
│      ████████████████████    │
│                              │
│  #2  David Rodriguez    174  │
│      Metro Dental Midtown    │
│      ██████████████████░░    │
│                              │
│  #3  Ashley Williams    163  │
│      Family Dental SS        │
│      █████████████████░░░    │
│                              │
│  #4  Jessica Park       156  │
│      Bright Smiles Decatur   │
│      ████████████████░░░░    │
│                              │
│  #5  Robert Chang       148  │
│      Smile Center Roswell    │
│      ███████████████░░░░░    │
│                              │
│  ...                         │
│                              │
│  YOUR POSITION: #1 of 28     │
│  champions in Southeast      │
│                              │
├──────────────────────────────┤
│ [Today] [Capture] [Huddle] [Hub] │
└──────────────────────────────┘
```

**Filtering:** By region (Southeast, Metro Atlanta, etc.) and by time period (This Week, This Month, All Time).

**This interface is UNCHANGED from the original design.** Champion leaderboards track staff activity metrics — zero PHI involvement.

---

## Flow 3: CCA Academy — Certification Pathway

### Step 3.1: Academy Portal (CO-2)

Jamie L. (new front desk hire at Smile Center Buckhead, 17 days since hire, no certification started) receives an email from Culture OS:

```
Subject: Welcome to CCA — Start Your Bronze Certification
From: SGA Culture Team <culture@sgadental.com>

Hi Jamie,

Welcome to the SGA team! The Cinnamon Certified Agent (CCA) program
is your pathway to mastering hospitality excellence.

Start your Bronze certification today:
[Begin CCA Bronze →]

Your colleagues at Smile Center Buckhead:
- Dr. Patel: Gold certified
- Maria Santos: Silver certified
- Lisa Torres: Bronze certified

Join them!
```

Jamie clicks the link and opens the CCA Academy Portal:

```
┌──────────────────────────────────────────────────────────────────────────┐
│  CCA Academy                                          Jamie L. | Bronze │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  YOUR CERTIFICATION PATH                                                │
│                                                                          │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐               │
│  │  BRONZE       │   │  SILVER       │   │  GOLD         │               │
│  │  ○ Not Started│   │  🔒 Locked    │   │  🔒 Locked    │               │
│  │               │   │               │   │               │               │
│  │  4 modules    │   │  6 modules    │   │  4 modules    │               │
│  │  ~8 hours     │   │  ~12 hours    │   │  ~10 hours    │               │
│  │  Est: 2 weeks │   │  Est: 4 weeks │   │  Est: 3 weeks │               │
│  │               │   │               │   │               │               │
│  │  [Start →]    │   │               │   │               │               │
│  └──────────────┘   └──────────────┘   └──────────────┘               │
│                                                                          │
│  BRONZE MODULES                                                         │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  Module 1: The SGA Hospitality Standard          ○ Not Started     │  │
│  │  - Video: Introduction to Hospitality (Grant) — 12 min            │  │
│  │  - Reading: SGA Hospitality Manifesto — 15 min                    │  │
│  │  - Scenario Assessment: First Impressions — 10 min                │  │
│  │  [Begin Module →]                                                  │  │
│  ├────────────────────────────────────────────────────────────────────┤  │
│  │  Module 2: Phone Excellence                      🔒 Complete M1    │  │
│  │  - Video: The First 7 Seconds on the Phone — 15 min               │  │
│  │  - Interactive: Call Handling Simulator — 20 min                   │  │
│  │  - Scenario Assessment: Difficult Callers — 15 min                │  │
│  ├────────────────────────────────────────────────────────────────────┤  │
│  │  Module 3: Trust Transfers                       🔒 Complete M2    │  │
│  │  - Video: The Warm Handoff (Grant) — 18 min                       │  │
│  │  - Reading: Trust Transfer Playbook — 10 min                      │  │
│  │  - Scenario Assessment: Handoff Role Play — 15 min                │  │
│  ├────────────────────────────────────────────────────────────────────┤  │
│  │  Module 4: Review & Certification Exam           🔒 Complete M3    │  │
│  │  - Comprehensive assessment — 30 min                              │  │
│  │  - Minimum passing score: 80%                                     │  │
│  │  - On pass: Credly Bronze badge issued                            │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

### Step 3.2: Module Completion Flow

Jamie completes Module 1. The flow is:

1. **Video**: Grant training video hosted on Mux. Standard video player with play/pause, scrub, speed control (1x, 1.5x, 2x). Completion tracked when >90% of video is watched.

2. **Reading**: Formatted text content with highlight and note-taking capability. Completion tracked when user scrolls to end.

3. **Scenario Assessment**: Interactive branching scenario.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Scenario Assessment: First Impressions                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SCENARIO 1 of 5                                                        │
│                                                                          │
│  A new patient walks through the door. They look nervous and are        │
│  clutching their paperwork. The waiting room has 3 other patients.      │
│  What do you do first?                                                  │
│                                                                          │
│  ○ A) "Sign in at the desk and have a seat."                           │
│  ○ B) Stand up, make eye contact, smile, and say "Welcome to           │
│       Smile Center! I'm Jamie. Are you here for your appointment        │
│       today?"                                                           │
│  ○ C) Point to the sign-in sheet without looking up.                   │
│  ○ D) Finish your current task first, then acknowledge them.           │
│                                                                          │
│  [Submit Answer]                                                        │
└──────────────────────────────────────────────────────────────────────────┘
```

After answering:

```
│  ✓ Correct! (B is the SGA standard)                                    │
│                                                                          │
│  WHY: The hospitality standard requires immediate acknowledgment.       │
│  Standing up signals respect. Using your name creates personal          │
│  connection. The nervous patient needs to feel welcomed, not processed. │
│                                                                          │
│  PLAYBOOK REFERENCE: First Impressions Playbook, Section 1.2           │
│  [View Playbook →]                                                      │
│                                                                          │
│  [Next Scenario →]                                                      │
```

### Step 3.3: Badge Issuance

After completing all 4 Bronze modules and passing the certification exam:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  🎉 CONGRATULATIONS, JAMIE!                                             │
│                                                                          │
│  You've earned your CCA Bronze Certification.                           │
│                                                                          │
│  ┌────────────────────────┐                                            │
│  │  [Credly Badge Image]  │                                            │
│  │  CCA Bronze Certified  │                                            │
│  │  Jamie L.              │                                            │
│  │  Smile Center Buckhead │                                            │
│  │  March 25, 2026        │                                            │
│  └────────────────────────┘                                            │
│                                                                          │
│  Your badge has been issued to your Credly account.                     │
│  Share it on LinkedIn, your email signature, or your resume.            │
│                                                                          │
│  Next: Silver Certification unlocked!                                   │
│  [Begin Silver Track →]  [Share Badge →]  [View Certificate]           │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**What happens behind the scenes:**
- Culture OS records certification completion
- Credly API call issues the digital badge
- Kafka event: `sgadental.ws3.training-completed` with `certification_tier: bronze, staff_id, practice_id`
- Consumed by:
  - Network Intelligence: Practice Deep Dive Culture tab updates CCA roster
  - AI Analysis Engine: certification gap alerts recalculate
  - Morning Huddle: next huddle includes recognition for Jamie's achievement
- iSolved HR record updated with certification status

**This interface is UNCHANGED from the original design.** CCA Academy is purely educational content and staff development tracking — zero PHI.

---

## Flow 4: Recognition & Rewards

### Step 4.1: Giving Recognition

Maria wants to recognize Jamie for completing Bronze certification. She opens the Recognition Feed from the Champion Hub (or from the Huddle screen):

```
┌──────────────────────────────┐
│  Recognition Feed             │
│  [Give Recognition +]         │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────┐    │
│  │  [Gen4 branded card]  │    │
│  │                       │    │
│  │  Lisa Torres → Jamie L│    │
│  │  "Welcome to the team │    │
│  │  and congrats on your │    │
│  │  Bronze cert! You're  │    │
│  │  already making a     │    │
│  │  difference."         │    │
│  │                       │    │
│  │  ♥ 8  💬 2           │    │
│  │  3 hours ago          │    │
│  └──────────────────────┘    │
│                              │
│  ┌──────────────────────┐    │
│  │  [Gen4 branded card]  │    │
│  │                       │    │
│  │  SYSTEM → Maria Santos│    │
│  │  "12-week huddle       │    │
│  │  streak! Longest in   │    │
│  │  the Southeast."      │    │
│  │                       │    │
│  │  ♥ 14  💬 5          │    │
│  │  Yesterday             │    │
│  └──────────────────────┘    │
│                              │
│  LEADERBOARD (This Month)    │
│  #1 Maria Santos    — 14 rec│
│  #2 Dr. Patel       — 11 rec│
│  #3 Lisa Torres     — 9 rec │
│                              │
│  BONUS POOL                  │
│  $250 remaining this month   │
│  Next distribution: April 1  │
│                              │
├──────────────────────────────┤
│ [Today] [Capture] [Huddle] [Hub] │
└──────────────────────────────┘
```

Maria taps **"Give Recognition +"**:

```
┌──────────────────────────────┐
│  Give Recognition             │
│                              │
│  Who: [Jamie L. ▾]           │
│  (searchable staff list)     │
│                              │
│  For: [Certification ▾]     │
│  Options: Hospitality,       │
│  Teamwork, Patient Care,     │
│  Certification, Above & Beyond│
│                              │
│  Message:                    │
│  ┌──────────────────────┐    │
│  │ Jamie, congrats on   │    │
│  │ earning your Bronze  │    │
│  │ certification! Your  │    │
│  │ phone skills are     │    │
│  │ improving every day. │    │
│  └──────────────────────┘    │
│                              │
│  [Cancel]  [Send Recognition]│
└──────────────────────────────┘
```

**What happens behind the scenes:**
- Recognition post created in Culture OS
- Gen4 brand template applied automatically (pre-cached from Content Engine Asset Registry — weekly batch sync, per the T-16 fix documented in cross-workstream flows)
- Post appears in the Recognition Feed for all Smile Center Buckhead staff
- Points awarded: Maria +3 (for giving), Jamie +5 (for receiving)
- Kafka event: `sgadental.ws3.recognition-created`
- Next morning's huddle display includes Jamie's recognition in the Recognition panel

**Brand template application (cross-workstream, invisible to user):**
Recognition cards use brand-compliant visual formatting (colors, fonts, logo) from the practice's brand universe (Gen4). These templates are cached locally in Culture OS (`ws3.brand_templates` table) via a weekly batch sync from the Content Engine's Asset Registry. The user never interacts with the Content Engine — the brand compliance is automatic and invisible.

**This interface is UNCHANGED from the original design.** Recognition is staff-to-staff interaction with no PHI involvement.

---

## Flow 5: H-Score Dashboard — Lisa Torres (Office Manager)

### Step 5.1: Detailed H-Score View (CO-5)

Lisa opens the H-Score Dashboard from her desktop (or expands the H-Score widget in the PWA Today screen).

```
┌──────────────────────────────────────────────────────────────────────────┐
│  H-Score Dashboard — Smile Center Buckhead            [30d | 60d | 90d] │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  COMPOSITE H-SCORE                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │  64 / 100          Network Average: 74                           │    │
│  │  ◐ Gauge (red zone)                                              │    │
│  │  ── 90-day trend showing decline over last 2 weeks ──           │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  SUB-SCORE BREAKDOWN                                                     │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │  CINNAMON PHONE QUALITY (35% weight)                             │    │
│  │  Score: 58 / 100     ▼20 from 78 (2 weeks ago)                  │    │
│  │  ████████████░░░░░░░░░░                                         │    │
│  │  ── trend sparkline ──                                          │    │
│  │                                                                  │    │
│  │  Contributing factors:                                           │    │
│  │  • Call answer rate: 82% (target: 95%)                          │    │
│  │  • Average hold time: 2:15 (target: <0:45)                      │    │
│  │  • Call quality rating: 3.1/5 (target: 4.0)                     │    │
│  │  • Trust transfer completion: 61% (target: 80%)                 │    │
│  ├──────────────────────────────────────────────────────────────────┤    │
│  │  SWELL REVIEWS (45% weight)                                      │    │
│  │  Score: 71 / 100     ▼3 from 74                                  │    │
│  │  ██████████████░░░░░░                                           │    │
│  │                                                                  │    │
│  │  Contributing factors:                                           │    │
│  │  • Google rating: 4.3 stars (142 reviews)                       │    │
│  │  • New reviews this month: 8 (target: 15)                       │    │
│  │  • Review response rate: 100% ✓                                 │    │
│  │  • Sentiment: 88% positive                                      │    │
│  ├──────────────────────────────────────────────────────────────────┤    │
│  │  DENTAL INTEL (20% weight)                                       │    │
│  │  Score: 74 / 100     ▼2 from 76                                  │    │
│  │  ██████████████░░░░░░                                           │    │
│  │                                                                  │    │
│  │  Contributing factors:                                           │    │
│  │  • Morning huddle compliance: 64% (target: 90%)                 │    │
│  │  • Schedule utilization: 78% (non-PHI aggregate)                │    │
│  │  • Operational protocol adherence: 82%                          │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  12-BEHAVIOR BREAKDOWN                                                  │
│  ┌────────────────────────────────────┬───────┬──────────────────────┐  │
│  │ Behavior                           │ Score │ Trend                │  │
│  │ Greeting & First Impression        │  82   │ ▲ improving          │  │
│  │ Phone Etiquette                    │  54   │ ▼ declining          │  │
│  │ Warm Handoff / Trust Transfer      │  48   │ ▼ declining          │  │
│  │ Scheduling Communication           │  71   │ → stable             │  │
│  │ Treatment Presentation             │  78   │ → stable             │  │
│  │ Financial Discussion               │  65   │ → stable             │  │
│  │ Follow-Up Commitment               │  69   │ → stable             │  │
│  │ Checkout Experience                │  73   │ ▲ improving          │  │
│  │ Post-Visit Follow-Up              │  62   │ → stable             │  │
│  │ Review Request                     │  71   │ ▲ improving          │  │
│  │ Team Collaboration                 │  80   │ → stable             │  │
│  │ Office Environment                 │  88   │ → stable             │  │
│  └────────────────────────────────────┴───────┴──────────────────────┘  │
│                                                                          │
│  AI INSIGHT: "Phone Etiquette and Warm Handoff scores are the primary  │
│  drivers of your H-Score decline. Both correlate with the new front    │
│  desk hire (Jamie L., started 3/3). Recommend accelerated CCA Bronze   │
│  enrollment and 1:1 phone coaching."                                    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**Data sources:**
- All H-Score data comes from three vendor integrations: Cinnamon (phone quality analysis), Swell (review management and sentiment), Dental Intel (operational compliance). Culture OS consumes their aggregate APIs and computes the composite score.
- The 12-behavior breakdown maps vendor metrics to the SGA hospitality behavior framework. Each behavior draws from one or more vendor sub-metrics.
- AI Insight is generated by the Culture OS analysis engine correlating H-Score changes with staff data (new hires, certification gaps, huddle compliance).

**This interface is UNCHANGED from the original design.** H-Score is entirely derived from aggregate vendor scores and staff data — zero PHI.

---

## Flow 6: Playbook Library (CO-6)

### Step 6.1: Browsing Playbooks

Any staff member can access playbooks from the PWA Settings menu or directly in the Huddle screen.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Playbook Library                                    [Search playbooks]  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  CATEGORIES                                                              │
│  [Phone Scripts] [Hospitality] [Follow-Up] [Onboarding] [Clinical Ops] │
│                                                                          │
│  PHONE SCRIPTS                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  Phone Scripts v2.4                           Updated: March 15    │  │
│  │                                                                    │  │
│  │  New Patient Call                                                  │  │
│  │  ────────────────────────                                         │  │
│  │  1. Greeting: "Thank you for calling [Practice].                  │  │
│  │     This is [Name], how can I help you today?"                    │  │
│  │                                                                    │  │
│  │  2. If insurance question:                                        │  │
│  │     "Let me look into that for you..."                            │  │
│  │                                                                    │  │
│  │  3. If scheduling:                                                │  │
│  │     "I'd love to get you scheduled. What day and time work        │  │
│  │      best for you? Dr. [Name] has availability on..."            │  │
│  │                                                                    │  │
│  │  [What Changed in v2.4 ▾]                                        │  │
│  │  ┌──────────────────────────────────────────────────────────┐     │  │
│  │  │  + Added: "Let me introduce you to..." warm handoff      │     │  │
│  │  │    language for transfers to treatment rooms              │     │  │
│  │  │  ~ Changed: Insurance verification script updated for    │     │  │
│  │  │    new carrier partnerships                               │     │  │
│  │  └──────────────────────────────────────────────────────────┘     │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  GRANT TRAINING VIDEOS                                                  │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  [▶] The First 7 Seconds — 15 min                                │  │
│  │  [▶] The Warm Handoff — 18 min                                   │  │
│  │  [▶] Handling Objections with Empathy — 22 min                   │  │
│  │  [▶] The 5-Star Review Conversation — 12 min                     │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  PRACTICE-TYPE PROTOCOLS                                                │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │  Your practice type: General Practice                              │  │
│  │  Applicable protocols: GP Standard Operations v3.1                │  │
│  │  [View Protocol →]                                                │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
```

**Offline behavior:** Full playbook content is cached in the service worker (cache-first strategy, 7-day TTL, updated on-demand). Staff can read any playbook without internet connectivity.

**This interface is UNCHANGED from the original design.** Playbooks are reference documents — zero PHI.

---

## Flow 7: Trust Transfer Tracker (CO-7)

### Step 7.1: Practice-Level Aggregate View

Lisa opens the Trust Transfer Tracker to investigate the declining trust transfer scores identified in the H-Score dashboard.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Trust Transfer Tracker — Smile Center Buckhead           [This Month ▾]│
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  PRACTICE AGGREGATE SCORE                                                │
│  Trust Transfer: 61 / 100       Network Average: 78                    │
│  ── 90-day trend showing decline ──                                    │
│                                                                          │
│  SCORE BY TEAM MEMBER                                                    │
│  ┌────────────────────┬─────────┬────────┬───────────────────────────┐  │
│  │ Team Member        │ Score   │ Trend  │ Detail                    │  │
│  │ Dr. Patel          │ 91      │ → stbl │ [View Detail]             │  │
│  │ Maria Santos       │ 85      │ ▲ +3  │ [View Detail]             │  │
│  │ Lisa Torres        │ 74      │ → stbl │ [View Detail]             │  │
│  │ Dr. Kim            │ 72      │ → stbl │ [View Detail]             │  │
│  │ Front Desk Team    │ 48      │ ▼ -12 │ [View Detail]             │  │
│  │   Jamie L. (new)   │ 34      │ — new  │ [View Detail]             │  │
│  │   Sandra M.        │ 62      │ → stbl │ [View Detail]             │  │
│  └────────────────────┴─────────┴────────┴───────────────────────────┘  │
│                                                                          │
│  TRUST TRANSFER BEHAVIORS MEASURED                                       │
│  • Verbal warm handoff ("Let me introduce you to...")                   │
│  • Patient name usage during transfer                                   │
│  • Context sharing ("I mentioned to Dr. Patel that...")                 │
│  • Confirmation of comfort ("Is there anything else...?")              │
│                                                                          │
│  AI INSIGHT: "Jamie L.'s trust transfer score of 34 is significantly   │
│  below the practice average of 61. Primary gap: warm handoff language  │
│  is absent in 78% of analyzed interactions. Recommend Module 3 of      │
│  CCA Bronze (Trust Transfers) as immediate priority."                   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Step 7.2: Individual Staff Detail — The ONE Deferred Feature

Lisa clicks **"View Detail"** for Jamie L.:

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Trust Transfer Detail — Jamie L.                                        │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  AGGREGATE SCORES (from Cinnamon analysis)                              │
│  Overall trust transfer: 34 / 100                                       │
│  Warm handoff completion: 22%                                           │
│  Patient name usage: 45%                                                │
│  Context sharing: 18%                                                   │
│  Comfort confirmation: 31%                                              │
│                                                                          │
│  TREND (17 days since hire)                                              │
│  ── sparkline showing slight upward trend from 28 to 34 ──             │
│                                                                          │
│  TRAINING RECOMMENDATIONS                                                │
│  ⓘ CCA Bronze Module 3: Trust Transfers — Not started                  │
│  ⓘ Grant Video: "The Warm Handoff" — Not watched                       │
│  ⓘ Playbook: Trust Transfer Quick Reference — Not viewed               │
│  [Assign Training →]                                                    │
│                                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  INDIVIDUAL CALL ANALYSIS                                                │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │                                                                  │    │
│  │  [illustration: headphones with waveform]                        │    │
│  │                                                                  │    │
│  │  Individual call transcript analysis lets you listen to          │    │
│  │  specific calls and see NLP-tagged trust transfer moments.       │    │
│  │                                                                  │    │
│  │  This feature requires call recording access integration.        │    │
│  │  Coming in Phase B.                                              │    │
│  │                                                                  │    │
│  │  In the meantime, aggregate scores above show Jamie's overall    │    │
│  │  trust transfer performance based on Cinnamon's analysis of      │    │
│  │  all calls.                                                      │    │
│  │                                                                  │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

**This is the ONE deferred feature in all of Culture OS.**

**What's available in Phase A:**
- Aggregate trust transfer scores per staff member (from Cinnamon's API — these are computed scores, not raw call data)
- Trend over time per staff member
- Behavior breakdown (warm handoff completion rate, name usage rate, etc.)
- AI-generated training recommendations correlated with score gaps
- Links to relevant CCA modules and playbooks

**What's deferred to Phase B:**
- Individual call transcript drill-down: listening to specific calls, reading transcripts, seeing NLP-tagged trust transfer moments within the conversation
- This requires access to call recordings stored in Cinnamon's platform, which contain verbal PHI (patient names, appointment details, treatment discussions)

**How the UI handles the absence:**
- The "Individual Call Analysis" section shows a purposeful empty state with an illustration and explanation
- It explains what the feature does and when it will be available
- It explicitly notes that the aggregate scores above still provide useful data
- The empty state is designed as a Phase B invitation, not a broken feature

**Why aggregate scores are still useful without individual call drill-down:**
Lisa can still see that Jamie's trust transfer score is 34 (far below the 61 practice average). She can see exactly which behaviors are weakest (context sharing at 18%, warm handoff completion at 22%). She can assign targeted training. She does not need to listen to individual calls to take action — the aggregate patterns are sufficient for coaching and development planning.

**Phase B Addition:** When call recording access is activated, the empty state is replaced with a scrollable list of recent calls:

```
INDIVIDUAL CALLS (Phase B)
┌────────┬──────────┬──────────┬───────────────────────────┐
│ Time   │ Duration │ TT Score │ Key Moments               │
│ 9:15am │ 3:12     │ 42       │ ⚠ No warm handoff at 1:45│
│ 9:48am │ 2:05     │ 68       │ ✓ Name used, no context  │
│ 10:22am│ 4:30     │ 31       │ ⚠ No trust transfer      │
└────────┴──────────┴──────────┴───────────────────────────┘
[Listen] [Read Transcript] buttons per call
```

---

## Flow 8: Culture Admin Console — HQ Culture Team (CO-8)

### Step 8.1: Curriculum Management

The HQ Culture Team manages the CCA curriculum, deploys playbook updates, configures huddle content, and manages the champion network.

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Culture Admin Console                                 HQ Culture Team   │
├──────────────────────────────────────────────────────────────────────────┤
│  │                                                                       │
│  │ Curriculum   ┌──────────────────────────────────────────────────┐     │
│  │              │  CCA PROGRAM STATUS                              │     │
│  │ Playbooks    │                                                  │     │
│  │              │  Network-wide Certification                      │     │
│  │ Huddles      │  Gold: 42 staff | Silver: 128 | Bronze: 267    │     │
│  │              │  In Progress: 89 | Not Started: 134             │     │
│  │ Champions    │                                                  │     │
│  │              │  This Month                                      │     │
│  │ Rewards      │  Certifications earned: 18 (▲ from 12 last mo) │     │
│  │              │  Modules completed: 214                          │     │
│  │ Settings     │  Avg completion time (Bronze): 11 days          │     │
│  │              │                                                  │     │
│  │              │  MANAGE CURRICULUM                               │     │
│  │              │  [Edit Bronze Modules]                           │     │
│  │              │  [Edit Silver Modules]                           │     │
│  │              │  [Edit Gold Modules]                             │     │
│  │              │  [Upload New Training Video]                     │     │
│  │              │  [Edit Assessment Questions]                     │     │
│  │              └──────────────────────────────────────────────────┘     │
│  │                                                                       │
└──────────────────────────────────────────────────────────────────────────┘
```

### Step 8.2: Playbook Deployment

```
│  PLAYBOOK MANAGEMENT                                                    │
│                                                                          │
│  ┌────────────────────┬─────────┬──────────────┬────────────────────┐   │
│  │ Playbook           │ Version │ Deployed     │ Adoption           │   │
│  │ Phone Scripts      │ v2.4    │ 3/15 (42/43) │ 35% accessed      │   │
│  │ Hospitality Guide  │ v3.1    │ 3/01 (43/43) │ 78% accessed      │   │
│  │ Trust Transfer     │ v1.2    │ 2/15 (43/43) │ 82% accessed      │   │
│  │ Onboarding Checklist│ v2.0   │ 1/20 (43/43) │ 91% accessed      │   │
│  └────────────────────┴─────────┴──────────────┴────────────────────┘   │
│                                                                          │
│  ⚠ Phone Scripts v2.4: 8 practices have not accessed the update        │
│  [Send Reminder Notification]                                           │
│                                                                          │
│  [Create New Playbook]  [Edit Existing]  [View Adoption Analytics]     │
```

### Step 8.3: Champion Network Management

```
│  CHAMPION NETWORK                                                        │
│                                                                          │
│  Active Champions: 28 across 43 practices                               │
│  Practices without champion: 15                                         │
│                                                                          │
│  TOP PERFORMERS                                                          │
│  ┌────────────────┬──────────────────┬────────┬──────────────────────┐  │
│  │ Champion       │ Practice         │ Score  │ Status               │  │
│  │ Maria Santos   │ Smile Ctr Bkhd   │ 186 pts│ Active (12-wk strk) │  │
│  │ David Rodriguez│ Metro Dental Mid  │ 174 pts│ Active (8-wk strk)  │  │
│  │ Ashley Williams│ Family Dental SS  │ 163 pts│ Active (6-wk strk)  │  │
│  └────────────────┴──────────────────┴────────┴──────────────────────┘  │
│                                                                          │
│  INACTIVE CHAMPIONS (>14 days no activity)                              │
│  ┌────────────────┬──────────────────┬────────┬──────────────────────┐  │
│  │ Sarah M.       │ Family Dental SS  │ — pts  │ On leave (iSolved)  │  │
│  │ Chris P.       │ Bright Smiles DA  │ 42 pts │ Last active: 3/2    │  │
│  └────────────────┴──────────────────┴────────┴──────────────────────┘  │
│                                                                          │
│  [Assign New Champion]  [Send Champion Brief]  [View Regional Report]  │
```

### Step 8.4: Huddle Content Configuration

```
│  HUDDLE CONTENT MANAGEMENT                                               │
│                                                                          │
│  This Week's Focus Behavior: Warm Handoffs                              │
│  Next Week's Scheduled: Review Request Conversations                    │
│                                                                          │
│  BEHAVIOR ROTATION SCHEDULE                                              │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │ Week of 3/17: Warm Handoffs                                      │    │
│  │ Week of 3/24: Review Request Conversations                       │    │
│  │ Week of 3/31: Phone Etiquette (Greeting & Hold)                  │    │
│  │ Week of 4/7:  Financial Discussion Comfort                       │    │
│  │ ...                                                               │    │
│  └──────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  IMPACT STORY CONFIGURATION                                              │
│  Source: [Auto-generated from culture/brand metrics ▾]                  │
│  Override: [Custom story text]                                          │
│                                                                          │
│  Preview next huddle content:                                           │
│  [Preview Friday Huddle →]                                              │
│                                                                          │
│  HUDDLE COMPLIANCE                                                       │
│  ┌──────────────────────────────────────────────────────────────────┐    │
│  │ Network completion rate (this week): 87%                         │    │
│  │ Practices with zero huddles: 6                                   │    │
│  │ [View Non-Compliant Practices]                                   │    │
│  └──────────────────────────────────────────────────────────────────┘    │
```

**This entire interface is UNCHANGED from the original design.** Culture admin functions manage curricula, playbooks, champions, and huddle content — all staff-facing, zero PHI.

---

## Flow 9: Cross-Workstream Data Flows

### Step 9.1: H-Score Events Flow to WS1 Executive Dashboard

```
Culture OS (WS3)                              Network Intelligence (WS1)
┌─────────────────────────┐                   ┌──────────────────────────┐
│ H-Score Calculation      │                   │ Executive Dashboard      │
│ Daily batch at 6 AM ET   │                   │                          │
│                          │                   │ KPI Bar: Avg H-Score     │
│ Cinnamon API → 35%       │   Kafka event:    │ Portfolio Heatmap:       │
│ Swell API → 45%          │──────────────────▸│   H-Score color layer   │
│ Dental Intel API → 20%   │   sgadental.ws3.  │ Practice Deep Dive:     │
│                          │   hscore-         │   Culture tab            │
│ Composite: 64            │   calculated      │ AI Alerts: H-Score      │
│ Sub-scores: 58, 71, 74   │                   │   anomaly detection     │
│ Practice: sga-0142       │                   │ Intervention Queue:     │
│                          │                   │   Culture Risk ranking  │
└─────────────────────────┘                   └──────────────────────────┘
```

**Latency budget:** H-Score calculated at 6 AM ET. Must appear on Executive Dashboard by 6:30 AM ET (30-minute max latency). In practice, Kafka consumer processes within seconds.

**Stale data handling:** If H-Score is >48 hours old, dashboards show "Updated 2 days ago" in 11px muted text. No error state — just transparency.

### Step 9.2: Champion Roster Flows to WS2 Content Engine

```
Culture OS (WS3)                              Content Engine (WS2)
┌─────────────────────────┐                   ┌──────────────────────────┐
│ Champion Assignment       │                   │ Champions Section        │
│                          │                   │                          │
│ Maria Santos assigned     │   Kafka event:    │ Champion leaderboard     │
│ as champion for           │──────────────────▸│ includes Maria's         │
│ Smile Center Buckhead     │   sgadental.ws3.  │ content metrics          │
│                          │   champion-        │                          │
│ Champion data:            │   assigned        │ Capture briefs routed    │
│ - staff_id               │                   │ to Maria as champion     │
│ - practice_id            │                   │                          │
│ - role: primary_champion │                   │ Champion content          │
│                          │                   │ scorecard displayed      │
└─────────────────────────┘                   └──────────────────────────┘

Content Engine (WS2)                          Culture OS (WS3)
┌─────────────────────────┐                   ┌──────────────────────────┐
│ Content Engagement        │                   │ Champion Hub             │
│                          │                   │                          │
│ Maria's capture stats:    │   Kafka event:    │ Content Score: 88        │
│ - 6 captures last week   │──────────────────▸│ Captures: 6 last week   │
│ - Quality: 4.0/5         │   sgadental.ws2.  │ Streak: 8 weeks         │
│ - Streak: 8 weeks        │   content-        │ Published: 4 this month │
│ - Published: 4 this mo   │   engagement-     │ Reach: 2.3K             │
│                          │   updated         │                          │
└─────────────────────────┘                   └──────────────────────────┘
```

### Step 9.3: Practice Enrollment Triggers Culture Onboarding

```
Network Intelligence (WS1)                    Culture OS (WS3)
┌─────────────────────────┐                   ┌──────────────────────────┐
│ Enrollment Pipeline       │                   │ Auto-Onboarding          │
│                          │                   │                          │
│ "Dental Arts Decatur"     │   Kafka event:    │ 1. Create practice entry │
│ status: active            │──────────────────▸│ 2. Assign huddle template│
│ milestone: fully_active   │   sgadental.ws1.  │ 3. Generate CCA Bronze   │
│                          │   onboarding-     │    invitations (16 staff)│
│                          │   status-changed  │ 4. Assign default        │
│                          │                   │    champion (if known)   │
│                          │                   │ 5. Schedule first huddle │
└─────────────────────────┘                   └──────────────────────────┘
```

### Cross-Workstream Data Flow Summary Table

| Data Element | Source (WS) | Consumer (WS) | Kafka Event | PHI Status | Phase A Active |
|---|---|---|---|---|---|
| H-Score composite + sub-scores | WS3 Culture OS | WS1 Network Intelligence | `ws3.hscore-calculated` | Non-PHI | Yes |
| CCA certification status | WS3 Culture OS | WS1 Network Intelligence | `ws3.training-completed` | Non-PHI | Yes |
| Huddle completion | WS3 Culture OS | WS1 Network Intelligence | `ws3.huddle-completed` | Non-PHI | Yes |
| Champion assignment | WS3 Culture OS | WS2 Content Engine | `ws3.champion-assigned` | Non-PHI | Yes |
| Champion story submission | WS3 Culture OS | WS2 Content Engine | `ws3.champion-story-submitted` | Non-PHI | Yes |
| Content engagement stats | WS2 Content Engine | WS3 Culture OS (Champion Hub) | `ws2.content-engagement-updated` | Non-PHI | Yes |
| Content capture brief | WS2 Content Engine | WS3 Culture OS (Champion Hub) | `ws2.capture-brief-generated` | Non-PHI | Yes |
| Brand templates | WS2 Content Engine (Asset Registry) | WS3 Culture OS (Recognition Feed) | Weekly batch sync (not real-time) | Non-PHI | Yes |
| Enrollment event | WS1 Practice Platform | WS3 Culture OS | `ws1.onboarding-status-changed` | Non-PHI | Yes |
| Staff roster | Shared (Practice Data Hub / iSolved) | All workstreams | PostgreSQL shared table | Non-PHI | Yes |

**Every single cross-workstream data flow for Culture OS is active in Phase A.** No cross-workstream touchpoints are deferred.

---

## What's Different from the Original Design — Summary

| Area | Original (Full PHI) | Phase A (PHI-Free) | Impact |
|---|---|---|---|
| **Morning Huddle Impact Story** | Funnel metrics ("Show rate improved 5%") | Culture/brand metrics ("Google rating improved from 4.1 to 4.3") | One content source changes; display identical |
| **Trust Transfer Tracker — individual calls** | Drill-down to individual call transcripts with NLP-tagged moments | Empty state: "Coming in Phase B" | Aggregate scores still available; coaching recommendations still generated |
| **All other interfaces** | Fully functional | Fully functional | No difference whatsoever |

## The Phase A Culture OS Value Proposition

Culture OS is the most complete workstream in Phase A. It delivers the full hospitality transformation program:

1. **Daily rituals**: Morning huddles with H-Score, focus behaviors, recognition, impact stories — building culture through consistent daily practice
2. **Continuous development**: CCA Academy with Bronze/Silver/Gold certification tracks, Grant training videos, scenario assessments, Credly digital badges
3. **Champion program**: Unified scorecard combining culture and content metrics, capture missions, leaderboards, cohort competition
4. **Recognition system**: Staff-to-staff recognition with brand-compliant templates, leaderboards, bonus pool tracking
5. **Performance visibility**: H-Score dashboard with 12-behavior breakdown, trust transfer tracking (aggregate), trend analysis, AI-generated coaching insights
6. **Knowledge management**: Playbook library with version control, adoption tracking, offline access
7. **HQ administration**: Curriculum management, playbook deployment, champion network oversight, huddle content configuration
8. **Cross-workstream integration**: H-Score flows to executive dashboards, champion roster flows to content engine, enrollment events trigger culture onboarding

The only missing capability is the ability to listen to individual call recordings for coaching purposes. Everything else — the measurement, the training, the rituals, the recognition, the administration — operates at full capacity. Culture OS proves the thesis that hospitality culture can be built and measured without touching a single patient record.

---

## Phase B Addition Callouts

| Phase A State | Phase B Addition | Trigger |
|---|---|---|
| Morning Huddle Impact Story: culture/brand metrics | +funnel stories ("Show rate improved 5% since confirmation texts") | `pms_integration_active` flag |
| Trust Transfer Tracker: aggregate scores per staff member | +individual call drill-down with transcript, NLP tagging, playback | Call recording access activation |
| H-Score: computed from vendor aggregate APIs | No change — H-Score calculation is identical in both phases | N/A |
| All other interfaces | No change | N/A |

---

*This journey specification covers all WS3 Culture OS flows in Phase A (PHI-free) configuration. All 8 of 8 interfaces ship. Culture OS is almost entirely PHI-free by design — it was built around aggregate vendor scores and staff development data, not patient records. The single deferred feature (individual call transcript analysis) is handled with a purposeful empty state while aggregate scores continue to provide actionable coaching data. Culture OS is the strongest proof point that SGA's platform delivers immediate, complete value without PHI.*
