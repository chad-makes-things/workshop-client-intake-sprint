# Epic: Workshop Intake Engine (WIE)

## Epic Overview

**Name:** Workshop Intake Engine

**Objective:** Replace manual, multi-channel client material collection with a guided, AI-augmented intake system that scrapes the client's web presence, walks them through a branching conversational flow, accepts multi-modal input, and delivers structured framework-ready output to the facilitator -- reducing per-client intake effort from 3-5 hours to under 15 minutes of facilitator review.

**Sprint Origin:** Client Onboarding Intake Sprint (2026-03-07). Solution: "The Interview" + "The Head Start" hybrid. Full sprint artifacts at `resources/problem-sprints/client-onboarding-intake/`.

---

### Personas

| Persona | Description | Key Needs |
|---------|-------------|-----------|
| **Client** | Business representative (founder, marketing lead, ops person) signing up for the workshop. Varying digital sophistication -- from polished brand decks to "most of what we know is in our heads." | Clear guidance on what to provide; low-friction input; ability to pause and resume; confidence that their effort is captured and valued. |
| **Facilitator** | Workshop business owner. Currently the bottleneck for all intake logistics. Designs the question tree and framework. Reviews structured output. | Structured output in their existing framework format; gap detection; one-click follow-up; source attribution; zero time spent chasing materials. |

---

### Success Metrics

| Metric | Baseline (Current) | Target (V1) | Measurement |
|--------|-------------------|-------------|-------------|
| Facilitator intake time per client | 3-5 hours | < 30 minutes (review + follow-up) | Time tracking on dashboard |
| Client intake completion rate | ~60% (estimated, with chasing) | > 85% without facilitator intervention | Submission rate / bookings |
| Time from booking to complete intake | 7-14 days (with follow-up cycles) | < 5 days median | Booking date to submission timestamp |
| Framework section coverage at submission | Variable, many gaps | > 75% of sections at "sufficient" threshold | Completeness scoring engine |
| Concurrent client capacity (no added labor) | 3-4 clients | 10+ clients | Dashboard pipeline count |

---

## User Stories by Development Phase

---

### Phase 1: Intake Flow Engine + Session Manager

*The core interview experience. A client can access the intake via a link, watch the kickoff video, self-segment, answer branching questions via text input, review a summary, and submit. Sessions persist across devices and browser sessions. No pre-population or file processing yet.*

---

#### WIE-001: Magic Link Access

**As a** client, **I want to** access my intake session via a unique link without creating an account **so that** I can start immediately without friction.

**Acceptance Criteria:**

1. **Given** a client has a valid magic link URL, **when** they click it, **then** the system loads their intake session without requiring login, password, or account creation.
2. **Given** a client clicks an expired magic link (>30 days past workshop date), **when** the page loads, **then** the system displays a message explaining the link has expired and provides a contact path to the facilitator.
3. **Given** a client clicks a magic link on a different device than where they started, **when** the page loads, **then** the system restores their exact session state including current position and all previous answers.
4. **Given** two browser tabs open the same magic link simultaneously, **when** the client submits an answer in one tab, **then** the other tab does not overwrite that answer or corrupt session state.
5. **Given** a client's magic link token, **when** an unauthorized party attempts to guess or enumerate tokens, **then** the token space is large enough (UUID v4 or equivalent) to make brute-force infeasible.

**Priority:** must_have
**Size:** M
**Dependencies:** None (foundational)
**Technical Notes:** Magic link tokens stored server-side in PostgreSQL. Token embedded in URL path, not query string (cleaner sharing). Secure HTTP-only cookie set on first visit for same-device return. Token refreshes on each access per ADR-4. Redis cache for fast session lookup.

---

#### WIE-002: Facilitator Video Kickoff

**As a** client, **I want to** see a personal video from the facilitator before the interview begins **so that** the experience feels human and I understand what to expect.

**Acceptance Criteria:**

1. **Given** a client accesses the intake for the first time, **when** the page loads, **then** the system displays a video player with the facilitator's kickoff video (60-90 seconds) and a single "Let's Go" button below it.
2. **Given** the client has watched the video and clicked "Let's Go," **when** they return to the intake later, **then** the system skips the video and resumes at their last position.
3. **Given** a client on a slow connection, **when** the video fails to load within 5 seconds, **then** the system displays a text-based welcome message with the same content and the "Let's Go" button remains accessible.
4. **Given** a client on mobile, **when** they view the kickoff page, **then** the video and button render correctly and the video plays inline (not full-screen forced).

**Priority:** must_have
**Size:** S
**Dependencies:** WIE-001
**Technical Notes:** Video hosted on CDN (S3 + CloudFront or equivalent). Single video file reused for all clients. Facilitator records once. Page design is minimal -- no sidebar, no nav, no login wall. Progress tracker initializes at 0%.

---

#### WIE-003: Self-Segmentation

**As a** client, **I want to** select which description best matches my situation **so that** the interview adapts to my level of preparation.

**Acceptance Criteria:**

1. **Given** the client has clicked "Let's Go," **when** the first screen loads, **then** the system displays a single question with 4 selectable options rendered as large, tappable cards (Typeform-style).
2. **Given** the client selects an option, **when** they confirm, **then** the system records their segmentation choice and configures the branching question path accordingly.
3. **Given** the client selects "We have brand materials, pitch decks, and/or market research ready to share," **when** the interview proceeds, **then** the question path emphasizes upload prompts with brief contextual questions.
4. **Given** the client selects "Most of what we know is in our heads," **when** the interview proceeds, **then** the question path emphasizes open-ended narrative questions with greater depth.
5. **Given** a client who selected "documents ready" but provides minimal content through the first 3 sections, **when** the system detects thin responses, **then** the flow adapts by surfacing additional guided questions rather than dead-ending.

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-001, WIE-008 (Framework Template Store)
**Technical Notes:** Segmentation choice stored in Intake Session record. Branching paths defined in the question tree JSON (ADR-1). The 4 options use client language, never framework jargon. No option should imply inadequacy. Mid-flow adaptation requires the Intake Flow Engine to evaluate answer depth against section thresholds.

---

#### WIE-004: Branching Question Flow

**As a** client, **I want to** answer one question at a time in a conversational sequence **so that** I am not overwhelmed by a long form and can focus on each topic.

