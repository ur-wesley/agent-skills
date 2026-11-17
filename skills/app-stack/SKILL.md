---
name: app-stack
description: Write docs/STACK.md from a platform pick (desktop, web, web-fullstack, or cli) using prescribed stack skills. Use when choosing or recording app tech, or after app-brief or app-existing confirms the picture. Do not invent a new stack.
---

# App Stack

Writes `docs/STACK.md` in the **target app repo**. Create `docs/` if missing. Language = interview language.

If `docs/STACK.md` already exists and the user did not ask to replace it, update in place.

## Inputs

Read `docs/IDEA.md` if present (constraints, extras). Skip questions already answered there or in chat.

If run **after app-brief or app-existing**: do not re-interview. Write from the confirmed picture (platform already picked; if the picture chose record-actual vs prescribed-as-target, honor that).

If run **solo** and platform unknown: ask once — **desktop / web / web-fullstack / cli** — with a recommended pick from the idea. Wait.

**Do not invent a stack.** Fill from existing skills. Point at those skills; do not copy their STACK.md.

| Platform | Fill from |
| --- | --- |
| desktop | `tauri-solid-desktop` + `solidjs-ui` + `ts-styleguide` |
| web | `solidjs-ui` + `ts-styleguide` (Solid SPA, Bun, Oxc, knip) |
| web-fullstack | `bun-hono-fullstack` + `bun-testing-e2e` + `bun-docker-deploy` + `solidjs-ui` + `ts-styleguide` |
| cli | `ts-styleguide` only (Bun, Oxc, knip, no UI) |

This-app extras only: hosting, extra crates/libs named in the interview. If none, write `None`.

## File

```markdown
# Stack

## Platform

{desktop | web | web-fullstack | cli}

## Prescribed

| Layer | Choice |
| --- | --- |
| {layer} | {choice} |

Skills: {comma-separated skill names}

## Extras

- {hosting, extra lib, or None}

## Commands

{platform defaults below}
```

### Prescribed tables

**desktop**

| Layer | Choice |
| --- | --- |
| Desktop | Tauri v2 + Rust |
| Frontend | SolidJS, Tailwind 4, Kobalte / Ark UI primitives |
| Icons | Iconify MDI (`i-mdi-<icon>`) |
| Logic | `@ur-wesley/ts-prelude/<subpath>` |
| Tooling | Bun, Oxc (oxlint + oxfmt), knip, lefthook |

Commands: `mise install`, `bun install`, `bun tauri dev`. Never `npm` / `pnpm`.

**web**

| Layer | Choice |
| --- | --- |
| App | SolidJS SPA |
| UI | Tailwind 4, Kobalte / Ark UI primitives |
| Icons | Iconify MDI (`i-mdi-<icon>`) |
| Logic | `@ur-wesley/ts-prelude/<subpath>` |
| Tooling | Bun, Oxc (oxlint + oxfmt), knip, lefthook |

Commands: `bun install`, `bun dev`. Never `npm` / `pnpm`.

**web-fullstack**

| Layer | Choice |
| --- | --- |
| Backend | Bun + Hono (`@hono/bun`), `bun:sqlite`, `better-auth`, Zod v4 |
| Frontend | SolidJS, TanStack Router/Query/Form, Tailwind 4, Kobalte / Ark UI primitives |
| Icons & Fonts | Iconify MDI (`i-mdi-<icon>`), self-hosted `@fontsource/*` |
| Logic & Logging | `@ur-wesley/ts-prelude/<subpath>` (incl. `/log`) |
| Testing & E2E | `bun:test` (Solid TSX preload) + `Bun.WebView` 3-tier E2E |
| Tooling & Deploy | Bun, Oxc, knip, lefthook, multi-stage Docker (`bun build --compile`) |

Commands: `bun install`, `bun dev`, `bun run check:all`, `bun run e2e`. Never `npm` / `pnpm`.

**cli**

| Layer | Choice |
| --- | --- |
| Runtime | Bun |
| Tooling | Oxc (oxlint + oxfmt), knip, lefthook |

Commands: `bun install`, `bun <entry>`. Never `npm` / `pnpm`. No frontend deps.

## Out of scope

Do not scaffold the repo. `ts-styleguide` / `tauri-solid-desktop` do that when asked.
