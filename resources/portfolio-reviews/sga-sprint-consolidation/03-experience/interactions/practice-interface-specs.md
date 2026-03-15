# SGA Dental Partners — Practice Interface Interaction Specs

**Date**: 2026-03-14
**Author**: UX Designer Agent (Phase 6)
**Surface**: Practice Operations (WS1 Surface A)
**Interfaces Covered**: PO-1 through PO-9 (Practice Operations) + PWA mobile interactions
**Carry-Forward Items Addressed**: PWA role-based screen filtering (Prototype Lead #4), offline behavior and sync patterns, WCAG offline indicators

---

## 1. PWA Role-Based Screen Filtering

### 1.1 Role Definitions and Screen Access

The PWA shows only screens relevant to the authenticated user's role (from Auth & Identity Service). This replaces the original 7-screen-for-everyone design.

| Role | Visible Screens | Bottom Nav Items | Rationale |
|------|----------------|-----------------|-----------|
| **Office Manager** | Today, Patients, Automations, Storefront | 4 items in bottom nav | Manages daily operations, patient outreach, automation oversight, digital presence |
| **Front Desk** | Today, Patients, Calls | 3 items in bottom nav | Handles calls, patient check-in, immediate outreach tasks |
| **Clinical Staff / Champion** | Today, Capture, Huddle | 3 items in bottom nav | Content capture between patients, morning huddle reference, daily agenda awareness |
| **Champion (dual role)** | Today, Capture, Huddle, Champion Hub | 4 items in bottom nav | Same as clinical staff plus champion scorecard and missions |

**Implementation**:
- Role is determined at login from the Auth & Identity Service JWT token
- The `role` claim in the JWT maps to a screen visibility configuration
- Staff with multiple roles (e.g., office manager who is also champion) see the union of their role screens (max 5)
- Bottom navigation renders only the authorized screens. No hidden screens, no locked icons, no "upgrade to access" gates
- If a user's role changes (e.g., champion status revoked), the PWA removes that screen on next session refresh

### 1.2 Bottom Navigation Pattern

```
Office Manager:
┌──────────────────────────────────────────┐
│  [ Today ]  [ Patients ]  [ Auto ]  [ Site ]  │
└──────────────────────────────────────────┘

Front Desk:
┌──────────────────────────────────────────┐
│     [ Today ]     [ Patients ]    [ Calls ]    │
└──────────────────────────────────────────┘

Clinical Staff / Champion:
┌──────────────────────────────────────────┐
│     [ Today ]     [ Capture ]    [ Huddle ]    │
└──────────────────────────────────────────┘

Champion (dual role):
┌──────────────────────────────────────────┐
│  [ Today ]  [ Capture ]  [ Huddle ]  [ Hub ]  │
└──────────────────────────────────────────┘
```

**Navigation Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Tap nav item | Content area transitions with 200ms crossfade. Active item: filled icon + label in `accent-primary`. Inactive items: outline icon + label in `text-muted` |
| Long press nav item | Tooltip shows full label (e.g., "Automations" if truncated to "Auto") |
| Swipe left/right on content | Navigates to adjacent screen. Swipe gesture matches bottom nav order |
| Screen reader | Each nav item announces: "[Screen name], tab [position] of [total]" |

### 1.3 PWA Settings and Profile Access

A user avatar / initials circle in the top-right corner of every screen opens a slide-up panel:

```
┌──────────────────────────────┐
│  Lisa Torres                 │
│  Office Manager              │
│  Smile Center Buckhead       │
│                              │
│  Playbooks                   │  ← Available to all roles
│  Settings                    │  ← Notification prefs, display prefs
│  Help & Support              │
│  Sign Out                    │
│                              │
│  ─── ─── ─── (drag to close)│
└──────────────────────────────┘
```

This keeps Playbooks accessible to all roles without consuming a bottom nav slot.

---

## 2. Today Screen (PO-1: Practice Daily Agenda)

### 2.1 Layout

The Today screen is the default landing for all roles. It is organized by time of day.

**Mobile Layout** (375px-428px width):
```
┌──────────────────────────────┐
│  SGA ♦ Smile Center Buckhead │  ← Practice name + logo
│  Friday, March 14            │  ← Date
├──────────────────────────────┤
│                              │
│  MORNING BRIEFING            │  ← Section header
│  ┌──────────────────────┐    │
│  │  Action Card 1       │    │  ← Swipeable horizontal cards
│  │  7 confirmations due │    │
│  │  [Review & Send]     │    │
│  └──────────────────────┘    │
│  ● ○ ○                      │  ← Pagination dots (3 cards)
│                              │
│  H-SCORE                     │
│  ┌──────────────────────┐    │
│  │  76  ▲2 this week    │    │  ← Compact H-Score widget
│  │  Focus: Warm Handoffs│    │
│  └──────────────────────┘    │
│                              │
│  TODAY'S SCHEDULE            │
│  ┌──────────────────────┐    │
│  │  24 appointments     │    │
│  │  3 new patients      │    │
│  │  2 high-value cases  │    │
│  └──────────────────────┘    │
│                              │
│  QUICK STATS                 │
│  ┌──────────────────────┐    │
│  │  Msgs sent: 12       │    │
│  │  Responses: 8        │    │
│  │  Captures: 1         │    │
│  └──────────────────────┘    │
│                              │
│  [▼ End-of-Day Scorecard]    │  ← Collapsed by default, expands after 4 PM
│                              │
├──────────────────────────────┤
│  [ Today ]  [ ... ]  [ ... ] │  ← Bottom nav (role-filtered)
└──────────────────────────────┘
```

### 2.2 Action Cards

3 AI-generated action cards are the centerpiece of the morning briefing. They are presented as swipeable horizontal cards.

**Card Types** (AI selects top 3 from available actions):

| Card Type | Source | Example |
|-----------|--------|---------|
| Confirmation batch | Automation Engine | "7 patients need confirmation for tomorrow. AI has drafted messages." |
| Follow-up due | Comms Engine | "3 treatment follow-ups due today. Mrs. Patterson's implant consult was 6 days ago." |
| Content capture opportunity | Content Engine (via Kafka) | "2 implant cases today. Capture checklist ready." |
| Reactivation candidates | Funnel Data | "5 patients overdue >6 months. Reactivation messages drafted." |
| Rebooking needed | PMS Data | "8 patients from last week haven't rebooked. Follow up today." |
| Escalation | Escalation Manager | "2 Voice AI calls flagged for review. Listen and resolve." |

**Card Anatomy**:
```
┌──────────────────────────────────┐
│  [icon]  CONFIRMATIONS           │  ← Category icon + label
│                                  │
│  7 patients need confirmation    │  ← Primary message, 16px semibold
│  for tomorrow. AI has drafted    │
│  personalized messages.          │
│                                  │
│  ⏱ ~3 min to complete            │  ← Time estimate
│                                  │
│  [Review & Send →]               │  ← Primary action button
└──────────────────────────────────┘
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Swipe left | Reveals next action card. Smooth 300ms spring animation. Pagination dots update |
| Swipe right | Returns to previous card |
| Tap action button | Navigates to the relevant workflow (e.g., Patient Outreach Queue filtered to confirmations). Transition: slide-left into the detail view |
| Complete all actions | Cards show checkmark overlay. Section header changes to "Morning Briefing -- Complete!" |
| Card priority | Cards are ordered by time-sensitivity. Confirmations (tomorrow's patients) before follow-ups (6-day-old consult) before captures (today's cases) |

### 2.3 H-Score Widget (Cross-Workstream Touchpoint T-1)

**Data Source**: `sgadental.ws3.hscore-calculated` event, consumed via the shared data layer. Daily batch update at 6 AM ET.

**Compact View** (default on Today screen):
```
┌──────────────────────────────┐
│  ■ 76  ▲2 this week         │  ← Score + trend
│  Focus today: Warm Handoffs  │  ← Today's behavior from Culture OS
└──────────────────────────────┘
```

**Expanded View** (tap to expand):
```
┌──────────────────────────────────┐
│  Practice H-Score                │
│  ■ 76  ▲2 this week             │
│                                  │
│  ── 7-day sparkline ──          │
│                                  │
│  Phone Quality (35%)    82      │  ← Sub-score bars
│  ████████████████░░░░           │
│  Reviews (45%)          71      │
│  ██████████████░░░░░░           │
│  Dental Intel (20%)     74      │
│  ██████████████░░░░░░           │
│                                  │
│  Focus: Warm Handoffs            │
│  "Introduce every patient to     │
│  their next caregiver by name."  │
│                                  │
│  [Close]                         │
└──────────────────────────────────┘
```

**This is a modal that opens within Practice Operations.** The user never navigates to Culture OS. The modal uses the same card and typography patterns as the rest of Practice Operations.

**Stale Data Handling**: If the H-Score is >48 hours old, display below the score: "Updated 2 days ago" in 11px `text-muted`. No error state -- just transparency.

### 2.4 Content Capture Brief (Cross-Workstream Touchpoint T-2)

When the AI Analysis Engine determines there are capture-worthy procedures today (from PMS appointment data), Action Card #3 is a content capture brief.

**Card Content**:
```
┌──────────────────────────────────┐
│  [camera icon]  CONTENT CAPTURE  │
│                                  │
│  2 implant cases and 1 full-     │
│  arch consult today. Before/     │
│  after photos for implants       │
│  would be great.                 │
│                                  │
│  📋 Capture checklist:           │
│  □ Before photo (implant #1)     │
│  □ After photo (implant #1)      │
│  □ Before photo (implant #2)     │
│  □ After photo (implant #2)      │
│                                  │
│  [Open Camera →]                 │
└──────────────────────────────────┘
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Tap "Open Camera" | Opens the Capture screen (Content Engine mobile interface) as a full-screen overlay. The user stays within the PWA -- it is not a navigation to a separate app |
| Checklist items | Tappable to mark complete. Completion state persists locally (IndexedDB) and syncs when online |
| Dismiss | Swipe the card away. It moves to "Dismissed" and does not reappear today |

### 2.5 End-of-Day Scorecard

Collapsed by default. Automatically expands after 4 PM local practice time.

**Expanded View**:
```
┌──────────────────────────────────┐
│  END-OF-DAY SCORECARD            │
│  Friday, March 14                │
│                                  │
│  Actions Completed    3 / 3  ✓   │
│  Calls Handled        22         │
│    Human: 18  |  Voice AI: 4     │
│  Messages Sent        12         │
│  Responses Received   8 (67%)    │
│  Content Captures     2          │
│  Revenue Impact Est.  $3,200     │
│                                  │
│  Compared to last Friday:        │
│  Calls ▲3  |  Msgs ▼1  |  Rev ▲$400 │
│                                  │
└──────────────────────────────────┘
```

---

## 3. Patient Outreach Queue (PO-2)

### 3.1 Layout

**Mobile Layout**:
```
┌──────────────────────────────┐
│  Patient Outreach            │
│  [Pending (7)] [Sent (12)] [Escalated (2)] │  ← Segmented control
├──────────────────────────────┤
│                              │
│  ┌──────────────────────┐    │
│  │  David Kim            │   │
│  │  Confirmation for Thu │   │
│  │  10 AM with Dr. Patel │   │
│  │                       │   │
│  │  "Hi David, this is   │   │  ← AI-drafted message preview
│  │  Smile Center..."     │   │
│  │                       │   │
│  │  [Edit] [Approve ✓]  │   │
│  └──────────────────────┘    │
│                              │
│  ┌──────────────────────┐    │
│  │  Mrs. Patterson       │   │
│  │  Follow-up (Day 6)   │   │
│  │  Implant consultation │   │
│  │  ...                  │   │
│  └──────────────────────┘    │
│                              │
│  [Approve All (7) ✓]         │  ← Batch approve button
│                              │
└──────────────────────────────┘
```

### 3.2 Message Card Interactions

| Interaction | Behavior |
|-------------|----------|
| Tap card | Expands to show full AI-drafted message text, patient context (last visit, funnel stage, communication history), and channel indicator (SMS/Email) |
| "Edit" | Opens inline text editor. The AI-drafted message is pre-filled and fully editable. Character count shown for SMS (160 char limit indicator per WCAG carry-forward). Save returns to card view |
| "Approve" | Sends the message immediately via Communications Service. Card animates to "Sent" state (checkmark, green tint, slides to "Sent" tab after 2 seconds) |
| "Approve All" | Confirmation dialog: "Send 7 messages now? [Cancel] [Send All]". On confirm, all cards animate to sent state in sequence (150ms stagger) |
| Swipe left on card | Reveals "Skip" action (red). Skipping removes the message from the queue without sending. Reason required: "Not now", "Wrong patient", "Manual handling" |
| Swipe right on card | Quick approve (same as tap "Approve") |

**SMS Character Limit (WCAG Carry-Forward)**:

When editing an SMS message:
```
┌──────────────────────────────────┐
│  Edit Message                    │
│                                  │
│  ┌────────────────────────────┐  │
│  │  Hi David, this is Smile   │  │
│  │  Center Buckhead. Looking  │  │
│  │  forward to seeing you     │  │
│  │  Thursday at 10am...       │  │
│  └────────────────────────────┘  │
│                                  │
│  142 / 160 characters            │  ← Counter
│  ████████████████████████░░░░░   │  ← Visual bar
│                                  │
│  ⚠ Messages over 160 characters │  ← Warning at 140+
│  will be split into 2 texts.     │
│                                  │
│  [Cancel]  [Save]                │
└──────────────────────────────────┘
```

- Counter turns `status-warning` (amber) at 140 characters
- Counter turns `status-critical` (red) at 161+ characters with explanatory text about message splitting
- Screen reader announces character count on every keystroke pause (300ms debounce)

### 3.3 Segmented Control Tabs

| Tab | Content |
|-----|---------|
| Pending | Messages awaiting approval. Badge shows count. Default view |
| Sent | Messages sent today. Shows delivery status: "Delivered", "Read", "Responded", "Failed". Tap to see response |
| Escalated | Messages flagged by the Escalation Manager for human review. Reason shown: "Patient requested human callback", "Negative sentiment detected", "Opt-out attempt" |

---

## 4. Call Dashboard (PO-3)

### 4.1 Desktop Layout (Front Desk Primary Interface)

The Call Dashboard is primarily a desktop interface for the front desk workstation. It is the one Practice Operations screen optimized for desktop first.

```
┌──────────────────────────────────────────────────────────────┐
│  Calls Today                                    [Live: 2]    │
├──────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────┐  ┌───────────────────┐ │
│  │  LIVE CALLS                     │  │  CONTEXT CARD     │ │
│  │                                 │  │                   │ │
│  │  📞 (555) 867-5309              │  │  David Kim        │ │
│  │  David Kim — Existing Patient   │  │  Last visit: 2/14 │ │
│  │  Duration: 2:34                 │  │  Next appt: 3/18  │ │
│  │  [Transfer] [Hold] [End]        │  │  Implant consult  │ │
│  │                                 │  │  scheduled        │ │
│  │  📞 (555) 234-7890              │  │                   │ │
│  │  Unknown Caller — New Lead      │  │  Preferred: SMS   │ │
│  │  Voice AI handling              │  │  Notes: Anxious   │ │
│  │  Duration: 1:12                 │  │  about dental     │ │
│  │                                 │  │  work. Be gentle. │ │
│  └─────────────────────────────────┘  └───────────────────┘ │
├──────────────────────────────────────────────────────────────┤
│  TODAY'S CALL LOG                                            │
│  ┌─────────────┬────────┬──────────┬─────────┬───────────┐  │
│  │ Time        │ Caller │ Type     │ Handler │ Outcome   │  │
│  │ 9:15 AM     │ Kim, D │ Existing │ Human   │ Scheduled │  │
│  │ 9:02 AM     │ New    │ New Lead │ Voice AI│ Booked    │  │
│  │ 8:47 AM     │ Patel  │ Existing │ Human   │ Info Req  │  │
│  │ 8:30 AM     │ New    │ Unknown  │ Voice AI│ Voicemail │  │
│  └─────────────┴────────┴──────────┴─────────┴───────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Context Card Interactions

When a call comes in, the context card auto-populates:

| Patient Type | Context Card Content |
|-------------|---------------------|
| Known patient (caller ID match) | Name, last visit date, upcoming appointments, preferred name, communication history summary, funnel stage, channel preference, any notes |
| Partial match | "Possible match: [Name]. Verify identity." Shows candidate matches from PMS |
| Unknown / New caller | "New caller. Start intake?" with a button to begin new patient capture |

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Incoming call | Context card slides in from right (300ms). If multiple live calls, cards stack vertically. Active call is highlighted |
| Click patient name | Expands to full patient detail (appointment history, communication log). Opens in a side panel without leaving the Call Dashboard |
| "Transfer" button | Opens a provider/staff picker dropdown. Transferring logs the handoff in the call record |
| "Log Outcome" (after call ends) | Dropdown: "Scheduled appointment", "Information request", "Insurance question", "Callback requested", "No action needed". Outcome feeds into ClickHouse analytics |
| Voice AI active | If Voice AI is handling a call, the live call card shows "Voice AI handling" badge with a live transcript streaming in real-time below |

### 4.3 Mobile View (PO Calls Screen — Office Managers Only)

On mobile (PWA), the Calls screen shows a simplified view:

```
┌──────────────────────────────┐
│  Voice AI Activity           │
│                              │
│  Today: 4 calls handled      │
│  ✓ 2 appointments booked     │
│  ⚠ 1 escalation (flagged)   │
│  ✗ 1 went to voicemail       │
│                              │
│  [Review Transcripts]        │
│                              │
│  MISSED CALLS (3)            │
│  ┌──────────────────────┐    │
│  │  (555) 345-6789      │    │
│  │  9:15 AM — New caller │    │
│  │  Auto-text sent ✓    │    │
│  │  [Call Back] [View]  │    │
│  └──────────────────────┘    │
│                              │
└──────────────────────────────┘
```

---

## 5. Automation Status Board (PO-5)

### 5.1 Layout

```
┌──────────────────────────────┐
│  Automations                 │
│  [Active (4)] [Paused (1)]   │
├──────────────────────────────┤
│                              │
│  CONFIRMATIONS               │
│  Status: Autonomous ●        │  ← Green dot = autonomous
│  23 sent today | 19 confirmed│
│  Trust level: ████████░░ 80% │
│                              │
│  FOLLOW-UPS                  │
│  Status: Monitored ●         │  ← Amber dot = monitored
│  5 drafts awaiting approval  │
│  Trust level: ██████░░░░ 60% │
│  [Review Drafts]             │
│                              │
│  REBOOKING                   │
│  Status: Supervised ●        │  ← Blue dot = supervised
│  8 candidates identified     │
│  Trust level: ████░░░░░░ 40% │
│  [Review & Approve]          │
│                              │
│  REACTIVATION                │
│  Status: Supervised ●        │
│  12 candidates this week     │
│  [Review Campaign]           │
│                              │
└──────────────────────────────┘
```

### 5.2 Graduation Status Interactions

Each automation type progresses through trust levels: Supervised > Monitored > Autonomous.

| Trust Level | Behavior | User Action Required |
|-------------|----------|---------------------|
| Supervised (0-50%) | All messages require explicit approval before sending | Review and approve every message |
| Monitored (51-80%) | Messages send automatically but user can review/override within 30-min window | Review daily summary; override if needed |
| Autonomous (81-100%) | Messages send automatically. User sees results in scorecard | Monitor weekly performance |

**Graduation Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Tap automation card | Expands to show: active sequences count, recent performance (delivery rate, response rate, conversion rate), graduation progress bar, "Pause" toggle |
| Tap "Pause" | Confirmation: "Pause [type] automation? Active sequences will complete but no new ones will start." Toggle switches to paused state. Card moves to "Paused" tab |
| Tap "Resume" (from Paused tab) | Confirmation: "Resume [type] automation? New sequences will start from the queue." |
| Trust level bar tap | Modal showing graduation criteria: "Confirmations will graduate to Monitored after 50 consecutive messages with <2% override rate and >90% delivery rate. Current: 47/50 consecutive, 0.8% override, 94% delivery." |

---

## 6. Storefront Manager (PO-6)

### 6.1 Desktop Layout (Weekly Interaction)

```
┌──────────────────────────────────────────────────────────────┐
│  Storefront                                                   │
│  Digital Health Score: 88 / 100                               │
│  ┌──────────────────────────────────────────────────────────┐│
│  │  ⚠ 1 issue: Provider photo for Dr. Kim is 8 months old  ││
│  └──────────────────────────────────────────────────────────┘│
├──────────────────────────────────────────────────────────────┤
│  [Website] [Google Business] [Directories] [Asset Gallery]   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Website Tab:                                                │
│  ┌──────────────────────────┐  ┌─────────────────────────┐  │
│  │  Live Website Preview    │  │  Quick Actions           │  │
│  │  (iframe, 60% width)    │  │  □ Update provider photo │  │
│  │                          │  │  □ Add new gallery images│  │
│  │                          │  │  □ Update hours          │  │
│  │                          │  │                          │  │
│  └──────────────────────────┘  │  Recent Content (from    │  │
│                                │  Content Engine):         │  │
│                                │  [img] Implant B/A 3/12  │  │
│                                │  [img] Team photo 3/10   │  │
│                                │  [Use on Website]        │  │
│                                └─────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

### 6.2 Asset Gallery (Cross-Workstream Touchpoint T-3)

**Data Source**: Asset Registry (owned by Content Engine WS2). Practice Operations reads approved assets filtered to this practice.

```
┌──────────────────────────────────────────────────────────────┐
│  Asset Gallery                         [Filter ▾] [Sort ▾]  │
│  12 approved assets available                                │
├──────────────────────────────────────────────────────────────┤
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐                    │
│  │ img  │  │ img  │  │ img  │  │ img  │                    │
│  │      │  │      │  │      │  │      │                    │
│  │B/A   │  │Team  │  │Test- │  │Promo │                    │
│  │Implnt│  │Photo │  │monial│  │Flyer │                    │
│  │3/12  │  │3/10  │  │3/8   │  │3/5   │                    │
│  └──────┘  └──────┘  └──────┘  └──────┘                    │
│                                                              │
│  Selected: 2 assets                                          │
│  [Use on Website]  [Push to GMB]  [Download]                │
└──────────────────────────────────────────────────────────────┘
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Tap asset thumbnail | Lightbox opens with full-size preview, metadata (type, date, brand universe), usage history ("Published to Instagram 3/13") |
| Tap to select (checkbox mode) | Checkbox appears on top-left of each thumbnail. Selected count updates. Batch action bar appears at bottom |
| "Use on Website" | Confirmation: "Add 2 photos to your website gallery? [Cancel] [Add]". Assets are placed on the practice website via Channel Sync Engine. Success toast: "2 photos added to your website." |
| "Push to GMB" | Same pattern. Assets are synced to Google Business Profile. May take 24-48h to appear (Google processing). Toast: "2 photos queued for Google Business Profile. They'll appear within 48 hours." |
| Filter | By type: Before/After, Team Photo, Testimonial, Social Post, Flyer. By date range |
| Sort | By date (newest first, default), by type, by usage count |

**Key Design Note**: The user sees "Asset Gallery" and "approved photos." They do not see "Content Engine" or "WS2." The cross-workstream data transfer is invisible.

---

## 7. Offline Behavior and Sync Patterns

### 7.1 Offline Capability Matrix

| Screen | Offline Capable | Cached Data | User Actions Offline |
|--------|----------------|-------------|---------------------|
| Today | Partial | Morning briefing agenda, action card list, H-Score, schedule summary. Cached at 6 AM and on every app open | Read agenda. Cannot approve/send messages |
| Patients | Partial | Patient list with basic info. Cached on last successful load | Browse patients. Cannot send messages or update records |
| Calls | No | Requires live connection | Banner: "Calls require an internet connection" |
| Automations | Partial | Current automation status and trust levels. Cached on last load | View status. Cannot pause/resume |
| Storefront | No | Website preview requires connection. Asset gallery thumbnails cached | View cached thumbnails only |
| Capture | Yes | Camera interface works offline. Photos/videos stored in IndexedDB | Take photos, record video, tag with metadata. Upload queues for when connection returns |
| Huddle | Yes | Huddle agenda cached nightly at midnight local time | View full huddle content |
| Champion Hub | Partial | Scorecard and missions cached. Leaderboard requires connection | View scorecard and current missions |
| Playbooks (via Settings) | Yes | Full playbook content cached. Updated weekly or on-demand | Read all playbook content |

### 7.2 Offline Indicator (WCAG Carry-Forward Item)

When the PWA loses connectivity, a persistent banner appears at the top of every screen:

```
┌──────────────────────────────────────┐
│  ■ You're offline — showing cached   │  ← Banner with icon + text
│    data. Last updated: 8:15 AM       │
└──────────────────────────────────────┘
```

**Design Requirements**:
- Banner uses `surface-warning` background with `text-on-warning` text
- Leading icon: cloud with diagonal line through it (not color-dependent)
- Text explicitly states "offline" and provides the last sync timestamp
- Banner is **not** dismissible while offline (it persists until connection restores)
- Screen reader announces: "You are offline. Showing cached data last updated at 8:15 AM"
- When connection restores: banner transitions to green "Back online — syncing..." for 3 seconds, then disappears. Any queued actions (captures, message approvals) sync automatically

### 7.3 Offline Action Queuing

Actions taken while offline are queued in IndexedDB and synced on reconnection:

| Action | Offline Behavior | Sync Behavior |
|--------|-----------------|---------------|
| Content capture (photo/video) | Stored in IndexedDB with metadata (practice, procedure type, date, tags). Success feedback: "Photo saved. Will upload when online." | On reconnection: uploads queue in order. Progress indicator: "Uploading 3 of 5 captures..." Each successful upload shows a brief toast |
| Checklist completion | State stored locally | Synced silently |
| Scorecard/huddle viewed | View tracked locally | Engagement metrics synced silently |
| Message approval attempt | Shows toast: "Messages can't be sent while offline. They'll be ready to approve when you reconnect." | Approval queue is refreshed on reconnection (messages may have been sent by desktop user in the interim) |

### 7.4 Background Sync and Service Worker

**Service Worker Caching Strategy**:

| Resource | Strategy | TTL |
|----------|----------|-----|
| App shell (HTML, CSS, JS) | Cache-first | Until next deployment (versioned) |
| Practice profile data | Stale-while-revalidate | 24 hours |
| Today's agenda | Network-first with cache fallback | Refreshed on every app open |
| Playbook content | Cache-first, update on explicit pull | 7 days |
| Huddle content | Cache-first, updated nightly | 24 hours |
| Asset gallery thumbnails | Cache-first | 7 days |
| API responses (patient data, messages) | Network-only | No cache (PHI concern) |

**Critical Note on PHI**: Patient-specific data (names, contact info, communication history) is **never cached** in the service worker or IndexedDB. The offline "Patients" screen shows only the patient list with names (cached in encrypted IndexedDB), not full records. This is a HIPAA requirement.

**Push Notifications** (Web Push API):

| Notification Type | Trigger | Sound |
|------------------|---------|-------|
| Escalation alert | Voice AI escalates a call to human | Default system sound |
| New action card | AI generates a time-sensitive action (e.g., patient cancellation) | Default system sound |
| Capture reminder | Content Engine sends daily capture brief | Silent (badge only) |
| Huddle reminder | 5 minutes before scheduled huddle time | Default system sound |

Notification preferences are configurable per type in Settings. All can be individually enabled/disabled.

---

## 8. Morning Huddle Screen (PO-8 / CO-1)

### 8.1 TV Display Mode

When displayed on the break room TV (via Dental Intel integration or direct URL):

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│                    SMILE CENTER BUCKHEAD                          │
│                    Friday Morning Huddle                          │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │  H-SCORE         │  │  FOCUS TODAY     │  │  RECOGNITION  │  │
│  │                  │  │                  │  │               │  │
│  │    76            │  │  Warm Handoffs   │  │  Shoutout to  │  │
│  │   ▲ 2 this week │  │                  │  │  Maria for    │  │
│  │                  │  │  "Introduce      │  │  3 five-star  │  │
│  │  Phone: 82      │  │  every patient   │  │  reviews      │  │
│  │  Reviews: 71    │  │  to their next   │  │  this week!   │  │
│  │  DI: 74         │  │  caregiver by    │  │               │  │
│  │                  │  │  name."          │  │               │  │
│  └──────────────────┘  └──────────────────┘  └───────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────────┐│
│  │  IMPACT STORY: Our show rate improved 5% since we started   ││
│  │  the confirmation text sequence. That's ~4 more patients/wk.││
│  └──────────────────────────────────────────────────────────────┘│
│                                                                  │
│  [Auto-advances in 30s]  ● ○ ○ ○                               │
└──────────────────────────────────────────────────────────────────┘
```

The huddle display auto-cycles through 4 panels every 30 seconds. Total huddle content: ~2 minutes of passive viewing, leaving 3 minutes for team discussion.

### 8.2 PWA Huddle Screen

Same content as TV display, formatted for mobile (vertical scroll instead of auto-cycle). Includes a "Mark Huddle Complete" button at the bottom. Tapping it logs completion to Culture OS.

---

## 9. Voice AI Monitor (PO-4)

### 9.1 Voice AI TTY/TDD Support (WCAG Carry-Forward Item)

The Voice AI system must support TTY/TDD relay services for deaf and hard-of-hearing callers. This is an ADA requirement for healthcare-adjacent phone services.

**Implementation Requirements**:

| Requirement | Specification |
|-------------|--------------|
| TTY/TDD relay detection | Twilio Voice detects relay calls via 2-tone handshake. Routes to a separate handler that presents text-based interaction |
| Relay call handling | Voice AI switches to text-based mode. Responses are sent as text through the relay service. All booking and information capabilities remain available |
| Transcript logging | TTY/TDD calls are logged with a "relay" flag. Transcripts are stored in the same format as voice transcripts |
| UI indicator | In the Voice AI Monitor, relay calls show a TTY badge: "[TTY] Call from relay service — handled via text" |
| Fallback | If relay detection fails, the Voice AI's natural language processing handles relay operator verbal pass-through |

### 9.2 Transcript Review Interactions

```
┌──────────────────────────────┐
│  Voice AI Transcripts        │
│  Today: 4 calls handled      │
├──────────────────────────────┤
│                              │
│  ┌──────────────────────┐    │
│  │  9:02 AM — New Caller │    │
│  │  Duration: 3:12       │    │
│  │  Outcome: Booked ✓    │    │
│  │  Quality: ████░ 82%   │    │
│  │  [Listen] [Read]      │    │
│  └──────────────────────┘    │
│                              │
│  ┌──────────────────────┐    │
│  │  8:47 AM — Known Pt  │    │
│  │  Duration: 1:45       │    │
│  │  Outcome: Info Given  │    │
│  │  ⚠ Flagged: Caller    │    │
│  │  seemed frustrated    │    │
│  │  [Listen] [Read]      │    │
│  └──────────────────────┘    │
│                              │
└──────────────────────────────┘
```

| Interaction | Behavior |
|-------------|----------|
| "Listen" | Audio player inline. Play/pause, 15-second skip, playback speed (1x, 1.5x, 2x) |
| "Read" | Full transcript view with speaker labels (AI / Caller). Flagged moments highlighted in `status-warning` |
| "Flag for Training" | Opens a note field. Flag is sent to Culture OS Trust Transfer Tracker. Appears in the practice's weekly training review |

---

## 10. Playbook Viewer (PO-9)

### 10.1 Access and Layout

Playbooks are accessible from the Settings/Profile slide-up panel on mobile (all roles) and from a dedicated section on desktop.

```
┌──────────────────────────────┐
│  Playbooks                   │
│  [Phone Scripts] [Follow-Up] │
│  [Onboarding] [Hospitality]  │
├──────────────────────────────┤
│                              │
│  Phone Scripts v2.3          │
│  Updated: March 10, 2026     │
│                              │
│  New Patient Call             │
│  ────────────────────────    │
│  1. Greeting: "Thank you     │
│     for calling [Practice].  │
│     This is [Name], how      │
│     can I help you today?"   │
│                              │
│  2. If insurance question:   │
│     "Let me look into that   │
│     for you..."              │
│                              │
│  [What Changed in v2.3 ▾]    │  ← Expandable version diff
│                              │
└──────────────────────────────┘
```

**Offline**: Full playbook content is cached. Available without connection.

**Version Diff**: "What Changed" expands to show additions (green highlight) and removals (red strikethrough) from the previous version. Office managers can see exactly what HQ updated.

---

*This specification covers all Practice Operations interfaces (PO-1 through PO-9) and the PWA mobile experience with role-based filtering. All cross-workstream touchpoints (T-1, T-2, T-3, T-5) are specified to feel native to Practice Operations -- users never see workstream labels or navigate to external surfaces.*