**Acceptance Criteria:**

1. **Given** the client has completed self-segmentation, **when** the interview begins, **then** questions are presented one at a time with progressive disclosure (Typeform-style).
2. **Given** the client submits an answer, **when** the next question loads, **then** it is determined by the branching logic in the question tree based on the segmentation path and prior answers.
3. **Given** the client is on a question that has conditional follow-ups, **when** they answer in a way that triggers a follow-up (e.g., naming competitors), **then** the system presents the follow-up question before advancing to the next section.
4. **Given** the client answers "we don't really have competitors," **when** the system receives this response, **then** it pivots to an alternative question ("Who were your customers using before they found you?") rather than dead-ending the section.
5. **Given** the client is progressing through questions, **when** they look at the interface, **then** a progress bar shows estimated completion percentage and updates after each answer.
6. **Given** the interview flow, **when** any question is displayed, **then** the client can see which framework section they are in (e.g., "Your Market," "Your Competition") without exposing internal framework terminology.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-003, WIE-008
**Technical Notes:** The Intake Flow Engine traverses a directed graph (JSON) where nodes are questions and edges are conditional transitions. Each question node references a framework section key. The engine must handle all 4 segmentation paths with 20-40 questions each. Build a question tree validator that verifies all paths are reachable and all terminal nodes produce sufficient framework coverage.

---

#### WIE-005: Auto-Save and Session Persistence

**As a** client, **I want** my progress to be saved automatically after every interaction **so that** I can close the browser and return without losing any work.

**Acceptance Criteria:**

1. **Given** the client submits an answer, edits content, or completes any interaction, **when** the action completes, **then** the system auto-saves session state within 2 seconds without any visible "save" button or dialog.
2. **Given** the client closes the browser tab mid-session, **when** they return via the magic link (same or different device), **then** the system restores their exact position including the current question, all previous answers, and UI state.
3. **Given** the client loses internet connectivity while answering a question, **when** connectivity is restored, **then** the system retries the save and does not lose the in-progress answer.
4. **Given** the client navigates away from the intake page, **when** leaving, **then** the system does NOT display a "are you sure you want to leave?" dialog.
5. **Given** the client returns after 7 days of inactivity, **when** they access the magic link, **then** the session restores fully with a brief "Welcome back" message summarizing where they left off.

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-001
**Technical Notes:** Auto-save fires on every interaction via API call. Session state stored in PostgreSQL with Redis cache for fast reads. State includes: current step, current section, all submissions, segmentation choice, UI state. Optimistic saves with retry on failure. Consider service worker for offline queuing of saves.

---

#### WIE-006: Text Input for Questions

**As a** client, **I want to** type my answers into a text field for each question **so that** I can provide my responses in the most straightforward way.

**Acceptance Criteria:**

1. **Given** a question is displayed, **when** the client views the input area, **then** a text field is presented as the default input method.
2. **Given** the client types an answer and submits, **when** the system processes the submission, **then** the answer is tagged to the correct framework section based on the question's pre-defined mapping.
3. **Given** the client submits an empty text field, **when** the system processes the submission, **then** the system allows progression (no mandatory fields) but marks the section as incomplete in internal tracking.
4. **Given** the client types a very long answer (>5000 characters), **when** they submit, **then** the system accepts the full response without truncation.

**Priority:** must_have
**Size:** S
**Dependencies:** WIE-004
**Technical Notes:** Text stored in the Submission table with `input_type: text` and the appropriate `framework_section` key. All submissions linked to the active Intake Session.

---

#### WIE-007: Review and Submit

**As a** client, **I want to** see a summary of everything I have provided before submitting **so that** I can verify completeness and add to any thin sections.

**Acceptance Criteria:**

1. **Given** the client reaches the end of the question flow, **when** the summary screen loads, **then** each framework section is displayed with a brief excerpt of the content provided and a status indicator (green checkmark for sufficient, yellow flag for thin).
2. **Given** a section is marked with a yellow flag, **when** the client taps on it, **then** the system opens that section for additional input with a contextual prompt (e.g., "You might want to add more here -- where do you want to be in 3 years?").
3. **Given** a client wants to submit with yellow-flagged sections remaining, **when** they tap "Submit," **then** the system allows submission without blocking -- yellow flags are suggestions, not requirements.
4. **Given** the client taps "Submit," **when** the submission processes, **then** the system displays a confirmation message, packages all inputs into a structured framework document, and calculates a completeness score.
5. **Given** the client has submitted, **when** they access the magic link again, **then** the system shows a "Thank you, your submission has been received" page rather than re-opening the interview.
6. **Given** a submission is completed, **when** the system processes it, **then** a notification is sent to the facilitator including: client name, company name, completeness score, and a link to the dashboard view.

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-004, WIE-006
**Technical Notes:** Completeness scoring uses the minimum thresholds defined per Framework Section. The summary screen is scannable -- section titles, short excerpts, status icons. No full content dumps. Completeness score is internal (facilitator-facing) and never displayed as a numeric grade to the client. The client sees green/yellow indicators only.

---

#### WIE-008: Framework Template Store (Initial)

**As a** facilitator, **I want** the question tree and framework sections to be stored as configuration **so that** the interview structure can evolve without code changes.

**Acceptance Criteria:**

1. **Given** the system initializes, **when** the Intake Flow Engine loads, **then** it reads the question tree structure and framework section definitions from a JSON configuration store, not from hardcoded application logic.
2. **Given** a framework section definition, **when** it is read by the system, **then** it includes: section key, display name, description, minimum completeness threshold, and display order.
3. **Given** a question tree configuration, **when** the Intake Flow Engine traverses it, **then** each node contains: question text, target framework section, input prompt type, conditional edges to next nodes based on answer values, and segmentation path applicability.
4. **Given** a change to the question tree configuration, **when** a client has an in-progress session using the previous version, **then** the client's session continues using the version they started with (versioned templates).
5. **Given** a new question tree version, **when** it is deployed, **then** only new sessions use the updated tree.

**Priority:** must_have
**Size:** L
**Dependencies:** None (foundational)
**Technical Notes:** Stored in PostgreSQL (Framework Section table + question_tree JSON column). Versioned via template_version integer. The question tree is a directed graph: nodes are questions, edges are conditional transitions. Build a CLI or admin validation tool that checks: all paths reachable, all terminal nodes produce framework coverage, no orphan nodes. Framework categories from architecture doc: market context, customer profile, competitive landscape, brand positioning, company story, aspirations.

