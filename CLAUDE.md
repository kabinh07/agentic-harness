# CLAUDE.md (this repo)

This repo *is* the `agentic-harness` Claude Code plugin — not an instance of
the harness. Don't confuse the two:

- `commands/`, `agents/`, `.claude-plugin/` — the plugin payload. These ship
  to whoever installs the plugin.
- `templates/` — the skeleton (`CLAUDE.md`, `config/project.config.yaml`,
  `docs/`, `TASK_LOG.md`, `BRD/`) that `/agentic-harness:init` copies into a
  *consuming* project. Never fill these in with real project data here —
  they must stay generic placeholders, since every install starts from a
  copy of them.

If you're working on the harness itself (adding a command, fixing a phase in
`architect.md`, changing the config schema), edit `commands/`, `agents/`, or
`templates/` directly and keep them consistent with each other — e.g. a new
config key in `templates/config/project.config.yaml` needs the commands that
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
  init.md               # scaffold templates/ into a new project
  configure.md           # BRD -> config + business-logic.md + TASK_LOG.md
  architect.md             # orchestrator: segment, delegate, test, gate, review
  run-manager.md             # goal-owning, periodic health-check / task-queuing
  clickup-log.md              # example tool-mirror skill (ClickUp)
agents/
  test-writer.md      # -> agentic-harness:test-writer, standing test author
templates/             # copied into a project by /agentic-harness:init
  CLAUDE.md
  config/project.config.yaml
  docs/business-logic.md
  docs/engineering-standards.md
  TASK_LOG.md
  BRD/.gitkeep
```

See `README.md` for install instructions and `templates/CLAUDE.md` for what
a consuming project's own `CLAUDE.md` looks like.
