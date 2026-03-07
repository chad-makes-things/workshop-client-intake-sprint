# Product Architecture -- Workshop Intake System

## System Name

**Workshop Intake Engine** -- An AI-powered guided intake system that replaces fragmented, manual pre-workshop material collection with a conversational flow that web-scrapes first, guides second, and delivers framework-ready output to the facilitator.

## Problem Statement

Pre-workshop client material collection is manual, fragmented, and unscalable. Materials arrive via text, email, Dropbox, and file shares in the form of PDFs, anecdotes, website snapshots, and conversations. The facilitator manually collects, organizes, and maps these inputs into a structured prompt framework that generates a market narrative -- the foundation of the first workshop session. This process takes 3-5 hours per client and scales linearly with client volume, creating a revenue ceiling.

## User Types

### Client
The business representative (founder, marketing lead, or operations person) signing up for the workshop. Varying levels of digital sophistication -- from polished brand decks to "most of what we know is in our heads." The client completes the guided intake flow, reviews pre-populated content, answers questions, uploads files, and submits.

### Facilitator
The workshop business owner. Currently the bottleneck for all intake logistics. In the new system, the facilitator designs the question tree, records the kickoff video, reviews structured submissions on a dashboard, approves follow-ups, and triggers narrative generation. The facilitator's intellectual architecture (the prompt framework) is embedded in the system; the manual labor is eliminated.

---

## Core Capabilities

1. **Scrape and analyze** a client's public web presence to pre-populate framework sections
2. **Guide clients** through a branching conversational intake flow adapted to their sophistication level
3. **Accept multi-modal input** -- text, voice recordings, file uploads, and pasted links -- within the flow
4. **Process and extract** structured content from uploaded files, voice recordings, and linked resources
5. **Map all inputs** to the facilitator's prompt framework categories automatically
6. **Detect gaps** in framework coverage and generate targeted follow-up questions
7. **Persist and resume** client sessions across devices and time without data loss
8. **Present structured output** to the facilitator in a dashboard organized by framework section with source attribution

---

## Components

### 1. Web Scraping Engine

- **Responsibility:** Crawl a client's public web presence and extract structured content for framework pre-population.
- **Interfaces:**
  - **In:** Company URL, company name (from booking trigger)
  - **Out:** Structured content package (extracted text, images, metadata) mapped to framework categories; scrape status and confidence scores
- **Dependencies:** AI Content Processor (for content analysis and framework mapping)
- **Key Behaviors:**
  - Crawls homepage, about page, product/service pages, blog, team page
  - Pulls LinkedIn company page, social media bios, review sites (G2, Glassdoor), press mentions
  - Handles sites behind basic login walls gracefully (skip and flag)
  - Extracts clean text, strips navigation/boilerplate
  - Runs asynchronously on booking confirmation; completes before welcome email sends
  - Respects robots.txt and rate limits
- **Storyboard Steps Served:** Step 1 (background scraping before email), Step 4 (pre-populated review content)

### 2. Intake Flow Engine

- **Responsibility:** Manage the client-facing guided interview experience -- question sequencing, branching logic, progress tracking, and input collection.
- **Interfaces:**
  - **In:** Client session token, segmentation choice, answer submissions (text/voice/file/link), pre-populated content from Web Scraping Engine
  - **Out:** Collected answers tagged to framework slots, progress state, completion status
- **Dependencies:** Web Scraping Engine (pre-populated content), AI Content Processor (for adaptive follow-ups), Session Manager (state persistence), File Processing Pipeline (uploaded files)
- **Key Behaviors:**
  - Renders one question at a time, Typeform-style progressive disclosure
  - Branches question paths based on self-segmentation and prior answers
  - Switches between "review mode" (pre-populated content for confirm/edit/rewrite) and "question mode" (guided questions for gaps)
  - Accepts text, voice recording, file upload, and link paste for every question
  - Shows progress bar and completion estimates
  - Generates summary review screen at completion with section-level completeness indicators
  - Never blocks submission due to incomplete sections
