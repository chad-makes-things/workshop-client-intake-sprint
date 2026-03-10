# Design Specifications: The Practice Concierge

**Document**: UX Design Spec
**Sprint**: Digital Storefront at Scale
**Created**: 2026-03-10
**Status**: Draft

---

## Design Philosophy

The Practice Concierge serves three distinct audiences through three interfaces, unified by a single design system. The core principle: **reduce cognitive load for people who have 5 minutes between patients.** Every screen must answer "what do I do right now?" within 3 seconds of loading.

The brand personality — Sage (knowledgeable, evidence-based) + Caregiver (warm, protective) — manifests in the UI through opinionated recommendations with visible reasoning, progressive disclosure that never overwhelms, and a tone that feels like a trusted colleague rather than enterprise software.

---

## Screen Inventory

### Interface 1: Practice Chat (Concierge)

The primary practice-facing interface. A conversational UI that handles enrollment, verification, approvals, change reporting, and ongoing communication.

| Screen | Purpose | Phase |
|--------|---------|-------|
| PC-01: Welcome / Landing | Entry point from invitation link. Practice name, concierge intro, "Start" CTA | 1 |
| PC-02: Chat Interface | Core conversation view. Message thread + quick-reply buttons + file upload | 1 |
| PC-03: Data Verification Cards | Inline cards showing harvested data (team, services, hours) with confirm/edit actions | 1 |
| PC-04: Template Picker | Side panel or inline carousel showing 2-3 template previews with practice data populated | 1 |
| PC-05: Site Preview | Full-screen preview of generated site (mobile toggle, desktop toggle, page navigation) | 1 |
| PC-06: Approval Summary | Complete storefront package review (website + GMB + social + lead capture) | 1 |
| PC-07: Go-Live Confirmation | Deployment confirmation with links to all live assets | 1 |
| PC-08: Performance Summary | Weekly/monthly stats card (visitors, leads, reviews, health score) | 2 |
| PC-09: Change Report Flow | Guided flow for reporting staff changes, hours updates, service changes | 2 |
| PC-10: Content Approval | Inline preview of scheduled social posts with approve/reject/edit actions | 3 |

### Interface 2: HQ Admin Dashboard

The marketing team's operational hub. Practice management, QC workflows, batch operations, and content pipeline.

| Screen | Purpose | Phase |
|--------|---------|-------|
| HQ-01: Network Overview | Enrollment progress, health score distribution, alert feed, content pipeline status | 1 |
| HQ-02: Practice List | Filterable/sortable table of all 260+ practices with status, score, last activity | 1 |
| HQ-03: Practice Detail | Per-practice deep dive: profile data, health scorecard, interaction history, assets | 1 |
| HQ-04: QC Queue | Items awaiting creative review: new sites, flagged content, change requests | 2 |
| HQ-05: QC Review View | Side-by-side preview (before/after or site + checklist) with approve/reject/notes | 2 |
| HQ-06: Batch Operations | Template update interface with network-wide preview and selective deployment | 2 |
| HQ-07: Content Pipeline | Scheduled content across all practices, editorial calendar view, bulk actions | 3 |
| HQ-08: Enrollment Campaign Manager | Create/manage invitation batches, track open/click/enroll rates | 1 |
| HQ-09: Template Manager | Edit/preview templates, manage SGA-approved color palettes and components | 2 |
| HQ-10: Harvest Monitor | Batch harvest progress, failure log, data quality summary | 1 |

### Interface 3: Executive Dashboard

Leadership visibility into network-wide digital health. Read-only, data-dense, designed for Monday morning and board meetings.

| Screen | Purpose | Phase |
|--------|---------|-------|
| EX-01: Network Health Overview | Single-screen: health score gauge, tier distribution, trend sparklines, top alerts | 1 |
| EX-02: Enrollment Progress | Funnel visualization: invited > engaged > enrolled > live > optimized | 1 |
| EX-03: Health Score Breakdown | Drill into the 5 score components across the network (website, GMB, social, reviews, leads) | 2 |
| EX-04: Practice Comparison | Rank practices by health score, identify bottom 10%, compare tiers | 2 |
| EX-05: ROI Summary | Cost per practice setup, lead generation totals, patient acquisition attribution | 3 |
| EX-06: Export / Report Builder | Generate PDF/PPT summaries for leadership meetings | 2 |

