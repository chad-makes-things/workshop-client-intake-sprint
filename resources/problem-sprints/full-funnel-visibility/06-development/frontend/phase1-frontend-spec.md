# Phase 1 Frontend Specification

**Role**: Frontend Engineer
**Sprint**: Full Funnel Visibility (The Funnel OS)
**Date**: 2026-03-10
**Phase 1 Scope**: Month 1 -- Admin/config interface for 10 pilot practices + reusable component library foundation for Stage 3
**Source Artifacts**: `03-decide/storyboard.md`, `04-architect/architecture.md`, `handoff-to-dev.md`

---

## 1. Application Architecture

Phase 1 delivers one web application with a reusable component library, both from the monorepo shared with the Digital Storefront sprint (Practice Concierge).

| App | Purpose | Users | Framework |
|-----|---------|-------|-----------|
| `apps/funnel-admin` | Admin interface for managing pilot practices, connector status, funnel data review | HQ ops team, tech lead | Next.js 14 (App Router) |

The reusable component library (`packages/funnel-ui`) houses all visualization and data display components that will power the custom Executive Dashboard (Stage 3) and Practice Nerve Center (Stage 3). Phase 1 builds and tests these components in the admin interface; Stage 3 promotes them to production-facing surfaces.

Shared dependencies with Digital Storefront sprint:
- `packages/ui` -- base component library (buttons, inputs, cards, badges) from Concierge sprint
- `packages/shared` -- shared TypeScript types and utilities
- `packages/funnel-ui` -- NEW: funnel-specific visualization components (this spec)

---

## 2. Component Tree

### 2.1 Admin Interface (`apps/funnel-admin`)

```
FunnelAdminApp
├── AuthLayout                              # Login page for HQ team
│   └── LoginForm                           # Email + password (shared with Concierge dashboard)
├── AdminLayout                             # Authenticated shell with nav
│   ├── TopNav                              # Funnel OS logo, user menu, system status
│   │   ├── SystemStatusBadge              # Overall system health (all connectors OK / errors)
│   │   ├── NotificationBell               # Connector alerts, data pipeline warnings
│   │   └── UserMenu                       # Profile, logout
│   ├── Sidebar                            # Navigation
│   │   ├── NavItem                        # Practices, Connectors, Data Quality, Component Library
│   │   └── EnvironmentBadge              # "Pilot" badge (10 practices)
│   │
│   ├── PracticeListPage                   # Overview of all 10 pilot practices
│   │   ├── PilotSummaryBar               # Quick stats: total practices, connected, errors, last sync
│   │   ├── PracticeGrid                  # Card grid (not table -- 10 practices fits in cards)
│   │   │   └── PracticeCard              # [REUSABLE] Summary card per practice
│   │   │       ├── PracticeHeader        # Name, location, specialty pill
│   │   │       ├── ConnectorStatusRow    # Icons for each connector type with status dots
│   │   │       ├── FunnelMiniChart       # [REUSABLE] Compact 5-stage funnel bars
│   │   │       ├── HealthIndicator       # [REUSABLE] Green/yellow/red with trend arrow
│   │   │       └── LastSyncTimestamp     # "Last sync: 12 min ago"
│   │   └── AddPracticeButton             # Opens practice enrollment form
│   │
│   ├── PracticeDetailPage                 # Deep dive on one practice
│   │   ├── PracticeDetailHeader          # Name, location, overall health, last sync
│   │   │   └── HealthIndicator           # [REUSABLE]
│   │   ├── TabNav                        # Funnel Data, Connectors, Alerts, Raw Events
│   │   ├── FunnelDataTab
│   │   │   ├── FunnelChart               # [REUSABLE] Full 5-stage funnel visualization
│   │   │   ├── ConversionRateRow         # Stage-to-stage conversion percentages
│   │   │   ├── TrendLineGrid            # [REUSABLE] 5 sparklines, one per funnel stage
│   │   │   │   └── TrendLine            # [REUSABLE] Week-over-week sparkline
│   │   │   └── TimeRangeToggle          # 7d / 30d / 90d
│   │   ├── ConnectorsTab
│   │   │   ├── ConnectorList            # All connectors for this practice
│   │   │   │   └── ConnectorCard        # Per-connector status and config
│   │   │   │       ├── ConnectorHeader  # Platform name, icon, status badge
│   │   │   │       ├── ConnectorMetrics # Records synced, last sync, error count
│   │   │   │       ├── ConfigPanel      # API key (masked), polling frequency, toggle
│   │   │   │       └── SyncButton       # Manual "Sync Now" trigger
│   │   │   └── AddConnectorButton       # Opens connector configuration wizard
│   │   ├── AlertsTab
│   │   │   ├── AlertList                # Alerts for this practice
│   │   │   │   └── AlertCard            # [REUSABLE] AI alert with severity + action
│   │   │   └── AlertFilters             # Severity, type, acknowledged/unacknowledged
│   │   └── RawEventsTab
│   │       ├── EventTable               # Paginated table of raw funnel events
│   │       │   └── EventRow             # Timestamp, stage, source, handler, value
│   │       ├── EventFilters             # Stage, source, date range
│   │       └── Pagination               # Page controls
│   │
│   ├── ConnectorManagementPage            # Global connector overview
│   │   ├── ConnectorTypeGrid             # Cards per connector type (PMS, Phone, AI Agent, Ads)
│   │   │   └── ConnectorTypeCard         # Type name, icon, practices connected, error count
│   │   ├── ConnectorConfigWizard         # Step-by-step connector setup
│   │   │   ├── SelectPlatformStep       # Choose platform (Dentrix, Eaglesoft, etc.)
│   │   │   ├── CredentialsStep          # API key / webhook URL / DB connection input
│   │   │   ├── MappingStep             # Map source fields to funnel schema (auto-suggested)
│   │   │   ├── ScheduleStep            # Polling frequency, sync window
│   │   │   └── TestConnectionStep      # Validate connection, show sample data
│   │   └── ConnectorHealthTable         # All connectors across all practices
│   │       └── ConnectorRow             # Practice, platform, status, last sync, records, errors
│   │
│   ├── DataQualityPage                    # Data pipeline health monitoring
│   │   ├── PipelineSummary              # Events processed (24h), error rate, avg latency
│   │   ├── StageDistributionChart       # Bar chart: events per funnel stage
│   │   ├── DataGapAlerts                # Missing data warnings (e.g., "No phone data from Practice X in 48h")
│   │   └── RecentErrorLog              # Last 50 pipeline errors with stack traces
│   │
│   └── ComponentLibraryPage               # Living preview of all reusable components
│       ├── FunnelChartDemo              # Interactive FunnelChart with mock data controls
│       ├── HealthIndicatorDemo          # All states: green, yellow, red, with/without trend
│       ├── PracticeCardDemo             # PracticeCard with different data scenarios
│       ├── AlertCardDemo                # Alert card at each severity level
│       ├── ActionCardDemo               # [STAGE 3] Preview of Nerve Center action cards
│       ├── TrendLineDemo               # Sparklines with various data patterns
│       └── DrillDownNavDemo            # Breadcrumb navigation demo
│
└── ErrorBoundary
    └── ErrorFallback                     # "Something went wrong" with retry
```

### 2.2 Reusable Component Library (`packages/funnel-ui`)

