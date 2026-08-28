---
name: co-review
description: Ask a child coding-agent CLI — Codex, Claude Code, or Cursor Agent — to review work (a plan, a diff, or the current session's output), then implement the findings you agree with and argue the ones you do not. Use when the user asks for a second-opinion review from another agent, or invokes /co-review.
---

# Getting a second-opinion review from a child agent

You are the **implementer and the reviewer's counterpart**, not its assistant.
The reviewer reviews and reports; it changes nothing.
You hold the edits, so every fix in the working tree is yours to make — and yours to defend.

This is a **collaboration between two skeptics**, not a review you take dictation from.
The reviewer's findings are claims until you verify them, and your rebuttals are claims until it accepts them.
Neither side treats the other's output as fact.

The reviewer is whichever coding-agent CLI the user picks in section 1 — **Codex** (`codex`), **Claude Code** (`claude`), or **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
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

The loop is always: **preflight, brief, launch and wait, read the review, remediate or argue, send it back, close out.**

## 1. Preflight: find the harnesses, then agree on one

This runs **before you read anything at all** — not the code, not the plan, not the diff.
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

**If no candidate survives this step**, stop and follow section 5 — do not start reading the code, and do not begin reviewing it yourself.

### Step 3 — put the choice to the user

Skip this step entirely when the user already named a harness and model in their request; that is their choice and you use it.

Otherwise, present the menu **immediately, before reading anything else**, in the format the [embedded harness CLI reference](#harness-cli-reference) §8 lays out — display names only, one line per harness, and the three things they must choose set out so none of them can be missed:

```
Pick a harness and model for the review:

🤖 **Codex** — GPT-5.6 Sol · GPT-5.6 Terra · GPT-5.6 Luna · GPT-5.5 · GPT-5.4
🖱️ **Cursor** — Cursor Grok 4.6 · Composer 2.5 · Claude Opus 5 / Sonnet 5 / Fable 5 · GPT-5.6 Sol / Luna · Kimi K3 · GLM 5.2

⛔ **Claude Code** — unavailable: out of usage limits until Aug 20

Then tell me:

  1️⃣  **Model** — any name above
  2️⃣  **Reasoning effort** — `low` · `medium` · `high` · `xhigh` · `max`
  3️⃣  **Speed** — ⚡ `fast` or 🐢 `normal`

⭐ **Recommended:** Codex · GPT-5.6 Sol · `high` · 🐢 normal — the depth a review needs.
```

Fill it in per §8's rules: one ⛔ line for each harness that dropped out with the actual reason in a few words, Cursor shortlisted to the prominent families, and no effort or speed knob offered that the models in hand do not have.

**The recommended default is Codex `gpt-5.6-sol` at `high` reasoning, in normal mode** — the depth a review needs, without the cost of `xhigh` or the throughput trade of fast. If Codex is not a candidate, recommend the equivalent through Cursor: GPT-5.6 Sol, high. If neither is in the listings you actually got, **offer no recommended default** — present what is there and let the user choose; never silently substitute a neighbouring model.

The user answers conversationally — "sol on high", "opus, think hard", "grok, fast". You hold the full listings, so you resolve that into the exact model string and flags. Answer follow-up questions about what else is available from those listings; do not make the user read a slug.

**A reviewer from a different model family than the author is worth more than a stronger model from the same one.** If this session's work was written by one harness, say so and lean toward reviewing it with another.

### Step 4 — lock it in

Translate the choice into concrete flags using the [embedded harness CLI reference](#harness-cli-reference) §3, and use them verbatim in **every** round of this review.
Do not change harness, model, or effort mid-review on your own.

## 2. Establish the subject of the review

The review has exactly one subject, and it comes from one of two places.

**When the user names it, that is the subject — and you do not go reading.** A plan path, a session file, a branch diff, a feature, a module: take it as given. You do **not** open the code, the plan, or the diff to prepare. The reviewer has its own context window and its own file access; reading first burns your context on material you will read again while remediating, and it primes you to agree with whichever findings match what you already noticed. **You dive into the codebase after the first review lands, not before.**

**When the user does not name it, the subject is this session's work.** Whatever this chat has been doing — authoring a plan, executing a task, supervising a delegated run — is what gets reviewed. Reconstruct that from the conversation you already have, not from fresh file reads.

Either way, the brief you hand the reviewer contains:

- **the goal of the work**, summarized briefly — what this was trying to achieve, and any decisions that constrain how,
- **the files changed so far, committed or not** — the full list, as one list, with no distinction drawn between staged, unstaged, and committed work; the review covers all of it,
- **the documents it should read in full** — a plan, a session file, a spec — named by path, never summarized by you,
- **a log of anything you changed since the previous round** (round 2 onward; see section 7).

Keep the summary short and factual. It exists so the reviewer knows what to open and what "correct" means here, not to pre-argue the conclusion. Do not editorialize about quality, do not list your own suspicions as findings, and do not tell it where you think the problems are — a brief that seeds the answer gets you your own opinion back with a second signature on it.

`git status --short` and `git log --oneline` to assemble the changed-file list are fine — that is the one cheap command this section licenses, and it is not the same as reading the files.

Before every reviewer launch, capture a content baseline under the locally ignored `.agent-runs/` directory as three separate artifacts: `git diff --cached --binary HEAD` for the index, `git diff --binary` for unstaged tracked changes, and a null-safe manifest containing the path, file type, and SHA-256 content hash of every untracked file.
Run the embedded harness CLI reference §4's local-ignore preflight before writing that baseline.
The separate binary snapshots preserve staged-versus-unstaged state and detect changes to already-modified binary files.
The baseline is evidence of the exact pre-review tree; a `git status --short` string alone is not sufficient because a reviewer can change an already-modified file without changing its status code.

Tell the reviewer to read and follow any applicable `AGENTS.md` and `CLAUDE.md` files before reviewing, because the harnesses do not auto-load the same instruction filenames.
Do not restate those files' conventions, rules, or commands in the prompt.

## 3. Ask for a review, and be explicit that it changes nothing

The reviewer must be able to substantiate its claims — run the type-checker, run the linter, execute a script to prove a framework behaves the way it says.
That capability is for **evidence, not repair**.
Only Cursor has a mode that enforces the split (see section 4); on Codex and Claude Code the prompt instruction is the entire guard, so state it plainly there and verify afterwards that it held.

State in the prompt, plainly:

- **Do not modify, create, or delete any file in the repository.** Report what should change; leave the changing to the supervisor.
- Produce a **final written review** as the deliverable: numbered findings, each with the file and line it concerns, what is wrong, why it matters, and what it would take to fix.
- **Separate blocking defects from judgment calls**, and say which each is.
- **Give evidence, not assertion.** A claim about how a framework, type, or query behaves should come with the command run and its output, or be labelled as an inference.
- **Say explicitly when it finds nothing.** "No blocking findings" is a valid and useful review; padding a clean review with speculative nits wastes a round.
- Look for functionality gaps and requirements silently skipped, not only for defects in what was written.

Verify the no-edit instruction held rather than trusting it: after the run exits, compare the tracked diff and untracked path/type/content hashes with the pre-review content baseline.
If the reviewer changed anything, restore only the proven delta introduced after that baseline and tell it so in the next round.
Never reset, check out, or overwrite a whole path that was already dirty; if the reviewer's delta cannot be isolated safely, stop and ask the user rather than risking their prior work.

## 4. Launch it as a managed asynchronous task

Use the parent-specific long-running process facility from the adjacent guide you loaded for every launch and resume.
Retain the returned task or session handle and keep the shell command itself attached inside that managed session; do not add shell `&` or invent a detached process.

Take the launch command for the chosen harness from the [embedded harness CLI reference](#harness-cli-reference) §4, in that section's **review** posture:

- **Codex** — `--sandbox workspace-write`, with the no-edit instruction from section 3 doing the work.
- **Claude Code** — `--permission-mode bypassPermissions`, likewise guarded only by the instruction. Do **not** try to harden it with `--disallowed-tools`: verified, it blocks the edit tools and the model writes the file through Bash instead.
- **Cursor Agent** — `--mode ask --trust` in place of `-f`. Genuinely read-only — it runs shell commands but refuses writes outright — and the only real enforcement any of the three offers. The `--trust` is load-bearing; without it an unfamiliar workspace fails the run before the model sees the prompt.

**Never launch in plan mode, on any harness.** Plan mode changes what the agent is _for_: instead of reviewing, it drafts a proposal for your approval, which is not the deliverable this skill asked for. Verified on both CLIs that offer it — Cursor answered a work request with "I'll outline that plan for your approval", and Claude wrote a plan file to `~/.claude/plans/` rather than doing the task. Normal mode, or Cursor's `ask` mode, and nothing else.

All three share the same shape — prompt on stdin as a quoted heredoc, transcript to `<LOG>`, final message to `<RESP>`, exact id to `<SESSION>`:

```
<LOG>  = .agent-runs/co-review-<slug>-r1.log
<RESP> = .agent-runs/co-review-<slug>-r1.response.log
```

Before creating `.agent-runs/`, run the local-ignore preflight in the [embedded harness CLI reference](#harness-cli-reference) §4. After it passes, the run files cannot reach a commit or contaminate the content baseline used to confirm the reviewer changed nothing. Never write these files to `/tmp`.

For Codex, `-o` writes `<RESP>` itself.
For Claude Code and Cursor Agent there is no such flag, so **once the process has exited** use the file-to-file materialization command in the [embedded harness CLI reference](#harness-cli-reference) §4. It writes the compact session metadata and final review into `<RESP>` without exposing `<LOG>` to the parent context.

### `<RESP>` is the only file you ever read

- **`<RESP>` — compact session metadata plus the review. This is the only transcript-bearing agent-run file you read or print.**
- **`<SESSION>` — one exact session id. Read it only when resuming.**
- **`<LOG>` — the streamed transcript. You never read or print this file.** Not with Read, `cat`, `head`, `tail`, or a `grep` that writes to the tool result; not whole, not in part, not while it runs, not after it exits, not out of curiosity, and not for failure diagnosis.

The `<LOG>` holds the reviewer's entire intermediate reasoning — every tool call, every file it opened, every hypothesis it discarded — routinely tens of thousands of tokens. Reading it is the single most expensive mistake available in this skill, and it buys nothing: a run that finished has already told you everything it concluded, in `<RESP>`.

The shell may mechanically transform `<LOG>` into `<RESP>` after exit or extract only the exact session id into `<SESSION>`, with no output entering the parent context. Those are the only permitted contacts with the transcript. The parent reads `<RESP>` and reads `<SESSION>` only to resume.

Non-negotiable details:

- **Do not write the prompt to a file of its own.** The heredoc on stdin is the prompt; a separate file only adds a step and a leftover artifact.
- **Every round writes its own pair of files** (`-r1.*`, `-r2.*`, …). Never reuse or append to a previous round's files; you need each round's review intact to see which findings actually closed.
- **The harness, model, effort, and mode come from the section 1 choice**, not from a template you half-remember.

## 5. If delegation is impossible, offer the other harness before offering yourself

Delegation can fail before any work happens: no harness is installed, the account is out of usage/quota, authentication is missing or expired, the model is refused, or the process exits immediately with an error instead of running.

When that happens:

1. **Abort the loop.** Do not retry the same command in a loop, and do not silently fall back to reviewing the work yourself.
2. Tell the user plainly what blocked it, quoting the actual error.
3. **If another candidate harness is still standing, offer it first.** This is the common case for a Codex quota block, whose listing looked healthy in section 1 and only failed at launch. Switching harness still gets them a second opinion; reviewing it yourself does not. Say which harness and model you would switch to, and wait for the user's go-ahead.
4. **Only when no harness remains**, ask whether they want you to review it yourself instead — and **wait for an explicit yes**. A self-review is a weaker thing than the second opinion they asked for, so it needs their consent, not your inference.

A switch of harness restarts the review at round 1: a new reviewer has none of the previous session's context, so brief it from scratch rather than handing it a ledger of findings it never made.

One exception to "do not retry": a transient "model is at capacity" response is worth a single retry after a short wait. A second capacity failure is a block — report it and offer the alternatives above.

If the block appears mid-review (a resume round fails after earlier rounds succeeded), the same rules apply: report which findings are already remediated, what is on disk, and ask before continuing.

## 6. While it runs: wait on the managed handle

Say what you launched — harness, model, one sentence — then wait on the saved task or session handle until the process exits.
Use the parent's wait operation rather than polling files, sleeping in the shell, or doing unrelated work.
If a bounded wait returns while the process is still active, wait again on the same handle and keep any user-facing status update brief.

While a run is in flight, do **not**:

- touch the `<LOG>` in any way,
- open the `<RESP>` (it is not written until the process exits),
- start reading source files "to prepare" for the findings,
- begin editing anything, or narrate findings that have not arrived.

If the user asks for a status update, tell them it is still running and that you will report when it exits — that is the whole answer, and it needs no file access. If they explicitly ask you to check whether the run is alive, check the **process**, not the transcript:

```bash
pgrep -fl '<harness> .*<slug>' || echo "not running"
```

The process check answers the question.
Follow the adjacent parent guide for any parent-specific execution requirement.
**Reading the transcript never becomes acceptable, not even on request.**
It would also poison the review: you remediate against the finished write-up, never against half-formed findings glimpsed mid-run.

If the user reports the run appears wedged and wants it stopped, kill it with the matching `pkill` from the [embedded harness CLI reference](#harness-cli-reference) §7 and then follow section 5.

### If the exit itself signals failure

The managed process result carries the exit status. When it is non-zero, or `<RESP>` is empty or missing, the run died rather than finished — crash or panic, model at capacity, rate limit, quota exhaustion, expired authentication, or a parent tool that terminated the process.

Diagnose it only by using the [embedded harness CLI reference](#harness-cli-reference) §7's bounded file-to-file extractor, which writes the diagnostic into `<RESP>` without printing transcript content. Then read `<RESP>`. If no structured diagnostic can be materialized, report that fact; **never tail or otherwise inspect `<LOG>`**.

Then follow section 5 — say what failed, quote the evidence, and ask before doing anything else.

If a synchronous parent-tool call terminated the run, relaunch the identical command once through §4's managed asynchronous facility.

## 7. Read the review, then remediate or argue — never merely comply

When the managed process reports completion, then and only then:

1. Read the round's **`<RESP>`** — it contains compact session metadata and the review. **This is the only transcript-bearing agent-run file you read. Never read `<LOG>`, including on failure.**
2. Compare the tree byte-for-byte with the pre-review content baseline and confirm the reviewer changed nothing. Treat `git status --short` only as a summary. If content changed, restore only the isolated post-baseline delta; if isolation is uncertain, stop and ask the user.
3. **Now** open the code. Read the files each finding names, and the files those depend on, so you can judge the finding on the system as it actually is.
4. **Verify every finding independently.** A finding is a hypothesis about the code until you have seen the code confirm it. Reproduce claimed failures. Run the type-checker and linter yourself rather than accepting a reported result — in either direction, red or green.

Then sort every finding into exactly one of three buckets:

- **Confirmed** — you reproduced it, or read the code and agree. Fix it.
- **Rejected** — you looked and it is wrong: it misreads the code, misses a guard elsewhere, or contradicts a decision the task deliberately recorded. **Do not implement it.** Argue it in the next round with the evidence that refutes it.
- **Judgment call** — it is defensible either way. Say which way you went and why, and let the reviewer contest it.

**Never implement a finding you do not agree with.** Silent compliance is the failure mode this skill exists to prevent: it launders a wrong review into the codebase and destroys the value of the second opinion. If the reviewer is wrong, say so, with evidence. If it turns out to be right after all, accept it and move on without ceremony.

The symmetric obligation is just as binding: a finding you dislike is not thereby wrong. Rejecting one requires evidence you actually gathered, not a preference.

### Nothing closes on one side's word — including yours

**No item is settled until both sides have said so explicitly.** Sorting a finding into a bucket is your _opening position_, not a verdict. It becomes a closed item only when the reviewer has seen what you did about it and agreed, in its own words, in a later round.

This binds you in both directions, and the second is the one that gets skipped:

- **A finding you rejected** stays open until the reviewer says your rebuttal convinced it. Its silence is not assent, and neither is a label it applied before it saw your argument — "non-blocking", "a recommendation", "a nit", "judgment call" describes the finding's _severity_, never its _resolution_. Declining an item because it was labelled minor is deciding unilaterally.
- **A finding you accepted and fixed** stays open until the reviewer has re-reviewed that fix. A fix it has never seen is unverified by the reviewer, however confident you are and however green your own checks came back. Your own tsc/lint/test pass is necessary, not sufficient — the entire point of the second opinion is that it is not yours.

The same rule governs anything a _later_ round raises, including a defect in your own remediation. A round-1 finding and a round-3 aside are the same kind of object and need the same explicit close. There is no tier of item small enough to close by yourself.

Two consequences follow, and neither is optional:

- **Every round you change something, there is another round.** If you touched the tree in response to round N, round N+1 exists — even if the change was three lines, even if you are certain, even if the round otherwise ended in agreement. The last word on a change is never the author's.
- **Track the open set explicitly.** Keep a list of every item raised by either side and its state: `open`, `you-agreed-awaiting-review`, `you-rejected-awaiting-response`, or `closed-both-agreed`. Only the last state is done. Carry that list into each round's prompt so both sides are reasoning about the same ledger, and never let an item leave it silently.

The one thing that closes without a round trip is a **clean round-1 review you verified and agree with** — there is nothing in dispute and nothing you changed, so there is nothing for a second round to confirm. The moment you edit anything, that exemption is gone.

### Log what you changed, every round

Keep a running list of the edits you made in response to each round — one line per fix, naming the file and what changed — and **include it in the next round's prompt**. Also list what you fixed that the reviewer never raised, and any trivial cleanups you made along the way.

Otherwise it re-reviews a working tree that no longer matches the state its own findings describe, and will either re-raise closed issues or read your fixes as new defects.

You may fix anything that needs no debate without waiting to be told — a typo, a missing `await` in an unambiguous spot, a rename applied in four of five places, an unformatted file. Log it and move on.

### Stage only task-owned remediation before the next round

Once you have finished remediating a round and are about to send it back, stage only the paths or hunks that wholly belong to this review's remediation:

```bash
git add -- <task-owned-path>...
```

Never use `git add -A`, `git add .`, or another broad pathspec here.
For a file that mixes pre-existing user edits with remediation, stage only proven remediation hunks; if that cannot be done safely, leave it unstaged and save a round boundary under `.agent-runs/` for the next comparison.

This is a review tool, not a commit. Safe staging draws a line under remediation already accounted for so the next round can focus on what changed after it. Never stage the user's baseline, never `git commit` unless the user asks, and mention any staged paths in the close-out.

## 8. Send it back to the same session

**Every rebuttal, remediation report, and follow-up goes back to the same session.** Never start a fresh run for a later round — that throws away the context that makes iteration cheaper than a restart. The session id is constant for the whole review; only the file names change.

Capture the session id once during round 1 using the per-harness form in the [embedded harness CLI reference](#harness-cli-reference) §6. Confirm the task-local `<SESSION>` file is non-empty, then read that exact file for every later round; never rediscover the session from `<LOG>`, modification time, or `--last`.

Then resume with that section's command for your harness, writing **this round's** files (`-r2.response.log` and `-r2.log`) into `.agent-runs/`, never a previous round's, with the prompt as a heredoc on stdin, and keeping the review posture from section 4 — including `--mode ask --trust` on Cursor, and never plan mode. Watch the two harness-specific traps it documents: Codex's `resume` rejects `--sandbox` and needs every flag before the session id, and Claude Code must never be given `--no-session-persistence`. Resume rounds use the same parent-managed asynchronous facility as round 1.

The re-review prompt contains, in this order:

1. **The open ledger** — every item either side has raised that is not yet closed by both of you, with its current state. This goes first because it defines the round's scope. An item you believe is closed still appears, marked as closed by mutual agreement in round N, so neither side silently loses track of it.
2. **What you changed**, per finding — the fix and the file, plus anything you changed that it never raised.
3. **What you rejected**, per finding — the evidence that refutes it, stated as an argument it can contest rather than a verdict.
4. **The judgment calls** you made and the reasoning, offered for disagreement.
5. **A request to re-review the current tree** and confirm each finding is closed, plus check that the fixes introduced nothing new.
6. **An explicit invitation to push back.** Say you want the correct outcome, not agreement, and that a reviewer who caves to a wrong rebuttal is worse than one that argues. Ask it to say plainly when your rebuttal has convinced it, so a closed finding does not stay open out of politeness.
7. **A demand for a verdict on every open item, by name.** Ask it to close or contest each one individually rather than summarizing the round — a round that says "no blocking findings remain" while leaving three items unaddressed has settled nothing. Say that an item it considers minor still needs an explicit disposition, and that if it now agrees with a position of yours it should say so rather than going quiet.

Then wait on that round's managed process handle until it exits.
The rules in section 6 and the adjacent parent guide apply to every round without exception, especially the rule that the parent reads only `<RESP>` plus the one-line `<SESSION>` metadata needed to resume.

**Re-verify from scratch every round.** A fix can introduce a fresh regression, and a regression in round N+1 is not covered by the verification you did in round N.

## 9. When to stop and ask the operator

Keep iterating on your own. Do not narrate each round to the user and wait for permission — resolving findings is the work, not a decision point.

**Escalate only when the review surfaces something that genuinely needs the developer's decision**: a product or business question the code cannot answer, a requirement that appears to contradict itself, a trade-off with real consequences either way, or a change in scope beyond what was asked.

Match the escalation to the size of the question:

- **One narrow question** — ask it, in one sentence, and wait.
- **A few related questions** — ask them as a short numbered list, and wait.
- **A question too broad for that** — the decision reshapes the approach, or has several coupled unknowns — pause the review and interview the user about the coupled decisions before resuming. Keep the questions scoped to the blocker; do not turn the review into a new planning project.

Escalate with the review paused, not abandoned: keep the session id, and resume the same session once the user has answered.

**A disagreement with the reviewer is not by itself an escalation.** Two models arguing about what the code does almost always converges — one of you produces evidence and the other accepts it. Take it to the user only when it truly will not reconcile, and then present both positions and the evidence for each, not just yours.

## 10. Close out

The review ends in exactly one of two states:

- **Resolved** — every item on the ledger is closed by **both** sides explicitly, and the reviewer has re-reviewed the final state of the tree. A clean round-1 review that you verified and agree with closes here immediately; do not send a round trip to confirm agreement you already have. Any review in which you edited something reaches this state only after a round that saw those edits.
- **Escalated** — an issue needs the developer and cannot be settled between the two of you.

**Before you write a word of the close-out, walk the ledger and check the state of every item.** If any item is `you-agreed-awaiting-review` or `you-rejected-awaiting-response`, the review is **not** resolved and you do not get to close it — send another round instead. "The reviewer said no blocking findings remain" does not discharge this: that sentence is about severity, and an unaddressed item is unaddressed regardless of how either side graded it. Closing out with items in those states is the specific failure this section exists to prevent, and it is worse than an extra round, because it hands the user a conclusion built partly on your unreviewed word while implying both sides signed it.

Leave the files in the locally ignored `.agent-runs/` directory. Give the user the relevant `<RESP>` path in the close-out; do not open, summarize, or link `<LOG>` from the parent response.

Report to the user:

- **which harness and model reviewed the work**, in one line, including any mid-review switch and why,
- **what the review found and what you changed**, in a few lines,
- **which findings you rejected, and why** — this matters as much as the fixes, because it is where you overrode the reviewer — and say explicitly that it accepted each rejection, since a rejection it never conceded is not a resolved item,
- **which checks you ran yourself** and their results, and **which of your fixes the reviewer re-reviewed** — the user needs to know what carries two signatures and what carries one,
- **what remains unverified and who has to do it** — anything needing a browser, a live service, or credentials is the user's to run, and you must list the exact scenarios,
- **anything still open**, if the review escalated,
- a suggested commit message covering the remediation as a whole and its goal, not just the last round.

Note that the changes are staged (from the per-round staging step) and say so. Never **commit** unless the user asks.
If the work is iterating, update the previous commit message rather than inventing a second one.

## Harness CLI reference

> The verified command forms for the three coding-agent CLIs that the `co-implement`, `co-review`, and `orchestrate` skills delegate to.
> Every command below was run on this machine and behaved as described.
> Prefer these forms verbatim; when one fails, report what it printed rather than inventing a variant.

The three harnesses are **Codex** (`codex`), **Claude Code** (`claude`), and **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
In every command below, replace `<CURSOR_CMD>` with the Cursor executable found during availability checks and keep that choice for the whole task.
All three take the prompt on stdin, run headless, emit JSON, and can resume a session by id.

This reference describes the child harnesses being invoked.
Instructions that depend on which agent loaded the skill belong in the adjacent `codex.md`, `claude.md`, or `cursor.md` parent guide and must not be inferred from the child command being run.

### 1. Availability

```bash
which codex; which claude; which cursor-agent || which agent
```

A harness that prints no path is not installed and is not a candidate.
For Cursor, prefer `cursor-agent`; try the legacy `agent` executable only when `cursor-agent` is absent.
`which` exits non-zero for a missing binary, so run the checks as one line and read the paths, not the overall exit status.

**A parent may delegate to the child CLI from the same product.**
Do not remove a harness merely because it matches the agent that loaded the skill.

**Do not narrate any of this.** Which binaries exist, which ones failed availability checks and why, what you are about to run next — none of it is news to the user, and all of it is plumbing they asked you to handle. Run the commands and go straight to the menu in §8. The first thing the user should see from the preflight is the menu itself.

### 2. Model listing, and what a dead harness looks like

#### Codex

```bash
codex debug models | jq -r '
  ["MODEL", "REASONING EFFORTS", "MODES"],
  (
    .models[]
    | select(.visibility == "list")
    | [
        .slug,
        ([.supported_reasoning_levels[].effort] | join(", ")),
        (if any(.service_tiers[]?; .id == "priority") then "normal, fast" else "normal" end)
      ]
  )
  | @tsv
' | column -t -s $'\t'
```

**Never read the raw output of `codex debug models`** — the catalog carries every model's full system prompt and runs to roughly 250 KB. Always pipe it through `jq`.

The `MODEL` column is the exact `-m` value, `REASONING EFFORTS` is that row's permitted `model_reasoning_effort` levels, and `MODES` says whether the row accepts a fast tier.

> **Verified caveat, and it matters:** `codex debug models` succeeded on this machine while the account was fully out of usage quota.
> The catalog is served independently of the run quota, so a clean listing is **not** proof that Codex can execute anything.
> Codex's exhaustion surfaces only on the first real run — see §7.

#### Claude Code

```bash
echo "/model" | claude -p --output-format json | jq -r '.result'
```

Returns the current model and the alias list, e.g. `sonnet, opus, haiku, fable, best, sonnet[1m], opus[1m], fable[1m], opusplan, default, or a full model ID`.

#### Cursor Agent

```bash
<CURSOR_CMD> --list-models 2>&1 | sed 's/\x1b\[[0-9;]*[a-zA-Z]//g'
```

The raw output is ANSI-coloured and needs the `sed` filter to be readable.
Each line is `<slug> - <Display Name>`, and the list is long — roughly ninety entries.

Cursor encodes reasoning effort and speed **in the slug itself**: `cursor-grok-4.6-xhigh-fast` is one model, one effort, one mode.
There are no separate effort or speed flags. `<CURSOR_CMD> status` reports the logged-in account if you need to distinguish an auth failure from a listing failure.

#### Reading a failure

A listing that reports **not authenticated**, prompts for login, reports **expired credentials**, reports a **quota or usage limit**, exits non-zero, or returns nothing means that harness is out of service.
Drop it from the candidates and **keep its exact message** — the skill reports it to the user in the §8 menu.

### 3. Selecting model, effort, and speed

|                  | Model                                                    | Reasoning effort                                                               | Fast mode                                                         |
| ---------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Codex**        | `-m <slug>`                                              | `-c 'model_reasoning_effort="<effort>"'` — only a level that model's row lists | `-c 'service_tier="priority"'`; omit the flag entirely for normal |
| **Claude Code**  | `--model <alias-or-id>`                                  | `--effort low\|medium\|high\|xhigh\|max`                                       | not selectable from the CLI                                       |
| **Cursor Agent** | `--model <slug>` — effort and speed are part of the slug | in the slug (`-low`/`-medium`/`-high`/`-xhigh`/`-max`)                         | in the slug (`-fast` suffix)                                      |

Two verified traps:

- **Codex's fast tier is `priority`, not `fast`.** That is the `id` the model catalog gives it; `fast` is only its display name. The tier string is not validated locally, so a wrong value fails at request time or is silently ignored rather than erroring.
- **Claude's `--effort` only applies to models that have effort levels.** Passing `--effort low` together with `--model haiku` did not run Haiku — the run came back attributed to Sonnet 5. Pass `--effort` only with Opus, Sonnet, or Fable; omit it entirely for Haiku.

### 4. Launching a round

#### Always use the parent's managed asynchronous process facility

Child runs routinely outlive a synchronous shell-tool call.
Submit every launch and resume through the supervising agent's managed long-running process facility, retain the returned task or session handle, and wait on that handle until the process exits.
Use the exact facility required by the parent guide loaded from the skill entrypoint.
An unlisted parent must use its native managed background-task or yielded-session facility.

The shell block below remains a foreground command *inside* that managed session so its post-exit extraction runs in order.
Do not add shell `&`, `nohup`, or a detached subprocess of your own.
If the parent has no managed asynchronous process facility, stop and explain that this workflow cannot safely run there.

#### The shape

Every launch is the same shape: the prompt arrives as a quoted heredoc on stdin, the transcript goes to `<LOG>`, the final message ends up in `<RESP>`, and the exact resumable session id goes to `<SESSION>`.
The heredoc avoids every quoting problem an inline prompt creates and leaves no temp file behind.

Before the first launch in a working repository, add `/.agent-runs/` to Git's local exclude file and verify that Git ignores it:

```bash
agent_runs_exclude_path="$(git rev-parse --git-path info/exclude)" || exit 1
mkdir -p "$(dirname "$agent_runs_exclude_path")"
touch "$agent_runs_exclude_path"
grep -qxF '/.agent-runs/' "$agent_runs_exclude_path" || \
  printf '\n/.agent-runs/\n' >> "$agent_runs_exclude_path"
git check-ignore -q --no-index .agent-runs/.ignore-check || {
  echo "error: .agent-runs/ is not ignored" >&2
  exit 1
}
mkdir -p .agent-runs
```

This changes only local Git metadata, not the repository's tracked `.gitignore`.
If the directory is not a Git working tree or the ignore check fails, stop before launching a harness.

All three run files live in **`.agent-runs/` at the repository root**, after that preflight has made the directory locally ignored.
Use the same task-and-round stem for `<LOG>` and `<RESP>`, and one stable `.agent-runs/<task>.session` path for `<SESSION>` across every round.
Never write them to `/tmp`, and never anywhere tracked.
`<W>` below is the absolute path of the working directory.

#### Codex

```bash
codex_run_status=0
codex exec \
  -m <model> \
  -c 'model_reasoning_effort="<effort>"' \
  -c 'service_tier="priority"' \
  --sandbox workspace-write \
  --json \
  -o <RESP> \
  - > <LOG> 2>&1 <<'PROMPTEOF' || codex_run_status=$?
<the prompt>
PROMPTEOF
jq -Rr 'fromjson? | select(.type == "thread.started") | .thread_id // empty' \
  < <LOG> 2>/dev/null | head -n 1 > <SESSION>
exit "$codex_run_status"
```

Codex writes the final message to `-o` itself, so no extraction step is needed.
`--json` makes stdout a JSONL event stream; its first `thread.started` event carries the exact resumable `thread_id`.
The post-exit extractor ignores non-JSON stderr, writes only that id to `<SESSION>`, and preserves the Codex process's exit status.
The trailing `-` is required for it to read the heredoc from stdin; because the heredoc closes stdin, never add a `< /dev/null` guard as well or the prompt arrives empty.
Drop the `service_tier` line for normal mode.
Keep the selected model's default context and compaction settings unless the user explicitly asks to override them and the model catalog confirms the requested values are supported.

Separately, **ask the user** whether to add `-c 'features.memories=false'` any time Codex is chosen as the implementor or reviewer — it disables Codex's cross-session memory for that run. This one is not a default: confirm it in chat before the first launch of a Codex round, and carry the answer for every later round in the same task.

#### Claude Code

```bash
claude -p \
  --model <model> \
  --effort <effort> \
  --permission-mode bypassPermissions \
  --output-format stream-json --verbose \
  --add-dir "<W>" \
  > <LOG> 2>&1 <<'PROMPTEOF'
<the prompt>
PROMPTEOF
```

**Never pass `--no-session-persistence`** — it discards the session and makes §6's resume impossible.

#### Cursor Agent

```bash
<CURSOR_CMD> -p \
  --model <model> \
  -f \
  --output-format stream-json \
  --workspace "<W>" \
  > <LOG> 2>&1 <<'PROMPTEOF'
<the prompt>
PROMPTEOF
```

`-f` (force) is what makes it non-interactive with write access.

#### Review posture: normal mode, never plan mode

A review must not edit the tree, but **plan mode is not how you get that, on any harness.** Plan mode changes what the agent is _for_: it stops reviewing and starts drafting a proposal for approval, which is a different deliverable than the one the skill asked for. Verified on both CLIs that offer it — Cursor answered a work request with "I'll outline that plan for your approval", and Claude wrote a plan file to `~/.claude/plans/` instead of doing the task. **Never launch a review, or an implementation, in plan mode.**

Run reviews in normal mode, with one exception where a genuine read-only mode exists:

|                  | Review posture                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------ |
| **Codex**        | `--sandbox workspace-write`, exactly as above. The no-edit instruction in the prompt is the guard.           |
| **Claude Code**  | `--permission-mode bypassPermissions`, exactly as above. The no-edit instruction in the prompt is the guard. |
| **Cursor Agent** | `--mode ask --trust` in place of `-f`. Genuinely read-only and still runs shell commands.                    |

Cursor's **ask** mode is the one real enforcement available: it ran `ls -1` and reported the count, then refused the file creation outright — _"I can't create h.txt while Ask mode is on (writes are blocked)"_. Prefer it for Cursor reviews. `--trust` is not optional alongside it: Cursor gates an unfamiliar workspace behind a trust prompt that `-f` happens to satisfy but `--mode ask` does not, so without it the run exits non-zero with _"To proceed… Pass --trust, --yolo, or -f if you trust this directory"_ and never reaches the model.

**Do not try to enforce read-only on Claude with `--disallowed-tools`.** Verified: `--disallowed-tools "Edit,Write,NotebookEdit,MultiEdit"` blocked the edit tools and the model simply wrote the file through Bash instead. Denying tools moves the write, it does not prevent it. For Codex and Claude, the prompt instruction plus the caller's separate pre-run and post-run content snapshots are the guard. A `git status --short` comparison alone cannot detect changes to an already-dirty path.

#### Materializing the final response — Claude and Cursor only

Neither CLI has Codex's `-o`. Once the process has exited, mechanically transform the last result record into `<RESP>` with **all output redirected to that file**:

```bash
grep '"type":"result"' <LOG> \
  | tail -n 1 \
  | jq -r '"SESSION_ID: \(.session_id // "")\nIS_ERROR: \(.is_error // false)\nSUBTYPE: \(.subtype // "")\nAPI_ERROR_STATUS: \(.api_error_status // "")\n\n\(.result // empty)"' \
  > <RESP>
sed -n 's/^SESSION_ID: //p' <RESP> | head -n 1 > <SESSION>
```

This is a mechanical file-to-file extraction, not permission for the parent agent to inspect `<LOG>`. The command must not print any transcript-derived byte to the tool result or parent context.

Claude's `.result` is the final assistant message alone.
**Cursor's `.result` is every assistant message of the turn concatenated**, so its narration arrives along with its conclusion — still far cheaper than the transcript, but do not mistake the leading sentences for the deliverable.

### 5. The three files, and the two you are allowed to read

- **`<RESP>` — the compact control header plus final message. This is the only transcript-bearing agent-run file the parent ever reads or prints.**
- **`<SESSION>` — one exact session id and nothing else.** The parent may read it only to resume the same task.
- **`<LOG>` — the streamed transcript.** Written continuously while the run is alive. **The parent never reads or prints this file.** Not with Read, `cat`, `head`, `tail`, `grep` that writes to stdout, or any tool call whose result enters the parent context; not whole, not in part, not while it runs, not after it exits, and not even for failure diagnosis.

The `<LOG>` holds the harness's entire intermediate reasoning — every tool call, every file it opened, every thought it discarded. It is routinely tens of thousands of tokens, and pouring it into your context is the exact cost these skills exist to avoid. A run that succeeded has already told you everything it concluded, in `<RESP>`. Reading the transcript on top of that buys nothing and can cost more context than the entire task it was supervising.

The only permitted contact with `<LOG>` is a post-exit shell transformation whose stdout and stderr are fully redirected into `<RESP>`, or the metadata-only extraction of `thread.started.thread_id` into `<SESSION>`.
The parent then reads `<RESP>` and, only when resuming, `<SESSION>`.
There is no diagnostic exception and no fallback tail.

### 6. Resuming the same session

Every later round goes back to the **same session**; a fresh launch throws away the context that makes iteration cheaper than a restart.

Every round reads the exact id captured during round 1 from its task-local `<SESSION>` file.
Never discover a Codex session by newest modification time, global `--last`, or the parent process's `CODEX_THREAD_ID`: concurrent Codex activity can make all three identify the wrong thread.

```bash
test -s <SESSION>
session_id=$(sed -n '1p' <SESSION>)
```

For a legacy Codex run that predates `<SESSION>`, correlate the task log's **birth/creation time** with rollout files created in the same launch window, then save the matched filename id to `<SESSION>` before resuming.
Never fall back to the globally newest modified rollout unless you have proved no other Codex activity occurred.

```bash
# Codex — every flag BEFORE the session id; `resume` rejects --sandbox
codex exec resume \
  -m <model> \
  -c 'model_reasoning_effort="<effort>"' \
  -c 'service_tier="priority"' \
  -c 'sandbox_mode="workspace-write"' \
  --json \
  -o <RESP> \
  "$session_id" - > <LOG> 2>&1 <<'PROMPTEOF'
<the prompt>
PROMPTEOF

# Claude Code — same flags as the launch, plus --resume
claude -p --resume <session-id> --model <model> ... > <LOG> 2>&1 <<'PROMPTEOF'
<the prompt>
PROMPTEOF

# Cursor Agent — same flags as the launch, plus --resume
<CURSOR_CMD> -p --resume <session-id> --model <model> ... > <LOG> 2>&1 <<'PROMPTEOF'
<the prompt>
PROMPTEOF
```

All three keep the same session id across rounds, so the id captured after round 1 stays valid for the whole task.
If the exact id is unrecoverable, stop and report that fact rather than guessing with `--last`.

Resume rounds use the same parent-managed asynchronous process facility. Every word of §4's launch rule applies to every round.

### 7. Failure signatures, materialized into the response file

Read the exit status first; a `<RESP>` that is empty or missing means the same thing.

|                  | Signature of a dead run                                                                                                                                                                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Codex**        | Non-zero exit; the transcript ends in `{"type":"error","message":…}` and `{"type":"turn.failed",…}`. A usage-limit block reads _"You've hit your usage limit… try again at &lt;date&gt;"_ and, as noted in §2, is invisible to `codex debug models` — it appears only here. |
| **Claude Code**  | Non-zero exit, or a result record with `"is_error":true`; `.subtype` and `.api_error_status` name the cause.                                                                                                                                                                |
| **Cursor Agent** | Non-zero exit, or no `"type":"result"` record in the stream at all. An untrusted workspace fails this way, asking for `--trust`, `--yolo`, or `-f`.                                                                                                                         |

When a run dies and normal response materialization left `<RESP>` empty, mechanically extract a bounded diagnostic **into `<RESP>`**. These commands must not emit transcript-derived output to the parent context:

```bash
# Codex
{
  printf 'RUN_FAILED\n'
  jq -Rr '
    fromjson?
    | select(.type == "error" or .type == "turn.failed")
    | .message // (if (.error | type) == "object" then .error.message else .error end) // empty
  ' <LOG> 2>/dev/null | tail -n 3
} > <RESP>

# Only if this round emitted no thread.started event, so no agent transcript started:
if test "$(wc -l < <RESP>)" -eq 1 \
  && ! jq -Re 'fromjson? | select(.type == "thread.started") | .thread_id' <LOG> >/dev/null 2>&1; then
  {
    printf 'RUN_FAILED\n'
    sed $'s/\\033\\[[0-9;]*[a-zA-Z]//g' <LOG> \
      | awk 'NF && $0 !~ /^[[:space:]]*\\{/ { lines[++n] = substr($0, 1, 500) } END { first = n > 3 ? n - 2 : 1; for (i = first; i <= n; i++) print lines[i] }'
  } > <RESP>
fi

# Claude Code
{
  printf 'RUN_FAILED\n'
  grep -o '"result":"[^"]*"' <LOG> | tail -n 1
} > <RESP>

# Cursor Agent
{
  printf 'RUN_FAILED\n'
  grep -o '"error"[^,]*' <LOG> | tail -n 3
} > <RESP>
```

Then read `<RESP>`, never `<LOG>`. If the bounded extractor finds nothing, report `RUN_FAILED` with no structured diagnostic and offer the fallback harness. **Never tail the transcript.** Missing diagnostics are preferable to polluting the parent context with an unbounded agent trace.

Kill a wedged run by matching the harness and the slug:

```bash
pkill -f 'codex exec.*<slug>'
pkill -f 'claude .*<slug>'
pkill -f '<CURSOR_CMD> .*<slug>'
```

### 8. The model menu, as the user sees it

The listings are your reference, not the user's.
Show **display names only** — no slugs, no effort suffixes, no table, no commentary about how you obtained them.

Format it so the three things the user has to choose are unmissable:

```
Pick a harness and model:

🤖 **Codex** — GPT-5.6 Sol · GPT-5.6 Terra · GPT-5.6 Luna · GPT-5.5 · GPT-5.4
🖱️ **Cursor** — Cursor Grok 4.6 · Composer 2.5 · Claude Opus 5 / Sonnet 5 / Fable 5 · GPT-5.6 Sol / Luna · Kimi K3 · GLM 5.2

⛔ **Claude Code** — unavailable: out of usage limits until Aug 20

Then tell me:

  1️⃣  **Model** — any name above
  2️⃣  **Reasoning effort** — `low` · `medium` · `high` · `xhigh` · `max`
  3️⃣  **Speed** — ⚡ `fast` or 🐢 `normal`

⭐ **Recommended:** Codex · GPT-5.6 Luna · `xhigh` · ⚡ fast
```

Rules for filling that template in:

- **One line per available harness**, with the ⛔ line repeated for each harness that dropped out, naming the actual reason in a few words.
- **Cursor lists around ninety entries and dumping them is useless.** Shortlist the prominent families only — **Cursor Grok, Composer, the latest Claude, the latest GPT, the latest Kimi, the latest GLM** — collapsing each family's effort and speed variants into one name.
- **Offer only effort levels the shortlisted models actually support**, and say so if the user's pick narrows them.
- **Say when speed is not a choice.** Claude Code has no fast mode, and Codex's `gpt-5.4-mini` has no fast tier; do not offer a knob that does not exist for the model in hand.
- **One line of justification for the recommendation, at most.** "Quick, strong, and very cheap" is enough.

The user picks conversationally — "sol on high", "the fast one", "grok, cheap and quick".
You hold the full listing, so you translate that into the exact slug and flags from §3; do not make them read a slug.
Answer any follow-up about what else is available from the listing you already have.
