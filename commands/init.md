---
name: init
description: Scaffold this project with the agentic-harness template files (CLAUDE.md, config/project.config.yaml, docs/, TASK_LOG.md, BRD/) if they don't already exist. Run once right after installing the plugin, before /agentic-harness:configure.
---

# /agentic-harness:init

Copies the harness skeleton bundled with this plugin into the current
project. Commands like `/agentic-harness:configure` and
`/agentic-harness:architect` expect `config/project.config.yaml`,
`docs/business-logic.md`, `docs/engineering-standards.md`, `TASK_LOG.md`,
and `BRD/` to already exist in the project — this command creates them.

## Steps

1. For each pair below, copy the template to the destination **only if the
   destination doesn't already exist** — never overwrite a file the project
   already has:
   - `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md` → `CLAUDE.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/config/project.config.yaml` → `config/project.config.yaml`
   - `${CLAUDE_PLUGIN_ROOT}/templates/docs/business-logic.md` → `docs/business-logic.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/docs/engineering-standards.md` → `docs/engineering-standards.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/TASK_LOG.md` → `TASK_LOG.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/BRD/.gitkeep` → `BRD/.gitkeep`
2. Create missing parent directories (`config/`, `docs/`, `BRD/`) as needed.
3. If `CLAUDE.md` already existed in the project and doesn't reference this
   harness's load order (`config/project.config.yaml` → `docs/business-logic.md`
   → `docs/engineering-standards.md` → `TASK_LOG.md`), don't touch it —
   tell the user they'll need to merge the harness's load-order section in
   by hand instead of silently appending to instructions they already wrote.
4. Report (≤6 lines): files created, files skipped (already existed), and
   the next step — drop a BRD into `BRD/`, then run `/agentic-harness:configure`.
