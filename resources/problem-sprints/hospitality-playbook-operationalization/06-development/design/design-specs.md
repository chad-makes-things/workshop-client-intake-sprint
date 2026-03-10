# UX Design Specifications: The Hospitality Playbook Platform

**Sprint**: Hospitality Playbook Operationalization
**Solution**: The Hospitality Playbook Platform
**Design Phase**: Phase 1 (Morning Huddle OS + CCA Academy Bronze + H-Score Engine)

---

## Design Philosophy

The Hospitality Playbook Platform is the infrastructure for a hospitality brand. It should feel like one.

**Not this**: An LMS. An HR compliance portal. A dental practice management tool. An operations dashboard.

**This**: A premium culture platform — warm, human, purposeful, and slightly aspirational. Something that feels closer to the Ritz Carlton app than to a dental scheduling system. The platform is delivering "Caring with Charisma" — it must embody it in every pixel.

**Three design principles**:
1. **Warm and premium, not clinical and cold** — Gold and dark tones, not blue and white. Every surface should feel intentional, not utilitarian.
2. **Zero friction for daily use** — The morning huddle loads in one tap. Module videos play without login friction. The H-Score is readable in 3 seconds. Daily-use flows must be effortless.
3. **Achievement feels real** — Certification milestones, score improvements, leaderboard positions — these are human moments of recognition, not database confirmations. Design them accordingly.

---

## Color System

```
PRIMARY BACKGROUNDS:
  Background base:        #0f0f0f  (near-black — warm, not cool)
  Card surface:           #171717
  Card surface elevated:  #1e1e1e
  Card surface active:    #252525

ACCENT — WARM GOLD:
  Gold primary:           #c9a84c  (the brand accent — warm, premium)
  Gold light:             #e8c96a  (hover states, highlights)
  Gold dim:               #8a6f30  (borders, subtle elements)
  Gold glow:              rgba(201, 168, 76, 0.12)  (background glows)

TEXT:
  Text primary:           #f0ece4  (warm white — not pure #fff)
  Text secondary:         #8a857c  (muted body text)
  Text tertiary:          #5a5550  (labels, metadata)

SEMANTIC COLORS:
  Success / Exceptional:  #4a9e6e
  Warning / Developing:   #c47a35
  Danger / Emerging:      #c05a4a
  Info / Established:     #c9b94c

BORDERS:
  Standard border:        #2a2825
  Gold border:            #3a3020
  Active border:          rgba(201, 168, 76, 0.4)

H-SCORE LEVEL COLORS:
  Emerging (0-39):        #c05a4a  (red)
  Developing (40-59):     #c47a35  (orange)
  Established (60-74):    #c9b94c  (warm yellow)
  Exceptional (75-89):    #4a9e6e  (green)
  Elite (90-100):         #c9a84c  (gold — crown level)
```

---

## Typography

```
FONT STACK:
  Primary: -apple-system, BlinkMacSystemFont, 'Segoe UI', Georgia, serif
  (System serif stack — reads as human and credentialed, not corporate and tech)

SCALE:
  Display:    clamp(40px, 5vw, 64px) — hero headlines only
  H1:         clamp(28px, 4vw, 42px) — section titles
  H2:         24px — component titles
  H3:         20px — card titles
  H4:         14px / 600 weight / 0.05em letter-spacing / UPPERCASE — labels
  Body:       16px / 1.75 line-height
  Body small: 14px / 1.65 line-height
  Caption:    12px / 1.5 line-height
  Label:      11px / 0.12em letter-spacing / UPPERCASE / 600 weight

WEIGHT USAGE:
  Headlines:  300-400 (light — premium, not heavy)
  Body:       400
  UI labels:  600
  Never:      700+ except for explicit emphasis
```

---

## 8 Key Screens

### Screen 1: Practice Dashboard

**What it is**: The home screen for a practice owner or Champion. H-Score prominently displayed, today's huddle status, staff certification summary, and this week's opportunity.

**Layout**: Two-column (desktop); single column stacked (mobile/tablet)

**Left column**:
- H-Score dial (large SVG gauge, 0-100, color-coded to level)
- Level label with trend: "Established — ↑7 this month"
- Three component bars (Swell 45%, Cinnamon 35%, Dental Intel 20%) each with sub-score breakdown on tap
- "This Week's Opportunity" recommendation card — gold border, specific action text

