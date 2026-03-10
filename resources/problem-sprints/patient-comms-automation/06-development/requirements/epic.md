# Epic: The Comms Engine — Autonomous Patient Communications Playbook

**Sprint**: Patient Communications Automation
**Product Manager**: SGA Dental Partners — Technology & Operations
**Date**: 2026-03-10
**Status**: Draft

---

## Epic Description

The Comms Engine is an autonomous AI agent system that executes the full patient communications playbook across SGA's 260+ dental practices. Each practice receives its own agent instance that operates the four-stage communications protocol: lead response (< 60 seconds, 24/7), treatment follow-up (3/7/14/30-day cadences), rebooking recovery (same-day detection), and database reactivation (tiered monthly campaigns).

The system is built on commodity infrastructure (Twilio + NexHealth + Claude LLM) rather than vendor platforms, delivering the same or better outcomes at $47/practice/month vs. $189–299/month for competitors — while the playbook intelligence remains SGA's proprietary IP.

Four architectural layers power the system:

1. **Data Integration Layer** — NexHealth Synchronizer for 20+ PMS systems, phone system webhooks, web form triggers, AI note-taker bridge
2. **Comms Engine Core** — Playbook Engine (rule-based cadences), Agent Orchestrator (260+ instances), Message Generator (LLM-powered), Channel Router, Escalation Manager
3. **Communication Transport** — Twilio SMS/MMS/Voice/Email (HIPAA BAA), Voice AI Engine (LLM-powered conversational voice)
4. **Monitoring & Learning** — Outcome Tracker, Network Learning Engine, Compliance Auditor, Graduation Manager

Human involvement is limited to ~20 minutes/week per practice: a weekly summary delivered to the office manager with only the items requiring human judgment.

---

## Business Objectives

| # | Objective | Current State | Target State |
|---|-----------|---------------|--------------|
| BO-1 | Lead response time | 47-hour industry average (many leads never contacted) | <60 seconds, 24/7 including weekends and after-hours |
| BO-2 | Treatment follow-up coverage | <5% of presented plans receive any follow-up | 100% of presented plans on automated 3/7/14/30-day cadence |
| BO-3 | Rebooking rate | 68% of patients leave with a future appointment | 94% rebooking rate with same-day recovery for the gap |
| BO-4 | Dormant patient reactivation | Zero systematic outreach to lapsed patients | Monthly tiered campaigns across all 260+ practices |
| BO-5 | Cost per practice | $189–299/month (vendor platforms, rented intelligence) | $47/month (owned infrastructure + proprietary IP) |

---

## Success Metrics

| Metric | Baseline | Target | Measurement Method | Timeline |
|--------|----------|--------|-------------------|----------|
| Lead response time | 47-hour average | <60 seconds (p95) | Timestamp: lead event → first contact attempt | Stage 1 (Month 1-2) |
| Lead-to-appointment conversion | 12% | 38% | Funnel: `lead_generated` → `lead_booked` | Stage 1 |
| Treatment acceptance rate | 31% | 47% | PMS: treatment plan created vs. accepted | Stage 2 (Month 2-3) |
| Rebooking rate | 68% | 94% | Appointments completed with next appointment booked within 7 days | Stage 3 (Month 3-4) |
| Reactivation response rate | 0% (no outreach exists) | Tier 1: 18% / Tier 2: 16% / Tier 3: 6% | Campaign sends vs. rebooked appointments | Stage 4 (Month 4-6) |
| Human time per practice per week | Hours (manual comms) | ≤20 minutes | Weekly summary interaction logging (open time + action time) | Stage 2 |
| Practice autonomy adoption | 0% (no automation) | ≥80% of practices in Monitored or Autonomous mode within 90 days of deployment | Graduation model tracking | Stage 2 |
| Cost per practice per month | $189–299 (vendor) | $47 (Twilio + NexHealth + compute) | Infrastructure cost reporting | Stage 1 |

---

## Staged Delivery

| Stage | Timeline | Scope | Practices |
|-------|----------|-------|-----------|
| **Stage 1** | Month 1–2 | Lead response (voice AI inbound, missed call text-back, form fill auto-response, 3-touch nurture sequence) | 5 pilot |
| **Stage 2** | Month 2–3 | Treatment follow-up (NexHealth PMS integration, 4-touch cadence, AI note-taker bridge, high-value 5-touch) | 20 |
| **Stage 3** | Month 3–4 | Rebooking (same-day recovery, hygiene recall sequences, post-appointment detection) | 50 |
| **Stage 4** | Month 4–6 | Reactivation (dormant patient segmentation, tiered outreach, outbound voice AI, network learning engine goes live) | 100+ |
| **Stage 5** | Month 6–12 | Full network + HQ Playbook Manager + monthly optimization loop | 260+ |

---

## Epic 1: Event Integration

> Connect PMS, phone, forms, and AI note-takers to the Comms Engine event stream.

### EI-1: NexHealth Webhook Integration (Stage 1) — P0, 8 pts

