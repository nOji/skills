---
name: orchestrate
description: Run an existing task series through Codex, Claude Code, or Cursor Agent with an approved dependency schedule, optional parallel execution, opt-in reviews, and commits after completed steps.
---

# Coordinate a task series

You coordinate execution, review exchanges, tracking, and authorized commits.
Child agents implement and review.
Read only instructions governing the run, task or plan documents, dependency and ownership records, orchestration/session tracking, and final agent reports or bounded diagnostics after the relevant process exits.
Do not read implementation modules, tests, source diffs, or produced artifacts, including after a child exits.
Do not reason through the implementation, verify technical claims yourself, or decide technical findings.
Delegate source inspection, implementation decisions, verification, and content-based ownership checks to the appropriate implementor or reviewer.
Mechanical Git metadata, process/session identifiers, and ownership-baseline capture are allowed; keep source and patch content out of your context.

Read the [harness reference](#harness-cli-reference) before running children and the [review protocol](#review-protocol) for reviewed tasks.
Read only the adjacent parent guide matching the agent that loaded this skill: `codex.md`, `claude.md`, or `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Agree on the run

Use the user's request and task documents to enumerate the series without redefining its tasks.
Resolve only missing decisions:

- Tasks, dependencies, and any shared files or mutable resources.
- Implementor harness and model, including per-task overrides when requested.
- Only for requested reviews: coverage and reviewer harness/model.
- Execution order and which independent tasks may overlap.
- Step and batch boundaries, and any override to the default commit policy.
- Existing tracking, outcome-log, and commit conventions.

Use the harness reference's discovery and model-selection instructions only for implementors and explicitly requested reviewers.
Retain selections and full-access authorization already provided.
Keep successful discovery and preservation of normal configuration silent.
Do not inspect, announce, or ask about memory settings unless the user requested that information or a change, or a specific launch failure requires it.
Leave task-level product decisions to the implementor unless the requested plan makes them a prerequisite to starting the run.

**Reviews are opt-in; the default is OFF.**
If the user did not explicitly request reviews for this run, set `Review: OFF · not requested` and skip reviewer discovery, selection, skill-availability checks, and launches.
Do not ask whether to add reviews or propose a reviewer by default.
An available `co-review` skill, a selected implementor model, or generic review guidance does not authorize reviews.
A review checkpoint explicitly included in the task series the user asked to execute counts as requested; ordinary implementor verification does not.
For requested reviews, support all tasks, selected tasks, or a combined checkpoint, with different reviewers when requested.
Resolve only missing reviewer choices; do not automatically reuse the implementor's model or broaden the requested coverage.

**Default to staging and committing after each completed orchestration step.**
A step normally contains one task's implementation and all its assigned reviews.
When tasks share a required combined review, group them into one step and designate an implementor to finalize it.
A step is ready to commit only when implementation, required reviews, and tracking are complete, every finding is resolved, and no blocker or required user decision remains.
Include this policy in the pre-run confirmation so the user's explicit approval authorizes those commits.
Honor an explicit no-commit instruction or another user-selected policy; do not infer its reversal from a general request to orchestrate.
This policy authorizes local commits only, not pushes.

**Propose parallel execution when independence is plausible, and obtain approval for the concrete schedule.**
Consider shared edits, generated output, test databases, ports, and other mutable resources, not just task numbering.
Serialize conflicting phases or use separate resources already supported by the project.
When independence is uncertain, use sequential execution.

Default to running the entire requested task series and stopping at its end.
Use an intermediate batch boundary only when the user supplied one; do not add a batch-size question by default.
An approved concurrent batch must settle all its active task and review sessions before its confirmation boundary.
Place batch boundaries between complete steps so a required combined review and its commit are not split across a stopping point.

## 2. Select the review route

Skip this section when reviews are OFF.
For each reviewed task, check whether its implementor can invoke an installed, enabled `co-review` skill using the harness reference's discovery and invocation rules.
Check the actual child environment; a skill visible to the orchestrator alone is insufficient.
Do not install, rebuild, or reconfigure skills to make this route available.

When usable, **prefer resuming the original implementor with co-review**.
Show this route in the confirmation's Review field without an explanatory paragraph.
The implementor keeps its task context and owns every fix.

Otherwise use **orchestrator-mediated review**.
Launch a separate reviewer, send it the implementor's final report and the review brief, and relay the exchange yourself.
Both routes use the shared mutual-agreement protocol and the user's selected reviewer.
Honor an explicit route preference.

For a combined review, identify the author session responsible for responding and making any cross-task fixes before starting that checkpoint.
Do not let two author sessions make competing integration fixes.

## 3. Present the plan and get confirmation

Resolve genuinely missing run choices first with a concise plain-text question; do not combine an unresolved choice with approval of an incomplete run.
Confirm before launch.
Use exactly the following eight Markdown list fields, in this order, followed by one short plain-text confirmation question.
This is the required format, not an expandable example.
Keep the entire confirmation within 120 words, excluding link targets.
Do not add a heading, task table, introductory or closing narration, repeated model settings, or unrequested configuration details.

- 🎯 **Scope:** <plan link> · <requested task range>
- 📁 **Workspace:** <absolute path>
- 🤖 **Implementor:** <harness/model> · <effort> · <speed>
- 🔍 **Review:** OFF · not requested
- 🔀 **Schedule / stop:** <dependency order or parallel groups> · <stopping boundary>
- 💾 **Commit:** each completed step · no pushes
- 🔓 **Access:** <full host or requested restrictions>
- ⏳ **Wait / reads:** any child exit · quiet otherwise · plan/session documents + final reports

Approve this run and per-step local commits?

Replace field values with the actual run settings; show requested review coverage, reviewer settings, and route only in the Review field.
Adapt the Commit field and question to an explicit commit-policy override.
Keep complete task names, per-task overrides, grouped-step finalizers, and shared-resource constraints in the orchestration record; link that record from the relevant field when the settings cannot fit.
Use the established tracking location or the harness's locally ignored run directory for that record.
If higher-priority instructions require an approval-source explanation, append only one short sentence and include it in the same word limit.
Reuse approval of this exact plan rather than asking again.

## 4. Execute the approved schedule

Maintain a compact record per task:

- Task scope, dependencies, owning step, file/resource ownership, and selected settings.
- Implementor session ID and, when applicable, reviewer session ID.
- Review route, round, finding ledger or delegated review report, and response paths.
- State: `waiting`, `implementing`, `reviewing`, `fixing`, `finalizing`, `done`, `blocked`, or `failed`.

Before a launch, capture the harness reference's ownership baseline.
For tasks launched together, use the same initial batch boundary and record each task's ownership.
Give each task and role separate run files.

Build a minimal prompt from the user's task or its definition.
Add only decisions needed for this execution: its review assignment, ownership, shared-resource coordination, and commit timing.
Treat applicable base prompts and global/project instruction files as available through the child's normal loading mechanisms.
Do not copy, summarize, repeat, or add reminders to read those instructions unless the user explicitly asks.
Apply the same rule to resumes and nested agents; pass execution-specific decisions the child does not already have.

For concurrent work, include this brief instruction:

> Other tasks are running in this checkout.
> Unrelated changes are expected; leave them intact and do not stage or commit them.
> Stay within this task's ownership and report an overlap so the conflicting phase can be serialized.

Keep shared tracking writes and Git index operations serialized too.
The implementor must wait for its commit turn even if its normal task instructions suggest committing immediately.

Launch ready tasks through the parent's managed asynchronous facility.
Once all currently ready launches are dispatched, remain idle until at least one active managed process exits or the user supplies new input.
Wait for any active handle, not for the entire batch or just the first process launched.
If the facility only supports bounded per-handle waits, cycle blocking waits across the active handles using lifecycle status alone.
A timeout or still-running status means re-enter the wait; it does not justify another progress message or inspection.
Do not poll files, Git changes, tracking documents, response-file existence, or streamed transcripts to infer progress.
Do not narrate unchanged status, elapsed time, observed edits, or the absence of a response.
After an exit, read only that process's final response or bounded failure diagnostic, update the orchestration record, and dispatch any newly ready implementation, review, or finalization work.
Then return to the idle wait while other processes remain active.
User input may interrupt the wait for steering, cancellation, or an explicit status request.

A task with an assigned individual review enters it as soon as its implementor finishes, while independent tasks continue.
A combined review starts only after every included implementation and any prerequisite individual review is complete.
Keep its author from editing the reviewed work during a reviewer round.
A dependent step becomes ready only after its prerequisites have completed their assigned reviews, tracking, and required commits.
For ordered tasks inside a combined-review step, explicitly approve which completed implementations unlock the next internal task; downstream steps still wait for the whole step to finish.
Tasks with no assigned review proceed directly to finalization unless their step still requires a combined review.

## 5. Run the assigned review

Apply this section only to explicitly requested reviews.

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

## 6. Finalize and commit a step

Confirm that every task in the step has complete implementation and required review reports, with no unresolved finding, blocker, or required user decision.
Confirm its state-tracking convention from the permitted tracking records and agent reports.
If tracking is missing or wrong, resume the responsible implementor to correct it; do not silently advance or edit its task content yourself.
Bookkeeping outside the review subject may follow sign-off; changes to reviewed work require re-review.

Write a concise outcome record in the agreed location: what completed, review disposition, response paths, and any remaining verification or user action.
Distinguish verified review conclusions from an unreviewed implementor's own report.

When the approved policy calls for a commit, resume the original implementor, or the designated finalizer for a grouped step, to stage and commit only changes proven to belong to that step.
Use this delegated finalization for sequential and concurrent runs; do not inspect the source diff or perform the content-based staging yourself.
Serialize all staging and commit follow-ups so they cannot race over the shared index.
Have the finalizer preserve pre-existing and other agents' changes and return the commit hash and ownership summary.
Use inherited commit conventions without repeating them in the prompt.
If ownership cannot be established, keep the step blocked instead of guessing or staging broadly.
If there are no step-owned changes, record that outcome without creating an empty commit.
When the approved policy disables commits, skip the staging/commit follow-up and record that policy.
If a commit is required, keep the step in `finalizing` until it succeeds or a no-change outcome is established; a commit failure is a blocker.

Mark the step done only after its required finalization is complete.
Report its outcome briefly and launch newly ready work within the approved batch.

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