---

#### WIE-009: Intake Session Data Model and API

**As a** developer, **I want** the core data model and API endpoints for intake sessions **so that** the frontend can create, read, update, and submit sessions.

**Acceptance Criteria:**

1. **Given** a new client booking webhook, **when** the system receives it, **then** it creates Client, Workshop Booking, and Intake Session records with a generated magic link token.
2. **Given** a valid magic link token, **when** the API receives a GET request, **then** it returns the full session state: current position, segmentation choice, all submissions, completion percentage, and session status.
3. **Given** a client submits an answer, **when** the API receives the POST request, **then** it creates a Submission record tagged with the correct framework section, input type, and session reference, and updates the session's current step and completion percentage.
4. **Given** a client taps "Submit," **when** the API receives the submission request, **then** it transitions the session status to "submitted," sets the submitted_at timestamp, and triggers the facilitator notification.
5. **Given** any API request with an invalid or expired token, **when** the system processes it, **then** it returns a 401 response with a clear error message.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-008
**Technical Notes:** Implements the data model from the architecture doc (Client, Workshop Booking, Intake Session, Submission, Framework Section entities). API is RESTful, Node.js/TypeScript. PostgreSQL for persistence, Redis for session cache. Webhook integration point for existing booking/CRM system -- initially can be a manual trigger or simple API call.

---

### Phase 2: Web Scraping Engine + AI Content Processor

*The "Head Start" feature. Before the client begins, the system scrapes their web presence and generates pre-populated content for framework sections. The client reviews AI-drafted content and confirms, edits, or rewrites.*

---

#### WIE-010: Web Scraping Engine

**As a** system, **I want to** crawl a client's public web presence after booking confirmation **so that** framework sections can be pre-populated before the client starts the interview.

**Acceptance Criteria:**

1. **Given** a new workshop booking with a company URL, **when** the booking is confirmed, **then** the system queues a scrape job that crawls the company website (homepage, about, services/products, team, blog index) within a 25-page cap.
2. **Given** a scrape job is running, **when** a page returns JavaScript-rendered content (empty in static scrape), **then** the system falls back to headless browser rendering (Tier 2) for that page.
3. **Given** a scrape job encounters a page behind a login wall or returning 403/401, **when** processing that page, **then** the system skips the page, logs it as inaccessible, and continues crawling remaining pages.
4. **Given** a scrape job, **when** all target pages are processed, **then** the system extracts clean text content (stripped of navigation, footers, boilerplate) with page type classification (homepage, about, services, etc.).
5. **Given** a company with a very thin web presence (fewer than 3 pages with meaningful content), **when** the scrape completes, **then** the system flags the result as "thin" so downstream email copy and pre-population behavior adapt.
6. **Given** a scrape job, **when** it runs, **then** it respects robots.txt, enforces rate limiting (max 2 requests/second per domain), and completes within 5 minutes.
7. **Given** a scrape job fails entirely (site unreachable, DNS failure), **when** the failure is detected, **then** the system marks the job as failed and does not block the welcome email or intake flow -- the client proceeds without pre-populated content.

**Priority:** must_have
**Size:** XL
**Dependencies:** WIE-009
**Technical Notes:** Two-tier approach per ADR-2. Tier 1: HTTP + cheerio/jsdom for static HTML. Tier 2: Playwright for JS-rendered pages. Priority pages crawled first. LinkedIn and social profiles via separate rate-limited workers. Store raw and extracted content in Scraped Page records. Scrape runs asynchronously via BullMQ job queue. Cap at 25 pages per site, depth limit of 2 levels from homepage.

---

#### WIE-011: AI Content Pre-Population

**As a** client, **I want to** see AI-drafted content based on my company's web presence when I start the interview **so that** I can review and correct rather than write from scratch.

**Acceptance Criteria:**

1. **Given** a completed scrape job with extracted content, **when** the AI Content Processor runs, **then** it generates draft content mapped to each framework section (market context, customer profile, competitive landscape, brand positioning, company story) with confidence scores.
2. **Given** pre-populated content for a framework section, **when** the client reaches that section in the interview, **then** the system displays the draft with clear visual marking that it is AI-generated, along with three response options: "This is right," "Close, but let me edit," and "This is off -- let me rewrite."
3. **Given** the client selects "Close, but let me edit," **when** the editor opens, **then** they can modify the text inline and submit the edited version.
4. **Given** the client selects "This is off -- let me rewrite," **when** the editor opens, **then** the pre-populated text is cleared and replaced with a blank text field (or the guided question for that section).
5. **Given** the client confirms or edits pre-populated content, **when** the system saves the submission, **then** it records the input type (pre-populated-confirmed, pre-populated-edited, or pre-populated-rewritten) and preserves the original AI draft for delta tracking.
6. **Given** a thin scrape result for a particular section, **when** the client reaches that section, **then** the system skips the pre-populated review mode and presents guided questions instead, with a transition like "We couldn't find much about [topic] online -- let's build this together."
7. **Given** the client edits pre-populated content, **when** the system stores the submission, **then** it logs the delta between the AI draft and the client's version as a high-value signal.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-010, WIE-004, WIE-008
**Technical Notes:** AI Content Processor uses LLM (Claude or GPT-4) with framework-specific system prompts. Structured output (JSON mode) for reliable section mapping. Confidence scores per section determine whether to show pre-populated review mode vs. question mode. Threshold: if confidence < 0.4 for a section, default to question mode. Delta tracking stored in Submission.original_pre_populated_content.

---

#### WIE-012: Welcome Email with Head Start Tease

**As a** client, **I want to** receive a warm, personal email with a single link to start my intake **so that** I know exactly what to do and feel that the workshop team has already invested in understanding my business.

**Acceptance Criteria:**

