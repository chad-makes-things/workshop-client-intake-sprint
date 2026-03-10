# Solution Approaches

## Target
Full Lifecycle Playbook — automated patient communications across all stages: lead response, treatment follow-up, rebooking, and database reactivation for 260+ dental practices.

---

## Solution 1: The Comms Engine — Autonomous Agent Playbook

### Narrative
An autonomous AI agent system — not a dashboard, not a CRM — that executes the entire patient communications playbook without human intervention. Each practice gets its own agent instance that connects to the practice's phone system, PMS, and messaging channels. The agent follows a master playbook (designed by HQ) with practice-level customization. It handles inbound leads (sub-60-second response), outbound treatment follow-up (3/7/14/30-day cadences), rebooking enforcement, and lapsed patient reactivation — all automatically.

The 20-minute weekly human touchpoint is the ONLY interface: the agent sends the office manager a summary of what it did this week, what needs human attention (3-5 items max), and what's coming next week. Everything else runs autonomously.

HQ sees network-wide comms performance through the Funnel OS dashboard (from prior sprint). The agent IS the execution layer that the Funnel OS monitors. Extends the Practice Concierge agent pattern — this is the same agent, now with communication capabilities.

### How Each Stage Works

**Stage 1 — Lead Response (< 60 seconds)**
- Inbound call → AI voice answers (or picks up after 3 rings if front desk busy)
- Form fill → Instant text + email response with booking link
- Missed call → Immediate text-back ("Sorry we missed you, want to schedule?")
- Lead enters nurture sequence if not immediately booked (Day 1, 3, 7 texts)

**Stage 2 — Treatment Follow-Up (3/7/14/30-day cadence)**
- Treatment presented + not accepted → Trigger captured from PMS or AI note-taker
- Day 3: Friendly text from provider ("Hi [name], thinking about you — any questions about the [treatment] we discussed?")
- Day 7: Value-add message (educational content about the treatment)
- Day 14: Soft close ("We have openings next week if you'd like to move forward")
- Day 30: Final touch ("Just checking in — we're here whenever you're ready")
- $20K+ cases: Extended to 5+ touches with phone call escalation at touch 4

**Stage 3 — Rebooking**
- Patient completes treatment → Agent checks: is 6-month appointment booked?
- If no → Same-day text ("Thanks for coming in today! Let's get your next visit on the calendar")
- Day 1, 3, 7 follow-up if not rebooked
- Hygiene recall at 5 months (reminder), 5.5 months (urgent), 6 months (re-engagement)

**Stage 4 — Database Reactivation**
- Monthly scan of patients not seen in 12+ months
- Segmented outreach by last treatment type and patient value
- AI voice outbound for high-value patients ($5K+ lifetime)
- Text/email for standard reactivation
- 3-touch sequence over 30 days, then quarterly thereafter

### Architecture
- **Orchestration**: AI agent per practice (extends Practice Concierge)
- **Data**: NexHealth Synchronizer for PMS integration → Funnel OS data layer
- **Comms transport**: Twilio (SMS, voice, email) with HIPAA BAA
- **Intelligence**: LLM-powered message personalization + network learning engine
- **Playbook**: Configurable rule engine — HQ sets defaults, practices customize within guardrails

### Strengths
- Matches the user's stated vision: "agent army, not a dashboard"
- Extends existing Practice Concierge and Funnel OS architectures
- No new UI for practice staff to learn — agent operates autonomously
- 20-minute weekly touchpoint is the only human interface
- Network learning: what works at one practice improves all 260

### Risks
- PMS integration quality via NexHealth determines system reliability
- AI voice quality must be indistinguishable from human for patient trust
- HIPAA compliance across autonomous AI communications is complex
- Initial playbook design requires deep clinical input to get messaging right

### Feasibility
- Month 1-2: Core agent + lead response automation (highest ROI, simplest triggers)
- Month 2-3: Treatment follow-up (requires PMS treatment plan data)
- Month 3-4: Rebooking automation (requires appointment data)
- Month 4-6: Reactivation engine (requires patient history analysis)
- Month 6+: Network learning engine + continuous optimization

---

## Solution 2: The Playbook Platform — Configurable Automation Workflows

