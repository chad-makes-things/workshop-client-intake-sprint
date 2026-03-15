# SGA Dental Partners — Unified Dashboard Interaction Specs

**Date**: 2026-03-14
**Author**: UX Designer Agent (Phase 6)
**Surface**: Network Intelligence (WS1 Surface B)
**Interfaces Covered**: NI-1 (Executive Dashboard), NI-2 (Portfolio Heatmap), NI-3 (Intervention Queue), NI-5 (Practice Deep Dive)
**Carry-Forward Items Addressed**: Content Engine pipeline health touchpoint (Design Director advisory), heatmap color alternatives (WCAG 2.1 AA)

---

## 1. Executive Dashboard (NI-1)

### 1.1 Layout Specification

**Viewport**: Desktop-first, minimum 1280px wide. Optimized for 1920x1080 and above.

**Layout Grid**: 12-column grid, 24px gutters, 32px outer margins.

```
┌──────────────────────────────────────────────────────────────────┐
│  Top Bar: SGA logo | "Network Intelligence" | Search | User Menu │
├──────────────────────────────────────────────────────────────────┤
│  Side Nav (240px collapsed to 64px icon-only)                    │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  KPI Bar (5 cards, full width, 120px height)               │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │  ┌─────────────────────┐  ┌─────────────────────────────┐ │  │
│  │  │  AI Alerts Panel    │  │  Portfolio Heatmap           │ │  │
│  │  │  (4 col, 400px min) │  │  (8 col, 600px min)         │ │  │
│  │  │                     │  │                              │ │  │
│  │  │                     │  │                              │ │  │
│  │  └─────────────────────┘  └─────────────────────────────┘ │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │  Trend Charts (full width, 360px height)                   │  │
│  │  [Metric selector] [Time range: 30d | 60d | 90d | Custom] │  │
│  └────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 KPI Bar Interactions

The KPI bar contains 5 metric cards. Each is a self-contained interactive element.

**KPI Cards**:

| Position | Metric | Source | Update Frequency |
|----------|--------|--------|-----------------|
| 1 | Total Production (MTD) | ClickHouse funnel data | Real-time (15-min refresh) |
| 2 | New Patients (MTD) | ClickHouse funnel data | Real-time (15-min refresh) |
| 3 | Show Rate (network avg) | ClickHouse funnel data | Real-time (15-min refresh) |
| 4 | Treatment Acceptance (network avg) | ClickHouse funnel data | Real-time (15-min refresh) |
| 5 | Network H-Score (avg) | Culture OS via Kafka `sgadental.ws3.hscore-calculated` | Daily batch (6 AM ET) |

**Per-Card Anatomy**:
```
┌─────────────────────────┐
│  Metric Label    [i]    │  ← Label in 12px semibold caps, info tooltip icon
│  $4.2M                  │  ← Primary value in 28px bold
│  ▲ 6.2% vs target       │  ← Comparison line in 14px, green/red + arrow icon
│  ── sparkline ──        │  ← 30-day sparkline, 32px height, no axis labels
└─────────────────────────┘
```

**Interaction States**:

| State | Behavior |
|-------|----------|
| Default | Card displays current value, comparison, sparkline |
| Hover | Card background shifts to `surface-hover` token. Cursor changes to pointer. Subtle elevation increase (2px shadow) |
| Click | Opens a drill-down panel below the KPI bar (pushes content down, does not overlay). Shows: 30/60/90 day trend chart for that metric, top 5 and bottom 5 practices for that metric, comparison vs prior period |
| Active (drilled) | Card border changes to `accent-primary`. A collapse chevron appears in top right. Clicking again collapses the panel |
| Loading | Shimmer animation on value and sparkline areas. Label remains visible |
| Error | Value displays "---" in muted text. Tooltip on hover: "Unable to load. Last known: $3.9M (2h ago)" |
| Stale | For H-Score (daily batch): if >48h since last update, show subtle timestamp: "Updated 2 days ago" in 11px muted text below sparkline |

**Comparison Toggle**: A small toggle below the KPI bar switches the comparison baseline:
- "vs. Target" (default)
- "vs. Prior Month"
- "vs. Prior Year"

Changing the toggle re-animates all 5 comparison lines with a 200ms ease transition.

### 1.3 AI Alerts Panel

**Purpose**: Surface the top 5 anomalies requiring executive attention, ranked by revenue impact.

**Data Source**: AI Analysis Engine (Python service querying ClickHouse, producing ranked anomaly objects).

**Alert Card Anatomy**:
```
┌──────────────────────────────────────────────┐
│  [!] HIGH IMPACT                 3 hours ago │  ← Severity badge + timestamp
│                                              │
│  Southeast region show rate dropped 4.2%     │  ← Title: 16px semibold, max 2 lines
│  week-over-week.                             │
│                                              │
│  Root cause: 3 practices had front desk      │  ← AI summary: 14px regular, max 3 lines
│  turnover simultaneously.                    │
│                                              │
│  [View Practices]  [Dismiss]                 │  ← Actions: primary + ghost buttons
└──────────────────────────────────────────────┘
```

**Severity Levels**:

| Level | Badge Color | Icon | Criteria |
|-------|-------------|------|----------|
| High Impact | `status-critical` (red) + text label | Filled circle with ! | Estimated weekly revenue impact > $10,000 |
| Medium Impact | `status-warning` (amber) + text label | Filled triangle with ! | Estimated weekly revenue impact $2,000-$10,000 |
| Advisory | `status-info` (blue) + text label | Filled circle with i | Pattern detected, no immediate revenue impact |

**WCAG Note**: Severity is conveyed via icon shape (circle, triangle, info-circle) in addition to color. Screen readers announce: "High impact alert: [title]".

**Interaction States**:

| Interaction | Behavior |
|-------------|----------|
| Scroll | Alerts panel scrolls independently of the rest of the dashboard. Max 5 visible; "View all alerts" link at bottom |
| Click "View Practices" | Navigates to Practice Deep Dive (NI-5) with the relevant practices pre-filtered. URL updates: `/practices?alert=<alert_id>` |
| Click "Dismiss" | Alert fades out (300ms opacity transition). Dismissed alerts move to a "Dismissed" section accessible via a filter toggle. Dismissal is per-user, not global |
| New alert arrives | Alert slides in from top of the panel with 400ms ease-out animation. Badge count on the alerts panel header increments |
| Empty state | Panel shows: illustration + "No anomalies detected. The network is performing within normal ranges." |
| Loading | 3 skeleton card placeholders with shimmer animation |
| Error | "Unable to load alerts. Retrying..." with a subtle pulsing indicator. Auto-retry every 30 seconds, max 3 attempts, then show "Contact support" link |

### 1.4 Content Engine Pipeline Health (NEW TOUCHPOINT)

**Carry-forward item**: Design Director advisory identified a missing touchpoint -- executives have no visibility into whether the Content Engine pipeline itself is backed up.

**Placement**: New widget in the Executive Dashboard, positioned below the Trend Charts section OR as an expandable row within the KPI bar drill-down for "Content Activity."

**Widget Design**:
```
┌──────────────────────────────────────────────────────────────┐
│  Content Pipeline Health                    [Last 7 days ▾]  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐     │
│  │  47  │──▸│  38  │──▸│  12  │──▸│  31  │──▸│  28  │     │
│  │Captured│  │Processing│ │In Review│ │Approved│ │Published│ │
│  │  ▲12% │  │  ●OK  │  │  ▲3d avg│ │  ●OK  │  │  ▲8%  │  │
│  └──────┘   └──────┘   └──────┘   └──────┘   └──────┘     │
│                                                              │
│  ⚠ 4 assets stuck in review >5 days  [View in Content Engine]│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Data Source**: Content Engine events via Kafka:
- `sgadental.ws2.capture-media-received` (Captured count)
- `sgadental.ws2.content-asset-created` (Processing count)
- `sgadental.ws2.content-asset-approved` (Approved count)
- `sgadental.ws2.content-asset-published` (Published count)
- "In Review" count derived from assets created but not yet approved/rejected

