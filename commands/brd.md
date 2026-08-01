---
name: brd
description: Build planning/BRD.md by calibrated questioning (grill-me style) from an idea, existing codebase, or raw material in planning/input/ — never fabricates business logic. Module-prefixed requirements (FR-<MODULE>-##), versioned, formatted as a standalone consulting-grade document.
---

# /agentic-harness:brd

First planning stage. Produces `planning/BRD.md`: the business requirements
document every later stage traces back to, by ID. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md` for document format,
versioning, calibration, questioning, the approval gate, and bookkeeping.

## Step 0 — Calibration (if not already set)

If `planning.knowledge_level`/`planning.pressure_level` are unset (this
command invoked directly, not via `/agentic-harness:plan`), run the
Calibration step from `planning-protocol.md` now, once, before anything
else. Record both fields; every later stage this session reuses them.

## Sources, in priority order

1. **Command args** — if the user gave a description directly
   (`/agentic-harness:brd <idea>`), start from that.
2. **`planning/input/`** — any file that isn't `.gitkeep`. Read all of them;
   if multiple conflict, flag the conflict rather than silently picking the
   newest.
3. **Existing codebase** (`planning.entry_point == existing-project`) —
   dispatch `agentic-harness:codebase-analyst` and use its Purpose,
   Feature inventory, and Actors/roles sections as a first draft — present
   every inferred item back to the user for correction, never assert it.
   The analyst's report is necessarily technical (it read code); restate
   each item at BRD altitude (see Output section below) before it reaches
   the document — the technical detail isn't discarded, it seeds the SRS.
4. **Nothing** — start from a blank slate and ask.

Whatever the source, the BRD you produce is confirmed by the user, not
copied verbatim from a source document — a dropped-in doc or an analyst
report is raw material, not the artifact itself.

## Question ladder

Work `planning-protocol.md`'s ladder (goal fit → constraint reality →
option pressure → execution/scope → failure modes → validation →
reversibility), mapped to a BRD's content:

- **Goal fit** — problem/pain (what's broken or missing today, for whom),
  users & personas, business objectives, "what would make this not worth
  doing."
- **Constraint reality** — regulatory/budget/deadline/mandated-tech
  constraints, hard limits that can't move.
- **Option pressure** — where a real alternative exists (build vs. adopt,
  narrow vs. broad scope), surface it and ask which way, don't assume.
- **Execution/scope** — explicit in-scope and out-of-scope; identify
  natural **modules** here — these become the `FR-<MODULE>-##` prefixes
  both this document and the SRS use, so get them right before writing
  requirement rows. Derive modules from *this* project, not a template:
  for an AI/agentic system that's typically things like generation/meta
  -agent, model/inference config, runtime/orchestration, tool/connector
  integration, retrieval/knowledge, guardrails/approval, evaluation
  /observability, memory — not the conventional-app default of
  Authentication/User Management/Notifications (include those too, only
  if the system genuinely has them alongside its AI core). See
  `planning-protocol.md`'s Document format section for the full guidance.
- **Failure modes** — edge cases: each integration failing, scale ceiling,
  multi-tenancy, i18n, offline, data retention/privacy/deletion, "how does
  this fail in real use."
- **Validation** — success criteria, specific and measurable ("95%
  on-time," not "fast"); "what would you check to know this worked."
- **Reversibility** — what's hardest to undo once built; stakeholders who'd
  need to sign off on an irreversible choice.

Stop a rung once it's clear enough per the stopping conditions in
`planning-protocol.md` — don't force every question in every rung.

## Output — `planning/BRD.md`

Every requirement, objective, and scope line goes through
`planning-protocol.md`'s **BRD vs SRS altitude** check before it's
written — no database/schema names, library/protocol/vendor names,
algorithms, file paths, or status codes. If the natural answer to a
question came out technical (common when the source is
`agentic-harness:codebase-analyst` or an existing codebase), restate it
one altitude up before it goes in the table; the technical version isn't
lost, it's exactly what `/agentic-harness:srs` elaborates into next.

Follow `planning-protocol.md`'s Document format exactly:

```markdown
# Business Requirements Document (BRD)

## <Project Name>

| | |
|---|---|
| **Document title** | <Project Name> — Business Requirements |
| **Version** | 0.1 (Draft) |
| **Date** | <today> |
| **Author** | <user/org, ask if unclear> |
| **Status** | For review |

---

## 1. Purpose
## 2. Background
## 3. Business Objectives
(bulleted, priority-ordered where the user states or implies one)
## 4. Scope
### 4.1 In scope
### 4.2 Out of scope
## 5. Stakeholders & User Roles
(table: Role | Description)
## 6. Functional Requirements

Requirements are grouped by module. Each has a unique ID for traceability
(`FR-<MODULE>-##`). Priority uses **M** (Must-have), **S** (Should-have),
**C** (Could-have).

### 6.1 <Module Name>
*<one-line module description>*

| ID | Requirement | Priority |
|---|---|---|
| FR-<MOD>-01 | ... | M |

(repeat per module identified in the execution/scope rung)

## 7. Non-Functional Requirements
(table: ID | Requirement — flat is fine at BRD level; SRS categorizes further)
## 8. Assumptions & Constraints
(bulleted — confirmed background conditions, not uncertain ones)
## 9. Glossary
(table: Term | Meaning)

---

## Open Items (TBD)

1. <unresolved item> (§<section it affects>)

---
*End of document — Draft v0.1. Open items: <1-line summary, or "none">.*
```

## Approval gate

Per `planning-protocol.md` (includes Versioning — archive before any
rewrite). On approval: write the file (`Status: Approved`), set
`planning.stages.brd.status: approved` + `approved_on`, then tell the user
`/agentic-harness:configure` will run next (or run it yourself if invoked
via `/agentic-harness:plan`) to turn these requirements into
`planning/BUSINESS_GOALS.md` and `planning/project.config.yaml`.

## Report (≤6 lines)

```
planning/BRD.md: v<version> (<written/updated>)
Modules: N, Requirements: N (M/S/C: n/n/n)
Open items: N
Status: draft/approved
Next: /agentic-harness:configure, then /agentic-harness:srs
```
