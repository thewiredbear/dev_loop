# {{PROJECT_NAME}} — Development Plan

**Version:** {{VERSION}}  |  **Status:** {{STATUS}}

---

## Philosophy

- **Tests first.** Every phase begins by writing failing tests that define the target behavior.
- **Phase-by-phase.** No phase begins until the previous phase is green.
- **Regression guard.** The full test suite runs after every phase; no regressions permitted to ship.
- **Spec-driven.** Implementation follows the test spec, not the other way around.

---

## Phase Template (repeat for each phase)

---

## Phase N: {{PHASE_NAME}}

**WHAT:** {{deliverables — what exists at the end of this phase}}
**WHY:** {{business or product value unlocked by this phase}}
**HOW:** {{high-level implementation approach}}

### Test Cases

File: `{{test_file_path}}`

- [ ] `test_{{name}}` — {{description of what is verified}}
- [ ] `test_{{name}}` — {{description of what is verified}}
- [ ] `test_{{name}}` — {{description of what is verified}}

### TODO

- [ ] {{task}}
- [ ] {{task}}
- [ ] {{task}}

**Exit criteria:** All test cases above pass. No regressions in prior phases.

---

<!-- Copy the Phase block above for each additional phase -->

---

## Regression Test Matrix

| Phase | Test File | # Tests | Last Run | Result |
|-------|-----------|---------|----------|--------|
| Phase 1 | `{{path}}` | {{n}} | — | — |
| Phase 2 | `{{path}}` | {{n}} | — | — |
| Phase N | `{{path}}` | {{n}} | — | — |

> "Last Run" and "Result" are filled in by the dev cycle log, not manually.

---

## Test Infrastructure

**Test runner:** {{e.g. pytest / jest / go test}}
**Coverage tool:** {{e.g. coverage.py / nyc}}
**CI trigger:** {{e.g. on push to main / manual}}
**Run command:** `{{command to execute full suite}}`

---

*Last updated: {{DATE}}*
