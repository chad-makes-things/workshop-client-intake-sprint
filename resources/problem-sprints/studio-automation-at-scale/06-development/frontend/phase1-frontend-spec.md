# Phase 1 Frontend Specification

**Frontend Engineer:** Development Team — Frontend Role
**Sprint source:** Studio Automation at Scale
**Date:** 2026-03-10
**Scope:** Studio Analytics Dashboard + Studio Admin Interface (internal-only)

---

## Important Framing

**Practice managers do not have a frontend app.** Their interface is the agent's reply emails and Teams messages. The frontend build is entirely internal — for the Creative Director and Studio Admin. This keeps Phase 1 scope tight and ensures the user-facing experience is the agent's conversational output, not a web form.

**Two surfaces to build:**
1. **Studio Analytics Dashboard** — CD and Studio Admin's visibility into request volume, turnaround, medium split, and brand kit health
2. **Studio Admin Interface** — Practice profile management, brand kit health, template library view

---

## Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Framework | React 18 + Vite | Standard, fast build; no SSR needed for internal tool |
| Routing | React Router v6 | Simple client-side routing |
| State | TanStack Query (React Query) | Server state management for dashboard data; handles caching, refetch, loading states |
| Styling | Tailwind CSS | Utility-first; matches design spec color tokens; fast to implement |
| Charts | Recharts | Lightweight, React-native; sufficient for bar/line charts needed in dashboard |
| Auth | MSAL (Microsoft Authentication Library) | Azure AD SSO — CD and Studio Admin authenticate with their Microsoft 365 accounts |
| Hosting | Azure Static Web Apps | Integrated with Azure AD for auth; serves from CDN |

---

## Routes

```
/                         → Dashboard (redirect to /dashboard)
/dashboard                → Studio Analytics Dashboard
/dashboard/requests       → All requests list (with filters)
/dashboard/requests/:id   → Request detail view
/admin                    → Studio Admin home
/admin/practices          → Practice list with brand health scores
/admin/practices/:id      → Practice profile editor
/admin/templates          → Template library view
```

---

## Component Tree

```
App
├── AuthProvider (MSAL)
├── Layout
│   ├── TopNav (SGA Studio logo, user avatar, nav links)
│   └── Outlet (page content)
│
├── DashboardPage
│   ├── KPITiles (4 tiles: requests, auto-fulfill rate, turnaround, print volume)
│   ├── MediumSplitChart (horizontal bar: digital-only / digital+print / print-only)
│   ├── RecentRequestsList (last 10 requests with status icons)
│   │   └── RequestRow (practice name, type, channel, status, turnaround)
│   ├── BrandHealthSummary (3 status buckets with counts)
│   └── InactivePracticesAlert (count + "Send proactive check-in" action)
│
├── RequestsPage
│   ├── FilterBar (date range, practice, channel, status, collateral type)
│   ├── RequestsTable (sortable columns)
│   └── Pagination
│
├── RequestDetailPage
│   ├── RequestHeader (practice, status badge, channel, created at)
│   ├── ConversationTimeline (full conversation log, styled like email thread)
│   ├── AssetsPanel (preview thumbnails, download links)
│   └── OrderPanel (print order details if placed)
│
├── AdminPage
│   └── AdminNav (Practices, Templates)
│
├── PracticesPage
│   ├── SearchBar
│   ├── PracticeTable (name, platform, brand health score badge, last request)
│   └── Pagination
│
├── PracticeDetailPage
│   ├── PracticeHeader (name, platform badge, edit button)
│   ├── BrandHealthScoreCard
│   │   ├── ScoreRing (circular progress: 0-100)
│   │   └── ScoreBreakdown (logo, colors, fonts, photos — with missing items flagged)
│   ├── ContactInfoForm (editable: name, email, phone, address)
│   ├── AssetLibrary (logo files + photos in a grid; upload dropzone)
│   └── RequestHistory (last 10 requests from this practice)
│
└── TemplatesPage
    ├── TemplateGrid (8 template cards)
    │   └── TemplateCard (thumbnail, name, type, last used, Canva link)
    └── AddTemplateButton (opens Canva, admin registers on return)
```

---

## Key Component Specs

### KPITiles

```tsx
interface KPITile {
  label: string;
  value: string | number;
  trend: 'up' | 'down' | 'neutral';
  trendValue: string; // "↑ 4 vs last week"
  trendPositive: boolean; // is the trend direction good?
}

// Tiles:
// 1. Requests This Week — up is neutral
// 2. Auto-Fulfill Rate — up is positive (target: ≥80%)
// 3. Avg Turnaround — down is positive (target: ≤4 hours)
// 4. Print Volume vs. Baseline — down is positive (target: -30%)
```

**Visual:** 4-column grid. Each tile: large value (2rem, navy), trend line (small, color-coded), label (muted). Trend color: green if `trendPositive`, red if not, gray if neutral.

---

### MediumSplitChart

```tsx
interface MediumSplitData {
  week: string;
  digitalOnly: number;     // percentage
  digitalAndPrint: number;
  printOnly: number;
}
```

**Visual:** Stacked horizontal bar chart, one bar per week. Colors: teal (digital-only), navy (digital+print), amber (print-only). Legend below chart. 8-week rolling window. Goal line: "30% print reduction target" marker.

---

### BrandHealthScoreCard

