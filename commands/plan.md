---
name: plan
description: Planning-phase driver — figures out where the project starts (idea, existing codebase, existing BRD, or existing SRS) and walks BRD -> SRS -> design (optional) -> features -> ADRs -> epics, asking whatever it needs and never inventing an answer. Resumable at any stage.
---

# /agentic-harness:plan

Single entry point for the whole planning phase — the layer between "I have
an idea" and "TASKS.md has a real backlog." Doesn't do the planning work
itself; it orients (what stage are we at, what's the entry point), then
invokes each stage command in order, honoring gates and letting the user
stop/resume at any point. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md` for the questioning,
approval-gate, and bookkeeping rules every stage shares.

## Invocation

```
/agentic-harness:plan                # resume at the first non-approved stage, walk forward
/agentic-harness:plan status         # print the stage table, stop
/agentic-harness:plan <stage>        # jump straight to one stage (brd|srs|design|features|adr|epics)
```

## Phase 0 — Preconditions

If `planning/project.config.yaml` doesn't exist, stop and tell the user to
run `/agentic-harness:init` first.

## Phase 1 — Determine entry point (first run only)

If `planning.entry_point` is already set, skip this phase.

1. Check, in order: does `planning/SRS.md` exist? `planning/BRD.md`? any
   non-`.gitkeep` file in `planning/input/`? does the project have real
   code beyond the harness scaffold itself?
2. Present what you found via `AskUserQuestion` and let the user confirm or
   correct — don't assume "there's a `src/`" means "start from
   existing-project" if the user says it's an unrelated scratch repo.
3. Record `planning.entry_point`: `idea` | `existing-project` | `brd` | `srs`.
4. If `existing-project`: note this for `/agentic-harness:brd` and
   `/agentic-harness:adr` — they'll dispatch `agentic-harness:codebase-analyst`
   rather than starting from a blank BRD.

## Phase 2 — Shared scope questions (first run only)

Ask once, batched in one `AskUserQuestion` call, and record:
- **Has UI?** (yes / no / later) → `planning.has_ui`. `no` means the design
  stage is skipped entirely and never re-asked unless the user later runs
  `/agentic-harness:plan design` themselves. `later` means ask again next run.
- **Design source**, only if `has_ui` isn't `no`: which external tool will
  produce the actual design (Claude design / Stitch / Figma / other/none
  yet) → `planning.design_source`. This doesn't block anything; it's
  context `/agentic-harness:design`'s brief mode uses to phrase its output
  usefully.

## Phase 3 — Walk stages

Stage order: `brd` → `configure` (existing command, not part of this file's
scope beyond invoking it) → `srs` → `design` (skippable) → `features` →
`adr` → `epics`.

For `/agentic-harness:plan` (no stage arg): starting from
`planning.entry_point`'s natural starting stage (usually `brd`, or `srs` if
entry point was `srs`), invoke each stage's command in turn. Before
invoking a stage, check its gate (each stage command states its own
precondition, typically "predecessor `approved`"); if the gate fails, stop
and tell the user which predecessor needs attention — don't skip ahead.
After `brd` reaches `approved`, invoke `/agentic-harness:configure` before
moving to `srs`, so `planning/BUSINESS_GOALS.md` exists as the rubric later
stages reference.

For `/agentic-harness:plan <stage>`: jump straight there; that stage's own
gate still applies.

After each stage completes (approved or skipped), report its outcome in
one line and continue to the next unless the user stops you.

## Phase 4 — Handoff

Once `epics` reaches `approved` (or the user stops the walk early), report:

```
Planning phase: <N>/6 stages approved (<list any skipped/pending>)
TASKS.md: <N> rows seeded (or "not yet — epics stage still pending")
planning/README.md: up to date
Next: run /agentic-harness:architect to start implementation
```

## Status mode

`/agentic-harness:plan status` reads `planning.stages` from
`planning/project.config.yaml` and prints the same table
`planning/README.md` carries, plus current `entry_point`/`has_ui`/
`design_source`. Makes no changes.

## Key invariants

- Never silently pick an entry point, has_ui answer, or design source —
  confirm via `AskUserQuestion` even when the evidence looks obvious.
- Never invoke a stage whose gate isn't satisfied without the user
  explicitly overriding.
- `planning/README.md` and `planning/project.config.yaml`'s `planning.*`
  block stay in sync after every stage — this file's job across the whole
  walk, not any one stage command's alone.
