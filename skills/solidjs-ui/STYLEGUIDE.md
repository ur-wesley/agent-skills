# SolidJS UI Style Guide

Conventions for building scalable, fast, type-safe, accessible UIs with SolidJS, Tailwind CSS v4, and headless primitives (`@kobalte/core`, `@ark-ui/solid`, `@corvu/drawer`, `cmdk-solid`, `solid-sonner`).

---

## 1. Core Principles

- **Own your component layer**: wrap `@kobalte/core` / `@ark-ui/solid` / `@corvu/drawer` in `src/components/ui/` using atomic composition (single files for small atoms, growing into `src/components/ui/<name>/` folders of internal atoms + `use-<name>.ts` hooks + `index.ts` when a component outgrows one file). Place organisms in their own directories under `src/components/<organism>/`. Build directly on primitives rather than third-party component generators.
- **Flat visual hierarchy**: max 1 level of visual containment; never nest cards inside cards.
- **Spacing & typography over borders**: let Gestalt proximity, alignment, and type scale organize the screen before reaching for `1px` borders or heavy shadows.
- **Tokens & class guard**: map runtime CSS vars (`--theme-*`, `--cp-*`) via `@theme inline` in `src/index.css` and enforce semantic classes via `scripts/class-guard.ts`.
- **Strict separation**: UI renders state; logic decides state. `components/ui/**` stays 100% domain-agnostic.
- **Strong typing & immutability**: no `any`, explicit exported prop/return types, `Readonly<T>`, `readonly T[]`, `readonly [state, actions]` context tuples, and `@ur-wesley/ts-prelude/<subpath>`.

---

## 2. UX & Layout Craft (Avoiding Visual Noise)

### 2.1 Flat Spatial Hierarchy (No Deeply Nested Containers)

- **Ban the "Card-in-Card" anti-pattern**: never wrap a page in a container card, place section cards inside it, and put bordered sub-cards inside those ("UI Russian nesting dolls").
- **Max 1 level of containment**: a view may group related items into a single surface/card, or keep the canvas flat with background sections — never nest bordered cards inside cards unless functionally required (e.g., an embedded interactive widget or distinct drag item).
- **Spacing over borders**:
  - Use generous gaps (`gap-6`, `gap-8`) between sections and tighter gaps (`gap-2`, `gap-3`) within a group (Gestalt Law of Proximity).
  - Use typographic contrast (weight, size, muted secondary text) and subtle tonal shifts (`bg-surface` vs `bg-card`) rather than `1px` grid lines around every block.
  - **Box audit rule**: count the visible rectangles on any screen; if removing a border or card wrapper leaves the grouping clear via alignment and whitespace, delete the wrapper.

### 2.2 Visual Hierarchy & Elevation Restraint

- **1–2 focal points per view**: limit high-contrast primary fills (`bg-accent text-on-accent`) to primary actions; secondary actions stay ghost/outline/tonal so they don't compete.
- **Scroll-plane performance**: keep high-density list/grid/calendar items synthetic (backgrounds, gradients, hairlines) with **zero `backdrop-filter`** and **zero per-card container queries**. Reserve frosted blur/elevation for floating chrome (toolbars, dialogs, popovers, toasts).
- **Squircle / consistent radii**: stick to `DESIGN.md` radius tokens (`sm`, `md`, `lg`, `pill`). Never invent ad-hoc `rounded-[13px]` values.

### 2.3 Interaction Craft & Perceived Performance

- **No overloaded click triggers**: never assign single-click to select/open and double-click to edit on the same target (forces a 250–400ms browser disambiguation delay). Give secondary actions dedicated targets.
- **0ms `pointerdown` feedback, forgiving `click` commit**: depress/highlight immediately on `pointerdown`; resolve navigation/mutation on `pointerup`/`click` so dragging off cancels safely.
- **Action-first + Undo over modal fatigue**:
  - For reversible actions (toggle, archive, move, delete item), mutate optimistically at `0ms`, dispatch in the background, and show a `toast()` with an **Undo** action (or rollback + retry toast on failure).
  - For truly destructive/irreversible actions, use an inline `HoldToConfirmButton` or a typed-confirmation dialog rather than reflexive "Are you sure?" popups.
- **Smart tooltip warm-up/cool-down**: group toolbar tooltips so the first hover waits `~500ms` (cold), moving to sibling buttons opens at `0ms` (warm), and leaving resets after `~500ms` (cool-down).
- **Fitts's Law hitboxes**: keep small `16px` visual icons inside `>= 44×44px` interactive hit targets on mobile (via padding or pseudo-elements).
- **Perceived speed windows**:
  - `0–100ms`: instant active state / optimistic toggle.
  - `100–300ms`: crisp transform/opacity transitions (no layout-thrashing animations).
  - `300ms–1s`: layout-matching skeleton screens instead of generic centered spinners.
  - `> 1s`: non-blocking progress indicator while keeping the rest of the workspace interactive.
