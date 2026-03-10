# UX Design Specification — The Funnel OS

> Unified Patient Funnel Operating System for SGA Dental Partners (260+ practices)
> Full Funnel Visibility — Sprint Development Package

---

## Design Principles

1. **Surface, don't bury** — The most important information is visible immediately. No navigation required to see what matters. Alerts, funnel health, and daily priorities are the first things users see, not something they hunt for.

2. **Explain the why, not just the what** — Every metric, alert, and recommendation includes causal context. "Phone answer rate dropped" is not enough. "Phone answer rate dropped because you lost a front desk person last Tuesday" is the standard.

3. **One action per card** — Practice staff get tappable cards with exactly one thing to do. No multi-step workflows, no ambiguity. Tap the card, do the thing, move on.

4. **Same data, different lens** — The CEO and the office manager see the same funnel data. The executive dashboard shows the network view; the Nerve Center shows the practice view. Same source of truth, different resolution.

5. **Zero learning curve at the practice level** — If the office manager needs training to use the Nerve Center, the design failed. Three to five cards, each with a single action, refreshed daily. That is the entire interface.

6. **Color means something** — Green, yellow, and red are never decorative. They always map to funnel health thresholds. Every color carries a status semantic, and every status has a non-color indicator alongside it.

7. **Extend, don't duplicate** — The Nerve Center is a module within the Practice Concierge app, not a separate product. Shared navigation, shared design system, shared mental model.

---

## Screen Inventory

### 1. Executive Dashboard

| Screen | Description | Primary Users | Device |
|--------|-------------|---------------|--------|
| Network Funnel View | 5-stage horizontal funnel with aggregate numbers, conversion rates, color-coded health, trend arrows | CEO, VPs | Desktop |
| AI Alerts Sidebar | Top 5 alerts ranked by revenue impact, each with root cause explanation and recommended action | CEO, VPs | Desktop |
| Ops Queue | 14+ practices needing attention, ranked by revenue impact, assignable to team members | HQ Ops Team | Desktop |
| Region Drill-Down | Regional funnel view with practice heatmap, color-coded by health score | CEO, VPs, Regional VPs | Desktop |
| Practice Detail (Exec) | Individual practice funnel, 5 stages with week-over-week trends, intervention history, staffing context | VPs, Ops Team | Desktop |
| Stage Deep Dive | Single funnel stage across all 260+ practices, ranked by performance | VPs, Ops Team | Desktop |
| Weekly Summary | AI-generated weekly narrative with trends, interventions taken, outcomes measured | CEO, Leadership | Desktop |
| Monthly Report | Month-end aggregate with financials, conversion rates, year-over-year comparison | CEO, Investors | Desktop |
| Ops Queue Detail | Single practice issue with full context, AI recommendation, action assignment, and outcome tracking | Ops Team Members | Desktop |

### 2. Practice Nerve Center

| Screen | Description | Primary Users | Device |
|--------|-------------|---------------|--------|
| Daily Agenda | 3-5 priority action cards ranked by revenue impact, refreshed each morning | Office Manager, Front Desk | iPad, Phone |
| Callback Card (Expanded) | Missed calls list with patient name, phone number pre-loaded, and estimated revenue per callback | Front Desk | iPad, Phone |
| Confirm Card (Expanded) | At-risk appointments with show rate prediction, one-tap text confirmation button | Front Desk | iPad, Phone |
| Follow-Up Card (Expanded) | Unscheduled treatment plans with patient name, treatment value, and suggested follow-up script | Office Manager | iPad, Phone |
| End-of-Day Scorecard | Daily summary: calls answered, patients booked, show rate, production, revenue attributed to actions | Office Manager | iPad, Phone |
| Practice Performance | Weekly/monthly trends for the practice's funnel, benchmarked against its own baseline | Office Manager | iPad |
| Module Switcher (Tab Bar) | Navigation between Concierge (storefront/brand) and Nerve Center (funnel/operations) | All Practice Staff | iPad, Phone |

### 3. Role-Based View Variants

| Role | Sees | Does Not See |
|------|------|-------------|
| CEO / VP | Network funnel, region drill-down, AI alerts, weekly/monthly summaries | Ops queue assignment controls (view only) |
| HQ Ops Team Lead | Ops queue with assignment, practice detail, stage deep dive | Monthly investor-level summaries |
| HQ Ops Team Member | Assigned practice issues, practice detail, action execution | Queue ranking controls, assignment of others |
| Office Manager | Full daily agenda, all card types, end-of-day scorecard, practice performance | Network or region views |
| Front Desk | Call queue (callback cards), confirmation list (confirm cards), scheduling actions | Scorecard totals, practice performance trends |

---

## Key Screen Wireframe Descriptions

### Executive Dashboard — Network Funnel View

**Layout**: Full-width desktop layout. Fixed top bar with SGA logo, search, notification bell, and user avatar. The main content area is divided into two zones: the funnel visualization (left, ~70% width) and the AI alerts sidebar (right, ~30% width).

**Funnel Visualization**:

```
+-----------------------------------------------------------------------+
|  FUNNEL OS          [Search practices...]     [Bell icon]  [Avatar]   |
+-----------------------------------------------------------------------+
|                                                                       |
|  Network Health   |  This Week  |  Mar 3-9, 2026                     |
|                                                                       |
|  +------------------------------------------------------------------+ |
|  |  LEADS GENERATED                                    4,200  +3%   | |
|  |  [============================================] GREEN             | |
|  |                                                                   | |
|  |  CONNECTED                                          3,150  75%   | |
|  |  [====================================]       GREEN               | |
|  |                                                                   | |
|  |  BOOKED                                             2,520  80%   | |
|  |  [==============================]             GREEN               | |
|  |                                                                   | |
|  |  SHOWED                                             2,142  85%   | |
|  |  [========================]                   YELLOW               | |
|  |                                                                   | |
|  |  PRODUCED                                           $3.2M  +1.5% | |
|  |  [====================]                       GREEN               | |
|  +------------------------------------------------------------------+ |
|                                                                       |
|  Drill-down: [Network] > [Southeast] > [Towne Lake Dental]          |
|                                                                       |
+-----------------------------------------------------------------------+
```

