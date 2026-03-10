# Solution Approaches — Practice + Brand Media Strategy

## Solution 1: "The Content Engine" — Full Custom Mobile App + AI Pipeline

### The Big Idea
A purpose-built mobile app (iOS/Android) for Content Champions and SGA leadership that acts as a daily media strategist in their pocket. The app connects to the PMS to know what cases are coming, pushes smart capture prompts, provides a dead-simple camera interface, and feeds everything into an AI-powered production pipeline that handles editing, formatting, captioning, and multi-channel distribution automatically.

### How It Works

**Layer 1: The Smart Prompt Engine**
- Integrates with practice management systems (Dentrix, Eaglesoft, Open Dental, etc.) via API or data sync
- Scans tomorrow's schedule every evening for content-worthy cases (implant, full-arch, cosmetic, new patient consult)
- Pushes a notification to the Content Champion: "Tomorrow at 2 PM — Sarah's full-arch implant. Capture before photo at check-in."
- For SGA brand: Weekly topic prompts pushed to leadership (Jensen, Ellingson, Wong, Olson) based on a content calendar: "This week's topic: Why clinical autonomy matters. Record 4 min."
- Post-op triggers: 2 weeks after surgery, pushes "Time for Sarah's after photo. She's on tomorrow's schedule at 10 AM."

**Layer 2: The Capture Interface**
- Opens directly to camera — no menus, no navigation
- Guided capture mode: "Step 1: Take the before photo. Step 2: Have patient sign consent (digital signature on screen). Step 3: Done."
- Video capture with countdown timer: "Record for 60 seconds. Topic on screen as teleprompter."
- Auto-tags content with patient case ID, practice ID, content type, capture date
- Uploads in background over WiFi — champion doesn't wait

**Layer 3: The AI Production Pipeline**
- Raw content hits the cloud → automated processing begins
- Photos: Color correction, dental-specific cropping (smile focus), before/after side-by-side generation, watermark/branding overlay
- Video: AI transcription → auto-captioning → highlight detection → cut into 15s/30s/60s clips → format for vertical (Reels/Shorts/TikTok) and horizontal (YouTube)
- AI-generated captions, hashtags, and descriptions per platform
- Content enters a review queue for HQ marketing team

**Layer 4: Distribution Engine**
- HQ reviews/approves content in a web dashboard (batch approve)
- Approved content auto-routes based on rules:
  - Before/after → Practice Instagram + Practice website (digital storefront) + HQ content library
  - Patient testimonial → Practice Facebook + YouTube Shorts pipeline + Google Business Profile
  - Doctor topic video → Practice YouTube + SGA LinkedIn + repurposing queue
  - Leadership thought piece → SGA LinkedIn + SGA YouTube + email newsletter excerpt
  - Partner story → SGA LinkedIn + recruitment channels + sales enablement library
- Scheduling engine posts at optimal times per platform

**Layer 5: Scoreboard & Feedback**
- Content Champion sees: their capture streak, content published count, which posts went live
- Practice Manager sees: practice content health score, comparison to network average
- HQ Dashboard: network-wide content pipeline, practice-by-practice production, channel performance
- Monthly "Content Champion of the Month" recognition

### Playbook System
Each practice is assigned a playbook on setup:
- **Playbook 1 (Implant/Cosmetic)**: Daily prompts, before/after + testimonial + topic video, full hardware kit
- **Playbook 2 (Specialty)**: 2-3x/week prompts, before/after + case study captures, basic hardware kit
- **Playbook 3 (Growth GP)**: 2-3x/week prompts, smile makeovers + culture + testimonials, basic kit
- **Playbook 4 (Standard GP)**: Weekly prompts, review captures + team photos, phone only
- **Playbook 5 (SGA Brand)**: Weekly topic prompts to leadership, event capture playbook, milestone prompts

### Hardware Kits (Ship-in-a-Box)
| Kit | Contents | For |
|-----|----------|-----|
| **Pro Kit** | Phone tripod, ring light, Insta360 camera, phone mount (operatory), teleprompter app license | Playbook 1 |
| **Standard Kit** | Phone tripod, phone mount, basic ring light | Playbooks 2 & 3 |
| **Starter Kit** | Phone mount (desk clip) | Playbook 4 |
| **Leadership Kit** | Compact tripod, lavalier mic, ring light | Playbook 5 |

