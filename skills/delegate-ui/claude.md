# Claude Code parent guidance

Apply this guide only when Claude Code loaded the skill.
Child CLI settings remain in the embedded harness reference.

Run child CLIs through the parent's approved unsandboxed Bash path.
When the Bash tool exposes `dangerouslyDisableSandbox` and the parent sandbox would restrict the run, use it with the existing full-access authorization.
The child's own sandbox flags do not remove the parent's sandbox.
Respect an enforced rejection instead of bypassing it through another command form.

Use Bash's `run_in_background: true` and retain the returned task ID.
Wait on that task until the invocation and post-exit extraction finish.
Keep the shell block attached inside the managed task; do not add `&` or `nohup`.
Do not return from a headless parent while its required child work is still running.
Use the same process facility and approved access for every resume, without repeating permission questions.
If no managed asynchronous facility is available, report the limitation.