**As** a backend engineer,
**I want** a webhook receiver that ingests real-time events from NexHealth (new patient form, appointment created/completed/cancelled, treatment plan created/updated),
**So that** the Comms Engine receives practice PMS data within seconds of events occurring without polling.

**Acceptance Criteria:**

- **Given** a new patient form is submitted at a practice, **When** NexHealth fires the webhook, **Then** the system receives it within 30 seconds, validates the signature, and publishes a `lead_generated` event to the internal event queue.
- **Given** an appointment is completed in the PMS, **When** NexHealth fires the `appointment.completed` webhook, **Then** the system publishes an `appointment_completed` event including: practice_id, appointment_id (hashed), provider_id, service_type, and has_future_appointment boolean.
- **Given** a treatment plan is created with status "presented" and no follow-up appointment, **When** the webhook is received, **Then** a `treatment_plan_presented` event is published including: treatment_type, value_estimate, and patient demographics tier (over_55 / under_55).
- **Given** a NexHealth webhook with an invalid signature, **When** received, **Then** the system returns 401, logs the rejection, and does not process the payload.
- **Given** a webhook delivery failure on NexHealth's side, **When** NexHealth retries, **Then** the system is idempotent — duplicate event_ids do not create duplicate downstream actions.

---

### EI-2: Phone System Event Integration (Stage 1) — P0, 5 pts

**As** a backend engineer,
**I want** to receive inbound call events and missed call events from Twilio (which answers or monitors the practice phone line),
**So that** the Lead Response stage is triggered within seconds of any phone interaction.

**Acceptance Criteria:**

- **Given** an inbound call to a practice phone number, **When** Twilio fires the call webhook, **Then** the system determines: (a) is an agent available to answer? and responds with TwiML to connect the Voice AI, or (b) route to human if in business hours and front desk available.
- **Given** a call that goes unanswered (no pickup within 4 rings), **When** the call ends without answer, **Then** a `missed_call` event is published, triggering missed call text-back within 90 seconds.
- **Given** a call handled by the Voice AI that results in an appointment booking, **When** the call concludes, **Then** a `lead_booked` funnel event is written to the Funnel OS data layer with source: `voice_ai`, handler: `comms_engine`, and estimated_value from the treatment type matched.
- **Given** a missed call to a practice between 10 PM and 7 AM, **When** the text-back is triggered, **Then** it is queued for delivery at 7 AM next morning, not sent immediately.

---

### EI-3: Web Form Webhook Integration (Stage 1) — P0, 3 pts

**As** a backend engineer,
**I want** to receive form submission webhooks from practice websites,
**So that** the Lead Response stage fires within 30 seconds of a "Request a Consultation" form fill.

**Acceptance Criteria:**

- **Given** a form submission from a practice website, **When** the webhook is received, **Then** the system parses: patient first name, interest/service type, preferred contact method, and practice ID; and publishes a `form_fill_lead` event.
- **Given** a `form_fill_lead` event, **When** the Lead Response Orchestrator receives it, **Then** the immediate outreach (SMS + email) fires within 30 seconds, and the 3-touch nurture sequence (4-hour, 24-hour) is scheduled.
- **Given** the form submission at any hour, **When** the immediate outreach fires, **Then** the message is appropriate for time-of-day (evening/weekend forms get a message acknowledging the timing: "We'll have someone follow up first thing tomorrow").

---

### EI-4: AI Note-Taker Bridge (Stage 2) — P1, 5 pts

**As** a backend engineer,
**I want** to ingest transcription output from in-room AI note-takers and extract treatment mentions and patient sentiment,
**So that** treatment follow-up messages can be personalized with context from the actual appointment conversation.

**Acceptance Criteria:**

- **Given** an AI note-taker transcription is available after an appointment, **When** the bridge processes it, **Then** it extracts: treatment types mentioned, any patient objections flagged (cost concern, fear, spouse decision, insurance question), and patient sentiment (positive/neutral/hesitant).
- **Given** extracted sentiment data, **When** the treatment follow-up message is generated, **Then** the Claude prompt includes patient objection context — e.g., "patient expressed cost concern" → message includes financing options; "needs to discuss with spouse" → message frames as joint decision.
- **Given** a transcription that contains no treatment discussions, **When** processed, **Then** no treatment follow-up event is generated (note-taker bridge supplements, does not replace, PMS treatment plan trigger).

---

## Epic 2: Playbook Engine

> The rule engine that defines when, what, and to whom the Comms Engine communicates.

### PE-1: Cadence Definitions (Stage 1) — P0, 8 pts

**As** a product manager,
**I want** a configurable cadence system that defines multi-touch communication sequences per playbook stage,
**So that** the Comms Engine executes the standardized SGA Patient Communication Protocol without hardcoded business logic.

**Acceptance Criteria:**

