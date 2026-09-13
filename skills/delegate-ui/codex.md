# Codex parent guidance

Apply this guide only when Codex loaded the skill.
Child CLI settings remain in the embedded harness reference.

Use Codex's approved host command path for child discovery, launches, resumes, and lifecycle operations.
If the parent command tool is sandboxed, set `sandbox_permissions: "require_escalated"` and give a task-specific justification referring to the authorized run.
If it already executes unrestricted on the host, use that path directly.
Do not mistake a failure inside the parent's sandbox for an unavailable child harness.

Carry the user's full-access authorization through the complete task, including other harnesses, resumes, and nested reviews.
Do not ask again for access already granted.
An actual host approval rejection must be reported and respected.

Launch with a short initial yield, retain the returned process-session handle, and wait through Codex's managed input/wait operation until exit.
Keep invocation and post-exit extraction in the same attached shell block.
Use the same mechanism for every round and keep each concurrent task's handle distinct.
Do not add shell backgrounding or read streamed transcripts.
If no managed asynchronous facility is available, report the limitation.

When constructing multiline shell in JavaScript, preserve literal quoting, backslashes, and skill mentions.
A quoted heredoc must reach the shell unchanged.
