---
name: configure
description: Bootstrap the whole harness from a BRD — fill project.config.yaml, generate business-logic.md, seed TASK_LOG.md, update CLAUDE.md
---

# /configure

Turns a dropped-in BRD into a working harness: `config/project.config.yaml`,
`docs/business-logic.md`, `TASK_LOG.md`, and this repo's own `CLAUDE.md`
header. Run once at project start, and again whenever the BRD changes
materially.

## Steps

### Step 1 — Find the BRD
Look in `BRD/` for any file that isn't `.gitkeep` (`.md`, `.txt`, `.pdf`,
`.docx`, etc). If none exists, stop and tell the user to drop one in `BRD/`
first — do not fabricate business logic from nothing.

If multiple files exist, read all of them; treat the most recently modified
as authoritative if they conflict, and flag the conflict to the user rather
than silently picking.

### Step 2 — Extract from the BRD
Read the BRD fully. Pull out:
- **Project name/description** — one line each.
- **Business goals, in priority order.** If the BRD doesn't state an
  explicit order, infer one from emphasis/repetition/"must have" vs.
  "nice to have" language, and flag the inferred order to the user for
  confirmation rather than asserting it silently.
- **Success criteria / metrics** — anything measurable (e.g. "95% on-time",
  "zero data loss", "under 2s response").
- **Constraints / invariants** — hard rules the solution must never violate.
- **Tools/systems mentioned** — ClickUp, Jira, GitHub, Slack, a specific
  API, a database, etc. Note only what's *mentioned*; don't assume a tool
  is wanted just because it's common.

### Step 3 — Write `docs/business-logic.md`
Replace the placeholder content with the extracted goals (priority-ordered
list), success criteria, and constraints. Keep it human-readable — this is
what a person skims to understand what the project is judged against, and
what `/run-manager` uses as its rubric.

### Step 4 — Fill `config/project.config.yaml`
- `project.name`, `project.description` from Step 2.
- `manager.business_goals`: same priority-ordered list as
  `docs/business-logic.md`, condensed to short strings — these two files
  must stay in sync.
- `manager.analysis_command` / `manager.report_path`: leave blank if the
  project has no analysis tooling yet (typical for a brand-new project) —
  don't invent a script that doesn't exist. Note in the report (Step 6)
  that these need filling once such tooling exists.
- `tools.*`: for each tool mentioned in the BRD, set `enabled: true`. Any
  ID that requires a real external lookup (a ClickUp parent task ID, a
  GitHub repo slug the user hasn't stated, a Slack channel ID) — do NOT
  guess. Use AskUserQuestion to collect exactly the missing IDs, batched
  into one question set.
- Set `project.configured: true` only after the above is actually filled
  in (name/description/goals at minimum — tool IDs can stay pending if the
  user defers them, but note that in the report).

### Step 5 — Seed `TASK_LOG.md`
Leave the schema as-is (table header, status legend, source convention).
Only touch `_Last updated:` (set to today) and, if any `tools.*.enabled` is
true, confirm the header note about mirroring already names the right
skill/date — update `mirror_from_date` in config to today if this is the
first `/configure` run.

### Step 6 — Update `CLAUDE.md`
Nothing structural changes — the load order and agent descriptions stay
generic. Do not inline project specifics into `CLAUDE.md` itself; they
belong in `config/project.config.yaml` and `docs/business-logic.md`. If a
prior `/configure` run left stale placeholder text anywhere, clean it up,
but the file's job is to point at config, not duplicate it.

### Step 7 — Report (≤10 lines)
```
Project: <name>
Goals extracted: N (see docs/business-logic.md)
Tools enabled: <list, or "none">
Tools needing IDs still: <list, or "none">
manager.analysis_command: <set / not yet — project has no analysis tooling>
config.project.configured: true/false
Next: <e.g. "run /architect to start", or "add analysis_command once a report generator exists">
```
