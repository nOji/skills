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

## Harness CLI reference

Shared launch, invocation, ownership, and recovery instructions for Codex, Claude Code, Cursor Agent, and Muse Code (`muse`).
Parent-specific process tools belong in the adjacent parent guides.

Checked against official documentation and local CLI help on 2026-09-06: Codex 0.153.4, Claude Code 2.1.246, and Cursor Agent 2026.08.25-3e8eec8.
Muse additions checked on 2026-10-14 against official documentation, local help, the live provider catalog, and Muse Code 1.4.7 (`1.4.7-R6101.1`) probes with `muse-spark-1.3`.
This is not a claim that every model, account, integration, or command combination was exercised.
Use the installed CLI's help and current catalog when a version differs; never invent flags or model IDs.

### 1. Discover and select

Before studying implementation files, find the installed executables.
A parent may delegate to the same product.

```bash
command -v codex
command -v claude
command -v cursor-agent || command -v agent
command -v muse
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

For Muse, inspect the installed command surfaces independently:

```bash
muse --version
muse exec --help
muse model-profile show muse-spark-1.3 --effort high
```

`model-profile show` explains local harness tuning; it is not an authenticated catalog or quota check.
Version 1.4.7 has no `--list-models` or `auth status` command.
Use interactive `/models`, or the read-only MSP `model/list` request through `muse serve` after its initialization handshake.
The request accepts `{}` before creating a session; retain the catalog's `source`, `modelId`, `variants`, and `defaultReasoningEffort`, not raw configuration.
See [Muse catalog discovery](https://meta-models.github.io/muse-code-sdk/next/cookbook/list-models-and-switch-mid-session/).
The inspected `providerCatalog` row advertised `muse-spark-1.3` with `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`, and a default of `max`.
Generic CLI help says the default is `high`; explicitly pass the selected effort rather than relying on either default.

Use Muse's existing login; the documented credential order is `META_API_KEY`, stored API key, then browser login.
If setup is required, `muse login` starts browser authentication and `muse auth set --api-key-stdin` stores a key without putting it in command arguments.
Do not read credential files or replace an existing login merely to discover models.
See [Muse authentication](https://dev.meta.ai/docs/muse-code/auth).

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
| Muse Code | `--model <id>`, `--reasoning-effort <catalog-level>` (alias `--effort`) | No independent fast/service-tier flag in 1.4.7; effort and contributor mode are separate choices |

Omit effort flags for models that do not support them.
Do not invent Cursor suffixes from a family name.
If an explicit normal-speed choice conflicts with saved fast-mode settings, resolve that per-run setting before launch rather than silently retaining fast mode.
Claude fast mode requires supported account access and may use separately billed usage credits; offer it only with that distinction clear.
See [Claude fast mode](https://code.claude.com/docs/en/fast-mode).

Muse Spark 1.3 probes succeeded at every catalog effort from `minimal` through `max` with `--contributor-mode off`.
`none` is listed by generic help but rejected by the Meta provider with exit 2.
`ultra` completed with an explicit stderr warning that its gate was closed and `xhigh` was used instead; do not offer it as a verified Spark 1.3 tier.
Check catalog support and report runtime fallback warnings even when the process succeeds.
These observations establish invocation support, not relative reasoning quality or latency.
See [Meta reasoning levels](https://dev.meta.ai/docs/reasoning).

Muse's `--contributor-mode on|off` overrides this run without saving a preference.
Contributor mode changes data-use and billing terms; it is not a speed tier.
For standard Spark 1.3, use `--model muse-spark-1.3 --contributor-mode off` to avoid inheriting a contributor preference.
Enable contributor mode only when selected by the user, and recheck the resulting catalog and supported efforts; Meta excludes `max` from `muse-spark-1.3-contributor`.
A local probe with `--model muse-spark-1.3 --reasoning-effort high --contributor-mode on` completed with exit 0 and the expected terminal response, without fallback warnings.
Its runtime model metadata retained `muse-spark-1.3`; do not require a `-contributor` suffix in that field as proof that the per-run flag was accepted.
See [Muse model variants](https://dev.meta.ai/docs/models).

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
| Muse Code | `--yolo` on both fresh `exec` and exact-session continuation |

Claude's permission mode and Bash sandbox are separate controls.
Cursor's force flag and sandbox setting are also separate.
The Cursor MCP flag approves the configured servers for this run.
Muse's `--yolo` disables child approvals and sandboxing and trusts the workspace for this run.
`--disable-approval` alone retains its sandbox; `--disable-sandbox` alone retains approvals.
Workspace trust loads project instructions, skills, and hooks; it is separate from permission to execute tools.
See [Muse permissions](https://dev.meta.ai/docs/muse-code/permissions).
These options do not override explicit administrative restrictions or create missing credentials.
See [Claude permissions](https://code.claude.com/docs/en/permissions), [Claude sandboxing](https://code.claude.com/docs/en/sandboxing), and [Cursor CLI parameters](https://cursor.com/docs/cli/reference/parameters).

Start in the task's actual working directory with the normal user identity, environment, login, settings, skills, plugins, and configured integrations.
When the parent is instructed to work in a Git worktree, whether selecting an existing one or creating one, resolve that worktree's absolute path before delegating and use it as `agent_workspace` for every child launch and resume.
Explicitly tell each child, including implementors and reviewers, to work in that worktree and pass its absolute path in the task prompt.
Require children to carry the same worktree path and instruction forward to any agents they summon, including nested reviews, unless the user explicitly assigns a different worktree to that task.
Keep the harness's default system prompt, memory, context, and compaction behavior unless the user chose an override.
Do not use `env -i`, an artificial home/config directory, `--ignore-user-config`, `--ignore-rules`, `--bare`, `--safe-mode`, `--strict-mcp-config`, or tool allowlists to simplify delegation.
Do not pass flags that disable skills, session persistence, or otherwise remove normal capabilities.
Merge necessary per-run settings without discarding unrelated settings.

Normal configuration is each child's own configuration; a CLI does not automatically inherit another product's settings or the parent chat's desktop-only tools.
Preserve ordinary automatic instruction discovery instead of copying those instructions into the task prompt.
Do not claim unavailable integrations are present.
See [Claude headless context loading](https://code.claude.com/docs/en/headless) and [Codex instruction discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md).

Muse normally loads user settings from `~/.config/muse/settings.json` (respecting its configured XDG location), and discovers project instructions inside the Git boundary.
At each directory level its instruction precedence is `AGENTS.md`, `CLAUDE.md`, `.agents/AGENTS.md`, then `.claude/CLAUDE.md`; the first existing file wins at that level.
Project instructions require workspace trust.
Retain its normal personal context, plugins, MCP configuration, observers, and session logging; do not add `--no-foreign-personal-context`, `--disable-reminders`, or `--disable-web-tools` to simplify a launch.
Use repeatable `-c KEY=VALUE` for necessary per-run overrides, with JSON values; unknown or conflicting keys refuse startup.
See [Muse configuration and context](https://dev.meta.ai/docs/muse-code/configuration).

Run implementors and reviewers in normal agent mode.
A reviewer receives a no-repair role instruction, with normal tools available for verification.
Plan mode changes the deliverable and is not a substitute for review.

### 3. Minimal prompts and explicit skills

Pass the user's task and only missing execution-specific context: task scope, selected reviewer, approved access, ownership, concurrency, and coordination boundaries.
Include a task document when the user named it.
Treat applicable base prompts and global/project instruction files as available through each child's normal loading mechanisms.
Do not copy, summarize, repeat, or add reminders to read these instructions unless the user explicitly asks.
Apply this rule to launches, resumes, and nested delegation, while preserving each harness's own normal configuration.
Pass runtime decisions the child does not already have; do not restate inherited conventions.
Ask for an extra report field only when the workflow needs it and it is otherwise missing.

When delegating a skill, put the explicit invocation at the beginning of the child's user prompt:

| Child | Prompt form |
| --- | --- |
| Codex | `$co-review`, or the path-qualified mention `[$co-review](/absolute/skills/co-review/SKILL.md)` |
| Claude Code | `/co-review <task and supplied choices>` |
| Cursor Agent | `/co-review <task and supplied choices>` |
| Muse Code | Interactive `/<skill-name>` is documented; headless expansion is not verified, so use the calling workflow's embedded route unless native loading has been established |

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
| Muse Code | `muse skills list --enabled-only --workspace "$agent_workspace" --json`; inspect the chosen ID with `muse skills inspect` |

Follow symlinks and actual configuration; do not assume a directory exists or a parent-visible skill is registered in every child.
File existence alone does not prove that a particular CLI version will expand its command.
If native loading fails, report the failure and use the workflow's embedded alternative.
Do not install a missing skill, enable a disabled one, or disguise its contents as a workaround.

Muse discovers user skills in its config directory and `~/.agents/skills`, project skills in `.agents/skills`, and enabled plugin/built-in skills.
Compatibility discovery can also include personal and project Claude/Codex skill directories; inspect the actual catalog and its diagnostics.
`--enabled-only` includes `user-invocable-only` skills, which the model cannot load with `read_skill`; listing them does not establish headless invocation support.
An offline 1.4.7 echo probe returned `/co-review …` literally and emitted no skill activation record; do not assume Claude-style slash expansion in `muse exec`.
See [Muse skill discovery and activation](https://meta-models.github.io/muse-code-sdk/next/guides/extend/skills/).
This reference adds Muse as a child harness; the existing parent guides do not establish a Muse-native coordinator or native nested-skill route.

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
For Muse, define `agent_finish_muse_round` from its section before the invocation and set `agent_contributor_mode` to the approved `on` or `off` value.

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

#### Muse Code

Use `muse exec`, not interactive `muse resume`, for unattended work.
Pass the prompt as a quoted argument or through `--prompt-file /dev/stdin` on macOS/Linux; plain stdin alone is not the documented prompt interface.
The latter form was exercised with a quoted heredoc, preserving shell metacharacters without creating a prompt file.
Do not combine this stdin use with `--api-key-stdin`.

```bash
agent_exit=0
muse exec \
  --model "$agent_model" --reasoning-effort "$agent_effort" \
  --contributor-mode "$agent_contributor_mode" \
  --yolo --workspace "$agent_workspace" \
  --json --prompt-file /dev/stdin \
  > "$agent_log" 2>&1 <<'AGENT_PROMPT' || agent_exit=$?
