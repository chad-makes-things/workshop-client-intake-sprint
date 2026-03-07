# Solution Sketches -- Client Onboarding Intake

## Sprint Target

**Target A: Guided Intake and Collection (Steps 2-6)**
Design the system that replaces scattered multi-channel collection with a guided, flexible intake experience.

---

## Solution 1: "The Interview" -- Conversational Guided Intake Flow

### Premise

Replace the "send us your stuff" request with a TurboTax-style guided interview that walks each client through a branching conversation, extracting everything the framework needs through questions, uploads, and links -- one step at a time.

### Key Insight

Clients do not know what you need or why you need it. When you say "send your business materials," they freeze, procrastinate, or send the wrong things. But when you ask "who is your ideal customer and why do they pick you over alternatives?" -- they can answer that. The interview extracts the same information the documents would have contained, through questions the client can actually engage with.

### How It Works

**Step 1: The Kickoff Link.** After a client signs up for the workshop, the facilitator sends a single link with a short personalized video message: "Hey [Name], excited to work together. Before our workshop, I need to learn about your business. This will take about 20 minutes -- you can pause and come back anytime. Let's get started." The video preserves the personal touch. The link opens the interview.

**Step 2: Self-Segmentation.** The first screen asks the client to describe their situation: "Which sounds most like you?" Options range from "We have a brand deck, market research, and positioning documents ready to share" to "We're early stage -- most of what we know is in our heads." This answer determines the path depth and question style for the entire interview.

**Step 3: The Guided Conversation.** The interview presents one question or prompt at a time, Typeform-style. For the "documents-ready" client, the flow focuses on uploads with brief context questions: "Upload your brand deck" followed by "What part of this deck best represents how you see your market?" For the "it's-in-our-heads" client, the flow asks narrative questions: "Describe your customer in a few sentences -- who are they, what are they struggling with?" Both paths map to the same framework slots, but through different mechanisms.

**Step 4: Embedded Capture.** Throughout the interview, the client can upload files, paste links, type answers, or record a short voice/video clip. Each input is tagged to the framework category it serves. The client never sees the framework -- they just answer questions. The system maps answers to slots behind the scenes.

**Step 5: Progress and Pause.** A progress bar shows completion. The client can close the browser and come back -- their progress is saved. Automated reminders nudge them if they stop partway: "You're 60% done -- about 8 minutes left. Your workshop is in 12 days."

**Step 6: Review and Submit.** At the end, the client sees a summary: "Here's what we captured." They can edit, add, or confirm. On submission, the facilitator receives a notification with a structured view of everything collected, organized by framework category.

### Why It Might Work

- Eliminates the ambiguity that causes clients to procrastinate or send wrong materials.
- Progressive disclosure prevents overwhelm -- one question is always answerable.
- Branching paths handle the sophistication spectrum without making anyone feel inadequate.
- The facilitator's video kickoff preserves warmth and personal connection.
- Framework mapping happens silently -- the client never needs to understand the structure.
- Pause-and-resume with reminders accommodates busy clients without manual follow-up.

### Why It Might Fail

- Some clients may resist a 20-minute guided flow, especially senior executives who prefer to delegate.
- The branching logic must be carefully designed -- wrong branches could miss critical information.
- Clients who strongly prefer texting or emailing may resent being directed to a web flow.
- If the questions feel generic or templated, the personal brand of the workshop suffers.
- Requires significant upfront design work to map the facilitator's tacit framework knowledge into question sequences.

### Feasibility

**Medium-High.** Tools like Typeform, Tally, or a custom-built flow can handle the conversational UI, branching logic, file uploads, and progress tracking. The harder work is designing the question tree and mapping it to the prompt framework -- this requires close collaboration with the facilitator. No novel technology, but meaningful design effort.

### Sprint Questions Addressed