```tsx
interface BrandHealthScore {
  total: number; // 0-100
  breakdown: {
    logo: { score: number; maxScore: 30; issues: string[] };
    colors: { score: number; maxScore: 20; issues: string[] };
    fonts: { score: number; maxScore: 20; issues: string[] };
    photos: { score: number; maxScore: 30; issues: string[] };
  };
}
```

**Visual:**
- Score ring: circular progress indicator, color: <40 red, 40-69 amber, ≥70 teal
- Breakdown list: each category shows score/maxScore + any issue notes in small red text
- "Needs asset lift" banner if score < 40 — sticky at top of the card

---

### AssetLibrary (Practice Admin)

```tsx
// Accepts: SVG, PNG, JPG, PDF
// Max file size: 20MB per file
// Auto-sync to Canva brand kit on successful upload
```

**Dropzone behavior:**
- Drag-and-drop or click to upload
- Immediate preview after upload
- File type auto-categorization: SVG/PNG files → logo library; JPG files → photo library
- Upload triggers `PATCH /api/practices/:id/assets` → backend uploads to Azure Blob → syncs to Canva brand kit
- Error states: file too large, unsupported type, Canva sync failure (with retry button)

---

### ConversationTimeline (Request Detail)

Visual representation of the request conversation — styled like an email thread for familiarity:

```
[Practice Manager] ashley@townlakedental.com — 10:32 AM
  "We're running a teeth whitening special in April..."

[Studio Agent] studio@sgadental.com — 10:32 AM
  "Hi Ashley! Got your request for the April whitening special..."

[Practice Manager] ashley@townlakedental.com — 10:51 AM
  "Existing patients mainly, but new patients too..."

[Studio Agent] studio@sgadental.com — 10:52 AM
  "Here are your three April whitening special assets..."
  [Preview thumbnail: email header]
  [Preview thumbnail: TV slide]
  [Preview thumbnail: desk flyer]

[Practice Manager] ashley@townlakedental.com — 11:05 AM
  "These look great! On the flyer can you change the CTA to 'Ask us today'..."

[Studio Agent] studio@sgadental.com — 11:07 AM
  "Done! Here's the updated flyer..."

[CD Review] Marcus — 11:28 AM
  ✅ Approved All — 58 seconds

[Studio Agent] studio@sgadental.com — 11:28 AM
  "All three assets are approved..."
```

**Visual rules:**
- Practice manager messages: right-aligned bubble (blue)
- Agent messages: left-aligned (gray)
- CD review: full-width event marker (teal)
- Asset thumbnails rendered inline where they appear in conversation
- Timestamps shown relative ("10 minutes ago") on hover shows absolute

---

## Dashboard Data Fetching

```tsx
// React Query hooks — all auto-refresh every 60 seconds

export function useDashboardSummary(period: 'week' | 'month') {
  return useQuery({
    queryKey: ['dashboard', 'summary', period],
    queryFn: () => api.get(`/api/analytics/summary?period=${period}`),
    refetchInterval: 60_000,
  });
}

export function useMediumSplit(weeks = 8) {
  return useQuery({
    queryKey: ['dashboard', 'medium-split', weeks],
    queryFn: () => api.get(`/api/analytics/medium-split?weeks=${weeks}`),
    refetchInterval: 60_000,
  });
}

export function useBrandHealth() {
  return useQuery({
    queryKey: ['dashboard', 'brand-health'],
    queryFn: () => api.get('/api/analytics/brand-health'),
    refetchInterval: 300_000, // refresh every 5 min — changes slowly
  });
}

export function useRequests(filters: RequestFilters) {
  return useQuery({
    queryKey: ['requests', filters],
    queryFn: () => api.get('/api/requests', { params: filters }),
  });
}
```

---

## Auth Flow (Azure AD SSO)

```tsx
// Only CD (Marcus) and Studio Admin have access
// Azure AD group: "SGA Studio Team"

const msalConfig = {
  auth: {
    clientId: process.env.VITE_AZURE_CLIENT_ID,
    authority: `https://login.microsoftonline.com/${process.env.VITE_AZURE_TENANT_ID}`,
    redirectUri: window.location.origin,
  },
};

// Route protection: if not in "SGA Studio Team" AD group → redirect to access denied
// Token passed as Bearer in all API requests
```

---

## Error States

| State | Display |
|-------|---------|
| API request fails | Toast notification: "Couldn't load data. Retrying..." — auto-retry 3× |
| Dashboard data stale (>5 min old) | Subtle banner: "Data may be outdated — [Refresh]" |
| Asset upload fails | Inline error below dropzone with specific error message |
| Canva sync failure | Error on brand health card: "Last Canva sync failed — [Retry]" |
| No requests in selected period | Empty state: illustration + "No requests yet in this period" |

---

## Accessibility

- All interactive elements keyboard-accessible
- Focus ring visible on all interactive elements (Tailwind `focus:ring-2`)
- Status icons always paired with text labels (not icon-only)
- Color not used as the only indicator of status (shape + color + text)
- WCAG AA minimum contrast ratio on all text
- Dashboard KPI values available as screen reader text (not just visual ring)

---

## Phase 1 Exclusions

- No practice manager portal (their interface is email/Teams)
- No mobile-responsive optimization for admin (desktop only — CD review is via Teams notification, not this dashboard)
- No real-time WebSocket updates (60-second polling is sufficient for Phase 1)
- No dark mode
- No print-specific styles for dashboard
