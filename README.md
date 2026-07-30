# agentic-harness

A Claude Code plugin for a reusable, end-to-end agentic dev workflow: a
question-driven **planning phase** (BRD → SRS → design context → features →
ADRs → epics) that hands off to a **manager** agent owning the business
goals and a senior-engineer **architect** agent that segments the codebase,
creates and removes scoped subagents (and temporary swarms for cross-segment
work), and staffs a standing test-writer — all gated on a durable `TASKS.md`,
with optional mirroring to external trackers (ClickUp, etc).

Start from a bare idea, an existing codebase, an existing BRD, or an
existing SRS — the harness asks what it doesn't know instead of guessing,
and leaves every planning artifact in one shareable `planning/` folder.

## Install

From inside Claude Code, in any project:

```
/plugin marketplace add /path/to/agentic-harness
/plugin install agentic-harness
```

(or, once published, `/plugin marketplace add kabinh07/agentic-harness`).

This gives you every command below, available in every project:

**Planning phase**
- `/agentic-harness:plan` — driver/wizard, walks BRD → SRS → design →
  features → ADR → epics
- `/agentic-harness:brd`
- `/agentic-harness:srs`
- `/agentic-harness:design`
- `/agentic-harness:features`
- `/agentic-harness:adr`
- `/agentic-harness:epics`

**Bootstrap & execution**
- `/agentic-harness:init`
- `/agentic-harness:configure`
- `/agentic-harness:architect`
- `/agentic-harness:manager`
- `/agentic-harness:test`
- `/agentic-harness:clickup-log`

**Agents**
- `agentic-harness:test-writer`
- `agentic-harness:codebase-analyst`

## Quickstart (in a project you want the harness in)

```
/agentic-harness:init          # scaffolds CLAUDE.md, TASKS.md, planning/ (config, docs, input/)
/agentic-harness:plan          # walks the whole planning phase, asking what it needs
```

`/agentic-harness:plan` figures out your entry point — a bare idea (just
tell it), an existing codebase (it surveys it via
`agentic-harness:codebase-analyst`), material dropped in `planning/input/`,
or an existing `planning/BRD.md`/`planning/SRS.md` — then drives each stage
command in order, asking clarifying questions and never inventing an
answer: BRD → `/agentic-harness:configure` → SRS → design (skippable, only
if the project has a UI) → features → ADRs → epics. Each stage ends with an
approval gate; unresolved items land under that artifact's "Assumptions
(unvalidated)" instead of being guessed. `planning/README.md` tracks stage
status for the whole bundle so you can hand it to a teammate as-is.

Once `/agentic-harness:epics` has seeded `TASKS.md`, run
`/agentic-harness:architect` — its first job is always to segment the
codebase into owned areas and create a subagent per segment under
`.claude/agents/` (project-local, not part of this plugin) before it
dispatches any task.

## How a task moves through the system

1. **`/agentic-harness:manager`** checks project state against
   `planning/BUSINESS_GOALS.md`'s priority-ordered goals and writes a task to
   `TASKS.md` — every row tagged with the exact goal it serves. No goal,
   no task; observations that don't trace to a goal get reported, not
   queued. (`/agentic-harness:epics` seeds the initial backlog the same
   way, from the planning phase's epics/stories instead of a live analysis.)
2. **`/agentic-harness:architect`** picks it up. If the codebase has grown
   (or shrunk) since the last segmentation, it re-segments first
   (`architecture.segments` in config) — creating subagents for new areas
   **and removing** `.claude/agents/<segment>.md` files for areas that no
   longer exist or have merged.
3. Architect dispatches the task to that segment's subagent via the Agent
   tool — the subagent gets the task, its own `owns_paths` boundary, and
   the goal it serves; nothing else. This is what keeps context small on
   both sides: architect never holds implementation detail, the subagent
   never holds the rest of the codebase. Delegation is the default;
   architect implements directly only when it's not worth a dispatch (a
   one-line fix, a change no single segment owns). For work that
   genuinely spans segments, architect can form a temporary **swarm** —
   several segment agents dispatched in parallel on one task, each still
   bounded to its own paths — torn down once the task closes.
4. The subagent (or architect-direct, or swarm) implements the change,
   scoped to its own area(s).
5. Architect dispatches the standing `agentic-harness:test-writer` agent
   (also invocable directly via `/agentic-harness:test`) to write tests for
   exactly that change — deliberately never the same agent that wrote the
   implementation. No implementation task is done without tests from this
   independent pass.
6. Architect gates completion on three things before touching the status
   column: the test-writer's tests existing, the segment's tests passing
   (including the new ones), and an architecture/standards review against
   `planning/ENGINEERING_STANDARDS.md` (stayed in bounds, no dead code, no
   premature abstraction, matches existing patterns). Any gate failing
   sends it back or marks it ⚠ BLOCKED — never ✅ DONE on "looks right."
7. `TASKS.md` gets the final status; any enabled tool mirror
   (`config.tools.*`) syncs it out.

## What's plugin-shipped vs. project-specific

| Plugin-shipped (this repo) | Project-specific (grown per install) |
|---|---|
| `commands/*.md` phase structure (plan/brd/srs/design/features/adr/epics/configure/architect/manager/test/clickup-log) | `planning/project.config.yaml` values |
| `agents/test-writer.md`, `agents/codebase-analyst.md` | `planning/BUSINESS_GOALS.md`, `planning/SRS.md`, `planning/FEATURES.md`, `planning/adr/*.md` content |
| `templates/CLAUDE.md` load order | `planning/ENGINEERING_STANDARDS.md`'s "Project-specific additions" |
| `templates/TASKS.md` schema | `architecture.segments`/`architecture.swarms` and their `.claude/agents/*.md` subagents |

The orchestration shell doesn't change between projects. What changes is
the config and planning artifacts it reads and the segment subagents it
dispatches to — and those are grown by the planning-phase commands and
`/agentic-harness:architect` itself, not hand-authored per project.

## Adding project-specific segment agents

You don't hand-author segment agents — `/agentic-harness:architect` does
that as part of Phase 1.5 ("Segment & Staff") the first time it runs
against real code, and re-segments (creating and removing agents as
needed) whenever areas appear or disappear that the current segmentation
doesn't cover. Force a re-scan any time with `/agentic-harness:architect
resegment`.