1. **Given** the scrape job and AI pre-population are complete, **when** processing finishes, **then** the system sends a welcome email to the client from the facilitator's name and email address.
2. **Given** a rich scrape result, **when** the email is composed, **then** it includes the line "We've already done some homework on [Company Name] to give you a head start" and a single "Start Your Workshop Prep" button.
3. **Given** a thin scrape result, **when** the email is composed, **then** the copy adjusts to "We've pulled together some initial thoughts on [Company Name] -- you'll fill in the rest" to calibrate expectations.
4. **Given** a failed scrape, **when** the email is composed, **then** the copy omits the "homework" reference entirely and focuses on the guided interview value proposition.
5. **Given** the email, **when** the client reads it, **then** the sender name, reply-to address, and tone match the facilitator's personal voice -- not system-generated language.

**Priority:** must_have
**Size:** S
**Dependencies:** WIE-010, WIE-011, WIE-001
**Technical Notes:** Transactional email via SendGrid or Postmark. Email templates stored with dynamic content slots. Three template variants: rich-scrape, thin-scrape, no-scrape. Email held until scrape + AI processing completes (target: within 5 minutes of booking). If scrape takes longer than 10 minutes, send the no-scrape variant and backfill pre-populated content when the client accesses the flow.

---

### Phase 3: Facilitator Dashboard

*The facilitator's view of all client intake data. Organized by framework section, with gap detection, source attribution, follow-up management, and framework-ready output compilation.*

---

#### WIE-013: Client Pipeline View

**As a** facilitator, **I want to** see all active clients in a pipeline view with status indicators **so that** I can track intake progress across my client portfolio at a glance.

**Acceptance Criteria:**

1. **Given** the facilitator accesses the dashboard, **when** the page loads, **then** it displays all clients with active workshop bookings in a list/card view showing: client name, company name, workshop date, intake status (not started / in progress / submitted / narrative ready), and completeness percentage.
2. **Given** multiple clients at different stages, **when** the facilitator views the pipeline, **then** clients are sorted by workshop date (nearest first) with visual urgency indicators for clients whose workshops are within 7 days and intake is incomplete.
3. **Given** a client has submitted their intake, **when** the facilitator views the pipeline, **then** that client's card shows the submission date, completeness score, number of file uploads, and a "Review" link.
4. **Given** the facilitator clicks on a client, **when** the detail view loads, **then** it navigates to the per-client framework view (WIE-014).

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-009, WIE-007
**Technical Notes:** Dashboard is a separate authenticated section of the Next.js application. Facilitator authentication via standard email/password or magic link (separate from client magic links). Pipeline data sourced from Workshop Booking and Intake Session tables.

---

#### WIE-014: Per-Client Framework View

**As a** facilitator, **I want to** view a client's intake data organized by framework section **so that** I can review content in the structure I use for narrative generation, not in submission chronology.

**Acceptance Criteria:**

1. **Given** the facilitator opens a client's detail view, **when** the page loads, **then** content is organized by framework section (tabs or cards) matching the facilitator's prompt framework structure, not by timestamp or submission order.
2. **Given** a framework section, **when** the facilitator views it, **then** it displays the merged content from all sources with a content type breakdown showing which portions are AI-drafted, client-edited, client-authored, file-extracted, or voice-transcribed.
3. **Given** a framework section, **when** the facilitator views it, **then** it shows a completeness score and confidence level for that section.
4. **Given** a framework section with source attribution, **when** the facilitator hovers or clicks on a content segment, **then** the system indicates the source (e.g., "from uploaded competitive matrix, page 3" or "from client voice recording").
5. **Given** any framework section, **when** the facilitator views it, **then** they can edit the compiled content directly in the dashboard before narrative generation.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-013, WIE-008
**Technical Notes:** Compiled Section records aggregate all submissions for a given booking + framework section. Content provenance tracked via Submission.input_type and source metadata. Facilitator edits saved to Compiled Section with facilitator_edited flag.

---

#### WIE-015: Gap Detection and Follow-Up

**As a** facilitator, **I want** the system to identify gaps in framework coverage and draft follow-up questions I can approve and send **so that** I do not have to manually review every section and compose follow-up messages.

**Acceptance Criteria:**

1. **Given** a client submission, **when** the system analyzes it against framework requirements, **then** it identifies sections below the minimum completeness threshold and flags them as gaps.
2. **Given** a detected gap, **when** the facilitator views the section, **then** a "Request Follow-Up" button is visible with a system-drafted follow-up question displayed for preview.
3. **Given** the facilitator clicks "Request Follow-Up," **when** the follow-up preview loads, **then** the drafted question is written in the facilitator's voice (not system-speak) and the facilitator can edit it before approving.
4. **Given** the facilitator approves a follow-up, **when** they confirm, **then** the system sends the follow-up to the client via their preferred channel (email or SMS) with a magic link to re-enter the flow at the relevant section.
5. **Given** a client responds to a follow-up, **when** their response is received, **then** the dashboard updates the relevant section's content and completeness score in near real-time.
6. **Given** a follow-up has been sent, **when** the facilitator views the client's record, **then** the follow-up status is visible (drafted / approved / sent / answered).

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-014, WIE-011
**Technical Notes:** Gap detection uses LLM analysis against framework section requirements defined in Framework Template Store. Follow-up questions generated by LLM with facilitator voice/tone calibration prompts. Follow-Up records track full lifecycle. Client re-entry via magic link opens the flow at the specific section, not from the beginning.

---

#### WIE-016: Generate Framework-Ready Package

**As a** facilitator, **I want to** compile all client intake data into my existing prompt framework format with one click **so that** I can feed it directly into my narrative generation pipeline.

**Acceptance Criteria:**

1. **Given** the facilitator is viewing a client's framework view, **when** they click "Generate Narrative Package," **then** the system compiles all framework sections into a structured document matching the facilitator's prompt framework format.
2. **Given** a generated package, **when** the facilitator reviews it, **then** each prompt slot is filled with merged content from all sources, with source citations (e.g., "from client voice recording," "from web scrape," "from uploaded document, page 3").
3. **Given** a generated package, **when** the facilitator reviews it, **then** confidence levels are marked per section so the facilitator knows which sections are well-supported vs. thin.
4. **Given** the compiled package, **when** the facilitator finds an awkward AI merge, **then** they can edit the content inline before triggering narrative generation.
5. **Given** the package is finalized, **when** the facilitator approves it, **then** the system exports it in the format required by the existing AI narrative generation pipeline.

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-014
**Technical Notes:** Output format must match the facilitator's existing prompt framework exactly -- the system adapts to the facilitator's workflow, not the reverse. Export format TBD based on facilitator's current pipeline (likely structured JSON or formatted markdown). This is the bridge between the intake system and the narrative generation system.

