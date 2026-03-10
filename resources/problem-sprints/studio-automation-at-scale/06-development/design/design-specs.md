# Design Specifications

**UX Designer:** Development Team — UX Role
**Sprint source:** Studio Automation at Scale
**Date:** 2026-03-10
**Source:** Storyboard (8 steps), Solution A narrative, HTML visualization

---

## Design Principle: Email is a Product Surface

The practice manager's primary interface is **email** — not a web app. The Studio Agent's reply emails are the UX. This fundamentally changes the design brief: we are designing structured, information-dense emails that guide a non-designer through a creative workflow. The email format is a first-class product surface, not an afterthought.

**Voice principle:** Friendly, knowledgeable colleague — not a support ticket system. "Hi Ashley! Got your request." not "Ticket #4471 received."

---

## Screen Inventory

### Surface 1 — Agent Reply Email (Practice Manager View)

The most-used surface. Every practice manager interaction happens here.

**Email 1.1 — Intake Acknowledgment + Clarifying Questions**

```
FROM: Studio Agent <studio@sgadental.com>
TO: ashley@townlakedental.com
SUBJECT: Re: [original subject]

Hi Ashley!

Got your request for the April whitening special. Quick questions before
I get started:

  1. Who's the main audience — existing patients, new patients, or both?
  2. Any deadline for when materials need to be ready?
  3. Would you like to feature a specific doctor, or keep it
     practice-branded?

Also — before I build a flyer, I want to show you something that
might work even better: a patient email + in-office TV slide would
reach all your active patients for free, and I can have them ready today.

Happy to do both if you still want print too. Just let me know!

— Studio Agent
studio@sgadental.com
```

**Design requirements:**
- Maximum 3 numbered questions — always numbered, never bulleted
- Medium challenge always present when print is implied — positioned after the questions, not before
- Sign-off is always "Studio Agent" — not a person's name
- Reply-to is `studio@sgadental.com`
- No HTML styling in Phase 1 (plain text email is safer for delivery and accessibility)
- Phase 2: consider branded HTML email template with practice logo in header

---

**Email 1.2 — Preview Delivery**

```
FROM: Studio Agent <studio@sgadental.com>
TO: ashley@townlakedental.com
SUBJECT: Re: [original subject]

Hi Ashley,

Here are your three April whitening special assets:

  ┌─────────────────────────────────┐
  │  Patient Email Header            │
  │  [Inline image: 600px preview]  │
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  In-Office TV Slide              │
  │  [Inline image: 800px preview]  │
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  Desk Flyer (4×6 print)         │
  │  [Inline image: 600px preview]  │
  └─────────────────────────────────┘

Everything look right? Reply with:
  → "Looks good" to send for final review
  → "Change [the CTA / the headline / etc.]" to make a text update

— Studio Agent
```

**Design requirements:**
- Assets embedded as inline images (not attachments) — visible without clicking
- Preview images: min 600px wide, JPG
- Two-option reply guidance always at the bottom — never more than 2 options
- Label each asset by name + format/size
- No numbered list for the options — → bullet styling avoids visual confusion with the question numbers from intake

---

**Email 1.3 — Approval Confirmation + Print Order**

```
FROM: Studio Agent <studio@sgadental.com>
TO: ashley@townlakedental.com
SUBJECT: Re: [original subject] — Assets Approved ✓

Hi Ashley,

All three assets are approved and on their way:

  ✓ Patient email — scheduled for April 1 to your active patient list
  ✓ TV slide — delivered below, ready to upload to your display
  ✓ Desk flyer — ready to order

  [TV slide image — full size]

For the desk flyer:

  → ORDER 50 COPIES ($XX.XX — ships in 5 business days)
     [This link triggers the Gelato order — one click, ships to your office]

  → DOWNLOAD PDF INSTEAD
     [For local printing or digital use]

Anything else you need for the April promotion?

— Studio Agent
```

