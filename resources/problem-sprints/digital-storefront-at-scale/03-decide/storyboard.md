# Storyboard: The Practice Concierge

## Solution Summary
Every SGA practice gets an AI concierge — a conversational agent that is their single point of contact with SGA marketing. The concierge handles digital storefront enrollment (both new acquisitions and existing practices), ongoing change management, content approvals, and Q&A. Behind the concierge, an orchestration layer routes actions to website templates, GMB, social publishing, CRM, and the executive dashboard.

Two user journeys are storyboarded:
- **Journey A**: Existing practice enrollment (the 260+ modernization)
- **Journey B**: HQ team experience (managing the network)

---

## Journey A: Existing Practice Enrollment

### Step 1: Discovery & Pre-Population
**Scene**: Before any practice interaction, the system silently harvests public data for every practice in the SGA network.
**Action**: Automated scrapers pull current website content, Google My Business listings, social media profiles (Facebook, Instagram), online reviews (Google, Yelp), directory listings (Healthgrades, Zocdoc), and staff information from public sources.
**System Response**: For each practice, the system builds a "Practice Profile Draft" — a structured data record with: practice name, address, phone, hours, provider names and headshots (if available), services listed, current website URL and screenshots, GMB status (claimed/unclaimed, completeness score), social handles and recent activity, review summary (avg rating, count, sentiment), and identified gaps (missing GMB, no social presence, outdated website).
**User Reaction**: No practice interaction yet. This happens entirely in the background.
**Critical Detail**: The scraping must be thorough enough that the practice's first interaction feels like "you already know us" — not "tell us everything from scratch." This is the "Head Start" pattern from the client onboarding sprint. The quality of this pre-population determines first-impression impact.

### Step 2: Enrollment Invitation
**Scene**: HQ marketing sends enrollment invitations to practices in batches of 20-30. The invitation introduces the Practice Concierge and provides a link.
**Action**: Each practice receives a personalized message (email + SMS): "Hi [Practice Name] team — we've built something to make your digital presence effortless. Your SGA Marketing Concierge already knows a lot about your practice. Click here to meet them and get your storefront upgraded in about 15 minutes."
**System Response**: The invitation includes a preview of what the system already knows ("We found your Google listing, your Facebook page, and your current website — let's make them all best-in-class").
**User Reaction**: The office manager or practice owner clicks through during a 5-minute break between patients. The "we already know about you" hook creates curiosity rather than dread.
**Critical Detail**: The invitation must NOT feel like "another thing HQ is making us do." It must feel like a gift. The preview data demonstrates value before asking for anything. The "15 minutes" time commitment is honest and low-friction.

### Step 3: Concierge Welcome & Data Verification
**Scene**: The practice opens the Concierge chat interface. The concierge greets them by name and presents what it already knows.
**Action**: The concierge says: "Hey there — I'm your SGA Marketing Concierge. I've already pulled together what I could find about [Practice Name]. Let me show you what I've got, and you can tell me what's right and what needs updating."
**System Response**: The concierge presents the Practice Profile Draft in digestible chunks:
- "Here's your team as I found them: [list of providers]. Anyone missing or no longer with you?"
- "Your current website is at [URL]. Here's a screenshot — I'll show you the upgrade in a minute."
- "Your Google Business Profile is [status]. Your reviews average [X stars] across [Y reviews]."
- "I found your Facebook page at [URL] — last post was [date]."
**User Reaction**: The practice is impressed ("you did your homework") and corrects a few things: "Dr. Miller left 6 months ago" and "we added a new hygienist, Sarah." The corrections take 3-4 minutes.
**Critical Detail**: The concierge must handle corrections gracefully and immediately: "Got it — I'll remove Dr. Miller from everything and add Sarah. Do you have a photo of Sarah, or should I leave a placeholder for now?" Every correction triggers a downstream cascade (remove Dr. Miller from website draft, GMB, social bios). The concierge's tone is Sage + Caregiver — knowledgeable, warm, never bureaucratic.

