---
name: brd
description: Build planning/BRD.md by questioning the user from an idea, existing codebase, or raw material in planning/input/ — never fabricates business logic, asks through edge cases and open questions.
---

# /agentic-harness:brd

First planning stage. Produces `planning/BRD.md`: the business requirements
document every later stage traces back to. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md` for questioning, the
approval gate, and bookkeeping.

## Sources, in priority order

1. **Command args** — if the user gave a description directly
   (`/agentic-harness:brd <idea>`), start from that.
2. **`planning/input/`** — any file that isn't `.gitkeep`. Read all of them;
   if multiple conflict, flag the conflict rather than silently picking the
   newest.
3. **Existing codebase** (`planning.entry_point == existing-project`) —
   dispatch `agentic-harness:codebase-analyst` and use its Purpose,
   Feature inventory, and Actors/roles sections as a first draft — present
   every inferred item back to the user for correction, never assert it.
4. **Nothing** — start from a blank slate and ask.

Whatever the source, the BRD you produce is confirmed by the user, not
copied verbatim from a source document — a dropped-in doc or an analyst
report is raw material, not the artifact itself.

## Question bank

Work through these, batching related questions per
`planning-protocol.md`'s Questioning rules:

- **Problem / pain** — what's broken or missing today, for whom.
- **Users & personas** — who uses this, what do they need from it,
  anything distinguishing role-by-role.
- **Business goals, in priority order** — if the source material implies
  an order (emphasis, "must have" vs "nice to have") propose it and ask for
  confirmation; don't assert an inferred order silently.
- **Success criteria** — measurable, specific ("95% on-time", "under 2s
  response", "zero data loss") not vague ("fast", "reliable").
- **Scope** — explicitly in and explicitly out. Out-of-scope items matter
  as much as in-scope ones for keeping later stages honest.
- **Constraints** — regulatory, budget, deadline, mandated tech/vendor,
  anything the solution must never violate.
- **Stakeholders** — who approves, who's affected, who's consulted.
- **Risks** — what could derail this.
- **Edge cases** — ask through these explicitly, don't wait for the user to
  volunteer them: what happens when each external integration fails; is
  there a scale ceiling (10 users vs 10M); multi-tenancy; i18n/locale;
  offline/degraded-connectivity behavior; data retention, privacy, and
  deletion requirements.

## Output — `planning/BRD.md`

```markdown
# Business Requirements Document

## Purpose
## Problem
## Stakeholders
## Personas
## Business Goals (priority order)
1. ...
## Success Criteria
## Scope
### In scope
### Out of scope
## Constraints & Invariants
## Risks
## Assumptions (unvalidated)
## Open Questions
## Glossary
```

## Approval gate

Per `planning-protocol.md`. On approval: write the file, set
`planning.stages.brd.status: approved` + `approved_on`, then tell the user
`/agentic-harness:configure` will run next (or run it yourself if invoked
via `/agentic-harness:plan`) to turn these goals into
`planning/BUSINESS_GOALS.md` and `planning/project.config.yaml`.

## Report (≤6 lines)

```
planning/BRD.md: <written/updated>
Goals: N (priority-ordered)
Open questions: N
Assumptions (unvalidated): N
Status: draft/approved
Next: /agentic-harness:configure, then /agentic-harness:srs
```
