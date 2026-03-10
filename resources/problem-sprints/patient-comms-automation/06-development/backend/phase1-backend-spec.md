# Phase 1 Backend Specification — The Comms Engine

**Role**: Backend Engineer
**Sprint**: Patient Communications Automation
**Phase**: Stage 1 — Lead Response (5 Pilot Practices)
**Date**: 2026-03-10
**Source Artifacts**: `04-architect/architecture.md`, Tech Lead Implementation Plan

---

## Phase 1 Scope

Stage 1 delivers Lead Response for 5 pilot practices:
1. **Voice AI inbound answering** — answer calls within 4 rings, book appointments via NexHealth
2. **Missed call text-back** — SMS within 90 seconds of a missed inbound call
3. **Form fill auto-response** — SMS + email within 30 seconds of a web form submission
4. **Lead nurture sequence** — 3-touch cadence (immediate, +4h, +24h) if patient doesn't respond

This spec covers the backend services that implement Phase 1 only. Treatment follow-up, rebooking, and reactivation are Stage 2–4.

---

## Service Architecture

```
                        ┌─────────────────────────────────────────────┐
                        │                   AWS ALB                    │
                        └──────────────┬──────────────────────────────┘
                                       │
                        ┌──────────────▼──────────────┐
                        │        comms-api             │
                        │       (FastAPI app)          │
                        │                              │
                        │  ┌──────────┐ ┌──────────┐  │
                        │  │ /webhooks│ │  /api    │  │
                        │  └────┬─────┘ └────┬─────┘  │
                        └───────┼─────────────┼────────┘
                                │             │
                   ┌────────────▼─┐       ┌───▼────────────────┐
                   │  Kafka MSK   │       │   PostgreSQL RDS    │
                   │  Topics:     │       │   (operational DB)  │
                   │  nexhealth-  │       └────────────────────┘
                   │  events      │
                   │  twilio-     │       ┌────────────────────┐
                   │  events      │       │   voice-ai-service  │
                   │  form-fill   ├──────►│  (FastAPI + Claude) │
                   └──────┬───────┘       └────────────────────┘
                          │
                   ┌──────▼───────┐
                   │  Temporal    │
                   │  Workers     │
                   │  (workflow   │
                   │  execution)  │
                   └──────┬───────┘
                          │
           ┌──────────────▼──────────────┐
           │      Message Services        │
           ├─────────────────────────────┤
           │  Message Generator (Claude) │
           │  Communication Dispatcher   │
           │  (Twilio SMS + Email)        │
           └─────────────────────────────┘
```

---

## Service 1: Event Ingestion (comms-api webhook handlers)

Receives events from NexHealth, Twilio, and web forms. Validates, normalizes, and publishes to Kafka. Must respond within 3 seconds (Twilio timeout) or 30 seconds (NexHealth timeout).

### POST /webhooks/nexhealth

Receives all NexHealth webhooks. Dispatches based on `event_type` field.

**Request body** (NexHealth standard format):
```json
{
  "event_type": "appointment.completed",
  "location_id": "loc_abc123",
  "data": {
    "appointment": {
      "id": "appt_xyz",
      "patient_id": "pat_123",
      "provider_id": "prov_456",
      "service_type": "hygiene_cleaning",
      "start_time": "2026-03-10T10:00:00Z",
      "status": "completed"
    }
  },
  "timestamp": "2026-03-10T10:45:00Z"
}
```

