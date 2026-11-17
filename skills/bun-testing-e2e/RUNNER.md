# Bun E2E Runner & Prod-Safety Reference

Implementation details for `bun-testing-e2e`.

## 1. `package.json` Scripts

```json
{
  "scripts": {
    "test": "bun test --timeout 30000",
    "e2e": "bun e2e/run.ts",
    "e2e:fullstack": "bun e2e/run.ts --fullstack",
    "e2e:real": "bun e2e/run.ts --real"
  }
}
```

## 2. Cross-Platform `e2e/run.ts` Pattern

```ts
#!/usr/bin/env bun
import { mkdtempSync } from 'node:fs';
import { tmpdir } from 'node:os';
import { dirname, join } from 'node:path';

const projectRoot = join(dirname(import.meta.path), '..');
const fullstackOnly = process.argv.includes('--fullstack');
const real = process.argv.includes('--real');
const scratch = mkdtempSync(join(tmpdir(), 'app-e2e-'));
const dbPath = join(scratch, 'e2e.db');

const excluded = new Set(['e2e/ui-boot.ts', 'e2e/ui-helpers.ts']);
const fakeFiles = [
  'e2e/api.e2e.test.ts',
  'e2e/webview.e2e.test.ts',
  'e2e/fullstack.e2e.test.ts',
];

const files = (fullstackOnly ? ['e2e/fullstack.e2e.test.ts'] : fakeFiles).filter(
  (f) => !excluded.has(f),
);

let failed = false;
for (const file of files) {
  const childEnv: Record<string, string> = {};
  for (const [k, v] of Object.entries(process.env)) {
    if (v !== undefined) childEnv[k] = v;
  }
  childEnv['NODE_ENV'] = 'test';
  childEnv['E2E_AUTH_BYPASS'] = '1';
  childEnv['E2E_MODE'] = real ? 'real' : 'fake';
  childEnv['AUTH_DB_PATH'] = dbPath;
  childEnv['BETTER_AUTH_SECRET'] =
    process.env.BETTER_AUTH_SECRET?.trim() || 'e2e-test-secret-0123456789abcdef0123456789';

  const needsBootTimeout = file === 'e2e/fullstack.e2e.test.ts' || file.endsWith('.ui.e2e.test.ts');
  const args = needsBootTimeout ? ['test', '--timeout', '300000', file] : ['test', file];
  const proc = Bun.spawnSync([process.execPath, ...args], {
    cwd: projectRoot,
    env: childEnv,
    stdin: 'ignore',
    stdout: 'inherit',
    stderr: 'inherit',
  });
  if (proc.exitCode !== 0) failed = true;
}

if (failed) process.exit(1);
```

## 3. Shared Tier C Helpers

- **`e2e/ui-boot.ts`**: starts `Bun.serve` on `127.0.0.1:0`, builds/starts `vite preview` (or `vite dev` fallback) with `E2E_BACKEND_URL` pointing to the ephemeral backend, opens `Bun.WebView` (`backend: { type: 'chrome' }`), attaches page error listeners, and handles teardown.
- **`e2e/ui-helpers.ts`**: exposes `resetAndLogin()`, `waitForSelector()`, `dumpFailureArtifacts()` (PNG + DOM HTML to `<tmp>/app-e2e/`).
- **`e2e/*.ui.e2e.test.ts`**: modular per-feature journeys using real `data-testid` hooks and reload persistence checks.

## 4. Env Matrix

| Var | Default | Purpose |
| --- | --- | --- |
| `E2E_AUTH_BYPASS` | unset | `1` mounts `/api/e2e/*` (never in prod) |
| `E2E_MODE` | `fake` | `fake` (in-memory stubs) vs `real` (opt-in throwaway creds) |
| `AUTH_DB_PATH` | `<tmp>/e2e.db` | Must include `e2e` in E2E mode |
| `VITE_E2E_BYPASS` | unset | `1` + `DEV` renders bypass login button |
| `E2E_BACKEND_URL` | `http://localhost:7370` | Vite `server.proxy` & `preview.proxy` target |

## 5. Prod-Safety Checklist

- [ ] `E2E_AUTH_BYPASS` unset in every production environment
- [ ] `NODE_ENV=production` in production (`server/e2e.ts` throws on import)
- [ ] `AUTH_DB_PATH` points to production volume (never `*e2e*`)
- [ ] `VITE_E2E_BYPASS` unset in prod builds (verify `dist/` contains no `e2e-bypass-login` / `E2E_AUTH_BYPASS`)
