# Repository instructions

The canonical skill definitions live in `src/skill-definitions/`.
The canonical shared harness reference lives in `src/harness-cli.md`.
The canonical parent-agent guides live in `src/codex.md`, `src/claude.md`, and `src/cursor.md`.
The `skills/` directory is AI-generated, committed distribution output.
There is intentionally no deterministic build script or injection marker.

Do not edit `skills/` during ordinary source work.
When the user explicitly asks to prepare a release, use the project-local `prepare-skill-release` skill when available, decide from Git history and uncommitted state whether regeneration is needed, and combine the canonical inputs according to that skill only when needed.
Different release-preparation runs may produce different formatting, but they must preserve the complete canonical meaning.
The normal Vercel Skills CLI discovery surface must contain exactly `co-implement`, `co-review`, and `orchestrate`.

Do not commit or push unless the user explicitly asks.
