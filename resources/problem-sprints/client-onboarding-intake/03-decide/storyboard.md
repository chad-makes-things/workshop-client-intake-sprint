# Storyboard -- "The Interview" + "The Head Start" Hybrid

## Selected Direction

Solution 1 (The Interview) with Solution 3's web-scraping pre-population integrated as a core feature. A TurboTax-style guided intake flow where the system does its homework first, and the client reviews pre-filled content before filling gaps through branching questions, file uploads, and flexible input.

---

## Step 1: The Welcome Email

**Scene:** The client -- a founder named Sarah -- signed up for the workshop two days ago. She is in her inbox on a Tuesday morning, scanning emails between meetings. She sees a message from the facilitator with the subject line: "Getting started on your workshop -- one link, 20 minutes."

**Action:** Sarah opens the email. It contains two short paragraphs from the facilitator -- warm, personal, first-name -- and a single button: "Start Your Workshop Prep." Below the button, a one-line note: "You can pause and come back anytime. We've already done some homework on [Company Name] to give you a head start."

**System Response:** Behind the scenes, the system triggered the web scraping pipeline the moment Sarah's workshop booking was confirmed (two days ago). By the time Sarah opens this email, the system has already crawled her company website (homepage, about, services/products, team, blog), LinkedIn company page, social profiles, and any press mentions or review sites. An AI model has read all harvested content and generated a first-pass draft mapped to the framework categories: market context, customer profile, competitive landscape, brand positioning, and company story. The email was held until the scrape and draft were complete.

**User Reaction:** Sarah is curious about the "homework" line. She expected a long questionnaire or a vague "send us your materials" request. The idea that they have already started is unexpected. She clicks the button.

**Critical Detail:** The email must come from the facilitator's actual name and email address, not a system address. The tone must match the facilitator's natural voice -- this is the first brand impression of the intake experience. The "homework" tease must be specific enough to intrigue without overpromising. If the scrape found very little (thin web presence), the email copy adjusts: "We've pulled together some initial thoughts on [Company Name] -- you'll fill in the rest" rather than "We've already done our homework," so expectations are calibrated.

---

## Step 2: The Facilitator Video Kickoff

**Scene:** Sarah clicks the link and lands on a clean, branded page. No form fields yet. Instead, a short video (60-90 seconds) of the facilitator plays -- or is ready to play -- front and center.

**Action:** Sarah watches the video. The facilitator speaks directly to camera: "Hey, welcome. Before our workshop, I need to understand your business -- your market, your customers, what makes you different. This little interview is how we do that. It takes about 20 minutes, but you can stop and come back whenever. We've already pulled some info from your website to save you time -- you'll see it as you go. Just review what we found, correct anything that's off, and fill in the gaps. Let's get into it." Below the video, a single button: "Let's Go."

**System Response:** The page loads the client's session, linking their identity to the pre-scraped data and the framework template. A progress tracker initializes at 0%. The system logs the session start time.

**User Reaction:** The video makes this feel human, not robotic. Sarah recognizes the facilitator from the sales conversation. The 20-minute estimate and the "pause anytime" framing reduce anxiety. She does not feel like she is about to fill out a government form. She clicks "Let's Go."

**Critical Detail:** The video must be recorded once and reused for all clients -- it is not personalized per client (that would not scale). But it must feel personal: direct eye contact, conversational tone, first-person language. The facilitator's face and voice are the bridge between the relational brand and the systematized intake. If this video feels scripted or corporate, the personal touch scores collapse. The page design must be minimal -- no sidebar navigation, no login wall, no branding clutter.

---

## Step 3: Self-Segmentation

**Scene:** After the video, the first screen appears. It is a single question with three to four options, large and tappable, Typeform-style.

**Action:** Sarah reads: "Before we dive in, which of these sounds most like your situation?" The options are:

- "We have brand materials, pitch decks, and/or market research ready to share"
- "We have some materials but they're scattered or outdated"
- "Most of what we know is in our heads -- we haven't documented much"
- "We're early stage -- still figuring out our positioning"

Sarah selects: "We have some materials but they're scattered or outdated."

**System Response:** The system records her segmentation choice and configures the interview path. Her selection means: (a) the interview will present pre-scraped content for review AND ask upload prompts for existing documents, (b) questions will go to moderate depth -- more than the "brand deck ready" path (which focuses on uploads with brief context) but less narrative-heavy than the "it's all in our heads" path (which asks open-ended storytelling questions), (c) the system flags that she likely has materials to upload but may need prompting to locate them.

