# Config Templates

Read when scaffolding or aligning a generic TypeScript repo. Adjust package names and workspace entries as needed. For SolidJS JSX/lint/test-preload extensions, see `solidjs-ui`.

## Dependencies

```bash
bun add -D oxlint oxlint-tsgolint oxfmt knip lefthook typescript @types/bun
```

TypeScript 7+ required for type-aware linting.

## mise.toml

```toml
[tools]
bun = "latest"
```

## package.json (single-package)

```json
{
  "name": "my-app",
  "private": true,
  "type": "module",
  "packageManager": "bun@1.4.4",
  "engines": {
    "bun": ">=1.4.0"
  },
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "fmt": "oxfmt",
    "fmt:check": "oxfmt --check",
    "check": "oxlint && oxfmt --check",
    "deadcode": "knip",
    "check:all": "oxlint && oxfmt --check && knip && bun run test",
    "prepare": "lefthook install"
  },
  "devDependencies": {
    "@types/bun": "latest",
    "knip": "latest",
    "lefthook": "latest",
    "oxfmt": "latest",
    "oxlint": "latest",
    "oxlint-tsgolint": "latest",
    "typescript": "^7.0.2"
  }
}
```

## package.json (monorepo root)

```json
{
  "name": "my-monorepo",
  "private": true,
  "type": "module",
  "packageManager": "bun@1.4.4",
  "engines": {
    "bun": ">=1.4.0"
  },
  "workspaces": ["apps/*", "packages/*"],
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "fmt": "oxfmt",
    "fmt:check": "oxfmt --check",
    "check": "oxlint && oxfmt --check",
    "deadcode": "knip",
    "prepare": "lefthook install"
  },
  "devDependencies": {
    "@types/bun": "latest",
    "knip": "latest",
    "lefthook": "latest",
    "oxfmt": "latest",
    "oxlint": "latest",
    "oxlint-tsgolint": "latest",
    "typescript": "^7.0.2"
  }
}
```

## .oxlintrc.json (Lean Baseline)

Use for small libraries, CLIs, or initial scaffolding:

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
  "rules": {
    "no-console": "error"
  },
  "ignorePatterns": ["dist", "build", "coverage", "node_modules", "bun.lock"],
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

Do not set `plugins` unless intentionally listing all active plugins — setting `plugins` replaces Oxlint defaults.

## .oxlintrc.json (Production App Profile)

Use for mature, multi-contributor, or AI-assisted apps. Pair `no-console: "error"` with `@ur-wesley/ts-prelude/log`:

```jsonc
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "plugins": ["typescript", "unicorn", "oxc", "import", "jsdoc", "vitest"],
  "options": {
    "typeAware": true,
    "typeCheck": true
  },
  "categories": {
    "correctness": "error",
    "suspicious": "error",
    "pedantic": "error"
  },
  "rules": {
    "no-unused-vars": "error",
    "no-console": "error",
    "no-alert": "error",
    "no-nested-ternary": "error",
    "unicorn/no-nested-ternary": "error",
    "max-depth": ["error", { "max": 4 }],
    "max-params": ["error", { "max": 4 }],
    "max-lines": ["error", { "max": 400, "skipBlankLines": true, "skipComments": true }],
    "max-lines-per-function": "off",
    "no-inline-comments": "off",
    "import/max-dependencies": "off",
    "import/no-unassigned-import": "off",
    "unicorn/prefer-query-selector": "off",
    "typescript/no-explicit-any": "error",
    "typescript/no-non-null-assertion": "error",
    "typescript/consistent-type-imports": "error",
    "typescript/no-floating-promises": "error",
    "typescript/no-misused-promises": "error",
    "typescript/await-thenable": "error",
    "typescript/no-unnecessary-type-assertion": "warn",
    "typescript/no-unsafe-assignment": "warn",
    "typescript/no-unsafe-argument": "warn",
    "typescript/no-unsafe-call": "warn",
    "typescript/no-unsafe-member-access": "warn",
    "typescript/no-unsafe-return": "warn",
    "typescript/no-unsafe-type-assertion": "warn",
    "typescript/prefer-nullish-coalescing": "off",
    "typescript/prefer-readonly-parameter-types": "off",
    "typescript/no-confusing-void-expression": "off",
    "typescript/strict-boolean-expressions": "off",
    "typescript/strict-void-return": "off",
    "typescript/array-type": "off",
    "import/first": "error",
    "import/no-named-export": "off",
    "import/group-exports": "off",
    "import/no-default-export": "off",
    "import/unambiguous": "off",
    "no-underscore-dangle": "off",
    "unicorn/no-useless-undefined": "off"
  },
  "env": {
    "builtin": true
  },
  "ignorePatterns": ["dist/**", "build/**", "coverage/**", "node_modules/**", "bun.lock"],
  "overrides": [
    {
      "files": ["**/*.test.{ts,tsx}", "**/*.spec.{ts,tsx}"],
      "rules": {
        "vitest/no-conditional-in-test": "off",
        "typescript/no-unsafe-type-assertion": "off",
        "typescript/no-unnecessary-type-assertion": "off",
        "typescript/restrict-template-expressions": "off",
        "typescript/no-base-to-string": "off",
        "max-lines": "off"
      }
    }
  ]
}
```