**Design requirements:**
- Checkmarks (✓) for approved/deployed items — visual confirmation hierarchy
- Print order link is prominent — rendered as a full-line CTA, not inline text
- Price and lead time shown before the click — no surprises
- TV slide delivered inline as a full-size image — ready to screenshot and upload
- One question at the end inviting follow-up — keeps the conversation open

---

### Surface 2 — CD Mobile Review Notification (Teams)

**Design brief:** The CD reviews from his phone between patients and calls. This notification must be completable in under 60 seconds without opening any other app.

**Notification layout:**

```
╔══════════════════════════════════════════╗
║  🎨 Studio Agent                         ║
║  Towne Lake Dental — 3 assets for review ║
╠══════════════════════════════════════════╣
║                                          ║
║  [Thumbnail: Email Header — 120px]       ║
║  [Thumbnail: TV Slide — 120px]           ║
║  [Thumbnail: Desk Flyer — 120px]         ║
║                                          ║
║  Whitening Special · April 2026          ║
║  Requested by: Ashley                    ║
║                                          ║
║  ┌──────────┐ ┌────────────┐ ┌────────┐  ║
║  │ APPROVE  │ │   CHANGE   │ │ REJECT │  ║
║  │   ALL    │ │            │ │        │  ║
║  └──────────┘ └────────────┘ └────────┘  ║
╚══════════════════════════════════════════╝
```

**Design requirements:**
- Three thumbnails visible without scrolling on a standard iPhone (375px viewport)
- Thumbnail size: 100-120px square, cropped to show key visual — not full asset preview
- Three-button action bar always visible at the bottom — no scrolling required to reach it
- Approve button: primary color (filled) — most common action should be most prominent
- Change button: secondary (outlined) — opens a text field overlay: "What needs to change?"
- Reject button: tertiary (text-only, destructive color) — least common action, lowest visual weight
- Practice name, collateral type, and requester name always shown — CD must know context before approving

---

### Surface 3 — Studio Analytics Dashboard (Web)

**Primary users:** Creative Director, Studio Admin
**Access:** Internal — SGA Microsoft ecosystem (SSO via Azure AD)
**Frequency:** CD checks daily; Studio Admin checks weekly

**Dashboard layout:**

```
┌─────────────────────────────────────────────────────┐
│  SGA Studio Dashboard          [Week ▾] [All Practices ▾] │
├──────────────┬──────────────┬──────────────┬──────────────┤
│ Requests     │ Auto-Fulfill │ Avg Turnaround│ Print Volume │
│ This Week    │ Rate         │              │ vs. Baseline │
│  23          │  83%         │  58 min      │  -34%        │
│  ↑ 4 vs last │              │  ↓ from 4.2d │  ↓ good      │
├──────────────┴──────────────┴──────────────┴──────────────┤
│                                                           │
│  MEDIUM SPLIT (this week)                                 │
│  ████████████████░░░░░░░░░░░  71% Digital Only           │
│  ██████░░░░░░░░░░░░░░░░░░░░░  22% Digital + Print        │
│  ██░░░░░░░░░░░░░░░░░░░░░░░░░   7% Print Only             │
│                                                           │
├───────────────────────────────────────────────────────────┤
│  RECENT REQUESTS                           [View All →]   │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ✅ Towne Lake Dental · Whitening Flyer · 52 min      │  │
│  │ ✅ Dakota Dental · Business Cards · 34 min           │  │
│  │ ⏳ Monterey Coast Perio · Event Signage · In Queue  │  │
│  │ ✅ East Sac Dental · Appt Reminder · 1h 12 min      │  │
│  └─────────────────────────────────────────────────────┘  │
├───────────────────────────────────────────────────────────┤
│  BRAND KIT HEALTH                                         │
│  ✅ 11 practices — Complete (score ≥70)                   │
│  ⚠️  3 practices — Partial (score 40-69)                  │
│  🔴  1 practice — Needs asset lift (score <40)            │
│  [View All →]                                             │
├───────────────────────────────────────────────────────────┤
│  INACTIVE PRACTICES (no request in 90+ days)              │
│  3 practices — [Send proactive check-in →]                │
└───────────────────────────────────────────────────────────┘
```