```
packages/funnel-ui/
├── FunnelChart                            # 5-stage funnel visualization
├── HealthIndicator                        # Green/yellow/red status with trend
├── PracticeCard                           # Practice summary card with key metrics
├── AlertCard                              # AI alert with severity and action
├── ActionCard                             # [STAGE 3] Practice Nerve Center action card
├── TrendLine                              # Sparkline/trend chart
├── DrillDownNav                           # Breadcrumb-style hierarchical navigation
├── ConnectorStatusDot                     # Small status indicator for connector health
├── FunnelMiniChart                        # Compact version of FunnelChart for cards
├── MetricCard                             # Single metric with label, value, trend
├── TimeRangeToggle                        # 7d / 30d / 90d selector
└── SeverityBadge                          # Alert severity pill (critical, warning, info)
```

---

## 3. Routes and Page Structure

### Next.js App Router File Structure

```
apps/funnel-admin/
├── app/
│   ├── layout.tsx                        # Root layout (providers, fonts, metadata)
│   ├── loading.tsx                       # Root loading state (full-page skeleton)
│   ├── error.tsx                         # Root error boundary
│   ├── not-found.tsx                     # 404 page
│   ├── login/
│   │   └── page.tsx                      # Login page
│   ├── (authenticated)/
│   │   ├── layout.tsx                    # AdminLayout (sidebar + topnav, auth check)
│   │   ├── page.tsx                      # Redirects to /practices
│   │   ├── practices/
│   │   │   ├── page.tsx                  # PracticeListPage
│   │   │   ├── loading.tsx              # Practice grid skeleton
│   │   │   └── [practiceId]/
│   │   │       ├── page.tsx             # PracticeDetailPage (default: Funnel Data tab)
│   │   │       ├── loading.tsx          # Practice detail skeleton
│   │   │       ├── connectors/
│   │   │       │   └── page.tsx         # ConnectorsTab
│   │   │       ├── alerts/
│   │   │       │   └── page.tsx         # AlertsTab
│   │   │       ├── events/
│   │   │       │   └── page.tsx         # RawEventsTab
│   │   │       └── connectors/
│   │   │           └── new/
│   │   │               └── page.tsx     # AddConnector wizard for this practice
│   │   ├── connectors/
│   │   │   ├── page.tsx                 # ConnectorManagementPage
│   │   │   ├── loading.tsx
│   │   │   └── new/
│   │   │       └── page.tsx             # ConnectorConfigWizard (global)
│   │   ├── data-quality/
│   │   │   ├── page.tsx                 # DataQualityPage
│   │   │   └── loading.tsx
│   │   └── components/
│   │       └── page.tsx                 # ComponentLibraryPage (dev-only in prod)
│   └── api/
│       └── auth/
│           └── [...nextauth]/
│               └── route.ts             # NextAuth route handler
```

### Route Table

| Route | Page | Description |
|-------|------|-------------|
| `/login` | `LoginPage` | HQ team authentication |
| `/` | Redirect | Redirects to `/practices` |
| `/practices` | `PracticeListPage` | Grid of 10 pilot practices with status |
| `/practices/:id` | `PracticeDetailPage` | Practice detail, defaults to funnel data tab |
| `/practices/:id/connectors` | `ConnectorsTab` | Connector configuration for practice |
| `/practices/:id/alerts` | `AlertsTab` | Practice-specific alerts |
| `/practices/:id/events` | `RawEventsTab` | Raw funnel event log |
| `/practices/:id/connectors/new` | `AddConnector` | Add connector wizard scoped to practice |
| `/connectors` | `ConnectorManagementPage` | Global connector health overview |
| `/connectors/new` | `ConnectorConfigWizard` | Global connector setup wizard |
| `/data-quality` | `DataQualityPage` | Pipeline health monitoring |
| `/components` | `ComponentLibraryPage` | Living component library preview (dev only) |

### Loading States

Every route group has a `loading.tsx` that renders a skeleton matching the page layout. Skeletons use the same grid/card structure as the actual content with pulsing gray rectangles (`bg-slate-800 animate-pulse`).

---

## 4. Reusable Component Library Specifications

### 4.1 FunnelChart

Full 5-stage funnel visualization showing the patient journey with volume bars, conversion rates, and color-coded health.

```typescript
interface FunnelChartProps {
  data: FunnelStageData[];
  orientation?: 'horizontal' | 'vertical';   // horizontal for dashboard, vertical for detail
  showConversionRates?: boolean;               // show % between stages
  showTrend?: boolean;                         // show week-over-week change arrows
  highlightStage?: FunnelStage;                // highlight a specific stage
  onStageClick?: (stage: FunnelStage) => void; // drill-down handler
  timeRange?: '7d' | '30d' | '90d';
  size?: 'compact' | 'default' | 'large';     // compact for cards, large for detail pages
  className?: string;
}

interface FunnelStageData {
  stage: FunnelStage;
  label: string;                               // "Leads Generated", "Connected", etc.
  count: number;                               // absolute count
  previousCount?: number;                      // prior period for trend calculation
  conversionRate?: number;                     // % from previous stage (null for first stage)
  health: 'good' | 'watch' | 'alert';         // determines bar color
  benchmark?: number;                          // baseline value for comparison
}

type FunnelStage =
  | 'lead_generated'
  | 'lead_connected'
  | 'lead_booked'
  | 'patient_showed'
  | 'production_completed';
```

**Visual Design**:
- Horizontal bars, widest at top (lead_generated), narrowing down to production_completed
- Bar color: `#10b981` (good), `#f59e0b` (watch), `#ef4444` (alert)
- Bar background: `#1e293b` (the unfilled portion showing capacity/benchmark)
- Conversion rate labels between bars: "75% connected" with a downward arrow connector
- Trend arrows to the right of each bar: green up arrow or red down arrow with percentage
- Built with Recharts `BarChart` in horizontal layout, custom bar shapes for the funnel taper

**States**:
- Loading: 5 skeleton bars with pulse animation
- Empty: "No funnel data available" with illustration
- Error: "Failed to load funnel data" with retry button
- Partial: Some stages have data, others show "Awaiting data" in muted text

**Responsive**:
- Desktop (lg+): Full horizontal funnel, 600px wide, labels inline
- Tablet (md): Full horizontal funnel, labels below bars
- Mobile (sm): Switches to vertical stack, each stage is a full-width row

### 4.2 HealthIndicator

A compact status indicator showing health state with optional trend direction.

```typescript
interface HealthIndicatorProps {
  status: 'good' | 'watch' | 'alert' | 'unknown';
  label?: string;                              // e.g., "Phone Answer Rate"
  value?: string | number;                     // e.g., "84%" or 84
  trend?: 'up' | 'down' | 'flat';
  trendValue?: string;                         // e.g., "+3%" or "-12%"
  size?: 'sm' | 'md' | 'lg';                  // sm for inline, lg for cards
  showPulse?: boolean;                         // animated pulse on alert status
  className?: string;
}
```

**Visual Design**:
- Circle indicator: 8px (sm), 12px (md), 16px (lg)
- Colors: `#10b981` (good), `#f59e0b` (watch), `#ef4444` (alert), `#64748b` (unknown)
- Trend arrow: rotated chevron icon, colored green (up = good) or red (down = bad)
- Note: trend direction meaning is context-dependent -- for some metrics, "up" is bad (e.g., no-show rate). The parent component sets the color via `trend` + `status`, not the HealthIndicator itself.
- Alert status with `showPulse`: concentric ring animation (subtle, 2s cycle)