---

## Wireframe Descriptions

### WF-01: Chat Enrollment Flow (PC-02 through PC-07)

The enrollment conversation is a linear flow within the chat interface. The concierge drives the conversation; the practice responds with taps, typed corrections, and file uploads.

**Layout**: Full-screen mobile-first chat. Message thread occupies 75% of the viewport. Bottom 25% is the input area: text field, quick-reply buttons, and attachment icon.

**Message Types**:
- **Agent message**: Left-aligned bubble, dark background (charcoal), white text. Avatar icon (small concierge mark) at top-left of first message in a sequence.
- **User message**: Right-aligned bubble, accent color background (SGA green), white text.
- **Data verification card**: Full-width inline card within the chat thread. Light card surface against the dark chat background. Contains structured data (e.g., team member list) with action buttons at the bottom ("Looks right" / "Edit").
- **Template preview card**: Full-width inline card with a preview thumbnail, template name, and the concierge's recommendation reasoning below. Tap to expand to full preview (PC-05).
- **Approval card**: Full-width card with checklist of all storefront components, each with a green checkmark or orange flag. Single "Approve All" button at the bottom.

**Quick Reply Buttons**: Appear above the text input when the concierge offers choices. Horizontal scrollable row. Each button is a pill shape with concise label. Tapping sends the selection as a user message.

**Flow Sequence**:
1. Welcome message + "Let's get started" button
2. Team verification card (list of names with confirm/edit per person)
3. Services verification card
4. Hours and contact verification card
5. Template recommendation with preview cards
6. Full storefront preview (expand to PC-05)
7. Approval summary card
8. Go-live confirmation with links

**Key Interaction**: At each verification step, "Looks right" advances to the next step. "Edit" expands an inline edit form within the card — the practice never leaves the chat context. Edits are saved immediately and the card updates in place.

**Progress Indicator**: A subtle step indicator at the top of the chat (dots or a thin progress bar) showing where the practice is in the enrollment flow. Not intrusive — practices should not feel like they are filling out a form.

### WF-02: Template Picker (PC-04)

**Trigger**: The concierge sends a message like "Based on your specialty, I recommend Template A. Here are your options:" followed by template preview cards.

**Layout**: Each template appears as an inline card in the chat thread. The card contains:
- Template name and specialty label (e.g., "Perio Pro — Referral Optimized")
- Preview thumbnail showing the homepage with the practice's real data (name, team photo, colors)
- Mobile and desktop toggle icons in the thumbnail corner
- The concierge's recommendation badge on the recommended template ("Recommended" pill in accent color)
- A one-line reasoning statement below the thumbnail ("Referral-optimized layouts convert 23% better for periodontists")
- "Select This Template" button

**Expanded Preview (PC-05)**: Tapping the thumbnail opens a full-screen preview overlay. The preview shows the actual generated site in an iframe-like view. Navigation tabs at the top cycle through pages (Home, About, Services, Team, Contact). A toggle switches between mobile and desktop viewport. A floating "Select This Template" button persists at the bottom. "Back to chat" closes the overlay.

