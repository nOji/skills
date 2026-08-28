---
name: prepare-skill-release
description: Determine whether this repository's three publishable skills need regeneration, then prepare them from the canonical skill definitions, shared child-harness reference, and parent-agent guides when needed. Use after source changes and before committing or publishing a release.
metadata:
  internal: true
---

# Prepare the publishable skills

Treat `src/skill-definitions/`, `src/harness-cli.md`, `src/codex.md`, `src/claude.md`, `src/cursor.md`, and `src/explicit-only.openai.yaml` as canonical sources.
Treat `skills/` as generated, committed distribution output.

This skill is the development build step.
There is intentionally no committed build script or injection marker.
Different agent runs may produce different formatting or wording, but they must preserve the complete meaning of the canonical inputs.
Invoking this skill does not by itself mean that the generated files should be rewritten.

## Decide whether regeneration is needed

1. Inspect both uncommitted surfaces with `git status --short`, the relevant unstaged diff, and the relevant staged diff.
Include the canonical inputs, this release skill, and `skills/` in that inspection so pending source changes and pending generated changes are both visible.
2. Inspect Git history for the canonical inputs and `skills/` with `git log` and targeted `git show` or historical diffs as needed.
Establish whether the most recent committed semantic changes to the canonical inputs were already incorporated into the generated output.
Do not rely only on the current working-tree diff, because it cannot reveal an earlier committed source-only change.
3. Compare the current generated output with the complete canonical meaning.
Use history to establish chronology and intent, and use the current index and working tree to account for pending changes.
4. Regenerate only when a canonical input has changed semantically since the generated output was last aligned, when the generated structure is missing or stale, or when the user explicitly asks to force regeneration.
5. Do not regenerate merely because this skill was invoked, because only repository or release-workflow documentation changed, or because an agent could express the same canonical meaning with different formatting.
Avoid a formatting-only rewrite of already aligned generated files.
6. If regeneration is unnecessary, leave `skills/` byte-for-byte unchanged and report the history and uncommitted-state evidence that supported that decision.

## Generate only when needed

1. Read all three files under `src/skill-definitions/` and read `src/harness-cli.md`, `src/codex.md`, `src/claude.md`, `src/cursor.md`, and `src/explicit-only.openai.yaml` completely.
2. Inspect the current `skills/` output for useful continuity, but treat canonical sources as authoritative when they disagree.
3. For each of `co-implement`, `co-review`, and `orchestrate`, write `skills/<name>/SKILL.md` as one coherent skill containing the complete canonical definition and the complete shared harness reference exactly once.
4. Ensure each generated `SKILL.md` tells Codex, Claude Code, or Cursor to read exactly its one adjacent parent guide: Codex reads `codex.md`, Claude Code reads `claude.md`, and Cursor reads `cursor.md`.
5. Copy `src/codex.md`, `src/claude.md`, and `src/cursor.md` into each generated skill directory as `codex.md`, `claude.md`, and `cursor.md` without blending their parent-agent instructions into the shared child-harness reference.
6. Preserve valid YAML frontmatter and ensure its `name` matches the output directory.
7. Integrate the harness as an embedded reference section, adjust heading levels and internal references as needed, and remove any source-only release commentary.
8. Preserve the distinction between the agent loading the skill and the child harness it invokes.
Do not move child-Codex, child-Claude, or child-Cursor launch instructions into a parent guide merely because they mention that product.
9. Write `src/explicit-only.openai.yaml` to `skills/<name>/agents/openai.yaml` for each public skill.
10. Remove stale files from each generated skill directory, especially standalone `harness-cli.md` files, source markers, temporary files, and symlinks.
11. Confirm the publishable tree contains exactly the three public `SKILL.md` entrypoints and that each skill directory contains only `SKILL.md`, `codex.md`, `claude.md`, `cursor.md`, and `agents/openai.yaml`.
12. Run the available Agent Skills validator on each public skill when possible.
13. Run `npx skills@latest add . --list` when network access is available and confirm normal discovery contains exactly `co-implement`, `co-review`, and `orchestrate`.
14. Review the generated changes for semantic completeness, broken local links, duplicated harness content, loader-versus-child instruction leakage, and accidental loss of source instructions.
15. Summarize why regeneration was needed, what was generated, the checks performed, and any agent judgment that could make a future run differ.

Do not modify the canonical source files while preparing a release unless the user separately asks for source changes.
Never commit or push unless the user explicitly asks.
