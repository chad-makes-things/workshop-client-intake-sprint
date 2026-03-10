# Frontend Phase 1 Specification: The Hospitality Playbook Platform

**Focus**: Practice Dashboard + Morning Huddle Player
**Stack**: Next.js 14 (App Router) / TypeScript / Tailwind CSS / Zustand / Mux Player / PWA

---

## Component Tree

```
app/
├── layout.tsx                    # Root layout — fonts, global CSS, auth provider
├── (auth)/
│   ├── login/page.tsx            # Practice owner / staff login
│   └── onboarding/page.tsx       # First-time activation flow
├── (dashboard)/
│   ├── layout.tsx                # Dashboard shell — sticky nav, practice context
│   ├── page.tsx                  # Practice Dashboard (H-Score + today's huddle)
│   ├── huddle/
│   │   ├── page.tsx              # Huddle player entry (redirects to today's)
│   │   └── [sessionId]/page.tsx  # Active huddle player (full-screen)
│   ├── team/
│   │   ├── page.tsx              # Team overview — all staff certification status
│   │   └── [staffId]/page.tsx    # Individual staff certification progress
│   └── leaderboard/page.tsx      # Regional + network leaderboard
├── (certification)/
│   ├── layout.tsx                # Cert shell — minimal chrome, mobile-first
│   ├── page.tsx                  # My certification progress (Bronze journey)
│   └── module/
│       ├── [moduleId]/page.tsx   # Individual module viewer (video + scenario)
│       └── complete/page.tsx     # Certification achievement screen
└── (champion)/
    └── hub/page.tsx              # Champion hub (resources, community, impact)

components/
├── ui/
│   ├── HScoreGauge.tsx           # SVG arc gauge with level color + trend
│   ├── HScoreBreakdown.tsx       # Expandable component score breakdown
│   ├── CertBadge.tsx             # Bronze/Silver/Gold badge with animation
│   ├── ReviewCard.tsx            # Patient review with trust transfer signal
│   ├── HuddleCard.tsx            # Huddle status with streak indicator
│   ├── LeaderboardRow.tsx        # Practice rank row
│   └── ProgressBar.tsx           # Module or certification progress
├── huddle/
│   ├── HuddlePlayer.tsx          # Full-screen huddle orchestrator
│   ├── segments/
│   │   ├── CultureMoment.tsx     # Grant/Champion video segment
│   │   ├── YesterdaysWins.tsx    # Live review + H-Score trend
│   │   ├── TodaysFocus.tsx       # Behavior focus + role-play card
│   │   ├── Accountability.tsx    # Trust transfer tap + cert spotlight
│   │   └── Intention.tsx        # Closing intention card
│   └── HuddleProgress.tsx        # 5-segment progress indicator
├── certification/
│   ├── ModuleVideo.tsx           # Mux video player with no-scrub enforcement
│   ├── ScenarioCard.tsx          # Response option selector + feedback
│   ├── CertificationPath.tsx     # Bronze/Silver/Gold journey visualization
│   └── AchievementOverlay.tsx    # Certificate generation celebration screen
└── layout/
    ├── Nav.tsx                   # Sticky top nav
    └── PracticeContext.tsx       # Practice name + H-Score chip in nav
```

---

## Routes and Navigation

```typescript
const routes = {
  // Auth
  '/login':               'Login page',
  '/onboarding':          'First-time setup + practice activation',

  // Dashboard (requires auth)
  '/':                    'Practice Dashboard — H-Score + today's huddle + team',
  '/huddle':              'Redirect to today's huddle or "no huddle yet" state',
  '/huddle/:sessionId':   'Active huddle player (full-screen)',
  '/team':                'All staff with certification status',
  '/team/:staffId':       'Individual staff certification progress',
  '/leaderboard':         'Regional + network leaderboard',

  // Certification (staff-facing, minimal nav)
  '/my-certification':    'My Bronze progress + module list',
  '/module/:moduleId':    'Module viewer (video + scenario)',
  '/certified':           'Achievement screen (Bronze completion)',

  // Champion (Gold-certified Champions only)
  '/champion':            'Champion hub',
};

// Navigation guard
const roleRoutes = {
  staff:            ['/', '/my-certification', '/module/:moduleId', '/certified'],
  champion:         [...staffRoutes, '/champion'],
  practice_owner:   [...championRoutes, '/team', '/team/:staffId', '/leaderboard'],
  regional_director:['...all routes', '/network'], // Phase 2
};
```

