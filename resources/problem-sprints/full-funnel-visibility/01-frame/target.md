# Sprint Target

## Target Options

Based on the problem map, there are three distinct areas we could focus this sprint on:

---

### Target A: The Data Integration Layer (Steps 2, 10 — "Connect Everything")

**Focus**: Build the middleware that connects the 15 core platforms into a unified data model. The plumbing. Get all patient funnel data flowing into a single place so everything downstream (visibility, AI, automation) becomes possible.

**What we'd solve**: The "data lives everywhere" problem. 75 systems → 1 unified data layer.

**What we'd defer**: The executive dashboard, the AI analysis, the practice-facing interface. Those are layers on top — valuable but dependent on the data integration being solved first.

| Criterion | Score (1-5) |
|-----------|:-----------:|
| Addresses the sprint goal | 4 — Necessary but not sufficient. Integration alone doesn't create visibility. |
| Tests the hardest assumption | 5 — "Can we actually connect 15 different platforms?" is the foundational technical question. If this can't be answered, nothing else matters. |
| Provides visible value in 30 days | 2 — A data integration layer is invisible to end users. Leadership won't see a dashboard. Practices won't see an interface. The value is enabling, not visible. |
| Enables Stage 2 | 5 — Everything — dashboards, AI, automation — is built on this layer. |
| Risk of being wrong | 3 — If we focus here and the integrations prove harder than expected, we've spent the sprint on plumbing with nothing to show. |
| **Total** | **19/25** |

---

### Target B: The Executive Visibility Dashboard (Steps 10, 7, 8 — "See the Funnel")

**Focus**: Build the executive-facing real-time funnel dashboard. The view that shows lead → connected → booked → showed → produced across all 260+ practices. With drill-down capability. The thing the CEO opens every morning.

**What we'd solve**: The "30-day lag" problem. Executives can see funnel health in real-time instead of waiting for end-of-month reconciliation.

**What we'd defer**: The full data integration (MVP could start with a subset of platforms). The practice-facing interface. The AI analysis layer.

| Criterion | Score (1-5) |
|-----------|:-----------:|
| Addresses the sprint goal | 5 — This IS the sprint goal stated in the simplest form: "see the funnel in real-time." |
| Tests the hardest assumption | 3 — Assumes integration is solvable. Tests whether the funnel model itself (5 steps, drill-down) is the right abstraction for decision-making. |
| Provides visible value in 30 days | 5 — A working dashboard, even with partial data, is immediately visible and useful. Leadership can see it, react to it, and provide feedback. |
| Enables Stage 2 | 4 — The dashboard defines the data model that the AI layer and practice interface will use. But it's top-down (executive) not bottom-up (practice). |
| Risk of being wrong | 3 — If the dashboard is built but data quality is poor (because integration isn't solved), it shows garbage. "Real-time garbage" is worse than "delayed truth." |
| **Total** | **20/25** |

---

### Target C: The Practice Nerve Center (Steps 4, 5, 9 — "Do the Right Thing in 20 Minutes")

**Focus**: Build the practice-facing unified interface — the "one screen" that replaces 10 tools and tells practice staff exactly what to do in their 20 minutes/day. Prioritized action agenda, not raw data. Driven by funnel performance.

**What we'd solve**: The "20 minutes, 10 tools" problem. Practice staff get a single interface that surfaces the highest-impact actions.

**What we'd defer**: Full executive dashboard (though summary data feeds up). Full AI analysis (though basic prioritization logic is embedded). Full integration (MVP works with phone + PMS data only).

| Criterion | Score (1-5) |
|-----------|:-----------:|
| Addresses the sprint goal | 3 — Solves the practice-level visibility problem but doesn't directly address executive visibility. |
| Tests the hardest assumption | 4 — Tests the behavioral bet: will practice staff actually use a unified interface? This is where most DSO tech initiatives die. |
| Provides visible value in 30 days | 4 — Practice staff could start using a simplified version within 30 days. Behavioral change is visible. |
| Enables Stage 2 | 3 — Creates the practice engagement channel but doesn't establish the full data layer or executive view. |
| Risk of being wrong | 4 — If practices won't adopt it, you learn that fast. But if you only focus here, you miss the executive visibility that motivates the whole initiative. |
| **Total** | **18/25** |

---

## Recommendation: Target B + A Hybrid — "Real-Time Funnel Dashboard Powered by an Integration Layer"

### Why This Hybrid

The executive visibility dashboard (Target B) IS the sprint goal — it's the thing that replaces the 30-day lag and enables proactive management. But a dashboard without data is a pretty picture. The integration layer (Target A) is the foundation that makes the dashboard truthful.

**The hybrid approach:**
1. **Build the integration layer for the 3-4 most critical data sources first** — PMS (production, appointments, show rates), phone system (answer rates, call volume), and whatever lead capture exists today (even if it's just form submissions and call tracking). Don't try to connect all 15 platforms in 30 days.
2. **Build the executive dashboard on top of those 3-4 sources** — It won't be complete, but it will demonstrate the funnel model and prove the concept. Executives can see real-time data from the most important funnel steps.
3. **Define the AI analysis layer architecture** — What anomalies should the system detect? What thresholds matter? How should alerts be prioritized? This is the design, not the build.
4. **Prototype the practice-facing interface** — Use the dashboard data model to show what the "20-minute agenda" could look like. Don't build it, but storyboard it.

**Why not Target C first?**
The practice interface is critical, but it's downstream. If you build a practice interface without the data layer and dashboard, you're building another tool that doesn't connect. The integration layer → dashboard → practice interface is the right build order.

**Why not Target A alone?**
Pure integration work has no visible output. Executives need to see something. The dashboard provides the forcing function that validates whether the integration is capturing the right data.

### Target Statement

**We are targeting the executive visibility problem — specifically, the ability to see real-time patient funnel health (lead → connected → booked → showed → produced) across all 260+ practices from a single dashboard — powered by an integration layer that connects the 3-4 most critical data sources (PMS, phone, lead capture) into a unified funnel data model. If this works, it proves the data integration approach, validates the funnel model, and creates the foundation for the AI analysis layer and practice-facing interface in Stage 2.**

### Success Criteria (30-Day)
1. A working executive dashboard showing real-time funnel data from at least 10 pilot practices
2. Integration connectors for PMS (appointments, production), phone system (answer rates), and at least one lead source
3. Funnel data model documented and validated against real practice data
4. AI analysis requirements defined (anomaly types, thresholds, alert format)
5. Practice-facing interface concept (storyboard, not built)
