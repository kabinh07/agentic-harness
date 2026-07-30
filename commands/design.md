---
name: design
description: Two modes -- 'brief' emits planning/DESIGN_BRIEF.md from the SRS for an external design tool (Claude design, Stitch, Figma); default distills whatever design output landed in planning/design-assets/ into planning/DESIGN.md, a design-consistency context file. This plugin never designs; it briefs and then documents. Skippable entirely for projects with no UI.
---

# /agentic-harness:design

Third planning stage, and the only optional one. This plugin does not
produce design — you design externally (Claude design, Stitch, Figma,
whatever). What this command does: turn the approved SRS into a design
brief you can hand to that tool, and, once you've designed, distill the
result into a consistency-context file implementers and the design tool
itself can be pointed back at. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

## Gate

`planning.stages.srs.status` must be `approved`.

## Skip check (do this first, every invocation)

If `planning.has_ui == false`: report `Design stage: skipped (no UI)` and
stop — do not ask again this session. If `planning.has_ui == "later"`: ask
now whether it's resolved; if still undecided, stay `pending` and stop.

## Mode: `brief`

`/agentic-harness:design brief`

1. Read `planning/SRS.md`. Derive the screen/flow inventory from FRs ×
   user classes — every FR that implies a user-facing surface gets at
   least one screen or state.
2. Ask (batched): brand/tone descriptors, if any exist already; target
   platforms/viewports; accessibility target (e.g. WCAG AA); anything the
   user already knows they want visually (reference products, existing
   brand assets).
3. Write `planning/DESIGN_BRIEF.md`:
   ```markdown
   # Design Brief

   ## Product summary
   ## Users
   ## Screen / flow inventory
   (derived from SRS FRs, one entry per screen/state, FR refs)
   ## Key interactions
   ## Brand / tone
   ## Accessibility target
   ## Platforms / viewports
   ## Ask list
   (explicit asks for the design tool/session — copy-paste ready)
   ```
4. Report: tell the user this is ready to paste into
   `planning.design_source` (or whichever tool they're using), and that
   once they've designed, dropping the output into
   `planning/design-assets/` and running `/agentic-harness:design` (default
   mode) produces the consistency file.

## Mode: default (consistency context)

`/agentic-harness:design`

1. Read everything in `planning/design-assets/` — HTML, screenshots, token
   exports, style guides, whatever landed there. If it's empty, tell the
   user to drop design output there (or run `brief` mode first if they
   haven't designed yet) and stop.
2. Extract what you can directly (colors, type scale, spacing units, if
   present in machine-readable form); ask about anything ambiguous or only
   visible in a screenshot (exact hex values, spacing scale, breakpoints,
   motion/transition conventions) rather than guessing pixel values.
3. Write `planning/DESIGN.md` — a CLAUDE.md for design, not a design asset:
   ```markdown
   # Design Context

   > Consistency reference for implementers. Canonical design source:
   > <link/path to the real design file, e.g. planning/design-assets/...>

   ## Tokens
   Color, typography, spacing, radius, shadow, motion — as a table or list,
   each with its intended use.

   ## Component inventory
   Component name, states (default/hover/focus/disabled/error), notes.

   ## Layout / grid rules
   ## Accessibility rules
   (contrast minimums, focus visibility, keyboard nav expectations)

   ## Do-nots
   (explicit anti-patterns to avoid — e.g. "never introduce a second
   font family", "never use a color outside the token list")

   ## Canonical source
   Where the actual design file/prototype lives.
   ```

## Approval gate

Per `planning-protocol.md`, for whichever mode produced a final artifact
(brief mode doesn't gate the way default mode does — a brief is an
intermediate hand-off, not a stage-completing artifact; only default
mode's `planning/DESIGN.md` sets `planning.stages.design.status`).

## Report (≤6 lines)

```
Mode: brief/default
Written: <planning/DESIGN_BRIEF.md or planning/DESIGN.md, or "skipped">
Open questions: N
Status: pending/draft/approved/skipped
Next: /agentic-harness:features
```
