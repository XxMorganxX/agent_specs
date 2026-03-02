# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a standalone **agent-spec** repository — a source of truth for cross-project coding principles. It contains no application code, dependencies, or build system.

## Structure

```
templates/
  CLAUDE.md          ← single canonical source for all design principles (edit here)
principles/
  agent-setup.md     ← workspace meta-behavior; not part of the template
.cursor/rules/
  principles.mdc     → ../../templates/CLAUDE.md   (symlink)
  agent-setup.mdc    → ../../principles/agent-setup.md   (symlink)
```

## Working in This Repo

- **All design principle edits go to `templates/CLAUDE.md`** — that is the single source of truth. The `.cursor/rules/principles.mdc` symlink reflects changes automatically.
- `principles/agent-setup.md` is workspace-specific behavior (how the coding agent should act in this multi-root setup); it is not copied to new projects.
- When starting a new project, copy `templates/CLAUDE.md` into its root.
- Do not modify these files unless explicitly asked.
- This repo has its own independent git history and can be pushed/pulled separately to sync preferences across machines.
