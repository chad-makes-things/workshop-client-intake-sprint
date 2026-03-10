# Epic: The Content Concierge — Practice Media Capture, Production & Distribution

**Sprint**: Practice Media Strategy
**Product Manager**: SGA Dental Partners — Technology & Marketing
**Date**: 2026-03-10
**Status**: Draft

---

## Epic Description

The Content Concierge is a media capture, production, and distribution platform that solves the single biggest bottleneck in SGA's content strategy: **practices never capture content**, despite having strategy, channels, and audiences ready.

The system connects practice management data (Sikka ONE API) to a smart prompt engine that knows what to capture and when. It provides a zero-friction capture interface (SMS in Phase 1, mobile app in Phase 2), processes raw content through an AI production pipeline, and distributes finished assets across practice and brand channels automatically.

Five playbooks serve the full SGA network:
1. **Implant/Cosmetic** — 10–15 high-value practices; before/afters, testimonials, procedure videos
2. **Specialty** (ortho, perio, oral surgery) — 20–30 practices; case documentation, specialist authority content
3. **Growth GP** — 40–60 multi-location practices; patient education, team content, procedure variety
4. **Standard GP** — 150–180 practices; relationship-first content, community presence
5. **SGA Brand** — Leadership (Jensen, Ellingson, Wong, Olson); LinkedIn authority, B2B, recruitment

The platform integrates with the broader SGA ecosystem:
- **Practice Concierge** (Digital Storefront sprint) — Processed content auto-populates practice websites
- **Funnel OS** (Full Funnel Visibility sprint) — Content volume + engagement feeds into funnel attribution
- **Patient Comms Engine** (Patient Comms sprint) — Testimonial content triggers patient re-engagement flows

---

## Business Objectives

| # | Objective | Current State | Target State |
|---|-----------|---------------|--------------|
| BO-1 | Activate content capture across 260+ practices | ~5% of practices capture content consistently | 80% of high-value practices capturing weekly by Phase 3 |
| BO-2 | Eliminate content production bottleneck | Manual editing by 1-2 people; days to publish | AI pipeline produces publish-ready content within 2 hours |
| BO-3 | Distribute to all channels without manual work | Each publish requires manual upload to each platform | One capture → auto-distributed to 3–6 platforms |
| BO-4 | Fuel SGA brand content machine | Leadership content sporadic, 1-2 pieces/month | 4–8 brand content pieces/week across leadership team |
| BO-5 | Build content library for paid media and sales enablement | Before/afters scattered, no system | Organized library of 500+ practice-approved assets tagged for paid media use |
| BO-6 | Prove content drives patient revenue | "Content is good" belief; no attribution | Content → engagement → consultation booking attribution within 6 months |

---

## Success Metrics

| Metric | Baseline | Target | Measurement | Timeline |
|--------|----------|--------|-------------|----------|
| Practice content participation rate | ~5% active weekly | 80% of Playbook 1–2 practices, 50% Playbook 3 | Weekly captures ≥ target per playbook | Phase 2 (Month 4) |
| Captures per practice per week | ~0 | Playbook 1–2: 5/wk; Playbook 3: 3/wk; Playbook 4: 2/wk | PostgreSQL capture_assignments completion rate | Phase 2 |
| Time from capture to publish | 48–72 hours (manual) | <2 hours (Phase 2, AI pipeline) | Timestamp: captured_at → Distribution.published_at | Phase 2 |
| Content pieces produced per capture | 1 (manual post) | 4–10 (AI multi-format) | ProcessedAsset count per Capture | Phase 2 |
| Channel coverage | 1–2 channels per practice | 3–5 channels per practice | Distribution records per practice per week | Phase 2 |
| Brand content volume (SGA) | ~2 pieces/month | 20+ pieces/week | Published Distribution records, account=SGA brand | Phase 2 |
| Content → consultation attribution | 0% (no tracking) | 5% of consultations attributed to content | UTM tracking + booking source analysis | Phase 4 |
| Paid media asset library | 0 tagged assets | 500+ assets tagged `paid_media_eligible` | ContentLibrary.tag = 'paid_media_eligible' | Phase 3 |

---

## Staged Delivery