**States**:
- `good`: solid green circle, optional "On Track" label
- `watch`: solid yellow circle, optional "Watch" label
- `alert`: solid red circle with optional pulse, optional "Needs Attention" label
- `unknown`: gray circle, "No Data" label

**Responsive**: Scales via `size` prop, no breakpoint-specific behavior.

### 4.3 PracticeCard

Summary card for a practice showing identity, connector health, funnel snapshot, and overall status.

```typescript
interface PracticeCardProps {
  practice: PracticeSummary;
  onSelect?: (practiceId: string) => void;     // card click handler
  showConnectors?: boolean;                    // show connector status row
  showFunnel?: boolean;                        // show mini funnel chart
  variant?: 'default' | 'compact';            // compact for lists
  className?: string;
}

interface PracticeSummary {
  id: string;
  name: string;
  location: string;                            // "Woodstock, GA"
  specialty: string;                           // "General", "Perio", "Oral Surgery"
  healthStatus: 'good' | 'watch' | 'alert' | 'unknown';
  connectors: ConnectorStatus[];
  funnelSnapshot?: FunnelStageData[];          // latest funnel data for mini chart
  lastSyncAt?: string;                         // ISO timestamp
  totalLeads7d?: number;
  totalProduction7d?: number;
}

interface ConnectorStatus {
  type: 'pms' | 'phone' | 'ai_agent' | 'ads' | 'hr';
  platform: string;                            // "Dentrix", "RingCentral", etc.
  status: 'connected' | 'disconnected' | 'error' | 'syncing';
  lastSyncAt?: string;
  errorMessage?: string;
}
```

**Visual Design**:
- Card background: `#111827`, border: `1px solid #1e293b`, border-radius: 12px
- Hover: border transitions to `#334155`, subtle shadow, cursor pointer
- Left border accent: 3px solid, color matches `healthStatus`
- Header: Practice name (text-lg, `#f1f5f9`), location (text-sm, `#94a3b8`)
- Specialty pill: small rounded badge with muted background
- Connector row: row of 5 small icons (PMS, Phone, AI, Ads, HR) with `ConnectorStatusDot` under each
- Mini funnel: compact 5-bar visualization (60px tall), no labels
- Footer: "Last sync: 12 min ago" in `#64748b`

**States**:
- Default: all data populated
- No connectors: "No connectors configured" with setup CTA
- Error: red left border, error icon in connector row, "X connectors have errors" warning
- Loading: skeleton card matching layout

**Responsive**:
- Desktop: 320px wide cards in a responsive grid (auto-fill, minmax 300px)
- Tablet: 2 columns
- Mobile: single column, full width

### 4.4 AlertCard

An AI-generated alert showing what happened, why it matters, and what to do about it.

```typescript
interface AlertCardProps {
  alert: FunnelAlert;
  onAcknowledge?: (alertId: string) => void;
  onDismiss?: (alertId: string) => void;
  onActionClick?: (alertId: string, action: RecommendedAction) => void;
  expanded?: boolean;
  className?: string;
}

interface FunnelAlert {
  id: string;
  practiceId: string;
  practiceName: string;
  severity: 'critical' | 'warning' | 'info';
  category: 'phone' | 'show_rate' | 'production' | 'leads' | 'staffing' | 'automation';
  title: string;                               // "Phone answer rate dropped 22%"
  description: string;                         // AI-generated explanation with root cause
  detectedAt: string;                          // ISO timestamp
  funnel_stage: FunnelStage;                   // which funnel stage is affected
  metric: {
    name: string;                              // "Phone Answer Rate"
    current: number;                           // 62
    baseline: number;                          // 84
    unit: '%' | '$' | 'count';
  };
  rootCause?: string;                          // "Front desk headcount dropped from 3 to 2"
  revenueImpact?: {
    estimated: number;                         // estimated weekly revenue at risk
    confidence: 'high' | 'medium' | 'low';
  };
  recommendedAction?: RecommendedAction;
  acknowledged: boolean;
  acknowledgedBy?: string;
  acknowledgedAt?: string;
}

interface RecommendedAction {
  id: string;
  description: string;                         // "Activate AI phone agent overflow"
  owner: 'practice_staff' | 'hq_ops' | 'automation';
  estimatedImpact: string;                     // "Could recover ~$2,400/week"
  steps?: string[];                            // step-by-step instructions
}
```

**Visual Design**:
- Card background: `#111827`, border-radius: 12px
- Left border: 4px solid, color by severity: `#ef4444` (critical), `#f59e0b` (warning), `#3b82f6` (info)
- Header row: severity badge (colored pill), title (bold, `#f1f5f9`), timestamp (muted)
- Practice name as a subtle tag below the header
- Metric display: current value vs baseline, with delta highlighted in red/green
- Description: `#94a3b8` text, expandable if longer than 2 lines
- Root cause callout: indented block with `#1e293b` background, "Why: " prefix in bold
- Recommended action: bordered section at bottom with action description, owner badge, impact estimate
- Action buttons: "Acknowledge" (primary), "Dismiss" (ghost), "Take Action" (if recommendedAction exists)

**States**:
- Unacknowledged: normal display, left border fully opaque
- Acknowledged: left border at 50% opacity, "Acknowledged by [name] at [time]" footer
- Dismissed: hidden (filtered out)
- Expanded: shows full description, root cause, recommended action steps
- Collapsed: shows title, severity, practice name, timestamp only

**Responsive**:
- Desktop: max-width 700px, inline action buttons
- Tablet: full width, inline action buttons
- Mobile: full width, stacked action buttons

### 4.5 ActionCard (Stage 3 -- Spec Now, Build Later)

Practice Nerve Center action card. Each card represents one prioritized task for practice staff.

```typescript
interface ActionCardProps {
  action: NerveAction;
  onComplete?: (actionId: string, outcome: ActionOutcome) => void;
  onDefer?: (actionId: string, reason: string) => void;
  onExpand?: (actionId: string) => void;
  position?: number;                           // 1, 2, 3 -- priority rank
  className?: string;
}

interface NerveAction {
  id: string;
  practiceId: string;
  type: 'callback' | 'confirm' | 'followup' | 'review' | 'schedule';
  priority: 1 | 2 | 3 | 4 | 5;
  title: string;                               // "Call back 4 missed patients"
  description: string;                         // AI-generated context
  estimatedImpact: {
    revenue: number;                           // $1,200
    metric: string;                            // "2 new patients"
  };
  estimatedDuration: number;                   // minutes to complete
  items: ActionItem[];                         // individual tasks within this action
  script?: string;                             // suggested phone/text script
  expiresAt?: string;                          // time-sensitive actions
  createdAt: string;
  completedAt?: string;
}

interface ActionItem {
  id: string;
  type: 'call' | 'text' | 'email' | 'review' | 'schedule';
  patientName?: string;
  patientPhone?: string;
  appointmentId?: string;
  appointmentType?: string;
  appointmentValue?: number;
  status: 'pending' | 'completed' | 'skipped' | 'failed';
  completedAt?: string;
  outcome?: string;
}

interface ActionOutcome {
  itemsCompleted: number;
  itemsSkipped: number;
  notes?: string;
  appointmentsBooked?: number;
  revenueGenerated?: number;
}
```

