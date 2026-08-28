---
name: co-implement
description: Delegate a task to another coding-agent CLI — Codex, Claude Code, or Cursor Agent — and supervise it as a reviewer, watching only for failure while it runs. Use when the user asks to implement something with another agent, or invokes /co-implement.
---

# Supervising a delegated implementation

You are the **supervisor and reviewer**, not the implementer.
The delegate writes the code; you decide whether it is correct, push back until it is, and sign off at the end.
The one thing you implement yourself is the fix too trivial to be worth a round trip — see section 7.

The delegate is whichever coding-agent CLI the user picks in section 1 — **Codex** (`codex`), **Claude Code** (`claude`), or **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
Every command form for all three lives in the [embedded harness CLI reference](#harness-cli-reference).
**Read that section during section 1**, and take the commands from it verbatim rather than from memory.

When you are the **Codex** parent, run every Codex, Claude Code, or Cursor Agent command through Codex's elevated host-shell path exactly as the [embedded harness CLI reference](#harness-cli-reference) §0 requires. Never use a parent-sandbox startup or authentication failure to conclude that the host harness is broken or logged out.

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

**Say none of this out loud.** Which binaries exist, which one you dropped and why, what you are about to run next — none of it is news to the user, and all of it is the plumbing they asked you to handle. Run steps 1 to 3 silently; the first thing the user sees from this section is the menu in step 4.

### Step 2 — drop yourself

**Whichever harness you are running as is not a candidate.** Delegating to your own CLI buys no second pair of eyes and no fresh context window — it only pays for a subprocess that thinks the way you already do.
Running inside Claude Code, that means `claude` is excluded even when it is installed.

### Step 3 — ask each survivor for its models

Run the listing command for every remaining candidate — the three forms are in the [embedded harness CLI reference](#harness-cli-reference) §2.
This is also the availability check, and it is where a broken harness reveals itself: **not authenticated**, a login prompt, expired credentials, a quota or usage-limit error, a non-zero exit, or empty output all mean that harness is out of service.

Drop it from the candidates, but **keep its exact message** — you report it in step 4. A harness the user believes they have access to, silently missing from the menu, is a worse outcome than a slow round.

One verified asymmetry you must not paper over: **Codex's model listing succeeds even when its account is out of usage quota.** A clean listing proves Codex can be _asked_ about models, not that it can _run_ one. Codex's exhaustion appears only on the first real launch, and section 5 handles it there.

**If no candidate survives this step**, stop and follow section 5 — do not read the task, and do not start implementing it yourself.

### Step 4 — put the choice to the user

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

### Step 5 — lock it in

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

Use the parent-specific long-running process facility in the [embedded harness CLI reference](#harness-cli-reference) §4 for every launch and resume.
Retain the returned task or session handle and keep the shell command itself attached inside that managed session; do not add shell `&` or invent a detached process.

Take the launch command for the chosen harness from the [embedded harness CLI reference](#harness-cli-reference) §4.
When the parent is Codex, the launch itself — and every later resume — must use `sandbox_permissions: "require_escalated"` with a task-specific approval justification, regardless of the chosen harness. Do not launch it inside the parent sandbox first.
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

The process check answers the question. When the parent is Codex, run this lifecycle check through the elevated host shell for every harness. **Reading the transcript never becomes acceptable, not even on request.**

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

Then wait on that round's managed process handle until it exits. The same rules from section 4 apply to every round without exception, especially the host-shell requirement and the rule that the parent reads only `<RESP>` plus the one-line `<SESSION>` metadata needed to resume.

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

<a id="harness-cli-reference"></a>
## Harness CLI reference

> The verified command forms for the three coding-agent CLIs that the `co-implement`, `co-review`, and `orchestrate` skills delegate to.
> Every command below was run on this machine and behaved as described.
> Prefer these forms verbatim; when one fails, report what it printed rather than inventing a variant.

The three harnesses are **Codex** (`codex`), **Claude Code** (`claude`), and **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
In every command below, replace `<CURSOR_CMD>` with the Cursor executable found during availability checks and keep that choice for the whole task.
All three take the prompt on stdin, run headless, emit JSON, and can resume a session by id.

### 0. A Codex parent must use the host shell for every harness

When the supervising parent is **Codex**, every external coding-agent CLI command — **Codex**, **Claude Code**, or **Cursor Agent** — must run outside the parent's sandbox through the elevated host-shell path (`sandbox_permissions: "require_escalated"`).
Set that on Codex's command-tool call; it is not a harness CLI flag.
This includes availability and model discovery, launches, resumes, provider status checks, and lifecycle checks or kills.
The child still receives its documented sandbox or permission flags: those configure the child only after it starts and cannot escape the parent's sandbox during initialization.
Supply a concise, task-specific approval justification on the first command; **do not try the parent sandbox first or diagnose its startup/authentication failure as a harness failure**.

Keep the command's working directory at `<W>` and preserve the parent-managed asynchronous launch rule below.
Host-shell elevation changes only where the harness process starts; it does not broaden the task, authorize extra edits, or relax any review/implementation guard.

### 1. Availability

```bash
which codex; which claude; which cursor-agent || which agent
```

A harness that prints no path is not installed and is not a candidate.
For Cursor, prefer `cursor-agent`; try the legacy `agent` executable only when `cursor-agent` is absent.
`which` exits non-zero for a missing binary, so run the checks as one line and read the paths, not the overall exit status.

**Exclude the harness you are yourself.** Delegating to your own CLI buys no second opinion and no fresh context window — it only pays for a subprocess that thinks the way you already do. Running inside Claude Code, `claude` is out.

**Do not narrate any of this.** Which binaries exist, which one you excluded and why, what you are about to run next — none of it is news to the user, and all of it is plumbing they asked you to handle. Run the commands and go straight to the menu in §8. The first thing the user should see from the preflight is the menu itself.

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

- **Codex parent:** use command execution with a short initial yield, retain the returned session id, and continue waiting through the session wait/input tool without reading `<LOG>`.
- **Claude Code parent:** use the Bash tool with `run_in_background: true` and retain its task id.
- **Other parents:** use their native managed background-task or yielded-session facility.

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
