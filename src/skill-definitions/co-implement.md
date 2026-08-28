---
name: co-implement
description: Delegate a task to a child coding-agent CLI — Codex, Claude Code, or Cursor Agent — and supervise it as a reviewer, watching only for failure while it runs. Use when the user asks to implement something with another agent, or invokes /co-implement.
---

# Supervising a delegated implementation

You are the **supervisor and reviewer**, not the implementer.
The delegate writes the code; you decide whether it is correct, push back until it is, and sign off at the end.
The one thing you implement yourself is the fix too trivial to be worth a round trip — see section 7.

The delegate is whichever coding-agent CLI the user picks in section 1 — **Codex** (`codex`), **Claude Code** (`claude`), or **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
Every command form for all three lives in the [embedded harness CLI reference](#harness-cli-reference).
**Read that section during section 1**, and take the commands from it verbatim rather than from memory.

Before preflight, identify which agent loaded this skill.
If it is one of the three listed parents, read exactly one adjacent guide:

- Codex reads `codex.md`.
- Claude Code reads `claude.md`.
- Cursor reads `cursor.md`.

If none of those describes the loading agent, do not read an unrelated guide.
Use the shared constraints and the loading agent's native managed asynchronous process facility.
That guide governs the supervising parent agent only.
Commands and flags for a selected child harness remain in the embedded harness CLI reference, even when the child happens to use the same product name as one of the guides.

The loop is always: **preflight, delegate, wait for the exit, review, iterate, sign off.**

## 1. Preflight: find the harnesses, then agree on one

This runs **before you read anything at all** — not the task, not the plan, not a single source file.
It is four steps and it ends with a model chosen.

### Step 1 — what this machine has

The very first tool call:

```bash
which codex; which claude; which cursor-agent || which agent
```

A harness with no path is not installed and plays no further part.

**Say none of this out loud.** Which binaries exist, which ones failed availability checks and why, what you are about to run next — none of it is news to the user, and all of it is the plumbing they asked you to handle. Run steps 1 and 2 silently; the first thing the user sees from this section is the menu in step 3.

### Step 2 — ask each survivor for its models

Run the listing command for every remaining candidate — the three forms are in the [embedded harness CLI reference](#harness-cli-reference) §2.
This is also the availability check, and it is where a broken harness reveals itself: **not authenticated**, a login prompt, expired credentials, a quota or usage-limit error, a non-zero exit, or empty output all mean that harness is out of service.

Drop it from the candidates, but **keep its exact message** — you report it in step 3. A harness the user believes they have access to, silently missing from the menu, is a worse outcome than a slow round.

One verified asymmetry you must not paper over: **Codex's model listing succeeds even when its account is out of usage quota.** A clean listing proves Codex can be _asked_ about models, not that it can _run_ one. Codex's exhaustion appears only on the first real launch, and section 5 handles it there.

**If no candidate survives this step**, stop and follow section 5 — do not read the task, and do not start implementing it yourself.

### Step 3 — put the choice to the user

Skip this step entirely when the user already named a harness and model in their request; that is their choice and you use it.

Otherwise, present the menu **immediately, before reading anything else**, in the format the [embedded harness CLI reference](#harness-cli-reference) §8 lays out — display names only, one line per harness, and the three things they must choose set out so none of them can be missed:

```
Pick a harness and model:

🤖 **Codex** — GPT-5.6 Sol · GPT-5.6 Terra · GPT-5.6 Luna · GPT-5.5 · GPT-5.4
🖱️ **Cursor** — Cursor Grok 4.6 · Composer 2.5 · Claude Opus 5 / Sonnet 5 / Fable 5 · GPT-5.6 Sol / Luna · Kimi K3 · GLM 5.2

⛔ **Claude Code** — unavailable: out of usage limits until Aug 20

Then tell me:

  1️⃣  **Model** — any name above
  2️⃣  **Reasoning effort** — `low` · `medium` · `high` · `xhigh` · `max`
  3️⃣  **Speed** — ⚡ `fast` or 🐢 `normal`

⭐ **Recommended:** Codex · GPT-5.6 Luna · `xhigh` · ⚡ fast — quick, strong, and very cheap.
```

Fill it in per §8's rules: one ⛔ line for each harness that dropped out with the actual reason in a few words, Cursor shortlisted to the prominent families, and no effort or speed knob offered that the models in hand do not have.

**The recommended default is Codex `gpt-5.6-luna` at `xhigh` reasoning, in fast mode** — fast, performant, and extremely cheap. If Codex is not a candidate, recommend the equivalent through Cursor: GPT-5.6 Luna, extra-high, fast. If neither is in the listings you actually got, **offer no recommended default** — present what is there and let the user choose; never silently substitute a neighbouring model.

The user answers conversationally — "luna, fast", "sol on high", "grok but cheap". You hold the full listings, so you resolve that into the exact model string and flags. Answer follow-up questions about what else is available from those listings; do not make the user read a slug.

### Step 4 — lock it in

Translate the choice into concrete flags using the [embedded harness CLI reference](#harness-cli-reference) §3, and use them verbatim in **every** round of this task.
Do not change harness, model, or effort mid-task on your own.

## 2. Delegate first, do not study first

Hand the task to the delegate **before** you understand it in depth.
Do not read the source files or the surrounding modules before launching the first run.
The delegate has its own context window and its own file access; making it re-derive what you already read wastes nothing, while your pre-reading burns your context on material you will read again during review anyway.

**Plan documents are the exception to what you may read.** When the task is "continue the plan" or otherwise does not name a specific session or step, you may read the plan's own documents (its `plan.md`, its index, its session files' headers) far enough to identify which session or step is next, and tell the delegate exactly that. That is navigation, not study — it stops the moment you know which document to point at. It does not license reading the code that session touches.

Pass the user's request through **as-is**. Add only the instruction to report, at the end, the exact list of changed files and anything deliberately not done.

Tell the delegate to read and follow any applicable `AGENTS.md` and `CLAUDE.md` files before acting, because the harnesses do not auto-load the same instruction filenames.
Do not restate those files' conventions, rules, or commands in the prompt; repeating them wastes space and risks passing a stale version.

Do not paraphrase the task into your own plan, and do not narrow it.
If the request names a document to follow (a plan, a spec, a session file), tell the delegate to read that document in full rather than summarizing it yourself.

The one exception: run a single cheap command if you genuinely cannot construct the prompt without it (for example, listing a directory to learn which session is next).
One command, not an investigation.

Before the first delegate launch, capture a content baseline of the working tree without opening the code as three separate artifacts: `git diff --cached --binary HEAD` for the index, `git diff --binary` for unstaged tracked changes, and a null-safe manifest containing the path, file type, and SHA-256 content hash of every untracked file.
Keep that baseline under the locally ignored `.agent-runs/` directory after running the embedded harness CLI reference §4's ignore preflight.
The tree does not need to be clean, but pre-existing user changes must never be attributed to the delegate, staged, reverted, or committed as part of this task.

## 3. Launch it as a managed asynchronous task

Use the parent-specific long-running process facility from the adjacent guide you loaded for every launch and resume.
Retain the returned task or session handle and keep the shell command itself attached inside that managed session; do not add shell `&` or invent a detached process.

Take the launch command for the chosen harness from the [embedded harness CLI reference](#harness-cli-reference) §4.
All three share the same shape — prompt on stdin as a quoted heredoc, transcript to `<LOG>`, final message to `<RESP>`, exact id to `<SESSION>`:

```
<LOG>  = .agent-runs/co-implement-<slug>-r1.log
<RESP> = .agent-runs/co-implement-<slug>-r1.response.log
```

Before creating `.agent-runs/`, run the local-ignore preflight in the [embedded harness CLI reference](#harness-cli-reference) §4. After it passes, the run files stay out of commits and `git status`. Never write these files to `/tmp`.

For Codex, `-o` writes `<RESP>` itself.
For Claude Code and Cursor Agent there is no such flag, so **once the process has exited** use the file-to-file materialization command in the [embedded harness CLI reference](#harness-cli-reference) §4. It writes the compact session metadata and final report into `<RESP>` without exposing `<LOG>` to the parent context.

### `<RESP>` is the only file you ever read

- **`<RESP>` — compact session metadata plus the final report. This is the only transcript-bearing agent-run file you read or print.**
- **`<SESSION>` — one exact session id. Read it only when resuming.**
- **`<LOG>` — the streamed transcript. You never read or print this file.** Not with Read, `cat`, `head`, `tail`, or a `grep` that writes to the tool result; not whole, not in part, not while it runs, not after it exits, not out of curiosity, and not for failure diagnosis.

The `<LOG>` holds the delegate's entire intermediate reasoning — every tool call, every file it opened, every thought it discarded — routinely tens of thousands of tokens. Reading it is the single most expensive mistake available in this skill, and it buys nothing: a run that finished has already told you everything it concluded, in `<RESP>`.

The shell may mechanically transform `<LOG>` into `<RESP>` after exit or extract only the exact session id into `<SESSION>`, with no output entering the parent context. Those are the only permitted contacts with the transcript. The parent reads `<RESP>` and reads `<SESSION>` only to resume.

Non-negotiable details:

- **Do not write the prompt to a file of its own.** The heredoc on stdin is the prompt; a separate file only adds a step and a leftover artifact.
- **Every round writes its own pair of files** (`-r1.*`, `-r2.*`, …). Never reuse or append to a previous round's files; you need each round's final report intact to compare what changed.
- **The harness, model, effort, and mode come from the section 1 choice**, not from a template you half-remember.
- The delegate needs write access to do the work: `--sandbox workspace-write` for Codex, `--permission-mode bypassPermissions` for Claude Code, `-f` for Cursor Agent.
- **Never launch in plan mode, on any harness.** Plan mode changes what the agent is _for_: instead of implementing, it drafts a proposal for approval and writes nothing. Verified on both CLIs that offer it — Cursor answered a work request with "I'll outline that plan for your approval", and Claude wrote a plan file to `~/.claude/plans/` rather than doing the task. Normal mode, always.

## 4. While it runs: wait on the managed handle

Say what you launched — harness, model, one sentence — then wait on the saved task or session handle until the process exits.
Use the parent's wait operation rather than polling files, sleeping in the shell, or doing unrelated work.
If a bounded wait returns while the process is still active, wait again on the same handle and keep any user-facing status update brief.

While a run is in flight, do **not**:

- touch the `<LOG>` in any way,
- open the `<RESP>` (it is not written until the process exits),
- run `git status`, `git diff`, or read source files "to prepare" for the review,
- start reviewing, or narrate results that have not arrived.

If the user asks for a status update, tell them it is still running and that you will report when it exits — that is the whole answer, and it needs no file access. If they explicitly ask you to check whether the run is alive, check the **process**, not the transcript:

```bash
pgrep -fl '<harness> .*<slug>' || echo "not running"
```

The process check answers the question.
Follow the adjacent parent guide for any parent-specific execution requirement.
**Reading the transcript never becomes acceptable, not even on request.**

If the user reports the run appears wedged and wants it stopped, kill it with the matching `pkill` from the [embedded harness CLI reference](#harness-cli-reference) §7 and then follow section 5.

### If the exit itself signals failure

The managed process result carries the exit status. When it is non-zero, or `<RESP>` is empty or missing, the run died rather than finished — crash or panic, model at capacity, rate limit, quota exhaustion, expired authentication, or a parent tool that terminated the process.

Diagnose it only by using the [embedded harness CLI reference](#harness-cli-reference) §7's bounded file-to-file extractor, which writes the diagnostic into `<RESP>` without printing transcript content. Then read `<RESP>`. If no structured diagnostic can be materialized, report that fact; **never tail or otherwise inspect `<LOG>`**.

Then follow section 5 — say what failed, quote the evidence, and ask before doing anything else.

If a synchronous parent-tool call terminated the run, relaunch the identical command once through §4's managed asynchronous facility.

## 5. If delegation is impossible, offer the other harness before offering yourself

Delegation can fail before any work happens: no harness is installed, the account is out of usage/quota, authentication is missing or expired, the model is refused, or the process exits immediately with an error instead of running.

When that happens:

1. **Abort the loop.** Do not retry the same command in a loop, and do not silently fall back to implementing it yourself.
2. Tell the user plainly what blocked it, quoting the actual error.
3. **If another candidate harness is still standing, offer it first.** This is the common case for a Codex quota block, whose listing looked healthy in section 1 and only failed at launch. Switching harness preserves the whole point of the skill; doing the work yourself does not. Say which harness and model you would switch to, and wait for the user's go-ahead.
4. **Only when no harness remains**, ask whether they want you to do the task yourself, unsupervised and undelegated — and **wait for an explicit yes**. Implementing it yourself is a different mode of work than they asked for, so it needs their consent, not your inference.

A switch of harness restarts the task at round 1: a new harness has none of the previous session's context, so brief it from scratch rather than handing it a critique.

One exception to "do not retry": a transient "model is at capacity" response is worth a single retry after a short wait. A second capacity failure is a block — report it and offer the alternatives above.

If the block appears mid-task (a resume round fails after earlier rounds succeeded), the same rules apply: report where the work stands, what is already on disk, and ask before continuing by hand.

## 6. Review only after the process exits

When the managed process reports completion, then and only then:

1. Read the round's **`<RESP>`** — it contains compact session metadata and the delegate's final report (its claims, its file list, what it says it skipped). **This is the only transcript-bearing agent-run file you read. Never read `<LOG>`, including on failure.**
2. Compare the current content with the initial user-work baseline and the previous round boundary, then read the **actual task-owned diff** of every changed file. `git status --short` is only a path summary; it is not a content baseline. When the previous round was safely staged, plain `git diff` is the new delta. For mixed-ownership paths left unstaged, compare against the saved round boundary instead.
3. Read the files the change depends on but did not touch, so you can judge whether the change fits the system.
4. **Verify the claims independently.** If the delegate says the type-checker and linter pass, run them yourself. A green claim you did not reproduce is not evidence.

Review against the task's own acceptance criteria, not your taste.
Look specifically for:

- requirements in the task that the diff silently skipped,
- behavior that contradicts a decision the task recorded,
- scope creep: unrelated files or unrelated lines rewritten,
- claims in the completion summary that the diff does not support,
- error paths, not just happy paths: what the caller actually receives when something fails,
- changes that type-check but are still wrong, especially where a type checker structurally cannot catch the mistake.

### Fix the trivial things yourself instead of sending a round for them

A resume round costs a launch, a wait, and a full re-review. That price is worth paying for a real defect and absurd for a typo.

**When a fix is small and obvious enough that there is no plausible disagreement, just make it yourself.** Typos, a wrong import path, a stale comment, a missing `await` in an unambiguous spot, a rename applied in four of five places, formatting a file the delegate forgot to format — apply the edit and move on.

The test is _disagreement_, not size: would a competent implementer, shown this, do anything other than say "yes, correct"? If the answer is no, do it yourself. If you can imagine the delegate pushing back with a reason you would have to weigh — a design choice, a trade-off, anything where its reasoning might beat yours — that is a critique, not an edit, and it goes back to the session.

Two consequences:

- When your own fixes clear the last of it, **there is no next round.** Fix, verify, sign off.
- When you do send a round, **tell the delegate what you already changed and why**, in one line per fix. Otherwise it reviews a working tree that no longer matches its own last report, and may undo you.

Do not use this to rewrite the delegate's work under the label of "small fixes" — the moment you are making a judgment call, you have left this rule.

### Stage only the reviewed, task-owned diff before the next round

Once you have finished reviewing a round and are about to send a critique back, stage only paths or hunks that wholly belong to this delegated task:

```bash
git add -- <task-owned-path>...
```

Never use `git add -A`, `git add .`, or another broad pathspec here.
If a file contains both pre-existing user work and task-owned hunks, stage only the proven task hunks with an interactive or patch-based path-limited operation; if that cannot be done safely, leave the file unstaged and save a round boundary under `.agent-runs/` for the next comparison.

This is a review tool, not a commit. Safe staging draws a line under work you have already read so the next round's `git diff` usually shows only the delegate's response instead of the whole accumulated change again.

Stage only after the review of that round is complete — staging first would erase the boundary you are trying to create. Never stage anything present in the initial user-work baseline, never `git commit` unless the user asks, and mention any staged paths in the sign-off so the user is not surprised.

## 7. Iterate on the same session, and push back hard

**Every rebuttal, critique, or follow-up goes back to the same session.** Never start a fresh run for a later round — that throws away the context that makes iteration cheaper than a restart. The session id is constant for the whole task; only the file names change.

Capture the session id once during round 1 using the per-harness form in the [embedded harness CLI reference](#harness-cli-reference) §6. Confirm the task-local `<SESSION>` file is non-empty, then read that exact file for every later round; never rediscover the session from `<LOG>`, modification time, or `--last`.

Then resume with that section's command for your harness, writing **this round's** files (`-r2.response.log` and `-r2.log`), never a previous round's, with the prompt as a heredoc on stdin. Watch the two harness-specific traps it documents: Codex's `resume` rejects `--sandbox` and needs every flag before the session id, and Claude Code must never be given `--no-session-persistence`.

Then wait on that round's managed process handle until it exits.
The same rules from section 4 and the adjacent parent guide apply to every round without exception, especially the rule that the parent reads only `<RESP>` plus the one-line `<SESSION>` metadata needed to resume.

How to write the critique:

- Number the issues and make each one concrete: what is wrong, where, and why it matters.
- **Explicitly invite pushback**, in the prompt itself: you want the correct outcome, not agreement. A capable implementer that caves to a wrong review produces worse code than one that argues.
- Demand evidence over assertion. When the delegate claims a framework behaves some way, make it demonstrate the behavior rather than assert it.
- Separate blocking defects from judgment calls, and say which is which. Let it keep a defensible choice you merely dislike.
- When it pushes back and it is right, accept it and move on without ceremony.

Keep iterating until no blocking issue remains.
**Re-review every round's diff from scratch**: a fix can introduce a fresh regression, and a regression in round N+1 is not covered by the verification you did in round N.

## 8. Sign off

Leave the files in the locally ignored `.agent-runs/` directory. Give the user the relevant `<RESP>` path in the sign-off; do not open, summarize, or link `<LOG>` from the parent response.

Report to the user:

- **which harness and model did the work**, in one line, including any mid-task switch and why,
- what landed, in a few lines, including any trivial fixes you applied yourself,
- which checks you ran **yourself** and their results,
- what remains unverified and who has to do it (anything needing a browser, a live service, or credentials is the user's to run, and you must list the exact scenarios),
- a suggested commit message covering the whole change and its goal, not just the last round.

Note that the change is staged (from the per-round staging step) and say so. Never **commit** unless the user asks.
If the work is iterating, update the previous commit message rather than inventing a second one.
