# Harness CLI reference

Launch, continuation, local run records, output-extraction, and CLI-failure commands for Codex and Claude Code child processes.
Use this file only when the skill selected harness CLIs as its child runtime.
The skill's embedded delegation rules still govern selection, access, prompts, ownership, sessions, and recovery; this file adds the CLI mechanics.
Parent-specific process tools belong in the adjacent parent guides.

Checked against official documentation and local CLI help on 2026-09-06: Codex 0.153.4 and Claude Code 2.1.246.
This is not a claim that every model, account, integration, or command combination was exercised.
Use the installed CLI's help and current catalog when a version differs; never invent flags or model IDs.

## 1. Discover and select

Before studying implementation files, find the installed executables.
A parent may delegate to the same product.

```bash
command -v codex
command -v claude
```

Treat each returned path independently; a missing executable does not invalidate the other candidates.
Check available models and authentication without printing credentials or full configuration.

For Codex, filter the catalog before it enters the parent context:

```bash
set -o pipefail
codex debug models | jq -r '
  .models[] | select(.visibility == "list")
  | [.slug,
     ([.supported_reasoning_levels[]?.effort] | join(", ")),
     ([.service_tiers[]?.id] | join(", "))]
  | @tsv
'
```

Never print the raw catalog: it includes model prompts.
A successful listing does not establish remaining execution quota.

For Claude, `claude auth status` checks authentication.
Use the installed CLI's supported aliases and the [model configuration reference](https://code.claude.com/docs/en/model-config), accounting for known account or configured restrictions.
Claude does not document a general `--list-models` equivalent.
Do not present a model's prose response to `/model` as an authoritative account catalog.

| Harness | Model and effort | Speed |
| --- | --- | --- |
| Codex | `-m <id>`, `-c 'model_reasoning_effort="<effort>"'` | Fast uses `-c 'service_tier="priority"'` when the catalog supports it; normal uses the configured standard setting |
| Claude Code | `--model <alias-or-id>`, `--effort <supported-level>` | Supported Opus models can use `--settings '{"fastMode":true}'` |

