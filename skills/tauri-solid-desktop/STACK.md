# Stack Reference

Prescribed dependencies, structure, and code patterns for Tauri + SolidJS desktop apps.

## Required dependencies

### Runtime

- `solid-js`, `@tauri-apps/api`, `@tauri-apps/cli`

### UI

- `tailwindcss` v4, `@tailwindcss/vite` (or `@tailwindcss/postcss`), `tailwind-merge`, `clsx`, optional `class-variance-authority`
- `@kobalte/core`, `@ark-ui/solid`, `@corvu/drawer`, `cmdk-solid`, `solid-sonner`
- `@iconify/tailwind4`, `@iconify-json/mdi`
- Self-hosted `@fontsource/*` packages (zero Google Fonts requests)

### Logic

- `@ur-wesley/ts-prelude` (subpath imports per `solidjs-ui` — never package root)
- `@solid-primitives/*` as needed (event-bus, keyboard, i18n)

### Quality

- `oxlint`, `oxfmt`, `typescript`, `vitest`, `vite`, `vite-plugin-solid`

### Release

- `bumpp` (devDependency)
- `tauri-version` (optional — Cargo.lock sync)

### Tooling

- `packageManager: "bun@..."` in `package.json`
- `bun.lock` (no `package-lock.json`)
- `mise.toml` with `bun = "latest"`

### Optional (common in mature apps)

- `@tanstack/solid-query` — server/async state in features
- `@tanstack/solid-router` — typed routing

## Version files

Bump all three together on every release:

| File | Field |
| ---- | ----- |
| `package.json` | `"version"` |
| `src-tauri/Cargo.toml` | `version = "..."` in `[package]` |
| `src-tauri/tauri.conf.json` | `"version"` |

## bump.config.ts

```ts
import { defineConfig } from "bumpp";

export default defineConfig({
  tag: true,
  commit: true,
  push: true,
  tagName: "v%s",
  files: [
    "package.json",
    "src-tauri/Cargo.toml",
    "src-tauri/tauri.conf.json",
  ],
});
```

With Cargo.lock handling:

```ts
import { defineConfig } from "bumpp";
import tauri from "tauri-version";

export default defineConfig({
  all: true,
  execute: tauri(),
});
```

Install: `bun add -d bumpp tauri-version`

## package.json scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "oxlint",
    "format": "oxfmt .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "tauri": "tauri",
    "release": "bumpp",
    "release:patch": "bumpp patch",
    "release:minor": "bumpp minor",
    "release:major": "bumpp major"
  }
}
```

## Folder structure

```
src/
├── components/
│   ├── ui/            # atomic primitive wrappers (single file, or folder of atoms when complex)
│   ├── layout/        # reusable page/shell/split-pane layouts
│   └── <organism>/    # organisms in their own directories under src/components/
├── features/<name>/   # components, hooks, actions per feature
├── services/          # Tauri invoke wrappers (ResultAsync via ts-prelude /result)
├── lib/               # utils (cn), contexts
├── types/
└── index.css          # tailwindcss v4 + @theme inline + @iconify/tailwind4 plugin
src-tauri/
├── src/               # commands + domain modules
├── Cargo.toml
└── tauri.conf.json
mise.toml
bump.config.ts
```

## index.css / App.css (Tailwind 4 + Iconify)

```css
@import "tailwindcss";

@plugin "@iconify/tailwind4" {
  prefix: "i";
}
```

## Code snippets

### ResultAsync + Tauri invoke

```ts
import { invoke } from "@tauri-apps/api/core";
import { ResultAsync } from "@ur-wesley/ts-prelude/result";

export function tauriInvoke<T>(cmd: string, args?: Record<string, unknown>): ResultAsync<T, StableError> {
  return ResultAsync.fromPromise(invoke<T>(cmd, args), mapInvokeError);
}
```

Use in features/services — map errors to UI messages, never throw raw invoke errors.

### match on command result

```ts
import { match, P } from "@ur-wesley/ts-prelude/match";

match(result)
  .with({ status: "ok", data: P.select() }, (data) => setItems(data))
  .with({ status: "error", code: "NOT_FOUND" }, () => setEmpty())
  .with({ status: "error" }, ({ message }) => toast.error(message))
  .exhaustive();
```

### pipe data shaping

```ts
import { pipe } from "@ur-wesley/ts-prelude/pipe";
import { filter } from "@ur-wesley/ts-prelude/data/filter";
import { groupBy } from "@ur-wesley/ts-prelude/data/groupBy";

const activeByCategory = pipe(
  projects,
  filter((p) => p.isActive),
  groupBy((p) => p.category),
);
```

### MDI icon

```tsx
<span class="i-mdi-folder-open size-4 text-muted-foreground" />
```

Dynamic icon name:

```tsx
<span class={cn("size-4", `i-mdi-${iconName}`)} />
```

### Custom primitive component

```tsx
import { Button } from '../components/ui/button';

<Button variant="outline" size="sm" onClick={handleClick}>
  <span class="i-mdi-plus size-4" aria-hidden="true" />
  Add
</Button>
```

### Solid context store

```ts
const StoreContext = createContext<readonly [StoreState, StoreActions]>();

export function useStore(): readonly [StoreState, StoreActions] {
  const ctx = useContext(StoreContext);
  if (!ctx) throw new Error("useStore must be used within StoreProvider");
  return ctx;
}
```

### Thin Tauri command (Rust)

```rust
#[tauri::command]
pub async fn list_projects(state: State<'_, AppState>) -> Result<Vec<Project>, StableError> {
    projects::list(&state).await.map_err(Into::into)
}
```
