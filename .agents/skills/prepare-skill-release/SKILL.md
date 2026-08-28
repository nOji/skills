---
name: prepare-skill-release
description: Prepare this repository's three publishable skills by using agent judgment to combine each canonical skill definition with the canonical harness CLI reference. Use after source changes and before committing or publishing a release.
metadata:
  internal: true
---

# Prepare the publishable skills

Treat `src/skill-definitions/`, `src/harness-cli.md`, and `src/explicit-only.openai.yaml` as canonical sources.
Treat `skills/` as generated, committed distribution output.

This skill is the development build step.
There is intentionally no committed build script or injection marker.
Different agent runs may produce different formatting or wording, but they must preserve the complete meaning of the canonical inputs.

1. Read all three files under `src/skill-definitions/` and read `src/harness-cli.md` and `src/explicit-only.openai.yaml` completely.
2. Inspect the current `skills/` output for useful continuity, but treat canonical sources as authoritative when they disagree.
3. For each of `co-implement`, `co-review`, and `orchestrate`, write `skills/<name>/SKILL.md` as one coherent skill containing the complete canonical definition and the complete harness reference exactly once.
4. Preserve valid YAML frontmatter and ensure its `name` matches the output directory.
5. Integrate the harness as an embedded reference section, adjust heading levels and internal references as needed, and remove any source-only release commentary.
6. Write `src/explicit-only.openai.yaml` to `skills/<name>/agents/openai.yaml` for each public skill.
7. Remove stale files from each generated skill directory, especially standalone `harness-cli.md` files, source markers, temporary files, and symlinks.
8. Confirm the publishable tree contains exactly the three public `SKILL.md` entrypoints and that each skill directory contains only `SKILL.md` and `agents/openai.yaml`.
9. Run the available Agent Skills validator on each public skill when possible.
10. Run `npx skills@latest add . --list` when network access is available and confirm normal discovery contains exactly `co-implement`, `co-review`, and `orchestrate`.
11. Review the generated changes for semantic completeness, broken local links, duplicated harness content, and accidental loss of source instructions.
12. Summarize what was generated, the checks performed, and any agent judgment that could make a future run differ.

Do not modify the canonical source files while preparing a release unless the user separately asks for source changes.
Never commit or push unless the user explicitly asks.
