# Phase 1 Frontend Specification: The Practice Concierge

## Overview

Phase 1 delivers two interfaces: the **Practice Concierge Chat** (practice-facing enrollment flow) and the **HQ Enrollment Dashboard** (admin view of enrollment progress + QC queue). The Executive Dashboard is Phase 2.

**Stack**: Next.js 14 (App Router) + TypeScript + Tailwind CSS + React Query + Zustand

---

## Routes

| Route | Interface | Description |
|-------|-----------|-------------|
| `/chat/:practiceSlug` | Practice Concierge | Conversational enrollment + ongoing interactions |
| `/chat/:practiceSlug/preview` | Practice Concierge | Full-page storefront preview (website, GMB, social) |
| `/admin` | HQ Dashboard | Network enrollment overview + practice list |
| `/admin/practice/:id` | HQ Dashboard | Practice detail drill-down |
| `/admin/qc` | HQ Dashboard | QC review queue |
| `/admin/batch` | HQ Dashboard | Batch operations |
| `/login` | Auth | HQ team authentication |

---

## Component Tree

### Practice Concierge Chat (`/chat/:practiceSlug`)

```
ChatLayout
├── ChatHeader
│   ├── PracticeName
│   ├── ConnectionStatus (online indicator)
│   └── HelpMenu
├── MessageList
│   ├── MessageBubble (text messages)
│   │   ├── AgentMessage (left-aligned, concierge avatar)
│   │   └── UserMessage (right-aligned)
│   ├── DataVerificationCard (structured data with edit/confirm)
│   │   ├── FieldRow (label + value + edit button)
│   │   ├── TeamMemberList (with add/remove)
│   │   └── ConfirmButton
│   ├── TemplatePickerCard (2-3 template previews)
│   │   ├── TemplatePreview (thumbnail + name + recommendation badge)
│   │   ├── RecommendationReasoning (why the agent suggests this one)
│   │   └── SelectButton
│   ├── StorefrontPreviewCard (full package review)
│   │   ├── PreviewTabs (Website | GMB | Social | Lead Capture)
│   │   ├── DeviceSwitcher (mobile | desktop)
│   │   └── ApproveRejectButtons
│   ├── ActionCard (generic approve/reject/flag)
│   │   ├── ActionTitle
│   │   ├── ActionDescription
│   │   └── ActionButtons (primary + secondary + flag)
│   ├── ProgressIndicator (enrollment step tracker)
│   │   └── StepDot (completed | active | upcoming)
│   └── TypingIndicator (animated dots)
├── ChatInput
│   ├── TextInput (auto-expanding textarea)
│   ├── QuickReplies (contextual suggestion chips)
│   │   └── QuickReplyChip
│   ├── PhotoUpload (for team photos)
│   └── SendButton
└── EnrollmentProgress (sticky bottom bar showing current step)
```

### HQ Dashboard (`/admin`)

```
AdminLayout
├── AdminNav
│   ├── Logo
│   ├── NavLinks (Overview | Practices | QC Queue | Batch)
│   └── UserMenu
├── OverviewPage (/admin)
│   ├── StatsGrid
│   │   ├── StatCard (Total Practices)
│   │   ├── StatCard (Enrolled)
│   │   ├── StatCard (Blitz Deployed)
│   │   └── StatCard (Pending)
│   ├── EnrollmentProgress (bar chart or funnel)
│   ├── RecentActivity (timeline of recent enrollments/changes)
│   └── AlertsList (practices needing attention)
├── PracticeListPage (/admin/practices)
│   ├── SearchBar
│   ├── FilterBar (status, platform, specialty, health tier)
│   ├── PracticeTable
│   │   └── PracticeRow (name, status, health score, last interaction, actions)
│   └── Pagination
├── PracticeDetailPage (/admin/practice/:id)
│   ├── PracticeHeader (name, status badge, platform tag)
│   ├── TabNav (Overview | Digital | Concierge | History)
│   ├── OverviewTab
│   │   ├── PracticeInfo (address, phone, specialty, team)
│   │   ├── HealthScoreCard (composite + breakdown)
│   │   └── EnrollmentTimeline
│   ├── DigitalTab
│   │   ├── WebsiteStatus (template, URL, performance metrics)
│   │   ├── GMBStatus (completeness, last post, review summary)
│   │   └── SocialStatus (handles, last post dates, follower counts)
│   ├── ConciergeTab
│   │   ├── ConversationLog (recent messages)
│   │   ├── InteractionStats (frequency, satisfaction)
│   │   └── PendingActions (items awaiting practice response)
│   └── HistoryTab
│       └── ChangeLog (all changes with timestamps and sources)
├── QCQueuePage (/admin/qc)
│   ├── QueueStats (pending, approved today, flagged)
│   ├── QueueList
│   │   └── QueueItem (practice, type, preview, approve/flag buttons)
│   └── QueueFilters (type: site, social, GMB)
└── BatchPage (/admin/batch)
    ├── OperationSelector (template update, content push, data sync)
    ├── PracticeSelector (select all, filter, individual)
    ├── PreviewPanel (show affected practices + change preview)
    └── ExecuteButton (with confirmation modal)
```

