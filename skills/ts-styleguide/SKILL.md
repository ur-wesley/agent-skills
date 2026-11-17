---
name: ts-styleguide
description: Generic TypeScript project toolchain — Bun runtime/package manager, Oxc (oxlint type-aware + type-check, oxfmt), knip dead-code checks, lefthook, mise, strict tsconfig, and optional greenfield monorepo. Use when scaffolding or working on TS repos, lint/format/knip/hooks, oxlint/oxfmt/lefthook/tsconfig, or Bun workspaces. SolidJS UI/JSX rules live in solidjs-ui.
---

# TS Styleguide

Generic TypeScript toolchain and code-quality baseline. Framework-agnostic.

## Precedence

- **solidjs-ui** owns all SolidJS, JSX, Solid lint rules, TSX test preload, and UI rules.
- **This skill** owns generic TS tooling: Bun, Oxc, `knip`, `lefthook`, `mise`, `tsconfig`, scripts, and greenfield layout.
- Overrides `setup-pre-commit` on TS repos — no Husky, Prettier, ESLint, or lint-staged.
- **tauri-solid-desktop** owns Tauri/Rust/release. Do not auto-rewrite `tsc --noEmit` or Vitest in those repos unless asked to migrate.

## Non-negotiable

### Bun & Runtime Pinning

- Runtime and package manager only.
- `packageManager` + `engines.bun` in `package.json`, `bun.lock`, and `mise.toml` (`bun = "latest"` or pinned). No `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`.
- Commands: `bun`, `bun run`, `bunx`, `bun add -D`. Never `npm`, `pnpm`, `yarn`, `npx`.

### Oxc (`oxlint` + `oxfmt`)

- Deps: `oxlint`, `oxlint-tsgolint`, `oxfmt`. TypeScript **7+** for type-aware linting.
- Config: `.oxlintrc.json` (jsonc OK). Not `oxlint.config.ts`.
- Root options: `typeAware: true`, `typeCheck: true` — replaces `tsc --noEmit`.
- Baseline categories: `correctness` + `suspicious` as `error`.
- **Production App profile** (see [CONFIG.md](CONFIG.md)): enable `pedantic: "error"`, `no-console: "error"` (pair with `@ur-wesley/ts-prelude/log`), `no-nested-ternary`, `max-depth: 4`, `max-params: 4`, `max-lines: 400`, `typescript/no-explicit-any`, `typescript/no-non-null-assertion`, `typescript/no-floating-promises`, `typescript/consistent-type-imports`, plus `**/*.test.*` relaxations.
- Do not set `plugins` unless intentionally listing every required plugin — it replaces Oxlint defaults.
- Format: `.oxfmtrc.json` or `.oxfmtrc.jsonc` (`printWidth` 80 or 100, `singleQuote`, `semi`, `trailingComma`). Greenfield: no ESLint, Prettier, Biome, Husky.

### Dead-Code Hygiene (`knip`)

- Add `knip` + `knip.json` in apps to fail on unused files, exports, types, dependencies, binaries, and duplicates.
- Configure `entry` and `project` globs explicitly (plus `ignoreDependencies` only for CLI/build-plugin packages that are not imported in TS).

### Scripts

Root `package.json`:

| Script | Command |
| --- | --- |
| `lint` | `oxlint` |
| `lint:fix` | `oxlint --fix` |
| `fmt` | `oxfmt` |
| `fmt:check` | `oxfmt --check` |
| `check` | `oxlint && oxfmt --check` |
| `deadcode` | `knip` |
| `check:all` | `oxlint && oxfmt --check && knip && bun run test` |
| `prepare` | `lefthook install` |

No separate `typecheck` script needed when `typeAware` + `typeCheck` are enabled (`oxlint` covers it).

### lefthook

- `bun add -D lefthook`, `prepare: "lefthook install"`.
- **Pre-commit**: either parallel read-only checks (`oxlint {staged_files}` + `oxfmt --check {staged_files}`) for zero surprise hook edits, or auto-fix (`oxfmt --write` + `oxlint --fix` with `stage_fixed: true`). Pick one per repo; default to read-only parallel checks in strict app repos.
- **Pre-push**: `bun run check` (or `bun run check:all`).

### tsconfig

- `module` / `target`: `ESNext`, `moduleResolution`: `bundler`.
- `strict`, `noUncheckedIndexedAccess`, `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`, `isolatedModules`, `verbatimModuleSyntax`, `skipLibCheck`, `noEmit`, `allowImportingTsExtensions`.
- `"types": ["bun"]` when using Bun APIs.
- No `baseUrl`. No path aliases unless a bundler already requires them.
- Scope `include` explicitly (`["src"]`, or `["src", "server", "scripts", "test", "e2e"]` for fullstack apps). Never root `"include": ["**/*"]`.
- SolidJS apps add JSX compiler options per `solidjs-ui`.

### Testing (`bun:test`)

- `mock.module(...)` is hoisted by Bun and resolves specifiers relative to the **calling file**. Always keep it a direct call in the test file prefixed with `void` (`void mock.module('./mod', () => ({ ... }))`) to satisfy `typescript/no-floating-promises` without lint disables or wrapper helpers.

### Greenfield monorepo (multi-package only)

Single-package repos stay flat `src/`. Monorepo layout:

```
package.json          # workspaces: apps/*, packages/*
bun.lock
mise.toml
knip.json
tsconfig.json         # "files": [], references only
tsconfig.base.json
.oxlintrc.json
.oxfmtrc.json
lefthook.yml
apps/<name>/
packages/<name>/
```

- Bun `workspaces`. No catalog unless the user asks.
- Libraries: `composite` + `declaration` so type-aware lint resolves `.d.ts`.
- Root oxlint/oxfmt/knip/lefthook only — not duplicated per package.

## Workflows

### Greenfield

1. Single vs monorepo?
2. Write configs from [CONFIG.md](CONFIG.md).
3. `bun install` → `bunx lefthook install` → `bun run check`.

### Existing repo

1. Add/align Bun + Oxc + `knip` + `lefthook` + `mise.toml` + `tsconfig`.
2. Do not invent a monorepo.
3. Remove ESLint/Prettier/Husky only when replacing them in this change.

### Day-to-day

- `bun run check` / `check:all` / `lint` / `fmt`.
- Never add a second linter or formatter.

## Out of scope

SolidJS JSX/lint/preload (see `solidjs-ui`), E2E browser testing (see `bun-testing-e2e`), Docker packaging (see `bun-docker-deploy`), commitlint.

## Templates

Copy-paste configs: [CONFIG.md](CONFIG.md).
