# agentic-harness

Reusable scaffold for the agentic dev workflow used on `pep_routing_solution_parcel`:
a manager agent that owns the business goals, a senior-engineer architect
agent that segments the codebase and staffs a scoped subagent per segment,
a durable `TASK_LOG.md` gated on passing tests, and optional mirroring to
external trackers (ClickUp, etc). This repo is the generic shell — drop it
into a new project and configure it from a BRD.

## Quickstart

```bash
# 1. Copy this harness into a new (or existing) project
cp -r agentic-harness/{CLAUDE.md,config,BRD,docs,TASK_LOG.md,.claude} /path/to/new-project/

# 2. Drop a BRD (any format) into BRD/
cp ~/Downloads/my-project-brd.pdf /path/to/new-project/BRD/

# 3. From inside the new project, in Claude Code:
/configure
```

`/configure` reads the BRD and fills `config/project.config.yaml` (business
goals, tool IDs, a first-pass component breakdown if the BRD names any)
and `docs/business-logic.md` (the human-readable rubric), seeds
`TASK_LOG.md`, and asks for anything it can't infer (e.g. a ClickUp parent
task ID needs a real lookup, not a guess).

Once there's real code, run `/architect` — its first job is always to
segment the codebase into owned areas and create a subagent per segment
under `.claude/agents/` (see below) before it dispatches any task.

## How a task moves through the system

1. **`/run-manager`** checks project state against `docs/business-logic.md`'s
   priority-ordered goals and writes a task to `TASK_LOG.md` — every row
   tagged with the exact goal it serves. No goal, no task; observations
   that don't trace to a goal get reported, not queued.
2. **`/architect`** picks it up. If the codebase has grown since the last
   segmentation, it re-segments first (`architecture.segments` in config)
   and creates/updates the relevant `.claude/agents/<segment>.md` subagent.
3. Architect dispatches the task to that segment's subagent via the Agent
   tool — the subagent gets the task, its own `owns_paths` boundary, and
   the goal it serves; nothing else. This is what keeps context small on
   both sides: architect never holds implementation detail, the subagent
   never holds the rest of the codebase. Delegation is the default;
   architect implements directly only when it's not worth a dispatch (a
   one-line fix, a change no single segment owns).
4. The subagent (or architect-direct) implements the change, scoped to its
   own area.
5. Architect dispatches the standing `test-writer` agent to write tests
   for exactly that change — deliberately never the same agent that wrote
   the implementation. No implementation task is done without tests from
   this independent pass.
6. Architect gates completion on three things before touching the status
   column: the test-writer's tests existing, the segment's tests passing
   (including the new ones), and an architecture/standards review against
   `docs/engineering-standards.md` (stayed in bounds, no dead code, no
   premature abstraction, matches existing patterns). Any gate failing
   sends it back or marks it ⚠ BLOCKED — never ✅ DONE on "looks right."
7. `TASK_LOG.md` gets the final status; any enabled tool mirror
   (`config.tools.*`) syncs it out.

## What's generic vs. project-specific

| Generic (ships as-is) | Project-specific (fill in / grows per project) |
|---|---|
| `CLAUDE.md` load order | `config/project.config.yaml` values |
| `architect.md` phase structure (segment → dispatch → gate → review) | `docs/business-logic.md` content |
| `run-manager.md` phase structure | `manager.analysis_command` (your test/report script) |
| `TASK_LOG.md` schema | `architecture.segments` and their `.claude/agents/*.md` subagents |
| `clickup-log.md` (any tool-mirror skill) | `docs/engineering-standards.md`'s "Project-specific additions" |

The orchestration shell doesn't change between projects. What changes is
the config it reads and the segment subagents it dispatches to — and
those are grown by `/architect` itself, not hand-authored per project.

## Layout

```
CLAUDE.md                    # load order: config -> business-logic -> standards -> TASK_LOG
config/project.config.yaml   # tools, IDs, business-goal priorities, architecture.segments
BRD/                         # drop the business requirements doc here
docs/
  business-logic.md           # generated rubric — what /run-manager judges against
  engineering-standards.md    # code-quality bar /architect enforces at review time
TASK_LOG.md                  # durable task history (Goal-tagged, test-gated)
.claude/
  commands/
    configure.md               # bootstrap: BRD -> config + business-logic.md + TASK_LOG.md
    architect.md                 # orchestrator: segment, delegate, test, gate, review
    run-manager.md                 # goal-owning, periodic health-check / task-queuing agent
    clickup-log.md                  # example tool-mirror skill (ClickUp)
  agents/
    test-writer.md               # ships as-is — standing, cross-cutting test author
                                  # everything else here: NOT hand-authored — /architect
                                  # creates one segment agent per codebase area, first run
```

## Adding project-specific skills

You don't hand-author segment agents — `/architect` does that as part of
Phase 1.5 ("Segment & Staff") the first time it runs against real code,
and re-segments whenever new areas appear that no existing segment's
`owns_paths` covers. Force a re-scan any time with `/architect resegment`.