**Visual Design** (Stage 3 implementation, spec for design consistency):
- Card: `#111827` background, border-radius: 16px (slightly larger for touch targets)
- Priority number: large circle (48px) with rank number, color gradient by priority
- Icon by type: phone icon (callback), check icon (confirm), clipboard (followup), chart (review), calendar (schedule)
- Title: text-lg bold, `#f1f5f9`
- Impact badge: green pill showing "$1,200 | 2 new patients"
- Duration badge: clock icon + "~8 min"
- Expandable item list: each ActionItem as a row with checkbox, patient name, one-tap action
- One-tap actions: phone icon dials number, message icon opens pre-filled SMS
- Script section: collapsible, shows suggested script text in a styled blockquote
- Progress bar: fills as items are completed (e.g., 2/4 done)
- "Done" button: marks the full action as complete, prompts for outcome

**States**:
- Pending: full color, actionable
- In progress: progress bar visible, some items checked
- Completed: green border, check icon, shows outcome summary
- Deferred: muted, moved to bottom of list, "Deferred: [reason]" tag
- Expired: red border, "This action has expired" overlay

**Responsive** (tablet-first for Nerve Center):
- iPad (md): full-width cards, stacked, large touch targets (56px tap zones)
- Phone (sm): full-width, simplified layout, essential info only
- Desktop (lg+): max-width 500px, centered content area

### 4.6 TrendLine

Compact sparkline chart showing week-over-week data for a single metric.

```typescript
interface TrendLineProps {
  data: TrendDataPoint[];
  color?: string;                              // line color, defaults to accent blue
  height?: number;                             // chart height in px, default 40
  width?: number | '100%';                     // chart width, default '100%'
  showArea?: boolean;                          // filled area under line
  showTooltip?: boolean;                       // hover tooltip with value
  showEndpoint?: boolean;                      // dot on the last data point
  referenceValue?: number;                     // horizontal reference line (baseline)
  trend?: 'up' | 'down' | 'flat';             // determines endpoint dot color
  animate?: boolean;                           // line draw animation on mount
  className?: string;
}

interface TrendDataPoint {
  date: string;                                // ISO date string or label
  value: number;
  label?: string;                              // tooltip label override
}
```

**Visual Design**:
- Line: 2px stroke, smooth curve (monotoneX interpolation)
- Default color: `#3b82f6` (accent blue)
- Area fill: gradient from line color at 20% opacity to transparent
- Reference line: 1px dashed `#334155`
- Endpoint dot: 6px circle, green if trend=up, red if trend=down, blue if flat
- Tooltip: dark tooltip (`#1e293b`) showing date + value on hover
- Built with Recharts `LineChart` in a minimal configuration (no axes, no grid)

**States**:
- Loading: flat gray line with pulse animation
- No data: "---" text centered
- Single point: dot only, no line
- Normal: smooth line with optional area fill

**Responsive**: Scales to container width. Height is fixed via prop.

### 4.7 DrillDownNav

Breadcrumb-style navigation for hierarchical drill-down (Network > Region > Practice > Stage).

```typescript
interface DrillDownNavProps {
  levels: DrillDownLevel[];
  onNavigate: (levelIndex: number) => void;    // navigate to a level
  className?: string;
}

interface DrillDownLevel {
  label: string;                               // "Network", "Southeast", "Towne Lake Dental"
  value?: string;                              // optional subtitle value ("260 practices", "42 practices")
  icon?: React.ReactNode;                      // optional level icon
  active?: boolean;                            // current level (last item)
}
```

**Visual Design**:
- Horizontal breadcrumb with chevron separators
- Inactive levels: `#60a5fa` text (clickable link style)
- Active level: `#f1f5f9` text, bold
- Separator: `#64748b` chevron-right icon (16px)
- Optional value: `#94a3b8` in parentheses after label
- Hover on inactive: underline

**States**:
- Single level: just the label, no separators
- Multi-level: labels connected by chevrons
- Deep hierarchy: on mobile, collapses middle levels to "..." with dropdown

**Responsive**:
- Desktop: full breadcrumb inline
- Mobile: shows only current level + back arrow to previous level (no full breadcrumb)

---

## 5. State Management

### 5.1 Server State: React Query (TanStack Query)

All API data flows through React Query. No server data stored in component state or Zustand.

```typescript
// Query key conventions
const queryKeys = {
  practices: {
    all: ['practices'] as const,
    list: () => ['practices', 'list'] as const,
    detail: (id: string) => ['practices', 'detail', id] as const,
    funnel: (id: string, range: string) => ['practices', id, 'funnel', range] as const,
    events: (id: string, filters: EventFilters) => ['practices', id, 'events', filters] as const,
  },
  connectors: {
    all: ['connectors'] as const,
    byPractice: (practiceId: string) => ['connectors', 'practice', practiceId] as const,
    detail: (connectorId: string) => ['connectors', 'detail', connectorId] as const,
    health: () => ['connectors', 'health'] as const,
  },
  alerts: {
    all: ['alerts'] as const,
    byPractice: (practiceId: string) => ['alerts', 'practice', practiceId] as const,
    active: (filters?: AlertFilters) => ['alerts', 'active', filters] as const,
  },
  dataQuality: {
    summary: () => ['data-quality', 'summary'] as const,
    errors: (limit: number) => ['data-quality', 'errors', limit] as const,
  },
  system: {
    status: () => ['system', 'status'] as const,
  },
} as const;
```

**Query Client Configuration**:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,          // 30 seconds -- connector data changes frequently
      gcTime: 5 * 60_000,         // 5 minutes cache lifetime
      retry: 2,                   // retry failed queries twice
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 10_000),
      refetchOnWindowFocus: true,  // refresh when tab regains focus
      refetchOnReconnect: true,    // refresh when network reconnects
    },
    mutations: {
      retry: 1,
    },
  },
});
```

**Core Hooks**:

```typescript
// Practice list (all 10 pilot practices)
function usePractices() {
  return useQuery({
    queryKey: queryKeys.practices.list(),
    queryFn: () => api.practices.list(),
    staleTime: 60_000,            // practices don't change often
  });
}

// Practice detail with funnel data
function usePractice(practiceId: string) {
  return useQuery({
    queryKey: queryKeys.practices.detail(practiceId),
    queryFn: () => api.practices.get(practiceId),
  });
}

// Funnel data for a practice with time range
function usePracticeFunnel(practiceId: string, range: '7d' | '30d' | '90d') {
  return useQuery({
    queryKey: queryKeys.practices.funnel(practiceId, range),
    queryFn: () => api.practices.getFunnel(practiceId, range),
    staleTime: 15_000,            // funnel data is frequently updated
  });
}

// Connector health across all practices (auto-refresh every 30s)
function useConnectorHealth() {
  return useQuery({
    queryKey: queryKeys.connectors.health(),
    queryFn: () => api.connectors.getHealth(),
    refetchInterval: 30_000,      // poll every 30 seconds for connector status
  });
}

// Connectors for a specific practice
function usePracticeConnectors(practiceId: string) {
  return useQuery({
    queryKey: queryKeys.connectors.byPractice(practiceId),
    queryFn: () => api.connectors.listByPractice(practiceId),
  });
}

// Active alerts with filtering
function useAlerts(filters?: AlertFilters) {
  return useQuery({
    queryKey: queryKeys.alerts.active(filters),
    queryFn: () => api.alerts.list(filters),
    refetchInterval: 60_000,      // check for new alerts every minute
  });
}