- **Q1 (Will clients submit through a system?):** YES -- directly tested. The interview IS the submission mechanism.
- **Q3 (Sophistication spectrum?):** YES -- branching paths adapt to client readiness.
- **Q4 (Personal touch?):** PARTIALLY -- facilitator video helps, but it is still a web form at its core.

---

## Solution 2: "The Inbox" -- Multi-Channel Smart Receiver

### Premise

Do not ask clients to change their behavior at all. Instead, build an intelligent backend that accepts materials from any channel the client already uses -- text, email, Dropbox, file share, voice memo -- and automatically routes, categorizes, and organizes everything into the framework structure. The client's experience does not change. The facilitator's experience transforms completely.

### Key Insight

The hardest behavioral change is one you do not have to make. Clients already send materials. They send them via text because that is easy. They email PDFs because that is familiar. They share Dropbox links because that is what they use at work. The problem is not the sending -- it is the receiving. The fix is not a new front door for the client; it is a smart mailroom on the facilitator's side.

### How It Works

**Step 1: The Unified Endpoint.** When a client signs up, the system creates a unique set of intake channels: a dedicated email address (acme-workshop@intake.example.com), an SMS number, and an upload link. The facilitator sends the client a message: "Send me your materials however is easiest -- here's your email and text number. Drop files, links, photos, voice memos, whatever you've got. I'll organize everything on my end."

**Step 2: Materials Arrive Wherever.** The client texts a photo of their product. Emails a PDF pitch deck. Shares a Dropbox link to a brand folder. Sends a voice memo describing their competition. Forwards an old email thread with customer feedback. Nothing changes for them -- they do what they already do.

**Step 3: The Smart Mailroom.** Behind the scenes, every inbound item flows to a single processing pipeline. The system: downloads attachments, scrapes linked websites, transcribes voice memos, extracts text from PDFs, and parses email content. Each piece of content is timestamped, source-tagged, and stored.

**Step 4: AI Categorization.** An AI layer reads each processed item and maps it to the framework categories: market context, competitive landscape, customer profile, brand positioning, etc. Items that do not fit cleanly are flagged for facilitator review. The system builds a living "framework fill" dashboard showing what has been received and what gaps remain.

**Step 5: Automated Gap Nudges.** When the system detects a gap (e.g., no competitive information received), it generates a specific follow-up message: "Thanks for everything so far! One thing I haven't seen yet is anything about your competitors. Could you send me the names of 2-3 companies your customers might also consider?" This nudge goes out via the same channel the client has been using.

**Step 6: Facilitator Dashboard.** The facilitator sees a real-time dashboard per client: what has been received, how it maps to the framework, what gaps remain, and a confidence score for each framework category. When coverage reaches a threshold, the system signals "ready for narrative generation."

### Why It Might Work

- Zero behavioral change for the client. They send materials the way they already want to.
- Solves the facilitator's aggregation burden completely -- no more downloading from five channels.
- Automated categorization and gap detection replace the facilitator's manual review.
- The follow-up nudges are specific and helpful, not generic nagging.
- Scales effortlessly to 10+ concurrent clients because the system, not the facilitator, is the aggregation point.
- Preserves the personal, informal feel of the client relationship.

### Why It Might Fail

- Without guided structure, clients may send too little, too late. The "send whatever" approach relies on client initiative, which is the very thing that currently fails.
- AI categorization of unstructured, multi-format inputs is technically complex. A blurry photo of a whiteboard or a rambling voice memo is hard to process reliably.
- The facilitator loses visibility into the intake process -- they cannot see the client's experience, only the result. If something goes wrong, it is hard to diagnose.
- "Send whatever you want" may produce a flood of irrelevant materials alongside useful ones. Signal-to-noise ratio could be low.
- Building and maintaining a multi-channel ingestion pipeline (SMS, email, Dropbox, file share) is an engineering challenge.

### Feasibility

**Medium.** The multi-channel ingestion pipeline is the hardest part. Email forwarding and file upload are straightforward. SMS processing and Dropbox integration add complexity. AI categorization of heterogeneous content is feasible but requires tuning and error handling. This solution has lower design effort but higher engineering effort than Solution 1.

