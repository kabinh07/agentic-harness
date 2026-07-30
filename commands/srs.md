---
name: srs
description: Build planning/SRS.md from the approved BRD — functional/non-functional requirements, acceptance criteria, and a traceability matrix back to BRD goals. Questions the user for anything the BRD doesn't pin down.
---

# /agentic-harness:srs

Second planning stage. Turns `planning/BRD.md`'s business goals into
concrete, testable requirements. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

## Gate

`planning.stages.brd.status` must be `approved`. If not, stop and tell the
user to finish `/agentic-harness:brd` first.

## Inputs

Read `planning/BRD.md` in full. If `planning.entry_point == existing-project`,
also dispatch `agentic-harness:codebase-analyst` for the Data model and
External integrations sections — existing systems already have real
interfaces and constraints an idea-stage SRS wouldn't need to invent.

## Question bank

- Per BRD goal: what does "done" concretely mean — what would you check to
  confirm it's met?
- Performance: latency/throughput budgets, expected load, growth rate.
- Auth model: who authenticates how; roles and permissions, if any beyond
  the BRD's personas.
- Data: volume today and expected growth; retention and deletion rules if
  not already pinned down in the BRD.
- Integrations: contract shape for each external system named in the BRD
  (sync/async, expected failure modes, rate limits).
- Failure handling: what should happen when each critical path fails —
  retry, fallback, user-visible error, silent degrade.
- Observability: what needs to be logged/monitored/alerted for this to be
  operable.
- Deployment target: where this runs, if that constrains anything
  (offline-capable, on-prem, specific cloud, edge).

## Output — `planning/SRS.md`

IEEE-830-shaped:

```markdown
# Software Requirements Specification

## 1. Introduction
Purpose, scope, references to planning/BRD.md.

## 2. Overall Description
User classes, operating environment, dependencies/assumptions.

## 3. External Interfaces
APIs, UIs (referenced, not designed here), hardware/other-system interfaces.

## 4. Functional Requirements
### FR-001: <title>
- **Priority:** must/should/could
- **Serves goal:** <BRD goal>
- **Description:** ...
- **Acceptance criteria:** Given/When/Then, one or more per FR

(repeat per requirement)

## 5. Non-Functional Requirements
### NFR-001: <title>
- **Category:** performance | security | reliability | scalability |
  observability | maintainability | compliance | accessibility
- **Serves goal:** <BRD goal>
- **Requirement:** measurable statement

## 6. Data Requirements
Entities, retention, migration notes.

## 7. Traceability Matrix
| Requirement | BRD Goal |
|---|---|
| FR-001 | ... |

## Assumptions (unvalidated)
## Open Questions
```

## Orphan check

Every BRD goal should be served by at least one FR or NFR. A goal with
nothing tracing to it is reported to the user, not silently left — either
the SRS is missing a requirement or the goal needs revisiting in the BRD.

## Approval gate

Per `planning-protocol.md`. On approval, set
`planning.stages.srs.status: approved` + `approved_on`.

## Report (≤6 lines)

```
planning/SRS.md: <written/updated>
FRs: N, NFRs: N
Traceability: N/N BRD goals covered (or list gaps)
Open questions: N
Status: draft/approved
Next: /agentic-harness:design (if has_ui) or /agentic-harness:features
```