- **Storyboard Steps Served:** Steps 2-6 (video kickoff through flexible input), Step 8 (review and submit)

### 3. AI Content Processor

- **Responsibility:** Analyze, categorize, and synthesize content from all sources (web scrape, client answers, files, voice, links) against the facilitator's prompt framework.
- **Interfaces:**
  - **In:** Raw content (text, transcriptions, extracted file content, scraped web content) with source metadata
  - **Out:** Framework-mapped content sections, confidence scores per section, gap analysis, draft pre-populated content, follow-up question suggestions
- **Dependencies:** Framework Template Store (prompt framework structure and slot definitions)
- **Key Behaviors:**
  - Maps incoming content to framework categories: market context, customer profile, competitive landscape, brand positioning, company story, aspirations
  - Generates pre-populated drafts from web-scraped content for client review
  - Merges content from multiple sources (web scrape + client edits + uploads + voice) into coherent section drafts
  - Detects gaps -- framework sections with insufficient or missing coverage
  - Generates targeted follow-up questions for gap sections in the facilitator's voice
  - Tracks content provenance (AI-drafted vs. client-authored vs. file-extracted vs. voice-transcribed)
  - Logs deltas between AI drafts and client edits as high-value signals
- **Storyboard Steps Served:** Steps 1 (AI draft generation), 4 (pre-populated content), 5 (adaptive follow-ups), 9 (gap detection), 10 (framework compilation)

### 4. File Processing Pipeline

- **Responsibility:** Ingest, process, and extract structured content from uploaded files and pasted links.
- **Interfaces:**
  - **In:** Uploaded files (PDF, DOCX, PPTX, images, audio/video), pasted URLs
  - **Out:** Extracted text content, transcriptions, parsed data, processing status
- **Dependencies:** AI Content Processor (for categorization after extraction)
- **Key Behaviors:**
  - Extracts text from PDFs (OCR for scanned documents)
  - Parses DOCX and PPTX files for text and structure
  - Transcribes audio recordings (voice memos, podcast links) using speech-to-text
  - Scrapes pasted URLs (blog posts, articles, podcast pages) for content
  - Processes uploaded images with OCR where relevant
  - Handles processing asynchronously -- confirms receipt to client immediately, processes in background
  - Returns structured content with source attribution (e.g., "from uploaded competitive matrix, page 3")
- **Storyboard Steps Served:** Steps 5 (file uploads in context), 6 (voice recording, link pasting)

### 5. Session Manager

- **Responsibility:** Persist client session state across devices, browser sessions, and time intervals.
- **Interfaces:**
  - **In:** Client identity token, session state updates (every answer/edit/upload)
  - **Out:** Restored session state, session metadata (last activity, completion %, device info)
- **Dependencies:** None (foundational service)
- **Key Behaviors:**
  - Auto-saves after every client interaction -- no manual save required
  - Generates magic-link URLs for session resumption without login/password
  - Restores exact position in the interview flow on return, including mid-section state
  - Tracks cross-device usage (laptop to phone) seamlessly
  - Provides session metadata to the Notification Service for reminder timing
  - Enforces session security via time-limited tokens
- **Storyboard Steps Served:** Step 7 (pause and return), Step 8 (submission)

### 6. Notification Service

- **Responsibility:** Send timely, contextual communications to clients and the facilitator.
- **Interfaces:**
  - **In:** Trigger events (scrape complete, session paused, submission received, follow-up needed), message templates, recipient preferences
  - **Out:** Sent messages (email, SMS), delivery confirmations
- **Dependencies:** Session Manager (session state for reminder timing), AI Content Processor (follow-up question content)
- **Key Behaviors:**
  - Sends welcome email with intake link after web scrape completes
  - Sends pause reminders calibrated to workshop date (gentle when distant, direct when close)
  - Limits reminder frequency (max one per 24 hours) to avoid spamming
  - Sends submission confirmation to client
  - Sends structured notification to facilitator on submission (completeness score, file count, review-ready link)
  - Sends facilitator-approved follow-up questions to client
  - Adapts channel (email vs. SMS) based on client preference
  - All client-facing messages use the facilitator's name and voice, not system-speak
