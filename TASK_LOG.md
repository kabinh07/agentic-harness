# Task Log

> Single source of truth for all agentic work in this repo. `/architect` reads
> this before planning; `/run-manager` appends new tasks here — every task
> must cite the business goal (from `docs/business-logic.md`) it serves; a
> task serving no goal doesn't get queued without an explicit reason. Any
> enabled tool-mirror skill (see `config/project.config.yaml`'s `tools:`
> block, e.g. `/clickup-log`) mirrors every write to the external tracker,
> scoped to rows dated `tools.<name>.mirror_from_date` onward — no backfill.

_Last updated: —_

## Status legend
⏳ TODO · 🔄 IN_PROGRESS · ✅ DONE (tests passing) · ⚠ BLOCKED · ❌ FAILED

A task only reaches ✅ DONE once its tests pass — see `architect.md` Phase 3.

## Source convention
`Manager YYYY-MM-DD` (from /run-manager) · `User YYYY-MM-DD` (direct ask) · `Auto YYYY-MM-DD` (self-queued)

## Priority
🔴 FAIL-level / urgent · 🟡 WARNING-level / soon · 🟢 nice-to-have

| # | Task | Goal | Source | Priority | Status | Agent | Started | Completed |
|---|---|---|---|---|---|---|---|---|

## Completed Tasks

_(none yet)_

## Manager Run History

_(none yet)_
