# PHI-Free User Journey Redesign

**Date**: 2026-03-20
**Purpose**: Accelerate development by deferring PHI dependencies to Phase B

## Overview

The original 3-workstream platform requires PHI across 21 data flows (7 high-risk). By redesigning user journeys to defer PHI, ~65% of platform features can begin development immediately without BAA procurement, PHI filters, or HIPAA compliance middleware blocking the build.

## Deliverables

- [phi-free-journeys.html](phi-free-journeys.html) — Interactive review page with redesigned journeys, phase plan, and comparison

## Three Redesigned Journeys

| Workstream | Journey | PHI Status |
|------------|---------|------------|
| WS1: Practice Platform | Practice Enrollment to Executive Visibility | PHI-free (practice/staff data only) |
| WS2: Content Engine | Brand Content Creation to Multi-Channel Distribution | PHI-free (no patient photos) |
| WS3: Culture OS | H-Score Improvement Through Culture Rituals | Already PHI-free (aggregate vendor scores) |

## Phase Plan

- **Phase A (Weeks 1-14)**: Build everything that doesn't touch PHI. Full Culture OS, Content Engine (minus patient photos), enrollment pipeline, storefronts, executive dashboards with aggregate KPIs.
- **Phase B (Weeks 15-24)**: Layer in PHI after BAAs are executed. PMS integration, patient communications, concierge agent, automation engine, patient photo pipeline, voice AI.

## Key Insight

WS3 (Culture OS) is already almost entirely PHI-free. WS2 only needs patient photos deferred. WS1 has the most significant scope change — all patient-facing features move to Phase B — but enrollment, storefronts, and aggregate dashboards are complete workflows on their own.