**User Reaction:** Sarah appreciates that the system is not assuming she is starting from zero -- or that she has everything polished. The "scattered or outdated" option feels like it was written for her. She does not feel categorized or judged.

**Critical Detail:** The segmentation options must use client language, not framework jargon. No option should feel like a confession of inadequacy -- "We haven't documented much" is neutral, not "We don't have anything." The system must handle clients who select the wrong segment gracefully; if a "documents ready" client turns out to have very little, the interview must adapt mid-flow rather than dead-ending. This segmentation determines the entire question tree, so the branching logic must be rigorously tested across all four paths.

---

## Step 4: Pre-Populated Review -- "Here's What We Found"

**Scene:** The interview transitions into its first substantive section. Instead of a blank text field, Sarah sees a card with a heading -- "Your Market" -- and a paragraph of text already filled in, lightly highlighted to signal it was auto-generated.

**Action:** Sarah reads the pre-populated content: "Based on what we found, [Company Name] operates in the direct-to-consumer wellness space, offering [specific products/services pulled from website]. Your market includes health-conscious consumers aged 25-45 who prioritize [value proposition pulled from site copy]." Below the text, three options: "This is right," "Close, but let me edit," and "This is off -- let me rewrite." Sarah clicks "Close, but let me edit." An inline editor opens. She adjusts the customer age range and adds a sentence about a new market segment they are targeting that is not on the website yet.

**System Response:** The system saves her edits, tags the section as "client-reviewed and modified," and advances to the next framework area. The progress bar moves. The system also logs what was changed -- the delta between the AI draft and the client's version -- because these deltas are often the most valuable inputs (they reveal what the public narrative misses about the company's actual strategy).

**User Reaction:** Sarah is impressed. The draft is not perfect, but it is 70% right -- and that is dramatically better than a blank page. She feels like the system "gets" her business, or at least has made a credible attempt. The edit interaction feels quick and low-effort. She thinks: "This is way better than trying to write this from scratch."

**Critical Detail:** The pre-populated content must be clearly marked as AI-generated so the client understands they are reviewing, not reading final copy. The three response options (confirm / edit / rewrite) must be visually distinct and always available -- forcing a client to accept inaccurate content because there is no easy "fix" button would destroy trust. For thin-web-presence clients where the AI draft is sparse or weak, the system must shift to a question-based approach for that section instead of displaying a thin draft: "We couldn't find much about your market online -- let's build this together" followed by guided questions. The transition between "review mode" and "question mode" must be seamless.

---

## Step 5: The Guided Deep-Dive -- Branching Questions

**Scene:** After reviewing two to three pre-populated sections, the interview reaches an area where the web scrape found little -- Competitive Landscape. The system does not have enough to pre-populate, so it switches to guided questions.

**Action:** Sarah sees: "Now let's talk about your competition. Who are the 2-3 companies your customers would consider instead of you?" A text field waits below. Sarah types three competitor names. The system immediately responds with a follow-up: "What makes you different from [Competitor 1]?" She answers. Then: "Is there anything [Competitor 2] does better than you right now?" She pauses, thinks, then types a candid answer about their faster shipping. Another follow-up: "Do you have any competitive analysis documents or comparison charts you could upload?" Sarah remembers a slide deck with a competitive matrix. She clicks "Upload a file" and drags in the PDF.

**System Response:** Each answer is tagged to the Competitive Landscape framework slot. The uploaded PDF is processed: text extracted, key competitive claims identified, content mapped to supplement her typed answers. The system merges her conversational answers with the document content to build a rich competitive profile. The branching logic adapts: because she uploaded a competitive document, the system skips three follow-up questions about competitive positioning that would have been redundant. The progress bar jumps forward.

**User Reaction:** The one-question-at-a-time rhythm feels manageable. Sarah is not staring at a 30-field form. She is having something that resembles a conversation -- a question, her answer, a smart follow-up. The upload prompt came at exactly the right moment: she would not have thought to send that slide deck unprompted, but when asked in context, it was obvious. She feels like the system is listening.