---

## State Management

### Server State (React Query)

```typescript
// Practice data
const usePractice = (slug: string) => useQuery(['practice', slug], fetchPractice)
const usePractices = (filters: PracticeFilters) => useQuery(['practices', filters], fetchPractices)

// Conversation
const useConversation = (practiceSlug: string) => useQuery(
  ['conversation', practiceSlug], fetchConversation
)
const useSendMessage = () => useMutation(sendMessage, {
  onSuccess: () => queryClient.invalidateQueries(['conversation'])
})

// Enrollment
const useEnrollmentStatus = (practiceSlug: string) => useQuery(
  ['enrollment', practiceSlug], fetchEnrollmentStatus
)

// Dashboard
const useNetworkStats = () => useQuery(['network-stats'], fetchNetworkStats)
const useQCQueue = () => useQuery(['qc-queue'], fetchQCQueue)
const useAlerts = () => useQuery(['alerts'], fetchAlerts)

// Mutations
const useApproveStorefront = () => useMutation(approveStorefront)
const useSelectTemplate = () => useMutation(selectTemplate)
const useVerifyData = () => useMutation(verifyPracticeData)
const useBatchOperation = () => useMutation(executeBatchOperation)
```

### Client State (Zustand)

```typescript
interface ChatStore {
  isInputFocused: boolean
  activePreview: 'website' | 'gmb' | 'social' | null
  previewDevice: 'mobile' | 'desktop'
  quickRepliesVisible: boolean
  uploadingPhoto: boolean
  setInputFocused: (focused: boolean) => void
  setActivePreview: (preview: string | null) => void
  togglePreviewDevice: () => void
}

interface AdminStore {
  selectedPractices: string[]
  activeFilters: PracticeFilters
  qcSortOrder: 'newest' | 'oldest' | 'priority'
  sidebarCollapsed: boolean
  togglePracticeSelection: (id: string) => void
  setFilters: (filters: PracticeFilters) => void
}
```

---

## Key Component Specifications

### ChatInterface

The primary practice-facing component. Must handle streaming responses, action cards, and inline previews.

```typescript
interface ChatMessage {
  id: string
  role: 'agent' | 'user' | 'system'
  type: 'text' | 'data-verification' | 'template-picker' | 'storefront-preview' | 'action-card' | 'progress'
  content: string
  metadata?: {
    verificationData?: PracticeData
    templates?: Template[]
    recommendation?: string
    recommendationReasoning?: string
    storefrontPreview?: StorefrontPreview
    actionType?: string
    enrollmentStep?: number
  }
  timestamp: string
}
```

**Streaming behavior**: Agent responses stream token-by-token using Server-Sent Events. The typing indicator shows during the initial connection, then transitions to streaming text. Action cards render after the full response is received.

**Quick replies**: Context-sensitive chips appear above the input based on the current enrollment step:
- Verification: "Looks correct", "I need to make changes", "Skip for now"
- Template selection: "Show me more options", "I like the recommendation", "Can I customize?"
- Approval: "Approve all", "I have feedback", "Schedule for later"

### TemplatePickerCard

```typescript
interface TemplatePickerProps {
  templates: Template[]
  recommendedId: string
  recommendationReasoning: string
  onSelect: (templateId: string) => void
  onPreview: (templateId: string) => void
}
```

Renders 2-3 template options as cards with:
- Template thumbnail (practice data pre-populated, not placeholder)
- Template name + specialty tag
- Recommendation badge ("Recommended" with green accent) on the suggested option
- Reasoning text below the recommended card (e.g., "Perio sites convert 23% better with referral-optimized layouts")
- "Preview" button (opens full-page preview in modal/slide-over)
- "Select" button (primary CTA)

### DataVerificationCard

```typescript
interface DataVerificationProps {
  sections: VerificationSection[] // team, services, contact, hours
  confidenceScores: Record<string, 'high' | 'medium' | 'low'>
  onConfirm: (verified: PracticeData) => void
  onEdit: (field: string, value: string) => void
}
```

Renders harvested data in editable sections. Each field shows:
- Current value (from harvester)
- Confidence indicator (green dot = high, yellow = medium, red = low)
- Inline edit button (pencil icon → expands to text input)
- For team members: add/remove buttons, photo upload slot

### HealthScoreCard (HQ Dashboard)

