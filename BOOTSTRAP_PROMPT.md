# BOOTSTRAP_PROMPT.md
# The Universal Project Kickoff Prompt

This file contains the single prompt you paste into a fresh Claude Code chat to bootstrap any project from scratch using the full automated workflow framework.

## How to use

1. Open a fresh Claude Code chat in your project folder (or an empty folder)
2. Copy everything below the `---` separator
3. Paste it into the chat
4. Describe your project idea when asked
5. Answer the initial clarifying questions (5-7 max)
6. Let the pipeline run

The framework will handle everything else: research, planning, multi-perspective review, alignment, spec creation, automation file setup, and handoff to the development cycle.

---

## FULL VERSION (recommended for new projects)

Copy everything below this line into a fresh Claude Code chat:

---

You are about to bootstrap a complete, spec-driven software project from scratch using an automated multi-phase workflow. Your job is to guide the process from raw idea to a fully planned, automation-ready codebase.

This pipeline has 8 phases. Some phases run autonomously. Some require user input. You will be told clearly which is which at each step.

Skills repo (default): https://github.com/jeffallan/claude-skills
If the user specifies a different skills repo, use that instead.

---

# PHASE A: UNDERSTAND THE PROJECT

Ask the user to describe their project idea. Be warm but brief. Then ask up to 7 clarifying questions — no more — chosen from this list based on what the description leaves ambiguous:

1. What is the core problem this solves, and who are the primary users?
2. What is your preferred tech stack, or should I recommend one?
3. What is your target deployment environment? (local, VPS, cloud provider, serverless, mobile, etc.)
4. Are there any external APIs, services, or data sources this must integrate with?
5. What is the rough timeline and team size? (solo weekend project vs. 3-month team build)
6. Is there any existing infrastructure, codebase, or database this must connect to?
7. Are there hard constraints I should know about? (budget, compliance, language, framework, no-cloud, etc.)

Do not ask all 7 if the description answers some of them already. Keep it conversational. After the user answers, confirm your understanding with a 3-sentence project summary and ask: "Does this capture it correctly?"

Do not proceed to Phase B until the user confirms.

---

# PHASE B: SKILL ACQUISITION

Once the project is confirmed, spawn a subagent (model: sonnet) with this task:

> You are a skill acquisition agent. Your job is to visit the skills repository, evaluate each available skill against the project description below, and download the relevant skill files into the current project directory.
>
> Skills repo: [INSERT SKILLS REPO URL]
> Project description: [INSERT CONFIRMED PROJECT SUMMARY]
>
> Steps:
> 1. Fetch the skills repo README or index to get the list of available skills. If the repo is on GitHub, use the GitHub API or browse the raw file list.
> 2. For each skill file found, evaluate: does this skill apply to this project? Score it: ESSENTIAL / USEFUL / NOT NEEDED.
> 3. Download all ESSENTIAL and USEFUL skill files into the current project directory as `[skill-name]-skill.md`.
> 4. For each downloaded skill, write a one-line explanation: what it is, why it was selected, and which phase of development it will be used in.
> 5. Categorize all downloaded skills into: Planning, Backend, Frontend, Security, Quality/DevOps.
> 6. Output a skill manifest in this format:
>
> ## Skill Manifest
> ### Planning
> - skill-name.md — [what/why/when]
> ### Backend
> - skill-name.md — [what/why/when]
> ### Frontend
> - skill-name.md — [what/why/when]
> ### Security
> - skill-name.md — [what/why/when]
> ### Quality/DevOps
> - skill-name.md — [what/why/when]

Present the skill manifest to the user. Let them know which skills were acquired. Then proceed automatically to Phase C — no user input needed.

---

# PHASE C: PLANNING & RESEARCH

Spawn a subagent (model: sonnet) with this task:

> You are a planning and research agent. Your job is to produce a thorough PLANNING.md for the project described below.
>
> Project description: [INSERT CONFIRMED PROJECT SUMMARY + USER ANSWERS]
>
> Steps:
> 1. Research the external APIs, libraries, and services mentioned in the project. For each:
>    - Find the real base URL and authentication method (OAuth2, API key, JWT, etc.)
>    - Document rate limits if publicly available
>    - Note any known limitations, quirks, or gotchas
>    - Identify which SDK or library to use (prefer official)
> 2. Research the recommended architectural patterns for this type of project.
> 3. Identify the top 3 risks or unknowns that could derail the project.
> 4. Write PLANNING.md with these sections:
>    - Requirements (aligned): what the system must do, from user answers
>    - System Architecture: components, data flow, service boundaries
>    - External Integrations: APIs, services, auth methods, rate limits, SDKs
>    - Tech Stack: recommended stack with justification for each choice
>    - Risks & Open Questions: top risks and what needs to be decided
>    - MVP Scope: what is in and out of scope for the first version
>
> Be specific. Include real API endpoint paths, real auth flows, real library names and versions. Do not use vague placeholders.

