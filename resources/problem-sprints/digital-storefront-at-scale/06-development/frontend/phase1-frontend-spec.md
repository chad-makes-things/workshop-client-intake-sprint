# Phase 1 Frontend Specification

**Role**: Frontend Engineer
**Sprint**: Digital Storefront at Scale
**Date**: 2026-03-10
**Phase 1 Scope**: Concierge chat for practices + basic HQ enrollment dashboard
**Source Artifacts**: `03-decide/storyboard.md`, `04-architect/product-architecture.md`, `06-development/technical/implementation-plan.md`

---

## 1. Application Architecture

Phase 1 delivers two web applications from the monorepo:

| App | Purpose | Users | Framework |
|-----|---------|-------|-----------|
| `apps/concierge` | Practice-facing concierge chat | Office managers, practice owners | Next.js 14 (App Router) |
| `apps/dashboard` | HQ enrollment dashboard + executive view | VP Marketing, HQ marketing team | Next.js 14 (App Router) |

Both apps share a component library (`packages/ui`) and type definitions (`packages/shared`).

---

## 2. Component Tree

### 2.1 Practice Concierge (`apps/concierge`)

```
ConciergeApp
├── EnrollmentLayout                    # Layout for the enrollment/chat experience
│   ├── PracticeHeader                  # Practice name, logo, enrollment step indicator
│   │   ├── StepIndicator               # Visual progress: Verify > Template > Approve > Live
│   │   └── PracticeBadge               # Name + specialty pill
│   ├── ChatInterface                   # Core chat experience
│   │   ├── MessageList                 # Scrollable message area
│   │   │   ├── MessageBubble           # Single message (user or assistant)
│   │   │   │   ├── UserMessage         # Right-aligned, practice brand color
│   │   │   │   └── AssistantMessage    # Left-aligned, SGA brand
│   │   │   │       ├── MarkdownContent # Rendered markdown (lists, bold, links)
│   │   │   │       └── TypingIndicator # Animated dots during streaming
│   │   │   ├── ActionCard              # Interactive card embedded in chat
│   │   │   │   ├── ConfirmationCard    # "Approve these changes?" with Accept/Reject
│   │   │   │   ├── DataVerificationCard # Practice data fields with edit/confirm toggles
│   │   │   │   └── TemplatePickerCard  # Inline template selection (see TemplatePicker)
│   │   │   ├── InlinePreview           # Embedded site/template preview
│   │   │   │   ├── PreviewFrame        # iframe with preview URL
│   │   │   │   └── PreviewControls     # Desktop/mobile toggle, open in new tab
│   │   │   └── ProgressCard            # Enrollment step completion summary
│   │   ├── MessageInput                # Bottom-fixed input area
│   │   │   ├── TextArea                # Auto-growing textarea
│   │   │   ├── SendButton              # Submit message
│   │   │   └── QuickActions            # Suggested action chips (context-dependent)
│   │   └── ConnectionStatus            # Online/offline/reconnecting indicator
│   └── Sidebar (desktop only)          # Practice summary panel
│       ├── PracticeSnapshot            # Key practice data at a glance
│       ├── EnrollmentProgress          # Step checklist with timestamps
│       └── QuickLinks                  # "View preview", "Contact HQ"
└── ErrorBoundary                       # Graceful error handling
    └── ErrorFallback                   # "Something went wrong" with retry
```

### 2.2 HQ Admin Dashboard (`apps/dashboard`)

