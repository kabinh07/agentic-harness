---
name: run-manager
description: Run the manager agent — analyse project state against docs/business-logic.md goals and write new tasks to TASK_LOG.md
---

# /run-manager

Analyses project state against the priority-ordered business goals in
`docs/business-logic.md` (mirrored, condensed, in `config/project.config.yaml`'s
`manager.business_goals`). Writes tasks to `TASK_LOG.md`. Never fixes
anything itself — it queues work for `/architect`.

## Usage

```
/run-manager
/run-manager --run-analysis    # run config.manager.analysis_command first, then analyse
```

## Steps

### Step 0 — Preconditions
If `config.project.configured` is `false`, stop — tell the user to run
`/configure` first. If `config.manager.analysis_command` is blank, this
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
Only add 🔴/🟡 tasks (see Severity below). Format:
`| N | <task with specifics> | Manager YYYY-MM-DD | 🔴/🟡 | ⏳ TODO | <skill or —> | — | — |`

For each tool under `config.tools` with `enabled: true`, invoke its
mirror skill (e.g. `/clickup-log`) to sync each new row, respecting that
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

### Step 9 — Invoke `/architect` for pending tasks

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
