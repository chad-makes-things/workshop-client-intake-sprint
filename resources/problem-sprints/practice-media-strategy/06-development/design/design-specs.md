# UX Design Specification — The Content Concierge

> Practice Media Capture, Production & Distribution Platform for SGA Dental Partners
> Practice Media Strategy — Sprint Development Package

---

## Design Principles

1. **Zero-friction capture** — A champion in a busy dental practice has 30 seconds between patients. The capture action must take under 60 seconds end-to-end, with zero login, zero navigation, zero forms. SMS reply is the correct Phase 1 interface.

2. **Guidance over instruction** — Champions aren't photographers. Every prompt includes specific, actionable guidance ("Front + 45-degree angle, fill frame with smile area"). Not "take a photo." Context eliminates excuses.

3. **Celebrate the streak, not the task** — Behavior change requires positive reinforcement. The streak counter is the most important UI element in Phase 1. Every confirmation surfaces it. When it breaks, the system responds warmly, not punitively.

4. **HQ sees everything; practices see only their content** — The content platform is multi-tenant. Practice champions see their practice. HQ sees the network. These are different resolution levels of the same data, not different systems.

5. **Editors, not curators** — The Airtable review queue is an editorial workflow, not a curation feed. Editors apply judgment (approve/reject) and produce captions. The system does formatting and distribution. Editors shouldn't be doing distribution work.

6. **Phase 1 is human-assisted; Phase 2 is AI-assisted** — The design system must accommodate the handoff. Nothing in Phase 1 should be "workarounds" — the SMS flow, the Airtable queue, and the admin dashboard are appropriate for Phase 1 scale. Phase 2 replaces the delivery mechanism (app, AI pipeline), not the mental model.

7. **Brand consistency across all 260 practices** — Content Concierge output should be visually indistinguishable in quality from agency-produced content. Template application and brand guidelines are not optional enhancements — they are part of the core value proposition.

---

## Screen Inventory

### Phase 1 Admin Dashboard (Internal HQ Tool)

| Screen | Path | Primary Users | Device |
|--------|------|---------------|--------|
| Dashboard Home | `/` | HQ Admin | Desktop |
| Practices List | `/practices` | HQ Admin | Desktop |
| Practice Detail | `/practices/[id]` | HQ Admin | Desktop |
| Add Practice | `/practices/new` | HQ Admin | Desktop |
| Champions Management | `/practices/[id]/champions` | HQ Admin | Desktop |
| Captures Monitor | `/captures` | HQ Admin, Editor | Desktop |
| Assignments Monitor | `/assignments` | HQ Admin | Desktop |
| Playbook Library | `/playbooks` | HQ Admin | Desktop |
| Sync Health | `/sync` | HQ Admin | Desktop |

### Phase 1 Champion Interface (SMS/WhatsApp)

| Interaction | Trigger | Channel |
|-------------|---------|---------|
| Onboarding welcome | Practice created + champion added | SMS |
| Daily capture brief | Nightly scheduler | SMS |
| Capture confirmation | Inbound MMS/WhatsApp received | SMS reply |
| Weekly recap | Monday 8 AM | SMS |
| Rejection feedback | Editor marks rejected in Airtable | SMS |
| STATUS command response | Champion texts STATUS | SMS reply |

### Phase 2 (Planned — Design Direction Only)

| Screen | Users | Device |
|--------|-------|--------|
| Mobile App: Home / Today's Assignments | Champions | iOS + Android |
| Mobile App: Capture Mode | Champions | iOS + Android |
| Mobile App: Content Feed | Champions | iOS + Android |
| Mobile App: Scorecard | Champions | iOS + Android |
| HQ Dashboard: Review Queue | Editors, HQ Admin | Desktop |
| HQ Dashboard: Network Pipeline | HQ Admin | Desktop |
| HQ Dashboard: Practice Health Grid | HQ Admin | Desktop |
| HQ Dashboard: Analytics | HQ Admin | Desktop |

