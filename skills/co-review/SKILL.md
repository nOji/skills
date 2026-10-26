---
name: co-review
description: Obtain an independent review of a plan, diff, or task output from Codex or Claude Code, then resolve findings through evidence, fixes, and rebuttals until both participants agree.
---

# Resolve an independent review

You are the author and the reviewer's counterpart.
The child reviewer reports findings; you own fixes.
Use the **mutual-agreement loop** in the [review protocol](#review-protocol).

Before running a child, choose its runtime in the [child runtime](#child-runtime) section and follow the [delegation rules](#delegation-rules).

## Child runtime

Choose exactly one runtime before discovering or launching any child.

Check whether you can call an `orchestrator_capabilities` tool.
Its name may carry an MCP server prefix, such as `mcp__t3-code__orchestrator_capabilities` or `mcp__t3_code__orchestrator_capabilities`.
It may be listed only as a deferred or lazily loaded tool; load its schema and make one direct call before concluding it is absent.
If the call succeeds, use **orchestrator tools**.
Otherwise use **harness CLIs**.
Keep the chosen runtime for the whole task, including continuations and nested reviews you launch.

### Orchestrator tools

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

### Harness CLIs

Read the adjacent `harness-cli.md` completely before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md` and Claude Code reads `claude.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Configure the reviewer

Before studying the review subject, discover available harnesses and models using the selected runtime.
Reuse choices and authorization already supplied by the user or an orchestrator acting on the user's approved plan.
Do not repeat a model menu, permission question, or review-scope question that has already been answered.

Require the user's explicit selections under the delegation rules before launching the reviewer.

Settle review continuity as the review protocol describes.
Keep the chosen reviewer settings, review continuity, and full host access across all rounds.
If this skill was delegated, return genuinely missing user decisions to the coordinator rather than waiting for interactive input inside a headless run.

## 2. Brief and launch

If the user names a subject, use it.
Otherwise review the current conversation's work, including relevant committed and uncommitted changes.
Reconstruct the goal and file list from existing context and cheap Git metadata; do not pre-read the code to seed the review.

Build the brief from the shared review protocol.
Include the original implementor's completion report when this review follows delegated implementation.
Use the delegation rules' minimal-prompt rule; do not repeat default instructions or ask for routine output the reviewer already knows to provide.

Capture a pre-review content baseline and the task's ownership boundary.
In parallel work, retain the coordinator's ownership and shared-resource constraints.
Launch the reviewer with normal evidence-gathering capabilities and the protocol's no-repair instructions.
Do not use plan mode.

## 3. Exchange evidence and fixes

Wait for the reviewer run to finish, then read its final response.
Check that it did not alter the reviewed work, accounting for expected concurrent changes.
Now inspect the relevant files, verify each finding, and respond as confirmed, rejected, or a judgment call.

Maintain the shared ledger.
Make accepted fixes, log all changes, and send the ledger, fixes, rebuttals, and judgment calls to the reviewer according to the selected review continuity.
Require an explicit disposition for every open item and a review of the affected final state.
Every remediation edit requires another review.

Use the shared ownership and staging rules.
When other tasks are active, do not stage or commit without the coordinator's serialized turn.
Continue autonomously until the mutual-agreement completion conditions hold or a user decision is required.

## 4. Close out or return to the coordinator

Before claiming resolution, check every ledger item and confirm the reviewer saw the final changes.
A clean first review that you verified and accepted closes immediately.
A review with unverified fixes or unanswered rebuttals remains open.

Report the reviewer, final verdict, ledger, fixes, accepted rebuttals, checks performed by each participant, and remaining limitations.
Link the final response file and mention any staging.
Follow the user's commit-message convention for remediation; commit only when authorized.

When invoked by an orchestrator, return that report to it.
Keep the task in review until both participants have agreed, and leave orchestration tracking and commit timing to the approved schedule.
On process failure, use the delegation rules' failure and recovery rules and preserve the sessions and completed work.

## Delegation rules

Shared selection, access, prompt, ownership, session, and failure rules for every child agent, whichever runtime launches it.
The [child runtime](#child-runtime) section decides how children are discovered, launched, continued, and cancelled.

### 1. Select children

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

### 2. Preserve normal capabilities

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

### 3. Minimal prompts and explicit skills

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

### 4. Run records and ownership

Before creating run artifacts in a Git working tree, establish a local ignore rule:

```bash
agent_exclude_path="$(git rev-parse --git-path info/exclude)" || exit 1
mkdir -p "$(dirname "$agent_exclude_path")"
touch "$agent_exclude_path"
grep -qxF '/.agent-runs/' "$agent_exclude_path" ||
  printf '\n/.agent-runs/\n' >> "$agent_exclude_path"
git check-ignore -q --no-index .agent-runs/.ignore-check || exit 1
test -z "$(git ls-files -- .agent-runs)" || {
  echo "Run artifacts are already tracked; resolve that before delegation." >&2
  exit 1
}
mkdir -p .agent-runs
```

This changes local Git metadata, not tracked `.gitignore`.
If the directory is not a Git working tree or run artifacts cannot be kept untracked, stop before launching.

Capture ownership without dumping source into the parent context:
save `git diff --cached --binary HEAD` and `git diff --binary` as separate files, plus a NUL-safe untracked-file manifest with path, type, and SHA-256 content hash.
For an unborn repository, use the index-versus-empty-tree baseline instead of `HEAD`.
Retain content needed for any safe restoration; hashes alone cannot restore a modified untracked file.

Capture the initial baseline before implementation and a review baseline before each reviewer round.
Preserve pre-existing user work and other agents' changes.
Only stage, revert, or commit deltas whose ownership is established.
Never use broad staging such as `git add .` or `git add -A`.
In concurrent work, keep ownership boundaries and serialize all shared-index changes.

Give each task and role a unique stem, such as `.agent-runs/<task>-<role>`, and a new round-specific final-response file, such as `...-r1.response.md`, for every round.
Keep nested review artifacts distinct.
Store artifacts here rather than in `/tmp`; pass prompts directly instead of creating prompt files.
Read only each round's final response, never a streamed transcript, including for progress or failure diagnosis.

### 5. Sessions and continuations

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

### 6. Failure and recovery

A failed or cancelled run, a missing final response, or a missing required terminal result is a process failure.
Keep failure reporting separate from a successful run that asks a task-level question.
Materialize a bounded diagnostic into a round-specific response file and keep an existing useful final report intact.

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

## Review protocol

Use this protocol when assessing an implementation, plan, or other task output.
The skill assigns the roles and the completion rule:

| Workflow | Author | Reviewer | Coordinator | Completion |
| --- | --- | --- | --- | --- |
| co-implement | Child implementor | Parent | Parent | Supervisor sign-off |
| co-review | Parent | Child reviewer | Parent | Mutual agreement |
| orchestrate, relayed review | Child implementor | Child reviewer | Orchestrator | Mutual agreement |
| orchestrate, delegated co-review | Original implementor running co-review | Its child reviewer | Orchestrator tracks completion | Mutual agreement |

The author owns fixes.
The reviewer independently assesses the work.
A coordinator that is neither author nor reviewer routes their reports and tracks resolution without deciding technical findings itself.

### Review continuity

When a child reviewer takes part, settle how its sessions carry across rounds before the first reviewer launch.
Reuse a choice the user or an approved orchestration plan already supplied; otherwise ask in plain text together with the other missing reviewer choices, for example:

> How should the reviewer handle follow-up rounds?
> - **Continuous:** one reviewer session handles every round and builds on its earlier reasoning.
> - **Fresh each round:** each round starts a new reviewer session that reassesses the current work from the brief, ledger, and latest responses, without earlier context.

Continuous rounds are cheaper and keep the reviewer's context; fresh rounds reduce anchoring on earlier conclusions.
Apply the choice to every reviewer round of the exchange and forward it as a supplied decision to any nested review.
It governs only child reviewer sessions; the author always continues its own session.
A workflow whose reviewer is the parent itself does not ask.

### Review brief

Give the reviewer the task, its constraints, the task-owned changed-file list, and the author's completion report when available.
Include task or specification paths supplied by the user; preserve their scope.
Treat the completion report as claims to check against the actual work.
Do not seed the review with the coordinator's suspicions or copy instructions the child normally loads.

Add only these review-specific instructions:

- Inspect the actual work and relevant dependencies against the task's requirements.
- Report defects, missing requirements, regressions, and unsupported completion claims.
- Do not repair the work, edit task definitions or tracking, or stage or commit.
- Give numbered findings with location, consequence, evidence, and what would resolve the issue.
- Separate blocking defects from judgment calls.
- State when no findings remain.
- Assess rebuttals independently and explicitly accept a convincing correction.

Both participants may use the harness's normal tools and authorized services to gather evidence.
Review is a role restriction, not a reason to remove shell, network, database, or browser capabilities.
Run checks appropriate to the work; documentation-only work does not call for code tests.

### Evidence and responses

Neither participant treats the other's report as proof.
Read the relevant work, reproduce claimed failures when practical, and distinguish observations from inferences.
Consider skipped requirements, integration behavior, error paths, and unintended scope changes.
Avoid speculative findings or changes justified only by personal taste.

The author should be careful of scope creep and should not hesitate to push back on critiques that are non-issues or introduce work beyond the user's request, explaining why.
When sending review feedback to a child author, include this instruction in the follow-up.

For each finding, the author chooses one response:

| Response | Required action |
| --- | --- |
| Confirmed | Fix it and report the change and verification |
| Rejected | Give evidence explaining why the finding is incorrect or the requested change exceeds the user's scope |
| Judgment call | State the chosen approach and its trade-off |

Invite pushback in both directions.
Accept the stronger evidence, regardless of which agent supplied it.
A finding's severity does not determine whether it has been resolved.

### Mutual-agreement loop

Use this loop for co-review and every orchestration review.
Keep stable finding IDs and a compact ledger with these states:

- `open`
- `fixed-awaiting-review`
- `rebutted-awaiting-response`
- `closed-both-agreed`

1. The reviewer returns its findings.
2. The author verifies each finding, makes the fixes it accepts, and answers every item.
3. Send the updated ledger and the author's response to the reviewer according to the selected continuity: continue the same reviewer session, or start a fresh reviewer session with the fresh-round brief from the delegation rules.
   Include all edits since its last review, including changes it did not request.
4. The reviewer inspects the current work, checks the fixes for regressions, and explicitly closes or contests each item by ID.
5. Return any contested or new findings to the same author session and repeat.

Preserve the author's session identifier and every reviewer session identifier throughout the exchange.
When a coordinator relays messages, pass the participants' final reports and evidence faithfully; do not substitute the coordinator's technical verdict.
Compact repeated history into the ledger, but retain unresolved arguments and evidence.

Every edit after a review requires another review of the affected final state.
A rejection remains open until the reviewer explicitly accepts the rebuttal.
Silence, a lower severity, or a general "no blockers" statement does not close an unaddressed item.
Retain mutually closed items with their resolution so they do not disappear from the record.

Completion requires every item to be `closed-both-agreed`, the author's acceptance of the final disposition, and a reviewer verdict covering the final task state.
A clean first review that the author verifies and accepts needs no extra round.
If another task changes a relevant dependency after sign-off, recheck the affected conclusions before treating them as final.

### Supervisor sign-off

For co-implement, the parent reviewer independently checks the task-owned diff and completion claims.
Send substantive defects and contestable choices back to the same implementor session.
Re-review its subsequent changes until no blocking issue remains.

The supervisor may directly fix an obvious, uncontested mistake.
If that clears the last issue, verify and sign off without another delegate round.
If another round is needed, tell the implementor what the supervisor changed.
This exception does not apply to the mutual-agreement loop.

### Protect ownership during review

Use the content baselines and ownership rules in the delegation rules.
The author pauses edits to the reviewed task while its reviewer runs.
Independent tasks may continue only where their files and mutable resources do not conflict with that review.

After review, check for unintended changes to the reviewed work.
In a shared checkout, other agents' expected changes are not reviewer edits.
Never restore a repository-wide snapshot over concurrent work.
Undo only a delta proven to belong to the reviewer; if ownership cannot be established, preserve the files and resolve the conflict before continuing.

In a single-writer workflow, stage only reviewed task-owned hunks when that gives the next round a useful diff boundary.
Leave mixed-ownership paths unstaged when separation is uncertain.
During parallel work, keep the shared index unchanged until the coordinator grants a serialized staging or commit turn.

### Decisions and close-out

Continue technical discussion autonomously while the participants are making progress.
Ask the user only for a requirement, scope, or consequential product decision the task does not answer, or a disagreement the participants cannot reconcile with evidence.
Present both positions and the remaining question; preserve the sessions for resumption.
Do not call an unresolved exchange complete.

Report the outcome, accepted fixes, accepted rebuttals, checks actually performed, and any remaining limitation.
For mutual-agreement reviews, include the final ledger and both participants' explicit disposition.
Keep reports concise and reference the final response files rather than streamed transcripts.