### Strengths
- Fully integrated, purpose-built for the exact problem
- AI pipeline eliminates HQ bottleneck on content production
- PMS integration makes prompts intelligent, not generic
- Scales to 260+ locations with playbook system
- Single app serves both practice-level and brand-level capture

### Risks
- Custom app development is expensive and takes time
- PMS integration complexity (multiple systems across 260 locations)
- Adoption risk — will champions actually use it daily?
- AI processing quality may not meet brand standards initially

### Estimated Build Complexity
**High** — custom mobile app (React Native), cloud AI pipeline, PMS integrations, distribution API integrations, web dashboard. 4-6 months to MVP.

---

## Solution 2: "The Content Coach" — AI-Powered SMS/WhatsApp Bot + Existing Tools

### The Big Idea
Instead of a custom app, use the phone's native capabilities. An AI-powered conversational bot (via SMS or WhatsApp) acts as the Content Champion's daily coach. It sends prompts, receives photos/videos as replies, processes them through an AI pipeline, and distributes through existing social management tools (Sprout Social, Later, or SOCi). No app download. No new behavior beyond texting.

### How It Works

**Morning Brief (7:30 AM daily)**
The bot texts the Content Champion:
```
Good morning! 📸 Today's content opportunities:

1. Sarah M. — Full-arch implant, 2 PM
   → Capture: Before photo at check-in
   → Reply with photo when ready

2. Dr. Williams — Topic of the week:
   "3 signs you need an implant"
   → Record 2 min video, send here

Reply 1, 2, or SKIP
```

**Capture Flow**
- Champion replies "1" → bot says "Great. Take the before photo and send it here."
- Champion takes photo with phone camera, sends as text/WhatsApp message
- Bot confirms: "Got it! I'll process this and have it ready by tomorrow."
- Post-op (2 weeks later): "Sarah's after photo is due! She's at 10 AM tomorrow. Reply with the photo after her visit."

**SGA Brand Flow**
- Leadership receives weekly text: "This week's topic: What clinical autonomy means in practice. Record 2-4 min and send here. Need a script? Reply SCRIPT."
- If they reply SCRIPT, bot sends 3 bullet points to cover
- Leader records on phone, sends video via text/WhatsApp
- Bot: "Got it! We'll cut this into a LinkedIn clip and YouTube Short."

**Backend Processing**
- Same AI pipeline as Solution 1 (color correction, cropping, captioning, cutting)
- Content enters HQ review queue (web dashboard — could be a simple Airtable/Notion + Zapier setup or custom)
- Approved content pushed to Sprout Social / Later / SOCi for scheduling and distribution
- Digital storefront updated via API

**Consent Handling**
- Bot sends a link to a digital consent form before first capture per patient
- Patient signs on champion's phone (mobile web form)
- Consent stored and linked to patient case ID

### Strengths
- **Zero friction adoption** — no app download, no new interface to learn, everyone knows how to text
- **Fast to build** — Twilio/WhatsApp Business API + AI pipeline + existing social tools
- **Low cost** — no custom mobile app development
- **Works on any phone** — not dependent on iOS/Android app stores
- **Natural behavior** — staff already text photos to each other

### Risks
- Limited UX — can't show scoreboards, dashboards, or rich interfaces via SMS
- Media quality through MMS/WhatsApp compression
- Harder to do guided multi-step capture (before + consent + after as linked sequence)
- PMS integration still needed for smart prompts
- Less "product" feel — may feel scrappy vs. premium

### Estimated Build Complexity
**Medium** — Twilio/WhatsApp API, AI pipeline, web dashboard, PMS integration, social tool integrations. 2-3 months to MVP.

---

## Solution 3: "The Media Playbook Platform" — Structured Workflow App + Human Production Team

### The Big Idea
A lightweight web app (mobile-responsive, no native app needed) that manages the capture WORKFLOW but keeps humans in the production loop. The platform is a task management system specifically for content capture — it assigns capture tasks to champions, tracks completion, collects media, and routes it to a centralized production team (in-house or outsourced) who handles all editing and distribution. AI assists but doesn't replace the human editors.

### How It Works

**Content Calendar + Task System**
- HQ creates a monthly content calendar per playbook type
- Calendar generates specific capture tasks assigned to each practice's Content Champion
- Tasks appear in a mobile-friendly web dashboard: "This week: 2 before/afters, 1 testimonial video, 1 office culture photo"
- PMS integration adds case-specific tasks: "Thursday: Capture before photo for patient #4521 (full-arch)"
- SGA brand tasks: "This week: Jensen records Q4 growth update (3 min). Wong records podcast teaser (90 sec)."

