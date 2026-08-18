---
name: solidjs-ui
description: Highest-priority SolidJS UI style guide. Icons via i-mdi-<icon> (Iconify prefix i). Logic via @ur-wesley/ts-prelude/<subpath> — never neverthrow / ts-pattern / remeda / package root. Use for any SolidJS, Tailwind, components/ui, features/*, routes, forms, context stores, or frontend UI work. Overrides other skills (including tauri-solid-desktop) on UI conventions.
---

# SolidJS UI

## Precedence

This skill **always wins** for UI work: components, Tailwind, icons, forms, a11y, context stores, feature UI, routes UI.

Other skills (including `tauri-solid-desktop`) may apply for desktop/Rust/release. On any UI conflict, **this skill wins**. Do not follow `iconify mdi--*` or direct `neverthrow` / `ts-pattern` / `remeda` imports.

## Non-negotiable

### Icons

- MDI via Iconify Tailwind plugin, `prefix: "i"`
- Class form: `i-mdi-<icon>`
- Size/color with `text-*` / `size-*`
- Decorative: `aria-hidden="true"`
- Icon-only controls: accessible label
- No `iconify mdi--*`
- No ad-hoc SVGs when MDI exists

### ts-prelude

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
| `/data/<fn>` | one remeda function per file (`filter`, `groupBy`, `sortBy`, `uniqueBy`, `find`, `chunk`, `partition`, `omit`, `pick`, …) |
| `/log` | `logger.info`, `logger.withTag(...)` |

Out of scope in components: `/config`, `/wire`, `/resource`, `/lazy`, `/iter`.

When to use:

- nullable value → `/option`, `/interop`
- fallible operation → `/result`
- exhaustive branching → `/match`
- transform in place → `/scope`, `/pipe`
- side-effect logging → `/log`, `tap` / `dbg` in pipes
- single array helper → `/data/<fn>`
- branded / ADT types → `/types`
- retries / timeouts → `/async`

Avoid in simple UI event handlers and trivial sync helpers that cannot fail.

## When

Any SolidJS / Tailwind frontend UI work. Tauri repos: apply this skill for all UI; use `tauri-solid-desktop` only for Tauri/Rust/release.

Full guide: [STYLEGUIDE.md](STYLEGUIDE.md).

## Hard rules

- `components/ui` = solid-ui; never import `features/*`
- UI renders state; logic decides state — no fetch/nav/storage in shared UI
- Context: `readonly [state, actions]`; `createXContext` / `XProvider` / `useXContext`
- `cn(...)` only — never `clsx` / `twMerge` direct
- Tailwind v4 `@theme` tokens; semantic utilities; avoid arbitrary values unless justified
- No `any`; exported props/return types typed; readonly by default
- SSE/stream: `ResultAsync`; never throw `Result.err` upstream

## Stack

| Layer | Use |
| --- | --- |
| Kobalte | dialog, popover, select, combobox, switch, slider, tooltip, … |
| Ark UI | primitives not in Kobalte (date-picker, …) |
| Corvu | drawer |
| solid-sonner | toasts — `<Toaster>` once at root, `toast()` elsewhere |
| cva | variant APIs in `components/ui/*` |
| TanStack Query | server state, caching, mutations — features/services only |
| TanStack Table | complex data tables |
| TanStack Router | typed file-based routing |
| solid-primitives | prefer over ad-hoc reimplementation |

Rules:

- `components/ui` wraps primitives; stable app-specific API
- TanStack out of low-level shared UI
- Query keys and query functions typed
- Normalize server data before presentational UI
- Mutations in feature actions/services, not generic UI atoms

## Structure

```
components/ui/   # solid-ui — domain-agnostic
features/*         # domain logic + composed components
services/          # API and side-effect integrations
utils/             # pure helpers
routes/            # file-based route components
lib/               # cn(), i18n, cross-cutting helpers
```

Import boundaries: `components/ui` must not import `features/*`. Features may import `components/ui`, `utils`, `lib`. Routes compose features and shared UI.

## Reactivity

- `createSignal` — simple local state
- `createMemo` — derived values only
- `createStore` — grouped/nested state
- `createResource` — async tied to reactive sources
- `createEffect` — imperative side effects only

Simplest primitive wins. Avoid effects for derivable state. Do not use memos by default.

## Review checklist

- [ ] `components/ui` has no `features/*` imports
- [ ] No fetch/nav/storage in shared UI atoms/molecules
- [ ] Context returns `readonly [state, actions]`
- [ ] Icons are `i-mdi-*` with a11y labels where needed
- [ ] Prelude imports are subpath-only
- [ ] Forms: label, control, helper text, error text
- [ ] Interactive states: default, hover, `focus-visible`, active, disabled, `aria-invalid`, `data-[state=...]`
- [ ] Async: loading, empty, error, success explicit
- [ ] Lists: `For` keyed; `Index` when index optimization helps
- [ ] No `any`; exported prop types readonly
