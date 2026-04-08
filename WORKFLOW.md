# Workflow Methodology Reference

This document describes each stage of the Claude Code Development Workflow Framework in detail. It is the canonical reference for understanding what happens, why it happens, and how to intervene if something goes wrong.

For a quick start, see README.md. For the prompt that kicks everything off, see BOOTSTRAP_PROMPT.md.

---

## Overview

```
User describes idea
        |
        v
  STAGE 1: PLAN
  - Acquire skills
  - Research + draft PLANNING.md
  - 5-agent review → 10 questions
  - User answers → reconcile plan
        |
        v
  STAGE 2: SPEC
  - DEVPLAN.md (phased, test-first)
  - WORKPLAN.md (skills × phases)
  - Automation files (CLAUDE.md, DEVLOG.md, KICKOFF_PROMPT.md)
        |
        v
  STAGE 3: POC (optional)
  - Local-only simplified build
  - LESSONS_LEARNED.md
        |
        v
  STAGE 4: BUILD
  - Paste KICKOFF_PROMPT.md
  - Autonomous phase-by-phase development
  - Resumable across sessions
        |
        v
    Shipped software
```

---

## Stage 1: PLAN

**Trigger:** User describes a project idea (name, purpose, rough feature list, any known constraints).

**Goal:** Produce a fully aligned PLANNING.md with zero open blockers.

---

### 1.1 Skill Acquisition

Before any planning starts, Claude identifies what domain knowledge it needs and downloads the corresponding skill files.

**What happens:**
- Claude searches a skills repository (the default is github.com/jeffallan/claude-skills, but any source works — a private repo, a local folder, a curated list)
- Each skill is evaluated against the project's specific needs: does this project require auth? Then download the security skill. Does it have a REST API? Download the API design skill.
- Skill `.md` files are saved into the project root
- Claude produces a brief rationale for each skill: what it is, why this project needs it, which phases it will be used in

**Skill categories this framework typically pulls from:**

| Category | Example skills |
|----------|---------------|
| Planning | architecture-designer, feature-forge |
| Backend | fastapi-expert, postgres-pro, api-designer, database-optimizer |
| Frontend | nextjs-developer |
| Security | secure-code-guardian, security-reviewer |
| Quality / DevOps | test-master, code-reviewer, devops-engineer, debugging-wizard, fullstack-guardian |

**Why this step matters:**
Skills are not decoration. They are loaded into subagent context windows during Stage 4 build cycles. A builder subagent loaded with `test-master` and `fastapi-expert` will write better code and more comprehensive tests than a generic agent. Get the skills right in Stage 1.

---

### 1.2 Initial Planning

With skills in hand, Claude researches the project domain and drafts PLANNING.md.

**What happens:**
- Web search for real API documentation, library comparisons, authentication patterns, database design considerations, security best practices — whatever the project actually needs
- No placeholder research. If the project uses SendGrid for email, Claude reads the SendGrid docs. If it uses JWT auth, Claude looks at current best practices for token storage and rotation.
- PLANNING.md is drafted with the following structure:

```
PLANNING.md structure:
- Executive Summary
- Requirements (functional + non-functional, each with a confidence level)
- Architecture Diagram (ASCII)
- Component Deep-Dives (one section per major component)
- API Design (endpoints, payloads, response shapes)
- Data Models (schema, relationships, indexes)
- External Dependencies (libraries, APIs, services — with versions)
- Security Considerations
- Open Questions (anything that needs user input before proceeding)
- Assumptions Made
- Decisions Needed
```

**Confidence levels on requirements** are important. A requirement marked HIGH confidence is something Claude inferred clearly from the project description. LOW confidence means Claude made an assumption and the user should verify it. This gives the user a fast way to spot where to focus during the alignment step.

---

### 1.3 Multi-Perspective Review

Once PLANNING.md exists, 5 parallel subagents review it simultaneously — each from a different domain perspective.

**The 5 reviewers:**

| Agent | Loaded with | Focus |
|-------|------------|-------|
| Architecture & Planning | architecture-designer, feature-forge | System design, scalability, component boundaries |
| Backend & Data | fastapi-expert, postgres-pro, api-designer, database-optimizer | API design, data modeling, performance |
| Frontend & UX | nextjs-developer, fullstack-guardian | User flows, state management, API consumption |
| Security | secure-code-guardian, security-reviewer | Auth, data handling, attack surface |
| Quality & DevOps | test-master, code-reviewer, devops-engineer, debugging-wizard | Testability, CI/CD, deployment, observability |

