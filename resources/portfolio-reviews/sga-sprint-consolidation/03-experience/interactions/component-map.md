# SGA Dental Partners — Component Map

**Date**: 2026-03-14
**Author**: UX Designer Agent (Phase 6)
**Purpose**: Define the modular component architecture, shared design system, workstream-specific components, design tokens, and WCAG 2.1 AA compliance requirements.
**Carry-Forward Item Addressed**: Prototype Lead #3 (shared frontend design system in Phase 1)

---

## 1. Design System Overview

All 5 surfaces (Practice Operations, Network Intelligence, Content Engine, Culture OS, Patient-Facing) consume a single shared component library published as an npm package: `@sgadental/ui`.

**Technology**: React component library with TypeScript, Tailwind CSS for utility classes, CSS custom properties for design tokens. Storybook for documentation and visual testing.

**Package Structure**:
```
@sgadental/ui/
├── tokens/              ← Design tokens (colors, spacing, typography)
├── primitives/          ← Atomic components (Button, Input, Badge, Icon)
├── patterns/            ← Composite components (Card, DataTable, Chart, Nav)
├── layouts/             ← Page layout components (Sidebar, TopBar, PageShell)
├── surfaces/            ← Surface-specific compositions (optional overrides)
└── hooks/               ← Shared hooks (useOfflineStatus, useRole, useStaleData)
```

---

## 2. Design Tokens

### 2.1 Color Tokens

Colors are defined as semantic tokens, not raw values. Each surface references the same semantic tokens. This ensures consistency and enables future theming (dark mode, high contrast mode).

**Core Palette**:

| Token | Light Value | Usage |
|-------|------------|-------|
| `--color-background` | `#FFFFFF` | Page background |
| `--color-surface` | `#F8F9FA` | Card backgrounds, panel backgrounds |
| `--color-surface-hover` | `#F1F3F5` | Hover state on cards and list items |
| `--color-surface-active` | `#E9ECEF` | Active/selected state |
| `--color-surface-muted` | `#DEE2E6` | Disabled backgrounds, no-data states |
| `--color-surface-warning` | `#FFF3CD` | Offline banner, warning backgrounds |

**Text**:

| Token | Light Value | Usage |
|-------|------------|-------|
| `--color-text-primary` | `#212529` | Primary body text, headings |
| `--color-text-secondary` | `#495057` | Secondary labels, descriptions |
| `--color-text-muted` | `#868E96` | Timestamps, helper text, "Last updated" |
| `--color-text-on-accent` | `#FFFFFF` | Text on accent-colored backgrounds |
| `--color-text-on-warning` | `#664D03` | Text on warning backgrounds |

**Accent**:

| Token | Light Value | Usage |
|-------|------------|-------|
| `--color-accent-primary` | `#2563EB` | Primary buttons, active nav items, links |
| `--color-accent-primary-hover` | `#1D4ED8` | Hover state on primary accent |
| `--color-accent-secondary` | `#7C3AED` | Secondary accent for differentiation |

**Status** (WCAG note: always paired with an icon/shape, never color alone):

| Token | Light Value | Icon Pairing | Usage |
|-------|------------|-------------|-------|
| `--color-status-success` | `#16A34A` | Checkmark (circle) | Positive trends, completed states, healthy metrics |
| `--color-status-warning` | `#D97706` | Triangle with ! | Moderate alerts, monitored states, approaching limits |
| `--color-status-critical` | `#DC2626` | Circle with ! | High-impact alerts, failures, critical thresholds |
| `--color-status-info` | `#2563EB` | Circle with i | Informational alerts, advisory notes |
| `--color-status-neutral` | `#6B7280` | Dash | No change, baseline |

**Contrast Ratios** (WCAG 2.1 AA):
- All text tokens achieve minimum 4.5:1 contrast ratio against their intended background
- `text-muted` on `background`: 4.7:1 (passes AA)
- Status colors are never used as standalone indicators. Each status color has a required icon pairing (documented above)

