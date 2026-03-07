# Sprint Learnings — Client Onboarding Intake

**Sprint Date:** 2026-03-07
**Solution:** "The Interview" + "The Head Start" Hybrid (Workshop Intake Engine)

---

## Key Learnings

### 1. The problem is behavioral, not just technical
The initial framing could have led to a purely technical solution (better file aggregation, smarter email parsing). But the sprint revealed that the root cause is behavioral: clients don't know what to send, feel overwhelmed, and procrastinate. The winning solution addresses this by replacing ambiguity with structure — guided questions that extract the same information documents would have contained.

### 2. Web scraping pre-population is a force multiplier, not a standalone solution
The Head Start approach (scrape first, client reviews) tested well as a feature but poorly as a primary mechanism. For thin-web-presence clients, it produces embarrassingly little. As a feature embedded within The Interview's guided flow, it becomes the "we did our homework" moment that elevates the entire experience.

### 3. The highest-quality solution doesn't scale
The Kickoff Call scored a perfect 65/65 — live conversation produces the richest, most nuanced input. But at 45 minutes per client, it reintroduces the facilitator bottleneck. This tension between quality and scalability is the central tradeoff. The Interview is the best compromise: it sacrifices some conversational richness for unlimited scalability.

### 4. Voice input bridges the quality gap
Adding voice recording to the guided flow partially bridges the gap between typed form responses and live conversation. Clients who are "talkers, not writers" can speak their answers. The transcription captures narrative richness that typed answers rarely achieve.

### 5. The facilitator's expertise lives in the question tree
The most important architectural insight: the facilitator isn't replaced — they're embedded. Their expertise is encoded in the branching question logic and framework mapping. This reframe (from "automate the facilitator's work" to "leverage the facilitator's architecture") is critical for adoption.

### 6. Self-segmentation solves the sophistication spectrum
Rather than building one path that works poorly for everyone, the self-segmentation at Step 3 routes clients to appropriate depth and question styles. This was a cleaner solution than trying to detect sophistication automatically.

---

## What the Sprint Did NOT Resolve

### Framework mapping automation (Target B territory)
The sprint focused on intake and collection (Steps 2-6). The downstream challenge — automatically organizing collected materials into the prompt framework (Steps 7-10) — was scoped out. The architecture assumes the framework mapping is embedded in the question tree design, but the actual framework→prompt compilation (Step 10) needs its own sprint.

### AI narrative generation quality
Sprint Question 2 (narrative quality) is partially answered: the intake system produces structured, framework-mapped inputs. But whether the AI narrative generation from those inputs matches the facilitator's manual quality is untested. That lives downstream of this sprint's scope.

### Facilitator adoption in practice
Sprint Question 5 (facilitator trust) is addressed architecturally — the system reflects the facilitator's framework and provides source attribution. But real adoption requires building it, using it with actual clients, and iterating on the gaps.

---

## Risks to Monitor

| Risk | Severity | Mitigation |
|------|----------|------------|
| Clients abandon the interview mid-flow despite pause/resume | High | Monitor completion rates; A/B test flow length and question phrasing |
| Web scraping produces inaccurate pre-population that damages trust | Medium | Clearly label AI-generated content; provide easy correction UX |
| Branching logic becomes unmanageable as framework evolves | Medium | Store branching rules as JSON config (ADR-1); build admin tooling |
| Voice transcription quality insufficient for framework mapping | Low | Use high-quality STT models; allow client to review transcription |
| Facilitator continues manual review of everything despite system output | Medium | Build confidence gradually; show source attribution; track time savings |

---

## Recommended Next Steps

1. **Build a clickable prototype** of the interview flow (Steps 1-8) to test with 2-3 actual clients before engineering
2. **Map the full question tree** — the facilitator needs to articulate their tacit framework knowledge into branching question sequences
3. **Run a Target B sprint** on automated framework mapping (Steps 7-10) once intake is validated
4. **Test web scraping** on 10-15 past client websites to calibrate pre-population quality expectations
5. **Design the facilitator dashboard** (Step 9) with the facilitator as primary user — their workflow determines adoption

---

## Sprint Metadata

| Metric | Value |
|--------|-------|
| Phases completed | 8/8 (Setup through Validate) |
| Solutions generated | 5 |
| Solutions evaluated | 5 |
| User decisions | 2 (Target A confirmed, Solution 1 selected) |
| Architecture components | 8 |
| ADRs written | 5 |
| Storyboard steps | 10 |
| Files produced | 12 (including this one) |