| Phase | Timeline | Scope | Practices |
|-------|----------|-------|-----------|
| **Phase 1** | Weeks 1–6 | SMS/WhatsApp capture bot + Sikka ONE sync + S3 storage + Airtable review queue + human editors + manual distribution | 10–15 pilot (Playbook 1–2) |
| **Phase 2** | Months 2–4 | React Native app + AI photo/video pipeline + automated distribution + HQ dashboard + scorecard | 40–65 (Playbooks 1–3) + SGA brand |
| **Phase 3** | Months 4–6 | Full 260+ practice rollout + testimonial kiosks + passive cameras + AI auto-approval + advanced analytics | All 260+ practices |
| **Phase 4** | Months 6–12 | Content intelligence ML model + predictive prompt optimization + self-service onboarding | All practices + franchise-ready |

---

## User Stories

### Epic 1: Prompt Engine

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| PE-1 | As a content champion, I receive a daily SMS brief at 7:30 AM listing what to capture today based on tomorrow's appointments | P0 | Phase 1 |
| PE-2 | As the HQ content team, I can configure which procedure codes trigger capture assignments for each practice | P0 | Phase 1 |
| PE-3 | As a content champion, I am notified at 30, 60, and 90 days post-treatment to capture testimonial follow-ups | P1 | Phase 1 |
| PE-4 | As SGA brand leadership, I receive a weekly LinkedIn topic prompt with a script outline | P1 | Phase 2 |
| PE-5 | As a content champion, I receive Event Mode prompts when a 4Front or Pikos event is happening at my practice | P2 | Phase 2 |

### Epic 2: Capture Layer

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| CL-1 | As a content champion, I can capture and submit content by replying to an SMS with a photo or video | P0 | Phase 1 |
| CL-2 | As a content champion, I can use WhatsApp instead of SMS for easier media sharing | P1 | Phase 1 |
| CL-3 | As a content champion, I receive immediate confirmation when my capture is received | P0 | Phase 1 |
| CL-4 | As a content champion, I see my streak and weekly capture count in every confirmation SMS | P1 | Phase 1 |
| CL-5 | As a content champion, I can use a mobile app with guided camera overlays and in-app assignments | P0 | Phase 2 |
| CL-6 | As a patient, I can self-record a testimonial at the checkout kiosk after my visit | P2 | Phase 3 |
| CL-7 | As a practice, passive cameras in the operatory capture procedure content automatically | P3 | Phase 3 |

### Epic 3: Production Pipeline

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| PP-1 | As the HQ content team, I can view all incoming captures in an Airtable review queue | P0 | Phase 1 |
| PP-2 | As an editor, I can mark captures as approved, in production, or rejected with feedback | P0 | Phase 1 |
| PP-3 | As the system, I automatically resize and reformat approved photos for each target platform | P0 | Phase 2 |
| PP-4 | As the system, I generate AI-written captions, hashtags, and descriptions per platform | P0 | Phase 2 |
| PP-5 | As the system, I apply dental-specific color correction and smart crop to before/after photos | P1 | Phase 2 |
| PP-6 | As the system, I auto-caption videos with burned-in text and generate transcript | P1 | Phase 2 |
| PP-7 | As the system, I detect highlight moments in videos and generate 15s/30s/60s clips | P1 | Phase 2 |
| PP-8 | As the system, I apply practice branding (logo, colors) to processed assets | P1 | Phase 2 |
| PP-9 | As the HQ team, content types with >95% approval rate are auto-approved with spot-check only | P2 | Phase 3 |

### Epic 4: Review & Approval

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| RV-1 | As an editor, I can review content in Airtable with a clear status workflow | P0 | Phase 1 |
| RV-2 | As an editor, I can send feedback to the capturing champion when rejecting content | P1 | Phase 1 |
| RV-3 | As the HQ team, I can review all practice content in a kanban queue in the HQ dashboard | P0 | Phase 2 |
| RV-4 | As the HQ team, I can edit AI-generated captions before approval | P0 | Phase 2 |
| RV-5 | As the HQ team, I can star high-quality content for premium distribution treatment | P1 | Phase 2 |

