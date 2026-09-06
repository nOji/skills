# Cursor parent guidance

Apply this guide only when Cursor loaded the skill.
Child CLI settings remain in the embedded harness reference.

Use Cursor's approved host execution path so an outer terminal sandbox does not restrict the child CLI.
Carry existing full-access authorization through launches, resumes, and nested reviews.
Use the actual execution options exposed by the parent tool; do not invent a tool parameter or assume child flags remove the outer sandbox.
Respect an enforced rejection.

Launch through Cursor's native managed background-task or yielded-session facility.
Retain each task's handle and wait until its invocation and post-exit extraction finish.
Keep the shell block attached inside that facility; do not add `&`, `nohup`, or another detached process.
Do not read streamed transcripts.
Use the same process facility for every round.
If no managed asynchronous facility is available, report the limitation.