**Each agent produces exactly 2 questions.** Total: 10 questions.

**What makes a good review question:**
- It is specific to this project, not generic ("How will you handle token expiry for the mobile app when the user is mid-session?" not "What is your auth strategy?")
- It unlocks a real decision that affects architecture or implementation
- It cannot be answered by reading PLANNING.md again — it surfaces something genuinely missing or ambiguous

**What to do if the questions feel too generic:**
Re-run the review agents with a more specific instruction: "Do not ask questions that can be answered by reading the plan. Ask only questions whose answers will change the architecture."

---

### 1.4 Client Alignment

**What happens:**
1. The 10 questions are presented to the user, organized by domain
2. The user answers — answers can be brief; one sentence is enough
3. The same 5 subagents re-run, this time analyzing the impact of the user's answers on the current plan
4. A reconciliation agent reads all 5 impact analyses and updates PLANNING.md accordingly
5. If new open questions emerge from the answers, the cycle repeats

**When to stop:**
The alignment loop ends when PLANNING.md has no remaining open questions, no LOW confidence requirements that the user hasn't addressed, and no decisions marked as "needed before proceeding."

**Practical note:**
Most projects need only one round of alignment. Complex projects with many external integrations may need two. If you find yourself on a third round, the project scope may need to be narrowed before proceeding to Stage 2.

**Output of Stage 1:**
- PLANNING.md — fully aligned, all requirements locked, no open blockers
- All skill `.md` files in the project root

---

## Stage 2: SPEC

**Trigger:** PLANNING.md is complete and aligned. User confirms readiness to proceed.

**Goal:** Convert the plan into executable automation files that allow Claude Code to build autonomously.

---

### 2.1 Dev Plan

DEVPLAN.md is the construction blueprint. It breaks the project into numbered phases, each of which is small enough to build and test in a single subagent context window.

**DEVPLAN.md structure per phase:**

```
## Phase N: [Name]

**WHAT:** List of concrete deliverables (files, endpoints, UI components, migrations)
**WHY:** The business or user value this phase delivers
**HOW:** Implementation approach, key decisions, libraries to use
**DEPENDENCIES:** Which earlier phases must be complete first
**TEST CASES:**
  - test_[name]: what it verifies
  - test_[name]: what it verifies
  - ...
```

**Test-first rule:** Test cases are named and listed in DEVPLAN.md before any code is written. During Stage 4, the builder subagent writes these tests first, then writes code to make them pass. A phase is not complete until all its named tests are green.

**Phase sizing:** Each phase should represent roughly 2-4 hours of focused development. If a phase feels too large, split it. Phases that are too large cause context overflow mid-build and leave things in a broken state.

**Phase ordering principles:**
- Infrastructure before features (DB schema before API endpoints, API before UI)
- Auth before anything that requires auth
- Core data flows before edge cases
- Each phase should be independently testable

---

### 2.2 Work Orchestration

WORKPLAN.md maps the skill files to the phases that need them.

**WORKPLAN.md structure:**

```
## Phase N: [Name]

Builder skills: [skill-a, skill-b]
Reviewer skills: [skill-c]
Parallel with: [Phase M] (if applicable)
Blocked by: [Phase X]
```

**Why this matters:**
During Stage 4, when the builder subagent is spawned for Phase N, it is loaded with exactly the skills listed in WORKPLAN.md for that phase. A database migration phase gets the postgres-pro skill. A React component phase gets the nextjs-developer skill. This keeps subagent context focused and skill-aligned.

**Parallelism:**
Some phases can run in parallel — for example, writing tests for Phase 3 while Phase 2 is being reviewed. WORKPLAN.md marks these explicitly. The orchestrating agent uses this to schedule work efficiently.

---

### 2.3 Automation Files

Three files are generated to enable fully autonomous, resumable dev cycles.

**CLAUDE.md**
Auto-loaded by Claude Code at the start of every session. Contains:
- Project name, description, and tech stack
- Current phase and status
- Dev cycle protocol (how to run a cycle: read DEVLOG, ask questions, build, test, review, log, stop)
- Key architectural decisions (so Claude never re-debates settled questions)
- Where to find everything (which files do what)

