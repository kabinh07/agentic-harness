---
name: srs
description: Build planning/SRS.md from the approved BRD, elaborating the same FR-<MODULE>-## IDs (never renumbering) into detailed system features, data requirements, interfaces, and NFRs, with an Appendix A traceability matrix and Appendix B open items. Versioned, calibrated questioning per planning-protocol.md.
---

# /agentic-harness:srs

Second planning stage. Turns `planning/BRD.md`'s module-grouped functional
requirements into detailed, testable system features — same IDs, more
detail, never a fresh numbering scheme. Follows
`${CLAUDE_PLUGIN_ROOT}/docs/planning-protocol.md`.

This is the first document in the planning phase allowed to go
technical — per planning-protocol.md's **BRD vs SRS altitude** section,
the BRD states *what* the business needs; every SRS requirement below
elaborates the same ID with *how*: data model, specific mechanisms,
technology choices, exact thresholds. Naming a table/column, a
library/protocol, an algorithm, or a status code is expected here, not a
violation — that's the whole point of this stage existing separately
from the BRD.

## Gate

`planning.stages.brd.status` must be `approved`. If not, stop and tell the
user to finish `/agentic-harness:brd` first.

While reading the BRD (Inputs below), if its own requirement rows are
already written at SRS altitude (names tables/columns, specific
libraries, algorithms — see planning-protocol.md's BRD vs SRS altitude
section) rather than business capability/outcome language, stop and flag
it to the user before elaborating further — elaborating an already
-technical BRD just compounds the altitude problem instead of adding a
real second layer of detail. Offer to send it back through
`/agentic-harness:brd` for a rewrite at the correct altitude.

## Inputs

Read `planning/BRD.md` in full — its §5 Stakeholders & User Roles, §6
Functional Requirements (modules and IDs to inherit), §7 NFRs, §8
Assumptions & Constraints. If `planning.entry_point == existing-project`,
also dispatch `agentic-harness:codebase-analyst` for Data model and
External integrations facts — existing systems already have real
interfaces and constraints an idea-stage SRS shouldn't invent.

## Question ladder

Continue `planning-protocol.md`'s ladder from where the BRD left off — go
deeper per module, don't restart at goal fit:

- **Constraint reality / execution** — per BRD requirement, what does
  "done" concretely mean; latency/throughput budgets; auth model; data
  volume/growth; integration contract shape (sync/async, failure modes,
  rate limits).
- **Failure modes** — what happens when each critical path fails: retry,
  fallback, user-visible error, silent degrade (should never be the
  answer without the user explicitly choosing it).
- **Validation** — acceptance criteria per requirement (Given/When/Then),
  observability needs (what must be logged/monitored/alerted).
- **Reversibility** — deployment target and any migration/rollback
  implication if this changes underlying data or infra.

## Question ladder is per-module, not per-question

Work through one BRD module at a time (matches how the output is
structured) rather than jumping across modules — easier for the user to
stay in context, and keeps each `AskUserQuestion` batch coherent.

## Output — `planning/SRS.md`

Follow `planning-protocol.md`'s Document format:

```markdown
# Software Requirements Specification

## <Project Name>

| | |
|---|---|
| **Document title** | <Project Name> — Software Requirements Specification |
| **Version** | 0.1 (Draft) |
| **Date** | <today> |
| **Author** | <user/org> |
| **Based on** | <Project Name> BRD v<BRD's current version> |
| **Status** | For review |

## 1. Introduction
### 1.1 Purpose
### 1.2 Document Conventions
(state: requirement IDs carry a module prefix, e.g. FR-AUTH-01; IDs
inherited from the BRD are preserved, IDs added here continue each
module's sequence; priority M/S/C; "the system shall..." phrasing)
### 1.3 Intended Audience
### 1.4 Product Scope
### 1.5 Definitions, Acronyms, Abbreviations
(table)
### 1.6 References
(the BRD, by version)

## 2. Overall Description
### 2.1 Product Perspective
### 2.2 Product Functions (summary)
### 2.3 User Classes and Characteristics
(table — elaborates BRD §5)
### 2.4 Operating Environment
### 2.5 Design and Implementation Constraints
### 2.6 Assumptions and Dependencies

## 3. System Features (Functional Requirements)

*Each feature lists description & priority, actors, preconditions, a
primary flow (where it aids clarity), the detailed functional
requirements, and business rules / error handling.*

### 3.1 <Module Name>
Priority: M/S/C · Actors: ... · Precondition: ...

**Primary flow** (only where it aids clarity):
1. ...

| ID | Requirement | Priority |
|---|---|---|
| FR-<MOD>-01 | The system shall ... | M |

(repeat per module inherited from BRD §6 — elaborate existing IDs with
"the system shall" phrasing + acceptance detail; append new IDs
continuing that module's sequence for anything the BRD didn't cover;
flag any wholly new module the SRS uncovers that the BRD didn't
anticipate — that's a signal the BRD may need a follow-up amendment, not
just a note here)

*Business rules / errors: ...*

## 4. Data Requirements
### 4.1 Core Entities
(table: Entity | Purpose)
### 4.2 Key Attributes — <primary entity/entities>
### 4.3 Entity Relationships (summary)
### 4.4 <State machine, if the domain has one> (table: From | To | Trigger/actor | Guard/rule)

## 5. External Interface Requirements
### 5.1 User Interfaces
### 5.2 Software Interfaces (API — functional)
(table: Operation | Purpose | Auth)
### 5.3 Communication Interfaces
### 5.4 Hardware Interfaces

## 6. Non-Functional Requirements
(table: ID | Category | Requirement — ID = NFR-<CATEGORY>-##, categories:
performance/security/reliability/scalability/observability/
maintainability/compliance/accessibility/usability/auditability/data)

## 7. Other Requirements
(retention, localization, compliance — bulleted; flag "(open item)" inline
for anything not yet decided, and mirror it into Appendix B)

## Appendix A — Traceability Matrix (BRD → SRS)
| BRD requirement | Covered by SRS |
|---|---|
| FR-AUTH-01..05 | §3.1 (FR-AUTH-01..05, elaborated + FR-AUTH-06/07) |

## Appendix B — Open Items (TBD)

1. <unresolved item> (§<section>)

---
*End of document — Draft v0.1. Open items: <1-line summary, or "none">.*
```

## Orphan check

Every BRD requirement ID should appear in Appendix A, covered by at least
one SRS elaboration. A BRD requirement with nothing tracing to it is
reported to the user, not silently left — either the SRS is missing
elaboration or the requirement needs revisiting.

## Approval gate

Per `planning-protocol.md` (Versioning applies — archive before rewrite).
On approval, set `planning.stages.srs.status: approved` + `approved_on`.

## Report (≤6 lines)

```
planning/SRS.md: v<version> (<written/updated>, based on BRD v<x>)
Modules elaborated: N, Requirements: N (M/S/C: n/n/n), NFRs: N
Traceability: N/N BRD requirements covered (or list gaps)
Open items: N
Status: draft/approved
Next: /agentic-harness:design (if has_ui) or /agentic-harness:features
```