- **Storyboard Steps Served:** Steps 1 (welcome email), 7 (smart reminders), 9 (facilitator notification, follow-up delivery)

### 7. Facilitator Dashboard

- **Responsibility:** Present all client intake data to the facilitator in a structured, framework-organized view with actionable controls.
- **Interfaces:**
  - **In:** Compiled framework data per client, completeness metrics, content provenance metadata, gap analysis
  - **Out:** Follow-up approvals, manual edits, narrative generation triggers
- **Dependencies:** AI Content Processor (framework-mapped content, gap detection), Session Manager (client progress data), Notification Service (follow-up delivery)
- **Key Behaviors:**
  - Displays all active clients in a pipeline view with status indicators
  - Per-client view organized by framework section (tabs or cards), not submission chronology
  - Shows content type breakdown per section (AI-drafted, client-edited, client-authored, file-extracted, voice-transcribed)
  - Highlights gaps with one-click follow-up approval (system drafts the message, facilitator approves)
  - Allows facilitator to edit compiled content before narrative generation
  - Displays completeness score and confidence levels per section
  - Provides source attribution for every piece of content
  - Triggers "Generate Narrative Package" to compile framework-ready prompt inputs
- **Storyboard Steps Served:** Steps 9 (facilitator receives structured output), 10 (framework-ready package)

### 8. Framework Template Store

- **Responsibility:** Maintain the facilitator's prompt framework structure -- the categories, slots, required depth, and mapping rules that define what the intake system collects.
- **Interfaces:**
  - **In:** Framework definition updates from the facilitator
  - **Out:** Framework structure (categories, slots, required fields, depth thresholds), question tree configuration, mapping rules
- **Dependencies:** None (foundational reference)
- **Key Behaviors:**
  - Stores the canonical prompt framework: market context, customer profile, competitive landscape, brand positioning, company story, aspirations (and any sub-categories)
  - Defines minimum completeness thresholds per section
  - Provides the question tree structure and branching logic for the Intake Flow Engine
  - Enables the facilitator to update framework structure without code changes
  - Versioned -- changes to the framework do not break in-progress intakes
- **Storyboard Steps Served:** All steps (the framework is the structural backbone of the entire system)

---

## Data Model

### 1. Client
- `id` (UUID)
- `name` (string)
- `email` (string)
- `phone` (string, optional)
- `company_name` (string)
- `company_url` (string)
- `segmentation_level` (enum: documents-ready, scattered-outdated, in-our-heads, early-stage)
- `notification_preference` (enum: email, sms, both)
- `created_at` (timestamp)
- **Relationships:** has one Workshop Booking, has one Intake Session, has many Submissions

### 2. Workshop Booking
- `id` (UUID)
- `client_id` (FK -> Client)
- `workshop_date` (date)
- `booking_date` (timestamp)
- `status` (enum: booked, intake-in-progress, intake-complete, narrative-ready, workshop-complete)
- **Relationships:** belongs to Client, has one Scrape Job, has one Intake Session

### 3. Scrape Job
- `id` (UUID)
- `booking_id` (FK -> Workshop Booking)
- `target_url` (string)
- `status` (enum: queued, crawling, processing, complete, failed)
- `pages_crawled` (integer)
- `started_at` (timestamp)
- `completed_at` (timestamp)
- **Relationships:** belongs to Workshop Booking, has many Scraped Pages, produces many Pre-Populated Sections

### 4. Scraped Page
- `id` (UUID)
- `scrape_job_id` (FK -> Scrape Job)
- `url` (string)
- `page_type` (enum: homepage, about, services, blog, team, linkedin, social, review-site, press)
- `raw_content` (text)
- `extracted_content` (text)
- `scraped_at` (timestamp)
- **Relationships:** belongs to Scrape Job