- **Given** a `treatment_plan_presented` event, **When** the Playbook Engine evaluates it, **Then** it schedules the standard 4-touch cadence: Day 3 (personal check-in), Day 7 (educational content), Day 14 (financing options), Day 30 (final soft touch).
- **Given** a treatment value > $15,000, **When** the Playbook Engine evaluates it, **Then** it schedules the extended 5-touch cadence: same Day 3/7/14 plus Day 21 (video testimonial) and Day 28 (voice call outreach).
- **Given** a cadence is active for a patient who responds and books, **When** the booking event is received, **Then** all remaining touches in that cadence are cancelled within 60 seconds.
- **Given** a cadence definition is updated in the HQ Playbook Manager, **When** the update is published, **Then** existing in-progress workflows are unaffected; only new triggers use the updated cadence.
- **Given** a practice configured with a specialty variant (cosmetic vs. family vs. specialty), **When** cadences are evaluated, **Then** the practice-type-specific template variants are used instead of the network default.

---

### PE-2: Compliance Rules Engine (Stage 1) — P0, 5 pts

**As** a compliance officer,
**I want** the Playbook Engine to enforce HIPAA communication rules, patient opt-out, quiet hours, and consent requirements automatically,
**So that** no communication is sent in violation of HIPAA or TCPA regulations.

**Acceptance Criteria:**

- **Given** a patient has opted out via any channel (texted STOP, called to request no contact, opted out via practice), **When** any outreach is attempted, **Then** the system checks opt-out status before sending and suppresses all outreach — no exceptions.
- **Given** a message is about to be sent to a patient, **When** the compliance check runs, **Then** it verifies: (a) practice has HIPAA BAA with Twilio on file, (b) patient has provided consent (captured at intake), (c) current time is within allowed communication hours (8 AM–8 PM local practice time).
- **Given** a message template, **When** reviewed by the compliance engine, **Then** it flags any PHI in subject lines (diagnosis, treatment type in SMS preview), any language that could be construed as a medical claim, and any missing opt-out instructions in email.
- **Given** the TCPA requirement for A2P SMS, **When** messages are sent, **Then** all practice numbers are registered under 10DLC and messages include opt-out instructions on first contact.

---

### PE-3: A/B Testing Framework (Stage 4) — P1, 5 pts

**As** a product manager,
**I want** to run A/B tests on message templates across random practice cohorts,
**So that** we can identify which message variants produce better response and booking rates network-wide.

**Acceptance Criteria:**

- **Given** an A/B test is configured in the HQ Playbook Manager with two template variants, **When** the test is activated, **Then** practices are randomly assigned to control or treatment cohorts (stratified by specialty and size).
- **Given** a practice assigned to a test cohort, **When** a relevant trigger event fires, **Then** that practice's agent uses the assigned variant for the test duration, not the default template.
- **Given** an A/B test has reached minimum sample size (configurable, default 500 messages per variant), **When** the significance check runs (weekly), **Then** the system calculates response rate, booking rate, and statistical significance (p < 0.05 threshold), and flags the winner for HQ review.
- **Given** a winning variant is confirmed, **When** HQ approves, **Then** the winning template is promoted to the network default and all practices switch to it on the next trigger event.

---

## Epic 3: Lead Response

> Achieve < 60-second first contact for every inbound lead, 24/7.

### LR-1: Voice AI Inbound Answering (Stage 1) — P0, 13 pts

**As** a practice,
**I want** every inbound call to be answered within 4 rings by an AI agent that can book appointments in real time,
**So that** no lead is lost due to an unanswered phone.

**Acceptance Criteria:**

- **Given** an inbound call to a practice's Twilio number, **When** no human answers within 3 rings, **Then** the Voice AI answers with: practice name, provider-specific greeting, and an open-ended prompt.
- **Given** the Voice AI is handling a call, **When** the patient states their interest (e.g., "Invisalign"), **Then** the agent queries NexHealth's real-time schedule for the appropriate provider, presents 2–3 available slots, and confirms the patient's preferred time.
- **Given** a patient confirms an appointment time during the call, **When** the agent processes the booking, **Then** it writes the appointment to the PMS via NexHealth write API within the call, sends a confirmation SMS to the patient within 60 seconds of call end, and logs a `lead_booked` funnel event.
- **Given** the Voice AI encounters a question it cannot answer (e.g., complex insurance question, clinical question), **When** detected, **Then** the agent gracefully transfers: "Let me connect you with someone who can answer that directly" and routes to the practice's human fallback number.
- **Given** a Voice AI call that occurs after business hours, **When** the patient calls, **Then** the agent answers, collects name/interest/callback number, schedules an agent-initiated callback during the next business day morning, and confirms via text.
- **Given** a Voice AI conversation, **When** it concludes, **Then** the full call summary (duration, patient interest, outcome, booked appointment if any) is logged to the communication_events table with no PHI in the log body.

---

### LR-2: Missed Call Text-Back (Stage 1) — P0, 5 pts

**As** a practice,
**I want** missed inbound calls to receive an automatic text-back within 90 seconds,
**So that** potential patients who couldn't get through are re-engaged before they call a competitor.

