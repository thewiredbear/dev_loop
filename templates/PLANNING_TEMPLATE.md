# {{PROJECT_NAME}} — Planning Document

**Status:** {{STATUS}}  <!-- e.g. DRAFT | IN REVIEW | APPROVED -->
**Date:** {{DATE}}
**Version:** {{VERSION}}
**Author(s):** {{AUTHORS}}

---

## 1. Requirements

| # | Requirement | Source | Confidence | Notes |
|---|-------------|--------|------------|-------|
| R1 | {{requirement}} | {{stakeholder}} | HIGH / MED / LOW | {{notes}} |
| R2 | | | | |
| R3 | | | | |

> Add rows as needed. Confidence reflects how well-understood the requirement is, not priority.

---

## 2. System Architecture

```
{{REPLACE THIS BLOCK WITH ASCII ARCHITECTURE DIAGRAM}}

Example placeholder:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  {{Layer A}} │────▶│  {{Layer B}} │────▶│  {{Layer C}} │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Key design decisions:**
- {{decision 1}}
- {{decision 2}}

---

## 3. {{TECHNICAL_SECTION_1}}

> Replace this heading with the first major technical area (e.g. "Data Model", "API Design", "Auth").

{{Describe the approach, data structures, contracts, or constraints for this area.}}

---

## 4. {{TECHNICAL_SECTION_2}}

{{Repeat section 3 pattern for each additional technical area.}}

---

## Open Questions

> Use the following callout prefixes consistently:
> - **OPEN QUESTION** — unanswered, needs research or stakeholder input
> - **RESOLVED** — answered; include the answer inline
> - **DECISION NEEDED** — requires a human/team decision before work can proceed
> - **ASSUMPTION** — proceeding on this basis; flag if wrong

- **OPEN QUESTION:** {{question}}
- **RESOLVED:** {{question}} → {{answer}}
- **DECISION NEEDED:** {{question}}
- **ASSUMPTION:** {{assumption}}

---

## 5. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| {{risk description}} | HIGH / MED / LOW | HIGH / MED / LOW | {{mitigation strategy}} |

---

## 6. MVP Scope

### In Scope
- {{feature or capability}}
- {{feature or capability}}

### Deferred (Post-MVP)
- {{feature or capability}} — *reason for deferral*
- {{feature or capability}} — *reason for deferral*

---

## 7. Milestones

| Milestone | Description | Target Date | Owner | Done? |
|-----------|-------------|-------------|-------|-------|
| M1 | {{milestone}} | {{date}} | {{owner}} | [ ] |
| M2 | | | | [ ] |
| M3 | | | | [ ] |

---

*Last updated: {{DATE}}*