**Handler logic**:
```python
@router.post("/webhooks/nexhealth")
async def nexhealth_webhook(request: Request, db: AsyncSession = Depends(get_db)):
    # 1. Validate HMAC signature (NexHealth-Signature header)
    validate_nexhealth_signature(request)

    payload = await request.json()
    event_type = payload["event_type"]
    location_id = payload["location_id"]

    # 2. Look up practice by NexHealth location_id
    practice = await get_practice_by_location_id(db, location_id)
    if not practice:
        return {"status": "ignored", "reason": "unknown_location"}

    # 3. Idempotency check (NexHealth retries on timeout)
    event_id = payload.get("event_id")
    if event_id and await event_already_processed(db, event_id):
        return {"status": "duplicate", "event_id": event_id}

    # 4. Dispatch to appropriate Kafka topic
    if event_type in ("patient.form_submitted",):
        await kafka_producer.send("nexhealth-events", key=location_id,
                                   value={"type": "form_fill", "practice_id": str(practice.id), "payload": payload})
    elif event_type == "appointment.completed":
        await kafka_producer.send("nexhealth-events", key=location_id,
                                   value={"type": "appointment_completed", "practice_id": str(practice.id), "payload": payload})
    # ... other event types

    return {"status": "accepted"}
```

**Response**: `200 {"status": "accepted"}` — always return 200 immediately; processing is async.

**HIPAA note**: The raw NexHealth payload (which may contain PHI) is held in Kafka for max 24 hours and never persisted to the operational database. Only hashed identifiers are stored in PostgreSQL.

---

### POST /webhooks/twilio/voice

Twilio calls this endpoint when an inbound call arrives. Must respond with TwiML within 5 seconds to avoid Twilio timeout.

**Request** (Twilio standard form-encoded POST):
```
CallSid=CA1234...
From=+16015550100
To=+16015559876
CallStatus=ringing
```

**Handler logic**:
```python
@router.post("/webhooks/twilio/voice")
async def twilio_voice_webhook(
    CallSid: str = Form(...),
    From: str = Form(...),
    To: str = Form(...),
    db: AsyncSession = Depends(get_db)
):
    # 1. Validate Twilio signature
    validate_twilio_signature(request)

    # 2. Look up practice by Twilio phone number (To)
    practice = await get_practice_by_phone(db, To)
    if not practice:
        return TwiML("<Response><Hangup/></Response>")

    # 3. Check quiet hours (practice timezone)
    if is_quiet_hours(practice):
        # After hours: Voice AI still answers, collects callback info
        twiml = build_after_hours_twiml(practice)
        return Response(content=twiml, media_type="application/xml")

    # 4. Start Voice AI session
    session_id = str(uuid4())
    await kafka_producer.send("twilio-events", value={
        "type": "inbound_call",
        "call_sid": CallSid,
        "from_number_hash": sha256(From),
        "practice_id": str(practice.id),
        "session_id": session_id
    })

    # 5. Return TwiML to connect to Voice AI service via Media Streams
    twiml = f"""
    <Response>
        <Connect>
            <Stream url="wss://comms.sga.com/voice-ai/{session_id}">
                <Parameter name="practice_id" value="{practice.id}"/>
                <Parameter name="call_sid" value="{CallSid}"/>
            </Stream>
        </Connect>
    </Response>
    """
    return Response(content=twiml, media_type="application/xml")
```

---

### POST /webhooks/twilio/voice/status

Receives call completion events. Handles missed calls (CallStatus = no-answer, busy).

```python
@router.post("/webhooks/twilio/voice/status")
async def twilio_voice_status(
    CallSid: str = Form(...),
    CallStatus: str = Form(...),  # completed | no-answer | busy | failed
    To: str = Form(...),
    From: str = Form(...),
    db: AsyncSession = Depends(get_db)
):
    validate_twilio_signature(request)
    practice = await get_practice_by_phone(db, To)

    if CallStatus in ("no-answer", "busy"):
        # Missed call — trigger text-back workflow
        await kafka_producer.send("twilio-events", value={
            "type": "missed_call",
            "call_sid": CallSid,
            "from_number_hash": sha256(From),
            "practice_id": str(practice.id),
            "occurred_at": datetime.utcnow().isoformat()
        })

    # Log to communication_events
    await log_communication_event(db, {
        "practice_id": practice.id,
        "event_type": "call_" + CallStatus.replace("-", "_"),
        "channel": "voice",
        "direction": "inbound",
        "twilio_sid": CallSid,
        "stage": "lead_response"
    })

    return {"status": "ok"}
```

---

