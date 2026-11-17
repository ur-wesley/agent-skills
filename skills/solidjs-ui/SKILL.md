---
name: solidjs-ui
description: Highest-priority SolidJS UI, tooling, and UX design skill. Covers bespoke primitive-based components (Kobalte, Ark UI, Corvu, solid-sonner), SolidJS Oxlint/tsconfig/bun:test TSX preload, flat UX layout without nested containers, @theme inline tokens + class-guard, Iconify MDI (i-mdi-*), and @ur-wesley/ts-prelude/<subpath>. Use for any SolidJS, Tailwind, components/*, features/*, routes, forms, context stores, or frontend UI work.
---

# SolidJS UI

## Precedence

This skill **always wins** for frontend, SolidJS tooling, and UI/UX work: components, Tailwind, icons, forms, a11y, context stores, `eslint-plugin-solid`, TSX test preload, feature UI, and routes.

Other skills (including `tauri-solid-desktop` or `bun-hono-fullstack`) apply for Rust/desktop/backend/release. On any UI or SolidJS conflict, **this skill wins**. Never use `iconify mdi--*` or direct `neverthrow` / `ts-pattern` / `remeda` imports.

## Non-negotiable

### Bespoke Primitive Components

Build your own domain-agnostic components directly on headless primitives:

| Layer | Use |
| --- | --- |
| `@kobalte/core` | dialog, popover, select, combobox, switch, slider, tooltip, tabs, context-menu |
| `@ark-ui/solid` | primitives not in Kobalte (date-picker, color-picker, etc.) |
| `@corvu/drawer` / `@corvu/calendar` | drawers and mini-calendars |
| `cmdk-solid` | command palette / spotlight search |
| `solid-sonner` | toasts — mount `<Toaster>` once at root, call `toast()` elsewhere |
| `@tanstack/solid-query` | server state, caching, mutations — features/services only |
| `@tanstack/solid-form` + `zod` | typed form state and schema validation |
| `@tanstack/solid-table` | complex data tables |
| `@tanstack/solid-router` | typed file-based or code-based routing |
| `@solid-primitives/*` | keyboard, i18n, storage, geolocation, timer, media, observers |

### UX & Design Language (Flat Hierarchy First)

- **No deep container nesting**: ban "card-in-card" / boxes-within-boxes. Enforce **max 1 level of visual containment**.
- **Spacing & typography over borders**: group with whitespace (Gestalt proximity), alignment, font scale/weight, and subtle tonal surface shifts before adding `1px` borders or shadows.
- **Restrained elevation**: keep scrollable lists/grids flat and synthetic (zero `backdrop-filter` in high-density scroll planes); reserve glass/blur for floating chrome (dialogs, toolbars, toasts).
- **Perceived speed & interaction craft**:
  - Never overload single-click and double-click on the same element (eliminates 250–400ms lag).
  - Give `0ms` active feedback on `pointerdown`; commit on `click` / `pointerup`.
  - Prefer optimistic 0ms updates + "Undo" toast over blocking "Are you sure?" modals; use `HoldToConfirmButton` or typed confirmation for destructive actions.
  - Group toolbar tooltips (500ms warm-up → 0ms sweep → 500ms cool-down) and keep touch targets `>= 44×44px` (Fitts's Law).
  - Use progressive disclosure and layout-matching skeletons (`300ms–1s`), not generic blocking spinners.

### Icons & Fonts

- **Icons**: `@iconify/tailwind4` (`prefix: "i"`) + `@iconify-json/mdi` → `i-mdi-<icon>`. Decorative: `aria-hidden="true"`. Icon-only controls: accessible label. Never `iconify mdi--*` or ad-hoc SVGs when MDI exists.
- **Fonts**: self-hosted `@fontsource/*` / `@fontsource-variable/*` (zero Google Fonts egress) + `@utility tnum` for tabular numbers.

### Tokens & Class Guard

- Map runtime CSS vars (`--theme-*`, `--cp-*`) in `src/index.css` via `@theme inline` (`--color-fg`, `--color-accent`, `--color-card`, `--color-glass`, etc.) and use semantic utilities (`text-fg`, `bg-accent/12`).
- `cn(...)` (`clsx` + `tailwind-merge`) only — never `clsx`/`twMerge` directly or `['a', cond && 'b'].filter(Boolean).join(' ')`.
- Enforce with `scripts/class-guard.ts`: ban `[var(--theme-*)]`, arbitrary hex `-[#...]`, `color-mix` with mapped vars, and tiny text (`text-[8-10px]`).
- `cva` is optional — use it for `>= 3` variants, or write a tiny local variant resolver so `knip` stays clean.

### `ts-prelude`

Always `@ur-wesley/ts-prelude/<subpath>`. Never package root (`VERSION` only). Never import `neverthrow`, `ts-pattern`, or `remeda` directly.

| Subpath | Used for |
| --- | --- |
| `/result` | `Result`, `ResultAsync`, `ok`, `err`, `matchResult`, `runCatching`, `traverse`, `combineWithAllErrors` |
| `/option` | `Option<T>`, `fromNullable`, `map`, `andThen`, `getOrElse`, `zip` |
| `/match` | `match` + `R` / `O` / `P` for exhaustive branching |
| `/pipe` | `pipe` / `flow` / `tap` / `dbg` |
| `/scope` | `let_`, `run`, `apply`, `also`, `with_`, `ifSome`, `ifOk`, `ifErr`, `require` |
| `/interop` | `fromNullable`, `toOption`, `toResult` at API boundaries |
| `/types` | `brand`, `tag`, `assertNever`, `refine`, `head` |
| `/record` | `copy`, `update`, `updatePath` |
| `/async` | `retry`, `withTimeout`, `race`, `parallel`, `asyncTraverse` |
| `/data/<fn>` | one remeda function per file (`filter`, `groupBy`, `sortBy`, `uniqueBy`, `find`, …) |
| `/log` | `logger.info`, `logger.withTag(...)` (required when `no-console: "error"`) |

Avoid in trivial sync helpers or simple DOM event handlers that cannot fail.

## Structure & Naming

Build atomic components inside `src/components/ui/`, scaling from single files to sub-atom folders as they grow, and place larger organisms in their own directories directly under `src/components/`:

```
src/components/
  ui/                   # domain-agnostic atoms & molecules (Kobalte/Ark UI/Corvu)
    button.tsx          # small primitive stays a single file
    combobox/           # outgrown component -> folder of internal atoms + hook + index.ts
      combobox.tsx
      combobox-trigger.tsx
      combobox-option-row.tsx
      use-combobox.ts
      index.ts
  calendar/             # organisms live in their own dir under src/components/
  block-editor/
  bottom-nav/
  layout/               # reusable page/shell/split-pane layouts
src/features/*          # domain logic, query hooks, composed feature views
src/forms/              # TanStack Form + Zod definitions
src/services/           # API / Tauri invoke / side-effect integrations
src/core/               # constants.ts, storage.ts, domain stores
src/context/            # readonly [state, actions] providers
src/routes/             # router views
src/lib/                # cn(), query-keys.ts, query-client.ts, i18n, helpers
```

- **Filenames**: `kebab-case` (`recipe-create-form.tsx`, `use-button.ts`); exported component symbols `PascalCase` (`RecipeCreateForm`).
- **Import boundaries**: `components/ui/**` must NEVER import `features/*`, fetch data, navigate, or read storage directly. Organism folders under `components/<name>/` may compose `components/ui` and headless hooks while staying clean of feature-specific business rules when reusable.

## SolidJS Tooling & Reactive Testing

This skill owns SolidJS-specific config extensions on top of `ts-styleguide` (details in [STYLEGUIDE.md](STYLEGUIDE.md)):

- **`tsconfig.json`**: `"jsx": "preserve"`, `"jsxImportSource": "solid-js"`, `"lib": ["ESNext", "DOM", "DOM.Iterable"]`.
- **`.oxlintrc.json`**: add `"jsPlugins": ["eslint-plugin-solid"]`, enable `solid/*` rules (`components-return-once`, `no-destructure`, `prefer-for`, `reactivity`, `event-handlers`, etc.), and allow `max-lines: 500` on `**/*.tsx`.
- **`bun:test` TSX preload**: configure `bunfig.toml` (`preload = ["./test/solid-tsx-preload.ts"]`) with `happy-dom`, `babel-preset-solid`, swapping `solid-js` server builds to `dist/dev.js` (so `Portal` works and unowned-root warnings remain observable for reactive-leak tests), and a network-blocking `fetch` stub.

## Reactivity & Contexts

- `createSignal` (local state), `createMemo` (derived values only), `createStore` (grouped state), `createResource` (reactive async), `createEffect` (imperative side effects only).
- Context stores: `createXContext` / `XProvider` / `useXContext` returning `readonly [state, actions]`.
- Storage contract: boot-critical prefs (`theme-init`, locale, sidebar) read synchronously via safe wrappers (`src/core/storage.ts`, zero FOUC); large domains use memory cache + IndexedDB (`localforage`). Never hardcode storage keys outside `src/core/constants.ts`.

Full guide, configs, `DESIGN.md` template, and UX checklist: [STYLEGUIDE.md](STYLEGUIDE.md).
