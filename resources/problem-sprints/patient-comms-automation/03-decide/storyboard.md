# Storyboard: The Comms Engine

## Solution Selected
**The Comms Engine** — Autonomous AI Agent Playbook. Each of SGA's 260+ practices gets its own Comms Engine agent instance that executes the full patient communications playbook autonomously: lead response (< 60 seconds), treatment follow-up (3/7/14/30-day cadences), rebooking enforcement, and database reactivation. The agent extends the Practice Concierge, feeds data to the Funnel OS, and limits human involvement to ~20 minutes/week.

Enhanced with graduated autonomy (human-in-the-loop for first 30 days) and protocol-driven playbook design (standardized cadences and compliance rules).

---

## Storyboard (10 Steps)

### Step 1: The Missed Call — Lead Response in 8 Seconds

**Scene**: 10:15 AM, Tuesday. A potential new patient named Maria searches "Invisalign near me," finds SGA's Garner Dental Group in Mississippi, and calls the number on the Google listing. The front desk has two patients checking in and can't pick up. The phone rings 3 times.

**Action**: On the 4th ring, the Comms Engine's voice agent answers: "Thanks for calling Garner Dental Group, this is the scheduling assistant. How can I help you today?" Maria says she's interested in Invisalign. The agent checks the PMS schedule via NexHealth, confirms Dr. Patterson does Invisalign consultations, finds an opening Thursday at 2 PM, and books the appointment. It sends Maria a confirmation text with the address, a link to fill out new patient forms, and a note: "Dr. Patterson is excited to meet you!"

**System Response**: The Comms Engine logs the event to the Funnel OS data layer: Lead Generated (inbound call, source: Google organic) → Connected (agent answered, ring 4) → Booked (Invisalign consult, Thursday 2 PM, est. value: $5,500). Maria enters the pre-appointment nurture sequence: Day -1 reminder text, Day-of morning confirmation. The agent also logs: "Front desk missed call — 2 of 3 calls AI-answered today" for the Practice Nerve Center.

**User Reaction**: Maria has no idea she spoke to AI. She has an appointment, a confirmation text, and forms to fill out. The front desk sees the booking appear in the PMS and the missed-call stat in their end-of-day summary. No action needed.

**Critical Detail**: The voice agent must sound natural and practice-specific — using the practice name, provider names, and correct service offerings. The 8-second response (3 rings + pickup) is the difference between a $5,500 patient and a lost lead. The agent must check real-time PMS availability, not a stale cache.

---

### Step 2: The Unchecked Inbox — Form Fill Rescue

**Scene**: 11:30 PM, Sunday night. A couple in their 40s — David and Sarah — are browsing dental implant options. David fills out the "Request a Consultation" form on an SGA practice's website in Arizona. In the old world, this form sits in an inbox until someone checks it Monday afternoon — or Wednesday. Or never.

**Action**: Within 30 seconds of form submission, the Comms Engine sends David a personalized text: "Hi David, thanks for reaching out to Desert Ridge Dental about implants. Dr. Kim specializes in full-arch restorations and has openings this week. Would Tuesday at 10 AM or Thursday at 3 PM work better? Reply with your preference or call us at [number]." Simultaneously, an email goes out with more detail: Dr. Kim's bio, a patient testimonial about implants, and the same scheduling options.

**System Response**: The Comms Engine received the form webhook, parsed it (name: David, interest: implants, practice: Desert Ridge), matched the interest to the appropriate provider (Dr. Kim — implant specialist), checked PMS availability, and generated the personalized outreach. If David doesn't respond within 4 hours, a follow-up text goes out: "Just checking in — still interested in learning about implant options? Happy to answer any questions." If no response in 24 hours, a final text with a direct booking link.

**User Reaction**: David replies at 7 AM Monday: "Thursday at 3 works." The agent confirms, books in PMS, sends confirmation. When the front desk arrives at 8 AM Monday, the appointment is already booked. The form inbox has zero unworked items.

**Critical Detail**: The 30-second response at 11:30 PM on a Sunday is impossible for any human team. This is the capability gap that only automation fills. The 3-touch sequence (immediate + 4hr + 24hr) is critical — research shows 391% conversion increase with sub-1-minute response. The agent must correctly match patient interest to provider specialty.

---

