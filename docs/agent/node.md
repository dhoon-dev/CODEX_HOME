# Node.js Toolchain Guide

## 1) Repository-first detection
Follow the repository's package manager based on lockfiles and scripts:
- If `pnpm-lock.yaml` exists: use pnpm (`pnpm install`, `pnpm test`, `pnpm lint`).
- If `yarn.lock` exists: use yarn (`yarn install`, `yarn test`, `yarn lint`).
- If `bun.lockb` exists: use bun (`bun install`, `bun test`, `bun run lint`).
- If `package-lock.json` exists: use npm (`npm ci`/`npm install`, `npm test`, `npm run lint`).
- If multiple lockfiles exist, follow CI/build scripts to resolve the intended toolchain.

## 2) Default commands (fallback-only)
- Prefer existing package scripts over ad-hoc commands (`<pm> run <script>`).
- Install dependencies with the lockfile-matching package manager.
- When no package manager is clearly established and a one-off tool is needed, use `npx <tool>`.

## 3) Safety rules
- Do not switch package managers or regenerate lockfiles unless explicitly requested.
- Avoid global installs for project-scoped tasks.
- Explain before dependency upgrades or large formatting changes.