- **Progressive disclosure**: keep primary views uncluttered; move advanced options, bulk tools, and secondary settings into popovers, drawers (`@corvu/drawer`), sheets, or `cmdk-solid` command palettes.

### 2.4 Mobile, Viewport & Accessibility Polish

- App shell: `100dvh` (`100vh` fallback); scroll containers below a floating bottom nav carry `pb-nav-safe` (`max(5.5rem, env(safe-area-inset-bottom) + 4.5rem)`).
- Mobile inputs/command inputs: `>= 16px` font size on touch viewports to prevent iOS Safari auto-zoom.
- Semantic HTML first (`<button>`, `<nav>`, `<main>`, `<label>`); never clickable `<div>`s.
- Keyboard accessible everywhere (`Tab`, `Enter`, `Space`, `Escape`), visible `focus-visible` rings, WCAG `4.5:1` text contrast, never color-only state, and `@media (prefers-reduced-motion: reduce)` support.

---

## 3. SolidJS Tooling Extensions (Owned by This Skill)

Add these on top of `ts-styleguide` in every SolidJS repo.

### 3.1 `tsconfig.json` SolidJS Additions

```json
{
  "compilerOptions": {
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "jsx": "preserve",
    "jsxImportSource": "solid-js"
  }
}
```

### 3.2 `.oxlintrc.json` SolidJS Additions

Install `eslint-plugin-solid` and enable Solid rules + `.tsx` line budget:

```jsonc
{
  "plugins": ["typescript", "unicorn", "oxc", "import", "jsdoc", "vitest"],
  "jsPlugins": ["eslint-plugin-solid"],
  "rules": {
    "solid/components-return-once": "error",
    "solid/event-handlers": "error",
    "solid/imports": "error",
    "solid/jsx-no-duplicate-props": "error",
    "solid/jsx-no-script-url": "error",
    "solid/jsx-no-undef": "error",
    "solid/jsx-uses-vars": "error",
    "solid/no-destructure": "error",
    "solid/no-innerhtml": "error",
    "solid/no-react-deps": "error",
    "solid/no-react-specific-props": "error",
    "solid/no-unknown-namespaces": "error",
    "solid/prefer-for": "error",
    "solid/reactivity": "error",
    "solid/self-closing-comp": "error",
    "solid/style-prop": "error"
  },
  "overrides": [
    {
      "files": ["**/*.tsx"],
      "rules": {
        "max-lines": ["error", { "max": 500, "skipBlankLines": true, "skipComments": true }]
      }
    }
  ]
}
```

### 3.3 `bun:test` Solid TSX Preload (`bunfig.toml` + `test/solid-tsx-preload.ts`)

`bun test` defaults to Solid's `server` build (where `render`/`Portal` throw) and uses React-style `jsxDEV` transforms. Wire `bunfig.toml`:

```toml
[test]
preload = ["./test/solid-tsx-preload.ts"]
```

In `test/solid-tsx-preload.ts` (requires `happy-dom`, `@babel/core`, `@babel/plugin-syntax-jsx`, `@babel/preset-typescript`, `babel-preset-solid`):

1. Install `happy-dom` `Window`, `document`, `navigator`, and DOM element constructors (`HTMLDivElement`, `HTMLHeadElement`, `SVGElement`, `MutationObserver`, `ResizeObserver`, `requestAnimationFrame`, etc.) onto `globalThis` before any test loads.
2. Stub `fetch` to reject unexpected external URLs so unit tests never leak network calls.
3. Register a `Bun.plugin` `onLoad`:
   - Compile `*.tsx` with `babel-preset-solid` (`{ generate: 'dom', hydratable: false }`). Keep bare `solid-js*` specifiers untouched so module identity never forks.
   - Intercept `solid-js/(dist|web/dist|store/dist)/server.js` and serve the **diagnostic `dev.js` client builds** (`dist/dev.js`, `web/dist/dev.js`, `store/dist/dev.js`). Using `dev.js` rather than production `solid.js` keeps unowned-root warnings observable for reactive-leak tests (`computations/cleanups created outside a createRoot`).

---

## 4. Tech Stack & Headless Primitives

