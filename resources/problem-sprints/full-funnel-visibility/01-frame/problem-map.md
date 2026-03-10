# Problem Map

## Actors

| Actor | Description |
|-------|-------------|
| **Executive Leadership** | CEO, VP-level leaders who need real-time visibility into the economic health of all 260+ practices. Currently waiting 30+ days post month-end for incomplete data. Need to spot problems early and allocate resources proactively. |
| **HQ Operations Team** | Central team responsible for managing practice performance, identifying underperforming practices, and coordinating interventions. Currently flying blind — assembling reports manually from multiple systems. |
| **Practice Office Manager** | The person at each practice who is the primary interface with HQ. Has ~20 minutes/day for HQ-directed activities across 10+ tools. Responsible for ensuring the front desk is running well. |
| **Front Desk Staff** | The people answering phones, booking appointments, confirming visits, handling walk-ins. They are the funnel. Their actions (or inactions) directly determine conversion rates. They don't think in "funnel metrics." |
| **AI Phone Agents** | Sierra, Agent Agnes, and other AI phone systems that answer calls when staff can't, route conversations, and integrate with PMS for automated actions. A growing but not yet standardized layer. |
| **Patient / Prospective Patient** | The person calling, filling out a form, or walking in. They experience the funnel. Their journey from "I need a dentist" to "I'm in the chair" is what the entire system needs to track and optimize. |
| **Advertising / Lead Gen Partners** | External agencies and platforms generating leads (Google Ads, Meta, LSAs, etc.). SGA doesn't always control this directly — some practices have their own ad relationships. The top of the funnel. |

---

## The Patient Funnel (The P&L We're Managing)

```
FUNNEL STEP          WHAT HAPPENS                                   KEY METRICS
─────────────────────────────────────────────────────────────────────────────────

┌─────────────────┐
│ 1. LEADS        │  Patient needs a dentist. Sees an ad, finds     • Leads generated / week
│    GENERATED    │  a GMB listing, gets a referral, visits the     • Cost per lead
│                 │  website. A "lead" enters the system.           • Lead source attribution
└────────┬────────┘                                                  • Form fills, calls, chats
         │
         ▼
┌─────────────────┐
│ 2. LEADS        │  Lead calls or is called back. Phone is         • Phone answer rate
│    CONNECTED    │  answered (by human or AI). Form submission      • Speed to first contact
│    WITH         │  gets a response. Chat is engaged. The lead     • Connection rate (% of leads
│                 │  has a two-way interaction with the practice.     contacted within 5 min)
└────────┬────────┘                                                  • Call quality score (AI analysis)
         │
         ▼
┌─────────────────┐
│ 3. LEADS        │  Connection converts to a scheduled              • Booking rate (% of connections
│    BOOKED       │  appointment. Patient is in the PMS              that book)
│                 │  with a date and time. Confirmation              • Time from contact to appointment
│                 │  sequence triggered.                              • Appointment type distribution
└────────┬────────┘                                                  • Cancellation/reschedule rate
         │
         ▼
┌─────────────────┐
│ 4. PATIENT      │  Patient actually arrives for the                • Show rate (% of booked that
│    SHOWS UP     │  appointment. Different service types             arrive)
│                 │  have wildly different show rates                 • No-show rate by service type
│                 │  (routine cleaning ~85% vs. implant              • Confirmation touchpoints
│                 │  consult ~60-65%).                                completed
└────────┬────────┘                                                  • Same-day cancellations
         │
         ▼
┌─────────────────┐
│ 5. PRODUCTION / │  Patient accepts treatment. Revenue is           • Case acceptance rate
│    IN THE CHAIR │  generated. For simple services, this is         • Production per visit
│    ("LEAD SOLD")│  same-visit. For complex (implants,             • Treatment plan value
│                 │  ortho), there's a consult → acceptance          • Revenue per lead (full funnel
│                 │  → treatment pipeline.                             ROI)
└─────────────────┘

TOTAL FUNNEL: Lead → Connected → Booked → Showed → Produced = Practice P&L
```

---

## Journey Map (System View)