**Right column**:
- Today's Huddle card: status (Complete / Not yet run / Missed), streak counter, "Start Huddle" CTA
- Staff Certification card: Bronze % complete, Silver % complete, Gold count; list of staff with certification status badges
- Recent Reviews card: last 3 Swell reviews with sentiment badges; trust transfer signals flagged with a gold marker

**Mobile layout**: H-Score dial full-width, then cards stack vertically. "Start Huddle" button sticky at bottom of screen.

**Critical design requirement**: The H-Score number must be readable in < 1 second from across a break room (high contrast, large type, clear level color). The dial should be 180px+ diameter on tablet.

**Emotional tone**: Accountability without anxiety. A 54/100 score should feel like "here's how to grow" not "here's your grade."

---

### Screen 2: Morning Huddle Player

**What it is**: The full-screen experience that runs during the morning huddle. Displayed on a tablet or monitor in the break room.

**Layout**: Full-screen (16:9 optimized). Zero chrome during playback. Minimal controls.

**Structure (matches the 5-minute template)**:
1. Culture Moment (0:00-1:00): Video frame full-screen. Grant or Champion on camera. Autoplay. Skip not available (prevents cherry-picking).
2. Yesterday's Wins (1:00-2:00): Dark card slides in. Swell review displayed large. Star rating. Patient first name. Gold "Trust Transfer Signal" badge if detected. Animated in gently.
3. Today's Focus (2:00-3:30): Behavior title, week's theme, scripted exercise prompt. 30-second team activity card. Large text, readable from 10 feet.
4. Accountability (3:30-4:30): Trust Transfer Commitment tap (one large button). Certification spotlight (if any earned since last huddle). Environmental check (3-item checklist tappable).
5. Intention (4:30-5:00): The day's intention statement. Large serif text. Slow fade-in. Read aloud by facilitator.

**Post-huddle**: "Mark Complete" button. Wow story submission option. Tomorrow's theme preview.

**Key interaction**: Single tap to advance between segments. No menus. No navigation. No distractions. This is a ceremony, not a dashboard.

**Critical design requirement**: Must be usable by someone who has never seen the platform before. Zero learning curve. The first huddle experience must be zero-friction.

**Emotional tone**: Morning ritual. Warm, energizing, communal. Not a slideshow.

---

### Screen 3: Staff Certification Progress Page

**What it is**: A staff member's personal certification home — their progress through the Bronze track, upcoming modules, and their credential record.

**Layout**: Single column, scrollable.

**Top**: Name, practice name, certification tier badge (Bronze / Silver / Gold / None), days since enrolled.

**Certification Journey Visualization**: A horizontal path (desktop) or vertical progress (mobile) showing the 3 tiers. Bronze illuminated (current tier). Silver and Gold shown as aspirational future states. Each tier shows: requirements remaining, estimated time to complete, reward earned.

**Module Grid**: 10 module cards in a 2×5 grid (desktop) or 1×10 list (mobile). Each card shows: module number and title, completion status (locked / in progress / complete with score), duration, and if complete: completion date and score.

**Credential Section**: If Bronze certified — shows the issued certificate, the Credly badge widget, LinkedIn share button, and verification URL. The jacket icon for Gold tier with "14 months remaining" progress indicator.

**Critical design requirement**: The progress toward Gold should always be visible — even from day 1. The jacket is the aspiration. Show it early, show the path.

**Emotional tone**: Achievement and aspiration. This is a personal milestone tracker, not an assignment queue.

---

### Screen 4: Module Viewer

**What it is**: The learning experience for a single CCA certification module. Used on phone or tablet, typically during a break.

**Layout**: Full-screen video (top half), controls and content (bottom half).

**Video section**: Plays autoplay. Grant on camera. 4-6 minutes. Closed captions on by default. Playback speed control (0.75x, 1x, 1.25x). No scrubbing until second view (prevents skip-ahead on first watch).

**After video**: Scenario card slides up from bottom. Question text (2-4 sentences). 2-3 response options as large tappable cards. On selection: immediate feedback — correct/incorrect, Grant's audio explanation (3-5 seconds), then "Next" button. On incorrect: can retry once.

**Progress indicator**: Module number "3 of 10" with a thin progress bar. Breadcrumb of completed modules.

