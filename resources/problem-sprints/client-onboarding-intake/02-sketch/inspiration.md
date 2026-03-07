# Research Inspiration -- Client Onboarding Intake

## Sprint Target Reminder

**Target A: Guided Intake and Collection (Steps 2-6)**
Design the system that replaces scattered multi-channel collection with a guided, flexible intake experience.

---

## 1. Content Snare -- Structured Document Collection Portal

- **Category:** Direct Competitor
- **Source:** contentsnare.com
- **Description:** Content Snare replaces the email-and-text chase with a structured portal where clients see exactly what is needed, upload files into labeled slots, and answer specific questions. Automated reminders follow up on missing items without the service provider lifting a finger. Claims to reduce document collection time by 71%.
- **Big Idea:** The "labeled slot" model -- showing clients a visual checklist of what is needed and letting them fill in the blanks at their own pace -- transforms a vague request ("send me your materials") into a concrete, completable task. Clients procrastinate on ambiguity; they act on clarity.
- **Relevance to Sprint:** Directly addresses Q1 (will clients submit through a system?) by reframing the ask from "send us stuff" to "fill in these specific boxes." Also addresses Q3 (sophistication spectrum) because clients with few materials simply have fewer boxes to fill, rather than feeling inadequate.

---

## 2. TurboTax Interview Flow -- Guided Question-Based Intake

- **Category:** Analogous Solution (different industry, same pattern)
- **Source:** Intuit TurboTax (appcues.com analysis)
- **Description:** TurboTax turned the daunting task of filing taxes into a conversational interview. One question at a time, progressively disclosed based on prior answers. The first question is emotional: "How are you feeling about doing your taxes?" Document upload is embedded mid-flow, with OCR automatically extracting data from uploaded W-2s and receipts. Users self-segment early (simple vs. complex return), and the system adapts the depth of the interview accordingly.
- **Big Idea:** Start with emotion, not logistics. Let the client self-identify their complexity level, then adapt the depth of what you ask. Embed document capture inside the conversation rather than making it a separate step. The interview IS the intake.
- **Relevance to Sprint:** Powerful model for Q3 (handling the sophistication spectrum). A client with a brand deck uploads it mid-conversation; a client with nothing answers guided questions that extract the same information narratively. Same flow, different paths.

---

## 3. Typeform -- Conversational Progressive Forms

- **Category:** Adjacent Product (form tool, not intake-specific)
- **Source:** typeform.com
- **Description:** Typeform pioneered "one question at a time" form design. Progressive disclosure means the respondent sees only the current question, reducing overwhelm. Conditional logic branches the path based on answers. File upload is native. In 2025, Typeform added AI features: describe a form in plain English and AI generates it; upload a PDF and AI converts it to a structured form.
- **Big Idea:** Progressive disclosure is the antidote to the "wall of fields" problem. When you show someone 30 fields at once, they close the tab. When you show them one question at a time, they complete it. The conversational frame makes data collection feel like dialogue, not paperwork.
- **Relevance to Sprint:** Addresses Q4 (keeping the experience personal). A Typeform-style flow can feel like a conversation with the facilitator rather than a bureaucratic form. Also relevant for Q1 -- higher completion rates come from reducing perceived effort.

---

## 4. SmartVault SmartRequest AI -- Prior-Context-Driven Intake

- **Category:** Analogous Solution (accounting/tax industry)
- **Source:** smartvault.com
- **Description:** SmartRequest AI analyzes a client's prior-year tax return to auto-generate a custom questionnaire and document request list for the current year. Instead of sending every client the same generic checklist, each client gets a tailored list based on what the system already knows about them. Clients upload documents through a clean portal, and SmartVault organizes everything into the right folder structure automatically.
- **Big Idea:** Use what you already know about the client to reduce what you ask. If the system can scrape their website, pull their LinkedIn, or reference a prior engagement, the intake should start from a position of partial knowledge -- not from zero. Clients are delighted when a system says "we already found this about you -- is it still accurate?" instead of "tell us everything from scratch."
- **Relevance to Sprint:** Directly addresses the "head start" concept in the sprint target description. Auto-scraping the client's website before intake begins could pre-populate 30-40% of the framework, making the client's job dramatically smaller.

---

## 5. Gallabox / WhatsApp Business Flows -- Meet-Them-Where-They-Are Intake

- **Category:** Adjacent Product (conversational commerce)
- **Source:** gallabox.com, WhatsApp Business API
- **Description:** WhatsApp Business Flows allow businesses to create interactive forms, file uploads, and structured data collection inside WhatsApp itself. No new app to download, no portal to log into. Gallabox layers AI chatbot logic on top -- asking questions, routing answers, processing uploaded images and PDFs, and syncing to CRMs. Clients respond in the same app they use to text their family.
- **Big Idea:** Instead of asking clients to go to your system, bring the system to the channel they already live in. If clients currently text materials, let them keep texting -- but to a smart endpoint that structures and organizes what they send. The behavioral change is zero; the backend change is everything.
- **Relevance to Sprint:** A radical answer to Q1. If clients won't go to a portal, don't build a portal -- build an intelligent receiver that accepts inputs from SMS, email, and messaging apps and organizes them behind the scenes. Also strong on Q3: the least digitally sophisticated clients can literally text a photo and have it processed.

