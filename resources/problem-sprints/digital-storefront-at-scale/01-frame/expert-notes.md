# Expert Notes & HMW Analysis

## How Might We Notes

### Theme 1: Onboarding Speed (Steps 1-3)
1. HMW reduce asset access from weeks to hours by integrating digital onboarding into the deal process itself?
2. HMW pre-populate practice digital profiles using publicly available data (website scraping, GMB data, social profiles) before we even ask the practice for anything?
3. HMW create a "digital intake agent" that walks practice staff through asset handoff in a single conversational session?
4. HMW eliminate the password recovery problem by building new accounts from scratch when recovery takes longer than creation?
5. HMW use the deal team as a forcing function — making digital asset handoff a closing checklist item?

### Theme 2: Template-First Quality (Steps 3-4)
6. HMW build 3-5 specialty-specific website templates (GP, Perio, Full Arch, Pediatric) that are 90% done and only need practice-specific content?
7. HMW come in with such a forceful, opinionated design perspective that practices take a back seat and just choose between 2-3 options?
8. HMW auto-generate practice websites from structured data (practice name, specialty, team, photos, colors) without any design conversation?
9. HMW handle the 10% of practices that care deeply about their site while not letting them slow down the 90% who don't?
10. HMW ensure every template is mobile-optimized, SEO-structured, chatbot-enabled, and referral-optimized by default?

### Theme 3: Automation-First Curation (Steps 7-9)
11. HMW automate social media publishing across 260+ practices so the central team only reviews exceptions?
12. HMW detect practice-level changes (staff departure, service change) automatically instead of waiting for someone to tell us?
13. HMW build event-triggered cascading updates so one change notification propagates across all 7+ digital touchpoints?
14. HMW create a "practice health score" that automatically flags when a storefront is decaying (no posts in 30 days, reviews dropping, etc.)?
15. HMW use the hub-and-spoke content model to generate 70-80% of content centrally while allowing 20-30% local customization?

### Theme 4: Practice Engagement (Step 7-8)
16. HMW build a 24/7 agent interface that practices actually want to use instead of ignoring like emails?
17. HMW make the agent opinionated — guiding practices toward the right answer instead of just doing whatever they ask?
18. HMW handle practice pushback on design/content decisions with data-driven reasoning built into the agent?
19. HMW reduce the "nuisance factor" so practices see HQ marketing as a service, not a burden?
20. HMW design the agent so it proactively suggests actions ("Dr. Johnson left — should we also update signage and schedule a social post?") instead of just responding to requests?

### Theme 5: Executive Visibility (Step 10)
21. HMW build a real-time network health dashboard that requires zero manual data entry?
22. HMW give executives a single view showing which practices are at baseline, optimized, or falling behind?
23. HMW create automated alerts when a practice drops below digital health thresholds?
24. HMW measure ROI of digital investment per practice without creating a reporting burden?

### Theme 6: Scale Architecture (Cross-cutting)
25. HMW design this so acquiring 20-50 practices at once doesn't break the system?

---

## Theme Summary

| Theme | HMW Count | Priority |
|-------|-----------|----------|
| Onboarding Speed | 5 | HIGH — current bottleneck, blocks everything |
| Template-First Quality | 5 | HIGH — determines setup speed and consistency |
| Automation-First Curation | 5 | HIGH — the 365-day problem, the scale problem |
| Practice Engagement | 5 | HIGH — behavioral adoption is a make-or-break risk |
| Executive Visibility | 4 | MEDIUM — important but downstream of the above |
| Scale Architecture | 1 | HIGH — cross-cutting constraint |

---

## Expert Insights

### DSO Operations Expert
**Insight**: In multi-location healthcare, the practices that perform best digitally are the ones where the corporate team does 95% of the work and the practice does 5% (approving, providing photos, flagging changes). The worst-performing networks are ones where corporate provides "tools and resources" and expects practices to self-serve. Practices are running clinical businesses — they will never prioritize digital marketing. The system must be designed so the default state (doing nothing) still results in a functional, updated storefront.

**Sprint Question Implication**: Directly validates SQ2 and SQ3 — the system must be automation-first with practice engagement as enhancement, not requirement.

### Marketing Technology Expert
**Insight**: Template-based site generation at scale is a solved problem in other industries (real estate, hospitality, franchises). The pattern is: structured data model → template engine → auto-generated sites with override capabilities. The key innovation needed here is the intake/data capture layer — getting the structured data that populates the template. Conversational AI for intake (like what SGA has started with the GPT-based workbook) is the right approach, but it needs to be integrated into a pipeline, not a one-off tool.

**Sprint Question Implication**: Validates SQ1 — the technical path exists. The risk is in the data capture pipeline, not the site generation.

### Dental Industry Expert
**Insight**: SGA's competitive landscape shows that no major DSO has solved practice-level digital presence at scale. Heartland (1,800 practices) has a reputation for corporate control that stifles local identity. Aspen (1,300) uses a franchise model with uniform branding that patients see through. PDS has owner-operators managing their own digital presence with wildly inconsistent results. If SGA cracks "best-in-class digital presence at scale while preserving local brand identity," it becomes a genuine competitive moat — both for patient acquisition AND practice acquisition (showing prospects "here's what your digital presence looks like when you join us").

**Sprint Question Implication**: This isn't just an operational efficiency play — it's a competitive differentiator and acquisition tool. Amplifies the importance of SQ1 and SQ2.

### AI/Automation Expert
**Insight**: The practice-facing agent is the right abstraction layer. Rather than giving practices access to a dashboard, CMS, social media scheduler, and review management tool (all of which they'll ignore), give them a single conversational interface that handles everything. The agent routes to the right system behind the scenes. The key design decision is the agent's personality — it needs to be opinionated (SGA's "Sage + Caregiver" brand personality), proactive (suggesting actions, not just responding), and trusted (providing reasoning for recommendations, not just orders). This is exactly the pattern described in the SGA chatbot flow design, extended from patient/prospect-facing to practice-facing.

**Sprint Question Implication**: Directly addresses SQ3. The agent's value proposition to the practice is "I save you time and make you look great without you having to think about it."

### Change Management Expert
**Insight**: The cascading update problem (doctor leaves → 7+ touchpoints need updating) is a classic event-driven architecture challenge. The solution is: (1) define the event taxonomy (staff change, service change, hours change, etc.), (2) map each event to its downstream effects, (3) build automated propagation with human approval gates where needed. The detection layer is the hard part — practice-level changes are often communicated informally. The agent interface is the detection mechanism: if the practice tells the agent about a change, the cascade begins. If they don't tell anyone, integrate with HR/payroll systems (iSolved, which SGA uses) to detect staff changes automatically.

**Sprint Question Implication**: Validates SQ4. The detection problem is solvable through two channels: agent-reported (practice tells the agent) and system-detected (HR/payroll integration).