### Narrative
A workflow automation platform where HQ designs communication playbooks as visual workflows (trigger → sequence → escalation), then deploys them across the network. Think Zapier/n8n but purpose-built for dental patient communications with HIPAA compliance baked in.

Each communication stage is a workflow template. The Treatment Follow-Up workflow: "Treatment plan created in PMS → Wait 3 days → Send text template A → Wait 4 days → Send text template B → Wait 7 days → Send text template C → If no response → Escalate to office manager's weekly list." HQ builds the workflow, the platform executes it across 260 practices simultaneously.

Office managers don't interact with the platform directly. They receive a weekly email digest: "This week, 47 messages were sent on your behalf. 12 patients responded. 3 need your attention." The 20-minute touchpoint is handling those 3 items.

### How It's Different from Solution 1
Solution 1 is agent-first (LLM-driven, each practice has an autonomous agent). Solution 2 is workflow-first (rule-based, deterministic sequences with AI for personalization only). Solution 2 is more predictable and auditable but less adaptive.

### Strengths
- Highly predictable — every message is based on a defined workflow
- Easy to audit for HIPAA compliance (every message traced to a rule)
- HQ has complete control over what gets sent
- Visual workflow builder makes playbook changes fast
- Lower AI risk — no autonomous agent making decisions

### Risks
- Less adaptive than an agent — can't handle edge cases or novel situations
- Workflow complexity grows quickly when handling all 4 stages + variants
- May feel robotic to patients if personalization is limited to template variables
- Doesn't leverage the Practice Concierge agent pattern (parallel system)

### Feasibility
- Month 1: Workflow engine + lead response workflows
- Month 2: Treatment follow-up workflows
- Month 3: Rebooking + reactivation workflows
- Month 4: Network analytics + workflow optimization

---

## Solution 3: The Vendor Stack — Best-of-Breed Integration

### Narrative
Instead of building, assemble the best existing dental communication tools into a coordinated stack. Use TrueLark or Arini for AI voice (inbound + reactivation outbound), RevenueWell or Weave for automated messaging (reminders, follow-ups, recalls), NexHealth for PMS integration, and a custom orchestration layer that coordinates them all and feeds data into the Funnel OS.

The orchestration layer is thin — it doesn't send messages or make calls. It just ensures the right vendor tool fires at the right time and captures the outcome data. Think of it as the "conductor" that keeps the vendor orchestra playing the same song.

### Strengths
- Fastest time to value — vendors are already built and working
- Leverages proven dental-specific AI (TrueLark, Arini) with years of training data
- Lower development cost (orchestration layer only)
- Each vendor handles their own HIPAA compliance

### Risks
- **$400K-$700K/year in vendor costs** at 260 locations
- Vendor lock-in across 3-4 platforms
- Integration brittleness — vendors change APIs, add/remove features
- No network learning across practices (each vendor treats practices independently)
- Data lives in vendor silos, partially mitigating the Funnel OS vision
- Patient experience inconsistency — different vendors have different "voices"

### Feasibility
- Month 1: Vendor selection + contracts + initial integrations
- Month 2-3: Orchestration layer + Funnel OS data feed
- Month 3-4: Full deployment across 260 practices

---

## Solution 4: The Smart Playbook — AI-Augmented Human Coaching

### Narrative
Rather than automating communications directly, build a system that coaches practice staff to communicate better. The AI analyzes every patient interaction (calls via phone system, treatment presentations via note-taker, form fills, appointment outcomes) and generates real-time coaching for staff.

The office manager's 20-minute daily session becomes a coaching dashboard: "Today, your front desk missed 4 calls during the 11am-1pm rush. Here's a script for calling them back." "Your TC presented Invisalign to 3 patients this week but didn't follow up on any. Here are personalized follow-up messages ready to send — just tap to approve."

The AI does the THINKING (what to communicate, when, to whom, what to say). The HUMAN does the SENDING (one-tap approval). This solves the TC "feels salesy" problem differently — the TC is in control, they're just getting better recommendations.

Over time, as trust builds, more communications shift to fully automated. The system earns the right to autonomy by proving its recommendations work.

