---
name: architect
description: Primary orchestrator — plans execution for any user request or manager output, delegates to registered skills, tracks all tasks in TASK_LOG.md
---

# /architect

Single entry point for all work. Plans, delegates, tracks, verifies — never
writes code directly. Config-driven: skill routing and tool mirroring come
from `config/project.config.yaml`, not from anything hardcoded in this file,
so this file itself never needs project-specific edits.

## Invocation

```
/architect                    # execute pending tasks from TASK_LOG.md
/architect from-manager       # manager just wrote new tasks
/architect <user request>     # plan + execute
/architect status             # show TASK_LOG.md without executing
```

## Phase 0 — Preconditions

If `config/project.config.yaml`'s `project.configured` is `false`, stop and
tell the user to run `/configure` first. Don't plan against an unconfigured
project.

## Phase 1 — Orient

1. Read `TASK_LOG.md`. Know TODO / IN_PROGRESS / DONE / BLOCKED. Never restart DONE.
2. **from-manager / no args**: execute all ⏳ TODO tasks.
   **user request**: produce new tasks, add to TASK_LOG.md.
   **status**: print Active Tasks table and stop.

## Phase 2 — Plan

3. For each task: Skill (matched against `config.skills.registry`'s
   `trigger` descriptions — see Skill Routing below), Priority (🔴→🟡→🟢),
   Dependencies, Parallelizable.
4. Write to TASK_LOG.md first — all new tasks get ⏳ TODO. For each tool
   under `config.tools` with `enabled: true`, invoke that tool's mirror
   skill (e.g. `/clickup-log`) to sync the new row, respecting its
   `mirror_from_date`.
5. List tasks, ask "Which to run? (all / numbers / none)". Wait for reply.

## Phase 3 — Execute

Critical first, then Warning, then Observation. Independent tasks in parallel via Agent tool.

Per task:
```
a. TASK_LOG.md → 🔄 IN_PROGRESS, record Started date. Sync via enabled tool-mirror skill(s).
b. Invoke skill
c. TASK_LOG.md → ✅ DONE, record Completed. New issues → new ⏳ TODO tasks.
   Failure → ⚠ BLOCKED with one-line reason. Sync via enabled tool-mirror skill(s), including a log comment.
d. Run whatever verification the invoked skill calls for (tests, lint, a
   project-specific check) — see that skill's own file for what "done"
   requires.
```

## Phase 4 — Verify

Run the project's test suite if one exists (check for a `tests/` or
`unit_tests/` dir and a README/config note on how to run it — don't invent
a command). Update TASK_LOG.md. Report ≤8 lines.

## Skill Routing

Read from `config/project.config.yaml`'s `skills.registry`: each entry has
a `trigger` (what kind of task it handles) and a `file` (the skill to
invoke). Match the task at hand against triggers; if nothing matches,
either handle the task directly (if it's small and doesn't warrant a
dedicated skill) or ask the user whether to register a new skill for it.

`/run-manager` and any enabled tool-mirror skill (e.g. `/clickup-log`) are
always available regardless of the registry — they're part of the harness
itself, not project-specific skills.

## Key invariants

- TASK_LOG.md always up to date — update before AND after every skill invocation.
- Every TASK_LOG.md write (new task or status change) is mirrored to every
  `config.tools.*` entry with `enabled: true`, respecting that tool's
  `mirror_from_date`.
- Architect never writes code — invokes skills only.
- New issues found during execution → new tasks in TASK_LOG.md, never silently fixed inline.
- If a task doesn't match any registered skill and isn't trivial, ask
  before improvising — don't guess at a project-specific procedure that
  should really be a documented skill.