After the subagent finishes, read PLANNING.md and present a 5-bullet summary to the user. Tell them Phase D is starting. No user input needed yet.

---

# PHASE D: MULTI-PERSPECTIVE REVIEW

Spawn 5 subagents IN PARALLEL (model: sonnet each). Each subagent reviews PLANNING.md from a single domain perspective. Load each subagent with its category's skill files (from the skill manifest produced in Phase B).

Subagent prompts:

**Subagent 1 — Planning/Architecture Perspective:**
> You are a planning and architecture reviewer. Read PLANNING.md. From the perspective of system design, architecture, and requirements clarity: identify the 2 most critical questions or risks that must be resolved before development begins. Each question should be specific, actionable, and concern something that would cause rework if answered wrong later. Output exactly 2 questions, numbered.

**Subagent 2 — Backend Perspective:**
> You are a backend engineering reviewer. Read PLANNING.md. From the perspective of backend implementation, APIs, data modeling, and infrastructure: identify the 2 most critical questions or risks that must be resolved before development begins. Each question should be specific and actionable. Output exactly 2 questions, numbered.

**Subagent 3 — Frontend Perspective:**
> You are a frontend engineering reviewer. Read PLANNING.md. From the perspective of UI/UX, frontend architecture, browser compatibility, and user flows: identify the 2 most critical questions or risks that must be resolved before development begins. Each question should be specific and actionable. Output exactly 2 questions, numbered.

**Subagent 4 — Security Perspective:**
> You are a security reviewer. Read PLANNING.md. From the perspective of authentication, authorization, data protection, attack surface, and compliance: identify the 2 most critical questions or risks that must be resolved before development begins. Each question should be specific and actionable. Output exactly 2 questions, numbered.

**Subagent 5 — Quality/DevOps Perspective:**
> You are a quality and DevOps reviewer. Read PLANNING.md. From the perspective of testability, CI/CD, deployment, observability, and operational reliability: identify the 2 most critical questions or risks that must be resolved before development begins. Each question should be specific and actionable. Output exactly 2 questions, numbered.

Collect all 10 questions. Deduplicate if two subagents raised the same question. Present the final list to the user, grouped by perspective, numbered 1–10.

Tell the user: "Please answer as many of these as you can. For any you want to skip or decide later, just say 'skip' or 'TBD'. Your answers will shape the final spec."

---

# PHASE E: ALIGNMENT

Wait for the user to answer the questions.

Once answers are received, spawn 5 subagents IN PARALLEL (model: sonnet each). Each receives the original PLANNING.md, the 10 questions, and the user's answers. Each subagent analyzes the impact of the answers on their domain and produces a list of changes to make to PLANNING.md.

Then spawn a reconciliation subagent (model: sonnet):

> You are a reconciliation agent. You have received impact analyses from 5 domain reviewers. Your job is to update PLANNING.md to incorporate all resolved decisions from the user's answers, resolve any conflicts between domain recommendations, and mark all resolved questions as RESOLVED with the decision recorded inline. Also identify: did any new questions emerge from the answers that must be resolved before development? If yes, list them. If no, state "No new questions."

If new questions emerge: present them to the user. Repeat the answer → reconcile loop until no new questions remain.

When all questions are resolved, tell the user: "All questions resolved. PLANNING.md is locked. Proceeding to spec and automation setup."

---

# PHASE F: SPEC & AUTOMATION FILES

Create the following files. Use subagents (sonnet) as needed. These files drive the entire development cycle.

**1. DEVPLAN.md** — The spec-driven development plan
Create a phased plan where:
- Each phase delivers a vertical slice of working, tested behavior
- Tests are written FIRST (failing tests are specifications)
- Each phase lists: deliverables, named test cases (20-40 per phase), dependencies, and skill files to load
- Phases are ordered: scaffold → core services → integrations → API layer → frontend → security → testing → deploy
- Include a regression rule: tests from Phase N must stay green in Phase N+1
- Include explicit gate conditions (e.g., "All X tests green before next phase begins")

**2. WORKPLAN.md** — Skill-to-phase orchestration map
For each phase in DEVPLAN.md:
- List the primary and secondary skill files to load into the builder subagent
- One-line explanation of why each skill is relevant to this phase
- Note which phases can run in parallel vs. must be sequential

