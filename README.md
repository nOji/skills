# Agent Harness Skills

Three user-invoked [Agent Skills](https://agentskills.io/) for delegating coding work to Codex, Claude Code, or Cursor Agent.

This repository is designed for installation through Vercel's [`skills` CLI](https://github.com/vercel-labs/skills).
It is a GitHub-hosted skill collection, not an npm package and not a native Claude Code plugin.

> [!IMPORTANT]
> Before publishing, replace every `nOji/skills` placeholder in this README with the final GitHub repository slug.

## Included skills

| Skill | Purpose |
| --- | --- |
| [`co-implement`](./skills/co-implement/SKILL.md) | Delegates one implementation to a child coding-agent CLI, then reviews and iterates on the result. |
| [`co-review`](./skills/co-review/SKILL.md) | Requests an independent review, verifies each finding, and iterates until both agents agree or a user decision is required. |
| [`orchestrate`](./skills/orchestrate/SKILL.md) | Runs an ordered queue of delegated tasks in user-approved batches without acting as the implementer or reviewer. |

Each skill is designed for explicit user invocation, and the bundled Codex policy disables implicit invocation.
Invoke it using the syntax supported by your agent, such as `$co-review` in Codex or `/co-review` in Claude Code.

## Installation

Inspect the three available skills without installing them:

```bash
npx skills@latest add nOji/skills --list
```

Open the interactive skill and agent picker:

```bash
npx skills@latest add nOji/skills
```

Install one skill:

```bash
npx skills@latest add nOji/skills@co-review
```

Install all three globally for Codex without prompts:

```bash
npx skills@latest add nOji/skills --skill '*' --agent codex --global --yes
```

Project installation is the default when `--global` is omitted.
Installed skills can be refreshed later with:

```bash
npx skills@latest update
```

You can also use one skill for a single session without permanently installing it:

```bash
npx skills@latest use nOji/skills@co-review --agent codex
```

## Runtime requirements

The Vercel installer supports many agents, but these particular workflows have a narrower runtime contract.

- Use macOS, Linux, or WSL with Git, a POSIX-compatible shell, `jq`, and standard Unix utilities available.
- Use Node.js 22.20 or newer with npm to run the current `skills@latest` CLI.
- Install and authenticate at least one supported child harness: Codex, Claude Code, or Cursor Agent.
- The child harness may use the same product as the supervising agent.
- Use current child-harness CLI releases that accept the model-listing, permission, and resume options verified by the shared harness reference.
- Allow the supervising agent to manage long-running child processes asynchronously and, when the parent is Codex, request its documented host-shell approval.

The skills perform live availability and model checks before delegating.
An installed skill cannot provide credentials, usage quota, or a missing coding-agent CLI.

## Repository design

Canonical maintainer sources and public installation files are deliberately separate:

```text
src/
  codex.md                       # Guidance only for a Codex parent agent
  claude.md                      # Guidance only for a Claude Code parent agent
  cursor.md                      # Guidance only for a Cursor parent agent
  harness-cli.md
  explicit-only.openai.yaml
  skill-definitions/
    co-implement.md
    co-review.md
    orchestrate.md

skills/                              # Generated and committed
  co-implement/
    SKILL.md                         # Definition plus embedded harness
    codex.md
    claude.md
    cursor.md
    agents/openai.yaml
  co-review/
    SKILL.md                         # Definition plus embedded harness
    codex.md
    claude.md
    cursor.md
    agents/openai.yaml
  orchestrate/
    SKILL.md                         # Definition plus embedded harness
    codex.md
    claude.md
    cursor.md
    agents/openai.yaml
```

`src/harness-cli.md` is the single source of truth for shared command forms.
The three parent-agent guides contain only behavior that changes because Codex, Claude Code, or Cursor loaded the skill; commands for invoking those products as child harnesses remain in `src/harness-cli.md`.
Before a release, an AI agent first compares Git history and uncommitted changes to determine whether the generated output is stale.
When regeneration is needed, it combines each canonical definition with the canonical harness inside the generated `SKILL.md` and copies all three parent-agent guides beside it.
Each generated entrypoint directs Codex, Claude Code, or Cursor to read exactly the one parent guide that matches itself.
There is intentionally no committed build script or injection marker, so formatting may vary between release-preparation runs while the complete source meaning must remain intact.
The publishable distribution contains exactly the three skills shown above.

## Maintainer workflow

1. Edit `src/harness-cli.md`, one of the three parent-agent guides, `src/explicit-only.openai.yaml`, or a file under `src/skill-definitions/`.
2. Invoke the repository owner's project-local `prepare-skill-release` skill.
3. Let that agent inspect both Git history and uncommitted changes and skip regeneration when the distribution is already semantically aligned.
4. When regeneration is needed, let the agent combine the canonical inputs semantically, copy the parent-agent guides, and replace the three generated skill directories.
5. Review the canonical and generated diff carefully because generation is intentionally agent-driven rather than byte-deterministic.
6. Confirm `npx skills@latest add . --list` reports exactly the three public skills.
7. Commit both the source changes and any regenerated `skills/` output.

During ordinary development, never edit `skills/` directly.

## Publish this repository

No npm publication, marketplace submission, Changesets setup, or plugin manifest is required.
Once the repository is public on GitHub, users can install directly from its `nOji/skills` slug.

1. Choose the final GitHub owner and repository name, then replace the README placeholders.
2. In GitHub, create an empty public repository without generating another README, license, or `.gitignore`.
3. Initialize and push this working draft when you are ready:

```bash
git init -b main
git add .
git commit -m "Initial public release"
git remote add origin git@github.com:nOji/skills.git
git push -u origin main
```

4. Verify remote discovery:

```bash
npx skills@latest add nOji/skills --list
```

The result should contain exactly `co-implement`, `co-review`, and `orchestrate`.

5. Smoke-test an actual copy install in a disposable directory:

```bash
skill_test_dir="$(mktemp -d)"
cd "$skill_test_dir"
git init
npx skills@latest add nOji/skills@co-review --agent codex --copy --yes
```

6. Add a short GitHub description and topics such as `agent-skills`, `codex`, `claude-code`, and `cursor`.
7. Keep the included MIT license, and retain any third-party notices if substantive upstream material is added later.

Tags, GitHub Releases, a changelog, and branch protection are useful later, but none is required for the first usable public release.

Skills can appear on [skills.sh](https://skills.sh/) after the public repository receives an install; separate registration is not required.

## License

[MIT](./LICENSE)