---

## Admin Dashboard Design Specs (Phase 1)

### Design System

**Framework**: shadcn/ui components with Tailwind CSS
**Typography**: Inter (system stack fallback)
**Color Palette**:
- Primary: `#1a1a2e` (SGA dark navy)
- Accent: `#4f8ef7` (active states, links, CTAs)
- Success green: `#22c55e` (on-track status)
- Warning yellow: `#f59e0b` (behind status)
- Error red: `#ef4444` (no activity status)
- Background: `#f8fafc`
- Surface: `#ffffff`
- Border: `#e2e8f0`

**Spacing**: 4px base grid (Tailwind defaults)
**Border radius**: `rounded-lg` (8px) for cards; `rounded-md` (6px) for buttons and inputs

---

### Dashboard Home (`/`)

**Purpose**: Morning glance — is the content pipeline healthy?

**Layout**: Full-width header → Metrics bar (5 cards) → Two-column body (practice grid left, activity feed right)

**Header** (56px, sticky):
```
[SGA Logo] Content Concierge Admin    [User avatar] [Sign out]
```

**Metrics Bar** (5 equal-width cards, horizontal row):

```
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ Captures Today   │ │ Active Practices  │ │ Pending Review   │ │ Sent Today       │ │ Weekly Rate      │
│       47         │ │       14          │ │       23         │ │       38          │ │     82%          │
│ ↑ 12 vs. avg     │ │  of 15 enrolled  │ │ in Airtable      │ │ assignments       │ │ of target        │
│ [green dot]      │ │ [neutral dot]     │ │ [yellow dot]     │ │ [green dot]       │ │ [green dot]      │
└──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘ └──────────────────┘
```

Color logic for status dots:
- Captures Today: green ≥2 per active practice, yellow ≥1, red <1
- Pending Review: green <20, yellow 20–50, red >50
- Weekly Rate: green ≥80%, yellow 50–80%, red <50%

**Practice Grid** (left column, ~65% width):

Grid of practice cards, 3 columns on 1440px, 2 columns on 1280px.

Each practice card (120px height):
```
┌─────────────────────────────────────┐
│ [●] Innovative Dental Springfield   │
│     Jessica Martinez · Champion     │
│                                     │
│ ████████████░░ 4 / 5 this week      │
│ Last capture: 2 hours ago           │
└─────────────────────────────────────┘
```

- Left dot color: green (on track), yellow (behind by 1–2), red (0 captures today + behind)
- Progress bar: filled portion = captures/target ratio, capped at 100%
- Card hover: subtle shadow elevation
- Card click: navigate to `/practices/[id]`

**Activity Feed** (right column, ~35% width):

Scrollable list, most recent first, grouped by "Today" / "Yesterday":
```
TODAY
  ✅ 2 photos — Innovative Dental Springfield (Jessica) — 2 min ago
  📅 Daily briefs sent — 14 champions — 7:30 AM
  ✅ 1 video — 4Front Implants Tulsa (Dr. Chen) — 1 hr ago

YESTERDAY
  ⚠️  Sync failed — Riverside Family Dental — 11:23 PM
  ✅ 4 photos — Summit Ortho Denver (Maria) — 3:45 PM
```

Each item is clickable → relevant Practice Detail or Captures Monitor filtered view.

---

### Practices List (`/practices`)

**Layout**: Full-width table with sticky header

**Search + Filter bar** (above table):
```
[🔍 Search practices...]  [Playbook ▾]  [Health ▾]  [Status ▾]  [+ Add Practice]
```

**Table columns**:

| Column | Width | Notes |
|--------|-------|-------|
| Practice | 22% | Bold name + city, state below in gray |
| Playbook | 12% | Colored badge: Implant/Cosmetic (purple), Specialty (blue), Growth GP (green), Standard GP (gray) |
| Champion | 14% | Name |
| Captures | 14% | `4 / 5` with mini bar |
| Last Capture | 12% | Relative time |
| Sync | 10% | ✓ green / ⚠ yellow / ✗ red |
| Actions | 8% | Kebab menu: Edit, Deactivate |