```
DashboardApp
├── AuthLayout                          # Login page
│   └── LoginForm                       # Email + password
├── DashboardLayout                     # Authenticated layout with nav
│   ├── TopNav                          # SGA logo, user menu, notifications
│   │   ├── NotificationBell            # Alert count badge
│   │   └── UserMenu                    # Dropdown: profile, logout
│   ├── Sidebar                         # Navigation
│   │   ├── NavItem                     # Overview, Practices, Enrollment, Alerts
│   │   └── PlatformFilter              # Quick filter: SGA / Gen4 / MODIS
│   ├── OverviewPage                    # Executive dashboard view
│   │   ├── EnrollmentSummaryCard       # Donut chart: pending/invited/enrolled/blitz
│   │   ├── HealthScoreCard             # Network average score with trend arrow
│   │   ├── TierDistributionCard        # Bar chart: Optimized/Baseline/At Risk
│   │   ├── AlertsList                  # Top 5 alerts requiring attention
│   │   └── RecentActivityFeed          # Latest enrollment events, changes
│   ├── PracticeListPage                # Sortable, filterable practice table
│   │   ├── FilterBar                   # Status, specialty, platform, health tier, search
│   │   ├── PracticeTable               # Paginated table with key columns
│   │   │   └── PracticeRow             # Name, specialty, status, health, last interaction
│   │   ├── BulkActionBar               # Select all, batch invite, batch export
│   │   └── Pagination                  # Page controls
│   ├── PracticeDetailPage              # Deep dive on a single practice
│   │   ├── PracticeHeader              # Name, status badge, health score
│   │   ├── TabNav                      # Overview, Team, Services, Digital, Conversations
│   │   ├── OverviewTab
│   │   │   ├── EnrollmentTimeline      # Enrollment step history with timestamps
│   │   │   ├── HealthScoreBreakdown    # Radar chart: website, GMB, social, reviews, leads
│   │   │   └── RecentChanges           # Change event log
│   │   ├── TeamTab
│   │   │   ├── TeamMemberList          # Cards with photo, name, role, status
│   │   │   └── AddTeamMemberForm       # Manual add form (for HQ overrides)
│   │   ├── ServicesTab
│   │   │   └── ServiceList             # Editable service list
│   │   ├── DigitalTab
│   │   │   ├── WebsiteCard             # Status, URL, template, last deployed, preview link
│   │   │   ├── GmbCard                 # Status, completeness score (Phase 2)
│   │   │   └── SocialCard              # Status, handles (Phase 2)
│   │   └── ConversationsTab
│   │       └── ConversationList        # Conversation summaries with expand to full transcript
│   ├── EnrollmentPage                  # Enrollment pipeline view
│   │   ├── PipelineView               # Kanban-style columns: Pending > Invited > In Progress > Enrolled
│   │   │   └── PracticeCard            # Mini card with name, days in stage
│   │   ├── InvitePanel                 # Select practices + send invitations
│   │   └── EnrollmentStats            # Conversion rates per stage
│   └── AlertsPage                      # All active alerts
│       ├── AlertFilters                # Severity, type, acknowledged/unacknowledged
│       └── AlertCard                   # Alert detail with practice link and action buttons
└── ErrorBoundary
```

### 2.3 Executive Dashboard (within `apps/dashboard`)

The executive dashboard is a view within the HQ dashboard, accessible via a dedicated route. It surfaces summary-level data without practice-level controls.

```
ExecutiveDashboardPage
├── NetworkHealthHero                   # Large number: network average health score
│   ├── TrendIndicator                  # Up/down arrow with percentage change
│   └── TimePeriodToggle                # 7d / 30d / 90d
├── EnrollmentProgressBar               # Full-width progress: X/267 enrolled
├── MetricGrid (2x3)
│   ├── MetricCard                      # Total practices
│   ├── MetricCard                      # Enrolled (concierge)
│   ├── MetricCard                      # Blitz deployed
│   ├── MetricCard                      # Average health score
│   ├── MetricCard                      # Active alerts
│   └── MetricCard                      # Concierge interactions (7d)
├── TierDistributionChart               # Stacked bar: Optimized / Baseline / At Risk
├── TopAlerts                           # Top 3 critical alerts with one-click navigation
└── PlatformBreakdown                   # Health scores split by SGA / Gen4 / MODIS
```

---

## 3. Routes and Page Structure

### Concierge App (`apps/concierge`)

| Route | Page | Description |
|-------|------|-------------|
| `/enroll/:slug/:token` | `EnrollmentPage` | Practice enrollment entry point (validates token, loads practice context, opens chat) |
| `/chat/:slug/:token` | `ChatPage` | Ongoing concierge chat (post-enrollment) |
| `/preview/:slug/:templateId` | `PreviewPage` | Full-page template preview with approve/reject controls |

### Dashboard App (`apps/dashboard`)

| Route | Page | Description |
|-------|------|-------------|
| `/login` | `LoginPage` | HQ team login |
| `/` | `OverviewPage` | Executive summary dashboard |
| `/executive` | `ExecutiveDashboardPage` | Executive-only read view |
| `/practices` | `PracticeListPage` | Filterable practice list |
| `/practices/:id` | `PracticeDetailPage` | Practice detail with tabs |
| `/practices/:id/:tab` | `PracticeDetailPage` | Direct tab link (team, services, digital, conversations) |
| `/enrollment` | `EnrollmentPage` | Enrollment pipeline kanban |
| `/enrollment/invite` | `InvitePage` | Batch invitation form |
| `/alerts` | `AlertsPage` | Alert management |
| `/settings` | `SettingsPage` | User management, API keys (Phase 1 minimal) |