### Sprint Questions Addressed

- **Q1 (Will clients submit through a system?):** REFRAMED -- the system meets clients where they are, so the answer is "they already do."
- **Q3 (Sophistication spectrum?):** YES -- every channel is accepted, from text messages to cloud file shares.
- **Q4 (Personal touch?):** YES -- the client's experience is unchanged; they still feel like they are working directly with the facilitator.
- **Q5 (Facilitator adoption?):** PARTIALLY -- depends on whether the facilitator trusts AI categorization.

---

## Solution 3: "The Head Start" -- Website-First Intake with AI Pre-Population

### Premise

Before asking the client for anything, the system scrapes their website, social profiles, and any publicly available information to pre-build 40-60% of the framework. The client then reviews, corrects, and supplements what the system already found -- turning intake from "build from scratch" into "review and refine."

### Key Insight

Most of what the facilitator needs to know about a client's business already exists in public. Their website describes their market, their about page reveals their positioning, their blog shows their thought leadership, their LinkedIn profiles reveal the team, and their social media reveals their voice. Asking a client to re-type all of this is disrespectful of their time. A system that arrives having already done its homework earns trust and reduces the client's burden to only the things that are genuinely private or undocumented.

### How It Works

**Step 1: The Trigger.** When a client signs up for the workshop, the facilitator enters their company name and website URL. That is the only input needed to start.

**Step 2: The Automated Harvest.** The system crawls the client's website -- homepage, about page, product/service pages, blog, team page. It pulls their LinkedIn company page, any Glassdoor or G2 reviews, relevant press mentions, and social media bios. All content is extracted, cleaned, and stored.

**Step 3: The AI Draft.** An AI model reads all harvested content and generates a first-pass framework fill: a draft of the client's market context, competitive positioning (based on similar companies found during the crawl), customer profile (inferred from website copy), and brand voice (inferred from content tone). This draft is 40-60% complete -- good enough to show, not good enough to use.

**Step 4: The "Is This Right?" Review.** The client receives a link to review the AI's draft: "We did some homework on [Company Name]. Here's what we found. Please review each section -- correct anything that's wrong, add what's missing, and flag anything that surprised you." Each section has inline editing, a "this is wrong" button, and a "add more" prompt. The client's job is editorial, not authorial.

**Step 5: Supplemental Upload.** After reviewing the draft, the client is prompted to upload any internal materials that the system could not find publicly: internal strategy docs, customer research, pitch decks, competitive analysis. Because the client has just seen the framework and understands what each section needs, their uploads are targeted and relevant.

**Step 6: Gap Closure.** Any sections the AI could not populate and the client did not supplement are flagged. The system generates specific questions for the missing areas. The facilitator can send these as a short follow-up conversation, not a long intake form.

### Why It Might Work

- Dramatically reduces the client's burden. Reviewing is easier than creating.
- The "we already did our homework" moment builds trust and demonstrates the workshop's value before it even begins.
- Clients see the framework in the context of their own business, which helps them understand what is needed.
- The AI draft gives the client something to react to, which is psychologically easier than filling a blank page.
- Public information is often accurate for the "what" of a business; the client only needs to add the "why" and the "where we're going."
- Works exceptionally well for digitally mature clients with rich web presence.

### Why It Might Fail

- Clients with minimal web presence get a thin, possibly embarrassing draft. "We found almost nothing about your company" is not a great first impression.
- The AI draft may be wrong in ways that are subtly damaging -- an incorrect competitor, a misread market position -- and if the client does not catch it, the error propagates into the narrative.
- Some clients may be uncomfortable with the level of automated research. "How do you know all this about us?" can feel invasive rather than impressive.
- The review-and-edit UX must be exceptionally well designed. If it feels like correcting a bad report, it creates frustration instead of delight.
- Companies with very private or pre-launch businesses have almost no public information. This approach has a floor problem.

