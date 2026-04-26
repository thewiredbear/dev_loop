# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A documentation-only framework that other projects copy in to drive Claude Code through a structured plan → spec → (POC) → build pipeline. There is no code, no build, no test runner, and no package manager here — every change is to Markdown.

The framework's "execution" happens when a user pastes `BOOTSTRAP_PROMPT.md` into a fresh Claude Code chat in some other project. That prompt instructs Claude Code to spawn subagents, generate per-project artifacts (PLANNING.md, DEVPLAN.md, WORKPLAN.md, CLAUDE.md, DEVLOG.md, KICKOFF_PROMPT.md, optionally LESSONS_LEARNED.md), and then run resumable build cycles against them.

## Repository layout and how the pieces fit together

Three top-level documents form a layered contract — change one and the others usually need to change too:

- `README.md` — user-facing overview and quick start. Describes the four stages and the artifacts they produce.
- `WORKFLOW.md` — the canonical methodology reference. Each of the 4 stages (PLAN / SPEC / POC / BUILD) is described in full, including the 5 review-subagent personas, the test-first build loop, blocker handling, and troubleshooting recipes.
- `BOOTSTRAP_PROMPT.md` — the actual prompt pasted into Claude Code. It implements WORKFLOW.md as 8 phases (A–H) with concrete subagent prompts. The default skills repo is `https://github.com/jeffallan/claude-skills`. Has both a "FULL VERSION" and a "QUICK START VERSION" below separator lines that users copy from.

`templates/` holds the seed files Claude is supposed to produce during the pipeline. Filenames map directly to artifacts referenced by README/WORKFLOW/BOOTSTRAP:

| Template | Produced in stage / phase |
|---|---|
| `PLANNING_TEMPLATE.md` | Stage 1 / Phase C |
| `DEVPLAN_TEMPLATE.md`, `WORKPLAN_TEMPLATE.md`, `CLAUDE_TEMPLATE.md`, `DEVLOG_TEMPLATE.md`, `KICKOFF_PROMPT_TEMPLATE.md` | Stage 2 / Phase F |
| `POC_SETUP_TEMPLATE.md`, `LESSONS_LEARNED_TEMPLATE.md` | Stage 3 / Phase G |

Templates use `{{PLACEHOLDER}}` syntax for fields the bootstrap pipeline fills in. `templates/test.md` is a stray scratch file, not part of the framework.

## Editing rules specific to this repo

- **Keep the three layers consistent.** If you add/rename a stage, artifact, or subagent in one of `README.md`, `WORKFLOW.md`, `BOOTSTRAP_PROMPT.md`, update the others. The 4 stages (README/WORKFLOW) and 8 phases A–H (BOOTSTRAP_PROMPT) describe the same pipeline at different granularities.
- **Template filenames are part of the contract.** `BOOTSTRAP_PROMPT.md` and `WORKFLOW.md` reference the artifacts by name (e.g. `DEVPLAN.md`, `KICKOFF_PROMPT.md`). When you add or rename a template in `templates/`, also update those references.
- **The 5 review personas** (Architecture, Backend, Frontend, Security, Quality/DevOps) and their skill assignments appear in both `WORKFLOW.md` §1.3 and `BOOTSTRAP_PROMPT.md` Phase D. Keep them aligned.
- **Default subagent model in the framework is `sonnet`**, escalating to `opus` only for hard debugging or architectural decisions (see `templates/CLAUDE_TEMPLATE.md`). Reflect this when writing or editing subagent prompts.
- **Test-first is a hard rule of the methodology** — when editing DEVPLAN/WORKPLAN templates or the build-cycle protocol, don't introduce paths that let code be written before tests exist.