**Critical Detail:** The branching logic is the intellectual core of the system. Each question must map to a specific framework slot, and follow-ups must be conditional on previous answers. If a client says "we don't really have competitors," the system must not dead-end -- it should pivot: "Who were your customers using before they found you?" The upload prompt must be contextual, not generic -- "Upload your competitive analysis" works here because the conversation is about competition. A random "upload any files" prompt mid-interview would feel like a form, not a conversation. File processing must be fast enough that the system can adapt subsequent questions based on what was in the uploaded document.

---

## Step 6: Flexible Input -- Voice, Links, and Freeform

**Scene:** The interview reaches Brand Story and Positioning. Sarah has strong feelings about this but has never written them down. The text field feels limiting.

**Action:** Sarah sees: "What's the story behind [Company Name]? How did it start, and what drives it today?" Below the question, multiple input options are visible: a text field, a microphone icon labeled "Record your answer," and a link icon labeled "Paste a link." Sarah taps the microphone icon and speaks for 90 seconds about how she started the company after her own frustrating experience as a customer. She finishes, sees a waveform confirming the recording, and taps "Done." The next question asks about brand voice: "If your brand were a person, how would they talk?" Sarah types a few sentences. Two questions later, the system asks about thought leadership: "Do you have any blog posts, podcast appearances, or articles that capture your perspective?" Sarah pastes a link to a podcast episode.

**System Response:** The voice recording is transcribed in near real-time, then mapped to the Brand Story framework slot. The transcription captures the founder narrative, emotional drivers, and origin story -- information that typed answers rarely convey with the same richness. The pasted podcast link is queued for processing: the system will pull the episode, transcribe it, and extract relevant positioning and perspective content. All inputs -- typed, spoken, and linked -- merge into a unified Brand Story and Positioning section in the framework. The system notes that Sarah's verbal storytelling was significantly richer than her typed answers, which may inform how the facilitator approaches the workshop.

**User Reaction:** The voice option is a relief. Sarah is a talker, not a writer. Speaking her founder story felt natural -- almost like being interviewed. She did not have to organize her thoughts into perfect paragraphs. The link paste felt effortless -- she already knew that podcast was a good representation of her thinking, and dropping it in took two seconds. She thinks: "They're making it easy for me to give them what I actually have, not what I think they want."

**Critical Detail:** Voice recording must work reliably on mobile and desktop with a single tap -- no app downloads, no permissions nightmares. The transcription must be fast enough that the client can review it before moving on (or at minimum, confirm it captured something). The system must handle all three input types (text, voice, links) for every question, not just designated ones -- clients should never feel forced into a single input mode. Link processing (scraping a podcast, pulling a blog post) can happen asynchronously after submission, but the client must see confirmation that the link was received.

---

## Step 7: The Pause and Return

**Scene:** Sarah is 65% through the interview. Her phone rings -- it is a client call she cannot miss. She has been doing this on her laptop during a quiet afternoon block.

**Action:** Sarah closes her browser tab. No save button, no warning dialog. She simply leaves.

**System Response:** The system auto-saved after every answer. Her session state is preserved exactly where she left off. Four hours later, the system sends an automated message (email or SMS, based on her preference): "You're 65% done -- about 8 minutes left. Your workshop is in 11 days. Pick up where you left off: [link]." The message includes a one-line summary of what is already captured to reinforce that her effort was not lost.

**User Reaction:** When Sarah returns that evening on her phone, she taps the link and lands exactly where she stopped -- mid-section, with all previous answers intact. There is no login wall, no "find your session" confusion. She thinks: "Oh good, it saved everything." She finishes the remaining 35% in 10 minutes on her couch.

**Critical Detail:** Auto-save must be invisible and bulletproof. No "are you sure you want to leave?" dialogs -- these feel desperate and annoying. The session link must work across devices without requiring account creation or login. A magic link or token-based URL is preferable. The reminder messages must be helpful, not nagging -- one reminder at 24 hours is fine; three reminders in 48 hours is spam. The system must adapt reminder timing to the workshop date: if the workshop is 3 weeks away, reminders are gentle; if 3 days away, they are more direct. The reminder must never imply the client is behind or failing -- the tone is "we saved your progress" not "you haven't finished."

---

## Step 8: Review and Submit

**Scene:** Sarah reaches the end of the interview. A summary screen appears showing all framework sections with her content -- some pre-populated and confirmed, some edited from the AI draft, some built from her answers and uploads.

