---
name: delegate-ui
description: Implement a complete feature as the parent agent while delegating only UI presentation and visual design to a user-selected child agent, then review its functional integration. Use when the user requests UI-only delegation or invokes delegate-ui.
---

# Implement the feature and delegate its UI

You own delivery of the complete feature and all logic and data that support it.
The child owns UI presentation and visual design within the user's requirements.
Delegate only that UI work, after preparing its prerequisites yourself.

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
Do not read the adjacent `harness-cli.md`, `codex.md`, `claude.md`, or `cursor.md`; they apply only to harness CLIs.

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
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Select the UI worker

Use the selected runtime to discover available harnesses and models.
Reuse the user's supplied harness, model, and execution settings.
If the harness or model is missing, ask the user to choose the missing value rather than selecting a default.
Resolve remaining execution choices through the selected runtime and retain the selected settings across rounds.
If this skill was delegated, return missing user decisions to the coordinating parent.
You may prepare the feature while awaiting a selection, but do not launch an unselected UI worker.

## 2. Prepare the complete functional foundation

Capture the delegation rules' initial ownership baseline before implementation.
Implement the required backend and frontend logic yourself, including data structures, API integration, transformations, feature state, validation, permissions, and action handlers as applicable.
Expose ready-to-use values and callbacks at the UI boundary, with clear types or contracts.
Prepare the loading, empty, error, success, and disabled conditions that the requested feature needs.
Verify this foundation with checks appropriate to the task before handing it off.

Do not leave the child to invent data shapes, fetch or persist business data, define feature state transitions, or finish application logic.
The child may bind supplied values and callbacks to UI elements and implement presentation behavior such as focus, hover, and animation.
Feature behavior and domain decisions remain yours even when their code lives in a UI file.

Insert concise comment placeholders at the intended UI integration points using the file's native comment syntax.
Give each placeholder a stable, searchable identifier and describe the UI's purpose, available data and state, callbacks, and required behavior.
Keep the surrounding code valid and the supplied interface usable without choosing the child's visual design.
When a comment is impractical, identify the existing component or symbol and the precise region to modify in the handoff instead.
Use current line numbers as navigation aids alongside paths and stable anchors, since edits can move lines.

## 3. Batch the UI handoff

Finish all parent-owned prerequisites for the requested scope, then send one consolidated prompt covering every UI integration point.
Do not launch a separate UI session for each placeholder or file.
Capture a handoff baseline so your implementation can be distinguished from the child's changes.

Use the delegation rules' minimal-prompt and worktree instructions.
Include only the execution context and UI contract the child needs:

- The feature's user-facing purpose and observable behavior that must work.
- Allowed files or regions, identified by path and placeholder, component, or symbol, with current line references where useful.
- Available values, their shapes, state meanings, callbacks, and relevant interface definitions.
- Required UI states, interactions, accessibility behavior, and any explicit user design requirements or supplied references.
- The protected logic and interfaces, the missing-prerequisite return path, and the child's freedom to choose visual design.

Express the boundary as an actionable instruction, for example:

> Implement the presentation for each integration point listed below using the supplied values and callbacks.
> Locate each region by its file path and placeholder identifier or component symbol; line references are navigation aids.
> Choose the layout, styling, typography, and visual treatment within the user's requirements and the project's applicable UI conventions.
> Preserve the supplied data shapes, feature state, validation, permissions, API behavior, and action-handler semantics.
> If an interface or behavior is missing, report the affected integration point, the required value or callback, and the interaction it must support; return that prerequisite to me instead of implementing it yourself or inventing substitute data.
> Remove completed handoff placeholders and report any unfinished UI work or unmet prerequisites.

Follow this instruction with the actual integration-point list and contracts; do not send a generic brief without the task-specific details.
Allow edits to necessary presentation assets or styles in the handoff scope, while explicitly protecting logic even in shared component files.
Launch one UI implementor through the selected runtime and retain its session identifier.
Wait for its run to finish before inspecting its work or editing the handed-off regions.

