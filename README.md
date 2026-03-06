# Autonomous Bug Solver

Portable multi-agent prompt pack for autonomous bug fixing inside another project's repository.

## What this repo is

This repository is **not** an executable application. It is a reusable set of agent specifications that a host coding agent can load inside a target project and use to:

1. fetch a bug from Linear,
2. analyze the codebase,
3. apply a fix,
4. write and run tests, and
5. prepare or open a pull request.

The `agents/` folder is the product. The quality goal is **portable, deterministic, tool-agnostic execution** across hosts such as Claude CLI, Antigravity, and similar repo-aware coding agents.

## Repository contents

- `agents/ORCHESTRATOR.md` — top-level workflow manager
- `agents/FETCH_PLAN_AGENT.md` — issue retrieval and plan generation
- `agents/FIXER_AGENT.md` — minimal bug fix application
- `agents/TESTING_AGENT.md` — test writing and execution
- `agents/PR_AGENT.md` — branch / commit / PR artifact generation
- `AGENT_PROTOCOL.md` — shared runtime contract for all agents
- `HOST_QUICKSTART.md` — copy-paste startup prompts for external coding agents
- `.env.example` — environment variables expected by Linear-aware flows

## Design principles

- **Portable** — avoid host-specific assumptions when possible
- **Capability-based** — use available tools; degrade gracefully when blocked
- **Deterministic** — use strict input/output contracts and stable status values
- **Safe** — never invent missing data; prefer partial completion over unsafe actions
- **Minimal** — fix only the reported bug and keep diffs small

## How to use in a target repository

1. Copy this prompt pack into the target repo, typically as `agents/` plus top-level docs.
2. Add a project-level `.env` using `.env.example` as reference if Linear integration is needed.
3. Optionally use `HOST_QUICKSTART.md` for copy-paste startup prompts.
4. In the host tool, instruct it to read `AGENT_PROTOCOL.md` first.
5. Then instruct it to use `agents/ORCHESTRATOR.md` as the entrypoint.
6. Start the workflow with a command such as `start` or an equivalent host-specific trigger.

## Required host capabilities

Best results come from hosts that support most of the following:

- reading files
- editing files
- running shell commands
- reading git state
- making network requests to Linear
- optionally creating branches / commits / pull requests

If a host does **not** support one or more capabilities, agents should follow the fallback rules in `AGENT_PROTOCOL.md` and return the best possible artifacts instead of failing silently.

## Supported runtime modes

- `FULL_AUTO` — read, edit, execute, network, and git/PR actions available
- `CONSTRAINED` — can complete most work but some actions are blocked
- `DRY_RUN` — analyze and generate artifacts without mutating repo state
- `MANUAL_HANDOFF` — produce exact commands, diffs, or PR text for a human/operator

## Success criteria for this prompt pack

This prompt pack is successful when a host agent can:

- follow a predictable sequence,
- emit structured outputs,
- preserve safety when tools are limited,
- avoid tool-specific breakage,
- and remain understandable to humans reviewing the run.

## Recommended target-repo additions

For best results in the target repository, include:

- `README.md`
- `CONTRIBUTING.md`
- project manifest (`package.json`, `pyproject.toml`, `go.mod`, etc.)
- existing test configuration
- CI configuration
- branch protection and review rules

## Important note

This prompt pack prefers autonomy, but host-tool policy always wins. If a host refuses a write, command, network request, push, or PR action, the agents must switch to structured fallback behavior instead of looping or asking repetitive permission questions.