### POST /webhooks/twilio/message/inbound

Receives patient replies to outbound SMS. Handles STOP, YES, and free-text responses.

```python
@router.post("/webhooks/twilio/message/inbound")
async def twilio_inbound_sms(
    MessageSid: str = Form(...),
    From: str = Form(...),
    To: str = Form(...),
    Body: str = Form(...),
    db: AsyncSession = Depends(get_db)
):
    validate_twilio_signature(request)
    practice = await get_practice_by_phone(db, To)
    patient = await get_patient_by_phone_hash(db, sha256(From), practice.id)

    body_upper = Body.strip().upper()

    # TCPA opt-out handling (MUST process before anything else)
    if body_upper in ("STOP", "STOPALL", "UNSUBSCRIBE", "CANCEL", "END", "QUIT"):
        await process_opt_out(db, patient, channel="sms")
        return Response(content="<Response/>", media_type="application/xml")

    # Route response to active sequence
    await kafka_producer.send("twilio-events", value={
        "type": "sms_reply",
        "message_sid": MessageSid,
        "from_hash": sha256(From),
        "practice_id": str(practice.id),
        "patient_id": str(patient.id) if patient else None,
        "body_sentiment": await classify_response_intent(Body),  # book|positive|negative|question
        "received_at": datetime.utcnow().isoformat()
    })

    return Response(content="<Response/>", media_type="application/xml")
```

---

### POST /webhooks/forms/{practice_id}

Receives web form submissions from practice websites.

**Request body**:
```json
{
  "patient_first_name": "David",
  "patient_last_name": "S",
  "phone": "+14805550100",
  "email": "david@example.com",
  "interest": "dental_implants",
  "message": "Interested in full arch",
  "preferred_contact": "sms"
}
```

**Handler logic**:
```python
@router.post("/webhooks/forms/{practice_id}")
async def form_submission(
    practice_id: UUID,
    body: FormSubmissionPayload,
    db: AsyncSession = Depends(get_db)
):
    practice = await get_practice(db, practice_id)

    # Hash PHI immediately — never store plain text
    phone_hash = sha256(normalize_phone(body.phone))
    email_hash = sha256(normalize_email(body.email)) if body.email else None

    # Upsert patient (create if first contact)
    patient = await upsert_patient(db, {
        "practice_id": practice_id,
        "phone_hash": phone_hash,
        "email_hash": email_hash,
        "preferred_channel": body.preferred_contact or "sms"
    })

    # Check opt-out before proceeding
    if patient.opt_out_at:
        return {"status": "opted_out"}

    # Publish to Kafka — include PHI (first name, phone) only in message payload
    # PHI is NOT persisted to DB — only used for immediate message generation
    await kafka_producer.send("form-fill-events", value={
        "type": "form_fill",
        "practice_id": str(practice_id),
        "patient_id": str(patient.id),
        "patient_first_name": body.patient_first_name,  # PHI: only in Kafka, consumed once
        "phone": body.phone,
        "email": body.email,
        "interest": body.interest,
        "received_at": datetime.utcnow().isoformat()
    })

    return {"status": "received"}
```

---

## Service 2: Voice AI (voice-ai-service)

Handles the real-time bidirectional audio stream between Twilio and Claude. This service runs as a separate ECS service (CPU-intensive, latency-sensitive).

### WebSocket Handler: `/voice-ai/{session_id}`

