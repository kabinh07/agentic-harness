---
name: init
description: Scaffold this project with the agentic-harness template files (CLAUDE.md, TASKS.md, planning/ with project.config.yaml, BUSINESS_GOALS.md, ENGINEERING_STANDARDS.md, README.md, input/) if they don't already exist. Run once right after installing the plugin, before /agentic-harness:plan.
---

# /agentic-harness:init

Copies the harness skeleton bundled with this plugin into the current
project. Commands like `/agentic-harness:plan`, `/agentic-harness:configure`,
and `/agentic-harness:architect` expect `CLAUDE.md`, `TASKS.md`, and a fully
populated `planning/` directory to already exist in the project — this
command creates them.

## Steps

1. For each pair below, copy the template to the destination **only if the
   destination doesn't already exist** — never overwrite a file the project
   already has:
   - `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.md` → `CLAUDE.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/TASKS.md` → `TASKS.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/project.config.yaml` → `planning/project.config.yaml`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/BUSINESS_GOALS.md` → `planning/BUSINESS_GOALS.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/ENGINEERING_STANDARDS.md` → `planning/ENGINEERING_STANDARDS.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/README.md` → `planning/README.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/input/.gitkeep` → `planning/input/.gitkeep`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/adr/.gitkeep` → `planning/adr/.gitkeep`
   - `${CLAUDE_PLUGIN_ROOT}/templates/planning/design-assets/.gitkeep` → `planning/design-assets/.gitkeep`
2. Create missing parent directories (`planning/`, `planning/input/`,
   `planning/adr/`, `planning/design-assets/`) as needed. Do not create
   `planning/BRD.md`, `planning/SRS.md`, etc. — those are written by the
   planning-phase stage commands, not scaffolded empty.
3. If `CLAUDE.md` already existed in the project and doesn't reference this
   harness's load order (`planning/project.config.yaml` →
   `planning/BUSINESS_GOALS.md` → `planning/ENGINEERING_STANDARDS.md` →
   `TASKS.md`), don't touch it — tell the user they'll need to merge the
   harness's load-order section in by hand instead of silently appending to
   instructions they already wrote.
4. Report (≤6 lines): files created, files skipped (already existed), and
   the next step — run `/agentic-harness:plan` (it will ask whether you're
   starting from an idea, an existing codebase, an existing BRD, or an
   existing SRS, and drive the rest of the planning phase from there).
