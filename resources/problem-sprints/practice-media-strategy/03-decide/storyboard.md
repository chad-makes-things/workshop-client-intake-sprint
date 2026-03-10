# Storyboard — The Content Concierge

## Scenario 1: Practice-Level — "Sarah's Full-Arch Implant" (Implant/Cosmetic Playbook)

This traces a single patient case from scheduling through published content, showing every touchpoint with the Content Concierge system.

---

### Scene 1: Case Identified (Tuesday Evening)

**Who**: The System (Content Concierge)
**What happens**: Nightly PMS sync via Sikka ONE API pulls tomorrow's schedule for Innovative Dental of Springfield. The system identifies:
- **Sarah M., 2:00 PM** — Full-arch implant procedure (CPT code flagged as high-value content opportunity)
- Patient has signed a media consent form (captured digitally at her consultation visit 3 weeks ago)

**System action**: Generates a content capture assignment and queues it for the Content Champion.

---

### Scene 2: Morning Prompt (Wednesday, 7:30 AM)

**Who**: Jessica (Dental Assistant, Content Champion)
**Where**: Her phone, before she arrives at the practice

**Phase 1 (SMS)**:
```
Good morning Jessica! 📸

Today's content opportunity:

Sarah M. — Full-arch implant, 2 PM
✅ Media consent on file

CAPTURE:
→ Before photo at check-in (smile, front view)
→ Before photo (retracted, clinical)

Reply READY when she checks in, and I'll walk you through it.
```

**Phase 2 (Mobile App)**:
Jessica opens the Content Concierge app. Her dashboard shows:
- **Today**: 1 capture assignment (Sarah M., full-arch, 2 PM)
- **This week**: 3/5 captures complete (progress bar)
- **Streak**: 🔥 12 days

She taps into Sarah's assignment and sees:
- Capture checklist: Before photo (smile) ○ | Before photo (clinical) ○ | Testimonial video ○
- Patient consent status: ✅ Signed 2/18
- Guidance: "Lighting tip: Use the ring light in Operatory 2. Position patient facing the window."

---

### Scene 3: Before Photo Capture (Wednesday, 1:50 PM)

**Who**: Jessica + Sarah (patient)
**Where**: Operatory 2, check-in area

Sarah checks in. Jessica opens the app, taps "Start Capture."

The camera opens with an overlay guide:
- Outline showing ideal framing for a dental before photo (face centered, mouth-level)
- Text prompt: "Smile photo — ask patient to smile naturally"

Jessica takes 2 photos in 20 seconds:
1. Natural smile (front view)
2. Retracted clinical view

The app auto-tags: Patient ID (linked from PMS), Practice: Innovative Dental Springfield, Content Type: Before/After - Before, Case: Full-Arch Implant, Date: 2026-03-11.

Upload begins in background. Jessica's checklist updates:
- Before photo (smile) ✅ | Before photo (clinical) ✅ | Testimonial video ○

**Total time**: 30 seconds.

---

### Scene 4: Surgery Happens (Wednesday, 2:00-5:00 PM)

**Who**: Dr. Williams + surgical team
**Where**: Operatory 2

Dr. Williams performs the full-arch implant procedure. An **Insta360 camera** mounted in the operatory (Phase 3 hardware) captures the procedure passively. The AI will later mine this footage for:
- Time-lapse of the procedure (30-second process video)
- The moment temporary teeth are fitted (the "reveal" potential)

Jessica is focused on assisting — the system handles passive capture.

---

### Scene 5: Post-Op Capture Opportunity (Wednesday, 5:15 PM)

**Who**: Jessica + Sarah
**Where**: Recovery area

Sarah is recovering. She sees her temporary teeth for the first time in a mirror. Jessica gets a notification:

**App notification**: "Sarah's procedure is complete. Now is a great time for a reaction video! Tap to capture."

Jessica asks Sarah: "Would you like to see your new smile? Can we capture your reaction?" Sarah agrees enthusiastically.

Jessica taps "Capture Testimonial" — the app opens to video mode with a 60-second timer:
- On-screen prompt (visible only to Jessica): "Let her react naturally. Then ask: 'How do you feel right now?'"
- Jessica records Sarah's genuine reaction: tears, smile, "Oh my God, is that really me?"

45-second video captured. Auto-uploaded. Checklist complete:
- Before photo (smile) ✅ | Before photo (clinical) ✅ | Testimonial video ✅

App shows: "All captures complete! 🎉 You'll see the finished content within 48 hours."

**Total active capture time across the entire day**: Under 2 minutes.

---

### Scene 6: AI Production Pipeline (Wednesday Night → Thursday)

**Who**: The System (automated) + HQ Production Team
**Where**: Cloud

Raw content enters the production pipeline:

**Photo Processing (Automated)**:
1. Before photos → color correction (Cloudinary API) → dental-specific cropping (smile focus)
2. AI generates before/after side-by-side template (before on left, "after coming soon" placeholder on right)
3. Practice branding overlay applied (Innovative Dental logo, colors)
4. Formatted for: Instagram square, Instagram Story vertical, Facebook, digital storefront banner

