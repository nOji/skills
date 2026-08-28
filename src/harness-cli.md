# Harness CLI reference

> The verified command forms for the three coding-agent CLIs that the `co-implement`, `co-review`, and `orchestrate` skills delegate to.
> Every command below was run on this machine and behaved as described.
> Prefer these forms verbatim; when one fails, report what it printed rather than inventing a variant.

The three harnesses are **Codex** (`codex`), **Claude Code** (`claude`), and **Cursor Agent** (`cursor-agent`, with `agent` as a legacy fallback).
In every command below, replace `<CURSOR_CMD>` with the Cursor executable found during availability checks and keep that choice for the whole task.
All three take the prompt on stdin, run headless, emit JSON, and can resume a session by id.

## 0. A Codex parent must use the host shell for every harness

When the supervising parent is **Codex**, every external coding-agent CLI command — **Codex**, **Claude Code**, or **Cursor Agent** — must run outside the parent's sandbox through the elevated host-shell path (`sandbox_permissions: "require_escalated"`).
Set that on Codex's command-tool call; it is not a harness CLI flag.
This includes availability and model discovery, launches, resumes, provider status checks, and lifecycle checks or kills.
The child still receives its documented sandbox or permission flags: those configure the child only after it starts and cannot escape the parent's sandbox during initialization.
Supply a concise, task-specific approval justification on the first command; **do not try the parent sandbox first or diagnose its startup/authentication failure as a harness failure**.

Keep the command's working directory at `<W>` and preserve the parent-managed asynchronous launch rule below.
Host-shell elevation changes only where the harness process starts; it does not broaden the task, authorize extra edits, or relax any review/implementation guard.

## 1. Availability

```bash
which codex; which claude; which cursor-agent || which agent
```

A harness that prints no path is not installed and is not a candidate.
For Cursor, prefer `cursor-agent`; try the legacy `agent` executable only when `cursor-agent` is absent.
`which` exits non-zero for a missing binary, so run the checks as one line and read the paths, not the overall exit status.

**Exclude the harness you are yourself.** Delegating to your own CLI buys no second opinion and no fresh context window — it only pays for a subprocess that thinks the way you already do. Running inside Claude Code, `claude` is out.

**Do not narrate any of this.** Which binaries exist, which one you excluded and why, what you are about to run next — none of it is news to the user, and all of it is plumbing they asked you to handle. Run the commands and go straight to the menu in §8. The first thing the user should see from the preflight is the menu itself.

## 2. Model listing, and what a dead harness looks like

### Codex

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

### Claude Code

```bash
echo "/model" | claude -p --output-format json | jq -r '.result'
```

Returns the current model and the alias list, e.g. `sonnet, opus, haiku, fable, best, sonnet[1m], opus[1m], fable[1m], opusplan, default, or a full model ID`.

### Cursor Agent

```bash
<CURSOR_CMD> --list-models 2>&1 | sed 's/\x1b\[[0-9;]*[a-zA-Z]//g'
```

The raw output is ANSI-coloured and needs the `sed` filter to be readable.
Each line is `<slug> - <Display Name>`, and the list is long — roughly ninety entries.

Cursor encodes reasoning effort and speed **in the slug itself**: `cursor-grok-4.6-xhigh-fast` is one model, one effort, one mode.
There are no separate effort or speed flags. `<CURSOR_CMD> status` reports the logged-in account if you need to distinguish an auth failure from a listing failure.

### Reading a failure

A listing that reports **not authenticated**, prompts for login, reports **expired credentials**, reports a **quota or usage limit**, exits non-zero, or returns nothing means that harness is out of service.
Drop it from the candidates and **keep its exact message** — the skill reports it to the user in the §8 menu.

## 3. Selecting model, effort, and speed

|                  | Model                                                    | Reasoning effort                                                               | Fast mode                                                         |
| ---------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Codex**        | `-m <slug>`                                              | `-c 'model_reasoning_effort="<effort>"'` — only a level that model's row lists | `-c 'service_tier="priority"'`; omit the flag entirely for normal |
| **Claude Code**  | `--model <alias-or-id>`                                  | `--effort low\|medium\|high\|xhigh\|max`                                       | not selectable from the CLI                                       |
| **Cursor Agent** | `--model <slug>` — effort and speed are part of the slug | in the slug (`-low`/`-medium`/`-high`/`-xhigh`/`-max`)                         | in the slug (`-fast` suffix)                                      |

Two verified traps:

