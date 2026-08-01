---
name: configure
description: Bootstrap the whole harness from a BRD — fill project.config.yaml, generate BUSINESS_GOALS.md, seed TASKS.md, update CLAUDE.md
---

# /agentic-harness:configure

Turns an approved BRD into a working harness: `planning/project.config.yaml`,
`planning/BUSINESS_GOALS.md`, `TASKS.md`, and this repo's own `CLAUDE.md`
header. Normally invoked by `/agentic-harness:plan` right after
`/agentic-harness:brd` is approved, but can be run standalone. Run again
whenever the BRD changes materially.

## Steps

### Step 1 — Find the BRD
Look for `planning/BRD.md` first (written by `/agentic-harness:brd`). If it
doesn't exist, fall back to any file in `planning/input/` that isn't
`.gitkeep` (`.md`, `.txt`, `.pdf`, `.docx`, etc). If neither exists, stop and
tell the user to run `/agentic-harness:brd` (or drop material into
`planning/input/`) first — do not fabricate business logic from nothing.

If multiple raw files exist in `planning/input/`, read all of them; treat
the most recently modified as authoritative if they conflict, and flag the
conflict to the user rather than silently picking.

### Step 2 — Extract from the BRD
Read the BRD fully (see `planning-protocol.md`'s Document format — §3
Business Objectives, §6 Functional Requirements grouped by module with
M/S/C priority, §7 NFRs, §8 Assumptions & Constraints). Pull out:
- **Project name/description** — one line each, from the header table /
  §1 Purpose.
- **Business goals, in priority order.** Start from §3's Business
  Objectives (bulleted, in the order stated); cross-check against which
  modules carry mostly **M**-priority requirements in §6 vs. mostly
  **S**/**C** — a module that's all Should/Could-have is a lower-priority
  goal even if §3 didn't rank it explicitly. Flag any inferred ordering to
  the user for confirmation rather than asserting it silently.
- **Success criteria / metrics** — anything measurable in §3 or the
  requirement rows themselves (e.g. "95% on-time," "zero data loss,"
  "under 2s response").
- **Constraints / invariants** — §8 Assumptions & Constraints, plus §4.2
  Out of scope (what's explicitly excluded is itself a constraint).
- **Tools/systems mentioned** — ClickUp, Jira, GitHub, Slack, a specific
  API, a database, etc. Note only what's *mentioned*; don't assume a tool
  is wanted just because it's common.

### Step 3 — Write `planning/BUSINESS_GOALS.md`
Replace the placeholder content with the extracted goals (priority-ordered
list), success criteria, and constraints. Keep it human-readable — this is
what a person skims to understand what the project is judged against, and
what `/agentic-harness:manager` uses as its rubric.

### Step 4 — Fill `planning/project.config.yaml`
- `project.name`, `project.description` from Step 2.
- `manager.business_goals`: same priority-ordered list as
  `planning/BUSINESS_GOALS.md`, condensed to short strings — these two files
  must stay in sync.
- `manager.analysis_command` / `manager.report_path`: leave blank if the
  project has no analysis tooling yet (typical for a brand-new project) —
  don't invent a script that doesn't exist. Note in the report (Step 9)
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
If `planning/FEATURES.md` already exists (the planning phase reached the
features stage before this ran), seed one `architecture.segments` entry per
feature's "proposed owning area" — group features that named the same area
into one segment. Otherwise, seed one entry per **BRD §6 module**
(e.g. `FR-RUNTIME-*`, `FR-TOOL-*`, `FR-AUTH-*` — whatever modules this
BRD actually defined → one segment candidate per module prefix)
— this is usually a better starting signal than free-form component
naming, since the module boundaries were already chosen deliberately for
the requirement IDs. Either way: `name`, `description`, `trigger` (what
kind of task routes here). Leave `owns_paths`, `agent`, and `test_command`
blank — there's no code yet to own paths or a test command to run.
`/agentic-harness:architect` fills those in and creates the actual
`.claude/agents/<segment>.md` subagent the first time it runs against real
code (see `architect.md` Phase 1.5). If the BRD has only one module or is
too abstract to name real components yet, leave `segments: []` — don't
invent a breakdown from nothing; that's `/agentic-harness:architect`'s job
once there's code to look at.

### Step 6 — Confirm `planning/ENGINEERING_STANDARDS.md`
The generic defaults ship as-is. Only add to "Project-specific additions"
if the BRD states an explicit, concrete convention (a required framework,
a compliance rule, a hard performance budget) — don't pad it with generic
best-practice filler.

### Step 7 — Seed `TASKS.md`
Leave the schema as-is (table header, status legend, source convention,
Goal column). Only touch `_Last updated:` (set to today) and, if any
`tools.*.enabled` is true, confirm the header note about mirroring already
names the right skill/date — update `mirror_from_date` in config to today
if this is the first `/agentic-harness:configure` run. If
`/agentic-harness:epics` hasn't run yet, `TASKS.md` stays otherwise empty —
this command doesn't invent task rows itself.

### Step 8 — Update `CLAUDE.md`
Nothing structural changes — the load order and agent descriptions stay
generic. Do not inline project specifics into `CLAUDE.md` itself; they
belong in `planning/project.config.yaml` and `planning/BUSINESS_GOALS.md`.
If a prior `/agentic-harness:configure` run left stale placeholder text
anywhere, clean it up, but the file's job is to point at config, not
duplicate it.

### Step 9 — Update `planning/project.config.yaml`'s stage status
Set `planning.stages.brd.status: approved` (with today's `approved_on` and
`version` matching the BRD header table's current Version) if this ran
from an approved `planning/BRD.md` — `/agentic-harness:configure` doesn't
gate on this itself, but keeps the record consistent for
`/agentic-harness:plan` and `planning/README.md`.

### Step 10 — Report (≤10 lines)
```
Project: <name>
Goals extracted: N (see planning/BUSINESS_GOALS.md)
Segments seeded: N (or "none — no code/features yet, /agentic-harness:architect will segment on first run")
Tools enabled: <list, or "none">
Tools needing IDs still: <list, or "none">
manager.analysis_command: <set / not yet — project has no analysis tooling>
config.project.configured: true/false
Next: <e.g. "run /agentic-harness:srs to continue planning", or "run /agentic-harness:architect to start">
```