**Capture + Upload**
- Champion opens the web app on phone → sees today's tasks
- Taps a task → camera opens with guidance overlay: "Take the before photo. Smile in frame. Good lighting."
- Photo/video uploads to cloud storage (S3/GCS) with metadata tags
- Task marked complete → next task appears
- Progress bar: "3 of 5 captures complete this week"

**Human Production Team**
- Raw content arrives in a production dashboard (Airtable, Frame.io, or custom)
- Production team (2-3 editors, could be outsourced) handles:
  - Color correction and clinical-grade photo editing
  - Video editing: cuts, captions, platform formatting
  - Brand template application
  - Caption and hashtag writing
- Edited content enters approval queue
- HQ approves → content scheduled via Sprout Social/Later

**Quality Over Automation**
- Human editors ensure brand consistency and clinical accuracy
- Can handle nuanced decisions AI can't: "This before/after is too graphic for Instagram but fine for the clinical case library"
- Editors learn each practice's style over time

### Strengths
- **Higher quality output** — human editors > AI editing (for now)
- **Simpler tech** — web app + cloud storage + production tools, no AI pipeline to build
- **Faster to launch** — could be running in 4-6 weeks with existing tools
- **Brand safety** — human review catches problems before they post
- **Flexible** — easy to adjust playbooks without rebuilding AI models

### Risks
- **Doesn't scale linearly** — more practices = more editors needed
- Production team becomes a bottleneck at 100+ active practices
- Human turnaround time (24-72 hours) vs. AI (minutes)
- Higher ongoing labor cost vs. AI pipeline
- Still needs the behavior change at practice level (same adoption challenge)

### Estimated Build Complexity
**Low-Medium** — mobile-responsive web app, cloud storage, integration with production tools (Frame.io/Airtable), social scheduling tool. 4-6 weeks to MVP. But ongoing staffing cost for production team.

---

## Solution 4: "The Passive Capture Network" — Hardware-First, Behavior-Second

### The Big Idea
Flip the model. Instead of asking humans to remember to capture content, install capture infrastructure IN the practice that records passively. Insta360 cameras in operatories, Ring-style cameras at the front desk for patient arrivals, always-on recording that an AI system reviews to identify and extract content-worthy moments. Supplement with minimal active capture (before/after photos via simple kiosk).

### How It Works

**Hardware Installation Per Practice**
- **Operatory cameras** (Insta360 or similar): Mounted in 1-2 operatories designated for high-value cases. Records continuously during procedures. Patient consent obtained at intake (digital form on check-in tablet).
- **Testimonial station**: iPad on a stand near checkout with a simple prompt: "Tell us about your experience today! Tap to record." 60-second limit. Auto-uploads.
- **Before/after kiosk**: Tablet with guided photo capture at check-in. "Step into the frame. Smile. Now show your teeth." Two photos, 15 seconds, done.
- **Ray-Ban Meta glasses**: For doctors willing to wear them — POV content during procedures (with consent).

**AI Content Mining**
- Continuous recordings processed by AI overnight
- AI identifies: reveal moments, patient reactions, interesting procedure segments, team interactions
- Extracts highlight clips automatically
- Tags with metadata: procedure type, emotional intensity, content category
- Surfaces highlights in HQ dashboard: "12 new clips extracted from 6 practices today"

**Active Capture Layer (Minimal)**
- The before/after kiosk handles the highest-value capture with zero champion involvement
- Testimonial station captures patient stories without staff needing to hold a phone
- For doctor topic videos and leadership content: simple "record booth" setup in a quiet room with ring light and tripod permanently installed

**Brand Content**
- SGA HQ has a dedicated recording setup (permanent studio corner)
- Leadership walks in, reads topic prompt on teleprompter screen, records, walks out
- System handles the rest

### Strengths
- **Removes human behavior from the equation** — the #1 risk factor
- **Captures content that would never be captured otherwise** — AI mining finds moments staff wouldn't think to record
- **Consistent quality** — fixed camera positions, controlled lighting
- **Scales well** — install once, capture forever
- **Patient-driven testimonials** — self-serve kiosk removes staff friction entirely

