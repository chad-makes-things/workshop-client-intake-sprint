# UX Design Specification — The Practice Concierge

> AI Concierge Platform for SGA Dental Partners (260+ practices)
> Digital Storefront at Scale — Sprint Development Package

---

## Screen Inventory

### 1. Practice Concierge (Chat Interface)

| Screen | Description |
|--------|-------------|
| Welcome / Onboarding | First-contact greeting with practice context already loaded |
| Data Verification | Harvested practice data presented for confirm/correct |
| Template Selection | 2-3 recommended website templates with previews |
| Content Review | AI-generated copy presented section by section for approval |
| Photo / Asset Upload | Guided upload flow for team photos, office images |
| Final Preview | Full rendered site preview with device switcher |
| Go-Live Confirmation | Summary checklist and single-tap publish approval |
| Post-Launch Check-in | 48-hour follow-up with performance snapshot |
| Ongoing Support | Persistent chat for edits, questions, requests |

### 2. HQ Admin Dashboard

| Screen | Description |
|--------|-------------|
| Network Overview | High-level enrollment progress across all 260+ practices |
| Enrollment Pipeline | Kanban-style view of practices by stage |
| QC Review Queue | Sites pending quality review with inline preview |
| Practice List | Searchable, filterable table of all practices |
| Practice Detail | Deep dive into a single practice's enrollment and site status |
| Batch Operations | Multi-select interface for bulk actions |
| Template Manager | Template library with usage stats and versioning |
| Content Standards | Brand guidelines and content rules configuration |
| Alert Center | Flagged issues, stale enrollments, QC failures |

### 3. Executive Dashboard

| Screen | Description |
|--------|-------------|
| Network Health | Single-screen KPI overview with health score |
| Tier Breakdown | Practices segmented by engagement tier |
| Enrollment Velocity | Time-series enrollment progress with projections |
| Performance Leaderboard | Top/bottom performing practice sites |
| Alert Summary | Critical issues requiring executive attention |
| ROI Report | Cost-per-site, time savings, conversion metrics |

---

## Key Screen Wireframe Descriptions

### Chat Enrollment Flow

**Welcome Screen**

The chat opens with a branded header showing the SGA Dental Partners logo and the concierge's name. The first message is a warm, context-aware greeting:

```
"Hi Dr. Martinez — I'm your Practice Concierge. I've already
pulled together what I know about Bright Smile Dental in
Scottsdale. Let's get your new website up in about 15 minutes.
Ready to start?"
```