---

## State Management (Zustand)

### Huddle Session Store

```typescript
// stores/huddleSession.ts
interface HuddleSegment {
  id: 'culture_moment' | 'wins' | 'focus' | 'accountability' | 'intention';
  complete: boolean;
  completedAt?: Date;
}

interface HuddleSessionStore {
  // Session state
  sessionId: string | null;
  practiceId: string;
  status: 'not_started' | 'in_progress' | 'complete';
  currentSegmentIndex: number; // 0-4
  segments: HuddleSegment[];

  // Content data (loaded from API)
  contentPackage: ContentPackage | null;
  latestReview: Review | null;
  hScoreToday: number;
  hScoreDelta: number;

  // Actions
  startSession: (sessionId: string) => void;
  advanceSegment: () => Promise<void>;  // Marks current complete, increments index, API call
  markTrustTransferCommit: () => Promise<void>;
  completeHuddle: () => Promise<void>;

  // Computed
  currentSegment: HuddleSegment;
  isLastSegment: boolean;
  completionPercent: number;
}

const useHuddleSession = create<HuddleSessionStore>((set, get) => ({
  sessionId: null,
  practiceId: '',
  status: 'not_started',
  currentSegmentIndex: 0,
  segments: defaultSegments(),
  contentPackage: null,
  latestReview: null,
  hScoreToday: 0,
  hScoreDelta: 0,

  startSession: async (sessionId) => {
    set({ sessionId, status: 'in_progress' });
    await api.patch(`/huddles/${sessionId}/segment`, { segment: 'culture_moment' });
  },

  advanceSegment: async () => {
    const { sessionId, currentSegmentIndex, segments } = get();
    const current = segments[currentSegmentIndex];

    // Mark current segment complete via API
    await api.patch(`/huddles/${sessionId}/segment`, {
      segment: current.id,
      complete: true
    });

    // Update local state
    const updatedSegments = segments.map((s, i) =>
      i === currentSegmentIndex ? { ...s, complete: true, completedAt: new Date() } : s
    );
    set({ segments: updatedSegments, currentSegmentIndex: currentSegmentIndex + 1 });
  },

  markTrustTransferCommit: async () => {
    const { sessionId } = get();
    await api.post(`/huddles/${sessionId}/commit-trust`);
  },

  completeHuddle: async () => {
    const { sessionId } = get();
    await api.post(`/huddles/${sessionId}/complete`);
    set({ status: 'complete' });
  },

  get currentSegment() { return get().segments[get().currentSegmentIndex]; },
  get isLastSegment() { return get().currentSegmentIndex === 4; },
  get completionPercent() { return (get().segments.filter(s => s.complete).length / 5) * 100; },
}));
```

### Practice Dashboard Store

```typescript
// stores/practiceDashboard.ts
interface PracticeDashboardStore {
  hScore: number;
  hScoreLevel: 'emerging' | 'developing' | 'established' | 'exceptional' | 'elite';
  hScoreComponents: {
    swell: { score: number; starRating: number; velocity: number; sentiment: number };
    cinnamon: { score: number; answerRate: number; greeting: number; hold: number };
    dentalIntel: { score: number; huddleCompletion: number; certRate: number };
  };
  hScore7dayDelta: number;
  hScoreSparkline: number[];
  opportunity: { component: string; action: string; estimatedImpact: string };
  todayHuddle: { status: string; streakDays: number };
  staffCertSummary: { bronze: number; silver: number; gold: number; none: number; total: number };
  recentReviews: Review[];
  loading: boolean;
  lastFetched: Date | null;
  fetch: (practiceId: string) => Promise<void>;
  refresh: () => Promise<void>;
}
```

