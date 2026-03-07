# Sprint Workspace — Problem Sprint Workflow

> This workspace runs structured design sprints using the shared Skills Library.
> Present a problem, and this workspace facilitates exploration and solution architecture.

## How This Workspace Works

This is a **Sprint-focused workspace**. When you present a problem, the workspace runs the Problem Sprint workflow — a structured process adapted from Jake Knapp's Sprint methodology that takes you from problem framing through solution architecture.

### Quick Start

1. **Present your problem** — describe the challenge, who it affects, and any context
2. **The sprint runs** — agents frame the problem, generate solutions, and pause for your decisions
3. **You decide** — at two critical points, you choose the direction
4. **Output** — a complete sprint package with architecture, storyboard, and HTML visualization

### Running a Sprint

To start a sprint, simply describe your problem. The Facilitator agent will activate and run the full workflow defined in `../skills-library/orchestration/problem-sprint.md`.

You can also explicitly activate the sprint:
```
python ../skills-library/execution/activate_skill.py --agent sprint/facilitator --workspace .
```

## Skills Library

This workspace uses the shared skills library at `../skills-library/`.

**Available commands:**
- List all skills: `python ../skills-library/execution/activate_skill.py --list`
- Activate an agent: `python ../skills-library/execution/activate_skill.py --agent {team/name} --workspace .`
- Ingest reference material: `python ../skills-library/execution/ingest_reference.py --source {path} --domain {slug} --workspace .`

## Sprint Team (Primary Workflow)

| Agent | Role | Skills |
|-------|------|--------|
| `sprint/facilitator` | Manages sprint flow, presents at decision points, handles output | sprint-methodology |
| `sprint/problem-mapper` | Frames the problem: goal, map, HMW notes, target | problem-mapping, sprint-methodology |
| `sprint/solution-designer` | Generates 3-5 distinct solution approaches with research | solution-sketching, sprint-methodology |
| `sprint/decision-architect` | Evaluates options, facilitates user decision, builds storyboard | sprint-decision-making, sprint-methodology |
| `sprint/prototype-builder` | Designs architecture, builds HTML visualization | product-architecture, visual-prototyping |

### Sprint Phases

1. **Setup** — Facilitator creates sprint folder structure
2. **Frame (Monday)** — Problem Mapper frames goal, map, questions, target
3. **User Target Confirmation** — You confirm or redirect the target
4. **Sketch (Tuesday)** — Solution Designer generates 3-5 approaches
5. **Decide (Wednesday)** — Decision Architect evaluates; **you select the direction**
6. **Storyboard** — Decision Architect builds detailed user journey
7. **Architect (Thursday)** — Prototype Builder designs architecture + HTML
8. **Validate (Friday)** — Facilitator assembles and validates output
9. **Approval & Push** — You approve; optional push to GitHub

### Critical Decision Points

- **Phase 3**: Confirm the sprint target (which part of the problem to focus on)
- **Phase 5**: Select the solution direction (sprint HALTS until you decide)

## All Available Teams

While the Sprint team is the primary workflow, all skills library teams are accessible:

| Team | Use Case |
|------|----------|
| `sprint/` | Problem sprints (primary) |
| `development/` | Software development — PM, design, engineering, QA |
| `marketing/` | Content, campaigns, analytics |
| `finance/` | Financial planning, revenue ops |
| `logistics/` | Supply chain, warehouse, fleet |
| `editorial/` | Writing, editing, publishing |
| `video/` | Video production |

Activate any agent: `python ../skills-library/execution/activate_skill.py --agent {team/agent-name} --workspace .`

## Sprint Output Structure

Each sprint creates a folder in `resources/problem-sprints/{slug}/`:

```
resources/problem-sprints/{slug}/
  README.md              # Sprint overview with links to all files
  01-frame/              # Goal, map, HMW notes, target
  02-sketch/             # Research inspiration + solution approaches
  03-decide/             # Decision record + storyboard
  04-architect/          # Architecture document + HTML visualization
  05-validate/           # Learnings and validation
```

## Shared Context (.context/)

The `.context/` directory keeps state across sprints and agents:
- `decisions.json` — Why choices were made (prevents contradictions)
- `artifacts.json` — Registry of all created outputs (prevents duplication)
- `constraints.json` — Active constraints
- `state.json` — Current progress across all active missions

**Every agent checks `.context/state.json` before starting work.**

## Orchestration Patterns

- **Primary**: `../skills-library/orchestration/problem-sprint.md` — Full sprint workflow
- **Supporting**: `../skills-library/orchestration/sprint-cycle.md`, `sprint-planning-workflow.md`

## Mission Templates

- `../skills-library/missions/problem-sprint-template.md` — Structured problem definition for sprints

## Operating Principles

1. **Present the problem clearly** — the quality of output is bounded by the quality of the input
2. **Trust the process** — the sprint phases are designed to prevent premature convergence
3. **Make decisions at decision points** — the sprint halts for your input at critical gates
4. **Quality gates are non-negotiable** — each phase validates before passing to the next
5. **All thinking is documented** — every decision, tradeoff, and rationale is captured

## File Organization

- `resources/problem-sprints/` — Sprint output (one folder per sprint)
- `resources/books/` — Reference materials (e.g., Sprint methodology book)
- `resources/knowledge-base/` — Ingested domain knowledge
- `directives/` — Activated skill directives
- `execution/` — Workspace-specific scripts
- `.context/` — Shared state for multi-agent coordination
- `.tmp/` — Intermediate processing files (never commit)
- `.workflow/` — Project tracking