CLAUDE.md is the single source of truth for "what is this project and how do I work on it." Keep it updated. If the project pivots, update CLAUDE.md before starting the next cycle.

**DEVLOG.md**
A running log of every dev cycle. Each entry records:
- Date and session number
- Phases attempted
- Phases completed
- Tests written and results
- Blockers encountered
- What to do next

DEVLOG.md is what allows the framework to resume across sessions. The first thing a builder agent does in Stage 4 is read DEVLOG.md to find the current state.

**KICKOFF_PROMPT.md**
The prompt users paste to start a dev cycle. Contains:
- Instruction to read CLAUDE.md and DEVLOG.md first
- Instruction to ask any one-time questions before building
- The build-test-review-log cycle protocol
- Instruction to stop cleanly and summarize when done or blocked
- Any session-specific overrides the user wants to apply

Users do not modify KICKOFF_PROMPT.md between cycles unless they want to change the build protocol. Session-specific instructions (e.g., "focus only on Phase 4 today") can be appended at the bottom.

**Output of Stage 2:**
- DEVPLAN.md
- WORKPLAN.md
- CLAUDE.md
- DEVLOG.md (initialized, ready for first entry)
- KICKOFF_PROMPT.md

---

## Stage 3: POC (Optional)

**Trigger:** User wants to validate the core flow before committing to a full build.

**Goal:** A working, local-only prototype that proves the project is buildable and surfaces real-world complexity before it becomes expensive.

---

**What the POC covers:**
- The single most important user flow (e.g., for an attendance system: clock in, clock out, view report)
- All components are simplified — no external services, no production auth, flat files instead of a database if necessary
- Everything runs locally with a single command

**How it's structured:**
- A `poc/` subfolder is created
- POC gets its own simplified versions of PLANNING.md, DEVPLAN.md, and DEVLOG.md
- Typically 5-6 phases instead of the 10+ in the MVP
- Same test-first protocol applies — even a POC should have tests

**What POC is not:**
- Not a throw-away spike. It should be clean enough to reference.
- Not a full feature build. Scope is ruthlessly limited to the core loop.
- Not optional if you're uncertain about the architecture. If you're unsure whether your chosen stack can do what you need, do the POC.

**LESSONS_LEARNED.md:**
At the end of the POC, a review agent writes LESSONS_LEARNED.md. This document captures:
- What worked as expected
- What was harder than planned
- What assumptions were wrong
- Specific architectural changes to make in the MVP
- Anything that should be added to or removed from DEVPLAN.md

LESSONS_LEARNED.md is read by the Stage 2 agent when generating the MVP DEVPLAN.md. It directly shapes the MVP build plan.

**Output of Stage 3:**
- `poc/` folder with working local prototype
- LESSONS_LEARNED.md with concrete recommendations for MVP

---

## Stage 4: BUILD

**Trigger:** User pastes KICKOFF_PROMPT.md into Claude Code.

**Goal:** Build the project phase by phase, with tests, with reviews, resumable indefinitely.

---

### The Build Cycle Protocol

Each time KICKOFF_PROMPT.md is pasted, the following happens:

**Step 1 — Orient**
Agent reads CLAUDE.md (project context) and DEVLOG.md (current state). Identifies the next phase to build.

**Step 2 — Ask questions**
Before any code is written, the agent asks any one-time questions it needs answered. These are questions that cannot be inferred from existing documentation — environment variables, API keys that aren't in the project, a decision that wasn't resolved in planning. The user answers. This happens once per cycle, at the start.

**Step 3 — Build autonomously**

For each phase in scope for this cycle:

```
1. Spawn builder subagent
   - Loaded with skills from WORKPLAN.md for this phase
   - Reads DEVPLAN.md for this phase's spec

2. Builder writes tests first
   - All test cases named in DEVPLAN.md are implemented
   - Tests are run — they should fail (nothing is built yet)

3. Builder writes code
   - Implements the deliverables listed in DEVPLAN.md
   - Runs tests after each logical unit of code

4. Fix failures
   - If tests fail, builder attempts to fix (max 3 attempts)
   - If still failing after 3 attempts, builder stops and reports the blocker

5. Spawn reviewer subagent
   - Loaded with review skills from WORKPLAN.md
   - Checks code quality, test coverage, security, adherence to plan
   - Produces a brief review with any required changes

6. Builder addresses review feedback

7. Update DEVLOG.md
   - Phase marked complete (or blocked)
   - Tests written and results logged
   - Any decisions made during the build recorded

8. Move to next phase
```