- **Codex's fast tier is `priority`, not `fast`.** That is the `id` the model catalog gives it; `fast` is only its display name. The tier string is not validated locally, so a wrong value fails at request time or is silently ignored rather than erroring.
- **Claude's `--effort` only applies to models that have effort levels.** Passing `--effort low` together with `--model haiku` did not run Haiku — the run came back attributed to Sonnet 5. Pass `--effort` only with Opus, Sonnet, or Fable; omit it entirely for Haiku.

## 4. Launching a round

### Always use the parent's managed asynchronous process facility

Child runs routinely outlive a synchronous shell-tool call.
Submit every launch and resume through the supervising agent's managed long-running process facility, retain the returned task or session handle, and wait on that handle until the process exits.

- **Codex parent:** use command execution with a short initial yield, retain the returned session id, and continue waiting through the session wait/input tool without reading `<LOG>`.
- **Claude Code parent:** use the Bash tool with `run_in_background: true` and retain its task id.
- **Other parents:** use their native managed background-task or yielded-session facility.

The shell block below remains a foreground command *inside* that managed session so its post-exit extraction runs in order.
Do not add shell `&`, `nohup`, or a detached subprocess of your own.
If the parent has no managed asynchronous process facility, stop and explain that this workflow cannot safely run there.

### The shape

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

### Codex

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

### Claude Code

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

### Cursor Agent

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

### Review posture: normal mode, never plan mode

A review must not edit the tree, but **plan mode is not how you get that, on any harness.** Plan mode changes what the agent is _for_: it stops reviewing and starts drafting a proposal for approval, which is a different deliverable than the one the skill asked for. Verified on both CLIs that offer it — Cursor answered a work request with "I'll outline that plan for your approval", and Claude wrote a plan file to `~/.claude/plans/` instead of doing the task. **Never launch a review, or an implementation, in plan mode.**

Run reviews in normal mode, with one exception where a genuine read-only mode exists:

|                  | Review posture                                                                                               |
| ---------------- | ------------------------------------------------------------------------------------------------------------ |
| **Codex**        | `--sandbox workspace-write`, exactly as above. The no-edit instruction in the prompt is the guard.           |
| **Claude Code**  | `--permission-mode bypassPermissions`, exactly as above. The no-edit instruction in the prompt is the guard. |
| **Cursor Agent** | `--mode ask --trust` in place of `-f`. Genuinely read-only and still runs shell commands.                    |

Cursor's **ask** mode is the one real enforcement available: it ran `ls -1` and reported the count, then refused the file creation outright — _"I can't create h.txt while Ask mode is on (writes are blocked)"_. Prefer it for Cursor reviews. `--trust` is not optional alongside it: Cursor gates an unfamiliar workspace behind a trust prompt that `-f` happens to satisfy but `--mode ask` does not, so without it the run exits non-zero with _"To proceed… Pass --trust, --yolo, or -f if you trust this directory"_ and never reaches the model.

**Do not try to enforce read-only on Claude with `--disallowed-tools`.** Verified: `--disallowed-tools "Edit,Write,NotebookEdit,MultiEdit"` blocked the edit tools and the model simply wrote the file through Bash instead. Denying tools moves the write, it does not prevent it. For Codex and Claude, the prompt instruction plus the caller's separate pre-run and post-run content snapshots are the guard. A `git status --short` comparison alone cannot detect changes to an already-dirty path.

### Materializing the final response — Claude and Cursor only

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

## 5. The three files, and the two you are allowed to read

- **`<RESP>` — the compact control header plus final message. This is the only transcript-bearing agent-run file the parent ever reads or prints.**
- **`<SESSION>` — one exact session id and nothing else.** The parent may read it only to resume the same task.
- **`<LOG>` — the streamed transcript.** Written continuously while the run is alive. **The parent never reads or prints this file.** Not with Read, `cat`, `head`, `tail`, `grep` that writes to stdout, or any tool call whose result enters the parent context; not whole, not in part, not while it runs, not after it exits, and not even for failure diagnosis.

The `<LOG>` holds the harness's entire intermediate reasoning — every tool call, every file it opened, every thought it discarded. It is routinely tens of thousands of tokens, and pouring it into your context is the exact cost these skills exist to avoid. A run that succeeded has already told you everything it concluded, in `<RESP>`. Reading the transcript on top of that buys nothing and can cost more context than the entire task it was supervising.

The only permitted contact with `<LOG>` is a post-exit shell transformation whose stdout and stderr are fully redirected into `<RESP>`, or the metadata-only extraction of `thread.started.thread_id` into `<SESSION>`.
The parent then reads `<RESP>` and, only when resuming, `<SESSION>`.
There is no diagnostic exception and no fallback tail.

## 6. Resuming the same session

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

## 7. Failure signatures, materialized into the response file

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

## 8. The model menu, as the user sees it

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