```python
@app.websocket("/voice-ai/{session_id}")
async def voice_ai_websocket(websocket: WebSocket, session_id: str):
    await websocket.accept()

    # Load practice context from session_id
    context = await load_voice_session_context(session_id)
    practice = context["practice"]

    # Initialize Claude conversation with practice system prompt
    claude_messages = []
    system_prompt = build_voice_ai_system_prompt(practice)

    # Initialize TTS stream (Twilio expects audio back in G.711 ulaw format)
    tts_client = ElevenLabsStreamingClient()  # or Twilio's built-in TTS

    async with anthropic.AsyncAnthropic() as claude:
        # Initial greeting
        greeting = f"Thanks for calling {practice.name}, this is the scheduling assistant. How can I help you today?"
        await stream_tts_to_twilio(websocket, tts_client, greeting)

        # Audio processing loop
        async for twilio_message in websocket.iter_json():
            if twilio_message["event"] == "media":
                # Decode Twilio audio (G.711 ulaw → PCM)
                audio_chunk = base64.b64decode(twilio_message["media"]["payload"])

                # Send to STT (speech-to-text) — accumulate until end of utterance
                transcript = await stt_client.process_chunk(audio_chunk)
                if not transcript:
                    continue

                # Add to Claude conversation
                claude_messages.append({"role": "user", "content": transcript})

                # Check if scheduling intent detected — query NexHealth for real-time availability
                intent = await detect_scheduling_intent(transcript)
                if intent.wants_appointment:
                    availabilities = await nexhealth_client.get_availabilities(
                        location_id=practice.nex_health_location_id,
                        provider_id=intent.provider_id,
                        service_type=intent.service_type
                    )
                    # Inject availability into Claude context
                    availability_context = format_availabilities_for_claude(availabilities)
                    system_context = system_prompt + "\n\nCURRENT AVAILABILITY:\n" + availability_context

                # Get Claude's response (streaming)
                response_text = ""
                async with claude.messages.stream(
                    model="claude-opus-4-6",
                    max_tokens=150,   # Keep responses brief for natural voice
                    system=system_context,
                    messages=claude_messages
                ) as stream:
                    async for text_chunk in stream.text_stream:
                        response_text += text_chunk
                        # Stream TTS as Claude generates (reduce latency)
                        if len(text_chunk) > 20 or text_chunk.endswith((".", "?", "!")):
                            await stream_tts_to_twilio(websocket, tts_client, text_chunk)

                claude_messages.append({"role": "assistant", "content": response_text})

                # Check for booking completion
                booking = await detect_booking_confirmation(response_text, claude_messages)
                if booking:
                    await nexhealth_client.create_appointment(
                        location_id=practice.nex_health_location_id,
                        patient_id=booking.nex_health_patient_id,
                        provider_id=booking.provider_id,
                        start_time=booking.start_time,
                        service_type=booking.service_type
                    )
                    await publish_booking_event(practice.id, booking)

            elif twilio_message["event"] == "stop":
                # Call ended
                await log_call_outcome(session_id, claude_messages)
                break
```

**System prompt structure**:
```
You are the scheduling assistant for {practice.name}. Your name is "the scheduling assistant."

PRACTICE INFORMATION:
- Practice name: {practice.name}
- Location: {practice.city}, {practice.state}
- Specialty: {practice.specialty}
- Providers: {provider_list with specialties}
- Services offered: {service_list}

YOUR ROLE:
- Answer inbound calls professionally and warmly
- Help patients schedule appointments
- Answer common questions about services, hours, location, and insurance
- For questions you cannot answer (clinical questions, complex insurance), offer to have someone call them back

BOOKING INSTRUCTIONS:
- When a patient wants to schedule, identify the appropriate provider based on their need
- Present 2-3 available time slots (availability provided in context)
- Confirm patient's name, date of birth (for existing patients), and contact number
- Confirm the booking verbally and inform them a text confirmation will follow

ESCALATION TRIGGERS (transfer to human or offer callback):
- Complex insurance questions
- Clinical questions (pain, emergency)
- Patient complaint or upset patient
- Request to speak to a specific person

TONE:
- Warm and professional, not robotic
- Use the patient's name once you have it
- Never say "I'm an AI" unless directly asked; if asked, answer honestly
- Keep responses brief (1-3 sentences for most turns)
```

---

## Service 3: Lead Response Orchestrator (Temporal Worker)

Temporal workflows for the lead response stage. Runs as a dedicated Temporal worker.

### LeadResponseWorkflow