### Feasibility

**Medium.** Web scraping and content extraction are well-established. AI summarization of website content is reliable with current models. The editorial review interface requires custom UI work but nothing exotic. The main risk is handling edge cases: sites behind login walls, thin web presences, incorrect AI inferences. Overall a proven technical pattern with meaningful design nuance.

### Sprint Questions Addressed

- **Q1 (Will clients submit through a system?):** REFRAMED -- the client's job is reduced from "submit everything" to "review and correct." Much lower barrier.
- **Q3 (Sophistication spectrum?):** PARTIALLY -- works brilliantly for clients with rich web presence; weaker for minimal-presence clients.
- **Q4 (Personal touch?):** YES -- "we did our homework" feels attentive and premium, not robotic.
- **Q5 (Facilitator adoption?):** PARTIALLY -- depends on AI draft quality. If the facilitator has to redo the AI's work, trust erodes.

---

## Solution 4: "The Kickoff Call" -- Structured Live Interview with AI Capture

### Premise

Do not collect materials asynchronously at all. Instead, schedule a focused 45-minute video call where the facilitator (or an AI-assisted interviewer) walks the client through the framework live, capturing everything in real time through conversation, screen sharing, and live document review. The system records, transcribes, and structures the call into framework-ready output.

### Key Insight

The current process fails because it asks clients to do the facilitator's job -- gather, organize, and deliver materials against a framework the client does not understand. But the facilitator already knows how to extract this information through conversation. The highest-quality intake the facilitator has ever done probably happened over a phone call, not through a file upload. Lean into the strength: the human interview. Then use AI to do the structuring that used to happen after.

### How It Works

**Step 1: The Booking.** When a client signs up, they receive a Calendly link for a "Workshop Prep Call" -- a 45-minute video meeting scheduled 2-3 weeks before the workshop. The booking page sets expectations: "We'll walk through your business together. No prep needed on your end -- just bring yourself and your screen."

**Step 2: The Pre-Call Scrape.** Before the call, the system automatically scrapes the client's website and generates a lightweight brief for the interviewer: key facts, potential gaps, suggested deep-dive areas. The interviewer walks in informed, not cold.

**Step 3: The Live Interview.** The call follows a structured-but-flexible interview guide mapped to the framework. The interviewer asks about market, customers, competitors, positioning, and aspirations. When the client says "we have a deck that shows this," the interviewer says "great, can you screen-share it?" or "drop it in the chat." Documents are captured in-session. Stories and anecdotes are captured as spoken narrative.

**Step 4: AI Real-Time Processing.** The entire call is recorded and transcribed in real time. An AI system running alongside the call maps each segment of conversation to framework categories. By the time the call ends, a first-pass framework fill exists -- not from documents, but from conversation.

**Step 5: The Follow-Up Package.** Within 24 hours, the client receives a summary: "Here's what we captured from our conversation." Any gaps are identified: "We didn't get to discuss your competitive landscape in depth -- could you send over any materials you have, or answer these two questions?" The follow-up is targeted and minimal.

**Step 6: Optional Document Supplement.** If the client has documents they mentioned during the call but did not share in-session, a simple upload link lets them drop files. Because the conversation already established context, these uploads are supplementary, not foundational.

### Why It Might Work

- Highest-fidelity information capture. Conversation extracts nuance, stories, and strategic thinking that no form or file upload ever captures.
- Zero prep burden on the client. "Just show up" is the lowest possible ask.
- The facilitator (or trained interviewer) can probe, follow up, and adapt in real time -- no form can do this.
- Preserves and even enhances the personal, relational feel of the workshop brand.
- Works equally well for all sophistication levels. A founder with nothing written and a CMO with a full brand deck are both served by conversation.
- The AI structuring happens on the backend -- the client never interacts with technology.

### Why It Might Fail

