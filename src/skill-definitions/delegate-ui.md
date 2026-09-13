---
name: delegate-ui
description: Implement a complete feature as the parent agent while delegating only UI presentation and visual design to a user-selected child agent, then review its functional integration. Use when the user requests UI-only delegation or invokes delegate-ui.
---

# Implement the feature and delegate its UI

You own delivery of the complete feature and all logic and data that support it.
The child owns UI presentation and visual design within the user's requirements.
Delegate only that UI work, after preparing its prerequisites yourself.

Read the [harness reference](#harness-cli-reference) before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Select the UI worker

Use the harness reference to discover available CLIs and models.
Reuse the user's supplied harness, model, and execution settings.
If the harness or model is missing, ask the user to choose the missing value rather than selecting a default.
Resolve remaining execution choices through the harness reference and retain the selected settings across rounds.
If this skill was delegated, return missing user decisions to the coordinating parent.
You may prepare the feature while awaiting a selection, but do not launch an unselected UI worker.

## 2. Prepare the complete functional foundation

Capture the harness reference's initial ownership baseline before implementation.
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

Use the harness reference's minimal-prompt and worktree instructions.
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
Launch one UI implementor through the harness reference's managed process facility and retain its session ID.
Wait for it to exit before inspecting its work or editing the handed-off regions.

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
On launch or resume failure, follow the harness reference's recovery rules; preserve completed work and the session rather than silently switching models or taking over the UI design.

## 5. Deliver the complete feature

Verify the final integrated feature after the last changes and distinguish completed checks from anything that could not be verified.
Report the parent-owned functionality, delegated UI work, selected UI implementor, final response path, and any remaining limitation.
Do not claim completion while required functionality or UI work remains unfinished.
Follow the user's commit-message convention for the complete task change and commit only when authorized.
