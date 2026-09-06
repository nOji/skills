# Review protocol

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

## Review brief

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

## Evidence and responses

Neither participant treats the other's report as proof.
Read the relevant work, reproduce claimed failures when practical, and distinguish observations from inferences.
Consider skipped requirements, integration behavior, error paths, and unintended scope changes.
Avoid speculative findings or changes justified only by personal taste.

For each finding, the author chooses one response:

| Response | Required action |
| --- | --- |
| Confirmed | Fix it and report the change and verification |
| Rejected | Give evidence explaining why the finding is incorrect |
| Judgment call | State the chosen approach and its trade-off |

Invite pushback in both directions.
Accept the stronger evidence, regardless of which agent supplied it.
A finding's severity does not determine whether it has been resolved.

## Mutual-agreement loop

Use this loop for co-review and every orchestration review.
Keep stable finding IDs and a compact ledger with these states:

- `open`
- `fixed-awaiting-review`
- `rebutted-awaiting-response`
- `closed-both-agreed`

1. The reviewer returns its findings.
2. The author verifies each finding, makes the fixes it accepts, and answers every item.
3. Send the updated ledger and the author's response to the same reviewer session.
   Include all edits since its last review, including changes it did not request.
4. The reviewer inspects the current work, checks the fixes for regressions, and explicitly closes or contests each item by ID.
5. Return any contested or new findings to the same author session and repeat.

Preserve both session IDs throughout the exchange.
When a coordinator relays messages, pass the participants' final reports and evidence faithfully; do not substitute the coordinator's technical verdict.
Compact repeated history into the ledger, but retain unresolved arguments and evidence.

Every edit after a review requires another review of the affected final state.
A rejection remains open until the reviewer explicitly accepts the rebuttal.
Silence, a lower severity, or a general "no blockers" statement does not close an unaddressed item.
Retain mutually closed items with their resolution so they do not disappear from the record.

Completion requires every item to be `closed-both-agreed`, the author's acceptance of the final disposition, and a reviewer verdict covering the final task state.
A clean first review that the author verifies and accepts needs no extra round.
If another task changes a relevant dependency after sign-off, recheck the affected conclusions before treating them as final.

## Supervisor sign-off

For co-implement, the parent reviewer independently checks the task-owned diff and completion claims.
Send substantive defects and contestable choices back to the same implementor session.
Re-review its subsequent changes until no blocking issue remains.

The supervisor may directly fix an obvious, uncontested mistake.
If that clears the last issue, verify and sign off without another delegate round.
If another round is needed, tell the implementor what the supervisor changed.
This exception does not apply to the mutual-agreement loop.

## Protect ownership during review

Use the content baselines and ownership rules in the harness reference.
The author pauses edits to the reviewed task while its reviewer runs.
Independent tasks may continue only where their files and mutable resources do not conflict with that review.

After review, check for unintended changes to the reviewed work.
In a shared checkout, other agents' expected changes are not reviewer edits.
Never restore a repository-wide snapshot over concurrent work.
Undo only a delta proven to belong to the reviewer; if ownership cannot be established, preserve the files and resolve the conflict before continuing.

In a single-writer workflow, stage only reviewed task-owned hunks when that gives the next round a useful diff boundary.
Leave mixed-ownership paths unstaged when separation is uncertain.
During parallel work, keep the shared index unchanged until the coordinator grants a serialized staging or commit turn.

## Decisions and close-out

Continue technical discussion autonomously while the participants are making progress.
Ask the user only for a requirement, scope, or consequential product decision the task does not answer, or a disagreement the participants cannot reconcile with evidence.
Present both positions and the remaining question; preserve the sessions for resumption.
Do not call an unresolved exchange complete.

Report the outcome, accepted fixes, accepted rebuttals, checks actually performed, and any remaining limitation.
For mutual-agreement reviews, include the final ledger and both participants' explicit disposition.
Keep reports concise and reference the final response files rather than streamed transcripts.