```
ACTORS                     JOURNEY                                                         PROBLEM

                           ┌──────────┐   ┌──────────┐   ┌──────────┐
Advertising /              │ 1. Lead  │   │ 2. Lead  │   │ 3. Lead  │
Lead Gen Partners ────────▶│ Generated│──▶│ Captured │──▶│ Routed   │
                           │ (ads,    │   │ (form,   │   │ (to right│
                           │ organic) │   │ call,    │   │ practice/│
                           └──────────┘   │ chat)    │   │ person)  │
                                          └──────────┘   └────┬─────┘
                                                              │
                           ┌──────────┐   ┌──────────┐       │
Front Desk Staff ─────────▶│ 4. Phone │──▶│ 5. Book  │◀──────┘
AI Phone Agents ──────────▶│ Answered │   │ Appoint- │
                           │ (human   │   │ ment     │
                           │ or AI)   │   │          │
                           └──────────┘   └────┬─────┘
                                               │
                           ┌──────────┐   ┌──────────┐   ┌──────────┐
Front Desk / ─────────────▶│ 6.Confirm│──▶│ 7.Patient│──▶│ 8.Treat- │    Real-time
Practice Staff             │ & Remind │   │ Arrives  │   │ ment &   │    visibility into
                           │          │   │          │   │ Productn │    EVERY step of
                           └──────────┘   └──────────┘   └────┬─────┘    this journey
                                                              │           across 260+
                           ┌──────────┐   ┌──────────┐       │           practices
Office Manager ───────────▶│ 9. Daily │──▶│ 10.Report│◀──────┘
                           │ Ops &    │   │ to HQ    │
                           │ Follow-up│   │          │               ┌──────────┐
                           └──────────┘   └────┬─────┘               │ 12. Exec │
                                               │                     │ Dashboard│
HQ Operations ────────────────────────────────▶│                     │ & AI     │
Team                       ┌──────────┐        │                     │ Analysis │
                           │ 11.      │◀───────┘                     └──────────┘
                           │ Aggregate│                                    ▲
                           │ & Analyze│────────────────────────────────────┘
                           └──────────┘

Executive Leadership ──────────────────────────────────────────────▶ NEEDS: Real-time
                                                                      health of all
                                                                      260+ funnels
```

---

## Journey Steps (Detail)

### Step 1: Lead Generated
- **Description**: A prospective patient sees an ad, finds the practice on Google, gets a referral, or visits the website. A "lead" enters the top of the funnel.
- **Critical**: YES
- **Current State**: SGA doesn't directly control all ad spend — some practices have their own agency relationships. Lead source attribution is inconsistent. No standard way to measure "how many leads did this practice get this week?"
- **Risk**: If you can't count leads accurately, every downstream metric is unreliable. Garbage in, garbage out.

### Step 2: Lead Captured
- **Description**: The lead fills out a form, calls the practice, or initiates a chat. Their information is captured in some system.
- **Critical**: YES — THIS IS THE BIGGEST MISSING PIECE
- **Current State**: No standard CRM across practices. Form submissions go to different places depending on the practice. Some practices have no form capture at all. There is no unified "lead entered the system" event.
- **Risk**: Without a standard lead capture point, the funnel has no measurable starting gate. Leads fall through cracks because nobody knows they existed.

### Step 3: Lead Routed
- **Description**: The captured lead is routed to the right person — front desk for scheduling, specific provider for specialty, AI agent for after-hours.
- **Critical**: MEDIUM
- **Current State**: Routing is manual and practice-dependent. There's no standard triage logic. After-hours calls may go to voicemail (lost) or to an AI agent (captured, but not always routed back correctly).
- **Risk**: Slow or incorrect routing kills conversion. A patient who fills out a form and waits 3 days for a callback is already calling the practice down the street.

### Step 4: Phone Answered / Connection Made
- **Description**: The lead has a two-way interaction with the practice — phone call answered (by human or AI), chat responded to, form follow-up call made.
- **Critical**: YES — THIS IS WHERE MOST LEADS DIE
- **Current State**: Multiple phone systems across practices (no single standard). AI phone agents (Sierra, Agent Agnes) are being adopted but not universal. Phone answer rates vary wildly — some practices answer 90%, some answer 50%. Missed calls are often not tracked or followed up.
- **Risk**: A lead who can't connect is a lead you paid for and lost. Phone answering is the single highest-leverage point in the funnel. AI conversation analysis exists but isn't feeding back into any unified system.

