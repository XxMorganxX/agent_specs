---
description: Explains the agent-spec workspace structure and how it provides cross-project guidance
alwaysApply: true
---

# Agent Spec Workspace

This workspace uses a multi-root layout where `agent-spec/` exists as an independent repository alongside the main project. Both folders are open in the same workspace, allowing rules from each to load into agent context automatically.

## How This Works

- `agent-spec/` contains rules and templates that define how the coding agent should approach tasks
- `templates/CLAUDE.md` is the single canonical source for all design principles — edit it there, nowhere else
- `.cursor/rules/` contains two symlinks: one to this file, one to `templates/CLAUDE.md`
- These rules apply across any project that shares a workspace with this folder
- The main project has its own git history; agent-spec has its own — they're fully independent
- Copy `templates/CLAUDE.md` into any new project so the principles travel with it

## Your Behavior

When working in this workspace:

1. **Focus edits on the main project** — agent-spec is configuration, not the codebase you're building
2. **Consult the rules here** for guidance on design, documentation, and coding principles
3. **Do not modify agent-spec** unless explicitly asked to update these rules
4. **Treat these rules as preferences** — they describe how I want you to think about code, not rigid constraints

## Updating Agent Spec

If asked to update rules or add new guidance, edit `agent-spec/templates/CLAUDE.md` — that is the single canonical file. The `.cursor/rules/principles.mdc` symlink will automatically reflect any changes. This repo can be pushed/pulled independently to keep preferences synced across machines and projects.