**Design requirements:**
- Four KPI tiles at top: always visible, always current week vs. prior week
- Medium split bar chart: update weekly — this is the primary behavioral change metric
- Recent requests list: status icons (✅ complete, ⏳ in queue, ⚠️ CD review needed, 🔴 human queue)
- Brand kit health: traffic light system, not a table — studio needs to know at a glance which practices need attention
- Inactive practices section: actionable (sends proactive outreach from the agent, not a manual email)
- No pagination on initial view — top 5 most recent requests visible; "View All" goes to full log

---

### Surface 4 — Studio Admin Interface (Web)

**Primary users:** Studio Admin
**Access:** Internal — Azure AD SSO

**Practice profile editor:**
- Searchable practice list with brand health score badge
- Practice detail view: contact info, brand kit link (opens Canva), uploaded assets (logo files, photos), last 10 requests
- Asset upload: drag-and-drop logo files + photos; auto-syncs to Canva brand kit on save
- Brand health score breakdown: shows exactly which elements are missing (missing horizontal logo: -15 pts, etc.)

**Template library view:**
- 8 Phase 1 templates shown as thumbnail cards
- Each template: name, collateral type, last used date, Canva link (opens template in Canva for studio admin to edit)
- "Add template" button — opens Canva for the admin to build a new template; once saved in Canva, admin registers it in the system (name, type, variable field map)

---

## Design System

### Colors (aligned with SGA brand)
| Token | Value | Use |
|-------|-------|-----|
| `--navy` | #1B3A5C | Primary brand, headings, hero backgrounds |
| `--teal` | #0D9488 | Positive actions, success states, "approved" indicators |
| `--blue` | #2563EB | Interactive elements, links, primary buttons |
| `--amber` | #D97706 | Warning states, "in review" indicators |
| `--red` | #DC2626 | Destructive actions, "needs attention" indicators |
| `--gray-100` | #F1F5F9 | Dashboard backgrounds |

### Typography
- **Dashboard headings:** System UI, -apple-system, 700 weight
- **Body / data labels:** System UI, 400-500 weight
- **Email:** Plain text — no font spec required (client-rendered)
- **Monospace (request IDs, timestamps):** system monospace

### Accessibility
- All dashboard color usage must pass WCAG AA contrast ratio (4.5:1 for body text)
- CD review notification: button touch targets minimum 44×44px on mobile
- All status icons (✅ ⚠️ 🔴) must have text labels alongside — not icon-only
- Email content: plain text in Phase 1 ensures maximum accessibility and delivery reliability

---

## Interaction Patterns

### The Medium Challenge (Critical Pattern)
- **Trigger:** Any request containing print-implying language
- **Placement:** After the clarifying questions, before the sign-off
- **Tone:** "Before I build X, here's something that might work even better" — never "Are you sure you want print?"
- **Structure:** Specific benefit (reach, cost, speed) + specific alternative format + non-blocking close ("Happy to do both")
- **Never:** Make the challenge feel like a refusal. The practice always gets what they asked for. The challenge surfaces an option.

### The Revision Loop (Max 2)
- **Round 1:** Agent updates variable, sends revised preview in <2 min
- **Round 2:** Agent updates variable, sends revised preview in <2 min
- **Round 3 trigger:** Agent messages: "I want to make sure we get this perfect — let me hand this to our designer who'll have more flexibility with the layout. You'll hear back within 1 business day." Routes to human queue.

### CD Review Escalation
- 2-hour mark: Teams reminder notification (same format, adds "(Reminder)" to title)
- 4-hour mark: auto-escalation message to backup reviewer — CD notification updated to "Review completed by [backup]"
- CD is never blocked from approving after escalation — they see the outcome in their notification history

---

## Phase 2 Design Extensions (Not in Scope for Phase 1)

- Branded HTML email template for agent replies (practice logo in header, SGA footer)
- TV slide auto-deployment confirmation in the approval email
- Slack channel intake UI
- Practice manager self-service portal (Solution B elements) as a complement to the agent
- Proactive monthly content calendar UI (Solution C evolution)
