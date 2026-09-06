---
name: co-review
description: Obtain an independent review of a plan, diff, or task output from Codex, Claude Code, or Cursor Agent, then resolve findings through evidence, fixes, and rebuttals until both participants agree.
---

# Resolve an independent review

You are the author and the reviewer's counterpart.
The child reviewer reports findings; you own fixes.
Use the **mutual-agreement loop** in the [review protocol](#review-protocol).

Read the [harness reference](#harness-cli-reference) before running a child.
Read only the adjacent guide matching the parent that loaded this skill: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
For another parent, use its native managed asynchronous process facility.

## 1. Configure the reviewer

Before studying the review subject, discover available harnesses and models using the harness reference.
Reuse choices and authorization already supplied by the user or an orchestrator acting on the user's approved plan.
Do not repeat a model menu, permission question, or review-scope question that has already been answered.

The preferred recommendation is Codex GPT-5.6 Sol, `high`, normal; use the equivalent through Cursor if Codex is unavailable.
Recommend it only when the current catalog supports that combination.
Consider a different model family from the author when recommending an independent reviewer.
Otherwise present the available choices without inventing a replacement default.

Keep the chosen reviewer settings and full host access across all rounds.
If this skill was delegated, return genuinely missing user decisions to the coordinator rather than waiting for interactive input inside a headless run.

## 2. Brief and launch

If the user names a subject, use it.
Otherwise review the current conversation's work, including relevant committed and uncommitted changes.
Reconstruct the goal and file list from existing context and cheap Git metadata; do not pre-read the code to seed the review.

Build the brief from the shared review protocol.
Include the original implementor's completion report when this review follows delegated implementation.
Use the harness reference's minimal-prompt rule; do not repeat default instructions or ask for routine output the reviewer already knows to provide.

Capture a pre-review content baseline and the task's ownership boundary.
In parallel work, retain the coordinator's ownership and shared-resource constraints.
Launch the reviewer with normal evidence-gathering capabilities and the protocol's no-repair instructions.
Do not use plan mode.

## 3. Exchange evidence and fixes

Wait for the managed process to exit, then read its final response.
Check that it did not alter the reviewed work, accounting for expected concurrent changes.
Now inspect the relevant files, verify each finding, and respond as confirmed, rejected, or a judgment call.

Maintain the shared ledger.
Make accepted fixes, log all changes, and send the ledger, fixes, rebuttals, and judgment calls back to the exact same reviewer session.
Require an explicit disposition for every open item and a review of the affected final state.
Every remediation edit requires another review.

Use the shared ownership and staging rules.
When other tasks are active, do not stage or commit without the coordinator's serialized turn.
Continue autonomously until the mutual-agreement completion conditions hold or a user decision is required.

## 4. Close out or return to the coordinator

Before claiming resolution, check every ledger item and confirm the reviewer saw the final changes.
A clean first review that you verified and accepted closes immediately.
A review with unverified fixes or unanswered rebuttals remains open.

Report the reviewer, final verdict, ledger, fixes, accepted rebuttals, checks performed by each participant, and remaining limitations.
Link the final response file and mention any staging.
Follow the user's commit-message convention for remediation; commit only when authorized.

When invoked by an orchestrator, return that report to it.
Keep the task in review until both participants have agreed, and leave orchestration tracking and commit timing to the approved schedule.
On process failure, use the harness reference's recovery rules and preserve the sessions and completed work.

## Harness CLI reference

Shared launch, invocation, ownership, and recovery instructions for Codex, Claude Code, and Cursor Agent.
Parent-specific process tools belong in the adjacent parent guides.

Checked against official documentation and local CLI help on 2026-09-06: Codex 0.153.4, Claude Code 2.1.246, and Cursor Agent 2026.08.25-3e8eec8.
This is not a claim that every model, account, integration, or command combination was exercised.
Use the installed CLI's help and current catalog when a version differs; never invent flags or model IDs.

### 1. Discover and select

Before studying implementation files, find the installed executables.
A parent may delegate to the same product.

```bash
command -v codex
command -v claude
command -v cursor-agent || command -v agent
```

Treat each returned path independently; a missing executable does not invalidate the other candidates.
Use the discovered Cursor executable as `agent_cursor_cmd` throughout the task.
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

For Cursor:

```bash
set -o pipefail
"$agent_cursor_cmd" --list-models 2>&1 \
  | sed $'s/\033\\[[0-9;]*[A-Za-z]//g'
```

For Claude, `claude auth status` checks authentication.
Use the installed CLI's supported aliases and the [model configuration reference](https://code.claude.com/docs/en/model-config), accounting for known account or configured restrictions.
Claude does not document a general `--list-models` equivalent.
Do not present a model's prose response to `/model` as an authoritative account catalog.

Retain actual discovery errors and mark unavailable harnesses with a short reason.
Authentication and catalogs do not guarantee quota or model access; handle execution failures when they occur.

Present a compact menu only for choices still missing.
Show display names, one line per harness, and ask for the model, supported effort, and available speed choice.
Shortlist Cursor's model families rather than printing every variant.
Use the calling skill's recommendation only when available.
Treat selections supplied through an approved orchestration prompt as already answered.
Do not switch harness, model, effort, or requested speed on your own.
If runtime metadata reports a substituted model, surface it rather than attributing the work to the requested model.

| Harness | Model and effort | Speed |
| --- | --- | --- |
| Codex | `-m <id>`, `-c 'model_reasoning_effort="<effort>"'` | Fast uses `-c 'service_tier="priority"'` when the catalog supports it; normal uses the configured standard setting |
| Claude Code | `--model <alias-or-id>`, `--effort <supported-level>` | Supported Opus models can use `--settings '{"fastMode":true}'` |
| Cursor Agent | `--model <catalog-entry>` | Use the catalog's exact effort/speed variant or documented parameterized model form |

Omit effort flags for models that do not support them.
Do not invent Cursor suffixes from a family name.
If an explicit normal-speed choice conflicts with saved fast-mode settings, resolve that per-run setting before launch rather than silently retaining fast mode.
Claude fast mode requires supported account access and may use separately billed usage credits; offer it only with that distinction clear.
See [Claude fast mode](https://code.claude.com/docs/en/fast-mode).

### 2. Preserve normal capabilities

Use full host access for the authorized task, including local services, Docker, networking, resumes, and nested reviews.
Carry existing authorization forward; do not insert another approval question at every launch or review round.
Honor a user-requested restriction and any enforced host or organization policy.

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
| Cursor Agent | `--force --sandbox disabled --trust --approve-mcps` |

Claude's permission mode and Bash sandbox are separate controls.
Cursor's force flag and sandbox setting are also separate.
The Cursor MCP flag approves the configured servers for this run.
These options do not override explicit administrative restrictions or create missing credentials.
See [Claude permissions](https://code.claude.com/docs/en/permissions), [Claude sandboxing](https://code.claude.com/docs/en/sandboxing), and [Cursor CLI parameters](https://cursor.com/docs/cli/reference/parameters).

Start in the task's actual working directory with the normal user identity, environment, login, settings, skills, plugins, and configured integrations.
Keep the harness's default system prompt, memory, context, and compaction behavior unless the user chose an override.
Do not use `env -i`, an artificial home/config directory, `--ignore-user-config`, `--ignore-rules`, `--bare`, `--safe-mode`, `--strict-mcp-config`, or tool allowlists to simplify delegation.
Do not pass flags that disable skills, session persistence, or otherwise remove normal capabilities.
Merge necessary per-run settings without discarding unrelated settings.

Normal configuration is each child's own configuration; a CLI does not automatically inherit another product's settings or the parent chat's desktop-only tools.
Preserve ordinary automatic instruction discovery instead of copying those instructions into the task prompt.
Do not claim unavailable integrations are present.
See [Claude headless context loading](https://code.claude.com/docs/en/headless) and [Codex instruction discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

Run implementors and reviewers in normal agent mode.
A reviewer receives a no-repair role instruction, with normal tools available for verification.
Plan mode changes the deliverable and is not a substitute for review.

### 3. Minimal prompts and explicit skills

Pass the user's task and only missing execution-specific context: task scope, selected reviewer, approved access, ownership, concurrency, and coordination boundaries.
Include a task document when the user named it.
Do not append generic requests to read `AGENTS.md`, `CLAUDE.md`, context documents, or conventions the child normally discovers.
Do not restate routine commit-message or reporting instructions already provided by the user's environment.
Ask for an extra report field only when the workflow needs it and it is otherwise missing.

When delegating a skill, put the explicit invocation at the beginning of the child's user prompt:

| Child | Prompt form |
| --- | --- |
| Codex | `$co-review`, or the path-qualified mention `[$co-review](/absolute/skills/co-review/SKILL.md)` |
| Claude Code | `/co-review <task and supplied choices>` |
| Cursor Agent | `/co-review <task and supplied choices>` |

Use the actual registered command for a namespaced plugin skill.
The skill name is prompt content, not a shell command or a model flag.
Use a quoted heredoc or correctly quoted argument so the shell does not expand `$co-review` or interpret Markdown.
Codex documents explicit `$skill` invocation; Claude documents slash-skill expansion in `-p` prompts; Cursor documents explicit slash-skill invocation.
See [Codex skills](https://learn.chatgpt.com/docs/build-skills), [Claude headless skills](https://code.claude.com/docs/en/headless), and [Cursor skills](https://cursor.com/docs/skills).

Before offering an installed skill route, resolve its real entrypoint and confirm it is enabled and usable by the selected child in this working directory.
Use an existing skill listing when available, then check the relevant normal discovery locations:

| Child | Normal locations to check |
| --- | --- |
| Codex | Applicable ancestor `.agents/skills/` directories and `~/.agents/skills/`, plus configured plugin skills |
| Claude Code | Applicable `.claude/skills/`, `~/.claude/skills/`, and installed plugin skills |
| Cursor Agent | Applicable `.agents/skills/`, `.cursor/skills/`, their user-level equivalents, and supported compatibility/plugin locations |

Follow symlinks and actual configuration; do not assume a directory exists or a parent-visible skill is registered in every child.
File existence alone does not prove that a particular CLI version will expand its command.
If native loading fails, report the failure and use the workflow's embedded alternative.
Do not install a missing skill, enable a disabled one, or disguise its contents as a workaround.

Forward already approved reviewer settings and access as user decisions.
Explicit invocation does not itself override permissions; the launch settings establish the authorized access.
An unattended child returns any genuinely missing decision to its coordinator instead of waiting on an interactive question.

### 4. Run records and ownership

Before creating run artifacts in a Git working tree, establish a local ignore rule:

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

Capture ownership without dumping source into the parent context:
save `git diff --cached --binary HEAD` and `git diff --binary` as separate files, plus a NUL-safe untracked-file manifest with path, type, and SHA-256 content hash.
For an unborn repository, use the index-versus-empty-tree baseline instead of `HEAD`.
Retain content needed for any safe restoration; hashes alone cannot restore a modified untracked file.

Capture the initial baseline before implementation and a review baseline before each reviewer round.
Preserve pre-existing user work and other agents' changes.
Only stage, revert, or commit deltas whose ownership is established.
Never use broad staging such as `git add .` or `git add -A`.
In concurrent work, keep ownership boundaries and serialize all shared-index changes.

Each task and role gets a unique stem:
`.agent-runs/<task>-<role>-r1.log`, `...-r1.response.md`, and one stable `.agent-runs/<task>-<role>.session`.
Use new log/response files for every round and keep nested review artifacts distinct.
Store artifacts here rather than in `/tmp`; pass prompts directly instead of creating prompt files.

The parent reads only the final response after exit and the one-line session file when resuming.
Never read or print the streamed log, including for progress or failure diagnosis.
Only mechanical post-exit extraction of final output, session metadata, or bounded errors may consume it, with output redirected to files.

### 5. Launch and resume

Use the parent's managed asynchronous process facility and retain its handle.
Keep the shell block attached to that facility; do not add `&`, `nohup`, or an unmanaged detached process.
If the parent cannot manage the process lifetime, report that limitation.
Wait on the handle until exit; use it for lifecycle checks and cancellation.

In the examples, set `agent_model`, `agent_effort`, `agent_workspace`, `agent_log`, `agent_response`, and `agent_session` to the chosen values and task-local paths.
Set the command tool's working directory to `agent_workspace`.
The snippets use Bash-compatible shell syntax.
Include the invocation, its post-exit extraction, and exit-status propagation in the same managed shell block.
For every resume, require a nonempty saved session file and read its exact ID into `agent_session_id`.
For Claude and Cursor, define `agent_finish_json_round` from the finalizer section inside that shell block before the invocation.

#### Codex

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

For later rounds, read the saved ID and replace the invocation with:

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

#### Claude Code

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

#### Cursor Agent

```bash
agent_exit=0
"$agent_cursor_cmd" -p \
  --model "$agent_model" \
  --force --sandbox disabled --trust --approve-mcps \
  --workspace "$agent_workspace" \
  --output-format stream-json \
  > "$agent_log" 2>&1 <<'AGENT_PROMPT' || agent_exit=$?
<user task, or /co-review followed by the task and supplied choices>
AGENT_PROMPT
agent_finish_json_round
exit "$agent_exit"
```

For resume, add `--resume "$agent_session_id"` after `-p`, retaining the exact model variant and all other settings.
See [Cursor headless execution](https://cursor.com/docs/cli/headless) and [sandbox controls](https://cursor.com/docs/cli/overview).

#### JSON finalizer for Claude and Cursor

Define this shell function before the Claude or Cursor invocation and call it only after that CLI exits.
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
Claude's terminal result is its final report; Cursor's result can concatenate assistant text from the turn.
See [Cursor output format](https://cursor.com/docs/cli/reference/output-format).

Never replace a saved session ID with a different returned ID.
Use that exact task-local ID for every continuation; never use `--last`, `--continue`, file modification order, or the parent's session ID.
If no exact ID can be recovered from this task's own structured metadata, report that fact rather than guessing.

### 6. Failure and recovery

A non-zero exit, missing final response, materialized error, or missing required terminal result is a process failure.
Keep failure reporting separate from a successful process that asks a task-level question.

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

Correct an accidental launch restriction to the already-authorized full-access settings and resume the same task without asking for permission again.
For a parent tool that prematurely ended the process, retry once through its proper managed asynchronous facility.
A transient capacity failure permits one short delayed retry.
Do not retry quota, authentication, policy, or repeated capacity failures in a loop.

Report the actual blocker and current work.
Offer another available harness before offering to take over yourself, and wait for authorization to change the selected harness or role.
A new harness requires a fresh task brief; the previous harness's session ID is not portable.
An enforced denial is not a reason to clear security settings or bypass a host control.

In orchestration, stop new launches and settle already-active work before presenting blockers.
Preserve completed implementation, review ledgers, and resumable sessions.
Cancel through the managed handle or a verified task-specific process ID; do not use a broad name-based kill.

## Review protocol

Use this protocol when assessing an implementation, plan, or other task output.
The skill assigns the roles and the completion rule:

| Workflow | Author | Reviewer | Coordinator | Completion |
| --- | --- | --- | --- | --- |
| co-implement | Child implementor | Parent | Parent | Supervisor sign-off |
| co-review | Parent | Child reviewer | Parent | Mutual agreement |
| orchestrate, relayed review | Child implementor | Child reviewer | Orchestrator | Mutual agreement |
| orchestrate, delegated co-review | Original implementor running co-review | Its child reviewer | Orchestrator tracks completion | Mutual agreement |

The author owns fixes.
The reviewer independently assesses the work.
A coordinator that is neither author nor reviewer routes their reports and tracks resolution without deciding technical findings itself.

### Review brief

Give the reviewer the task, its constraints, the task-owned changed-file list, and the author's completion report when available.
Include task or specification paths supplied by the user; preserve their scope.
Treat the completion report as claims to check against the actual work.
Do not seed the review with the coordinator's suspicions or copy instructions the child normally loads.

Add only these review-specific instructions:

- Inspect the actual work and relevant dependencies against the task's requirements.
- Report defects, missing requirements, regressions, and unsupported completion claims.
- Do not repair the work, edit task definitions or tracking, or stage or commit.
- Give numbered findings with location, consequence, evidence, and what would resolve the issue.
- Separate blocking defects from judgment calls.
- State when no findings remain.
- Assess rebuttals independently and explicitly accept a convincing correction.

Both participants may use the harness's normal tools and authorized services to gather evidence.
Review is a role restriction, not a reason to remove shell, network, database, or browser capabilities.
Run checks appropriate to the work; documentation-only work does not call for code tests.

### Evidence and responses

Neither participant treats the other's report as proof.
Read the relevant work, reproduce claimed failures when practical, and distinguish observations from inferences.
Consider skipped requirements, integration behavior, error paths, and unintended scope changes.
Avoid speculative findings or changes justified only by personal taste.

For each finding, the author chooses one response:

| Response | Required action |
| --- | --- |
| Confirmed | Fix it and report the change and verification |
| Rejected | Give evidence explaining why the finding is incorrect |
| Judgment call | State the chosen approach and its trade-off |

Invite pushback in both directions.
Accept the stronger evidence, regardless of which agent supplied it.
A finding's severity does not determine whether it has been resolved.

### Mutual-agreement loop

Use this loop for co-review and every orchestration review.
Keep stable finding IDs and a compact ledger with these states:

- `open`
- `fixed-awaiting-review`
- `rebutted-awaiting-response`
- `closed-both-agreed`

1. The reviewer returns its findings.
2. The author verifies each finding, makes the fixes it accepts, and answers every item.
3. Send the updated ledger and the author's response to the same reviewer session.
   Include all edits since its last review, including changes it did not request.
4. The reviewer inspects the current work, checks the fixes for regressions, and explicitly closes or contests each item by ID.
5. Return any contested or new findings to the same author session and repeat.

Preserve both session IDs throughout the exchange.
When a coordinator relays messages, pass the participants' final reports and evidence faithfully; do not substitute the coordinator's technical verdict.
Compact repeated history into the ledger, but retain unresolved arguments and evidence.

Every edit after a review requires another review of the affected final state.
A rejection remains open until the reviewer explicitly accepts the rebuttal.
Silence, a lower severity, or a general "no blockers" statement does not close an unaddressed item.
Retain mutually closed items with their resolution so they do not disappear from the record.

Completion requires every item to be `closed-both-agreed`, the author's acceptance of the final disposition, and a reviewer verdict covering the final task state.
A clean first review that the author verifies and accepts needs no extra round.
If another task changes a relevant dependency after sign-off, recheck the affected conclusions before treating them as final.

### Supervisor sign-off

For co-implement, the parent reviewer independently checks the task-owned diff and completion claims.
Send substantive defects and contestable choices back to the same implementor session.
Re-review its subsequent changes until no blocking issue remains.

The supervisor may directly fix an obvious, uncontested mistake.
If that clears the last issue, verify and sign off without another delegate round.
If another round is needed, tell the implementor what the supervisor changed.
This exception does not apply to the mutual-agreement loop.

### Protect ownership during review

Use the content baselines and ownership rules in the harness reference.
The author pauses edits to the reviewed task while its reviewer runs.
Independent tasks may continue only where their files and mutable resources do not conflict with that review.

After review, check for unintended changes to the reviewed work.
In a shared checkout, other agents' expected changes are not reviewer edits.
Never restore a repository-wide snapshot over concurrent work.
Undo only a delta proven to belong to the reviewer; if ownership cannot be established, preserve the files and resolve the conflict before continuing.

In a single-writer workflow, stage only reviewed task-owned hunks when that gives the next round a useful diff boundary.
Leave mixed-ownership paths unstaged when separation is uncertain.
During parallel work, keep the shared index unchanged until the coordinator grants a serialized staging or commit turn.

### Decisions and close-out

Continue technical discussion autonomously while the participants are making progress.
Ask the user only for a requirement, scope, or consequential product decision the task does not answer, or a disagreement the participants cannot reconcile with evidence.
Present both positions and the remaining question; preserve the sessions for resumption.
Do not call an unresolved exchange complete.

Report the outcome, accepted fixes, accepted rebuttals, checks actually performed, and any remaining limitation.
For mutual-agreement reviews, include the final ledger and both participants' explicit disposition.
Keep reports concise and reference the final response files rather than streamed transcripts.
