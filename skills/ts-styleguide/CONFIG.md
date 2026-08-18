# Config templates

Read when scaffolding or aligning a repo. Adjust package names and workspace entries as needed.

## Dependencies

```bash
bun add -D oxlint oxlint-tsgolint oxfmt lefthook typescript
```

TypeScript 7+ required for type-aware linting.

## package.json (single-package)

```json
{
  "name": "my-app",
  "private": true,
  "type": "module",
  "packageManager": "bun@1.2.0",
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "fmt": "oxfmt",
    "fmt:check": "oxfmt --check",
    "check": "oxlint && oxfmt --check",
    "prepare": "lefthook install"
  },
  "devDependencies": {
    "lefthook": "latest",
    "oxfmt": "latest",
    "oxlint": "latest",
    "oxlint-tsgolint": "latest",
    "typescript": "^7.0.0"
  }
}
```

## package.json (monorepo root)

```json
{
  "name": "my-monorepo",
  "private": true,
  "type": "module",
  "packageManager": "bun@1.2.0",
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "fmt": "oxfmt",
    "fmt:check": "oxfmt --check",
    "check": "oxlint && oxfmt --check",
    "prepare": "lefthook install"
  },
  "devDependencies": {
    "lefthook": "latest",
    "oxfmt": "latest",
    "oxlint": "latest",
    "oxlint-tsgolint": "latest",
    "typescript": "^7.0.0"
  }
}
```

## .oxlintrc.json

```jsonc
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "options": {
    "typeAware": true,
    "typeCheck": true
  },
  "categories": {
    "correctness": "error",
    "suspicious": "error"
  },
  "ignorePatterns": ["dist", "build", "coverage", "node_modules"],
  "overrides": [
    {
      "files": ["**/*.{ts,tsx}"],
      "rules": {
        "typescript/no-explicit-any": "error"
      }
    }
  ]
}
```

Do not set `plugins` unless adding — it replaces Oxlint defaults.

## .oxfmtrc.jsonc

```jsonc
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "printWidth": 80,
  "ignorePatterns": ["dist", "build", "coverage", "node_modules"]
}
```

## lefthook.yml

```yaml
pre-commit:
  commands:
    fmt:
      glob: "*.{ts,tsx,js,jsx,mjs,cjs,json,jsonc,md}"
      run: bunx oxfmt --write {staged_files}
      stage_fixed: true
    lint-fix:
      glob: "*.{ts,tsx,js,jsx,mjs,cjs}"
      run: bunx oxlint --fix {staged_files}
      stage_fixed: true

pre-push:
  commands:
    check:
      run: bun run check
```

## tsconfig.json (single-package)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true,
    "noEmit": true
  },
  "include": ["src"]
}
```

## tsconfig.base.json (monorepo)

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true,
    "noEmit": true,
    "composite": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

## tsconfig.json (monorepo root)

```json
{
  "files": [],
  "references": [
    { "path": "./apps/web" },
    { "path": "./packages/shared" }
  ]
}
```

## tsconfig.json (monorepo app, e.g. apps/web)

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": false,
    "declaration": false,
    "declarationMap": false,
    "noEmit": true,
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src"]
}
```

## tsconfig.json (monorepo library, e.g. packages/shared)

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src"]
}
```

Build libraries before type-aware lint in monorepos:

```bash
bun run --filter './packages/*' build
bun run check
```

## .vscode/settings.json

```json
{
  "editor.defaultFormatter": "oxc.oxc-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnSaveMode": "file",
  "[typescript]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[jsonc]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  }
}
```

Install the `oxc.oxc-vscode` extension. Oxlint and oxfmt must be local devDependencies.

## .gitignore (recommended)

```
node_modules
dist
build
coverage
*.tsbuildinfo
```
