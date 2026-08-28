# Cursor parent guidance

Read this file only when **Cursor is the agent that loaded the skill** and is supervising the delegated work.
These rules are about Cursor as the parent agent.
They do not replace or relocate the child-Cursor launch, model, resume, review-mode, or failure commands in the embedded harness CLI reference.

## Use Cursor's managed asynchronous task facility

Launch and resume child runs through Cursor's native managed background-task or yielded-session facility.
Retain the returned task or session handle and wait on it until the process exits.

Keep the harness shell block in the foreground inside that managed task so its post-exit extraction runs in order.
Do not add shell `&`, `nohup`, or another detached subprocess.
If no managed asynchronous process facility is available, stop and explain that the workflow cannot run safely.
