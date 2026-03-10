# Phase 1 Frontend Specification — The Comms Engine

**Role**: Frontend Engineer
**Sprint**: Patient Communications Automation
**Phase**: Stage 1 — Supervised Mode Approval Interface (Practice-Facing PWA)
**Date**: 2026-03-10
**Source Artifacts**: `03-decide/storyboard.md` (Step 7, Step 9), `04-architect/architecture.md` (Layer 5), UX Design Specs

---

## Phase 1 Scope

Stage 1 frontend delivers:
1. **Comms Engine module** added to the existing Practice Concierge PWA (new tab in the tab bar)
2. **Daily Digest view** — list of pending messages for supervised mode approval
3. **Message Approval Card** — individual message review with approve/edit/reject
4. **Message Edit Drawer** — inline editing for supervised mode modifications
5. **Weekly Summary view** — read-only 3-section weekly summary
6. **Autonomy Status screen** — current autonomy levels for each playbook stage

Phase 1 does NOT include: HQ Playbook Manager (separate app, Stage 5), Graduation Report screen (Stage 2), Reactivation Campaign Preview (Stage 4).

---

## Component Tree

```
App (extends Practice Concierge)
├── PracticeAuthProvider (SHARED — existing)
├── Router
│   ├── /concierge/*       ← EXISTING Practice Concierge routes (unchanged)
│   └── /comms/*           ← NEW Comms Engine module
│       ├── /comms         → CommsEngineLayout (default: DailyDigestView)
│       ├── /comms/digest  → DailyDigestView
│       ├── /comms/summary → WeeklySummaryView
│       └── /comms/status  → AutonomyStatusView
│
└── CommsEngineLayout
    ├── TabBar (SHARED — existing, extended with "Comms" tab)
    ├── CommsEngineModule
    │   ├── DailyDigestView
    │   │   ├── DigestHeader
    │   │   │   ├── PracticeNameLabel
    │   │   │   ├── DigestDateLabel
    │   │   │   └── AutonomyBadge
    │   │   ├── DigestSummaryBanner
    │   │   ├── PendingMessageList
    │   │   │   └── PendingMessageCard (× N)
    │   │   │       ├── CardHeader (stage label, patient display, touch number)
    │   │   │       ├── MessagePreview (truncated, expandable)
    │   │   │       ├── CardActions (ApproveButton, EditButton, RejectButton)
    │   │   │       └── CardExpandedContent (full message, patient context, scheduled time)
    │   │   ├── ApproveAllButton
    │   │   └── DigestEmptyState
    │   │
    │   ├── MessageEditDrawer (overlay, slides up from bottom)
    │   │   ├── DrawerHeader (patient display, stage label)
    │   │   ├── MessageTextField (multiline editable)
    │   │   ├── CharacterCountIndicator
    │   │   └── DrawerActions (CancelButton, SaveAndApproveButton)
    │   │
    │   ├── WeeklySummaryView
    │   │   ├── SummaryHeader (date range)
    │   │   ├── SummarySection × 3 (Engine Activity / Needs Attention / Coming This Week)
    │   │   │   ├── SectionTitle
    │   │   │   ├── SectionItems (list)
    │   │   │   └── AttentionItemCard (expandable, for "Needs Attention" items only)
    │   │   └── SummaryEmptyState
    │   │
    │   └── AutonomyStatusView
    │       ├── StatusHeader
    │       └── AutonomyStageCard × 4 (Lead / Treatment / Rebooking / Reactivation)
    │           ├── StageName
    │           ├── AutonomyBadge
    │           └── StageDescription
    │
    └── ApproveAllConfirmationSheet (bottom sheet modal)
        ├── ConfirmationCount
        └── ConfirmActions (CancelButton, ConfirmApproveButton)
```

---

## Routes

