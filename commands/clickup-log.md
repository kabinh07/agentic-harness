---
name: clickup-log
description: Mirror TASKS.md entries as ClickUp subtasks under config.tools.clickup.parent_task_id, and log status changes as comments. Invoke every time TASKS.md is written to, for tasks dated on/after config.tools.clickup.mirror_from_date only.
---

# /agentic-harness:clickup-log

Mirrors `TASKS.md` into ClickUp as subtasks under the parent task
identified by `config.tools.clickup.parent_task_id` (list
`config.tools.clickup.list_id`). Keeps ClickUp and TASKS.md in sync going
forward. Never backfill — tasks logged before
`config.tools.clickup.mirror_from_date` have no ClickUp counterpart and stay
that way.

If `config.tools.clickup.enabled` is `false`, this skill is a no-op — don't
invoke it, and don't ask the user to enable it unprompted.

## When to invoke

Any time `TASKS.md` is written to — new row added, status changed, task
completed — whether done through `/agentic-harness:architect`,
`/agentic-harness:manager`, `/agentic-harness:epics`, or a direct manual
edit. Treat a TASKS.md edit and its ClickUp sync as one atomic step (see
`architect.md`'s "TASKS.md always up to date" invariant).

## Steps

### 1. New task added to TASKS.md (⏳ TODO row)

`clickup_create_task` with:
- `parent`: `config.tools.clickup.parent_task_id`
- `list_id`: `config.tools.clickup.list_id`
- `name`: short title (≤ ~70 chars — the headline, not the full TASKS paragraph)
- `markdown_description`: 2-4 sentence summary (what / why / root cause if known, plus the Goal column's value so the business rationale travels with the task) — not the full verbose TASKS text
- `start_date`: the TASKS "Started" date (YYYY-MM-DD)
- `due_date`: only if a real deadline exists; otherwise omit
- `status`: mapped per the table below

Record the returned ClickUp task ID back into the TASKS.md row (append
`[CU: <id>]` to the Task cell) so later updates can find it without searching.

### 2. Status changes (🔄 IN_PROGRESS / ✅ DONE / ⚠ BLOCKED)

- `clickup_update_task` on the mirrored subtask's ID, `status` mapped per the table below.
- `clickup_create_comment` on that subtask with a short log of what changed — mirror the TASKS update, condensed to a few lines, not pasted verbatim.

### 3. Never backfill

If asked to "sync everything" or "catch up ClickUp," only sync rows dated
on/after `config.tools.clickup.mirror_from_date`. Confirm with the user
before touching earlier rows.

### 4. Task rows seeded by `/agentic-harness:epics`

Rows seeded straight into `TASKS.md` by the planning phase's
`/agentic-harness:epics` (source `Planning YYYY-MM-DD`) mirror the same as
any other new row — the `[F-##/S-##]` trace tag in the Task cell travels
into the ClickUp description alongside the Goal, so the story/feature
context isn't lost on the ClickUp side either.

## Status mapping (TASKS → ClickUp)

| TASKS | ClickUp status |
|---|---|
| ⏳ TODO | `to do` |
| 🔄 IN_PROGRESS | `in progress` |
| ✅ DONE | `done` |
| ⚠ BLOCKED | `stuck` |

## Key invariants

- Parent task/list come only from config — never hardcode an ID here, and
  don't create subtasks anywhere else without being told.
- TASKS.md stays the verbose source of truth; ClickUp gets the headline, timeline, and status only.
- If the ClickUp MCP is unavailable, unauthenticated, or `config.tools.clickup.enabled` is `false`, don't block the TASKS.md write — note the sync was skipped (one line) and continue. TASKS.md always takes priority.
- Never invoke for entries dated before `config.tools.clickup.mirror_from_date`.