```typescript
interface HealthScoreProps {
  overall: number // 0-100
  breakdown: {
    website: number
    gmb: number
    social: number
    reviews: number
    leads: number
  }
  trend: 'up' | 'down' | 'stable'
}
```

Circular progress indicator for overall score (color-coded: green 80+, yellow 60-79, red <60). Horizontal bar breakdown for each category. Trend arrow with percentage change.

---

## Chat UX Patterns

### Enrollment Progress Bar
Sticky bar at bottom of chat showing enrollment steps:
```
[1. Verify] → [2. Template] → [3. Review] → [4. Approve] → [5. Live!]
```
Current step is highlighted. Completed steps show checkmarks. Tapping a completed step scrolls to that conversation point.

### Action Cards
Rich interactive cards embedded in the chat:
- **Single-action**: "Approve" button (green) with optional "Flag" (orange)
- **Multi-choice**: 2-3 options as cards (like template picker)
- **Confirmation**: Summary of changes + "Confirm" / "Edit" buttons
- All actions send a message to the conversation log for auditability

### Streaming Responses
```
1. User sends message → input clears, message appears right-aligned
2. Typing indicator appears (3 animated dots, left-aligned with agent avatar)
3. SSE connection opens → typing indicator transitions to streaming text
4. Text streams token-by-token into an agent message bubble
5. If response includes an action card → card renders after text completes
6. Quick reply chips update based on new context
```

---

## Responsive Breakpoints

| Breakpoint | Target | Layout Changes |
|------------|--------|----------------|
| 375px | Mobile (phone) | Single column, bottom-anchored input, full-width cards |
| 768px | Tablet | Chat gets wider margins, admin sidebar appears |
| 1024px | Desktop | Admin gets full sidebar + content layout |
| 1440px | Wide desktop | Max-width container, extra whitespace |

### Mobile-Specific Patterns
- **Chat**: Input pinned to bottom, keyboard pushes content up, 44px minimum touch targets
- **Template picker**: Horizontal scroll cards (swipe to compare) instead of grid
- **Data verification**: Accordion sections (tap to expand/collapse) instead of all-visible
- **Admin dashboard**: Stats stack vertically, practice table becomes card list

---

## Phase 1 Scope

### Included
- Practice Concierge Chat (full enrollment flow: welcome → verify → template → review → approve → live)
- Streaming responses with typing indicator
- Data verification cards with inline editing
- Template picker with recommendation + preview
- Storefront preview (website only, GMB + social in Phase 2)
- Enrollment progress indicator
- HQ Dashboard: enrollment overview + practice list + practice detail
- QC Queue: basic approve/flag workflow
- Mobile-responsive across all views
- Authentication for HQ team

### Excluded (Phase 2+)
- Executive dashboard (health scores, KPIs, network-wide metrics)
- Batch operations interface
- Change management flows in chat
- Social/GMB preview in storefront review
- Voice channel for concierge
- Push notifications
- Content approval workflow
- Review management interface

---

## API Integration

### Chat → Backend

```typescript
// Send message
POST /api/conversations/:practiceSlug/messages
Body: { content: string, type: 'text' | 'action' }
Response: SSE stream of agent response tokens

// Get conversation history
GET /api/conversations/:practiceSlug
Response: { messages: ChatMessage[], enrollmentStep: number }

// Submit data verification
POST /api/practices/:slug/verify
Body: { verified: PracticeData, corrections: Correction[] }

// Select template
POST /api/practices/:slug/template
Body: { templateId: string }

// Approve storefront
POST /api/practices/:slug/approve
Body: { approved: boolean, feedback?: string }
```

### Admin → Backend

```typescript
// Network stats
GET /api/admin/stats
Response: { total, enrolled, blitzDeployed, pending, healthAvg }

// Practice list
GET /api/admin/practices?status=enrolled&specialty=perio&page=1
Response: { practices: Practice[], total: number }

// QC queue
GET /api/admin/qc-queue
Response: { items: QCItem[], pendingCount: number }

// Approve QC item
POST /api/admin/qc/:itemId/approve
Body: { approved: boolean, notes?: string }
```

---

## Performance Targets

| Metric | Target |
|--------|--------|
| Chat initial load (LCP) | < 1.5s |
| Message send → first token | < 800ms |
| Template preview load | < 2s |
| Admin dashboard load | < 2s |
| Practice list (260+ rows) | < 1s with virtualization |
| Mobile Lighthouse score | > 90 |

### Optimization Strategies
- React Query stale-while-revalidate for dashboard data
- Virtual scrolling for practice list (260+ rows)
- Lazy load template preview images
- SSE for chat streaming (not polling)
- Prefetch next enrollment step data while current step is active
- Service worker for offline chat history cache
