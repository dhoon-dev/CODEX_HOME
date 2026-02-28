# Python Toolchain Guide

## 1) Repository-first detection (do not override existing choices)

Follow the repository's existing Python toolchain when it is clearly indicated:

- If `uv.lock` exists: use uv (`uv sync`, `uv run ...`).
- If `poetry.lock` exists: use Poetry (`poetry install`, `poetry run ...`).
- If `Pipfile` exists: use Pipenv (`pipenv install`, `pipenv run ...`).
- If `requirements.txt` is the primary spec and no lockfile-based tool is
  indicated: use `python -m pip install -r requirements.txt` (plus
  `-c constraints.txt` when constraints are present).

## 2) Default commands (fallback-only)

- For project commands in ambiguous repos, prefer uv execution: `uv run <command>`.
- For one-off tools with no dependency assumptions, use uvx (for example:
  `uvx ruff check .`, `uvx black .`, `uvx mypy .`).
- Prefer existing `make test`, `tox`, `nox`, or CI commands when present.

## 3) Safety rules (to avoid breaking team/OSS repos)

- Do not introduce a new package manager (e.g., do not add Poetry/uv
  lockfiles) unless the project explicitly wants it.
- Explain before changing dependencies or formatting large portions of the codebase.
- Keep changes minimal and validate with the repo's existing test/lint entrypoints.