Omit effort flags for models that do not support them.
If an explicit normal-speed choice conflicts with saved fast-mode settings, resolve that per-run setting before launch rather than silently retaining fast mode.
Claude fast mode requires supported account access and may use separately billed usage credits; offer it only with that distinction clear.
See [Claude fast mode](https://code.claude.com/docs/en/fast-mode).

## 2. Full-access CLI settings

There are two boundaries: where the parent starts the CLI, and where the child executes its tools.
The parent must use its approved host execution facility.
Disabling the child's sandbox cannot escape an outer sandbox.
Likewise, starting the CLI on the host does not remove a sandbox selected by the child's flags.
Codex's `workspace-write` profile therefore must not be the default for these full-access workflows.
See [Codex non-interactive execution](https://learn.chatgpt.com/docs/non-interactive-mode) and [permissions](https://learn.chatgpt.com/docs/permissions).

| Child | Full-access launch settings |
| --- | --- |
| Codex | `--dangerously-bypass-approvals-and-sandbox`, on both launch and resume |
| Claude Code | `--permission-mode bypassPermissions --settings '{"sandbox":{"enabled":false}}'` |

Claude's permission mode and Bash sandbox are separate controls.
These options do not override explicit administrative restrictions or create missing credentials.
See [Claude permissions](https://code.claude.com/docs/en/permissions) and [Claude sandboxing](https://code.claude.com/docs/en/sandboxing).

Start the CLI in the task's working directory; when the delegation rules assign a worktree, use its absolute path as `agent_workspace` for every launch and continuation.
Do not use `env -i`, an artificial home/config directory, `--ignore-user-config`, `--ignore-rules`, `--bare`, `--safe-mode`, `--strict-mcp-config`, or tool allowlists to simplify delegation.
Do not pass flags that disable skills, session persistence, or otherwise remove normal capabilities.
Merge necessary per-run settings without discarding unrelated settings.
See [Claude headless context loading](https://code.claude.com/docs/en/headless) and [Codex instruction discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

## 3. Skill invocation in CLI prompts

Use a quoted heredoc or correctly quoted argument so the shell does not expand `$co-review` or interpret Markdown.
Claude documents slash-skill expansion in `-p` prompts.

## 4. Run files

Before creating run artifacts or launching a child in a Git working tree, establish a local ignore rule:

```bash
agent_exclude_path="$(git rev-parse --git-path info/exclude)" || exit 1
mkdir -p "$(dirname "$agent_exclude_path")"
touch "$agent_exclude_path"
grep -qxF '/.agent-runs/' "$agent_exclude_path" ||
  printf '\n/.agent-runs/\n' >> "$agent_exclude_path"
git check-ignore -q --no-index .agent-runs/.ignore-check || exit 1
test -z "$(git ls-files -- .agent-runs)" || {
  echo "Run artifacts are already tracked; resolve that before delegation." >&2
  exit 1
}
mkdir -p .agent-runs
```

This changes local Git metadata, not tracked `.gitignore`.
If the directory is not a Git working tree or run artifacts cannot be kept untracked, stop before launching.
Store run artifacts under `.agent-runs/` rather than in `/tmp`.
Keep nested review artifacts distinct.

Store the delegation rules' ownership baselines here:
save `git diff --cached --binary HEAD` and `git diff --binary` as separate files, plus the NUL-safe untracked-file manifest and content needed for safe restoration.
For an unborn repository, use the index-versus-empty-tree baseline instead of `HEAD`.
Use this directory for an orchestration record when there is no established tracking location.

Give each task and role a unique stem, such as `.agent-runs/<task>-<role>`.
Use `.agent-runs/<task>-<role>-r1.log` and `...-r1.response.md`, with new log and response files for every round.
Store each session's exact ID in its own one-line session file.
A continued session keeps one stable `.agent-runs/<task>-<role>.session` file.
A fresh session for a later round writes a new round-specific file, such as `.agent-runs/<task>-<role>-r2.session`, and never overwrites an earlier session file.

The parent reads only the final response after exit and the one-line session file when continuing.
Never read or print the streamed log, including for progress or failure diagnosis.
Only mechanical post-exit extraction of final output, session metadata, or bounded errors may consume it, with output redirected to files.

## 5. Launch, continue, or start fresh

Use the parent's managed asynchronous process facility and retain its handle.
Keep the shell block attached to that facility; do not add `&`, `nohup`, or an unmanaged detached process.
If the parent cannot manage the process lifetime, report that limitation.
Wait on the handle until exit; use it for lifecycle checks and cancellation.

In the examples, set `agent_model`, `agent_effort`, `agent_workspace`, `agent_log`, `agent_response`, and `agent_session` to the chosen values and task-local paths.
Set the command tool's working directory to `agent_workspace`.
The snippets use Bash-compatible shell syntax.
Include the invocation, its post-exit extraction, and exit-status propagation in the same managed shell block.
For every continuation, require a nonempty saved session file and read its exact ID into `agent_session_id`.
For a fresh session, use the launch form with a new session file and a self-contained brief; never add a resume or session flag.
For Claude, define `agent_finish_json_round` from the finalizer section inside that shell block before the invocation.

### Codex

```bash
agent_exit=0
codex exec \
  --dangerously-bypass-approvals-and-sandbox \
  -m "$agent_model" \
  -c "model_reasoning_effort=\"$agent_effort\"" \
  --json -o "$agent_response" \
  - > "$agent_log" 2>&1 <<'AGENT_PROMPT' || agent_exit=$?
<user task, preceded by an explicit skill mention when requested>
AGENT_PROMPT

if ! test -s "$agent_session"; then
  jq -Rr 'fromjson? | objects | select(.type == "thread.started") | .thread_id // empty' \
    "$agent_log" 2>/dev/null | head -n 1 > "$agent_session"
fi
exit "$agent_exit"
```

Add `-c 'service_tier="priority"'` only for a selected, supported fast tier.
Codex writes the final response through `-o`.
The structured `thread.started.thread_id` identifies this exact child.
See [Codex exec output](https://learn.chatgpt.com/docs/non-interactive-mode).

To continue the session, read the saved ID and replace the invocation with:

```bash
agent_session_id="$(sed -n '1p' "$agent_session")"
test -n "$agent_session_id" || exit 1
agent_exit=0
codex exec resume \
  --dangerously-bypass-approvals-and-sandbox \
  -m "$agent_model" \
  -c "model_reasoning_effort=\"$agent_effort\"" \
  --json -o "$agent_response" \
  "$agent_session_id" - > "$agent_log" 2>&1 <<'AGENT_PROMPT' || agent_exit=$?
<follow-up for this same task>
AGENT_PROMPT
exit "$agent_exit"
```

Repeat the selected speed setting too.
All flags precede the session ID.
Do not add `--sandbox` to `exec resume`; the full-access flag above is supported on both forms.

### Claude Code

```bash
agent_exit=0
claude -p \
  --model "$agent_model" --effort "$agent_effort" \
  --permission-mode bypassPermissions \
  --settings '{"sandbox":{"enabled":false}}' \
  --output-format stream-json --verbose \
  > "$agent_log" 2>&1 <<'AGENT_PROMPT' || agent_exit=$?
<user task, or /co-review followed by the task and supplied choices>
AGENT_PROMPT
agent_finish_json_round
exit "$agent_exit"
```

Omit `--effort` for unsupported models.
If speed was explicitly chosen, merge `"fastMode":true` or `"fastMode":false` into that same settings object as appropriate.
For resume, add `--resume "$agent_session_id"` after `-p`, retaining all other settings and using new round files.
Never disable session persistence.
The working directory controls normal project discovery; `--add-dir` only adds other directories when needed.
See [Claude CLI flags](https://code.claude.com/docs/en/cli-reference).

### JSON finalizer for Claude

Define this shell function before the Claude invocation and call it only after that CLI exits.
It is a local shell helper, not an installed command.
It extracts the terminal report and session metadata without exposing the transcript:

```bash
agent_finish_json_round() {
jq -Rsr --argjson code "$agent_exit" '
  [split("\n")[] | fromjson? | objects] as $events
  | ([$events[] | select(.type == "result")] | last) as $r
  | ([$events[] | select(.type == "system" and .subtype == "init")] | first) as $init
  | "RUN_STATUS: \(if $code == 0 and $r != null and $r.is_error != true
                       and $r.subtype == "success"
                       and (($r.result // "") | length) > 0
                    then "COMPLETED" else "FAILED" end)\n"
    + "SESSION_ID: \($r.session_id // $init.session_id // "")\n"
    + "MODEL: \($init.model // "")\n"
    + "SUBTYPE: \($r.subtype // "")\n"
    + "API_ERROR_STATUS: \($r.api_error_status // "")\n\n"
    + ($r.result // "No terminal report; use bounded failure extraction.")
' "$agent_log" > "$agent_response" 2>/dev/null

if ! test -s "$agent_session"; then
  sed -n 's/^SESSION_ID: //p' "$agent_response" | head -n 1 > "$agent_session"
fi
}
```

A `FAILED` header is a failed run even if the CLI exited zero.
A successful process report can still describe a blocked or incomplete task; inspect its meaning.
Claude's terminal result is its final report.

Never replace a saved session ID with a different returned ID.
Use that exact task-local ID for every continuation; never use `--last`, `--continue`, file modification order, or the parent's session ID.
If no exact ID can be recovered from this task's own structured metadata, report that fact rather than guessing.

## 6. CLI failure diagnostics

A non-zero exit, missing final response, materialized error, or missing required terminal result is a process failure.

For failures without a useful final diagnostic, mechanically extract only explicit error records into a new response file:

```bash
jq -Rsr '
  [split("\n")[] | fromjson? | objects
   | select(.type == "error" or .type == "turn.failed"
            or (.type == "result" and .is_error == true))
   | (.message // (if (.error | type) == "object" then .error.message else .error end)
      // .result // .errors // empty)
   | if type == "string" then . else tojson end]
  | "RUN_FAILED\n" + (.[-3:] | join("\n") | .[0:2000])
' "$agent_log" > "$agent_response" 2>/dev/null
```

Keep an existing final report intact when it is useful; use a separate round-specific failure-response path in that case.
For a startup failure with no structured agent events, a bounded file-to-file extraction of the CLI's plain error is also acceptable.
If no diagnostic is available, report the exit status and missing diagnostic.
Do not inspect or tail the transcript.
For a parent tool that prematurely ended the process, retry once through its proper managed asynchronous facility.
Follow the delegation rules for the remaining recovery and reporting steps.
