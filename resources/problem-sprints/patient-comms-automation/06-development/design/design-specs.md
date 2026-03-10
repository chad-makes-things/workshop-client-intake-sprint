# UX Design Specification — The Comms Engine

> Autonomous Patient Communications Playbook for SGA Dental Partners (260+ practices)
> Patient Communications Automation — Sprint Development Package

---

## Design Principles

1. **The engine works, not the staff** — Practice-facing interfaces exist only for the 20-minute weekly touchpoint and supervised-mode approval. The default state for a practice is: no action needed today. Design for that default, not for a power user dashboard.

2. **Escalate judgment, not execution** — The Comms Engine only surfaces items requiring human decision. Every item in the daily digest or weekly summary must be something a human genuinely needs to act on. If the engine can handle it, it should handle it silently.

3. **Warm but not salesy** — Every message template and every UI string uses warm, professional dental care language. "No pressure" is a core tone principle. The interface reflects this: nothing aggressive, no red urgency banners for things that aren't urgent.

4. **One screen, one decision** — The daily digest is a single list. The weekly summary is a single message. No navigation required to see what matters. HQ interfaces are richer, but practice interfaces must pass the "office manager checking phone between patients" test.

5. **HQ controls the playbook, practices trust the agent** — The Playbook Manager is for HQ. Practices do not configure cadences, templates, or thresholds. Their only configuration is reviewing messages in supervised mode and approving graduation.

6. **Color means autonomy status** — Green = Autonomous mode, Yellow = Monitored mode, Red = Supervised mode. These colors appear in the Practice Health Grid (HQ) and in the autonomy badge on the practice interface. Colors are always paired with a text label (never color-only).

7. **Extend the Practice Concierge — don't duplicate** — The approval interface is a new tab/module within the Practice Concierge app. Shared navigation, shared auth, shared design system. A practice staff member uses one app, one login.

---

## Screen Inventory

### 1. Practice-Facing Interface (Mobile PWA — extends Practice Concierge app)

| Screen | Description | Primary Users | Device |
|--------|-------------|---------------|--------|
| Daily Digest | List of N pending messages for supervised approval, sorted by scheduled send time | Office Manager | iPhone, iPad |
| Message Approval Card (expanded) | Full context for a single pending message: patient first name, appointment type, proposed message, edit/approve/reject | Office Manager | iPhone, iPad |
| Approve All Confirmation | Single-tap bulk approval with count summary before confirming | Office Manager | iPhone, iPad |
| Message Edit Drawer | Slide-up editor for modifying a pending message before approving | Office Manager | iPhone, iPad |
| Weekly Summary View | Formatted 3-section weekly summary (Engine Activity / Attention Items / Coming This Week) | Office Manager | iPhone, iPad |
| Attention Item Card | Expanded view of a weekly summary escalation with patient context and action prompt | Office Manager | iPhone, iPad |
| Graduation Report | 30-day quality summary with edit rate, approval rate, response rate; recommendation to advance autonomy level | Office Manager | iPhone, iPad |
| Autonomy Status Screen | Current autonomy levels for each playbook stage (Lead / Treatment / Rebooking / Reactivation) | Office Manager | iPhone, iPad |
| Tab Bar Module Switcher | Navigation between Practice Concierge (storefront) and Comms Engine (communications) tabs | All Practice Staff | iPhone, iPad |

### 2. HQ-Facing Interface (Desktop Web — standalone admin app)

| Screen | Description | Primary Users | Device |
|--------|-------------|---------------|--------|
| Network Comms Dashboard | 4-stage performance metrics (Lead Response / Treatment Follow-Up / Rebooking / Reactivation) with network-wide KPIs | HQ Ops, VP Operations | Desktop |
| Practice Health Grid | Heatmap of all 260+ practices colored by autonomy level and performance; filterable by region/specialty/stage | HQ Ops | Desktop |
| Playbook Designer | Visual timeline cadence designer with draggable touch points, delay configuration, channel selectors, branching logic | HQ Strategy, Product | Desktop |
| Template Editor | Message template editor with variable picker, live preview, Claude-powered suggestions, compliance checker | HQ Content, Ops | Desktop |
| A/B Test Manager | Test configuration, practice cohort assignment, results dashboard, winner promotion | HQ Product, Data | Desktop |
| Playbook Evolution Report | Monthly network learning output: what changed, why, data supporting the change, projected impact | HQ VP, Product | Desktop |
| Practice Detail (HQ) | Single practice view: autonomy levels, message history (anonymized), performance metrics, override controls | HQ Ops | Desktop |
| Practice Override | Force a practice to Supervised mode for any/all communication types, with reason logging | HQ Ops | Desktop |
| Reactivation Campaign Monitor | Network-wide view of monthly reactivation campaigns: tier counts, send rates, response rates, revenue attributed | HQ Ops | Desktop |

