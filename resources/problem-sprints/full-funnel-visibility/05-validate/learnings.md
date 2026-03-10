# Sprint Learnings: Full Funnel Visibility

## Sprint Summary

| Field | Value |
|-------|-------|
| **Sprint** | Full Funnel Visibility |
| **Date** | 2026-03-10 |
| **Problem** | 75+ disconnected tech systems across 260+ practices create 30-day visibility lag into patient funnel health |
| **Target** | All three layers: Data Integration (A) + Executive Dashboard (B) + Practice Nerve Center (C), with A+B prioritized |
| **Solution** | The Funnel OS — full-stack unified patient funnel operating system with staged delivery |
| **Outcome** | Complete architecture with 6 components, 5-stage roadmap, 5 ADRs, HTML visualization |

---

## Key Learnings

### 1. The Problem Is Integration, Not Intelligence
The hardest part of this sprint wasn't designing the AI or the dashboard — it was grappling with the data integration challenge. 15 core platforms across 260+ practices, each with different APIs, data schemas, and export capabilities. The insight: **if you solve the integration layer, the intelligence and experience layers are comparatively straightforward.** The data plumbing is 60% of the work and 80% of the risk.

### 2. The Funnel Model Is the Organizing Principle
The 5-stage patient funnel (Lead → Connected → Booked → Showed → Produced) emerged as the single most powerful organizing concept. Every metric, every alert, every action can be mapped to a funnel stage transition. This model isn't just a dashboard view — it's the data schema, the AI's monitoring framework, and the practice staff's action agenda all in one. **The funnel IS the product.**

### 3. Practice Staff Time Is the Binding Constraint
The "20 minutes a day, 10 tools" framing reframed the entire solution. Any system that adds complexity — even valuable complexity — will fail at the practice level. The Practice Nerve Center must be radically simple: 3 cards, each tappable, each with one action. The constraint forced the design toward an "agenda" pattern rather than a "dashboard" pattern. **Don't show practice staff data. Show them actions.**

### 4. The Practice Concierge Convergence Is Non-Negotiable
The Storefront sprint produced the Practice Concierge. This sprint produced the Practice Nerve Center. They share users, data, and adoption challenges. Building them as two separate tools would create the exact fragmentation problem we're solving. **ADR 4 (Nerve Center extends Concierge) may be the most important architectural decision in the sprint.** It means the two sprint outputs converge into a single practice-facing tool.

### 5. Per-Practice Baselines Are Essential
The temptation is to set network-wide thresholds (e.g., "80% phone answer rate is the standard"). But a 2-chair rural practice and a 12-chair metro practice have fundamentally different "normal." Per-practice baselines trained on each practice's own 90-day history are the only way to generate alerts that are actually meaningful. **The AI must be contextual, not standardized.**

### 6. Staged Delivery Solves the Time-to-Value Problem
The Funnel OS's biggest weakness — the full platform takes 6-12 months — was solved by borrowing the "BI tool as Stage 1 dashboard" pattern. Executives see real data in 30 days via Metabase. The custom platform replaces it in Month 3-5. **The BI tool is disposable infrastructure, not the product.** This pattern (buy temporary, build permanent) should be applied more broadly.

---

## Unresolved Questions

### Q1: Which PMS vendors to prioritize?
The architecture assumes PMS connectors for Dentrix, Eaglesoft, and Open Dental. But the actual PMS distribution across SGA's 260+ practices is unknown. **Next step: audit the PMS landscape to determine which 2-3 connectors cover 80%+ of practices.**

### Q2: Data quality from legacy systems
Some practices may have PMS data that is incomplete, inconsistent, or entered differently (e.g., appointment types coded differently). The funnel model depends on mapping PMS data to standard funnel stages. **Risk: data normalization across 260+ practices is a significant, underestimated effort.**

### Q3: AI phone agent integration depth
Sierra and Agent Agnes are mentioned, but the depth of their API integration is unknown. Can we get conversation transcripts, intent classifications, booking confirmations via API? Or only basic call metadata? **The quality of the AI phone agent data directly determines the quality of the "Connected" stage metrics.**

### Q4: Practice adoption strategy
The Nerve Center is only useful if practices use it. The adoption strategy (pilot → rollout) is defined, but the change management approach (how do you get 260 office managers to use a new tool?) is not. **This is a people problem, not a technology problem. It needs a dedicated adoption plan.**

### Q5: Cost model
The architecture document specifies a 10-person team. The infrastructure costs (ClickHouse, Kafka, AWS) are estimated but not budgeted. **A cost model mapping each stage to team size + infrastructure cost + timeline is needed before executive approval.**

---

## Connection to Other Sprints

| Sprint | Connection |
|--------|-----------|
| **Digital Storefront at Scale** | The Practice Concierge (Storefront sprint) and Practice Nerve Center (this sprint) converge into a single practice-facing tool. They share data infrastructure, UI patterns, and the practice relationship. |
| **Practice Operations (future)** | The Funnel OS provides the data foundation for practice-level operational management. A future sprint could address how individual practice P&Ls are managed through the same data layer. |

---

## Quality Validation

| Check | Status |
|-------|--------|
| All 5 folders populated with required files | ✅ |
| README.md links to all files | ✅ |
| HTML opens in browser with all sections | ✅ |
| Sprint narrative coherent (problem → framing → solutions → decision → architecture) | ✅ |
| Storyboard has 10 detailed steps covering full user journey | ✅ |
| Architecture covers all storyboard steps | ✅ |
| 6 components with clear responsibilities and interfaces | ✅ |
| 5 ADRs documenting significant decisions | ✅ |
| HTML has no external dependencies | ✅ |
