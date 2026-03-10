# SGA Dental Partners -- Conversational Marketing Funnel (Chatbot Flow)

**Prepared:** March 2026
**Status:** Ready for implementation
**Audiences:** B2B (practice owners, specialists, associates) + B2C (patients)
**Platforms:** Website widget (sgadental.com), MODIS patient widget (modisdental.com / idspringfield.com), Facebook Messenger, Instagram DM
**Brand archetype:** Sage (knowledgeable, helpful, never pushy) + Caregiver (protective, supportive)

---

## Table of Contents

1. [Bot Canvas](#1-bot-canvas)
2. [Entry Points and Platform Configuration](#2-entry-points-and-platform-configuration)
3. [Initial Routing](#3-initial-routing)
4. [B2B Flow: Dental Professionals](#4-b2b-flow-dental-professionals)
5. [B2C Flow: Patients](#5-b2c-flow-patients)
6. [Fallback and Recovery System](#6-fallback-and-recovery-system)
7. [Lead Scoring Model](#7-lead-scoring-model)
8. [Handoff Protocols](#8-handoff-protocols)
9. [Rich Media Moments](#9-rich-media-moments)
10. [Integration Specifications](#10-integration-specifications)
11. [Platform-Specific Adaptations](#11-platform-specific-adaptations)
12. [Performance Metrics](#12-performance-metrics)

---

## 1. Bot Canvas

| Section | Definition |
|---------|-----------|
| **Bot Name** | "SGA Guide" (internal reference). No named persona displayed to users -- appears as "SGA Dental Partners" on the B2B widget and as the practice name (e.g., "Innovative Dental") on patient-facing widgets. |
| **Persona** | Sage archetype: knowledgeable, direct, evidence-backed, never pushy. Speaks like a trusted colleague who has seen every question before and respects your time. |
| **Core Value Proposition** | Get the right information or the right person in under 2 minutes -- no forms, no hold music, no runaround. |
| **Primary Users** | (1) Practice owners exploring partnership, (2) Specialist dentists evaluating MODIS, (3) Associate dentists exploring careers, (4) Implant patients researching treatment, (5) General dentistry patients seeking appointments |
| **Conversation Topics** | Partnership exploration, career opportunities, implant information, appointment scheduling, insurance questions, location finder |
| **Integrations** | CRM (HubSpot/Salesforce), email marketing (ActiveCampaign/Mailchimp), scheduling (Calendly for B2B, practice PMS for B2C), Google Sheets (lead log backup), Slack (team notifications) |
| **Escalation Rules** | Human handoff on: explicit request, frustration detected, high-value lead ready now, complex clinical questions, pricing negotiation |
| **Success Metrics** | Engagement rate > 5%, completion rate > 45%, chat-to-lead rate > 20%, chat-to-meeting/booking rate > 8% |
| **Out of Scope** | Contract negotiation, specific deal terms/multiples, clinical diagnosis, prescription advice, insurance claims resolution, billing disputes |

---

## 2. Entry Points and Platform Configuration

### Entry Point 1: SGA Dental Partners Website Widget (sgadental.com)

**Placement:** All pages. Persistent bottom-right widget icon.

**Trigger Logic:**

| Trigger | Page Context | Opening Message | Delay |
|---------|-------------|-----------------|-------|
| Time-based | Homepage | "Welcome to SGA Dental Partners. Are you a dental professional or a patient? I can point you in the right direction." | 15 seconds |
| Page-specific | /partners or /partnership | "Exploring what partnership looks like? I can answer your initial questions or connect you with our team." | 8 seconds |
| Page-specific | /careers or /join-us | "Looking for your next career move? I can help you find the right fit across our network." | 8 seconds |
| Page-specific | /locations or /find-a-dentist | "Need help finding a practice near you? Tell me your city or zip code." | 10 seconds |
| Scroll-based | Blog/content pages | "Have a question about what you're reading? I'm here if you need anything." | 50% scroll depth |
| Return visitor | Any page (recognized via cookie) | "Welcome back. Anything I can help with today?" | 10 seconds |
| Exit intent | /partners, /careers | "Before you go -- would you like me to send you our Partnership Guide or open positions by email?" | On exit signal |

### Entry Point 2: MODIS / Practice Patient Widget

**Placement:** Implant pages, patient-facing pages on modisdental.com, idspringfield.com, and individual practice sites.

**Trigger Logic:**

| Trigger | Page Context | Opening Message | Delay |
|---------|-------------|-----------------|-------|
| Time-based | Implant service page | "Thinking about dental implants? I can help you understand your options -- no pressure, just information." | 12 seconds |
| Time-based | General service pages | "Looking to schedule an appointment or have a question? I can help." | 15 seconds |
| Page-specific | Pricing/cost page | "Have questions about cost or financing? I can walk you through what to expect." | 8 seconds |
| Page-specific | Before/after gallery | "See a transformation that gives you hope? I can help you take the first step." | 10 seconds |
| Scroll-based | Testimonial/review page | "Ready to start your own story? I can connect you with our team." | 60% scroll depth |
| Exit intent | Implant pages | "Before you go -- would you like a free Smile Assessment? It takes 2 minutes and gives you a personalized look at your options." | On exit signal |

### Entry Point 3: Facebook Messenger

**Trigger:** Linked from Meta ads (B2B and B2C), Facebook page CTA button, and organic post CTAs.

**Ad-Specific Opening Messages:**

| Ad Campaign | Messenger Opening |
|-------------|------------------|
| B2B practice partnership ad | "Thanks for your interest in SGA Dental Partners. I have a few quick questions to make sure I connect you with the right person on our team." |
| B2C implant ad (Dr. Olson content) | "Thanks for reaching out! Are you interested in learning more about dental implants, or would you like to schedule a consultation?" |
| B2C general dentistry ad | "Hi there! Are you looking for a dentist near you, or do you have a specific question I can help with?" |
| Associate recruitment ad | "Thanks for checking us out! Are you looking for open positions, or do you want to learn more about what it's like to work at SGA?" |

**Organic Entry (Facebook page CTA):**
Uses the same initial routing question as the website widget (see Section 3).

### Entry Point 4: Instagram DM Auto-Reply

**Trigger Keywords and Responses:**

| Keyword(s) in DM | Auto-Reply |
|-------------------|-----------|
| "partnership," "partner," "sell practice," "DSO" | "Thanks for reaching out about partnership. I'd love to learn a bit about your practice so I can connect you with the right person. Are you a practice owner, associate, or specialist?" |
| "implants," "teeth," "dentures," "smile" | "Thanks for your message! Are you looking for information about dental implants, or would you like to schedule a consultation? I can help either way." |
| "jobs," "hiring," "career," "associate," "work" | "We'd love to tell you about opportunities at SGA. Are you a dentist, hygienist, or looking for a front office role? I'll point you in the right direction." |
| "location," "appointment," "dentist near me" | "I'd be happy to help you find a practice. What city or zip code are you in?" |
| Any other message | "Thanks for reaching out to SGA Dental Partners! I can help with partnership inquiries, career opportunities, or patient questions. Which of those brings you here today?" |

**Instagram Story Reply Triggers:**
When an Instagram story includes a question sticker or poll related to dental topics, DM replies to that story receive the keyword-matched response above.

---

## 3. Initial Routing

The first interaction determines which conversation branch the user enters. This is the most critical moment -- get them to the right flow fast.

### Routing Question

```
BOT: Welcome to SGA Dental Partners. I'm here to help -- how can I point
     you in the right direction?

     A) I'm a dental professional exploring partnership or career opportunities
     B) I'm a patient looking for dental care or implant information
     C) Something else

     [Button: A - Dental Professional]
     [Button: B - Patient]
     [Button: C - Something Else]
```

### Routing Logic

```
User selects A (Dental Professional)
  --> B2B FLOW (Section 4)

User selects B (Patient)
  --> B2C FLOW (Section 5)

User selects C (Something Else)
  --> CLARIFICATION SUB-FLOW:

     BOT: No problem. I can help with a few things:
          - Partnership inquiries (for dental practice owners and specialists)
          - Career opportunities (for dentists, hygienists, and staff)
          - Patient appointments and dental care information
          - Media or press inquiries
          - Vendor or supplier inquiries

          Which of those is closest?

          [Button: Partnership]    --> B2B Owner/Specialist sub-flow
          [Button: Careers]        --> B2B Associate/Staff sub-flow
          [Button: Patient Care]   --> B2C Flow
          [Button: Media/Press]    --> MEDIA HANDOFF (see Section 6)
          [Button: Vendor]         --> VENDOR HANDOFF (see Section 6)

User types free text instead of selecting a button
  --> AI INTENT CLASSIFIER routes to appropriate flow based on keywords:
      - "sell," "practice," "valuation," "partner" --> B2B Owner flow
      - "job," "career," "associate," "hiring" --> B2B Associate flow
      - "implant," "teeth," "appointment," "dentist" --> B2C flow
      - Unclear intent --> Re-prompt with button options
```

### Shortcut: Skip to Action

On every screen after the initial routing, a persistent option is available:

```
[Quick Action: Skip to scheduling]     --> Calendly (B2B) or practice scheduler (B2C)
[Quick Action: Talk to a person]       --> Human handoff (see Section 8)
[Quick Action: Send me info by email]  --> Email capture + relevant PDF delivery
```

---

## 4. B2B Flow: Dental Professionals

### 4.1 B2B Sub-Routing

```
BOT: Great -- welcome. To make sure I connect you with the right information,
     are you a:

     A) Practice owner or partner considering a partnership
     B) Associate dentist exploring career opportunities
     C) Specialist (periodontist, oral surgeon, implantologist) exploring partnership
     D) Office manager or staff member with a question

     [Button: A - Practice Owner]
     [Button: B - Associate Dentist]
     [Button: C - Specialist]
     [Button: D - Office Manager/Staff]
```

---

### 4.2 Practice Owner Flow (Dr. Mark Hendricks Path)

**Goal:** Qualify the owner, identify their primary pain, deliver value, and route to BD team or lead magnet.

#### Decision Tree

```
Q1: Role Confirmation
  "Practice owner or partner"
    |
    v
Q2: Practice Size
  "How many operatories does your practice have?"
  [1-3] [4-6] [7-10] [10+]
    |
    v
Q3: Biggest Challenge
  "What is the single biggest challenge you are dealing with right now?"
  [Admin burden]  [Growth ceiling]  [Succession planning]
  [Staffing]      [Clinical development]  [Something else]
    |
    v
Q4: Partnership Awareness
  "Have you explored DSO or partnership options before?"
  [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
    |
    v
Q5: Timeline
  "What is your timeline for making a change?"
  [Just exploring]  [6-12 months]  [Ready to have a real conversation]
    |
    v
ROUTING LOGIC:
  - "Ready now" + any practice size --> IMMEDIATE BD HANDOFF
  - "6-12 months" + 4+ ops        --> OFFER: Confidential Partnership Guide + book a call
  - "6-12 months" + 1-3 ops       --> OFFER: Practice Growth Resources + email nurture
  - "Just exploring" + any size    --> OFFER: Practice Partnership Assessment + email nurture
  - "New to me" + any timeline     --> OFFER: "7 Out of 10 Dentists Regret..." article + assessment
```

#### Response Scripts

**Q1 Response (after role confirmation):**

```
BOT: Glad to have you here. SGA Dental Partners is a partnership platform
     backed by the Thurston Group -- 38 years, 8 dental and healthcare
     platforms, $4B+ returned to partners. We are built differently than
     what you may have heard about DSOs.

     Let me ask a few quick questions so I can point you to the most
     relevant information. This takes about 90 seconds.

     How many operatories does your practice have?

     [1-3]  [4-6]  [7-10]  [10+]
```

**Q3 Response Scripts (per challenge selected):**

**Admin Burden:**
```
BOT: That is the number one thing we hear. As one of our partners, Dr.
     Godat, put it: "SGA supports us and manages things behind the scenes.
     Our partnership allows us to focus on changing lives."

     We handle billing, payroll, HR, compliance, recruiting, and procurement
     -- so you can get back to the chair.

     Quick question: have you explored DSO or partnership options before?

     [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
```

**Growth Ceiling:**
```
BOT: Growing beyond a certain point as a solo owner is brutally hard --
     the capital, the recruiting, the marketing. You hit a ceiling because
     there are only so many hours in a day.

     SGA's three-vertical platform gives partners access to technology
     investment, a recruiting engine, and a peer community of 100+
     clinicians who push each other to think bigger.

     Have you explored DSO or partnership options before?

     [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
```

**Succession Planning:**
```
BOT: That is a conversation more dentists need to have earlier than they
     think. Too many wait until they are burned out or their practice value
     has started to decline.

     SGA's equity partnership model gives you a liquidity event today while
     keeping you active in clinical work as long as you want. No forced
     retirement date. No "earn-out and leave" structure.

     Have you explored DSO or partnership options before?

     [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
```

**Staffing:**
```
BOT: You are not alone -- over 76% of dental practices report "extremely
     challenging" hygienist recruiting. Solo practices just cannot compete
     with the benefits and career pathways that larger platforms offer.

     SGA provides 401(k) through Voya, Cigna healthcare, PTO, CE stipends,
     and a dedicated recruiting team. Our staff retention rate across the
     network is 85%.

     Have you explored DSO or partnership options before?

     [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
```

**Clinical Development:**
```
BOT: This is where SGA is genuinely different. We are the only dental
     partnership with a built-in path from general dentistry to advanced
     implantology -- through the Pikos Institute, our annual 4Front
     symposium, and one-on-one clinical mentorship.

     One of our partners was in a "spot of complacency" after 20 years.
     After joining, he learned all-on-four procedures, grew revenue 16.5%,
     and said: "It is fun again."

     Have you explored DSO or partnership options before?

     [Yes, actively looking]  [Looked into it casually]  [No, this is new to me]
```

**Something Else (free text):**
```
BOT: Tell me a bit more about what you are dealing with -- I want to make
     sure I point you to the right resources.

     [Free text input]

     --> AI classifies into one of the above categories or routes to
         general resources if unmatched.
```

**Q4 Response Scripts (per awareness level):**

**"Yes, actively looking":**
```
BOT: Good -- then you already know the landscape. You have probably heard
     the stat: 7 out of 10 dentists report being frustrated or regretful
     after a DSO sale. We built SGA specifically because of that number.

     The short version: your name stays on the door, your team stays in the
     chairs, you retain equity in the combined platform, and you get access
     to a peer community and education system no other DSO can match.

     What is your timeline for making a change?

     [Just exploring for now]  [6-12 months]  [Ready to have a real conversation]
```

**"Looked into it casually":**
```
BOT: Most dentists start with a few late-night Google searches and some
     conversations with peers. That is completely normal.

     Here is what makes SGA different in two sentences: We are backed by
     the Thurston Group (38 years, $4B+ returned, 8 dental platforms). And
     our partners keep their name, their team, and their autonomy -- while
     owning equity in something far larger than a solo practice.

     What is your timeline for making a change?

     [Just exploring for now]  [6-12 months]  [Ready to have a real conversation]
```

**"No, this is new to me":**
```
BOT: That is completely fine -- no pressure here. Let me give you a
     quick orientation.

     A DSO partnership means you get operational support (billing, HR,
     payroll, compliance, recruiting) and a liquidity event -- while keeping
     your practice name, your team, and your clinical autonomy.

     Not all partnerships are the same, though. SGA is what is called a
     "4th generation" model: clinician-focused, peer-driven, backed by a PE
     firm with a 38-year dental track record. The opposite of the corporate
     cookie-cutter model you may have heard about.

     What is your timeline for thinking about this?

     [Just exploring for now]  [Maybe 6-12 months]  [Sooner than I expected]
```

**Q5 Routing and Final Response Scripts:**

**"Ready to have a real conversation" (ANY practice size):**
```
BOT: Let's make that happen. Our business development team can have a
     confidential, no-obligation conversation with you -- no NDAs at this
     stage, no commitments. Just professionals talking.

     I can get you scheduled this week. What works better for you?

     [Book a call now]  --> Opens Calendly embed (BD team calendar)
     [Have someone reach out to me]  --> LEAD CAPTURE (name, email, phone, practice name)

     ** SYSTEM ACTION: Tag as `pa_hot_lead`. Slack notification to BD team
     immediately. CRM record created with conversation transcript. **
```

**"6-12 months" + 4+ operatories:**
```
BOT: Smart timing -- most of our best partnerships start with a
     conversation 6-12 months before the actual transition.

     Two things I would recommend:

     1. Download our Confidential Partnership Guide -- it walks through
        the equity model, what changes and what does not, and partner
        stories. [Download the Guide]

     2. Book a no-pressure introductory call when you are ready. No
        timeline, no commitment. [Book a Call]

     Can I send the guide to your email?

     [Yes, send it to me]  --> LEAD CAPTURE (name, email, practice name, state)
     [I will download it now]  --> Direct PDF link
     [Book a call instead]  --> Calendly embed

     ** SYSTEM ACTION: Tag as `pa_tier_growth`. Enter PA-Indoctrination
     email sequence. CRM record created. **
```

**"6-12 months" + 1-3 operatories:**
```
BOT: SGA partners with practices of all sizes, and we are happy to have
     a conversation whenever the time is right.

     In the meantime, I would recommend our Practice Partnership Assessment
     -- it is a quick self-evaluation that helps you gauge where you are on
     the readiness spectrum. No sales pitch, just useful perspective.

     [Take the Assessment]  --> Links to assessment landing page
     [Send me information by email]  --> LEAD CAPTURE (name, email)

     ** SYSTEM ACTION: Tag as `pa_tier_foundation`. Enter nurture sequence
     (educational content, not sales). CRM record created. **
```

**"Just exploring" (ANY practice size):**
```
BOT: No rush at all. The best time to learn about your options is before
     you need them.

     Our Practice Partnership Assessment takes about 3 minutes and gives
     you a readiness score across operational, financial, growth, and
     personal dimensions. It is private -- nobody sees your answers but you
     (and us, if you choose to share).

     [Take the Assessment]  --> Links to assessment landing page
     [Just send me something to read]  --> LEAD CAPTURE (name, email) + sends
        "7 Out of 10 Dentists Regret..." article link

     ** SYSTEM ACTION: Tag as `pa_assessment_lead` or `pa_content_lead`.
     Enter PA educational nurture. CRM record created. **
```

---

### 4.3 Specialist Flow (Dr. James Whitfield Path)

**Goal:** Qualify the specialist, surface MODIS-specific value, route to Marcus Morosin (VP BD) or specialist content.

#### Decision Tree

```
Q1: Specialty Confirmation
  "What is your specialty?"
  [Periodontics]  [Oral Surgery]  [Implantology/Prosthodontics]  [Other Specialty]
    |
    v
Q2: Practice Situation
  "Do you currently own your practice, or are you an associate at a specialty group?"
  [Owner]  [Associate]  [Partner in a group]
    |
    v
Q3: Biggest Challenge
  "What is the biggest challenge you are facing right now?"
  [Capital for growth]  [Recruiting associates]  [Marketing/patient acquisition]
  [Succession planning]  [Competing with larger groups]  [Something else]
    |
    v
Q4: Familiarity with MODIS
  "Are you familiar with MODIS Dental Partners or the Pikos Institute?"
  [Yes, I know MODIS/Pikos]  [I have heard of Pikos but not MODIS]  [Neither]
    |
    v
Q5: Timeline
  "Where are you in your thinking about a partnership or platform?"
  [Actively evaluating options]  [Starting to research]  [Early -- just learning]
    |
    v
ROUTING LOGIC:
  - "Actively evaluating" + Owner --> IMMEDIATE HANDOFF to Marcus Morosin
  - "Actively evaluating" + Associate/Partner --> OFFER: call with BD team
  - "Starting to research" + any --> OFFER: Specialist's Guide + call option
  - "Early" + any --> OFFER: Specialist's Guide + MODIS Community Conversations podcast
```

#### Response Scripts

**After Specialty Confirmation:**
```
BOT: Welcome. MODIS Dental Partners -- our specialty vertical within SGA
     -- was built specifically for [periodontists / oral surgeons /
     implantologists] by people who understand your world.

     Dr. Michael Pikos is a founding partner, and the Pikos Institute is
     integrated into the platform. This is the only specialty-focused DSO
     with that kind of clinical credibility.

     A couple of quick questions so I can connect you with the most
     relevant information.
```

**Challenge: Capital for Growth:**
```
BOT: You are not the only specialist who can see the opportunity but
     cannot reach it alone. A CT scanner, a surgical suite, an in-house
     lab, a marketing engine -- priced out individually, that is $500K+
     in capital.

     MODIS provides that infrastructure so you can focus on what you do
     best: clinical work. Your skills deserve an infrastructure to match.

     Are you familiar with MODIS Dental Partners or the Pikos Institute?
```

**Challenge: Competing with Larger Groups:**
```
BOT: The full-arch market is exploding, and the practices with the
     marketing machine, the in-house lab, and the YouTube channel are
     capturing that demand. A solo specialist with a referral pad cannot
     compete at that level alone.

     MODIS was built to solve exactly that problem -- capital, marketing,
     recruiting, and a peer network of elite implantologists, all while
     you keep your clinical autonomy and practice identity.

     Are you familiar with MODIS or the Pikos Institute?
```

**Familiarity: "I know Pikos but not MODIS":**
```
BOT: Then you already know the clinical standard MODIS is built on.
     Dr. Pikos chose to bring the Institute into MODIS because he saw an
     opportunity to build a specialty platform that matched his clinical
     philosophy -- excellence, education, and collaboration.

     MODIS is backed by the Thurston Group, the same PE firm behind Smile
     Doctors, US Endodontics Partners, and US Oral Surgery Management.
     38 years, $4B+ returned to partners. This is not their first
     specialty dental platform.

     Where are you in your thinking about a partnership?

     [Actively evaluating options]  [Starting to research]  [Early -- just learning]
```

**"Actively evaluating" + Owner:**
```
BOT: Then the most valuable thing I can do is connect you directly with
     Marcus Morosin, our VP of Business Development. He works exclusively
     with specialists and can have a confidential conversation about what
     partnership would look like for your specific practice.

     No NDAs at this stage. No pressure. Just a conversation between
     professionals who speak the same language.

     [Book a call with Marcus]  --> Calendly (Marcus Morosin's calendar)
     [Have Marcus reach out to me]  --> LEAD CAPTURE (name, email, phone, practice, specialty)

     ** SYSTEM ACTION: Tag as `sp_hot_lead`. Immediate Slack notification
     to Marcus Morosin. CRM record with full conversation transcript. **
```

**"Starting to research":**
```
BOT: Good place to be. I would recommend two resources:

     1. "The Specialist's Guide to DSO Partnership" -- a 20-page white
        paper that covers valuation differences for specialty practices,
        the capital gap, scope preservation, and partner case studies.
        [Download the Guide]

     2. The MODIS Community Conversations Podcast -- hear directly from
        partners at Pensacola Perio, Troy Family Dental, and others about
        what changed after partnering. [Listen Now]

     Want me to send the guide to your email?

     [Yes, send it over]  --> LEAD CAPTURE (name, email, specialty, practice name)
     [I will grab it myself]  --> Direct link

     ** SYSTEM ACTION: Tag as `sp_guide_download`. Enter SP-Indoctrination
     sequence. CRM record created. **
```

---

### 4.4 Associate Dentist Flow (Dr. Priya Nair Path)

**Goal:** Understand what the associate is looking for, connect them to open positions or career resources.

#### Decision Tree

```
Q1: Confirmation
  "Are you a dentist, hygienist, or looking for a front office/admin role?"
  [Dentist]  [Hygienist]  [Front Office/Admin]
    |
    v
  If Dentist:

Q2: What Matters Most
  "What matters most to you in your next position?"
  [Clinical mentorship and growth]  [Compensation and benefits]
  [Work-life balance]  [Path to ownership/equity]  [Location]
    |
    v
Q3: Experience Level
  "How long have you been practicing?"
  [Still in school/residency]  [0-3 years]  [4-10 years]  [10+ years]
    |
    v
Q4: Geography
  "Do you have a location preference?"
  [Southeast US]  [Southwest US]  [Midwest]  [West Coast]  [Open to anything]
    |
    v
ROUTING:
  Match available openings --> Show 1-3 relevant positions
  No match --> Offer to be notified when positions open + career guide download
```

#### Response Scripts

**After "Clinical mentorship and growth":**
```
BOT: You are in the right place. SGA is the only dental partnership with
     a built-in path from general dentistry to advanced implantology.

     Our associates get paired with clinical mentors, access to the Pikos
     Institute for hands-on training, and an annual peer learning symposium
     called 4Front. One of our associates went from doing bread-and-butter
     fillings to placing full-arch implants -- and said "I love my
     profession again."

     This is not a production treadmill. Your career is not a number here.

     How long have you been practicing?
```

**After "Path to ownership/equity":**
```
BOT: Most DSOs talk about a "pathway to partnership" without ever defining
     it. Here is the reality at SGA: new doctors can invest in the company
     immediately as a Class A shareholder. That is not a recruiting line --
     it is a real equity position from day one.

     How long have you been practicing?
```

**Position Matching Response (when matches exist):**
```
BOT: Based on what you have told me, here are the positions that match
     your interests:

     1. [Position Title] -- [Practice Name], [City, State]
        Highlights: [mentorship program / implant training / etc.]
        [View Details]

     2. [Position Title] -- [Practice Name], [City, State]
        Highlights: [benefits / equity / etc.]
        [View Details]

     Want to apply directly, or would you like to talk to someone on
     our team first?

     [Apply Now]  --> Links to application
     [Talk to someone first]  --> LEAD CAPTURE + recruiter notification
     [Send me the Career Guide too]  --> LEAD CAPTURE + PDF delivery
```

**No Position Match:**
```
BOT: I do not have an opening that matches your exact criteria right now,
     but our network is growing and new positions open regularly.

     Two things I can do:

     1. Notify you when a matching position opens.
        [Yes, keep me posted]  --> LEAD CAPTURE (name, email, specialty, geography pref)

     2. Send you "The Associate Dentist's Career Decision Guide" -- it
        covers solo vs. DSO vs. partnership track with real numbers, and
        what to look for in any opportunity.
        [Send the Guide]  --> LEAD CAPTURE + PDF delivery

     ** SYSTEM ACTION: Tag as `dr_passive_candidate`. Enter DR-Passive
     Candidate Nurture (monthly). CRM record created. **
```

---

### 4.5 Office Manager/Staff Flow (Tamara Reeves Path)

**Goal:** Provide reassurance, answer common post-acquisition questions, and route to the right internal team.

```
BOT: Thanks for reaching out. Are you at a practice that recently joined
     the SGA network, or are you looking for career opportunities?

     [My practice recently joined SGA]  [Looking for a job]

If "My practice recently joined SGA":

BOT: Welcome to the SGA family. Transitions can feel uncertain, so I want
     to make sure you have what you need.

     What can I help with?

     [Benefits and payroll questions]
     [Understanding what changes and what stays the same]
     [Who to contact for support]
     [Something else]

If "Benefits and payroll":
BOT: Great question. SGA provides enhanced benefits including Cigna
     healthcare, Mutual of Omaha, 401(k) through Voya, PTO, and CE
     stipends. Payroll transitions to iSolved, which replaces manual
     processing.

     Your PeopleOps team is the best resource for specific questions
     about your benefits enrollment and timeline.

     [Connect me with PeopleOps]  --> Routes to internal support email/form
     [I have another question]    --> Returns to question menu

If "What changes and what stays the same":
BOT: The short answer: your practice keeps its name. Your team stays.
     Your patients will not notice a difference -- except that things run
     smoother behind the scenes.

     What changes: payroll moves to iSolved (no more manual processing),
     HR and compliance get centralized support, and your benefits package
     gets upgraded. What stays: your office culture, your patient
     relationships, your daily workflow.

     As one of our leaders described it: "After onboarding is completed,
     there is an abundance of relief."

     [I have a specific concern]  --> Free text input --> Route to PeopleOps
     [That helps, thank you]      --> Graceful close
```

---

## 5. B2C Flow: Patients

### 5.1 B2C Sub-Routing

```
BOT: I am happy to help. What brings you in today?

     [Dental implants or missing teeth]
     [General dentistry (cleaning, checkup, filling)]
     [Cosmetic dentistry (whitening, veneers)]
     [Emergency (pain, broken tooth, swelling)]
     [I need help finding a location]

     [Button: Implants/Missing Teeth]
     [Button: General Dentistry]
     [Button: Cosmetic]
     [Button: Emergency]
     [Button: Find a Location]
```

---

### 5.2 Implant Patient Flow (Linda Dawson Path)

**Goal:** Empathize, educate, reduce fear, and move toward consultation booking or Smile Assessment.

#### Decision Tree

```
Q1: Current Situation
  "What brings you in today?"
  --> User selected "Dental implants or missing teeth"
    |
    v
Q2: Specific Situation
  "Can you tell me a bit more about your situation?"
  [I wear dentures and want something permanent]
  [I have failing or missing teeth]
  [I have been told I need implants]
  [I am researching options for someone I love]
    |
    v
Q3: Biggest Concern
  "What is your biggest concern right now?"
  [Cost and affordability]
  [Fear of surgery or pain]
  [Not sure which option is right for me]
  [Finding the right doctor]
  [Travel or logistics]
    |
    v
Q4: Location
  "Do you have a specific practice or doctor in mind, or would you like
   help finding one?"
  [I want to see Dr. Olson / Innovative Dental]
  [I want a MODIS location near me]
  [Help me find the right doctor]
    |
    v
Q5: Next Step Preference
  "What would be most helpful for you right now?"
  [Schedule a free consultation]
  [Take the Smile Assessment first]
  [Talk to someone about cost and financing]
  [Send me information to review on my own]
    |
    v
ROUTING:
  "Schedule" --> Appointment booking (practice scheduler or Calendly)
  "Smile Assessment" --> Link to assessment + email capture
  "Cost/financing" --> Financing information + consultation CTA
  "Send info" --> Email capture + PT-Implant Nurture sequence
```

#### Response Scripts

**Q2: "I wear dentures and want something permanent":**
```
BOT: You are not alone in feeling that way. Thousands of patients come to
     us saying the same thing -- the slipping, the adhesive, the foods
     you cannot eat, the moments you hold back from smiling.

     As Dr. Olson says: "If you have to think about your teeth, those
     moments are not there."

     Fixed dental implants can change that. Patients tell us it is like
     getting their life back -- eating a steak, laughing without worrying,
     smiling in every photo.

     Can I ask what your biggest concern is right now?

     [Cost and affordability]
     [Fear of surgery or pain]
     [Not sure which option is right]
     [Finding the right doctor]
     [Travel or logistics]
```

**Q2: "I have failing or missing teeth":**
```
BOT: I understand -- dealing with failing teeth affects so much more than
     your mouth. It touches your confidence, your social life, what you
     eat, how you feel every day.

     The good news: modern dental implants can often be placed and
     restored in a single day. Many of our patients walk in with teeth
     they are ashamed of and walk out the same day with a smile they
     cannot stop showing off.

     What is your biggest concern right now?

     [Cost and affordability]
     [Fear of surgery or pain]
     [Not sure which option is right]
     [Finding the right doctor]
```

**Q3: "Cost and affordability":**
```
BOT: I appreciate you being upfront about that -- cost is the number one
     question we get, and you deserve a straight answer.

     Full-arch dental implants typically range from $20,000 to $50,000
     per arch, depending on the complexity of your case, the materials
     used, and your specific anatomy. That is a wide range because every
     patient is different.

     Here is what is important to know:
     - Financing is available starting around $199-$349/month
     - Many patients qualify with a credit score of 675+
     - Some insurance plans cover a portion of the procedure
     - Implants are a long-term investment -- they can last a lifetime,
       while dentures need replacing every 5-7 years

     As Dr. Olson puts it: "I want to be transparent because I think it
     is difficult for people to find that answer."

     The best way to get a number specific to you is a free consultation
     -- no obligation, no pressure. Would you like to schedule one?

     [Yes, schedule a free consultation]
     [Take the Smile Assessment first]  (gets personalized estimate range)
     [Send me the financing guide by email]
```

**Q3: "Fear of surgery or pain":**
```
BOT: That is completely natural -- you are not the first person to feel
     this way, and you will not be the last.

     Here is what patients tell us after the procedure: the anticipation
     was worse than the reality. Most full-arch implant procedures use
     IV sedation, so you are comfortable throughout. Many patients
     describe waking up and being surprised it is already over.

     One of our patients, Scott, said his biggest worry beforehand was
     the surgery. Afterward? His first thought was what to eat. He chose
     a medium-rare New York strip. "Nicely seasoned, cooked perfectly...
     it was awesome."

     Would you like to watch a few patient stories from people who felt
     the same way you do now?

     [Yes, show me patient stories]  --> Links to 2-3 transformation videos
     [I would rather talk to someone]  --> Human handoff or consultation booking
     [Schedule a consultation]
```

**Q3: "Not sure which option is right":**
```
BOT: That makes sense -- there are a lot of options and a lot of
     confusing information online. Fixed implants, snap-in dentures,
     All-on-4, zygomatic implants... it can feel overwhelming.

     The truth is, the right option depends on your specific anatomy,
     bone density, and goals. That is why the consultation is so
     important -- it is where a doctor reviews your imaging and gives
     you a personalized recommendation.

     Our Smile Assessment is a good first step. It takes 2 minutes and
     gives you a personalized overview of your likely options and an
     estimated timeline.

     [Take the Smile Assessment]  --> Link to online assessment
     [Skip ahead and schedule a consultation]  --> Booking
     [Send me information to read first]  --> Email capture
```

**Q4: "Help me find the right doctor":**
```
BOT: I would be happy to help. What city or zip code are you in?

     [Free text input: city/zip]

     --> LOCATION MATCHING LOGIC:
     If within 60 miles of an SGA/MODIS implant practice:
       "Great news -- [Practice Name] in [City] is part of our network
        and specializes in dental implants. Dr. [Name] has [X years
        experience / X reviews / specialty credential].
        [Learn more about this practice]  [Schedule a consultation]"

     If no nearby implant practice:
       "We do not have an implant-focused practice in your immediate
        area yet, but many of our patients travel to our centers of
        excellence. Innovative Dental in Springfield, MO and the Pikos
        Institute in Tampa Bay are two of the most experienced implant
        practices in the country.

        Would you like to explore a virtual consultation? It is free
        and lets you get a preliminary assessment without traveling.
        [Book a virtual consultation]  [Find a general dentist near me instead]"
```

**Q5: "Schedule a free consultation":**
```
BOT: Wonderful. Let me get you set up.

     I just need a few details:

     - Your name: [free text]
     - Best phone number: [free text]
     - Best email: [free text]
     - Preferred practice: [dropdown or auto-filled from Q4]
     - Preferred day/time: [calendar widget]

     [Submit]

     BOT: You are all set. You will receive a confirmation at [email]
     within the next few minutes. If you have any questions before your
     appointment, reply here or call [practice phone number] directly.

     One more thing -- you might enjoy this short video of a patient
     who was in a very similar situation to yours:
     [Video: Patient Transformation Thumbnail]

     ** SYSTEM ACTION: Tag as `pt_consultation_booked`. Create CRM
     record with conversation transcript. Send confirmation email.
     Notify practice front desk via integration. Remove from
     PT-Implant Nurture if active. **
```

**Q5: "Send me information to review on my own":**
```
BOT: Of course -- no rush. I will send you some information that answers
     the most common questions.

     What is your first name and email address?

     [First name: free text]
     [Email: free text]

     BOT: Done. Check your inbox for:
     - A financing overview with monthly payment examples
     - 2-3 patient transformation stories
     - A link to our free Smile Assessment when you are ready

     You can come back to this chat anytime. We are here when you are.

     ** SYSTEM ACTION: Tag as `pt_email_nurture`. Enter PT-Implant
     Nurture email sequence (6 emails, 14 days). CRM record created. **
```

---

### 5.3 General Dentistry Patient Flow

**Goal:** Route to the nearest practice and facilitate appointment booking quickly.

```
Q1: (Already selected "General dentistry")

BOT: Let's get you scheduled. Are you looking for:

     [A regular checkup or cleaning]
     [A specific issue (filling, crown, toothache)]
     [A new patient appointment]

Q2: Location

BOT: What city or zip code are you in?

     [Free text input]

     --> LOCATION MATCH:
     "I found [1-3 practices] near you:

      1. [Practice Name] -- [Address] -- [Distance]
         [Schedule Online]  [Call: (xxx) xxx-xxxx]

      2. [Practice Name] -- [Address] -- [Distance]
         [Schedule Online]  [Call: (xxx) xxx-xxxx]

      Which one works for you?"

Q3: Insurance

BOT: Do you have dental insurance?

     [Yes]  [No]  [Not sure]

     If Yes:
     "Great. When you call or book online, have your insurance card ready
      and the team will verify your benefits before your appointment.
      Most major plans are accepted."

     If No:
     "No problem. Many of our practices offer payment plans and new
      patient specials. The front desk team can walk you through options
      when you book."

     If Not sure:
     "No worries. The practice team can help you figure that out. Just
      bring any insurance cards you have to your appointment."

Q4: Booking

BOT: Would you like to schedule now or have the office call you?

     [Book online now]  --> Practice scheduler embed/link
     [Have them call me]  --> LEAD CAPTURE (name, phone, preferred time)
     [I just needed the info, thanks]  --> Graceful close

     ** SYSTEM ACTION: If "have them call me," notify practice front desk
     via Slack or email with lead details. Tag as `pt_general_callback`.
     CRM record created. **
```

---

### 5.4 Emergency Patient Flow

**Goal:** Get them to call the nearest practice immediately. Do not delay with questions.

```
BOT: If you are experiencing a dental emergency, the fastest way to
     get help is to call your nearest SGA practice directly.

     [Find nearest practice + phone number]  --> Location lookup
     [I am not sure if this is an emergency]  --> Brief triage below

     If "not sure":

     BOT: Here are some situations that need same-day attention:
     - Severe tooth pain or swelling
     - A knocked-out or broken tooth
     - Bleeding that will not stop
     - Signs of infection (fever, swelling, pus)

     Does any of that describe your situation?

     [Yes]  --> "Please call [nearest practice phone] right now. They can
               see you today."
     [No]   --> "It sounds like this can wait for a regular appointment.
               Would you like to schedule one?"
               [Schedule an appointment]  --> General dentistry flow
```

---

### 5.5 Cosmetic Dentistry Flow

```
BOT: What are you interested in?

     [Teeth whitening]
     [Veneers or bonding]
     [Smile makeover consultation]
     [Something else]

     --> For all selections:

BOT: Cosmetic procedures are best discussed in person so the doctor
     can evaluate your specific goals. The first step is a consultation
     at your nearest practice.

     What city or zip code are you in?

     [Free text]  --> Location match + scheduling flow (same as general dentistry)
```

---

## 6. Fallback and Recovery System

### Universal Fallback Responses

Every dead end has a recovery path. The bot never leaves a user stranded.

**When the bot does not understand the input:**
```
BOT: I did not quite catch that. Could you rephrase, or pick one of
     these options?

     [Talk to a dental professional (partnership/careers)]
     [Get help as a patient]
     [Talk to a real person right now]
```

**When the user expresses frustration:**
```
Trigger: Sentiment analysis detects frustration (e.g., "this is useless,"
"I need a real person," "stop asking me questions")

BOT: I apologize -- I can tell this is not what you need right now.
     Let me connect you with a person who can help directly.

     [Connect me now]  --> Human handoff (see Section 8)
     [Send me a number to call]  --> Display relevant phone number

     ** SYSTEM ACTION: Tag as `escalation_frustration`. Log conversation
     for QA review. **
```

**When the user asks a question outside scope:**
```
BOT: That is a great question, but it is outside what I can help with
     here. Let me connect you with the right person.

     [Connect me with someone]  --> Human handoff
     [I will email instead]  --> Display relevant email address

     For clinical questions: "I am not qualified to give clinical advice,
     but one of our doctors can help. Would you like to schedule a
     consultation?"
```

**When the user goes silent mid-conversation (2+ minutes of inactivity):**
```
BOT: Still there? No worries if you got pulled away. You can pick up
     where you left off anytime.

     [Continue where I left off]
     [Start over]
     [Send me info by email instead]  --> LEAD CAPTURE
```

**After 10 minutes of inactivity:**
```
BOT: It looks like you stepped away. I will leave this conversation open
     so you can come back to it. If you would like, I can send you a
     summary and relevant links by email.

     [Send me a summary]  --> LEAD CAPTURE (if not already captured)
     [No thanks]          --> Close with "We are here whenever you need us."
```

### Specific Fallback Routing

| Input Type | Detected Intent | Route |
|-----------|----------------|-------|
| "Press" / "media" / "journalist" | Media inquiry | "For press inquiries, please email [media@sgadental.com]. Can I help with anything else?" |
| "Vendor" / "supplier" / "sales rep" | Vendor inquiry | "For vendor and supplier inquiries, please email [procurement@sgadental.com]. Thank you." |
| "Complaint" / "unhappy" / "bad experience" | Patient complaint | "I am sorry to hear that. Let me connect you with our patient experience team right away." --> Human handoff with `escalation_complaint` tag |
| "Insurance" / "bill" / "payment" | Billing question | "For billing and insurance questions, the best resource is your practice directly. [Find your practice and phone number]." |
| Profanity or abuse | Abusive input | "I understand you are frustrated. I want to help, but I am not able to continue this conversation as-is. Would you like me to connect you with a person?" If continued: "I am going to close this chat. You can reach us at [phone] or [email] when you are ready." |

---

## 7. Lead Scoring Model

### B2B Lead Scoring (Practice Owners and Specialists)

| Signal | Points | Rationale |
|--------|--------|-----------|
| Visits /partners or /partnership page + starts chat | +30 | High intent |
| Identifies as practice owner | +20 | Decision maker |
| Identifies as specialist | +20 | High-value vertical |
| 7+ operatories | +15 | Scale practice |
| 4-6 operatories | +10 | Mid-size practice |
| 1-3 operatories | +5 | Smaller practice |
| Challenge: succession planning | +15 | Near-term timeline signal |
| Challenge: admin burden or staffing | +10 | Core SGA value prop match |
| Previously explored DSO options | +10 | Educated buyer |
| Timeline: "Ready now" | +25 | Immediate pipeline |
| Timeline: "6-12 months" | +15 | Near-term pipeline |
| Timeline: "Just exploring" | +5 | Nurture track |
| Mentions a competitor by name | +10 | Active comparison shopper |
| Asks about equity or financial terms | +10 | Serious evaluation signal |
| Asks to speak to a person | +10 | High engagement |
| Downloads Partnership Guide or Specialist Guide | +15 | Content engagement |
| Completes Practice Partnership Assessment | +20 | Deep engagement |
| Returns to chat within 7 days | +10 | Sustained interest |

**Scoring Tiers:**

| Score | Classification | Action |
|-------|---------------|--------|
| 70+ | Hot Lead | Immediate Slack notification to BD team. Priority outreach within 24 hours. |
| 40-69 | Warm Lead | Enter engagement email sequence. BD team follows up within 72 hours. |
| 15-39 | Nurture Lead | Enter educational nurture. Monthly check-in content. |
| 0-14 | Cold/Early | Add to general newsletter. Re-evaluate if they return. |

### B2C Lead Scoring (Implant Patients)

| Signal | Points | Rationale |
|--------|--------|-----------|
| Visits implant page + starts chat | +20 | Intent signal |
| Situation: "wear dentures, want permanent" | +15 | High-urgency case |
| Situation: "told I need implants" | +15 | Doctor-referred |
| Concern: cost (engaged with financing info) | +10 | Active evaluation |
| Completes Smile Assessment | +20 | Deep engagement |
| Asks about specific doctor (Dr. Olson) | +10 | Provider-aware |
| Requests consultation scheduling | +25 | Conversion intent |
| Returns to chat within 14 days | +10 | Sustained interest |
| Watches patient transformation video in chat | +5 | Emotional engagement |

**Scoring Tiers:**

| Score | Classification | Action |
|-------|---------------|--------|
| 50+ | Hot Patient Lead | Fast-track: notify practice front desk for personal outreach within same business day. |
| 25-49 | Warm Patient Lead | Enter PT-Implant Nurture email sequence. |
| 0-24 | Early Interest | Offer Smile Assessment and educational content. |

---

## 8. Handoff Protocols

### When to Hand Off to a Human

| Trigger | Priority | Handoff Target |
|---------|----------|---------------|
| User explicitly asks to speak to a person | Immediate | Route based on flow: BD team (B2B), front desk (B2C), PeopleOps (staff) |
| Lead score > 70 (B2B) | Immediate | BD team via Slack notification with transcript |
| Lead score > 50 (B2C implant) | High | Practice front desk call-back within same business day |
| User expresses frustration or anger | Immediate | Appropriate team + `escalation_frustration` tag |
| User mentions competitor and asks for comparison | High | BD team (B2B) or scheduled callback |
| Complex clinical question | Medium | "Let me connect you with a doctor" --> consultation booking |
| Pricing negotiation or specific deal terms (B2B) | High | BD team directly -- bot should never discuss deal multiples or equity percentages |
| Patient complaint about care received | Immediate | Patient experience team with `escalation_complaint` tag |
| User has repeated the same question 3 times | Medium | Human handoff -- bot is failing |

### Handoff Execution Protocol

**Step 1: Prepare the user**
```
BOT: I would like to connect you with [specific person/team] who can
     help with this directly. They will have the full context of our
     conversation so you will not need to repeat anything.

     [Connect me now]  [Schedule a time instead]
```
Never say "I'll transfer you to sales." Name the person or their role and expertise.

**Step 2: Transfer context to the human**

The handoff message sent to the receiving team via Slack or CRM includes:
- Full conversation transcript
- Lead score and scoring signals
- Contact information (if captured)
- Flow path taken (which questions, which answers)
- Specific questions or concerns raised
- Recommended next step based on conversation

**Step 3: Warm introduction (when live transfer is available)**
```
[Human Rep]: Hi [Name], this is [Rep Name] from SGA. I can see you have
been asking about [specific topic]. Let me pick up right where we left off.
```

**Step 4: Fallback when no human is available**
```
BOT: Our team is currently unavailable, but I have logged your
     conversation and flagged it as a priority.

     You will hear from [specific person or team name] by [specific
     time -- e.g., "tomorrow morning by 10 AM" or "within 2 business
     hours"].

     In the meantime, here is their direct contact information:
     [Name]: [email] / [phone]

     ** SYSTEM ACTION: Create priority CRM task with deadline.
     Send confirmation email to user with expected response time. **
```

Never leave a qualified lead at a dead end. If no human is available, set a specific expectation and follow through.

### Business Hours and After-Hours Logic

| Scenario | Bot Behavior |
|----------|-------------|
| During business hours (M-F 8am-6pm ET) | Offer live transfer or same-day callback |
| After hours + B2B hot lead | "Our team is off for the evening, but I have flagged your conversation as a priority. You will hear from [Name] first thing tomorrow morning." + CRM priority task |
| After hours + B2C patient | "Our offices are closed right now, but I can get you scheduled. [Book online] or [request a callback tomorrow]" |
| Weekend + any lead | "We are closed for the weekend. I have saved your conversation and someone will reach out Monday morning. If this is urgent: [emergency number]." |

---

## 9. Rich Media Moments

Points in the conversation where images, videos, or interactive elements increase engagement and conversion.

### B2B Flow -- Rich Media Insertion Points

| Conversation Moment | Media Type | Asset |
|--------------------|-----------|-------|
| After challenge identification (admin burden, staffing) | Infographic | "What SGA Handles vs. What You Handle" -- visual split showing back-office functions SGA absorbs |
| After mentioning the Thurston track record | Stat card | "38 Years. 8 Platforms. $4B+ Returned." -- branded visual with Thurston Group logo |
| After mentioning peer collaboration | Short video (60s) | 4Front Symposium highlight reel -- dentists collaborating, not corporate training |
| After mentioning clinical development | Short video (90s) | Dr. Cavendish transformation story -- "20 years in, and it is fun again" |
| After mentioning practice preservation | Carousel / image set | Logos of practices that kept their names post-partnership: Dental Partners of SW Georgia, Periodontal Associates of Memphis, Innovative Dental |
| Before CTA (book a call) | Photo | Headshot and brief bio of the BD team member they will be speaking with |

### B2C Implant Flow -- Rich Media Insertion Points

| Conversation Moment | Media Type | Asset |
|--------------------|-----------|-------|
| After user describes denture frustration | Video (2-3 min) | Patient transformation matching their situation -- e.g., Janice (87yo, "my teeth don't fall out anymore") for older patients |
| After addressing cost concern | Infographic | "What $249/month Looks Like" -- financing breakdown with monthly payment, term, and total investment context |
| After addressing surgery fear | Video (90s) | Scott's steak moment -- vivid, sensory, specific. "First meal with new teeth" |
| After addressing "not sure which option" | Interactive graphic | Simple visual comparing dentures vs. snap-in vs. fixed implants (3-column comparison) |
| Before scheduling CTA | Before/after photo | Transformation photo of a patient similar in age/gender to the user (if demographics captured) |
| Post-booking confirmation | Video (60s) | "What to Expect at Your First Visit" -- walkthrough of the consultation process, friendly, reassuring |

### B2C General Dentistry Flow -- Rich Media

| Conversation Moment | Media Type | Asset |
|--------------------|-----------|-------|
| After location match | Photo | Exterior and interior photos of the matched practice |
| After showing practice options | Star rating + review snippet | Google rating badge + top review quote for each practice |

---

## 10. Integration Specifications

### CRM Fields to Populate

**On every lead capture (all flows):**

| CRM Field | Source | Notes |
|-----------|--------|-------|
| `first_name` | User-provided | Captured at lead capture moment |
| `last_name` | User-provided | Captured at lead capture moment |
| `email` | User-provided | Required for all lead captures |
| `phone` | User-provided | Optional for B2B exploring; required for B2B hot leads and B2C bookings |
| `lead_source` | System-detected | Values: `chatbot_web_sga`, `chatbot_web_modis`, `chatbot_messenger`, `chatbot_instagram` |
| `lead_source_detail` | System-detected | Specific page URL or ad campaign ID that triggered the conversation |
| `audience_type` | Bot-determined | Values: `practice_owner`, `specialist`, `associate`, `patient_implant`, `patient_general`, `office_manager`, `other` |
| `lead_score` | Calculated | Numeric score from Section 7 |
| `conversation_transcript` | System-generated | Full chat transcript attached as note or linked document |
| `chatbot_flow_path` | System-generated | Encoded path through decision tree (e.g., `B2B > Owner > 4-6ops > Admin > Casual > 6-12mo`) |

**B2B-specific fields:**

| CRM Field | Source | Notes |
|-----------|--------|-------|
| `practice_name` | User-provided | Captured during qualification |
| `practice_state` | User-provided or inferred | State/region of practice |
| `operatory_count` | Bot Q&A | Values: `1-3`, `4-6`, `7-10`, `10+` |
| `primary_challenge` | Bot Q&A | Values: `admin_burden`, `growth_ceiling`, `succession`, `staffing`, `clinical_development`, `other` |
| `partnership_awareness` | Bot Q&A | Values: `active`, `casual`, `new` |
| `timeline` | Bot Q&A | Values: `exploring`, `6-12_months`, `ready_now` |
| `specialty` | Bot Q&A (specialist flow) | Values: `periodontics`, `oral_surgery`, `implantology`, `other_specialty` |

**B2C-specific fields:**

| CRM Field | Source | Notes |
|-----------|--------|-------|
| `patient_situation` | Bot Q&A | Values: `dentures`, `failing_teeth`, `doctor_referred`, `researching_for_other` |
| `primary_concern` | Bot Q&A | Values: `cost`, `fear`, `unsure_options`, `finding_doctor`, `travel` |
| `preferred_practice` | Bot Q&A or location match | Practice name or ID |
| `insurance_status` | Bot Q&A | Values: `yes`, `no`, `unsure` |
| `smile_assessment_complete` | System | Boolean |
| `consultation_booked` | System | Boolean + date/time |

### Email Tags to Apply

| Tag | Applied When | Triggers |
|-----|-------------|----------|
| `pa_hot_lead` | Owner/specialist, timeline = ready now | Immediate BD notification, priority CRM task |
| `pa_assessment_lead` | Owner completes or starts assessment | PA-Indoctrination email sequence |
| `pa_content_lead` | Owner requests content only | PA educational nurture (slow drip) |
| `pa_tier_strong` | Assessment score 80-100 | Fast-track email variant, BD priority outreach |
| `pa_tier_growth` | Assessment score 50-79 | Standard engagement sequence |
| `pa_tier_foundation` | Assessment score 0-49 | Educational content track |
| `sp_hot_lead` | Specialist, actively evaluating | Immediate Marcus Morosin notification |
| `sp_guide_download` | Specialist downloads guide | SP-Indoctrination email sequence |
| `dr_active_candidate` | Associate applies or requests callback | Recruiter notification |
| `dr_passive_candidate` | Associate engaged but did not apply | DR-Passive Candidate monthly nurture |
| `dr_guide_download` | Associate downloads career guide | DR-Nurture email sequence |
| `pt_consultation_booked` | Patient books consultation | Remove from nurture, start pre-appointment reminders |
| `pt_email_nurture` | Patient requests info by email | PT-Implant Nurture sequence (6 emails, 14 days) |
| `pt_smile_assessment` | Patient starts/completes assessment | Assessment-specific follow-up |
| `pt_general_callback` | General patient requests callback | Practice front desk notification |
| `escalation_frustration` | Frustration detected, handed off | QA review queue |
| `escalation_complaint` | Patient complaint | Patient experience team queue |

### Slack Notification Channels

| Event | Channel | Message Format |
|-------|---------|---------------|
| B2B hot lead (score > 70) | `#bd-hot-leads` | ":fire: New hot lead: [Name], [Practice Name], [State]. Score: [X]. Timeline: [ready now / 6-12mo]. Challenge: [X]. [Link to CRM record]" |
| Specialist hot lead | `#modis-bd` | ":fire: New specialist lead: [Name], [Specialty], [City]. Score: [X]. [Link to CRM record]" |
| Patient consultation booked | `#patient-bookings` | "New consultation booked: [Name] at [Practice Name], [Date/Time]. Source: chatbot. [Link to CRM record]" |
| Escalation (frustration or complaint) | `#escalations` | ":warning: Chatbot escalation: [Type]. [Name if captured]. [Link to transcript]" |
| After-hours hot lead | `#bd-hot-leads` (with @mention) | Same as above + "After hours -- priority follow-up needed by [time]." |

### Scheduling Integration

| Flow | Scheduler | Configuration |
|------|----------|--------------|
| B2B Owner discovery call | Calendly | BD team round-robin calendar. 30-min slots. Auto-confirmation email with Zoom link. |
| B2B Specialist call with Marcus | Calendly | Marcus Morosin's personal calendar. 30-min slots. |
| B2C Implant consultation | Practice PMS or Calendly | Per-practice scheduling. If PMS integration is not available, use Calendly per-location. |
| B2C General appointment | Practice PMS | Direct link to online scheduling for matched practice. Fallback: phone number display. |

---

## 11. Platform-Specific Adaptations

### Web Widget (sgadental.com + practice sites)

| Feature | Implementation |
|---------|---------------|
| Button options | Full button/card UI with up to 5 options per screen |
| Rich media | Inline images, video thumbnails with click-to-play, carousels |
| Forms | Inline form fields for lead capture (no redirect) |
| Scheduling | Embedded Calendly or PMS scheduler within the widget |
| Persistent elements | "Skip to scheduling" and "Talk to a person" always visible as footer links |
| Typing indicator | Show "..." typing animation to feel conversational, not instant |
| Message pacing | 1-2 second delay between bot messages to feel natural (not wall of text) |
| Minimize/close | User can minimize widget at any time; conversation persists on return |
| Mobile | Full-screen takeover on mobile when opened; collapsible to icon |

### Facebook Messenger

| Feature | Adaptation |
|---------|-----------|
| Button options | Max 3 buttons per message (Messenger API limit). For 4+ options, use Quick Replies or split into two messages. |
| Rich media | Images and video links (not inline video). Use Messenger "generic template" for cards with image + title + buttons. |
| Forms | No inline forms. Lead capture via sequential questions (name, then email, then phone -- one per message). |
| Scheduling | Send Calendly link or practice phone number. No embed capability. |
| Persistent menu | Configure Messenger persistent menu with: "Partnership Inquiry", "Patient Care", "Careers", "Talk to a Person" |
| 24-hour rule | After 24 hours of inactivity, Messenger restricts proactive messaging. Capture email early so follow-up can continue via email if the Messenger window closes. |
| Message tags | Use `CONFIRMED_EVENT_UPDATE` tag for appointment confirmations. Use `POST_PURCHASE_UPDATE` for post-booking follow-up. All other re-engagement must happen within 24-hour window or via email. |
| Ad-to-Messenger | Click-to-Messenger ads bypass the 24-hour rule for the initial conversation. Capture email within the first exchange. |

### Instagram DM

| Feature | Adaptation |
|---------|-----------|
| Button options | Instagram DM supports Quick Reply buttons (max 13 per message, but keep to 3-4 for clarity). |
| Rich media | Single image per message. No carousels in DM. No video embed -- send YouTube links. |
| Forms | No inline forms. Sequential question capture like Messenger. |
| Scheduling | Send booking link. No embed. |
| Auto-reply triggers | Keyword-based auto-replies (configured in Section 2). Supports Story mentions and DM keywords. |
| Ice Breakers | Configure 4 "Ice Breaker" prompts that appear when a user first opens the DM thread: (1) "Explore partnership opportunities" (2) "Learn about dental implants" (3) "See career openings" (4) "Find a dentist near me" |
| 24-hour rule | Same as Messenger. Capture email early. |
| Story interaction | When users reply to an Instagram Story, the auto-reply system activates based on detected keywords in their reply. |

### WhatsApp (Future -- Planned)

| Feature | Adaptation |
|---------|-----------|
| Button options | Max 3 buttons per interactive message. Use numbered lists for 4+ options. |
| Rich media | Images, documents (PDF), and video supported. |
| Forms | No inline forms. Sequential capture. |
| Scheduling | Send booking link. |
| Template messages | Pre-approved message templates required for outbound. Build templates for: appointment confirmation, follow-up reminder, consultation reminder. |
| Business API | Requires WhatsApp Business API setup. 24-hour conversation window applies. |

---

## 12. Performance Metrics

### Primary Chatbot KPIs

| Metric | Definition | Target | Measurement |
|--------|-----------|--------|-------------|
| **Engagement Rate** | Conversations started / unique visitors (web) or unique ad impressions (social) | Web: > 5%, Social: > 3% | Weekly |
| **Routing Accuracy** | Users who reach the correct flow on the first attempt / total conversations | > 90% | Weekly (QA sample review) |
| **Completion Rate** | Conversations that reach a terminal action (booking, download, handoff, email capture) / conversations started | > 45% | Weekly |
| **Chat-to-Lead Rate** | Leads captured (email or phone) / conversations started | > 20% | Weekly |
| **Chat-to-Action Rate** | High-value actions (meeting booked, consultation scheduled, assessment started, guide downloaded) / conversations started | > 12% | Weekly |
| **Handoff Success Rate** | Handoffs where the human team responds within SLA / total handoffs | > 95% | Weekly |
| **Drop-off Points** | Screen-by-screen abandonment rates | Identify and fix any screen with > 40% drop-off | Biweekly |
| **Bot Confusion Rate** | Conversations where fallback is triggered / total conversations | < 15% | Weekly |

### Funnel-Specific Chatbot Metrics

| Funnel | Chatbot Metric | Target |
|--------|---------------|--------|
| PA (Practice Acquisition) | Chat-to-BD-call rate | > 8% of B2B owner conversations |
| PA | Assessment starts from chatbot | > 30% of "just exploring" owners |
| SP (Specialist) | Chat-to-Marcus-call rate | > 12% of specialist conversations |
| DR (Recruitment) | Chat-to-application rate | > 15% of associate conversations |
| PT (Implant) | Chat-to-consultation-booked rate | > 10% of implant conversations |
| PT (Implant) | Chat-to-Smile-Assessment rate | > 25% of implant conversations that do not book directly |
| PT (General) | Chat-to-appointment rate | > 20% of general patient conversations |

### Optimization Cadence

| Cadence | Review | Action |
|---------|--------|--------|
| Weekly | Drop-off analysis by screen | Rewrite any screen with > 40% abandonment. Test alternative phrasing. |
| Weekly | Fallback trigger log | Add new intent patterns for common unrecognized inputs. |
| Biweekly | Conversation transcript QA (sample 20 conversations) | Identify tone issues, missed routing, awkward phrasing. |
| Monthly | Lead quality review with BD team | Are chatbot-generated leads converting at expected rates? Tighten or loosen qualification if needed. |
| Monthly | A/B test results | Test greeting messages, question order, CTA phrasing. Roll winner, test new challenger. |
| Quarterly | Full flow audit | Walk through every path end-to-end. Verify integrations, update content, refresh media assets. |

### A/B Testing Priority Queue

| Test | Hypothesis | Metric |
|------|-----------|--------|
| Greeting message: question-based vs. statement-based | "Are you a dental professional or patient?" vs. "Welcome to SGA. I can help with partnership inquiries, patient care, or career opportunities." | Engagement rate |
| Q3 (challenge) format: buttons vs. free text | Free text may surface richer pain language for BD prep, but buttons have higher completion | Completion rate + lead quality |
| B2C implant flow: lead with empathy vs. lead with social proof | "You are not alone..." vs. "3,390+ patients have trusted Dr. O..." | Chat-to-consultation rate |
| CTA phrasing: "Book a call" vs. "Have a confidential conversation" | "Confidential conversation" may reduce pressure perception | Click-through rate on final CTA |
| Exit intent offer: PDF vs. assessment vs. webinar | Which exit capture converts the highest percentage of abandoning visitors | Lead capture rate on exit |
| Rich media timing: video early in flow vs. before CTA | Does video before the CTA increase conversion, or does it slow the flow and increase drop-off | Completion rate + action rate |

---

## Appendix A: Complete Decision Tree (Text Diagram)

```
ENTRY
  |
  v
ROUTING: "Dental professional or patient?"
  |
  +-- A: Dental Professional
  |     |
  |     v
  |   SUB-ROUTE: "Owner, Associate, Specialist, or Staff?"
  |     |
  |     +-- Owner --------> OWNER FLOW
  |     |                     Q2: Operatory count
  |     |                     Q3: Biggest challenge
  |     |                     Q4: Partnership awareness
  |     |                     Q5: Timeline
  |     |                       |
  |     |                       +-- Ready now ---------> BD HANDOFF (Calendly)
  |     |                       +-- 6-12mo + 4+ ops --> Partnership Guide + call option
  |     |                       +-- 6-12mo + 1-3 ops -> Assessment + nurture
  |     |                       +-- Exploring ---------> Assessment + educational content
  |     |
  |     +-- Associate ----> ASSOCIATE FLOW
  |     |                     Q2: Dentist/Hygienist/Staff?
  |     |                     Q3: What matters most?
  |     |                     Q4: Experience level
  |     |                     Q5: Geography preference
  |     |                       |
  |     |                       +-- Match found -------> Show positions + apply/talk CTA
  |     |                       +-- No match ----------> Notify me + career guide
  |     |
  |     +-- Specialist ---> SPECIALIST FLOW
  |     |                     Q2: Specialty type
  |     |                     Q3: Owner/Associate/Partner?
  |     |                     Q4: Biggest challenge
  |     |                     Q5: Familiarity with MODIS/Pikos
  |     |                     Q6: Timeline
  |     |                       |
  |     |                       +-- Active + Owner ----> Marcus Morosin HANDOFF
  |     |                       +-- Active + other ----> BD call option
  |     |                       +-- Research ----------> Specialist's Guide + podcast
  |     |                       +-- Early -------------> Guide + educational content
  |     |
  |     +-- Staff --------> STAFF FLOW
  |                           "Recently joined SGA?" or "Looking for a job?"
  |                             |
  |                             +-- Joined SGA -------> Benefits/payroll/what changes
  |                             +-- Looking for job ---> Career page link
  |
  +-- B: Patient
  |     |
  |     v
  |   SUB-ROUTE: "What brings you in?"
  |     |
  |     +-- Implants ------> IMPLANT FLOW
  |     |                     Q2: Specific situation
  |     |                     Q3: Biggest concern
  |     |                     Q4: Location/doctor preference
  |     |                     Q5: Next step preference
  |     |                       |
  |     |                       +-- Schedule ----------> Booking flow
  |     |                       +-- Assessment --------> Smile Assessment link
  |     |                       +-- Cost/financing ----> Info + consultation CTA
  |     |                       +-- Send info ---------> Email capture + nurture
  |     |
  |     +-- General -------> GENERAL FLOW
  |     |                     Q2: Type of visit
  |     |                     Q3: Location
  |     |                     Q4: Insurance
  |     |                     Q5: Book or callback
  |     |
  |     +-- Cosmetic ------> COSMETIC FLOW --> Location match --> Scheduling
  |     |
  |     +-- Emergency -----> EMERGENCY FLOW --> Nearest practice phone number
  |     |
  |     +-- Find Location -> LOCATION FINDER --> Zip/city --> Practice match
  |
  +-- C: Something Else
        |
        v
      CLARIFICATION: Partnership / Careers / Patient / Media / Vendor
        --> Routes to appropriate flow above
        --> Media/Vendor: email address display
```

---

## Appendix B: Conversation Tone Reference Card

Quick reference for anyone writing or reviewing bot scripts.

### Voice Constants (All Flows)

- Evidence over claims: "Our partners can tell you" not "We are the best"
- Specific over generic: "85% staff retention" not "great culture"
- Direct over hedged: "Here is how it works" not "We believe we offer"
- Peer-level: "You have been doing this long enough to know" not "Let us show you a better way"

### B2B Tone

- Respectful of their expertise and their fears
- Acknowledge DSO skepticism before countering it
- Mirror their possessive language: "your practice," "your team," "your patients"
- Never say "your asset," "the business," or "the opportunity"
- Back every claim with a name and a number

### B2C Patient Tone

- Warm, human, un-clinical
- Lead with the before-state they are living in right now
- Use sensory language: eating, smiling, laughing, taking photos
- Never minimize fear or financial anxiety
- Show transformation through someone who looks like them

### What We Never Do

- Pretend to be human (bot identity is always clear)
- Use discount or urgency language ("limited time," "act now," "don't miss out")
- Ask questions we do not use to determine the next action
- Leave a user at a dead end with no recovery option
- Discuss specific deal multiples, equity percentages, or contract terms in the bot
- Provide clinical diagnosis or treatment recommendations
- Push when someone says they are "just exploring" -- respect their timeline

---

## Appendix C: Implementation Checklist

### Phase 1: Foundation (Weeks 1-2)

- [ ] Select chatbot platform (Drift, Intercom, Tidio, or custom)
- [ ] Build initial routing flow (Section 3)
- [ ] Build B2B Owner flow (Section 4.2) -- all 5 questions + response scripts
- [ ] Build B2C Implant flow (Section 5.2) -- all 5 questions + response scripts
- [ ] Build B2C General flow (Section 5.3) -- location matching + scheduling
- [ ] Build Emergency flow (Section 5.4)
- [ ] Configure fallback responses (Section 6)
- [ ] Connect CRM integration (field mapping per Section 10)
- [ ] Connect Calendly for BD team scheduling
- [ ] Set up Slack notification channels (Section 10)
- [ ] Deploy web widget on sgadental.com (homepage + /partners + /careers)
- [ ] Deploy patient widget on modisdental.com implant pages

### Phase 2: Expansion (Weeks 3-4)

- [ ] Build Specialist flow (Section 4.3)
- [ ] Build Associate flow (Section 4.4)
- [ ] Build Staff/Office Manager flow (Section 4.5)
- [ ] Build Cosmetic flow (Section 5.5)
- [ ] Configure Facebook Messenger entry point with persistent menu
- [ ] Configure Instagram DM auto-replies and Ice Breakers
- [ ] Integrate rich media assets (videos, infographics, photos per Section 9)
- [ ] Implement lead scoring model (Section 7)
- [ ] Set up email tag automation (Section 10)
- [ ] Deploy widget on individual practice sites (top 10 by traffic)

### Phase 3: Optimization (Weeks 5-8)

- [ ] Run first A/B test on greeting message
- [ ] Review first 100 conversation transcripts for QA
- [ ] Fix top 3 drop-off points identified in analytics
- [ ] Add new intent patterns to fallback system based on unrecognized inputs
- [ ] Conduct lead quality review with BD team
- [ ] Launch after-hours and weekend logic
- [ ] Set up abandoned chat recovery (email follow-up for conversations that captured email but did not convert)
- [ ] Begin monthly optimization cadence (Section 12)

### Phase 4: Scale (Weeks 9-12)

- [ ] Deploy widget to all SGA practice sites
- [ ] Launch WhatsApp Business entry point (if approved)
- [ ] Implement cross-funnel routing automation (chatbot leads feeding email sequences per funnel architecture)
- [ ] Build chatbot-specific dashboard in analytics platform
- [ ] Train BD team on chatbot-generated lead handling protocol
- [ ] Train practice front desk teams on chatbot patient lead callbacks
- [ ] Document learnings and update this flow document

---

*Document total: ~6,800 words*
*Sources: Product brief, buyer personas, voice of customer research, brand positioning, funnel architecture, chatbot-lead-gen skill framework*
*Aligns to: PA, DR, PT, SP funnel prefixes from funnel_architecture.md*