### 3. Role-Based View Variants

| Role | Sees | Does Not See |
|------|------|-------------|
| CEO / VP Operations | Network Comms Dashboard, Playbook Evolution Report, high-level Practice Health Grid | Individual practice message history, override controls |
| HQ Ops Team Lead | Full Practice Health Grid, Practice Detail, Override controls, Reactivation Monitor | A/B test configuration (view-only) |
| HQ Content / Product | Playbook Designer, Template Editor, A/B Test Manager | Practice override controls, individual practice data |
| Office Manager | Daily Digest, Weekly Summary, Graduation Report, Autonomy Status | Network views, any HQ interface |
| Front Desk Staff | Daily Digest (read-only for Monitored mode — no approval needed), no edit access | All performance data, all HQ interfaces |

---

## Key Screen Wireframe Descriptions

### Practice Interface — Daily Digest

**Layout**: Full-screen mobile. Fixed header with practice name and digest date. Scrollable list of pending message cards. Fixed footer with "Approve All" button.

```
+------------------------------------------+
|  Comms Engine          [Concierge] [Comms]|
|  Garner Dental  •  Monday Digest          |
|  ● Supervised Mode — Treatment Follow-Up  |
+------------------------------------------+
|                                           |
|  3 messages pending review                |
|  Auto-sends by 2:00 PM if not reviewed   |
|                                           |
|  ┌──────────────────────────────────────┐ |
|  │ Treatment Follow-Up · Day 3          │ |
|  │ James H. — Porcelain Veneers         │ |
|  │ ─────────────────────────────────── │ |
|  │ "Hi James, hope you're having a      │ |
|  │ good weekend. I know the veneer      │ |
|  │ conversation was a lot to take in…"  │ |
|  │                                      │ |
|  │  [✓ Approve]  [✎ Edit]  [✗ Reject]  │ |
|  └──────────────────────────────────────┘ |
|                                           |
|  ┌──────────────────────────────────────┐ |
|  │ Treatment Follow-Up · Day 3          │ |
|  │ Lisa M. — Full-Arch Restoration      │ |
|  │ ─────────────────────────────────── │ |
|  │ "Hi Lisa, just checking in on the    │ |
|  │ treatment Dr. Chen recommended…"     │ |
|  │                                      │ |
|  │  [✓ Approve]  [✎ Edit]  [✗ Reject]  │ |
|  └──────────────────────────────────────┘ |
|                                           |
+------------------------------------------+
|        [✓ Approve All 3 Messages]         |
+------------------------------------------+
```

**Message Card Behavior**:
- Tap card body to expand: shows full proposed message, patient context (appointment date, treatment discussed, any note-taker flags), and the 3 action buttons.
- Approve: card collapses with green checkmark, message queued for send.
- Edit: opens Message Edit Drawer.
- Reject: card collapses with gray strikethrough; message is cancelled (not rescheduled).

**Empty State**: "No messages pending review today. The Comms Engine is running autonomously."

---

### Practice Interface — Message Edit Drawer

**Layout**: Slide-up drawer from bottom, covers lower 70% of screen. Dismissible with swipe down.

```
+------------------------------------------+
|  ▽ Edit Message                           |
|  James H. — Veneer Follow-Up · Day 3     |
+------------------------------------------+
|                                           |
|  ┌──────────────────────────────────────┐ |
|  │ Hi James, hope you're having a good   │ |
|  │ weekend. I know the veneer            │ |
|  │ conversation was a lot to take in.    │ |
|  │ If you and your wife have any         │ |
|  │ questions, I'd love to chat — no      │ |
|  │ pressure at all. We also have         │ |
|  │ flexible financing options if that    │ |
|  │ helps. Just reply here or call us.    │ |
|  │                                       │ |
|  │                           152 chars   │ |
|  └──────────────────────────────────────┘ |
|                                           |
|  [Cancel]           [Save & Approve →]    |
|                                           |
+------------------------------------------+
```

**Behavior**:
- Character count updates live; SMS limit is 160 chars (warn at 140, hard limit at 160 with MMS fallback note).
- Edit is logged as a training signal to the message quality model.
- "Save & Approve" queues the edited version for send.