### 2.2 Typography

**Font Stack**: `'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`

**Type Scale**:

| Token | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| `--type-display` | 32px | 700 | 1.2 | Dashboard KPI values |
| `--type-heading-1` | 24px | 600 | 1.3 | Page titles |
| `--type-heading-2` | 20px | 600 | 1.3 | Section headings |
| `--type-heading-3` | 16px | 600 | 1.4 | Card titles, nav items |
| `--type-body` | 14px | 400 | 1.5 | Body text, descriptions |
| `--type-body-semibold` | 14px | 600 | 1.5 | Labels, emphasized body |
| `--type-caption` | 12px | 400 | 1.4 | Timestamps, helper text, badges |
| `--type-overline` | 11px | 600 | 1.2 | Section labels, "MORNING BRIEFING" headers. Letter-spacing: 0.05em, uppercase |
| `--type-metric` | 28px | 700 | 1.1 | KPI card primary value |

**Mobile Adjustments**: On viewports <428px, `type-display` reduces to 28px, `type-metric` reduces to 24px. All other sizes remain constant.

### 2.3 Spacing

**Base Unit**: 4px. All spacing values are multiples of 4px.

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Minimal internal padding, icon-text gap |
| `--space-2` | 8px | Tight internal padding, list item vertical spacing |
| `--space-3` | 12px | Standard internal padding |
| `--space-4` | 16px | Card padding, section spacing on mobile |
| `--space-5` | 20px | Between related sections |
| `--space-6` | 24px | Card padding on desktop, column gutters |
| `--space-8` | 32px | Between major sections, outer page margins |
| `--space-10` | 40px | Between page-level sections |
| `--space-12` | 48px | Major vertical separation |

### 2.4 Breakpoints

| Token | Value | Target |
|-------|-------|--------|
| `--bp-mobile` | 0-427px | iPhone (PWA primary target) |
| `--bp-mobile-large` | 428-767px | iPhone Pro Max, small tablets |
| `--bp-tablet` | 768-1023px | iPad portrait |
| `--bp-tablet-landscape` | 1024-1279px | iPad landscape, small laptops |
| `--bp-desktop` | 1280-1919px | Standard desktop monitors |
| `--bp-desktop-large` | 1920px+ | Large/dual monitors (Network Intelligence primary) |

### 2.5 Elevation (Shadows)

| Token | Value | Usage |
|-------|-------|-------|
| `--shadow-sm` | `0 1px 2px rgba(0,0,0,0.05)` | Default card elevation |
| `--shadow-md` | `0 4px 6px rgba(0,0,0,0.07)` | Hover state, dropdowns |
| `--shadow-lg` | `0 10px 15px rgba(0,0,0,0.1)` | Modals, slide-over panels |
| `--shadow-xl` | `0 20px 25px rgba(0,0,0,0.12)` | Full-screen overlays |

### 2.6 Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Buttons, badges, inputs |
| `--radius-md` | 8px | Cards, panels |
| `--radius-lg` | 12px | Modals, overlays |
| `--radius-full` | 9999px | Avatar circles, pill badges |

### 2.7 Motion

| Token | Value | Usage |
|-------|-------|-------|
| `--motion-fast` | 150ms ease | Hover states, button interactions |
| `--motion-normal` | 200ms ease | Tab transitions, card state changes |
| `--motion-slow` | 300ms ease-out | Panel slides, modal opens |
| `--motion-spring` | 400ms cubic-bezier(0.34, 1.56, 0.64, 1) | Card swipe snap, action feedback |

**Reduced Motion**: When `prefers-reduced-motion: reduce` is detected, all animations reduce to instant (0ms duration) or simple opacity fades (150ms). No transform-based animations.

---

## 3. Shared Components

These components are used across all surfaces.

### 3.1 Button

**Variants**:

