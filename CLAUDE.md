# CLAUDE.md

This file governs how Claude Code works in this repo. It is deliberately
generic — project specifics live in `config/project.config.yaml` and
`docs/business-logic.md`, not here, so this harness can be dropped into any
new project unchanged.

## Load order (do this before any development work)

1. `config/project.config.yaml` — tools enabled, external IDs, business-goal
   priorities, codebase segments (`architecture.segments`). If
   `project.configured` is `false`, stop and run `/configure` first (see
   below) — do not guess at IDs or goals.
2. `docs/business-logic.md` — the business goals/success-criteria this
   project is judged against, in priority order. Generated from the BRD.
   Every task, from either agent, must trace back to a goal here.
3. `docs/engineering-standards.md` — the code-quality bar `/architect`
   enforces on every task before marking it done.
4. `TASK_LOG.md` — durable task history. Single source of truth for what's
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

- **`/run-manager`** — the business-goal owner. Runs
  `config.manager.analysis_command` (project-specific — a test suite,
  report generator, benchmark script, whatever produces signal), evaluates
  the result against `docs/business-logic.md`'s priority-ordered goals,
  and appends new 🔴/🟡 tasks to `TASK_LOG.md` — every task tagged with
  the exact goal it serves. It does not fix things itself and does not
  decide segmentation — it queues work for `/architect`.
- **`/architect`** — the senior-engineer orchestrator, owns the technical
  side: efficiency, integrity, scalability, quality. Reads `TASK_LOG.md` +
  config, keeps `config/project.config.yaml`'s `architecture.segments`
  current by categorizing the codebase into owned areas, and creates a
  scoped subagent per segment under `.claude/agents/`. It **delegates
  implementation to the owning segment subagent by default** (via the
  Agent tool, so each subagent's context stays scoped to its own area
  instead of the whole codebase) — it writes code directly only when
  delegation genuinely isn't worth it (a one-line fix, a cross-cutting
  change no segment owns). Either way, before marking a task ✅ DONE it
  gates on three things: the standing `test-writer` agent having written
  independent tests for the change, those tests passing, and an
  architecture/standards review against `docs/engineering-standards.md`.
- **Tool-mirror skills** (e.g. `/clickup-log`) — mirror every `TASK_LOG.md`
  write to whichever external tracker is enabled under `tools:` in config.
  Only mirror rows dated on/after that tool's `mirror_from_date`; never
  backfill history written before the mirror was wired up.

## Subagents (`.claude/agents/`)

Two kinds:

- **Segment subagents** — not hand-authored; `/architect` creates and
  maintains these as it segments the codebase (see `architect.md`
  Phase 1.5). Each one is scoped to a single area (`owns_paths`), knows
  its own test command, and is the only thing that ever edits its area.
  This is what keeps both architect's own context and each subagent's
  context small: architect only ever holds routing decisions, not
  implementation detail, and each subagent only ever holds its one
  segment, not the whole codebase. Genuinely per-project — encodes this
  specific codebase's structure.
- **`test-writer`** — the one standing exception, shipped as a ready-made
  template (`.claude/agents/test-writer.md`), not authored per project.
  Cross-cutting, owns no paths, and is dispatched after every
  implementation task to write that change's tests — deliberately never
  the same agent that wrote the implementation.