Each funnel stage is a horizontal bar. The bar width represents volume relative to the stage above. To the right of each bar: the absolute number, the conversion rate from the prior stage (or trend for Leads Generated and Produced), and a color-coded health badge (green dot, yellow dot, or red dot with text label). The entire row is clickable, drilling into the Stage Deep Dive for that stage.

Above the funnel, a breadcrumb trail shows the current drill-down level: Network > Region > Practice. Clicking any breadcrumb navigates back up.

Below the funnel, four KPI cards in a horizontal row:

| Connection Rate | Booking Rate | Show Rate | Avg Production/Patient |
|:-:|:-:|:-:|:-:|
| 75% | 80% | 85% | $1,495 |
| +2% vs last week | Flat | -1.5% (watch) | +3% vs last week |

Each KPI card shows the metric prominently, a micro-sparkline (last 8 weeks), and a comparison to the prior period. The show rate card has a yellow border because it dropped, drawing attention without requiring the user to read every number.

**AI Alerts Sidebar**:

A vertical stack of 5 alert cards, each with a colored left border indicating severity. Each card contains:

```
+----------------------------------------+
|  [!] CRITICAL                          |
|  Southeast phone answer rate -12%      |
|                                        |
|  Root cause: Staffing change at 3      |
|  practices (front desk -1 headcount)   |
|                                        |
|  Revenue impact: ~$14,200/week         |
|                                        |
|  [View Practices]  [Assign to Team]    |
+----------------------------------------+
```

- **Title**: Brief metric statement
- **Root cause**: AI-generated causal explanation (2 lines max)
- **Revenue impact**: Estimated weekly revenue at risk
- **Actions**: View the affected practices (drill-down) or assign to an ops team member

The sidebar scrolls independently of the main funnel view. A "View All Alerts" link at the bottom navigates to the full alert queue.

---

### Executive Dashboard — Ops Queue

**Layout**: Same top bar. Left sidebar navigation (collapsed to icons by default, expandable): Dashboard, Ops Queue (active), Practices, Reports, Settings. The main content area is a ranked list.

**Queue Header**:

```
+-----------------------------------------------------------------------+
|  Ops Queue                                           14 open items    |
|  [Filter: All] [Region: All] [Type: All] [Assigned: All]            |
+-----------------------------------------------------------------------+
```

**Queue Items**:

Each queue item is a card in a vertical list:

```
+-----------------------------------------------------------------------+
|  #1  Towne Lake Dental, Woodstock GA              PHONE ANSWER RATE  |
|      [RED badge] Answer rate: 62% (baseline: 84%)                    |
|                                                                       |
|      Root cause: Front desk headcount dropped 3 → 2 on Mar 3         |
|      AI recommendation: Activate AI phone agent overflow              |
|      Revenue impact: $4,800/week                                      |
|                                                                       |
|      [Assigned: Sarah M.]  [View Practice]  [Mark Resolved]          |
+-----------------------------------------------------------------------+
```

Items are ranked by revenue impact (highest first). The rank number is displayed prominently on the left. Each card shows: practice name and location, issue type tag, current metric vs. baseline, root cause, AI recommendation, estimated revenue impact, and assignment/action controls.

An "Already addressed" section at the bottom (collapsed by default) shows items where the practice or automation has already resolved the issue, so HQ does not duplicate effort.

---

### Executive Dashboard — Region Drill-Down

**Layout**: Triggered when clicking a region in the breadcrumb or funnel. The funnel visualization now shows region-level aggregates. Below the funnel, a heatmap grid replaces the KPI cards:

```
+-----------------------------------------------------------------------+
|  Southeast Region — 42 Practices                                      |
|                                                                       |
|  [Funnel visualization — region-level, same 5 stages]                |
|                                                                       |
|  Practice Heatmap:                                                    |
|  +--------+--------+--------+--------+--------+--------+             |
|  | Towne  | Garner | Bright | Peak   | River  | Cedar  |             |
|  | Lake   | Dental | Smile  | Dental | Oak    | Family |             |
|  | [RED]  | [GRN]  | [GRN]  | [YLW]  | [GRN]  | [GRN]  |             |
|  | 62%    | 91%    | 88%    | 74%    | 85%    | 90%    |             |
|  +--------+--------+--------+--------+--------+--------+             |
|  | ...    | ...    | ...    | ...    | ...    | ...    |             |
+-----------------------------------------------------------------------+
```

Each cell in the heatmap represents a practice, color-coded by overall health score (composite of all 5 funnel stages). The cell shows the practice name and its lowest-performing metric. Clicking a cell navigates to the Practice Detail view. Red practices sort to the top-left for immediate visibility.

---

### Executive Dashboard — Practice Detail (Exec View)

**Layout**: Full-width. Practice name, location, and health score badge in the header. Tabs below: Funnel, Interventions, Staffing, Timeline.

**Funnel Tab** (default):

The same 5-stage funnel visualization, now showing this single practice's data. Each stage shows: current value, week-over-week trend, baseline comparison, and whether AI has flagged it. Below the funnel, a line chart shows the last 8 weeks of each stage's performance, allowing executives to see trends at a glance.

**Interventions Tab**:

A vertical timeline of all AI-generated alerts and interventions for this practice, with outcomes:

