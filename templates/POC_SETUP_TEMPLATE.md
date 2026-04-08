# POC Setup Guide

> Use this document to scope and structure a Proof of Concept iteration.

---

## When to Create a POC

**Do create a POC when:**
- A core technical assumption is unvalidated (e.g. "will this API do what we need?").
- The integration between two systems is uncertain.
- You need to demo something interactive before committing to full architecture.
- The team disagrees on approach and needs evidence, not debate.

**Skip the POC when:**
- The tech stack is already proven in a prior project.
- All requirements are well-understood and low-risk.
- Time-to-market pressure outweighs learning value.

---

## How to Simplify for a POC

| Production Concern | POC Simplification |
|--------------------|--------------------|
| External services (payments, email, SMS) | Replace with local mocks or stubs |
| Auth / identity | Hardcode a test user; skip OAuth flows |
| Database | Use SQLite or in-memory store |
| Infrastructure (cloud, containers) | Run everything on localhost |
| Error handling | Log and crash; no recovery logic |
| Security hardening | Defer entirely |
| Performance optimization | Defer entirely |
| Multi-environment config | Single `.env.local` file |

> **Rule of thumb:** If a simplification doesn't change what you're trying to prove, make it.

---

## POC Phase Structure

A typical POC has 5–6 phases. Adapt as needed.

| Phase | Name | What It Proves |
|-------|------|----------------|
| 1 | Scaffold & Hello World | The stack runs end-to-end locally |
| 2 | Core Data Model | The primary entities can be stored and retrieved |
| 3 | Key Integration | The most uncertain external dependency works |
| 4 | Primary User Flow | The central use case is demonstrable |
| 5 | Secondary Flow(s) | Supporting flows work well enough to demo |
| 6 | Polish & Demo Prep | The POC can be shown without embarrassment |

> Phases 5 and 6 are optional. Stop when the learning goals are met.

---

## What the POC Must Prove

Before writing a single line of code, answer these questions:

- **Primary hypothesis:** {{What are we trying to validate?}}
- **Success criteria:** {{How will we know the POC succeeded?}}
- **Failure criteria:** {{What result would change our approach?}}
- **Time box:** {{Maximum number of days/hours for the POC}}

---

## What the POC Can Skip

The following are explicitly out of scope for every POC:

- Production error handling and recovery
- Security review and hardening
- Performance benchmarking
- Automated CI/CD
- Multi-environment configuration
- Accessibility compliance
- Comprehensive test coverage (smoke tests only)

---

## How LESSONS_LEARNED Feeds into MVP

After the POC is complete:

1. Fill out `LESSONS_LEARNED.md` for this iteration.
2. Review the "Recommendations for Next Iteration" table.
3. For each HIGH-priority recommendation, create or update a requirement in `PLANNING.md`.
4. Update the Risk Register in `PLANNING.md` with any risks that materialized.
5. Revise the MVP Scope section — promote, demote, or remove items based on evidence.
6. Draft a new `DEVPLAN.md` from scratch using the updated planning doc.

> The POC is only valuable if its learnings change something in the plan.

---

*Last updated: {{DATE}}*