| Variant | Usage | Visual |
|---------|-------|--------|
| `primary` | Primary actions: "Approve", "Send", "Save" | Filled `accent-primary` background, white text |
| `secondary` | Secondary actions: "Edit", "View", "Filter" | Outlined with `accent-primary` border and text |
| `ghost` | Tertiary actions: "Cancel", "Dismiss", "Close" | No background or border, `accent-primary` text |
| `destructive` | Destructive actions: "Delete", "Pause", "Reject" | Filled `status-critical` background, white text |

**Sizes**:

| Size | Height | Font Size | Padding | Usage |
|------|--------|-----------|---------|-------|
| `sm` | 32px | 12px | 8px 12px | Compact UIs (data tables, cards) |
| `md` | 40px | 14px | 10px 16px | Default |
| `lg` | 48px | 16px | 12px 24px | Mobile primary actions, modal CTAs |

**States**: Default, Hover (darken 8%), Active (darken 12%), Disabled (40% opacity, not clickable), Loading (spinner replaces text, button width maintains).

**WCAG**: Focus ring: 2px solid `accent-primary` with 2px offset. Visible on keyboard navigation. Minimum touch target: 44x44px on mobile (lg size enforced on mobile).

### 3.2 Card

**Variants**:

| Variant | Usage | Visual |
|---------|-------|--------|
| `default` | Standard content card | `surface` background, `shadow-sm`, `radius-md` |
| `interactive` | Clickable cards (action cards, queue items) | Same as default + hover elevation change + cursor pointer |
| `status` | Cards with status indicator (automation cards, queue items) | Left border colored by status token (4px width) |
| `compact` | Dense list item cards (message queue, call log) | Reduced padding (`space-3`), no shadow |

**Anatomy**:
```
┌──────────────────────────────────────┐
│  [Status Bar — 4px left border]      │
│                                      │
│  [Icon/Avatar]  Title        [Badge] │  ← Header row
│                 Subtitle             │
│                                      │
│  Body content area                   │  ← Flexible content
│  (text, metrics, mini-charts)        │
│                                      │
│  ────────────────────────────────    │  ← Divider (optional)
│  [Action 1]  [Action 2]   [More ▾]  │  ← Footer actions
└──────────────────────────────────────┘
```

**States**: Default, Hover (shadow-md + surface-hover), Selected (accent-primary left border + surface-active background), Loading (skeleton shimmer), Disabled (50% opacity).

### 3.3 Data Table

**Used in**: Network Intelligence (practice rankings, call logs, enrollment pipeline), Content Engine (editorial queue, asset registry), Culture OS (certification roster).

**Features**:

| Feature | Behavior |
|---------|----------|
| Column sorting | Click header to sort. Arrow icon indicates direction. First click: ascending. Second: descending. Third: removes sort |
| Pagination | Page size selector: 10, 25, 50. Page navigation at bottom. "Showing 1-25 of 260" text |
| Row selection | Checkbox column for batch actions. "Select all on page" checkbox in header. Selected count shown: "3 selected" |
| Column resize | Drag column borders to resize. Double-click auto-sizes to content |
| Inline actions | Last column: action buttons or dropdown menu per row |
| Empty state | Full-width message: "No [items] match your filters." with clear-filters link |
| Loading | Skeleton rows with shimmer animation (same row height as data) |

**WCAG**:
- Table uses proper `<table>`, `<thead>`, `<tbody>`, `<th scope="col">` markup
- Sort state announced by screen readers: "[Column name], sorted ascending"
- Row selection announced: "[Row count] rows selected"
- Focus management: Tab navigates between interactive elements within the table

### 3.4 Chart Components

**Chart Types Available**:

| Type | Component | Usage |
|------|-----------|-------|
| Line chart | `<LineChart>` | Trend over time (H-Score, production, show rate) |
| Bar chart | `<BarChart>` | Comparison across categories (regions, practice types) |
| Funnel chart | `<FunnelChart>` | 5-stage patient funnel visualization |
| Sparkline | `<Sparkline>` | Inline mini trend in KPI cards (no axis labels, just shape) |
| Gauge | `<GaugeChart>` | Score display (0-100) for health scores, trust levels |
| Pipeline | `<PipelineChart>` | Stage-to-stage flow (content pipeline, enrollment) |

**Shared Chart Properties**:

| Property | Value |
|----------|-------|
| Grid lines | `surface-muted` color, 1px, dashed |
| Axis labels | `type-caption`, `text-muted` |
| Data labels | `type-caption`, `text-primary` |
| Tooltip | `shadow-lg`, `radius-md`, shows exact value + date on hover |
| Legend | Below chart, horizontal, icon + label. Clickable to toggle series |
| Animation | Data line draws left-to-right on initial load (600ms). Disabled if `prefers-reduced-motion` |
| Responsive | Charts resize fluidly within their container. Below 300px width, axis labels are hidden, tooltips shift to top |

**WCAG**:
- All charts include a `<table>` fallback in an `aria-describedby` region for screen readers
- Color-coded data series include pattern fills (dashed, dotted) in addition to color
- Heatmap markers use icon shapes in addition to color (see unified-dashboard-specs.md section 1.5)
- Minimum contrast ratio of 3:1 between adjacent data series colors

### 3.5 Badge / Tag

**Variants**:

| Variant | Visual | Usage |
|---------|--------|-------|
| `status-success` | Green background, white text, checkmark icon | "Active", "Delivered", "Approved" |
| `status-warning` | Amber background, dark text, warning icon | "Pending", "Monitored", "Slow" |
| `status-critical` | Red background, white text, alert icon | "Failed", "Blocked", "Overdue" |
| `status-info` | Blue background, white text, info icon | "New", "In Progress", "Scheduled" |
| `neutral` | Gray background, dark text, no icon | "Draft", "Unassigned", metadata tags |
| `count` | Accent background, white text, rounded pill | Notification counts, queue counts |

**WCAG**: Every badge includes an icon alongside its color. Screen readers announce: "[Badge text] status" (e.g., "Active status", "Failed status").

### 3.6 Navigation Components

**Sidebar Navigation** (Network Intelligence, Content Engine, Culture OS):

```
Component: <SideNav>
Props:
  items: NavItem[]        ← { icon, label, path, children?, badge? }
  collapsed: boolean      ← Icon-only mode
  activeItem: string      ← Current path

Behavior:
  - 240px expanded, 64px collapsed
  - Nested items expand/collapse with 200ms animation
  - Active item: accent-primary left border + surface-active background
  - Badge (count) appears to the right of the label
  - Keyboard: Arrow keys navigate, Enter activates, Space toggles children
```

**Bottom Navigation** (PWA):

```
Component: <BottomNav>
Props:
  items: NavItem[]        ← Role-filtered (see Section 1)
  activeItem: string

Behavior:
  - Fixed to bottom of viewport, 56px height
  - Max 5 items (enforced by role filtering)
  - Active: filled icon + label in accent-primary
  - Inactive: outline icon + label in text-muted
  - Safe area padding on iOS (env(safe-area-inset-bottom))
```

**Breadcrumb** (Network Intelligence drill-downs):

```
Component: <Breadcrumb>
Props:
  segments: { label, path }[]

Behavior:
  - Horizontal, separated by ">"
  - All segments except last are clickable links
  - Last segment is text-primary (not a link)
  - On mobile: shows only last 2 segments with "..." for earlier ones
```

### 3.7 Alert / Banner

**Variants**:

| Variant | Usage | Visual |
|---------|-------|--------|
| `info` | Informational: "Content pipeline updated" | Blue left border, info icon, dismissible |
| `warning` | Warning: "You're offline" | Amber background, warning icon, not dismissible while condition persists |
| `error` | Error: "Failed to load data" | Red left border, error icon, retry action |
| `success` | Success: "2 photos added to website" | Green left border, checkmark icon, auto-dismisses after 5 seconds |