## .oxfmtrc.json

```json
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 100,
  "singleQuote": true,
  "semi": true,
  "trailingComma": "all",
  "ignorePatterns": ["dist/**", "build/**", "coverage/**", "node_modules/**", "bun.lock"]
}
```

(`.oxfmtrc.jsonc` with `printWidth: 80` is also valid — stay consistent per repo.)

## knip.json

```json
{
  "$schema": "https://unpkg.com/knip@6/schema.json",
  "entry": ["src/index.ts", "scripts/**/*.ts", "test/**/*.ts", "src/**/*.test.ts"],
  "project": ["src/**/*.ts", "scripts/**/*.ts", "test/**/*.ts"],
  "ignore": [],
  "ignoreDependencies": [],
  "rules": {
    "files": "error",
    "dependencies": "error",
    "devDependencies": "error",
    "unlisted": "error",
    "binaries": "error",
    "unresolved": "error",
    "exports": "error",
    "nsExports": "error",
    "types": "error",
    "nsTypes": "error",
    "enumMembers": "error",
    "namespaceMembers": "error",
    "duplicates": "error"
  }
}
```

## lefthook.yml

Read-only parallel pre-commit (recommended for strict app repos):

```yaml
pre-commit:
  parallel: true
  jobs:
    - name: oxlint staged
      glob: '*.{js,jsx,ts,tsx}'
      run: bunx oxlint {staged_files}
    - name: oxfmt check staged
      glob: '*.{js,jsx,ts,tsx,json,jsonc,css,md}'
      run: bunx oxfmt --check {staged_files}
pre-push:
  jobs:
    - name: check
      run: bun run check
```

Auto-fix alternative (`stage_fixed: true`):

```yaml
pre-commit:
  commands:
    fmt:
      glob: '*.{ts,tsx,js,jsx,mjs,cjs,json,jsonc,css,md}'
      run: bunx oxfmt --write {staged_files}
      stage_fixed: true
    lint-fix:
      glob: '*.{ts,tsx,js,jsx,mjs,cjs}'
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
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "allowImportingTsExtensions": true,
    "skipLibCheck": true,
    "noEmit": true,
    "types": ["bun"]
  },
  "include": ["src"]
}
```

For full-stack single repos, scope `include` explicitly to source roots (`["src", "server", "scripts", "test", "e2e"]`).

## Monorepo tsconfig templates

`tsconfig.base.json`:

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "allowImportingTsExtensions": true,
    "skipLibCheck": true,
    "noEmit": true,
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "types": ["bun"]
  }
}
```

`tsconfig.json` (monorepo root):

```json
{
  "files": [],
  "references": [{ "path": "./apps/web" }, { "path": "./packages/shared" }]
}
```

`tsconfig.json` (monorepo app, e.g. `apps/web`):

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

`tsconfig.json` (monorepo library, e.g. `packages/shared`):

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
  "[typescript]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[typescriptreact]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[javascript]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[javascriptreact]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[json]": { "editor.defaultFormatter": "oxc.oxc-vscode" },
  "[jsonc]": { "editor.defaultFormatter": "oxc.oxc-vscode" }
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