### 5. Intake Session
- `id` (UUID)
- `client_id` (FK -> Client)
- `booking_id` (FK -> Workshop Booking)
- `magic_link_token` (string, unique)
- `current_step` (integer)
- `current_section` (string)
- `completion_percentage` (decimal)
- `segmentation_choice` (string)
- `started_at` (timestamp)
- `last_activity_at` (timestamp)
- `submitted_at` (timestamp, nullable)
- `status` (enum: not-started, in-progress, paused, submitted)
- **Relationships:** belongs to Client, belongs to Workshop Booking, has many Submissions

### 6. Submission
- `id` (UUID)
- `session_id` (FK -> Intake Session)
- `framework_section` (string -- e.g., "competitive-landscape", "brand-story")
- `input_type` (enum: text, voice, file, link, pre-populated-confirmed, pre-populated-edited, pre-populated-rewritten)
- `content` (text -- raw text, transcription, or extracted content)
- `original_pre_populated_content` (text, nullable -- for delta tracking)
- `file_reference` (string, nullable -- path/URL to stored file)
- `confidence_score` (decimal, nullable)
- `created_at` (timestamp)
- **Relationships:** belongs to Intake Session, belongs to Framework Section

### 7. Framework Section
- `id` (UUID)
- `template_version` (integer)
- `section_key` (string -- e.g., "market-context", "customer-profile")
- `section_name` (string -- display name)
- `description` (text)
- `minimum_completeness_threshold` (decimal)
- `display_order` (integer)
- `question_tree` (JSON -- branching logic, questions, follow-ups per segmentation path)
- **Relationships:** has many Submissions, has many Compiled Sections

### 8. Processed File
- `id` (UUID)
- `submission_id` (FK -> Submission)
- `original_filename` (string)
- `file_type` (enum: pdf, docx, pptx, image, audio, video)
- `file_size` (integer)
- `storage_path` (string)
- `extracted_text` (text)
- `processing_status` (enum: queued, processing, complete, failed)
- `processed_at` (timestamp)
- **Relationships:** belongs to Submission

### 9. Voice Recording
- `id` (UUID)
- `submission_id` (FK -> Submission)
- `duration_seconds` (integer)
- `storage_path` (string)
- `transcription` (text)
- `transcription_status` (enum: queued, processing, complete, failed)
- `recorded_at` (timestamp)
- **Relationships:** belongs to Submission

### 10. Compiled Section
- `id` (UUID)
- `booking_id` (FK -> Workshop Booking)
- `framework_section_id` (FK -> Framework Section)
- `merged_content` (text -- unified content from all sources)
- `source_breakdown` (JSON -- which content came from which source type)
- `completeness_score` (decimal)
- `gap_detected` (boolean)
- `facilitator_edited` (boolean)
- `compiled_at` (timestamp)
- **Relationships:** belongs to Workshop Booking, belongs to Framework Section

### 11. Follow-Up
- `id` (UUID)
- `booking_id` (FK -> Workshop Booking)
- `framework_section` (string)
- `question_text` (text)
- `status` (enum: drafted, approved, sent, answered)
- `channel` (enum: email, sms)
- `response_content` (text, nullable)
- `created_at` (timestamp)
- `sent_at` (timestamp, nullable)
- `answered_at` (timestamp, nullable)
- **Relationships:** belongs to Workshop Booking

### 12. Notification Log
- `id` (UUID)
- `recipient_id` (FK -> Client or Facilitator)
- `recipient_type` (enum: client, facilitator)
- `notification_type` (enum: welcome-email, pause-reminder, submission-confirmation, facilitator-alert, follow-up-delivery)
- `channel` (enum: email, sms)
- `content_preview` (text)
- `sent_at` (timestamp)
- `delivered` (boolean)
- **Relationships:** belongs to Client or Facilitator

