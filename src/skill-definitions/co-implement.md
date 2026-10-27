---
name: co-implement
description: Delegate one implementation to Codex or Claude Code, then independently review and iterate on its work. Use when the user requests a supervised implementation by another agent or invokes co-implement.
---

# Supervise one implementation

You supervise and review; the child agent implements.
Follow the **supervisor sign-off** mode in the [review protocol](#review-protocol).

Before running a child, choose its runtime in the [child runtime](#child-runtime) section and follow the [delegation rules](#delegation-rules).

## 1. Configure the delegate

Before studying implementation files, use the selected runtime to discover available harnesses and models.
Ask only for choices the user has not already supplied.
Require the user's explicit selections under the delegation rules before launching the implementor.

Keep the approved harness, model, effort, speed, and full-access launch settings for every round.

## 2. Delegate before studying

Pass the user's request through without narrowing it or replacing it with your own plan.
Point to task documents the user named.
You may navigate plan documents just far enough to identify the next requested step, but do not study the implementation before delegation.

Follow the delegation rules' minimal-prompt rule.
Add only task-specific information the delegate lacks and the changed-file or incomplete-work reporting needed to supervise this task.
Do not add reminders about automatically loaded instructions, context files, or routine commit-message conventions.

Capture the initial ownership baseline under the delegation rules.
Launch one implementor through the selected runtime with the authorized full-access settings.

## 3. Wait, then review

Announce the harness, model, and task briefly.
Wait on the runtime's handle until the run finishes.
Do not inspect source changes or start reviewing while the implementor is active.

After exit, read the final response and inspect every task-owned change against the initial baseline and previous round boundary.
Read relevant dependencies and independently check the implementor's claims against the task.
Use the review protocol's evidence rules and checks appropriate to the work.

Apply only obvious, uncontested fixes yourself.
Send substantive defects and disputed choices back to the same implementor session, with concrete evidence and an invitation to disagree.
Record any fixes you made so the implementor does not unknowingly undo them.

## 4. Iterate and sign off

Use the review protocol's ownership and staging rules before the next round.
Continue the exact recorded implementor session with a distinct round record and the same launch settings.
Review each round's new changes and any affected earlier conclusions.
Continue until no blocking issue remains.

If a launch or continuation fails, follow the delegation rules' failure and recovery rules.
Do not silently change models, switch harnesses, or take over the implementation.

Report the implementor, what changed, your own verification, any remaining limitation, and the final response reference.
Mention staged changes only when you actually staged them.
Follow the user's commit-message convention for the complete task change.
Commit only when authorized.