// Raw funnel events (paginated)
function usePracticeEvents(practiceId: string, filters: EventFilters) {
  return useQuery({
    queryKey: queryKeys.practices.events(practiceId, filters),
    queryFn: () => api.practices.getEvents(practiceId, filters),
    placeholderData: keepPreviousData,  // smooth pagination transitions
  });
}

// System status (auto-refresh every 15s)
function useSystemStatus() {
  return useQuery({
    queryKey: queryKeys.system.status(),
    queryFn: () => api.system.getStatus(),
    refetchInterval: 15_000,
  });
}
```

**Mutations with Optimistic Updates**:

```typescript
// Update connector configuration
function useUpdateConnector(connectorId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (config: ConnectorConfig) =>
      api.connectors.update(connectorId, config),
    onMutate: async (config) => {
      await queryClient.cancelQueries({
        queryKey: queryKeys.connectors.detail(connectorId),
      });
      const previous = queryClient.getQueryData(
        queryKeys.connectors.detail(connectorId)
      );
      queryClient.setQueryData(
        queryKeys.connectors.detail(connectorId),
        (old: Connector) => ({ ...old, config: { ...old.config, ...config } })
      );
      return { previous };
    },
    onError: (_err, _config, context) => {
      queryClient.setQueryData(
        queryKeys.connectors.detail(connectorId),
        context?.previous
      );
      toast.error('Failed to update connector configuration');
    },
    onSettled: () => {
      queryClient.invalidateQueries({
        queryKey: queryKeys.connectors.detail(connectorId),
      });
      queryClient.invalidateQueries({
        queryKey: queryKeys.connectors.health(),
      });
    },
    onSuccess: () => {
      toast.success('Connector configuration updated');
    },
  });
}

// Trigger manual sync
function useTriggerSync(connectorId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: () => api.connectors.triggerSync(connectorId),
    onMutate: async () => {
      // Optimistically set status to "syncing"
      queryClient.setQueryData(
        queryKeys.connectors.detail(connectorId),
        (old: Connector) => ({ ...old, status: 'syncing' })
      );
    },
    onSettled: () => {
      // Refetch after a delay to let sync complete
      setTimeout(() => {
        queryClient.invalidateQueries({
          queryKey: queryKeys.connectors.detail(connectorId),
        });
      }, 5000);
    },
  });
}

// Acknowledge alert
function useAcknowledgeAlert() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (alertId: string) => api.alerts.acknowledge(alertId),
    onMutate: async (alertId) => {
      await queryClient.cancelQueries({ queryKey: queryKeys.alerts.all });
      // Optimistically mark as acknowledged in all alert queries
      queryClient.setQueriesData(
        { queryKey: queryKeys.alerts.all },
        (old: FunnelAlert[] | undefined) =>
          old?.map((a) =>
            a.id === alertId
              ? { ...a, acknowledged: true, acknowledgedAt: new Date().toISOString() }
              : a
          )
      );
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.alerts.all });
    },
  });
}
```

### 5.2 Client State: Zustand

Minimal client-only state for UI concerns.

```typescript
interface AdminUIState {
  // Sidebar
  sidebarCollapsed: boolean;
  toggleSidebar: () => void;

  // Practice detail active tab (synced with URL, but cached for back navigation)
  lastActiveTab: Record<string, string>;  // practiceId -> tab name
  setLastActiveTab: (practiceId: string, tab: string) => void;

  // Connector config wizard
  wizardStep: number;
  wizardData: Partial<ConnectorConfig>;
  setWizardStep: (step: number) => void;
  updateWizardData: (data: Partial<ConnectorConfig>) => void;
  resetWizard: () => void;

  // Data quality page filters
  dataQualityTimeRange: '1h' | '24h' | '7d';
  setDataQualityTimeRange: (range: '1h' | '24h' | '7d') => void;
}

const useAdminUI = create<AdminUIState>((set) => ({
  sidebarCollapsed: false,
  toggleSidebar: () => set((s) => ({ sidebarCollapsed: !s.sidebarCollapsed })),

  lastActiveTab: {},
  setLastActiveTab: (practiceId, tab) =>
    set((s) => ({ lastActiveTab: { ...s.lastActiveTab, [practiceId]: tab } })),

  wizardStep: 0,
  wizardData: {},
  setWizardStep: (step) => set({ wizardStep: step }),
  updateWizardData: (data) =>
    set((s) => ({ wizardData: { ...s.wizardData, ...data } })),
  resetWizard: () => set({ wizardStep: 0, wizardData: {} }),

  dataQualityTimeRange: '24h',
  setDataQualityTimeRange: (range) => set({ dataQualityTimeRange: range }),
}));
```

---

## 6. API Integration

### 6.1 API Client Setup

The frontend communicates with the FastAPI backend. All API calls go through a typed client layer.

```typescript
// lib/api-client.ts
const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000/api/v1';

class ApiError extends Error {
  constructor(
    public status: number,
    public statusText: string,
    public body: unknown,
  ) {
    super(`API Error ${status}: ${statusText}`);
    this.name = 'ApiError';
  }
}

async function apiRequest<T>(
  path: string,
  options: RequestInit = {},
): Promise<T> {
  const url = `${API_BASE}${path}`;
  const headers: Record<string, string> = {
    'Content-Type': 'application/json',
    ...getAuthHeaders(),
    ...(options.headers as Record<string, string>),
  };

  const response = await fetch(url, {
    ...options,
    headers,
  });

  if (!response.ok) {
    const body = await response.json().catch(() => null);
    throw new ApiError(response.status, response.statusText, body);
  }

  // Handle 204 No Content
  if (response.status === 204) return undefined as T;

  return response.json();
}

