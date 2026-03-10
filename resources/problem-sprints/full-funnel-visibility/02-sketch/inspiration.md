# Inspiration Research

## 1. Dental Intelligence (dental analytics platform)
- **What they do**: Connects to 90+ PMS systems, pulls production, scheduling, hygiene, and case acceptance data into a unified dashboard. AI identifies "morning huddle" priorities for each practice.
- **What's relevant**: They solved the PMS integration problem at scale — proving it's possible to pull data from dozens of different PMS vendors through a single middleware layer. Their "morning huddle" feature is essentially a daily action agenda generated from practice data.
- **What's missing**: Phone/lead funnel visibility. They focus on in-practice operations (production, scheduling, hygiene recall) not the pre-appointment funnel. No lead tracking, no phone answer rate monitoring, no AI call analysis.

## 2. Liine (dental call tracking + AI analysis)
- **What they do**: AI-powered call tracking specifically for dental. Listens to every call, determines if it was a new patient, whether the appointment was booked, and why calls convert or don't. Provides conversion coaching.
- **What's relevant**: They solve Step 4 (Phone Answered) with deep intelligence. Their AI doesn't just track answer rates — it analyzes WHY calls don't convert (front desk said something wrong, patient had insurance question that wasn't answered, etc.).
- **What's missing**: Only covers phone calls. No form tracking, no chat, no downstream funnel (show rates, production). It's a point solution for one step of the funnel.

## 3. Patient Prism (real-time new patient call tracking)
- **What they do**: AI listens to calls in real-time. When a new patient call doesn't convert, the system alerts the office within minutes with coaching on how to call back and recover the patient.
- **What's relevant**: Real-time intervention based on AI analysis — exactly the pattern the sprint goal describes. The "recover lost patients" model is a concrete example of AI-driven automation acting on funnel data.
- **What's missing**: Same limitation as Liine — phone-only, single funnel step. Doesn't connect to PMS data, production, or the full funnel.

## 4. Weave (dental communication platform)
- **What they do**: Unified phone, text, email, reviews, and payments for dental practices. When a patient calls, the front desk sees their full history (upcoming appointments, overdue balances, recall status) on screen.
- **What's relevant**: They unified the communication layer — phone, text, email through one system. Their patient context popup (showing history when a call comes in) is a model for how the funnel data should surface at the practice level.
- **What's missing**: No network-level visibility. Designed for individual practices, not DSOs managing 260+ locations. No AI analysis layer. No executive dashboard.

## 5. Agent Agnes (AI phone agent + PMS integration)
- **What they do**: AI agent that answers phone calls, integrates with practice management software, and triggers actions based on conversation content (reschedule missed appointments, confirm upcoming visits, handle outbound recall calls).
- **What's relevant**: Already mentioned by the user as a reference. This is the model for AI agents that sit on top of existing practice software and take actions. The conversation → action pattern is exactly what the Practice Nerve Center needs.
- **What's missing**: It's a point solution for phone engagement. Doesn't create the visibility layer or the executive dashboard. The data it generates (call outcomes, actions taken) needs to feed INTO a unified funnel view.

## 6. Salesforce Health Cloud (healthcare CRM)
- **What they do**: Healthcare-specific CRM with patient journey mapping, care coordination, and provider network management. Connects to EHR/PMS systems. Provides executive-level analytics across patient populations.
- **What's relevant**: The architectural pattern — CRM as the unified data layer that connects provider systems, patient data, and business analytics. Their "patient journey" model maps directly to the patient funnel concept.
- **What's missing**: Massively over-engineered for this use case. 12-18 month implementation timeline. Expensive. Designed for health systems, not DSOs. Would require extensive customization.

## 7. Domo / Looker / Power BI (BI platforms with healthcare connectors)
- **What they do**: Business intelligence platforms that connect to hundreds of data sources, build dashboards, and enable self-service analytics. Some have pre-built healthcare/dental connectors.
- **What's relevant**: The visualization layer. These platforms have solved the "connect to many sources, show one dashboard" problem. Pre-built connectors for common platforms accelerate integration.
- **What's missing**: They're tools, not solutions. Someone still has to define the data model, build the integrations, design the dashboards, and maintain them. No AI analysis built in (though some have ML features). No practice-facing action layer.

## 8. Samsara (fleet/operations real-time visibility)
- **What they do**: IoT platform for physical operations — real-time visibility into vehicle fleets, equipment, facilities. Dashboard shows live status of every asset. Alerts when something is off. Automated workflows for common issues.
- **What's relevant**: The pattern is identical: "I manage 260+ distributed locations and I need to see real-time health of each one." Samsara's approach — sensor data → unified platform → real-time dashboard → automated alerts → action — is the exact architecture this sprint needs, translated from physical operations to patient funnel operations.
- **What's missing**: Different domain entirely. But the architectural pattern transfers perfectly.

## 9. HubSpot Operations Hub (RevOps data unification)
- **What they do**: Connects CRM data with third-party apps, syncs data bidirectionally, provides data quality automation, and creates unified reporting across marketing → sales → service.
- **What's relevant**: The "RevOps" concept — unified visibility from marketing lead through closed deal through customer success — is the exact same funnel pattern. Their data sync approach (bidirectional, automated, quality-checked) is a model for the integration layer.
- **What's missing**: Not dental-specific. Would need significant customization. The dental PMS ecosystem is more fragmented than typical SaaS stacks.

## 10. Practice by Numbers (dental analytics, acquired by Planet DDS)
- **What they do**: Dental-specific analytics platform that connects to PMS data and provides KPI dashboards, goal tracking, and practice performance benchmarks.
- **What's relevant**: Dental-specific data model and KPI framework. They've already defined what "good" looks like for production, case acceptance, hygiene recall, etc. Their benchmarking data across dental practices is valuable for establishing the AI layer's baselines.
- **What's missing**: Limited to PMS data. No phone/lead funnel. No multi-practice executive view designed for DSOs at 260+ scale.

---

## Key Patterns Across Inspiration

1. **The integration layer is solvable** — Dental Intelligence connects to 90+ PMS systems. The technology exists. The question is scope and speed.
2. **Phone/call AI is mature** — Liine, Patient Prism, Agent Agnes all prove that AI call analysis and real-time intervention work in dental. This doesn't need to be invented.
3. **No one does full-funnel** — Every solution covers 1-2 steps of the funnel. Nobody connects lead gen → phone → booking → show → production in a single view. This is the gap.
4. **The Samsara pattern applies** — Real-time visibility into distributed operations with automated alerts and actions. Same architecture, different domain.
5. **Practice-facing interface is the adoption key** — Weave's caller context popup, Dental Intelligence's morning huddle, Agent Agnes's action-based automation. The practice interface must be action-oriented, not data-oriented.
6. **Buy + build + integrate** is the realistic approach — No single vendor solves this. The solution is middleware that unifies best-of-breed point solutions.