### Step 4: Storefront Generation & Template Selection
**Scene**: With verified data in hand, the concierge generates the practice's new digital storefront.
**Action**: The concierge says: "Based on your practice type [GP / Perio / Full Arch / Pediatric], I've prepared your new website. Here are two options — both are optimized for mobile, SEO, and patient conversion. Take a look:"
**System Response**: The system presents 2-3 template previews, populated with the practice's real data (name, team, services, photos, colors). Each template is specialty-optimized:
- GP template: appointment-booking optimized, new patient focused
- Perio template: referral-optimized, specialist credibility focused
- Full Arch template: transformation stories, financing calculator, Dr. Olson content integration
The concierge explains WHY: "I'm recommending Template A because periodontist sites convert 23% better with referral-optimized layouts. Your referring GPs will find it easier to send patients your way."
**User Reaction**: The practice picks a template. For the 90% who don't have strong opinions, the concierge's recommendation carries the day. For the 10% with opinions, the concierge walks through customization options (color palette from SGA-approved options, hero image selection, content emphasis).
**Critical Detail**: The "guardrailed decision" pattern — every option is SGA-approved. The practice feels ownership ("I chose this") but quality is guaranteed regardless of what they pick. If the practice doesn't respond within 3 weeks, the concierge auto-deploys the recommended template: "I went ahead and launched your new site with my recommendation — you can always adjust later."

### Step 5: Full Storefront Review & Approval
**Scene**: The concierge presents the complete digital storefront package for final review.
**Action**: "Here's your complete digital upgrade — everything is ready to go live:"
**System Response**: The concierge shows:
- **Website preview**: Full site with all pages populated, mobile view, desktop view
- **Google Business Profile**: Updated listing with correct hours, services, providers, photos
- **Social handles**: Instagram and Facebook profiles with cover images, bios, and first 4 scheduled posts
- **Lead capture**: Contact form connected, thank-you email preview, appointment request flow
- "I'll publish everything when you give the thumbs up. Or flag anything you'd like to change."
**User Reaction**: Most practices approve immediately. Some flag a photo they don't like or a service description that needs tweaking. The concierge handles changes in real-time: "Updated! Here's the new version."
**Critical Detail**: The HQ creative team gets a notification in their QC queue before final deployment. They have a 24-hour window to flag anything egregious (wrong branding, inappropriate image, broken layout). 99% pass without issue. This is the quality checkpoint that prevents mistakes at scale without slowing down the pipeline.

### Step 6: Go-Live & Activation
**Scene**: Everything goes live. The practice has a best-in-class digital storefront.
**Action**: The concierge says: "You're live! Here's what just happened:"
**System Response**:
- Website deployed at [practice-url].com
- Google Business Profile updated and verified
- Instagram and Facebook profiles refreshed with first week of content scheduled
- Contact forms active, connected to [CRM], auto-responses enabled
- "I'll be here 24/7 if anything changes — a new team member, updated hours, a special event. Just tell me and I'll handle everything."
**User Reaction**: The practice is pleasantly surprised at how painless it was. The office manager bookmarks the concierge link. The first real test comes when they have a change to report.
**Critical Detail**: The concierge sends a follow-up 1 week later: "Your new site has had [X] visitors this week. [Y] people submitted appointment requests. Here's how you compare to other [specialty] practices in the network." This first data delivery reinforces the value and builds the habit of checking in.