**Acceptance Criteria:**

- **Given** a missed inbound call (no answer, no voicemail left), **When** detected, **Then** a text-back is sent within 90 seconds: "Hi! We missed your call at [Practice Name]. We'd love to help — what can we assist you with? Reply here or call us back."
- **Given** the missed call occurred between 10 PM and 7 AM, **When** the text-back fires, **Then** it is queued until 7 AM local time with a modified message: "We saw your missed call last night — we're open now and happy to help!"
- **Given** a patient responds to the text-back, **When** the response is received, **Then** the Lead Response Orchestrator handles it: if they want to book, it offers scheduling options; if they have a question, it answers or routes to human; it logs the response as `lead_connected`.
- **Given** no response to the text-back within 4 hours, **When** the timer fires, **Then** a second text goes out: "Still here if you need us! [3 available appointment times]." If no response in 24 hours, the lead sequence ends.

---

### LR-3: Form Fill Auto-Response (Stage 1) — P0, 5 pts

**As** a practice,
**I want** web form submissions to receive a personalized response within 30 seconds at any hour,
**So that** the 391% conversion advantage of sub-1-minute response is captured for every form lead.

**Acceptance Criteria:**

- **Given** a form fill webhook received, **When** the Lead Response Orchestrator fires, **Then** an SMS and email are sent within 30 seconds: SMS with 2 specific available times for the relevant provider; email with provider bio, patient testimonial relevant to the inquiry, and the same scheduling options.
- **Given** the form fill includes a specific treatment interest (e.g., "dental implants"), **When** the message is generated, **Then** the Claude message generator matches the interest to the correct provider specialty in NexHealth, and the message references that provider by name.
- **Given** a patient replies to the scheduling text and selects a time, **When** the response is received, **Then** the appointment is booked in the PMS, a confirmation is sent, and the lead sequence ends (no further nurture touches).
- **Given** no response to the immediate outreach in 4 hours, **When** the timer fires, **Then** touch 2 sends: "Just checking in — still interested in [treatment type]? Happy to answer any questions." If no response in 24 hours, touch 3 sends a direct booking link and ends the sequence.

---

## Epic 4: Treatment Follow-Up

> Ensure 100% of presented treatment plans receive systematic, personalized follow-up.

### TF-1: PMS Treatment Plan Trigger (Stage 2) — P0, 5 pts

**As** a backend engineer,
**I want** treatment plan creation events from NexHealth to reliably trigger the follow-up sequence,
**So that** no treatment plan falls through the cracks.

**Acceptance Criteria:**