**Action:** Sarah scrolls through the summary. Each section has a green checkmark (complete) or a yellow flag (thin -- optional to add more). She sees her competitive landscape section is marked green -- her typed answers plus the uploaded PDF produced a thorough section. Her "Aspirations" section is yellow: she gave a brief answer and the system notes "You might want to add more here -- where do you want to be in 3 years?" Sarah taps in and adds two more sentences. She scrolls to the bottom and taps "Submit."

**System Response:** The system packages all inputs into a structured framework document: each section contains the merged content from pre-population, client edits, typed answers, transcribed voice clips, and processed file uploads. Metadata is attached -- which sections were AI-drafted vs. client-authored, which had file uploads, which were thin. A completeness score is calculated. A notification is sent to the facilitator: "Sarah Chen at [Company Name] has completed intake. Completeness: 87%. 3 file uploads processed. Review ready."

**User Reaction:** Sarah feels a sense of completion. The summary showed her that the system captured a lot -- more than she realized she had given. The yellow flags felt like gentle suggestions, not demands. Tapping "Submit" felt like handing off something substantial, not dropping files into a void. She thinks: "That was actually... fine? Maybe even good?"

**Critical Detail:** The summary must be scannable, not overwhelming. Section titles with brief excerpts and status indicators -- not full text dumps. The yellow flags must be optional: a client who wants to submit at 75% completeness should be able to. The system must never block submission because of incomplete sections -- that would re-create the "I can't finish this" paralysis that causes the current process to fail. The completeness score sent to the facilitator is for internal use only -- the client never sees a grade.

---

## Step 9: The Facilitator Receives Structured Output

**Scene:** The facilitator -- working from home office -- receives the notification on their dashboard. It is one of four active clients in the pipeline. They click into Sarah's intake.

**Action:** The facilitator opens a structured view of Sarah's submission. The dashboard shows the framework sections as tabs or cards, each with a completeness indicator and a content type breakdown (AI-drafted, client-edited, client-authored, file-extracted, voice-transcribed). The facilitator clicks into "Competitive Landscape" and sees: three named competitors with Sarah's descriptions of differentiation, plus extracted content from her uploaded competitive matrix PDF, all merged into a coherent section. They click "Brand Story" and read the transcription of Sarah's 90-second voice recording -- the founder narrative is vivid and specific, the kind of story that writes itself into a market narrative. The facilitator flags one section -- "Market Sizing" -- where Sarah's input was thin and the web scrape had nothing. They click "Request Follow-Up" and the system generates a targeted two-question follow-up message to Sarah.

**System Response:** The follow-up message is sent to Sarah: "Thanks for completing your workshop prep -- great stuff. Quick follow-up: could you share any data or estimates on your total addressable market? Even a rough sense of market size helps us build your narrative." The dashboard updates to show a pending follow-up for Sarah's intake. The facilitator did not write the follow-up -- the system drafted it based on the gap, and the facilitator approved it with one click.

**User Reaction (Facilitator):** The facilitator sees structured, framework-ready content without having manually chased, downloaded, organized, or categorized a single file. They did not send 6 emails asking for materials. They did not download a PDF from Dropbox, a photo from text messages, and a doc from email. They see one unified view per client, organized by framework section, with clear gaps identified. They think: "I can actually work with this. This is what I needed, in the shape I needed it." The follow-up flow means they did not have to compose a message -- just confirm it. For four concurrent clients, this dashboard view replaces what used to be 15+ hours of manual collection and organization.

**Critical Detail:** The facilitator dashboard must present content by framework section, not by submission chronology. The facilitator does not care that Sarah uploaded a PDF at 3:47 PM -- they care that the Competitive Landscape section now has three named competitors with differentiation statements. Content provenance (AI-drafted vs. client-authored vs. file-extracted) must be visible but not intrusive -- it helps the facilitator assess reliability. The one-click follow-up must draft messages in the facilitator's voice, not system-speak. The facilitator must be able to edit the draft before sending. If the facilitator cannot trust the system's gap detection, they will revert to manually reviewing raw inputs -- and the system has failed.

---

## Step 10: Framework-Ready Output for Narrative Generation

**Scene:** Three days later. Sarah's follow-up answers have arrived. The facilitator's dashboard shows her intake at 94% completeness. The workshop is in 8 days.

