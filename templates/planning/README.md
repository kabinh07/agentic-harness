# Planning

Everything in this folder is one shareable bundle — drop the whole
`planning/` directory in front of a teammate and they can see exactly where
the project stands, what's decided, what's still open, and what was assumed
without confirmation. Maintained by `/agentic-harness:plan` and the stage
commands it drives (`:brd`, `:srs`, `:design`, `:features`, `:adr`,
`:epics`); status here always matches `project.config.yaml`'s
`planning.stages`.

_Last updated: —_

## Stage status

| Stage | Artifact | Status | Approved |
|---|---|---|---|
| BRD | [BRD.md](BRD.md) | pending | — |
| SRS | [SRS.md](SRS.md) | pending | — |
| Design | [DESIGN.md](DESIGN.md) | pending | — |
| Features | [FEATURES.md](FEATURES.md) | pending | — |
| ADRs | [adr/](adr/README.md) | pending | — |
| Epics | [EPICS.md](EPICS.md) | pending | — |

Status values: `pending` (not started) · `draft` (written, not yet
reviewed) · `in-review` · `approved` · `skipped` (e.g. Design, for a
project with no UI).

## Open questions

_(none yet — populated as stage commands surface things they can't answer
without you)_

## Assumptions (unvalidated)

_(none yet — anything a stage command couldn't get an answer for lands
here instead of being silently guessed; resolve and move to the relevant
artifact when confirmed)_