```python
@workflow.defn
class LeadResponseWorkflow:
    """
    Executes the 3-touch lead nurture sequence.
    Sleeps between touches using Temporal's durable timers (no cron required).
    """

    @workflow.run
    async def run(self, input: LeadResponseInput) -> str:
        # Touch 1: Immediate (fires as soon as workflow starts)
        result_1 = await workflow.execute_activity(
            send_lead_response_message,
            LeadMessageInput(
                practice_id=input.practice_id,
                patient_id=input.patient_id,
                touch_number=1,
                context=input.context  # first_name, interest, provider_name
            ),
            start_to_close_timeout=timedelta(minutes=2)
        )

        if result_1.patient_booked or result_1.patient_opted_out:
            return "completed_early"

        # Wait for response (4 hours) — Temporal durable sleep
        response = await workflow.wait_condition(
            lambda: self.patient_responded,
            timeout=timedelta(hours=4)
        )

        if self.patient_responded:
            await workflow.execute_activity(handle_lead_response, ...)
            return "responded"

        # Touch 2: +4 hours (no response)
        result_2 = await workflow.execute_activity(
            send_lead_response_message,
            LeadMessageInput(practice_id=input.practice_id, patient_id=input.patient_id, touch_number=2, context=input.context),
            start_to_close_timeout=timedelta(minutes=2)
        )

        if result_2.patient_booked or result_2.patient_opted_out:
            return "completed_early"

        # Wait for response (24 hours from Touch 1)
        await workflow.wait_condition(
            lambda: self.patient_responded,
            timeout=timedelta(hours=20)  # 24h total - 4h already elapsed
        )

        if self.patient_responded:
            return "responded"

        # Touch 3: +24 hours (no response — final touch)
        await workflow.execute_activity(
            send_lead_response_message,
            LeadMessageInput(practice_id=input.practice_id, patient_id=input.patient_id, touch_number=3, context=input.context),
            start_to_close_timeout=timedelta(minutes=2)
        )

        return "sequence_completed"

    @workflow.signal
    def patient_responded(self, response_data: dict):
        self.patient_responded = True
        self.response_data = response_data
```

### MissedCallTextBackWorkflow

```python
@workflow.defn
class MissedCallTextBackWorkflow:

    @workflow.run
    async def run(self, input: MissedCallInput) -> str:
        # Check quiet hours
        practice = await workflow.execute_activity(get_practice_config, input.practice_id)

        send_at = calculate_send_time(
            now=workflow.now(),
            practice_timezone=practice.timezone,
            quiet_hours_start=practice.quiet_hours_start,
            quiet_hours_end=practice.quiet_hours_end
        )

        if send_at > workflow.now():
            # In quiet hours — sleep until appropriate send time
            await workflow.sleep(send_at - workflow.now())

        # Send missed call text-back
        result = await workflow.execute_activity(
            send_missed_call_textback,
            MissedCallTextInput(
                practice_id=input.practice_id,
                patient_phone_hash=input.phone_hash,
                is_after_hours=input.is_after_hours
            ),
            start_to_close_timeout=timedelta(minutes=2)
        )

        if result.patient_responded:
            return "responded"

        # 4-hour follow-up if no response
        await workflow.sleep(timedelta(hours=4))

        # ... (same 2-touch pattern as LeadResponseWorkflow)
        return "completed"
```

---

## Service 4: Message Generation (Activity — called by Temporal workers)

```python
@activity.defn
async def generate_lead_response_message(input: MessageGenerationInput) -> str:
    """
    Calls Claude API to generate a personalized lead response message.
    Input contains patient context, practice context, touch number.
    PHI (patient first name) is used only for generation and never stored.
    """
    async with anthropic.AsyncAnthropic() as client:
        response = await client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=160,  # SMS limit
            system="""You write patient communications for dental practices.
                      Write warm, professional messages that sound human and never salesy.
                      Always use the patient's first name once.
                      Never mention diagnoses or sensitive medical information.
                      Keep SMS messages under 160 characters when possible.""",
            messages=[{
                "role": "user",
                "content": f"""Write a {input.touch_number} touch lead response SMS for:
                    Practice: {input.practice_name} ({input.practice_specialty})
                    Provider: {input.provider_name}
                    Patient: {input.patient_first_name}
                    Interest: {input.treatment_interest}
                    Touch {input.touch_number} purpose: {input.touch_purpose}
                    Available times to offer: {input.available_times}

                    Return ONLY the message text. No quotes. No explanation."""
            }]
        )

    message_text = response.content[0].text

    # HIPAA compliance: scrub before logging
    # (message_text contains patient name — never log it)
    return message_text
```

