---
name: orchestrate
description: Orchestrate a series of tasks by delegating each one to a coding-agent CLI (Codex, Claude Code, or Cursor Agent), one task at a time, stopping every N tasks for the user's confirmation. Use when the user wants a numbered list of tasks, plan sessions, or steps executed sequentially by another harness while this agent just supervises, logs, and commits. Not for a single supervised implementation or review.
---

# Orchestrating a series of delegated tasks

You are the **orchestrator**, not the implementer and not the reviewer. A different coding-agent CLI does every task; you launch it, wait on its managed process handle, record what it reported, keep any task-tracking file honest, stage and commit its work, and move to the next task — pausing every N tasks for the user to say "continue."

This workflow is deliberately narrow: you do not review the delegate's work, push back on it, or iterate against acceptance criteria. You trust the delegate's own report. If the user wants a supervised, argued-over implementation of one task, use a dedicated single-task supervision workflow instead. This skill is for **running a queue**.

All harness command forms — launch, resume, model listing, background rules, failure signatures — live in the [embedded harness CLI reference](#harness-cli-reference). **Read that section before launching anything**, and take every command from it verbatim rather than from memory. When you are a Codex parent, follow the embedded harness CLI reference §0's elevated host-shell rule for every Codex, Claude Code, or Cursor Agent command.

## 1. Gather the task series and the operating parameters

Before running anything, get four things from the user — ask for whichever aren't already given:

1. **What the tasks are.** A list, a range of numbered items, a set of files (e.g. session files in a plan folder), or a description you can enumerate yourself once. Enumerate the full ordered list back to the user before starting if it was implicit (a range, a folder glob) rather than named explicitly — an orchestrator that silently miscounts the queue is worse than one that asks.
2. **The harness and model.** Follow the embedded harness CLI reference §1–3 and §8 to find installed harnesses, list their models, and present the menu. Skip the menu if the user already named a harness and model. If the chosen harness is **Codex**, ask once whether to add `-c 'features.memories=false'` (see the note in that reference) and reuse that answer for every task in the run.
3. **Batch size — how many tasks to run before stopping for confirmation.** Ask directly: "How many tasks at a time before I stop for your OK?" Do not assume 1, 3, or "all of them." A user asking to run "the rest" or naming an explicit range without a batch size is choosing to run that whole range without stopping — that is a valid answer, not a gap to fill in.
4. **Whether to commit after each task**, and if so, whether there's a repo convention to follow for commit messages (check `CLAUDE.md`/`AGENTS.md` at the repo root — e.g. a required body, a forbidden co-author trailer). Most delegates will suggest a commit message in their final report; confirm whether to use it verbatim or adapt it.

If the task series has its own **state-tracking convention** — a status table, a `READY`/`WIP`/`DONE`-style column, a progress log the delegate is expected to update as part of doing the task — get that convention from the user or from the series' own instructions now, not per-task. You will re-verify it after every single task in section 3.

## 2. Scope discipline

**Do not read source files the tasks touch.** Your job is to launch, wait, and record — not to audit the implementation. Reading into the target codebase defeats the reason this skill delegates in the first place: it burns the exact context budget the delegate's fresh process exists to spare you.

The only files you read directly are:

- The task's own definition (a session file, a numbered list item, a task description) — enough to write the prompt.
- Any state-tracking file named in section 1, to confirm it was updated correctly.
- `<RESP>` files, per the embedded harness CLI reference §5 — never `<LOG>`.

## 3. The per-task loop

For each task in the current batch, in order:

1. **Baseline and launch.** Run the embedded harness CLI reference §4's local-ignore preflight before each task, then capture three separate ownership artifacts under `.agent-runs/`: `git diff --cached --binary HEAD`, `git diff --binary`, and a null-safe manifest containing the path, file type, and SHA-256 content hash of every untracked file. Build the prompt from the task's own file or description — keep it short and point at the file path rather than restating its contents; a well-authored task file (a plan session, a ticket) is written to be self-sufficient for a fresh-context agent. Tell the delegate to read and follow any applicable `AGENTS.md` and `CLAUDE.md` files, because the harnesses do not auto-load the same instruction filenames. Do not add scope, acceptance criteria, or other instructions of your own — the task defines its own done-ness. Launch through the embedded harness CLI reference §4's parent-managed asynchronous facility, writing `<RESP>`/`<LOG>` under `.agent-runs/` at the repo root.
2. **Wait on the managed handle.** Do not do other work, read other files, or start the next task while one is running. Use the parent-specific wait operation from the embedded harness CLI reference §4 until the process exits.
3. **Read only `<RESP>` plus the one-line `<SESSION>` metadata when resuming.** Never touch `<LOG>` — not even on failure; the embedded harness CLI reference §7 has the bounded failure extraction for that case.
4. **If the task signals it is blocked** — waiting on missing information, a decision only the user can make, credentials, access it doesn't have — stop the batch immediately, even mid-batch. Print the delegate's blocking question verbatim to the user, wait for their answer, then **resume the same harness session** (embedded harness CLI reference §6) with that answer. Do not skip ahead to the next task while one is blocked, and do not answer on the user's behalf.
5. **Verify the state-tracking convention**, if one applies. Check the file(s) the delegate was supposed to update. If it correctly reflects the task as complete (and unblocks whatever it was supposed to unblock), proceed. **If it did not update correctly, resume the same session** and ask it to fix the tracking state before moving on — don't edit the tracking file yourself and don't silently continue with a stale state, and don't launch a fresh session to do another agent's bookkeeping.
6. **Log the outcome.** Distill `<RESP>` into a short per-task record: what was implemented (without the commit-message boilerplate, which belongs in the commit itself) and any verification/testing steps the user should run. Write it wherever the user asked in section 1, or propose a sensible default (e.g. a `.dev/`-style log directory) if they didn't say.
7. **Commit**, if section 1 said to. Compare the completed tree with that task's ownership baseline, then stage only paths and hunks proven to belong to the completed task, preserving any pre-existing user work. Commit with the delegate's suggested message (verbatim or adapted per the user's convention from section 1). Never use `git add -A` or `git add .`. The local-ignore preflight in the embedded harness CLI reference §4 must already protect `.agent-runs/`; attempts to add it should be treated as a no-op, not an error.
8. **Report the task and move on.** A short status line is enough per task; save the fuller readout for the batch summary in section 4.

## 4. Batch boundaries

After completing the number of tasks the user set in section 1:

- Print a consolidated summary: one line per task (what it was, DONE/blocked/failed), a pointer to the per-task logs, and any cross-cutting issue you had to intervene on (like a stale tracking state you had to ask a session to fix).
- State what the next task in the queue is.
- **Stop and wait for the user to say "continue"** before launching anything further. Do not pre-launch the next batch's first task speculatively while waiting.

If the user says "continue," repeat section 3 for the next batch of the same size, unless they give a new batch size or say to run the rest of the queue.

## 5. Failures

A task that fails outright (the embedded harness CLI reference §7's dead-run signatures) is not the same as a task that reports being blocked. Materialize the bounded diagnostic into `<RESP>` per §7, report it to the user with the exact failure signature, and stop the batch — don't retry the same launch speculatively, and don't silently skip to the next task in the queue.

## 6. What this skill is not

- Not a reviewer. If the user wants the delegate's work checked against acceptance criteria and pushed back on, use a dedicated supervised implementation or review workflow.
- Not a planner. It runs a task series someone already wrote; it does not decide what the tasks should be.
- Not an editor of the tasks themselves. If a task's own file is wrong or stale, surface that to the user rather than correcting it yourself mid-run, unless the user has explicitly asked you to also maintain that content.