### Step 3: The Treatment That Almost Disappeared — Day 3 Follow-Up

**Scene**: Wednesday afternoon. A patient named James visited Garner Dental for a routine cleaning. During the visit, Dr. Patterson noticed significant wear on his teeth and recommended a full set of porcelain veneers — approximately $18,000. James said, "That's a lot to think about. Let me talk to my wife." He left without scheduling. In the old world, this is the last anyone would ever mention veneers to James.

**Action**: The Comms Engine captured the treatment presentation from two sources: (1) the treatment plan entered in the PMS by the treatment coordinator, and (2) the AI note-taker in the operatory, which flagged "veneers discussed, patient expressed financial concern, wants to discuss with spouse." Three days later — Saturday morning — James receives a text from "Dr. Patterson": "Hi James, hope you're having a good weekend. I know the veneer conversation was a lot to take in. If you and your wife have any questions, I'd love to chat — no pressure at all. We also have flexible financing options if that helps. Just reply here or call the office."

**System Response**: The Comms Engine's treatment follow-up module was triggered by the PMS event: "Treatment plan created, status: presented, not accepted." The $18K value flagged it as a high-value case, activating the extended cadence: Day 3 (personal check-in), Day 7 (educational content about veneers + before/after photos), Day 14 (financing information + specific monthly payment options), Day 21 (patient testimonial from a similar case), Day 30 (final soft touch). The AI note-taker data enriched the message: it mentioned "financial concern" and "spouse," so the agent included financing options and framed it as a joint decision.

**User Reaction**: James shows his wife the text. She says, "That's actually nice that they followed up." James replies Monday: "Can you send me the financing details?" The agent responds in 45 seconds with a breakdown: "$375/month for 48 months, 0% interest for the first 12." James books the consult for the following week. The TC, who never had to make the uncomfortable follow-up call, sees the booking and prepares the financial presentation.

**Critical Detail**: The message comes "from Dr. Patterson" — personal, warm, specific to what was discussed. The TC never had to feel "salesy" because they never sent it. The AI note-taker data (financial concern, spouse) made the message contextually aware, not generic. The $18K case is now in the pipeline because of a text that cost $0.01 to send.

---

### Step 4: The $20K Case — The 5-Touch Close

**Scene**: Three weeks into the follow-up sequence for a $22,000 full-arch restoration case at a practice in Georgia. The patient, Linda, has received 3 automated touches and hasn't responded to any of them. In the old world, this case is dead. In the Comms Engine world, touches 4 and 5 are where 90% of high-value cases close.

**Action**: Touch 4 (Day 21): The agent sends a video testimonial link from a patient who had the same procedure at an SGA practice — a 2-minute video of someone showing their before/after and talking about how it changed their confidence. The text reads: "Hi Linda, I thought you might enjoy hearing from one of our patients who went through the same procedure. Her story is pretty incredible. [link]"

Touch 5 (Day 28): The agent escalates to a warm phone call. But not from a TC — from the Comms Engine's voice agent. It calls Linda: "Hi Linda, this is the team at [Practice Name]. We noticed you've been thinking about the restoration Dr. Chen recommended. We just wanted to check in and see if there are any questions we can answer. We know it's a big decision." If Linda engages, the agent handles questions and offers to book. If she says "not now," the agent gracefully exits and schedules a 90-day check-in.

**System Response**: The Comms Engine's high-value case protocol activated because treatment value > $15K. This protocol extends the standard 4-touch sequence to 5+ touches and adds voice outreach at touch 4 or 5. The network learning engine has data showing that video testimonials at touch 4 generate 3x the response rate of text-only messages. If Linda books, the Funnel OS records: Treatment Presented → 5 touches → Accepted (Day 28). This data feeds the network model, confirming the 5-touch pattern.

**User Reaction**: Linda watched the video. She calls back the next day and books the procedure. $22,000 in production that would have been lost without the follow-up sequence. The practice didn't lift a finger.

**Critical Detail**: 90% of $20K+ cases close between touch 4 and 5. Most practices give up after touch 1 (if they follow up at all). The Comms Engine doesn't give up — it follows the protocol. The voice call at touch 5 is the agent, not a human — solving the capacity problem. The video testimonial at touch 4 is curated from the SGA network (260 practices = massive testimonial library).

---

### Step 5: The Checkout That Didn't Rebook — Same-Day Recovery

