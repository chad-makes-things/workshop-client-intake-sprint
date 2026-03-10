# Phase 1 Frontend Specification — Content Concierge

**Sprint**: Practice Media Strategy
**Phase**: 1 — The SMS Coach (Weeks 1–6)
**Scope**: Admin Configuration Dashboard (internal web app) + SMS UX design
**Users**: HQ admin team (practice onboarding, capture monitoring), human editors (Airtable-based)
**Date**: 2026-03-10

---

## Phase 1 Frontend Philosophy

Phase 1 has no mobile app and no consumer-facing UI — the champion experience is entirely SMS-based. The frontend in Phase 1 is the **internal admin dashboard** used by the SGA HQ team to configure practices, monitor capture activity, and support the launch.

This is a **low-UI phase**: Airtable handles the editor review queue. The admin dashboard fills the gap for what Airtable can't do — practice configuration, playbook management, champion onboarding, and capture health monitoring.

**Phase 2 adds**: React Native mobile app (camera, assignments, scorecard), Next.js HQ dashboard with full review queue.

---

## 1. Admin Dashboard — Overview

### Users

| Role | What They Do | Access |
|------|-------------|--------|
| HQ Admin | Configure practices, onboard champions, set playbooks, monitor captures | Full access |
| Content Editor | View capture queue status (read-only) | Read-only: captures view |

### Tech Stack

| Component | Tech | Rationale |
|-----------|------|-----------|
| Framework | Next.js 14 (App Router) | Server components for fast initial load; no SPA overhead for internal tool |
| UI Library | shadcn/ui + Tailwind CSS | Rapid component assembly; accessible defaults |
| Data Fetching | SWR (client) + Server Actions (mutations) | Simple; no GraphQL needed for internal tool |
| Auth | NextAuth.js with Google OAuth | SGA team uses Google Workspace |
| Deployment | Vercel | Zero-config deployment; preview branches for staging |
| API | Admin Service (port 8003) via Next.js API routes | Single backend surface |

---

## 2. Screen Inventory

| Screen | Path | Users | Description |
|--------|------|-------|-------------|
| Dashboard Home | `/` | HQ Admin | Network health summary: total captures today, active practices, pending review count, recent activity feed |
| Practices List | `/practices` | HQ Admin | All practices with health status, last capture, weekly capture count vs. target |
| Practice Detail | `/practices/[id]` | HQ Admin | Practice config, champion list, recent captures, assignment history, sync status |
| Add Practice | `/practices/new` | HQ Admin | Multi-step form: practice info → playbook assignment → champion onboarding |
| Champion Management | `/practices/[id]/champions` | HQ Admin | List champions, add/edit, view SMS opt-in status |
| Captures Monitor | `/captures` | HQ Admin, Editor | Live capture feed across all practices; filter by practice/date/type |
| Assignments Monitor | `/assignments` | HQ Admin | Today's and upcoming assignments across all practices |
| Playbook Library | `/playbooks` | HQ Admin | View and edit playbook guidance text per procedure code |
| Sync Health | `/sync` | HQ Admin | Sikka ONE API sync history per practice; re-trigger sync manually |
| Settings | `/settings` | HQ Admin | Twilio config, Airtable base URL, notification preferences |

---

## 3. Screen Specifications

### 3.1 Dashboard Home (`/`)

**Purpose**: Morning glance — is the system working? Is content flowing?

**Layout**: Single-page with 3 zones

**Zone 1 — Key Metrics Bar** (top, horizontal cards)

| Metric | Calculation | Color Threshold |
|--------|-------------|-----------------|
| Captures Today | COUNT captures WHERE date = today | Green ≥ (active_practices × 2), Yellow ≥ (active_practices × 1), Red < that |
| Active Practices | COUNT practices WHERE active | — |
| Pending Review | Airtable records WHERE status = "Needs Review" | Green <20, Yellow 20–50, Red >50 |
| Assignments Sent Today | COUNT assignments WHERE due_date = today | — |
| Weekly Capture Rate | Total this week / (practices × target) | Green ≥80%, Yellow 50–80%, Red <50% |

**Zone 2 — Practice Grid** (middle)

Grid of practice cards (2 or 3 columns). Each card shows:
- Practice name
- Champion name
- Captures this week vs. target (e.g., "4 / 5")
- Last capture timestamp
- Status dot: green (on track), yellow (behind), red (no captures in 3+ days)