**Step 4 — Stop cleanly**
The agent stops when:
- All phases for this cycle are complete
- A phase is blocked and cannot proceed
- Context limit is approaching

When stopping, the agent writes a session summary to DEVLOG.md and specifies exactly what the next cycle should do.

**Step 5 — User reviews**
User inspects the output: runs the app, reads the tests, checks DEVLOG.md. Provides any feedback. Can edit DEVLOG.md directly to add notes or correct the record.

**Step 6 — Repeat**
Paste KICKOFF_PROMPT.md again. The cycle continues from where it left off.

---

### Handling Blockers

A blocker is any situation where the build cannot proceed without information or intervention from outside the current context:

- A required API key or credential isn't available
- An external service is behaving differently than documented
- A test reveals a fundamental design flaw that requires replanning
- A dependency has breaking changes that affect the architecture

**What to do:**
1. The agent records the blocker in DEVLOG.md with full context
2. The agent stops the cycle
3. The user resolves the blocker (gets the API key, researches the dependency change, updates PLANNING.md if needed)
4. The next cycle resumes with the blocker resolved

Do not push past blockers. A build that continues past a fundamental problem creates technical debt that compounds every subsequent phase.

---

### Context Limit Management

Claude Code has a context window limit. Long cycles will eventually hit it. The framework handles this through DEVLOG.md:

- The agent monitors context usage and stops proactively before the limit is reached
- Everything needed to resume is written to DEVLOG.md before stopping
- The next cycle starts fresh with a clean context, reading DEVLOG.md to reconstruct state

This means each cycle is self-contained. The session history is DEVLOG.md, not the chat transcript.

---

## The Feedback Loop

```
POC Build
    |
    v
LESSONS_LEARNED.md
    |
    v
MVP PLANNING.md refinement (Stage 1 if needed, or directly to Stage 2)
    |
    v
BUILD cycle 1 → DEVLOG.md
    |
    v
User review + feedback → DEVLOG.md notes
    |
    v
BUILD cycle 2 → DEVLOG.md
    |
    v
... (repeat until all phases complete)
    |
    v
Final review → ship
```

Every artifact in this framework is designed to feed forward into the next stage. PLANNING.md feeds DEVPLAN.md. DEVPLAN.md feeds each build cycle. DEVLOG.md feeds the next cycle. LESSONS_LEARNED.md feeds the MVP plan. Nothing is a dead end — everything is input to something downstream.

---

## Troubleshooting

**The review questions feel generic.**
The review agents need to be given the actual PLANNING.md content, not a summary. If questions are generic, re-run the review with more explicit instructions: "Read the full plan and ask questions specific to this project's architecture, not general best-practice questions."

**A phase keeps failing its tests.**
Stop the cycle. Do not attempt a fourth time. Read the test output carefully. The most common causes: the test spec in DEVPLAN.md was ambiguous, the implementation is missing a dependency, or the test itself has a bug. Fix the root cause before resuming.

**DEVLOG.md is getting too long.**
After 10+ cycles, DEVLOG.md can become unwieldy. Create a `DEVLOG_ARCHIVE.md`, move all completed-phase entries there, and keep only the last 2 cycles and the "what to do next" section in the active DEVLOG.md. Update CLAUDE.md to reference both files.

**The plan drifted from PLANNING.md during the build.**
This is normal and expected. At the end of each cycle, the agent notes any deviations in DEVLOG.md. If the drift is significant, update PLANNING.md to reflect the new reality before the next cycle. PLANNING.md should always describe the system as it is, not as it was originally imagined.

**Skills aren't loading correctly.**
Verify the skill `.md` files are in the project root. Skill files must be in the same directory where Claude Code is running. If a subagent is missing domain knowledge it should have, check that the relevant skill is listed in WORKPLAN.md for that phase and that the file exists.

**The agent is re-asking questions that were already answered.**
CLAUDE.md is not being read, or its content is stale. Add the answered questions as explicit statements to CLAUDE.md under a "Settled Decisions" section. KICKOFF_PROMPT.md should always instruct the agent to read CLAUDE.md before asking any questions.
