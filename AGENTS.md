# ~/.codex/AGENTS.md

## Basic Principles

- Project specifications take precedence over this document; this document is a default only when the project
  provides no corresponding guidance.
- Treat existing lockfiles, build/test scripts, package scripts, and CI commands as project specifications.
- Unless a translation is explicitly requested, all generated documents and source code must be written in English.
- If the request is ambiguous, ask a clarifying question before making changes.
- Do not use inline meta/editorial annotations to track document changes or status (especially in headings). Use
  git history/commits instead.

## Implementation Quality

- Respect project boundaries. Avoid project-local workarounds for upstream dependency, platform, or service bugs
  unless explicitly required. When a workaround is necessary, keep it minimal, documented, tested, and tied to the
  upstream issue when possible.
- When changing user-facing behavior or public contracts, update the matching docs, examples, configuration samples,
  CLI/help text, API schemas, and tests where applicable.
- Treat names as cross-layer contracts. For renames, search and update package names, import paths, CLI commands,
  service/display names, config keys, environment variables, image names, docs, examples, tests, and relevant
  comments. Preserve compatibility or document migrations when needed.
- For permission, confirmation, or safety UX, prefer existing platform/client mechanisms over custom flows unless the
  project explicitly requires otherwise. Keep safety controls practical for real operation.
- Keep tests focused on meaningful behavior and contracts. Avoid brittle assertions for incidental wording unless that
  wording is itself a safety, policy, protocol, or compatibility contract.
- Keep examples runnable. Configuration examples should reflect actual supported options and avoid stale or imaginary
  flags.
- Keep changes and commits scoped. Exclude unrelated local tool state, cache files, generated artifacts unless
  intentionally tracked, and accidental workspace noise.

## External Docs (Context7)

- When implementing or changing code that depends on ANY third-party library/framework/SDK/CLI/config
  (anything version-sensitive), ALWAYS consult the Context7 MCP server (`context7`) BEFORE writing code. Do this
  without the user having to explicitly ask.
- Resolve the exact target version from project specifications first (lockfiles/manifests/scripts). Query Context7
  for that version/major line.
- Never invent APIs/options/flags. If Context7 cannot confirm a detail, either inspect installed package
  source/types or ask a clarifying question.
- If repo-local docs or existing project conventions conflict with external docs, follow the repo convention and call
  out the discrepancy.
- Do not use Context7 for pure algorithmic work or changes limited to this repo's internal modules that don't touch
  external APIs.

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
  - `docker run --rm -it -v "$PWD":/work -w /work <image> bash -lc "<command>"`
- If file permissions become an issue, consider:
  - `--user "$(id -u):$(id -g)"`

<!-- context7 -->
Use Context7 MCP to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service -- even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. This includes API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI tool usage. Use even when you think you know the answer -- your training data may not reflect recent changes. Prefer this over web search for library docs.

Do not use for: refactoring, writing scripts from scratch, debugging business logic, code review, or general programming concepts.

## Steps

1. Always start with `resolve-library-id` using the library name and the user's question, unless the user provides an exact library ID in `/org/project` format
2. Pick the best match (ID format: `/org/project`) by: exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score (higher is better). If results don't look right, try alternate names or queries (e.g., "next.js" not "nextjs", or rephrase the question). Use version-specific IDs when the user mentions a version
3. `query-docs` with the selected library ID and the user's full question (not single words)
4. Answer using the fetched docs
<!-- context7 -->