```
Mar 7 — Alert: Show rate dropping (68%, baseline 82%)
         Action taken: Automated confirmation texts sent to 6 patients
         Outcome: 4 confirmed, show rate recovered to 78% next day

Mar 3 — Alert: Phone answer rate dropped (62%, baseline 84%)
         Action taken: Assigned to Sarah M. in ops queue
         Outcome: Pending — AI phone overflow activated Mar 8
```

Each entry shows the alert, what action was taken (by whom — automation, practice staff, or HQ), and the measured outcome.

---

### Practice Nerve Center — Daily Agenda

**Layout**: Full-screen on iPad (landscape or portrait). On phone, single-column vertical scroll. No sidebar navigation — the module switcher is a tab bar at the bottom.

**Greeting Header**:

```
+-----------------------------------------------+
|  Good morning, Jennifer.                      |
|  Here are your 3 priorities for today.        |
|  Estimated time: 18 minutes                   |
+-----------------------------------------------+
```

The greeting uses the staff member's first name (pulled from the practice profile). The priority count and time estimate are calculated by the AI based on card types and historical completion times for this user.

**Action Cards** (stacked vertically):

```
+-----------------------------------------------+
|  [Phone icon]  CALLBACK                    #1 |
|                                                |
|  4 missed calls from yesterday                 |
|  Est. 2 new patients, $1,200 revenue          |
|                                                |
|  [Tap to start calls]                         |
+-----------------------------------------------+

+-----------------------------------------------+
|  [Calendar icon]  CONFIRM                  #2 |
|                                                |
|  3 at-risk appointments for tomorrow           |
|  Show rate prediction: 55% without action     |
|                                                |
|  [Tap to send confirmations]                  |
+-----------------------------------------------+

+-----------------------------------------------+
|  [Clipboard icon]  FOLLOW UP              #3 |
|                                                |
|  2 unscheduled treatment plans                 |
|  Combined value: $8,400                        |
|                                                |
|  [Tap to view scripts]                        |
+-----------------------------------------------+
```

