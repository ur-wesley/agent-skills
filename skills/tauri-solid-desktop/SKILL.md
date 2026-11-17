---
name: tauri-solid-desktop
description: Builds Tauri v2 desktop apps with SolidJS, Tailwind 4, Kobalte/Ark UI primitives, Iconify MDI icons, Bun, @ur-wesley/ts-prelude, and solid-primitives. UI conventions follow solidjs-ui. Use when creating or modifying Tauri SolidJS desktop apps, writing Tauri commands, or releasing via bumpp (syncing package.json, Cargo.toml, and tauri.conf.json).
---

# Tauri Solid Desktop

**UI work follows `solidjs-ui`.** That skill always wins on components, Tailwind, icons, and ts-prelude. Use this skill for Tauri/Rust/release only when UI conventions conflict.

## Quick start

1. Confirm Tauri + SolidJS repo with `src-tauri/` and Bun (`bun.lock`, `packageManager`)
2. Read project overrides (see discovery below); apply prescribed stack from [STACK.md](STACK.md)
3. Implement using stack conventions
4. Verify before done (see below)
5. Suggest gaps in chat only — never auto-create missing docs

## Stack at a glance

| Layer | Choices |
| ----- | ------- |
| Desktop | Tauri v2 + Rust |
| Frontend | SolidJS, Tailwind 4, Kobalte / Ark UI primitives (`components/ui`) |
| Icons | Iconify MDI via `@iconify/tailwind4` — `i-mdi-<icon>` per `solidjs-ui` |
| Logic | `@ur-wesley/ts-prelude/<subpath>` per `solidjs-ui` |
| Primitives | solid-primitives (event-bus, keyboard, i18n) |
| Tooling | Bun, mise, Oxlint + Oxfmt, knip |
| Release | bumpp → tag `v*` → GitHub Actions → Releases + updater |

Details, deps, snippets: [STACK.md](STACK.md). Setup, Rust, CI: [REFERENCE.md](REFERENCE.md).

## Discovery (per-repo overrides)

Read in order; skip missing files:

1. `package.json` — Bun, stack deps, release scripts
2. `bump.config.ts` — version file list
3. `mise.toml`, `.env.example`
4. `DESIGN.md`, `ai/styleguide-tailwind.md`, `ai/styleguide-rust.md`
5. `src-tauri/tauri.conf.json`
6. `.github/workflows/*`
7. `docs/RELEASE.md`

Summarize in chat: aligned, missing, divergent. Stack itself is prescribed.

## Setup

```bash
mise install
bun install
cp .env.example .env   # if present
bun tauri dev
```

Never `npm run` or `pnpm`. Use `bun`, `bun run`, `bunx` only.

## Project structure

```
src/components/ui/     # atomic primitive wrappers (file -> folder of atoms when it grows)
src/components/layout/ # reusable page/shell/split-pane layouts
src/components/<org>/  # organisms in their own directories under src/components/
src/features/<name>/   # feature components, hooks, actions
src/services/          # Tauri invoke wrappers (ResultAsync via ts-prelude /result)
src/lib/               # cn(), contexts
src-tauri/src/         # thin commands + modules
```

Import rule: `components/ui` must not import from `features/*`.

## Coding rules

- No `any`; explicit types; `readonly` where possible
- Context stores: `readonly [state, actions]` tuple
- TanStack Query for async/server state in features — not in `components/ui`
- Prefer `createMemo` / `For` / `Show` over unnecessary effects

## Library workflows

| Module | Use when |
| ------ | -------- |
| `@ur-wesley/ts-prelude/result` | Tauri invoke, fetch, any fallible IO at service boundary |
| `@ur-wesley/ts-prelude/match` | Branching on unions, command results, status enums |
| `@ur-wesley/ts-prelude/pipe`, `/data/<fn>` | Array/object transforms |
| solid-primitives | Cross-cutting concerns (keyboard, i18n, event-bus) |

See `solidjs-ui` for full ts-prelude subpath table and UI conventions.

## UI workflow

Follow `solidjs-ui`. Summary:

- Build domain-agnostic UI primitives directly on `@kobalte/core`, `@ark-ui/solid`, `@corvu/drawer`, and `solid-sonner`
- Tailwind utilities + `@theme inline` design tokens from `src/index.css` / `App.css`
- Icons: `i-mdi-<icon>` classes (Iconify prefix `i`)
- Flat UX hierarchy (no nested card-in-card containers); spacing over borders

## Tauri workflow

- Commands < 30 LOC; delegate to modules
- `thiserror` (`AppError`) internally; `StableError` only at command boundary
- 500 LOC max per Rust file

## Release workflow

1. `rtk bun run lint` + `rtk bun run typecheck`
2. `bun run release` or `bun run release:patch`
3. bumpp bumps `package.json`, `src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`
4. Commit + tag `vX.Y.Z` + push → CI builds (no CI version rewriting)

See [REFERENCE.md](REFERENCE.md) for CI secrets, verify-before-done checklist, gap templates, and anti-patterns.