Click → navigates to Practice Detail.

**Zone 3 — Activity Feed** (right sidebar or bottom)

Chronological list of recent events:
```
✅ 2 photos received — Innovative Dental Springfield (Jessica M.) — 2 min ago
📅 Daily brief sent — 14 champions — 7:30 AM
✅ 3 photos received — 4Front Implants Tulsa (Dr. Chen) — 1 hr ago
⚠️  Sikka sync failed — Riverside Family Dental — 11:23 PM yesterday
```

Each event links to the relevant practice or capture.

---

### 3.2 Practices List (`/practices`)

**Layout**: Table with search + filter

**Columns**:

| Column | Description |
|--------|-------------|
| Practice Name | Clickable → Practice Detail |
| Location | City, State |
| Playbook | Badge: Implant/Cosmetic, Specialty, Growth GP, Standard GP |
| Champion | Champion name |
| Captures This Week | "4 / 5" (count / weekly target) with color bar |
| Last Capture | Relative time ("3 hours ago", "2 days ago") |
| Sync Status | Green checkmark / yellow warning / red X |
| Actions | Edit, Deactivate |

**Filters** (top bar):
- Search by name
- Filter: Playbook type
- Filter: Status (Active / Inactive)
- Filter: Health (On Track / Behind / No Captures)

**Empty State**: "No practices yet. Add your first practice →"

---

### 3.3 Practice Detail (`/practices/[id]`)

**Tab structure**: Overview | Champions | Captures | Assignments | Config

**Overview Tab**

Top section: Practice name, location, playbook badge, champion name, Sikka ID, timezone.

Metrics row:
- Captures this week: `{n}` / `{target}` with progress bar
- Streak: `{n}` days (practice-level, based on any capture per day)
- Last Sikka sync: `{relative time}` + sync status
- Airtable pending: `{n}` items in review

Chart: Bar chart — daily capture count for last 14 days. Target line overlay.

**Champions Tab**

Table: Name, Role, Phone, SMS Opt-In, WhatsApp Opt-In, Captures This Week, Last Active.

Actions: Add Champion, Edit, Remove.

**Add Champion Form**:
```
Name: [text input]
Role: [dropdown: Champion | Doctor | Leader]
Phone: [E.164 phone input with validation]
Notify via: [SMS | WhatsApp | Both]
[Send Welcome SMS] button — sends: "Welcome to Content Concierge! Text your photos here..."
```

**Captures Tab**

Table of all captures for this practice, most recent first.

| Column | Description |
|--------|-------------|
| Date | Relative + absolute |
| Type | Photo / Video badge |
| Content Type | Before Photo, After Photo, Testimonial, etc. |
| Champion | Who sent it |
| S3 Thumbnail | Small preview (photos only) |
| Airtable Status | Needs Review / In Production / Approved / Rejected |
| Actions | View full, View in Airtable |

**Assignments Tab**

Table of capture assignments.

| Column | Description |
|--------|-------------|
| Due Date | Date |
| Champion | Who's assigned |
| Content Type | Type + Procedure |
| Patient | Initials only |
| Status | Pending / Captured / Expired / Skipped |
| Source | PMS / Content Calendar / Post-Op |

Actions: Mark as skipped, view capture (if captured).

**Config Tab**

Editable form:
```
Playbook Type: [dropdown]
Weekly Capture Target: [number input]
Sikka Practice ID: [text input] [Test Connection]
Timezone: [dropdown]
Active: [toggle]
```

---

### 3.4 Add Practice (`/practices/new`)

Multi-step wizard (3 steps, progress indicator at top):

**Step 1: Practice Info**
```
Practice Name: [text]
Location (City, State): [text]
PMS System: [dropdown: Open Dental | Dentrix | Eaglesoft | Other]
Sikka Practice ID: [text] [Test Connection button]
Timezone: [dropdown, auto-suggest from state]
```

