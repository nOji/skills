---
name: co-implement
description: Delegate one implementation to Codex, Claude Code, or Cursor Agent, then independently review and iterate on its work. Use when the user requests a supervised implementation by another agent or invokes co-implement.
---

# Supervise one implementation

You supervise and review; the child agent implements.
Follow the **supervisor sign-off** mode in the [review protocol](#review-protocol).

Read the [harness reference](#harness-cli-reference) before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Configure the delegate

Before studying implementation files, use the harness reference to discover available CLIs and models.
Ask only for choices the user has not already supplied.
The preferred recommendation is Codex GPT-5.6 Luna, `xhigh`, fast; use the equivalent through Cursor if Codex is unavailable.
Recommend it only when the current catalog supports that combination.
Otherwise present the available choices without substituting a different default.

Keep the approved harness, model, effort, speed, and full-access launch settings for every round.

## 2. Delegate before studying

Pass the user's request through without narrowing it or replacing it with your own plan.
Point to task documents the user named.
You may navigate plan documents just far enough to identify the next requested step, but do not study the implementation before delegation.

Follow the harness reference's minimal-prompt rule.
Add only task-specific information the delegate lacks and the changed-file or incomplete-work reporting needed to supervise this task.
Do not add reminders about automatically loaded instructions, context files, or routine commit-message conventions.

Capture the initial ownership baseline under the locally ignored `.agent-runs/` directory.
Launch one implementor using the harness reference's managed process and full-access command.

## 3. Wait, then review

Announce the harness, model, and task briefly.
Wait on the managed handle until it exits.
Do not inspect source changes or start reviewing while the implementor is active.

After exit, read the final response and inspect every task-owned change against the initial baseline and previous round boundary.
Read relevant dependencies and independently check the implementor's claims against the task.
Use the review protocol's evidence rules and checks appropriate to the work.

Apply only obvious, uncontested fixes yourself.
Send substantive defects and disputed choices back to the same implementor session, with concrete evidence and an invitation to disagree.
Record any fixes you made so the implementor does not unknowingly undo them.

## 4. Iterate and sign off

Use the review protocol's ownership and staging rules before the next round.
Resume the exact saved session with a new pair of round files and the same launch settings.
Review each round's new changes and any affected earlier conclusions.
Continue until no blocking issue remains.

If a launch or resume fails, follow the harness reference's recovery rules.
Do not silently change models, switch harnesses, or take over the implementation.

Report the implementor, what changed, your own verification, any remaining limitation, and the final response path.
Mention staged changes only when you actually staged them.
Follow the user's commit-message convention for the complete task change.
Commit only when authorized.