**Completion screen**: Module complete animation (subtle particle effect, gold color). Score displayed. "Continue to Module 4" or "Back to my progress."

**Critical design requirement**: The video and scenario must feel like a single continuous experience, not a video followed by a quiz. The transition should be smooth — the scenario slides up without leaving the video context.

**Emotional tone**: Learning, not testing. The scenario is a "let's practice together" moment, not an exam.

---

### Screen 5: Certification Achievement Screen

**What it is**: The moment a staff member earns their Bronze certification. This is a designed milestone — it should feel earned and meaningful.

**Layout**: Full-screen celebration. Dark background. Gold certificate centered.

**Elements**:
- Animated gold particle effect (subtle, 3 seconds)
- Certificate displayed: "[Staff Name] has earned the CCA Bronze Certification — Caring with Charisma Academy"
- Date, practice name, unique verification URL
- Three action buttons, stacked:
  1. "Share to LinkedIn" (primary, gold) — opens Credly LinkedIn publish flow
  2. "Send to Email" (secondary) — sends PDF certificate
  3. "See My Progress" (tertiary) — returns to progress page
- Below certificate: "What's next?" card showing Silver requirements and the Gold jacket image

**Critical design requirement**: The certificate must look like a real certificate — not a "completion badge." Serif font, formal layout, the CCA seal visible. Staff should want to screenshot it.

**Notification to practice owner**: Triggered simultaneously — a push/email notification: "[Staff Name] has earned their CCA Bronze certification. Celebrate at tomorrow's huddle."

**Emotional tone**: This is a moment. Design it like one.

---

### Screen 6: Network Leaderboard

**What it is**: The competitive ranking of practices by H-Score within a region or across the network. Visible in the monthly huddle segment and in the practice dashboard.

**Layout**: Two-tab view (Regional / Network). List of ranked practices.

**Each row**: Rank number, practice name (anonymized to non-owners — "Practice in Phoenix, AZ"), H-Score with level color badge, trend arrow (up/down/flat), and "Most Improved" badge where applicable.

**Current practice row**: Always visible, even if not in top 25. Highlighted with gold border. Shows exact rank.

**Top 3**: Larger cards with H-Score prominently displayed. Month-over-month improvement shown.

**Filter options**: Current month / Last 3 months / Since launch. Practice type filter (rural / suburban / urban / etc.) — so rural practices can see themselves ranked against peers.

**Critical design requirement**: The leaderboard must not feel punitive. Bottom-ranked practices should not be shown unless a practice owner is looking at their own ranking. The public leaderboard shows only top 25 (anonymized positions 6-25).

**Emotional tone**: Healthy competition, not surveillance.

---

### Screen 7: Trust Transfer Daily Checklist

**What it is**: A lightweight in-huddle confirmation — one tap to commit the team to the trust transfer protocol for today. Appears during the Accountability segment of the huddle player.

**Layout**: Single full-screen card during huddle. Large, readable from 10 feet.

**Elements**:
- Title: "Today's Trust Transfer Commitment"
- Statement: "Every patient handoff today will include a warm introduction to the next provider."
- Three role buttons (optional): "Front Desk ready," "Clinical Team ready," "Doctor ready"
- Large "Commit" button (primary gold)
- On commit: subtle checkmark animation, logged to system

**Post-huddle view**: In the post-huddle review (accessible after completion), the commitment appears with a timestamp and a note if yesterday's reviews contained trust transfer signals.

**Critical design requirement**: This must feel like a team commitment ritual, not a checkbox. The design should reinforce the collective nature — the whole team taps, not one person filing a form.

**Emotional tone**: Intentional, communal, quick.

---

### Screen 8: Champion Hub

**What it is**: The Champion's dedicated space — their facilitation resources, wow story queue, regional community, and upcoming calls.

**Layout**: Three-column dashboard (desktop); tabbed (mobile): Resources / Community / My Impact