### Risks
- **High upfront hardware cost** — cameras, tablets, installation across 260 locations
- **Privacy/HIPAA concerns** — continuous recording in healthcare settings requires robust consent and data handling
- **AI content mining is immature** — may produce low-quality extractions
- **Less authentic feel** — fixed camera content lacks the spontaneous, personal quality of phone-captured content
- **Maintenance burden** — hardware breaks, needs updates, IT support across 260 locations
- **Doesn't solve brand content well** — leadership still needs to actively record

### Estimated Build Complexity
**High** — hardware procurement/installation, IoT management platform, AI video mining pipeline, consent management system. 6-9 months to pilot at 10 practices.

---

## Solution 5: "The Content Concierge" — Hybrid System (Recommended)

### The Big Idea
Combine the best elements of Solutions 1-4 into a phased, hybrid approach. Start with a smart mobile app (Solution 1's prompt engine + capture UX) augmented by an AI production pipeline, but launch fast by beginning with SMS/WhatsApp prompts (Solution 2) while the app is built. Layer in passive capture hardware (Solution 4) at high-value practices over time. Use human editors (Solution 3) as a quality backstop during the AI pipeline's learning phase.

### The Name: **Content Concierge**
Named to connect with the existing "Practice Concierge" concept from the digital storefront sprint. The Content Concierge is the media arm of the Practice Concierge — it knows your practice, knows your cases, and orchestrates your content strategy automatically.

### Phase 1: Launch (Weeks 1-6) — "The SMS Coach"
- Deploy Twilio/WhatsApp-based prompt system for 10-15 pilot practices
- PMS data sync (batch, not real-time) to identify upcoming high-value cases
- Champions receive daily SMS prompts with capture assignments
- Photos/videos sent via text → uploaded to cloud
- Human production team (2 editors) handles processing
- Manual distribution through existing social tools
- SGA leadership gets weekly topic prompts via same SMS system
- **Goal**: Prove the behavior change. Get content flowing.

### Phase 2: Scale (Months 2-4) — "The Mobile App"
- Launch Content Concierge mobile app (React Native)
- Guided capture interface with camera-first UX
- In-app consent capture (digital signature)
- AI production pipeline goes live (color correction, auto-cropping, caption generation)
- Automated channel routing with HQ approval queue
- Content scorecard and practice leaderboard
- Roll to 40-65 high-value practices (Playbooks 1-2)
- Brand playbook live for SGA leadership team
- **Goal**: Scale capture and automate production.

### Phase 3: Accelerate (Months 4-6) — "The Full Network"
- Roll to all 260+ practices (Playbooks 3-4 adapted for GP use)
- AI pipeline handles 80%+ of production automatically
- Human editors focus on premium content (YouTube long-form, major transformation stories)
- Passive capture hardware piloted at top 10 implant practices (Insta360 in operatories, testimonial kiosks)
- Content performance analytics tied to patient acquisition metrics
- **Goal**: Full network coverage. Content pipeline fueling growth.

### Phase 4: Optimize (Months 6-12) — "The Content Intelligence Layer"
- AI learns which content types drive actual patient bookings (not just views)
- System automatically adjusts prompts: "Practices that capture testimonials on Tuesdays get 2x more engagement — shifting your prompt day"
- Predictive content suggestions: "Based on your case mix this month, here's your optimal content calendar"
- Passive capture hardware expanded based on Phase 3 pilot results
- Integration with paid media pipeline: high-performing organic content auto-surfaces for ad spend
- **Goal**: Self-optimizing content system.

### Architecture Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                     CONTENT CONCIERGE                           │
├──────────┬──────────┬──────────┬──────────┬────────────────────┤
│  PROMPT  │ CAPTURE  │PRODUCTION│  REVIEW  │   DISTRIBUTION     │
│  ENGINE  │ LAYER    │ PIPELINE │  QUEUE   │   ENGINE           │
├──────────┼──────────┼──────────┼──────────┼────────────────────┤
│ PMS Sync │ Mobile   │ AI Color │ HQ Dash  │ Practice Social    │
│ Case ID  │ App      │ AI Crop  │ Batch    │ Digital Storefront │
│ Content  │ SMS/     │ AI Video │ Approve  │ SGA Brand Channels │
│ Calendar │ WhatsApp │ Cut      │ Human QC │ YouTube Pipeline   │
│ Playbook │ Kiosk    │ AI       │ Brand    │ LinkedIn (B2B)     │
│ Rules    │ Passive  │ Captions │ Safety   │ Email Newsletter   │
│          │ Cameras  │ AI       │          │ Sales Enablement   │
│          │          │ Hashtags │          │ Paid Media Feed    │
└──────────┴──────────┴──────────┴──────────┴────────────────────┘
         │                  │                        │
         v                  v                        v
   ┌──────────┐      ┌──────────┐            ┌──────────┐
   │ Practice │      │ Content  │            │ Growth   │
   │ Mgmt     │      │ Score-   │            │ Metrics  │
   │ Systems  │      │ card     │            │ Dashboard│
   └──────────┘      └──────────┘            └──────────┘
```

### The 5 Playbooks

| # | Playbook | Audience | Captures/Week | Prompt Source | Distribution |
|---|----------|----------|---------------|-------------|-------------|
| 1 | **Implant/Cosmetic** | Patients (B2C) | 5-7 | PMS cases + content calendar | Practice IG, YouTube, Digital Storefront, Paid Media |
| 2 | **Specialty** | Patients + Referring GPs | 2-3 | PMS cases + referral events | Practice website, Referring GP materials, Case library |
| 3 | **Growth GP** | Local patients | 2-3 | Content calendar + seasonal | Practice FB/IG, Google Business, Digital Storefront |
| 4 | **Standard GP** | Local patients | 1 | Content calendar (minimal) | Google Business, Practice website |
| 5 | **SGA Brand** | Dentists (B2B) | 3-5 | Content calendar + events + milestones | LinkedIn, SGA YouTube, Recruitment, Sales Enablement |

### Strengths
- **Fast to start** — SMS pilot in weeks, not months
- **Progressively sophisticated** — each phase adds capability without starting over
- **Risk-mitigated** — prove behavior change before investing in AI pipeline
- **Connects to existing systems** — Practice Concierge, Digital Storefront, social strategy
- **Serves both audiences** — practice-level (B2C) and brand-level (B2B) with shared infrastructure
- **Human + AI** — editors ensure quality while AI scales

### Risks
- Phased approach means full capability takes 6+ months
- Still depends on behavior change at practice level (SMS pilot tests this cheaply)
- Multiple moving parts to coordinate
- PMS integration remains the hardest technical challenge

### Estimated Build Complexity
- Phase 1: **Low** (4-6 weeks) — SMS bot + cloud storage + human editors
- Phase 2: **Medium** (8-12 weeks) — Mobile app + AI pipeline
- Phase 3: **Medium** (8-12 weeks) — Network rollout + hardware pilot
- Phase 4: **Low** (ongoing) — Optimization and learning
- **Total to full capability**: 6-9 months

---

## Solution Comparison Matrix

| Criteria | 1: Content Engine | 2: SMS Coach | 3: Media Playbook | 4: Passive Capture | 5: Content Concierge |
|----------|:-:|:-:|:-:|:-:|:-:|
| **Time to first content** | 4-6 months | 2-3 weeks | 4-6 weeks | 6-9 months | 2-3 weeks |
| **Behavior change required** | High | Low | Medium | Very Low | Low → Medium |
| **Production quality** | High (AI) | Medium (AI) | High (human) | Variable (AI) | High (hybrid) |
| **Scalability** | Excellent | Good | Limited | Excellent | Excellent |
| **Brand content support** | Strong | Good | Good | Weak | Strong |
| **PMS integration** | Deep | Light | Light | Deep | Progressive |
| **Upfront cost** | $$$$  | $ | $$ | $$$$$ | $ → $$$$ |
| **Ongoing cost** | $$ | $ | $$$$ | $$ | $$ |
| **Risk level** | High | Low | Medium | Very High | Low (phased) |
| **Connects to Practice Concierge** | Yes | Partial | No | No | Yes (by design) |

---

## Facilitator Recommendation

**Solution 5: The Content Concierge** is the recommended approach. It:

1. **Starts fast** — SMS pilot proves the concept in weeks, not months
2. **De-risks** — don't build the app until you've proven champions will capture content
3. **Scales** — same architecture handles 10 practices or 260
4. **Serves both audiences** — practice B2C content and SGA brand B2B content
5. **Connects** — extends the Practice Concierge ecosystem from the digital storefront sprint
6. **Phases naturally** — each phase delivers standalone value, not dependent on future phases

The key insight: **the biggest risk isn't technology — it's behavior change.** Starting with SMS eliminates every technology barrier (no download, no new interface, no learning curve) and isolates the one variable that matters: will champions capture content when prompted? If yes, invest in the full platform. If no, you've learned that for the cost of a Twilio account.