**Row health color**: Left border stripe — green, yellow, or red — matching health status.

**Empty state**: Centered illustration + "No practices yet. Ready to add your first?" + `[Add Practice]` button.

---

### Practice Detail (`/practices/[id]`)

**Layout**: Breadcrumb → Title bar → Tab nav → Tab content

**Title bar**:
```
← Practices
Innovative Dental Springfield
Springfield, IL · Implant/Cosmetic · [Active badge]
```

**Tab navigation** (5 tabs):
`Overview` | `Champions` | `Captures` | `Assignments` | `Config`

**Overview Tab**

Two-column layout:

Left (60%):
- Stats row: Captures This Week (`4 / 5`), Practice Streak (`12 days`), Last Sync (`2 hours ago`), Airtable Pending (`3`)
- Capture history chart: Bar chart, 14 days, each bar = daily captures. Target line overlaid as dashed horizontal line. Tooltip on hover shows count + date.

Right (40%):
- Quick info card: PMS system, Sikka ID, Timezone, Created date
- Recent 3 captures as thumbnail row (photos) or file icon (video) with timestamp

**Champions Tab**

Table:
```
Name         Role        Phone           SMS  WhatsApp  Captures/Week  Last Active
Jessica M.   Champion    +1 417-555-...  ✓    —         4              Today
Dr. Patel    Doctor      +1 417-555-...  ✓    ✓         1              Yesterday
```

`[+ Add Champion]` button opens a slide-over (not a new page):
```
Slide-over panel (360px wide):
  Add Champion
  ─────────────────────────────
  Name *
  [text input]

  Role *
  [dropdown: Champion | Doctor | Leader]

  Phone *
  [phone input with +1 prefix, format: (XXX) XXX-XXXX]

  Notify via *
  [checkbox: SMS] [checkbox: WhatsApp]

  [☐] Send welcome SMS now

  [Cancel] [Add Champion]
```

**Captures Tab**

Filter bar: Date range | Media type | Airtable status

Table with thumbnail preview column (40×40px for photos, video icon for video):
```
Date            Champion     Type            Preview  Status
Today 2:14 PM   Jessica M.   Before Photo    [img]    Needs Review  [Open in Airtable ↗]
Today 2:13 PM   Jessica M.   After Photo     [img]    Needs Review  [Open in Airtable ↗]
Yesterday       Dr. Patel    Topic Video     [▶]      In Production [Open in Airtable ↗]
```

**Assignments Tab**

Table with status badges:
- `Pending` (gray) — not yet captured
- `Captured` (green) — media received
- `Expired` (yellow) — past due, not captured
- `Skipped` (red) — champion sent SKIP

Skipped and expired rows: muted gray row.

**Config Tab**

Editable form (inline edit, not a modal):
```
Playbook Type        [Implant/Cosmetic ▾]
Weekly Target        [5 ____] captures/week
PMS System           [Open Dental ▾]
Sikka Practice ID    [practice_id_here      ] [Test Connection]
Timezone             [America/Chicago ▾]
Active               [● ON]

                             [Save Changes]
```

"Test Connection" triggers API call → shows inline result: "✓ Connected — 12 appointments tomorrow" or "✗ Failed: invalid practice ID"

---

### Add Practice Wizard (`/practices/new`)

**Layout**: Centered card (640px max-width), step progress indicator at top

**Progress indicator** (3 steps):
```
━━━━━━●━━━━━━━━━━━━━━━━━━━━━
  1. Practice Info    2. Playbook    3. Champion
```

**Step 1 — Practice Info**

```
Practice Name *
[Springfield Dental Partners                    ]

City, State *
[Springfield, IL                                ]

PMS System *
[Open Dental            ▾]

Sikka Practice ID
[practice_sid_here      ] [Test Connection]

Timezone *
[America/Chicago        ▾]

                    [Next: Playbook Assignment →]
```