**Resources tab**:
- Facilitation guide for this week (downloadable)
- Grant's latest Culture Brief (video, 10 min)
- Weekly Culture Card (this week's hospitality theme one-pager)
- Wow story submission form (prominent)
- Upcoming Regional Champion Call details (date, call link, agenda)

**Community tab**:
- Regional cohort list (15-20 Champions in region)
- Network wow story feed (stories shared this week, attributed to Champions)
- Champion directory search

**My Impact tab**:
- Huddles facilitated (count, streak)
- Staff onboarded through Bronze (count)
- Wow stories shared (count, reach in practices)
- My progress toward Gold certification
- My Champion tier (Candidate / Certified / Master)

**Critical design requirement**: The Champion Hub must feel like an elite club, not a staff portal. The resources are curated, not overwhelming. The community section creates real connection — photos (optional), names, practice locations visible.

**Emotional tone**: Pride, community, purpose.

---

## Component Library

### Video Player
- Full-screen capable (tablet/break room mode)
- Warm loading state (gold shimmer, not blue spinner)
- Captions on by default
- No scrubbing bar on first watch (forced full viewing)
- Playback speed: subtle control, not prominent
- After completion: smooth transition to scenario — no "quiz" framing

### H-Score Gauge
- SVG arc gauge (0-100 range)
- Color transitions smoothly between levels on animation
- Number displayed large inside gauge
- Level label below number
- Trend arrow (↑↓→) with magnitude ("+7 this month")
- On tap: expands to show component breakdown

### Certification Badge
- Three variants: Bronze (copper tones), Silver (silver tones), Gold (gold tones + glow)
- Used in: staff profiles, certification progress, leaderboard rows, Champion hub
- Accessible: aria-label includes tier name and certification date
- Micro-animation on first display (scale in, 200ms)

### Huddle Card
- Shows: date, theme, completion status, duration
- Completion status variants: Complete (green border), In Progress (gold border), Missed (red border), Not Yet Run (neutral)
- Streak indicator: flame icon + count (matches streak milestone milestones at 7, 14, 30, 60, 100)

### Leaderboard Row
- Rank number (gold if top 3), practice name, H-Score badge, trend arrow
- Current practice highlighted with gold left border
- "Most Improved" badge: subtle, warm, not flashy

### Review Surface
- Patient first name (if available)
- Star rating with filled/unfilled stars
- Review text (truncated at 120 chars with expand)
- Trust Transfer Signal badge: warm gold tag, "Trust Transfer Signal Detected"
- Staff mention badge: name highlighted in gold text

---

## Accessibility Requirements

- WCAG 2.1 AA compliance minimum
- Tap targets minimum 44×44px (critical for break room tablet use with thick fingers)
- Color is never the only signal — all status information has text/icon backup
- Captions on all video content by default (dental break rooms are loud)
- High contrast mode supported — gold accent remains visible
- Screen reader friendly: all SVG diagrams have aria-labels
- Keyboard navigation for all interactive elements (for desktop practice management workflows)

---

## Mobile-First Requirements

**Primary use cases that must work perfectly on mobile**:
1. CCA module completion on a staff member's personal phone during a 15-minute break
2. H-Score review by a practice owner on their phone before the morning huddle
3. Wow story submission by a Champion immediately after a patient interaction
4. Leaderboard check by a practice owner

**Minimum viewport**: 375px (iPhone SE)
**Primary tablet target**: 768px (iPad in landscape break room mount)
**Desktop**: 1100px+

**Mobile-specific requirements**:
- Module video player full-screen on mobile with tap-to-pause
- H-Score gauge readable on 375px screen (minimum 120px diameter)
- Certification progress list scrollable without horizontal overflow
- All CTA buttons minimum 48px height, full-width on mobile

---

## Animation Guidance

**Principle**: Warm and deliberate. Never flashy. Animations should feel like they belong in a hospitality setting — smooth, unhurried, intentional.

**Approved motion patterns**:
- Fade-in on scroll (opacity: 0 → 1, 300ms ease-in)
- Scale-in for achievement moments (transform: scale(0.95) → scale(1), 200ms ease-out)
- Slide-up for bottom sheets and scenario cards (transform: translateY(20px) → translateY(0), 250ms ease-out)
- Gold particle effect for certification achievement (CSS only, 3-second duration, then fade)
- H-Score number count-up animation on first load (0 → actual score, 1200ms)

**Prohibited motion patterns**:
- Bounce/spring physics (too playful for a premium hospitality brand)
- Full-page transitions that take > 400ms
- Auto-playing animations on repeat (distract during huddle)
- Parallax scrolling (causes accessibility issues, feels gimmicky)

**CSS-only requirement**: No JavaScript animation libraries. All motion via CSS transitions and animations. This keeps the bundle light and the animations performant on older tablets.
