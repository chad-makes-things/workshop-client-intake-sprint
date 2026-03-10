# Target Recommendation

## Three Candidate Targets

### Target A: Automated Treatment Follow-Up Engine
**Map area**: Treatment Presentation → Follow-up → Close
**Score**: 23/25

### Target B: Lead Response & Capture System
**Map area**: Lead arrives (call/form) → Response → Booked
**Score**: 20/25

### Target C: Full Lifecycle Playbook (All Stages) — SELECTED
**Map area**: Entire patient communication journey — lead response, treatment follow-up, rebooking, reactivation
**Score**: 19/25

---

## User Selection: Target C — Full Lifecycle Playbook

**User rationale**: Robust DevOps team can handle the full scope. The playbook will be comprehensive from the start and expanded during development. All stages from lead capture through reactivation should be covered in a single coherent system.

**Reframe**: The lower feasibility score (2/5) that brought Target C to 19/25 was based on a single sprint's design capacity. The user's dev team capacity changes this calculus — the sprint designs the full system, the dev team builds it in stages. The architecture must be modular so each stage (lead response, treatment follow-up, rebooking, reactivation) can be developed and deployed independently while sharing the same underlying automation engine.

**Sprint will design**: A complete patient communications automation playbook covering:
1. **Lead Response** — Inbound calls, form fills, missed call recovery, lead nurture sequences
2. **Treatment Follow-Up** — Post-presentation cadences (3/7/14/30-day), high-value case escalation
3. **Rebooking** — Checkout enforcement, automated reminders, gap recovery
4. **Database Reactivation** — Lapsed patient outreach, segmented re-engagement campaigns
5. **Cross-cutting**: HIPAA compliance, PMS data triggers, personalization, staff 20-min weekly touchpoint, network-wide analytics

**Integration requirement**: Must connect to Practice Concierge (agent interface) and Funnel OS (data/visibility layer) from prior sprints.
