# Delegation rules

Shared selection, access, prompt, ownership, session, and failure rules for every child agent, whichever runtime launches it.
The [child runtime](#child-runtime) section decides how children are discovered, launched, continued, and cancelled.

## 1. Select children

Retain actual discovery errors and mark unavailable harnesses with a short reason.
Authentication and catalogs do not guarantee quota or model access; handle execution failures when they occur.

Present a compact menu only for choices still missing.
Show display names, one line per harness, and ask for the model, supported effort, and available speed choice.
Shortlist large model families rather than printing every variant.
Require explicit user choices for the harness and model, plus effort and speed wherever supported.
Do not supply default recommendations or select missing values on the user's behalf, including from saved CLI settings.
Reuse choices the user already supplied; ask only for missing supported choices and wait for them before launching a child.
Treat selections supplied through an approved orchestration prompt as already answered.
If delegated, return missing user decisions to the coordinator instead of waiting for interactive input inside a headless run.
Do not switch harness, model, effort, or requested speed on your own.
If runtime metadata reports a substituted model, surface it rather than attributing the work to the requested model.

## 2. Preserve normal capabilities

Use full host access for the authorized task, including local services, Docker, networking, continuations, and nested reviews.
Carry existing authorization forward; do not insert another approval question at every launch or review round.
Honor a user-requested restriction and any enforced host or organization policy.

Start each child in the task's actual working directory with the normal user identity, environment, login, settings, skills, plugins, and configured integrations.
When the parent is instructed to work in a Git worktree, whether selecting an existing one or creating one, resolve that worktree's absolute path before delegating and use it for every child launch and continuation.
Explicitly tell each child, including implementors and reviewers, to work in that worktree and pass its absolute path in the task prompt.
Require children to carry the same worktree path and instruction forward to any agents they summon, including nested reviews, unless the user explicitly assigns a different worktree to that task.

Keep each child's default system prompt, memory, context, and compaction behavior unless the user chose an override.
Do not remove skills, session persistence, integrations, or other normal capabilities to simplify delegation.
Normal configuration is each child's own configuration; a child does not automatically inherit another product's settings or the parent chat's desktop-only tools.
Preserve ordinary automatic instruction discovery instead of copying those instructions into the task prompt.
Do not claim unavailable integrations are present.

Run implementors and reviewers in normal agent mode.
A reviewer receives a no-repair role instruction, with normal tools available for verification.
Plan mode changes the deliverable and is not a substitute for review.

## 3. Minimal prompts and explicit skills

Pass the user's task and only missing execution-specific context: task scope, selected reviewer, approved access, ownership, concurrency, and coordination boundaries.
Include a task document when the user named it.
Treat applicable base prompts and global/project instruction files as available through each child's normal loading mechanisms.
Do not copy, summarize, repeat, or add reminders to read these instructions unless the user explicitly asks.
Apply this rule to launches, continuations, and nested delegation, while preserving each harness's own normal configuration.
Pass runtime decisions the child does not already have; do not restate inherited conventions.
Ask for an extra report field only when the workflow needs it and it is otherwise missing.

When delegating a skill, put the explicit invocation at the beginning of the child's user prompt:

| Child | Prompt form |
| --- | --- |
| Codex | `$co-review`, or the path-qualified mention `[$co-review](/absolute/skills/co-review/SKILL.md)` |
| Claude Code | `/co-review <task and supplied choices>` |

Use the actual registered command for a namespaced plugin skill.
The skill name is prompt content, not a shell command or a model flag.
Codex documents explicit `$skill` invocation; Claude documents slash-skill expansion in headless prompts.
See [Codex skills](https://learn.chatgpt.com/docs/build-skills) and [Claude headless skills](https://code.claude.com/docs/en/headless).

Before offering an installed skill route, resolve its real entrypoint and confirm it is enabled and usable by the selected child in this working directory.
Use an existing skill listing when available, then check the relevant normal discovery locations:

| Child | Normal locations to check |
| --- | --- |
| Codex | Applicable ancestor `.agents/skills/` directories and `~/.agents/skills/`, plus configured plugin skills |
| Claude Code | Applicable `.claude/skills/`, `~/.claude/skills/`, and installed plugin skills |

Follow symlinks and actual configuration; do not assume a directory exists or a parent-visible skill is registered in every child.
File existence alone does not prove that a particular child version will expand its command.
If native loading fails, report the failure and use the workflow's embedded alternative.
Do not install a missing skill, enable a disabled one, or disguise its contents as a workaround.

Forward already approved reviewer settings, review continuity, and access as user decisions.
Explicit invocation does not itself override permissions; the launch settings establish the authorized access.
An unattended child returns any genuinely missing decision to its coordinator instead of waiting on an interactive question.

## 4. Run records and ownership

Use the selected runtime's run records for session identifiers, round results, and bounded failure diagnostics.
Keep each task, role, round, and nested review distinct.
Pass prompts directly instead of creating prompt files.
Read only each round's final response, never a streamed transcript, including for progress or failure diagnosis.

Capture ownership without dumping source into the parent context:
retain staged and unstaged content baselines separately, plus a NUL-safe untracked-file manifest with path, type, and SHA-256 content hash.
For an unborn repository, use the index-versus-empty-tree baseline instead of `HEAD`.
Retain content needed for any safe restoration; hashes alone cannot restore a modified untracked file.

Capture the initial baseline before implementation and a review baseline before each reviewer round.
Preserve pre-existing user work and other agents' changes.
Only stage, revert, or commit deltas whose ownership is established.
Never use broad staging such as `git add .` or `git add -A`.
In concurrent work, keep ownership boundaries and serialize all shared-index changes.

## 5. Sessions and continuations

A session is one child conversation, identified by the exact identifier its runtime returned.
Record that identifier for every task and role.
Never replace a recorded identifier with a different one, guess one from recency, or use the parent's own identifier.
If no exact identifier can be recovered from this task's own runtime metadata, report that fact rather than guessing.

A continuation sends a follow-up to the same session, which keeps its earlier context.
Resuming a session means continuing it.
A fresh session starts a new child conversation with the same selected settings and a self-contained brief; it shares no context with earlier sessions.
Workflows say which of the two each round uses.
A fresh reviewer round must receive the original review brief, the current ledger, the author's responses and fixes since the previous round, and every unresolved objection.

A successful run can still describe a blocked or incomplete task; inspect its meaning.

## 6. Failure and recovery

A failed or cancelled run, a missing final response, or a missing required terminal result is a process failure.
Keep failure reporting separate from a successful run that asks a task-level question.
Retain a bounded diagnostic in the selected runtime's record for that round and keep an existing useful final report intact.

Correct an accidental launch restriction to the already-authorized full-access settings and continue the same task without asking for permission again.
For a parent tool that prematurely ended the child, retry once through its proper managed facility.
A transient capacity failure permits one short delayed retry.
Do not retry quota, authentication, policy, or repeated capacity failures in a loop.

Report the actual blocker and current work.
Offer another available harness before offering to take over yourself, and wait for authorization to change the selected harness or role.
A new harness requires a fresh task brief; the previous harness's session identifier is not portable.
An enforced denial is not a reason to clear security settings or bypass a host control.

In orchestration, stop new launches and settle already-active work before presenting blockers.
Preserve completed implementation, review ledgers, and resumable sessions.
Cancel through the runtime's own task handle or a verified task-specific process ID; do not use a broad name-based kill.