**Pipeline Stage Indicators**:
- Each stage shows a count and a health indicator
- Health indicator uses icon + text (not color alone): checkmark "OK", warning triangle "Slow", alert circle "Blocked"
- "In Review" stage shows average review time. If >3 business days, it flags as "Slow"
- If any stage has assets stuck >5 days, a warning banner appears at the bottom

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Hover on stage | Tooltip shows: "38 assets in AI processing. Average processing time: 4.2 minutes. 0 failures in last 24h" |
| Click on stage | Expands to show per-practice breakdown of that stage (which practices have assets in this stage) |
| Click "View in Content Engine" | Opens Content Engine Pipeline view in a new tab. This is a documented exception to the "no cross-linking" rule -- it is an administrative diagnostic action, not a daily workflow |
| Time range selector | Dropdown: Last 24h, Last 7 days (default), Last 30 days. Changes all stage counts |

**Alert Integration**: If the Content Engine pipeline is significantly backed up (>20 assets in review for >3 days), the AI Analysis Engine generates an alert that appears in the AI Alerts Panel: "Content pipeline bottleneck: 23 assets awaiting review for 4+ days across 12 practices. Storefront freshness scores may degrade."

### 1.5 Portfolio Heatmap (NI-2)

**Purpose**: Geographic visualization of practice performance across the SGA network.

