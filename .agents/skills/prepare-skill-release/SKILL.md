---
name: prepare-skill-release
description: Prepare every canonical skill for an explicitly requested release by discovering source definitions, checking publication coverage and staleness, and regenerating only missing or outdated standalone distributions.
metadata:
  internal: true
---

# Prepare the publishable skills

This is the repository's agent-driven build step.
Run it only when release preparation is requested; source editing alone does not authorize generation.
There is no committed build script or injection marker.
Formatting may vary, but the complete canonical meaning must survive.

## Discover the complete source inventory

Enumerate every Markdown definition under `src/skill-definitions/`, including untracked files:

```bash
rg --files --hidden --no-ignore -g '*.md' src/skill-definitions
```

Every definition in this directory is a publishable skill.
Internal maintainer skills live outside this canonical directory.
Read every discovered definition and validate its YAML frontmatter, unique `name`, and agreement between that name and its filename stem.
Use the canonical name for `skills/<name>/`; do not rename a skill during generation.
Report invalid or ambiguous definitions explicitly instead of silently excluding them or claiming complete coverage.

Build a temporary coverage inventory recording each source path, canonical name, required shared modules, output directory, and freshness decision.
Derive the expected public skill set from this inventory, never from a fixed count, a remembered list, README examples, or the existing `skills/` directories.
A newly added source must be included even when no generated directory or skill-specific example exists yet.

## Canonical inputs and composition

| Input | Purpose |
| --- | --- |
| Every definition under `src/skill-definitions/` | Skill-specific roles, choices, workflow, and completion rules |
| `src/harness-cli.md` | Child CLI commands, capabilities, invocation, records, and recovery |
| `src/review-protocol.md` | Shared evidence, dialogue, ownership, and review completion |
| `src/codex.md`, `src/claude.md`, `src/cursor.md` | Instructions for the agent loading the skill |
| `src/explicit-only.openai.yaml` | Public Codex invocation policy |

Each public `SKILL.md` embeds its complete definition and each required shared module exactly once.
Determine module composition from that definition's references, roles, and completion rules:

- Include the harness reference when the skill launches or resumes child agents or refers to that reference.
- Include the review protocol when the definition invokes the shared protocol or its supervisor-sign-off or mutual-agreement modes.
- Preserve a definition's self-contained review workflow when it uses its own roles and completion rules; reviewing work alone does not require the shared protocol.

For the current workflows, co-implement uses the shared supervisor-sign-off mode; co-review and orchestration reviews use shared mutual agreement.
Delegate-ui uses its own parent-led functional review and requires the harness reference without the shared review protocol.
These are composition examples, not an allowlist; derive any additional skill's requirements from its source.

The distribution must work without access to `src/` or another installed public skill.
Orchestrate's optional use of an installed co-review is an optimization; its embedded review route remains complete.

## Decide whether regeneration is needed

1. Inspect Git status, staged and unstaged diffs, and relevant untracked inputs.
   Include every inventoried definition, the shared inputs, this preparation skill, and `skills/`.
2. Inspect their Git history to determine whether committed source changes were incorporated into the distribution.
   A clean working tree does not establish that generated output is current.
3. Compare the expected canonical names with the generated names before checking content.
   Missing output for an added source, or extra output after a source removal or rename, is release drift.
4. Compare each generated skill with its complete canonical meaning and required module composition, including its review completion rules and optional routes.
5. Generate missing skills and refresh only outputs with semantic drift, missing or stale structure, a changed required module composition, or an explicit request to force regeneration.
   Leave already aligned skill files byte-for-byte unchanged.

If the complete inventory and all generated content are aligned, leave `skills/` byte-for-byte unchanged and report the supporting history and current-state evidence.
Do not rewrite solely for formatting, unrelated documentation changes, or because this skill was invoked.

## Generate when needed

1. Read every canonical input completely.
   Use existing generated output for continuity, with canonical sources authoritative on disagreement.
2. Compose each missing or stale standalone `skills/<name>/SKILL.md` from its definition and required modules.
   Keep each module exactly once, with coherent heading levels and working internal references.
   Preserve full command examples, permission behavior, prompt boundaries, ownership protections, and completion rules.
3. Keep the role mapping unambiguous.
   Co-implement uses supervisor sign-off; co-review and orchestration reviews use mutual agreement.
   Do not turn the orchestrator into an implementor or technical reviewer.
   For delegate-ui, preserve the parent's ownership of feature logic and functional review and the child's ownership of presentation and visual design.
   Preserve additional skills' roles as defined by their canonical sources.
4. Ensure the entrypoint directs the loading agent to exactly its matching adjacent parent guide.
   Copy the three parent guides beside every entrypoint without merging their tool instructions into the child harness module.
5. Preserve valid YAML frontmatter with a name matching the source inventory and output directory.
   Copy `src/explicit-only.openai.yaml` to each `agents/openai.yaml`.
6. Remove stale generated files and directories, standalone module copies, source-only commentary, temporary files, and symlinks.
   Confirm stale output ownership from Git status and history, preserving unrelated work.
   Do not leave links back to canonical source paths.

Each public directory must contain only `SKILL.md`, `codex.md`, `claude.md`, `cursor.md`, and `agents/openai.yaml`.
Normal discovery must expose exactly the canonical names from the complete source inventory; this internal preparation skill stays outside that public surface.

## Review the result

Compare source and generated name sets and report any missing or extra skills; equal counts alone do not establish coverage.
Check every generated skill against all its required inputs for semantic completeness, module duplication, local links and anchors, YAML identity, and parent-versus-child instruction placement.
Check that orchestration retains per-task reviewer choices, approved parallel scheduling, both review routes, agreement before dependent work, and serialized shared writes.
Check that delegate-ui retains parent-owned prerequisites, one batched UI handoff, functional review, design autonomy, and completion of the integrated feature.
Apply each additional definition's own completion requirements to its generated result.

Run an available Agent Skills validator for every public skill and for this preparation skill when modified.
When network access is available, run `npx skills@latest add . --list` and compare the discovered name set with the canonical inventory, reporting any missing or unexpected entrypoint.
Do not substitute code builds, linters, or application tests for these document and distribution checks.

Report the complete inventory, why each output was generated, refreshed, or left unchanged, checks performed, and any unresolved limitation.
Do not claim complete release preparation while a canonical definition remains unaccounted for or a discovery mismatch remains unresolved.
Do not modify canonical sources during preparation unless separately requested.
Commit and push only when explicitly authorized.