---

### Practice Interface — Weekly Summary View

**Layout**: Full-screen read-only view formatted like a well-structured text message — not a dashboard. Three clearly labeled sections.

```
+------------------------------------------+
|  ← Comms Engine     Weekly Summary       |
|  March 9–15, 2026                         |
+------------------------------------------+
|                                           |
|  ━━━ WHAT THE ENGINE DID ━━━━━━━━━━━━━━  |
|                                           |
|  Last week: 23 messages sent              |
|  • 8 treatment follow-ups                 |
|  • 6 lead responses (2 calls answered,   |
|    4 missed-call text-backs)              |
|  • 5 rebooking nudges                     |
|  • 4 reactivation texts                  |
|                                           |
|  Results:                                 |
|  ✓ 4 appointments booked from leads      |
|  ✓ 2 treatment plans accepted ($14,200)  |
|  ✓ 7 patients rebooked                   |
|  ✓ 3 dormant patients scheduled          |
|                                           |
|  Estimated production influenced: $28,600 |
|                                           |
|  ━━━ NEEDS YOUR ATTENTION ━━━━━━━━━━━━━  |
|                                           |
|  1. Robert Chen called back about his     |
|     crown. He has insurance questions.    |
|     → Call him: (601) 555-0192            |
|     Context: $2,800 crown presented 3/3  |
|                                           |
|  2. Maria Lopez (Invisalign consult Thu)  |
|     has a complex medical history flag.   |
|     → Flag chart for Dr. Patterson       |
|                                           |
|  ━━━ COMING THIS WEEK ━━━━━━━━━━━━━━━━━  |
|                                           |
|  • 5 treatment follow-ups entering Day 14 |
|  • 12 hygiene recalls going out          |
|  • 2 high-value reactivation calls       |
|                                           |
+------------------------------------------+
```

**Behavior**: Pure read-only. Attention items have a "tap to see more" expansion for full patient context if needed. No navigation away from this screen is required to handle the items.

---

### HQ Interface — Network Comms Dashboard

**Layout**: Full-width desktop. Fixed top bar with logo, search, notifications. 4-column card layout for playbook stages. AI alerts sidebar.

```
+----------------------------------------------------------------------+
|  COMMS ENGINE HQ    [Search practices...]  [Bell]  [Avatar]          |
+----------------------------------------------------------------------+
|                                                                      |
|  Network Performance   |  This Week: Mar 9–15, 2026  |  [Export]    |
|                                                                      |
|  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     |
|  │  LEAD RESPONSE  │  │ TREATMENT F/U   │  │   REBOOKING     │     |
|  │                 │  │                 │  │                 │     |
|  │  97.3% <60s     │  │  100% covered   │  │  94% rate       │     |
|  │  ↑ from 41.2%   │  │  47% acceptance │  │  ↑ from 68%     │     |
|  │                 │  │  ↑ from 31%     │  │                 │     |
|  │  Conv: 38%      │  │  +$2.4M/qtr     │  │  Same-day: 82%  │     |
|  │  ● Autonomous   │  │  ● 18/20 auto   │  │  ● 47/50 auto   │     |
|  └─────────────────┘  └─────────────────┘  └─────────────────┘     |
|                                                                      |
|  ┌─────────────────┐                                                 |
|  │  REACTIVATION   │                                                 |
|  │                 │                                                 |
|  │  1,840 / qtr    │                                                 |
|  │  Tier 1: 18%    │                                                 |
|  │  Tier 2: 16%    │                                                 |
|  │  $24M pipeline  │                                                 |
|  │  ● 94/100 auto  │                                                 |
|  └─────────────────┘                                                 |
|                                                                      |
|  ─────────────────────────────────────────────────────────────────  |
|  Practice Attention   14 practices need review                       |
|  [View Practice Health Grid →]                                       |
|                                                                      |
+----------------------------------------------------------------------+
```

---

### HQ Interface — Practice Health Grid

**Layout**: Full-width grid of practice cards, color-coded. Filter bar at top (region, specialty, autonomy level, performance tier).

