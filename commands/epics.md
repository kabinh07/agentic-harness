---
name: epics
description: Build planning/EPICS.md (Epics -> Stories with Given/When/Then acceptance criteria -> lower-level tasks) from FEATURES.md, BUSINESS_GOALS.md, and ADRs, then seed TASKS.md with the initial backlog, mirroring to any enabled external tool. Last planning stage -- hands off to /agentic-harness:architect.
---

# /agentic-harness:epics

Sixth and final planning stage. Turns the approved feature/ADR set into an
implementation-ready backlog: Epics → Stories → lower-level tasks, then
seeds `TASKS.md` so `/agentic-harness:architect` has real work queued from
its first run. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

## Gate

`planning.stages.adr.status` must be `approved`.

## Inputs

`planning/FEATURES.md`, `planning/BUSINESS_GOALS.md`, `planning/adr/`.

## Process

1. **Epics** — one per `planning/BUSINESS_GOALS.md` goal (1:1, so they
   never drift apart). Name it after the goal, not a technical area.
2. **Stories** — break each epic's features into stories:
   `As a <role>, I want <capability>, so that <benefit>`, with Given/When/Then
   acceptance criteria and a rough estimate. Ask the user to confirm
   estimates/ordering rather than asserting priority silently within an
   epic.
3. **Lower-level tasks** — decompose each story into implementation-sized
   tasks: small enough to land as one `/agentic-harness:architect` dispatch
   and one test-gate pass. Each task must trace Task → Story → Epic →
   Feature → FR → Goal; a task that can't trace this way gets fixed (find
   the real trace) or dropped, not queued anyway.

## Output — `planning/EPICS.md`

Follows `planning-protocol.md`'s Document format and Versioning — header
table with Version/Status, archive-then-write on any rewrite.

```markdown
# Epics

## <Project Name>

| | |
|---|---|
| **Document title** | <Project Name> — Epics |
| **Version** | 0.1 (Draft) |
| **Date** | <today> |
| **Based on** | FEATURES v<x>, ADRs |
| **Status** | For review |

---

## E-01: <name> (→ Goal: <business goal>)

### S-01: <story title>
As a <role>, I want <capability>, so that <benefit>.

**Acceptance criteria**
- Given ... when ... then ...

**Estimate:** ...
**Features:** F-##

#### Tasks
- [ ] T-01: <implementation-sized task> — traces to S-01/F-##/FR-<MODULE>-##/<goal>

(repeat per epic/story)

## Open Items (TBD)
1. <unresolved item> (§<epic/story it affects>)

---
*End of document — Draft v0.1. Open items: <1-line summary, or "none">.*
```

## Seed `TASKS.md`

For every T-## above, append a row using the existing schema:

```
| N | <task, specifics from EPICS.md> [E-##/S-##/F-##] | <goal> | Planning YYYY-MM-DD | 🔴/🟡/🟢 | ⏳ TODO | — | — | — |
```

- Priority derived from the story's epic → goal priority order in
  `planning/BUSINESS_GOALS.md` (top-priority goal's tasks default 🔴/🟡,
  lower-priority default 🟡/🟢) — use judgment, don't mechanically force
  every row to the same priority.
- Agent column stays `—`; `/agentic-harness:architect` resolves segment
  assignment at Phase 1.5/dispatch time using each feature's proposed
  owning area, not this command.
- For each tool under `config.tools` with `enabled: true`, invoke its
  mirror skill (e.g. `/agentic-harness:clickup-log`) to sync each new row,
  respecting `mirror_from_date`.
- Update `TASKS.md`'s `_Last updated:` line.

## Approval gate

Per `planning-protocol.md` (Versioning applies — archive before rewrite),
applied to the epic/story/task backlog as a whole. On approval, set
`planning.stages.epics.status: approved` + `approved_on`.

## Report (≤8 lines)

```
planning/EPICS.md: v<version> — N epics, N stories, N tasks
TASKS.md: N rows seeded
Traceability: all tasks trace to a goal (or list exceptions)
Status: draft/approved
Planning phase complete: <yes/no — list any stage still pending/draft>
Next: run /agentic-harness:architect to begin implementation
```