---

## Service 5: Communication Dispatcher (Activity)

```python
@activity.defn
async def dispatch_sms(input: SMSDispatchInput) -> DispatchResult:
    """
    Sends SMS via Twilio. Handles supervised mode (pending_approvals) vs autonomous.
    """
    practice = await get_practice_config(input.practice_id)
    agent = await get_agent_instance(input.practice_id)
    autonomy_level = getattr(agent, f"{input.stage}_level")

    if autonomy_level == "supervised":
        # Don't send — create pending approval record
        approval = await create_pending_approval(
            practice_id=input.practice_id,
            sequence_id=input.sequence_id,
            patient_id=input.patient_id,
            stage=input.stage,
            touch_number=input.touch_number,
            proposed_channel="sms",
            template_id=input.template_id,
            auto_send_at=datetime.utcnow() + timedelta(hours=2)
        )
        await trigger_digest_notification(input.practice_id)
        return DispatchResult(status="pending_approval", approval_id=str(approval.id))

    # Autonomous or Monitored — send immediately via Twilio
    # Retrieve actual phone number from NexHealth (not stored in our DB)
    patient_phone = await nexhealth_client.get_patient_phone(
        input.nex_health_patient_id,
        input.nex_health_location_id
    )

    message_sid = await twilio_client.messages.create_async(
        to=patient_phone,
        from_=practice.twilio_phone_number,
        body=input.message_text
    )

    # Log event (no PHI — message_text not stored)
    await log_communication_event({
        "practice_id": input.practice_id,
        "patient_id": input.patient_id,
        "event_type": "sms_sent",
        "stage": input.stage,
        "channel": "sms",
        "direction": "outbound",
        "twilio_sid": message_sid,
        "template_id": input.template_id,
        "touch_number": input.touch_number,
        "sequence_id": input.sequence_id
    })

    return DispatchResult(status="sent", twilio_sid=message_sid)
```

---

## API Endpoints (Phase 1)

### GET /practices/{id}/approvals/digest

Returns today's pending approvals for supervised mode.

**Response (200)**:
```json
{
  "practice_id": "uuid",
  "practice_name": "Garner Dental Group",
  "digest_date": "2026-03-10",
  "auto_send_at": "2026-03-10T14:00:00Z",
  "pending_count": 3,
  "approvals": [
    {
      "id": "uuid",
      "patient_display": "James H.",
      "stage": "lead_response",
      "touch_number": 1,
      "channel": "sms",
      "proposed_message": "Hi James, thanks for reaching out...",
      "scheduled_send_at": "2026-03-10T12:00:00Z",
      "auto_send_at": "2026-03-10T14:00:00Z"
    }
  ]
}
```

**HIPAA note**: `patient_display` is first name + last initial only (never full name, DOB, or account number). `proposed_message` is returned for display but is never stored in the communication_events table.

---

### POST /practices/{id}/approvals/{approval_id}/decide

Approve, edit, or reject a single pending message.

**Request body**:
```json
{
  "decision": "approved",  // approved | edited | rejected
  "edited_content": "Hi James, just wanted to follow up..."  // required if decision == "edited"
}
```

**Response (200)**:
```json
{
  "approval_id": "uuid",
  "decision": "approved",
  "send_queued": true,
  "send_at": "2026-03-10T12:05:00Z"
}
```

