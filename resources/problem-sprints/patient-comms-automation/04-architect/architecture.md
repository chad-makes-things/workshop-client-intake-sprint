# Architecture: The Comms Engine

## Overview
The Comms Engine is an autonomous AI agent system that executes the full patient communications playbook across SGA's 260+ dental practices. Each practice gets its own agent instance that handles lead response, treatment follow-up, rebooking, and database reactivation — autonomously. It extends the Practice Concierge agent pattern and feeds data to the Funnel OS.

---

## System Architecture

### Layer 1: Data Integration (shared with Funnel OS)
Connects to practice PMS/EHR systems to receive trigger events and read/write patient and appointment data.

- **NexHealth Synchronizer** — Universal API for 20+ dental PMS systems (Dentrix, Eaglesoft, Open Dental, Denticon, etc.)
- **Event Stream** — Real-time webhooks for: new patient form, appointment created/completed/cancelled, treatment plan created/updated, patient status changed
- **AI Note-Taker Bridge** — Ingests transcription output from in-room AI note-takers, extracts treatment mentions and patient sentiment
- **Phone System Integration** — Connects to practice phone systems for call events (answered, missed, voicemail, duration)

### Layer 2: Comms Engine Core (new — this sprint)
The intelligence and orchestration layer that decides what to communicate, when, to whom, and via which channel.

- **Playbook Engine** — Rule engine executing the SGA Patient Communication Protocol. Manages cadences, triggers, escalation rules, and compliance checks.
- **Agent Orchestrator** — Manages 260+ agent instances (one per practice). Each instance maintains practice-specific context: provider roster, specialties, schedule patterns, communication history, patient preferences.
- **Message Generator** — LLM-powered message composition with personalization. Uses patient context (name, treatment, provider, history), practice context (specialty, tone), and network-optimized templates.
- **Channel Router** — Decides optimal channel (SMS, voice, email) based on patient demographics, time of day, message type, and historical response data.
- **Escalation Manager** — Identifies when a communication needs human intervention and routes to the office manager's weekly summary or real-time alert.

### Layer 3: Communication Transport
The infrastructure layer that sends and receives messages across all channels.

- **Twilio (HIPAA/BAA)** — SMS, MMS, voice calls, email delivery. HIPAA-eligible with Business Associate Agreement.
- **Voice AI Engine** — LLM-powered conversational voice for inbound call answering and outbound reactivation calls. Integrates with Twilio Voice.
- **Email Service** — Transactional and campaign email via Twilio SendGrid or Amazon SES (both HIPAA-eligible).

### Layer 4: Monitoring & Learning (shared with Funnel OS)
Network-wide analytics, performance tracking, and continuous optimization.

- **Outcome Tracker** — Measures: message sent → response received → action taken → revenue produced. Attributes revenue to specific communications.
- **Network Learning Engine** — Analyzes communication performance across all 260 practices. Identifies winning patterns. Updates playbook defaults.
- **Compliance Auditor** — Logs every communication for HIPAA audit trail. Monitors for PHI leaks, consent violations, opt-out compliance.
- **Graduation Manager** — Tracks each practice's trust level (Supervised → Monitored → Autonomous). Manages the graduated autonomy protocol.

### Layer 5: Human Interface (minimal)
The 20-minute weekly touchpoint — the only practice-facing UI.