**Customization**: After selecting a template, the concierge asks "Want to adjust anything?" Quick replies offer: "Change colors" / "Change hero image" / "Looks perfect." Each customization opens a picker within the chat: color swatches (SGA-approved palette only), hero image options (from the practice's harvested photos + SGA stock library).

### WF-03: HQ Network Dashboard (HQ-01)

**Layout**: Dark theme. Left sidebar navigation (collapsed to icons on mobile). Main content area with a 3-column grid on desktop, single column on mobile.

**Top Bar**: SGA logo mark, current user name, notification bell with unread count, settings gear.

**Sidebar Navigation**:
- Dashboard (home icon) — HQ-01
- Practices (building icon) — HQ-02
- QC Queue (checkmark icon) — HQ-04, with badge count for pending items
- Content (calendar icon) — HQ-07
- Campaigns (mail icon) — HQ-08
- Templates (layers icon) — HQ-09
- Harvest (download icon) — HQ-10
- Settings (gear icon)

**Main Content — Network Overview (HQ-01)**:

Row 1 — Key Metrics (4 cards, horizontal on desktop, 2x2 on tablet, stacked on mobile):
- Enrollment Progress: "187 / 260 Enrolled" with circular progress ring and trend arrow
- Network Health: "72 / 100" score with color-coded gauge (red < 50, yellow 50-74, green 75+)
- Content Pipeline: "1,847 posts scheduled this week" with "12 need QC" link to queue
- Active Alerts: "6 alerts" with severity breakdown (2 critical, 4 warning)

Row 2 — Tier Distribution + Alert Feed (2-column on desktop):
- Left: Horizontal stacked bar showing Optimized (green) / Baseline (blue) / Auto-deployed (yellow) / Pending (gray) with counts
- Right: Scrolling alert feed showing most recent alerts with timestamp, practice name, alert type, and action link

Row 3 — Enrollment Funnel (full width):
- Horizontal funnel: Invited (260) > Opened (220) > Engaged (190) > Enrolled (175) > Live (165) > Optimized (45)
- Each stage is clickable to filter the practice list (HQ-02)

Row 4 — Recent Activity (full width):
- Timeline of recent system events: enrollments completed, sites deployed, changes propagated, QC items resolved

### WF-04: Executive Dashboard (EX-01)

**Layout**: Dark theme, designed for projection on a large screen as well as laptop viewing. No sidebar — full-screen data display. Minimal navigation (tabs across the top: Overview / Enrollment / Health / ROI).

**Overview Tab**:

Top Section — The Big Number:
- Centered, large: "Network Digital Health: 72/100" with a semicircular gauge
- Below: "Up from 34/100 at program launch" with a sparkline showing the 90-day trend

Middle Section — Four Quadrant Grid:
- Top-left: Enrollment funnel (simplified: % coverage with breakdown)
- Top-right: Health tier distribution (donut chart with counts)
- Bottom-left: Top 5 alerts requiring attention (card list)
- Bottom-right: This Week's Wins (practices enrolled, sites launched, reviews responded to)

Bottom Section — Practice Map (optional):
- Geographic distribution of practices, color-coded by health score tier
- Hover/tap for practice name and score

---

## Interaction Patterns

### IP-01: Chat Message Flow

The concierge controls the conversation pace. Each message from the concierge includes an expected response type that determines what input controls appear.

| Response Type | Input Control | Example |
|--------------|--------------|---------|
| Free text | Text input field (default) | "Tell me about any changes to your services" |
| Single choice | Quick reply buttons (1 selected, others disappear) | "Which template do you prefer?" |
| Confirmation | Binary quick replies ("Yes" / "No" or "Looks right" / "Edit") | "Is this your current team?" |
| Multi-select | Checkbox-style quick replies (select multiple, then "Done") | "Which services do you offer?" |
| File upload | Attachment button highlighted + drag-drop zone | "Upload a photo of your new team member" |
| Date | Date picker or quick replies with common options | "When is Dr. Johnson's last day?" |
| Rating | Star selector or emoji scale | "How was the enrollment process?" |

**Typing Indicator**: When the concierge is processing (generating a site, fetching data), show a "thinking" animation in the chat — three animated dots in an agent message bubble. For long operations (site generation, 30-60 seconds), replace with a progress message: "Generating your website preview... this takes about 30 seconds."

**Session Persistence**: Conversations persist across sessions. If a practice closes the browser and returns hours later, the chat picks up exactly where it left off. The concierge acknowledges the gap: "Welcome back! We were in the middle of reviewing your team. Ready to continue?"

**5-Minute Window Design**: Every interaction segment is designed to be completable in under 5 minutes. If a flow has multiple segments (verification takes 3 segments), the concierge explicitly offers save points: "That's your team verified! We can do services next, or you can come back later — I'll pick up right where we left off."

### IP-02: Approval Workflow

Two-stage approval: practice approval followed by HQ QC.

**Practice Approval**:
1. Concierge presents the approval summary card in chat
2. Practice taps "Approve All" or flags individual items
3. Flagged items open an inline edit flow
4. Re-approval after changes
5. On approval, status changes to "Pending QC"

**HQ QC**:
1. Item appears in QC queue (HQ-04) with "NEW" badge
2. Reviewer opens the review view (HQ-05): site preview on the left, checklist on the right
3. Checklist items: Brand compliance, content accuracy, layout integrity, SEO basics, mobile responsiveness
4. Approve: item moves to deployment queue
5. Reject: reviewer adds notes; concierge notifies the practice with the feedback
6. Auto-approve: if no action after 48 hours and no critical flags, item auto-approves

**Status Indicators**:
- Draft (gray) > Practice Review (blue) > Pending QC (yellow) > Approved (green) > Live (green with checkmark) > Flagged (orange)

### IP-03: Batch Operations

**Initiation**: Admin selects "New Batch Operation" from the batch operations screen (HQ-06).

**Operation Types**:
- Template update (change to a template that affects all sites using it)
- Brand update (logo, tagline, color change across network)
- Compliance update (legal text, privacy policy, accessibility fix)
- Content campaign (deploy a seasonal message or promotion across selected practices)

**Flow**:
1. Select operation type
2. Select scope (all practices / specific template / specific tier / manual selection)
3. Preview: system generates before/after views for a sample of affected sites
4. Review affected practice count and any flagged conflicts
5. Confirm: operation enters the deployment queue
6. Monitor: real-time progress bar showing deployed / total, with failure list
7. Complete: summary report with success count, failure count, and notification log

---

## Design System

### Colors

The platform uses a **dark theme as the primary interface**. Dark backgrounds reduce eye strain for the HQ team working in the dashboard all day and provide a premium feel for the practice chat.

**Core Palette**:

| Token | Hex | Usage |
|-------|-----|-------|
| `--bg-primary` | `#0F1117` | Main background (dashboard, chat) |
| `--bg-secondary` | `#1A1D27` | Card surfaces, sidebar, elevated elements |
| `--bg-tertiary` | `#252836` | Hover states, input fields, nested cards |
| `--text-primary` | `#F0F0F5` | Headlines, primary content |
| `--text-secondary` | `#9BA1B0` | Descriptions, metadata, timestamps |
| `--text-tertiary` | `#5A6070` | Placeholders, disabled text |
| `--accent-primary` | `#4ADE80` | SGA green — CTAs, success states, health indicators, links |
| `--accent-primary-hover` | `#22C55E` | Hover state for primary accent |
| `--accent-secondary` | `#3B82F6` | Informational states, secondary actions, progress indicators |
| `--accent-warning` | `#F59E0B` | Warning states, pending items, moderate health scores |
| `--accent-danger` | `#EF4444` | Error states, critical alerts, low health scores |
| `--accent-muted` | `#6366F1` | Tags, badges, decorative accents |
| `--border-default` | `#2A2D3A` | Card borders, dividers |
| `--border-focus` | `#4ADE80` | Focus rings for accessibility |

**Agent Message Bubble**: `--bg-secondary` background with `--text-primary` text
**User Message Bubble**: `--accent-primary` background with `#0F1117` text (dark on green)
**Data Verification Cards**: `--bg-tertiary` background with `--border-default` border

**Health Score Colors**:
- 85-100 (Optimized): `--accent-primary` (green)
- 60-84 (Baseline): `--accent-secondary` (blue)
- 40-59 (Auto-deployed): `--accent-warning` (amber)
- 0-39 (Critical): `--accent-danger` (red)

### Typography

| Token | Font | Size | Weight | Line Height | Usage |
|-------|------|------|--------|-------------|-------|
| `--type-display` | Inter | 32px / 2rem | 700 | 1.2 | Dashboard big numbers, health scores |
| `--type-h1` | Inter | 24px / 1.5rem | 600 | 1.3 | Page titles, section headers |
| `--type-h2` | Inter | 20px / 1.25rem | 600 | 1.3 | Card titles, sub-sections |
| `--type-h3` | Inter | 16px / 1rem | 600 | 1.4 | List headers, form labels |
| `--type-body` | Inter | 14px / 0.875rem | 400 | 1.5 | Default text, chat messages, descriptions |
| `--type-body-sm` | Inter | 13px / 0.8125rem | 400 | 1.5 | Metadata, timestamps, help text |
| `--type-caption` | Inter | 12px / 0.75rem | 500 | 1.4 | Badges, labels, status indicators |
| `--type-mono` | JetBrains Mono | 13px / 0.8125rem | 400 | 1.5 | Code, URLs, technical values |

**Font Loading**: Inter is loaded via Google Fonts with `font-display: swap`. JetBrains Mono is loaded only on admin pages where technical values appear.

### Spacing Scale

Based on a 4px grid with a primary increment of 8px.

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Tight spacing: between icon and label, inline elements |
| `--space-2` | 8px | Default spacing: between related elements, padding-sm |
| `--space-3` | 12px | Chat message padding, input padding |
| `--space-4` | 16px | Card padding, section gaps on mobile |
| `--space-5` | 20px | Card padding (desktop), list item spacing |
| `--space-6` | 24px | Section separation, card margins |
| `--space-8` | 32px | Major section gaps, page padding |
| `--space-10` | 40px | Dashboard section separation |
| `--space-12` | 48px | Page-level vertical rhythm |

### Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Badges, small elements |
| `--radius-md` | 8px | Buttons, inputs, cards |
| `--radius-lg` | 12px | Chat bubbles, large cards |
| `--radius-xl` | 16px | Modal windows, overlay panels |
| `--radius-full` | 9999px | Pills, quick-reply buttons, avatars |

### Component Library

#### Buttons

| Variant | Background | Text | Border | Usage |
|---------|-----------|------|--------|-------|
| Primary | `--accent-primary` | `--bg-primary` | none | Main CTAs: "Approve All", "Select Template", "Deploy" |
| Secondary | transparent | `--accent-primary` | `--accent-primary` 1px | Secondary actions: "Edit", "View Details", "Export" |
| Ghost | transparent | `--text-secondary` | none | Tertiary actions: "Cancel", "Back", "Skip" |
| Danger | `--accent-danger` | `--text-primary` | none | Destructive: "Reject", "Remove", "Delete" |

All buttons: `--radius-md`, 40px height (default), 32px height (compact). Min-width 80px. Hover darkens 10%. Active depresses 1px. Disabled at 40% opacity.

#### Cards

- **Surface**: `--bg-secondary` background, `--border-default` 1px border, `--radius-lg`
- **Elevated**: Same as surface with `box-shadow: 0 4px 12px rgba(0,0,0,0.3)`
- **Interactive**: Surface card with hover state (border shifts to `--accent-primary` at 40% opacity)
- **Padding**: `--space-5` on desktop, `--space-4` on mobile

#### Chat Bubbles

- **Agent**: `--bg-secondary`, `--text-primary`, `--radius-lg` with bottom-left squared (`border-bottom-left-radius: --radius-sm`). Max-width 85% of container.
- **User**: `--accent-primary`, `--bg-primary` text, `--radius-lg` with bottom-right squared. Max-width 75% of container.
- **System**: Centered text, `--text-tertiary`, no bubble. Used for timestamps and status updates ("Session resumed").

#### Quick Reply Buttons

- Pill shape: `--radius-full`, `--bg-tertiary` background, `--text-primary` text, `--border-default` border
- Hover: border shifts to `--accent-primary`
- Selected: `--accent-primary` background, `--bg-primary` text
- Horizontal scrollable row, 8px gap between pills
- Tap target minimum 44x44px (accessibility)

#### Data Verification Cards (Chat Inline)

- Full-width within chat, `--bg-tertiary` background, `--border-default` border, `--radius-lg`
- Header: icon + title (e.g., "Your Team") in `--type-h3`
- Body: list of items with edit icons per row
- Footer: action buttons ("Looks Right" primary, "Edit" secondary)
- Editable state: items become inline text inputs, footer changes to "Save Changes" / "Cancel"

#### Health Score Gauge

- Semicircular arc gauge (0-100)
- Arc color segments: red (0-39), amber (40-59), blue (60-84), green (85-100)
- Center: large number in `--type-display`, label below in `--type-caption`
- Available in three sizes: large (dashboard hero), medium (practice detail), small (list item)

#### Alert Badge

- Pill shape: severity color background at 15% opacity, severity color text, `--radius-sm`
- Critical: `--accent-danger` treatment
- Warning: `--accent-warning` treatment
- Info: `--accent-secondary` treatment
- Count badge: solid severity color, white text, `--radius-full`, 20x20px minimum

#### Status Indicator

- Small dot (8px) with label: Draft (gray), In Review (blue), Pending QC (amber), Approved (green), Live (green + checkmark), Flagged (orange)
- Used in practice list rows, QC queue items, content pipeline

#### Table (Practice List)

- Striped rows: alternating `--bg-primary` and `--bg-secondary`
- Sticky header row with sort controls
- Row hover: `--bg-tertiary`
- Row click navigates to practice detail
- Columns: Practice Name, Platform (SGA/Gen4/MODIS), Specialty, Status, Health Score (with mini gauge), Last Activity, Actions
- Filterable by: status, platform, specialty, health tier, enrollment method
- Searchable by practice name

---

## Brand Personality in UI

### Sage + Caregiver Manifested

The concierge is not a generic chatbot. Its personality comes through in every interaction:

**Opinionated Recommendations with Reasoning**:
Every recommendation includes a "because" statement. The UI supports this with a distinct visual pattern:
- Recommendation text in `--type-body` weight 600
- Reasoning text immediately below in `--type-body` weight 400, `--text-secondary` color
- Example: **"I recommend the Perio Pro template"** / "Periodontist sites with referral-optimized layouts see 23% higher conversion from referring GPs."

**Never Bureaucratic**:
- No form-style inputs in the chat flow. Data collection happens conversationally.
- No "Step 3 of 7" progress bars. Use subtle dot indicators only.
- No corporate language in system messages. "Your site is ready for review" not "Storefront package pending approval."

**Warm but Efficient**:
- Greetings are short and personalized: "Hey, Riverside Dental team" not "Welcome to the SGA Digital Storefront Enrollment Platform."
- Confirmations are celebratory but brief: "You're live!" with a small animation, not a 4-paragraph email.
- Error handling is empathetic and solution-oriented: "That photo didn't upload — it might be too large. Try one under 10MB, or I can work with what we have for now."

**Proactive, Not Passive**:
- The concierge initiates conversations, not just responds. UI supports push notifications / unread indicators.
- Dashboard alerts are written as recommendations, not just data: "Riverside Dental's review score dropped 0.6 stars in 14 days — 3 recent negative reviews mention wait times. Suggest flagging for the concierge to check in."

### Concierge Avatar

- A simple, custom icon mark — not a human face, not a generic robot. Think: a stylized dental-shield mark or an abstract "care" symbol that reads as helpful and knowledgeable.
- Appears at 32x32px in chat, 24x24px in notifications, 48x48px in the welcome screen.
- Color: `--accent-primary` (SGA green) on `--bg-secondary`.

---

## Accessibility Requirements (WCAG 2.1 AA)

### Color Contrast

- All text meets 4.5:1 contrast ratio against its background (AA for normal text)
- Large text (18px+ or 14px+ bold) meets 3:1 minimum
- The dark theme palette has been designed with these ratios:
  - `--text-primary` (#F0F0F5) on `--bg-primary` (#0F1117): 15.2:1
  - `--text-secondary` (#9BA1B0) on `--bg-primary` (#0F1117): 7.1:1
  - `--text-primary` on `--bg-secondary` (#1A1D27): 12.8:1
  - `--accent-primary` (#4ADE80) on `--bg-primary`: 9.4:1
- Interactive elements with color-only indicators also include an icon or text label

### Keyboard Navigation

- All interactive elements are reachable via Tab key
- Focus rings use `--border-focus` (#4ADE80), 2px solid, 2px offset
- Chat quick-reply buttons navigable via arrow keys within the group
- Escape closes modals, overlays, and expanded previews
- Enter/Space activates buttons and toggles
- Skip-to-content link on all dashboard pages

### Screen Reader Support

- All images have descriptive `alt` text (team photos, site screenshots, charts)
- Chat messages include `role="log"` and `aria-live="polite"` for new message announcements
- Health score gauges include `aria-label` with the numeric value and tier name
- Status indicators include `aria-label` (not just color dots)
- Charts include a tabular data alternative accessible via screen reader
- Form inputs have associated labels (even visually hidden ones in the chat context)

### Motion & Interaction

- Typing indicators and loading animations respect `prefers-reduced-motion`
- No auto-playing video or audio
- Toast notifications persist for 8 seconds minimum (dismissible by user)
- Animations are under 300ms duration, ease-out timing

### Touch Targets

- All interactive elements meet 44x44px minimum touch target (WCAG 2.5.5)
- Quick-reply buttons have 8px minimum gap between them
- Chat input submit button is 48x48px
- Navigation sidebar items have 48px row height

---

## Mobile-First Design Requirements

### Rationale

Practice staff — office managers, front desk, and dentists — interact with the concierge on their phones between patients. The 5-minute window between appointments is the primary usage context. Desktop is secondary (used when the practice intentionally sits down to review their storefront).

HQ staff use desktop primarily but need mobile access for QC approvals when away from their desk.

### Breakpoints

| Breakpoint | Width | Target Device |
|------------|-------|--------------|
| Mobile (default) | 0 - 639px | Phones (practice staff primary device) |
| Tablet | 640 - 1023px | iPad, landscape phones |
| Desktop | 1024 - 1439px | Laptop (HQ team primary device) |
| Large Desktop | 1440px+ | External monitors, presentation displays |

### Mobile-Specific Patterns

**Practice Chat (Mobile Primary)**:
- Chat interface is full-viewport height with virtual keyboard awareness
- Input area pins to bottom, above the virtual keyboard
- Quick-reply buttons scroll horizontally (no wrapping, no second row)
- Template preview cards stack vertically, one per viewport
- Site preview uses mobile viewport by default (toggle to desktop is available but secondary)
- File upload supports camera capture directly (photo from phone camera)
- Session state persists if the browser is backgrounded and reopened

**HQ Dashboard (Desktop Primary, Mobile Capable)**:
- Navigation collapses to bottom tab bar on mobile (5 key items)
- Dashboard cards stack vertically on mobile
- Practice list becomes a card list (not a table) on mobile
- QC review uses swipe gestures: swipe right to approve, swipe left to reject
- Batch operations are desktop-only (too complex for mobile)

**Executive Dashboard (Desktop Primary)**:
- Mobile view shows the big number (health score) and alert list only
- Full quadrant view is desktop/tablet only
- "Share" button exports a screenshot or PDF for mobile viewing

### Performance Targets (Mobile)

- First Contentful Paint: < 1.5 seconds on 4G
- Time to Interactive: < 3 seconds on 4G
- Chat message send-to-response: < 2 seconds perceived (show typing indicator immediately)
- Template preview load: < 3 seconds per preview image
- Total page weight: < 500KB initial load (chat), < 1MB (dashboard)

---

## 5-Minute Window Design

The concierge chat is purpose-built for interrupted, 5-minute usage sessions. Every design decision supports this constraint.

### Principles

1. **Instant Context Restoration**: When a user returns, the chat shows the last 3 messages and the current step indicator. No "where was I?" confusion. The concierge explicitly welcomes them back with context: "Welcome back! We were picking your template."

2. **Micro-Completions**: Each verification step (team, services, hours, template, approval) is designed to complete in 2-4 minutes. The concierge offers explicit save points between steps: "Team is verified! Want to do services now, or come back later?"

3. **Minimal Typing**: Quick-reply buttons handle 80%+ of enrollment responses. Corrections use inline editing on pre-populated data (tap to fix), not blank text fields. Photo upload uses the phone camera directly — one tap.

4. **Progressive Loading**: Template previews load as thumbnail cards first (instant), with full-resolution previews loading on demand (tap to expand). No waiting for assets that may not be viewed.

5. **Offline Resilience**: If connectivity drops mid-conversation, typed messages queue locally and send when reconnected. A subtle "Reconnecting..." banner appears, but no data is lost. The chat never shows a blank error screen.

6. **Notification Re-engagement**: When the concierge needs the practice (QC complete, weekly report, detected change), a push notification or SMS brings them back to the exact context. The notification includes a one-line summary: "Your new site is approved and ready to launch — tap to go live."

### Enrollment Timing Budget

| Step | Target Duration | Interaction Type |
|------|----------------|-----------------|
| Welcome + team verification | 3-4 min | Confirm/edit pre-populated list |
| Services + hours verification | 2-3 min | Confirm/edit pre-populated data |
| Template selection | 2-3 min | View 2-3 previews, select one |
| Customization (optional) | 1-2 min | Color/image tweaks |
| Final review + approval | 2-3 min | Review checklist, tap approve |
| **Total** | **10-15 min** | **Across 2-3 sessions** |

The concierge explicitly sets this expectation in the welcome message: "This takes about 15 minutes total — we can do it all now or spread it across a few visits. Whatever works for you."
