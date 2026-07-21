---
name: clickup-log
description: Mirror TASK_LOG.md entries as ClickUp subtasks under config.tools.clickup.parent_task_id, and log status changes as comments. Invoke every time TASK_LOG.md is written to, for tasks dated on/after config.tools.clickup.mirror_from_date only.
---

# /clickup-log

Mirrors `TASK_LOG.md` into ClickUp as subtasks under the parent task
identified by `config.tools.clickup.parent_task_id` (list
`config.tools.clickup.list_id`). Keeps ClickUp and TASK_LOG.md in sync going
forward. Never backfill — tasks logged before
`config.tools.clickup.mirror_from_date` have no ClickUp counterpart and stay
that way.

If `config.tools.clickup.enabled` is `false`, this skill is a no-op — don't
invoke it, and don't ask the user to enable it unprompted.

## When to invoke

Any time `TASK_LOG.md` is written to — new row added, status changed, task
completed — whether done through `/architect`, `/run-manager`, or a direct
manual edit. Treat a TASK_LOG.md edit and its ClickUp sync as one atomic
step (see `architect.md`'s "TASK_LOG.md always up to date" invariant).

## Steps

### 1. New task added to TASK_LOG.md (⏳ TODO row)

`clickup_create_task` with:
- `parent`: `config.tools.clickup.parent_task_id`
- `list_id`: `config.tools.clickup.list_id`
- `name`: short title (≤ ~70 chars — the headline, not the full TASK_LOG paragraph)
- `markdown_description`: 2-4 sentence summary (what / why / root cause if known, plus the Goal column's value so the business rationale travels with the task) — not the full verbose TASK_LOG text
- `start_date`: the TASK_LOG "Started" date (YYYY-MM-DD)
- `due_date`: only if a real deadline exists; otherwise omit
- `status`: mapped per the table below

Record the returned ClickUp task ID back into the TASK_LOG.md row (append
`[CU: <id>]` to the Task cell) so later updates can find it without searching.

### 2. Status changes (🔄 IN_PROGRESS / ✅ DONE / ⚠ BLOCKED)

- `clickup_update_task` on the mirrored subtask's ID, `status` mapped per the table below.
- `clickup_create_comment` on that subtask with a short log of what changed — mirror the TASK_LOG update, condensed to a few lines, not pasted verbatim.

### 3. Never backfill

If asked to "sync everything" or "catch up ClickUp," only sync rows dated
on/after `config.tools.clickup.mirror_from_date`. Confirm with the user
before touching earlier rows.

## Status mapping (TASK_LOG → ClickUp)

| TASK_LOG | ClickUp status |
|---|---|
| ⏳ TODO | `to do` |
| 🔄 IN_PROGRESS | `in progress` |
| ✅ DONE | `done` |
| ⚠ BLOCKED | `stuck` |

## Key invariants

- Parent task/list come only from config — never hardcode an ID here, and
  don't create subtasks anywhere else without being told.
- TASK_LOG.md stays the verbose source of truth; ClickUp gets the headline, timeline, and status only.
- If the ClickUp MCP is unavailable, unauthenticated, or `config.tools.clickup.enabled` is `false`, don't block the TASK_LOG.md write — note the sync was skipped (one line) and continue. TASK_LOG.md always takes priority.
- Never invoke for entries dated before `config.tools.clickup.mirror_from_date`.