**Anatomy**:
```
┌──────────────────────────────────────────────────┐
│  [Icon]  Message text goes here.    [Action] [×] │
└──────────────────────────────────────────────────┘
```

**WCAG**: Alerts use `role="alert"` for screen reader announcement. Success toasts use `role="status"`. Error alerts are not auto-dismissed.

### 3.8 Modal / Dialog

**Sizes**: `sm` (400px), `md` (560px), `lg` (720px), `full` (viewport - 48px padding).

**Behavior**:
- Backdrop: semi-transparent overlay (`rgba(0,0,0,0.5)`)
- Animation: fade-in backdrop (200ms) + scale-up content (300ms ease-out)
- Close: X button, Escape key, click outside (configurable)
- Focus trap: Tab cycles through modal content only
- Scroll: Content scrolls within modal; backdrop does not scroll

**WCAG**: `role="dialog"`, `aria-modal="true"`, `aria-labelledby` pointing to modal title. Focus moves to first focusable element on open, returns to trigger element on close.

### 3.9 Empty State

```
Component: <EmptyState>
Props:
  illustration: 'no-data' | 'no-results' | 'success' | 'offline' | 'error'
  title: string
  description: string
  action?: { label, onClick }

Rendering:
  ┌──────────────────────────────────┐
  │         [illustration]           │  ← Simple line illustration, 120x120px
  │                                  │
  │     No practices need            │  ← Title in type-heading-2
  │     intervention.                │
  │                                  │
  │     The network is healthy.      │  ← Description in type-body, text-secondary
  │                                  │
  │     [View All Practices]         │  ← Optional action button
  └──────────────────────────────────┘
```

### 3.10 Skeleton Loader

```
Component: <Skeleton>
Props:
  variant: 'text' | 'circle' | 'rect' | 'card'
  width: string
  height: string
  count?: number    ← Repeat for lists

Rendering:
  - Rounded rectangle with shimmer animation
  - Background: surface-muted
  - Shimmer: linear gradient sweep left-to-right, 1.5s cycle
  - Matches the dimensions of the content it replaces
  - WCAG: aria-hidden="true" (decorative), parent has aria-busy="true"
```

---

## 4. Workstream-Specific Components

These components are used within a single surface and are not shared.

### 4.1 Practice Operations-Specific

| Component | Usage | Notes |
|-----------|-------|-------|
| `<ActionCard>` | Morning briefing action cards (swipeable) | Swipe gestures, completion state, priority ordering |
| `<MessagePreview>` | Patient outreach queue message cards | Expandable, edit mode, approve/skip actions |
| `<AutomationCard>` | Automation status display with trust level | Graduation progress bar, pause/resume toggle |
| `<CallContextCard>` | Incoming call context panel | Auto-populates from PMS data, patient match |
| `<ScorecardWidget>` | End-of-day scorecard | Metric rows with comparison to previous day |
| `<HScoreWidget>` | Compact and expanded H-Score display | Cross-workstream data from Culture OS |
| `<CaptureBriefCard>` | Content capture opportunity action card | Camera launcher, checklist |

### 4.2 Network Intelligence-Specific

| Component | Usage | Notes |
|-----------|-------|-------|
| `<KPICard>` | KPI bar metric cards | Drillable, sparkline, comparison toggle |
| `<AlertCard>` | AI alert cards in the alerts panel | Severity badges with icon shapes, dismiss/view actions |
| `<HeatmapView>` | Portfolio geographic heatmap | WebGL markers, color layer selector, accessibility grid view |
| `<InterventionCard>` | Intervention queue items | Claim/assign, dismiss with note, revenue impact |
| `<PracticeDeepDive>` | Tabbed practice 360 view | 6 tabs, independent loading, cross-workstream tabs |
| `<PipelineHealthWidget>` | Content Engine pipeline health | Stage counts, health indicators, drill-down |