**3. CLAUDE.md** — Auto-loaded project context
This file is automatically read by Claude Code on every session. It must contain:
- Project overview (2-3 sentences)
- Tech stack
- Key files and what they are
- Development rules (tests first, no skipping phases, regression guard)
- The automated dev cycle protocol (pre-flight, execution, post-flight)
- Subagent model guidance (sonnet for builders/reviewers, reasoning model only for complex decisions)
- Code organization rules (where files go, naming conventions)

**4. DEVLOG.md** — Progress tracker
Initialize with:
- Current status: Cycle 0, Phase 0, not started
- A table of all phases with columns: Phase | Status | Tests Written | Tests Passing | Notes
- All phases set to NOT STARTED

**5. KICKOFF_PROMPT.md** — Dev cycle trigger
A copy-paste prompt (everything below a separator line) that the user pastes into a fresh Claude Code chat to start or resume a dev cycle. It must:
- Instruct Claude to read CLAUDE.md, DEVLOG.md, DEVPLAN.md, WORKPLAN.md
- Check DEVLOG.md to find the current phase and resume from there
- Ask any pre-flight questions ONCE before going autonomous
- Execute the full builder → test → fix → reviewer → log → next phase loop
- After all phases complete: run full test suite, update DEVLOG.md, print summary, STOP

After creating all 5 files, tell the user what was created and move to Phase G.

---

# PHASE G: POC DECISION

Ask the user:

"Your project spec and automation files are ready. Before starting the full build, do you want:

**Option A — Local POC first:** I'll create a `poc/` folder with simplified versions of all planning and automation files scoped to a minimal proof of concept (core happy path only, no edge cases, no security hardening). This lets you validate the core idea quickly before committing to the full build.

**Option B — Go straight to MVP:** Skip the POC and begin the full development cycle immediately using the files just created.

Which would you prefer? (A or B)"

If the user chooses A (POC):
- Create a `poc/` folder
- Inside it, create simplified versions of DEVPLAN.md, WORKPLAN.md, CLAUDE.md, DEVLOG.md, and KICKOFF_PROMPT.md
- The POC plan should cover only the core happy path: the single most important user flow that proves the concept works
- Scope: 2-3 phases maximum, no security hardening phase, no load testing phase
- Tell the user: "POC files are in poc/. To start the POC build, paste KICKOFF_PROMPT.md from the poc/ folder into a fresh Claude Code chat."

If the user chooses B (MVP):
- Tell the user: "Ready to build. See Phase H for how to start."

---

# PHASE H: HANDOFF

Print a summary in this format:

---
## Bootstrap Complete

### What was created:
- PLANNING.md — Full project spec, architecture, all decisions recorded
- DEVPLAN.md — [N] phases, [N] named test cases across all phases
- WORKPLAN.md — Skill-to-phase map for all [N] phases
- CLAUDE.md — Auto-loaded project context for every Claude Code session
- DEVLOG.md — Progress tracker (all phases at NOT STARTED)
- KICKOFF_PROMPT.md — Dev cycle trigger prompt
[- poc/ — POC versions of all files (if POC was chosen)]

### Skill files downloaded: [N]
[List them]

### How to start your first dev cycle:
1. Open a new Claude Code chat in this project folder
2. Open KICKOFF_PROMPT.md
3. Copy everything below the separator line
4. Paste it into the new chat
5. Answer any pre-flight questions Claude asks
6. The build runs autonomously from there

### How the review loop works:
After each phase: builder writes tests first, then code. Tests run. Reviewer checks code quality. If anything fails, a fix agent resolves it. DEVLOG.md is updated. Next phase begins.

After all phases: Claude stops and waits for your review. Check DEVLOG.md for results. Then decide: ship it, fix something, or start the next iteration.

### Skills repo used: [URL]
### Total questions resolved: [N]
---

The bootstrap is complete. You are ready to build.

---

---

## QUICK START VERSION (minimal, for experienced users)

Copy everything below this line into a fresh Claude Code chat:

---

Bootstrap a new software project using the full automated workflow.

Skills repo: https://github.com/jeffallan/claude-skills

Ask me up to 7 clarifying questions about my project, then run autonomously through: skill acquisition from the repo, research and planning (PLANNING.md), 5-perspective review with 10 questions for me to answer, alignment until all questions are resolved, and creation of DEVPLAN.md / WORKPLAN.md / CLAUDE.md / DEVLOG.md / KICKOFF_PROMPT.md. Finally ask if I want a POC folder first or go straight to MVP. End with a handoff summary explaining how to start the first dev cycle.

My project idea: [DESCRIBE YOUR PROJECT HERE]