- **Primitives**: `@kobalte/core`, `@ark-ui/solid`, `@corvu/drawer`, `@corvu/calendar`, `cmdk-solid`, `solid-sonner`. Wrap them inside your own component directory with stable app-specific prop APIs. Never render raw `<select>`/`<dialog>` when a primitive covers keyboard navigation, focus trapping, and ARIA.
- **Variants & Class Composition**:
  - `cn(...)` in `src/lib/utils.ts` (`clsx` + `tailwind-merge`). Components call `cn(...)` only — never import `clsx`/`twMerge` directly or join class arrays manually.
  - `cva` (`class-variance-authority`) is recommended for primitives with `>= 3` orthogonal variants; for simpler 1–2 variant components, a small typed `Record<Variant, string>` map avoids extra dependencies (`knip` clean).
- **TanStack Suite**:
  - `@tanstack/solid-query` with centralized `src/lib/query-keys.ts` and `src/lib/query-client.ts`. Keep queries/mutations in `features/*` or `services/*` — never inside shared UI primitives. Normalize server DTOs before passing to presentational components.
  - `@tanstack/solid-form` + `zod` for typed forms (`src/forms/` or `features/*/forms`).
  - `@tanstack/solid-table` for complex grids/tables.
  - `@tanstack/solid-router` for typed routes.
- **`solid-primitives`**: prefer `@solid-primitives/keyboard`, `@solid-primitives/i18n`, `@solid-primitives/storage`, `@solid-primitives/geolocation`, `@solid-primitives/timer`, `@solid-primitives/media`, `@solid-primitives/resize-observer` over hand-rolled DOM/listener boilerplate.

---

## 5. Styling, Tokens, Typography & Class Guard

### 5.1 `@theme` + `@theme inline` (`src/index.css`)

Declare self-hosted fonts and static scales in `@theme`, and map dynamic theme CSS variables (`--theme-*`, `--cp-*`) in `@theme inline` so Tailwind v4 generates semantic utilities with opacity support (`bg-accent/12`, `text-fg-2`, `border-line`):

```css
@import '@fontsource/ibm-plex-sans-condensed/latin-400.css';
@import '@fontsource-variable/jetbrains-mono/index.css';
@import 'tailwindcss';

@plugin "@iconify/tailwind4" {
  prefix: 'i';
}

@custom-variant dark (&:where(.dark, .dark *, [data-theme*="dark"], [data-theme*="dark"] *));

@theme {
  --font-sans: 'IBM Plex Sans Condensed', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono Variable', ui-monospace, monospace;
}

@theme inline {
  --color-fg: var(--theme-text-primary);
  --color-fg-2: var(--theme-text-secondary);
  --color-fg-muted: var(--theme-text-muted);
  --color-accent: var(--theme-accent);
  --color-on-accent: var(--theme-on-accent);
  --color-danger: var(--theme-danger);
  --color-line: var(--theme-border);
  --color-card: var(--theme-card-surface);
  --color-glass: var(--theme-glass-surface);
  --color-surface: var(--theme-surface);
  --color-background: var(--theme-background);
}

@utility tnum {
  font-variant-numeric: tabular-nums lining-nums;
}
```

### 5.2 Automated Class Guard (`scripts/class-guard.ts`)

Run in `check:all` (`bun scripts/class-guard.ts`) over `src/**/*.tsx` (excluding `*.test.tsx`) to fail CI on:

- Arbitrary mapped var classes like `text-[var(--theme-text-primary)]` (use `text-fg`).
- Arbitrary hex classes like `bg-[#005FCC]` (define a token first).
- `color-mix(...)` in JSX classes referencing mapped vars (use `bg-accent/12`).
- Tiny illegible text classes (`text-[8px]`, `text-[9px]`, `text-[10px]`).

### 5.3 Icons

- MDI via `@iconify/tailwind4` (`prefix: "i"`): `<span class="i-mdi-plus size-4" aria-hidden="true" />`.
- Dynamic icons: `<span class={cn('size-4', `i-mdi-${props.icon}`)} aria-hidden="true" />`.
- Icon-only buttons must include `aria-label` (or `<span class="sr-only">`).

---

## 6. Project Structure, Naming & Import Boundaries

### 6.1 File & Symbol Naming

- **Filenames**: `kebab-case` everywhere under `src/**` (`recipe-create-form.tsx`, `use-button.ts`, `date-format.ts`). Never `RecipeCreateForm.tsx`.
- **Component exports**: `PascalCase` (`RecipeCreateForm`, `HoldToConfirmButton`).
- **Hooks/factories/context**: `createX`, `useX`, `getX`.
- **Actions**: verb-first (`toggle`, `open`, `close`, `setItems`, `submit`).