### Strengths
- Solves the staff adoption problem by making humans the hero, AI the assistant
- Builds trust incrementally — automation earns permission over time
- Higher-quality communications (human review catches tone issues)
- TC resistance addressed directly — they control the send button
- Valuable even if full automation is never achieved

### Risks
- Human bottleneck — if office managers don't do the 20 minutes, nothing gets sent
- Slower ROI — dependent on human compliance
- Doesn't solve the scale problem (260 offices × daily human decisions = variable outcomes)
- "Approve to send" fatigue — staff may rubber-stamp or ignore over time
- Doesn't address after-hours or instant response (still need AI for those)

### Feasibility
- Month 1: Coaching engine + daily digest for office managers
- Month 2: One-tap approval workflows for treatment follow-up
- Month 3: Reactivation recommendations + rebooking nudges
- Month 4-6: Graduated automation (auto-send for high-confidence recommendations)

---

## Solution 5: The Network Protocol — Standardized Comms Infrastructure

### Narrative
Design patient communications as a **protocol**, not a platform. Define the "SGA Patient Communication Protocol" — a specification for how patient communications should work at every stage, with exact timing, channel, message structure, escalation rules, and compliance requirements. Then implement the protocol using a headless EHR/CRM backend (like Canvas Medical or a custom build) that any practice's existing systems can plug into.

The protocol is the playbook. The headless backend is the execution engine. The Practice Concierge is the interface. The Funnel OS is the monitoring layer. This approach treats patient communications as infrastructure — like plumbing — rather than as a product.

Each practice connects via API (through NexHealth or direct PMS integration). The protocol engine receives events (new lead, treatment presented, appointment completed, patient lapsed) and executes the defined communication response. Practices don't configure anything — the protocol is the standard.

For the 5% of practices that already communicate well, the protocol captures what they do and propagates it. For the 95% that don't, the protocol installs best practices automatically.

### Strengths
- Most architecturally clean — protocol + engine + monitoring are separate concerns
- Fully standardized — every practice gets the same quality of communications
- Connects naturally to all three prior sprint architectures
- Scalable beyond dental — the protocol pattern works for any multi-location healthcare
- Captures institutional knowledge (what the top 5% of practices do) and deploys it network-wide
- No practice-level configuration required — "it just works"

### Risks
- Protocol rigidity — doesn't handle practice-specific nuances without escape hatches
- "One size fits all" may frustrate practices that want customization
- Headless EHR/CRM build is a significant infrastructure investment
- Protocol design requires deep clinical and operational input to get right
- Slower to iterate — changing a protocol is heavier than changing an agent prompt

### Feasibility
- Month 1: Protocol specification (the playbook document)
- Month 2-3: Headless engine build + PMS integration via NexHealth
- Month 3-4: Lead response + treatment follow-up stages live
- Month 4-6: Rebooking + reactivation stages + network rollout
- Month 6+: Protocol optimization based on network-wide outcome data

---

## Substitution Test
Each solution represents a genuinely different approach:

| Dimension | Sol 1: Comms Engine | Sol 2: Playbook Platform | Sol 3: Vendor Stack | Sol 4: Smart Playbook | Sol 5: Network Protocol |
|-----------|-------------------|------------------------|--------------------|--------------------|----------------------|
| **Core paradigm** | Autonomous AI agent | Deterministic workflows | Best-of-breed vendors | AI-coached humans | Standardized protocol |
| **Intelligence** | LLM-driven, adaptive | Rule-based, predictable | Vendor AI (varies) | AI recommends, human decides | Protocol-defined, data-optimized |
| **Human involvement** | 20 min/week review | Weekly digest | Minimal (vendor handles) | 20 min/DAY active coaching | Zero (protocol runs) |
| **Build vs. buy** | Build orchestration + AI | Build workflow engine | Buy + thin orchestration | Build coaching layer | Build protocol engine |
| **Customization** | Per-practice agent tuning | Per-practice workflow variants | Limited by vendor configs | Per-staff coaching style | Standard protocol, no customization |
| **Risk profile** | AI autonomy risk | Workflow complexity | Vendor dependency + cost | Human compliance risk | Protocol rigidity risk |
