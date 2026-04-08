# Claude Code Development Workflow Framework

A reusable, project-agnostic framework for running entire software projects through Claude Code using structured automation, subagent orchestration, and test-driven development cycles. Drop it into any project folder, paste a single prompt, and Claude Code handles planning, review, spec generation, and phase-by-phase building — resuming automatically across sessions.

---

## How It Works

The framework runs in four stages:

```
Stage 1: PLAN  →  Stage 2: SPEC  →  Stage 3: POC (optional)  →  Stage 4: BUILD
```

**Stage 1 — PLAN**
Skills are downloaded and evaluated. Claude researches your domain, creates a comprehensive PLANNING.md, then runs 5 parallel domain-specific subagents (Architecture, Backend, Frontend, Security, DevOps) to surface 10 critical questions. You answer them. The plan is reconciled. Repeat until zero open blockers.

**Stage 2 — SPEC**
PLANNING.md becomes the source of truth. Claude generates DEVPLAN.md (phased, test-first development), WORKPLAN.md (skills mapped to phases), CLAUDE.md (auto-loaded project context), DEVLOG.md (session tracker), and KICKOFF_PROMPT.md (the prompt you paste to start building).

**Stage 3 — POC (optional but recommended)**
A simplified, local-only version of the project is built in a `poc/` subfolder. No external dependencies. Proves the core flow works. A LESSONS_LEARNED.md captures insights that feed back into the MVP plan.

**Stage 4 — BUILD**
You paste KICKOFF_PROMPT.md. The agent reads DEVLOG.md, asks any one-time questions, then builds autonomously: tests first, code second, reviewer checks quality, log updated, next phase. Stops when done or blocked. You review, provide feedback, paste the prompt again. Repeat until shipped.

---

## Quick Start

```
1. Create a new project folder
2. Copy the `workflow/` folder into it
3. Open Claude Code in the project folder
4. Paste the contents of BOOTSTRAP_PROMPT.md into Claude Code
5. Describe your project idea when prompted
6. Claude runs the full pipeline
```

That's it. Claude takes it from there.

---

## Files in This Framework

| File | Purpose |
|------|---------|
| `README.md` | You are here — overview and quick start |
| `WORKFLOW.md` | Full methodology reference — read this before customizing |
| `BOOTSTRAP_PROMPT.md` | The prompt that starts the entire pipeline |
| `templates/` | Template files generated during the pipeline (PLANNING, DEVPLAN, WORKPLAN, CLAUDE, DEVLOG, KICKOFF_PROMPT) |

Files Claude generates in your project root during the pipeline:

| File | When Created | Purpose |
|------|-------------|---------|
| `PLANNING.md` | Stage 1 | Architecture, requirements, decisions |
| `DEVPLAN.md` | Stage 2 | Phased dev plan with test cases |
| `WORKPLAN.md` | Stage 2 | Skills-to-phases mapping |
| `CLAUDE.md` | Stage 2 | Auto-loaded project context for Claude Code |
| `DEVLOG.md` | Stage 2 | Progress log across all sessions |
| `KICKOFF_PROMPT.md` | Stage 2 | Prompt to paste at the start of each dev cycle |
| `LESSONS_LEARNED.md` | Stage 3 | POC insights that feed MVP planning |

---

## Requirements

- **Claude Code CLI** — required (this framework is built for it)
- **Git** — optional but strongly recommended for checkpointing between phases
- **Node.js** — for most frontend or fullstack projects
- **Python** — for most backend or API projects
- Other dependencies vary by project type

---

## The Dev Cycle Loop

Each build cycle follows this pattern:

```
Paste KICKOFF_PROMPT.md
       |
       v
Agent reads DEVLOG.md → finds where to resume
       |
       v
Agent asks one-time questions (architecture choices, env vars, etc.)
       |
       v
You answer
       |
       v
Agent builds autonomously:
  [ Write tests ] → [ Write code ] → [ Run tests ] → [ Fix failures (max 3x) ]
  → [ Reviewer checks quality ] → [ Update DEVLOG.md ] → [ Next phase ]
       |
       v
Agent stops (done, blocked, or context limit approaching)
       |
       v
You review output
       |
       v
Paste KICKOFF_PROMPT.md again → repeat
```

Each cycle typically covers 2-4 phases. DEVLOG.md remembers everything between sessions.

---

## Tips

- **Don't try to finish in one cycle.** The system is designed for incremental, resumable builds. Let it stop naturally.
- **Do the POC first.** It takes an hour and saves days. The lessons learned will reshape your MVP plan in ways you can't anticipate upfront.
- **Answer the Stage 1 questions seriously.** Those 10 questions unlock real architectural decisions. Vague answers produce vague plans.
- **DEVLOG.md is your memory.** If something goes wrong mid-session, you can edit DEVLOG.md manually to correct the record before the next cycle.
- **Each phase is a unit.** If a phase fails repeatedly, stop the cycle, investigate manually, then restart. Don't let broken tests accumulate.
- **Skills matter.** The framework pulls domain-specific skill files. Review what was downloaded in Stage 1 — they shape how Claude approaches each phase.
- **CLAUDE.md is auto-loaded.** Claude Code reads it at session start. Keep it current. If the project context drifts, update CLAUDE.md.
