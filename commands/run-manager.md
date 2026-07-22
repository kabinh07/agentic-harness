---
name: run-manager
description: Run the manager agent — analyse project state against docs/business-logic.md goals and write new tasks to TASK_LOG.md
---

# /agentic-harness:run-manager

Analyses project state against the priority-ordered business goals in
`docs/business-logic.md` (mirrored, condensed, in `config/project.config.yaml`'s
`manager.business_goals`). Writes tasks to `TASK_LOG.md`, each one tagged
with the exact goal it serves — this file is the guardrail that keeps
task creation tied to the BRD instead of drifting into whatever seems
locally interesting. Never fixes anything itself — it queues work for
`/agentic-harness:architect`, which segments, dispatches, and gates it.

## Usage

```
/agentic-harness:run-manager
/agentic-harness:run-manager --run-analysis    # run config.manager.analysis_command first, then analyse
```

## Steps

### Step 0 — Preconditions
If `config.project.configured` is `false`, stop — tell the user to run
`/agentic-harness:configure` first. If `config.manager.analysis_command` is blank, this
project has no analysis tooling yet; say so and stop (don't invent a
substitute check) unless the user gives you something concrete to run
instead.

### Step 1 — Optionally run analysis
If `--run-analysis` passed: run `config.manager.analysis_command` verbatim.

### Step 2 — Read the result
Read whatever `config.manager.report_path` points to (or the analysis
command's stdout, if it doesn't write a file). Extract whatever metrics
exist — don't assume a fixed schema; this varies per project.

### Step 3 — Read `docs/business-logic.md`
This is the rubric. Walk `manager.business_goals` in priority order and
judge the analysis result against each goal in turn, top priority first.

### Step 4 — Read `TASK_LOG.md`
Avoid duplicate tasks — check open ⏳/🔄 rows before adding a new one for
the same issue.

### Step 5 — Append new tasks
Only add 🔴/🟡 tasks (see Severity below). Every row must name the exact
goal (from `docs/business-logic.md`) the task closes the gap on — if a
finding doesn't trace to any documented goal, it's not a task, it's an
observation; note it in the run report instead of queuing it. Format:
`| N | <task with specifics> | <goal it serves> | Manager YYYY-MM-DD | 🔴/🟡 | ⏳ TODO | <agent segment, or — if unstaffed> | — | — |`

Leave the Agent column as `—` if `architecture.segments` doesn't yet have
a segment covering this task's area — `/agentic-harness:architect` resolves that at
dispatch time (Phase 1.5), the manager doesn't guess at segmentation.

For each tool under `config.tools` with `enabled: true`, invoke its
mirror skill (e.g. `/agentic-harness:clickup-log`) to sync each new row, respecting that
tool's `mirror_from_date`.

### Step 6 — Append Manager Run History row
### Step 7 — Update `_Last updated` timestamp

### Step 8 — Report (≤8 lines)
```
Status: OK/WARNING/FAIL
Goals evaluated: N (top priority: <goal 1>, result: ...)
Tasks added: N
Top issues: ...
```

### Step 9 — Invoke `/agentic-harness:architect` for pending tasks

## Severity

Project-specific severity thresholds live in
`config.manager.severity_thresholds` (metric → {warning, fail}), if the
project has defined any. Absent explicit thresholds, use judgment against
the priority order in `docs/business-logic.md`: violating goal 1 is always
🔴, violating a lower-priority goal is 🟡 unless it also blocks a
higher-priority one.

- 🔴 **FAIL**: any top-priority goal violated, or a stated hard constraint/invariant broken.
- 🟡 **WARNING**: a lower-priority goal under target, or a soft success-criterion missed.
- 🟢 **OK**: all goals met per `docs/business-logic.md`'s success criteria.