---

## H-Score Gauge Component

```typescript
// components/ui/HScoreGauge.tsx
interface HScoreGaugeProps {
  score: number;                          // 0-100
  level: string;
  delta: number;                          // 7-day change
  size?: 'sm' | 'md' | 'lg';            // 80px / 140px / 200px
  animated?: boolean;                     // Count-up animation on mount
  showComponents?: boolean;               // Expand to show component breakdown
}

// SVG arc gauge — 240° arc (120° to 300°, so opening is at the bottom)
// Stroke width: 8px (lg), 6px (md), 4px (sm)
// Background arc: #2a2825 (dark border color)
// Fill arc: color determined by level (see color system)
// Center: score number (large) + level label (small, uppercase)
// Below dial: delta with trend arrow (↑ green / ↓ red / → muted)

// Count-up animation on mount (CSS only):
// counter-reset: score 0;
// animation: count-up 1200ms ease-out forwards;
// @keyframes count-up: counter-increment: score [endValue]
// NOTE: CSS counter animation is limited — use a simple opacity + transform reveal
// instead of numeric count-up if CSS-only constraint makes count-up complex

// Level color mapping (returns CSS custom property reference):
const levelColors = {
  emerging:     'var(--red)',
  developing:   'var(--orange)',
  established:  'var(--yellow)',
  exceptional:  'var(--green)',
  elite:        'var(--gold)',
};
```

---

## Morning Huddle Player

```typescript
// components/huddle/HuddlePlayer.tsx
// Full-screen experience. Zero chrome during playback.

interface HuddlePlayerProps {
  practiceId: string;
  sessionId: string;
}

// Layout:
// - Full viewport height and width
// - Background: #0f0f0f
// - Top: thin 5-segment progress indicator (5 dots or line segments)
// - Content area: changes based on current segment
// - Bottom: minimal controls (advance button, or auto-advance on video completion)

// Segment rendering:
const segmentComponents = {
  culture_moment: CultureMoment,    // Full-screen Mux video
  wins: YesterdaysWins,             // Review card + H-Score sparkline
  focus: TodaysFocus,               // Behavior card + role-play prompt
  accountability: Accountability,   // Trust transfer tap + cert spotlight
  intention: Intention,             // Large text, serif, slow fade-in
};

// Auto-advance logic:
// - culture_moment: auto-advance at video end (no manual skip on first watch)
// - wins: 45-second timer OR manual tap "continue"
// - focus: 90-second timer (includes 30s role-play time) OR manual "done"
// - accountability: manual — requires trust transfer tap to unlock "continue"
// - intention: 30-second hold to allow reading aloud, then "complete huddle" button appears
```

### CultureMoment Segment

```typescript
// components/huddle/segments/CultureMoment.tsx
// Uses Mux Player React component
// - autoPlay: true
// - controls: minimal (pause only)
// - captions: on by default
// - disableScrubBar: true on first session view
// - onEnded: () => huddleStore.advanceSegment()
// - playbackRate options: 0.75, 1.0, 1.25 (subtle control, not prominent)

// Layout: video fills 100% of container
// Bottom overlay (appears last 10 seconds): "Coming up: Yesterday's Wins →"
```

### YesterdaysWins Segment

```typescript
// components/huddle/segments/YesterdaysWins.tsx
// Shows: H-Score trend sparkline + most recent flagged review

// H-Score section:
// - 7-day sparkline (SVG line chart, simple, no axes)
// - Today's score large, delta with arrow
// - "↑13 points this month" subtext

// Review section (if flagged review available):
// - ReviewCard component: reviewer first name + star rating + review text
// - Trust Transfer Signal badge (gold tag) if applicable
// - Staff mention callout if applicable

// If no flagged review available: show count of reviews received yesterday
// and most recent review (even if not flagged)

// Auto-advance timer: 45 seconds (visible countdown bar at bottom)
// "Continue" button available immediately (don't force wait)
```