| Path | Component | Description |
|------|-----------|-------------|
| `/comms` | `DailyDigestView` | Default comms view — daily digest (today's pending approvals) |
| `/comms/digest` | `DailyDigestView` | Same as `/comms` (canonical path) |
| `/comms/summary` | `WeeklySummaryView` | Read-only weekly summary |
| `/comms/status` | `AutonomyStatusView` | Current autonomy levels for all 4 stages |

Navigation source: TabBar "Comms" tab links to `/comms`. Weekly Summary can also be reached via deep link from the push notification.

---

## State Management (Zustand)

### digestStore

```typescript
interface DigestStore {
  // Data
  approvals: PendingApproval[];
  digestDate: string;
  autoSendAt: string | null;
  isLoading: boolean;
  error: string | null;

  // Edit drawer state
  editingApproval: PendingApproval | null;
  editDraftText: string;

  // Actions
  fetchDigest: (practiceId: string) => Promise<void>;
  approveMessage: (approvalId: string) => Promise<void>;
  editMessage: (approvalId: string, editedContent: string) => Promise<void>;
  rejectMessage: (approvalId: string) => Promise<void>;
  approveAll: (practiceId: string) => Promise<void>;
  openEditDrawer: (approval: PendingApproval) => void;
  closeEditDrawer: () => void;
  updateEditDraft: (text: string) => void;
}
```

### summaryStore

```typescript
interface SummaryStore {
  summary: WeeklySummary | null;
  isLoading: boolean;
  error: string | null;
  fetchSummary: (practiceId: string) => Promise<void>;
}

interface WeeklySummary {
  periodStart: string;
  periodEnd: string;
  engineActivity: EngineActivitySection;
  attentionItems: AttentionItem[];
  comingThisWeek: ComingItem[];
}
```

### agentStore

```typescript
interface AgentStore {
  agentStatus: AgentStatus | null;
  isLoading: boolean;
  fetchAgentStatus: (practiceId: string) => Promise<void>;
}

interface AgentStatus {
  leadResponseLevel: AutonomyLevel;
  treatmentFollowupLevel: AutonomyLevel;
  rebookingLevel: AutonomyLevel;
  reactivationLevel: AutonomyLevel;
  lastActivity: string;
  activeSequences: Record<string, number>;
}

type AutonomyLevel = 'supervised' | 'monitored' | 'autonomous';
```

---

## Component Specifications

### `DailyDigestView`

**Purpose**: Main practice-facing view. Shows today's pending message approvals.

**Props**: None (reads from `digestStore` + `usePracticeContext()`)

**Behavior**:
- On mount: calls `digestStore.fetchDigest(practiceId)`.
- If `approvals.length === 0`: renders `DigestEmptyState` ("No messages pending review today").
- If `approvals.length > 0`: renders `DigestSummaryBanner` + `PendingMessageList` + `ApproveAllButton`.
- `ApproveAllButton` is disabled until at least 1 approval is pending.
- When `approveAll` is called: shows `ApproveAllConfirmationSheet` first.

**Edge cases**:
- `autoSendAt` is in the past (user opens digest late): banner text changes to "These messages were auto-sent at [time]. No action needed."
- Network error on fetch: shows error state with retry button.
- All messages already resolved: shows `DigestEmptyState`.

---

### `PendingMessageCard`

**Purpose**: Displays a single pending message with patient context and approval actions.

**Props**:
```typescript
interface PendingMessageCardProps {
  approval: PendingApproval;
  onApprove: (id: string) => void;
  onEdit: (approval: PendingApproval) => void;
  onReject: (id: string) => void;
}

interface PendingApproval {
  id: string;
  patientDisplay: string;     // "James H." — first name + last initial only
  stage: 'lead_response' | 'treatment_followup' | 'rebooking' | 'reactivation';
  touchNumber: number;
  channel: 'sms' | 'email' | 'voice';
  proposedMessage: string;
  scheduledSendAt: string;
  autoSendAt: string;
  status: 'pending' | 'approved' | 'edited' | 'rejected' | 'auto_sent';
}
```

**Internal state**:
- `isExpanded: boolean` — toggles full message vs. truncated preview
- `resolvedState: 'approved' | 'rejected' | null` — for post-action visual feedback

**Behavior**:
- Default: truncated to 3 lines with "See full message" tap target.
- Tap card body: expands to show full message, patient context row, scheduled send time.
- Swipe right: triggers approve (with haptic feedback on iOS).
- Swipe left: triggers reject.
- Post-approve: card shows green checkmark background, message "Approved — sending at [time]". Does not disappear immediately (gives user visual confirmation).
- Post-reject: card shows gray strikethrough state, message "Rejected — not sending."
- Post-edit: card shows "Edited — sending your version at [time]."

**Stage label display**:
- `lead_response` → "Lead Response"
- `treatment_followup` → "Treatment Follow-Up · Day {touchNumber}"
- `rebooking` → "Rebooking"
- `reactivation` → "Reactivation"

---

### `MessageEditDrawer`

**Purpose**: Slide-up drawer for editing a pending message before approving.

**Props**:
```typescript
interface MessageEditDrawerProps {
  isOpen: boolean;
  approval: PendingApproval | null;
  draftText: string;
  onDraftChange: (text: string) => void;
  onSaveAndApprove: () => void;
  onCancel: () => void;
}
```

**Behavior**:
- Slides up from bottom when `isOpen = true`. Dismissible via swipe down or "Cancel".
- `MessageTextField`: multiline, auto-grows to max 6 lines.
- `CharacterCountIndicator`: shows `{current}/{limit}`. SMS limit = 160 chars. Color: green 0–140, amber 141–159, red 160+. At 160+: shows MMS warning.
- `SaveAndApproveButton`: enabled only when `draftText !== approval.proposedMessage` (must actually edit) and `draftText.trim().length > 0`.
- Keyboard avoidance: drawer shifts up when iOS keyboard appears.

**Accessibility**: `aria-label="Edit message for {approval.patientDisplay}"` on drawer. `role="dialog"`.

---

### `DigestSummaryBanner`

**Purpose**: Shows context about the digest: count, auto-send deadline.

**Props**:
```typescript
interface DigestSummaryBannerProps {
  pendingCount: number;
  autoSendAt: string;  // ISO timestamp
}
```

**Behavior**:
- Primary text: `"{pendingCount} message{s} pending your review"`
- Secondary text: `"Auto-sends at {time} if not reviewed"`
- If `autoSendAt < now + 30min`: secondary text becomes amber: `"Auto-sends in {X} minutes"`
- If `autoSendAt < now`: banner turns green: `"All messages were auto-sent at {time}. No action needed."`

---

### `WeeklySummaryView`

**Purpose**: Read-only display of the AI-generated weekly summary.

**Props**: None (reads from `summaryStore`)

**Behavior**:
- On mount: fetches weekly summary from `summaryStore`.
- Renders 3 sections: Engine Activity, Needs Attention, Coming This Week.
- "Needs Attention" section: if `attentionItems.length === 0`, shows: `"No action needed this week. Everything's running."`
- Each `AttentionItemCard` is expandable: shows full patient context on tap.
- No editing capability — this is read-only.
- No navigation away from this screen is required to act on any item (phone number is tappable for direct call).

---

### `AutonomyBadge`

**Purpose**: Reusable color-coded badge showing autonomy level.

**Props**:
```typescript
interface AutonomyBadgeProps {
  level: 'supervised' | 'monitored' | 'autonomous';
  size?: 'sm' | 'md';
}
```

**Rendering**:
- `supervised`: red dot + "Supervised" text
- `monitored`: amber dot + "Monitored" text
- `autonomous`: green dot + "Autonomous" text
- Dot is always paired with text label (never color-only — accessibility requirement)

---

### `AutonomyStatusView`

**Purpose**: Shows current autonomy levels for all 4 playbook stages.

**Props**: None (reads from `agentStore`)

**Renders**: 4 stage cards in vertical list.

```
┌──────────────────────────────────────┐
│ Lead Response                        │
│ ● Monitored                          │
│ Auto-sends immediately. You'll see   │
│ what was sent in your daily digest.  │
└──────────────────────────────────────┘
┌──────────────────────────────────────┐
│ Treatment Follow-Up                  │
│ ● Supervised                         │
│ Messages wait for your review before │
│ sending. Auto-sends in 2hrs if no   │
│ action.                              │
└──────────────────────────────────────┘
```

---

### `ApproveAllConfirmationSheet`

**Purpose**: Bottom sheet confirmation before bulk approving all messages.

**Props**:
```typescript
interface ApproveAllConfirmationSheetProps {
  isOpen: boolean;
  pendingCount: number;
  onConfirm: () => void;
  onCancel: () => void;
}
```

**Renders**: `"Approve all {N} messages? They will send as shown."` + Cancel + Confirm buttons.

---

## API Integration

All API calls use the shared `apiClient` from Practice Concierge (base URL, auth headers, error handling).

### React Query hooks (new, Comms Engine module)

```typescript
// hooks/useDigest.ts
export function useDigest(practiceId: string) {
  return useQuery({
    queryKey: ['digest', practiceId],
    queryFn: () => apiClient.get<DigestResponse>(`/practices/${practiceId}/approvals/digest`),
    staleTime: 60_000,   // 1 minute
    refetchOnWindowFocus: true,
  });
}

// hooks/useApprovalDecide.ts
export function useApprovalDecide(practiceId: string) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ approvalId, decision, editedContent }: ApprovalDecision) =>
      apiClient.post(`/practices/${practiceId}/approvals/${approvalId}/decide`, {
        decision, edited_content: editedContent
      }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['digest', practiceId] });
    }
  });
}

// hooks/useBulkApprove.ts
export function useBulkApprove(practiceId: string) {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: () => apiClient.post(`/practices/${practiceId}/approvals/bulk-approve`, {}),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['digest', practiceId] });
    }
  });
}

// hooks/useWeeklySummary.ts
export function useWeeklySummary(practiceId: string) {
  return useQuery({
    queryKey: ['summary', practiceId],
    queryFn: () => apiClient.get<WeeklySummary>(`/practices/${practiceId}/summary/weekly`),
    staleTime: 3_600_000,  // 1 hour (summary changes once/week)
  });
}

// hooks/useAgentStatus.ts
export function useAgentStatus(practiceId: string) {
  return useQuery({
    queryKey: ['agent-status', practiceId],
    queryFn: () => apiClient.get<AgentStatus>(`/practices/${practiceId}/agent`),
    staleTime: 300_000,    // 5 minutes
  });
}
```

### Offline action queue (IndexedDB via Dexie.js)

```typescript
// For offline approval actions (office manager in operatory without signal)
interface OfflineAction {
  id: string;
  type: 'approve' | 'edit' | 'reject' | 'bulk_approve';
  practiceId: string;
  approvalId?: string;
  editedContent?: string;
  queuedAt: string;
  synced: boolean;
}

// On reconnect: sync pending offline actions
async function syncOfflineActions() {
  const pending = await db.offlineActions.where('synced').equals(false).toArray();
  for (const action of pending) {
    await executeApprovalAction(action);
    await db.offlineActions.update(action.id, { synced: true });
  }
}
```

---

## TypeScript Interfaces

```typescript
// Autonomy
type AutonomyLevel = 'supervised' | 'monitored' | 'autonomous';
type PlaybookStage = 'lead_response' | 'treatment_followup' | 'rebooking' | 'reactivation';
type ApprovalDecisionType = 'approved' | 'edited' | 'rejected';
type MessageChannel = 'sms' | 'email' | 'voice';

// Digest
interface DigestResponse {
  practiceId: string;
  practiceName: string;
  digestDate: string;       // ISO date
  autoSendAt: string;       // ISO timestamp
  pendingCount: number;
  approvals: PendingApproval[];
}

interface PendingApproval {
  id: string;
  patientDisplay: string;
  stage: PlaybookStage;
  touchNumber: number;
  channel: MessageChannel;
  proposedMessage: string;
  scheduledSendAt: string;
  autoSendAt: string;
  status: 'pending' | 'approved' | 'edited' | 'rejected' | 'auto_sent';
}

// Weekly Summary
interface WeeklySummary {
  periodStart: string;
  periodEnd: string;
  engineActivity: {
    messagesSent: number;
    breakdown: Record<PlaybookStage, number>;
    results: {
      appointmentsBooked: number;
      treatmentPlansAccepted: number;
      treatmentRevenueEstimate: number;
      patientsRebooked: number;
      dormantPatientsScheduled: number;
    };
    estimatedProductionInfluenced: number;
  };
  attentionItems: AttentionItem[];
  comingThisWeek: ComingItem[];
}

interface AttentionItem {
  id: string;
  patientDisplay: string;
  summary: string;
  actionPrompt: string;
  phoneNumber?: string;
  context?: string;
}

interface ComingItem {
  description: string;
  count: number;
  stage: PlaybookStage;
}

// Agent Status
interface AgentStatus {
  practiceId: string;
  leadResponseLevel: AutonomyLevel;
  treatmentFollowupLevel: AutonomyLevel;
  rebookingLevel: AutonomyLevel;
  reactivationLevel: AutonomyLevel;
  lastActivity: string;
  activeSequences: {
    lead_response: number;
    treatment_followup: number;
    rebooking: number;
    reactivation: number;
  };
}
```

---

## PWA Configuration

### Service Worker (Workbox)

```javascript
// sw.js (Workbox configuration)
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { CacheFirst, StaleWhileRevalidate, NetworkFirst } from 'workbox-strategies';
import { BackgroundSyncPlugin } from 'workbox-background-sync';

// Precache app shell (HTML, CSS, JS bundles)
precacheAndRoute(self.__WB_MANIFEST);

// Daily digest: stale-while-revalidate (show cached, fetch fresh)
registerRoute(
  ({ url }) => url.pathname.includes('/approvals/digest'),
  new StaleWhileRevalidate({
    cacheName: 'digest-cache',
    plugins: [new ExpirationPlugin({ maxAgeSeconds: 7200 })]  // 2 hours
  })
);

// Weekly summary: cache-first (static once generated)
registerRoute(
  ({ url }) => url.pathname.includes('/summary/weekly'),
  new CacheFirst({
    cacheName: 'summary-cache',
    plugins: [new ExpirationPlugin({ maxAgeSeconds: 604800 })]  // 1 week
  })
);

// Approval actions: background sync for offline queuing
const bgSyncPlugin = new BackgroundSyncPlugin('approval-queue', {
  maxRetentionTime: 24 * 60  // 24 hours
});

registerRoute(
  ({ url }) => url.pathname.includes('/approvals/') && url.pathname.includes('/decide'),
  new NetworkFirst({ plugins: [bgSyncPlugin] }),
  'POST'
);
```

### Push Notification Handler

```javascript
// In service worker
self.addEventListener('push', (event) => {
  const data = event.data.json();

  if (data.type === 'new_digest') {
    event.waitUntil(
      self.registration.showNotification('Comms Engine — Review Needed', {
        body: `${data.count} message${data.count > 1 ? 's' : ''} ready for review before ${data.autoSendTime}`,
        icon: '/icons/comms-engine-icon-192.png',
        badge: '/icons/badge-72.png',
        data: { url: '/comms/digest' },
        actions: [
          { action: 'review', title: 'Review Now' }
        ]
      })
    );
  }

  if (data.type === 'graduation_recommended') {
    event.waitUntil(
      self.registration.showNotification('Comms Engine', {
        body: `${data.stageName} is ready to advance to ${data.nextLevel} mode`,
        icon: '/icons/comms-engine-icon-192.png',
        data: { url: '/comms/status' }
      })
    );
  }
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(clients.openWindow(event.notification.data.url));
});
```

### Web App Manifest additions

```json
// Additions to existing Practice Concierge manifest.json
{
  "shortcuts": [
    {
      "name": "Today's Digest",
      "short_name": "Digest",
      "description": "Review today's pending messages",
      "url": "/comms/digest",
      "icons": [{ "src": "/icons/digest-shortcut.png", "sizes": "192x192" }]
    }
  ]
}
```

---

## Accessibility Requirements

- **Touch targets**: All interactive elements ≥ 44×44px. Approve/Edit/Reject buttons on `PendingMessageCard` must each meet this threshold.
- **Color**: Autonomy levels use colored dot + text label (never color-only). All text contrasts meet WCAG AA (4.5:1 minimum).
- **Screen reader**: `PendingMessageCard` uses `role="article"`. Approve/Edit/Reject buttons have `aria-label="Approve message for {patientDisplay}"` etc.
- **Focus management**: When `MessageEditDrawer` opens, focus moves to the text field. When it closes, focus returns to the triggering Edit button.
- **Voice Control (iOS)**: All buttons have unique visible labels or `aria-label` — no two buttons on screen have the same label.
- **Dynamic type**: All text uses relative units (`rem`) and scales with iOS Dynamic Type settings.

---

## Integration with Practice Concierge

The Comms Engine module is added as a new tab to the existing Practice Concierge PWA. Changes required to the existing codebase:

1. **TabBar** (`src/components/TabBar.tsx`): Add "Comms" tab icon and route `/comms`.
2. **Router** (`src/App.tsx`): Add `/comms/*` route group.
3. **React Query client**: No changes — Comms Engine hooks use the same client instance.
4. **Auth**: No changes — Comms Engine uses the same `usePracticeContext()` hook.
5. **Design tokens**: Add 3 new tokens (`--autonomy-autonomous`, `--autonomy-monitored`, `--autonomy-supervised`) to the shared design token file.
6. **Push notification registration**: Add Comms Engine push notification types to the existing `pushNotificationService.ts`.

**Critical constraint**: The Comms Engine module must not affect the Concierge module's bundle size or loading time. It should be lazy-loaded: `const CommsEngineModule = React.lazy(() => import('./comms/CommsEngineModule'))`.

---

## Phase 1 File Structure

```
src/
└── comms/                          ← New module
    ├── CommsEngineModule.tsx       ← Module root, lazy-loaded
    ├── views/
    │   ├── DailyDigestView.tsx
    │   ├── WeeklySummaryView.tsx
    │   └── AutonomyStatusView.tsx
    ├── components/
    │   ├── PendingMessageCard.tsx
    │   ├── MessageEditDrawer.tsx
    │   ├── DigestSummaryBanner.tsx
    │   ├── DigestEmptyState.tsx
    │   ├── ApproveAllButton.tsx
    │   ├── ApproveAllConfirmationSheet.tsx
    │   ├── AutonomyBadge.tsx
    │   ├── AutonomyStageCard.tsx
    │   ├── SummarySection.tsx
    │   └── AttentionItemCard.tsx
    ├── hooks/
    │   ├── useDigest.ts
    │   ├── useApprovalDecide.ts
    │   ├── useBulkApprove.ts
    │   ├── useWeeklySummary.ts
    │   └── useAgentStatus.ts
    ├── store/
    │   ├── digestStore.ts
    │   ├── summaryStore.ts
    │   └── agentStore.ts
    ├── types/
    │   └── comms.types.ts
    └── offline/
        └── approvalQueue.ts        ← IndexedDB offline queue
```