---

### Phase 4: File Processing Pipeline + Voice Input

*Multi-modal capture capabilities. Clients can upload files (PDF, DOCX, PPTX), record voice answers, and paste links at any question. All inputs are processed asynchronously and mapped to framework sections.*

---

#### WIE-017: File Upload in Context

**As a** client, **I want to** upload files (PDFs, slide decks, documents) when a question prompts me for related materials **so that** the system can extract relevant content without me having to re-type it.

**Acceptance Criteria:**

1. **Given** any question in the interview flow, **when** the client views the input area, **then** a file upload option is visible alongside the text input (secondary, not primary).
2. **Given** the client uploads a file, **when** the upload initiates, **then** the system displays an immediate confirmation ("File uploading...") and allows the client to proceed to the next question without waiting for processing.
3. **Given** an uploaded PDF, DOCX, or PPTX file, **when** the system processes it, **then** it extracts text content, maps relevant sections to framework categories, and appends the extracted content to the appropriate framework sections.
4. **Given** an uploaded file that contains content relevant to multiple framework sections (e.g., a pitch deck), **when** the AI Content Processor analyzes it, **then** it distributes content across relevant sections with confidence scores rather than forcing a single-section tag.
5. **Given** a file that fails processing (corrupted, unsupported format, password-protected), **when** the failure is detected, **then** the system notifies the client that the file could not be processed and suggests re-uploading or providing the content as text.
6. **Given** an uploaded file, **when** the facilitator views the framework section it maps to, **then** source attribution shows the original filename and page reference (e.g., "from competitive-analysis.pdf, page 3").
7. **Given** a file upload, **when** the system stores it, **then** the file is saved to object storage (S3) organized by client/booking with a signed URL for secure, time-limited access.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-004, WIE-011
**Technical Notes:** File Processing Pipeline handles PDF (pdf.js + OCR for scans), DOCX, PPTX extraction. Processing runs async via BullMQ job queue. Processed File records track status. AI Content Processor does second-pass mapping for multi-section content per ADR-5. Max file size: 50MB. Supported formats: PDF, DOCX, PPTX, PNG, JPG.

---

#### WIE-018: Voice Recording Input

**As a** client, **I want to** record a voice answer for any question **so that** I can speak my thoughts naturally instead of typing, especially for narrative or story-driven questions.

**Acceptance Criteria:**

1. **Given** any question in the interview flow, **when** the client views the input area, **then** a microphone icon labeled "Record your answer" is visible as a secondary input option.
2. **Given** the client taps the microphone icon, **when** recording starts, **then** the system shows a live waveform or recording indicator with a "Done" button, and records audio using the Web Audio API without requiring any app download.
3. **Given** the client taps "Done," **when** the recording completes, **then** the system displays confirmation, uploads the recording, and queues it for transcription.
4. **Given** a completed voice recording, **when** the transcription completes (target: within 15 seconds for 90-second recordings), **then** the transcribed text is stored as a Submission tagged to the appropriate framework section.
5. **Given** the client records a voice answer, **when** they view the summary/review screen, **then** they can see the transcription of their recording and edit it if needed.
6. **Given** a client on mobile, **when** they tap the microphone icon, **then** recording works with a single tap without triggering intrusive browser permission dialogs beyond the initial microphone access request.
7. **Given** a transcription failure, **when** the failure is detected, **then** the system retains the audio file, notifies the client that transcription failed, and allows them to re-record or type their answer instead.

**Priority:** must_have
**Size:** L
**Dependencies:** WIE-004
**Technical Notes:** Web Audio API for in-browser recording. Audio uploaded to S3, transcribed via Whisper API, Deepgram, or AssemblyAI. Voice Recording records track duration, storage path, transcription status. Near-real-time transcription for in-flow recordings. The facilitator dashboard should flag voice-transcribed content distinctly -- verbal answers are often richer than typed ones, which is a signal for the facilitator.

---

#### WIE-019: Link Pasting and Extraction

**As a** client, **I want to** paste links to relevant content (blog posts, podcast episodes, articles) **so that** the system can extract useful information without me summarizing it manually.

**Acceptance Criteria:**

1. **Given** any question in the interview flow, **when** the client views the input area, **then** a link icon labeled "Paste a link" is visible as a secondary input option.
2. **Given** the client pastes a URL, **when** they submit it, **then** the system confirms receipt ("Link received -- we'll pull the content") and allows progression to the next question.
3. **Given** a pasted URL to a web page (blog post, article), **when** the system processes it, **then** it scrapes the page content, extracts relevant text, and maps it to the appropriate framework section(s).
4. **Given** a pasted URL to a podcast or audio content, **when** the system processes it, **then** it attempts to download and transcribe the audio, extracting relevant content for framework mapping.
5. **Given** a pasted URL that is unreachable (404, blocked, requires login), **when** the failure is detected, **then** the system marks the link as unprocessable and the facilitator sees it flagged on the dashboard.
6. **Given** link processing completes after the client has moved past that section, **when** the content is extracted, **then** the framework section's compiled content updates retroactively and the facilitator dashboard reflects the new content.

**Priority:** should_have
**Size:** M
**Dependencies:** WIE-004, WIE-010 (reuses scraping infrastructure)
**Technical Notes:** Link processing reuses Web Scraping Engine components for HTML pages. Podcast/audio extraction is a stretch goal -- may require specialized handling. Processing is async. Eventual consistency: compiled sections may update after client moves past them (per ADR-3 consequences). Submission.input_type = "link" with file_reference storing the URL.

---

### Phase 5: Notification Service + Smart Reminders

*Automated communications for the full lifecycle: welcome emails (already started in Phase 2), pause reminders, submission confirmations, facilitator alerts, and follow-up delivery.*

---

#### WIE-020: Smart Pause Reminders

**As a** client who paused mid-session, **I want to** receive a helpful, well-timed reminder with a link to resume **so that** I remember to complete the intake without feeling nagged.

**Acceptance Criteria:**