```
+----------------------------------------------------------------------+
|  Practice Health Grid   [Region ▾] [Specialty ▾] [Autonomy ▾]      |
+----------------------------------------------------------------------+
|                                                                      |
|  ● Autonomous (214)    ● Monitored (31)    ● Supervised (15)        |
|                                                                      |
|  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           |
|  │ ● Green  │  │ ● Green  │  │ ● Yellow │  │ ● Red    │           |
|  │ Garner   │  │ Desert   │  │ Summit   │  │ Peach    │           |
|  │ Dental   │  │ Ridge    │  │ Family   │  │ State    │           |
|  │ MS       │  │ AZ       │  │ CO       │  │ GA       │           |
|  │          │  │          │  │          │  │ New —    │           |
|  │ 97% <60s │  │ 99% <60s │  │ Monitored│  │ 30-day   │           |
|  │ 51% acc  │  │ 44% acc  │  │ 12 pend  │  │ supervised│          |
|  └──────────┘  └──────────┘  └──────────┘  └──────────┘           |
|                                                                      |
+----------------------------------------------------------------------+
```

**Color semantics**:
- Green (Autonomous): all 4 stages in autonomous mode, performance ≥ network average
- Yellow (Monitored): at least one stage in monitored mode, or performance 10-20% below network average
- Red (Supervised): any stage in supervised mode, or newly onboarded within 30 days
- Gray: inactive / not yet deployed

---

### HQ Interface — Playbook Designer

**Layout**: Split-pane desktop. Left: stage selector. Center: visual timeline. Right: touch point configuration panel.

```
+----------------------------------------------------------------------+
|  Playbook Designer                              [Save Draft] [Publish]|
+----------------------------------------------------------------------+
|  Stage: [Treatment Follow-Up ▾]                                      |
+------------------+----------------------------------+-----------------+
|  STAGES          |  TIMELINE                        |  TOUCH CONFIG   |
|                  |                                  |                 |
|  ○ Lead Response |  ●──────●──────●──────●          |  Touch 3        |
|  ● Treatment F/U |  D3     D7     D14    D30         |  ─────────────  |
|  ○ Rebooking     |         │                         |  Delay: Day 14  |
|  ○ Reactivation  |  [+ Add Touch]                   |  Channel: SMS   |
|                  |                                  |  Template:      |
|                  |  If value > $15K:                |  [Financing     |
|                  |  ●──────●──────●──────●──────●   |   Options v2 ▾] |
|                  |  D3  D7  D14  D21  D28           |                 |
|                  |                (voice)            |  [Preview →]    |
+------------------+----------------------------------+-----------------+
```

---

## Interaction Patterns

### Approval Interaction
1. Office manager receives an SMS notification: "3 messages need your review before 2 PM → [link]"
2. Tapping link opens the Daily Digest in the PWA (or prompts install on first use)
3. Manager reviews each card: full-swipe right = approve, full-swipe left = reject, tap = expand + edit
4. "Approve All" button with count confirmation: "Approve all 3 messages? This will send them as shown." → [Confirm]
5. After approval, screen shows: "✓ 3 messages approved. Next digest tomorrow at 8 AM."

### Graduation Recommendation
1. System detects graduation eligibility after 30 days of tracked performance
2. Item appears in weekly summary under "Attention": "Treatment Follow-Up is ready to graduate to Monitored mode. Edit rate: 2.1%, Approval rate: 97%, Response rate: 73%. Recommend advancing?"
3. Office manager taps "Approve Graduation" button in the weekly summary view
4. Autonomy badge on Autonomy Status Screen updates from red "Supervised" to yellow "Monitored"

### HQ Playbook Publish
1. HQ edits cadence or template in Playbook Designer / Template Editor
2. Clicks "Publish" → deployment options modal: Apply to All / Test Cohort / Schedule
3. Confirmation shows impact: "This change will affect 260 practices. Currently-active sequences will continue unchanged; only new triggers will use the updated cadence."
4. After publish, Playbook Evolution Report entry is auto-generated for the monthly report

---

## Component Library

All components extend the Practice Concierge component library. New components specific to Comms Engine:

| Component | Description | Props |
|-----------|-------------|-------|
| `AutonomyBadge` | Color-coded badge (green/yellow/red) with text label | `level: 'autonomous' \| 'monitored' \| 'supervised'` |
| `PendingMessageCard` | Digestible message preview card with approve/edit/reject actions | `message, patientFirstName, stage, scheduledSendAt, onApprove, onEdit, onReject` |
| `MessageEditDrawer` | Slide-up drawer for editing a pending message | `initialText, onSaveAndApprove, onCancel, charLimit` |
| `WeeklySummarySection` | Formatted section block for weekly summary (Engine Activity / Attention / Coming) | `title, items: SummaryItem[]` |
| `AttentionItemCard` | Expandable card for weekly summary escalations | `patientFirstName, context, actionPrompt, phoneNumber?` |
| `GraduationReportCard` | Summary of 30-day metrics with graduation recommendation | `stage, editRate, approvalRate, responseRate, complianceFlags, recommendGraduation, onApprove` |
| `PracticeHealthCard` | HQ grid card for a single practice with autonomy color coding | `practiceName, location, autonomyLevels, metrics` |
| `PlaybookTouchPoint` | Draggable touch point node for the cadence designer timeline | `touchNumber, delayDays, channel, templateId, onConfigure` |
| `TemplatePreview` | Live message preview with sample variable substitution | `templateBody, sampleContext: PatientContext & PracticeContext` |