**Base View**: Map of practice locations with circular markers. Default zoom level fits all practices. Clusters when zoomed out; individual markers when zoomed in.

**WCAG 2.1 AA Compliance (Carry-Forward Item)**:

The heatmap must not rely on color alone to convey performance data. Implementation:

| Indicator | Color | Shape Overlay | Text Label |
|-----------|-------|---------------|------------|
| High performing (>80 composite) | `status-success` (green) | Checkmark icon inside marker | Score number visible at zoom level 3+ |
| Moderate (60-80) | `status-warning` (amber) | Dash icon inside marker | Score number visible at zoom level 3+ |
| Low performing (<60) | `status-critical` (red) | Alert triangle icon inside marker | Score number visible at zoom level 3+ |
| No data | `surface-muted` (gray) | Question mark icon inside marker | "No data" on hover |

**Alternative View Toggle**: A button group above the map offers:
- "Map View" (default): Geographic heatmap
- "Grid View": Practices arranged in a sortable data table with colored status badges (icon + color + text label)
- "Chart View": Practices as a horizontal bar chart sorted by selected metric

This ensures users who cannot distinguish colors can use the Grid or Chart views for full data access.

**Color Layer Selector**:
```
Color by: [ Composite Health ▾ ]
Options:
  - Composite Health (default)
  - Funnel Performance
  - H-Score (from Culture OS)
  - Content Activity (from Content Engine)
  - Show Rate
  - Treatment Acceptance
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Hover on marker | Tooltip: Practice name, selected metric value, trend arrow, practice type badge |
| Click marker | Side panel slides in from right (480px width) with Practice Deep Dive summary. Panel does not navigate away from the map |
| Click cluster | Map zooms in to show individual markers within that cluster |
| Drag/zoom | Standard map interactions. Map state persists in URL query params for shareability |
| Filter bar | Above map: filter by Region, Practice Type, Location Type, Acquisition Date. Filters dim non-matching markers to 20% opacity rather than hiding them |
| Region click | Clicking a region label on the map filters to that region and auto-zooms |

**Loading State**: Map background loads first (gray tiles). Markers appear with a staggered fade-in (50ms delay between each, starting from center).

**Error State**: If marker data fails to load: map renders with a banner: "Practice data temporarily unavailable. Showing last known positions." Markers display with `surface-muted` color and cached scores.

### 1.6 Intervention Queue (NI-3)

**Purpose**: AI-ranked list of practices needing attention, sorted by estimated revenue impact.

**Layout**: Full-width sortable list, 14 items per page (matching the "Top 14" HQ standup pattern).

**Card Anatomy**:
```
┌──────────────────────────────────────────────────────────────────┐
│  #1  Smile Center Buckhead              Southeast | GP | Active  │
│                                                                  │
│  Primary Issue: Show rate dropped 4.2% WoW                      │
│  Root Cause (AI): New front desk hire — Cinnamon phone scores    │
│  22% below practice average. No CCA training started.            │
│                                                                  │
│  Est. Weekly Impact: -$3,200    Days Flagged: 5                  │
│  Assigned: Unassigned                                            │
│                                                                  │
│  [Assign to Me]  [View Practice]  [Dismiss with Note]            │
└──────────────────────────────────────────────────────────────────┘
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| "Assign to Me" | Button changes to "Assigned to [Name]" with user avatar. Assignment emits a notification to the assignee. Timestamp recorded |
| "View Practice" | Navigates to Practice Deep Dive (NI-5) for that practice |
| "Dismiss with Note" | Opens a text input modal. Required: dismissal reason (dropdown: "Resolved", "Known issue", "Low priority", "Duplicate"). Optional: free-text note. Dismissed items move to a "Resolved" tab |
| Sort | Column headers for Impact, Days Flagged, Region, Practice Type are sortable. Default sort: Impact descending |
| Filter | Filter bar: Region, Practice Type, Assigned/Unassigned, Severity |
| Empty state | "No practices need intervention. The network is healthy." with a subtle illustration |

