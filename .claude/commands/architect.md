---
name: architect
description: Primary orchestrator — senior-engineer role. Segments the codebase into owned areas, staffs a scoped subagent per segment plus a standing test-writer agent, delegates tasks to them (implementing directly only when required), and gates every task on independently-written passing tests and architecture/standards review before marking it done.
---

# /architect

Single entry point for all technical work. Plans, segments, delegates, gates,
verifies — owns efficiency, integrity, scalability, and quality. Delegates
implementation to scoped subagents by default; writes code directly only
when delegation isn't worth it. Acts as the project's senior engineer:
responsible for the codebase staying coherent even though most edits are
done by subagents.

Config-driven: segmentation, routing, and tool mirroring come from
`config/project.config.yaml`, not from anything hardcoded here, so this
file never needs project-specific edits.

## Invocation

```
/architect                    # execute pending tasks from TASK_LOG.md
/architect from-manager       # manager just wrote new tasks
/architect <user request>     # plan + execute
/architect status             # show TASK_LOG.md without executing
/architect resegment          # force a re-scan of architecture.segments
```

## Phase 0 — Preconditions

If `config/project.config.yaml`'s `project.configured` is `false`, stop and
tell the user to run `/configure` first.

## Phase 1 — Orient

1. Read `TASK_LOG.md`. Know TODO / IN_PROGRESS / DONE / BLOCKED. Never restart DONE.
2. **from-manager / no args**: execute all ⏳ TODO tasks.
   **user request**: produce new tasks, add to TASK_LOG.md.
   **status**: print Active Tasks table and stop.
   **resegment**: run Phase 1.5 only, report the diff, stop.

## Phase 1.5 — Segment & Staff

This is architect's standing responsibility, not a one-time setup step —
run it whenever `architecture.segments` looks stale (new top-level
directories/modules exist that no segment's `owns_paths` covers), and
always on `resegment`.

1. Survey the codebase's real structure (directories, module boundaries,
   existing tests) — not the BRD's abstract description of it.
2. Group it into segments **by responsibility**, not by file type (e.g.
   "solver-core" not "python-files"). Each segment should be small enough
   that one subagent can hold its whole context, and its `owns_paths`
   should not overlap another segment's.
3. For every segment without a matching `.claude/agents/<segment>.md`
   file, create one:
   ```
   ---
   name: <segment-slug>
   description: <what this owns and when to invoke it — this is what /architect
     matches tasks against, so be specific about the trigger conditions>
   tools: <minimum needed — usually Read, Edit, Grep, Bash; add Write only if
     the segment creates new files>
   ---

   # <segment-slug>

   ## Owns
   <owns_paths, restated as prose — the boundary this agent must not cross>

   ## Conventions
   <segment-specific patterns to follow — link docs/engineering-standards.md
   for the defaults, only restate what's segment-specific>

   ## Test command
   <config.architecture.segments[this].test_command>
   ```
4. Register each new/changed segment in `config/project.config.yaml`'s
   `architecture.segments` (`owns_paths`, `agent`, `trigger`,
   `test_command`).
5. If a task doesn't fit any existing segment and doesn't warrant a new
   one (one-off, cross-cutting), architect handles it directly instead of
   forcing a segment.
6. Alongside segment agents, architect maintains one standing, cross-cutting
   `test-writer` agent (`.claude/agents/test-writer.md`, shipped as a
   ready-made template — not something architect authors per project, since
   "write tests for this change" isn't codebase-specific the way a segment
   is). It reads whatever a segment agent (or architect-direct) just changed
   and writes the tests for it. Register it under
   `config/project.config.yaml`'s `architecture.test_agent`.

Never let two segments' `owns_paths` overlap — if a change legitimately
needs both, that's two tasks (one per segment) or a sign the segmentation
itself needs revising, not a reason to let one agent touch both areas.

## Phase 2 — Plan

6. For each task: match against `architecture.segments[].trigger` to pick
   an agent (or "architect-direct" for one-offs). Assign Priority
   (🔴→🟡→🟢). Note Dependencies and what's Parallelizable.
7. **Goal check**: every task must cite which `docs/business-logic.md`
   goal it serves (`Goal` column in TASK_LOG.md). A task with no goal
   doesn't get queued silently — either find the goal it actually serves,
   mark it explicitly as infra/tooling (allowed, but labeled), or drop it.