**Step 2 — Playbook Assignment**

Label: "Which playbook best describes this practice?"

4 selection cards in 2×2 grid (selected card has accent border + checkmark):

```
┌───────────────────────┐  ┌───────────────────────┐
│ ✓  Implant/Cosmetic   │  │    Specialty          │
│    10–15 high-value   │  │    Ortho, perio, OS,  │
│    practices. Before/ │  │    peds. Case docs,   │
│    after, testimonial │  │    specialist auth.   │
│    video playbook.    │  │                       │
│    Target: 5/week     │  │    Target: 4/week     │
└───────────────────────┘  └───────────────────────┘
┌───────────────────────┐  ┌───────────────────────┐
│    Growth GP          │  │    Standard GP        │
│    Multi-location,    │  │    1–2 locations.     │
│    high-volume. Edu-  │  │    Community-first.   │
│    cation + team      │  │    Maintenance +      │
│    content.           │  │    relationship.      │
│    Target: 3/week     │  │    Target: 2/week     │
└───────────────────────┘  └───────────────────────┘

Weekly Capture Target   [5 ____]  (auto-set, editable)

            [← Back]  [Next: Champion Onboarding →]
```

**Step 3 — Champion Onboarding**

```
Content Champion

Name *
[Jessica Martinez                   ]

Role *
[Champion                 ▾]

Phone *
[(417) 555-1234                      ]

Notify via    [☑ SMS]  [☐ WhatsApp]

[+ Add another champion]

─────────────────────────────────────
☑  Send welcome SMS immediately

Preview:
  "Hi Jessica! 👋 This is the SGA Content Concierge.
   You're set up to capture content for Springfield Dental Partners..."

─────────────────────────────────────

            [← Back]  [Add Practice & Send SMS]
```

On submit: loading state → success screen:
```
✅ Springfield Dental Partners is live!

Welcome SMS sent to Jessica Martinez.
First daily brief: tomorrow at 7:30 AM.

[View Practice] [Add Another Practice]
```

---

## SMS Conversation Design (Phase 1 Champion UX)

### Visual Design Constraints

SMS has no visual formatting. Design constraints:
- **Maximum 320 characters per message segment** (avoid multi-part SMS fees)
- **Use emoji sparingly** — 1–2 per message max; only for visual scanning, not decoration
- **No links in capture confirmations** — links in SMS are flagged as spam by carriers
- **Patient privacy**: never use full patient names in SMS; initials only if referenced
- **PHI-safe**: procedure type should be generic ("dental procedure") in SMS body when in doubt

### Tone Guidelines

| Context | Tone | Example |
|---------|------|---------|
| Daily brief | Friendly + specific | "Good morning Jessica! 3 captures today..." |
| Confirmation | Warm + brief | "Got it! ✅ 2 photos received." |
| Streak mention | Celebratory | "12-day streak 🔥" |
| Rejection feedback | Warm + constructive | "Quick note on your last photo: better lighting helps — try near a window!" |
| Weekly recap | Encouraging + data-forward | "Last week: 4 captures (target: 5). Published 12 content pieces." |
| Error/missing | Neutral + actionable | "Something went wrong with your last message. Could you try resending?" |

### Streak Celebration Tiers (reinforcement design)

| Streak | Response Modifier |
|--------|------------------|
| 3 days | "3-day streak 🔥" (first mention) |
| 7 days | "7-day streak! 🏆 You're building a habit." |
| 14 days | "14 days straight 🔥🔥 Top 10% of the network!" |
| 30 days | "30-day streak!! 🏆🏆🏆 SGA is noticing." |
| Streak breaks (after 7+) | "Your streak just reset, but 8 captures this month is real progress. Back at it today?" |

---

## Phase 2 Mobile App — Design Direction