**Scene**: 4:45 PM, Thursday. A hygiene patient, Tom, finishes his cleaning at a practice in Colorado. The front desk is busy with end-of-day checkout. Tom pays, says "I'll call to schedule my next one," and walks out. No 6-month appointment booked. In the old world, Tom disappears into the lapsed patient database.

**Action**: At 5:15 PM — 30 minutes after Tom's appointment ended — the Comms Engine detects that Tom's visit is marked "completed" in the PMS but no future appointment exists. It sends a text: "Hi Tom, thanks for coming in today! Dr. Martinez wants to make sure we get your next cleaning on the calendar. Here are a few openings in October: [3 date/time options]. Just tap to confirm!" If Tom doesn't respond, Day 1: "Just a quick reminder to book your next visit — maintaining every 6 months keeps everything healthy." Day 3: Final nudge with a direct booking link.

**System Response**: The Comms Engine's rebooking module monitors for the trigger: appointment completed + no future appointment within 7 months. The 30-minute delay is intentional — Tom is still in his car, likely checking his phone. The 3 suggested times are pulled from the PMS and filtered for the same provider (continuity) and similar time slot (habit). If Tom books via the link, it writes directly to the PMS. The Funnel OS records: Rebooking Rate for this practice this week.

**User Reaction**: Tom taps "Oct 14, 9 AM" from his car. Appointment booked. The front desk sees it in the PMS the next morning. Tom's 6-month continuity is preserved without anyone making a phone call.

**Critical Detail**: The 30-minute window is the sweet spot — patient remembers the visit, is likely looking at their phone, and hasn't yet forgotten. Same-day recovery is 4x more effective than next-week outreach. The suggested times must be real-time PMS availability, not generic. The booking link must write back to PMS — no manual entry by staff.

---

### Step 6: The Ghost Database — Reactivating 500 Dormant Patients

**Scene**: First week of the month. The Comms Engine runs its monthly reactivation scan across all practices. At a practice in Texas, it identifies 487 patients who haven't been seen in 12+ months. These patients are worth an estimated $6.3M in lifetime value. Currently, zero outreach is happening.

**Action**: The agent segments the 487 patients into tiers:
- **Tier 1** (67 patients): Had active treatment plans when they lapsed. High-value. Get AI voice outbound call + text.
- **Tier 2** (183 patients): Regular hygiene patients who stopped coming. Medium value. Get personalized text sequence.
- **Tier 3** (237 patients): Infrequent visitors, low engagement history. Get email + text with special offer.

Tier 1 calls go out over 2 weeks (5/day to not overwhelm the schedule). The agent calls: "Hi, this is [Practice Name]. We noticed it's been a while since your last visit, and we had some treatment we'd discussed — wanted to check in and see if you'd like to get that back on the calendar. We have openings this month." Tier 2 texts: "Hi [name], it's been a while! Dr. [name] would love to see you for a checkup. Here are some openings: [3 options]." Tier 3: Email with "We miss you" messaging + text follow-up 3 days later.

**System Response**: Over the month, the reactivation campaign produces: 12 Tier 1 patients rebook (18% response), 29 Tier 2 patients rebook (16%), 14 Tier 3 patients rebook (6%). Total: 55 reactivated patients = estimated $71,500 in near-term production. The Funnel OS records reactivation rates by tier, by message type, and by practice — feeding the network learning model. Practices with higher-than-average reactivation rates get their message templates promoted network-wide.

**User Reaction**: The office manager's weekly summary shows: "Reactivation campaign: 55 patients rebooked this month from dormant database. Estimated production: $71,500. No action needed from you — all appointments are on the schedule." The practice owner sees the number in their Funnel OS dashboard.

**Critical Detail**: 487 patients × $12-15K lifetime value = $6M+ sitting dormant. No human team can call 487 patients. The AI can. The tiering is critical — a lapsed implant patient gets a phone call, not a generic text. The 5-calls-per-day pacing prevents schedule overload from sudden reactivation volume.

---

### Step 7: The Office Manager's 20-Minute Monday — The Human Touchpoint

**Scene**: 7:30 AM, Monday. Jennifer, office manager at Garner Dental, opens her phone. She has a message from the Comms Engine — her weekly summary. It arrived at 6 AM, formatted as a clean summary with 3 sections.