Each card has:
- **Icon and type label**: top-left, color-coded by card type (blue for callback, green for confirm, amber for follow-up)
- **Priority rank**: top-right (#1, #2, #3)
- **Summary line**: what needs to be done, in plain language
- **Impact line**: estimated revenue or risk metric
- **Single CTA button**: full-width, tappable, leads to the expanded card view

Cards are large enough to tap without precision. Minimum card height: 120px on iPad, 100px on phone. The entire card is tappable, not just the button.

---

### Practice Nerve Center — Callback Card (Expanded)

Tapping the callback card opens the expanded view:

```
+-----------------------------------------------+
|  [Back arrow]  CALLBACKS — 4 missed calls     |
+-----------------------------------------------+
|                                                |
|  1. Maria Gonzalez                             |
|     (555) 234-5678       New patient inquiry  |
|     Yesterday 2:14 PM — rang 6 times          |
|     [Call Now]                                 |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  2. James Patterson                            |
|     (555) 345-6789       Whitening question   |
|     Yesterday 3:45 PM — rang 4 times          |
|     [Call Now]                                 |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  3. Lisa Chen                                  |
|     (555) 456-7890       Appointment request  |
|     Yesterday 4:10 PM — voicemail left        |
|     [Call Now]                                 |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  4. Robert Williams                            |
|     (555) 567-8901       Insurance question   |
|     Yesterday 4:32 PM — rang 3 times          |
|     [Call Now]                                 |
|                                                |
+-----------------------------------------------+
|  Completed: 0/4          [Mark All Done]      |
+-----------------------------------------------+
```

Each entry shows: patient name, phone number (tappable to initiate call on phone, displayed for manual dial on iPad), call reason (if detectable from voicemail or AI agent notes), timestamp, and ring count or voicemail indicator. The "Call Now" button is the primary action — on phone it initiates the call; on iPad it copies the number and shows a "calling" prompt.

As calls are completed, the entry gets a green checkmark and slides down. The "Completed: X/4" counter updates in real time. When all calls are done, the card in the daily agenda view shows a green "Done" badge.

---

### Practice Nerve Center — Confirm Card (Expanded)

```
+-----------------------------------------------+
|  [Back arrow]  CONFIRMATIONS — 3 at-risk      |
+-----------------------------------------------+
|                                                |
|  1. David Kim — Implant Consult ($4,500)      |
|     Tomorrow 9:00 AM                           |
|     Show prediction: 45% without confirmation  |
|     [Send Text]  [Call Instead]                |
|                                                |
|     Suggested text:                            |
|     "Hi David, we're looking forward to your  |
|      implant consultation tomorrow at 9 AM.    |
|      Reply YES to confirm or call us to        |
|      reschedule."                              |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  2. Amy Torres — Full Exam ($350)             |
|     Tomorrow 10:30 AM                          |
|     Show prediction: 60% without confirmation  |
|     [Send Text]  [Call Instead]                |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  3. Brian Scott — Cleaning ($150)             |
|     Tomorrow 2:00 PM                           |
|     Show prediction: 55% without confirmation  |
|     [Send Text]  [Call Instead]                |
|                                                |
+-----------------------------------------------+
|  Sent: 0/3    Confirmed: 0/3                  |
+-----------------------------------------------+
```

Appointments are ranked by value (highest first). Each shows: patient name, service type with value, appointment time, AI show rate prediction, and two action buttons. The suggested text is pre-written and service-type-aware (implant consult gets a reassuring tone; cleaning gets a casual tone). "Send Text" sends the pre-written message via Twilio. "Call Instead" initiates a phone call with a suggested script displayed on screen.

When a patient responds (text reply or call completed), the status updates in real time: "Confirmed," "Rescheduled to Friday 2 PM," or "No response yet."

---

### Practice Nerve Center — Follow-Up Card (Expanded)

```
+-----------------------------------------------+
|  [Back arrow]  FOLLOW-UPS — 2 treatment plans |
+-----------------------------------------------+
|                                                |
|  1. Patricia Adams — Crown (#14)              |
|     Treatment value: $1,200                    |
|     Diagnosed: Feb 28, 2026                    |
|     Last contact: None since diagnosis         |
|                                                |
|     Suggested script:                          |
|     "Hi Patricia, Dr. Martinez recommended a  |
|      crown on tooth #14 during your visit on   |
|      Feb 28. We'd love to get that scheduled   |
|      for you. Do you have any questions about  |
|      the procedure or your insurance coverage?" |
|                                                |
|     [Call Patient]  [Send Text]  [Skip Today] |
|                                                |
|  ─────────────────────────────────────────    |
|                                                |
|  2. Michael Brown — Full Arch ($7,200)        |
|     Treatment value: $7,200                    |
|     Diagnosed: Mar 1, 2026                     |
|     Last contact: Insurance verification sent  |
|                                                |
|     Suggested script:                          |
|     "Hi Michael, this is Jennifer from [practice|
|      name]. I'm following up on the treatment   |
|      plan Dr. Martinez discussed with you. Your |
|      insurance has been verified — would you    |
|      like to schedule your first appointment?" |
|                                                |
|     [Call Patient]  [Send Text]  [Skip Today] |
|                                                |
+-----------------------------------------------+
```

Follow-ups show the patient name, treatment type, dollar value, diagnosis date, and last contact status. The AI-generated script is contextual: it accounts for the treatment type, how long since diagnosis, and whether insurance has been verified. "Skip Today" defers the follow-up to tomorrow's agenda without dismissing it permanently.

---

### Practice Nerve Center — End-of-Day Scorecard

```
+-----------------------------------------------+
|  Your Day at a Glance                         |
|  Monday, March 9, 2026                        |
+-----------------------------------------------+
|                                                |
|  CALLS ANSWERED          12 / 13   92%   [G]  |
|  ───────────────────────────────────           |
|  PATIENTS BOOKED                4   +1 vs avg |
|  ───────────────────────────────────           |
|  SHOW RATE               88%        [G]       |
|  ───────────────────────────────────           |
|  PRODUCTION         $18,400    102% of target |
|  ───────────────────────────────────           |
|  REVENUE FROM YOUR ACTIONS          $3,600    |
|                                                |
+-----------------------------------------------+
|                                                |
|  Your Priorities: 2 of 3 completed            |
|  [check] Called back 4 patients → 2 booked    |
|  [check] Confirmed 3 appointments → all confirmed |
|  [arrow] Treatment plan follow-ups → tomorrow |
|                                                |
+-----------------------------------------------+
```

The scorecard is a simple vertical list of KPIs. Each metric shows: the label, the value, and a contextual comparison (percentage of target, change vs. daily average, or health indicator). Green [G] or red [R] badges appear only when the metric is notably above or below baseline.

The bottom section connects actions to outcomes: "You called back 4 patients this morning, and 2 of them booked appointments." This feedback loop is what builds the habit of using the 20-minute agenda. Incomplete priorities show where they were deferred to (tomorrow).

---

### Practice Nerve Center — Role-Based Views

**Office Manager View**: Sees the full daily agenda (all card types), the end-of-day scorecard, and a "Practice Performance" tab showing weekly/monthly funnel trends for their practice.

**Front Desk View**: Sees only the callback cards and confirm cards. The follow-up cards and scorecard are hidden. The interface is even simpler — just a call queue and a confirmation list. The tab bar shows: Calls, Confirmations, Concierge (link to the Practice Concierge module).

The role is determined by the user's profile, set during practice onboarding. Office managers can switch to the front desk view for training purposes.

---

## Design System

This design system is shared with the Practice Concierge (Digital Storefront sprint). All tokens, components, and patterns defined here extend the existing system. New additions specific to the Funnel OS are marked with "(NEW)".

### Colors

**Base Palette** (inherited from Practice Concierge):

| Token | Hex | Usage |
|-------|-----|-------|
| `--bg-primary` | `#0a0a0f` | Page background |
| `--bg-surface` | `#12121a` | Cards, panels, chat bubbles |
| `--bg-surface-hover` | `#1a1a2e` | Interactive surface hover state |
| `--bg-elevated` | `#1e1e30` | Modals, popovers, dropdown menus |
| `--border-default` | `#2a2a3d` | Card borders, dividers |
| `--border-focus` | `#4f8fff` | Focus rings, active borders |

**Funnel Health Colors** (NEW):

| Token | Hex | Usage |
|-------|-----|-------|
| `--health-green` | `#34d399` | On-track funnel stages, healthy metrics |
| `--health-green-bg` | `rgba(52, 211, 153, 0.12)` | Green health background tint |
| `--health-yellow` | `#fbbf24` | Watch — metric trending downward or near threshold |
| `--health-yellow-bg` | `rgba(251, 191, 36, 0.12)` | Yellow health background tint |
| `--health-red` | `#f87171` | Intervention needed — metric below threshold |
| `--health-red-bg` | `rgba(248, 113, 113, 0.12)` | Red health background tint |

**Accent Colors** (inherited):

| Token | Hex | Usage |
|-------|-----|-------|
| `--accent-blue` | `#4f8fff` | Primary actions, links, active states, callback cards |
| `--accent-blue-muted` | `rgba(79, 143, 255, 0.15)` | Blue backgrounds, selected states |
| `--accent-green` | `#34d399` | Success, approval, completed actions, confirm cards |
| `--accent-green-muted` | `rgba(52, 211, 153, 0.15)` | Success backgrounds |
| `--accent-amber` | `#fbbf24` | Warnings, in-progress, follow-up cards |
| `--accent-amber-muted` | `rgba(251, 191, 36, 0.15)` | Warning backgrounds |
| `--accent-red` | `#f87171` | Errors, critical alerts |
| `--accent-red-muted` | `rgba(248, 113, 113, 0.15)` | Error backgrounds |
| `--accent-teal` | `#2dd4bf` | AI-generated content, automation indicators |

**Text Colors** (inherited):

| Token | Hex | Usage |
|-------|-----|-------|
| `--text-primary` | `#f0f0f5` | Headings, primary content, metric values |
| `--text-secondary` | `#a0a0b8` | Body text, descriptions, root cause explanations |
| `--text-muted` | `#6b6b80` | Timestamps, helper text, placeholders |
| `--text-inverse` | `#0a0a0f` | Text on light/colored backgrounds |

### Typography

**Font Stack** (inherited):

```css
--font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
             Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans',
             'Helvetica Neue', sans-serif;

--font-mono: 'SF Mono', 'Fira Code', 'Fira Mono', 'Roboto Mono',
             Consolas, 'Courier New', monospace;
```

**Type Scale** (inherited, with Funnel OS usage notes):

| Token | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| `--text-xs` | 11px | 400 | 1.4 | Badges, timestamps, conversion rate labels |
| `--text-sm` | 13px | 400 | 1.5 | Helper text, root cause text, card metadata |
| `--text-base` | 15px | 400 | 1.6 | Body text, action descriptions, scripts |
| `--text-md` | 17px | 500 | 1.5 | Card titles, patient names, metric labels |
| `--text-lg` | 20px | 600 | 1.4 | Section headings, agenda greeting |
| `--text-xl` | 24px | 700 | 1.3 | Page titles, practice names in detail views |
| `--text-2xl` | 32px | 700 | 1.2 | KPI hero numbers, funnel stage values |
| `--text-3xl` | 48px | 800 | 1.1 | Production total, network health score |

**Funnel-Specific Type Additions** (NEW):

| Token | Size | Weight | Usage |
|-------|------|--------|-------|
| `--text-metric` | 28px | 700 | Scorecard values, conversion percentages |
| `--text-revenue` | 20px | 600 | Revenue impact numbers (always prefixed with $) |

### Spacing

**Base Unit**: 4px (inherited)

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Inline spacing, icon gaps |
| `--space-2` | 8px | Tight padding, small gaps |
| `--space-3` | 12px | Default padding inside components |
| `--space-4` | 16px | Standard content padding |
| `--space-5` | 20px | Card padding |
| `--space-6` | 24px | Section gaps, card margins |
| `--space-8` | 32px | Section padding |
| `--space-10` | 40px | Large section gaps |
| `--space-12` | 48px | Page section separation |
| `--space-16` | 64px | Major layout gaps |

**Border Radius** (inherited):

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Badges, health pills |
| `--radius-md` | 8px | Buttons, inputs, KPI cards |
| `--radius-lg` | 12px | Action cards, alert cards, funnel bars |
| `--radius-xl` | 16px | Modal dialogs, expanded card views |
| `--radius-full` | 9999px | Avatars, circular health indicators, status dots |

### Component Library

**Funnel Bar** (NEW)

```
Properties: stage (leads | connected | booked | showed | produced),
            value (number or currency), conversionRate (percentage),
            trend (up | flat | down), health (green | yellow | red),
            isClickable (boolean)
Layout: horizontal bar, full width of funnel container
Height: 56px (desktop), 48px (tablet)
Background: --bg-surface
Fill: bar proportional to value relative to first stage,
      colored by health (--health-green | --health-yellow | --health-red)
      at 20% opacity, with solid left border (3px) in health color
Right section: value (--text-2xl), conversion rate (--text-sm, --text-muted),
               trend arrow icon, health dot
Border: 1px solid --border-default
Radius: --radius-lg
Hover: bg --bg-surface-hover, cursor pointer
Spacing: --space-2 gap between bars (creates the funnel taper effect)
```

**Action Card (Nerve Center)** (NEW)

```
Properties: type (callback | confirm | follow-up), rank (number),
            summary (string), impact (string), count (number),
            isCompleted (boolean)
Background: --bg-surface
Border: 1px solid --border-default
Left border: 3px solid (callback: --accent-blue, confirm: --accent-green,
             follow-up: --accent-amber)
Radius: --radius-lg
Padding: --space-5
Min height: 120px (iPad), 100px (phone)
Layout:
  Row 1: icon (24px, type-colored) + type label (--text-xs, uppercase,
          type-colored) + rank badge (right-aligned, --text-xs, --text-muted)
  Row 2: summary text (--text-md, --text-primary)
  Row 3: impact text (--text-sm, --text-secondary)
  Row 4: CTA button (full-width, type-colored background at 15% opacity,
          type-colored text, --radius-md, 48px height)
Completed state: green checkmark overlay, card opacity 0.6,
                 summary gets strikethrough
Tap target: entire card is tappable (not just the button)
```

**Alert Card (Executive)** (inherited, extended)

```
Properties: severity (critical | warning | info), title, rootCause,
            revenueImpact, actions[], practiceCount
Background: --bg-surface
Left border: 3px solid (critical: --health-red, warning: --health-yellow,
             info: --accent-blue)
Radius: --radius-lg
Padding: --space-5
Layout:
  Row 1: severity badge (pill, colored) + title (--text-md)
  Row 2: root cause text (--text-sm, --text-secondary)
  Row 3: revenue impact (--text-revenue, --health-red or --health-yellow)
  Row 4: action buttons (--space-2 gap)
Icon: severity-specific icon (top-left of severity badge)
      critical: exclamation-triangle, warning: exclamation-circle,
      info: info-circle
```

**KPI Card** (NEW)

```
Properties: label, value, trend (up | flat | down), comparison (string),
            health (green | yellow | red | neutral), sparklineData[]
Background: --bg-surface
Border: 1px solid --border-default (health yellow/red: border in health color)
Radius: --radius-lg
Padding: --space-5
Layout:
  Label: --text-sm, --text-muted, uppercase
  Value: --text-2xl, --text-primary
  Sparkline: 80px wide, 24px tall, --text-muted stroke (1.5px),
             last point colored by health
  Comparison: --text-xs, health-colored text + trend arrow icon
Min width: 180px
```

**Ops Queue Item** (NEW)

```
Properties: rank, practiceName, location, issueType, currentMetric,
            baseline, rootCause, recommendation, revenueImpact,
            assignee, status (open | assigned | resolved)
Background: --bg-surface
Border: 1px solid --border-default
Radius: --radius-lg
Padding: --space-5
Layout:
  Header row: rank (--text-lg, --text-muted) + practice name (--text-md)
              + location (--text-sm, --text-muted) + issue type tag (right)
  Metric row: current value (--text-metric, health-colored)
              + "baseline: X" (--text-sm, --text-muted)
  Root cause: (--text-sm, --text-secondary, prefixed with "Root cause:" label)
  Recommendation: (--text-sm, --accent-teal, prefixed with "AI:" label)
  Revenue impact: (--text-revenue, --health-red or --health-yellow)
  Footer: assignee avatar + name, action buttons
Hover: bg --bg-surface-hover
```

**Scorecard Row** (NEW)

```
Properties: label, value, comparison, health (green | yellow | neutral)
Background: transparent
Layout: single row, space-between
  Left: label (--text-md, --text-secondary)
  Center: value (--text-metric, --text-primary)
  Right: comparison text (--text-sm, health-colored) + health dot
Border bottom: 1px solid --border-default (except last row)
Padding: --space-4 vertical
```

**Health Dot** (NEW)

```
Properties: health (green | yellow | red)
Shape: circle, 8px diameter
Color: --health-green | --health-yellow | --health-red
Accessible label: aria-label="Status: on track" | "Status: watch" |
                  "Status: needs intervention"
Always accompanied by a text label for accessibility
```

**Breadcrumb Trail** (NEW)

```
Properties: levels[] (each with label and onClick)
Layout: horizontal, --space-1 gap, separated by ">" in --text-muted
Font: --text-sm
Active level (last): --text-primary, weight 600
Clickable levels: --accent-blue, underline on hover
```

**Funnel Stage Pill** (NEW)

```
Properties: stage (leads | connected | booked | showed | produced)
Shape: pill (--radius-full)
Height: 24px
Padding: --space-1 vertical, --space-3 horizontal
Font: --text-xs, weight 600, uppercase
Colors by stage:
  leads: bg --accent-blue-muted, text --accent-blue
  connected: bg --accent-blue-muted, text --accent-blue
  booked: bg --accent-green-muted, text --accent-green
  showed: bg --accent-amber-muted, text --accent-amber
  produced: bg --accent-green-muted, text --accent-green
```

### Iconography

**Icon Style**: Outlined, 1.5px stroke, 24px default size (20px compact, 16px inline). Use system SF Symbols on iOS/iPadOS and a consistent outline icon set (Lucide or Heroicons Outline) on web.

**Funnel OS Icon Set** (NEW):

| Icon | Usage |
|------|-------|
| `phone-missed` | Callback cards, missed call alerts |
| `phone-outgoing` | Call Now action |
| `calendar-check` | Confirm cards, appointment confirmations |
| `clipboard-list` | Follow-up cards, treatment plans |
| `trending-up` / `trending-down` | Trend arrows on metrics |
| `alert-triangle` | Critical alerts |
| `alert-circle` | Warning alerts |
| `info` | Informational alerts |
| `target` | Funnel stage indicator |
| `users` | Practice/team context |
| `dollar-sign` | Revenue impact |
| `brain` | AI-generated content indicator |
| `check-circle` | Completed action |
| `arrow-right` | Drill-down affordance |
| `chevron-right` | Breadcrumb separator |
| `filter` | Filter controls |
| `user-plus` | Assignment |

---

## Interaction Patterns

### Drill-Down Navigation (Executive Dashboard)

The drill-down follows a strict hierarchy: **Network > Region > Practice > Funnel Stage**.

**Behavior**:
- Clicking a region on the network view zooms into that region's funnel + practice heatmap
- Clicking a practice on the region heatmap opens the practice detail with its 5-stage funnel
- Clicking a funnel stage (at any level) opens the stage deep dive, showing that stage's metric across all practices in the current scope
- The breadcrumb trail updates at each level and allows backward navigation
- Browser back button and keyboard Escape return to the previous level
- Transition: 200ms ease-out slide animation (respects `prefers-reduced-motion`)

**URL structure**: Each drill-down level has a unique URL for direct linking and sharing:
- `/dashboard` — Network view
- `/dashboard/region/southeast` — Region view
- `/dashboard/practice/towne-lake-dental` — Practice detail
- `/dashboard/stage/phone-answer-rate` — Stage deep dive

### Action Card Interaction (Nerve Center)

**Card tap flow**:
1. User taps action card on the daily agenda
2. Card expands to show the full list of items (callbacks, confirmations, or follow-ups)
3. User works through items sequentially (each with its own one-tap action)
4. As items are completed, they get a green checkmark and the counter updates
5. When all items are done (or user taps back arrow), they return to the daily agenda
6. The card on the agenda shows updated completion status ("2 of 4 done" or "Done")

**One-tap actions**:
- **Call Now**: On phone, initiates a native phone call. On iPad, copies number to clipboard and shows a visual prompt ("Dial 555-234-5678"). Call completion is detected (or manually confirmed by tapping "Call completed").
- **Send Text**: Sends the pre-written confirmation/follow-up text via Twilio. Shows a brief "Sent" confirmation animation (green checkmark, 800ms). No second confirmation dialog — the text is pre-approved by the AI and the user saw the content.
- **Skip Today**: Defers the item to tomorrow's agenda. A small "Skipped — will appear tomorrow" toast notification confirms the action.

### Real-Time Updates

**Executive Dashboard**:
- Funnel data refreshes via WebSocket push. When new data arrives, affected metrics animate smoothly (number counter transition, 300ms).
- Alert sidebar updates in real time. New alerts slide in from the top with a subtle pulse animation.
- Ops queue re-ranks when revenue impact estimates change. Items smoothly reorder (200ms transition).
- No manual refresh required. A "Last updated: 2 min ago" timestamp in the header confirms data freshness.

**Practice Nerve Center**:
- Daily agenda is generated once each morning (5:00 AM local time) and does not change mid-day. This prevents confusion from shifting priorities.
- Exception: urgent items (same-day cancellations, critical missed calls) can push a new card to the top of the agenda with a push notification.
- Confirmation responses (text replies from patients) update the confirm card in real time.
- End-of-day scorecard updates in real time as the practice's PMS records production and appointments.

### Ops Queue Assignment

1. Ops team lead sees the ranked queue
2. Taps "Assign" on a queue item
3. A dropdown shows team members (avatar + name + current workload count)
4. Selecting a team member assigns the item — it appears in that person's "My Assignments" filtered view
5. The assignee gets a push notification (if enabled) or sees the item at next login
6. The assignee can mark it "In Progress" or "Resolved" with an optional note
7. Resolved items move to the "Already addressed" section and feed into the AI learning loop

---

## Responsive Design

### Desktop (Executive Dashboard) — 1280px+

**Layout**:
- Fixed top bar: 56px height. SGA logo left, search center, notifications + avatar right.
- Collapsible left sidebar: 240px expanded, 56px collapsed (icon-only). Sections: Dashboard, Ops Queue, Practices, Reports, Settings.
- Main content: flexible width, max-width 1440px centered, with `--space-8` padding.
- AI alerts sidebar: 360px fixed width on right, collapsible to a floating button.

**Funnel visualization**: Horizontal bars, full width of main content area.

**KPI cards**: 4-across row below the funnel.

**Ops queue**: Single-column list, full width.

**Practice heatmap (region view)**: 6-column grid, responsive down to 4 columns.

### Tablet — iPad (Practice Nerve Center) — 768px-1279px

**Layout**:
- No sidebar. Bottom tab bar: 3 tabs (Nerve Center, Concierge, Profile). Tab bar height: 49px (iOS standard) + safe area.
- Full-screen content area with `--space-6` padding.
- Supports landscape and portrait orientation.

**Daily agenda**: Cards stack vertically, full width with `--space-4` margins.

**Expanded card view**: Full-screen takeover with back arrow in top-left.

**Scorecard**: Full width, single column.

**Portrait**: Single column, cards stack.

**Landscape**: Action cards can display in a 2-column grid if more than 3 cards. Expanded views remain full-screen.

### Mobile Phone (Office Manager on the go) — 320px-767px

**Layout**:
- Bottom tab bar: same 3 tabs, 49px + safe area.
- Content area with `--space-4` padding.
- No landscape optimization — portrait-only assumption.

**Daily agenda**: Cards stack vertically, full width. Card min-height: 100px.

**Expanded card view**: Full-screen, scrollable list of items.

**Scorecard**: Full width, single column, more compact (remove sparklines, keep just value + comparison).

**Key differences from iPad**:
- Greeting header is shorter (just "3 priorities today" — no estimated time)
- Action card impact text is truncated to one line with ellipsis
- Expanded cards show one item at a time (swipe to navigate) instead of a full scrollable list
- "Call Now" button initiates native phone call directly

### Touch Targets

| Element | Minimum Size | Recommended Size |
|---------|-------------|-----------------|
| Action card (full card) | 100px height | 120px height |
| CTA button | 44px x 44px | 48px height, full width |
| Call Now / Send Text | 44px x 44px | 48px height, min 120px width |
| Tab bar items | 44px x 44px | 49px height, equal width distribution |
| Drill-down funnel bar | 48px height | 56px height |
| Alert card actions | 44px x 44px | 36px height, 80px+ width |
| Spacing between targets | 8px minimum | 12px preferred |

---

## Accessibility Specification

### WCAG 2.1 AA Compliance (Minimum)

This platform serves 260+ dental practices with diverse staff. AA compliance is the floor, not the ceiling.

### Color Contrast

All text and interactive elements meet minimum contrast ratios:

| Element | Foreground | Background | Ratio | Requirement |
|---------|-----------|------------|-------|-------------|
| Primary text | `#f0f0f5` | `#0a0a0f` | 18.4:1 | 4.5:1 (AA) |
| Secondary text | `#a0a0b8` | `#0a0a0f` | 8.2:1 | 4.5:1 (AA) |
| Muted text | `#6b6b80` | `#0a0a0f` | 4.6:1 | 4.5:1 (AA) |
| Health green on surface | `#34d399` | `#12121a` | 8.7:1 | 4.5:1 (AA) |
| Health yellow on surface | `#fbbf24` | `#12121a` | 9.8:1 | 4.5:1 (AA) |
| Health red on surface | `#f87171` | `#12121a` | 5.8:1 | 4.5:1 (AA) |
| Accent blue on surface | `#4f8fff` | `#12121a` | 5.1:1 | 4.5:1 (AA) |
| Large text (--text-2xl+) | `#f0f0f5` | `#0a0a0f` | 18.4:1 | 3:1 (AA large) |

### Non-Color Indicators

Every piece of information conveyed by color also uses a secondary indicator:

| Information | Color Indicator | Non-Color Indicator |
|-------------|----------------|-------------------|
| Funnel stage health | Green/yellow/red bar fill | Text label: "on track" / "watch" / "intervention needed" |
| Alert severity | Left border color | Severity badge text: "CRITICAL" / "WARNING" / "INFO" + icon |
| Action card type | Left border color | Icon + type label text: "CALLBACK" / "CONFIRM" / "FOLLOW UP" |
| Metric trend | Green (up) / Red (down) | Arrow icon (up/down/flat) + text: "+3%" / "-1.5%" |
| Completed action | Green checkmark | Strikethrough text + "Done" label |
| Queue item rank | Revenue-impact-driven order | Explicit rank number (#1, #2, #3) |

### Screen Reader Support

**Executive Dashboard**:
- Funnel visualization: each bar has `role="listitem"` within a `role="list"` container labeled "Patient funnel stages." Each bar announces: "Leads Generated: 4,200, up 3% week over week, status on track."
- KPI cards: each has `role="group"` with `aria-label`: "Connection rate: 75%, up 2% versus last week."
- Alert sidebar: `role="log"` with `aria-live="polite"`. New alerts announced: "New critical alert: Southeast phone answer rate down 12%."
- Drill-down: breadcrumb uses `nav` with `aria-label="Funnel navigation"`. Current level announced on transition.

**Practice Nerve Center**:
- Daily agenda: `role="list"` with `aria-label="Today's priorities."` Each card announces: "Priority 1, callback: 4 missed calls from yesterday, estimated 2 new patients, $1,200 revenue."
- Expanded card items: `role="list"` with `aria-label="Callback list, 4 items."` Each item announces: "1 of 4: Maria Gonzalez, new patient inquiry, phone number 555-234-5678."
- Action buttons: descriptive labels — "Call Maria Gonzalez" not just "Call Now." "Send confirmation text to David Kim" not just "Send Text."
- Scorecard: `role="list"` with `aria-label="End of day scorecard."` Each row announces: "Calls answered: 12 of 13, 92%, status on track."
- Completion feedback: `aria-live="polite"` announces "Callback completed, 1 of 4 done" when an item is marked complete.

### Focus Management

- **Dashboard drill-down**: When drilling into a region or practice, focus moves to the page title of the new view. When navigating back via breadcrumb, focus returns to the breadcrumb item that was clicked.
- **Nerve Center expanded card**: When opening an expanded card, focus moves to the back arrow (first actionable element). When closing, focus returns to the card that was tapped on the agenda.
- **Ops queue assignment**: When assigning a team member, focus moves to the confirmation message. When dismissing, focus returns to the assign button.
- **Focus ring**: 2px solid `--border-focus` with 2px offset, visible during keyboard navigation. Hidden during pointer interaction (`:focus-visible` only).

### Keyboard Navigation

| Context | Key | Action |
|---------|-----|--------|
| Dashboard funnel | Tab | Move between funnel bars |
| Dashboard funnel | Enter | Drill into selected stage |
| Dashboard funnel | Escape | Navigate up one drill-down level |
| Breadcrumb | Tab | Move between breadcrumb items |
| Breadcrumb | Enter | Navigate to selected level |
| Alert sidebar | Tab | Move between alert cards |
| Alert sidebar | Enter | Expand selected alert |
| Ops queue | Tab | Move between queue items |
| Ops queue | Enter | Open selected item detail |
| Nerve Center agenda | Tab | Move between action cards |
| Nerve Center agenda | Enter | Expand selected card |
| Expanded card | Tab | Move between items and action buttons |
| Expanded card | Escape | Return to daily agenda |
| Expanded card | Enter/Space | Activate focused button (Call, Send, Skip) |
| Scorecard | Tab | Move between scorecard rows |
| Tab bar | Tab | Move between tabs |
| Tab bar | Enter | Switch to selected tab |

### Motion and Animation

- **Respect `prefers-reduced-motion`**: All transitions and animations disabled when OS-level reduced motion is active.
- **Drill-down transitions**: 200ms ease-out slide. Reduced to 0ms (instant swap) when reduced motion is active.
- **Metric counter animation**: 300ms number roll-up when new data arrives. Reduced to instant update.
- **Card completion**: 200ms green checkmark fade-in + 300ms card slide-down. Reduced to instant state change.
- **Alert arrival**: 200ms slide-in from top + subtle pulse. Reduced to instant appearance.
- **No auto-playing animations**: All animations trigger on user action or data update, never on a loop.

---

## Performance Targets

| Metric | Target | Context |
|--------|--------|---------|
| Executive Dashboard first paint | < 2 seconds | Desktop on broadband |
| Nerve Center first paint | < 1.5 seconds | iPad on WiFi, phone on 4G |
| Drill-down transition | < 500ms | Data pre-fetched for adjacent levels |
| Action card expansion | < 200ms | Data already loaded from agenda |
| Real-time metric update | < 200ms render | After WebSocket event received |
| Send Text action | < 1 second feedback | Time from tap to "Sent" confirmation |
| Offline resilience (Nerve Center) | Full agenda cached | Daily agenda cached at 5 AM; works offline |
| Data freshness (Dashboard) | < 4 hours old | Data no more than 4 hours stale |

---

*This design specification is the source of truth for all Funnel OS UI implementation. It extends the Practice Concierge design system and shares all base tokens and patterns. Deviations require review and approval through the development team's design review process.*