*Full spec developed at Phase 2 kickoff. This section captures design intent and constraints.*

### App Architecture (React Native)

**Navigation structure**:
```
Tab Bar (bottom):
  [📸 Today]  [📁 My Content]  [📊 Scorecard]  [👤 Profile]
```

**Today Tab** (Home):
- Top: Greeting + today's date
- Assignment cards: 1–3 cards (max), each showing content type, procedure hint, and guidance
- Capture button: large, primary CTA — opens camera immediately
- Streak indicator: top-right corner badge

**Capture Mode**:
- Full-screen camera view
- Overlay: framing guide (rule-of-thirds grid, smile area box for before/after)
- Bottom: shutter button (photo) + hold-to-record (video, 60-sec max)
- Flip camera button
- Flash toggle
- After capture: preview → "Looks good!" button or retake

**My Content Tab**:
- Grid of champion's published content pieces
- Each cell: thumbnail + platform icon overlay
- Tap: full-size view + "Published to Instagram/Facebook/etc." detail

**Scorecard Tab**:
- This week: capture count + progress bar to target
- Streak: number + calendar heatmap (GitHub-style, 12 weeks)
- Network ranking: "Top 15% of champions this week"
- Published count: lifetime
- Top performer: highest-engagement piece

### Key UX Decisions for Phase 2 App

**Camera-first, not menu-first**: The app opens to Today's Assignments. One tap → camera open. No login screen during active session (biometric auth, stay logged in).

**Guided overlays**: For before/after photos, overlay shows a soft rectangle on the smile area labeled "Fill frame with smile." This eliminates the most common capture quality failure.

**Assignment matching**: When champion submits a photo, the app auto-matches it to the day's assignment. For ambiguous cases (2+ assignments), prompt: "Which is this for?" with 2 tappable options. Never force the champion to navigate a list.

**Offline capture**: If network unavailable, capture is queued locally and uploaded when connection restored. Champion sees "3 captures queued" indicator.

---

## Airtable Review Queue Design (Phase 1 Editors)

*Not a coded UI — Airtable configuration design.*

### View Configuration

**Primary View: Review Kanban**

Columns (Airtable Kanban groups by Status):
- `Needs Review` — new captures from champions
- `In Production` — editor is working on caption/formatting
- `Approved` — ready to distribute
- `Rejected` — champion notified

Card fields visible in Kanban:
- Practice name
- Content type badge
- Champion name
- Media thumbnail (linked URL)
- Captured at (relative time)

**Secondary View: Today's Queue** (Grid view)

Filtered by: Captured At = Today
Sorted by: Practice, then Captured At
For editors starting their daily review session.

**Tertiary View: By Practice**

Grouped by Practice, filtered by Status ≠ Rejected.
For practice-focused review sessions (reviewing one practice at a time).

### Editor Workflow

1. Open Airtable → switch to "Today's Queue" view
2. Click first record → expand
3. View media (click URL → opens S3 presigned link in browser)
4. If good: change Status → "In Production", write caption in Editor Notes
5. When caption done: Status → "Approved" (triggers Slack notification to `#content-ready`)
6. If bad: Status → "Rejected", write feedback in Editor Notes (triggers SMS to champion)
7. When approved, editor (or future automation) posts to Sprout Social / Later for scheduled distribution

### Automation Configuration

**Automation 1: Approved → Slack**
- Trigger: Status field changed to "Approved"
- Action: Send Slack message to `#content-ready`: "✅ Approved: {Content Type} from {Practice} · {Airtable Record Link}"

**Automation 2: Rejected → SMS via webhook**
- Trigger: Status field changed to "Rejected"
- Action: HTTP POST to Capture Service webhook endpoint
  ```json
  {
    "capture_id": "{Capture ID field}",
    "editor_feedback": "{Editor Notes field}"
  }
  ```
- Capture Service sends SMS to champion: "Quick note on your last {content_type}: {editor_feedback}"