### 6.2 Atomic Growth & Import Boundaries

- **Atoms & molecules (`src/components/ui/`)**: start small primitives as single `kebab-case` files (`src/components/ui/button.tsx`). When a component outgrows one file (multiple sub-parts, variants, or stateful logic), promote it into a folder `src/components/ui/<name>/` containing its internal atoms (`<name>-trigger.tsx`, `<name>-row.tsx`), a `use-<name>.ts` hook for testable logic, and `index.ts`.
- **Organisms (`src/components/<organism>/`)**: larger multi-part compositions (e.g., `src/components/calendar/`, `src/components/block-editor/`, `src/components/wizard/`, `src/components/bottom-nav/`) live in their own directories directly under `src/components/` rather than inside `components/ui/`.
- **Layouts (`src/components/layout/`)**: reusable layout shells and templates (app shell, split-pane, sidebar frame, page headers) live in `src/components/layout/`, keeping layout structure separate from feature logic.
- `components/ui/**` must NOT import `features/*`, `routes/*`, `services/*`, or storage/fetch directly.
- `features/*` may import `components/*`, `core/*`, `utils/*`, `lib/*`, `services/*`.
- `routes/*` compose `features/*` and layout chrome.

---

## 7. Component, Reactivity & Storage Rules

- **Props**: export `interface XProps` (or `type`), mark fields `readonly`, never destructure reactive props (`solid/no-destructure`), forward with `splitProps` / `mergeProps` / `children(() => props.children)`.
- **Reactivity**: simplest primitive wins (`createSignal` > `createMemo` > `createStore` > `createResource` > `createEffect`). Never use `createEffect` to mirror derivable state.
- **Lists & control flow**: keyed `<For each={...}>` (or `<Index>` when index stability helps), `<Show>`, `<Switch>/<Match>`.
- **Context stores**: `createXContext` / `XProvider` / `useXContext` returning `readonly [state, actions]`. Throw if used outside provider.
- **Storage contract**:
  - **Sync small prefs** (theme, locale, sidebar state): read synchronously via safe `localStorage` wrappers (`src/core/storage.ts`, `src/theme-init.ts`) so first paint has zero FOUC.
  - **Large domain caches** (events, items, offline docs): session memory cache + IndexedDB (`localforage`).
  - **Single-source keys**: all storage key names live in `src/core/constants.ts` — never scatter raw `'app:*'` strings.
- **Error boundaries & async states**: wrap route/feature boundaries in `<ErrorBoundary>`, handle loading/empty/error/success explicitly, and wrap SSE/streams in `ResultAsync` (`/result`) — never `throw` a `Result.err` upstream.

---

## 8. `DESIGN.md` Living Spec Template

Every UI app should keep a root `DESIGN.md` combining machine-readable YAML frontmatter tokens with concise architectural rationale:

```markdown
---
version: 1.0
name: AppName
description: One-line product & visual system summary.
colors:
  primary: '#005FCC'
  on-primary: '#FFFFFF'
  text-primary: '#1D1D1F'
  surface-card: 'rgba(255, 255, 255, 0.7)'
typography:
  body: { fontFamily: '...', fontSize: '13px', fontWeight: 400, lineHeight: 1.4 }
rounded: { sm: '7px', md: '8px', lg: '16px', pill: '999px' }
spacing: { sm: '8px', md: '16px', lg: '24px' }
---

## Overview & Dark Mode
## Colors & Contrast
## Typography
## Layout & Flat Containment
## Elevation & Shapes
## Do's and Don'ts
```

---

## 9. Review Checklist

- [ ] Flat hierarchy: max 1 containment level; spacing/typography used instead of nested boxes
- [ ] Bespoke components wrapped directly from Kobalte / Ark UI / Corvu / `solid-sonner`
- [ ] All filenames `kebab-case`; exported components `PascalCase`; props `readonly` + typed
- [ ] `components/ui/**` has zero `features/*` / fetch / storage imports; organisms live in `src/components/<organism>/`
- [ ] `@theme inline` tokens used (`class-guard.ts` passes with 0 arbitrary var/hex/tiny-text violations)
- [ ] Self-hosted `@fontsource/*` fonts; `i-mdi-*` icons with `aria-hidden="true"` or labels
- [ ] No overloaded single/double click targets; `pointerdown` feedback; optimistic/undo or hold-to-confirm
- [ ] `eslint-plugin-solid` clean; `bun:test` TSX preload + reactive-leak checks green
- [ ] `@ur-wesley/ts-prelude/<subpath>` imports only (no root, no direct `neverthrow`/`ts-pattern`/`remeda`)
