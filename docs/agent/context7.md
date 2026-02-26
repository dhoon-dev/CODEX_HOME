# Context7 Query Quality Playbook

This document defines how to produce high-signal queries to the Context7 MCP server and how to validate what comes back.
The goal is to (a) target the exact pinned version used by the repo and (b) avoid hallucinated APIs.

## 1) Always collect the 4 inputs before querying
Before making a Context7 query, extract:

1. **Dependency identity**: package/library name (and submodule/feature if relevant)
2. **Pinned version**: exact version / tag / commit / baseline as enforced by the project
3. **Topic**: what you need (API reference, configuration, migration, auth, error, example)
4. **Target surface**: symbol / header / module / option name / exact error message

If any of these is unclear, resolve it from project specifications first (lockfiles/manifests/scripts). If still ambiguous, ask.

## 2) Version resolution rules by ecosystem

### Node.js / TypeScript
Prefer these sources (highest signal first):
- `pnpm-lock.yaml` (if present) and `packageManager` field in `package.json`
- `package-lock.json`
- `yarn.lock`
- `bun.lockb`
- `package.json` dependency ranges (only if no lockfile exists)

If multiple lockfiles exist or the package manager is unclear, treat it as ambiguous: follow the build scripts/CI commands, or ask.

### Python
Prefer these sources:
- `uv.lock`
- `poetry.lock`
- `pdm.lock`
- `requirements*.txt` and `constraints*.txt` (respect constraints)
- `pyproject.toml` dependency ranges (only if no lock/requirements exist)

If both a lockfile and requirements files exist, follow the tool invoked by scripts/CI.

### Go
- `go.mod` (module versions) and `go.sum` (integrity)
- If using vendoring, treat `vendor/` as the implemented truth; confirm against vendored source.

### Rust
- `Cargo.lock` (workspace-locked versions)
- `Cargo.toml` ranges only if no lockfile is present (rare for apps)

### Ruby
- `Gemfile.lock`

### PHP
- `composer.lock`

### .NET
- `packages.lock.json`
- `*.csproj` / `Directory.Packages.props` versions (as actually enforced)

### JVM (Gradle/Maven)
- Prefer lockfiles if present (`gradle.lockfile`, dependency lock state)
- Otherwise follow `build.gradle(.kts)` / `pom.xml` and any version catalogs (`libs.versions.toml`)

### C / C++
The critical step is identifying *how* the dependency is obtained, then extracting the pinned ref.
Treat moving targets (e.g., FetchContent `GIT_TAG` set to a branch like `main`/`master`) as unpinned; call it out and ask before relying on docs.

Prefer these sources (highest signal first):
- **vcpkg**
  - `vcpkg.json` (manifest) + `vcpkg-configuration.json` (registry/builtin-baseline) and any lock/baseline files used by the repo
- **Conan**
  - `conan.lock` (if present), otherwise `conanfile.py` / `conanfile.txt` + profiles used by CI
- **CMake FetchContent / ExternalProject**
  - `CMakeLists.txt` entries like `GIT_TAG`, `GIT_REPOSITORY`, `URL`, `URL_HASH`
  - CPM.cmake (`CPMAddPackage`) version/tag fields
- **git submodules**
  - `.gitmodules` + the pinned commit recorded in the repo
- **vendored source**
  - `third_party/` / `vendor/` directory contents (treat as truth; upstream docs may not match)
- **system package managers**
  - Homebrew/Apt/DNF/etc. only if the repo explicitly depends on system packages (then read CI scripts to learn the expected version)

Also collect build context because it affects API/ABI:
- compiler (MSVC/GCC/Clang), platform (Windows/macOS/Linux), and language standard (`-std=c++20`, etc.)
- build system (CMake/Meson/Bazel) and enabled features/options

## 2.5) Verify the resolved version (when ambiguous)
If the pinned version is unclear (no lockfile, multiple lockfiles, vendored patches, or moving git refs), confirm the *resolved* version locally before relying on docs.

- Node: use package-manager-specific equivalent commands (for example: `npm ls <pkg>`, `pnpm why <pkg>`, `yarn why <pkg>`) and/or inspect `node_modules/<pkg>/package.json` (when `node_modules` exists).
- Python: prefer `python -m pip show <pkg>` (or inspect the installed distribution metadata) and confirm `__version__` when available.
- C/C++: confirm vcpkg baseline/lock, conan.lock, FetchContent `GIT_TAG`/hash, or the submodule commit; for vendored deps, treat `third_party/` as authoritative.

## 3) Query formulation templates

### Minimal template
`"<dependency> <pinned-version> <topic> <target-surface>"`

### When errors drive the work
Include the exact error message (or its distinctive fragment):
`"<dependency> <version> <error message> <symbol>"`

### C / C++ specific template
Because ABI, compiler, and build options matter:
`"<library> <version/tag> <header or symbol> <topic> <compiler> <c++standard> <build-system> <package-manager>"`

## 4) Tighten in at most 2 follow-ups
Start with one query.
Then refine with up to 2 follow-ups:
- add the exact symbol/header/option
- add the exact error string
- disambiguate major versions explicitly

Avoid shotgun querying.

## 5) Validate what Context7 returns
Before committing code based on docs:
- Confirm the pinned version matches what the project uses (lockfile/tag/commit).
- Cross-check the surface locally when possible:
  - TS: installed type defs in `node_modules` / `.d.ts`
  - Python: installed package docs/source in venv/site-packages
  - C/C++: the installed headers and exported symbols (headers are the source of truth)
- If the repo vendors/patches a dependency, treat local source as authoritative and call out divergences.

## 6) If Context7 cannot confirm
Use this fallback sequence:
1. Inspect local installed package source/types/headers
2. Check repo-local docs (README, docs/, comments near usage)
3. Ask a clarifying question (if ambiguity remains)
4. Only then use general web search (if allowed/needed)

## 7) Output expectations
When you relied on Context7, your response must mention:
- the dependency name and pinned version/tag/commit you targeted
- the key API/config surface you used (symbols/options)
- any mismatch you detected between docs and the repo (and what you followed)
