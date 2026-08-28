---
name: orchestrate
description: Orchestrate a series of tasks by delegating them to child coding-agent CLIs (Codex, Claude Code, or Cursor Agent), as an ordered queue or as user-approved parallel waves, stopping at the user's chosen boundaries. Use when the user wants a numbered list of tasks, plan sessions, or steps executed by child harnesses while this agent supervises, logs, and coordinates commits. Not for a single supervised implementation or review.
---

# Orchestrating a series of delegated tasks

You are the **orchestrator**, not the implementer and not the reviewer.
Child coding-agent CLIs do every task; you launch them, wait on their managed process handles, record what they reported, keep any task-tracking file honest, coordinate task-owned commits, and move through the approved schedule — pausing at the user's chosen boundaries.

This workflow is deliberately narrow: you do not review the delegate's work, push back on it, or iterate against acceptance criteria.
You trust the delegate's own report.
If the user wants a supervised, argued-over implementation of one task, use a dedicated single-task supervision workflow instead.
This skill is for **running a queue**.

Before gathering the queue, identify which agent loaded this skill.
If it is one of the three listed parents, read exactly one adjacent guide:

- Codex reads `codex.md`.
- Claude Code reads `claude.md`.
- Cursor reads `cursor.md`.

If none of those describes the loading agent, do not read an unrelated guide.
Use the shared constraints and the loading agent's native managed asynchronous process facility.
That guide governs the supervising parent agent only.
Commands and flags for a selected child harness remain in the embedded harness CLI reference, even when the child happens to use the same product name as one of the guides.