function getAuthHeaders(): Record<string, string> {
  const token = getStoredToken();
  return token ? { Authorization: `Bearer ${token}` } : {};
}
```

### 6.2 API Endpoint Map

```typescript
const api = {
  // Authentication
  auth: {
    login: (email: string, password: string) =>
      apiRequest<AuthResponse>('/auth/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      }),
    refresh: () => apiRequest<AuthResponse>('/auth/refresh', { method: 'POST' }),
    logout: () => apiRequest<void>('/auth/logout', { method: 'POST' }),
  },

  // Practices
  practices: {
    list: () =>
      apiRequest<PracticeSummary[]>('/practices'),
    get: (id: string) =>
      apiRequest<PracticeDetail>(`/practices/${id}`),
    create: (data: CreatePracticeInput) =>
      apiRequest<PracticeDetail>('/practices', {
        method: 'POST',
        body: JSON.stringify(data),
      }),
    update: (id: string, data: Partial<PracticeDetail>) =>
      apiRequest<PracticeDetail>(`/practices/${id}`, {
        method: 'PATCH',
        body: JSON.stringify(data),
      }),
    getFunnel: (id: string, range: string) =>
      apiRequest<FunnelData>(`/practices/${id}/funnel?range=${range}`),
    getEvents: (id: string, filters: EventFilters) =>
      apiRequest<PaginatedResponse<FunnelEvent>>(
        `/practices/${id}/events?${toQueryString(filters)}`
      ),
  },

  // Connectors
  connectors: {
    getHealth: () =>
      apiRequest<ConnectorHealthSummary>('/connectors/health'),
    listByPractice: (practiceId: string) =>
      apiRequest<Connector[]>(`/practices/${practiceId}/connectors`),
    get: (connectorId: string) =>
      apiRequest<Connector>(`/connectors/${connectorId}`),
    create: (data: CreateConnectorInput) =>
      apiRequest<Connector>('/connectors', {
        method: 'POST',
        body: JSON.stringify(data),
      }),
    update: (connectorId: string, config: ConnectorConfig) =>
      apiRequest<Connector>(`/connectors/${connectorId}`, {
        method: 'PATCH',
        body: JSON.stringify(config),
      }),
    delete: (connectorId: string) =>
      apiRequest<void>(`/connectors/${connectorId}`, { method: 'DELETE' }),
    triggerSync: (connectorId: string) =>
      apiRequest<SyncResult>(`/connectors/${connectorId}/sync`, {
        method: 'POST',
      }),
    testConnection: (config: ConnectorConfig) =>
      apiRequest<TestConnectionResult>('/connectors/test', {
        method: 'POST',
        body: JSON.stringify(config),
      }),
  },

  // Alerts
  alerts: {
    list: (filters?: AlertFilters) =>
      apiRequest<FunnelAlert[]>(`/alerts?${toQueryString(filters || {})}`),
    acknowledge: (alertId: string) =>
      apiRequest<FunnelAlert>(`/alerts/${alertId}/acknowledge`, {
        method: 'POST',
      }),
    dismiss: (alertId: string) =>
      apiRequest<void>(`/alerts/${alertId}/dismiss`, { method: 'POST' }),
  },

  // Data Quality
  dataQuality: {
    getSummary: (range: string) =>
      apiRequest<DataQualitySummary>(`/data-quality/summary?range=${range}`),
    getErrors: (limit: number) =>
      apiRequest<PipelineError[]>(`/data-quality/errors?limit=${limit}`),
  },

  // System
  system: {
    getStatus: () => apiRequest<SystemStatus>('/system/status'),
  },
};
```

### 6.3 Error Handling Pattern

All API errors are handled at three levels:

```typescript
// Level 1: React Query global error handler (catches unhandled errors)
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      throwOnError: false,  // don't throw -- let components handle
    },
    mutations: {
      onError: (error) => {
        if (error instanceof ApiError) {
          if (error.status === 401) {
            // Token expired -- redirect to login
            window.location.href = '/login';
            return;
          }
          if (error.status === 403) {
            toast.error('You do not have permission to perform this action');
            return;
          }
          if (error.status >= 500) {
            toast.error('Server error. Please try again.');
            return;
          }
        }
        toast.error('Something went wrong. Please try again.');
      },
    },
  },
});

// Level 2: Per-query error handling in the hook
function usePracticeFunnel(practiceId: string, range: string) {
  return useQuery({
    queryKey: queryKeys.practices.funnel(practiceId, range),
    queryFn: () => api.practices.getFunnel(practiceId, range),
    retry: (failureCount, error) => {
      // Don't retry 404s or 403s
      if (error instanceof ApiError && [403, 404].includes(error.status)) {
        return false;
      }
      return failureCount < 2;
    },
  });
}

// Level 3: Component-level error display
function FunnelDataTab({ practiceId }: { practiceId: string }) {
  const { data, isLoading, isError, error, refetch } = usePracticeFunnel(practiceId, '7d');

  if (isLoading) return <FunnelChartSkeleton />;

  if (isError) {
    return (
      <ErrorState
        title="Failed to load funnel data"
        description={
          error instanceof ApiError && error.status === 404
            ? 'No funnel data has been collected for this practice yet.'
            : 'There was a problem loading funnel data. Please try again.'
        }
        onRetry={refetch}
      />
    );
  }

  return <FunnelChart data={data.stages} />;
}
```

### 6.4 Loading State Pattern

Every data-dependent view uses a three-state pattern:

```typescript
// Shared loading/error/empty components
function QueryState<T>({
  query,
  loading: LoadingSkeleton,
  empty: EmptyState,
  error: ErrorComponent,
  children,
}: {
  query: UseQueryResult<T>;
  loading: React.ComponentType;
  empty?: React.ComponentType;
  error?: React.ComponentType<{ error: Error; retry: () => void }>;
  children: (data: T) => React.ReactNode;
}) {
  if (query.isLoading) return <LoadingSkeleton />;
  if (query.isError) {
    const Err = ErrorComponent || DefaultErrorState;
    return <Err error={query.error} retry={query.refetch} />;
  }
  if (query.data === undefined || query.data === null) {
    const Empty = EmptyState || DefaultEmptyState;
    return <Empty />;
  }
  return <>{children(query.data)}</>;
}
```

---

## 7. WebSocket Foundation (Establish Pattern for Stage 3)

Phase 1 uses polling (React Query `refetchInterval`) for all real-time data. However, the WebSocket pattern should be established now so Stage 3 can upgrade specific data streams.

```typescript
// lib/websocket.ts -- Foundation, not active in Phase 1
import { useEffect, useRef, useCallback } from 'react';
import { useQueryClient } from '@tanstack/react-query';

interface WebSocketConfig {
  url: string;
  channels: string[];                         // e.g., ['practice:garner-dental', 'alerts']
  onMessage?: (event: WSEvent) => void;
  reconnectInterval?: number;
  maxReconnectAttempts?: number;
}

interface WSEvent {
  channel: string;
  type: 'funnel_update' | 'alert_new' | 'alert_resolved' | 'connector_status' | 'sync_complete';
  data: unknown;
  timestamp: string;
}