**Step 2: Playbook Assignment**
```
Playbook Type: [card selection — 4 cards with descriptions]

  ┌─────────────────┐  ┌─────────────────┐
  │ Implant/Cosmetic│  │   Specialty     │
  │ High-value,     │  │  Ortho, perio,  │
  │ before/after    │  │  peds, OS       │
  │ focused         │  │                 │
  └─────────────────┘  └─────────────────┘
  ┌─────────────────┐  ┌─────────────────┐
  │   Growth GP     │  │   Standard GP   │
  │  High-volume    │  │  1-2 locations, │
  │  multi-location │  │  maintenance    │
  └─────────────────┘  └─────────────────┘

Weekly Capture Target: [auto-set from playbook, editable]
```

**Step 3: Champion Onboarding**
```
Champion Name: [text]
Champion Phone: [phone input]
Notify via: [SMS | WhatsApp | Both]

[Add Another Champion] link

[Send Onboarding SMS] toggle — sends welcome + first guidance immediately
```

On completion: redirect to Practice Detail → Overview tab. Toast: "Practice added. Onboarding SMS sent to {name}."

---

### 3.5 Captures Monitor (`/captures`)

**Purpose**: Real-time feed of all captures across the network. Used by editors and HQ to monitor flow.

**Layout**: Table with live refresh (SWR, 30-second polling)

**Filters** (top):
- Date range (default: today)
- Practice (multi-select)
- Media type (Photo / Video / All)
- Airtable status (Needs Review / In Production / Approved / All)

**Table Columns**:

| Column | Description |
|--------|-------------|
| Time | Relative + absolute |
| Practice | Practice name |
| Champion | Who sent it |
| Media | Thumbnail (photo) or duration (video) |
| Type | Content type badge |
| Airtable | Status badge + "Open in Airtable" link |

**Stat bar above table**:
```
Today: 47 captures | 12 practices active | 23 pending review | 8 approved
```

---

### 3.6 Sync Health (`/sync`)

**Purpose**: Monitor Sikka ONE sync jobs. Debug when assignments are missing.

**Table**: One row per practice per sync job.

| Column | Description |
|--------|-------------|
| Practice | Name |
| Last Sync | Timestamp |
| Status | Success / Failed / In Progress |
| Appointments Pulled | Count |
| Assignments Created | Count |
| Error | Error message if failed |
| Actions | Re-trigger sync for this practice |

**Manual Sync Trigger**: Select practice(s) → "Run Sync Now" button. Shows real-time log in a slide-over panel.

---

### 3.7 Playbook Library (`/playbooks`)

**Purpose**: HQ team can edit the guidance text that goes into SMS prompts and assignments.

**Layout**: Sidebar navigation (playbook type) + main content (procedure cards)

**Playbook Sidebar**:
- Implant/Cosmetic
- Specialty
- Growth GP
- Standard GP
- SGA Brand

**Procedure Cards** (for selected playbook):

Each card shows:
```
Procedure Code: D6010 (Implant Placement)
Content Type: Before/After + Testimonial

Capture Guidance:
[editable textarea]
"Take before photo from front and 45° angle, fill frame with smile..."

Example Caption:
[editable textarea]
"Another smile transformation begins today! 🦷..."

[Save Changes] [Reset to Default]
```

---

## 4. SMS UX Design

The champion's primary interface in Phase 1 is SMS. This is not a visual screen spec — it's a conversation design spec.

### 4.1 Onboarding SMS Sequence (Day 1)

**Message 1** (sent immediately upon champion creation):
```
Hi {first_name}! 👋 This is the SGA Content Concierge.

You're set up to capture content for {practice_name}.

To send us a photo or video: just text it to this number.
To see your assignments for today: text STATUS
To stop messages: text STOP

Tomorrow morning at 7:30 AM, you'll get your first daily brief!
```

**Message 2** (sent 15 min later if WhatsApp opted in):
```
We can also connect via WhatsApp for easier photo sharing. Reply YES to switch to WhatsApp for your content captures.
```

### 4.2 Daily Brief Format

```
Good morning {first_name}! Here's your capture list for today:

1️⃣  BEFORE PHOTO — Full Arch Implant at 9:30 AM
💡 Front + 45° angle, fill frame with smile

2️⃣  TESTIMONIAL — Smile Makeover patient finishing today
💡 "Would you share your experience on video? Just 30 seconds!"

3️⃣  TEAM PHOTO — No appointment needed
💡 Capture your team in action — authentic beats perfect!

Reply with your photos/videos any time today. 📸
```