---

## 4. State Management

### 4.1 Server State: React Query (TanStack Query)

All API data is managed through React Query. No server data stored in local state.

```typescript
// Query key conventions
const queryKeys = {
  practices: {
    all: ['practices'] as const,
    list: (filters: PracticeFilters) => ['practices', 'list', filters] as const,
    detail: (id: string) => ['practices', 'detail', id] as const,
    team: (id: string) => ['practices', id, 'team'] as const,
    services: (id: string) => ['practices', id, 'services'] as const,
    health: (id: string) => ['practices', id, 'health'] as const,
  },
  enrollment: {
    status: (id: string) => ['enrollment', id, 'status'] as const,
    pipeline: ['enrollment', 'pipeline'] as const,
  },
  dashboard: {
    summary: ['dashboard', 'summary'] as const,
    alerts: (filters?: AlertFilters) => ['dashboard', 'alerts', filters] as const,
    healthHistory: (id: string, period: string) => ['dashboard', 'health', id, period] as const,
  },
  conversations: {
    list: (practiceId: string) => ['conversations', practiceId] as const,
    detail: (convId: string) => ['conversations', 'detail', convId] as const,
  },
  templates: {
    all: ['templates'] as const,
  },
};
```

**Query configuration**:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,        // 30 seconds before refetch
      gcTime: 5 * 60_000,       // 5 minutes cache lifetime
      retry: 2,
      refetchOnWindowFocus: true,
    },
  },
});
```

**Example hooks**:

```typescript
// Practice list with filtering
function usePractices(filters: PracticeFilters) {
  return useQuery({
    queryKey: queryKeys.practices.list(filters),
    queryFn: () => api.practices.list(filters),
    placeholderData: keepPreviousData,  // Smooth pagination
  });
}

// Practice detail
function usePractice(id: string) {
  return useQuery({
    queryKey: queryKeys.practices.detail(id),
    queryFn: () => api.practices.get(id),
  });
}

// Enrollment status with polling during active enrollment
function useEnrollmentStatus(practiceId: string) {
  return useQuery({
    queryKey: queryKeys.enrollment.status(practiceId),
    queryFn: () => api.enrollment.getStatus(practiceId),
    refetchInterval: (query) =>
      query.state.data?.status === 'in_progress' ? 5000 : false,
  });
}

// Dashboard summary (auto-refresh every 60s)
function useDashboardSummary() {
  return useQuery({
    queryKey: queryKeys.dashboard.summary,
    queryFn: () => api.dashboard.getSummary(),
    refetchInterval: 60_000,
  });
}
```

**Mutations with optimistic updates**:

```typescript
function useUpdatePractice(practiceId: string) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: Partial<Practice>) => api.practices.update(practiceId, data),
    onMutate: async (data) => {
      await queryClient.cancelQueries({ queryKey: queryKeys.practices.detail(practiceId) });
      const previous = queryClient.getQueryData(queryKeys.practices.detail(practiceId));
      queryClient.setQueryData(queryKeys.practices.detail(practiceId), (old: Practice) => ({
        ...old,
        ...data,
      }));
      return { previous };
    },
    onError: (_err, _data, context) => {
      queryClient.setQueryData(queryKeys.practices.detail(practiceId), context?.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: queryKeys.practices.detail(practiceId) });
      queryClient.invalidateQueries({ queryKey: queryKeys.practices.all });
    },
  });
}
```

### 4.2 Client State: Zustand

Client-only state (UI state, filters, chat state) managed with Zustand. Kept minimal -- most state lives on the server.

```typescript
// Chat state (Concierge app)
interface ChatState {
  conversationId: string | null;
  messages: ChatMessage[];
  isStreaming: boolean;
  pendingActions: ActionCard[];
  inputDraft: string;

  // Actions
  setConversationId: (id: string) => void;
  addMessage: (message: ChatMessage) => void;
  appendToLastMessage: (text: string) => void;
  setStreaming: (streaming: boolean) => void;
  addAction: (action: ActionCard) => void;
  resolveAction: (actionId: string, response: string) => void;
  setInputDraft: (draft: string) => void;
  reset: () => void;
}

