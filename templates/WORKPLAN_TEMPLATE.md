# {{PROJECT_NAME}} — Work Plan

**Version:** {{VERSION}}  |  **Status:** {{STATUS}}

---

## Execution Model

Each phase follows this fixed loop:

```
BUILDER subagent → run tests → REVIEWER subagent → update DEVLOG → next phase
```

1. **Builder** — implements the phase TODO list using the DEVPLAN spec.
2. **Test run** — automated suite executed; output captured.
3. **Reviewer** — audits code quality, correctness, and test coverage.
4. **Log** — DEVLOG updated with cycle entry, pass/fail counts, decisions.
5. **Gate** — only advance to the next phase if all exit criteria are met.

---

## Phase-to-Skill Mapping

| Phase | Builder Skill | Reviewer Skill | Parallel? | Notes |
|-------|--------------|----------------|-----------|-------|
| Phase 1 | `{{skill}}` | `{{skill}}` | No | {{note}} |
| Phase 2 | `{{skill}}` | `{{skill}}` | No | {{note}} |
| Phase N | `{{skill}}` | `{{skill}}` | No | {{note}} |

> Mark **Parallel: Yes** only when phases have zero shared file dependencies.

---

## Parallel vs Sequential Execution

- **Sequential (default):** Phases that touch overlapping files or share state must run one at a time.
- **Parallel (opt-in):** Independent sub-tasks within a phase (e.g. writing two unrelated test files) may run in parallel within a single subagent call.
- Never parallelize across phase boundaries.

---

## Review Gate

Before marking a phase complete, the reviewer subagent must confirm:

- [ ] All phase test cases pass.
- [ ] No tests from prior phases are newly failing.
- [ ] No hardcoded secrets, credentials, or environment-specific paths.
- [ ] Code follows project conventions (see CLAUDE.md).
- [ ] DEVLOG entry written.

If any item fails, the builder re-enters the phase. The phase number does not increment.

---

## Skill Usage Summary

| Skill | Purpose | Model |
|-------|---------|-------|
| `{{skill_name}}` | {{description}} | {{sonnet / opus}} |
| `{{skill_name}}` | {{description}} | {{sonnet / opus}} |

---

## Execution Order Diagram

```
{{REPLACE WITH EXECUTION ORDER DIAGRAM}}

Example:
[Phase 1] ──▶ [Phase 2] ──▶ [Phase 3]
                                │
                         [Phase 4a] ──▶ [Phase 5]
                         [Phase 4b] ──▶
```

---

*Last updated: {{DATE}}*
