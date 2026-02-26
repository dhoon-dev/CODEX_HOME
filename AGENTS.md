# ~/.codex/AGENTS.md

## Basic Principles
- Project specifications take precedence over this document; this document is a default only when the project provides no corresponding guidance.
- Treat existing lockfiles, build/test scripts, package scripts, and CI commands as project specifications.
- Unless a translation is explicitly requested, all generated documents and source code must be written in English.
- If the request is ambiguous, ask a clarifying question before making changes.

## External Docs (Context7)
- When implementing or changing code that depends on ANY third-party library/framework/SDK/CLI/config (anything version-sensitive),
  ALWAYS consult the Context7 MCP server (`context7`) BEFORE writing code. Do this without the user having to explicitly ask.
- Resolve the exact target version from project specifications first (lockfiles/manifests/scripts). Query Context7 for that version/major line.
- Never invent APIs/options/flags. If Context7 cannot confirm a detail, either inspect installed package source/types or ask a clarifying question.
- If repo-local docs or existing project conventions conflict with external docs, follow the repo convention and call out the discrepancy.
- Do not use Context7 for pure algorithmic work or changes limited to this repo’s internal modules that don’t touch external APIs.

## Context7 Query Quality (read before you query)
- Read a Context7 playbook before forming queries:
  - Prefer repo-local: ./docs/agent/context7.md
  - Otherwise use global: $CODEX_HOME/docs/agent/context7.md

## Context7 Output Hygiene
- When you relied on Context7, include in your response:
  - the library/package name and the pinned version you targeted
  - the key API/config surface you used (names of functions/classes/options)

## Toolchain Docs Index (read before deciding commands)
- Prefer repo-local docs when present:
  - ./docs/agent/python.md
  - ./docs/agent/node.md
  - ./docs/agent/cpp.md
  - ./docs/agent/context7.md
- Otherwise, use global docs under CODEX_HOME:
  - $CODEX_HOME/docs/agent/python.md
  - $CODEX_HOME/docs/agent/node.md
  - $CODEX_HOME/docs/agent/cpp.md
  - $CODEX_HOME/docs/agent/context7.md

## Defaults (fallback-only)
- Python tooling: follow python.md (prefer `uv run` for project env, `uvx` for one-off tools).
- Node.js tooling: follow node.md (prefer package scripts/lockfile toolchain; otherwise `npx`).
- C/C++ tooling: follow cpp.md (prefer CMake out-of-source builds when CMake is present).

## Missing Tools
- If a required tool is not installed locally, prefer running it via Docker.
- If Docker is unavailable, install the tool via the standard method for the environment and document what changed.

## Docker Conventions
- Default pattern:
  - docker run --rm -it -v "$PWD":/work -w /work <image> bash -lc "<command>"
- If file permissions become an issue, consider:
  - --user "$(id -u):$(id -g)"