**Video Processing (Automated)**:
1. Testimonial video → AI transcription → auto-generated captions (burned in, bold, high-contrast)
2. AI identifies highlight moment (the reaction: "Oh my God, is that really me?")
3. Cuts 3 versions: 45-second full, 15-second highlight reel, 30-second mid-length
4. Formats for vertical (Reels/Shorts/TikTok) and horizontal (YouTube)
5. AI generates captions: "Sarah hadn't smiled in years. Watch what happens when she sees her new teeth for the first time."
6. AI generates hashtags: #DentalImplants #SmileTransformation #BeforeAndAfter #InnovativeDental

**Human Review Queue (Thursday Morning)**:
Dakota's team sees 14 new content pieces in the review dashboard:
- 4 formatted before photos (different platform sizes)
- 3 video clips (different lengths)
- 4 AI-generated social captions (one per platform)
- 3 thumbnail options

They review in batch — approve 12, adjust 2 captions, reject 0. Total review time: 4 minutes.

---

### Scene 7: Distribution (Thursday, Automated)

**Who**: The System (Content Concierge distribution engine)
**What**: Approved content auto-routes based on Playbook 1 (Implant/Cosmetic) rules:

| Content Piece | Destination | Timing |
|---------------|-------------|--------|
| Before photo (smile) + "After coming soon" teaser | Innovative Dental Instagram | Thursday 11 AM |
| Testimonial highlight (15 sec) | Innovative Dental Instagram Reels | Thursday 7 PM |
| Before photo (clinical) | Digital Storefront — implant gallery | Immediately |
| Testimonial (45 sec full) | Innovative Dental Facebook | Friday 11 AM |
| Testimonial (30 sec) | YouTube Shorts pipeline (for Dr. Olson's team to review for channel) | Queued |
| Before photo | HQ content library (for paid media use) | Immediately |

---

### Scene 8: The After Photo (2 Weeks Later — Wednesday)

**Who**: Jessica + Sarah
**Where**: Innovative Dental, follow-up visit

2 weeks ago, the system noted Sarah's surgery date. Today, she's back for a follow-up. The system triggers:

**7:30 AM notification to Jessica**:
```
Sarah M. is back today at 10 AM for her 2-week follow-up.
Time for the AFTER photos! 📸

CAPTURE:
→ After photo (smile — match the before angle)
→ After photo (clinical)
→ Optional: "2 weeks later" video (30 sec)

Her before photos are in the app for reference.
```

Jessica opens the app. The before photos are displayed side-by-side with a camera viewfinder — she can match the exact angle.

She captures the after photos in 30 seconds. The system instantly generates:
- Before/after side-by-side (the money shot)
- Formatted for every platform
- Updated captions: "Sarah's smile transformation — 2 weeks after full-arch implants at Innovative Dental"

The before/after enters production → review → distribution within 24 hours.

**This single patient case has now produced**:
- 2 before photos, 2 after photos
- 1 before/after side-by-side (multiple formats)
- 1 reaction testimonial video (3 lengths)
- 1 procedure time-lapse (from Insta360, Phase 3)
- **= ~15-20 platform-specific content pieces from under 3 minutes of Jessica's time**

---

### Scene 9: Feedback Loop (Friday)

**Who**: Jessica
**Where**: Her phone

Jessica gets a notification: "Your content from Sarah's case just went live! 🎉"

She taps through and sees:
- The Instagram post with 47 likes and 8 comments in 2 hours
- The Reel with 1,200 views
- Her content scorecard: "This week: 5/5 captures complete. You're the #3 Content Champion in the SGA network!"

She screenshots this and texts it to her practice manager. She's proud of the work. The habit reinforces.

---

## Scenario 2: SGA Brand — "Jensen's Weekly Thought Leadership" (Brand Playbook)

### Scene 1: Weekly Topic Push (Monday, 8:00 AM)

**Who**: Lamonte Jensen (CEO, Gen4)
**Where**: His phone

**SMS/App notification**:
```
Good morning Lamonte. This week's content topic:

"Why the best dentists are joining partnerships, not selling"

Key points to cover:
→ The difference between selling and partnering
→ Equity participation as wealth building
→ One specific partner success story

Record 3-4 min on your phone when ready. Reply SCRIPT for a detailed outline.
```

Lamonte replies "SCRIPT." The system sends:
```
SCRIPT OUTLINE — 3 min video

HOOK (15 sec):
"Everyone asks me why successful dentists would join a partnership. They assume it's about cashing out. It's not."

POINT 1 (60 sec):
Selling vs. partnering — when you sell, you get a check and lose control. When you partner, you get liquidity AND keep equity in something growing.

POINT 2 (60 sec):
The wealth-building math — our partners don't just get a payout, they own shares in a platform that's compounding. Over time, that equity has outperformed the initial transaction.

POINT 3 (45 sec):
Specific example — Dr. Cavendish: revenues up 16.5%, EBITA beating plan, 100% staff retention. He didn't sell. He leveled up.

CLOSE (15 sec):
"If you're a practice owner thinking about your next chapter, the question isn't 'should I sell?' It's 'what am I building?'"

Total: ~3 min
```

### Scene 2: Capture (Tuesday, between meetings)

Lamonte sets his phone on the compact tripod from his Leadership Kit, clips on the lavalier mic, and records. 3 minutes 40 seconds. Sends via the app (or replies to the text with the video).

**Total effort**: 5 minutes including setup.

### Scene 3: AI Production (Tuesday night)

The system processes Lamonte's video:
1. **Full LinkedIn video** (3:40, horizontal, captions burned in)
2. **LinkedIn clip 1** (60 sec — the hook + selling vs. partnering point)
3. **LinkedIn clip 2** (45 sec — the Dr. Cavendish story)
4. **YouTube Short** (60 sec — the hook + best soundbite)
5. **Quote graphic** — "The question isn't 'should I sell?' It's 'what am I building?'" with Lamonte's headshot and SGA branding
6. **Email newsletter excerpt** — 200-word summary + link to full video
7. **Sales enablement clip** — The Dr. Cavendish segment, tagged for Marcus Morosin's outreach use

### Scene 4: Distribution

| Piece | Channel | Audience |
|-------|---------|----------|
| Full video | Lamonte Jensen's LinkedIn (personal) | Practice owners (PA funnel) |
| Clip 1 | SGA Dental Partners LinkedIn (company) | Broad B2B |
| Clip 2 | SGA YouTube channel | Dentists researching DSOs |
| Quote graphic | Lamonte's LinkedIn + SGA Instagram | B2B awareness |
| Newsletter excerpt | SGA partner newsletter | Existing partners + nurture list |
| Sales clip | Marcus Morosin's outreach library | Active acquisition prospects |

**One 4-minute recording → 7 distribution-ready pieces across 4 channels.**

---

## Scenario 3: Event Content — "4Front Symposium" (Brand Playbook — Event Mode)

### Scene 1: Event Mode Activated (Day Before Symposium)

HQ activates "Event Mode" in the Content Concierge for the 4Front Symposium. This pushes a special capture playbook to 5 designated event content champions (SGA marketing staff + 2 partner dentists who volunteered).

**Event capture checklist appears in their apps**:
- [ ] Venue setup / atmosphere shots (3-5 photos)
- [ ] Keynote speaker on stage (2 photos + 30-sec video)
- [ ] Partner interactions / networking moments (5 photos)
- [ ] At least 2 partner testimonials (60-sec videos): "Why did you come to 4Front?"
- [ ] Dr. Pikos teaching moment (photo + video)
- [ ] Team/group photo
- [ ] "Best moment of the day" video (60 sec, any champion)

### Scene 2: During the Event

Champions capture throughout the day. The app's event mode simplifies: tap → capture → auto-tag "4Front 2026" → upload.

By end of day: 42 photos, 8 videos uploaded from 5 champions.

### Scene 3: AI Processes Overnight

- Best photos auto-selected by AI (composition, lighting, facial expressions scored)
- Videos captioned, trimmed, formatted
- LinkedIn carousel created: "Inside 4Front 2026 — SGA's Annual Partner Symposium"
- Instagram Story sequence created (12 slides with event highlights)
- Two 45-second highlight reels generated

### Scene 4: Next-Day Distribution

- LinkedIn carousel → Lamonte Jensen + SGA company page (Monday morning after event)
- Instagram Stories → SGA Instagram (same day)
- Highlight reel → SGA YouTube + TikTok
- Partner testimonials → Recruitment channels + acquisition nurture emails
- Best photos → Digital storefront "About SGA" section

---

## The Content Concierge Dashboard (HQ View)

### What Dakota's Team Sees

**Network Overview** (real-time):
```
CONTENT CONCIERGE — NETWORK DASHBOARD

Active Practices: 47/65 (high-value practices)
Content Champions Active: 43

THIS WEEK:
├── Captures Submitted: 127
├── In Production: 34
├── In Review Queue: 18
├── Published: 75
└── Scheduled: 22

TOP PRACTICES (by content volume):
1. Innovative Dental Springfield — 12 captures
2. Garner Dental Group Birmingham — 9 captures
3. Dakota Dental Minneapolis — 8 captures

NEEDS ATTENTION:
⚠️ Troy Family Dental — 0 captures in 14 days
⚠️ Pensacola Perio — Champion not responding to prompts
⚠️ Wake Dental Care — Media consent expired for 3 patients

BRAND CONTENT:
├── Jensen: Weekly video ✅ (published Tuesday)
├── Ellingson: Weekly video ⏳ (due Thursday)
├── Wong: Podcast teaser ✅ (published Monday)
└── Olson: Auto-synced from YouTube pipeline
```

### Review Queue (Batch Approve):
Content pieces appear as cards. Dakota can:
- **Approve** (one tap — auto-distributes per playbook rules)
- **Edit caption** (inline text edit)
- **Reject** (with reason — champion gets feedback)
- **Star** (flags for premium treatment — human editor enhancement, potential Dr. Olson YouTube feature)
- **Bulk approve** (select all → approve → done in seconds)