---

## Integration Points

### Web Scraping
- **Headless browser** (Puppeteer/Playwright) for JavaScript-rendered pages
- **HTTP client** for static pages and API endpoints
- **LinkedIn API** (or scraping with rate limiting) for company profiles
- Respect robots.txt; implement retry and backoff logic

### AI / LLM Models
- **Content analysis and framework mapping** -- LLM (GPT-4 / Claude) for reading scraped content, client answers, and extracted file text, then mapping to framework categories
- **Draft generation** -- LLM for producing pre-populated section drafts from web-scraped content
- **Gap detection** -- LLM for analyzing compiled sections against framework requirements and identifying missing areas
- **Follow-up question generation** -- LLM for drafting targeted, voice-matched follow-up questions
- **Content merging** -- LLM for synthesizing multi-source content into coherent section drafts

### Speech-to-Text
- **Transcription service** (Whisper API, Deepgram, or AssemblyAI) for voice recordings and audio file processing
- Near-real-time transcription for in-flow voice recordings; batch for uploaded audio files

### Email
- **Transactional email** (SendGrid, Postmark, or AWS SES) for welcome emails, reminders, follow-ups, and facilitator notifications
- Emails send from the facilitator's name and reply-to address
- Template system supporting dynamic content and voice-matched copy

### SMS
- **SMS gateway** (Twilio) for pause reminders and follow-up delivery when client prefers text
- Magic-link URLs in SMS for session resumption

### File Storage
- **Object storage** (AWS S3 or equivalent) for uploaded files, voice recordings, and scraped content
- Signed URLs for secure, time-limited access
- Organized by client/booking for easy retrieval and cleanup

### File Processing
- **PDF extraction** -- pdf.js, Apache Tika, or cloud document AI for text extraction and OCR
- **Document parsing** -- libraries for DOCX/PPTX text and structure extraction
- **Image OCR** -- Cloud Vision API or Tesseract for text in images

### Booking System
- **Webhook integration** with existing booking/CRM system to trigger scrape jobs on new workshop bookings
- Client name, email, company URL passed via webhook payload

---

## Technology Approach Recommendation

### Frontend
- **Next.js** (React) single-page application for the client-facing intake flow and facilitator dashboard
- Server-side rendering for the initial page load (SEO irrelevant, but performance matters for first impression)
- Progressive Web App capabilities for mobile experience without app store friction
- Web Audio API for in-browser voice recording

### Backend
- **Node.js / TypeScript** API layer (Express or Fastify) for session management, flow logic, and dashboard endpoints
- **Background job queue** (BullMQ with Redis) for async processing: web scraping, file processing, transcription, AI analysis
- **PostgreSQL** as the primary database for all structured data
- **Redis** for session caching, job queues, and real-time progress tracking

### AI Layer
- **OpenAI API or Anthropic API** for content analysis, draft generation, gap detection, and follow-up generation
- Prompt engineering with framework-specific system prompts maintained in the Framework Template Store
- Structured output (JSON mode) for reliable framework mapping

### Infrastructure
- **Vercel** or **AWS** (ECS/Fargate) for application hosting
- **AWS S3** for file and media storage
- **Cloudflare** or **AWS CloudFront** for CDN and edge caching
- **Sentry** for error monitoring; **Datadog** or **CloudWatch** for observability

### Rationale
This stack prioritizes developer velocity (TypeScript end-to-end, React ecosystem), proven reliability at small-to-medium scale, and strong AI integration patterns. The business is a boutique workshop operation scaling from 3 to 10+ concurrent clients -- not a high-traffic consumer app. The architecture should be simple enough for a small team to maintain and extend, with clear separation between the intake flow, processing pipeline, and facilitator dashboard.

---

## Architecture Decision Records

### ADR-1: Branching Logic Architecture

**Context:** The intake flow must adapt based on client self-segmentation (4 paths) and prior answers within each path. Questions, depth, and input prompts change based on what the client selects. This branching logic is the intellectual core of the system.