const useChatStore = create<ChatState>((set, get) => ({
  conversationId: null,
  messages: [],
  isStreaming: false,
  pendingActions: [],
  inputDraft: '',

  setConversationId: (id) => set({ conversationId: id }),
  addMessage: (message) => set((s) => ({ messages: [...s.messages, message] })),
  appendToLastMessage: (text) => set((s) => {
    const messages = [...s.messages];
    const last = messages[messages.length - 1];
    if (last?.role === 'assistant') {
      messages[messages.length - 1] = { ...last, content: last.content + text };
    }
    return { messages };
  }),
  setStreaming: (streaming) => set({ isStreaming: streaming }),
  addAction: (action) => set((s) => ({ pendingActions: [...s.pendingActions, action] })),
  resolveAction: (actionId, response) => set((s) => ({
    pendingActions: s.pendingActions.filter((a) => a.id !== actionId),
  })),
  setInputDraft: (draft) => set({ inputDraft: draft }),
  reset: () => set({
    conversationId: null, messages: [], isStreaming: false,
    pendingActions: [], inputDraft: '',
  }),
}));

// Dashboard filter state
interface DashboardFilterState {
  statusFilter: EnrollmentStatus | 'all';
  specialtyFilter: Specialty | 'all';
  platformFilter: Platform | 'all';
  healthTier: 'all' | 'optimized' | 'baseline' | 'at_risk';
  searchQuery: string;
  sortField: string;
  sortOrder: 'asc' | 'desc';

  setFilter: (key: string, value: string) => void;
  resetFilters: () => void;
}