### TodaysFocus Segment

```typescript
// components/huddle/segments/TodaysFocus.tsx
// Large text card — readable from 10 feet

// Layout:
// Week theme label (small, uppercase, gold): "WEEK 3 — TRUST TRANSFER"
// Behavior title (large, 28-32px): "How the Whole Team Builds One Experience"
// Exercise prompt (body size): "Today, each person name-drops the next provider in the handoff."
// Practice script box (warm card background, gold border):
//   "Dr. [Name] is going to come in and ask you about..."
// Timer: 30-second visible countdown labeled "Team practice time"
// After timer: "Continue" button appears

// Font: Georgia/serif for the practice script (matches hospitality brand)
```

### Accountability Segment

```typescript
// components/huddle/segments/Accountability.tsx
// Trust transfer commitment tap — must happen before "Continue" unlocks

// Trust Transfer section:
// Large commitment statement (24px, centered):
//   "Today our team will complete every patient handoff with a warm introduction."
// Large "Commit" button (gold, full-width, 56px height)
// On tap: checkmark animation, button disabled, text changes to "✓ Committed"
// Side effect: API call to /huddles/:sessionId/commit-trust

// Certification spotlight section (below, only if applicable):
// "🎉 [Staff Name] earned their CCA Bronze certification since last huddle"
// Shows certificate preview (small card with name + tier)

// Environmental check (3 items, simple tappable checkmarks):
// ○ Welcome space ready
// ○ Refreshments stocked
// ○ Music/ambient on

// "Continue" button LOCKED until trust transfer committed
```

### Intention Segment

```typescript
// components/huddle/segments/Intention.tsx
// The closing ceremony — slow, warm, serif

// Background: subtle radial gradient from gold (very faint, 0.04 opacity) at center
// Text: the intention statement
//   Font: Georgia serif, 28-36px (clamp), font-weight: 300
//   Color: #f0ece4 (warm white)
//   Center-aligned, max-width: 520px
//   Fade-in animation: 1000ms ease-in
//   Example: "Today, every guest leaves knowing their name matters here."

// Below text: facilitator instruction (small, muted):
//   "Read aloud together. The team repeats."

// "Complete Huddle" button appears after 20 seconds (gold, full-width)
// On tap: triggers completeHuddle() → API → redirect to post-huddle summary
```

---

## Real-Time Updates

### H-Score Updates During Huddle

The H-Score may update during the huddle (e.g., a review arrives during the 5-minute window). Use React Server-Sent Events (SSE) or polling every 60 seconds to refresh the wins segment data.

```typescript
// In YesterdaysWins.tsx
// On mount: subscribe to SSE stream or set 60s polling interval
// If new review arrives with trust_transfer_signal: 'positive' → animate it in
// H-Score delta updates silently (no disruptive toast during huddle)
```

### Swell Review Live Surface

The "Yesterday's Wins" segment pulls the most recent flagged review from the API. On subsequent page visits (practice dashboard), reviews update via SWR with a 30-second revalidation interval.

```typescript
const { data: recentReviews } = useSWR(
  `/api/v1/practices/${practiceId}/reviews/recent`,
  fetcher,
  { refreshInterval: 30_000 }  // 30 seconds
);
```

---

## Mobile-First Responsive Requirements

### Breakpoints

```css
/* Mobile (default): 375px+ */
/* Tablet (huddle screen): 768px+ — primary use case for huddle player */
/* Desktop (dashboard): 1024px+ */
```

### Critical Mobile Behaviors

**Certification module viewer (phone)**:
- Video: full-width, 16:9 ratio, auto-plays with captions
- Scenario card: slides up from bottom (fixed positioned, bottom sheet)
- Response options: full-width buttons, 52px height, 16px gap
- Progress: compact top bar "3 of 10" with thin line indicator

**Practice dashboard (phone)**:
- H-Score gauge: 140px diameter, centered, full-width card
- Components: accordion (tap to expand), not always-visible breakdown
- "Start Huddle" CTA: sticky bottom bar (always visible)

