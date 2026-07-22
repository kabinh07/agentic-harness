---
name: configure
description: Bootstrap the whole harness from a BRD — fill project.config.yaml, generate business-logic.md, seed TASK_LOG.md, update CLAUDE.md
---

# /agentic-harness:configure

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
what `/agentic-harness:run-manager` uses as its rubric.

### Step 4 — Fill `config/project.config.yaml`
- `project.name`, `project.description` from Step 2.
- `manager.business_goals`: same priority-ordered list as
  `docs/business-logic.md`, condensed to short strings — these two files
  must stay in sync.
- `manager.analysis_command` / `manager.report_path`: leave blank if the
  project has no analysis tooling yet (typical for a brand-new project) —
  don't invent a script that doesn't exist. Note in the report (Step 8)
  that these need filling once such tooling exists.
- `tools.*`: for each tool mentioned in the BRD, set `enabled: true`. Any
  ID that requires a real external lookup (a ClickUp parent task ID, a
  GitHub repo slug the user hasn't stated, a Slack channel ID) — do NOT
  guess. Use AskUserQuestion to collect exactly the missing IDs, batched
  into one question set.
- Set `project.configured: true` only after the above is actually filled
  in (name/description/goals at minimum — tool IDs can stay pending if the
  user defers them, but note that in the report).

### Step 5 — First-pass segmentation (`architecture.segments`)
If the BRD names distinct components/subsystems (e.g. "a scraper, a
scoring engine, a dashboard"), seed `architecture.segments` with one entry
per component: `name`, `description`, `trigger` (what kind of task routes
here). Leave `owns_paths`, `agent`, and `test_command` blank — there's no
code yet to own paths or a test command to run. `/agentic-harness:architect` fills those
in and creates the actual `.claude/agents/<segment>.md` subagent the first
time it runs against real code (see `architect.md` Phase 1.5). If the BRD
is too abstract to name real components yet, leave `segments: []` —
don't invent a breakdown from nothing; that's `/agentic-harness:architect`'s job once
there's code to look at.

### Step 6 — Confirm `docs/engineering-standards.md`
The generic defaults ship as-is. Only add to "Project-specific additions"
if the BRD states an explicit, concrete convention (a required framework,
a compliance rule, a hard performance budget) — don't pad it with generic
best-practice filler.

### Step 7 — Seed `TASK_LOG.md`
Leave the schema as-is (table header, status legend, source convention,
Goal column). Only touch `_Last updated:` (set to today) and, if any
`tools.*.enabled` is true, confirm the header note about mirroring already
names the right skill/date — update `mirror_from_date` in config to today
if this is the first `/agentic-harness:configure` run.

### Step 8 — Update `CLAUDE.md`
Nothing structural changes — the load order and agent descriptions stay
generic. Do not inline project specifics into `CLAUDE.md` itself; they
belong in `config/project.config.yaml` and `docs/business-logic.md`. If a
prior `/agentic-harness:configure` run left stale placeholder text anywhere, clean it up,
but the file's job is to point at config, not duplicate it.

### Step 9 — Report (≤10 lines)
```
Project: <name>
Goals extracted: N (see docs/business-logic.md)
Segments seeded: N (or "none — no code yet, /agentic-harness:architect will segment on first run")
Tools enabled: <list, or "none">
Tools needing IDs still: <list, or "none">
manager.analysis_command: <set / not yet — project has no analysis tooling>
config.project.configured: true/false
Next: <e.g. "run /agentic-harness:architect to start", or "add analysis_command once a report generator exists">
```
