---
name: ts-styleguide
description: TypeScript project toolchain — Bun runtime/package manager, Oxc (oxlint type-aware + type-check, oxfmt), lefthook, strict tsconfig, optional greenfield monorepo. Use when scaffolding or working on TS repos, lint/format/hooks, oxlint/oxfmt/lefthook/tsconfig, or Bun workspaces.
---

# TS Styleguide

## Precedence

- **solidjs-ui** wins on UI.
- **This skill** wins on TS toolchain: Bun, Oxc, lefthook, tsconfig, scripts, greenfield layout.
- Overrides **setup-pre-commit** on TS repos — no Husky / Prettier / lint-staged.
- **tauri-solid-desktop** owns Tauri/Rust/release. Do not auto-rewrite `tsc --noEmit` or Vitest in those repos unless the user asks to migrate.

## Non-negotiable

### Bun

- Runtime and package manager only.
- `packageManager` field + `bun.lock`. No `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`.
- Commands: `bun`, `bun run`, `bunx`, `bun add -D`. Never `npm`, `pnpm`, `yarn`, `npx`.

### Oxc

- Deps: `oxlint`, `oxlint-tsgolint`, `oxfmt`. TypeScript **7+** for type-aware.
- Config: `.oxlintrc.json` (jsonc OK). Not `oxlint.config.ts`.
- Root options: `typeAware: true`, `typeCheck: true` — replaces `tsc --noEmit`.
- Categories: `correctness` + `suspicious` as error; pedantic off.
- Do not set `plugins` unless adding — it replaces Oxlint defaults.
- Format: `.oxfmtrc.jsonc`. Greenfield: no ESLint, Prettier, Biome, Husky.

### Scripts

Root `package.json`:

| Script | Command |
| --- | --- |
| `lint` | `oxlint` |
| `lint:fix` | `oxlint --fix` |
| `fmt` | `oxfmt` |
| `fmt:check` | `oxfmt --check` |
| `check` | `oxlint && oxfmt --check` |

No separate `typecheck` — `lint` / `check` type-check via config. Test runner not prescribed.

### lefthook

- `bun add -D lefthook`, `prepare: "lefthook install"`.
- `pre-commit`: `oxfmt` then `oxlint --fix` on staged files, `stage_fixed: true`.
- `pre-push`: `bun run check` (full type-aware + format check).

### tsconfig

- `module`/`target` ESNext, `moduleResolution: bundler`, `strict`, `noUncheckedIndexedAccess`, `isolatedModules`, `verbatimModuleSyntax`, `skipLibCheck`, `noEmit`.
- No `baseUrl`. No path aliases unless a bundler already requires them.
- Scope `include` to `src` (or package src). Never root `"include": ["**/*"]`.

### Greenfield monorepo (multi-package only)

Single-package repos stay flat `src/`. Monorepo layout:

```
package.json          # workspaces: apps/*, packages/*
bun.lock
tsconfig.json         # "files": [], references only
tsconfig.base.json
.oxlintrc.json
.oxfmtrc.jsonc
lefthook.yml
apps/<name>/
packages/<name>/
```

- Bun `workspaces`. No catalog unless the user asks.
- Libraries: `composite` + `declaration` so type-aware lint resolves `.d.ts`.
- Root oxlint/oxfmt/lefthook only — not duplicated per package.

## Workflows

### Greenfield

1. Single vs monorepo?
2. Write configs from [CONFIG.md](CONFIG.md).
3. `bun install` → `bunx lefthook install` → `bun run check`.

### Existing repo

1. Add/align Bun + Oxc + lefthook + tsconfig.
2. Do not invent a monorepo.
3. Remove ESLint/Prettier/Husky only when replacing them in this change.

### Day-to-day

- `bun run check` / `lint` / `fmt`.
- Never add a second linter or formatter.

## Out of scope

bun:test vs Vitest, Bun catalog, import/alias rules, commitlint, mise (optional if repo already has it).

## Templates

Copy-paste configs: [CONFIG.md](CONFIG.md).
