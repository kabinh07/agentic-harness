---
name: doctor
description: Compare this project's scaffolded files (CLAUDE.md, TASKS.md, planning/*) against the currently installed plugin's templates and report what's missing or stale -- a project bootstrapped before a template update (new config keys, new planning stages, new engineering-standard bullets) has no other way to find out. Report-only by default; --fix applies only the provably safe additions per file (never touches real project data -- task rows, business goals, filled-in config values, or CLAUDE.md, which stays report-only always).
---

# /agentic-harness:doctor

`/agentic-harness:init` copies `${CLAUDE_PLUGIN_ROOT}/templates/*` into a
project **only if the destination doesn't already exist** — by design, so
it never overwrites real content. That also means every already
-bootstrapped project is frozen at whatever the templates looked like on
install day. When this plugin's templates evolve — a new
`planning.stages` entry, a new config flag, a new
`ENGINEERING_STANDARDS.md` non-negotiable, an updated `TASKS.md` header —
an existing project has no way to find out it's missing them. This
command is that check: it diffs the project's live scaffolded files
against the *currently installed* plugin's own `templates/` (a live
structural comparison each run, no version-marker bookkeeping needed) and
reports drift, file by file.

This is a different tool from this repo's own `.claude/commands/check-
structure.md`, which audits agentic-harness's *own* internal doc
consistency for maintainers of the plugin and never ships to installers.
`doctor` ships in `commands/` like any other command — every project that
installs this plugin gets it.

## Usage

```
/agentic-harness:doctor          # report only, no edits
/agentic-harness:doctor --fix    # apply the safe additions below, then re-report what's left
```

## Precondition

If `CLAUDE.md`, `TASKS.md`, or `planning/project.config.yaml` don't exist
at all, stop and tell the user to run `/agentic-harness:init` first —
there's nothing scaffolded yet to reconcile.

## Per-file rules

Every scaffolded file mixes harness-authored boilerplate with real
project data differently, so each gets its own rule. **The governing
principle throughout: `--fix` only ever adds something the template has
and the project doesn't — it never edits an existing value, never removes
anything, never reorders anything already there.**

### `CLAUDE.md` — report only, never auto-fixed

Compare against `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md` section by
section (Load order, Bootstrapping a new project, Agents, Subagents).
Report which sections differ. **Never auto-fix this file, even under
`--fix`** — `init.md` already treats it as likely hand-customized ("if it
doesn't reference the harness's load order, don't touch it, tell the user
to merge by hand"); this command follows the same precedent, since there's
no reliable way to tell a stale copy from a deliberate customization.

### `TASKS.md` — fix the static header only

Everything **above** the `| # | Task | Goal | Source | Priority | Sprint |
Status | Agent | Started | Completed |` header row (the intro blockquote,
`## Status legend`, `## Source convention`, `## Priority`, `## Sprint`) is
harness-authored boilerplate. Diff it against
`${CLAUDE_PLUGIN_ROOT}/templates/TASKS.md`'s equivalent portion. Report if
it differs. `--fix` replaces only that header portion, byte-for-byte from
the current template — the table, every row in it, `## Completed Tasks`,
and `## Manager Run History` are never touched, read or reasoned about for
this comparison.

### `planning/project.config.yaml` — additive key-by-key only

Walk every key in `${CLAUDE_PLUGIN_ROOT}/templates/planning/project.config.yaml`,
including nested ones (every `planning.stages.<name>`, every `tools.<name>`
field, etc.). For each key **absent** from the project's file, queue an
addition: the template's default value and comment, inserted at the
position matching its neighbors in the template (e.g. a new
`planning.stages.dfd` entry goes between `design` and `erd` if the
project's file already has both of those). For each key **present** in
both, compare nothing beyond presence — never touch its value, never
"refresh" its comment, never reorder it. Report every queued addition with
its exact YAML snippet before `--fix` applies anything.

### `planning/ENGINEERING_STANDARDS.md` — append missing bullets, two sections only

Diff the bullet lists under `## Non-negotiable` and `## Judgment calls`
(architect enforces at review time) against
`${CLAUDE_PLUGIN_ROOT}/templates/planning/ENGINEERING_STANDARDS.md`'s
versions of those same two sections, matching by bullet text. Report any
template bullet missing from the project's copy. `--fix` appends the
missing bullet(s) to the matching section, preserving everything else in
the file exactly. **Never touch `## Project-specific additions`** — it's
entirely project-authored, not template-tracked, in either mode.

### `planning/README.md` — regenerated from config, not diffed against the template

This file's own convention (`planning-protocol.md`'s Bookkeeping section)
already makes `planning/project.config.yaml`'s `planning.stages` map the
source of truth for its stage table. So don't diff this file against the
*template* directly — instead, after the config fix above has run (or
against the project's current config if running report-only), add a row
for any stage present in `planning.stages` with no matching row in this
file's table, at whatever `status` the map records, in canonical stage
order (`brd, srs, design, dfd, erd, features, adr, epics`). Never touch an
existing row's Version/Status/Approved values. This means a `--fix` run
that adds `planning.stages.dfd`/`erd` to the config in the same pass also
gets those stages' rows added here — one `--fix` invocation, both files
land in sync.

### `planning/BUSINESS_GOALS.md` — skip once configured

If `project.configured: true`, skip entirely and say so — this file is
real content generated by `/agentic-harness:configure`, not
template-tracked once written. If `project.configured` is still `false`,
there's nothing to reconcile either: `init.md` already guarantees this
file is untouched from the template until configuration runs.

### `planning/{input,adr,design-assets,versions}/`

Trivial additive check: create any of these directories (with their
`.gitkeep`) that don't exist. Always safe, in either mode.

## Report

```
CLAUDE.md:                          up to date | N section(s) differ (report-only, never auto-fixed)
TASKS.md:                           up to date | header differs from template
planning/project.config.yaml:       up to date | missing N key(s): <list>
planning/ENGINEERING_STANDARDS.md:  up to date | missing N bullet(s): <list>
planning/README.md:                 up to date | missing N stage row(s): <list>
planning/BUSINESS_GOALS.md:         skipped (configured) | up to date (not yet configured)
planning/{input,adr,design-assets,versions}/: present | created N missing dir(s)

<if --fix> Fixed: N · Left for manual review: N (CLAUDE.md's diffs always land here, never applied)
Next: <e.g. "re-run /agentic-harness:doctor to confirm" if anything was fixed>
```

## Rules

- Never touch real project data in either mode: task rows, business
  goals, filled-in config values, segment/swarm definitions, or anything
  under `ENGINEERING_STANDARDS.md`'s Project-specific additions.
- Never auto-fix `CLAUDE.md` — report only, always.
- `--fix` only adds what's missing; it never edits, removes, or reorders
  what's already there in any file.
- If a project's file is *ahead* of the installed template in some way
  (e.g. a key the template no longer has), report it as a note, not an
  error or a removal candidate — the template may simply not have caught
  up, or the project may have a deliberate local addition.
