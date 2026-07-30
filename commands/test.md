---
name: test
description: Thin command wrapper around the standing agentic-harness:test-writer agent -- dispatches it to write/extend tests for a described or inferred change, then runs the relevant test command and reports pass/fail. Never edits production code, never marks a TASKS.md row done (that's architect's job).
---

# /agentic-harness:test

Runs the same test-generation-and-gate step
`/agentic-harness:architect` runs internally (Phase 3c/3d), invocable
directly when you want tests for a change without going through a full
architect dispatch — e.g. you hand-edited something yourself and want the
independent test pass before calling it done.

## Usage

```
/agentic-harness:test                          # infer the change from the working diff
/agentic-harness:test <description or paths>   # be explicit about what changed
```

## Steps

1. Determine the change: use the arg if given; otherwise inspect the
   working tree diff. If there's no diff and no arg, ask what changed —
   don't guess at scope.
2. Determine the owning segment, if any, from
   `planning/project.config.yaml`'s `architecture.segments` (match by
   `owns_paths`) — this gives the right `test_command` and conventions to
   hand the test-writer. If the change spans multiple segments or none,
   say so and proceed with the project's full test command instead.
3. Dispatch the standing `agentic-harness:test-writer` agent (Agent tool,
   `subagent_type: agentic-harness:test-writer`) with: the change (diff or
   the description given), the segment's `owns_paths` (if any), and — if
   inferable — the `planning/BUSINESS_GOALS.md` goal or `TASKS.md` row this
   change serves. If none of that is inferable, tell test-writer so
   explicitly rather than fabricating a goal for it.
4. Run the resulting test command (segment's `test_command`, or the
   project's full suite if none) and report the real pass/fail output.

## Rules

- Never edit production/implementation code yourself — if the change looks
  untestable as written, report that instead of working around it.
- Never mark anything in `TASKS.md` done — this command doesn't own task
  status, `/agentic-harness:architect` does. If invoked mid-architect-run,
  it's just Phase 3c/3d done standalone; the architect flow still owns the
  row.
- Never weaken or delete an existing test to make the suite green.

## Report (≤6 lines)

```
Change tested: <summary>
Segment: <name, or "none/full suite">
Tests added/changed: <summary>
Result: PASS/FAIL (<n passed, n failed>)
Next: <e.g. "run /agentic-harness:architect to mark this task done" if relevant>
```