### Step 5: Appointment Booked
- **Description**: The connection converts to a scheduled appointment. Patient is in the PMS with a date and time.
- **Critical**: YES
- **Current State**: Booking happens in the practice management software (PMS). Different practices use different PMS vendors. The "booking" data is locked inside each PMS. No standard way to see "how many new patient appointments were booked this week across the network?"
- **Risk**: Without booking visibility, you can't measure the connection-to-booking conversion rate — which tells you about front desk skill, script quality, and whether the practice is actually converting interest into appointments.

### Step 6: Confirmation & Reminders
- **Description**: Automated (or manual) confirmation texts, emails, and calls to ensure the patient remembers their appointment and shows up.
- **Critical**: MEDIUM (but high-leverage for show rate)
- **Current State**: Some practices have automated reminders through their PMS. Some use 3rd-party tools. Some do it manually. No standard across the network. The quality and timing of reminders varies wildly.
- **Risk**: Show rates are the difference between a productive day and a wasted chair. Confirmation systems are one of the most standardizable and automatable parts of the funnel.

### Step 7: Patient Arrives (Shows Up)
- **Description**: The patient actually arrives for their appointment.
- **Critical**: YES
- **Current State**: Show rates tracked in PMS but not surfaced anywhere at the network level. No real-time visibility into "how many appointments today are confirmed vs. at risk?" Different service types have dramatically different show rate profiles.
- **Risk**: No-shows are a silent revenue killer. Without real-time show-rate tracking, you can't intervene (e.g., overbooking, same-day outreach to at-risk appointments).

### Step 8: Treatment & Production
- **Description**: Patient accepts treatment. Revenue is generated. For simple services, this is same-visit. For complex cases, there's a consult → acceptance → scheduling → treatment pipeline.
- **Critical**: YES — THIS IS THE BOTTOM LINE
- **Current State**: Production data lives in PMS and financial systems. Monthly reconciliation takes 30+ days. No real-time production tracking at the network level.
- **Risk**: By the time you know a practice is behind on production targets, the month is over and you can't do anything about it.

### Step 9: Daily Operations & Follow-Up
- **Description**: Office manager and staff handle no-show follow-ups, unscheduled treatment plans, recall/recare outreach, and general practice operations.
- **Critical**: YES (in aggregate)
- **Current State**: Practice staff have ~20 min/day for HQ-directed activities. They're logging into 10+ tools. No prioritized agenda. No visibility into which activities have the highest ROI.
- **Risk**: When everything is a priority, nothing is. Staff do what's easiest or most familiar, not what's most impactful.

### Step 10: Report to HQ
- **Description**: Practice data flows (or doesn't) to HQ for aggregation, analysis, and decision-making.
- **Critical**: YES — THIS IS THE CORE BROKEN STEP
- **Current State**: Data lives in 75+ individual systems. Aggregation is manual. Reports are 30+ days lagged. No standard data model. HQ spends more time assembling data than analyzing it.
- **Risk**: This is the step where the entire visibility problem manifests. Everything upstream might be working fine at individual practices, but HQ can't see it, compare it, or act on it.

---

## Critical Moments

| Step | Risk | Opportunity |
|------|------|-------------|
| **Step 2: Lead Captured** | No standard CRM — leads fall through cracks, no measurable funnel start | A unified lead capture layer is the single biggest unlock. It's the missing standard piece. |
| **Step 4: Phone Answered** | Wildly variable answer rates. Missed calls = lost revenue. AI agents not integrated. | Phone answering is the highest-leverage single intervention point. AI analysis of calls already exists — it just needs to feed the funnel view. |
| **Step 8: Production** | 30-day lag to know if a practice is on track. Can't intervene retroactively. | Real-time production tracking enables same-week intervention: "You're 15% behind target — here's why and what to do." |
| **Step 10: Report to HQ** | 75+ systems, manual aggregation, 30-day lag, incomplete picture. | This is the core unlock: if you solve the data aggregation problem, every other insight becomes possible. |
| **Step 9: Daily Ops** | 20 min/day, 10 tools, no prioritization = lowest-value tasks get done. | A prioritized daily agenda replaces 10 tools with 1 interface and tells staff exactly what moves the needle today. |