Two action buttons sit below: **"Let's Go"** and **"Tell Me More"**. The background is dark (#0a0a0f) with the message bubble in surface color (#12121a) and a subtle blue left-border accent on the concierge's messages.

**Data Verification Screen**

The concierge presents harvested data in structured action cards — not plain text. Each card covers one data category:

- **Practice Info** — Name, address, phone, hours
- **Provider Details** — Doctor names, specialties, credentials
- **Services Offered** — Detected services with confidence indicators (high/medium/low shown as filled dots)

Each card has three response options: **"Looks Good"** (green), **"Small Fix"** (amber, opens inline edit), **"Wrong"** (red, opens replacement field). A progress bar at the top shows 5 steps, with Step 2 highlighted.

**Template Selection Screen**

Three template preview cards arranged vertically (mobile) or in a row (tablet+). Each card contains:

- A 16:9 thumbnail preview of the template rendered with the practice's actual data
- Template name and one-line description
- A tag row showing template traits (e.g., "Modern", "Photo-Forward", "Family-Friendly")
- A **"Recommended"** badge (accent blue with star icon) on the AI-selected best fit
- A reasoning line in muted text: *"Selected because your practice emphasizes cosmetic dentistry and has strong photography"*

A **"Why these three?"** expandable section at the bottom explains the matching logic. The primary CTA is **"Choose This One"** on each card.

**Approval Screen**

A section-by-section content walkthrough. Each section (Hero, About, Services, Team, Contact) is presented as a collapsible card with:

- The generated headline and body copy
- An inline preview showing how it renders on the template
- **"Approve"** / **"Edit"** / **"Regenerate"** action buttons

Approved sections get a green checkmark. The concierge tracks progress: *"3 of 5 sections approved — almost there."*

**Go-Live Screen**

A full-page preview with a device switcher toolbar (phone / tablet / desktop icons). Below the preview, a checklist summarizes:

- All data verified
- Template selected
- Content approved
- Contact info confirmed
- SSL certificate ready

A large green **"Publish My Site"** button anchored at the bottom. Tapping it triggers a brief celebration animation and a confirmation message with the live URL.

---

### Template Picker (Detail View)

Each template preview card is structured as follows:

```
+-------------------------------------+
|  +-------------------------------+  |
|  |                               |  |
|  |    [Template Preview Image]   |  |
|  |    Rendered with practice     |  |
|  |    data, not lorem ipsum      |  |
|  |                               |  |
|  +-------------------------------+  |
|                                     |
|  * Recommended                      |
|  Modern Cosmetic                    |
|  Clean lines, prominent before/     |
|  after gallery, bold CTAs           |
|                                     |
|  Modern - Photo-Forward - Premium   |
|                                     |
|  "Best match for your cosmetic      |
|   focus and high-quality photos"    |
|                                     |
|  [ Choose This Template ]           |
|  [ Preview Full Site ]              |
+-------------------------------------+
```

The recommended card has a subtle blue glow border (`box-shadow: 0 0 20px rgba(79, 143, 255, 0.15)`). Non-recommended cards have the standard surface border. The recommendation badge and reasoning are only present on the top-ranked card.

---

### HQ Dashboard

**Layout:** Fixed left sidebar navigation (collapsed to icons on smaller screens), top bar with search and alerts, main content area.

**Enrollment Progress Section**

A horizontal progress bar segmented by stage, with counts:

```
Not Started (142) > In Progress (67) > QC Review (28) > Live (23) > Monitoring (4)
```

Each segment is color-coded (gray, blue, amber, green, teal). Clicking a segment filters the practice list below.

**QC Queue**

A card-based queue, each card showing:

- Practice name and location
- Thumbnail of the site
- Auto-QC score (pass/warning/fail as a colored badge)
- Flagged issues count
- **"Review"** / **"Approve"** / **"Return to Practice"** action buttons
- Time in queue indicator

**Practice List**

A dense, filterable table:

| Practice | Location | Stage | Template | QC Score | Last Activity | Actions |
|----------|----------|-------|----------|----------|---------------|---------|

Filters across the top: Stage, Region, Template, QC Status. Bulk select checkboxes on the left. Batch action bar appears when items are selected.

**Batch Operations Panel**

When practices are selected, a sticky bottom bar appears:

```
+------------------------------------------------------+
|  12 practices selected                               |
|  [ Send Reminder ] [ Assign Template ] [ Export ]    |
|  [ Clear Selection ]                                 |
+------------------------------------------------------+
```

Each batch action opens a confirmation modal with a preview of affected practices.

---

### Executive Dashboard

**Network Health Score**

A large circular gauge (0-100) centered at the top, color-shifting from red through amber to green. The score synthesizes: enrollment velocity, site quality, uptime, and engagement. Surrounding the gauge, four satellite metrics:

- Enrollment Rate (% of 264 practices live)
- Avg QC Score
- Avg Time to Launch
- Practice Satisfaction Score

**Tier Breakdown**

A horizontal stacked bar or donut chart showing practices by tier:

- **Tier 1 — Flagship** (custom design, priority support)
- **Tier 2 — Standard** (template with customization)
- **Tier 3 — Essential** (template, minimal customization)
- **Not Started** (gray)

Each tier is clickable, drilling into a filtered practice list.

**Alert Cards**

A vertical stack of alert cards, severity-coded:

- **Critical** (red left border): "12 practices stalled in enrollment for 7+ days"
- **Warning** (amber left border): "QC queue backlog exceeding 48-hour SLA"
- **Info** (blue left border): "New template version available for review"

Each card has a **"View Details"** action and a **"Dismiss"** option.

**KPI Row**

Four KPI cards in a horizontal row:

| Metric | Value | Trend |
|--------|-------|-------|
| Sites Live | 23 / 264 | +8 this week |
| Avg Launch Time | 4.2 days | -1.1 days vs last month |
| Cost per Site | $127 | -34% vs manual |
| Enrollment NPS | 72 | +12 vs baseline |

Each card shows the value prominently, a sparkline trend, and a comparison to baseline or target.

---

### Practice Detail Drill-Down

**Header:** Practice name, address, tier badge, current stage pill, assigned concierge.

**Tabs:**

1. **Enrollment Timeline** — Vertical timeline showing every step completed, pending, or skipped. Each node shows timestamp, action taken, and who acted (AI or human).
2. **Site Preview** — Embedded iframe or screenshot of current site state with device switcher.
3. **Data & Content** — All harvested and approved data in editable card format.
4. **QC Results** — Auto-QC checklist with pass/fail per criterion (brand compliance, mobile rendering, load time, accessibility, content accuracy).
5. **Activity Log** — Full chat transcript and admin actions.

---

## Interaction Patterns

### Chat Message Flow

**Message Types:**

| Type | Appearance | Behavior |
|------|------------|----------|
| Concierge text | Dark surface bubble (#12121a), left-aligned, blue left accent border | Standard chat message |
| User text | Accent blue bubble (#4f8fff at 20% opacity), right-aligned | Standard chat reply |
| Action card | Full-width card with structured content and buttons | Replaces free-text input with guided options |
| Inline preview | Embedded thumbnail with "expand" affordance | Tap to view full-screen preview |
| Progress indicator | Thin horizontal bar below header | Updates as enrollment advances |
| System message | Centered, muted text, no bubble | Timestamps, stage transitions |

**Typing Indicator:** Three pulsing dots in a concierge-style bubble. Appears during AI processing. For longer operations (site generation), replaced with a progress message: *"Building your preview... this takes about 30 seconds."*

**Message Sequencing:** The concierge never sends more than two messages before waiting for user input. Action cards always appear as the final message in a sequence, ensuring the user's next step is always clear.

### Approval Workflows

**Single-Item Approval (Chat):**

Each approvable item presents three options as tappable buttons:

- **Approve** (green, checkmark icon) — confirms and advances
- **Edit** (amber, pencil icon) — opens inline editing
- **Flag** (red, flag icon) — marks for HQ review with optional note

Tapping any option provides immediate visual feedback (button fills with color, brief haptic on mobile) and the concierge acknowledges the choice before moving on.

**QC Approval (HQ Dashboard):**

Reviewer sees the site preview alongside the QC checklist. Each checklist item can be toggled pass/fail. At the bottom:

- **Approve** — publishes the site
- **Approve with Notes** — publishes with follow-up items logged
- **Return** — sends back to practice concierge with specific feedback

### Batch Operations

1. **Select** — Checkbox per row, or "Select All" (with filter awareness: "Select all 28 in QC Review")
2. **Action Bar** — Sticky bottom bar shows count and available actions
3. **Preview** — Before execution, a modal lists all affected practices with the pending action
4. **Confirm** — Single confirmation button with count: "Send Reminder to 28 Practices"
5. **Progress** — After confirmation, a progress indicator shows completion: "Sent 24 of 28..."
6. **Result** — Summary of successes and failures with retry option for failures

### Template Preview

**Device Switcher:** Three icons (phone, tablet, desktop) in a toolbar above the preview. Active device is highlighted in accent blue. The preview container resizes with a smooth transition animation.

**Interaction within Preview:**

- Scroll to navigate the full page
- Tap sections to highlight corresponding content card in the approval flow
- Pinch-to-zoom on mobile
- "Open in New Tab" link for full-browser preview

---

## Design System

### Colors

**Base Palette:**

| Token | Hex | Usage |
|-------|-----|-------|
| `--bg-primary` | `#0a0a0f` | Page background |
| `--bg-surface` | `#12121a` | Cards, panels, chat bubbles |
| `--bg-surface-hover` | `#1a1a2e` | Interactive surface hover state |
| `--bg-elevated` | `#1e1e30` | Modals, popovers, dropdown menus |
| `--border-default` | `#2a2a3d` | Card borders, dividers |
| `--border-focus` | `#4f8fff` | Focus rings, active borders |

**Accent Colors:**

| Token | Hex | Usage |
|-------|-----|-------|
| `--accent-blue` | `#4f8fff` | Primary actions, links, active states |
| `--accent-blue-muted` | `rgba(79, 143, 255, 0.15)` | Blue backgrounds, selected states |
| `--accent-green` | `#34d399` | Success, approval, live status |
| `--accent-green-muted` | `rgba(52, 211, 153, 0.15)` | Success backgrounds |
| `--accent-amber` | `#fbbf24` | Warnings, in-progress, edits |
| `--accent-amber-muted` | `rgba(251, 191, 36, 0.15)` | Warning backgrounds |
| `--accent-red` | `#f87171` | Errors, critical alerts, flags |
| `--accent-red-muted` | `rgba(248, 113, 113, 0.15)` | Error backgrounds |
| `--accent-teal` | `#2dd4bf` | Monitoring, secondary positive |

**Text Colors:**

| Token | Hex | Usage |
|-------|-----|-------|
| `--text-primary` | `#f0f0f5` | Headings, primary content |
| `--text-secondary` | `#a0a0b8` | Body text, descriptions |
| `--text-muted` | `#6b6b80` | Timestamps, helper text, placeholders |
| `--text-inverse` | `#0a0a0f` | Text on light/colored backgrounds |

### Typography

**Font Stack:**

```css
--font-sans: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
             Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans',
             'Helvetica Neue', sans-serif;

--font-mono: 'SF Mono', 'Fira Code', 'Fira Mono', 'Roboto Mono',
             Consolas, 'Courier New', monospace;
```

**Type Scale:**

| Token | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| `--text-xs` | 11px | 400 | 1.4 | Badges, fine print |
| `--text-sm` | 13px | 400 | 1.5 | Helper text, timestamps |
| `--text-base` | 15px | 400 | 1.6 | Body text, chat messages |
| `--text-md` | 17px | 500 | 1.5 | Card titles, labels |
| `--text-lg` | 20px | 600 | 1.4 | Section headings |
| `--text-xl` | 24px | 700 | 1.3 | Page titles |
| `--text-2xl` | 32px | 700 | 1.2 | Dashboard hero metrics |
| `--text-3xl` | 48px | 800 | 1.1 | Health score number |

### Spacing

**Base Unit:** 4px

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Inline spacing, icon gaps |
| `--space-2` | 8px | Tight padding, small gaps |
| `--space-3` | 12px | Default padding inside components |
| `--space-4` | 16px | Standard content padding |
| `--space-5` | 20px | Card padding |
| `--space-6` | 24px | Section gaps |
| `--space-8` | 32px | Section padding |
| `--space-10` | 40px | Large section gaps |
| `--space-12` | 48px | Page section separation |
| `--space-16` | 64px | Major layout gaps |

**Border Radius:**

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Badges, pills, small elements |
| `--radius-md` | 8px | Buttons, inputs |
| `--radius-lg` | 12px | Cards, panels |
| `--radius-xl` | 16px | Chat bubbles, modals |
| `--radius-full` | 9999px | Avatars, circular indicators |

### Component Library

**Chat Bubble**

```
Properties: variant (concierge | user | system), content, timestamp
Concierge: bg --bg-surface, left-aligned, 2px left border --accent-blue, radius-xl
User: bg --accent-blue-muted, right-aligned, radius-xl
System: no background, centered, --text-muted, --text-sm
Max width: 85% of container
Padding: --space-4 horizontal, --space-3 vertical
```

**Action Card**

```
Properties: title, description, actions[], variant (default | highlight)
Background: --bg-surface
Border: 1px solid --border-default (highlight variant: --accent-blue)
Radius: --radius-lg
Padding: --space-5
Actions: row of buttons, right-aligned, --space-2 gap
Shadow: highlight variant gets subtle blue glow
```

**Health Score Badge**

```
Properties: score (0-100), size (sm | md | lg)
Shape: circular gauge with score centered
Colors: 0-39 --accent-red, 40-69 --accent-amber, 70-100 --accent-green
Background: --bg-surface with colored ring (stroke-width scales with size)
Font: --text-2xl (lg), --text-lg (md), --text-base (sm), weight 700
```

**Alert Card**

```
Properties: severity (critical | warning | info), title, description, actions[]
Background: --bg-surface
Left border: 3px solid (critical: --accent-red, warning: --accent-amber, info: --accent-blue)
Radius: --radius-lg
Padding: --space-4
Icon: severity-specific icon in matching color, top-left
```

**Practice Card**

```
Properties: name, location, stage, tier, qcScore, lastActivity
Background: --bg-surface
Border: 1px solid --border-default
Radius: --radius-lg
Layout: name and tier badge on first line, location second line,
        stage pill and QC score on third line, last activity muted at bottom
Hover: bg --bg-surface-hover, border --border-focus
Padding: --space-5
```

**Template Preview Card**

```
Properties: name, description, tags[], thumbnail, isRecommended, reasoning
Background: --bg-surface
Border: 1px solid (recommended: --accent-blue, default: --border-default)
Shadow: recommended: 0 0 24px rgba(79, 143, 255, 0.12)
Radius: --radius-lg
Thumbnail: 16:9 ratio, radius-md on top corners, overflow hidden
Badge: "Recommended" positioned top-right of thumbnail, bg --accent-blue, --text-xs
Tags: row of pills below description
Padding: --space-5
```

**Status Pill**

```
Properties: label, variant (not-started | in-progress | review | live | monitoring)
Shape: pill (radius-full)
Height: 24px
Padding: --space-1 vertical, --space-3 horizontal
Font: --text-xs, weight 600, uppercase
Colors by variant:
  not-started: bg transparent, border --border-default, text --text-muted
  in-progress: bg --accent-blue-muted, text --accent-blue
  review: bg --accent-amber-muted, text --accent-amber
  live: bg --accent-green-muted, text --accent-green
  monitoring: bg rgba(45, 212, 191, 0.15), text --accent-teal
```

**Confidence Indicator**

```
Properties: level (high | medium | low)
Shape: three dots in a row, 6px diameter, --space-1 gap
High: all three filled --accent-green
Medium: two filled --accent-amber, one --border-default
Low: one filled --accent-red, two --border-default
Tooltip: "Data confidence: {level} -- sourced from {source}"
```

---

## Brand Personality in UI

### Sage + Caregiver Archetype

The Practice Concierge embodies a **Sage** (knowledgeable, analytical, trustworthy) paired with a **Caregiver** (supportive, reassuring, proactive) personality. This manifests in every interaction:

**Voice Principles:**

- **Knowledgeable but not condescending** — "Based on practices similar to yours in the Phoenix metro area, a photo-forward template drives 23% more appointment requests."
- **Warm but professional** — "Great choice, Dr. Martinez. I'll have your preview ready in about 30 seconds."
- **Confident but transparent** — "I'm 92% confident this is your correct address based on your Google Business listing. Can you confirm?"

### Opinionated Recommendations with Reasoning

Every recommendation the concierge makes is accompanied by visible reasoning. This is a core design principle, not an optional feature.

**Pattern:**

```
[Recommendation]
"I recommend the Modern Cosmetic template."

[Reasoning -- always visible, not hidden behind a toggle]
"Your practice emphasizes cosmetic dentistry (4 of 6 highlighted
services), and you have professional photography that this template
showcases prominently. Practices with similar profiles see 31% higher
engagement with this layout."
```

The reasoning text uses `--text-secondary` color and `--text-sm` size, sitting directly below the recommendation. Never require a tap to reveal reasoning.

### Proactive Suggestions over Passive Menus

The concierge does not present neutral menus. It leads with a recommendation and provides alternatives as secondary options.

**Do this:**
> "I'd suggest keeping your Monday-Thursday 8am-5pm hours on the site. Want to add your Friday 8am-1pm half-day too?"
> [ Add Friday Hours ] [ Keep Mon-Thu Only ] [ Edit All Hours ]

**Not this:**
> "What hours would you like to display?"
> [ Enter hours manually ]

### Confidence Indicators on Harvested Data

All pre-populated data shows its confidence level and source. This builds trust and saves correction time.

- **High confidence** (three green dots): Verified from multiple sources (e.g., Google Business Profile + state licensing board)
- **Medium confidence** (two amber dots): Single source or partially matched (e.g., only Google listing)
- **Low confidence** (one red dot): Inferred or outdated (e.g., data older than 12 months)

Users can tap any confidence indicator to see the source: *"From Google Business Profile, last updated January 2026."*

---

## Mobile-First Requirements

### Design for the 5-Minute Window

Dental practice staff interact with the concierge in short bursts between patients. Every design decision serves this constraint.

**Core Principles:**

1. **Resumability** — The chat always opens to exactly where the user left off, with a brief recap message: *"Welcome back! You approved 3 of 5 sections last time. Ready to review Services?"*
2. **Minimal input** — Every interaction defaults to tap-based responses. Free-text input is never required (always optional alongside structured options).
3. **Progress persistence** — Every approval and edit is saved immediately. No "submit form" patterns. No lost work.
4. **Time estimates** — Each step shows estimated time: *"This section usually takes about 2 minutes."*

### Touch Targets

- **Minimum touch target:** 44px x 44px (per Apple HIG and WCAG 2.5.5)
- **Recommended touch target:** 48px x 48px for primary actions
- **Spacing between targets:** minimum 8px to prevent mis-taps
- **Action buttons in chat:** full-width on mobile, minimum height 48px

### Input Positioning

- **Chat input bar:** fixed to bottom of viewport, above the device safe area
- **Quick-action buttons:** positioned above the input bar when contextually relevant
- **Action card buttons:** bottom of card, full-width stacked on mobile (side-by-side on tablet+)
- **Navigation:** bottom tab bar on mobile (Chat, Progress, Help), sidebar on tablet+

### Performance Targets

- **First meaningful paint:** < 1.5 seconds on 4G
- **Chat message response rendering:** < 200ms after receipt
- **Template preview load:** < 3 seconds (lazy-load with skeleton placeholder)
- **Offline resilience:** chat history cached locally, queued actions sync when reconnected

---

## Accessibility

### WCAG 2.1 AA Compliance

This platform serves a diverse user base across 260+ practices. Full AA compliance is a baseline requirement, not a stretch goal.

### Color Contrast

All text and interactive elements meet minimum contrast ratios against their backgrounds:

| Element | Foreground | Background | Ratio | Requirement |
|---------|-----------|------------|-------|-------------|
| Primary text | #f0f0f5 | #0a0a0f | 18.4:1 | 4.5:1 (AA) |
| Secondary text | #a0a0b8 | #0a0a0f | 8.2:1 | 4.5:1 (AA) |
| Muted text | #6b6b80 | #0a0a0f | 4.6:1 | 4.5:1 (AA) |
| Accent blue on surface | #4f8fff | #12121a | 5.1:1 | 4.5:1 (AA) |
| Green on surface | #34d399 | #12121a | 8.7:1 | 4.5:1 (AA) |
| Amber on surface | #fbbf24 | #12121a | 9.8:1 | 4.5:1 (AA) |
| Red on surface | #f87171 | #12121a | 5.8:1 | 4.5:1 (AA) |

**Non-color indicators:** All status information conveyed by color also uses a secondary indicator (icon, label, or pattern). Status pills include text labels. Confidence indicators include tooltips. Alert cards include severity icons.

### Focus Management in Chat Flow

- **Focus follows content:** When the concierge sends a new message with action buttons, focus moves to the first actionable element.
- **Skip link:** "Skip to latest message" link at the top of the chat for keyboard users.
- **Focus trapping:** Modals and overlays trap focus until dismissed.
- **Focus ring:** 2px solid `--border-focus` with 2px offset, visible on all interactive elements during keyboard navigation. Hidden during pointer interaction.
- **Chat input focus:** Pressing Enter in the input sends the message. Tab moves to action buttons. Shift+Tab returns to input.

### Screen Reader Support

- **Chat messages:** Each message has `role="log"` on the container and `aria-live="polite"` for new messages. Concierge messages are announced as "Concierge says: {message}". User messages are announced as "You said: {message}".
- **Action cards:** Structured with `role="group"` and `aria-label` describing the card purpose. Buttons within have descriptive labels: "Approve practice address" not just "Approve".
- **Status changes:** Stage transitions announced via `aria-live="assertive"`: "Enrollment stage updated to Content Review."
- **Progress indicators:** `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`, and `aria-valuetext` ("3 of 5 sections approved").
- **Charts and gauges:** Health score gauge includes `aria-label="Network health score: 74 out of 100, status good"`. Complex charts provide a data table alternative accessible via a toggle.

### Keyboard Navigation

| Context | Key | Action |
|---------|-----|--------|
| Chat | Enter | Send message |
| Chat | Tab | Move to next action button |
| Chat | Escape | Close any expanded preview |
| Action card | Enter/Space | Activate focused button |
| Dashboard table | Arrow keys | Navigate between rows |
| Dashboard table | Enter | Open practice detail |
| Modal | Escape | Close modal |
| Template preview | 1/2/3 | Switch device view (phone/tablet/desktop) |

### Motion and Animation

- **Respect `prefers-reduced-motion`:** All transitions and animations are disabled when the user's OS-level setting requests reduced motion.
- **No auto-playing animations:** The go-live celebration animation only plays on explicit user action and respects the reduced motion preference.
- **Transition durations:** Default 200ms for micro-interactions, 300ms for layout shifts. Reduced to 0ms when reduced motion is active.

---

*This design specification is the source of truth for all UI implementation. Deviations require review and approval through the QC workflow.*
