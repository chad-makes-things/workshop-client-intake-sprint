# Problem Map — Client Onboarding Intake

## Actors

| Actor | Description |
|-------|-------------|
| **Client** | The business signing up for the workshop. Could be a founder, marketing lead, or ops person. Varying digital sophistication. |
| **Facilitator** | The workshop business owner. Currently does all intake manually. The bottleneck. |
| **System / AI** | The future automated layer — intake platform, file processing, AI narrative synthesis. Does not exist yet. |
| **Workshop** | The workshop sessions themselves — the downstream consumer of the market narrative. |

---

## Journey Map (Left to Right)

```
CLIENT                    FACILITATOR                SYSTEM / AI               WORKSHOP
  |                           |                          |                        |
  |  1. SIGNS UP              |                          |                        |
  |  for workshop             |                          |                        |
  |                           |                          |                        |
  |  2. RECEIVES request      |  3. SENDS intake         |                        |
  |  for business materials   |  request (email/text)    |                        |
  |                           |                          |                        |
  |  4. GATHERS internal      |                          |                        |
  |  materials (or realizes   |                          |                        |
  |  they have very little)   |                          |                        |
  |           [!]             |                          |                        |
  |                           |                          |                        |
  |  5. SENDS materials       |  6. RECEIVES materials   |                        |
  |  via text, email,         |  across scattered        |                        |
  |  Dropbox, file share      |  channels                |                        |
  |           [!]             |           [!]            |                        |
  |                           |                          |                        |
  |                           |  7. DOWNLOADS and        |                        |
  |                           |  CONSOLIDATES files      |                        |
  |                           |  from all sources        |                        |
  |                           |           [!]            |                        |
  |                           |                          |                        |
  |  8. RESPONDS to           |  9. FOLLOWS UP for       |                        |
  |  follow-up questions      |  missing information     |                        |
  |  (often delayed)          |  or clarifications       |                        |
  |           [!]             |           [!]            |                        |
  |                           |                          |                        |
  |                           | 10. ORGANIZES materials  |                        |
  |                           |  into prompt framework   |                        |
  |                           |           [!]            |                        |
  |                           |                          |                        |
  |                           | 11. RUNS AI prompts      | 12. GENERATES market   |
  |                           |  with structured input   |  narrative draft       |
  |                           |                          |                        |
  |                           | 13. REVIEWS and EDITS    |                        |
  |                           |  the narrative           |                        |
  |                           |           [!]            |                        |
  |                           |                          |                        |
  |                           | 14. FINALIZES narrative  |                        |
  |                           |  for workshop use        |                        |
  |                           |                          |          15. CONSUMES  |
  |                           |                          |          narrative in  |
  |                           |                          |          Session 1     |
  |                           |                          |              [GOAL]    |
```

---

## Step Details and Risk Assessment

### Step 1: Client SIGNS UP for workshop
- **Actor:** Client
- **Risk:** Low
- **Notes:** This step works. The workshop sells well. No intervention needed here.

### Step 2: Client RECEIVES request for business materials
- **Actor:** Client
- **Risk:** Medium
- **Notes:** The request is informal — often verbal or a quick message. No standardized ask. Clients may not understand what is needed or why.

### Step 3: Facilitator SENDS intake request
- **Actor:** Facilitator
- **Risk:** Medium
- **Notes:** Currently ad hoc. No template, no checklist, no system. The facilitator adapts the ask per client, which is personal but unscalable.

### Step 4: Client GATHERS internal materials
- **Actor:** Client
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** This is where the spectrum problem hits hardest. Some clients have brand decks, market research, and positioning docs. Others have nothing but "our website, but we don't even like it." The client's ability to gather materials determines the quality of everything downstream. If they struggle here, they may ghost, delay, or send incomplete inputs.
- **Failure mode:** Client doesn't know what to send, feels overwhelmed, delays indefinitely.

### Step 5: Client SENDS materials via scattered channels
- **Actor:** Client
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** Materials arrive via text messages, emails, Dropbox links, secure file services, and verbal anecdotes. There is no single destination. The facilitator must monitor multiple channels and mentally track what has and hasn't arrived from each client.
- **Failure mode:** Materials get lost, duplicated, or arrive without context. Facilitator spends hours just locating and downloading files.

