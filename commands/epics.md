---
name: epics
description: Build planning/EPICS.md (plain-language Epics, scored WSJF, with EARS acceptance criteria, directly decomposed into implementation-sized Tasks) from FEATURES.md, BUSINESS_GOALS.md, and ADRs; decide delivery mode (whole project at once vs weekly sprints); then seed TASKS.md with the initial backlog, mirroring to any enabled external tool. Last planning stage -- hands off to /agentic-harness:architect.
---

# /agentic-harness:epics

Sixth and final planning stage. Turns the approved feature/ADR set into an
implementation-ready backlog: plain-language Epics decomposed straight into
implementation-sized Tasks (no intermediate story layer), scored and
sequenced, then seeds `TASKS.md` so `/agentic-harness:architect` has real
work queued from its first run. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

## Gate

`planning.stages.adr.status` must be `approved`.

## Inputs

`planning/FEATURES.md`, `planning/BUSINESS_GOALS.md`, `planning/adr/`.

## Process

1. **Epics** — one per `planning/BUSINESS_GOALS.md` goal (1:1, so they
   never drift apart).

   **Title in plain language, not a technical area.** An epic's title and
   its one-sentence user-visible outcome must read cleanly to someone
   outside engineering — no module names, no implementation nouns. Litmus
   test: *if you can't finish "When this ships, a `<role>` can `<do X>`,
   which they couldn't before" in one plain sentence, it isn't an epic —
   split it, or it's a chore, not an epic.* ("Customers can raise and
   track a support ticket," not "Ticket CRUD + status state machine.")

2. **Scope** — In scope / Out of scope (deferred to a later epic) bullets,
   confirmed with the user, not asserted.

3. **Acceptance criteria (EARS)** — per `planning-protocol.md`'s EARS
   section, one `EARS-<AREA>-#` line per behavior, each citing the FR/UC id
   it traces to. Cross-cutting NFRs from FEATURES/SRS bind here as EARS
   lines on the epic(s) they constrain — never a separate "NFR epic."

4. **WSJF score** — `(business_value + time_criticality + risk_reduction) /
   job_size`, each 1–10, relative across this project's epics (not an
   absolute scale). Derive a first pass from each epic's rolled-up
   FEATURES.md complexity/risk/priority fields, then confirm with the user
   rather than asserting it silently — WSJF ordering is what the delivery
   plan (step 6) sequences by. Mark the single highest-value epic ★ (the
   "wedge" — the reason the product exists; everything else is scaffolding
   around it).

5. **Tasks — decompose the epic directly into implementation-sized tasks**
   (no story layer in between): each one small enough to land as one
   `/agentic-harness:architect` dispatch and one test-gate pass, sized
   XS/S/M/L (split anything that wants to be XL), scored M/S/C (MoSCoW),
   with `depends_on` (other task ids, cross-epic is fine) and a trace tag
   (`F-##`/`FR-<MODULE>-##`). A task that can't trace this way gets fixed
   (find the real trace) or dropped, not queued anyway. If a task turns out
   to need its own acceptance line beyond the epic's EARS criteria (a
   sharp edge case specific to that task), add one `EARS-<AREA>-#` line
   inline under it — most tasks won't need this; the epic's criteria
   already cover them.

6. **Delivery mode** — ask the user once, via `AskUserQuestion`:
   **whole project at once**, or **weekly sprints** (Monday–Sunday, one
   review point per week; no other cadence — daily/biweekly/monthly aren't
   offered). Record `planning.delivery_mode` (`single_batch` |
   `sprint_weekly`) in `planning/project.config.yaml`. This is a
   per-project choice, not a per-epic one; re-decide it explicitly
   (`/agentic-harness:epics` again) rather than mixing modes mid-backlog.

   - **`single_batch`**: sequence epics by WSJF (highest first), respecting
     `depends_on` — write this order into the Delivery plan section, no
     sprint table.
   - **`sprint_weekly`**: assign epics/tasks into weekly sprints in WSJF +
     dependency order. Make **Sprint 1's** scope concrete (confirm with the
     user); later sprints are a provisional forward plan only — re-confirm
     each sprint's actual scope at its boundary (re-run
     `/agentic-harness:epics` to resequence, or let `/agentic-harness:manager`
     /`/agentic-harness:architect` carry forward anything unfinished into
     the next sprint rather than silently dropping it). Record each sprint
     in `planning.sprints` (`number`, `week_of`, `epics`, `status:
     planned|active|done`).

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

