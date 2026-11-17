# Reference

Setup checklist, Bun commands, Rust conventions, release/CI, and gap templates.

## Setup checklist

1. Clone with submodules if needed: `git clone --recurse-submodules <url>`
2. `mise install` — pins Bun and other tools
3. `bun install` — never npm/pnpm
4. Copy `.env.example` → `.env` if present
5. `bun tauri dev` — full desktop app with HMR
6. Install VS Code extensions: Tauri, rust-analyzer (recommended)

## Bun commands

| Task | Command |
| ---- | ------- |
| Install deps | `bun install` |
| Dev (Vite HMR) | `bun dev` |
| Desktop dev | `bun tauri dev` |
| Frontend build | `bun run build` |
| Production build | `bun tauri build` |
| Lint | `bun run lint` |
| Format | `bun run format` |
| Typecheck | `bun run typecheck` |
| Test | `bun run test` |
| Release (interactive) | `bun run release` |
| Release patch | `bun run release:patch` |

Prefix with `rtk` in agent sessions. Never `npm run` or `npm build`.

## Package manager rules

| Signal | Use |
| ------ | --- |
| `bun.lock` or `packageManager: bun@...` | Bun exclusively |
| Stray `package-lock.json` in Bun project | Flag in chat; remove it |

## Rust / Tauri conventions

### File organization

- 500 LOC max per file; consider splitting at 300
- Thin command files (< 30 LOC); logic in domain modules
- `folder/mod.rs` for multi-file modules

### Error handling

- `thiserror` enums (`AppError`) for internal Rust
- `StableError` only at Tauri command boundary (frontend contract)
- `From<AppError> for StableError` in `error.rs`
- Never construct `StableError` in business logic

### Naming

- `snake_case` — functions, variables, modules, files
- `PascalCase` — structs, enums, traits
- `SCREAMING_SNAKE_CASE` — constants

### Platform code

- Isolate `#[cfg(windows)]`, `#[cfg(target_os = "macos")]`, `#[cfg(unix)]` into dedicated modules
- No inline `#[cfg]` in shared functions

### Imports

Order: `std` → external crates → `crate::` → `super::`. No wildcard imports.

### Async

- `tokio::spawn` for background work
- Never block command thread with sync I/O

## Frontend conventions

Follow `solidjs-ui` (read `DESIGN.md` or `ai/styleguide-tailwind.md` when present). Defaults:

- Feature-based layout under `src/features/<name>/`
- Bespoke atomic primitive components in `components/ui/` (promoting to `components/ui/<name>/` when a component outgrows one file) and organisms in `components/<organism>/` — domain-agnostic; no feature imports in `components/ui`
- Flat visual hierarchy (max 1 containment level, spacing over borders)
- TanStack Query in features for async data — not in shared UI
- Accessibility required: semantic HTML, `focus-visible` styles, labels
- Icons: `i-mdi-<icon>` per `solidjs-ui` (Iconify tailwind4 plugin, prefix `i`)

## Release workflow (bumpp → CI)

### Local steps

1. Clean working tree
2. `rtk bun run lint` + `rtk bun run typecheck`
3. `bun run release` or `bun run release:patch|minor|major`
4. bumpp updates `package.json`, `src-tauri/Cargo.toml`, `src-tauri/tauri.conf.json`
5. bumpp commits, tags `vX.Y.Z`, pushes
6. CI triggers on `v*` tag push

### CI expectations

- Trigger: push tags matching `v*`, optional `workflow_dispatch`
- Steps: checkout (submodules), setup Bun + Rust, `bun install`, `bun tauri build`
- Changelog: `changelogen` between tags (CI or bumpp hook)
- Secrets: `GH_DEVICE_CLIENT_ID`, `TAURI_SIGNING_PRIVATE_KEY`, `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`, `GITHUB_TOKEN`
- Output: GitHub Release with signed installer + Tauri updater manifest

### Anti-patterns

- **CI rewriting versions from git tag** — masks drift; remove when bumpp is adopted
- **Version drift** — `package.json`, `Cargo.toml`, `tauri.conf.json` must match before tag
- **npm/npx in CI** — use `bun` and `bunx` instead

### tauri.conf.json essentials

- `build.beforeDevCommand`: `bun run dev`
- `build.beforeBuildCommand`: `bun run build`
- `plugins.updater.endpoints` — GitHub Releases `latest.json` URL
- `bundle.createUpdaterArtifacts` — enable for auto-update

## Verification before done

| Change type | Commands |
| ----------- | -------- |
| TypeScript/TSX | `rtk bun run lint`, `rtk bun run typecheck` |
| Logic with tests | `rtk bun run test` |
| Rust | `rtk cargo fmt --check`, `rtk cargo clippy` (in `src-tauri/`) |

## Gap suggestion template

Post in chat when repo diverges — never auto-create files:

```
## Convention gaps

- [ ] Missing bumpp / bump.config.ts — add prescribed config; sync three version files
- [ ] Version drift: package.json X vs Cargo.toml/tauri.conf.json Y
- [ ] CI re-syncs version from tag — remove PowerShell/sed step; use bumpp pre-push
- [ ] Missing `@ur-wesley/ts-prelude` — add to package.json for new code
- [ ] Styleguide uses `iconify mdi--*` — migrate to `i-mdi-*` per `solidjs-ui`
- [ ] README missing setup — add: mise install, bun install, bun tauri dev
- [ ] No lint/typecheck in CI — add bun run check (or lint + typecheck) to workflow
```

## Reference implementation

**project-vault** (`c:\Arbeit\project-vault`) is the canonical example:

| Area | Path |
| ---- | ---- |
| Stack guide | `docs/stack-guide.md` |
| UI style | `ai/styleguide-tailwind.md` |
| Rust style | `ai/styleguide-rust.md` |
| Shared primitives | `src/components/ui/` |
| Icons | `src/App.css`, feature components |
| ts-prelude /result | `src/services/tauri/utils.ts` |
| CI | `.github/workflows/build-windows.yml` |
| Release QA | `docs/RELEASE.md` |

Known gaps the skill should flag (not auto-fix):

- `@ur-wesley/ts-prelude`, `bumpp` not yet in dependencies
- Version drift between `package.json` and `src-tauri/*`
- CI PowerShell version-sync instead of bumpp
- Styleguide `iconify mdi--*` vs live `i-mdi-*` syntax per `solidjs-ui`