All child-harness command forms — launch, resume, model listing, review posture, and failure signatures — live in the [embedded harness CLI reference](#harness-cli-reference).
**Read that section before launching anything**, and take every child command from it verbatim rather than from memory.

## 1. Gather the task series and the operating parameters

Before running anything, get four things from the user — ask for whichever aren't already given:

1. **What the tasks are and how they relate.** The user may provide a list, a range of numbered items, a set of files (e.g. session files in a plan folder), or a description you can enumerate yourself once. Make a best effort to understand the requested series from that context and the task or plan files without reading implementation source. Enumerate the full task list back to the user before starting if it was implicit (a range, a folder glob) rather than named explicitly — an orchestrator that silently miscounts the queue is worse than one that asks. Identify explicit dependencies, shared ownership, and ordering constraints. If it is obvious that some tasks are independent and safe to run concurrently, propose the concrete schedule and ask the user to approve parallel execution before launching it. For example: "I can run Sessions 1–9 in parallel, wait for all of them to finish, then run Sessions 10 and 11 sequentially because they depend on that work. Would you like me to proceed that way?" If parallel safety is unclear, the user does not approve it, or the user explicitly requested sequential execution, keep the series sequential.
2. **The harness and model.** Follow the embedded harness CLI reference §1–3 and §8 to find installed harnesses, list their models, and present the menu. Skip the menu if the user already named a harness and model. If the chosen harness is **Codex**, ask once whether to add `-c 'features.memories=false'` (see the note in that reference) and reuse that answer for every task in the run.
3. **Batch size — how many tasks to run before stopping for confirmation.** Ask directly: "How many tasks at a time before I stop for your OK?" Do not assume 1, 3, or "all of them." A user asking to run "the rest" or naming an explicit range without a batch size is choosing to run that whole range without stopping — that is a valid answer, not a gap to fill in. For an approved parallel schedule, treat each parallel wave as indivisible and confirm the next boundary after the whole wave settles.
4. **Whether to commit after each task**, and if so, whether there's a repo convention to follow for commit messages (check `CLAUDE.md`/`AGENTS.md` at the repo root — e.g. a required body, a forbidden co-author trailer). Most delegates will suggest a commit message in their final report; confirm whether to use it verbatim or adapt it.

If the task series has its own **state-tracking convention** — a status table, a `READY`/`WIP`/`DONE`-style column, a progress log the delegate is expected to update as part of doing the task — get that convention from the user or from the series' own instructions now, not per-task. You will re-verify it after every single task in section 3.

## 2. Scope discipline

**Do not read source files the tasks touch.** Your job is to launch, wait, and record — not to audit the implementation. Reading into the target codebase defeats the reason this skill delegates in the first place: it burns the exact context budget the delegate's fresh process exists to spare you.

The only files you read directly are:

- The task's own definition (a session file, a numbered list item, a task description) — enough to write the prompt.
- Any state-tracking file named in section 1, to confirm it was updated correctly.
- `<RESP>` files, per the embedded harness CLI reference §5 — never `<LOG>`.

## 3. Sequential execution

For each task in the current batch, in order:

1. **Baseline and launch.** Run the embedded harness CLI reference §4's local-ignore preflight before each task, then capture three separate ownership artifacts under `.agent-runs/`: `git diff --cached --binary HEAD`, `git diff --binary`, and a null-safe manifest containing the path, file type, and SHA-256 content hash of every untracked file. Build the prompt from the task's own file or description — keep it short and point at the file path rather than restating its contents; a well-authored task file (a plan session, a ticket) is written to be self-sufficient for a fresh-context agent. Tell the delegate to read and follow any applicable `AGENTS.md` and `CLAUDE.md` files, because the harnesses do not auto-load the same instruction filenames. Do not add scope, acceptance criteria, or other instructions of your own — the task defines its own done-ness. Launch through the embedded harness CLI reference §4's parent-managed asynchronous facility, writing `<RESP>`/`<LOG>` under `.agent-runs/` at the repo root.
2. **Wait on the managed handle.** Do not do other work, read other files, or start the next task while one is running. Use the parent-specific wait operation from the adjacent guide you loaded until the process exits.
3. **Read only `<RESP>` plus the one-line `<SESSION>` metadata when resuming.** Never touch `<LOG>` — not even on failure; the embedded harness CLI reference §7 has the bounded failure extraction for that case.
4. **If the task signals it is blocked** — waiting on missing information, a decision only the user can make, credentials, access it doesn't have — stop the batch immediately, even mid-batch. Print the delegate's blocking question verbatim to the user, wait for their answer, then **resume the same harness session** (embedded harness CLI reference §6) with that answer. Do not skip ahead to the next task while one is blocked, and do not answer on the user's behalf.
5. **Verify the state-tracking convention**, if one applies. Check the file(s) the delegate was supposed to update. If it correctly reflects the task as complete (and unblocks whatever it was supposed to unblock), proceed. **If it did not update correctly, resume the same session** and ask it to fix the tracking state before moving on — don't edit the tracking file yourself and don't silently continue with a stale state, and don't launch a fresh session to do another agent's bookkeeping.
6. **Log the outcome.** Distill `<RESP>` into a short per-task record: what was implemented (without the commit-message boilerplate, which belongs in the commit itself) and any verification/testing steps the user should run. Write it wherever the user asked in section 1, or propose a sensible default (e.g. a `.dev/`-style log directory) if they didn't say.
7. **Commit**, if section 1 said to. Compare the completed tree with that task's ownership baseline, then stage only paths and hunks proven to belong to the completed task, preserving any pre-existing user work. Commit with the delegate's suggested message (verbatim or adapted per the user's convention from section 1). Never use `git add -A` or `git add .`. The local-ignore preflight in the embedded harness CLI reference §4 must already protect `.agent-runs/`; attempts to add it should be treated as a no-op, not an error.
8. **Report the task and move on.** A short status line is enough per task; save the fuller readout for the batch summary in section 5.

## 4. User-approved parallel execution

Use parallel execution only for the exact independent wave the user approved.
Before launching the wave, capture the ownership artifacts from section 3 for every task against the same pre-wave tree state.
Launch one implementor session per task through the parent-managed asynchronous facility.
In every parallel implementor prompt, say that other agents are working concurrently in the same repository or folder, so unrelated changes may appear while it works; those changes are expected, must not be treated as corruption or reverted, and must not be staged or committed.

Listen for all running task handles rather than waiting for only one predetermined task.
As each implementor finishes, read its `<RESP>`, record its result, and keep listening until every already-launched task has finished, failed, or reported a blocker.
Do not start a dependent wave until every prerequisite task has completed successfully and any required task-owned commit exists.

If commits are enabled for the run, ask each finished implementor — by resuming that same harness session — to commit **only its own changes** using its suggested commit message, adapted only for the repository convention agreed in section 1.
Serialize these commit follow-ups so parallel sessions never race over the shared Git index.
The resumed implementor must compare against its pre-wave ownership baseline, exclude pre-existing and concurrent-agent changes, avoid `git add -A` and `git add .`, and report the resulting commit hash.
The orchestrator must not take over a parallel task's commit merely because its implementor has already returned once.

If one parallel task is blocked or fails, stop launching new work but continue listening for and recording every task that is already running.
Present the blocker or failure after the active wave settles, and do not launch dependent work.

## 5. Batch boundaries

After completing the number of sequential tasks the user set in section 1, or after an approved parallel wave settles:

- Print a consolidated summary: one line per task (what it was, DONE/blocked/failed), a pointer to the per-task logs, and any cross-cutting issue you had to intervene on (like a stale tracking state you had to ask a session to fix).
- State what the next task in the queue is.
- **Stop and wait for the user to say "continue"** before launching anything further. Do not pre-launch the next batch's first task speculatively while waiting.

If the user says "continue," follow the approved schedule for the next sequential batch or parallel wave, unless they change the schedule, batch size, or remaining scope.

## 6. Failures

A task that fails outright (the embedded harness CLI reference §7's dead-run signatures) is not the same as a task that reports being blocked. Materialize the bounded diagnostic into `<RESP>` per §7, report it to the user with the exact failure signature, and stop the batch — don't retry the same launch speculatively, and don't silently skip to the next task in the queue.

## 7. What this skill is not

- Not a reviewer. If the user wants the delegate's work checked against acceptance criteria and pushed back on, use a dedicated supervised implementation or review workflow.
- Not a planner. It runs a task series someone already wrote; it does not decide what the tasks should be. Inferring and proposing an execution schedule from obvious dependencies is orchestration, not permission to redefine the tasks.
- Not an editor of the tasks themselves. If a task's own file is wrong or stale, surface that to the user rather than correcting it yourself mid-run, unless the user has explicitly asked you to also maintain that content.