## Delivery plan

**Mode:** Whole project at once | Weekly sprints

<if "Whole project at once">
Epics run in this order (WSJF, respecting dependencies):
1. E-01, E-03, E-02, ...

<if "Weekly sprints">
Sprints run Monday–Sunday. Sprint 1's scope is committed; later sprints are
provisional and get re-confirmed at their boundary.

| Sprint | Week of | Epics in scope | Status |
|---|---|---|---|
| Sprint 1 | <YYYY-MM-DD> | E-01 | planned |
| Sprint 2 | <YYYY-MM-DD> | E-02, E-03 | planned (provisional) |

## E-01 — <plain-language title> ★ (mark only the wedge epic)

**When this ships:** <role> can <do X>, which they couldn't before.
**Goal:** → `<business goal from BUSINESS_GOALS.md>`
**WSJF:** (value <n> + time-criticality <n> + risk-reduction <n>) / size <n> = **<score>**

### Scope
**In scope**
- ...

**Out of scope**
- ... (deferred to E-##)

### Acceptance criteria (EARS)
- **EARS-<AREA>-1**: WHEN `<trigger>`, the system SHALL `<behavior>` (FR-...)
- **EARS-<AREA>-2**: IF `<error>`, THEN the system SHALL `<response>` (FR-...)

### Tasks
| Task | Description | Size | MoSCoW | Depends on | Traces to |
|---|---|---|---|---|---|
| T-01 | <implementation-sized task> | S | Must | — | F-##/FR-<MODULE>-## |

### Risks
| Risk | Mitigation |
|---|---|
| <risk> | <mitigation> |

(repeat per epic)

## Coverage check
Every Feature/FR lands in exactly one epic; orphans and duplicates are both
breakdown failures — list any, or "none".

## Open Items (TBD)
1. <unresolved item> (§<epic it affects>)

---
*End of document — Draft v0.1. Open items: <1-line summary, or "none">.*
```

## Seed `TASKS.md`

For every Task row above, append a row using the existing schema:

```
| N | <task, specifics from EPICS.md> [E-##/F-##] | <goal> | Planning YYYY-MM-DD | 🔴/🟡/🟢 | <Sprint N or —> | ⏳ TODO | — | — | — |
```

- Priority derived from the epic's WSJF rank and MoSCoW (Must → 🔴/🟡,
  Should/Could → 🟡/🟢) — use judgment, don't mechanically force every row
  to the same priority.
- **Sprint column**: the sprint number the task's epic is assigned to if
  `planning.delivery_mode == sprint_weekly`; `—` if `single_batch`.
  `/agentic-harness:architect` reads this to default its execution scope to
  the active sprint (see `architect.md` Phase 1) rather than the whole
  backlog.
- Agent column stays `—`; `/agentic-harness:architect` resolves segment
  assignment at Phase 1.5/dispatch time using each feature's proposed
  owning area, not this command.
- For each tool under `config.tools` with `enabled: true`, invoke its
  mirror skill (e.g. `/agentic-harness:clickup-log`) to sync each new row,
  respecting `mirror_from_date`.
- Update `TASKS.md`'s `_Last updated:` line.

## Approval gate

Per `planning-protocol.md` (Versioning applies — archive before rewrite),
applied to the epic/task backlog and delivery plan as a whole. On
approval, set `planning.stages.epics.status: approved` + `approved_on`.

## Report (≤9 lines)

```
planning/EPICS.md: v<version> — N epics (wedge: E-##), N tasks
Delivery mode: whole project at once | weekly sprints (Sprint 1: <epics>)
TASKS.md: N rows seeded
Traceability: all tasks trace to a goal (or list exceptions)
Status: draft/approved
Planning phase complete: <yes/no — list any stage still pending/draft>
Next: run /agentic-harness:architect to begin implementation
```
