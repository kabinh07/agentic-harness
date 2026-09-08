# Planning

Everything in this folder is one shareable bundle — drop the whole
`planning/` directory in front of a teammate and they can see exactly where
the project stands, what's decided, what's still open, and what was
assumed once confirmed. Maintained by `/agentic-harness:plan` and the stage
commands it drives (`:brd`, `:srs`, `:design`, `:dfd`, `:erd`, `:features`,
`:adr`, `:epics`); status here always matches `project.config.yaml`'s
`planning.stages`. Prior versions of every artifact live in `versions/` —
nothing is ever silently overwritten (see `planning-protocol.md`'s
Versioning section).

_Last updated: —_

## Calibration

Knowledge level: — · Pressure level: — *(set once, on the first
`/agentic-harness:plan` or `/agentic-harness:brd` run — see
`planning-protocol.md`)*

## Delivery

Mode: — *(set once, at `/agentic-harness:epics` — whole project at once or
weekly sprints)*. Current sprint: — *(sprint_weekly only; see
`planning/project.config.yaml`'s `planning.sprints` and `EPICS.md`'s
Delivery plan section)*

## Stage status

| Stage | Artifact | Version | Status | Approved |
|---|---|---|---|---|
| BRD | [BRD.md](BRD.md) | — | pending | — |
| SRS | [SRS.md](SRS.md) | — | pending | — |
| Design | [DESIGN.md](DESIGN.md) | — | pending | — |
| DFD | [DFD.md](DFD.md) | — | pending | — |
| ERD | [ERD.md](ERD.md) | — | pending | — |
| Features | [FEATURES.md](FEATURES.md) | — | pending | — |
| ADRs | [adr/](adr/README.md) | — | pending | — |
| Epics | [EPICS.md](EPICS.md) | — | pending | — |

Status values: `pending` (not started) · `draft` (written, not yet
reviewed) · `in-review` · `approved` · `skipped` (e.g. Design, for a
project with no UI; or ERD, for a project with no structured data model).

## Open Items (TBD)

_(none yet — populated as stage commands surface genuinely unresolved
decisions; each artifact keeps its own numbered Open Items section, this
is the cross-artifact rollup)_

## Assumptions & Constraints

_(none yet — confirmed background conditions land here once a stage
command gets a real answer, distinct from Open Items above which are still
unresolved)_
