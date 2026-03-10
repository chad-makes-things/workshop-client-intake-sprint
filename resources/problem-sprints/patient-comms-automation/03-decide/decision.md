# Decision Record

## Evaluation Matrix

All 5 solutions evaluated against the 5 sprint questions. Scale: 1 (does not address) to 5 (directly and fully addresses).

### SQ1: Can we automate follow-up without it feeling "salesy" to TCs?

| Solution | Score | Rationale |
|----------|-------|-----------|
| **1: Comms Engine** | 5 | TCs are completely removed from the loop. Agent sends under their name but they don't have to do anything. Eliminates the emotional labor entirely. |
| **2: Playbook Platform** | 4 | Same auto-send pattern, but less personalized — template-based messages may feel more generic. |
| **3: Vendor Stack** | 3 | Vendors handle messaging but with less control over tone. Multiple vendor "voices" may be inconsistent. |
| **4: Smart Playbook** | 4 | TCs stay in control (approve-to-send), which addresses their resistance — but still requires them to engage daily. |
| **5: Network Protocol** | 5 | Protocol-defined messaging is standard — TCs don't have to decide or act. Like Solution 1 but protocol-driven rather than agent-driven. |

### SQ2: Can a standardized playbook work across 260+ practices with different patient mixes?

| Solution | Score | Rationale |
|----------|-------|-----------|
| **1: Comms Engine** | 5 | Agent adapts per practice — same playbook framework, LLM-driven customization for practice type, specialty, patient demographics. |
| **2: Playbook Platform** | 3 | Workflow variants per practice type are possible but workflow complexity grows exponentially with customization. |
| **3: Vendor Stack** | 2 | Vendor configurations are limited. Each vendor offers its own customization model. Inconsistent across the stack. |
| **4: Smart Playbook** | 3 | Coaching adapts to each practice's patterns, but human execution varies by staff quality. |
| **5: Network Protocol** | 3 | Standard protocol works for 90% of cases but needs escape hatches for specialty practices. Less adaptive than an agent. |

### SQ3: Will patients respond positively to automated communications?

| Solution | Score | Rationale |
|----------|-------|-----------|
| **1: Comms Engine** | 5 | LLM-generated messages with patient name, specific treatment, provider name, and contextual timing. Feels personal. Network learning optimizes for response rates. |
| **2: Playbook Platform** | 3 | Template-based messages with variable substitution. Personalization limited to merge fields. May feel formulaic over time. |
| **3: Vendor Stack** | 4 | Dental-specific vendors have years of message optimization data. Proven message templates. |
| **4: Smart Playbook** | 5 | Human-approved messages can be personally adjusted. Highest potential for genuine personal touch. |
| **5: Network Protocol** | 3 | Protocol-defined messages are consistent but may lack warmth. Standardization trades personality for reliability. |

### SQ4: Can we get data out of 75+ PMS systems to trigger automations?

| Solution | Score | Rationale |
|----------|-------|-----------|
| **1: Comms Engine** | 4 | Uses NexHealth Synchronizer for top 20 PMS systems (covers 80%+). Remaining 20% need custom connectors or manual event triggers. |
| **2: Playbook Platform** | 4 | Same NexHealth integration approach. Workflow triggers map to PMS events. |
| **3: Vendor Stack** | 4 | Vendors like NexHealth, Weave already have PMS integrations. Less custom work needed. |
| **4: Smart Playbook** | 3 | Needs the same PMS data plus additional analytics (call recordings, note-taker output) for coaching. More data dependencies. |
| **5: Network Protocol** | 4 | Protocol defines a minimum viable event set. NexHealth provides the integration layer. Clean separation of protocol from implementation. |

### SQ5: Can this integrate with Practice Concierge and Funnel OS?

| Solution | Score | Rationale |
|----------|-------|-----------|
| **1: Comms Engine** | 5 | The agent IS the Practice Concierge, extended with comms capabilities. Data flows directly into Funnel OS. Single coherent system. |
| **2: Playbook Platform** | 3 | Separate workflow engine that feeds data to Funnel OS. Parallel to Practice Concierge rather than extending it. Two systems for practices to potentially interact with. |
| **3: Vendor Stack** | 2 | Vendor data must be extracted and fed into Funnel OS. Vendors operate independently from Practice Concierge. Loosest integration. |
| **4: Smart Playbook** | 3 | Coaching layer could live within Practice Nerve Center (from Funnel OS). But adds complexity to an already ambitious system. |
| **5: Network Protocol** | 4 | Protocol engine is a clean layer between Funnel OS (data) and Practice Concierge (interface). Architecturally elegant but adds another system to build and maintain. |

