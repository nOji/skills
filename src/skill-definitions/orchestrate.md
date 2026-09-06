---
name: orchestrate
description: Run an existing task series through Codex, Claude Code, or Cursor Agent with an approved dependency schedule, optional parallel execution, per-task reviewers, and coordinated review and commit completion.
---

# Coordinate a task series

You coordinate execution, review exchanges, tracking, and authorized commits.
Child agents implement and review.
Do not study implementation source or decide technical findings yourself.
Use task definitions, dependency information, ownership metadata, tracking files, and final agent reports.

Read the [harness reference](#harness-cli-reference) before running children and the [review protocol](#review-protocol) for reviewed tasks.
Read only the adjacent parent guide matching the agent that loaded this skill: `codex.md`, `claude.md`, or `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Agree on the run

Use the user's request and task documents to enumerate the series without redefining its tasks.
Resolve only missing decisions:

- Tasks, dependencies, and any shared files or mutable resources.
- Implementor harness and model, including per-task overrides when requested.
- Review coverage and reviewer harness/model for each reviewed task.
- Execution order and which independent tasks may overlap.
- Batch boundaries and whether to commit after each completed task.
- Existing tracking and outcome-log conventions, and how to use delegates' commit suggestions.

Use the harness reference's discovery and model-selection instructions.
Retain selections and full-access authorization already provided.
Do not add a separate memory-settings question; preserve each harness's normal configuration unless the user requests a change.

**Ask about reviews unless the user explicitly declined them.**
Support reviewing all tasks, selected tasks, or none, with different reviewers for different tasks.
Do not treat an omitted reviewer assignment as permission to skip review.
A review explicitly requested for a group or the combined result is its own checkpoint with a defined subject and dependencies.

**Propose parallel execution when independence is plausible, and obtain approval for the concrete schedule.**
Consider shared edits, generated output, test databases, ports, and other mutable resources, not just task numbering.
Serialize conflicting phases or use separate resources already supported by the project.
When independence is uncertain, use sequential execution.

Ask for a batch size only when the requested stopping boundary is missing.
"The rest" or an explicit range without intermediate stops means the entire requested range.
An approved concurrent batch must settle all its active task and review sessions before its confirmation boundary.

## 2. Select the review route

For each reviewed task, check whether its implementor can invoke an installed, enabled `co-review` skill using the harness reference's discovery and invocation rules.
Check the actual child environment; a skill visible to the orchestrator alone is insufficient.
Do not install, rebuild, or reconfigure skills to make this route available.

When usable, **prefer resuming the original implementor with co-review**.
Tell the user that the implementor will run its selected reviewer and resolve the findings itself.
Show this route in the plan table.
The implementor keeps its task context and owns every fix.

Otherwise use **orchestrator-mediated review**.
Launch a separate reviewer, send it the implementor's final report and the review brief, and relay the exchange yourself.
Both routes use the shared mutual-agreement protocol and the user's selected reviewer.
Honor an explicit route preference.

For a combined review, identify the author session responsible for responding and making any cross-task fixes before starting that checkpoint.
Do not let two author sessions make competing integration fixes.

## 3. Present the plan and get confirmation

Present a readable Markdown table using actual task names and selected model display names.
For example:

| Order / ready condition | Task | Depends on | Implementor | Reviewer | Review route |
| --- | --- | --- | --- | --- | --- |
| Start together ∥ | Task A | — | Selected model | Selected reviewer | co-review |
| Start together ∥ | Task B | — | Selected model | None, as requested | — |
| After A is done | Task C | A | Selected model | Different reviewer | Relayed |

Below it, state the batch stopping points, commit policy, and full host access for the run, resumes, and nested reviews.
Explain any shared-resource phases that must run sequentially.
Ask for confirmation before the first launch.
Reuse approval of this exact plan rather than asking again.

## 4. Execute the approved schedule

Maintain a compact record per task:

- Task scope, dependencies, file/resource ownership, and selected settings.
- Implementor session ID and, when applicable, reviewer session ID.
- Review route, round, finding ledger or delegated review report, and response paths.
- State: `waiting`, `implementing`, `reviewing`, `fixing`, `finalizing`, `done`, `blocked`, or `failed`.

Before a launch, capture the harness reference's ownership baseline.
For tasks launched together, use the same initial batch boundary and record each task's ownership.
Give each task and role separate run files.

Build a minimal prompt from the user's task or its definition.
Add only decisions needed for this execution: its review assignment, ownership, shared-resource coordination, and commit timing.
Do not repeat automatically loaded agent instructions, ask it to read generic context documents, or restate routine reporting conventions.

For concurrent work, include this brief instruction:

> Other tasks are running in this checkout.
> Unrelated changes are expected; leave them intact and do not stage or commit them.
> Stay within this task's ownership and report an overlap so the conflicting phase can be serialized.

Keep shared tracking writes and Git index operations serialized too.
The implementor must wait for its commit turn even if its normal task instructions suggest committing immediately.

Launch ready tasks through the parent's managed asynchronous facility.
Listen to all active handles and process results as they arrive.
Do not inspect streamed transcripts or read implementation source while waiting.

A task enters review as soon as its implementor finishes, while independent tasks continue.
Keep its author from editing the reviewed work during a reviewer round.
A dependent task becomes ready only after its prerequisites have completed their assigned reviews, tracking, and required commits.
No-review tasks proceed directly to finalization.

## 5. Run the assigned review

### Preferred route: implementor-managed co-review

Resume the original implementor with the harness's explicit skill invocation.
Pass the selected reviewer harness/model/effort/speed, task scope, and the approved execution constraints.
Carry forward the run's full host authorization and any concurrency or commit restrictions.
These are supplied user decisions, so the child must not ask the user to choose them again.

The implementor runs co-review, owns the fixes and rebuttals, and returns the final mutual-agreement report.
Keep the task in `reviewing` or `fixing` until the report accounts for every finding and confirms review of the final work.
A report that merely says "implemented" or "no blockers" without settling open items is incomplete; resume the implementor to finish the exchange.

If co-review cannot be loaded, retain the implementation and switch to the embedded relayed route with the same reviewer selection.
Report the route change briefly.
A harness, model, access, or quota failure follows the failure rules instead; changing routes must not bypass that failure.

### Fallback route: relay the review exchange

Launch the selected reviewer with the shared protocol's brief and reviewer instructions.
Include the implementor's original final response as claims to verify, the task definition, and its ownership boundary.
The reviewer has full evidence-gathering access but does not repair the work.

After the reviewer exits:

1. Send its findings and evidence to the same implementor session.
   Ask for a response to every finding, accepted fixes, evidence for rebuttals, and a list of changes.
2. Update the ledger from the implementor's response.
3. Resume the same reviewer with that response and the ledger.
   Require review of the affected final state and an explicit disposition for each item.
4. Relay contested or new items back to the implementor and continue.

Send even a clean initial review to the implementor for acceptance.
If it accepts without changing the work, no extra reviewer round is needed.
Any further edit to the reviewed work requires another reviewer round.

You coordinate agreement; you do not decide that a finding is wrong, make a fix, or close an unanswered item yourself.
Finish only when the shared mutual-agreement completion conditions hold.

## 6. Finalize a task

Confirm required implementation and review reports are complete before marking the task done.
Verify its state-tracking convention.
If tracking is missing or wrong, resume the same implementor to correct it; do not silently advance or edit its task content yourself.
Bookkeeping outside the review subject may follow sign-off; changes to reviewed work require re-review.

Write a concise outcome record in the agreed location: what completed, review disposition, response paths, and any remaining verification or user action.
Distinguish verified review conclusions from an unreviewed implementor's own report.

If commits were approved, commit only after the task's review is resolved.
For sequential work, stage and commit only changes proven to belong to that task.
For concurrent work, resume the original implementor for its own commit and serialize these follow-ups so they cannot race over the shared index.
Preserve pre-existing and other agents' changes, use the agreed commit-message convention, and record the commit hash.
Do not create commits when the run's policy does not authorize them.

Report the task's outcome briefly and launch newly ready work within the approved batch.

## 7. Boundaries, blockers, and failures

At a batch boundary, let every already-started task and review in that batch settle.
Show a consolidated table with task status, review status, outcome or blocker, and commit when applicable.
Link the outcome records, identify the next tasks, and wait for the user's continuation before launching another batch.

If a task needs a user decision, stop launching new work.
Let already-active independent sessions settle, record all results, and present the blocking question without answering it on the user's behalf.
Resume the same affected session after the answer.
Do not start dependent work or call the task complete while blocked.

On process failure, preserve work and session IDs, materialize the bounded diagnostic using the harness reference, and stop new launches.
Do not skip the failed task, silently change models, or take over its implementation or review.
When a supplied full-access launch was accidentally restricted, correct the launch to the already-authorized settings and resume; no new permission question is needed.
An actual host-policy rejection, missing credential, or unavailable service remains a real blocker.