### 1.7 Practice Deep Dive (NI-5)

**Purpose**: Single-practice 360-degree view with data from all workstreams.

**Tab Structure**:

| Tab | Data Sources | Key Metrics |
|-----|-------------|-------------|
| Performance | ClickHouse funnel data | 5-stage funnel visualization, conversion rates, revenue trend, benchmarks vs cohort |
| Communications | ClickHouse comms events | Message volume, response rates, automation performance, Voice AI usage, channel mix |
| Digital Presence | Asset Registry, Channel Sync | Website health score, GMB status, directory sync, recent content, brand health score (WS2) |
| Culture | Culture OS via Kafka | H-Score composite + sub-scores, CCA certification status, huddle completion, trust transfer, champion activity |
| Content Pipeline | Content Engine via Kafka | Assets in pipeline by stage, capture frequency, champion content scorecard, time-in-review metrics |
| History | Practice Data Hub | Enrollment date, key events log, intervention history |

**Tab Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Tab click | Content area transitions with a 200ms horizontal slide. Active tab gets `accent-primary` underline. URL updates: `/practices/:id/performance` |
| Tab loading | Each tab loads independently. A tab's content shows a skeleton loader until its data resolves. Other tabs remain clickable |
| Tab error | Failed tab shows inline error with retry button. Other tabs are unaffected |
| Tab deep link | Direct URL to any tab is supported. Deep links from AI Alerts land on the relevant tab |

**Cross-Workstream Data Presentation**:

Culture Tab and Content Pipeline Tab display data sourced from WS3 and WS2 respectively. These tabs must:
- Use identical visual patterns as the Performance and Communications tabs (same chart components, same card styles, same typography)
- Show a subtle "Last updated" timestamp if data is >24h old
- Degrade gracefully: if Culture OS data is unavailable, the Culture tab shows "Culture data is temporarily unavailable. Last known H-Score: 76 (updated 3 hours ago)"
- Never expose workstream names ("Culture OS", "Content Engine") to the user. Labels use domain language: "Culture & Training", "Content Activity"

### 1.8 Cross-Workstream Navigation

