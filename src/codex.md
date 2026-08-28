# Codex parent guidance

Read this file only when **Codex is the agent that loaded the skill** and is supervising the delegated work.
These rules are about Codex as the parent agent.
They do not replace or relocate the child-Codex launch, model, resume, review-mode, or failure commands in the embedded harness CLI reference.

## Run every child-harness command on the host shell

Every external coding-agent CLI command must run outside Codex's sandbox through the elevated host-shell path by setting `sandbox_permissions: "require_escalated"` on Codex's command-tool call.
This applies whether the child harness is Codex, Claude Code, or Cursor Agent.
It includes availability and model discovery, launches, resumes, provider status checks, and lifecycle checks or kills.

The elevation setting belongs to the parent command-tool call, not to the child harness CLI.
Keep the command's working directory at the task's absolute working directory and continue to pass the child harness's own documented sandbox or permission flags.
Those child flags take effect only after the process starts and cannot bypass the parent sandbox during initialization.
Supply a concise, task-specific approval justification when requesting the host shell.
Do not try the parent sandbox first or interpret its startup or authentication failure as evidence that the host harness is unavailable.

Host-shell elevation changes only where the child process starts.
It does not broaden the task, authorize additional edits, or relax any implementation or review guard.

## Use Codex's managed asynchronous command session

Launch and resume child runs with command execution using a short initial yield.
Retain the returned session id and continue waiting through Codex's session wait or input operation until the process exits without reading `<LOG>`.

Keep the harness shell block in the foreground inside that managed session so its post-exit extraction runs in order.
Do not add shell `&`, `nohup`, or another detached subprocess.
Use the same elevated host-shell path for the launch, every resume, and any later lifecycle check or kill.
If no managed asynchronous command session is available, stop and explain that the workflow cannot run safely.
