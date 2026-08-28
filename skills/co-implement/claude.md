# Claude Code parent guidance

Read this file only when **Claude Code is the agent that loaded the skill** and is supervising the delegated work.
These rules are about Claude Code as the parent agent.
They do not replace or relocate the child-Claude launch, model, resume, review-mode, or failure commands in the embedded harness CLI reference.

## Use Claude Code's managed background task

Launch and resume child runs with the Bash tool's `run_in_background: true` option.
Retain the returned task id and wait on that task until the process exits.

Keep the harness shell block in the foreground inside that managed background task so its post-exit extraction runs in order.
Do not add shell `&`, `nohup`, or another detached subprocess.
If no managed background-task facility is available, stop and explain that the workflow cannot run safely.