## 4. Review functionality and supply missing prerequisites

Read the child's final response and inspect its changes against the handoff baseline.
Check that it preserved the supplied contracts and implemented the requested interactions, state rendering, callback bindings, and accessibility behavior.
Run appropriate integration checks and exercise the resulting UI when possible.
Review visual output only to establish that required content and controls are visible, usable, and functionally correct across required conditions.

Leave aesthetic decisions to the child.
Do not redesign its work, impose your own taste, or request cosmetic revisions based on personal preference.
Tie every requested correction to intended functionality, a protected contract, or an explicit user requirement; let the child choose the visual remedy.

If the child reports a missing prerequisite, implement it yourself and update the supplied contract before resuming the same session.
Do not accept a mock, dead control, or altered data shape as a substitute for completing the feature.
If the child changed protected logic, reconcile the task-owned changes using the ownership baseline, retain responsibility for that logic, and return only the remaining UI work to the child.
Preserve unrelated changes throughout.

Batch functional findings and any newly prepared prerequisites into one follow-up to the same UI session with the same selected settings.
Describe the affected integration points, expected behavior, and concrete evidence rather than prescribing a visual redesign.
Repeat until the complete feature works, required UI states are covered, and no handoff placeholders or missing prerequisites remain.
On launch or continuation failure, follow the delegation rules' failure and recovery rules; preserve completed work and the session rather than silently switching models or taking over the UI design.

## 5. Deliver the complete feature

Verify the final integrated feature after the last changes and distinguish completed checks from anything that could not be verified.
Report the parent-owned functionality, delegated UI work, selected UI implementor, final response path, and any remaining limitation.
Do not claim completion while required functionality or UI work remains unfinished.
Follow the user's commit-message convention for the complete task change and commit only when authorized.

## Delegation rules

Shared selection, access, prompt, ownership, session, and failure rules for every child agent, whichever runtime launches it.
The [child runtime](#child-runtime) section decides how children are discovered, launched, continued, and cancelled.

### 1. Select children

Retain actual discovery errors and mark unavailable harnesses with a short reason.
Authentication and catalogs do not guarantee quota or model access; handle execution failures when they occur.

Present a compact menu only for choices still missing.
Show display names, one line per harness, and ask for the model, supported effort, and available speed choice.
Shortlist large model families rather than printing every variant.
Use the calling skill's recommendation only when available.
Treat selections supplied through an approved orchestration prompt as already answered.
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
| Cursor Agent | `/co-review <task and supplied choices>` |
| Muse Code | Interactive `/<skill-name>` is documented; headless expansion is not verified, so use the calling workflow's embedded route unless native loading has been established |

Use the actual registered command for a namespaced plugin skill.
The skill name is prompt content, not a shell command or a model flag.
Codex documents explicit `$skill` invocation; Claude documents slash-skill expansion in headless prompts; Cursor documents explicit slash-skill invocation.
See [Codex skills](https://learn.chatgpt.com/docs/build-skills), [Claude headless skills](https://code.claude.com/docs/en/headless), and [Cursor skills](https://cursor.com/docs/skills).

Before offering an installed skill route, resolve its real entrypoint and confirm it is enabled and usable by the selected child in this working directory.
Use an existing skill listing when available, then check the relevant normal discovery locations:

| Child | Normal locations to check |
| --- | --- |
| Codex | Applicable ancestor `.agents/skills/` directories and `~/.agents/skills/`, plus configured plugin skills |
| Claude Code | Applicable `.claude/skills/`, `~/.claude/skills/`, and installed plugin skills |
| Cursor Agent | Applicable `.agents/skills/`, `.cursor/skills/`, their user-level equivalents, and supported compatibility/plugin locations |
| Muse Code | `muse skills list --enabled-only --workspace "<workspace>" --json`; inspect the chosen ID with `muse skills inspect` |

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
