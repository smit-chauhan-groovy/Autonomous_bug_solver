# Host Quickstart

Use this file when loading the prompt pack into another repository with a coding agent.

## Universal startup prompt

Copy this into the host tool after the prompt pack has been added to the target repository:

```text
Read AGENT_PROTOCOL.md first, then read agents/ORCHESTRATOR.md.
Treat this repository's prompt pack as the workflow contract.
Classify the current environment into FULL_AUTO, CONSTRAINED, DRY_RUN, or MANUAL_HANDOFF.
If the workspace contains multiple services or nested repositories, resolve workspace_root, service_root, and git_root automatically before acting.
Follow capability-based fallbacks instead of assuming every tool is available.
When ready, start the pipeline.
```

## Strict autonomous startup prompt

Use this if you want the strongest non-interactive behavior the host will permit:

```text
Read AGENT_PROTOCOL.md first, then read agents/ORCHESTRATOR.md.
Treat worker-agent handoffs as internal only.
Do not ask whether to continue between Fetch, Fix, Test, or PR phases.
If the workspace is a monorepo or microservice layout, automatically discover the correct service root and git root before running fixes, tests, or PR actions.
Continue automatically until DONE, a non-retryable blocker, or a host-enforced capability denial.
If the host blocks an action, treat it as a capability limit and return the best fallback artifacts instead of asking extra confirmation questions.
Start the pipeline now.
```

## Universal bug-fix prompt

```text
Use the prompt pack in AGENT_PROTOCOL.md and agents/ORCHESTRATOR.md.
Run the autonomous bug-fix workflow for this repository.
If any capability is blocked, return structured partial results with exact next actions.
Start now.
```

## Dry-run prompt

```text
Read AGENT_PROTOCOL.md and agents/ORCHESTRATOR.md.
Run the workflow in DRY_RUN mode.
Do not mutate files, branches, or pull requests.
Return the bug analysis, fix plan, test plan, proposed diff, and PR artifacts.
```

## Manual-handoff prompt

```text
Read AGENT_PROTOCOL.md and agents/ORCHESTRATOR.md.
Run the workflow in MANUAL_HANDOFF mode.
Do not rely on unavailable tools.
Generate exact patches, commands, commit message, and PR body for an operator to apply manually.
```

## Best practices for host tools

- Load `AGENT_PROTOCOL.md` before any agent file.
- Use `agents/ORCHESTRATOR.md` as the only entrypoint.
- Preserve JSON output shapes defined by the protocol.
- If a host asks for approval before commands or writes, complete as much as possible and fall back cleanly.
- Do not skip error reporting just because a tool was unavailable.

## Expected operator flow

1. Place this prompt pack in the target repository.
2. Ensure `.env` exists if Linear integration is needed.
3. Start the host with the universal startup prompt.
4. Let the host run the orchestrator.
5. Review final artifacts if the host ended in `partial` or `MANUAL_HANDOFF` mode.