**Action**: Jennifer reads the summary:

**Section 1 — What the Engine Did This Week (no action needed)**:
"Last week: 23 automated messages sent (8 treatment follow-ups, 6 lead responses, 5 rebooking nudges, 4 reactivation texts). Results: 4 new appointments booked from leads, 2 treatment plans accepted from follow-ups ($14,200), 7 patients rebooked, 3 reactivated patients scheduled. Total estimated production influenced: $28,600."

**Section 2 — Items Needing Your Attention (3 items)**:
1. "Patient Robert Chen called back about his crown recommendation but has questions about his insurance coverage. Can you call him today? His number: [xxx]. Context: He was presented a $2,800 crown on March 3, responded to Day 7 follow-up asking about insurance."
2. "Maria Lopez (Invisalign consult Thursday) has a complex medical history flag in her intake form. Dr. Patterson should review before the appointment."
3. "The reactivation campaign identified that Jim Torres has moved to a different state (returned text said 'wrong number, moved to Florida'). Should we update his record or transfer to an SGA practice in Florida?"

**Section 3 — Coming This Week (preview)**:
"This week: 5 treatment follow-ups entering Day 14 (soft close), 12 hygiene recalls going out (5-month reminder), 2 high-value reactivation calls scheduled."

**System Response**: The weekly summary is auto-generated. Section 2 items are things the Comms Engine identified as requiring human judgment — insurance questions, clinical review, and data decisions. These are escalations, not tasks the engine failed at.

**User Reaction**: Jennifer handles the 3 items during her first 20 minutes: calls Robert (5 min), flags Maria's chart for Dr. Patterson (2 min), updates Jim's record and asks the engine to check for SGA practices near his new Florida address (3 min). Done. The rest of the week, the engine runs autonomously.

**Critical Detail**: The 20-minute touchpoint is the ONLY interface. No dashboard to log into, no CRM to check, no inbox to monitor. The summary comes to Jennifer — she doesn't go to it. The 3 items are specific, actionable, and include context so she doesn't have to look anything up. If there were 0 items needing attention, the summary would just say "No action needed this week. Everything's running."

---

### Step 8: The Network Learning Loop — What Works Spreads

**Scene**: End of month. The Comms Engine's network intelligence layer runs its monthly optimization cycle across all 260+ practices.

**Action**: The system analyzes millions of data points: which message templates get the highest response rates, which follow-up cadences produce the most treatment acceptance, which reactivation approaches rebook the most patients, which voice scripts convert best. It discovers patterns:

- Treatment follow-up texts that mention the specific treatment BY NAME get 34% higher response than generic messages
- Saturday morning follow-ups (9-11 AM) get 2.1x the response rate of weekday messages
- For patients over 55, voice calls outperform texts by 3x for reactivation
- Practices that include financing in the Day 3 follow-up (not Day 14) close 22% more treatment
- The phrase "no pressure" in follow-ups increases response by 18%

**System Response**: The network learning engine updates the playbook defaults. All 260 practices now get financing information moved to Day 3. Treatment follow-ups now always include the specific treatment name. Reactivation for 55+ patients prioritizes voice over text. These changes propagate automatically — no practice-level configuration needed.

HQ receives a monthly "Playbook Evolution Report" showing what changed and why. Any practice can opt out of specific changes (guardrails), but defaults always update to the network best.

**User Reaction**: HQ's VP of Operations sees the report: "This month's playbook updates are projected to increase treatment acceptance by 4% network-wide based on A/B testing across 12 pilot practices." The VP doesn't have to act — the improvements are already deployed. They just need to know.

**Critical Detail**: 260 practices is a massive natural experiment. Every message sent is a data point. The learning loop is what makes this system exponentially more valuable than any vendor platform — TrueLark doesn't learn across your specific patient population. The Comms Engine does. Over 12 months, the playbook becomes SGA's most valuable IP — a network-optimized patient communication engine that no competitor can replicate.

---

### Step 9: The Graduated Autonomy — Earning Trust at a New Practice

**Scene**: A new practice acquisition — SGA acquires a 3-operatory family practice in Alabama. Day 1 of Comms Engine deployment. The office manager, Brenda, has never used any automation and is skeptical.