1. **Given** a client has paused the intake (no activity for 24+ hours), **when** the reminder window is reached, **then** the system sends a single reminder via the client's preferred channel (email or SMS).
2. **Given** a reminder is sent, **when** the client reads it, **then** the message includes: their completion percentage, estimated time remaining, a magic link to resume, and a one-line summary of what is already captured.
3. **Given** the system is evaluating reminder timing, **when** the workshop is more than 14 days away, **then** reminders are gentle and spaced at least 72 hours apart.
4. **Given** the system is evaluating reminder timing, **when** the workshop is within 7 days and intake is incomplete, **then** reminders become more direct (but never aggressive) and may be spaced at 24-hour intervals.
5. **Given** the system is evaluating reminder frequency, **when** a reminder has been sent within the last 24 hours, **then** no additional reminder is sent regardless of urgency.
6. **Given** a reminder message, **when** the client reads it, **then** the tone is "we saved your progress" -- never "you haven't finished" or "you're behind."
7. **Given** a client has submitted their intake, **when** the system evaluates reminders, **then** no further reminders are sent.

**Priority:** should_have
**Size:** M
**Dependencies:** WIE-005, WIE-001
**Technical Notes:** Notification Service checks session activity via Session Manager metadata. Reminder logic: check daily for sessions with status "in-progress" and last_activity_at > 24 hours. Reminder escalation tiers based on days until workshop. Max 1 reminder per 24 hours. All messages use facilitator's name and voice. Notification Log records track delivery. Email via SendGrid/Postmark, SMS via Twilio.

---

#### WIE-021: Submission Confirmation to Client

**As a** client who just submitted, **I want to** receive a confirmation message **so that** I know my materials were received and I understand what happens next.

**Acceptance Criteria:**

1. **Given** the client submits their intake, **when** the submission is processed, **then** the system sends a confirmation message via email (and optionally SMS).
2. **Given** the confirmation message, **when** the client reads it, **then** it thanks them by name, briefly summarizes what was captured (e.g., "6 sections completed, 2 files processed"), and describes the next step ("The facilitator will review your materials and may follow up with a quick question or two").
3. **Given** the confirmation message, **when** the client reads it, **then** the sender is the facilitator's name and email, not a system address.

**Priority:** should_have
**Size:** S
**Dependencies:** WIE-007
**Technical Notes:** Triggered by session status transition to "submitted." Template-based with dynamic content slots populated from submission metadata.

---

#### WIE-022: Facilitator Submission Alert

**As a** facilitator, **I want to** be notified immediately when a client submits their intake **so that** I can review it promptly and send follow-ups while the client is still engaged.

**Acceptance Criteria:**

1. **Given** a client submits their intake, **when** the submission is processed, **then** the facilitator receives a notification (email and/or dashboard push notification).
2. **Given** the facilitator notification, **when** they read it, **then** it includes: client name, company name, completeness score, number of file uploads processed, number of voice recordings, and a direct link to the client's dashboard view.
3. **Given** the facilitator clicks the dashboard link in the notification, **when** the page loads, **then** it opens directly to that client's per-client framework view (WIE-014).

**Priority:** should_have
**Size:** S
**Dependencies:** WIE-007, WIE-013
**Technical Notes:** Triggered alongside WIE-021 on submission. Facilitator notification includes richer metadata than client confirmation. Dashboard deep link includes client ID in the URL.

---

#### WIE-023: Client Notification Preferences

**As a** client, **I want to** choose whether I receive reminders via email, SMS, or both **so that** communications reach me through my preferred channel.

**Acceptance Criteria:**

1. **Given** the client starts the intake flow, **when** they complete the kickoff step, **then** the system asks for their notification preference (email only, SMS, or both) with email as the default.
2. **Given** the client selects SMS or both, **when** they confirm, **then** the system collects their phone number with clear messaging about how it will be used (reminders only, not marketing).
3. **Given** a stored notification preference, **when** the system sends any communication (reminders, follow-ups, confirmations), **then** it uses the client's selected channel(s).
4. **Given** a client who did not provide a phone number, **when** the system needs to send a notification, **then** it defaults to email without error.

**Priority:** could_have
**Size:** S
**Dependencies:** WIE-020
**Technical Notes:** Notification preference stored on Client record. Phone number is optional. SMS requires Twilio integration. Default to email-only if preference is not set.

---

#### WIE-024: Follow-Up Delivery via Notification Service

**As a** system, **I want to** deliver facilitator-approved follow-up questions to clients via their preferred channel **so that** gaps in framework coverage can be filled after initial submission.

**Acceptance Criteria:**

1. **Given** the facilitator approves a follow-up question (WIE-015), **when** the approval is confirmed, **then** the Notification Service sends the follow-up to the client via their preferred channel.
2. **Given** a follow-up message, **when** the client reads it, **then** it includes: a warm greeting, the specific question(s), and a magic link that opens the intake flow directly at the relevant framework section.
3. **Given** the client clicks the follow-up link, **when** the intake reopens, **then** they see only the relevant section(s) with the follow-up question(s) -- not the entire interview from the beginning.
4. **Given** the client answers the follow-up, **when** they submit, **then** the dashboard updates the relevant section and the Follow-Up record transitions to "answered."
5. **Given** a follow-up has been sent but not answered for 48+ hours, **when** the system evaluates, **then** it sends a single reminder (subject to the 24-hour frequency cap from WIE-020).

**Priority:** should_have
**Size:** M
**Dependencies:** WIE-015, WIE-020
**Technical Notes:** Follow-up delivery is a specialized notification type. Magic link for follow-up includes a section parameter that tells the Intake Flow Engine which section to open. Follow-Up record status lifecycle: drafted -> approved -> sent -> answered. Reminder for unanswered follow-ups shares the frequency cap with pause reminders.

---

### Additional Stories (Cross-Phase)

---

#### WIE-025: Mobile-Responsive Intake Experience

**As a** client, **I want** the entire intake experience to work seamlessly on my phone **so that** I can complete the interview on whatever device is convenient.

**Acceptance Criteria:**

1. **Given** a client accesses the intake on a mobile device (phone or tablet), **when** the page loads, **then** all elements (video, questions, input fields, buttons, progress bar) render correctly and are usable without horizontal scrolling.
2. **Given** a client is answering questions on mobile, **when** they interact with text fields, **then** the keyboard does not obscure the input area or the submit button.
3. **Given** a client switches from desktop to mobile mid-session, **when** they resume via the magic link, **then** the mobile layout loads with their session state intact.
4. **Given** a client on mobile, **when** they use voice recording, **then** the microphone button and recording interface are thumb-accessible and work with a single tap.

