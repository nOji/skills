# Child runtime

Choose exactly one runtime before discovering or launching any child.

Check whether you can call an `orchestrator_capabilities` tool.
Its name may carry an MCP server prefix, such as `mcp__t3-code__orchestrator_capabilities` or `mcp__t3_code__orchestrator_capabilities`.
It may be listed only as a deferred or lazily loaded tool; load its schema and make one direct call before concluding it is absent.
If the call succeeds, use **orchestrator tools**.
Otherwise use **harness CLIs**.
Keep the chosen runtime for the whole task, including continuations and nested reviews you launch.

## Orchestrator tools

Use the tools that accompany `orchestrator_capabilities`, with the same name prefix.
Do not read the adjacent `harness-cli.md`, `codex.md`, or `claude.md`; they apply only to harness CLIs.

| Operation | Tool use |
| --- | --- |
| Discover | Call `orchestrator_capabilities`; offer only its listed provider instances, models, model options, and runtime modes as harness, model, effort, and speed choices |
| Launch a session | `delegate_task` with the selected provider instance, model, and options as `target`, `mode: "async"`, a fitting `role`, the minimal task prompt, and a `clientRequestId` unique to this launch and stable across its retries |
| Continue a session | `t3_thread_send` to the session's `childThreadId` with a new `clientRequestId`; it returns the round's own `taskId` |
| Start a fresh session | A new `delegate_task` with a self-contained brief and the same selected settings |
| Wait | End the turn or idle until a completion notification arrives; call `task_status` only when the result is needed mid-turn |
| Read the result | `task_status`; its `summary` is the round's final response |
| Cancel | `task_cancel` with the round's `taskId` |

Request `runtimeMode: "full-access"` when full access is authorized and the target lists it; otherwise report the narrower available mode before launching.
Record each session's `childThreadId` as its session identifier and each round's `taskId` as that round's handle.
Save each round's `summary` to its round-specific response file under `.agent-runs/` before acting on it.
Treat a failed task, a provider error in its summary, or a missing summary as a process failure.
A completed task with pending child runs is not finished until those runs settle.

When a workflow continues the same session, use `t3_thread_send`, even for a review round; follow this workflow's continuity choice rather than any general default to start a new task per round.

## Harness CLIs

Read the adjacent `harness-cli.md` completely before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md` and Claude Code reads `claude.md`.
For another parent, use its native managed asynchronous process facility.
