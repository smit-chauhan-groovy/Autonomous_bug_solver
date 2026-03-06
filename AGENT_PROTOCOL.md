# Agent Protocol

This document defines the **shared runtime contract** for every agent in this repository.

## 1. Execution model

- `ORCHESTRATOR` is the entrypoint and workflow owner.
- All other agents are single-purpose workers.
- Every agent must read this file before executing its own role file.
- Do not assume the current working directory is the correct repo or service root.
- Resolve execution roots before running workflow steps.

### Root resolution model

Every run should track these roots:

- `workspace_root` — where the host started or where the prompt pack was loaded
- `service_root` — the most relevant service/package/app directory for the reported bug
- `git_root` — the repository root used for branch, commit, push, and PR operations

Rules:

1. Start discovery from `workspace_root`.
2. Scan the folder structure for manifests, service folders, and `.git` directories.
3. Infer `service_root` from bug context, file matches, manifests, and existing code layout.
4. Resolve `git_root` as the nearest matching git repository for the chosen `service_root`, or the monorepo root when git is shared.
5. Run build/test/lint commands from `service_root` or the nearest package root.
6. Run branch/commit/push/PR actions from `git_root`.
7. If multiple candidates exist, choose the best-supported one and record the evidence in workflow state instead of asking the human unless safe progress is impossible.

### Non-interactive autonomy contract

Once the workflow has started:

- do **not** ask the human whether to continue to the next phase,
- do **not** ask the human to approve internal handoffs between agents,
- treat worker-agent transitions as internal workflow steps only,
- continue automatically until success, a non-retryable blocker, or host-enforced capability limits,
- if the host denies an action or requests approval, treat that as a capability constraint and apply fallback behavior instead of asking an extra phase-by-phase question.

## 2. Runtime modes

Every run must classify itself into one of these modes before performing work:

- `FULL_AUTO` — read/write/execute/network/git/PR actions available
- `CONSTRAINED` — some actions available, some blocked by host policy
- `DRY_RUN` — analysis only; no persistent mutations allowed
- `MANUAL_HANDOFF` — prepare exact artifacts for a human or external runner

If the host does not expose an explicit mode, infer the most accurate mode from observed capabilities.

## 3. Capability rules

Agents must reason about capabilities instead of assuming a specific host tool.

### Capability categories

- `read_files`
- `write_files`
- `run_commands`
- `network_access`
- `git_access`
- `pr_access`

### Behavior rules

1. If a capability is available, use it.
2. If a capability is blocked, degrade gracefully.
3. Never invent missing API data, file contents, command output, or git state.
4. Never loop forever on a blocked capability.
5. Prefer `partial` completion with artifacts over opaque failure.
6. Do not convert internal workflow transitions into user approval prompts.
7. Do not treat `workspace_root` as `git_root` until repository discovery has confirmed it.

## 4. Standard status values

Every agent output must use one of these statuses:

- `success` — requested work completed
- `partial` — useful artifacts produced, but one or more blocked capabilities prevented full completion
- `failed` — work could not proceed or produced no safe result

## 5. Standard output envelope

Every agent should return results using this shape:

```json
{
  "status": "success | partial | failed",
  "agent": "ORCHESTRATOR | FETCH_PLAN_AGENT | FIXER_AGENT | TESTING_AGENT | PR_AGENT",
  "summary": "Short human-readable result",
  "warnings": ["Optional warning"],
  "errors": [
    {
      "code": "ERR_SOMETHING",
      "message": "Human-readable explanation",
      "retryable": false
    }
  ],
  "artifacts": {},
  "next_action": "What the orchestrator or operator should do next"
}
```

## 6. Standard error codes

Use these shared codes where applicable:

- `ERR_ENV_MISSING`
- `ERR_ENV_KEY_MISSING`
- `ERR_NETWORK_UNAVAILABLE`
- `ERR_LINEAR_UNREACHABLE`
- `ERR_LINEAR_AUTH`
- `ERR_CAPABILITY_BLOCKED`
- `ERR_PROJECT_UNSUPPORTED`
- `ERR_SERVICE_ROOT_UNRESOLVED`
- `ERR_GIT_ROOT_UNRESOLVED`
- `ERR_FILE_NOT_FOUND`
- `ERR_FIX_EMPTY_DIFF`
- `ERR_TEST_FRAMEWORK_UNDETECTED`
- `ERR_TEST_EXECUTION_FAILED`
- `ERR_GIT_UNAVAILABLE`
- `ERR_PR_PROVIDER_UNAVAILABLE`
- `ERR_PR_CREATION_BLOCKED`
- `ERR_RETRY_EXHAUSTED`

If a more specific code is needed, keep the `ERR_*` naming pattern.

## 7. Workflow state contract

The orchestrator owns workflow state. Worker agents may propose updates through their `artifacts`, but the orchestrator is responsible for consolidating state.

Recommended fields:

```json
{
  "run_id": "<uuid>",
  "mode": "FULL_AUTO | CONSTRAINED | DRY_RUN | MANUAL_HANDOFF",
  "workspace_root": "",
  "service_root": "",
  "git_root": "",
  "root_evidence": [],
  "capabilities": {},
  "bug": {},
  "plan": {},
  "fix": {},
  "tests": {},
  "pr": {},
  "audit": []
}
```

## 8. Retry rules

- Retry only when the error is plausibly transient.
- Do **not** retry missing files, missing env keys, unsupported project types, or blocked capabilities more than once.
- Use at most 3 fix/test loops unless the host explicitly asks for more.
- If the same failure repeats with no new signal, stop and emit `ERR_RETRY_EXHAUSTED`.

## 9. Fallback rules by capability

### If execution roots are unclear
- scan for manifests and `.git` directories before failing,
- prefer the root that contains the affected files,
- if no safe root can be resolved, return `failed` with `ERR_SERVICE_ROOT_UNRESOLVED` or `ERR_GIT_ROOT_UNRESOLVED`.

### If file writes are blocked
- Produce a minimal patch or explicit edit instructions.

### If command execution is blocked
- Produce the exact commands that should be run and explain why they were not executed.

### If network access is blocked
- Skip Linear/API calls and return `partial` or `failed` depending on whether meaningful work can continue.

### If git/PR actions are blocked
- Generate branch name, commit message, PR title, and PR body as artifacts.

## 10. Safety rules

- Never guess missing credentials.
- Never claim a command passed without executing it.
- Never claim a PR exists unless a URL or provider response confirms it.
- Never delete unrelated files.
- Never broaden the scope beyond the reported bug without explicitly flagging it.

## 11. Host compatibility guidance

These prompts are designed for multiple hosts. Avoid depending on host-specific permission files, magic slash commands, or proprietary tool names inside agent behavior unless clearly marked as optional examples.

## 12. Stop conditions

An agent must stop when:

- it has completed its task,
- required input is missing,
- a non-retryable error occurs,
- or host capability limits prevent safe progress.

When stopping early, the agent must still return the standard output envelope.

If stopping because of a blocked host action, explain the blocked capability and provide the next best fallback artifact instead of asking, "Should I continue?"