Rules:
- Max 3 assignments in SMS brief (prioritize by procedure value)
- If 0 assignments: no SMS sent (no "nothing today" messages — silence is correct)
- Procedure details are generic (avoid PHI in SMS body)
- Patient identified by initials only if referenced

### 4.3 Capture Confirmation

```
Got it! ✅ 2 photos received.

Your SGA editors will review and publish within 24 hours.

This week: 4 captures 🔥 (target: 5)
```

- Confirmation sent immediately after each MMS/WhatsApp batch
- Streak + weekly count always included — drives behavior

### 4.4 Rejection Feedback (triggered from Airtable automation)

When editor marks a capture "Rejected" in Airtable → webhook → SMS:

```
Quick note on your last {content_type}: {editor_notes}

No worries — try again next time! You've got this 💪
```

- Keep it warm and encouraging — champions are not trained photographers
- Never passive-aggressive or punitive

### 4.5 Weekly Recap (Monday 8:00 AM)

Sent every Monday:
```
{first_name}'s week in review 📊

Last week: {capture_count} captures (target: {target})
Published: {published_count} pieces of content
Top piece: {highest_engagement_content_type} got {engagement_summary}
Streak: {streak_days} days 🔥

This week's playbook focus: {weekly_theme}
Your first prompt arrives tomorrow at 7:30 AM!
```

---

## 5. Component Library (shadcn/ui Selections)

| Component | shadcn Component | Usage |
|-----------|-----------------|-------|
| Practice health card | `Card` | Dashboard grid |
| Data tables | `DataTable` (TanStack Table) | Practices, Captures, Assignments lists |
| Status badges | `Badge` | Playbook types, capture status, sync status |
| Multi-step form | `Stepper` (custom) + `Form` | Add Practice wizard |
| Practice detail tabs | `Tabs` | Practice Detail screen |
| Metric cards | `Card` + custom metric layout | Dashboard key metrics |
| Activity feed | Custom `ScrollArea` | Dashboard activity feed |
| Alert | `Alert` | Sync failure warnings, error states |
| Toast | `Sonner` (toast) | Confirmation messages |
| Date picker | `Calendar` + `Popover` | Date filters |
| Command palette | `Command` | Practice search |

---

## 6. Auth & Access Control

**Auth Provider**: NextAuth.js with Google OAuth
**Allowed Domains**: `@sgadental.com` (restrict via NextAuth `signIn` callback)

| Role | Capability | Assigned By |
|------|-----------|-------------|
| `hq_admin` | Full access to all admin functions | Auto-assigned to first Google login |
| `editor` | Read-only: captures, Airtable link | Manual assignment by admin |

**Implementation**:
```typescript
// middleware.ts — protect all /admin routes
// Role stored in NextAuth session via JWT callback
// Check role on each Server Action that mutates data
```

---

## 7. Deployment Configuration

### Vercel Project Settings

```
Framework: Next.js
Root Directory: /
Build Command: next build
Output Directory: .next
Node.js Version: 20.x
```

### Environment Variables (Vercel)

| Variable | Description |
|----------|-------------|
| `NEXTAUTH_URL` | https://admin.contentconcierge.sgadental.com |
| `NEXTAUTH_SECRET` | Random 32-char secret |
| `GOOGLE_CLIENT_ID` | Google OAuth app client ID |
| `GOOGLE_CLIENT_SECRET` | Google OAuth app client secret |
| `ADMIN_API_URL` | Internal Admin Service URL (ECS private) |
| `ADMIN_API_KEY` | Static API key for Admin Service |
| `AIRTABLE_BASE_URL` | Link to Airtable base for editor "Open in Airtable" links |

### Custom Domain
`admin.contentconcierge.sgadental.com` — Vercel DNS or CNAME to Vercel

---

## 8. Phase 1 Frontend Scope vs. Phase 2

| Feature | Phase 1 (This Spec) | Phase 2 |
|---------|--------------------|----|
| Champion interface | SMS only | React Native mobile app |
| Review queue UI | Airtable (external) | Next.js HQ dashboard built-in |
| Admin dashboard | This spec | Enhanced with analytics, scorecard |
| Real-time updates | SWR polling (30s) | WebSocket |
| Content preview | S3 presigned URL in table | In-app media viewer with annotation |
| Analytics | Basic capture counts | Full content → engagement attribution |
| Network scorecard | Practice grid cards | Ranking engine, champion leaderboard |
