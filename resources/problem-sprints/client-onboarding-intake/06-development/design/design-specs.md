# Design Specification — Workshop Intake Engine

**Sprint:** Client Onboarding Intake
**Role:** UX Designer
**Date:** 2026-03-07
**Input:** 10-step storyboard, product architecture (8 components, 12 entities, 5 ADRs), development handoff

---

## Table of Contents

1. [Screen-by-Screen Wireframe Descriptions](#1-screen-by-screen-wireframe-descriptions)
2. [Interaction Patterns](#2-interaction-patterns)
3. [Design System Foundation](#3-design-system-foundation)
4. [Accessibility Requirements](#4-accessibility-requirements)
5. [Key Design Decisions](#5-key-design-decisions)

---

## 1. Screen-by-Screen Wireframe Descriptions

### Global Layout Principles

All client-facing screens (Screens 1-8) share a minimal shell:
- **No sidebar navigation.** The flow is linear with branching; there is no reason to expose a nav menu.
- **No header logo bar.** The facilitator's brand is expressed through tone and video, not chrome.
- **Centered content column:** max-width 640px on desktop, full-width with 24px horizontal padding on mobile.
- **Progress indicator** (Screens 3-8): a thin horizontal bar pinned to the top of the viewport, filling left-to-right. Percentage label hidden -- the bar communicates progress through proportion alone.
- **Background:** Off-white (#FAFAF8) for client-facing screens. The facilitator dashboard uses a light neutral gray (#F4F4F2) to visually separate the two experiences.

---

### Screen 1: Welcome Email

**Note:** This is an email, not a web page. Specifications are for the HTML email template.

#### Layout Structure
- **Width:** 600px max, single-column, left-aligned text
- **From line:** Facilitator's real name and email address (e.g., "Sarah Kim <sarah@workshopname.com>")
- **Subject line:** "Getting started on your workshop -- one link, 20 minutes"

#### Content Hierarchy
1. **Primary:** Two short paragraphs in the facilitator's natural voice. First-person, warm, specific. Mentions the client's company name and references the "homework" the system has done.
2. **Secondary:** Single CTA button -- "Start Your Workshop Prep" -- centered, large (min 48px height, 200px width), high-contrast.
3. **Tertiary:** One-line note below the button: "You can pause and come back anytime. We've already done some homework on [Company Name] to give you a head start." Set in smaller type, muted color.

#### Conditional Copy
- **Rich web presence (scrape confidence high):** "We've already done our homework on [Company Name] to give you a head start."
- **Thin web presence (scrape confidence low):** "We've pulled together some initial thoughts on [Company Name] -- you'll fill in the rest."

#### Interactive Elements
| Element | Default | Hover | Active |
|---------|---------|-------|--------|
| CTA button | Solid primary color (#2B5CE6), white text, rounded corners (8px) | Slight darken (10%), subtle lift shadow | Scale down 1%, color darkens 15% |

#### Responsive Behavior
- Email is already single-column; on mobile email clients the CTA button stretches to full width minus padding.
- Text size bumps from 16px to 17px on mobile for readability.

#### Transition to Screen 2
- Clicking the CTA opens the intake URL in the user's default browser. The URL contains the magic-link session token.

---

### Screen 2: Video Kickoff

#### Layout Structure
- **Desktop:** Centered content column (max 640px). Video player occupies the top two-thirds of the viewport. Below: a single CTA button. Nothing else.
- **Mobile:** Video fills full width. CTA button sits below with generous top margin (32px).

#### Content Hierarchy
1. **Primary:** Embedded video player (16:9 aspect ratio, 60-90 seconds). Auto-play off. Poster frame shows the facilitator's face, mid-smile, with a play button overlay.
2. **Secondary:** CTA button -- "Let's Go" -- positioned below the video, centered.
3. **Tertiary:** None. The page is deliberately sparse to focus attention on the video.

#### Video Player Specifications
- Custom minimal player chrome: play/pause, scrub bar, mute toggle. No download button, no picture-in-picture, no playback speed control.
- On completion, the play button is replaced with a subtle checkmark and the "Let's Go" button pulses gently once to draw attention.

#### Interactive Elements
| Element | Default | Hover | Active | Disabled |
|---------|---------|-------|--------|----------|
| Play button (video overlay) | White circle with play triangle, 50% opacity background | Opacity increases to 80% | Scales down 2% | N/A |
| "Let's Go" button | Solid primary color, white text, rounded (8px), 48px height | Darken 10%, lift shadow | Scale down 1% | Gray background, 50% opacity (shown only if video hasn't loaded) |

#### Responsive Behavior
- **Desktop:** Video centered with comfortable vertical margin above.
- **Mobile:** Video edge-to-edge, pinned to top. CTA button padded 24px from edges, full-width.

#### Transition from Screen 1 / to Screen 3
- **In:** Page loads immediately on link click. No splash screen, no loading wall, no login prompt.
- **Out:** Clicking "Let's Go" triggers a smooth upward slide transition (300ms, ease-out) to Screen 3. The progress bar appears at the top and fills to its initial position (approximately 5%).

---

### Screen 3: Self-Segmentation

#### Layout Structure
- **Desktop:** Centered column (max 560px). Question text at top, four option cards stacked vertically below with 16px gaps between them.
- **Mobile:** Full-width with 24px padding. Option cards are full-width, stacked vertically.

#### Content Hierarchy
1. **Primary:** Question text -- "Before we dive in, which of these sounds most like your situation?" -- set in the heading typeface, 28px/32px desktop, 24px/28px mobile. Centered.
2. **Secondary:** Four option cards, each containing a single descriptive sentence. Cards are the primary interactive targets.

#### Option Card Content
Each card displays a single sentence in client language:
- "We have brand materials, pitch decks, and/or market research ready to share"
- "We have some materials but they're scattered or outdated"
- "Most of what we know is in our heads -- we haven't documented much"
- "We're early stage -- still figuring out our positioning"

#### Option Card Design
- **Dimensions:** Full-width of the content column, 72-88px height (flexible based on text length), rounded corners (12px).
- **Background:** White (#FFFFFF) with a 1px border (#E0E0DC).
- **Typography:** 17px body text, left-aligned with 20px left padding. Vertically centered within the card.

#### Interactive States
| State | Appearance |
|-------|------------|
| Default | White background, light border (#E0E0DC), no shadow |
| Hover (desktop) | Border color shifts to primary blue (#2B5CE6), subtle box shadow (0 2px 8px rgba(0,0,0,0.06)) |
| Pressed/Active | Background shifts to light blue tint (#EEF2FC), border becomes primary blue (2px) |
| Selected | Background light blue (#EEF2FC), left border accent (4px solid primary blue), checkmark icon appears at right edge |

#### Responsive Behavior
- Cards are already full-width on both breakpoints. On mobile, tap targets are naturally large (entire card is tappable, min 48px height).
- On very small screens (<360px), text may wrap to two lines; card height expands to accommodate.

#### Transition from Screen 2 / to Screen 4
- **In:** Slides up from below (300ms ease-out), overlapping the departing Screen 2.
- **Out:** After selection, a 400ms pause (the selected card holds its highlighted state), then the screen slides left while Screen 4 slides in from the right (350ms ease-out).

---

### Screen 4: Pre-Populated Review

**This is the most interaction-dense screen in the client flow.** It is the first point where the client encounters AI-generated content and must evaluate it. The design must make the three response actions unmistakably clear without overwhelming the user.

#### Layout Structure
- **Desktop:** Centered column (max 640px). A framework section card occupies the viewport. One card is shown at a time (not all sections stacked).
- **Mobile:** Full-width with 24px padding. Same one-card-at-a-time approach.

#### Content Hierarchy per Review Card
1. **Primary:** Section heading (e.g., "Your Market") -- 24px heading text, bold, positioned at the top of the card.
2. **Secondary:** Pre-populated content paragraph(s) -- 16px body text, displayed in a slightly recessed container (light warm gray background, #F7F6F3) with a thin left border accent in amber (#E8C547) to signal "this was written by AI, not you." The text itself is unhighlighted but the container treatment distinguishes it from client-authored content.
3. **Tertiary:** Three action buttons arranged horizontally below the content block.
4. **Quaternary:** A subtle helper line above the actions: "We found this on your website. How does it look?"

#### Three Action Buttons

These buttons are the critical interaction. They must be visually distinct, never ambiguous.

| Button | Label | Visual Treatment |
|--------|-------|-----------------|
| Confirm | "This is right" | Green-tinted background (#E8F5E9), dark green text (#2E7D32), checkmark icon left of text |
| Edit | "Close, but let me edit" | Amber-tinted background (#FFF8E1), dark amber text (#F57F17), pencil icon left of text |
| Rewrite | "This is off -- let me rewrite" | Light red-tinted background (#FFEBEE), dark red text (#C62828), refresh/redo icon left of text |

#### Button States
| State | Confirm | Edit | Rewrite |
|-------|---------|------|---------|
| Default | As above | As above | As above |
| Hover | Darken background 10%, slight lift | Darken background 10%, slight lift | Darken background 10%, slight lift |
| Active | Background darkens 15%, scale 99% | Background darkens 15%, scale 99% | Background darkens 15%, scale 99% |

#### Edit Mode (triggered by "Close, but let me edit")
- The pre-populated text container transforms into an editable text area. The background shifts from gray to white. A visible cursor appears at the point the user taps.
- The amber left border remains to remind the user this started as AI content.
- Two small buttons appear below the editor: "Save changes" (primary) and "Cancel" (text-only, muted).
- The text area auto-expands vertically as the user types -- no scrolling within the editor.

#### Rewrite Mode (triggered by "This is off -- let me rewrite")
- The AI-generated content fades to 30% opacity and slides up slightly. A blank text area appears below it, with placeholder text: "Write your version here..."
- The faded AI text serves as a reference the user can glance at while rewriting. A small "Dismiss" link collapses the AI text entirely if the user prefers a clean slate.
- Same "Save" / "Cancel" controls as Edit Mode.

#### Thin Web Presence Adaptation
- If the scrape produced insufficient content for a section, the review card is replaced with a guided question card (identical to Screen 5's layout). The transition is seamless -- no error message, no apology. The helper text reads: "We couldn't find much about your [section] online -- let's build this together."

#### Responsive Behavior
- **Desktop:** Three action buttons sit side by side in a row.
- **Mobile (< 640px):** Action buttons stack vertically, each full-width, with 12px gaps. Confirm is always topmost.

#### Transition Between Cards
- When the user confirms, saves an edit, or saves a rewrite, the current card slides left and the next framework section card slides in from the right (300ms ease-out).
- The progress bar advances with each completed card.

#### Transition to Screen 5
- After the last pre-populated section is reviewed, the flow transitions to Screen 5 (guided questions for sections without pre-populated content). The transition is the same slide-left animation -- the user should not perceive a mode change.

---

### Screen 5: Guided Deep-Dive (Branching Questions)

#### Layout Structure
- **Desktop:** Centered column (max 600px). Question text at top, input area below, contextual follow-up or upload prompt below that.
- **Mobile:** Full-width with 24px padding. Same vertical stack.

#### Content Hierarchy
1. **Primary:** The question itself -- 22px text, semi-bold weight. Conversational tone. E.g., "Who are the 2-3 companies your customers would consider instead of you?"
2. **Secondary:** Input area -- a text field (auto-expanding textarea) with a 2px bottom border in muted gray. No box outline -- the field feels open and inviting, not contained.
3. **Tertiary:** Input mode switcher (voice, file upload, link) -- small icon row positioned just below the right edge of the text field (see Screen 6 for full specification).
4. **Quaternary:** "Next" button -- appears only after the user has entered content (text, recording, upload, or link). Right-aligned, primary color.

#### Branching Follow-Up Behavior
- After the user answers and clicks "Next," the system may present a follow-up question. The follow-up slides in from below (250ms ease-out), pushing the previous question up. The previous answer is visible above in a collapsed state (first line of their answer, truncated with ellipsis, light gray background) so the user has context.
- Contextual upload prompts appear as gentle inline suggestions between questions: "Have a competitive analysis you could share?" with a dashed-border upload zone below. These are skippable -- the user can proceed without uploading.

#### Contextual Upload Zone
- Dashed border (2px, #C8C8C4), rounded corners (8px), 120px height.
- Center text: "Drop a file here or click to browse" in muted gray, 14px.
- File type icons (PDF, DOC, image) as small muted icons above the text.
- On file drop or selection, the zone collapses to a compact file chip (filename, file size, progress bar, checkmark on completion).

#### Interactive Elements
| Element | Default | Hover | Active | Disabled |
|---------|---------|-------|--------|----------|
| Text input field | 2px bottom border (#D0D0CC), placeholder text in light gray | Bottom border transitions to primary blue | Bottom border primary blue, 3px | N/A |
| "Next" button | Hidden (opacity 0) until content is entered | Visible: primary color, white text | Scale 99%, darken 15% | Gray, 50% opacity when input is empty |
| Upload zone | Dashed border, muted text | Border color shifts to primary blue, background tint (#F5F8FF) | Background tint deepens | N/A |

#### Responsive Behavior
- Identical layout on desktop and mobile -- the single-column, one-question-at-a-time approach is inherently responsive.
- On mobile, the "Next" button is pinned to the bottom of the viewport (fixed position, 16px padding, full-width) when the keyboard is not active. When the keyboard is active, the button remains in-flow below the text field.

#### Transition Between Questions
- **Forward:** Current question slides left, next slides in from right (300ms ease-out).
- **Follow-up branching:** Follow-up slides up from below (250ms ease-out), visually connecting it to the parent question.
- **After contextual upload:** The upload zone collapses to a file chip, then the next question slides in from the right after a 200ms pause.

---

### Screen 6: Flexible Input

**Note:** Screen 6 is not a separate page -- it is the input mode system available on every question across Screens 4 and 5. This specification defines the three alternative input modes that sit alongside the default text input.

#### Input Mode Switcher

A row of three small icon buttons, positioned at the bottom-right corner of the text input area:
- **Microphone icon** -- voice recording
- **Paperclip icon** -- file upload
- **Link icon** -- URL paste

Icons are 20px, muted gray (#8A8A86), with 12px spacing between them.

| State | Appearance |
|-------|------------|
| Default | Muted gray icons, no background |
| Hover (desktop) | Icon color transitions to primary blue, circular background tint appears (24px diameter) |
| Active | Icon color primary blue, background tint deepens |
| Recording active (mic) | Mic icon pulses red (#E53935), background circle solid light red |

#### Voice Recording UX

This is a critical flow for "talker" clients. It must work with a single tap on mobile.

**Recording Flow:**
1. **Tap mic icon** -- recording begins immediately. No permissions dialog on subsequent uses (browser remembers permission grant from first use).
2. **Recording active state:** The text input area transforms. The text field is replaced by a waveform visualization -- a horizontal bar of vertical lines oscillating in real-time, colored in primary blue. A timer counts up (MM:SS) at the left edge. The mic icon pulses red. A "Stop" label appears next to the mic icon.
3. **Tap stop (or mic icon again)** -- recording ends. The waveform freezes. Two buttons appear below: "Use this recording" (primary) and "Re-record" (text-only, muted). A small play button appears at the left of the frozen waveform for review.
4. **Tap "Use this recording"** -- the waveform collapses into a compact audio chip (waveform thumbnail, duration, play button). The "Next" button becomes active. The transcription processes asynchronously -- the user does not wait.
5. **Transcription available (async):** If the user is still on this screen when transcription completes, the audio chip expands to show a "View transcript" toggle. If they have moved on, the transcript is silently attached to the framework section.

**First-time permission flow:** On the very first recording attempt, the browser will request microphone permission. A helper tooltip appears above the mic icon before the user taps: "Tap to record your answer -- you can speak instead of typing." After the browser permission prompt, if denied, the mic icon becomes disabled with a tooltip: "Microphone access needed. Check your browser settings."

#### Link Paste UX

1. **Tap link icon** -- a compact input field appears below the text area, with placeholder: "Paste a URL (article, podcast, profile...)" and a "Add" button.
2. **Paste and tap "Add"** -- the URL collapses into a link chip displaying the domain name and a green checkmark. The system confirms: "Got it -- we'll pull the content from this link." The link processes asynchronously.
3. **Multiple links:** Each added link becomes a chip. Chips stack horizontally, wrapping to new lines as needed.

#### File Upload UX

1. **Tap paperclip icon** -- the system file picker opens (native OS dialog). Alternatively, on desktop, files can be dragged onto the text input area at any time (the input area shows a dashed border and "Drop file" overlay on drag-over).
2. **File selected/dropped** -- a file chip appears below the input area: filename, file size, a thin progress bar that fills during upload, then a green checkmark. Multiple files can be uploaded; chips stack.
3. **Supported formats note:** Small text below the first file chip on first upload: "We accept PDF, Word, PowerPoint, images, and audio files."

#### Responsive Behavior
- Input mode icons are the same size on mobile and desktop (touch targets are at least 44x44px including padding).
- Voice waveform is full-width on mobile.
- File chips and link chips wrap to new lines on narrow viewports.

---

### Screen 7: Pause and Return

**Note:** Screen 7 is not a page the user navigates to. It is a system behavior that is deliberately invisible to the user. This specification covers the absence of UI (no save dialogs) and the re-entry experience.

#### Auto-Save Behavior (Invisible)
- Every user action (answer submitted, edit saved, file uploaded, recording completed) triggers an auto-save to the server. There is no save button, no save indicator, no "saving..." spinner. The user should never think about saving.
- If the user closes the browser tab, closes the laptop, or navigates away, no "Are you sure you want to leave?" dialog appears. The session state is already saved.

#### Return Experience
- **Same device, same browser:** The user returns to the intake URL (from email, bookmark, or browser history). The page loads to exactly where they left off -- same question, same section, with all previous answers intact. A small, transient welcome-back message appears at the top for 3 seconds: "Welcome back -- you're [X]% done." then fades out.
- **Different device (magic link):** The user clicks the link in a reminder email or SMS. Same behavior -- exact position restored.
- **Visual continuity:** The progress bar immediately reflects their completion percentage. No "loading your session" interstitial -- the page renders with content.

#### Reminder Messages
- **Email reminder:** Subject: "You're [X]% done -- about [Y] minutes left." Body: Two sentences summarizing what was captured ("We have your market context and competitive landscape") and a CTA button: "Pick up where you left off." Facilitator's name in the from line.
- **SMS reminder (if preference is SMS):** "[Facilitator first name]: You're [X]% done with your workshop prep. About [Y] minutes left. [Magic link URL]"
- **Timing rules:** First reminder 24 hours after pause. No more than one reminder per 24-hour period. Urgency adapts to workshop date proximity. Reminders stop on submission.
- **Tone:** "We saved your progress" not "You haven't finished." Never guilt-trip.

---

### Screen 8: Review and Submit

#### Layout Structure
- **Desktop:** Centered column (max 680px). Framework section summary cards stacked vertically. Submit button at the bottom.
- **Mobile:** Full-width with 24px padding. Same vertical stack, larger touch targets.

#### Content Hierarchy
1. **Primary:** Heading -- "Here's what we've captured" -- 28px, centered, semi-bold.
2. **Secondary:** Subtitle -- "Review your answers below. Tap any section to add more." -- 16px, muted gray, centered.
3. **Core content:** Section summary cards, one per framework section.
4. **Terminal:** Submit button -- large, centered, full-width on mobile.

#### Section Summary Cards

Each card represents one framework section (Market Context, Customer Profile, Competitive Landscape, Brand Positioning, Company Story, Aspirations).

**Card layout:**
- Section name (18px, semi-bold) at the left.
- Status indicator at the right: green checkmark circle for complete sections, amber flag icon for thin sections.
- Below the section name: a two-line excerpt of the captured content (14px, muted gray, truncated with ellipsis).
- Below the excerpt: content source tags -- small pills (12px) indicating provenance: "AI-drafted," "Client-edited," "Voice recording," "File upload," "Link." Colored per content type (see Design System section).

**Expandable behavior:**
- Tapping a card expands it to show the full content for that section, with an "Edit" link in the top-right corner.
- Amber-flagged sections show a gentle suggestion when expanded: "You might want to add more here -- [contextual prompt]." with a text input area below.
- Expanded cards push subsequent cards down; only one card is expanded at a time.

#### Status Indicator Colors
| Status | Icon | Color | Meaning |
|--------|------|-------|---------|
| Complete | Checkmark in circle | Green (#2E7D32) | Section has sufficient content |
| Thin | Flag | Amber (#F9A825) | Section is submittable but could use more |

**Critical: Yellow flags are never blockers.** The submit button is always active regardless of how many sections are flagged.

#### Submit Button
| State | Appearance |
|-------|------------|
| Default | Primary blue (#2B5CE6), white text, "Submit your workshop prep", 56px height, full-width on mobile / 280px on desktop, rounded (12px) |
| Hover | Darken 10%, lift shadow |
| Active | Darken 15%, scale 99% |
| Submitting | Text changes to "Submitting...", a spinner replaces the arrow icon, button is non-interactive |
| Success | Background transitions to green (#2E7D32), text changes to "Submitted!", checkmark icon. Holds for 1.5 seconds, then transitions to a confirmation state. |

#### Post-Submission Confirmation
- The submit button and section cards fade out. A centered confirmation message appears:
  - Large checkmark icon (green, 64px)
  - "You're all set." (28px heading)
  - "Thanks for taking the time. [Facilitator name] will review everything and reach out if anything needs clarification." (16px body)
  - No further actions required. The page is a dead end by design -- the client's job is done.

#### Responsive Behavior
- Section cards are full-width on all breakpoints.
- Submit button is fixed to the bottom of the viewport on mobile (sticky), with a subtle top shadow to separate it from the scrollable content above.

---

### Screen 9: Facilitator Dashboard

**Note:** This is a separate application view accessible only to the facilitator. It has its own navigation and layout system, distinct from the client-facing flow.

#### Layout Structure
- **Desktop (primary use case):** Left sidebar (240px) with client list + navigation. Main content area (remaining width) showing the selected client's intake data. No mobile optimization required for V1 -- the facilitator works on a laptop.
- **Mobile (stretch):** Collapsed sidebar as a hamburger menu. Main content full-width. Not a V1 priority.

#### Sidebar: Client Pipeline
- List of all clients with active or completed intakes.
- Each client row shows: client name, company name, status pill (In Progress / Submitted / Reviewed), completion percentage.
- Active/in-progress clients sort to the top. Submitted-but-unreviewed clients are highlighted with a subtle blue left border.
- A count badge shows the number of unreviewed submissions.

#### Main Content: Per-Client View

**Organized by framework section, not by submission chronology.**

**Top bar:**
- Client name and company (24px heading).
- Workshop date with countdown ("in 11 days").
- Overall completeness score (e.g., "87%") displayed as a circular progress indicator.
- Action buttons: "Request Follow-Up" (secondary) and "Generate Narrative Package" (primary).

**Framework section tabs or cards:**
- Each framework section (Market Context, Customer Profile, Competitive Landscape, Brand Positioning, Company Story, Aspirations) is presented as a horizontal tab row at the top of the content area, or as a vertical card list -- tabs preferred for desktop width.
- Clicking a tab reveals the compiled content for that section.

**Section content area:**
- Merged content from all sources (web scrape, client text, voice transcription, file extraction).
- Content type badges inline with content blocks:
  - "AI-drafted" -- light blue pill
  - "Client-edited" -- teal pill
  - "Client-authored" -- dark teal pill
  - "File-extracted" -- purple pill
  - "Voice-transcribed" -- orange pill
- Completeness bar per section (green if above threshold, amber if below).
- Gap detection callout: if a section is thin, a yellow callout box appears: "Gap detected: [specific missing element]. Suggested follow-up: [draft question]." with an "Approve & Send" button.

#### Follow-Up Flow
1. Facilitator clicks "Request Follow-Up" (either from top bar for general follow-up, or from a specific gap callout).
2. A modal or side panel opens with the AI-drafted follow-up message, written in the facilitator's voice.
3. The facilitator can edit the message, then click "Send."
4. The message is sent to the client via their preferred channel. The dashboard shows a "Follow-up pending" badge on the relevant section.

#### "Generate Narrative Package" Button
- Opens a full-screen compilation view showing all framework sections with their compiled content, source attribution, and confidence levels.
- The facilitator reviews, can edit any section inline, then clicks "Generate" to trigger the narrative pipeline handoff.
- This is the bridge to the existing narrative generation system (out of scope for this sprint but the output format must match exactly).

---

### Screen 10: Framework-Ready Package

#### Layout Structure
- **Full-screen view** (accessed from the dashboard via "Generate Narrative Package").
- Left column (30%): Table of contents listing all framework sections with completeness indicators.
- Right column (70%): Selected section's compiled content, source attribution, and confidence level.

#### Content Per Section
- **Compiled text:** The merged narrative from all input sources for this framework slot.
- **Source attribution:** Inline annotations or a collapsible source panel showing where each piece of content originated:
  - "From client voice recording, transcribed"
  - "From uploaded competitive matrix, page 3"
  - "AI-drafted from website (client confirmed)"
  - "Client-authored via guided question"
- **Confidence indicator:** A small horizontal bar (green/amber/red) reflecting the AI's confidence in completeness and accuracy. Green = multiple corroborating sources. Amber = single source or thin coverage. Red = gap, needs follow-up.

#### Interactive Elements
- Inline editing on any section (click to edit, auto-save).
- "Generate" button at the bottom -- large, primary color. Triggers the narrative generation pipeline and shows a confirmation: "Narrative package sent to the generation pipeline."

---

## 2. Interaction Patterns

### 2.1 Question Transition Animations

**Forward navigation (question to question):**
- Current question container slides left and fades to 0 opacity over 300ms (ease-out curve: cubic-bezier(0.25, 0.1, 0.25, 1.0)).
- Next question container slides in from the right edge, starting at translateX(80px) and opacity 0, arriving at translateX(0) and opacity 1 over 300ms (same curve), with a 50ms delay after the exit animation begins.
- Net perceived duration: approximately 350ms. Fast enough to feel responsive, slow enough to register as directional movement.

**Branching follow-up (contextual sub-question):**
- The follow-up question fades in and slides up from translateY(24px) over 250ms. It appears below the parent question's collapsed answer, creating a visual parent-child relationship.
- The parent question's answer collapses to a single-line summary (first 80 characters + ellipsis) with a light gray background, making room for the follow-up.

**Backward navigation (if implemented):**
- Mirror of forward: current slides right, previous slides in from the left. Same timing.
- Back navigation is available via a small "Back" text link (with left-pointing chevron) in the top-left corner, below the progress bar.

### 2.2 File Upload Interaction

**Drag and drop (desktop):**
1. User drags a file over the browser window. The current question's input area gains a dashed blue border and a semi-transparent overlay: "Drop your file here." The rest of the page dims slightly (10% opacity overlay).
2. File is dropped. The overlay disappears. A file chip appears below the input area, showing: file icon (type-specific: PDF, DOC, image, etc.), filename (truncated at 30 characters), file size, and an upload progress bar (thin, primary blue, animating left-to-right).
3. Upload completes. Progress bar is replaced by a green checkmark. The chip is now static.
4. If upload fails: the checkmark is replaced by a red exclamation icon and "Upload failed -- try again" text. Tapping the chip retries.

**Click to browse:**
- Tapping the paperclip icon (or the contextual upload zone) opens the native OS file picker. Same chip behavior on selection.

**Multiple files:**
- Each file gets its own chip. Chips stack vertically with 8px gaps. A "Remove" (X) button appears on hover/tap for each chip.

### 2.3 Voice Recording Interaction

**Full sequence:**
1. User taps mic icon. (First time: browser permission dialog appears. Helper tooltip: "Allow microphone access to record your answer.")
2. Permission granted. Recording begins immediately -- no countdown, no "are you ready?" The text input area smoothly morphs (200ms) into the waveform display: background darkens slightly, a real-time waveform (thin vertical bars, primary blue, oscillating based on audio amplitude) fills the area. Timer (MM:SS) at the left edge. Pulsing red dot at the right edge near the mic icon.
3. User speaks. Waveform responds in real-time. Maximum recording length: 3 minutes. At 2:30, a gentle prompt appears: "Recording will stop in 30 seconds."
4. User taps mic icon again (or the "Stop" label). Recording stops. Waveform freezes at its final state.
5. Two buttons appear below the frozen waveform: "Use this recording" (primary) and "Re-record" (text-only). A small play/pause button at the left edge of the waveform allows review.
6. User taps "Use this recording." The waveform collapses (200ms) into a compact audio chip: small waveform thumbnail, duration label, play button. The "Next" button activates.
7. Transcription processes asynchronously. If the user remains on screen and transcription completes, a "View transcript" toggle appears on the audio chip. Tapping it expands a read-only transcript below the chip.

### 2.4 Pre-Populated Card Expand/Collapse (Screen 4)

**Edit mode activation:**
- User taps "Close, but let me edit." The pre-populated text container (gray background) transitions to white background (200ms). The text becomes editable -- a cursor appears. The amber left border remains. The three action buttons are replaced by "Save changes" and "Cancel."

**Edit mode deactivation (save):**
- User taps "Save changes." The editor transitions back to a static display (gray background returns). The content now shows the user's edited version. A subtle "Edited" badge appears in the top-right corner of the card. The card then slides left to the next section (300ms).

**Rewrite mode activation:**
- User taps "This is off -- let me rewrite." The AI content fades to 30% opacity and compresses vertically (200ms). A blank text area appears below with focus. The user writes freely.

**Rewrite mode deactivation (save):**
- User taps "Save." The faded AI content disappears entirely (150ms). The user's content is now the section content, marked with a "Rewritten" badge. Card transitions to next section.

### 2.5 Progress Indicator Behavior

**Style:** Thin horizontal bar (3px height) pinned to the top of the viewport, spanning full width. Primary blue fill on a light gray (#E8E8E4) track.

**Advancement:**
- The bar advances smoothly (300ms transition) each time the user completes a question, confirms/edits/rewrites a pre-populated section, or submits a file/recording.
- Progress calculation: total completed framework inputs / total expected inputs (varies by segmentation path). The denominator is dynamic -- if branching adds follow-up questions, the total adjusts so the bar never moves backward.
- The bar should never visibly jump backward. If additional questions are added by branching logic, the bar holds position and the newly added questions reduce the per-step increment going forward.

**Completion:**
- At 100%, the bar flashes gently once (opacity pulse) and transitions to green (#2E7D32).

### 2.6 Page Transition Patterns

| Transition | Animation | Duration | Easing |
|-----------|-----------|----------|--------|
| Screen 2 to Screen 3 | Slide up | 300ms | ease-out |
| Screen 3 to Screen 4 | Slide left | 350ms | ease-out |
| Between review cards (Screen 4) | Slide left | 300ms | ease-out |
| Screen 4 to Screen 5 | Slide left (seamless) | 300ms | ease-out |
| Between questions (Screen 5) | Slide left | 300ms | ease-out |
| Follow-up questions | Slide up / fade in | 250ms | ease-out |
| To Screen 8 (Review) | Fade + slight scale up (from 98% to 100%) | 400ms | ease-out |

All transitions use `will-change: transform, opacity` for GPU acceleration. No transition plays while the user is mid-input (typing, recording, uploading).

---

## 3. Design System Foundation

### 3.1 Color Palette

#### Primary
| Name | Hex | Usage |
|------|-----|-------|
| Primary Blue | #2B5CE6 | CTA buttons, active states, progress bar, links, focus rings |
| Primary Dark | #1E42B0 | Hover states on primary elements, active text links |

#### Secondary
| Name | Hex | Usage |
|------|-----|-------|
| Warm Off-White | #FAFAF8 | Client-facing page backgrounds |
| Light Gray | #F4F4F2 | Dashboard backgrounds, secondary surfaces |
| Card White | #FFFFFF | Card backgrounds, input fields |

#### Neutrals
| Name | Hex | Usage |
|------|-----|-------|
| Text Primary | #1A1A18 | Headings, body text, primary labels |
| Text Secondary | #6B6B66 | Subtitles, helper text, timestamps |
| Text Muted | #8A8A86 | Placeholder text, disabled labels, icon default states |
| Border Default | #E0E0DC | Card borders, dividers, inactive input borders |
| Border Subtle | #EEEEEA | Section separators, very subtle dividers |

#### Semantic
| Name | Hex | Light Tint | Usage |
|------|-----|------------|-------|
| Success Green | #2E7D32 | #E8F5E9 | Checkmarks, confirmed sections, submit success, progress complete |
| Warning Amber | #F9A825 | #FFF8E1 | Thin-section flags, edit mode accent, AI-content indicator |
| Error Red | #C62828 | #FFEBEE | Upload failures, rewrite mode accent, recording indicator |
| Info Blue | #1565C0 | #E3F2FD | Informational callouts, processing indicators |

#### Content Provenance (Dashboard)
| Source Type | Pill Color (bg/text) |
|-------------|---------------------|
| AI-drafted | #E3F2FD / #1565C0 |
| Client-edited | #E0F2F1 / #00695C |
| Client-authored | #B2DFDB / #004D40 |
| File-extracted | #F3E5F5 / #6A1B9A |
| Voice-transcribed | #FFF3E0 / #E65100 |

### 3.2 Typography

**Font family:** System font stack for performance and native feel:
```
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
```

If a branded typeface is specified by the facilitator, it replaces the system stack for headings only. Body text remains system fonts for legibility.

#### Type Scale

| Level | Size (desktop) | Size (mobile) | Weight | Line Height | Usage |
|-------|---------------|---------------|--------|-------------|-------|
| Display | 32px | 28px | 600 (semi-bold) | 1.2 | Screen 8 heading, confirmation messages |
| Heading 1 | 28px | 24px | 600 | 1.25 | Question text, section titles in review |
| Heading 2 | 24px | 20px | 600 | 1.3 | Pre-populated section headings, dashboard client name |
| Heading 3 | 20px | 18px | 500 (medium) | 1.35 | Card titles, sub-section headings |
| Body Large | 18px | 17px | 400 (regular) | 1.55 | Self-segmentation option text |
| Body | 16px | 16px | 400 | 1.6 | Primary body text, pre-populated content, answers |
| Body Small | 14px | 14px | 400 | 1.5 | Helper text, excerpts, content source tags |
| Caption | 12px | 12px | 500 | 1.4 | Provenance pills, timestamps, status labels |
| Label | 14px | 14px | 500 | 1.4 | Button text, input labels |

### 3.3 Spacing System

**Base unit:** 4px

| Token | Value | Usage |
|-------|-------|-------|
| space-1 | 4px | Icon internal padding, tight gaps |
| space-2 | 8px | Between inline elements, chip gaps |
| space-3 | 12px | Between stacked buttons, card internal padding (compact) |
| space-4 | 16px | Between cards, between form elements, card internal padding (default) |
| space-5 | 20px | Card left padding (with text), section internal margins |
| space-6 | 24px | Page horizontal padding (mobile), between content groups |
| space-8 | 32px | Between major sections, vertical margins around headings |
| space-10 | 40px | Top padding on question screens |
| space-12 | 48px | Vertical space between video and CTA (Screen 2) |
| space-16 | 64px | Top margin on Screen 2 (desktop) |

**Content column max-widths:**
- Client flow: 640px (review/submit at 680px)
- Dashboard main content: fluid (sidebar 240px, rest to main)
- Question text: 600px (slightly narrower for readability)

### 3.4 Component Inventory

These are the reusable components required across the system. Each should be built as an isolated, composable unit.

#### Client Flow Components
| Component | Variants | Used In |
|-----------|----------|---------|
| **ProgressBar** | Default, complete (green) | Screens 3-8 |
| **QuestionCard** | Text question, review card (with pre-populated content), upload prompt card | Screens 4, 5 |
| **OptionCard** | Default, hover, selected | Screen 3 |
| **ActionButtonRow** | Confirm/Edit/Rewrite trio | Screen 4 |
| **TextInput** | Single-line, auto-expanding textarea, inline editor | Screens 4, 5, 6 |
| **InputModeSwitcher** | Mic, paperclip, link icons with states | Screens 5, 6 |
| **VoiceRecorder** | Idle, recording (waveform), review (frozen waveform), chip (collapsed) | Screen 6 |
| **FileChip** | Uploading (with progress), complete (with checkmark), error (with retry) | Screens 5, 6 |
| **LinkChip** | URL entered, processing, confirmed | Screen 6 |
| **SectionSummaryCard** | Complete (green check), thin (amber flag), expanded (with edit) | Screen 8 |
| **SubmitButton** | Default, hover, submitting (spinner), success (green check) | Screen 8 |
| **ProvenancePill** | AI-drafted, client-edited, client-authored, file-extracted, voice-transcribed | Screens 8, 9, 10 |
| **TransientToast** | Welcome-back message, auto-save confirmation (if ever needed), error alerts | Screen 7 return, errors |
| **CTAButton** | Primary (solid), secondary (outline), text-only | All screens |

#### Dashboard Components
| Component | Variants | Used In |
|-----------|----------|---------|
| **ClientListItem** | In-progress, submitted, reviewed | Screen 9 sidebar |
| **FrameworkTab** | Default, active, with gap badge | Screen 9 main |
| **ContentBlock** | With provenance pill, with source annotation, editable | Screens 9, 10 |
| **GapCallout** | With "Approve & Send" action | Screen 9 |
| **FollowUpComposer** | Modal/panel with editable draft, send button | Screen 9 |
| **CompletenessIndicator** | Circular (overall), bar (per-section) | Screens 9, 10 |
| **ConfidenceBar** | Green / amber / red states | Screen 10 |
| **SourceAnnotation** | Inline tag or collapsible panel | Screen 10 |

### 3.5 Tone and Voice Guidelines for System Copy

The intake system speaks in the facilitator's voice -- warm, direct, and knowledgeable -- not in corporate system-speak. Every piece of copy the client sees should feel like it was written by a person who cares about their business, not generated by a platform.

#### Principles

| Principle | Do | Don't |
|-----------|-----|-------|
| **Conversational, not clinical** | "Who are the 2-3 companies your customers would consider instead of you?" | "Please list your primary competitors." |
| **Encouraging, not evaluating** | "You might want to add more here" | "This section is incomplete" |
| **Specific, not generic** | "Upload your competitive analysis" | "Upload relevant documents" |
| **Calm, not anxious** | "You can pause and come back anytime" | "Warning: unsaved changes will be lost" |
| **Respectful of all levels** | "Most of what we know is in our heads -- we haven't documented much" | "We don't have any materials" |
| **Action-oriented, not passive** | "Let's build this together" | "This section needs to be completed" |

#### Copy Patterns by Context

| Context | Pattern | Example |
|---------|---------|---------|
| Question prompts | Direct question, second person, present tense | "What makes you different from [Competitor 1]?" |
| Helper text | Parenthetical, lowercase start, brief | "(you can also record a voice answer or upload a file)" |
| Upload prompts | Contextual suggestion, optional framing | "Have a competitive analysis you could share?" |
| Error states | Blame the system, not the user | "Something went wrong on our end -- try again" |
| Empty states | Opportunity framing | "We couldn't find much about your market online -- let's build this together" |
| Confirmations | Brief, warm, forward-looking | "Got it. Next up: your brand story." |
| Reminders | Progress-affirming, time-contextual | "You're 65% done -- about 8 minutes left" |

---

## 4. Accessibility Requirements

### 4.1 WCAG 2.1 AA Compliance — Per Screen

#### All Client Screens (Global)
- All text meets 4.5:1 contrast ratio against its background (WCAG 1.4.3). Primary text (#1A1A18) on off-white (#FAFAF8) = 15.2:1. Muted text (#6B6B66) on off-white = 5.8:1 (passes AA). All pass.
- All interactive elements meet 3:1 contrast ratio against adjacent colors (WCAG 1.4.11).
- Touch targets are minimum 44x44px on mobile (WCAG 2.5.5 AAA target, adopted as our minimum).
- Animations respect `prefers-reduced-motion`: all slide/fade transitions are replaced with instant state changes when this preference is set.

#### Screen 2: Video Kickoff
- Video must have captions (either burned in or as a VTT track). The facilitator's video content must be captioned for deaf/hard-of-hearing users (WCAG 1.2.2).
- The play button on the video poster must be keyboard-focusable and activatable with Enter/Space.

#### Screen 3: Self-Segmentation
- Each option card is a focusable element (role="radio" within a radiogroup). Arrow keys navigate between options. Enter/Space selects.
- Selected state is communicated via `aria-checked="true"` and a visible style change (not color alone -- the left border accent + checkmark icon provide redundant cues).

#### Screen 4: Pre-Populated Review
- The AI-generated content container has an `aria-label` indicating it is system-generated: "Pre-populated content based on your website. Review and choose an action below."
- The three action buttons are clearly labeled and distinguishable by icon + text (not color alone). Each button has a unique `aria-label`.
- Edit mode: the text area receives focus automatically on activation. `aria-live="polite"` announces "Editing mode active" to screen readers.

#### Screen 5: Guided Questions
- Each question has a visible label (the question text itself) associated with the input via `aria-labelledby`.
- The "Next" button appears only when valid input exists; its appearance is announced via `aria-live="polite"`.
- Contextual upload zones have descriptive `aria-label`: "Optional: upload a file related to [section name]."

#### Screen 6: Voice Recording
- The mic icon button has `aria-label="Record a voice answer"`. During recording, it changes to `aria-label="Stop recording"`.
- Recording state is announced: "Recording started" and "Recording stopped, [duration]."
- The waveform visualization is decorative (`aria-hidden="true"`); the timer provides the functional information.
- "Use this recording" and "Re-record" buttons are keyboard-focusable.

#### Screen 8: Review and Submit
- Each section summary card is expandable; implemented as `<details>/<summary>` or a button with `aria-expanded`.
- Status indicators (green check, amber flag) have text alternatives: `aria-label="Complete"` and `aria-label="Could use more detail"`.
- The submit button has `aria-live="assertive"` on its container so state changes ("Submitting...", "Submitted!") are announced.

#### Screen 9: Dashboard
- Tab navigation for framework sections uses `role="tablist"`, `role="tab"`, and `role="tabpanel"` ARIA patterns.
- Follow-up composer modal traps focus while open (WCAG 2.4.3).
- Gap callout text is readable by screen readers; the "Approve & Send" button is clearly labeled.

### 4.2 Keyboard Navigation Flow

**Client flow (Screens 2-8):**
1. Tab lands on the primary interactive element of each screen (play button on Screen 2, first option card on Screen 3, first action button on Screen 4, text input on Screen 5).
2. Within option groups (Screen 3): Arrow keys move between options, Enter/Space selects.
3. Within Screen 4: Tab cycles through the three action buttons. Enter activates. In edit mode, Tab moves from the text area to "Save changes" to "Cancel."
4. Within Screen 5: Focus starts in the text input. Tab moves to the input mode icons (mic, file, link), then to the "Next" button.
5. "Back" link is always the first focusable element (after the progress bar, which is non-interactive).

**Dashboard (Screen 9):**
- Standard web application keyboard patterns. Tab navigates between sidebar items, framework tabs, and action buttons. Enter activates.

### 4.3 Screen Reader Considerations

- All page transitions announce the new screen context via an `aria-live="polite"` region: "Question 4 of 12: Who are your top competitors?"
- Progress bar has `role="progressbar"` with `aria-valuenow`, `aria-valuemin="0"`, and `aria-valuemax="100"`.
- File upload status changes are announced: "File [name] uploading," "File [name] uploaded successfully."
- Transcription availability is announced: "Transcript available for your voice recording."

### 4.4 Color Contrast Requirements

All color pairings used in the interface have been validated:

| Foreground | Background | Ratio | Pass? |
|-----------|------------|-------|-------|
| #1A1A18 (text) | #FAFAF8 (page bg) | 15.2:1 | AA + AAA |
| #6B6B66 (secondary text) | #FAFAF8 (page bg) | 5.8:1 | AA |
| #FFFFFF (button text) | #2B5CE6 (primary blue) | 4.8:1 | AA |
| #2E7D32 (confirm text) | #E8F5E9 (confirm bg) | 5.1:1 | AA |
| #F57F17 (edit text) | #FFF8E1 (edit bg) | 3.8:1 | AA (large text only) |
| #C62828 (rewrite text) | #FFEBEE (rewrite bg) | 5.9:1 | AA |
| #8A8A86 (muted) | #FAFAF8 (page bg) | 3.5:1 | Fails AA for body text |

**Action required:** The amber "Edit" button text (#F57F17 on #FFF8E1) at 3.8:1 fails AA for normal-size text. Darken the text color to #E65100 (ratio 5.2:1) to pass. The muted text color (#8A8A86 at 3.5:1) is only used for placeholder text and decorative labels, which are exempt per WCAG 1.4.3. If used for meaningful content, darken to #757571 (4.6:1).

### 4.5 Focus Management During Transitions

- When a new question slides in, focus moves to the primary interactive element (text input or first option) after the transition animation completes.
- When a modal opens (e.g., follow-up composer on the dashboard), focus traps inside the modal. On close, focus returns to the element that triggered the modal.
- When the user enters edit/rewrite mode on Screen 4, focus moves to the text area.
- When the user returns to a paused session (Screen 7 return), focus moves to the primary interactive element of the current question.
- Focus is never moved during an animation -- it moves only after the animation completes to prevent disorientation.

---

## 5. Key Design Decisions

### DD-1: One Question at a Time (No Full-Form View)

**Decision:** The client intake never shows more than one question or review card simultaneously. There is no "view all questions" mode, no scrollable form, and no section navigation menu.

**Rationale:** The sprint research identified that client procrastination and non-submission are driven by overwhelm, not laziness. Showing a 30-field form triggers "I'll do this later" behavior. TurboTax's one-question-at-a-time pattern (Inspiration #2) and Typeform's progressive disclosure (Inspiration #3) both demonstrate that reducing the visible ask to a single question dramatically increases completion rates. The storyboard was designed around this principle at every step.

**Tradeoff:** Users who want to skip ahead or see the full scope cannot. Power users (e.g., a CMO with everything ready to paste) may find the linear flow slower than a bulk upload. This is an acceptable tradeoff because: (a) the self-segmentation routes "documents-ready" clients to a faster path with more upload prompts and fewer narrative questions, and (b) the alternative (showing the full form) risks losing the 70% of clients who are not power users.

**What this means for implementation:** No skip-to-section navigation. No "view all" toggle. The back button allows returning to previous questions, but forward navigation is sequential only.

---

### DD-2: AI Content Visually Distinct but Not Stigmatized

**Decision:** Pre-populated AI content is displayed in a slightly recessed container with a warm gray background and an amber left border. It is never labeled with "AI-generated" text. The framing is "Here's what we found on your website" -- attribution is to the source (their website), not the generation method (AI).

**Rationale:** Labeling content as "AI-generated" triggers skepticism in many users -- they expect it to be wrong and scrutinize it negatively rather than reading it on its merits. The storyboard frames this content as research ("we did our homework"), which is accurate: the AI read their public website and summarized it. The visual treatment (recessed container, amber border) signals "this is a draft for your review" without triggering AI anxiety. The three action buttons (confirm / edit / rewrite) give the client full control regardless.

**Tradeoff:** Some transparency-focused stakeholders may argue that not labeling AI content is deceptive. The counter: the system is transparent about the source ("Based on what we found on your website") and gives the client full editorial control. The facilitator dashboard does show "AI-drafted" provenance labels because the facilitator needs to weight content reliability -- that is a professional tool context, not a client-facing one.

---

### DD-3: Yellow Flags as Suggestions, Never Blockers

**Decision:** On the review/submit screen (Screen 8), sections with thin content are flagged with an amber indicator and a gentle prompt ("You might want to add more here"). The submit button is always active. There is no minimum completeness threshold for submission.

**Rationale:** The current system fails because clients never submit at all. A system that blocks submission because a section is "incomplete" recreates the paralysis that causes the problem. The storyboard is explicit: "a client who wants to submit at 75% completeness should be able to." The facilitator has a follow-up mechanism (Screen 9) to request additional input on thin sections. Getting 75% submitted is infinitely better than getting 100% never started.

**Tradeoff:** Some submissions will be thin. The facilitator will need to send follow-ups. This is a known cost -- the system is designed to make follow-ups one-click. The alternative (blocking submission) has a higher cost: total non-submission, which is the status quo the sprint exists to solve.

---

### DD-4: System Font Stack Instead of Custom Typeface

**Decision:** The intake flow uses the system font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto...`) rather than loading a custom web font.

**Rationale:** Three factors drove this decision: (1) Performance -- the intake loads via a magic link, often on mobile. Every millisecond of load time risks abandonment. A custom font adds 50-150KB and a render-blocking request. (2) Native feel -- system fonts render with the sharpness and weight the user's device was designed for. The intake should feel like a natural part of the device experience, not a "branded portal." (3) The facilitator's brand is expressed through tone, video, and copy -- not through typeface. The video kickoff (Screen 2) establishes the brand far more effectively than a font choice.

**Tradeoff:** The intake will not have a distinctive typographic identity. If the facilitator has a strong brand with a specific typeface, it can be loaded for headings only (the spec accommodates this as an option). Body text remains system fonts in all cases.

---

### DD-5: Dashboard Organized by Framework Section, Not Chronology

**Decision:** The facilitator dashboard (Screen 9) presents client intake data organized by framework section (Market Context, Competitive Landscape, Brand Story, etc.), not by the order or time the client submitted content.

**Rationale:** The facilitator's mental model is the prompt framework, not the client's session timeline. When preparing for a workshop, the facilitator thinks "What do I have for Competitive Landscape?" not "What did the client submit at 3:47 PM?" The storyboard is explicit: "The facilitator does not care that Sarah uploaded a PDF at 3:47 PM -- they care that the Competitive Landscape section now has three named competitors with differentiation statements." Organizing by framework section means the dashboard directly maps to the facilitator's workflow -- one tab per section, with all relevant content (from any source, submitted at any time) aggregated.

**Tradeoff:** Content that spans multiple framework sections (e.g., a pitch deck that touches market, competition, and positioning) must be split and displayed in multiple tabs. The AI Content Processor handles this splitting, but the facilitator may occasionally need to cross-reference between tabs. Source attribution (provenance pills) mitigates this by letting the facilitator see that two different sections reference content from the same uploaded document.

---

## Appendix: Screen Flow Diagram (Text)

```
[Email (Screen 1)]
        |
        v
[Video Kickoff (Screen 2)]
        |
        v
[Self-Segmentation (Screen 3)]
        |
        +-- "documents-ready" path
        +-- "scattered-outdated" path
        +-- "in-our-heads" path
        +-- "early-stage" path
        |
        v
[Pre-Populated Review (Screen 4)]  <-- sections with scraped content
        |
        v
[Guided Deep-Dive (Screen 5)]  <-- sections without scraped content
  + Flexible Input (Screen 6) available on all questions
  + Pause/Return (Screen 7) available at any point
        |
        v
[Review & Submit (Screen 8)]
        |
        v
[Facilitator Dashboard (Screen 9)]
        |
        v
[Framework-Ready Package (Screen 10)]
```