**Decision:** Store the question tree as a JSON configuration in the Framework Template Store, not as hardcoded application logic. The tree is a directed graph where each node is a question/prompt, edges are conditional transitions based on answer values, and nodes reference framework section keys for mapping. The Intake Flow Engine traverses this graph at runtime.

**Alternatives Considered:**
- **Hardcoded branching in application code** -- rejected because the facilitator must be able to adjust question sequences without code deployments. The question tree is a content/strategy artifact, not an engineering artifact.
- **Third-party form builder (Typeform/Tally)** -- rejected because the pre-populated review mode (confirm/edit/rewrite of AI-drafted content) is not supported by any commercial form tool. The hybrid of review-mode and question-mode within a single flow requires custom rendering.
- **Rule engine (Drools, etc.)** -- rejected as over-engineered for the complexity level. A JSON graph with a lightweight traversal engine is sufficient for 4 segmentation paths with 20-40 questions each.

**Consequences:** The facilitator can modify the question tree through a configuration interface (or with developer support editing JSON). New framework sections can be added without application changes. The tradeoff is that complex branching logic in JSON requires careful testing -- a dedicated "question tree validator" should verify all paths are reachable and all terminal nodes produce complete framework coverage.

### ADR-2: Web Scraping Strategy

**Context:** The system must scrape client websites before the intake begins to pre-populate framework sections. Websites vary enormously: static HTML, JavaScript-rendered SPAs, sites behind login walls, minimal single-page sites, and complex multi-hundred-page corporate sites.

**Decision:** Use a two-tier scraping approach. Tier 1 (fast): HTTP-based scraping with cheerio/jsdom for static HTML pages -- covers the majority of small-to-mid-size business sites. Tier 2 (fallback): headless browser (Playwright) for JavaScript-rendered sites that return empty content in Tier 1. Limit crawl depth to 2 levels from homepage, with priority pages (about, services/products, team, blog index) crawled first. Cap total pages at 25 per site. LinkedIn and social profiles are scraped via separate, rate-limited workers.

**Alternatives Considered:**
- **Full recursive crawl** -- rejected because large corporate sites could produce thousands of pages, most irrelevant. The intake needs depth on key pages, not breadth across all pages.
- **Third-party scraping API (Diffbot, ScrapingBee)** -- viable but adds cost and vendor dependency for a core capability. Kept as a fallback option if in-house scraping proves unreliable.
- **Client-submitted URL list** -- rejected because it adds friction to the client and defeats the "we already did our homework" value proposition.

**Consequences:** The scrape must complete before the welcome email sends (target: under 5 minutes). Pages behind login walls are skipped and flagged. Thin-web-presence clients trigger an adjusted email copy ("We've pulled together some initial thoughts") rather than the full "we did our homework" messaging. The system must handle scrape failures gracefully -- a failed scrape should not block the intake flow; it simply means fewer pre-populated sections.

### ADR-3: Multi-Modal Input Handling

**Context:** Clients must be able to provide input via text, voice recording, file upload, and link pasting for any question in the flow. Each input type requires different processing and produces different content shapes.

**Decision:** Every question node in the intake flow supports all four input types simultaneously. The UI presents a text field as the default with voice, file, and link as secondary options. All non-text inputs are processed asynchronously: voice recordings are transcribed via speech-to-text API, files are processed through the File Processing Pipeline, and links are scraped. The client receives immediate confirmation ("Recording received" / "File uploading...") and can proceed to the next question without waiting for processing to complete. Processed content is merged into the framework section retroactively.

**Alternatives Considered:**
- **Input type per question** (some questions text-only, some voice-only) -- rejected because it removes client agency. A founder who prefers speaking should be able to voice-record any answer, not just designated ones.
- **Synchronous processing** (wait for transcription/extraction before advancing) -- rejected because processing latency would stall the conversational flow. A 90-second voice recording takes 10-15 seconds to transcribe; a PDF takes 5-30 seconds to extract. Blocking the client during this time breaks the rhythm.
- **Post-submission upload** (complete questions first, upload files after) -- rejected because contextual upload prompts within the flow produce higher-quality, more targeted uploads than a generic "upload your files" step at the end.

