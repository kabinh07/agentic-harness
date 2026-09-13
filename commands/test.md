---
name: test
description: Thin command wrapper around the standing agentic-harness:test-writer agent's TDD modes -- 'red' writes failing tests from a task spec before implementation exists, 'green' re-confirms those same tests now pass, and no-args/description falls back to characterization tests for already-existing code. Never edits production code, never marks a TASKS.md row done (that's architect's job).
---

# /agentic-harness:test

Runs the same test-generation-and-gate steps `/agentic-harness:architect`
runs internally (Phase 3 b/d), invocable directly when you want the RED
or GREEN half of the TDD cycle without going through a full architect
dispatch — e.g. you want failing tests for something before you (or a
subagent) write it, or you want the existing tests re-confirmed after
you finished it.

## Usage

```
/agentic-harness:test red <task/spec description>   # RED: write failing tests, before implementation exists
/agentic-harness:test green                          # GREEN: re-run the RED-phase tests, confirm they now pass
/agentic-harness:test                                # no spec, a diff already exists: characterization mode (retrofit)
/agentic-harness:test <description>                  # same, with an explicit description of what already changed
```

The last two forms are for code that's already been written outside the
RED→GREEN flow (hand-edited, or legacy code with no tests yet) — they
produce characterization tests, not TDD tests, and the report says so
explicitly. Prefer `red` first whenever there's no implementation yet.

## Steps

### `red <description>`
1. Determine the owning segment, if any, from
   `planning/project.config.yaml`'s `architecture.segments` (match by
   `owns_paths` or by asking) — this gives the right `test_command` and
   conventions to hand the test-writer. If none is inferable, say so and
   proceed with the project's full test command instead.
2. Dispatch `agentic-harness:test-writer` (Agent tool, `subagent_type:
   agentic-harness:test-writer`) in **Mode 1 (RED)** with: the task/spec
   description, the segment's `owns_paths` (if any), and — if inferable —
   the `planning/BUSINESS_GOALS.md` goal or `TASKS.md` row this serves.
3. Report the tests written and the confirmed failure output. Do not
   proceed to implement anything — that's a separate step (by you, a
   subagent, or a full `/agentic-harness:architect` dispatch).

### `green`
1. Dispatch `agentic-harness:test-writer` in **Mode 2 (GREEN)**, pointing
   it at the same tests a prior `red` invocation (or an architect RED
   phase) produced.
2. Run the resulting test command and report the real pass/fail output —
   never treat a rewritten/loosened test as a pass.

### no args / `<description>` (characterization, retrofit)
1. Determine the change: use the arg if given; otherwise inspect the
   working tree diff. If there's no diff and no arg, ask what changed —
   don't guess at scope.
2. Determine the owning segment as above.
3. Dispatch `agentic-harness:test-writer` in **Mode 3 (characterization)**
   with the change (diff or description given), the segment's
   `owns_paths` (if any), and the goal if inferable.
4. Run the resulting test command and report pass/fail — label the
   report as characterization tests, not TDD tests.

## Rules

- Never edit production/implementation code yourself — if the change
  looks untestable as written, report that instead of working around it.
- Never mark anything in `TASKS.md` done — this command doesn't own task
  status, `/agentic-harness:architect` does. If invoked mid-architect-run,
  it's just Phase 3 b/d done standalone; the architect flow still owns
  the row.
- Never weaken or delete an existing test to make the suite green, and
  never let a `green` invocation rewrite the tests it's checking.

## Report (≤6 lines)

```
Mode: RED / GREEN / characterization
Change tested: <task spec, or summary of what already exists>
Segment: <name, or "none/full suite">
Tests added/changed: <summary>
Result: <RED confirmed (failure output) | PASS/FAIL (n passed, n failed)>
Next: <e.g. "implement to pass these tests, then /agentic-harness:test green" if RED, or "run /agentic-harness:architect to mark this task done" if relevant>
```
