# Problem Map

## Map Overview
The patient communication lifecycle from first contact through long-term retention, showing where communications break down across SGA's practice network.

---

## Actors

| Actor | Description |
|-------|-------------|
| **New Patient (Lead)** | Someone who calls, fills out a web form, or clicks an ad — hasn't visited yet |
| **Existing Patient** | Active patient in the practice — comes for hygiene, may have outstanding treatment |
| **Lapsed Patient** | Was a patient, hasn't been seen in 12+ months — sitting in the database |
| **Front Desk Staff** | Answers phones, checks forms, schedules appointments — always multitasking |
| **Treatment Coordinator (TC)** | Presents treatment plans, handles financial conversations — key conversion role |
| **Office Manager** | Oversees daily operations, manages staff, owns the communication workflow |
| **Dentist/Provider** | Diagnoses, recommends treatment, relies on staff for follow-through |
| **AI Phone Agent (Sierra)** | Handles overflow calls, missed-call text-back — deployed inconsistently |
| **HQ Operations Team** | Monitors network-wide performance, intervenes when practices need help |

---

## Journey Map

```
NEW PATIENT LEAD PATH:

Ad/Search/Referral → Calls Practice ──→ Phone Answered? ──YES──→ Conversation ──→ Appointment Booked? ──YES──→ [BOOKED]
                  │                          │                                          │
                  │                          NO                                         NO
                  │                          ↓                                          ↓
                  │                    Voicemail/AI ──→ Text-back? ──→ [MAYBE]    Price/Insurance
                  │                          │              │          Objection        │
                  │                          │              NO         ↓                ↓
                  │                          ↓              ↓        [LOST]          [LOST]
                  │                    [LOST - nobody
                  │                     checks VM]
                  │
                  └→ Web Form Fill ──→ Form Checked? ──YES──→ Response ──→ Booked? ──→ [BOOKED]
                                          │                                    │
                                          NO (checked 1x/week)                NO
                                          ↓                                    ↓
                                    [LOST - days pass]                      [LOST]


TREATMENT PRESENTATION PATH:

Patient Visit → Diagnosis → Treatment Plan Presented → Accepts? ──YES──→ Scheduled ──→ [CONVERTED]
                                    │                       │
                                    │                       NO / "Let me think about it"
                                    │                       ↓
                                    │                  Follow-up? ──YES──→ Day 3 text ──→ 50% close
                                    │                       │                   │
                                    │                       NO (99% of cases)   ↓
                                    │                       ↓              Day 7, 14, 30 ──→ 90% close
                                    │                  [LOST - outstanding     by 5th touch ($20k+)
                                    │                   treatment, never
                                    │                   followed up]
                                    │
                                    └──→ AI Note-Taker captures ──→ Treatment mentioned? ──→ Should trigger
                                         in-room conversation           but DOESN'T            follow-up
                                                                        (no system              even if TC
                                                                         connects               didn't build
                                                                         the data)              a formal plan


REBOOKING PATH:

Treatment Complete → Patient at Front Desk → Rebook 6-month? ──YES──→ [RETAINED]
                                                    │
                                                    NO ("I'll call later")
                                                    ↓
                                              Reminder sent? ──→ Automated? ──→ [MAYBE]
                                                    │
                                                    NO
                                                    ↓
                                              [LAPSED - joins the
                                               database of ghosts]


DATABASE REACTIVATION PATH:

Lapsed Patient (12+ months) → In database → Anyone reach out? ──YES──→ Re-engaged ──→ [REACTIVATED]
                                                    │
                                                    NO (no system, no time)
                                                    ↓
                                              [PERMANENTLY LOST]
```

---

## Failure Points (Where Communications Break Down)

| # | Failure Point | Current State | Impact |
|---|--------------|---------------|--------|
| 1 | **Phone not answered** | Rings to VM, sometimes AI picks up | 30-40% of calls missed at many practices |
| 2 | **Form fills unchecked** | Inbox checked ~1x/week at some practices | Leads go cold in hours, not days |
| 3 | **No lead follow-up sequence** | One-and-done: if first contact fails, lead is abandoned | Massive funnel leakage at top |
| 4 | **Treatment plan not followed up** | TC presents, patient says "let me think," nobody follows up | 50%+ of closeable treatment lost |
| 5 | **TC resistance to follow-up** | 70% feel it's "salesy," refuse or avoid | Human behavior blocks even manual processes |
| 6 | **No CRM / task list** | No system to track who needs follow-up and when | Staff can't execute even when willing |
| 7 | **Rebooking not enforced** | Patients leave without 6-month appointment | Hygiene attrition compounds over time |
| 8 | **Lapsed patient database unworked** | Thousands of former patients, zero outreach | Cheapest acquisition channel (reactivation) is 100% ignored |
| 9 | **AI note-taker data disconnected** | Captures treatment mentions but doesn't trigger any action | Rich signal, zero execution |
| 10 | **Inconsistent AI deployment** | Some practices have voice AI, some don't; no standard config | Network-wide solution has practice-level gaps |

---

## Key Metrics (Current vs. Desired)

| Metric | Current (estimated) | Desired |
|--------|-------------------|---------|
| Lead response time | Hours to days | < 1 hour (automated) |
| Form fill response rate | ~30-50% ever contacted | 100% contacted within 1 hour |
| Treatment follow-up rate | < 5% of presented plans | 100% automated follow-up |
| Treatment acceptance (with follow-up) | ~30% | 50%+ (3-day follow-up alone adds 50%) |
| Rebooking at checkout | ~60-70% | 90%+ |
| Lapsed patient reactivation | ~0% | Monthly automated outreach |
| Staff time on comms | Unpredictable, often 0 | 20 min/week structured |
| Practices with standardized playbook | ~5% | 100% |