**Priority:** must_have
**Size:** M
**Dependencies:** WIE-002, WIE-004
**Technical Notes:** Next.js with responsive design. PWA capabilities for mobile experience without app store. Test on iOS Safari and Android Chrome as primary mobile browsers. Voice recording via Web Audio API must handle mobile browser permission flows gracefully.

---

#### WIE-026: Question Tree Validation Tool

**As a** developer (or facilitator with technical support), **I want** a validation tool that checks the question tree configuration for errors **so that** we can deploy changes confidently.

**Acceptance Criteria:**

1. **Given** a question tree JSON configuration, **when** the validator runs, **then** it verifies that all nodes are reachable from the entry point for each segmentation path.
2. **Given** a question tree, **when** the validator runs, **then** it verifies that all terminal nodes (end of each path) produce sufficient coverage across all framework sections.
3. **Given** a question tree, **when** the validator runs, **then** it reports any orphan nodes (nodes with no incoming edges) and any dead-end nodes (non-terminal nodes with no outgoing edges).
4. **Given** a valid question tree, **when** the validator completes, **then** it outputs a summary: total nodes, paths per segmentation type, framework section coverage per path, and estimated completion time per path.

**Priority:** should_have
**Size:** M
**Dependencies:** WIE-008
**Technical Notes:** Can be a CLI tool initially; admin UI can come later. Runs against the Framework Template Store. Critical for safe iteration on the question tree. Graph traversal algorithm that walks all 4 segmentation paths exhaustively.

---

#### WIE-027: Booking Webhook Integration

**As a** system, **I want to** receive booking confirmations from the existing booking/CRM system via webhook **so that** the intake process starts automatically when a workshop is booked.

**Acceptance Criteria:**

1. **Given** a new workshop booking in the existing booking/CRM system, **when** the booking is confirmed, **then** the system receives a webhook payload containing: client name, email, company name, company URL, and workshop date.
2. **Given** a valid webhook payload, **when** the system processes it, **then** it creates the Client, Workshop Booking, and Intake Session records and triggers the web scraping job (if Phase 2 is deployed).
3. **Given** an invalid or malformed webhook payload, **when** the system receives it, **then** it logs the error and does not create partial records.
4. **Given** a duplicate webhook (same booking sent twice), **when** the system processes it, **then** it idempotently ignores the second request without creating duplicate records.

**Priority:** should_have
**Size:** M
**Dependencies:** WIE-009
**Technical Notes:** Webhook endpoint with signature verification for security. Idempotency key based on booking ID or client email + workshop date. Initially can be supplemented with a manual "Create Intake" form in the facilitator dashboard for bookings not routed through the CRM.

---

## Technical Spikes

High-uncertainty items requiring time-boxed investigation before estimation or implementation.

---

### SPIKE-001: Question Tree Authoring and Complexity

**Objective:** Determine the optimal format and tooling for the facilitator to define and iterate on the branching question tree.

**Questions to Answer:**
1. How complex are the real branching paths when mapped against the facilitator's actual framework? (Are 4 paths x 20-40 questions realistic, or is the real tree deeper/shallower?)
2. Can the facilitator articulate their tacit question logic into a JSON-compatible format, or do they need a visual editor?
3. What is the right abstraction level -- full graph with conditional edges, or a simpler decision-tree structure?

**Time Box:** 3 days
**Output:** A documented question tree for one framework section across all 4 segmentation paths, a recommendation on authoring tooling (JSON editor vs. visual builder vs. spreadsheet-to-JSON converter), and a revised size estimate for WIE-008.

---

### SPIKE-002: Web Scraping Reliability and Quality

**Objective:** Test the two-tier scraping approach against real client websites to calibrate pre-population quality expectations.

**Questions to Answer:**
1. What percentage of past client websites return useful content via static scraping (Tier 1)?
2. How often is Tier 2 (headless browser) required?
3. What is the average scrape-to-draft quality for each framework section? (Is 70% accuracy realistic per the storyboard?)
4. How do LinkedIn company pages and social profiles perform as sources?

**Time Box:** 3 days
**Output:** Scrape results for 10-15 past client websites, quality assessment per framework section, a recommendation on scraping strategy adjustments, and confidence thresholds for pre-population vs. question-mode fallback.

---

### SPIKE-003: LLM Framework Mapping Accuracy

**Objective:** Validate that LLM-based content classification can reliably map unstructured inputs (uploaded files, voice transcriptions) to the correct framework sections.

**Questions to Answer:**
1. Given a sample pitch deck PDF, how accurately does the LLM distribute content across framework sections?
2. Given a voice transcription of a founder telling their story, how well does the LLM identify content relevant to brand story vs. market context vs. customer profile?
3. What prompt engineering is needed to match the facilitator's categorization instincts?
4. What is the cost per intake session at current LLM pricing?

**Time Box:** 3 days
**Output:** Accuracy metrics across 5-10 sample documents, recommended prompts for framework mapping, cost model per client intake, and confidence score thresholds for human review flagging.

---

### SPIKE-004: Voice Recording Browser Compatibility

**Objective:** Validate Web Audio API recording works reliably across target browsers and devices without app downloads or complex permission flows.

**Questions to Answer:**
1. Does in-browser voice recording work on iOS Safari, Android Chrome, desktop Chrome, desktop Safari, and desktop Firefox?
2. What is the user experience for microphone permissions on each platform?
3. What audio format and quality settings produce reliable transcription results?
4. What is the maximum recording duration before performance issues arise?

**Time Box:** 2 days
**Output:** Browser compatibility matrix, recommended audio settings, UX flow for permission handling per platform, and any fallback strategies needed.

---

## Epic-Level Definition of Done

The Workshop Intake Engine epic is complete when ALL of the following conditions are met:

### Functional Completeness
- [ ] A client can access the intake via magic link, watch the video kickoff, self-segment, complete branching questions, review pre-populated content, upload files, record voice answers, paste links, pause and resume across devices, review a summary, and submit -- all without creating an account.
- [ ] The facilitator can view all clients in a pipeline, drill into framework-organized per-client views, see gap detection with source attribution, approve and send follow-up questions, edit compiled content, and generate a framework-ready output package.
- [ ] The system scrapes client websites on booking, generates AI pre-populated content, processes uploaded files and voice recordings, sends smart reminders, and delivers follow-up questions -- all asynchronously without blocking the client flow.