---

## 6. Loom Async Video -- Narrative Capture Through Storytelling

- **Category:** Analogous Solution (async communication)
- **Source:** loom.com (now Atlassian)
- **Description:** Loom lets people record short video messages with screen sharing. The key insight is that some information is easier to tell than to type. Loom AI automatically transcribes, summarizes, and sections the video. In client-facing contexts, teams use Loom to collect context before meetings -- "record a 3-minute video walking me through your current situation."
- **Big Idea:** Not all client knowledge lives in documents. Some of it lives in the founder's head as stories, instincts, and anecdotes. A "tell me your story" prompt with video/voice capture may extract richer market context than any form could. The transcription becomes the structured input.
- **Relevance to Sprint:** Addresses Q3 for the "we don't have documents" client. Instead of asking them to produce artifacts they don't have, ask them to talk. A 5-minute video of a founder describing their market, competitors, and ideal customer may contain more signal than a polished brand deck. Also speaks to Q4 (personal touch) -- video feels warm, not transactional.

---

## 7. Collect (usecollect.com) -- AI-Powered Document Sorting and Gap Detection

- **Category:** Direct Competitor
- **Source:** usecollect.com
- **Description:** Collect automates client onboarding with AI-powered workflows. When documents arrive, AI sorts them into categories, detects what type of document each is, and flags what is still missing. Real-time CRM sync keeps teams updated. The system handles the cognitive load of "what have we received, what do we still need?"
- **Big Idea:** Automated gap detection flips the follow-up problem. Instead of the facilitator manually reviewing a folder and figuring out what is missing, the system maintains a living checklist and proactively identifies gaps. Follow-up becomes automatic and specific: "We still need your competitive landscape -- can you share who your top 3 competitors are?"
- **Relevance to Sprint:** Directly addresses Steps 5-6 (materials arriving, facilitator receiving). The gap detection concept is essential for reducing the follow-up cycle that currently adds days or weeks to the timeline. Also relevant for Q5 (facilitator trust) -- the facilitator can see at a glance what has been collected and what is missing.

---

## 8. Notion + Calendly Intake Pattern -- The "Living Document" Approach

- **Category:** Contrarian Approach
- **Source:** Observed pattern in boutique consulting firms
- **Description:** Some boutique consultants skip forms entirely. They create a shared Notion page for each client, seed it with a skeleton structure (sections like "Your Market," "Your Competitors," "Your Customer"), and invite the client to fill it in over time. A Calendly-booked 20-minute kickoff call walks the client through the structure. The client adds content -- text, links, uploaded files -- at their own pace. The consultant can see progress in real time and drop comments where more depth is needed.
- **Big Idea:** Make intake collaborative rather than extractive. Instead of "submit your materials to us," the frame becomes "let's build this together." The shared document is visible to both parties, creating accountability and transparency. The client can see their inputs alongside the framework, which teaches them what is needed and why.
- **Relevance to Sprint:** A contrarian answer to Q4 (impersonal experience). This approach is the most relational -- it preserves the feeling of working together while still structuring the collection. Risk: it requires client initiative and comfort with Notion-style tools, which may not work for all sophistication levels (Q3 tension).

---

## 9. FileInvite -- Deadline-Driven Document Collection with Status Tracking

- **Category:** Direct Competitor (lending/financial services focus)
- **Source:** fileinvite.com
- **Description:** FileInvite was built for complex lending workflows where dozens of documents must be collected from applicants who are overwhelmed and disorganized. The system sends a branded request with a checklist of specific documents needed, a deadline, and a simple upload interface. Automated reminders escalate in urgency as the deadline approaches. Both the requester and the client see a real-time status dashboard showing what has been received, what is pending, and what is overdue.
- **Big Idea:** Deadlines and status visibility drive completion. When clients can see "4 of 7 items submitted" with a countdown, the task feels finite and achievable. The escalating reminder pattern (gentle at first, urgent near deadline) mimics human follow-up behavior without requiring the human to do it.
- **Relevance to Sprint:** Addresses Q1 by using behavioral psychology (progress bars, deadlines, escalating nudges) to drive completion. The lending analogy is strong -- mortgage applicants are similarly disorganized and similarly motivated by a deadline. The workshop date IS the natural deadline.

---

## Summary: Transferable Insights

| # | Source | Big Idea |
|---|--------|----------|
| 1 | Content Snare | Turn a vague ask into labeled slots -- clarity drives completion |
| 2 | TurboTax | Start with emotion, self-segment by complexity, embed uploads in conversation |
| 3 | Typeform | One question at a time -- progressive disclosure reduces overwhelm |
| 4 | SmartRequest AI | Use existing knowledge to pre-populate -- never start from zero |
| 5 | WhatsApp Flows | Bring intake to the client's channel, not the other way around |
| 6 | Loom | Let clients tell their story on video -- not all knowledge is a document |
| 7 | Collect | AI-powered gap detection makes follow-up automatic and specific |
| 8 | Notion shared doc | Make intake collaborative, not extractive -- build it together |
| 9 | FileInvite | Deadlines + progress visibility + escalating reminders drive action |
