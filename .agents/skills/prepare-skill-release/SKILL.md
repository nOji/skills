---
name: prepare-skill-release
description: Check whether the three publishable skills are stale, and regenerate them from canonical definitions, shared harness and review modules, and parent guides only when needed for an explicitly requested release preparation.
metadata:
  internal: true
---

# Prepare the publishable skills

This is the repository's agent-driven build step.
Run it only when release preparation is requested; source editing alone does not authorize generation.
There is no committed build script or injection marker.
Formatting may vary, but the complete canonical meaning must survive.

## Canonical inputs and composition

| Input | Purpose |
| --- | --- |
| `src/skill-definitions/<name>.md` | Skill-specific roles, choices, workflow, and completion rules |
| `src/harness-cli.md` | Child CLI commands, capabilities, invocation, records, and recovery |
| `src/review-protocol.md` | Shared evidence, dialogue, ownership, and review completion |
| `src/codex.md`, `src/claude.md`, `src/cursor.md` | Instructions for the agent loading the skill |
| `src/explicit-only.openai.yaml` | Public Codex invocation policy |

Each public `SKILL.md` embeds its definition and the required shared modules exactly once:

| Public skill | Harness reference | Review protocol |
| --- | --- | --- |
| co-implement | Include | Include; supervisor sign-off |
| co-review | Include | Include; mutual agreement |
| orchestrate | Include | Include; assigned reviews use mutual agreement |

The distribution must work without access to `src/` or another installed public skill.
Orchestrate's optional use of an installed co-review is an optimization; its embedded review route remains complete.

## Decide whether regeneration is needed

1. Inspect Git status, staged and unstaged diffs, and relevant untracked inputs.
   Include all canonical inputs, this preparation skill, and `skills/`.
2. Inspect their Git history to determine whether committed source changes were incorporated into the distribution.
   A clean working tree does not establish that generated output is current.
3. Compare current generated content with the full canonical meaning, including both review completion modes and orchestration's optional routes.
4. Regenerate only for semantic drift, missing or stale structure, a changed required module composition, or an explicit request to force regeneration.

If already aligned, leave `skills/` byte-for-byte unchanged and report the supporting history and current-state evidence.
Do not rewrite solely for formatting, unrelated documentation changes, or because this skill was invoked.

## Generate when needed

1. Read every canonical input completely.
   Use existing generated output for continuity, with canonical sources authoritative on disagreement.
2. Compose each standalone `skills/<name>/SKILL.md` from its definition and required modules.
   Keep each module exactly once, with coherent heading levels and working internal references.
   Preserve full command examples, permission behavior, prompt boundaries, ownership protections, and completion rules.
3. Keep the role mapping unambiguous.
   Co-implement uses supervisor sign-off; co-review and orchestration reviews use mutual agreement.
   Do not turn the orchestrator into an implementor or technical reviewer.
4. Ensure the entrypoint directs the loading agent to exactly its matching adjacent parent guide.
   Copy the three parent guides beside every entrypoint without merging their tool instructions into the child harness module.
5. Preserve valid YAML frontmatter with a name matching the directory.
   Copy `src/explicit-only.openai.yaml` to each `agents/openai.yaml`.
6. Remove stale generated files, standalone module copies, source-only commentary, temporary files, and symlinks.
   Do not leave links back to canonical source paths.

Each public directory must contain only `SKILL.md`, `codex.md`, `claude.md`, `cursor.md`, and `agents/openai.yaml`.
Normal discovery must expose exactly co-implement, co-review, and orchestrate; this internal skill stays outside that public surface.

## Review the result

Check semantic completeness against every input, module duplication, local links and anchors, YAML identity, and parent-versus-child instruction placement.
Check that orchestration retains per-task reviewer choices, approved parallel scheduling, both review routes, agreement before dependent work, and serialized shared writes.
Run an available Agent Skills validator and, when network access is available, `npx skills@latest add . --list` to confirm the three public entrypoints.
Do not substitute code builds, linters, or application tests for these document and distribution checks.

Report why generation was needed or skipped, what changed, checks performed, and any unresolved limitation.
Do not modify canonical sources during preparation unless separately requested.
Commit and push only when explicitly authorized.