**Primary Navigation** (left sidebar, persistent):

```
Network Overview     ← default landing
Regions
Practices
Operations
  ├── HQ Standup Board
  ├── Enrollment Pipeline
  ├── QC Queues
  └── Playbook Manager
Analytics
Compliance
Settings
```

**Navigation Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Click nav item | Content area transitions. Active item gets `accent-primary` left border and `surface-active` background |
| Collapse sidebar | Click hamburger icon or press `[` key. Sidebar collapses to 64px icon-only mode. Content area expands to fill space |
| Breadcrumb | Appears below top bar for drilled views: "Network > Southeast > Smile Center Buckhead > Performance". Each segment is clickable |
| Keyboard | Arrow keys navigate sidebar items. Enter activates. Tab moves to content area |

**Global Search** (top bar):

| Interaction | Behavior |
|-------------|----------|
| Click search / press `/` | Search input expands. Focus moves to input field |
| Type query | Live results appear in dropdown after 2+ characters. Results grouped: Practices, Regions, Alerts, Staff |
| Select result | Navigates to the relevant view (practice deep dive, region view, alert detail) |
| Escape | Closes search dropdown, returns focus to previous element |

### 1.9 State Management

**Dashboard Auto-Refresh**:
- KPI bar: 15-minute auto-refresh cycle. A subtle countdown indicator in the top bar shows time until next refresh. Manual refresh via a refresh icon button
- AI Alerts: Server-Sent Events (SSE) for real-time push when new alerts are generated. Fallback: 5-minute polling
- Heatmap: Refreshes with KPI bar cycle. Map position/zoom state is preserved across refreshes
- Content Pipeline Health: 30-minute refresh cycle (content pipeline moves slower than operations data)

**Loading States** (per component):

| Component | Loading Pattern |
|-----------|----------------|
| KPI cards | Shimmer animation on value and sparkline. Label visible immediately |
| AI Alerts | 3 skeleton cards with pulsing animation |
| Heatmap | Map tiles load first, markers appear with staggered fade-in |
| Trend charts | Axis labels load first, data line animates left-to-right (600ms) |
| Content Pipeline | 5 skeleton stage boxes with shimmer |
| Practice Deep Dive tabs | Tab headers visible immediately. Content area shows skeleton loader per tab |

**Empty States**:

| Component | Empty State |
|-----------|-------------|
| KPI bar | Never empty (always has network data). If no practices active: "No active practices. Begin enrollment to see network metrics." |
| AI Alerts | "No anomalies detected. The network is performing within normal ranges." + subtle illustration |
| Heatmap | "No practices to display. Adjust filters or begin enrollment." |
| Intervention Queue | "No practices need intervention. Network health is strong." |
| Content Pipeline | "No content activity in selected period. Content capture has not started for this network." |

**Error States**:

| Error Type | Behavior |
|------------|----------|
| Component-level API failure | Affected component shows inline error with "Retry" button. Other components continue functioning. Error message: "[Component name] couldn't load. [Retry]" |
| Full page API failure | Full-width banner below top bar: "Some data couldn't load. We're working on it. Last successful update: [timestamp]." Individual components show their last cached values |
| Authentication expired | Modal overlay: "Your session has expired. [Sign in again]". No data visible behind modal |
| Slow connection | If any component takes >5s to load, show: "Loading is slower than usual..." below the skeleton |

**Data-Rich States** (performance at scale):

| Scenario | Handling |
|----------|----------|
| 260+ practice markers on heatmap | Cluster markers at low zoom levels. Individual markers at high zoom. WebGL rendering for smooth pan/zoom with 500+ markers |
| Large trend datasets (90 days, all practices) | Server-side aggregation via ClickHouse. Client receives pre-computed aggregates, not raw events |
| Intervention Queue with 50+ items | Paginated (14 per page). Server-side sort/filter. Total count shown: "Showing 1-14 of 47 practices" |