**Action:** The facilitator clicks "Generate Narrative Package" on Sarah's intake. The system compiles all framework sections -- pre-populated content, client edits, interview answers, file extractions, voice transcriptions, and follow-up responses -- into the structured prompt inputs that feed the market narrative AI. The facilitator reviews the compiled package: each prompt slot is filled, sources are cited (e.g., "from client voice recording" or "from uploaded competitive matrix, page 3"), and confidence levels are marked. The facilitator adjusts one section where the AI merge was awkward, approves the rest, and triggers narrative generation.

**System Response:** The structured prompt framework -- the same framework the facilitator previously assembled by hand over hours of downloading, reading, categorizing, and organizing scattered materials -- is now assembled automatically from the intake system's output. It feeds into the existing AI narrative generation pipeline (Steps 11-12 on the problem map). The facilitator's manual effort for this client's intake: approximately 15 minutes of review and one follow-up approval. Previous manual effort for the same work: 3-5 hours.

**User Reaction (Facilitator):** The output is in the exact format the facilitator already uses for prompt generation. Nothing was reformatted, re-categorized, or lost in translation. The facilitator's expertise is embedded in the question tree design and the framework mapping -- they are not being replaced, they are being leveraged. The system did the logistics (collecting, organizing, merging, gap-detecting) while preserving the facilitator's intellectual architecture (the framework itself). For the first time, the facilitator can see a path to 10 concurrent clients without 10x the prep work. They think: "This is the same quality I would have produced -- I just didn't have to do the manual labor to get here."

**Critical Detail:** The compiled output must map one-to-one to the facilitator's existing prompt framework. If the system introduces a new structure or reorganizes the framework, the facilitator will not trust it. The output format is sacred -- the system adapts to the facilitator's existing workflow, not the other way around. Source attribution in the compiled package is essential for facilitator confidence: they need to know whether a competitive claim came from the client's mouth, the client's uploaded document, or the AI's web scrape, because the reliability weighting differs. This step is the bridge between the intake system (the sprint's scope) and the narrative generation system (a future sprint's scope) -- it must produce a clean handoff.

---

## Storyboard Summary

| Step | Title | Actor | Key Moment |
|:----:|-------|-------|------------|
| 1 | The Welcome Email | Client | Receives single link with "we did our homework" tease |
| 2 | The Facilitator Video Kickoff | Client | Watches personal video; anxiety drops; clicks "Let's Go" |
| 3 | Self-Segmentation | Client | Selects sophistication level; system configures branching path |
| 4 | Pre-Populated Review | Client | Sees AI-drafted content from web scrape; edits and confirms |
| 5 | Guided Deep-Dive | Client | Answers branching questions; uploads files in context |
| 6 | Flexible Input | Client | Uses voice recording, link pasting, and text as needed |
| 7 | Pause and Return | Client + System | Leaves mid-session; returns seamlessly across devices |
| 8 | Review and Submit | Client | Scans summary; adds to flagged gaps; submits |
| 9 | Facilitator Receives Output | Facilitator | Views structured framework dashboard; approves follow-up |
| 10 | Framework-Ready Package | Facilitator | Compiles prompt inputs; triggers narrative generation |

---

## Sprint Questions Addressed by This Storyboard

| Question | How This Storyboard Answers It |
|----------|-------------------------------|
| Q1: Will clients submit through a structured system? | Steps 1-8 design the path from email to submission. One link, guided flow, pause-and-resume, no ambiguity about what to do next. |
| Q2: Will quality equal manual? | Steps 4-6 combine web-scraped pre-population, branching questions, file uploads, and voice input to produce multi-source framework-mapped content. Step 10 shows the output matching the facilitator's existing format. |
| Q3: Can it handle the sophistication spectrum? | Step 3 segments clients into appropriate paths. Step 4 adapts between review mode (rich web presence) and question mode (thin web presence). Step 5 shows branching depth adjusting per client. |
| Q4: Can it avoid feeling impersonal? | Step 2 preserves the facilitator's face and voice. Step 4's "we did our homework" creates a premium impression. Steps 5-6 feel conversational, not bureaucratic. |
| Q5: Will the facilitator trust and adopt this? | Steps 9-10 show the facilitator receiving exactly what they need in exactly the format they use, with source attribution and gap detection. The facilitator designed the question tree -- the system reflects their expertise. |
