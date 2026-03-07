# Phase 1 Frontend Specification -- Workshop Intake Engine

**Author:** Frontend Engineer
**Date:** 2026-03-07
**Phase Scope:** Core intake flow -- Steps 2, 3, 5, 7, 8 from storyboard
**Stack:** Next.js 14 App Router + TypeScript + Tailwind CSS
**Excluded from Phase 1:** AI pre-population (Step 4), voice recording (Step 6 mic), link scraping (Step 6 link), web scraping engine, facilitator dashboard

---

## Table of Contents

1. [Component Tree](#1-component-tree)
2. [Page/Route Structure](#2-pageroute-structure)
3. [State Management](#3-state-management)
4. [Key Component Specs](#4-key-component-specs)
5. [Data Fetching Strategy](#5-data-fetching-strategy)
6. [Testing Strategy](#6-testing-strategy)

---

## 1. Component Tree

### 1.1 Full Hierarchy

```
app/
  layout.tsx                          [Server] -- html, body, global providers
  intake/
    [token]/
      layout.tsx                      [Server] -- validates token, sets session context
      page.tsx                        [Server] -- redirects to /intake/[token]/flow
      flow/
        layout.tsx                    [Client] -- IntakeFlowShell (progress bar, transition container)
        page.tsx                      [Client] -- FlowRouter (state machine drives rendering)
        loading.tsx                   [Server] -- FlowSkeleton
        error.tsx                     [Client] -- FlowError
      review/
        page.tsx                      [Client] -- ReviewScreen
        loading.tsx                   [Server] -- ReviewSkeleton
      submitted/
        page.tsx                      [Server] -- SubmissionConfirmation
```

### 1.2 Component Hierarchy (Client Components)

```
IntakeFlowShell                       [Client] -- progress bar + transition wrapper
  ProgressBar                         [Client] -- thin top bar, animates on progress change
  TransitionContainer                 [Client] -- AnimatePresence wrapper for slide transitions
    FlowRouter                        [Client] -- reads state machine, renders current step
      VideoKickoff                    [Client] -- video player + "Let's Go" CTA
        VideoPlayer                   [Client] -- custom minimal video chrome
        CTAButton                     [Client] -- "Let's Go"
      SelfSegmentation               [Client] -- single question, 4 option cards
        SegmentationQuestion          [Client] -- heading text
        OptionCard                    [Client] -- tappable card (x4)
      QuestionFlow                   [Client] -- manages current question within a section
        QuestionCard                  [Client] -- question text + input area
          QuestionText                [Client] -- the question heading
          TextInput                   [Client] -- auto-expanding textarea
          InputModeSwitcher           [Client] -- file upload icon (Phase 1: file only, no mic/link)
          FileUploadZone              [Client] -- drag-drop or click-to-browse
            FileChip                  [Client] -- filename, size, progress, status
          NextButton                  [Client] -- appears when input exists
        CollapsedAnswer               [Client] -- previous answer summary (for follow-ups)
      ReviewScreen                    [Client] -- summary of all sections
        ReviewHeader                  [Client] -- "Here's what we've captured" heading
        SectionSummaryCard            [Client] -- one per framework section (expandable)
          StatusIndicator             [Client] -- green check or amber flag
          ContentExcerpt              [Client] -- two-line preview
          ExpandedContent             [Client] -- full content + inline edit
        SubmitButton                  [Client] -- submit with state transitions
      SubmissionConfirmation          [Server] -- static thank-you page
  WelcomeBackToast                   [Client] -- transient message on session resume
  AutoSaveIndicator                  [Client] -- invisible to user; debug-only status
```

### 1.3 Shared/Primitive Components

```
primitives/
  CTAButton                           -- primary/secondary/text-only variants
  Card                                -- base card with border, shadow, rounded corners
  IconButton                          -- small icon-based button with tooltip
  TransientToast                      -- auto-dismissing notification
  ScreenReaderAnnouncer               -- aria-live region for dynamic announcements
```

### 1.4 Server vs. Client Component Designation

| Component | Rendering | Rationale |
|-----------|-----------|-----------|
| Root layout | Server | Static HTML shell, no interactivity |
| Token layout | Server | Token validation and session hydration happen server-side |
| IntakeFlowShell | Client | Manages transitions, progress updates, auto-save -- all interactive |
| FlowRouter | Client | Drives the state machine, must respond to user input |
| VideoKickoff | Client | Video playback, button interaction |
| SelfSegmentation | Client | Interactive selection |
| QuestionFlow | Client | Core interactive flow |
| ReviewScreen | Client | Expandable cards, inline editing, submit |
| SubmissionConfirmation | Server | Static page, no interactivity after submission |

### 1.5 Key Props Interfaces

```typescript
// === Primitives ===

interface CTAButtonProps {
  label: string;
  variant: 'primary' | 'secondary' | 'text';
  onClick: () => void;
  disabled?: boolean;
  loading?: boolean;
  icon?: React.ReactNode;
  fullWidth?: boolean;
  size?: 'default' | 'large';
  className?: string;
}

interface CardProps {
  children: React.ReactNode;
  selected?: boolean;
  hoverable?: boolean;
  onClick?: () => void;
  className?: string;
  role?: string;
  ariaChecked?: boolean;
  ariaLabel?: string;
}

// === Progress ===

interface ProgressBarProps {
  percentage: number;        // 0-100
  isComplete: boolean;       // triggers green color + pulse
}

// === Video Kickoff ===

interface VideoKickoffProps {
  videoUrl: string;
  posterUrl: string;
  onStart: () => void;       // called when "Let's Go" is clicked
}

interface VideoPlayerProps {
  src: string;
  poster: string;
  onComplete?: () => void;   // called when video finishes
}

// === Self-Segmentation ===

type SegmentationLevel =
  | 'documents-ready'
  | 'scattered-outdated'
  | 'in-our-heads'
  | 'early-stage';

interface SelfSegmentationProps {
  onSelect: (level: SegmentationLevel) => void;
}

interface OptionCardProps {
  label: string;
  value: SegmentationLevel;
  selected: boolean;
  onSelect: (value: SegmentationLevel) => void;
}

// === Question Flow ===

interface QuestionNode {
  id: string;
  questionText: string;
  questionType: 'text' | 'textarea' | 'upload-prompt';
  inputHint?: string;
  frameworkSection: string;
  allowsFile: boolean;
  responseOptions?: string[];  // for multiple-choice questions
  isTerminal: boolean;
}

interface QuestionFlowProps {
  sessionId: string;
  segmentation: SegmentationLevel;
  initialNodeId?: string;     // for resume
  initialAnswers?: Record<string, string>;
  onComplete: () => void;     // triggers transition to review
  onProgressUpdate: (pct: number) => void;
}

interface QuestionCardProps {
  node: QuestionNode;
  existingAnswer?: string;
  onSubmit: (answer: SubmissionPayload) => void;
  onBack?: () => void;
  direction: 'forward' | 'follow-up';
}

interface SubmissionPayload {
  frameworkSection: string;
  questionNodeId: string;
  inputType: 'text' | 'file';
  content?: string;
}

interface TextInputProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  autoFocus?: boolean;
  ariaLabelledBy?: string;
  minRows?: number;
}

interface NextButtonProps {
  visible: boolean;
  onClick: () => void;
  disabled?: boolean;
}

interface CollapsedAnswerProps {
  questionText: string;
  answerPreview: string;      // first 80 chars + ellipsis
}

// === File Upload ===

interface FileUploadZoneProps {
  onFileSelect: (files: File[]) => void;
  contextPrompt?: string;     // e.g., "Have a competitive analysis?"
  disabled?: boolean;
}

interface FileChipProps {
  filename: string;
  fileSize: number;           // bytes
  status: 'uploading' | 'complete' | 'error';
  progress?: number;          // 0-100 for upload progress
  onRemove: () => void;
  onRetry?: () => void;
}

// === Input Mode Switcher ===

interface InputModeSwitcherProps {
  onFileClick: () => void;
  // Phase 2+: onMicClick, onLinkClick
  activeMode: 'text' | 'file' | null;
}

// === Review Screen ===

interface ReviewScreenProps {
  sessionId: string;
  onSubmit: () => void;
}

interface SectionSummaryCardProps {
  sectionKey: string;
  sectionName: string;
  status: 'complete' | 'thin';
  excerpt: string;
  submissions: SubmissionSummary[];
  isExpanded: boolean;
  onToggle: () => void;
  onEdit: (sectionKey: string, newContent: string) => void;
}

interface SubmissionSummary {
  id: string;
  inputType: 'text' | 'file';
  contentPreview: string;
  createdAt: string;
}

interface StatusIndicatorProps {
  status: 'complete' | 'thin';
  ariaLabel: string;
}

interface SubmitButtonProps {
  onClick: () => void;
  state: 'idle' | 'submitting' | 'success';
}

// === Welcome Back Toast ===

interface WelcomeBackToastProps {
  completionPct: number;
  visible: boolean;
  onDismiss: () => void;
}

// === Submission Confirmation ===

interface SubmissionConfirmationProps {
  facilitatorName: string;
  submittedAt: string;
}
```

---

## 2. Page/Route Structure

### 2.1 Next.js App Router Routes

```
/intake/[token]                   -- Token validation + redirect
/intake/[token]/flow              -- Main interview flow (video, segmentation, questions)
/intake/[token]/review            -- Review and submit screen
/intake/[token]/submitted         -- Post-submission confirmation
```

### 2.2 Route Details

#### `/intake/[token]` -- Token Entry Point

**Type:** Server Component (page.tsx)

**Behavior:**
1. Server-side: validate `token` against database (exists, not expired, not already submitted)
2. If invalid/expired: render `TokenExpired` page (static, no retry -- "This link has expired. Contact [facilitator name] for a new one.")
3. If already submitted: redirect to `/intake/[token]/submitted`
4. If valid: set HTTP-only JWT session cookie, redirect to `/intake/[token]/flow`
5. Log access in `session_access_log` (IP, user agent, timestamp)

**Server Action:**
```typescript
// app/intake/[token]/page.tsx
async function validateAndRedirect(token: string) {
  const session = await db.query.intakeSessions.findFirst({
    where: eq(intakeSessions.magicLinkToken, token)
  });

  if (!session || isExpired(session)) return <TokenExpired />;
  if (session.status === 'submitted') redirect(`/intake/${token}/submitted`);

  await setSessionCookie(session);
  await logAccess(session.id, headers());
  redirect(`/intake/${token}/flow`);
}
```

#### `/intake/[token]/flow` -- Main Interview Flow

**Type:** Client Component (wrapped in server layout for auth check)

**Layout hierarchy:**
```
app/intake/[token]/layout.tsx      [Server] -- auth middleware, provides SessionContext
  app/intake/[token]/flow/layout.tsx [Client] -- IntakeFlowShell (progress bar, transitions)
    app/intake/[token]/flow/page.tsx  [Client] -- FlowRouter
```

**Loading state:** `FlowSkeleton` -- centered content column with a subtle pulse animation (matching the off-white background). No spinner. The progress bar renders immediately at 0% or at the resumed percentage.

**Error state:** `FlowError` -- "Something went wrong on our end. Your answers are saved -- try refreshing." with a "Refresh" button. No technical details exposed to the client.

#### `/intake/[token]/review` -- Review and Submit

**Type:** Client Component

**Behavior:** Fetches all submissions for the session, groups by framework section, renders summary cards. Submit triggers `POST /api/intake/session/submit`.

**Navigation:** The user arrives here when the question flow state machine reaches a terminal state. The URL updates to `/review` via `router.push` so the browser back button returns to the last question.

**Loading state:** `ReviewSkeleton` -- section card placeholders (6 gray rectangles stacked).

**Error state:** Same pattern as flow error.

#### `/intake/[token]/submitted` -- Confirmation

**Type:** Server Component

**Behavior:** Static confirmation page. Fetches facilitator name and submitted timestamp from server. No further interactivity.

### 2.3 Layout Hierarchy

```
RootLayout (app/layout.tsx)
  -- Sets <html lang="en">, system font stack, off-white background
  -- Wraps children in QueryClientProvider (React Query)
  -- Wraps children in SessionProvider (custom context)

  TokenLayout (app/intake/[token]/layout.tsx)
    -- Server component: validates token on every request (middleware)
    -- Provides SessionContext with session data hydrated from server
    -- Sets viewport meta for mobile

    FlowLayout (app/intake/[token]/flow/layout.tsx)
      -- Client component: IntakeFlowShell
      -- Renders ProgressBar pinned to top
      -- Renders TransitionContainer around {children}
      -- Renders WelcomeBackToast (if resuming)
      -- Renders ScreenReaderAnnouncer
```

### 2.4 Middleware

```typescript
// middleware.ts
export function middleware(request: NextRequest) {
  // Rate limiting: max 5 requests per minute per IP to /intake/[token]
  // Session cookie refresh: if JWT nearing expiry, refresh
  // No auth check here -- that happens in the layout server component
}

export const config = {
  matcher: '/intake/:path*'
};
```

---

## 3. State Management

### 3.1 State Location Map

| State | Location | Rationale |
|-------|----------|-----------|
| Session identity (session_id, client_id, booking_id) | JWT in HTTP-only cookie + React context | Security: cookie is HttpOnly. Context makes it accessible to components. |
| Current token | URL param (`[token]`) | Shareable, bookmarkable, enables magic link re-entry |
| Current flow step (video / segmentation / question / review) | React state in FlowRouter + URL path (/flow vs /review) | URL distinguishes major phases; React state handles sub-steps within /flow |
| Current question node ID | React state in QuestionFlow | Changes rapidly during the flow; persisted server-side via auto-save |
| Segmentation choice | React state + server state (auto-saved) | Set once, drives branching; saved immediately to server |
| Answer text (in-progress) | Local React state in TextInput | Not saved until "Next" is clicked (debounced draft save optional) |
| Submitted answers (history) | Server state (submissions table) + React Query cache | Source of truth is server; cache prevents re-fetching on back navigation |
| File upload progress | Local React state in FileChip | Ephemeral UI state; server tracks processing status |
| Progress percentage | Derived from server response on each submission | Calculated server-side for accuracy; cached in React Query |
| Expanded section on review | Local React state in ReviewScreen | UI-only state, not persisted |
| Welcome-back toast visibility | Local React state in IntakeFlowShell | Shown once on resume, dismissed after 3 seconds |

### 3.2 Question Flow State Machine

The flow is modeled as a finite state machine with the following states:

```typescript
type FlowState =
  | { phase: 'video' }
  | { phase: 'segmentation' }
  | { phase: 'questions'; sectionKey: string; nodeId: string; direction: 'forward' | 'follow-up' }
  | { phase: 'review' }
  | { phase: 'submitted' };

type FlowEvent =
  | { type: 'START' }                                          // video -> segmentation
  | { type: 'SEGMENT'; level: SegmentationLevel }              // segmentation -> questions
  | { type: 'ANSWER'; nodeId: string; answer: SubmissionPayload } // questions -> next question
  | { type: 'BACK' }                                           // questions -> previous question
  | { type: 'SECTION_COMPLETE' }                               // questions -> next section or review
  | { type: 'SUBMIT' }                                         // review -> submitted
  | { type: 'RESUME'; state: FlowState };                      // any (from server restore)
```

**State machine implementation:**

```typescript
// lib/flow-machine.ts

interface FlowMachineState {
  current: FlowState;
  history: FlowState[];       // for back navigation
  segmentation: SegmentationLevel | null;
  answeredNodes: Set<string>; // track which nodes have been answered
  sectionOrder: string[];     // ordered list of framework section keys
  currentSectionIndex: number;
}

function flowReducer(state: FlowMachineState, event: FlowEvent): FlowMachineState {
  switch (event.type) {
    case 'START':
      return { ...state, current: { phase: 'segmentation' }, history: [...state.history, state.current] };

    case 'SEGMENT':
      // Segmentation determines the section order and question paths
      const sectionOrder = getSectionOrder(event.level);
      const firstSection = sectionOrder[0];
      return {
        ...state,
        segmentation: event.level,
        sectionOrder,
        currentSectionIndex: 0,
        current: { phase: 'questions', sectionKey: firstSection, nodeId: getEntryNode(firstSection, event.level), direction: 'forward' },
        history: [...state.history, state.current],
      };

    case 'ANSWER':
      // Server response tells us the next node
      // Handled async -- see QuestionFlow component
      return state;

    case 'BACK':
      const previous = state.history[state.history.length - 1];
      return { ...state, current: previous, history: state.history.slice(0, -1) };

    case 'SECTION_COMPLETE':
      const nextIndex = state.currentSectionIndex + 1;
      if (nextIndex >= state.sectionOrder.length) {
        return { ...state, current: { phase: 'review' } };
      }
      const nextSection = state.sectionOrder[nextIndex];
      return {
        ...state,
        currentSectionIndex: nextIndex,
        current: { phase: 'questions', sectionKey: nextSection, nodeId: getEntryNode(nextSection, state.segmentation!), direction: 'forward' },
        history: [...state.history, state.current],
      };

    case 'SUBMIT':
      return { ...state, current: { phase: 'submitted' } };

    case 'RESUME':
      return { ...state, current: event.state };

    default:
      return state;
  }
}
```

**The state machine lives in `useReducer` inside `FlowRouter`.** It is not stored in a global state management library -- the flow is linear and contained within a single page.

### 3.3 Branching State Tracking

Branching is resolved server-side. The client does not store the full question tree. The flow is:

1. Client submits an answer via `POST /api/intake/submissions`
2. Server saves the submission and returns `{ submission_id, updated_progress }`
3. Client calls `GET /api/intake/section/:sectionKey/next?current_node_id=X` with the current node ID
4. Server traverses the question tree graph, evaluates conditional edges based on the client's answer history, and returns the next `QuestionNode`
5. If the response includes `is_terminal: true`, the client dispatches `SECTION_COMPLETE`

**Why server-side resolution:**
- The question tree JSON can be large (hundreds of nodes across 4 paths). Loading it all client-side wastes bandwidth.
- Conditional edge evaluation requires the full answer history, which is server-side.
- The server can adapt questions based on async file processing results (Phase 2+).
- The client only needs to know: "What is the current question?" and "What is the next question?"

**Back navigation within a section:**
- The client maintains a local `history` stack of `(sectionKey, nodeId)` pairs.
- When the user navigates back, the client pops the stack and re-renders the previous question with its saved answer (fetched from React Query cache).
- Back navigation does not re-call the server for question resolution -- the question data is already cached.

### 3.4 Auto-Save Implementation

Auto-save happens at the interaction level, not on a timer:

```typescript
// hooks/useAutoSave.ts

function useAutoSave(sessionId: string) {
  const queryClient = useQueryClient();
  const saveMutation = useMutation({
    mutationFn: (payload: SubmissionPayload) =>
      fetch('/api/intake/submissions', {
        method: 'POST',
        body: JSON.stringify(payload),
        headers: { 'Content-Type': 'application/json' },
      }).then(res => res.json()),

    onSuccess: (data) => {
      // Update progress in cache
      queryClient.setQueryData(['session', sessionId], (old: SessionState) => ({
        ...old,
        completion_pct: data.updated_progress,
      }));
    },

    onError: (error, variables) => {
      // Queue for retry -- store in localStorage as fallback
      queueFailedSave(variables);
    },

    retry: 3,
    retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 10000),
  });

  return saveMutation;
}
```

**Save triggers:**
- User clicks "Next" (primary save)
- User selects a segmentation option
- File upload completes
- User navigates away from a question with content (save on blur/unmount)

**No timer-based auto-save in Phase 1.** Each discrete interaction triggers a save. If the user is mid-typing and closes the browser, the in-progress text is lost -- this is acceptable because:
- The storyboard specifies "auto-saved after every answer," not "auto-saved mid-keystroke"
- Timer-based saves add complexity (debouncing, conflict resolution) for minimal gain
- The user must click "Next" to advance, which is the natural save point

**Offline resilience (Phase 1 minimal):**
- If a save fails (network error), the mutation retries 3 times with exponential backoff.
- If all retries fail, the payload is saved to `localStorage` under key `wie_failed_saves_${sessionId}`.
- On next successful page load, the `useAutoSave` hook checks localStorage for queued saves and replays them.

### 3.5 Session Restoration on Resume

When a user returns to `/intake/[token]/flow`:

```typescript
// hooks/useSessionRestore.ts

function useSessionRestore(sessionId: string) {
  const { data, isLoading } = useQuery({
    queryKey: ['session', sessionId],
    queryFn: () => fetch('/api/intake/session').then(res => res.json()),
    staleTime: 0,  // always fetch fresh on mount
  });

  // Determine the FlowState to resume from
  const resumeState: FlowState | null = useMemo(() => {
    if (!data) return null;
    if (data.status === 'submitted') return { phase: 'submitted' };
    if (!data.segmentation_choice) return { phase: 'video' };
    if (!data.current_section) return { phase: 'segmentation' };
    return {
      phase: 'questions',
      sectionKey: data.current_section,
      nodeId: data.current_step.toString(),
      direction: 'forward',
    };
  }, [data]);

  return { resumeState, isLoading, showWelcomeBack: data?.status === 'in-progress' };
}
```

---

## 4. Key Component Specs

### 4.1 IntakeFlowShell

**Responsibility:** Top-level layout for the flow experience. Renders the progress bar, manages the animated transition container, and hosts the welcome-back toast.

**File:** `components/intake/IntakeFlowShell.tsx`

**Internal state:**
- `progressPct: number` -- current progress (updated by child events)
- `showWelcomeBack: boolean` -- true on session resume
- `isTransitioning: boolean` -- true during screen transitions (blocks user input)

**Event handlers:**
- `onProgressUpdate(pct: number)` -- updates progress bar
- `onWelcomeBackDismiss()` -- hides toast

**Responsive behavior:**
- Progress bar is always full-width, 3px height, pinned to top
- Content column: max-width 640px, centered with `mx-auto`
- Mobile: full-width with `px-6` (24px padding)

**Animation:**
- Progress bar fill: `transition-all duration-300 ease-out`
- On completion (100%): bar color transitions to green (#2E7D32), single opacity pulse

**Tailwind classes (progress bar):**
```
<div className="fixed top-0 left-0 right-0 h-[3px] bg-[#E8E8E4] z-50">
  <div
    className="h-full bg-[#2B5CE6] transition-all duration-300 ease-out"
    style={{ width: `${percentage}%` }}
    role="progressbar"
    aria-valuenow={percentage}
    aria-valuemin={0}
    aria-valuemax={100}
  />
</div>
```

---

### 4.2 VideoKickoff

**Responsibility:** Renders the facilitator's kickoff video and the "Let's Go" call-to-action. This is the first screen the client sees.

**File:** `components/intake/VideoKickoff.tsx`

**Props:** `VideoKickoffProps` (see Section 1.5)

**Internal state:**
- `hasVideoCompleted: boolean` -- true when video finishes playing
- `isVideoLoaded: boolean` -- true when video metadata loads

**Event handlers:**
- `onVideoEnd()` -- sets `hasVideoCompleted`, triggers subtle CTA pulse
- `onStart()` -- calls `props.onStart()`, dispatches `START` event to flow machine

**Responsive behavior:**
- Desktop: video centered, max-width 640px, comfortable vertical margin (64px top)
- Mobile: video edge-to-edge (negative margin to break out of padding), CTA full-width with 24px side padding

**Animation:**
- On video completion: CTA button pulses once (`animate-pulse` for one iteration)
- Transition out: entire screen slides up over 300ms ease-out

**HTML structure:**
```tsx
<div className="flex flex-col items-center pt-16 md:pt-20">
  <div className="w-full max-w-[640px] aspect-video">
    <VideoPlayer src={videoUrl} poster={posterUrl} onComplete={onVideoEnd} />
  </div>
  <div className="mt-8 md:mt-12">
    <CTAButton
      label="Let's Go"
      variant="primary"
      size="large"
      onClick={onStart}
      disabled={!isVideoLoaded}
    />
  </div>
</div>
```

**Accessibility:**
- Video must have captions (VTT track or burned-in)
- Play button keyboard-focusable, activatable with Enter/Space
- CTA button disabled state communicates via `aria-disabled` and tooltip: "Video is loading..."

---

### 4.3 SelfSegmentation

**Responsibility:** Presents a single question with four large tappable option cards. The client's selection determines the entire question path.

**File:** `components/intake/SelfSegmentation.tsx`

**Props:** `SelfSegmentationProps` (see Section 1.5)

**Internal state:**
- `selectedOption: SegmentationLevel | null`

**Event handlers:**
- `onOptionSelect(value: SegmentationLevel)` -- sets selected state, triggers 400ms pause, then calls `props.onSelect(value)`

**Responsive behavior:**
- Desktop: centered column, max-width 560px, cards stacked vertically with 16px gaps
- Mobile: full-width with 24px padding, cards full-width
- Cards: min-height 72px, flexible to accommodate text wrapping

**Animation:**
- Entrance: slide-up from below (300ms ease-out) -- arriving from Video Kickoff
- On selection: selected card holds highlighted state for 400ms, then entire screen slides left (350ms ease-out)

**Card interactive states (Tailwind):**
```
Default:     bg-white border border-[#E0E0DC] rounded-xl
Hover:       border-[#2B5CE6] shadow-sm
Selected:    bg-[#EEF2FC] border-l-4 border-l-[#2B5CE6] border-[#2B5CE6]
```

**Accessibility:**
- Card group has `role="radiogroup"`, `aria-label="Select your situation"`
- Each card has `role="radio"`, `aria-checked`, `tabIndex`
- Arrow keys navigate between options; Enter/Space selects
- Selected state communicated by border + checkmark icon (not color alone)

---

### 4.4 QuestionFlow

**Responsibility:** Manages the branching question sequence within and across framework sections. Fetches the next question from the server, renders `QuestionCard`, handles answer submission, and tracks history for back navigation.

**File:** `components/intake/QuestionFlow.tsx`

**Props:** `QuestionFlowProps` (see Section 1.5)

**Internal state:**
- `currentNode: QuestionNode | null` -- the active question
- `questionHistory: Array<{ node: QuestionNode; answer: string }>` -- for back navigation
- `isLoadingNext: boolean` -- true while fetching next question from server
- `transitionDirection: 'slide-left' | 'slide-up'` -- determines animation direction

**Event handlers:**
- `onAnswer(payload: SubmissionPayload)` -- saves via auto-save mutation, then fetches next question
- `onBack()` -- pops from history stack, renders previous question with saved answer
- `onFileUploaded(submissionId: string, file: File)` -- uploads file, shows chip

**Data flow on answer submission:**
```
User clicks "Next"
  -> POST /api/intake/submissions (save answer)
  -> GET /api/intake/section/:key/next?current_node_id=X (get next question)
  -> If response.is_terminal:
       -> Dispatch SECTION_COMPLETE to flow machine
       -> Flow machine advances to next section or review
  -> If response.node exists:
       -> Push current node+answer to history
       -> Set currentNode = response.node
       -> Animate slide-left transition
  -> If response.node is a follow-up (different detection TBD by API):
       -> transitionDirection = 'slide-up'
```

**Responsive behavior:**
- Same as SelfSegmentation: centered column, max-width 600px
- On mobile: "Next" button pinned to bottom of viewport when keyboard is inactive
- When keyboard is active (input focused): "Next" button returns to flow below the input

**Animation:**
- Forward question: current slides left (opacity 0) over 300ms, next slides in from right (translateX(80px) -> 0, opacity 0 -> 1) with 50ms delay. Easing: `cubic-bezier(0.25, 0.1, 0.25, 1.0)`
- Follow-up: slides up from translateY(24px) over 250ms. Parent question collapses to `CollapsedAnswer`.
- Back: mirror of forward (slide right). Same timing.
- All transitions use `will-change: transform, opacity` for GPU acceleration.
- If `prefers-reduced-motion` is set: instant state change, no animation.

---

### 4.5 QuestionCard

**Responsibility:** Renders a single question with its input area. Handles text entry and the "Next" button visibility.

**File:** `components/intake/QuestionCard.tsx`

**Props:** `QuestionCardProps` (see Section 1.5)

**Internal state:**
- `answer: string` -- current text input value
- `uploadedFiles: FileState[]` -- files attached to this question
- `showUploadZone: boolean` -- toggled by InputModeSwitcher

**Event handlers:**
- `onTextChange(value: string)` -- updates local answer state
- `onNextClick()` -- calls `props.onSubmit` with the answer payload
- `onFileSelect(files: File[])` -- initiates upload, shows FileChip

**Key behaviors:**
- The "Next" button is hidden (opacity 0, pointer-events none) until `answer.trim().length > 0` or `uploadedFiles.length > 0`
- The "Next" button fades in over 200ms when content exists
- TextInput auto-focuses on mount (after transition completes, ~350ms delay)
- If `existingAnswer` is provided (back navigation), pre-populate the TextInput

**HTML structure:**
```tsx
<div className="pt-10 max-w-[600px] mx-auto">
  <h2
    id={`question-${node.id}`}
    className="text-[22px] font-semibold text-[#1A1A18] leading-snug"
  >
    {node.questionText}
  </h2>

  <div className="mt-8 relative">
    <TextInput
      value={answer}
      onChange={onTextChange}
      placeholder={node.inputHint || "Type your answer..."}
      ariaLabelledBy={`question-${node.id}`}
    />
    {node.allowsFile && (
      <InputModeSwitcher
        onFileClick={() => setShowUploadZone(true)}
        activeMode={showUploadZone ? 'file' : null}
      />
    )}
  </div>

  {showUploadZone && (
    <FileUploadZone
      onFileSelect={onFileSelect}
      contextPrompt={node.inputHint}
    />
  )}

  {uploadedFiles.map(file => (
    <FileChip key={file.id} {...file} />
  ))}

  <div className="mt-6 flex justify-end">
    <NextButton
      visible={answer.trim().length > 0 || uploadedFiles.length > 0}
      onClick={onNextClick}
    />
  </div>
</div>
```

---

### 4.6 TextInput

**Responsibility:** Auto-expanding textarea that grows as the user types. Bottom-border-only style (no box outline).

**File:** `components/intake/TextInput.tsx`

**Props:** `TextInputProps` (see Section 1.5)

**Internal state:** None (controlled component)

**Key behaviors:**
- Auto-expands vertically: uses a hidden "shadow" div to measure content height, then sets textarea height to match
- Minimum height: 2 rows (~48px)
- No maximum height (grows infinitely)
- On focus: bottom border transitions from muted gray (#D0D0CC) to primary blue (#2B5CE6), 2px -> 3px

**Tailwind classes:**
```
<textarea
  className="w-full resize-none overflow-hidden bg-transparent
    border-0 border-b-2 border-[#D0D0CC]
    focus:border-b-[3px] focus:border-[#2B5CE6] focus:outline-none
    text-[16px] text-[#1A1A18] placeholder:text-[#8A8A86]
    leading-relaxed py-2 transition-colors duration-200"
  rows={minRows || 2}
  aria-labelledby={ariaLabelledBy}
/>
```

---

### 4.7 FileUploadZone

**Responsibility:** Drag-and-drop file upload area with click-to-browse fallback. Contextual prompt when relevant.

**File:** `components/intake/FileUploadZone.tsx`

**Props:** `FileUploadZoneProps` (see Section 1.5)

**Internal state:**
- `isDragOver: boolean` -- true when file is dragged over the zone

**Event handlers:**
- `onDragOver(e)` -- prevents default, sets isDragOver
- `onDragLeave()` -- clears isDragOver
- `onDrop(e)` -- extracts files, calls props.onFileSelect
- `onClick()` -- triggers hidden `<input type="file" multiple>` click

**Responsive behavior:**
- Same width as parent (full content column width)
- 120px height in default state
- On file drop, zone collapses and file chips appear

**Animation:**
- On drag over: border color shifts to blue, background tints to #F5F8FF (200ms)
- On file accepted: zone fades out (150ms), file chip fades in (200ms)

**Supported file types (Phase 1):**
```typescript
const ACCEPTED_TYPES = {
  'application/pdf': ['.pdf'],
  'application/msword': ['.doc'],
  'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
  'application/vnd.ms-powerpoint': ['.ppt'],
  'application/vnd.openxmlformats-officedocument.presentationml.presentation': ['.pptx'],
  'image/*': ['.png', '.jpg', '.jpeg', '.gif', '.webp'],
};
const MAX_FILE_SIZE = 25 * 1024 * 1024; // 25MB
```

**Accessibility:**
- `aria-label="Optional: upload a file related to [section name]"`
- Drop zone is keyboard-focusable; Enter/Space opens file picker
- File type restrictions communicated in helper text: "We accept PDF, Word, PowerPoint, and images"

---

### 4.8 FileChip

**Responsibility:** Compact display of an uploaded file with status indicator.

**File:** `components/intake/FileChip.tsx`

**Props:** `FileChipProps` (see Section 1.5)

**Internal state:** None (controlled)

**Visual states:**
```
Uploading:  file icon + filename + size + thin blue progress bar
Complete:   file icon + filename + size + green checkmark + remove (X) button
Error:      file icon + filename + red exclamation + "Upload failed -- try again" + retry button
```

**Responsive behavior:**
- Full-width card, 48px height, stacks vertically with 8px gaps
- Remove button appears on hover (desktop) or is always visible (mobile)

---

### 4.9 ReviewScreen

**Responsibility:** Summary of all framework sections with completeness indicators, expandable content, inline editing, and submit button.

**File:** `components/intake/ReviewScreen.tsx`

**Props:** `ReviewScreenProps` (see Section 1.5)

**Internal state:**
- `expandedSection: string | null` -- key of the currently expanded section
- `submitState: 'idle' | 'submitting' | 'success'`
- `editingSections: Record<string, string>` -- sections being edited with their current content

**Data fetching:**
- On mount: `GET /api/intake/session` returns all sections with their submissions and status
- Sections are sorted by `display_order` from the framework template

**Event handlers:**
- `onToggleSection(key: string)` -- expands/collapses a section (accordion pattern, one at a time)
- `onEditSection(key: string, content: string)` -- saves edit via `PUT /api/intake/submissions/:id`
- `onSubmit()` -- calls `POST /api/intake/session/submit`, transitions submitState

**Responsive behavior:**
- Desktop: max-width 680px centered column
- Mobile: full-width with 24px padding
- Submit button: sticky bottom on mobile (with shadow separator), centered on desktop

**Animation:**
- Entrance: fade in + scale up from 98% to 100% over 400ms ease-out
- Section expand: height auto-transition (200ms)
- Submit success: button transitions to green with checkmark (300ms), holds 1.5s, then fades to confirmation view

---

### 4.10 SectionSummaryCard

**Responsibility:** One card per framework section in the review screen. Shows status, excerpt, and expands to show full content.

**File:** `components/intake/SectionSummaryCard.tsx`

**Props:** `SectionSummaryCardProps` (see Section 1.5)

**HTML structure:**
```tsx
<div className="border border-[#E0E0DC] rounded-xl p-4 bg-white">
  <button
    onClick={onToggle}
    className="w-full flex items-center justify-between"
    aria-expanded={isExpanded}
  >
    <div>
      <h3 className="text-[18px] font-semibold text-[#1A1A18]">{sectionName}</h3>
      {!isExpanded && (
        <p className="text-[14px] text-[#6B6B66] mt-1 line-clamp-2">{excerpt}</p>
      )}
    </div>
    <StatusIndicator status={status} ariaLabel={status === 'complete' ? 'Complete' : 'Could use more detail'} />
  </button>

  {isExpanded && (
    <div className="mt-4 pt-4 border-t border-[#EEEEEA]">
      <ExpandedContent
        submissions={submissions}
        onEdit={(newContent) => onEdit(sectionKey, newContent)}
      />
      {status === 'thin' && (
        <p className="mt-3 text-[14px] text-[#6B6B66] italic">
          You might want to add more here.
        </p>
      )}
    </div>
  )}
</div>
```

---

### 4.11 SubmitButton

**Responsibility:** Large submit button with three visual states: idle, submitting, success.

**File:** `components/intake/SubmitButton.tsx`

**Props:** `SubmitButtonProps` (see Section 1.5)

**Visual states:**

| State | Background | Text | Icon |
|-------|-----------|------|------|
| idle | #2B5CE6 | "Submit your workshop prep" (white) | Right arrow |
| submitting | #2B5CE6 (60% opacity) | "Submitting..." (white) | Spinner |
| success | #2E7D32 | "Submitted!" (white) | Checkmark |

**Dimensions:**
- Desktop: 280px width, 56px height, centered
- Mobile: full-width (minus 24px padding each side), 56px height, sticky bottom

**Animation:**
- idle -> submitting: opacity reduces, spinner appears (200ms)
- submitting -> success: background color transitions to green (300ms), text and icon swap simultaneously
- success holds for 1.5s, then parent transitions to confirmation view

---

### 4.12 SubmissionConfirmation

**Responsibility:** Static post-submission page. The client's job is done.

**File:** `components/intake/SubmissionConfirmation.tsx` (or `app/intake/[token]/submitted/page.tsx` as a Server Component)

**Props:** `SubmissionConfirmationProps` (see Section 1.5)

**Content:**
```tsx
<div className="flex flex-col items-center justify-center min-h-[60vh] text-center px-6">
  <div className="w-16 h-16 rounded-full bg-[#E8F5E9] flex items-center justify-center mb-6">
    <CheckIcon className="w-8 h-8 text-[#2E7D32]" />
  </div>
  <h1 className="text-[28px] font-semibold text-[#1A1A18]">You're all set.</h1>
  <p className="mt-4 text-[16px] text-[#6B6B66] max-w-[400px]">
    Thanks for taking the time. {facilitatorName} will review everything
    and reach out if anything needs clarification.
  </p>
</div>
```

**No further actions.** No links, no buttons. This is a terminal page.

---

### 4.13 WelcomeBackToast

**Responsibility:** Transient notification shown when a user resumes a paused session.

**File:** `components/intake/WelcomeBackToast.tsx`

**Props:** `WelcomeBackToastProps` (see Section 1.5)

**Behavior:**
- Appears pinned to top of content area (below progress bar)
- Auto-dismisses after 3 seconds
- Content: "Welcome back -- you're {completionPct}% done."
- Click/tap anywhere on the toast dismisses it immediately

**Animation:**
- Entrance: slide down from translateY(-16px) + fade in (200ms)
- Exit: fade out (200ms)

---

### 4.14 ScreenReaderAnnouncer

**Responsibility:** Hidden aria-live region that announces dynamic content changes to screen readers.

**File:** `components/a11y/ScreenReaderAnnouncer.tsx`

**Implementation:**
```tsx
// Context-based announcer
const AnnouncerContext = createContext<(message: string) => void>(() => {});

function ScreenReaderAnnouncer({ children }: { children: React.ReactNode }) {
  const [message, setMessage] = useState('');

  return (
    <AnnouncerContext.Provider value={setMessage}>
      {children}
      <div aria-live="polite" aria-atomic="true" className="sr-only">
        {message}
      </div>
    </AnnouncerContext.Provider>
  );
}

// Usage in components:
const announce = useContext(AnnouncerContext);
announce('Question 4 of 12: Who are your top competitors?');
```

---

## 5. Data Fetching Strategy

### 5.1 Library Choice: React Query (TanStack Query)

React Query is used for all client-side data fetching. Rationale:
- Built-in cache management prevents redundant requests
- Mutation support with optimistic updates for auto-save
- Retry logic for network failures
- Stale-while-revalidate pattern for session restoration
- DevTools for debugging during development

**Not using Server Actions for data fetching** because the intake flow is highly interactive with frequent client-side state changes. Server Actions are used only for the initial token validation (server component).

### 5.2 API Endpoint Usage Map

| Component | Endpoint | Method | Trigger |
|-----------|----------|--------|---------|
| Token layout | `/api/intake/validate` (server-side) | Internal DB query | Page load |
| FlowRouter (resume) | `GET /api/intake/session` | GET | Flow page mount |
| FlowRouter (framework) | `GET /api/intake/framework` | GET | After segmentation |
| SelfSegmentation | `PATCH /api/intake/session` | PATCH | On option select |
| QuestionFlow | `GET /api/intake/section/:key/next` | GET | After each answer |
| QuestionCard (save) | `POST /api/intake/submissions` | POST | On "Next" click |
| QuestionCard (edit) | `PUT /api/intake/submissions/:id` | PUT | On "Save changes" |
| FileChip (upload) | `POST /api/intake/submissions/:id/file` | POST | On file select |
| ReviewScreen (load) | `GET /api/intake/session` | GET | Review page mount |
| SubmitButton | `POST /api/intake/session/submit` | POST | On submit click |

### 5.3 Query Key Structure

```typescript
const queryKeys = {
  session: (sessionId: string) => ['session', sessionId] as const,
  framework: (sessionId: string) => ['framework', sessionId] as const,
  nextQuestion: (sectionKey: string, nodeId: string) =>
    ['nextQuestion', sectionKey, nodeId] as const,
  submissions: (sessionId: string) => ['submissions', sessionId] as const,
};
```

### 5.4 Optimistic Updates for Auto-Save

When the user clicks "Next" on a question:

```typescript
const saveAnswerMutation = useMutation({
  mutationFn: async (payload: SubmissionPayload) => {
    const res = await fetch('/api/intake/submissions', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    if (!res.ok) throw new Error('Save failed');
    return res.json();
  },

  // Optimistic: update progress immediately
  onMutate: async (payload) => {
    await queryClient.cancelQueries({ queryKey: queryKeys.session(sessionId) });
    const previousSession = queryClient.getQueryData(queryKeys.session(sessionId));

    // Optimistically increment progress
    queryClient.setQueryData(queryKeys.session(sessionId), (old: SessionData) => ({
      ...old,
      completion_pct: Math.min(old.completion_pct + estimatedIncrement, 100),
    }));

    return { previousSession };
  },

  // Rollback on error
  onError: (err, payload, context) => {
    queryClient.setQueryData(queryKeys.session(sessionId), context?.previousSession);
  },

  // Sync with actual server response
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.session(sessionId) });
  },

  retry: 3,
  retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 10000),
});
```

**Key behavior:** The UI advances to the next question immediately after the save mutation fires, without waiting for the server response. If the save fails after retries, the user sees an error toast and can retry. This keeps the conversational flow feeling responsive.

### 5.5 File Upload Strategy

```typescript
async function uploadFile(submissionId: string, file: File, onProgress: (pct: number) => void) {
  const formData = new FormData();
  formData.append('file', file);

  // Use XMLHttpRequest for progress tracking (fetch doesn't support upload progress)
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('POST', `/api/intake/submissions/${submissionId}/file`);

    xhr.upload.addEventListener('progress', (e) => {
      if (e.lengthComputable) onProgress(Math.round((e.loaded / e.total) * 100));
    });

    xhr.addEventListener('load', () => {
      if (xhr.status >= 200 && xhr.status < 300) resolve(JSON.parse(xhr.responseText));
      else reject(new Error(`Upload failed: ${xhr.status}`));
    });

    xhr.addEventListener('error', () => reject(new Error('Upload failed')));

    // Include credentials (session cookie)
    xhr.withCredentials = true;
    xhr.send(formData);
  });
}
```

### 5.6 Error Handling and Retry Logic

| Scenario | Handling |
|----------|----------|
| Auto-save fails (network) | Retry 3x with exponential backoff. On final failure: queue to localStorage, show brief toast "Having trouble saving -- we'll retry." No blocking. |
| Auto-save fails (server 500) | Same as network failure. Log to error reporting (Sentry). |
| Next question fetch fails | Show inline error below question: "Couldn't load the next question. [Retry]" button. Block advancement until resolved. |
| File upload fails | FileChip shows error state with "Try again" button. Does not block question advancement. |
| Session restore fails | FlowError boundary: "Something went wrong loading your session. Your answers are saved. [Refresh]" |
| Token invalid/expired | Server-side render of TokenExpired page (no client-side error). |
| Submit fails | SubmitButton returns to 'idle' state. Toast: "Submission didn't go through -- please try again." |

### 5.7 Stale Time and Cache Configuration

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5 minutes
      gcTime: 30 * 60 * 1000,         // 30 minutes (garbage collection)
      refetchOnWindowFocus: false,     // avoid unexpected refetches during flow
      retry: 2,
    },
  },
});
```

| Query | staleTime | Rationale |
|-------|-----------|-----------|
| Session state | 0 (always refetch on mount) | Must be fresh on resume to restore correct position |
| Framework definition | Infinity | Framework doesn't change during a session |
| Next question | 0 | Must reflect latest answer history |
| Submissions (review) | 0 | Must show all saved answers accurately |

---

## 6. Testing Strategy

### 6.1 Unit Test Approach

**Framework:** Vitest + React Testing Library

**Coverage targets:** Aim for 90% coverage on utility functions, 80% on components.

#### Components to Unit Test

| Component | Key Test Cases |
|-----------|---------------|
| `OptionCard` | Renders label. Click triggers onSelect. Selected state shows checkmark + styling. Keyboard: Enter/Space selects. aria-checked attribute toggles. |
| `TextInput` | Controlled value updates. Auto-expands on multiline input. Focus border color change. Placeholder renders. aria-labelledby works. |
| `FileChip` | Renders filename and size. Shows progress bar when uploading. Shows checkmark when complete. Shows error state with retry button. Remove button calls onRemove. |
| `FileUploadZone` | Click triggers file picker. Drag-over shows highlight state. Drop calls onFileSelect with files. Rejects files exceeding size limit. Rejects unsupported file types. |
| `NextButton` | Hidden when visible=false. Shown when visible=true. Click calls onClick. Disabled state. |
| `StatusIndicator` | Renders green check for 'complete'. Renders amber flag for 'thin'. aria-label matches. |
| `SubmitButton` | Renders correct text for each state. Spinner shows during 'submitting'. Disabled during 'submitting'. Green transition on 'success'. |
| `ProgressBar` | Width matches percentage prop. Transitions smoothly. Green color at 100%. role=progressbar with aria values. |
| `WelcomeBackToast` | Shows when visible=true. Auto-dismisses after 3 seconds. Click dismisses. Shows correct completion percentage. |
| `ScreenReaderAnnouncer` | aria-live region renders. Announcements update the live region text. |

#### Utility Functions to Unit Test

| Function | Test Cases |
|----------|-----------|
| `flowReducer` | All state transitions. START: video -> segmentation. SEGMENT: segmentation -> questions with correct section order. SECTION_COMPLETE: advances to next section. SECTION_COMPLETE on last section: -> review. SUBMIT: -> submitted. RESUME: restores any state. BACK: pops history. |
| `getSectionOrder(level)` | Returns correct section order for each of the 4 segmentation levels. |
| `getEntryNode(sectionKey, level)` | Returns the correct entry node ID for each section and segmentation combo. |
| `formatFileSize(bytes)` | Formats bytes to human-readable (KB, MB). |
| `truncateAnswer(text, maxLen)` | Truncates at word boundary, adds ellipsis. |
| `validateFileType(file)` | Accepts supported MIME types. Rejects unsupported. |
| `validateFileSize(file)` | Accepts under 25MB. Rejects over. |
| `queueFailedSave / replayFailedSaves` | Stores to localStorage. Replays on next load. Clears after success. |

### 6.2 Integration Test Scenarios

**Framework:** Vitest + React Testing Library + MSW (Mock Service Worker) for API mocking

| Scenario | Description |
|----------|-------------|
| Full flow: video to submit | Render FlowRouter. Click "Let's Go". Select segmentation. Answer 3 questions. Arrive at review. Submit. Verify each API call was made with correct payloads. |
| Segmentation drives question path | Select "documents-ready" -> verify first question matches documents-ready entry node. Select "early-stage" -> verify different entry node. |
| Back navigation preserves answers | Answer Q1, advance to Q2, click Back. Verify Q1 re-renders with the saved answer pre-populated. |
| File upload within question flow | Answer a question, click file upload icon, select a file. Verify POST to /file endpoint. Verify FileChip renders with progress. Verify "Next" becomes visible. |
| Auto-save on answer submission | Click "Next" on a question. Verify POST /api/intake/submissions is called with correct payload. Verify progress bar updates. |
| Session resume from server state | Mock session API to return { status: 'in-progress', current_section: 'competitive-landscape', current_step: 5 }. Verify FlowRouter renders the correct question (not video or segmentation). Verify WelcomeBackToast appears. |
| Review screen shows all sections | Mock session API to return 6 sections with submissions. Verify 6 SectionSummaryCards render. Verify correct status indicators. Expand a section -- verify full content shows. |
| Submit with thin sections | Render ReviewScreen with 2 thin sections. Verify submit button is NOT disabled. Click submit. Verify POST /api/intake/session/submit fires. Verify confirmation renders. |
| Network failure on auto-save | Mock submissions endpoint to return 500. Click "Next". Verify retry attempts (3x). Verify error toast appears. Verify payload saved to localStorage. |
| File upload failure and retry | Mock file endpoint to return 500 once, then 200. Upload file. Verify error state on chip. Click retry. Verify success state. |

### 6.3 E2E Test Coverage

**Framework:** Playwright

**Environment:** Staging with seeded test data (test client, test booking, test session with magic link token)

#### Critical Flow Tests

| Test Name | Steps | Assertions |
|-----------|-------|------------|
| `intake-happy-path-documents-ready` | Open magic link. Watch video (skip). Click "Let's Go". Select "documents-ready". Answer 5 questions (text only). Review screen. Submit. | All API calls succeed. Confirmation page renders. Session status = submitted in DB. |
| `intake-happy-path-early-stage` | Same as above with "early-stage" path. | Different questions appear (verify question text). All sections represented in review. |
| `intake-pause-resume-same-device` | Start intake. Answer 3 questions. Close tab. Reopen same URL. | Flow resumes at question 4. Previous answers visible in review. Welcome-back toast appears. |
| `intake-pause-resume-cross-device` | Start intake on viewport 1280px. Answer 3 questions. Open same URL on viewport 375px (mobile). | Same resume behavior. Layout adapts to mobile. |
| `intake-file-upload` | Navigate to a question with file upload. Upload a PDF (test fixture). Verify chip shows. Click Next. Continue to review. Submit. | File appears in review under correct section. POST /file returns 202. |
| `intake-back-navigation` | Answer Q1, Q2, Q3. Click back twice. Verify Q1 renders with saved answer. Click Next through Q1, Q2 again. Verify no duplicate submissions. | Answer history is consistent. No duplicate POST calls for the same question_node_id. |
| `intake-review-edit` | Complete flow to review. Expand a section. Edit content. Save. Collapse. Re-expand. | Edited content persists. PUT call made with updated content. |
| `intake-submit-confirmation` | Complete and submit. Navigate to /intake/[token]. | Redirects to /submitted. No re-entry to flow. |
| `intake-expired-token` | Open magic link with expired token (seeded). | TokenExpired page renders. No flow access. |
| `intake-keyboard-navigation` | Navigate entire flow using only keyboard (Tab, Enter, Space, Arrow keys). | All elements focusable. All actions triggerable. No focus traps outside modals. |
| `intake-reduced-motion` | Set `prefers-reduced-motion: reduce`. Complete flow. | No slide/fade animations. Instant state changes. |

#### Performance Tests (Playwright + Lighthouse CI)

| Metric | Target | Measurement |
|--------|--------|-------------|
| LCP (first page load) | < 2.5s | Lighthouse on /intake/[token] |
| FID / INP | < 200ms | Lighthouse on flow page |
| CLS | < 0.1 | Lighthouse (transitions should not cause layout shift) |
| Auto-save round trip | < 200ms | Network waterfall timing |
| Session restore | < 500ms | Time from page load to interactive question rendering |
| Page weight (JS) | < 150KB gzipped | Bundle analysis |

#### Visual Regression Tests (Playwright Screenshots)

Capture screenshots at key states for regression detection:
- Video kickoff page (desktop + mobile)
- Segmentation screen with no selection
- Segmentation screen with selection
- Question screen with empty input
- Question screen with text entered
- Question screen with file chip
- Review screen collapsed
- Review screen with one section expanded
- Submit button in each state (idle, submitting, success)
- Confirmation page
- Token expired page

Screenshots compared against baselines using Playwright's `toMatchSnapshot()` with threshold of 0.2%.

### 6.4 Accessibility Testing

| Method | Scope | Frequency |
|--------|-------|-----------|
| axe-core (automated) | Every component in Storybook | On every PR (CI) |
| Playwright + axe | Every page in E2E suite | On every PR (CI) |
| Manual screen reader test | Full flow on VoiceOver (macOS) + NVDA (Windows) | Before each release |
| Keyboard-only navigation | Full flow | Before each release |
| Color contrast validation | All color pairs (see design spec Section 4.4) | Once during implementation, re-check on design changes |

**axe-core rules enforced (zero tolerance -- CI fails on violation):**
- color-contrast
- label
- aria-required-attr
- button-name
- image-alt
- keyboard-focusable
- tabindex

---

## Appendix A: File Structure

```
src/
  app/
    layout.tsx
    intake/
      [token]/
        layout.tsx
        page.tsx
        flow/
          layout.tsx
          page.tsx
          loading.tsx
          error.tsx
        review/
          page.tsx
          loading.tsx
        submitted/
          page.tsx

  components/
    intake/
      IntakeFlowShell.tsx
      VideoKickoff.tsx
      VideoPlayer.tsx
      SelfSegmentation.tsx
      OptionCard.tsx
      QuestionFlow.tsx
      QuestionCard.tsx
      TextInput.tsx
      InputModeSwitcher.tsx
      FileUploadZone.tsx
      FileChip.tsx
      NextButton.tsx
      CollapsedAnswer.tsx
      ReviewScreen.tsx
      ReviewHeader.tsx
      SectionSummaryCard.tsx
      StatusIndicator.tsx
      ExpandedContent.tsx
      SubmitButton.tsx
      SubmissionConfirmation.tsx
      WelcomeBackToast.tsx
      TokenExpired.tsx
      FlowSkeleton.tsx
      ReviewSkeleton.tsx
    a11y/
      ScreenReaderAnnouncer.tsx
    primitives/
      CTAButton.tsx
      Card.tsx
      IconButton.tsx
      TransientToast.tsx

  lib/
    flow-machine.ts           # State machine reducer + helpers
    query-keys.ts             # React Query key factory
    api-client.ts             # Typed fetch wrappers for all endpoints
    file-upload.ts            # XHR-based upload with progress
    auto-save.ts              # localStorage queue for failed saves
    format.ts                 # formatFileSize, truncateAnswer, etc.
    validation.ts             # validateFileType, validateFileSize
    constants.ts              # colors, timing, accepted file types

  hooks/
    useAutoSave.ts            # Mutation hook for saving answers
    useSessionRestore.ts      # Query hook for session restoration
    useFlowMachine.ts         # useReducer wrapper for flow state machine
    useNextQuestion.ts        # Query hook for fetching next question
    useFileUpload.ts          # Upload hook with progress tracking
    useAnnounce.ts            # Hook for screen reader announcements
    useMediaQuery.ts          # Responsive breakpoint detection
    useReducedMotion.ts       # prefers-reduced-motion detection

  types/
    intake.ts                 # All TypeScript interfaces from Section 1.5
    api.ts                    # API response types
    flow.ts                   # FlowState, FlowEvent, FlowMachineState

  styles/
    globals.css               # Tailwind directives, system font stack, base styles
```

---

## Appendix B: Tailwind Configuration

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        // Primary
        'primary': '#2B5CE6',
        'primary-dark': '#1E42B0',

        // Backgrounds
        'surface': '#FAFAF8',
        'surface-secondary': '#F4F4F2',
        'card': '#FFFFFF',

        // Text
        'text-primary': '#1A1A18',
        'text-secondary': '#6B6B66',
        'text-muted': '#8A8A86',

        // Borders
        'border-default': '#E0E0DC',
        'border-subtle': '#EEEEEA',

        // Semantic
        'success': '#2E7D32',
        'success-light': '#E8F5E9',
        'warning': '#F9A825',
        'warning-light': '#FFF8E1',
        'error': '#C62828',
        'error-light': '#FFEBEE',
        'info': '#1565C0',
        'info-light': '#E3F2FD',

        // Pre-populated content
        'ai-surface': '#F7F6F3',
        'ai-accent': '#E8C547',
      },
      fontFamily: {
        sans: [
          '-apple-system', 'BlinkMacSystemFont', '"Segoe UI"', 'Roboto',
          'Helvetica', 'Arial', 'sans-serif',
        ],
      },
      spacing: {
        // Design system spacing (4px base)
        '1': '4px',
        '2': '8px',
        '3': '12px',
        '4': '16px',
        '5': '20px',
        '6': '24px',
        '8': '32px',
        '10': '40px',
        '12': '48px',
        '16': '64px',
      },
      maxWidth: {
        'flow': '640px',
        'question': '600px',
        'segmentation': '560px',
        'review': '680px',
      },
      borderRadius: {
        'card': '12px',
        'button': '8px',
      },
      transitionTimingFunction: {
        'flow': 'cubic-bezier(0.25, 0.1, 0.25, 1.0)',
      },
      transitionDuration: {
        'question': '300ms',
        'follow-up': '250ms',
        'review-enter': '400ms',
      },
      keyframes: {
        'pulse-once': {
          '0%, 100%': { opacity: '1' },
          '50%': { opacity: '0.7' },
        },
      },
      animation: {
        'pulse-once': 'pulse-once 1s ease-in-out 1',
      },
    },
  },
  plugins: [],
};

export default config;
```

---

## Appendix C: Reduced Motion Support

All animated components check for `prefers-reduced-motion`:

```typescript
// hooks/useReducedMotion.ts
function useReducedMotion(): boolean {
  const [reducedMotion, setReducedMotion] = useState(false);

  useEffect(() => {
    const mql = window.matchMedia('(prefers-reduced-motion: reduce)');
    setReducedMotion(mql.matches);
    const handler = (e: MediaQueryListEvent) => setReducedMotion(e.matches);
    mql.addEventListener('change', handler);
    return () => mql.removeEventListener('change', handler);
  }, []);

  return reducedMotion;
}
```

When `reducedMotion` is true:
- `TransitionContainer` renders children with no animation (instant show/hide)
- Progress bar width changes with `transition-duration: 0ms`
- CTA button pulse animation is skipped
- File chip progress bar animates instantly to 100%

---

## Appendix D: Phase 1 Exclusions and Phase 2+ Hooks

These features are excluded from Phase 1 but the component architecture accommodates them:

| Feature | Phase | How Phase 1 Prepares |
|---------|-------|---------------------|
| AI pre-population (Step 4) | Phase 2 | `QuestionCard` already accepts `prePopulatedContent` prop (renders null in Phase 1). The confirm/edit/rewrite `ActionButtonRow` component will be built in Phase 2. |
| Voice recording | Phase 2+ | `InputModeSwitcher` renders only the file upload icon. Mic icon slot exists but is hidden. `VoiceRecorder` component is not built. |
| Link pasting | Phase 2+ | `InputModeSwitcher` has a link icon slot, hidden in Phase 1. |
| Facilitator dashboard | Phase 3 | Completely separate route tree (`/dashboard/*`). No Phase 1 code dependencies. |
| Reminder emails/SMS | Phase 5 | Server-side only. No frontend component needed. |
| Real-time processing status (SSE) | Phase 2+ | `GET /api/intake/events` endpoint exists in API design. Frontend SSE subscription hook (`useProcessingEvents`) not built until Phase 2. |