const useDashboardFilters = create<DashboardFilterState>((set) => ({
  statusFilter: 'all',
  specialtyFilter: 'all',
  platformFilter: 'all',
  healthTier: 'all',
  searchQuery: '',
  sortField: 'name',
  sortOrder: 'asc',

  setFilter: (key, value) => set({ [key]: value }),
  resetFilters: () => set({
    statusFilter: 'all', specialtyFilter: 'all', platformFilter: 'all',
    healthTier: 'all', searchQuery: '', sortField: 'name', sortOrder: 'asc',
  }),
}));
```

---

## 5. Key Component Specifications

### 5.1 ChatInterface

The core concierge interaction surface. Handles message streaming via SSE, inline action cards, and template previews.

**Props**: `practiceId: string`, `token: string`, `initialConversationId?: string`

**Behavior**:
1. On mount: load practice context and any existing conversation history
2. When user sends message: POST to `/api/v1/concierge/:practiceId/message`, open SSE stream
3. Stream events update chat store in real-time (content deltas, tool use, action cards)
4. Action cards are interactive -- practice can approve/reject inline
5. Template previews open in an inline iframe with desktop/mobile toggle
6. Auto-scroll to bottom on new content; pause auto-scroll if user has scrolled up
7. Persist draft message in Zustand (survives re-renders, not page refresh)
8. Keyboard: Enter to send, Shift+Enter for newline

**SSE Connection Pattern**:

```typescript
async function sendMessage(practiceId: string, message: string, conversationId: string | null) {
  const store = useChatStore.getState();
  store.addMessage({ role: 'user', content: message, createdAt: new Date() });
  store.setStreaming(true);
  store.addMessage({ role: 'assistant', content: '', createdAt: new Date() });

  const response = await fetch(`/api/v1/concierge/${practiceId}/message`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ conversation_id: conversationId, message, channel: 'web' }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const events = parseSSE(chunk);

    for (const event of events) {
      switch (event.type) {
        case 'message_start':
          store.setConversationId(event.data.conversation_id);
          break;
        case 'content_delta':
          store.appendToLastMessage(event.data.text);
          break;
        case 'tool_use':
          store.addMessage({
            role: 'system',
            content: `Executing: ${event.data.tool}`,
            type: 'tool_status',
            createdAt: new Date(),
          });
          break;
        case 'action_card':
          store.addAction(event.data);
          break;
        case 'message_end':
          store.setStreaming(false);
          break;
      }
    }
  }
}
```

**Error handling**: If the SSE stream errors, show an inline retry message: "Something went wrong. [Tap to retry]". The retry re-sends the last user message.

### 5.2 TemplatePicker

Displays 2-3 template options as preview cards during enrollment. One card has a "Recommended" badge based on the concierge's recommendation.

**Props**: `practiceId: string`, `specialty: Specialty`, `onSelect: (templateId: string) => void`

**Layout**: Horizontal scroll on mobile, 2-3 column grid on desktop.

**TemplatePreviewCard contents**:
- Template name and specialty label
- "Recommended" badge (amber, top-right corner) if this is the concierge's pick
- Preview thumbnail (screenshot of the template populated with practice data, lazy-loaded)
- 3 key selling points as a bullet list
- "Preview" button (opens full preview in iframe/modal)
- "Select This Template" button (primary action)

**Behavior**:
1. On render: fetch available templates for the practice's specialty
2. Pre-generate preview thumbnails via the template preview API
3. Recommendation badge is driven by the concierge's tool call (the `select_template` suggestion)
4. Clicking "Preview" opens the `InlinePreview` component with desktop/mobile toggle
5. Clicking "Select" calls `onSelect` which triggers the enrollment `select-template` API

**Recommendation reasoning**: Below the recommended card, display the concierge's reasoning in a callout box. Example: "I recommend this template because periodontist sites convert 23% better with referral-optimized layouts. Your referring GPs will find it easier to send patients your way."

### 5.3 DashboardView (Overview Page)

The landing page for HQ team. Shows network health at a glance with drill-down affordances.

**Layout**: Responsive grid. 4 metric cards across the top, 2x2 grid of charts below, activity feed on the right (or below on mobile).

**EnrollmentSummaryCard**:
- Donut chart (Recharts `PieChart`) showing enrollment status distribution
- Center text: "X / 267 enrolled"
- Legend with counts per status
- Click-through to `/enrollment`

**HealthScoreCard**:
- Large number: network average health score
- Trend arrow (up/down) with change from prior period
- Mini sparkline showing 30-day trend
- Color-coded: green (>70), yellow (50-70), red (<50)

**TierDistributionCard**:
- Horizontal stacked bar chart showing practice count per tier
- Optimized (green), Baseline (yellow), At Risk (red)
- Click-through: clicking a tier segment filters the practice list

**AlertsList**:
- Top 5 alerts sorted by severity then recency
- Each alert: severity icon, practice name, alert message, timestamp
- "View all alerts" link to `/alerts`
- Unacknowledged alerts have a yellow left border

**RecentActivityFeed**:
- Chronological list of enrollment events and practice changes
- Each item: timestamp, practice name, action description
- Auto-refreshes every 60 seconds (React Query `refetchInterval`)

### 5.4 PracticeDetail

Full drill-down view for a single practice. Tab-based navigation.

**Props**: Route param `id`, tab param determines active tab.

**PracticeHeader**:
- Practice name (large)
- Status badge (color-coded enrollment status pill)
- Health score (circular progress indicator, color-coded)
- Specialty pill
- Platform pill
- Action buttons: "Open Concierge" (opens chat in new tab), "Edit Practice"

**OverviewTab**:
- **EnrollmentTimeline**: Vertical timeline showing each enrollment event with timestamp and details. Uses `enrollment_event` records. Active step is highlighted.
- **HealthScoreBreakdown**: Radar chart (Recharts `RadarChart`) with 5 axes: Website, GMB, Social, Reviews, Leads. Shows current scores and 30-day-ago overlay for comparison.
- **RecentChanges**: Table of recent `change_event` records. Columns: date, entity, action, changes (JSON rendered as readable diff), sync status.

**TeamTab**:
- Card grid of team members. Each card: photo (or placeholder avatar), name, role, title, active/inactive badge, confidence indicator (harvested vs verified).
- Inactive members shown in a collapsed "Former Team" section.
- "Add Team Member" button opens a slide-over form.

**ServicesTab**:
- Sortable list of services with drag handle, name, description, primary badge.
- Inline editing (click to edit name/description).
- "Add Service" button at bottom.

**DigitalTab**:
- **WebsiteCard**: Status badge, live URL (clickable), template name, last deployed timestamp, "View Preview" button, "Regenerate" button.
- **GmbCard**: Status badge, completeness percentage bar, listing ID (Phase 2: full management).
- **SocialCard**: Instagram handle, Facebook URL, status badge (Phase 2: post scheduling).

**ConversationsTab**:
- Accordion list of conversations. Each: date, intent tag, summary, message count, resolved badge.
- Expand to see full message transcript (read-only, styled like the chat interface).

---

## 6. Chat UX Patterns

### 6.1 Typing Indicator

Shown when the SSE stream is active but between content deltas (brief pauses while Claude thinks).

- Three animated dots with staggered opacity pulsing (0.6s cycle)
- Appears immediately when `isStreaming` is true and no content has arrived yet
- Disappears as soon as the first `content_delta` arrives
- Re-appears during tool execution pauses (between `tool_use` and next `content_delta`)

### 6.2 Action Cards

Interactive cards embedded inline in the message flow. Used when the concierge needs explicit confirmation.

**ConfirmationCard** (staff change, data verification):

```
+--------------------------------------+
|  Staff Change                        |
|                                      |
|  I'll update the following:          |
|  - Remove Dr. Miller from website    |
|  - Remove from Google listing        |
|  - Update team page                  |
|                                      |
|  [  Approve  ]  [  Undo  ]          |
+--------------------------------------+
```

- Green primary button ("Approve" / "Looks Good" / "Go Live")
- Gray secondary button ("Undo" / "Wait" / "Let Me Check")
- Clicking either sends a message to the concierge with the action response
- After clicking, buttons become disabled, card shows "Approved" or "Undone" state
- Card has a subtle left border color indicating type (green for approval, blue for info, amber for caution)

**DataVerificationCard** (enrollment data review):

```
+--------------------------------------+
|  Practice Data                       |
|                                      |
|  Name: Bright Smile Dental    [ok]   |
|  Phone: (512) 555-0100        [ok]   |
|  Address: 123 Main St, Austin [edit] |
|  Team: 3 providers            [ok]   |
|                                      |
|  [ Everything looks correct ]        |
|  or click edit to fix any field      |
+--------------------------------------+
```

- Each field row has a checkmark (confirmed) and edit icon
- Clicking edit opens inline text editing for that field
- "Everything looks correct" button confirms all fields at once
- Corrections are batched and sent as a single message to the concierge
- Confidence indicators: green dot (high confidence from harvester), yellow dot (medium), red dot (low -- needs verification)

### 6.3 Inline Previews

When the concierge shows a template preview, it renders inline in the chat.

- iframe with preview URL, height 400px, scrollable
- Desktop/mobile toggle changes iframe width (100% vs 375px centered)
- "Open in New Tab" opens the preview URL directly
- Preview is lazy-loaded (only renders when scrolled into view)
- Loading state shows a skeleton placeholder with a pulsing animation
- Preview URL has an expiration (24 hours) -- show "Preview expired, regenerating..." if accessed after

### 6.4 Progress Indicators

During enrollment, the step indicator at the top of the chat shows progress.

```
  o--------o--------o--------o--------o
 Verify  Template  Review  Approve  Live