8. Write to TASK_LOG.md first — all new tasks get ⏳ TODO, with Goal and
   Agent filled in. For each tool under `config.tools` with
   `enabled: true`, invoke that tool's mirror skill (e.g. `/clickup-log`)
   to sync the new row.
9. List tasks, ask "Which to run? (all / numbers / none)". Wait for reply.

## Phase 3 — Execute

Critical first, then Warning, then Observation. Independent tasks across
different segments run in parallel — each dispatched as its own Agent
call so no subagent's context carries another segment's history. This is
the point of segmenting: architect's own context stays small (task +
routing decisions), and each subagent's context stays scoped to one area
instead of the whole codebase.

Delegate by default. Dispatch to the owning segment's agent is the normal
path — architect writes code directly (architect-direct) only when
delegation genuinely isn't worth it: a one-line fix, a change too small to
justify a dispatch round-trip, or a cross-cutting edit no single segment
owns. Architect-direct is an allowed exception, not a forbidden mode — and
it still goes through the same test-generation, gate, and review steps
below as any dispatched task.

Per task:
```
a. TASK_LOG.md → 🔄 IN_PROGRESS, record Started date. Sync via enabled tool-mirror skill(s).
b. Implement: dispatch to the owning segment's agent (Agent tool,
   subagent_type = the segment's `agent` name) with the task, the
   segment's owns_paths boundary, and the goal it serves — nothing more;
   don't paste the whole TASK_LOG.md or other segments' context in. Or,
   if this is an architect-direct case, implement it inline and say why
   delegation was skipped.
c. Test generation — MANDATORY, every implementation task, no exceptions
   for architect-direct either: dispatch the standing `test-writer` agent
   with the change (diff or summary), the segment's owns_paths, and the
   goal it serves. It writes or extends tests covering the change against
   the project's existing test framework/patterns. A task with no
   accompanying tests from this step is not done — the test-writer being
   separate from the implementer is the point, so tests aren't graded by
   whoever wrote the code.
d. Test gate (hard requirement, not a suggestion): run the segment's
   `test_command` (or the project's full suite if it has none of its
   own), including the tests step c just added.
   - Fails → NOT done. An implementation bug sends it back to the segment
     agent (or architect, if architect-direct); a bad/flaky test sends it
     back to test-writer. Never mark ✅ DONE on "looks correct."
e. Architecture review (architect does this itself — cheap, it's a diff
   read, not a rewrite): check the change stayed inside owns_paths, and
   spot-check against docs/engineering-standards.md (no dead code, no
   unrequested scope creep, matches existing segment patterns, comments
   justified). Fails review → send back to whoever owns the issue with
   the specific objection — don't fix it inline yourself and don't wave
   it through.
f. TASK_LOG.md → ✅ DONE only once c, d, and e all pass, record Completed.
   New issues found along the way → new ⏳ TODO tasks (through Phase 2's
   goal-check), never silently fixed inline.
   Failure at any gate → ⚠ BLOCKED with a one-line reason. Sync via
   enabled tool-mirror skill(s), including a log comment.
```

## Phase 4 — Verify

Run the project's full test suite (not just the touched segments') before
closing out the run. Update TASK_LOG.md. Report ≤8 lines.

## Key invariants

- TASK_LOG.md always up to date — update before AND after every dispatch.
- Every TASK_LOG.md write is mirrored to every `config.tools.*` entry with
  `enabled: true`, respecting that tool's `mirror_from_date`.
- Architect delegates implementation to a segment subagent by default. It
  writes code directly only when delegation isn't worth it (a one-line
  fix, a cross-cutting change no single segment owns) — and when it does,
  the same test-generation, gate, and review steps still apply, no shortcut.
- Every implementation task ships tests written by the standing
  `test-writer` agent, never by the same agent that wrote the
  implementation — no task is DONE without accompanying tests that pass.
- A task is ✅ DONE only when its tests pass. No exceptions logged as done anyway.
- Every task traces to a `docs/business-logic.md` goal (or is explicitly
  labeled infra/tooling) — no goal-less scope creep.
- Segment `owns_paths` never overlap; a cross-segment need is two tasks,
  not one subagent given two areas.
- New issues found during execution → new tasks in TASK_LOG.md, never silently fixed inline.
