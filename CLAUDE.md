# CLAUDE.md

This file governs how Claude Code works in this repo. It is deliberately
generic — project specifics live in `config/project.config.yaml` and
`docs/business-logic.md`, not here, so this harness can be dropped into any
new project unchanged.

## Load order (do this before any development work)

1. `config/project.config.yaml` — tools enabled, external IDs, business-goal
   priorities, registered skills. If `project.configured` is `false`, stop
   and run `/configure` first (see below) — do not guess at IDs or goals.
2. `docs/business-logic.md` — the business goals/success-criteria this
   project is judged against, in priority order. Generated from the BRD.
3. `TASK_LOG.md` — durable task history. Single source of truth for what's
   done, in progress, or pending. Read it before planning any new work to
   avoid duplicate tasks.

## Bootstrapping a new project

1. Drop a BRD (business requirements doc — any format: `.md`, `.txt`,
   `.pdf`, `.docx`) into `BRD/`.
2. Run `/configure`.
3. It fills `config/project.config.yaml` (project name/description,
   priority-ordered business goals), generates `docs/business-logic.md`,
   seeds `TASK_LOG.md`, and asks (via clarifying questions) for any external
   tool IDs it can't infer from the BRD text alone — e.g. a ClickUp parent
   task ID needs a real lookup, not a guess.
4. Re-run `/configure` any time the BRD changes materially.

## Agents

- **`/architect`** — orchestrator. Reads `TASK_LOG.md` + config, plans, and
  dispatches to the right skill. Project-specific skills are registered in
  `config/project.config.yaml`'s `skills.registry`; anything not matching a
  registered skill it either handles directly or asks about.
- **`/run-manager`** — periodic health-check / manager agent. Runs
  `config.manager.analysis_command` (project-specific — a test suite, report
  generator, benchmark script, whatever produces signal), evaluates the
  result against `docs/business-logic.md`'s priority-ordered goals, and
  appends new 🔴/🟡 tasks to `TASK_LOG.md`. It does not fix things itself —
  it queues work for `/architect`.
- **Tool-mirror skills** (e.g. `/clickup-log`) — mirror every `TASK_LOG.md`
  write to whichever external tracker is enabled under `tools:` in config.
  Only mirror rows dated on/after that tool's `mirror_from_date`; never
  backfill history written before the mirror was wired up.

## Adding project-specific skills

As the codebase grows, add `.claude/commands/<skill>.md` files for
recurring fix/build patterns specific to this project (a "Module Map" of
where things live plus a fix procedure — see any mature sibling project's
`fix-routing.md`-style skill for the shape). Register each one in
`config/project.config.yaml`'s `skills.registry` with a `trigger`
description so `/architect` knows when to dispatch to it. These skills are
the one part of this harness that is NOT meant to be generic — they encode
this specific codebase's structure.