### Quality Gates
- [ ] All must_have stories pass acceptance criteria with automated tests covering happy path and documented unhappy paths.
- [ ] Intake flow completion rate exceeds 85% in pilot testing with 5+ real clients.
- [ ] Facilitator review time per client is under 30 minutes in pilot testing.
- [ ] Framework-ready output matches the facilitator's existing prompt format with no structural translation required.
- [ ] Mobile experience is fully functional on iOS Safari and Android Chrome.
- [ ] Auto-save has zero data loss incidents across all pilot sessions.

### Performance
- [ ] Web scraping completes within 5 minutes of booking for 90% of client websites.
- [ ] AI pre-population completes within 2 minutes of scrape completion.
- [ ] Page load time for intake flow is under 2 seconds on 4G mobile connections.
- [ ] Voice transcription completes within 15 seconds for recordings under 2 minutes.
- [ ] File processing (PDF text extraction) completes within 30 seconds for files under 10MB.

### Security and Data
- [ ] Magic link tokens are cryptographically secure (UUID v4 or better) and expire 30 days after workshop date.
- [ ] All file uploads are stored with signed URLs (time-limited access).
- [ ] Client data is isolated by booking -- no cross-client data leakage.
- [ ] No client PII is logged in application logs.

### Facilitator Validation
- [ ] The facilitator has reviewed the framework-ready output for at least 3 pilot clients and confirmed it matches or exceeds the quality of their manual process.
- [ ] The facilitator has successfully used the dashboard to identify gaps, send follow-ups, and generate narrative packages without developer assistance.
- [ ] The facilitator confirms the question tree reflects their intake expertise and the system does not introduce unwanted structural changes to their framework.

---

## Story Dependency Map

```
Phase 1 (Core Flow)
  WIE-008 (Framework Template Store) ─────────────────────────────────┐
  WIE-001 (Magic Link Access) ──┬── WIE-002 (Video Kickoff) ─┐       │
                                ├── WIE-005 (Auto-Save)       │       │
                                │                              ├── WIE-003 (Self-Segmentation)
                                │                              │       │
                                │                              │   WIE-004 (Branching Questions)
                                │                              │       │
                                │                         WIE-006 (Text Input)
                                │                              │
                                │                         WIE-007 (Review & Submit)
                                │                              │
  WIE-009 (Data Model & API) ──┘                              │
       │                                                       │
Phase 2 (Head Start)                                          │
  WIE-010 (Web Scraping) ──── WIE-011 (AI Pre-Pop) ──────────┘
       │                           │
  WIE-012 (Welcome Email) ────────┘

Phase 3 (Dashboard)
  WIE-013 (Pipeline View) ── WIE-014 (Framework View) ── WIE-015 (Gap Detection)
                                      │
                                 WIE-016 (Generate Package)

Phase 4 (Multi-Modal)
  WIE-017 (File Upload) ── depends on WIE-004, WIE-011
  WIE-018 (Voice Recording) ── depends on WIE-004
  WIE-019 (Link Pasting) ── depends on WIE-004, WIE-010

Phase 5 (Notifications)
  WIE-020 (Smart Reminders) ── depends on WIE-005, WIE-001
  WIE-021 (Submission Confirm) ── depends on WIE-007
  WIE-022 (Facilitator Alert) ── depends on WIE-007, WIE-013
  WIE-023 (Notification Prefs) ── depends on WIE-020
  WIE-024 (Follow-Up Delivery) ── depends on WIE-015, WIE-020

Cross-Phase
  WIE-025 (Mobile Responsive) ── depends on WIE-002, WIE-004
  WIE-026 (Tree Validator) ── depends on WIE-008
  WIE-027 (Booking Webhook) ── depends on WIE-009
```

---

## Story Summary

| ID | Title | Phase | Priority | Size |
|----|-------|-------|----------|------|
| WIE-001 | Magic Link Access | 1 | must_have | M |
| WIE-002 | Facilitator Video Kickoff | 1 | must_have | S |
| WIE-003 | Self-Segmentation | 1 | must_have | M |
| WIE-004 | Branching Question Flow | 1 | must_have | L |
| WIE-005 | Auto-Save and Session Persistence | 1 | must_have | M |
| WIE-006 | Text Input for Questions | 1 | must_have | S |
| WIE-007 | Review and Submit | 1 | must_have | M |
| WIE-008 | Framework Template Store | 1 | must_have | L |
| WIE-009 | Intake Session Data Model and API | 1 | must_have | L |
| WIE-010 | Web Scraping Engine | 2 | must_have | XL |
| WIE-011 | AI Content Pre-Population | 2 | must_have | L |
| WIE-012 | Welcome Email with Head Start Tease | 2 | must_have | S |
| WIE-013 | Client Pipeline View | 3 | must_have | M |
| WIE-014 | Per-Client Framework View | 3 | must_have | L |
| WIE-015 | Gap Detection and Follow-Up | 3 | must_have | L |
| WIE-016 | Generate Framework-Ready Package | 3 | must_have | M |
| WIE-017 | File Upload in Context | 4 | must_have | L |
| WIE-018 | Voice Recording Input | 4 | must_have | L |
| WIE-019 | Link Pasting and Extraction | 4 | should_have | M |
| WIE-020 | Smart Pause Reminders | 5 | should_have | M |
| WIE-021 | Submission Confirmation to Client | 5 | should_have | S |
| WIE-022 | Facilitator Submission Alert | 5 | should_have | S |
| WIE-023 | Client Notification Preferences | 5 | could_have | S |
| WIE-024 | Follow-Up Delivery via Notification Service | 5 | should_have | M |
| WIE-025 | Mobile-Responsive Intake Experience | Cross | must_have | M |
| WIE-026 | Question Tree Validation Tool | Cross | should_have | M |
| WIE-027 | Booking Webhook Integration | Cross | should_have | M |

**Totals:** 27 stories | 16 must_have | 9 should_have | 1 could_have | 1 not_sized (spike work separate)