- **Weekly Summary Generator** — Produces the 3-section summary (what engine did, what needs attention, what's coming). Delivered via SMS or email to office manager.
- **Approval Interface** — During supervised mode: daily digest with approve/edit/reject actions for pending messages. Simple tap interface, not a dashboard.
- **HQ Playbook Manager** — Web interface for HQ to design, test, and deploy playbook updates network-wide. Visual cadence designer, template editor, A/B test configuration.

---

## Component Architecture

### Component 1: Practice Agent Instance
**Responsibility**: Autonomous execution of the playbook for a single practice.
**Interfaces**: Receives events from Data Integration layer. Sends messages via Communication Transport layer. Reports outcomes to Monitoring layer.
**Key behaviors**:
- Maintains practice context (providers, specialties, schedule, patient mix)
- Executes all 4 playbook stages (lead, treatment, rebooking, reactivation)
- Personalizes messages using LLM with practice and patient context
- Escalates to human when confidence is low or situation requires judgment
- Operates in Supervised, Monitored, or Autonomous mode based on trust level

### Component 2: Playbook Engine
**Responsibility**: Defines and executes the communication protocol — the rules for when, what, and how to communicate.
**Interfaces**: Receives trigger events. Outputs communication tasks to Agent Instances.
**Key behaviors**:
- Manages cadence definitions (e.g., treatment follow-up: Day 3, 7, 14, 30)
- Applies practice-type variants (cosmetic vs. family vs. specialty)
- Enforces compliance rules (opt-out, consent, quiet hours, HIPAA)
- Supports A/B testing at network level (Template A vs. Template B across random practice groups)
- Accepts playbook updates from HQ without agent restart

### Component 3: Message Generator
**Responsibility**: Composes personalized, natural-sounding messages for every communication.
**Interfaces**: Receives communication task + context. Outputs formatted message ready for delivery.
**Key behaviors**:
- LLM-powered composition with structured prompts per message type
- Merges patient data (name, treatment, provider, history), practice data (name, tone, specialty), and network-optimized templates
- Applies tone guidelines (warm, professional, never salesy)
- Generates multiple variants for A/B testing
- Validates PHI handling before output (no sensitive data in subject lines, etc.)

### Component 4: Voice AI Agent
**Responsibility**: Handles phone conversations — both inbound (answering calls) and outbound (reactivation, high-value follow-up).
**Interfaces**: Twilio Voice API for call handling. NexHealth for real-time PMS queries during calls.
**Key behaviors**:
- Natural conversational voice with practice-specific personality
- Real-time PMS schedule lookup during calls
- Books appointments directly into PMS via NexHealth write API
- Handles common questions (hours, insurance, directions, services)
- Graceful handoff to human when out of scope
- Records call summary and outcome for Funnel OS logging

### Component 5: Network Learning Engine
**Responsibility**: Continuous optimization of the playbook based on network-wide outcome data.
**Interfaces**: Reads outcome data from all 260 practices. Writes playbook updates to Playbook Engine.
**Key behaviors**:
- Analyzes message performance across demographics, time slots, message types, practice types
- Identifies statistically significant patterns (minimum sample size before promoting a change)
- Generates "Playbook Evolution Reports" for HQ
- Respects practice-level opt-outs (guardrails)
- Runs monthly optimization cycles with weekly trend monitoring

### Component 6: Graduation Manager
**Responsibility**: Manages the trust-building protocol for new practice deployments.
**Interfaces**: Reads message quality data and human edit rates. Writes autonomy level changes to Agent Instances.
**Key behaviors**:
- Tracks 3 autonomy levels per communication type per practice
- Supervised: Human reviews before send (daily digest with approve/edit)
- Monitored: Auto-sends with daily digest showing what was sent (human can pause)
- Autonomous: Auto-sends with weekly summary only
- Graduation criteria: <5% edit rate, >90% approval rate, >70% patient response rate, 0 compliance flags over 30 days
- Supports manual override by HQ (force practice to supervised mode)

---

## Data Flow

```
TRIGGER EVENT FLOW:
PMS (via NexHealth) ──→ Event Stream ──→ Playbook Engine ──→ Practice Agent Instance
Phone System ──────────→ Event Stream ──→    ↓                      ↓
AI Note-Taker ─────────→ Event Stream ──→    ↓               Message Generator
Web Form Webhook ──────→ Event Stream ──→    ↓                      ↓
                                              ↓               Channel Router
                                              ↓                      ↓
                                              ↓               Twilio (SMS/Voice/Email)
                                              ↓                      ↓
                                              ↓               Patient receives message
                                              ↓                      ↓
                                         Outcome Tracker ←── Patient responds (or doesn't)
                                              ↓
                                         Network Learning Engine
                                              ↓
                                         Playbook updated (network-wide)
```

```
HUMAN TOUCHPOINT FLOW:
Practice Agent Instance ──→ Escalation Manager ──→ Weekly Summary ──→ Office Manager
                                    ↓
                              (during supervised mode)
                                    ↓
                              Daily Digest ──→ Office Manager ──→ Approve/Edit ──→ Message sends
```

---

## Architectural Decision Records (ADRs)

### ADR-1: Agent-Per-Practice vs. Shared Agent Pool
**Decision**: One agent instance per practice.
**Context**: Could run a shared pool of agents that handle requests across practices, or dedicate an agent per practice.
**Rationale**: Dedicated agents maintain practice-specific context (providers, tone, patient history patterns) without context-switching overhead. At 260 practices, the cost of dedicated instances is manageable. The agent IS the practice's communication personality — sharing it dilutes that. Aligns with Practice Concierge pattern (one concierge per practice).
**Tradeoff**: Higher infrastructure cost vs. better personalization and simpler debugging (one practice's issues don't affect others).

### ADR-2: NexHealth as PMS Integration Layer (vs. Direct Integrations)
**Decision**: Use NexHealth Synchronizer API as the universal PMS connector.
**Context**: SGA practices use 75+ different PMS/EHR systems. Building direct integrations to each is infeasible.
**Rationale**: NexHealth supports 20+ major dental PMS systems, covering an estimated 80%+ of SGA practices. One API to maintain instead of 20+. NexHealth handles PMS-specific quirks, version changes, and authentication. For the remaining ~20% on unsupported PMS systems, build a lightweight adapter that manually captures trigger events (webhook-based, minimal PMS dependency).
**Tradeoff**: Vendor dependency on NexHealth + API cost at scale vs. years of custom integration work.

### ADR-3: Twilio for Communication Transport (vs. Vendor Platforms)
**Decision**: Use Twilio's HIPAA-eligible APIs for SMS, voice, and email.
**Context**: Could use dental-specific platforms (TrueLark, Weave, RevenueWell) that bundle communication + intelligence, or build on commodity infrastructure.
**Rationale**: At 260 practices, the cost differential is massive. Twilio: ~$47/practice/month (estimated for volume). Vendor platforms: $189-299/practice/month. Annual savings: $440K-$785K. More importantly, building on Twilio means SGA owns the intelligence layer — the playbook, the network learning, the agent behavior are all proprietary IP. Vendor platforms mean renting someone else's intelligence.
**Tradeoff**: Higher development effort vs. lower ongoing cost + full ownership of IP + no vendor lock-in.

### ADR-4: Graduated Autonomy Protocol (vs. Full Auto from Day 1)
**Decision**: New practices start in Supervised mode and graduate to Autonomous over 30-60 days.
**Context**: Could deploy fully autonomous from Day 1 (faster ROI) or require permanent human oversight (safer but doesn't scale).
**Rationale**: Graduated autonomy solves the adoption problem. Practice staff who are skeptical of AI get to review messages before they send. This builds trust AND generates fine-tuning data (their edits improve the agent). Once message quality is proven (< 5% edit rate), the practice graduates. This is how you get 260 practices to adopt AI communications without a training program or change management initiative.
**Tradeoff**: Slower initial ROI (30-day supervised period) vs. dramatically higher adoption rate and staff trust.

### ADR-5: Network Learning as Core Architecture (vs. Static Playbook)
**Decision**: The playbook is dynamic — updated monthly based on network-wide outcome data.
**Context**: Could deploy a fixed playbook (simpler, more predictable) or build a learning system that evolves.
**Rationale**: 260 practices sending thousands of messages daily is a massive natural experiment. A static playbook leaves this data on the table. The network learning engine turns SGA's scale into a competitive advantage — every message is a data point that improves the next message. Over 12 months, the optimized playbook becomes SGA's most valuable communication IP. No vendor platform can replicate this because no vendor has SGA's specific patient population data.
**Tradeoff**: More complex architecture + risk of learning engine introducing regressions vs. continuous improvement and massive long-term value.

---

## Integration with Prior Sprint Architectures

### Practice Concierge (Digital Storefront sprint)
The Comms Engine agent IS the Practice Concierge agent, extended with communication capabilities. Same agent instance, new skill set. The Practice Concierge already handles storefront management; it now also handles patient communications. This means practices interact with ONE agent, not two separate systems.

### Funnel OS (Full Funnel Visibility sprint)
The Comms Engine feeds all communication events and outcomes to the Funnel OS data layer. The Funnel OS dashboard shows communication performance alongside funnel metrics. The Practice Nerve Center (from Funnel OS storyboard) is the same weekly summary the Comms Engine generates — they're the same interface.

### Convergence Vision
```
Practice Concierge (agent interface)
    ├── Digital Storefront Management (Sprint 2)
    ├── Patient Communications (This Sprint)
    └── [Future: 5-Star Experience Culture, Staff Coaching, etc.]

Funnel OS (data + visibility)
    ├── Data Integration Layer
    ├── Executive Dashboard
    ├── Practice Nerve Center = Comms Engine Weekly Summary
    └── Network Learning Engine (shared)
```

---

## Deployment Roadmap

### Month 1-2: Lead Response (Highest ROI, Simplest Triggers)
- Deploy voice AI for inbound calls (inbound overflow + missed call recovery)
- Deploy form fill auto-response (webhook → instant text + email)
- Deploy missed-call text-back
- Lead nurture sequences (Day 1, 3, 7)
- Start with 5 pilot practices in Supervised mode

### Month 2-3: Treatment Follow-Up (Highest Revenue Impact)
- PMS integration via NexHealth for treatment plan triggers
- AI note-taker bridge for treatment mention detection
- 4-touch cadence (Day 3, 7, 14, 30) + high-value 5-touch extension
- Expand to 20 practices, graduate pilot practices to Monitored

### Month 3-4: Rebooking (Retention)
- Post-appointment rebooking detection and automated outreach
- Hygiene recall sequences (5-month, 5.5-month, 6-month)
- Expand to 50 practices

### Month 4-6: Reactivation (Database Revenue Recovery)
- Dormant patient identification and tiered segmentation
- Outbound AI voice for high-value reactivation
- Monthly reactivation campaigns
- Expand to 100+ practices, network learning engine goes live

### Month 6-12: Full Network + Optimization
- All 260+ practices live
- Network learning engine optimizing playbook monthly
- New practice onboarding automated (30-day graduated autonomy)
- Playbook expanding to new communication types as data reveals opportunities

---

## Tech Stack Summary

| Component | Technology | Why |
|-----------|-----------|-----|
| PMS Integration | NexHealth Synchronizer API | Universal dental PMS connector, 20+ systems |
| SMS/MMS | Twilio Messaging (HIPAA) | Commodity pricing, BAA, programmable |
| Voice AI | Twilio Voice + LLM (Claude/GPT) | Natural conversation, real-time PMS queries |
| Email | Twilio SendGrid or Amazon SES | HIPAA-eligible, high deliverability |
| Agent Runtime | Python/Node.js + LLM orchestration | Flexible, LLM-native |
| Playbook Engine | Custom rule engine (event-driven) | Configurable cadences, triggers, compliance |
| Data Store | PostgreSQL + event log (append-only) | HIPAA audit trail, analytics |
| Network Learning | Python ML pipeline (batch) | Monthly optimization cycles |
| HQ Dashboard | React web app | Playbook management, analytics |
| Infrastructure | AWS (HIPAA-eligible) or GCP | Scalable, compliant, managed services |