---

## Color System & Status Semantics

| Token | Hex | Semantic | Usage |
|-------|-----|---------|-------|
| `--autonomy-autonomous` | #22C55E | Practice in Autonomous mode | AutonomyBadge, Practice Health Grid |
| `--autonomy-monitored` | #F59E0B | Practice in Monitored mode | AutonomyBadge, Practice Health Grid |
| `--autonomy-supervised` | #EF4444 | Practice in Supervised/new mode | AutonomyBadge, Practice Health Grid |
| `--message-approved` | #DCFCE7 | Message approved (card background) | PendingMessageCard after approval |
| `--message-rejected` | #F3F4F6 | Message rejected (card background) | PendingMessageCard after rejection |
| `--metric-positive` | #16A34A | KPI trending above baseline | Network Comms Dashboard |
| `--metric-neutral` | #6B7280 | KPI within normal range | Network Comms Dashboard |
| `--metric-attention` | #D97706 | KPI 10-20% below baseline | Network Comms Dashboard |
| `--bg-primary` | #FFFFFF | Card backgrounds | Shared |
| `--bg-surface` | #F9FAFB | Page/screen backgrounds | Shared |

All color uses are paired with a non-color indicator (text label, icon, or pattern) for accessibility.

---

## Typography & Spacing

Inherits from Practice Concierge design system:

- **Font**: Inter (display), SF Pro (iOS system fallback)
- **Body copy**: 16px/24px line-height — legible on small screens without zoom
- **Message preview text in cards**: 14px/20px — enough to scan, tap to expand for full text
- **Weekly summary text**: 16px, formatted like a rich text message — NOT like a data dashboard
- **Section headers in weekly summary**: ALL CAPS, 12px, letter-spacing 0.1em, color `--metric-neutral`
- **Spacing scale**: 4px base unit. Cards use 16px padding. Grid gap is 12px on mobile, 24px on desktop.

---

## Accessibility Requirements

- **WCAG 2.1 AA minimum** across all practice-facing interfaces
- Color is never the only indicator of status — AutonomyBadge always shows text label alongside color
- All interactive elements have a minimum 44×44px touch target (critical for iPhone use between patients)
- Message cards are keyboard-navigable in HQ interfaces
- Screen reader labels on all icon buttons (approve, edit, reject use `aria-label` not just icons)
- The daily digest must be usable with Voice Control on iOS (labeled touch targets)
- Contrast ratios: all text on backgrounds meet 4.5:1 minimum; large text meets 3:1

---

## PWA Configuration

The practice-facing interface is a Progressive Web App built on React, extending the Practice Concierge PWA.

### Service Worker Caching Strategy
- **Daily digest data**: Cache-first with 2-hour TTL; stale while revalidate on re-open. Enables offline access to the current digest.
- **Weekly summary**: Cache indefinitely (static once generated); invalidated on new summary generation.
- **Message approval actions**: Queue offline actions in IndexedDB; sync on reconnect. Approve/reject actions taken offline will sync and fire.

### Push Notifications (Web Push API)
- Trigger: new daily digest available (8 AM) → push: "3 messages are ready for your review before 2 PM."
- Trigger: graduation recommendation available → push: "Treatment Follow-Up is ready to advance to Monitored mode."
- Practice notification opt-in is per-user, managed in app settings.

### Install Prompt
- First-visit overlay after 2nd digest interaction: "Add Comms Engine to your home screen for quick access."
- iOS: native "Add to Home Screen" instructions (Apple doesn't support native install prompt in Safari).
- Android: standard PWA `beforeinstallprompt` event.

### Supported Environments
- iPhone Safari (iOS 15+)
- iPad Safari (iOS 15+)
- Android Chrome (Chrome 90+)
- Desktop Chrome/Safari (for office managers using desktop for digest review)
