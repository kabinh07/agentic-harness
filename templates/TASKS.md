# Tasks

> Single source of truth for all agentic work in this repo. `/agentic-harness:architect` reads
> this before planning; `/agentic-harness:manager` and `/agentic-harness:epics` append new
> tasks here — every task must cite the business goal (from `planning/BUSINESS_GOALS.md`) it
> serves, and, if it came from the planning phase, its `[F-##/S-##]` trace tag; a task serving
> no goal doesn't get queued without an explicit reason. Any enabled tool-mirror skill (see
> `planning/project.config.yaml`'s `tools:` block, e.g. `/agentic-harness:clickup-log`) mirrors
> every write to the external tracker, scoped to rows dated `tools.<name>.mirror_from_date`
> onward — no backfill.

_Last updated: —_

## Status legend
⏳ TODO · 🔄 IN_PROGRESS · ✅ DONE (tests passing) · ⚠ BLOCKED · ❌ FAILED

A task only reaches ✅ DONE once its tests pass — see `architect.md` Phase 3.

## Source convention
`Manager YYYY-MM-DD` (from /agentic-harness:manager) · `Planning YYYY-MM-DD` (seeded by /agentic-harness:epics) · `User YYYY-MM-DD` (direct ask) · `Auto YYYY-MM-DD` (self-queued)

## Priority
🔴 FAIL-level / urgent · 🟡 WARNING-level / soon · 🟢 nice-to-have

| # | Task | Goal | Source | Priority | Status | Agent | Started | Completed |
|---|---|---|---|---|---|---|---|---|

## Completed Tasks

_(none yet)_

## Manager Run History

_(none yet)_