### Step 6: Facilitator RECEIVES materials across scattered channels
- **Actor:** Facilitator
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** Mirror of Step 5. The facilitator is the human aggregation layer. With 1-2 clients, this is manageable. At 5-10 concurrent clients, it is unsustainable. This is the core scaling bottleneck.
- **Failure mode:** Facilitator misses a file, mixes up client materials, or spends disproportionate time on logistics instead of value-adding work.

### Step 7: Facilitator DOWNLOADS and CONSOLIDATES files
- **Actor:** Facilitator
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** Every file must be downloaded, renamed, organized, and placed into the right client folder. PDFs may need text extraction. Voice memos may need transcription. Links may need scraping. This is pure manual labor with no leverage.
- **Failure mode:** Time sink scales linearly with client count. No delegation possible because the facilitator is the only one who knows the framework.

### Step 8: Client RESPONDS to follow-up questions
- **Actor:** Client
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** The facilitator almost always needs to go back for more. "What's your main competitor?" "Who is your ideal customer?" These follow-ups happen over the same scattered channels, creating another round of the collection problem.
- **Failure mode:** Client is slow to respond, answers are incomplete, the back-and-forth delays the entire timeline.

### Step 9: Facilitator FOLLOWS UP for missing information
- **Actor:** Facilitator
- **Risk:** HIGH
- **Notes:** The facilitator must review what was collected, identify gaps against the framework, and reach out for specific missing pieces. This requires deep knowledge of what the narrative needs.
- **Failure mode:** Facilitator forgets to ask for something critical; gap surfaces during the workshop.

### Step 10: Facilitator ORGANIZES materials into prompt framework
- **Actor:** Facilitator
- **Risk:** HIGH — CRITICAL MOMENT
- **Notes:** This is the intellectual core of the current process. Raw materials must be mapped into the structured framework of prompts that generates the market narrative. This is where the facilitator's expertise lives — knowing which material feeds which prompt, what to emphasize, what to discard.
- **Failure mode:** Framework mapping is inconsistent across clients. Quality depends entirely on the facilitator's time and attention for that specific client.

### Step 11: Facilitator RUNS AI prompts with structured input
- **Actor:** Facilitator + System/AI
- **Risk:** Medium
- **Notes:** Once the materials are organized into the framework, the AI prompt execution is relatively mechanical. The hard work was getting to this point.

### Step 12: System GENERATES market narrative draft
- **Actor:** System/AI
- **Risk:** Medium
- **Notes:** AI output quality depends on input quality. Garbage in, garbage out. If Steps 4-10 were incomplete, the narrative will have gaps.

### Step 13: Facilitator REVIEWS and EDITS the narrative
- **Actor:** Facilitator
- **Risk:** HIGH
- **Notes:** The facilitator must verify accuracy, fill gaps with their own knowledge, and ensure the narrative will work in the workshop context. Currently this is where the facilitator catches errors from upstream.
- **Failure mode:** Review becomes a rewrite. The "automation" didn't actually save time.

### Step 14: Facilitator FINALIZES narrative
- **Actor:** Facilitator
- **Risk:** Low
- **Notes:** Final polish and formatting. Low risk if Steps 1-13 went well.

### Step 15: Workshop CONSUMES narrative in Session 1 — GOAL
- **Actor:** Workshop
- **Risk:** Low (if upstream is solid)
- **Notes:** This is the goal state. A complete, accurate, high-quality market narrative arrives at the workshop ready to use. The quality of this output is entirely determined by the quality of the intake process.

---

## Critical Path Summary

The highest concentration of risk sits in **Steps 4 through 10** — the span from client gathering materials through facilitator organizing them into the prompt framework. This is the "messy middle" where:

- Multiple channels create fragmentation (Steps 5-6)
- Manual consolidation creates a scaling bottleneck (Step 7)
- Follow-up cycles create delays (Steps 8-9)
- Framework mapping requires expertise that can't be delegated (Step 10)

**The goal on the right:** A market narrative that is complete, accurate, and structured — ready to power Workshop Session 1.

**The bottleneck in the middle:** Everything between "client has materials" and "materials are organized into the prompt framework" is manual, fragmented, and person-dependent.