### Epic 5: Distribution Engine

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| DI-1 | As the HQ team, approved content is manually posted to practice channels via Sprout Social/Later | P0 | Phase 1 |
| DI-2 | As the system, I automatically distribute approved content to Instagram, Facebook, YouTube per playbook rules | P0 | Phase 2 |
| DI-3 | As the system, I schedule posts at AI-determined optimal times per practice | P1 | Phase 2 |
| DI-4 | As the system, I route before/after content to the Practice Concierge digital storefront gallery | P1 | Phase 2 |
| DI-5 | As the system, I tag appropriate content as paid_media_eligible and push to the content library | P1 | Phase 2 |
| DI-6 | As the system, leadership video content is scheduled to LinkedIn personal and company pages | P0 | Phase 2 |

### Epic 6: Scorecard & Analytics

| ID | Story | Priority | Phase |
|----|-------|----------|-------|
| SC-1 | As a content champion, I see my weekly capture count and streak in each confirmation SMS | P0 | Phase 1 |
| SC-2 | As a content champion, I receive a weekly SMS recap every Monday | P1 | Phase 1 |
| SC-3 | As a content champion, I see my scorecard in the mobile app | P0 | Phase 2 |
| SC-4 | As the HQ team, I see each practice's content health score in the HQ dashboard | P0 | Phase 2 |
| SC-5 | As the HQ team, I see network-wide content analytics tied to business outcomes | P1 | Phase 3 |
| SC-6 | As the HQ team, I can attribute specific content pieces to patient consultation bookings | P2 | Phase 4 |

---

## Dependencies

| Dependency | Owner | Required For | Status |
|------------|-------|-------------|--------|
| Sikka ONE API access + credentials | SGA IT / Sikka | Phase 1 | Not started — initiate immediately |
| Dentrix partner agreement | SGA Legal / Dentsply | Phase 2 | 4–6 week approval cycle |
| Eaglesoft partner agreement | SGA Legal / Patterson | Phase 2 | 4–6 week approval cycle |
| Meta (Instagram/Facebook) Business accounts per practice | SGA Marketing per practice | Phase 2 | High volume — batch onboarding workflow needed |
| HIPAA BAA: Twilio | SGA Legal | Phase 1 | Available via Twilio console |
| HIPAA BAA: Airtable | SGA Legal | Phase 1 | Enterprise tier required |
| HIPAA BAA: Cloudinary | SGA Legal | Phase 2 | Available via contract |
| Content champion identification (10–15 pilot practices) | SGA Operations | Phase 1 | Requires ops coordination |
| Patient consent form design and approval | SGA Legal / Compliance | Phase 1 | Critical — no captures without consent |
| Practice Concierge API (storefront integration) | Digital Storefront sprint team | Phase 2 | Storefront must be live |

---

## Constraints & Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Champions don't respond to SMS prompts | High | High | Daily brief is low-friction, opt-out-able. Start with most motivated practices. Gamification (streak) drives habit. Weekly recap shows impact. |
| Photo quality too low for content use | Medium | Medium | Guidance text in SMS coach. Phase 2 app has framing overlays. Editors can reject with coaching feedback. |
| HIPAA consent workflow slows capture | Medium | High | Phase 1: collect consent manually via paper, log in system. Phase 2: digital consent in app. Never capture without consent record. |
| PMS API coverage gaps (non-Sikka practices) | Low | Medium | Sikka ONE covers 400+ PMS systems. Manual assignment fallback via content calendar for non-PMS practices. |
| Airtable workflow not scalable beyond Phase 1 | High | Medium | Intentional — Airtable is Phase 1 only. Phase 2 HQ dashboard replaces it entirely. Set expectation with editors. |
| Social platform API policy changes (Meta) | Medium | High | Build platform integrations modularly. One API change shouldn't affect others. Monitor Meta policy changes. |

---

## Phase 1 Acceptance Criteria

Phase 1 is complete when:

1. **10–15 pilot practices** are active with champions enrolled and receiving daily briefs
2. **Daily brief SMS** fires at 7:30 AM local time for each active practice with correct appointment-based assignments
3. **Champion capture** via MMS or WhatsApp works end-to-end: photo received → S3 stored → Airtable record created → confirmation SMS sent, all within 30 seconds
4. **Airtable review queue** is operational with 2 editors reviewing captures and publishing manually
5. **Post-op follow-up** prompts firing at 14, 30, and 90 days for high-value procedures
6. **Admin dashboard** allows HQ team to add practices, configure champions, and monitor capture health without developer intervention
7. **Weekly capture rate** for pilot practices ≥ 50% of playbook target by Week 6