function useWebSocket(config: WebSocketConfig) {
  const wsRef = useRef<WebSocket | null>(null);
  const queryClient = useQueryClient();
  const reconnectCount = useRef(0);
  const maxAttempts = config.maxReconnectAttempts ?? 5;

  const connect = useCallback(() => {
    const ws = new WebSocket(config.url);

    ws.onopen = () => {
      reconnectCount.current = 0;
      // Subscribe to channels
      ws.send(JSON.stringify({ type: 'subscribe', channels: config.channels }));
    };

    ws.onmessage = (event) => {
      const wsEvent: WSEvent = JSON.parse(event.data);

      // Automatically invalidate relevant React Query caches
      switch (wsEvent.type) {
        case 'funnel_update':
          queryClient.invalidateQueries({ queryKey: ['practices', wsEvent.data.practiceId, 'funnel'] });
          break;
        case 'alert_new':
        case 'alert_resolved':
          queryClient.invalidateQueries({ queryKey: ['alerts'] });
          break;
        case 'connector_status':
        case 'sync_complete':
          queryClient.invalidateQueries({ queryKey: ['connectors'] });
          break;
      }

      config.onMessage?.(wsEvent);
    };

    ws.onclose = () => {
      if (reconnectCount.current < maxAttempts) {
        reconnectCount.current++;
        setTimeout(connect, (config.reconnectInterval ?? 3000) * reconnectCount.current);
      }
    };

    wsRef.current = ws;
  }, [config, queryClient, maxAttempts]);

  useEffect(() => {
    // Phase 1: WebSocket disabled, using polling instead
    // Uncomment in Stage 3:
    // connect();
    // return () => wsRef.current?.close();
  }, [connect]);

  return {
    isConnected: wsRef.current?.readyState === WebSocket.OPEN,
    send: (data: unknown) => wsRef.current?.send(JSON.stringify(data)),
  };
}
```

**Migration plan**: In Stage 3, for each data stream that needs real-time updates:
1. Enable the WebSocket hook in the relevant page
2. Remove the `refetchInterval` from the corresponding React Query hook
3. The WebSocket `onmessage` handler invalidates the cache, triggering a refetch -- same UI code, different trigger

---

## 8. Responsive Design

### 8.1 Breakpoint System

Desktop-first for the admin interface (HQ ops team uses desktops). Tablet-first for Nerve Center components (Stage 3).

| Name | Min Width | Target Devices | Priority |
|------|-----------|----------------|----------|
| `sm` | 640px | Large phones | Low (admin is not phone-primary) |
| `md` | 768px | Tablets, small laptops | Medium |
| `lg` | 1024px | Laptops | High |
| `xl` | 1280px | Desktops | Primary |
| `2xl` | 1536px | Large monitors | Supported |

### 8.2 Admin Interface Responsive Behavior

| Element | Desktop (xl+) | Laptop (lg) | Tablet (md) | Mobile (sm) |
|---------|---------------|-------------|-------------|-------------|
| Sidebar | Fixed, 240px | Collapsible, 240px/64px | Collapsible overlay | Bottom nav bar |
| TopNav | Full width, all elements | Full width | Hamburger menu | Hamburger menu |
| Practice grid | 3-4 columns | 2-3 columns | 2 columns | 1 column |
| FunnelChart | Horizontal, full labels | Horizontal, abbreviated | Horizontal, compact | Vertical stack |
| Practice detail | Tabs + content side by side | Tabs above content | Tabs above content | Accordion sections |
| Connector cards | 2 columns | 2 columns | 1 column | 1 column |
| Event table | Full columns | Hide low-priority columns | Horizontal scroll | Card view |
| Alert cards | 2 columns | 1 column | 1 column | 1 column |
| Data quality page | Charts side by side | Charts stacked | Charts stacked | Charts stacked |

### 8.3 Nerve Center Component Responsive Behavior (Stage 3, but design for it now)

The reusable components in `packages/funnel-ui` must work tablet-first since the Practice Nerve Center runs on front-desk iPads.

| Component | iPad (md) | Phone (sm) | Desktop (lg+) |
|-----------|-----------|------------|----------------|
| FunnelChart | Horizontal, touch-friendly bars (48px min height) | Vertical stack | Full horizontal with hover tooltips |
| HealthIndicator | Large (lg size), prominent | Medium (md size) | Medium (md size) |
| PracticeCard | Full width, all sections | Full width, compact variant | 320px card in grid |
| AlertCard | Full width, expanded by default | Full width, collapsed | Max 700px, collapsed |
| ActionCard | Full width, 56px tap targets, large buttons | Full width, essential info only | Max 500px, centered |
| TrendLine | Full width, 60px height | Full width, 40px height | Container width, 40px height |
| DrillDownNav | Back arrow + current level | Back arrow + current level | Full breadcrumb |

### 8.4 Touch Target Requirements

For all components that will be used in the Nerve Center (Stage 3):
- Minimum tap target: 44x44px (WCAG), preferred 48x48px
- Minimum spacing between adjacent targets: 8px
- Primary action buttons: 56px height on tablet
- Swipe gestures: ActionCard supports swipe-right-to-complete (Stage 3)

---

## 9. Design Tokens and Theme

### 9.1 Funnel OS Dark Theme

The Funnel OS uses a dark theme matching the HTML prototype. This is separate from the Concierge/Storefront light theme.

```css
:root {
  /* Background */
  --funnel-bg-dark: #0a0e1a;
  --funnel-bg-card: #111827;
  --funnel-bg-card-hover: #1a2332;
  --funnel-bg-elevated: #1e293b;
  --funnel-bg-input: #0f172a;

  /* Accent */
  --funnel-accent: #3b82f6;
  --funnel-accent-light: #60a5fa;
  --funnel-accent-glow: rgba(59, 130, 246, 0.15);

  /* Status */
  --funnel-status-good: #10b981;
  --funnel-status-watch: #f59e0b;
  --funnel-status-alert: #ef4444;
  --funnel-status-info: #3b82f6;
  --funnel-status-unknown: #64748b;

  /* Text */
  --funnel-text-primary: #f1f5f9;
  --funnel-text-secondary: #94a3b8;
  --funnel-text-muted: #64748b;

  /* Borders */
  --funnel-border: #1e293b;
  --funnel-border-light: #334155;
  --funnel-border-focus: #3b82f6;

  /* Funnel stage colors (for bars) */
  --funnel-stage-1: #3b82f6;     /* Leads Generated - blue */
  --funnel-stage-2: #60a5fa;     /* Connected - light blue */
  --funnel-stage-3: #8b5cf6;     /* Booked - purple */
  --funnel-stage-4: #10b981;     /* Showed - green */
  --funnel-stage-5: #f59e0b;     /* Produced - gold */

  /* Severity */
  --funnel-severity-critical-bg: rgba(239, 68, 68, 0.1);
  --funnel-severity-critical-border: #ef4444;
  --funnel-severity-warning-bg: rgba(245, 158, 11, 0.1);
  --funnel-severity-warning-border: #f59e0b;
  --funnel-severity-info-bg: rgba(59, 130, 246, 0.1);
  --funnel-severity-info-border: #3b82f6;

  /* Additional colors */
  --funnel-purple: #8b5cf6;
  --funnel-orange: #f97316;

  /* Typography */
  --funnel-font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --funnel-font-mono: 'JetBrains Mono', 'Fira Code', monospace;

  /* Spacing (4px base) */
  --funnel-space-1: 4px;
  --funnel-space-2: 8px;
  --funnel-space-3: 12px;
  --funnel-space-4: 16px;
  --funnel-space-5: 20px;
  --funnel-space-6: 24px;
  --funnel-space-8: 32px;
  --funnel-space-10: 40px;
  --funnel-space-12: 48px;
  --funnel-space-16: 64px;

  /* Radii */
  --funnel-radius-sm: 4px;
  --funnel-radius-md: 8px;
  --funnel-radius-lg: 12px;
  --funnel-radius-xl: 16px;
  --funnel-radius-full: 9999px;

  /* Shadows (darker for dark theme) */
  --funnel-shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3);
  --funnel-shadow-md: 0 4px 6px rgba(0, 0, 0, 0.4);
  --funnel-shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.5);
  --funnel-shadow-glow: 0 0 20px rgba(59, 130, 246, 0.15);

  /* Layout */
  --funnel-sidebar-width: 240px;
  --funnel-sidebar-collapsed: 64px;
  --funnel-topnav-height: 56px;
  --funnel-content-max-width: 1200px;
  --funnel-card-min-width: 300px;
  --funnel-tap-target-min: 44px;
}
```

### 9.2 Tailwind Configuration

```typescript
// tailwind.config.ts (extends shared config from packages/ui)
import sharedConfig from '@funnel-os/ui/tailwind.config';

