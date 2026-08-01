---
name: adr
description: Enumerate architecture decisions implied by the SRS/FEATURES, present real tradeoffs via AskUserQuestion, and write planning/adr/ADR-NNNN-*.md in Nygard format. For an existing codebase, reverse-engineers decisions already made (status Accepted (retroactive)) via codebase-analyst. Never invents a decision the user hasn't made or confirmed.
---

# /agentic-harness:adr

Fifth planning stage. Produces one Architecture Decision Record per
significant technical decision the project needs — the record of *why*,
not just *what*, that `/agentic-harness:architect` and future maintainers
read before assuming a decision is up for relitigating. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

## Gate

`planning.stages.features.status` must be `approved`.

## Inputs

`planning/SRS.md`, `planning/FEATURES.md`. If
`planning.entry_point == existing-project`, dispatch
`agentic-harness:codebase-analyst` for its "Implicit architectural
decisions" section.

## Decision inventory

Enumerate candidate decisions from what the SRS/Features actually imply —
don't manufacture a decision nobody needs a record for. Typical categories
to check (skip any that don't apply):

- Language / framework
- Datastore(s)
- Auth approach
- API style (REST/GraphQL/RPC/etc.)
- Async / queueing, if any workload implies it
- Deployment / hosting target
- Multi-tenancy model, if the BRD/SRS implies multiple tenants
- Testing strategy, if it's non-default for the stack
- Observability approach

## New project — per decision

1. Present 2-4 real options via `AskUserQuestion`, each with genuine
   tradeoffs (use `preview` for a config/code sketch where it clarifies the
   choice) — never a single "recommended" option dressed up as a question.
2. Record the choice. If the user defers ("decide later," "not sure yet"),
   write the ADR with **status `Proposed`**, not `Accepted` — never
   silently pick for them and call it accepted.
3. Write `planning/adr/ADR-NNNN-<slug>.md` (Nygard format):
   ```markdown
   # ADR-NNNN: <title>

   ## Status
   Proposed | Accepted | Accepted (retroactive) | Deprecated | Superseded by ADR-NNNN

   ## Context
   What forces led here — reference the FR/NFR/Feature that requires a decision.

   ## Decision
   What was decided.

   ## Consequences
   What this makes easier/harder, going forward.

   ## Alternatives considered
   Each option presented, with why it wasn't chosen.

   ## Related
   FR-<MODULE>-##, NFR-<CATEGORY>-##, F-##
   ```

   ADRs use Nygard `Status` transitions, not the header-table Version
   field other artifacts use — that's the versioning mechanism here.
   Never edit an `Accepted` ADR's Decision in place: if a decision changes,
   write a new `ADR-NNNN`, mark the old one `Superseded by ADR-NNNN`, and
   the new one's Context references what changed and why.

## Existing project — per implicit decision found by codebase-analyst

Write the same ADR shape, **status `Accepted (retroactive)`**, Context
section states it was reverse-engineered, Decision section cites the
evidence paths codebase-analyst returned. Present these to the user for
confirmation before finalizing — the analyst's inference could be wrong.

## Index

Maintain `planning/adr/README.md`:
```markdown
# Architecture Decision Records

| ADR | Title | Status |
|---|---|---|
| [0001](ADR-0001-slug.md) | ... | Accepted |
```

## Approval gate

Per `planning-protocol.md`, applied to the batch of ADRs produced this run
(present the index, not each ADR individually, for the approve/revise/skip
decision — revise means "which ADR needs another look," not redo all of
them). On approval, set `planning.stages.adr.status: approved` +
`approved_on`.

## Report (≤6 lines)

```
ADRs written: N (Accepted: N, Proposed: N, Accepted (retroactive): N)
planning/adr/README.md: updated
Open questions: N
Status: draft/approved
Next: /agentic-harness:epics
```