**Handler logic**:
- `approved`: updates `pending_approvals.status = 'approved'`, triggers Temporal signal to proceed with dispatch.
- `edited`: encrypts edited content with AES-256, stores in `edited_content_enc`, sets status = 'edited', triggers dispatch with edited content. Edit is logged as a training signal.
- `rejected`: sets status = 'rejected', sends Temporal signal to cancel this touch (sequence continues to next touch on schedule).

---

### POST /practices/{id}/approvals/bulk-approve

Approve all pending messages in today's digest.

**Request body**: `{}` (no body required)

**Response (200)**:
```json
{
  "approved_count": 3,
  "send_queued": true
}
```

---

### GET /practices/{id}/agent

**Response (200)**:
```json
{
  "practice_id": "uuid",
  "lead_response_level": "monitored",
  "treatment_followup_level": "supervised",
  "rebooking_level": "monitored",
  "reactivation_level": "supervised",
  "last_activity": "2026-03-10T09:45:00Z",
  "active_sequences": {
    "lead_response": 3,
    "treatment_followup": 0,
    "rebooking": 0,
    "reactivation": 0
  }
}
```

---

## HIPAA Compliance Notes

### Data Minimization
- Patient records in PostgreSQL contain only: `phone_hash` (SHA-256 of E.164 phone), `email_hash` (SHA-256 of lowercase email), and `nex_health_patient_id`.
- Patient names, DOBs, and addresses are never stored in our database — fetched from NexHealth at message generation time, used once, discarded.
- Message content (including patient names) is never stored in `communication_events` — only event metadata (event_type, channel, template_id, outcome).

### Audit Trail
- `communication_events` is append-only. No UPDATE or DELETE statements are permitted for this table (enforced via PostgreSQL role grants: `GRANT INSERT ON communication_events TO app_role`; no UPDATE/DELETE grant).
- All events include `created_at` timestamp, `practice_id`, `event_type`, and `twilio_sid` for cross-referencing with Twilio's own logs.
- Retained for 6 years per HIPAA requirements (RDS backup policy + S3 archive).

### Encryption
- Encryption at rest: RDS storage encryption (AES-256) enabled on all tables.
- Encryption in transit: TLS 1.2+ enforced on ALB and all inter-service communication.
- `edited_content_enc` in `pending_approvals`: edited message content (which contains patient name) is encrypted with AES-256-GCM before storage. Key stored in AWS Secrets Manager.

### PHI in Kafka
- Kafka topics are in-transit only — messages are consumed and deleted. Max retention: 24 hours.
- Kafka topics with PHI (`form-fill-events`) are encrypted in transit (MSK TLS).
- MSK cluster is in a private subnet with no public endpoint.

---

## Testing Strategy

### Unit Tests
- Message generation: test Claude prompt produces HIPAA-compliant output (no PHI in logs, no diagnosis language, character limit compliance)
- Compliance rules: test opt-out check, quiet hours check, consent check — all must block dispatch before Twilio is called
- Approval dispatch: test supervised/monitored/autonomous branching in `dispatch_sms`
- TwiML generation: test Voice AI TwiML response for valid XML structure and correct WebSocket URL

### Integration Tests
- NexHealth webhook → Kafka → Temporal workflow → message dispatch: end-to-end test with NexHealth sandbox
- Twilio inbound call → Voice AI WebSocket → Claude → TTS → booking → NexHealth write-back: end-to-end with Twilio test credentials
- Opt-out flow: SMS STOP → opt-out record → subsequent dispatch blocked
- Missed call at 10 PM → text-back queued for 7 AM next morning

### HIPAA Compliance Test
- Confirm no plain-text phone numbers or patient names in PostgreSQL tables
- Confirm no PHI in CloudWatch logs (structured logging with PHI fields scrubbed)
- Confirm `communication_events` rejects UPDATE and DELETE statements
- Confirm `edited_content_enc` is encrypted at rest and not readable from DB query

### Load Test (before Stage 1 go-live)
- Simulate 5 practices × 50 concurrent webhook events (NexHealth, Twilio)
- Confirm Twilio voice webhook responds within 3 seconds under load
- Confirm missed call text-back fires within 90 seconds under concurrent load
