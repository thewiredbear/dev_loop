# Dev Cycle Kickoff Prompt

> Copy this prompt verbatim to start a new dev cycle. Fill in {{placeholders}} before sending.

---

You are starting dev cycle {{CYCLE_NUMBER}} for **{{PROJECT_NAME}}**.

## Step 1 — Read context files (in this order)

1. `CLAUDE.md` — project overview, rules, tech stack
2. `DEVLOG.md` — current cycle, phase, and any blockers
3. `DEVPLAN.md` — phase specs, test cases, TODO lists
4. `WORKPLAN.md` — skill routing and execution model

Do not begin any implementation until you have read all four files.

## Step 2 — Pre-flight

- Check `DEVLOG.md` for the current phase and status.
- If status is BLOCKED, surface the blocker and ask for resolution before proceeding.
- If status is IN PROGRESS, resume from where the last cycle left off.
- If status is COMPLETE (all phases done), report that and stop.
- Ask any clarifying questions now. Do not ask mid-execution.

## Step 3 — Execution

For each phase (starting from the current phase in DEVLOG):

1. Spawn a **builder subagent** (`sonnet`) to implement the phase TODO list from DEVPLAN.
2. Run the test suite. Capture and report pass/fail/skip counts.
3. Spawn a **reviewer subagent** (`sonnet`) to check the review gate in WORKPLAN.
4. If the gate fails, loop back to the builder. Do not advance.
5. When the gate passes, mark the phase complete and continue to the next phase.

Run **one phase at a time**. Do not batch multiple phases in a single subagent call.

## Step 4 — Post-flight

After the phase (or phases, if the plan allows sequential auto-advance) completes:

1. Run the **full regression test suite** (all phases, not just the current one).
2. Update `DEVLOG.md`:
   - New cycle entry with date, phase, outcome, test counts, decisions, blockers.
   - Update the Phase Completion Log table.
3. Print a plain-text summary:
   - Phase(s) completed this cycle.
   - Total tests: N passed, N failed, N skipped.
   - Next phase (or "All phases complete").
4. **Stop.** Do not begin the next phase without a new kickoff.

## Constraints

- Subagents default to `sonnet`. Use `opus` only for hard debugging.
- Never skip the review gate, even if tests pass.
- Never commit secrets, credentials, or hardcoded environment paths.
- If a requirement is ambiguous, log an ASSUMPTION in DEVLOG and flag it in the summary.

---

*Template version: {{VERSION}}*
