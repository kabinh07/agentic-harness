# agentic-harness

Reusable scaffold for the agentic dev workflow used on `pep_routing_solution_parcel`:
a manager agent that watches project health against a business-logic rubric,
an architect agent that plans/delegates/tracks, a durable `TASK_LOG.md`, and
optional mirroring to external trackers (ClickUp, etc). This repo is the
generic shell — drop it into a new project and configure it from a BRD.

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
goals, tool IDs) and `docs/business-logic.md` (the human-readable rubric),
seeds `TASK_LOG.md`, and asks for anything it can't infer (e.g. a ClickUp
parent task ID needs a real lookup, not a guess).

## What's generic vs. project-specific

| Generic (ships as-is) | Project-specific (fill in per project) |
|---|---|
| `CLAUDE.md` load order | `config/project.config.yaml` values |
| `architect.md` phase structure | `docs/business-logic.md` content |
| `run-manager.md` phase structure | `manager.analysis_command` (your test/report script) |
| `TASK_LOG.md` schema | `skills.registry` entries |
| `clickup-log.md` (any tool-mirror skill) | leaf skills like `fix-routing.md` — a Module Map + fix procedure specific to one codebase |

The orchestration shell doesn't change between projects. What changes is
the config it reads and the leaf skills it dispatches to.

## Layout

```
CLAUDE.md                    # load order: config -> business-logic -> TASK_LOG
config/project.config.yaml   # tools, IDs, business-goal priorities, skill registry
BRD/                         # drop the business requirements doc here
docs/business-logic.md       # generated rubric — what /run-manager judges against
TASK_LOG.md                  # durable task history
.claude/commands/
  configure.md                # bootstrap: BRD -> config + business-logic.md + TASK_LOG.md
  architect.md                 # orchestrator
  run-manager.md                # periodic health-check / task-queuing agent
  clickup-log.md                 # example tool-mirror skill (ClickUp)
```

## Adding project-specific skills

Once code exists, add `.claude/commands/<skill>.md` files for recurring
fix/build patterns in that codebase (module map + procedure), and register
each with a `trigger` in `config/project.config.yaml`'s `skills.registry`
so `/architect` knows when to dispatch to it.
