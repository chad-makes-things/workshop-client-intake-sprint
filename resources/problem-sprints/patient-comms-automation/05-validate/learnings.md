# Sprint Learnings: Patient Communications Automation

## Sprint Summary
- **Problem**: Fragmented, manual patient communication across 260+ dental practices — leads unworked, treatment unfollowed, patients lapsing
- **Solution**: The Comms Engine — autonomous AI agent per practice executing a full lifecycle communication playbook
- **Target**: Full lifecycle (all 4 stages) — lead response, treatment follow-up, rebooking, database reactivation
- **Date**: 2026-03-10

---

## Key Learnings

### 1. The "Agent Army" Vision is Architecturally Correct
The user's instinct — "an agent army, not a dashboard" — maps precisely to the right architecture. Each practice gets a dedicated agent that maintains practice-specific context and operates autonomously. This aligns with the Practice Concierge pattern from the Storefront sprint and creates a natural extension point. The convergence of storefront management + patient communications + operations into a single per-practice agent is the long-term platform vision.

### 2. The TC Resistance Problem is a Design Constraint, Not a Feature Request
70% of treatment coordinators resist follow-up because it "feels salesy." The solution isn't training them to be more comfortable with sales — it's removing them from the loop entirely. Automated messages sent under their name, without their involvement, solves the behavioral problem more effectively than any coaching program. The graduated autonomy protocol ensures quality without requiring staff buy-in.

### 3. Network Learning is the Competitive Moat
At 260 practices, every automated message is a data point. The network learning engine — discovering that Saturday morning follow-ups get 2.1x response, or that patients 55+ prefer voice 3x over text — creates proprietary intelligence that no vendor platform can replicate. Over 12 months, SGA's communication playbook becomes its most valuable operational IP.

### 4. The Cost Arbitrage is Massive
Vendor platforms: $189-299/practice/month ($590K-$930K/year at 260 practices). The Comms Engine on Twilio + NexHealth: ~$47/practice/month ($147K/year). Annual savings of $440K-$785K — AND the custom system does more (network learning, Practice Concierge integration, full lifecycle automation). The build-vs-buy case is unambiguous at this scale.

### 5. Sub-60-Second Lead Response is the Headline Feature
Research data is overwhelming: responding within 1 minute increases conversions by 391%. The industry average is 47 hours. This single capability — instant, intelligent, 24/7 response to every lead — may deliver more ROI than all other features combined. It's also the easiest to deploy (Month 1) and the hardest for any human team to match.

---

## What This Sprint Didn't Solve

### 1. PMS Integration Depth
The architecture uses NexHealth for PMS integration, but the specific data quality, sync latency, and edge cases across 75+ PMS systems need real-world validation. The "minimum viable trigger set" (5-7 events) needs to be defined and tested against the top 5 PMS systems.

### 2. AI Voice Quality at Scale
The storyboard assumes natural, practice-specific voice AI for both inbound and outbound calls. Current voice AI quality varies. A thorough evaluation of voice AI providers (with dental-specific testing) is needed before deploying voice outbound at scale.

### 3. HIPAA Compliance Architecture
The architecture identifies HIPAA as a requirement and specifies Twilio BAA + compliance auditing, but the detailed compliance architecture (PHI handling in LLM prompts, data residency, patient consent management, breach notification protocol) needs dedicated legal and compliance review.

### 4. AI Note-Taker Integration
The treatment follow-up module uses AI note-taker output (in-room treatment mentions) as a trigger enrichment source. The specific integration path — which note-taker products, what data format, how to extract treatment signals reliably — is undefined and needs technical spike.

### 5. 5-Star Experience Culture (Separate Problem)
The user explicitly called out that in-practice culture, hospitality, and staff training is a separate problem from communications automation. This sprint addresses the automation; the culture problem needs its own sprint.

---

## Connections to Other Sprints

| Sprint | Connection |
|--------|-----------|
| **Digital Storefront at Scale** | Comms Engine agent = Practice Concierge agent extended. Same instance, new capabilities. |
| **Full Funnel Visibility** | Comms Engine feeds Funnel OS data layer. Practice Nerve Center = Comms Engine weekly summary. Network learning engine shared. |
| **Client Onboarding Intake** | Pattern reuse: graduated autonomy from Comms Engine could apply to client intake flows. |

---

## Recommended Next Steps

1. **Development Package** — Generate full specs (PM epic, UX design, tech lead plan, backend/frontend specs) for Phase 1 (Lead Response automation)
2. **PMS Integration Spike** — Technical evaluation of NexHealth Synchronizer with SGA's top 5 PMS systems
3. **Voice AI Evaluation** — Test 3-4 voice AI providers with dental-specific scenarios
4. **Compliance Review** — HIPAA architecture review with legal/compliance team
5. **Pilot Practice Selection** — Identify 5 practices for Month 1 pilot (mix of practice types, PMS systems, staff attitudes)
6. **Separate Sprint: 5-Star Experience Culture** — The in-practice hospitality/culture problem identified during framing
