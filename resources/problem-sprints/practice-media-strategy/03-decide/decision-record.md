# Decision Record — Practice + Brand Media Strategy

## Decision: Solution 5 — The Content Concierge (Hybrid Phased Approach)

**Date**: 2026-03-10
**Decision maker**: Chad Hancock
**Facilitator recommendation**: Solution 5

---

## What Was Decided

Build **The Content Concierge** — a phased, hybrid system that starts with SMS/WhatsApp prompts to prove behavior change, then scales into a full mobile app with AI production pipeline, and layers in passive capture hardware over time.

The system extends the Practice Concierge ecosystem from the digital storefront sprint, adding a media arm that handles content capture, production, and distribution for both practice-level (B2C patient acquisition) and SGA brand-level (B2B dentist acquisition/recruitment) content.

## Why This Over the Alternatives

| Solution | Why Not (or Why Partially) |
|----------|---------------------------|
| 1: Content Engine (full app first) | Too slow to start (4-6 months). Biggest risk is behavior, not tech. Don't build the app until you've proven champions will capture. |
| 2: SMS Coach (SMS only) | Good starting point but limited long-term. Can't show scoreboards, rich capture guidance, or dashboards via SMS. Incorporated as Phase 1. |
| 3: Media Playbook (human editors) | Doesn't scale linearly. More practices = more editors. Incorporated as quality backstop during AI ramp-up. |
| 4: Passive Capture (hardware-first) | Highest upfront cost, longest timeline, HIPAA complexity. Incorporated as Phase 3 layer for top practices only. |
| **5: Content Concierge (hybrid)** | **Starts fast (weeks), de-risks behavior change, scales to full platform, serves both audiences, connects to Practice Concierge ecosystem** |

## Key Technical Decision: Sikka ONE API for PMS Integration

Background research confirmed that **Sikka ONE API** connects to ~90% of dental PMS systems in the US/Canada (400+ systems including Dentrix, Eaglesoft, Open Dental). This is the critical enabler for appointment-triggered content capture prompts. Using Sikka eliminates the need to build individual PMS integrations.

## Key Market Insight: Clear Whitespace

The competitive landscape confirms no existing platform combines:
- Appointment-triggered capture prompts (PMS → notification)
- Guided mobile capture with templates
- AI auto-processing (color correction, formatting, captioning)
- Multi-channel distribution with playbook-based routing

Existing tools handle distribution (SOCi, Sprout) or processing (OpusClip, Cloudinary) but nobody owns the capture layer. This is the gap.

## Constraints Accepted

1. Phase 1 uses SMS (limited UX) to test behavior before building the app
2. Human editors needed during AI pipeline ramp-up (Phase 1-2 cost)
3. PMS integration depth is progressive — batch sync first, real-time later
4. Hardware (passive capture) only at top 10 implant practices, only in Phase 3
5. Full network rollout (260+ practices) takes 6+ months