**Huddle player (tablet — primary)**:
- Landscape optimized: left 60% video, right 40% supplementary content
- Portrait mode: stacked, video top half, content bottom half
- Target: iPad 768×1024 in landscape (810×1080 after title bar)
- Font sizes: all text must be legible at 2m viewing distance

### PWA Configuration

```typescript
// next.config.ts
const withPWA = require('@ducanh2912/next-pwa').default({
  dest: 'public',
  cacheOnFrontEndNav: true,
  aggressiveFrontEndNavCaching: true,
  reloadOnOnline: true,
  disable: process.env.NODE_ENV === 'development',
  workboxOptions: {
    // Pre-cache module videos for offline viewing
    runtimeCaching: [
      {
        urlPattern: /^https:\/\/stream\.mux\.com\/.*/,
        handler: 'CacheFirst',
        options: {
          cacheName: 'mux-video-cache',
          expiration: { maxAgeSeconds: 7 * 24 * 60 * 60 }, // 7 days
        },
      },
      {
        urlPattern: /^https:\/\/api\.playbook\.sga\.com\/api\/v1\/huddles\/.*\/today/,
        handler: 'NetworkFirst',
        options: {
          cacheName: 'huddle-content-cache',
          expiration: { maxAgeSeconds: 24 * 60 * 60 }, // 24 hours
          networkTimeoutSeconds: 5,
        },
      },
    ],
  },
});
```

### Offline Capability

The morning huddle must work with poor connectivity (many rural practices have unreliable internet).

**Strategy**: Service Worker pre-caches today's content package (including Mux video HLS manifest and first 2 minutes of segments) the night before (Sunday 11 PM).

```typescript
// Service Worker: pre-cache tomorrow's huddle content
self.addEventListener('message', (event) => {
  if (event.data.type === 'PRECACHE_HUDDLE') {
    const { videoUrls, contentPackageUrl } = event.data;
    // Cache content package JSON
    // Cache first 128KB of each Mux HLS manifest
    // Note: full video pre-caching at ~50MB would be impractical over cellular
    // Strategy: cache the manifest only; Mux adaptive bitrate handles low-bandwidth streaming
  }
});
```

---

## Design Token Mapping (Tailwind)

```typescript
// tailwind.config.ts
module.exports = {
  theme: {
    extend: {
      colors: {
        bg: '#0f0f0f',
        card: '#171717',
        'card-2': '#1e1e1e',
        'card-3': '#252525',
        border: '#2a2825',
        'border-gold': '#3a3020',
        text: '#f0ece4',
        muted: '#8a857c',
        dim: '#5a5550',
        gold: '#c9a84c',
        'gold-light': '#e8c96a',
        'gold-dim': '#8a6f30',
        'level-emerging': '#c05a4a',
        'level-developing': '#c47a35',
        'level-established': '#c9b94c',
        'level-exceptional': '#4a9e6e',
        'level-elite': '#c9a84c',
      },
      fontFamily: {
        sans: ['-apple-system', 'BlinkMacSystemFont', "'Segoe UI'", 'sans-serif'],
        serif: ['Georgia', 'serif'],
      },
    },
  },
};
```

---

## Performance Requirements

| Metric | Target | Measured On |
|--------|--------|------------|
| First Contentful Paint (dashboard) | < 1.5s | 4G mobile, cold load |
| Largest Contentful Paint (dashboard) | < 2.5s | 4G mobile |
| Huddle player first frame | < 2.0s | WiFi tablet |
| Module video time-to-play | < 3.0s | 4G mobile |
| H-Score gauge animation | 60fps on all supported devices | |
| Dashboard data API response | < 500ms | Server-side |

**Key optimizations**:
- Next.js Server Components for dashboard initial load (no client JS for static data)
- H-Score gauge as pure SVG (no canvas, no JS library)
- Mux Player lazy-loaded (not bundled on dashboard page)
- Module list paginated (10 modules is small, but general practice for larger Silver/Gold tracks)
- CSS animations only (no JS animation libraries per design specs)
