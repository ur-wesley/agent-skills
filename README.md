# Agent Skills

Personal [Agent Skills](https://agentskills.io/) for TypeScript/SolidJS/Tauri development and Fusion multi-agent workflows.

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
npx skills add ur-wesley/agent-skills --skill fusion-setup
npx skills add ur-wesley/agent-skills --skill cursor-fusion
npx skills add ur-wesley/agent-skills --skill app-brief
npx skills add ur-wesley/agent-skills --skill app-existing
npx skills add ur-wesley/agent-skills --skill app-scope
npx skills add ur-wesley/agent-skills --skill app-stack
npx skills add ur-wesley/agent-skills --skill app-brand
```

## Available skills

### Stack

#### ts-styleguide

TypeScript project toolchain — Bun runtime/package manager, Oxc (oxlint type-aware + type-check, oxfmt), lefthook, strict tsconfig, optional greenfield monorepo.

**Use when:** scaffolding or working on TS repos, lint/format/hooks, oxlint/oxfmt/lefthook/tsconfig, or Bun workspaces.

#### solidjs-ui

Highest-priority SolidJS UI style guide. Icons via `i-mdi-<icon>` (Iconify prefix `i`). Logic via `@ur-wesley/ts-prelude/<subpath>`.

**Use when:** any SolidJS, Tailwind, components/ui, features/*, routes, forms, context stores, or frontend UI work.

#### tauri-solid-desktop

Builds Tauri v2 desktop apps with SolidJS, Tailwind 4, solid-ui, Iconify MDI icons, Bun, `@ur-wesley/ts-prelude`, and solid-primitives. UI conventions follow `solidjs-ui`.

**Use when:** creating or modifying Tauri SolidJS desktop apps, writing Tauri commands, or releasing via bumpp.

### App briefing

Interview or discover first, then write `docs/IDEA.md`, `docs/STACK.md`, and `docs/BRAND.md` in the target app repo. Writers also run alone.

#### app-brief

Interview until the picture is complete (problem, users, MVP, platform, naming theme, vibe), then run the three writers.

**Use when:** starting a new app, "app brief", "scope this idea", "brand this app", or wanting all three docs together. Existing codebase → `app-existing`.

#### app-existing

Auto-discover idea, stack, and brand from an existing repo, grill only gaps, confirm the picture, then run the three writers.

**Use when:** the repo already has an app, "document this app", "existing project brief", "capture this repo", or wanting those docs for code that already exists. New/empty app → `app-brief`.

#### app-scope

Writes `docs/IDEA.md` — idea, audience, MVP scope, features, non-goals.

**Use when:** scoping an app, listing features, or after app-brief / app-existing confirms the picture.

#### app-stack

Writes `docs/STACK.md` from platform (desktop / web / cli) using prescribed stack skills. Does not invent a stack.

**Use when:** choosing or recording app tech, or after app-brief / app-existing confirms the picture.

#### app-brand

Writes `docs/BRAND.md` — name from a theme field, corporate design, concise logo prompt (no image unless asked).

**Use when:** naming an app, defining brand/CD, or after app-brief / app-existing confirms the picture.

### Fusion

#### fusion-setup

Sets up the OpenCode Fusion agent team — a main agent that plans and reviews but delegates all edits to a sidekick subagent, plus explore/search and optional specialists.

**Use when:** "set up fusion", "configure fusion", "install fusion", changing Fusion agent models, or using a subscription profile (OpenCode Go/Zen, ChatGPT, GitHub Copilot).

#### cursor-fusion

Cursor Fusion workflow (Devin Fusion-style). Orchestrates model split: Grok 4.5 plans/reviews, Composer edits, explore/cavecrew for search.

**Use when:** "use fusion", "delegate sidekick", "cursor fusion", or starting multi-file feature work.

## Skill structure

Each skill contains:

- `SKILL.md` — instructions for the agent
- `scripts/` — helper scripts for automation (optional)
- `references/` — supporting documentation (optional)

## License

MIT
