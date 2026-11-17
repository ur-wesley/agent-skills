# Agent Skills

Personal [Agent Skills](https://agentskills.io/) for Bun/TypeScript, SolidJS UI/UX, Hono full-stack backends, Tauri desktop, Docker packaging, and app briefing workflows.

[![skills.sh](https://skills.sh/b/ur-wesley/agent-skills)](https://skills.sh/ur-wesley/agent-skills)

## Installation

```bash
npx skills add ur-wesley/agent-skills
```

Install individual skills:

```bash
npx skills add ur-wesley/agent-skills --skill ts-styleguide
npx skills add ur-wesley/agent-skills --skill solidjs-ui
npx skills add ur-wesley/agent-skills --skill tauri-solid-desktop
npx skills add ur-wesley/agent-skills --skill bun-hono-fullstack
npx skills add ur-wesley/agent-skills --skill bun-testing-e2e
npx skills add ur-wesley/agent-skills --skill bun-docker-deploy
npx skills add ur-wesley/agent-skills --skill app-brief
npx skills add ur-wesley/agent-skills --skill app-existing
npx skills add ur-wesley/agent-skills --skill app-scope
npx skills add ur-wesley/agent-skills --skill app-stack
npx skills add ur-wesley/agent-skills --skill app-brand
npx skills add ur-wesley/agent-skills --skill project-vault
```

## Available skills

### Stack & Engineering

#### ts-styleguide

Generic TypeScript project toolchain — Bun runtime/package manager, Oxc (`oxlint` type-aware + type-check, `oxfmt`), `knip` dead-code checks, `lefthook`, `mise`, strict `tsconfig`, and optional greenfield monorepo.

**Use when:** scaffolding or working on TS repos, lint/format/knip/hooks, oxlint/oxfmt/lefthook/tsconfig, or Bun workspaces.

#### solidjs-ui

Highest-priority SolidJS UI, tooling, and UX design skill. Bespoke components built directly on Kobalte / Ark UI / Corvu / `solid-sonner`, SolidJS Oxlint/tsconfig/`bun:test` TSX preload, flat UX layout without nested containers, `@theme inline` + `class-guard`, `i-mdi-*` icons, and `@ur-wesley/ts-prelude/<subpath>`.

**Use when:** any SolidJS, Tailwind, `components/*`, `features/*`, routes, forms, context stores, or frontend UI/UX work.

#### tauri-solid-desktop

Builds Tauri v2 desktop apps with SolidJS, Tailwind 4, Kobalte/Ark UI primitives, Iconify MDI icons, Bun, `@ur-wesley/ts-prelude`, and solid-primitives. UI conventions follow `solidjs-ui`.

**Use when:** creating or modifying Tauri SolidJS desktop apps, writing Tauri commands, or releasing via `bumpp`.

#### bun-hono-fullstack

Full-stack Bun + Hono backend (`hono/client` RPC), `bun:sqlite`, `better-auth`, Zod v4 bounded validators, `@ur-wesley/ts-prelude/log` structured logging, security hardening (CSP, DNS-pinned SSRF guard, timing-safe tokens), and `/healthz` probes.

**Use when:** building or auditing Bun HTTP APIs, `server/routes/*`, `server/db/*`, auth, or full-stack SolidJS web apps.

#### bun-testing-e2e

Bun-native 3-tier zero-Playwright E2E testing (`app.request`, `Bun.WebView`, isolated fullstack `Bun.serve` + Vite + `Bun.WebView`) with cross-platform `e2e/run.ts` and prod-safety gates.

**Use when:** setting up or running E2E suites, isolated test DBs, or E2E auth bypass seams.

#### bun-docker-deploy

Multi-stage `oven/bun-alpine` → `alpine` Docker packaging using `bun build --compile --bytecode`, `tini` + `su-exec` non-root entrypoints, `/healthz` probes, and volume permissions.

**Use when:** creating or auditing `Dockerfile`, `docker-entrypoint.sh`, `.dockerignore`, or container deployments.

### Vault

#### project-vault

Project Vault MCP server operations for workspace and board automation — query projects, manage markdown kanban cards (`.vault/kanban/`), and orchestrate isolated git worktree coding agents.

**Use when:** managing projects, kanban cards, or agent worktrees in Project Vault.

### App briefing

Interview or discover first, then write `docs/IDEA.md`, `docs/STACK.md`, and `docs/BRAND.md` in the target app repo. Writers also run alone.

#### app-brief

Interview until the picture is complete (problem, users, MVP, platform, naming theme, vibe), then run the three writers.

**Use when:** starting a new app, "app brief", "scope this idea", "brand this app", or wanting all three docs together. Existing codebase → `app-existing`.

#### app-existing

Auto-discover idea, stack, and brand from an existing codebase, grill only gaps, confirm the picture, then run the three writers.

**Use when:** the repo already has an app, "document this app", "existing project brief", "capture this repo", or wanting those docs for code that already exists. New/empty app → `app-brief`.

#### app-scope

Writes `docs/IDEA.md` — idea, audience, MVP scope, features, non-goals.

**Use when:** scoping an app, listing features, or after `app-brief` / `app-existing` confirms the picture.

#### app-stack

Writes `docs/STACK.md` from platform (`desktop` / `web` / `web-fullstack` / `cli`) using prescribed stack skills. Does not invent a stack.

**Use when:** choosing or recording app tech, or after `app-brief` / `app-existing` confirms the picture.

#### app-brand

Writes `docs/BRAND.md` — name from a theme field, corporate design, concise logo prompt (no image unless asked).

**Use when:** naming an app, defining brand/CD, or after `app-brief` / `app-existing` confirms the picture.

## Skill structure

Each skill contains:

- `SKILL.md` — instructions for the agent
- `scripts/` — helper scripts for automation (optional)
- `references/` — supporting documentation (optional)

## License

MIT
