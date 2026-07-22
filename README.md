# agentic-harness

A Claude Code plugin for a reusable agentic dev workflow: a manager agent
that owns the business goals, a senior-engineer architect agent that
segments the codebase and staffs a scoped subagent per segment, a durable
`TASK_LOG.md` gated on passing tests, and optional mirroring to external
trackers (ClickUp, etc).

## Install

From inside Claude Code, in any project:

```
/plugin marketplace add /path/to/agentic-harness
/plugin install agentic-harness
```

(or, once published, `/plugin marketplace add kabinh07/agentic-harness`).

This gives you five commands and one agent, available in every project:

- `/agentic-harness:init`
- `/agentic-harness:configure`
- `/agentic-harness:architect`
- `/agentic-harness:run-manager`
- `/agentic-harness:clickup-log`
- agent `agentic-harness:test-writer`

## Quickstart (in a project you want the harness in)

```
/agentic-harness:init          # scaffolds CLAUDE.md, config/, docs/, TASK_LOG.md, BRD/
```

Then drop a BRD (business requirements doc — any format) into `BRD/`, and:

```
/agentic-harness:configure     # reads the BRD, fills config + business-logic.md, seeds TASK_LOG.md
```

Once there's real code, run `/agentic-harness:architect` — its first job is
always to segment the codebase into owned areas and create a subagent per
segment under `.claude/agents/` (project-local, not part of this plugin)
before it dispatches any task.

## How a task moves through the system

1. **`/agentic-harness:run-manager`** checks project state against
   `docs/business-logic.md`'s priority-ordered goals and writes a task to
   `TASK_LOG.md` — every row tagged with the exact goal it serves. No goal,
   no task; observations that don't trace to a goal get reported, not
   queued.
2. **`/agentic-harness:architect`** picks it up. If the codebase has grown
   since the last segmentation, it re-segments first
   (`architecture.segments` in config) and creates/updates the relevant
   `.claude/agents/<segment>.md` subagent.
3. Architect dispatches the task to that segment's subagent via the Agent
   tool — the subagent gets the task, its own `owns_paths` boundary, and
   the goal it serves; nothing else. This is what keeps context small on
   both sides: architect never holds implementation detail, the subagent
   never holds the rest of the codebase. Delegation is the default;
   architect implements directly only when it's not worth a dispatch (a
   one-line fix, a change no single segment owns).
4. The subagent (or architect-direct) implements the change, scoped to its
   own area.
5. Architect dispatches the standing `agentic-harness:test-writer` agent to
   write tests for exactly that change — deliberately never the same agent
   that wrote the implementation. No implementation task is done without
   tests from this independent pass.
6. Architect gates completion on three things before touching the status
   column: the test-writer's tests existing, the segment's tests passing
   (including the new ones), and an architecture/standards review against
   `docs/engineering-standards.md` (stayed in bounds, no dead code, no
   premature abstraction, matches existing patterns). Any gate failing
   sends it back or marks it ⚠ BLOCKED — never ✅ DONE on "looks right."
7. `TASK_LOG.md` gets the final status; any enabled tool mirror
   (`config.tools.*`) syncs it out.

## What's plugin-shipped vs. project-specific

| Plugin-shipped (this repo) | Project-specific (grown per install) |
|---|---|
| `commands/*.md` phase structure (configure/architect/run-manager/clickup-log) | `config/project.config.yaml` values |
| `agents/test-writer.md` | `docs/business-logic.md` content |
| `templates/CLAUDE.md` load order | `docs/engineering-standards.md`'s "Project-specific additions" |
| `templates/TASK_LOG.md` schema | `architecture.segments` and their `.claude/agents/*.md` subagents |

The orchestration shell doesn't change between projects. What changes is
the config it reads and the segment subagents it dispatches to — and those
are grown by `/agentic-harness:architect` itself, not hand-authored per
project.

## Adding project-specific segment agents

You don't hand-author segment agents — `/agentic-harness:architect` does
that as part of Phase 1.5 ("Segment & Staff") the first time it runs
against real code, and re-segments whenever new areas appear that no
existing segment's `owns_paths` covers. Force a re-scan any time with
`/agentic-harness:architect resegment`.