---

## 2. Filtering and Drill-Down Patterns

### 2.1 Universal Filter Bar

A consistent filter bar pattern used across Network Overview, Intervention Queue, Analytics, and QC Queues.

**Filter Bar Anatomy**:
```
┌──────────────────────────────────────────────────────────────────┐
│  Region: [All ▾]  Type: [All ▾]  Status: [All ▾]  [Clear All]  │
│  + Add Filter                                                    │
└──────────────────────────────────────────────────────────────────┘
```

**Filter Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Click dropdown | Opens filter menu with searchable checkbox list. Multi-select supported |
| Apply filter | Content updates immediately (no "Apply" button). URL updates with filter params. Active filter count badge appears on dropdown: "Region (2)" |
| "+ Add Filter" | Shows additional filter options: Location Type, Acquisition Date Range, Health Band, Champion Status |
| "Clear All" | Resets all filters. Content returns to unfiltered state. Button only visible when filters are active |
| Filter persistence | Filters persist in the URL query string and survive page refresh. Shareable via URL |

### 2.2 Drill-Down Pattern

The dashboard supports a consistent 3-level drill-down: Network > Region > Practice.

**Level 1: Network** (default):
- KPI bar shows network aggregates
- Heatmap shows all practices
- URL: `/dashboard`

**Level 2: Region** (click region on map or sidebar):
- KPI bar recalculates for selected region
- Heatmap zooms to region bounds
- Practice list below shows region's practices ranked
- Breadcrumb: "Network > Southeast"
- URL: `/dashboard/regions/southeast`

**Level 3: Practice** (click practice marker or list item):
- Full Practice Deep Dive (NI-5)
- Breadcrumb: "Network > Southeast > Smile Center Buckhead"
- URL: `/practices/sga-0142/performance`

**Drill-Down Transitions**:
- Level 1 > Level 2: Map zoom animation (600ms ease-in-out). KPI bar values animate from old to new (300ms counter animation). Content below crossfades (200ms)
- Level 2 > Level 3: Side panel slides in from right if triggered from map, or full-page navigation if triggered from list
- Back navigation: Breadcrumb click or browser back button. Map zoom reverses. Filters from previous level are preserved

### 2.3 Time Range Controls

**Global Time Range**: Affects all time-series data on the current view.

```
[ 7d ] [ 30d ] [ 60d ] [ 90d ] [ Custom ]
```

**Interactions**:

| Interaction | Behavior |
|-------------|----------|
| Click preset | All time-series components re-query and re-render. Active button gets filled treatment. 300ms transition |
| Click "Custom" | Date range picker opens. Start date and end date inputs. Max range: 365 days. Calendar visualization with selectable range |
| Period comparison | Toggle switch: "Compare to prior period." When active, all charts show a second data series in a lighter shade |

---

## 3. Responsive Behavior

Network Intelligence is desktop-first. It is not designed for mobile daily use. However, it must be usable on an iPad (1024px+) for regional directors during site visits.

### 3.1 Breakpoint Behavior

| Breakpoint | Layout Changes |
|------------|---------------|
| 1920px+ (large desktop) | Full layout as specified. Heatmap and alerts panel side-by-side |
| 1280-1919px (standard desktop) | Sidebar collapses to icon-only by default. Content area uses full width |
| 1024-1279px (iPad landscape) | KPI bar wraps to 2 rows (3+2 cards). Alerts panel moves below heatmap (full width). Side panel for Practice Deep Dive becomes full-width overlay |
| <1024px (tablet portrait, phone) | Banner: "Network Intelligence is designed for desktop. For mobile access, use the Practice Operations app." Basic KPI read-only view available |

---

*This specification covers NI-1, NI-2, NI-3, and NI-5 interactions at sufficient detail for frontend engineering implementation. All cross-workstream data surfaces use the same visual patterns as native data -- no workstream labels are exposed to users.*