### Step 7: Ongoing Change Management (The Moment of Truth)
**Scene**: Two months after go-live, a dentist leaves the practice.
**Action**: The office manager opens the concierge (which she now checks semi-regularly) and types: "Dr. Johnson is leaving at the end of the month."
**System Response**: The concierge immediately responds: "Got it. Here's everything I'll update when Dr. Johnson's last day arrives:
- ✅ Remove from website team page and provider bios
- ✅ Update Google Business Profile providers list
- ✅ Remove from social media bios
- ✅ Update any active ad campaigns featuring Dr. Johnson
- ✅ Draft a social post welcoming your next provider (when you're ready)
- 📋 Things for your team: update in-office signage, notify referring practices
Should I schedule these changes for [date], or do you need them sooner?"
**User Reaction**: The office manager is relieved — she was dreading the "7 things to update" cascade. She confirms the date. On Dr. Johnson's last day, everything updates automatically. She tells the office manager at the practice next door about the concierge.
**Critical Detail**: This is the moment where the concierge proves its ongoing value. If this interaction is smooth, the practice becomes a long-term engaged user. The proactive suggestion list ("here's what I'll handle + here's what you need to handle locally") demonstrates the agent's opinion and thoroughness. The executive dashboard logs this change event automatically.

### Step 8: Blitz Deploy (Bottom 20% — No Engagement Path)
**Scene**: 30 days after enrollment invitations, ~50 practices haven't engaged with the concierge at all.
**Action**: The system triggers the "Assembly Line" blitz for non-engaged practices. Default storefronts are generated from scraped data + best-guess template selection + SGA brand defaults.
**System Response**: The concierge sends a final message: "Hi [Practice Name] — I went ahead and launched your upgraded digital storefront based on what I found online. Everything is live:
- New website: [URL]
- Updated Google listing
- Fresh social profiles
If you'd like to personalize anything, I'm always here. Otherwise, I'll keep everything running smoothly."
**User Reaction**: Most of the 50 practices never respond — and that's fine. Their storefronts are live, functional, and better than what they had. A few will respond later with corrections, which the concierge handles.
**Critical Detail**: The blitz deploy uses conservative defaults (generic-but-clean template, scraped data only, no assumptions). It's better to have a clean generic storefront than no storefront. The executive dashboard shows these as "baseline — auto-deployed" vs. "enrolled — practice-customized," giving leadership visibility into engagement tiers.

---

## Journey B: HQ Team Experience

### Step 9: Network Dashboard
**Scene**: The VP of Marketing opens the SGA Digital Health Dashboard on Monday morning.
**Action**: She sees the network-wide view:
**System Response**:
- **Enrollment Progress**: 210/260 practices enrolled, 50 blitz-deployed, 0 remaining
- **Digital Health Score**: Network average 78/100 (up from 34/100 at sprint start)
- **Tier Breakdown**: 45 practices "Optimized" (85+), 165 practices "Baseline" (60-84), 50 practices "Auto-deployed" (40-59)
- **Alerts**: 3 practices with review score drops >0.5 stars, 2 practices with staff changes detected via iSolved, 1 practice with website downtime
- **Content Pipeline**: 1,847 social posts scheduled across all practices this week, 12 flagged for QC review
**User Reaction**: She clicks into the 3 practices with review drops to investigate, approves the 12 flagged posts (takes 10 minutes), and exports a network summary for the leadership meeting.
**Critical Detail**: The dashboard is automated — no manual data entry. Every metric pulls from live sources (website analytics, GMB API, social platforms, review aggregators). The health score is a composite of: website performance (speed, mobile, SEO), GMB completeness, social activity, review health, and lead capture status. Practices move between tiers automatically based on their scores.

### Step 10: Batch Operations & Network-Wide Updates
**Scene**: SGA rebrands its patient-facing tagline. Every practice website needs to be updated.
**Action**: The HQ web developer opens the admin interface and updates the tagline in the master template.
**System Response**: The system previews the change across all 260+ practice sites. Shows which sites are affected, what the change looks like on each template variant, and flags any sites where the change might break layout.
**User Reaction**: The developer reviews the preview, confirms, and the update propagates to all 260+ sites within an hour. No per-site manual work. The concierge notifies each practice: "We updated the SGA tagline across all sites — yours is live."
**Critical Detail**: This batch operation capability is what makes the small team viable at scale. Without it, a network-wide change is 260+ individual site updates. With it, it's one change + one review + one deploy. This applies to template updates, compliance changes, seasonal campaigns, and brand refreshes.
