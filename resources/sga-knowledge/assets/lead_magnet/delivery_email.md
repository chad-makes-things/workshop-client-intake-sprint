# Delivery Email: Practice Partnership Assessment

**Email type:** Lead magnet delivery (immediate send on form submission)
**Sequence tag:** `pa_assessment_complete`
**Sender name:** SGA Dental Partners
**Sender email:** partnerships@sgadental.com
**Trigger:** Form submission on sgadental.com/assessment
**Sets up:** PA-Indoctrination email sequence (5 emails, Days 0-7)

---

## Subject Line + A/B Variants

**Primary (A):**
Your Practice Partnership Assessment is ready

**Variant B:**
The 22 questions every practice owner should ask before signing anything

**Variant C:**
Here is your assessment -- start with Category 5

**Testing notes:**
- Variant A is straightforward delivery. Best for high-trust audiences who expect the download.
- Variant B leads with the due diligence checklist -- appeals to the skeptical, research-oriented owner who may be more motivated by protecting themselves than evaluating themselves.
- Variant C creates curiosity by pointing to a specific section (Personal Readiness), which is the most emotionally resonant category. Works best if the subject line piques the "what am I avoiding?" instinct.
- Split test A vs. B first (delivery vs. curiosity). Winner vs. C in round two.

---

## Email Body

**Preview text:** A self-evaluation scorecard for practice owners who want clarity, not a sales pitch.

---

Hi [First Name],

Your copy of *The Practice Partnership Assessment* is attached.

**[Download Your Assessment]** (button -- links to PDF)

This is a self-evaluation scorecard across six dimensions: financial health, operational burden, clinical trajectory, market position, personal readiness, and partnership fit. It takes about 15 minutes to complete -- and it was built to be useful whether you ever talk to SGA or anyone else.

A few things worth knowing before you start:

**1. Be honest with yourself.** The scoring only works if you answer as things are, not as you wish they were. Nobody sees your results but you.

**2. Do not skip Category 5.** Financial health and market position get all the attention in practice evaluations. But Personal Readiness -- your burnout level, your succession plan, your emotional relationship with the idea of sharing the load -- is where most owners discover something they have been quietly carrying for years.

**3. The due diligence checklist on pages 18-22 is the most important section.** Twenty-two questions to ask any DSO, IDSO, or equity partner who approaches you. These are the questions that expose clawback provisions, autonomy erosion, and equity traps before you sign anything. Print that section. Keep it in your desk drawer. Use it every time a broker calls.

---

**What happens next:**

Over the next week, I am going to send you a short series of emails that go deeper on the topics the assessment covers. Not sales pitches. Perspectives -- from someone who has watched hundreds of practice owners navigate this decision.

Tomorrow, we will start with the question every practice owner asks at 2 AM. It is not the question you think.

If any of those emails are not for you, unsubscribe with one click at the bottom. No hard feelings. We would rather have a small list of people who want to hear from us than a large list of people who do not.

---

Talk soon,

**The SGA Dental Partners Team**
Backed by the Thurston Group | 38 Years | 8 Dental & Healthcare Platforms | $4B+ Returned to Partners

P.S. If you already know where you stand and would rather skip the emails and have a direct conversation -- we respect that too. You can schedule a confidential call with our partnership team anytime: **[Schedule a Confidential Conversation]** (link to scheduling page). No brokers, no NDAs at this stage, no pressure. Just a conversation between professionals.

---

## Technical Specifications

**Send timing:** Immediate (within 60 seconds of form submission)

**Attachment:** PDF of the Practice Partnership Assessment (also hosted as a downloadable link for email clients that strip attachments)

**Plain text version:** Include a plain text fallback with the download link for email clients that do not render HTML

**Unsubscribe:** One-click unsubscribe link in footer (CAN-SPAM and GDPR compliant)

**Footer:**
```
SGA Dental Partners
43 Ford Way, Richmond Hill, GA 31324
partnerships@sgadental.com | sgadental.com/partners
You are receiving this email because you requested The Practice Partnership Assessment.
Unsubscribe with one click: [unsubscribe link]
```

**Tags applied on send:**
- `pa_assessment_complete` (already applied at form submission)
- `pa_email_delivered` (applied on successful delivery)

**Tags applied on engagement:**
- `pa_email_opened` (applied on open)
- `pa_pdf_downloaded` (applied on download link click)
- `pa_schedule_clicked` (applied on PS scheduling link click -- triggers high-priority BD alert)

**Conditional logic:**
- If `pa_schedule_clicked` is applied, suppress from PA-Indoctrination sequence and route directly to BD team for personal follow-up within 24 hours
- If email bounces, tag `pa_email_bounced` and trigger a re-send attempt after 2 hours with alternate subject line

---

## A/B Test Configuration

| Element | Variant A | Variant B | Variant C |
|---------|-----------|-----------|-----------|
| Subject line | Your Practice Partnership Assessment is ready | The 22 questions every practice owner should ask before signing anything | Here is your assessment -- start with Category 5 |
| Preview text | A self-evaluation scorecard for practice owners who want clarity, not a sales pitch. | A due diligence checklist built on 38 years of dental partnership experience. | The category most practice owners skip is the one that matters most. |
| Primary metric | Open rate | Open rate | Open rate |
| Secondary metric | PDF download click rate | PDF download click rate | PDF download click rate |
| Split | 33% / 33% / 34% | -- | -- |
| Winner criteria | Highest open rate after 500 sends or 48 hours (whichever comes first) | -- | -- |
| Auto-send winner | Yes -- remaining list receives winning variant | -- | -- |

---

## Sequence Handoff

This email is Day 0 of the PA-Indoctrination sequence. The next email in the sequence fires on Day 1:

**PA-Indoctrination Email 2:**
- Subject line direction: "The question every practice owner asks at 2 AM"
- Goal: Empathy and identification
- Key elements: Acknowledge the late-night googling, mirror their exhaustion, introduce the Gen4 philosophy
- Trigger: 24 hours after delivery email send (regardless of open)

The full PA-Indoctrination sequence (5 emails, Days 0-7) and PA-Engagement sequence (8 emails, Days 8-35) are documented in the funnel architecture at `/sgadental/marketing/strategy/funnel_architecture.md`.