- **Given** a treatment plan is created in the PMS with status "presented" and no follow-up appointment, **When** NexHealth fires the webhook, **Then** the system checks: (a) no existing active treatment follow-up sequence for this patient/plan, (b) plan value is captured, (c) patient is not opted out, then publishes a `treatment_plan_presented` event.
- **Given** a treatment plan where an appointment IS already scheduled, **When** the event is received, **Then** no follow-up sequence is created (patient already booked — don't over-communicate).
- **Given** a treatment plan that is updated (e.g., value increased, treatment changed), **When** the webhook fires, **Then** the active sequence is updated: message templates are regenerated with the new treatment context on the next scheduled touch.

---

### TF-2: Standard 4-Touch Follow-Up Cadence (Stage 2) — P0, 13 pts

**As** a practice,
**I want** every presented treatment plan to receive 4 automated follow-up touches over 30 days,
**So that** treatment acceptance rates improve without the treatment coordinator having to make uncomfortable follow-up calls.

**Acceptance Criteria:**

- **Given** a `treatment_plan_presented` event, **When** the Playbook Engine creates the follow-up sequence, **Then** 4 Temporal workflow activities are scheduled: Day 3 (personal check-in), Day 7 (educational content), Day 14 (financing options), Day 30 (final soft touch).
- **Given** Touch 1 (Day 3), **When** the message is generated, **Then** Claude generates a warm, personal check-in that: uses the patient's first name, mentions the treatment by specific name (not "your recommended procedure"), references the provider by name, and includes zero sales pressure.
- **Given** Touch 2 (Day 7), **When** the message generates, **Then** it includes educational content relevant to the specific treatment (e.g., for veneers: what the process is like, recovery time, typical patient experience) — pulled from a pre-approved content library, personalized by Claude.
- **Given** Touch 3 (Day 14), **When** the message generates, **Then** it leads with financing options: specific monthly payment estimates (calculated from plan value), financing partner information, and an invitation to discuss.
- **Given** Touch 4 (Day 30), **When** the message generates, **Then** it is a brief, warm final touch — no pressure, just keeping the door open: "Whenever you're ready, we're here. Just reply or call and we'll get you scheduled."
- **Given** any touch response where the patient books, **When** booking is detected, **Then** all remaining sequence touches are cancelled within 60 seconds, and a `treatment_accepted` funnel event is logged.
- **Given** a patient who responds negatively (e.g., "not interested", "going elsewhere"), **When** the response sentiment is detected by Claude, **Then** the sequence ends gracefully with a warm closing message and an opt-out confirmation.

---

### TF-3: High-Value 5-Touch Extension (Stage 2) — P1, 5 pts

**As** a practice,
**I want** treatment plans valued over $15,000 to receive 2 additional outreach touches including a voice call,
**So that** the highest-value cases get the highest-effort follow-up, maximizing the Comms Engine's revenue impact.

**Acceptance Criteria:**

- **Given** a treatment plan with value_estimate ≥ $15,000, **When** the sequence is created, **Then** the standard 4 touches are extended with: Touch 5 (Day 21) — video testimonial link; Touch 6 (Day 28) — outbound voice call from Voice AI.
- **Given** Touch 5 (Day 21), **When** the message generates, **Then** it includes a curated video testimonial link from an SGA patient with the same or similar procedure (testimonial library, filtered by treatment type and practice specialty).
- **Given** Touch 6 (Day 28), **When** the Temporal workflow fires the outbound call, **Then** the Voice AI calls the patient: introduces as calling from the practice, references the treatment (not as a sales call but as a personal follow-up), handles questions, and offers to book.
- **Given** an outbound Voice AI call where the patient is unavailable, **When** the call results in voicemail, **Then** a brief, warm voicemail is left (max 20 seconds), and an SMS follow-up is sent within 5 minutes: "Tried to reach you — happy to chat when it's convenient. Just reply here."

---

## Epic 5: Rebooking & Reactivation

> Ensure continuity of care and recover dormant patient revenue.

### RR-1: Same-Day Rebooking Detection (Stage 3) — P0, 8 pts

**As** a practice,
**I want** patients who complete an appointment without booking their next one to receive automated rebooking outreach within 30 minutes,
**So that** the same-day recovery window (highest conversion, patient still thinking about their visit) is never missed.

**Acceptance Criteria:**

- **Given** an appointment is marked completed in the PMS, **When** the NexHealth webhook fires, **Then** the system checks: does this patient have a future appointment within the next 7 months? If not, and service type requires follow-up (hygiene, recall), trigger rebooking sequence.
- **Given** a patient with no future appointment after a completed visit, **When** 30 minutes have elapsed, **Then** an SMS is sent: patient's first name, provider name, 3 specific available times for the same provider (same time slot preference where possible), and a direct booking link.
- **Given** the patient taps a suggested time or the booking link, **When** the booking is completed, **Then** the system writes the appointment to PMS via NexHealth, sends a confirmation, and logs `rebooking_recovered`.
- **Given** no response to the first rebooking touch, **When** Day 1 timer fires, **Then** a second nudge: "Just a quick reminder — keeping your 6-month schedule helps catch small things early!" with the same booking link.
- **Given** no response after Day 3, **When** the sequence ends, **Then** the patient is logged for the next monthly recall sequence (no immediate further contact).

---

### RR-2: Hygiene Recall Sequences (Stage 3) — P1, 5 pts

**As** a practice,
**I want** patients approaching their 6-month recall window to receive proactive scheduling outreach,
**So that** hygiene appointment continuity is maintained across the practice without staff manually calling recall lists.

**Acceptance Criteria:**

- **Given** a patient whose last hygiene appointment was 5 months ago, **When** the recall check runs (daily batch), **Then** a 3-touch recall sequence starts: Month 5 (save-your-spot text), Month 5.5 (reminder with available times), Month 6 (final recall with urgency).
- **Given** a recall sequence touch, **When** the patient books, **Then** the remaining recall touches are cancelled.
- **Given** a practice with providers who have different hygienists assigned, **When** the recall message is sent, **Then** it references the patient's specific hygienist by name for continuity.

---

### RR-3: Dormant Patient Segmentation & Reactivation (Stage 4) — P0, 13 pts

**As** a practice,
**I want** patients who haven't been seen in 12+ months to be identified, segmented by value tier, and contacted with tiered outreach,
**So that** the practice's dormant database is systematically converted to active revenue.

**Acceptance Criteria:**

- **Given** the monthly reactivation scan runs on the 1st of each month, **When** it evaluates the patient database via NexHealth, **Then** it identifies all patients with last_seen_date > 12 months ago and segments them: Tier 1 (active treatment plan when lapsed or high production history), Tier 2 (regular hygiene patient), Tier 3 (infrequent visitor).
- **Given** Tier 1 patients (active treatment plan), **When** the reactivation campaign launches, **Then** they receive an outbound Voice AI call sequence: call on Day 1, text follow-up on Day 3 if no response.
- **Given** Tier 2 patients, **When** the campaign launches, **Then** they receive a personalized text sequence: Day 1 (personal check-in with provider name), Day 5 (3 available times), Day 12 (final text with direct booking link).
- **Given** Tier 3 patients, **When** the campaign launches, **Then** they receive an email + text: email on Day 1 (welcome back messaging with practice news), text follow-up on Day 4.
- **Given** pacing requirements, **When** the Tier 1 voice campaign is active, **Then** calls are paced at maximum 5 per day per practice to avoid overwhelming the schedule with reactivations.
- **Given** a reactivation campaign that results in a booking, **When** the appointment is confirmed, **Then** the system logs `patient_reactivated` with tier, campaign_id, and estimated_production_value.
- **Given** a patient whose reactivation text bounces (number changed, wrong number response), **When** detected, **Then** the patient is flagged for manual record review in the next weekly summary.

---

## Epic 6: Graduated Autonomy & Human Interface

> Build trust through a supervised onboarding protocol that limits human work to 20 minutes/week.

### GA-1: Autonomy Level Management (Stage 1) — P0, 8 pts

**As** a practice operations lead,
**I want** each communication type to have its own autonomy level (Supervised / Monitored / Autonomous) that advances independently based on demonstrated quality,
**So that** practices trust the Comms Engine before giving it full autonomy.

**Acceptance Criteria:**

- **Given** a newly onboarded practice, **When** the agent instance is created, **Then** autonomy levels are initialized: lead_response = monitored, treatment_followup = supervised, rebooking = monitored, reactivation = supervised.
- **Given** a communication type in Supervised mode, **When** a message is ready to send, **Then** it is held in `pending_approvals` and included in the daily digest; it auto-sends if not reviewed within 2 hours.
- **Given** a communication type in Monitored mode, **When** a message is sent, **Then** it auto-sends without pre-approval, but the office manager sees it in the daily digest with a "sent" status (no approval required, just visibility).
- **Given** a communication type in Autonomous mode, **When** messages are sent, **Then** they are not included in the daily digest; only unusual outcomes (e.g., patient complaint response, compliance flag) surface in the weekly summary.
- **Given** an HQ administrator, **When** they use the Playbook Manager's practice override, **Then** they can force any practice to Supervised mode for any communication type, regardless of graduation status.

---

### GA-2: Graduation Criteria & Tracking (Stage 2) — P0, 5 pts

**As** a product manager,
**I want** the system to automatically track graduation eligibility and recommend advancement to Autonomous mode,
**So that** practices don't stay in supervised mode longer than necessary.

**Acceptance Criteria:**

- **Given** a communication type in Supervised mode, **When** the graduation evaluator runs (weekly), **Then** it calculates over the trailing 30 days: edit_rate (edits / total approvals), approval_rate (approved / total), patient_response_rate (responses / total sent), and compliance_flags count.
- **Given** graduation criteria are met (<5% edit rate, >90% approval rate, >70% patient response rate, 0 compliance flags over 30 days), **When** the evaluator confirms eligibility, **Then** the system generates a graduation report and delivers it to the office manager as part of the next weekly summary.
- **Given** the office manager receives a graduation recommendation, **When** they approve the graduation, **Then** the autonomy level advances (Supervised → Monitored, or Monitored → Autonomous).
- **Given** a practice that fails graduation criteria, **When** the evaluator runs, **Then** no recommendation is made; the system continues in current mode and extends the evaluation window by 30 days.

---

### GA-3: Weekly Summary Generator (Stage 1) — P0, 8 pts

**As** an office manager,
**I want** to receive a structured weekly summary every Monday at 7 AM that covers everything the engine did, what needs my attention, and what's coming,
**So that** my entire interaction with the Comms Engine takes ≤20 minutes per week.

**Acceptance Criteria:**

- **Given** it is Monday at 7 AM practice-local time, **When** the weekly summary job runs, **Then** the office manager receives an SMS with a formatted 3-section summary: Section 1 (Engine Activity — messages sent, actions taken, results, estimated revenue influenced), Section 2 (Items Needing Attention — specific, actionable items with context), Section 3 (Coming This Week — preview of upcoming automated touches).
- **Given** Section 2 (Items Needing Attention), **When** there are no items requiring human judgment, **Then** Section 2 reads: "No action needed this week. Everything's running."
- **Given** a patient response that requires human judgment (e.g., complex insurance question, clinical concern, records request), **When** identified, **Then** it is included in Section 2 with: patient first name, context sentence, and the action required (e.g., "Call Robert — he has questions about insurance coverage for his crown. Number: [xxx]").
- **Given** the summary is delivered, **When** the office manager reads it, **Then** the total action time for Section 2 items is ≤20 minutes as validated by session timing analytics.
- **Given** a practice in fully Autonomous mode with zero escalations, **When** the summary generates, **Then** it is a brief 2-section report: Engine Activity only + Coming This Week (no Attention section needed).

---

## Epic 7: Network Learning Engine

> Turn 260 practices' daily communications into a continuously improving playbook.

### NL-1: Communication Outcome Tracking (Stage 2) — P0, 5 pts

**As** a data engineer,
**I want** every communication event to be linked to a measurable outcome,
**So that** the Network Learning Engine has the ground truth data it needs to identify what works.

**Acceptance Criteria:**

- **Given** a message is sent to a patient, **When** a response or action occurs within the attribution window (7 days for treatment follow-up, 48 hours for lead response), **Then** the outcome is linked to the message event: outcome type (booked, responded_no_action, no_response, opted_out), time_to_outcome, and revenue_attributed (if appointment booked).
- **Given** a sequence results in a treatment plan being accepted, **When** the acceptance is detected in the PMS, **Then** the `treatment_accepted` event is attributed back to the specific message touch that preceded it (last-touch attribution), with the full sequence history logged.
- **Given** a message receives no response, **When** the attribution window closes, **Then** the `no_response` outcome is logged with: template_id, touch_number, practice_type, patient_demographics_tier, day_of_week, time_of_day — enabling the learning engine to identify patterns.

---

### NL-2: Monthly Playbook Optimization (Stage 4) — P1, 8 pts

**As** a product manager,
**I want** the Network Learning Engine to analyze outcome data monthly and update playbook defaults with winning patterns,
**So that** the Comms Engine's performance improves automatically over time without manual playbook curation.

**Acceptance Criteria:**

- **Given** the monthly optimization cycle runs on the last day of the month, **When** the analysis completes, **Then** it evaluates: response rate by template × touch_number × day_of_week × practice_type × demographics_tier, and identifies statistically significant patterns (minimum 500 sends per variant, p < 0.05).
- **Given** a significant pattern is identified (e.g., "Saturday 9–11 AM sends get 2.1x response rate"), **When** it is confirmed across ≥ 3 practice types and ≥ 100 practices, **Then** it is promoted as a new playbook default in the relevant cadence.
- **Given** a playbook update is ready to deploy, **When** it is published, **Then** HQ receives the Playbook Evolution Report showing: what changed, supporting data, projected impact on acceptance/response rates, and a 7-day rollback window.
- **Given** a practice that has opted out of specific playbook updates (guardrails), **When** a new default is deployed, **Then** that practice's cadence is unaffected — opt-outs are respected.

---

## Epic 8: HQ Playbook Manager

> Web interface for HQ to design, test, and deploy playbook updates network-wide.

### HQ-1: Visual Cadence Designer (Stage 5) — P1, 8 pts

**As** an HQ operations lead,
**I want** a visual cadence designer that lets me configure touch points, delays, channels, and branching without writing code,
**So that** playbook updates can be made by non-engineers in response to network learnings.

**Acceptance Criteria:**

- **Given** the HQ Playbook Manager is open, **When** I navigate to Cadence Designer, **Then** I see a visual timeline view with draggable touch points, configurable delay inputs (e.g., "Day 3"), channel selectors (SMS / email / voice), and a template selector per touch.
- **Given** a cadence change, **When** I save and publish, **Then** I am prompted to: (a) apply to all practices immediately, (b) deploy to a test cohort first, or (c) schedule for next Monday rollout.
- **Given** a cadence that includes branching (e.g., "if patient responds positively, skip to touch 5; if no response, send touch 3"), **When** I configure it, **Then** the visual designer shows branching paths with labeled conditions.

---

### HQ-2: Template Editor (Stage 5) — P1, 5 pts

**As** an HQ content strategist,
**I want** a template editor with variable substitution, tone preview, and Claude-powered suggestions,
**So that** message templates can be tested and refined before network deployment.

**Acceptance Criteria:**

- **Given** the template editor is open, **When** I write a template, **Then** I can insert variables from a picker ({{patient_first_name}}, {{treatment_type}}, {{provider_name}}, {{practice_name}}, {{financing_monthly_payment}}) and see a live preview with sample values.
- **Given** a template draft, **When** I click "Preview with Claude," **Then** the system generates 3 sample messages using the template with representative patient/practice contexts, showing what an actual message would look like.
- **Given** a template that contains a potential compliance issue (PHI in subject line, missing opt-out, medical claim language), **When** the compliance checker runs, **Then** it flags the issue with a specific explanation and suggested correction before the template can be published.

---

## Dependency Map

```
TIER 0 — Foundations (no dependencies, start immediately)
├── EI-2   Phone System Event Integration (Twilio setup)
├── PE-2   Compliance Rules Engine
└── GA-1   Autonomy Level Management (data model)

TIER 1 — Core Integration (depends on event model)
├── EI-1   NexHealth Webhook Integration
├── EI-3   Web Form Webhook Integration
└── GA-3   Weekly Summary Generator (can run with minimal data)

TIER 2 — Lead Response (depends on event integration + compliance)
├── EI-1 + EI-2 + PE-2 ──→ LR-1  Voice AI Inbound Answering
├── EI-2 + PE-2 ──────────→ LR-2  Missed Call Text-Back
└── EI-3 + PE-2 ──────────→ LR-3  Form Fill Auto-Response

                ┌──── STAGE 1 MILESTONE: Lead Response Live (5 Pilot Practices) ────┐

TIER 3 — Playbook Engine + Treatment Follow-Up (depends on NexHealth + lead stage)
├── EI-1  ──→ TF-1   PMS Treatment Plan Trigger
├── PE-1  ──→ TF-2   Standard 4-Touch Cadence (depends on Playbook Engine)
├── TF-2  ──→ TF-3   High-Value 5-Touch Extension
├── EI-4  ──→ TF-2*  AI Note-Taker enrichment (optional enhancement)
└── GA-2  ──→ GA-2   Graduation Criteria (needs 30 days of data from Stage 1)

                ┌──── STAGE 2 MILESTONE: Treatment Follow-Up (20 Practices) ────┐

TIER 4 — Rebooking (depends on NexHealth + treatment stage)
├── EI-1  ──→ RR-1   Same-Day Rebooking Detection
└── EI-1  ──→ RR-2   Hygiene Recall Sequences

                ┌──── STAGE 3 MILESTONE: Rebooking Live (50 Practices) ────┐

TIER 5 — Reactivation + Network Learning (depends on outcome data from Stages 1-3)
├── NL-1  ──→ RR-3   Dormant Patient Reactivation (needs outcome data for Tier segmentation)
├── NL-1  ──→ NL-2   Monthly Playbook Optimization
└── PE-3  ──→ PE-3   A/B Testing Framework (needs sufficient volume)

                ┌──── STAGE 4 MILESTONE: Reactivation + Network Learning (100+ Practices) ────┐

TIER 6 — HQ Management Tools (depends on stable playbook)
├── PE-3  ──→ HQ-1   Visual Cadence Designer
└── HQ-1  ──→ HQ-2   Template Editor

                ┌──── STAGE 5 MILESTONE: Full Network + Playbook Manager (260+ Practices) ────┐
```

---

## Story Point Summary

| Epic | Stories | Total Points | P0 Points | P1 Points | P2 Points |
|------|:-------:|:------------:|:---------:|:---------:|:---------:|
| Epic 1: Event Integration | 4 | 21 | 16 | 5 | 0 |
| Epic 2: Playbook Engine | 3 | 18 | 13 | 5 | 0 |
| Epic 3: Lead Response | 3 | 23 | 23 | 0 | 0 |
| Epic 4: Treatment Follow-Up | 3 | 23 | 18 | 5 | 0 |
| Epic 5: Rebooking & Reactivation | 3 | 26 | 21 | 5 | 0 |
| Epic 6: Graduated Autonomy | 3 | 21 | 21 | 0 | 0 |
| Epic 7: Network Learning | 2 | 13 | 5 | 8 | 0 |
| Epic 8: HQ Playbook Manager | 2 | 13 | 0 | 13 | 0 |
| **Total** | **23** | **158** | **117** | **41** | **0** |

**P0 stories (117 pts)** represent the minimum viable Comms Engine through Stage 4 (lead response + treatment follow-up + rebooking + reactivation + graduated autonomy).
**P0 + P1 stories (158 pts)** deliver the full product through Stage 5 including network learning and HQ Playbook Manager.

---

## Open Questions & Technical Spikes

| # | Question | Spike Required | Owner | Stage |
|---|----------|---------------|-------|-------|
| 1 | What is NexHealth's actual webhook reliability and latency for the 15–20 PMS systems in SGA's network? Which systems support real-time events vs. polling? | Yes — 2-week NexHealth API audit with sample practices on each PMS | Backend Eng | Stage 1 |
| 2 | Can the Twilio Voice + Claude LLM integration achieve <500ms turn latency for natural dental conversation? (Streaming TTS required) | Yes — latency prototype: Twilio Media Streams → Claude streaming → TTS back to call | Backend Eng | Stage 1 |
| 3 | What is the minimum sample size to detect statistically significant A/B test winners with SGA's daily message volume? (Need to validate the 500-message threshold) | Yes — power analysis using projected Stage 1 volume data | Data Eng | Stage 2 |
| 4 | Will 10DLC registration for 260+ practice phone numbers cause throughput bottlenecks? What is Twilio's per-number daily SMS limit? | Yes — Twilio A2P compliance review; evaluate Toll-Free SMS as an alternative for high-volume practices | Backend Eng | Stage 1 |
| 5 | How does the Comms Engine share the practice-facing interface with the Practice Concierge without coupling their release cycles? | Architecture review — module boundaries, feature flags, shared component library, separate deployment pipelines | Tech Lead | Stage 2 |
| 6 | Can the AI note-taker bridge integrate with the specific note-taker vendors deployed in SGA practices? (Verify which vendors: Dentrix Voice, Bola AI, etc.) | Vendor audit — identify which AI note-takers are in use; evaluate API availability vs. transcript export | Backend Eng | Stage 2 |

---

*This epic is derived from the Patient Communications Automation sprint (2026-03-10). Source artifacts: `01-frame/`, `03-decide/`, `04-architect/architecture.md`.*