---

## Scoring Summary

| Solution | SQ1 | SQ2 | SQ3 | SQ4 | SQ5 | **Total** |
|----------|-----|-----|-----|-----|-----|-----------|
| **1: The Comms Engine** | 5 | 5 | 5 | 4 | 5 | **24/25** |
| **2: Playbook Platform** | 4 | 3 | 3 | 4 | 3 | **17/25** |
| **3: Vendor Stack** | 3 | 2 | 4 | 4 | 2 | **15/25** |
| **4: Smart Playbook** | 4 | 3 | 5 | 3 | 3 | **18/25** |
| **5: Network Protocol** | 5 | 3 | 3 | 4 | 4 | **19/25** |

---

## Detailed Verdicts

### Solution 1: The Comms Engine — RECOMMENDED
**Score**: 24/25
**Verdict**: Best overall solution. Extends the Practice Concierge agent pattern (architectural coherence), delivers full autonomous automation (matches user vision of "agent army"), and uniquely enables network-wide learning across 260 practices. The LLM-driven personalization addresses patient response quality while eliminating TC resistance entirely.
**Key risk**: AI autonomy in healthcare comms requires careful HIPAA guardrails and message review protocols during rollout.
**Mitigation**: Start with human-in-the-loop review for first 30 days per practice, then graduate to full autonomy as message quality is validated.

### Solution 2: Playbook Platform
**Score**: 17/25
**Verdict**: Solid but limited. Deterministic workflows are predictable and auditable, which appeals to compliance teams. But workflow complexity explodes across 4 stages × 260 practices × practice variants. Doesn't extend Practice Concierge. Good as a fallback if AI autonomy proves too risky.

### Solution 3: Vendor Stack
**Score**: 15/25
**Verdict**: Fastest to deploy but worst long-term option. $400K-$700K/year in vendor costs, data stays in silos, and doesn't integrate with SGA's platform vision. Only recommended if development capacity is zero and you need something running immediately.

### Solution 4: Smart Playbook
**Score**: 18/25
**Verdict**: The most human-centered approach. Best patient communication quality because humans review before sending. But it doesn't scale — 260 offices × daily human decisions = wildly variable outcomes. The "approve to send" fatigue problem is real. Best elements (coaching, graduated autonomy) should be incorporated into Solution 1's rollout strategy.

### Solution 5: Network Protocol
**Score**: 19/25
**Verdict**: Most architecturally elegant. The protocol concept is powerful — standardize what "good patient communication" means, then execute it everywhere. But rigidity is the achilles heel. Dental practices aren't identical, and a protocol that can't flex loses the top 5% of practices while lifting the bottom 50%. Best element (protocol specification) should inform Solution 1's playbook design.

---

## Recommendation

**Solution 1: The Comms Engine** — with enhancements from Solutions 4 and 5:

- **From Solution 4**: Graduated autonomy during rollout. Agent starts with human-in-the-loop review (first 30 days per practice), then earns full autonomy as message quality validates. TC coaching data feeds into agent personalization.
- **From Solution 5**: Protocol-driven playbook design. The "SGA Patient Communication Protocol" defines the standard cadences, triggers, message structures, and compliance requirements. The Comms Engine executes the protocol intelligently rather than rigidly.

This gives you: **autonomous execution** (agent army) built on a **standardized protocol** (playbook) with **graduated trust** (human oversight during rollout).

---

## Options for Your Decision

| Option | Description |
|--------|-------------|
| **A: The Comms Engine (recommended)** | Autonomous AI agent per practice, extends Practice Concierge, full lifecycle automation with graduated rollout |
| **B: Playbook Platform** | Deterministic workflow automation — more predictable, less adaptive |
| **C: Vendor Stack** | Buy and orchestrate existing tools — fastest but most expensive and least integrated |
| **D: Smart Playbook** | AI-coached humans — best quality but doesn't scale |
| **E: Network Protocol** | Standard protocol engine — most elegant but most rigid |
| **F: Hybrid (specify)** | Mix elements from multiple solutions |
