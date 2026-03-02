---
description: Coding agent principles and guidelines for this repository
alwaysApply: true
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Terminology

**Coding agent** — the AI assistant (Claude Code, Cursor, etc.) reading and acting on these instructions.

**Target agent** — an AI agent being designed and built as part of this codebase. The [Building Target Agents](#building-target-agents) section instructs the coding agent on how to approach that work — it is not a description of the coding agent's own architecture.

---

## Code Organization

- Keep related code close together — colocate files that change together
- Prefer flat structures over deep nesting until complexity demands otherwise
- Group by feature or domain, not by technical layer
- When adding new code, look for existing patterns first and follow them

## Component Independence

Major subsystems (database, AI/ML integrations, UI, external APIs, etc.) should be treated as independent, swappable components.

- **Define interface layers between components** — each component exposes a contract (abstract class, protocol, interface, or typed function signatures) that other components depend on, never the concrete implementation
- **Keep provider-specific logic contained** — all details about a vendor, library, or service stay inside that component; nothing leaks into shared code
- **Depend on abstractions, not implementations** — when one component calls another, it references the interface, not the specific provider
- **Configuration drives which implementation is used** — swapping a provider should be a config or wiring change, not a code change across multiple files

When building a new feature that touches a distinct subsystem: define its interface first, build the implementation behind it, wire it up through a single entry point (factory, config, or dependency injection).

## Simplicity and Readability

- Write code that's obvious over code that's clever
- Favor explicit over implicit — make dependencies and data flow visible
- Keep functions focused on a single responsibility
- Avoid premature abstraction — wait until you see repetition before extracting

## Naming

- Names should describe what something is or does, not how it's implemented
- Be consistent with existing naming conventions in the codebase
- Longer, descriptive names over short, ambiguous ones
- Avoid abbreviations unless universally understood in the domain

## Documentation

- Document the "why" not the "what" — code shows what it does, comments explain intent
- Keep documentation close to the code it describes
- Prefer self-documenting structure over extensive inline comments

## When Adding Features

- Understand the existing approach before introducing new patterns
- Ask if a simpler solution exists before building something complex
- Consider how the change affects the rest of the codebase

## When Refactoring

- Make the smallest change that achieves the goal
- Preserve existing behavior unless explicitly changing it
- Separate refactoring commits from feature commits when practical

---

## Building Target Agents

A target agent is a stateful control system that uses a probabilistic planner to decide actions in a dynamic environment under token and latency constraints.

When the coding agent is asked to help design or build a target agent, all specification and implementation decisions must be centered around the six components below. **Do not write any target agent code until each component is specified.**

### The Six Components

1. **Model** — The inference endpoint and its configuration. Defines which model is called, with what parameters, under what constraints (max tokens, temperature, timeout). Responsible for the probabilistic planning step.

2. **Tool Registry** — The complete set of actions the target agent can take. Each tool has a schema (name, description, input/output types) and an implementation. The registry is the agent's action space; its design determines what the target agent can and cannot do.

3. **Control Loop** — The execution logic that drives the target agent forward. Decides when to call the model, when to dispatch a tool, when to stop, and when to hand off. This is where token budgets, iteration limits, and termination conditions live.

4. **State Store** — The working memory of a single run. Tracks what has happened within an execution: messages, tool call history, intermediate results. Scoped to a session or task; does not persist by default.

5. **Memory System** — Longer-lived context that persists across runs. May include conversation history, user preferences, retrieved documents, or episodic summaries. Distinct from state: state is ephemeral, memory is durable.

6. **Logging Layer** — Observability for every action the target agent takes. Captures model inputs/outputs, tool calls and their results, state transitions, latency, and token usage. Required for debugging non-deterministic behavior.

### Directory Structure

Each target agent lives in its own siloed directory. Target agents must not share implementation files — shared utilities belong in a separate `lib/` or `shared/` directory that no single agent owns.

Required layout:

```
agents/
  <agent-name>/
    README.md            # What this target agent does and how to run it
    model.md             # Model selection, parameters, provider interface
    tool-registry.md     # All tools: schemas, descriptions, dispatch logic
    control-loop.md      # Execution flow, termination conditions, token budget
    state.md             # State shape, lifetime, access patterns
    memory.md            # Memory strategy, persistence mechanism, retrieval
    logging.md           # What is logged, format, destination
    config.py            # All parameters and all prompts (see below)
    src/                 # Implementation files
```

All six component docs must exist before implementation begins. They are the spec — implementation follows the doc, not the other way around.

### config.py

`config.py` is the single source of truth for every configurable value and every prompt string in the target agent. Nothing in `src/` should define a prompt or a tuneable parameter inline.

**Parameters** — any value that may need tuning: model name, temperature, max tokens, timeouts, retry counts, batch sizes, thresholds. Define them as named constants at the top of `config.py`.

**Prompts** — every string that shapes model behavior: system prompt, tool descriptions, user-facing message templates, few-shot examples, chain-of-thought scaffolds. All of these live in `config.py`. This makes the target agent's behavior fully auditable from one file and eliminates hunting through `src/` to find or change what the model sees.

### Pre-Build Specification

Before writing any implementation code, the coding agent must work through these questions with the user for each component:

**Model**
- Which model and provider?
- What are the token and latency budgets per call?
- How is the system prompt structured?
- Is the model interface abstracted so the provider can be swapped?

**Tool Registry**
- What is the complete tool list for the first version?
- What does each tool's input and output schema look like?
- How are tools registered and dispatched — static list or dynamic registry?
- Are there tools that must never be called in sequence or in certain states?

**Control Loop**
- What is the termination condition — model signal, max iterations, or both?
- Where is the token budget tracked and enforced?
- What happens on a tool error — retry, skip, or abort?
- Is this a single-pass agent or does it run in a background process?

**State Store**
- What is the shape of state at each step?
- Is state in-memory only, or must it survive restarts?
- Who reads and writes state — only the loop, or also tools?

**Memory System**
- Does this target agent need memory across runs, or is ephemeral state sufficient?
- If yes: what is stored, when, and how is it retrieved?
- What is the retrieval strategy — recency, relevance, or both?
- How does retrieved memory get injected into the model context?

**Logging Layer**
- What events are logged — model calls, tool calls, state changes, errors?
- What is the log format and destination (stdout, file, external sink)?
- How will you debug a run that produced a wrong or unexpected result?

### Component Independence

Each of the six components should be independently replaceable. The model provider, tool implementations, and memory backend are the most likely to change — build interfaces for them from the start. The control loop depends on those interfaces, not the concrete implementations behind them.

### Documentation Requirement

Component docs are not optional. They record the decisions made, not just the final structure. If a component's design changes during implementation, update its doc before moving on.

---

## Repository Hygiene

Every project should be clone-and-run ready:

- **Always gitignore:** secrets and env files (`.env`, `.env.local`), build artifacts (`dist/`, `build/`), installed dependencies (`node_modules/`, `venv/`), caches, and OS artifacts (`.DS_Store`, `Thumbs.db`)
- **Always include:** dependency manifests (`package.json`, `requirements.txt`, etc.), lock files for reproducible installs, and `.env.example` showing required variables without real values
- If you add a feature that generates files, add corresponding gitignore entries
