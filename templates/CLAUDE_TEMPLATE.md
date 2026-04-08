# {{PROJECT_NAME}} — Claude Context

> This file is auto-loaded by Claude Code at session start. Keep it current.

---

## Project Overview

{{One paragraph describing what this project does, who uses it, and why it exists.}}

---

## Key Files to Read Before Acting

| File | Purpose |
|------|---------|
| `DEVLOG.md` | Current phase, cycle number, blocking issues |
| `DEVPLAN.md` | Phase specs, test cases, TODO lists |
| `WORKPLAN.md` | Skill routing, execution model, review gates |
| `PLANNING.md` | Architecture, requirements, open questions |
| `{{other key file}}` | {{purpose}} |

**Always read DEVLOG.md first.** It tells you where you are.

---

## Tech Stack

- **Language:** {{e.g. Python 3.12 / TypeScript 5 / Go 1.23}}
- **Framework:** {{e.g. FastAPI / Next.js / Gin}}
- **Database:** {{e.g. PostgreSQL / SQLite / None}}
- **Test runner:** {{e.g. pytest / jest / go test}}
- **Package manager:** {{e.g. uv / npm / go mod}}
- **Key dependencies:** {{list the 3-5 most important libraries}}

---

## Development Rules

1. **Tests first.** Write failing tests before implementation. Never skip this step.
2. **Phase-by-phase.** Complete and verify one phase before starting the next.
3. **No regressions.** Run the full suite after every phase. Zero tolerance for new failures.
4. **No secrets in code.** Use environment variables or a secrets manager. Never hardcode.
5. **Follow existing conventions.** Match the style, structure, and naming already in the codebase.
6. **Small commits.** One logical change per commit. Commit messages explain the why.
7. **Ask before deviating.** If the spec is ambiguous or wrong, flag it — don't silently improvise.

---

## Automated Dev Cycle Protocol

### Pre-flight
1. Read `DEVLOG.md` to find the current cycle and phase.
2. Read the relevant phase block in `DEVPLAN.md`.
3. If blockers exist in DEVLOG, surface them before proceeding. Ask clarifying questions.

### Execution
1. Spawn a **builder subagent** to implement the phase TODO list.
2. Run the test suite. Capture pass/fail counts.
3. Spawn a **reviewer subagent** to audit the implementation against the review gate checklist.
4. If the gate fails, re-enter the builder step. Do not advance the phase counter.

### Post-flight
1. Run the full regression test suite.
2. Update `DEVLOG.md` with a new cycle entry (phase, date, test counts, decisions, blockers).
3. Print a summary: phase completed, tests passed/failed, next phase.
4. Stop. Do not start the next phase without a new kickoff.

---

## Subagent Model Selection Guide

| Task | Model | Reason |
|------|-------|--------|
| Writing implementation code | `sonnet` | Fast, cost-efficient for routine coding |
| Writing test cases | `sonnet` | Straightforward and repetitive |
| Debugging a hard failure | `opus` | Stronger reasoning for complex root causes |
| Architectural decisions | `opus` | Higher-stakes, benefits from deeper analysis |
| Reviewing code quality | `sonnet` | Sufficient for checklist-based review |

> Default to `sonnet`. Escalate to `opus` only when explicitly warranted.

---

*Last updated: {{DATE}}*
