# Goal & Sprint Questions

## Long-Term Goal (30-Day)
Within 30 days, design and validate a fully automated patient communications playbook that can be deployed across SGA's 260+ dental practices — covering lead response, treatment follow-up, rebooking, and database reactivation — reducing manual communication effort to ~20 minutes/week per practice while increasing treatment acceptance rates by 30%+ and ensuring zero leads go unworked for more than 1 hour.

### Goal Validation
- **Specific**: Automated playbook covering 4 defined communication stages across 260+ practices
- **Ambitious**: From fragmented/manual to fully automated with measurable conversion improvements
- **Measurable**: 20 min/week human time, 30%+ treatment acceptance lift, <1 hour lead response SLA
- **Mechanism-focused**: Targets the automation gap, not just awareness of the problem

---

## Sprint Questions

### SQ1: Can we automate treatment follow-up without it feeling "salesy" to treatment coordinators?
- **Risk Level**: HIGH
- **Why it matters**: 70% of treatment coordinators resist follow-up because it "feels salesy." If the automation runs FROM them but WITHOUT their active involvement, does it solve the resistance problem or create a new trust/authenticity problem?
- **Testable via**: Would a treatment coordinator accept a system that sends follow-up texts under their name if they can review/customize the template?

### SQ2: Can a standardized playbook work across 260+ practices with different patient mixes, specialties, and staff cultures?
- **Risk Level**: HIGH
- **Why it matters**: A cosmetic-focused practice in Scottsdale has a fundamentally different patient communication need than a family practice in rural Mississippi. One-size-fits-all playbooks fail. But 260 custom playbooks are unscalable.
- **Testable via**: Can we define a base playbook with practice-type variants that covers 90%+ of communication scenarios?

### SQ3: Will patients respond positively to automated communications, or will it feel impersonal and damage the practice's relationship?
- **Risk Level**: MEDIUM
- **Why it matters**: Dental practices depend on personal relationships. A patient who gets a robotic "time to rebook" text may feel like a number. But a patient who never hears from the practice at all is worse.
- **Testable via**: Do automated texts with personalization (patient name, specific treatment discussed, provider name) achieve response rates comparable to manual outreach?

### SQ4: Can we get data out of disconnected PMS/EHR systems reliably enough to trigger automated communications?
- **Risk Level**: HIGH
- **Why it matters**: The automation depends on knowing: who visited, what treatment was presented, when appointments are, who's overdue. This data lives in 75+ different PMS systems. If we can't reliably extract trigger events, the entire playbook fails.
- **Testable via**: Can we define a minimum viable data set (5-7 trigger events) that can be extracted from the top 5 PMS systems covering 80%+ of practices?

### SQ5: Can this system integrate with or extend the Practice Concierge and Funnel OS architectures already designed?
- **Risk Level**: LOW
- **Why it matters**: SGA is building a Practice Concierge (agent-first interface) and Funnel OS (data/visibility layer). This patient comms system should be the EXECUTION layer — not a third parallel system. Architectural coherence matters for long-term maintainability and practice adoption.
- **Testable via**: Can we map every automated communication trigger to a Funnel OS data event and every human touchpoint to a Practice Nerve Center agenda item?