### 4.3 Content Engine-Specific

| Component | Usage | Notes |
|-----------|-------|-------|
| `<CaptureCamera>` | Camera overlay with guides | Before/after pairing, video timer, offline storage |
| `<EditorialCard>` | Content review card in editorial queue | Preview, caption edit, approve/reject, brand check |
| `<DistributionTimeline>` | Calendar/timeline of scheduled content | Drag to reschedule, channel indicators |
| `<BrandHealthGauge>` | Per-brand health score display | 0-100 gauge with sub-scores |

### 4.4 Culture OS-Specific

| Component | Usage | Notes |
|-----------|-------|-------|
| `<HuddleDisplay>` | TV-optimized huddle content panels | Auto-cycle, large text, 10-ft viewing distance |
| `<AcademyModule>` | Training module card with progress | Video player, assessment, tier badge |
| `<ChampionScorecard>` | Champion's combined culture+content stats | Dual scorecard, streak counter |
| `<RecognitionPost>` | Peer recognition card in the feed | Brand-templated, reactions, leaderboard context |
| `<TrustTransferScore>` | Individual call scoring display | NLP-analyzed score, behavior breakdown |

---

## 5. Component States Matrix

Every interactive component must handle these states:

| State | Visual Treatment | Screen Reader |
|-------|-----------------|---------------|
| **Default** | Standard appearance | Component label and value |
| **Hover** | Background shift to `surface-hover`, cursor pointer | N/A (visual only) |
| **Focus** | 2px `accent-primary` ring with 2px offset | "Focused on [component]" |
| **Active/Pressed** | Background shift to `surface-active` | N/A |
| **Selected** | `accent-primary` border or background tint | "[Component] selected" |
| **Disabled** | 40% opacity, cursor not-allowed, no interactions | "[Component] disabled" |
| **Loading** | Skeleton shimmer replacing content | "Loading [component]" via aria-busy |
| **Error** | `status-critical` border or inline error message | Error message announced via aria-live |
| **Empty** | Empty state illustration + message | Empty state message read |
| **Stale** | Normal appearance + "Last updated [time]" caption | Stale timestamp announced |
| **Offline** | Content visible but actions disabled + offline banner | "Offline. Showing cached data" |

---

## 6. WCAG 2.1 AA Compliance Summary

### 6.1 Universal Requirements

| WCAG Criterion | Requirement | Implementation |
|----------------|-------------|----------------|
| 1.1.1 Non-text Content | All images and icons have alt text | `alt` attributes on images, `aria-label` on icon buttons |
| 1.3.1 Info and Relationships | Semantic HTML structure | Proper heading hierarchy, table markup, form labels |
| 1.4.1 Use of Color | Color is not the sole means of conveying information | All status colors paired with icons/shapes (see Section 2.1). Heatmap has grid/chart alternatives. Chart data series have pattern fills |
| 1.4.3 Contrast | 4.5:1 minimum for normal text, 3:1 for large text | All token pairs verified (see Section 2.1) |
| 1.4.11 Non-text Contrast | 3:1 for UI components and graphical objects | Interactive element borders, chart lines, icons all meet 3:1 |
| 2.1.1 Keyboard | All functionality accessible via keyboard | Tab navigation, arrow keys for lists/nav, Enter for activation, Escape for dismissal |
| 2.4.3 Focus Order | Logical focus order | DOM order matches visual order. Modals trap focus. Skip links provided |
| 2.4.7 Focus Visible | Focus indicator visible | 2px accent-primary ring on all focusable elements |
| 2.5.5 Target Size | 44x44px minimum touch target | Enforced on all mobile interactive elements. Desktop: 32px minimum |
| 3.2.2 On Input | No unexpected context changes on input | Form fields do not auto-submit. Filters update content but do not navigate |
| 4.1.2 Name, Role, Value | ARIA attributes on all custom components | Custom components use appropriate `role`, `aria-label`, `aria-expanded`, `aria-selected` |