**Consequences:** The system must handle eventual consistency -- a framework section's compiled content may update after the client has moved past it, as async processing completes. The facilitator dashboard shows processing status indicators for pending items. The review/submit screen must reflect the most current state, including recently processed files and transcriptions.

### ADR-4: Session Management and Cross-Device Resumption

**Context:** Clients will start the intake on one device, close the browser, and return hours or days later -- potentially on a different device. The system must preserve their exact position without requiring account creation or login.

**Decision:** Use magic-link token-based sessions. When a client first accesses the intake, a unique session token is generated and embedded in the URL. This token is also stored in a secure HTTP-only cookie for same-device return. The session URL is included in all reminder communications. Auto-save fires after every interaction (answer submission, edit, upload). Session state includes: current position in the question tree, all submitted answers, all uploaded files, and UI state (which sections have been reviewed). Tokens expire 30 days after the workshop date.

**Alternatives Considered:**
- **Account creation with email/password** -- rejected because it adds friction that would reduce completion rates. The intake is a one-time flow, not an ongoing product. Forcing account creation for a 20-minute interview is disproportionate.
- **Email-based OTP (one-time password) on each return** -- rejected because it adds a step on every resume. Magic links in reminder emails achieve the same security without the extra friction.
- **Browser-only session (cookies/localStorage)** -- rejected because it does not support cross-device resumption, which is a core requirement (start on laptop, finish on phone).

**Consequences:** Magic links must be treated as sensitive -- they grant access to the client's intake data without authentication. Links should be single-use per click (the token refreshes on each access to prevent sharing). The system must handle concurrent access gracefully (e.g., client opens the link on two devices). Session data is stored server-side (PostgreSQL + Redis cache), not in the client's browser.

### ADR-5: Framework Mapping Approach

**Context:** All client inputs -- web-scraped content, typed answers, voice transcriptions, extracted file content, and pasted links -- must be mapped to the facilitator's prompt framework categories. This mapping is the bridge between raw client input and the structured output the facilitator needs for narrative generation.

**Decision:** Use a hybrid mapping approach. First-pass mapping is structural: each question in the intake flow is pre-tagged with its target framework section (e.g., the question "Who are your top competitors?" is tagged to `competitive-landscape`). This handles all content generated through the guided flow. Second-pass mapping is AI-driven: uploaded files, voice recordings, and links often contain content relevant to multiple framework sections. The AI Content Processor reads extracted content and distributes it across relevant sections with confidence scores. The facilitator reviews and can override AI mapping in the dashboard.

**Alternatives Considered:**
- **Purely structural mapping** (every input tagged to exactly one section by question position) -- rejected because uploaded files (e.g., a pitch deck) contain content spanning multiple framework sections. Forcing a single-section tag loses information.
- **Purely AI-driven mapping** (all content analyzed and categorized by LLM) -- rejected because it introduces unnecessary uncertainty for content that has an obvious home. If the client answered a question about competitors, that answer belongs in `competitive-landscape` with 100% confidence -- no AI judgment needed.
- **Client-directed mapping** (ask the client which section each upload belongs to) -- rejected because it exposes the framework structure to the client, who should never need to understand it. The framework is the facilitator's intellectual architecture, not the client's concern.

**Consequences:** The hybrid approach means most content (70-80%) is mapped with certainty via structural tagging, and only multi-section content (files, links, long voice recordings) requires AI classification. This reduces AI cost and error rate while preserving flexibility. The facilitator's dashboard shows provenance and mapping confidence, allowing quick review and override where needed. The AI mapping prompts must be carefully engineered to match the facilitator's categorization instincts -- initial calibration sessions with the facilitator are essential.
