# CLAUDE.md

This file governs how Claude Code works in this repo. It is deliberately
generic — project specifics live in `planning/project.config.yaml` and
`planning/BUSINESS_GOALS.md`, not here, so this harness can be dropped into
any new project unchanged. It's installed via the `agentic-harness` Claude
Code plugin, which provides:

- Planning-phase commands: `/agentic-harness:plan` (driver), `/agentic-harness:brd`,
  `/agentic-harness:srs`, `/agentic-harness:design`, `/agentic-harness:features`,
  `/agentic-harness:adr`, `/agentic-harness:epics`
- Bootstrap commands: `/agentic-harness:init`, `/agentic-harness:configure`
- Execution commands: `/agentic-harness:architect`, `/agentic-harness:manager`,
  `/agentic-harness:test`, `/agentic-harness:clickup-log`
- The standing `agentic-harness:test-writer` and `agentic-harness:codebase-analyst` agents

None of that lives in this repo.

## Load order (do this before any development work)

1. `planning/project.config.yaml` — tools enabled, external IDs, business-goal
   priorities, planning-phase stage status, codebase segments and swarms
   (`architecture.segments` / `architecture.swarms`). If `project.configured`
   is `false`, stop and run `/agentic-harness:plan` (or `/agentic-harness:configure`
   if a BRD already exists) first — do not guess at IDs or goals.
2. `planning/BUSINESS_GOALS.md` — the business goals/success-criteria this
   project is judged against, in priority order. Generated from the BRD.
   Every task, from any agent, must trace back to a goal here.
3. `planning/ENGINEERING_STANDARDS.md` — the code-quality bar
   `/agentic-harness:architect` enforces on every task before marking it done.
4. `TASKS.md` — durable task history, project root. Single source of truth
   for what's done, in progress, or pending. Read it before planning any new
   work to avoid duplicate tasks.

## Bootstrapping a new project

1. Install the `agentic-harness` plugin (see the plugin repo's README) and
   run `/agentic-harness:init` to scaffold this skeleton into the project
   if it isn't here yet (`CLAUDE.md`, `TASKS.md`, `planning/`).
2. Run `/agentic-harness:plan`. It figures out where you're starting from —
   a bare idea, an existing codebase, an existing BRD, or an existing SRS —
   and walks you through the planning phase, asking whatever it needs
   rather than assuming:
   - `/agentic-harness:brd` → `planning/BRD.md`
   - `/agentic-harness:configure` → fills `planning/project.config.yaml` +
     `planning/BUSINESS_GOALS.md` from the approved BRD
   - `/agentic-harness:srs` → `planning/SRS.md`
   - `/agentic-harness:design` (skippable — only if the project has a UI) →
     `planning/DESIGN_BRIEF.md` (for your external design tool) then
     `planning/DESIGN.md` (consistency context, once you've designed)
   - `/agentic-harness:features` → `planning/FEATURES.md`
   - `/agentic-harness:adr` → `planning/adr/ADR-NNNN-*.md`
   - `/agentic-harness:epics` → `planning/EPICS.md` (plain-language epics,
     EARS acceptance criteria, WSJF-scored, decomposed straight into
     implementation-sized tasks), decides delivery mode (whole project at
     once, or committed weekly sprints), and seeds `TASKS.md`
3. Calibrates once (knowledge/pressure level), then asks through a
   question ladder before writing anything final — never invents an
   answer it doesn't have. Unresolved items land under that artifact's
   Open Items (TBD) section; confirmed background conditions land under
   Assumptions & Constraints. `planning/README.md` tracks stage status
   (with each artifact's Version) and open items for the whole bundle at
   a glance. Every artifact rewrite is archived to `planning/versions/`
   first — nothing is silently overwritten.
4. Once `TASKS.md` has seeded rows, run `/agentic-harness:architect` to
   start implementation.
5. Re-run `/agentic-harness:plan <stage>` any time an earlier artifact
   changes materially — later stages are gated on their predecessor's
   approval, so a stale BRD blocks a new SRS run until reconciled.

## Agents

- **`/agentic-harness:manager`** — the business-goal owner. Runs
  `config.manager.analysis_command` (project-specific — a test suite,
  report generator, benchmark script, whatever produces signal), evaluates
  the result against `planning/BUSINESS_GOALS.md`'s priority-ordered goals,
  and appends new 🔴/🟡 tasks to `TASKS.md` — every task tagged with
  the exact goal it serves. It does not fix things itself and does not
  decide segmentation — it queues work for `/agentic-harness:architect`.
- **`/agentic-harness:architect`** — the senior-engineer orchestrator, owns
  the technical side: efficiency, integrity, scalability, quality. Reads
  `TASKS.md` + config, keeps `planning/project.config.yaml`'s
  `architecture.segments` current by categorizing the codebase into owned
  areas, and **creates and removes** the scoped subagent per segment under
  `.claude/agents/` as the codebase evolves (merged/deleted areas lose their
  agent too). For work that legitimately spans segments it can form a
  temporary swarm — several segment agents dispatched in parallel on one
  task, each still bounded to its own paths — recorded under
  `architecture.swarms` while active and torn down once the task closes.
  It **delegates implementation to the owning segment subagent by default**
  (via the Agent tool, so each subagent's context stays scoped to its own
  area instead of the whole codebase) — it writes code directly only when
  delegation genuinely isn't worth it (a one-line fix, a cross-cutting
  change no segment owns). Either way, before marking a task ✅ DONE it
  gates on three things: the standing `agentic-harness:test-writer` agent
  having written independent tests for the change, those tests passing, and
  an architecture/standards review against `planning/ENGINEERING_STANDARDS.md`.
- **`/agentic-harness:test`** — thin command wrapper around the standing
  test-writer agent, for running the test-gate directly instead of only as
  part of an architect dispatch. Never edits production code, never marks
  a task done.
- **Tool-mirror skills** (e.g. `/agentic-harness:clickup-log`) — mirror
  every `TASKS.md` write to whichever external tracker is enabled under
  `tools:` in config. Only mirror rows dated on/after that tool's
  `mirror_from_date`; never backfill history written before the mirror was
  wired up.

## Subagents

Three kinds:

- **Segment subagents (`.claude/agents/`, project-local)** — not
  hand-authored; `/agentic-harness:architect` creates, updates, and removes
  these as it segments (and re-segments) the codebase (see `architect.md`
  Phase 1.5). Each one is scoped to a single area (`owns_paths`), knows its
  own test command, and is the only thing that ever edits its area. This is
  what keeps both architect's own context and each subagent's context
  small: architect only ever holds routing decisions, not implementation
  detail, and each subagent only ever holds its one segment, not the whole
  codebase. Genuinely per-project — encodes this specific codebase's
  structure.
- **`agentic-harness:test-writer`** — standing exception, shipped by the
  plugin itself, not authored per project and not a file in this repo.
  Cross-cutting, owns no paths, dispatched after every implementation task
  (or directly via `/agentic-harness:test`) to write that change's tests —
  deliberately never the same agent that wrote the implementation.
- **`agentic-harness:codebase-analyst`** — standing, read-only. Surveys an
  existing codebase for the planning-phase commands (`:brd`, `:adr`) when
  starting from real code instead of a blank idea: feature inventory,
  implicit architectural decisions, data model — all with evidence paths,
  every inference labeled as inference, never asserted as fact.
