---
name: check-structure
description: Audit this repo's own internal cross-references for staleness -- root CLAUDE.md's Layout tree vs actual files, planning-stage ordinals/counts, the stage set across project.config.yaml/planning/README.md/BRD_TO_LLF_FLOW.md/plan.md, command-list membership in README.md/templates/CLAUDE.md, and each planning stage's gate chain. Report-only by default; --fix applies mechanical corrections. Repo-local dev tool -- not part of the shipped plugin payload, never appears as /agentic-harness:check-structure to a consuming project.
---

# /check-structure

This repo (`agentic-harness`) documents its own structure in half a dozen
places that all have to agree: the `commands/`/`agents/`/`docs/`/`templates/`
directories themselves, root `CLAUDE.md`'s Layout tree, `README.md`'s
command lists, `templates/CLAUDE.md`'s command lists,
`templates/planning/project.config.yaml`'s `planning.stages` map,
`templates/planning/README.md`'s stage table, `docs/BRD_TO_LLF_FLOW.md`'s
stage table + mermaid diagram, and each planning-stage command's ordinal
line ("Fourth planning stage...") and `## Gate`. Every time a stage or
command gets added, renamed, or removed, all of these have to be updated
together — this command is the mechanical check that they actually were.

This is a **repo-local dev tool**, deliberately kept in `.claude/commands/`
rather than `${CLAUDE_PLUGIN_ROOT}/commands/` — it audits *this repository's
own* files, which means nothing to a project that installs the plugin.
Never ship this file as part of the plugin payload.

## Canonical facts this check is built on

- **Planning stages, in order**: `brd`, `srs`, `design` (optional), `dfd`,
  `erd` (optional), `features`, `adr`, `epics` — 8 total. `design` and
  `erd` are the only two ever allowed a `skipped` status
  (`planning-protocol.md`'s Approval-gate skip table is the source of
  truth for this list — re-read it if this command hasn't been updated in
  a while, don't hardcode a stale copy blindly).
- **Bootstrap/execution commands**: `init`, `configure`, `architect`,
  `manager`, `test`, `clickup-log` (or whatever `commands/` actually
  contains that isn't a planning stage — Check D below derives this from
  disk, it doesn't hardcode the list).
- **Standing agents**: whatever exists under `agents/*.md`.

## Usage

```
/check-structure          # report only, no edits
/check-structure --fix    # apply mechanical fixes, then re-report what's left
```

## Checks

### A — Layout tree vs disk

Parse root `CLAUDE.md`'s `## Layout` fenced block. Diff its listed
`commands/*.md`, `agents/*.md`, `docs/*.md`, and `templates/**` entries
against `ls commands/*.md agents/*.md docs/*.md` and the actual
`templates/` tree (recursive). Report:
- Files on disk not listed in the tree ("undocumented addition").
- Entries in the tree with no matching file ("stale removal").

### B — Stage ordinal & count consistency

For each of the 8 canonical stage files (`commands/<stage>.md`), check its
opening ordinal line ("First planning stage." / "Second planning
stage." / ... / "Eighth and final planning stage.") matches its position
in the canonical order above. Then repo-wide grep for every place a stage
*count* is stated as a number or number-word (`"N/8 stages"`, `"eight
planning"`, `"eight stage"`, `"8 stage"`, and any leftover `"six stage"` /
`"six planning"` / `"/6 stages"` from before `dfd`/`erd` existed) and flag
any that doesn't say 8.

### C — Stage set across artifacts

Collect the stage set, in order, from each of:
1. `templates/planning/project.config.yaml`'s `planning.stages:` map keys.
2. `templates/planning/README.md`'s stage table rows.
3. `docs/BRD_TO_LLF_FLOW.md`'s stage-by-stage table rows and its mermaid
   flowchart's node sequence.
4. `commands/plan.md`'s stage-order prose (Phase 3) and its `<stage>` arg
   list (Invocation section).

Compare all four pairwise. Report any stage present in one list and
missing from another, and any ordering disagreement between lists that
are supposed to be sequential.

### D — Command-list membership

For every `commands/*.md` file, read its frontmatter `name:`. Confirm
`/agentic-harness:<name>` appears in:
- `README.md`'s command bullets (Planning phase / Bootstrap & execution
  sections, whichever applies).
- `templates/CLAUDE.md`'s command lists (same split).

Do the same for every `agents/*.md` file against README.md's Agents
bullets and `templates/CLAUDE.md`'s standing-agent references. Report:
- A command/agent file with no reference anywhere ("undocumented").
- A bullet/reference naming a command/agent file that doesn't exist
  ("dangling reference").

### E — Gate chain

For each of the 8 planning-stage command files, extract its `## Gate`
section's `planning.stages.<x>.status` reference(s). Confirm it names the
stage immediately preceding it in the canonical order (§ Canonical facts
above), tolerating the documented approved-or-skipped exception for
`design`/`erd`. Report any gap (a stage with no gate, or a gate pointing
at the wrong predecessor) and any cycle (a stage gating on itself or on a
later stage).

## Fix mode (`--fix`)

Only apply corrections that are mechanical and unambiguous given the
canonical facts above:
- Correct a stale ordinal word or count number to the right value.
- Add a missing command/agent bullet to README.md/templates/CLAUDE.md, in
  the section matching where its sibling commands already live — flag the
  placement for a quick human glance rather than assuming it's exactly right.
- Add a missing Layout-tree line for a file that exists on disk but isn't
  listed, positioned next to its nearest sibling in the existing order.

**Never** delete a file, and never remove a documented reference to a file
that's missing on disk without asking first — a "stale removal" might mean
the file was renamed and the rename wasn't finished, not that the
documentation is simply wrong. Report those for manual resolution even in
`--fix` mode.

After applying fixes, re-run all five checks and report what's still open.

## Report

```
Check A (Layout tree):        PASS | N issues
Check B (ordinals/counts):    PASS | N issues
Check C (stage set):          PASS | N issues
Check D (command membership): PASS | N issues
Check E (gate chain):         PASS | N issues

Issues (file:line — description), most actionable first:
...

<if --fix> Fixed: N · Left for manual review: N (listed above)
```