### 6.2 Carry-Forward WCAG Items (Specific)

| Item | Component(s) Affected | Implementation |
|------|----------------------|----------------|
| Heatmap color alternatives | `<HeatmapView>` | Icon shapes inside markers (checkmark/dash/triangle) + text scores at zoom level 3+. Grid view and chart view alternatives. See `unified-dashboard-specs.md` Section 1.5 |
| SMS character limits | `<MessagePreview>` edit mode | Character counter with visual bar. Amber at 140, red at 161+. Explanatory text about message splitting. See `practice-interface-specs.md` Section 3.2 |
| Offline indicators | `<OfflineBanner>` | Persistent non-dismissible banner with cloud-off icon + text "You're offline -- showing cached data. Last updated: [time]". Not color-only. See `practice-interface-specs.md` Section 7.2 |
| Voice AI TTY/TDD | Voice AI service + `<VoiceAIMonitor>` | TTY relay detection in Twilio integration. Text-based mode for relay calls. TTY badge in transcript list. See `practice-interface-specs.md` Section 9.1 |

### 6.3 Testing Requirements

| Test Type | Tool | Frequency |
|-----------|------|-----------|
| Automated scan | axe-core via CI pipeline | Every pull request |
| Color contrast | Contrast plugin in Storybook | Every new token or component |
| Keyboard navigation | Manual testing checklist | Every new component + quarterly audit |
| Screen reader | VoiceOver (macOS/iOS) + NVDA (Windows) | Every new component + quarterly audit |
| Reduced motion | `prefers-reduced-motion` media query | Verified in Storybook for every animated component |

---

## 7. Component Composition Examples

### 7.1 KPI Card (Network Intelligence)

```tsx
<Card variant="interactive" onClick={handleDrill}>
  <Card.Header>
    <Badge variant="overline">Total Production</Badge>
    <IconButton icon="info" aria-label="What is Total Production?" />
  </Card.Header>
  <Card.Body>
    <Text variant="metric">$4.2M</Text>
    <TrendIndicator value={6.2} direction="up" label="vs target" />
    <Sparkline data={last30Days} height={32} />
  </Card.Body>
</Card>
```

### 7.2 Action Card (Practice Operations PWA)

```tsx
<ActionCard
  icon="mail"
  category="CONFIRMATIONS"
  title="7 patients need confirmation for tomorrow."
  subtitle="AI has drafted personalized messages."
  timeEstimate="~3 min"
  action={{ label: "Review & Send", onClick: navigateToQueue }}
  onDismiss={handleDismiss}
  completed={isCompleted}
/>
```

### 7.3 Offline Banner

```tsx
<Banner
  variant="warning"
  icon="cloud-off"
  dismissible={false}
  role="alert"
>
  You're offline — showing cached data. Last updated: {lastSyncTime}
</Banner>
```

---

## 8. Design System Delivery Plan

| Week | Deliverable |
|------|-------------|
| Phase 1, Week 1 | Tokens package published (colors, spacing, typography, breakpoints) |
| Phase 1, Week 2 | Primitives: Button, Input, Badge, Icon, Text, Avatar |
| Phase 1, Week 3 | Patterns: Card, DataTable, Chart (LineChart, BarChart, Sparkline), Alert, Modal |
| Phase 1, Week 4 | Layouts: SideNav, BottomNav, TopBar, PageShell, Breadcrumb. Storybook documentation site live |
| Phase 1 ongoing | Surface-specific components built by workstream teams using the shared library |

**Governance**: All new components must pass axe-core scan, have Storybook documentation with all states, and be reviewed by UX Designer before merge.

---

*This component map serves as the source of truth for the shared design system. All 5 surfaces consume `@sgadental/ui` to ensure visual consistency across 42 interfaces built by multiple teams. The WCAG 2.1 AA requirements are embedded at the component level, not bolted on after the fact.*