<user task and supplied choices, using the embedded workflow when needed>
AGENT_PROMPT
agent_finish_muse_round
exit "$agent_exit"
```

For continuation, require the existing nonempty session file, read its UUID, and add `--session-id "$agent_session_id"` to the same `exec` invocation with new round files.
Repeat the model, effort, contributor mode, access, and workspace settings.
A local continuation recovered a token supplied only in the previous turn, using the same session UUID and a different selected effort.
Never substitute a run UUID, command ID, task ID, or display name for the saved session UUID.
Muse refuses a workspace mismatch; do not add `--allow-workspace-switch` unless the user actually assigned a different workspace.
Keep `--no-session-log` absent so continuation remains available.
See [Muse headless execution and continuation](https://dev.meta.ai/docs/muse-code/extending#headless-and-ci).

Additional controls from installed `exec --help`:

| Option | Use and boundary |
| --- | --- |
| `--parallel-tool-calls` / `--no-parallel-tool-calls` | Per-run model tool-call preference, not approval or speed control; both were accepted in successful probes, without measuring actual tool concurrency |
| `-c agents.execution_capacity=8` | Example of a JSON per-run setting; preserve normal defaults unless a concurrency choice requires an override |
| `--max-model-steps <N>` | Explicit work cap; reaching it fails the run rather than completing the task |
| `--output-schema <FILE>` | JSON schema for the final answer, Meta provider only; use only when the workflow requires structured output |
| `--max-tool-output-bytes <N>` | Tool-output cap (`0` or at least `9`); avoid arbitrary truncation for ordinary delegation |
| `--image <PATH>` | Repeatable local image attachment |
| `--worktree off\|create\|existing` | Muse-managed workspace choice; when the parent already chose a worktree, use that absolute directory directly |
| `--context-compaction-*` | Explicit compaction overrides; retain normal behavior unless requested |
| `--user-input-auto-resolve` | Offers the question tool but auto-cancels prompts; it cannot supply missing user decisions |

Do not copy interactive-only flags into `exec`; for example, root `--agents` is absent from the installed headless help.
The effort alias, JSON config example, heredoc prompt, and schema-constrained final response were exercised together.
Image attachment, worktree creation, compaction overrides, and live nested delegation were not exercised.

#### JSON finalizer for Muse

Muse 1.4.7 emits envelope records with `payload_type` and `payload`, not Claude/Cursor `type: result` events.
The root `session.run.linked` record identifies both the session (`stream.id`) and the run (`payload.run_stream.id`).
Filter terminal and model records to that exact session and run; nested tasks and observer completions are not the root answer.
`run.terminal.completed` carries the final text in `payload.text`, with `payload.terminal == "completed"`.
Keep this parser version-specific and recheck the event contract when upgrading.

Define this helper before the Muse invocation and call it only after the process exits:

```bash
agent_finish_muse_round() {
if ! jq -Rsr --argjson code "$agent_exit" '
  split("\n") as $lines
  | [$lines[] | fromjson? | objects] as $events
  | ([$events[] | select(.payload_type == "session.run.linked"
                         and .stream.kind == "session")] | first) as $root
  | ($root.stream.id // "") as $sid
  | ([$events[] | select(.payload_type == "session.run.linked"
                         and .stream.id == $sid)] | last) as $link
  | ($link.payload.run_stream.id // "") as $rid
  | [$events[] | select(.stream.id == $sid
                         and .payload.run_stream.id == $rid)] as $run
  | ([$run[] | select(.payload.kind == "run_terminal")] | last) as $end
  | ([$run[] | select(.payload_type == "run.model.configured")] | last) as $model
  | "RUN_STATUS: \(if $code == 0 and $sid != "" and $rid != ""
                       and $end.payload_type == "run.terminal.completed"
                       and $end.payload.terminal == "completed"
                       and (($end.payload.text // "") | length) > 0
                    then "COMPLETED" else "FAILED" end)\n"
    + "SESSION_ID: \($sid)\nMODEL: \($model.payload.model_id // "")\n"
    + "TERMINAL: \($end.payload.terminal // "missing")\n"
    + "REASON: \(($end.payload.reason // "") | tostring | .[0:2000])\n"
    + "EFFORT_DIAGNOSTIC: \([$lines[]
         | select(startswith("tbh: reasoning effort "))][-3:] | join("; ") | .[0:2000])\n\n"
    + ($end.payload.text // "No terminal report; use bounded failure extraction.")
' "$agent_log" > "$agent_response" 2>/dev/null; then
  printf 'RUN_STATUS: FAILED\nMuse final-output extraction failed.\n' > "$agent_response"
  agent_exit=1
  return
fi

agent_returned_session="$(sed -n 's/^SESSION_ID: //p' "$agent_response" | head -n 1)"
if test -s "$agent_session"; then
  if test "$(sed -n '1p' "$agent_session")" != "$agent_returned_session"; then
    printf '\nRUN_STATUS: FAILED\nSession identity missing or mismatched.\n' >> "$agent_response"
    agent_exit=1
  fi
elif test -n "$agent_returned_session"; then
  printf '%s\n' "$agent_returned_session" > "$agent_session"
fi
if grep -q '^RUN_STATUS: FAILED$' "$agent_response" && test "$agent_exit" -eq 0; then
  agent_exit=1
fi
}
```

An `EFFORT_DIAGNOSTIC` fallback requires reporting the actual setting and resolving the mismatch before further work, even with `RUN_STATUS: COMPLETED`.
The helper preserves a nonzero process status and treats a missing terminal answer or mismatched session as failure.
It extracts only terminal output, model/session metadata, and the bounded effort warning; never substitute streamed deltas or export the entire transcript into the parent context.
`muse export --session <uuid> --out <file>` is an offline audit surface, not the normal final-answer reader; its default export includes raw session content.

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

For Muse, the finalizer already extracts the root terminal reason, including failed or cancelled runs.
Documented process statuses are `0` for a completed turn, `1` for failure/cancellation or a step limit, `2` for usage errors, and `130`/`143` for SIGINT/SIGTERM.
A completed turn can still report an incomplete task.
When startup fails before JSON events, extract only a bounded plain diagnostic from the task log into the response file, such as the rejected effort or unavailable credential; never print the whole help/transcript.
Do not use the following Claude/Cursor/Codex event filter for Muse envelopes.

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