**Action**: The Comms Engine starts in "Supervised Mode." For the first 30 days:
- **Lead response**: Fully automated (no review needed — speed is critical)
- **Treatment follow-up**: Agent drafts messages, sends Brenda a daily digest: "I'd like to send these 3 messages today. Tap ✅ to approve all, or tap any message to edit." If Brenda doesn't respond within 2 hours, messages send automatically (with a note logged).
- **Rebooking**: Fully automated (simple, low-risk messages)
- **Reactivation**: Held until Day 30 review — Brenda sees a preview of the reactivation campaign before it launches.

During the 30-day supervised period, the engine tracks: message quality (did Brenda edit any? which ones? why?), response rates, booking outcomes. After 30 days, the system presents a report: "Over 30 days: 47 messages sent, 3 edited by Brenda (all minor tone adjustments), 0 flagged as inappropriate. Recommend: move to Full Autonomous mode for treatment follow-up." Brenda approves. The engine graduates.

**System Response**: The graduation model has 3 levels: Supervised (human reviews before send), Monitored (auto-sends but human gets daily digest with ability to pause), Full Autonomous (auto-sends, human gets weekly summary only). Each communication type graduates independently. Lead response starts at Monitored. Treatment follow-up starts at Supervised. Rebooking starts at Monitored. Reactivation starts at Supervised.

**User Reaction**: Brenda tells the practice owner: "I was skeptical, but this thing actually sends better messages than I would. And it's following up on treatment plans that we never would have gotten to." By month 2, she's fully autonomous and barely thinks about it. Her 20-minute Monday summary is the only touchpoint.

**Critical Detail**: The graduated autonomy model is the adoption strategy. It doesn't ask practices to trust AI blindly — it earns trust through demonstrated quality. The 30-day supervised period also trains the Comms Engine on this specific practice's communication preferences (Brenda's edits become fine-tuning data). If a practice has a bad experience during supervised mode, the system stays supervised longer — no forced graduation.

---

### Step 10: The CEO's Monthly View — Patient Comms as a Managed System

**Scene**: End of quarter. SGA's CEO opens the Funnel OS executive dashboard. A new section has appeared since the Comms Engine deployment: "Patient Communications Performance."

**Action**: The dashboard shows network-wide comms metrics:
- **Lead Response**: 97.3% of leads contacted within 60 seconds (up from ~40% pre-Engine). Conversion rate: 38% (up from 12%).
- **Treatment Follow-Up**: 100% of presented treatment plans receive automated follow-up (up from < 5%). Treatment acceptance rate: 47% (up from 31%). Additional production attributable to follow-up: $2.4M this quarter.
- **Rebooking**: 94% of patients leave with a future appointment (up from 68%). Same-day rebooking recovery: 82% success rate.
- **Reactivation**: 1,840 dormant patients reactivated this quarter across the network. Estimated lifetime value recovered: $24M pipeline.
- **Efficiency**: Average human time per practice: 18 minutes/week. Cost per practice: $47/month (Twilio + NexHealth + compute). Compare to vendor alternative: $189-299/month per practice.

Below the metrics, a trend chart shows month-over-month improvement in each category — the network learning loop making the playbook better every month.

**System Response**: Every number is drillable. The CEO clicks "Treatment Follow-Up" and sees a practice-by-practice heatmap. Green practices are above the network average. Three red practices stand out — all newly acquired within the last 60 days, still in supervised mode. The system notes: "3 practices in supervised mode; on track to graduate to autonomous by Month 3."

**User Reaction**: The CEO shares the dashboard in the quarterly investor call with the Thurston Group. "We've deployed an autonomous patient communications engine across all 260 practices. Treatment acceptance is up 16 points. We've reactivated 1,840 dormant patients worth $24M in lifetime pipeline. The system costs $47 per practice per month and requires 18 minutes of staff time per week. And it gets smarter every month." The investor response: "How fast can you scale this to the next 100 acquisitions?"

**Critical Detail**: The final metric that matters: $47/month per practice vs. $189-299/month for vendor platforms. At 260 practices, that's $145K/year vs. $590K-$930K/year. The Comms Engine costs less AND does more — because it's built on commodity infrastructure (Twilio, NexHealth) with proprietary intelligence on top. And unlike vendor platforms, the intelligence is SGA's IP — the network learning model trained on SGA's patient population. This is a competitive moat.