- Does not scale. A 45-minute call per client means the facilitator spends 7+ hours just on intake for 10 clients. Unless an interviewer is hired or AI conducts the call, this adds labor rather than removing it.
- Scheduling is friction. Getting a 45-minute block on a busy executive's calendar 2-3 weeks before a workshop can be difficult.
- If the facilitator is the interviewer, this reintroduces the bottleneck. If someone else interviews, they may miss the nuance the facilitator would have caught.
- The AI's ability to accurately categorize free-flowing conversation into framework slots is unproven and may require significant post-processing.
- Clients who are used to "just sending files" may find a mandatory call annoying.

### Feasibility

**High for V1, Low for Scale.** A facilitator-led call with Zoom recording and AI transcription (Otter.ai, Fireflies.ai) is immediately doable with no custom technology. The hard parts are scaling beyond the facilitator and ensuring AI categorization of unstructured conversation is accurate. This solution has the lowest technology barrier but the highest human labor cost.

### Sprint Questions Addressed

- **Q1 (Will clients submit through a system?):** REFRAMED -- the client does not submit anything. The facilitator extracts it live.
- **Q3 (Sophistication spectrum?):** YES -- conversation adapts to any client. The interviewer calibrates in real time.
- **Q4 (Personal touch?):** YES -- this IS personal. It is the most human solution in the set.
- **Q5 (Facilitator adoption?):** YES -- the facilitator already does this informally. This formalizes and captures what they naturally do.

---

## Solution 5: "The Shared Canvas" -- Collaborative Workspace with Async Co-Creation

### Premise

Create a shared digital workspace for each client -- a living document that both the facilitator and client can see and edit. The facilitator seeds it with the framework structure and any auto-scraped content. The client fills it in over 1-2 weeks at their own pace. Both parties can comment, ask questions, and refine in real time. Intake becomes a collaborative act, not a one-way submission.

### Key Insight

The current process treats the client as a supplier and the facilitator as a receiver. This creates a transactional dynamic: "give me your stuff." But the client is actually a collaborator -- they are the expert on their own business, and the workshop narrative is built from their knowledge. The intake should feel like the first act of collaboration, not a prerequisite to it. When both parties can see the same canvas, the quality of input rises because the client understands what they are contributing to.

### How It Works

**Step 1: The Canvas Creation.** When a client signs up, the system auto-creates a branded shared workspace (think Notion page, Google Doc, or custom web app). The workspace has the framework sections as headers -- Market Context, Customer Profile, Competitive Landscape, Brand Positioning, Aspirations -- with brief explanations of what each section needs and why.

**Step 2: The Auto-Seed.** Before the client ever sees the canvas, the system scrapes their website and populates each section with whatever it can find. Under "Market Context," it drafts a paragraph from their website copy. Under "Competitive Landscape," it lists companies found in similar spaces. Each auto-generated section is clearly marked: "We found this -- edit, add, or delete as needed."

**Step 3: The Invitation.** The facilitator sends the client a personal message with the canvas link: "I've set up our shared workspace. I've done some initial research on [Company] -- take a look and start adding your perspective. No rush, no rules. Drop in text, upload files, paste links, add images. Think of it as our shared notebook for the workshop."

**Step 4: The Async Dialogue.** Over the next 1-2 weeks, the client fills in sections at their own pace. They might type a paragraph about their customers on Monday, upload a competitive analysis PDF on Wednesday, and paste a link to a customer testimonial video on Friday. The facilitator can check in, leave comments ("This is great -- can you say more about why this customer segment is growing?"), and nudge specific sections that are empty.

**Step 5: The Completion Signal.** The workspace tracks completeness per section. When all sections reach a minimum threshold, the system notifies the facilitator: "Client workspace is ready for narrative generation." If sections remain empty as the workshop approaches, the system sends a focused nudge: "Two sections still need your input -- Competitive Landscape and Customer Profile. Could you spend 10 minutes on these?"

