# Validation & Learnings — Practice + Brand Media Strategy

## Sprint Summary

**Problem**: SGA Dental Partners has 260+ locations targeting 15% growth, but practices never capture content (before/afters, testimonials, video). The entire content-to-patient pipeline is starved because of broken human behavior at the practice level. The same gap exists at the SGA brand level — leadership has content to share but no system to make it simple.

**Solution**: The Content Concierge — a phased, hybrid system that starts with SMS prompts (test behavior cheaply), scales into a mobile app with AI production pipeline, and layers in passive capture hardware over time. Extends the Practice Concierge ecosystem.

**Key insight**: The biggest risk isn't technology — it's behavior change. Starting with SMS isolates that variable.

---

## Validation Against Success Criteria

| Criteria | Status | Evidence |
|----------|--------|----------|
| Full pipeline designed (case ID → alert → capture → upload → production → distribution) | ✅ | Architecture doc covers all 5 layers + data model |
| Mobile capture simpler than anything that exists | ✅ | Camera-first UX, guided overlays, <60 second capture. HTML prototype demonstrates. |
| PMS integration for proactive alerts | ✅ | Sikka ONE API identified — connects to 400+ dental PMS systems (~90% coverage). Appointment-triggered prompts designed. |
| Clear playbooks for practice types | ✅ | 5 playbooks defined (Implant/Cosmetic, Specialty, Growth GP, Standard GP, SGA Brand) |
| HTML prototype demonstrates champion experience | ✅ | 3-view prototype: HQ Dashboard, Mobile App (3 screens), Pipeline View |
| Storyboard traces real scenario | ✅ | 3 scenarios: Sarah's full-arch (practice), Jensen's thought leadership (brand), 4Front symposium (event) |

---

## Key Research Findings

### Market Whitespace Confirmed
- **No existing platform** combines PMS-triggered prompts + guided mobile capture + AI processing + multi-channel distribution for healthcare
- Closest competitors: Rallio (employee capture, not healthcare-specific), SOCi (distribution only), Greenfly (enterprise capture, not dental)
- The capture layer is the clear gap — everyone handles distribution, nobody handles capture

### Critical Enabler: Sikka ONE API
- Connects to ~90% of dental PMS systems (400+ systems including Dentrix, Eaglesoft, Open Dental)
- Can read appointment schedules, patient data, procedure codes
- Eliminates the need to build individual PMS integrations — single API for entire network
- This is the technical foundation for appointment-triggered capture prompts

### AI Pipeline Components Available
- Photo processing: Cloudinary (color correction, smart crop, overlays)
- Video cutting: OpusClip API (highlight detection, short-form generation)
- Transcription: Whisper API (video → text → captions)
- Text generation: Claude API (captions, hashtags, descriptions)
- All available as APIs — no need to build AI from scratch

---

## Architecture Decisions

| Decision | Rationale |
|----------|-----------|
| **Phase with SMS first** | Tests behavior change ($0 app development cost). If champions won't respond to texts, they won't use an app. |
| **React Native for mobile** | Cross-platform (iOS + Android) from single codebase. OTA updates via Expo. |
| **Sikka ONE API for PMS** | 90% coverage. One integration vs. building connectors for each PMS. |
| **Cloudinary for photo processing** | Managed service, dental-specific transformations possible, scales without ops burden. |
| **Claude API for text generation** | Best quality for nuanced dental/medical content captions. Context-aware. |
| **Playbook system** | Different practice types need different content strategies. Config-driven, not code-driven. |
| **Human editors as Phase 1-2 backstop** | AI production quality improves over time. Humans ensure brand safety during ramp. |

---

## Cost Estimates

### Phase 1: SMS Coach (Weeks 1-6)
| Item | Cost |
|------|------|
| Twilio SMS/WhatsApp setup | $2K |
| Sikka ONE API integration (dev) | $8K |
| S3 + basic cloud infra | $500/mo |
| 2 contract editors | $4K/mo |
| Sprout Social (manual distribution) | $500/mo |
| **Phase 1 Total** | ~$15K setup + $5K/mo |

### Phase 2: Mobile App + AI Pipeline (Months 2-4)
| Item | Cost |
|------|------|
| React Native app development | $45K |
| AI pipeline (Lambda, Cloudinary, Whisper, OpusClip, Claude) | $15K |
| Next.js HQ dashboard | $12K |
| Distribution engine (platform API integrations) | $8K |
| Cloud infrastructure | $3K/mo |
| AI API costs (processing) | $4K/mo |
| Editor team (reduced — AI handles 60%+) | $3K/mo |
| **Phase 2 Total** | ~$80K build + $10K/mo |

### Phase 3: Full Network + Hardware (Months 4-6)
| Item | Cost |
|------|------|
| Network rollout (onboarding, training) | $15K |
| Hardware kits (20 Pro + 40 Standard) | $45K |
| Testimonial kiosks (20 iPads + stands) | $15K |
| Insta360 cameras (10 operatories) | $8K |
| Platform scaling + infrastructure | $5K |
| Cloud + API costs (increased volume) | $8K/mo |
| **Phase 3 Total** | ~$88K + $18K/mo |

### Steady State (Month 6+)
| Item | Monthly Cost |
|------|-------------|
| Cloud infrastructure | $5K |
| AI processing APIs | $6K |
| Platform maintenance + updates | $4K |
| Content editor (1 FTE for premium content) | $5K |
| **Steady State** | ~$20K/mo |

---

## Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Champions don't capture (behavior change fails) | Medium | Critical | Phase 1 SMS pilot tests this for $5K before investing in app. Gamification, streaks, recognition. |
| PMS integration complexity | Medium | High | Sikka ONE API handles 90%. Remaining 10% can start with manual schedule export. |
| AI production quality insufficient | Low | Medium | Human editor backstop in Phase 1-2. AI improves with training data. |
| HIPAA compliance concerns | Medium | High | Digital consent system, PHI screening (AWS Rekognition), BAAs with all processors, encrypted storage. |
| Social platform API changes | Low | Medium | Distribution engine is modular — swap APIs without rebuilding pipeline. |
| Doctor resistance to recording | Medium | Medium | Minimal ask (4 min/week), teleprompter scripts provided, results feedback loop. |

---

## Connections to Other Sprints

| Sprint | Connection |
|--------|-----------|
| **Digital Storefront at Scale** | Content Concierge feeds fresh before/afters, testimonials, and practice content to the digital storefront via Practice Concierge API. Storefronts stop being static. |
| **Full Funnel Visibility** | Content performance data (engagement → consultation → booking) feeds into the Funnel OS. HQ can see which content types drive actual patient conversions. |
| **Patient Comms Automation** | Post-treatment follow-up triggers (from patient comms system) can feed content capture reminders — "Patient completed treatment → trigger after photo prompt." |

---

## Recommended Next Steps

1. **Immediate**: Select 10-15 pilot practices (prioritize implant/cosmetic with engaged staff)
2. **Week 1**: Set up Sikka ONE API access, Twilio account, S3 storage
3. **Week 2**: Build SMS prompt bot, test with 3 practices
4. **Week 3-4**: Onboard 10-15 pilot practices, hire 2 contract editors
5. **Week 4-6**: Measure: Are champions capturing? What's the response rate? Quality of raw content?
6. **Month 2**: If pilot validates behavior → green-light Phase 2 mobile app build
7. **Month 2**: Simultaneously begin SGA brand playbook with Jensen, Ellingson, Wong
