# CLAUDE.md (this repo)

This repo *is* the `agentic-harness` Claude Code plugin — not an instance of
the harness. Don't confuse the two:

- `commands/`, `agents/`, `.claude-plugin/` — the plugin payload. These ship
  to whoever installs the plugin.
- `templates/` — the skeleton (`CLAUDE.md`, `TASKS.md`, `planning/` with
  `project.config.yaml`, `BUSINESS_GOALS.md`, `ENGINEERING_STANDARDS.md`,
  `README.md`, `input/`, `adr/`, `design-assets/`) that `/agentic-harness:init`
  copies into a *consuming* project. Never fill these in with real project
  data here — they must stay generic placeholders, since every install
  starts from a copy of them.

If you're working on the harness itself (adding a command, fixing a phase in
`architect.md`, changing the config schema), edit `commands/`, `agents/`, or
`templates/` directly and keep them consistent with each other — e.g. a new
config key in `templates/planning/project.config.yaml` needs the commands that
read it updated too.

If you're testing the harness end-to-end, do that in a *separate* scratch
project with the plugin installed (`claude plugin marketplace add <this
repo path>` then `claude plugin install agentic-harness`) — not by filling
in `templates/` in place.

## Layout

```
.claude-plugin/
  plugin.json        # plugin manifest
  marketplace.json    # single-plugin local marketplace (source: "./")
commands/              # -> /agentic-harness:<name> slash commands
  plan.md               # planning-phase driver/wizard: idea/codebase/BRD/SRS -> epics
  brd.md                 # -> planning/BRD.md
  srs.md                  # -> planning/SRS.md
  design.md                 # -> planning/DESIGN_BRIEF.md, then planning/DESIGN.md (skippable)
  features.md                # -> planning/FEATURES.md
  adr.md                       # -> planning/adr/ADR-NNNN-*.md
  epics.md                      # -> planning/EPICS.md, seeds TASKS.md
  init.md               # scaffold templates/ into a new project
  configure.md           # BRD -> config + BUSINESS_GOALS.md + TASKS.md
  architect.md             # orchestrator: segment/swarm lifecycle, delegate, test, gate, review
  manager.md                # goal-owning, periodic health-check / task-queuing
  test.md                    # thin command wrapper around agentic-harness:test-writer
  clickup-log.md              # example tool-mirror skill (ClickUp)
agents/
  test-writer.md      # -> agentic-harness:test-writer, standing test author
  codebase-analyst.md  # -> agentic-harness:codebase-analyst, read-only existing-code surveyor
docs/
  planning-protocol.md # shared questioning/gating/bookkeeping rules for the six planning commands
templates/             # copied into a project by /agentic-harness:init
  CLAUDE.md
  TASKS.md
  planning/
    project.config.yaml
    BUSINESS_GOALS.md
    ENGINEERING_STANDARDS.md
    README.md
    input/.gitkeep
    adr/.gitkeep
    design-assets/.gitkeep
```

See `README.md` for install instructions and `templates/CLAUDE.md` for what
a consuming project's own `CLAUDE.md` looks like.
