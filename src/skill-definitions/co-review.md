---
name: co-review
description: Obtain an independent review of a plan, diff, or task output from Codex, Claude Code, or Cursor Agent, then resolve findings through evidence, fixes, and rebuttals until both participants agree.
---

# Resolve an independent review

You are the author and the reviewer's counterpart.
The child reviewer reports findings; you own fixes.
Use the **mutual-agreement loop** in the [review protocol](#review-protocol).

Read the [harness reference](#harness-cli-reference) before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Configure the reviewer

Before studying the review subject, discover available harnesses and models using the harness reference.
Reuse choices and authorization already supplied by the user or an orchestrator acting on the user's approved plan.
Do not repeat a model menu, permission question, or review-scope question that has already been answered.

The preferred recommendation is Codex GPT-5.6 Sol, `high`, normal; use the equivalent through Cursor if Codex is unavailable.
Recommend it only when the current catalog supports that combination.
Consider a different model family from the author when recommending an independent reviewer.
Otherwise present the available choices without inventing a replacement default.

Keep the chosen reviewer settings and full host access across all rounds.
If this skill was delegated, return genuinely missing user decisions to the coordinator rather than waiting for interactive input inside a headless run.

## 2. Brief and launch

If the user names a subject, use it.
Otherwise review the current conversation's work, including relevant committed and uncommitted changes.
Reconstruct the goal and file list from existing context and cheap Git metadata; do not pre-read the code to seed the review.

Build the brief from the shared review protocol.
Include the original implementor's completion report when this review follows delegated implementation.
Use the harness reference's minimal-prompt rule; do not repeat default instructions or ask for routine output the reviewer already knows to provide.

Capture a pre-review content baseline and the task's ownership boundary.
In parallel work, retain the coordinator's ownership and shared-resource constraints.
Launch the reviewer with normal evidence-gathering capabilities and the protocol's no-repair instructions.
Do not use plan mode.

## 3. Exchange evidence and fixes

Wait for the managed process to exit, then read its final response.
Check that it did not alter the reviewed work, accounting for expected concurrent changes.
Now inspect the relevant files, verify each finding, and respond as confirmed, rejected, or a judgment call.

Maintain the shared ledger.
Make accepted fixes, log all changes, and send the ledger, fixes, rebuttals, and judgment calls back to the exact same reviewer session.
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
On process failure, use the harness reference's recovery rules and preserve the sessions and completed work.