export default {
  presets: [sharedConfig],
  theme: {
    extend: {
      colors: {
        funnel: {
          dark: '#0a0e1a',
          card: '#111827',
          'card-hover': '#1a2332',
          elevated: '#1e293b',
          input: '#0f172a',
          accent: '#3b82f6',
          'accent-light': '#60a5fa',
          border: '#1e293b',
          'border-light': '#334155',
        },
        status: {
          good: '#10b981',
          watch: '#f59e0b',
          alert: '#ef4444',
          info: '#3b82f6',
          unknown: '#64748b',
        },
        stage: {
          1: '#3b82f6',
          2: '#60a5fa',
          3: '#8b5cf6',
          4: '#10b981',
          5: '#f59e0b',
        },
      },
      fontFamily: {
        sans: ['Inter', ...sharedConfig.theme.fontFamily.sans],
        mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
      },
      animation: {
        'pulse-slow': 'pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite',
        'glow': 'glow 2s ease-in-out infinite alternate',
      },
      keyframes: {
        glow: {
          '0%': { boxShadow: '0 0 5px rgba(59, 130, 246, 0.1)' },
          '100%': { boxShadow: '0 0 20px rgba(59, 130, 246, 0.3)' },
        },
      },
    },
  },
};
```

---

## 10. Performance Targets

| Metric | Target | Strategy |
|--------|--------|----------|
| Admin initial load (LCP) | < 2s | Code splitting, preloaded fonts |
| Practice list render (10 cards) | < 200ms | Lightweight card components |
| FunnelChart render | < 300ms | Recharts with memoized data |
| Connector status refresh | < 500ms | React Query background refetch |
| Page navigation (SPA) | < 100ms | Next.js App Router prefetching |
| Component library page | < 1.5s | Lazy-loaded component demos |
| Lighthouse (desktop) | > 90 | Dark theme, minimal images |
| Cumulative Layout Shift | < 0.1 | Skeleton loaders match content dimensions |

### Optimization Strategies

- **React Query stale-while-revalidate**: Show cached data immediately, refresh in background
- **Component memoization**: All reusable components wrapped in `React.memo` with stable prop references
- **Recharts lazy loading**: Chart libraries loaded via `next/dynamic` with SSR disabled
- **Code splitting**: Each route is a separate chunk via Next.js App Router
- **Prefetching**: Practice detail data prefetched on card hover in the practice grid
- **Image optimization**: Minimal images in admin UI; connector platform icons as inline SVGs
- **Font subsetting**: Inter loaded with `latin` subset only, `swap` display strategy

---

## 11. Phase 1 Scope and Delivery Schedule

### In Scope

| Surface | Features |
|---------|----------|
| Practice list | Grid of 10 pilot practices with PracticeCard, connector status dots, mini funnel chart, health indicator |
| Practice detail | Full funnel chart, trend lines, conversion rates, connector management, alert list, raw event log |
| Connector management | Global connector health overview, per-practice connector list, config wizard (credentials, polling, test), manual sync trigger |
| Data quality | Pipeline summary, event volume charts, data gap alerts, error log |
| Component library | Living demo page of all reusable components with mock data controls |
| Reusable components | FunnelChart, HealthIndicator, PracticeCard, AlertCard, TrendLine, DrillDownNav, ActionCard (spec + skeleton) |

### Out of Scope (Stage 2-3)

| Feature | Stage | Reason |
|---------|-------|--------|
| Custom Executive Dashboard | Stage 3 | Metabase serves as Stage 1 dashboard |
| Practice Nerve Center | Stage 3 | Requires AI Analysis Engine (Stage 2) |
| ActionCard full implementation | Stage 3 | Needs daily agenda generation from AI |
| WebSocket real-time updates | Stage 3 | Polling sufficient for 10 practices |
| Role-based access (CEO vs VP vs ops) | Stage 3 | Single admin role sufficient for Phase 1 |
| Mobile-optimized Nerve Center | Stage 3 | iPad/phone optimization with Concierge merge |
| Push notifications | Stage 3 | Browser notifications for critical alerts |
| Offline support (PWA) | Stage 4 | Needed when practices use Nerve Center |
| Dark/light mode toggle | Stage 3 | Dark mode only for Phase 1 admin |

### Delivery Schedule (Month 1)

| Week | Deliverable | Dependencies |
|------|-------------|-------------|
| 1 | Project scaffold (Next.js 14, Tailwind, React Query), design tokens, `packages/funnel-ui` setup, base components (ConnectorStatusDot, SeverityBadge, MetricCard, TimeRangeToggle) | Tailwind config, monorepo workspace |
| 2 | FunnelChart, HealthIndicator, TrendLine, FunnelMiniChart, DrillDownNav. Component library demo page with mock data. | Recharts integration |
| 3 | PracticeCard, AlertCard, ActionCard (skeleton). PracticeListPage, PracticeDetailPage with all tabs. | Backend API endpoints for practices and funnel data |
| 4 | ConnectorManagementPage, ConnectorConfigWizard, DataQualityPage. Integration testing with live backend. Polish, error states, loading skeletons. | Backend API endpoints for connectors and data quality |

---

## 12. Technical Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| Next.js | 14.x | App framework (App Router), shared with Concierge |
| React | 18.x | UI library |
| TypeScript | 5.4+ | Type safety |
| Tailwind CSS | 3.x | Utility-first styling, dark theme |
| @tanstack/react-query | 5.x | Server state management, polling, cache |
| zustand | 4.x | Client UI state (sidebar, wizard, filters) |
| recharts | 2.x | FunnelChart, TrendLine, bar charts, sparklines |
| date-fns | 3.x | Date formatting, relative timestamps |
| clsx | 2.x | Conditional class names |
| tailwind-merge | 2.x | Merge Tailwind classes without conflicts |
| @radix-ui/react-tabs | latest | Accessible tab navigation |
| @radix-ui/react-dialog | latest | Modal dialogs (connector wizard) |
| @radix-ui/react-dropdown-menu | latest | User menu, context menus |
| @radix-ui/react-tooltip | latest | Hover tooltips on charts and status dots |
| react-hot-toast | 2.x | Toast notifications (sync success, errors) |
| zod | 3.x | Runtime validation for API responses and connector config forms |
| react-hook-form | 7.x | Connector config wizard forms |
| next-auth | 5.x | Authentication (shared with Concierge dashboard) |

---

## 13. Accessibility Requirements

| Requirement | Implementation |
|-------------|---------------|
| Keyboard navigation | All interactive elements reachable via Tab. Practice cards focusable with Enter to navigate. Connector wizard navigable with arrow keys. |
| Screen reader | FunnelChart has `aria-label` describing the funnel state. HealthIndicator uses `aria-label` not color alone (e.g., "Status: Alert, Phone Answer Rate 62%"). AlertCard severity communicated via text, not just border color. |
| Color contrast | All text meets WCAG AA on dark backgrounds. `#f1f5f9` on `#111827` = 13.2:1 ratio. `#94a3b8` on `#111827` = 5.8:1 ratio. Status colors always paired with text labels. |
| Focus management | Focus ring uses `--funnel-accent` (blue) with 2px offset. Dialog traps focus. Wizard moves focus to the active step heading. |
| Reduced motion | `prefers-reduced-motion`: disable chart animations, pulse effects, glow animations. HealthIndicator pulse disabled. TrendLine draw animation disabled. |
| Touch targets | All buttons min 44x44px. Connector status dots have 44x44px hit area despite 12px visual size. Chart bars have 44px min height for click targets. |
| Chart accessibility | FunnelChart provides a visually hidden data table as an alternative. TrendLine has `aria-label` with trend summary (e.g., "Trending up 3% over 7 days"). |