```

- Completed steps: filled circle + solid line (SGA brand color)
- Current step: filled circle + pulsing animation
- Future steps: hollow circle + dashed line (gray)
- Clicking a completed step scrolls the chat to where that step was completed
- On mobile: compact mode shows icons only (checkmark, palette, eye, thumbs-up, rocket)

### 6.5 Quick Action Chips

Context-sensitive suggested actions below the message input.

During enrollment:
```
[Show me templates] [Update my team] [Change our hours] [I have questions]
```

Post-enrollment:
```
[Report a change] [View my site] [Check my scores] [Schedule a post]
```

- Chips change based on enrollment status and conversation context
- Clicking a chip fills the message input and auto-sends
- Chips disappear while streaming (to reduce visual noise)
- Maximum 4 chips visible; horizontal scroll if more
- Chips are generated by the backend based on enrollment step and conversation history

---

## 7. Mobile-First Responsive Breakpoints

### Breakpoint System

| Name | Min Width | Target Devices |
|------|-----------|---------------|
| `xs` | 0px | Small phones (iPhone SE) |
| `sm` | 640px | Large phones (iPhone 14 Pro Max) |
| `md` | 768px | Tablets (iPad Mini portrait) |
| `lg` | 1024px | Tablets landscape, small laptops |
| `xl` | 1280px | Laptops, desktops |
| `2xl` | 1536px | Large monitors |

### Concierge Chat Responsiveness

| Element | Mobile (xs-sm) | Tablet (md) | Desktop (lg+) |
|---------|---------------|-------------|----------------|
| Chat layout | Full screen, no sidebar | Full screen, no sidebar | Chat 65% + sidebar 35% |
| Message bubbles | Full width with 16px padding | Max 85% width | Max 70% width |
| Action cards | Full width, stacked buttons | Full width, inline buttons | Max 600px, inline buttons |
| Template picker | Horizontal scroll, 1 card visible | 2 cards visible | 3 cards in row |
| Inline preview | Full width, 300px height | Full width, 400px height | 600px width, 400px height |
| Message input | Fixed bottom, 48px height | Fixed bottom, 48px height | Fixed bottom, 56px height |
| Quick action chips | Horizontal scroll | Wrap to 2 rows | Single row |
| Step indicator | Compact (icons only) | Icons + abbreviated labels | Icons + full labels |

### Dashboard Responsiveness

| Element | Mobile (xs-sm) | Tablet (md-lg) | Desktop (xl+) |
|---------|---------------|----------------|----------------|
| Navigation | Bottom tab bar | Collapsible sidebar | Fixed sidebar (240px) |
| Metric cards | 1 column, full width | 2x2 grid | 4 across |
| Charts | Full width, stacked | 2 column grid | 2 column grid |
| Practice table | Card view (stacked) | Table with horizontal scroll | Full table |
| Practice detail | Stacked tabs (accordion) | Tab navigation | Tab navigation + sidebar |
| Alert cards | Full width, stacked | 2 column | 3 column |
| Pipeline kanban | Horizontal scroll, 1 column visible | 2 columns visible | All columns visible |
| Activity feed | Below charts | Below charts | Right sidebar (300px) |

### Critical Mobile Considerations

1. **Chat is the primary mobile interface.** Office managers will use the concierge on their phones between patients. The chat must be thumb-friendly: large tap targets (min 44x44px), bottom-anchored input, no small buttons.

2. **Dashboard is secondary on mobile.** HQ team primarily uses desktop. Mobile dashboard is for quick checks (health scores, alerts) not detailed management.

3. **Template preview on mobile**: Use the mobile preview by default (since the practice manager is on mobile, show them what their patients will see on mobile). Desktop preview is opt-in via toggle.

4. **Offline handling**: Chat messages typed while offline are queued and sent when reconnected. Show a "Messages will send when connected" banner. Dashboard shows stale data with a "Last updated X minutes ago" indicator.

5. **Keyboard handling**: On mobile, the message input must handle keyboard appearance gracefully. The message list scrolls up when the keyboard appears. The input stays anchored above the keyboard. Use `visualViewport` API for iOS safe area handling.

---

## 8. Design Tokens and Theme

### Color System

```css
:root {
  /* SGA Brand */
  --color-brand-primary: #1e3a5f;     /* SGA navy */
  --color-brand-secondary: #4a90d9;   /* SGA blue */
  --color-brand-accent: #f59e0b;      /* Amber (CTAs, badges) */

  /* Semantic */
  --color-success: #10b981;           /* Emerald 500 */
  --color-warning: #f59e0b;           /* Amber 500 */
  --color-error: #ef4444;             /* Red 500 */
  --color-info: #3b82f6;              /* Blue 500 */

  /* Health tiers */
  --color-health-optimized: #10b981;  /* Green */
  --color-health-baseline: #f59e0b;   /* Yellow */
  --color-health-at-risk: #ef4444;    /* Red */

  /* Enrollment status */
  --color-status-pending: #9ca3af;    /* Gray */
  --color-status-invited: #3b82f6;    /* Blue */
  --color-status-in-progress: #f59e0b; /* Amber */
  --color-status-enrolled: #10b981;   /* Green */
  --color-status-blitz: #8b5cf6;      /* Purple */

  /* Chat */
  --color-chat-user-bg: #1e3a5f;      /* Brand primary */
  --color-chat-user-text: #ffffff;
  --color-chat-assistant-bg: #f3f4f6; /* Gray 100 */
  --color-chat-assistant-text: #111827; /* Gray 900 */
  --color-chat-card-bg: #ffffff;
  --color-chat-card-border: #e5e7eb;

  /* Neutrals */
  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  /* Typography */
  --font-sans: 'Inter', -apple-system, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  --text-xs: 0.75rem;          /* 12px - timestamps, metadata */
  --text-sm: 0.875rem;         /* 14px - secondary text, table cells */
  --text-base: 1rem;           /* 16px - body text, chat messages */
  --text-lg: 1.125rem;         /* 18px - section headers */
  --text-xl: 1.25rem;          /* 20px - page subtitles */
  --text-2xl: 1.5rem;          /* 24px - page titles */
  --text-3xl: 1.875rem;        /* 30px - dashboard metric labels */
  --text-4xl: 2.25rem;         /* 36px - hero metric numbers */

  /* Spacing (4px base) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;

  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);

  /* Layout */
  --sidebar-width: 240px;
  --chat-sidebar-width: 320px;
  --chat-max-width: 720px;
  --chat-bubble-max-width: 70%;
  --action-card-max-width: 600px;
  --tap-target-min: 44px;
  --input-height: 48px;
  --input-height-desktop: 56px;
}
```

---

## 9. Performance Targets

| Metric | Target |
|--------|--------|
| Chat initial load (LCP) | < 1.5s |
| Message send to first token | < 800ms |
| Template preview load | < 2s |
| Dashboard initial load | < 2s |
| Practice list (260+ rows) | < 1s with virtualization |
| Mobile Lighthouse score | > 90 |
| Cumulative Layout Shift | < 0.1 |
| First Input Delay | < 100ms |

### Optimization Strategies

- **React Query stale-while-revalidate** for dashboard data (show cached, refresh in background)
- **Virtual scrolling** for practice list (260+ rows) using `@tanstack/react-virtual`
- **Lazy load** template preview images and iframe content
- **SSE** for chat streaming (not polling, not WebSocket)
- **Prefetch** next enrollment step data while current step is active
- **Code splitting** per route using Next.js dynamic imports
- **Image optimization** via Next.js `Image` component with Cloudflare R2 as origin
- **Service worker** for offline chat history cache (PWA-lite)

---

## 10. Phase 1 Scope Summary

### In Scope

| Surface | Features |
|---------|----------|
| Concierge Chat | Message streaming, action cards (confirm/reject), data verification cards, template picker with preview, enrollment step indicator, quick action chips, mobile-first |
| HQ Dashboard | Login, overview page (enrollment + health metrics), practice list with filters, practice detail (all tabs), enrollment pipeline kanban, alerts list |
| Executive View | Read-only summary with enrollment progress, health scores, tier distribution, top alerts |

### Out of Scope (Phase 2)

| Feature | Reason |
|---------|--------|
| SMS/voice chat channels | Web chat proves the pattern first |
| Content approval flows | Requires social media integration |
| Batch operations UI (template-wide changes) | Needs more templates and channel integrations |
| Real-time WebSocket updates | SSE is sufficient for Phase 1 chat; dashboard uses polling |
| Dark mode | Nice-to-have, not MVP |
| Practice self-service portal | Concierge chat handles all practice interactions |
| Advanced analytics/charts | Detailed performance analytics deferred |
| Push notifications (email/browser) | HQ team checks dashboard manually for Phase 1 |
| Review management interface | Requires review aggregation infrastructure |

### Phase 1 Delivery Schedule

| Week | Deliverable |
|------|-------------|
| 1 | Project scaffold, design tokens, shared component library (`packages/ui`: buttons, cards, inputs, badges, pills, avatars) |
| 2 | ChatInterface with SSE streaming, MessageBubble, TypingIndicator, MessageInput, QuickActions |
| 3 | ActionCard variants (confirmation, data verification, template picker), InlinePreview, StepIndicator |
| 4 | Dashboard: login, overview page with charts, practice list with filters, practice detail with tabs, enrollment pipeline, alerts |

---

## 11. Technical Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| Next.js | 14.x | App framework (App Router) |
| React | 18.x | UI library |
| TypeScript | 5.4+ | Type safety |
| Tailwind CSS | 3.x | Utility-first styling |
| @tanstack/react-query | 5.x | Server state management |
| @tanstack/react-virtual | 3.x | Virtual scrolling for practice list |
| zustand | 4.x | Client state management |
| recharts | 2.x | Dashboard charts (pie, bar, radar, sparkline) |
| date-fns | 3.x | Date formatting and relative time |
| clsx | 2.x | Conditional class names |
| tailwind-merge | 2.x | Merge Tailwind classes without conflicts |
| @radix-ui/react-* | latest | Accessible primitives (dialog, dropdown, tabs, tooltip, accordion) |
| react-markdown | 9.x | Render assistant markdown in chat bubbles |
| react-hot-toast | 2.x | Toast notifications (success, error, info) |
| iron-session | 8.x | Session management (dashboard auth) |
| zod | 3.x | Runtime validation for API responses and form data |
| react-hook-form | 7.x | Form management (dashboard forms) |

---

## 12. Accessibility Requirements

| Requirement | Implementation |
|-------------|---------------|
| Keyboard navigation | All interactive elements reachable via Tab. Action card buttons have focus rings. Enter/Space to activate. |
| Screen reader support | Chat messages use `role="log"` and `aria-live="polite"`. Action cards have descriptive `aria-label`. Status badges use `aria-label` not color alone. |
| Color contrast | All text meets WCAG AA (4.5:1 for normal text, 3:1 for large text). Health score colors paired with text labels. |
| Focus management | When action card appears, focus moves to the card. When modal opens, focus is trapped. On modal close, focus returns to trigger. |
| Reduced motion | Respect `prefers-reduced-motion`: disable typing indicator animation, step indicator pulse, chart transitions. |
| Touch targets | Minimum 44x44px for all interactive elements on mobile. Buttons have adequate spacing (8px minimum between adjacent targets). |
