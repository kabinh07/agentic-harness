# Planning protocol (shared by all planning-phase commands)

`/agentic-harness:brd`, `:srs`, `:design`, `:features`, `:adr`, and `:epics`
all follow this protocol instead of restating it. `/agentic-harness:plan`
follows it for its own cross-stage bookkeeping. If a stage command's own
file says something more specific, the stage file wins — this is the
default, not an override.

## Why a shared protocol

Six stage commands doing the same three things (asking, gating, recording)
should do them identically, or a teammate reading `planning/BRD.md` today
and `planning/FEATURES.md` next week hits a different convention each time.
This file is the one place those conventions live.

## Questioning

- Batch related questions into one `AskUserQuestion` call — it caps at 4
  questions per call, so group by topic (e.g. all four persona questions
  together, not one call per persona).
- Don't dump every open question on the user at once across unrelated
  topics. Work through one section of the artifact at a time, ask what
  that section needs, move on.
- When you have a reasonable default or a most-likely reading, make it the
  first option — the user is picking, not brainstorming from scratch.
- If the user answer implies a follow-up edge case (e.g. "yes, multi-tenant"
  implies "so what isolates tenants?"), ask that follow-up before moving on
  — don't note it as an open question when it was one question away from
  being answered.
- Prefer specific questions over open-ended ones once you have enough
  context to be specific ("Should login lock out after N failed attempts,
  and if so what's N?" beats "Any auth edge cases?").

## Never fabricate

- No detail invented to fill a gap, ever — not a plausible-sounding
  success metric, not a guessed persona, not an assumed NFR.
- If a question genuinely has no answer yet (user defers, doesn't know,
  says "decide later"), the item goes under that artifact's
  `## Assumptions (unvalidated)` section with a ⚠ marker, phrased as what
  was assumed and why, and is also added to `planning/README.md`'s
  Assumptions (unvalidated) log with a pointer back to the artifact.
- Inference from an existing codebase (via `agentic-harness:codebase-analyst`)
  is not fabrication — it's evidence-based — but it must be presented to the
  user as an inference to confirm or correct, never asserted silently as
  fact in the final artifact. Once confirmed, it's no longer an assumption.

## Approval gate

Every stage ends the same way:

1. Present a concise summary of what the artifact says (not the whole file
   dumped back at the user — the shape and key points).
2. Ask: approve as-is / revise (what to change) / skip this stage for now.
3. **Approve** → write the final artifact, set
   `planning.stages.<stage>.status: approved` and `approved_on: <today>` in
   `planning/project.config.yaml`.
4. **Revise** → apply the requested changes, re-present, loop.
5. **Skip** → only valid where the stage itself allows it (currently only
   `design`, gated on `planning.has_ui`) — set `status: skipped`, write
   nothing, and don't re-ask on future `/agentic-harness:plan` runs unless
   the user explicitly revisits it (`/agentic-harness:plan design`).

A later stage that requires an earlier one to be `approved` (see each
stage's own gate) refuses to run against a `draft` predecessor — tell the
user which stage is blocking and why, don't proceed anyway without the
user overriding explicitly.

## Bookkeeping

After every stage — approved, revised, or skipped — update, as one atomic
step:
- `planning/project.config.yaml`'s `planning.stages.<stage>`.
- `planning/README.md`'s stage status table, Open Questions section, and
  Assumptions (unvalidated) log.

Never let these drift out of sync with each other or with the artifact
files themselves — `planning/README.md` is what a teammate reads first, so
it must always reflect reality without them having to open every artifact.

## Traceability

Every artifact after the BRD should be able to point back up the chain:
SRS requirement → BRD goal; Feature → SRS requirement(s); ADR → Feature/SRS
requirement(s) it enables; Epic/Story/Task → Feature → SRS requirement →
BRD goal. A stage command that finds an item with no upstream trace
reports it as an orphan rather than silently keeping or dropping it — see
each stage's own orphan-check step.