**Step 6: The Handoff.** The completed canvas -- with all its text, uploads, links, and comments -- is the input for narrative generation. Nothing needs to be re-organized or re-formatted because the framework structure was the workspace structure all along.

### Why It Might Work

- The shared visibility creates accountability and transparency. The client can see what is done and what is not. The facilitator can see progress without asking.
- The collaborative frame feels premium and relational, not transactional. "Our shared workspace" signals partnership.
- The auto-seeded content reduces the blank-page problem and shows the client what is expected.
- Flexible input types (text, files, links, images) accommodate all sophistication levels and all material types.
- The facilitator's comments and nudges maintain the human touch within a structured system.
- The framework IS the workspace structure, so no post-collection reorganization is needed.

### Why It Might Fail

- Requires client initiative over 1-2 weeks. Without the forcing function of a call or a step-by-step flow, clients may procrastinate indefinitely.
- A shared canvas can feel overwhelming if there are too many sections or too much auto-generated content to review.
- Clients unfamiliar with collaborative document tools (Notion, Google Docs) may find the interface intimidating.
- The facilitator must actively monitor and engage with the canvas to maintain the collaborative feel -- passive monitoring defeats the purpose. This adds facilitator labor.
- File uploads into a rich document can get messy. A canvas full of PDFs, pasted text, and half-finished thoughts is harder to process than structured form responses.
- Without enforced structure, different clients will produce wildly different canvas formats, making downstream processing inconsistent.

### Feasibility

**Medium-High.** Notion, Coda, or a custom web app can support this pattern today. Auto-seeding from web scraping is feasible. The comment/nudge workflow is native to these tools. The main risk is building a clean enough interface that clients of all sophistication levels can use it without training. A custom-built version with a simpler UI than Notion would be ideal but requires development investment.

### Sprint Questions Addressed

- **Q1 (Will clients submit through a system?):** YES -- but depends on client self-motivation. The system provides structure; the client must provide initiative.
- **Q3 (Sophistication spectrum?):** PARTIALLY -- works well for clients comfortable with digital tools; may struggle with less tech-savvy clients.
- **Q4 (Personal touch?):** YES -- the collaborative, comment-based interaction preserves warmth and makes intake feel like the start of the workshop.
- **Q5 (Facilitator adoption?):** YES -- the facilitator sees everything in real time and can intervene naturally.

---

## Comparison Matrix

| Dimension | 1: The Interview | 2: The Inbox | 3: The Head Start | 4: The Kickoff Call | 5: The Shared Canvas |
|-----------|:-:|:-:|:-:|:-:|:-:|
| Client behavior change | Medium | None | Low | Low | Medium |
| Client effort | Medium | Low | Low | Low | Medium-High |
| Facilitator effort | Low (after setup) | Low | Low | High | Medium |
| Technology complexity | Medium | High | Medium | Low | Medium |
| Sophistication spectrum | Strong | Strong | Partial | Strong | Partial |
| Personal touch | Moderate | High | High | Highest | High |
| Scalability | High | High | High | Low | Medium |
| Q1 addressed | Direct | Reframed | Reframed | Reframed | Direct |
| Q3 addressed | Yes | Yes | Partial | Yes | Partial |
| Q4 addressed | Partial | Yes | Yes | Yes | Yes |
| Risk profile | Safe/Proven | Bold/Complex | Moderate | Bold/Labor | Moderate |

---

## A Note on These Solutions

These five approaches represent genuinely different mechanisms for solving the same target problem. They are not ranked. Each makes a different bet about what matters most:

- **The Interview** bets that structure and guidance will change client behavior.
- **The Inbox** bets that you should never ask clients to change at all.
- **The Head Start** bets that doing the work for the client is better than asking them to do it.
- **The Kickoff Call** bets that human conversation is irreplaceable and technology should support it, not replace it.
- **The Shared Canvas** bets that collaboration is the highest-trust way to collect information.

The right choice depends on what this business values most and what risk it is willing to take. That decision belongs to the next phase.
