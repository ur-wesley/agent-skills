---
name: bun-testing-e2e
description: Bun-native 3-tier testing and zero-Playwright E2E workflow using bun:test, Hono app.request (Tier A), Bun.WebView smoke (Tier B), and isolated fullstack Bun.serve + Vite + Bun.WebView journeys (Tier C). Use when setting up or running E2E suites, fake/isolated test databases, e2e/run.ts, or E2E auth bypass seams. For SolidJS TSX unit test preload, see solidjs-ui.
---

# Bun-Native Testing & E2E (Zero Playwright)

Full-stack E2E without Playwright or manual terminal setup. Every tier runs via `bun test` and a cross-platform Bun runner (`e2e/run.ts`).

- **Generic `bun:test` rules (`void mock.module`)**: see `ts-styleguide`.
- **SolidJS TSX unit test preload (`happy-dom`, `babel-preset-solid`, `dev.js`)**: see `solidjs-ui`.
- **This skill**: 3-tier E2E architecture, isolated DBs, fake-by-default external APIs, `Bun.WebView` journeys, and production safety.

## Three E2E Tiers

| Tier | Entry | Target | Mechanism |
| --- | --- | --- | --- |
| **Tier A (API)** | `e2e/api.e2e.test.ts` | Hono `server/app` in-process | `app.request(...)` — reset, login, CRUD, 401/403 negatives |
| **Tier B (Smoke)** | `e2e/webview.e2e.test.ts` | Harness page or `E2E_WEB_URL` | `Bun.WebView` — login renders, bypass works, workspace boots |
| **Tier C (Fullstack)** | `e2e/fullstack.e2e.test.ts` + `e2e/*.ui.e2e.test.ts` | Real `Bun.serve` + real built frontend | `Bun.WebView` (`chrome`) + `vite preview` (dev-mode build) + `E2E_BACKEND_URL` proxy |

## Non-negotiable Rules

### 1. Cross-Platform Runner (`e2e/run.ts`)

- Never use Unix-only `VAR=1 cmd` syntax in `package.json` scripts (breaks Windows `pwsh`).
- Use `e2e/run.ts` with `Bun.spawnSync` and an explicit `env` map:
  - Creates a fresh `<tmp>/app-e2e-*/e2e.db` per run (`AUTH_DB_PATH`).
  - Runs each tier file in its **own `bun test` process** so module-level DB handles and fake stores never leak across tiers or into unit tests.
  - Passes `--timeout 300000` for Tier C files that boot Vite + backend.
  - Excludes shared helper modules (`e2e/ui-boot.ts`, `e2e/ui-helpers.ts`) from direct execution.

### 2. Fake-by-Default External Integrations

- CI runs **fake mode** by default (`E2E_MODE=fake`) with zero external network calls (block or throw on real third-party egress in tests).
- Real third-party integration tests stay opt-in (`E2E_MODE=real`, e.g., `bun run e2e:real`) and skip cleanly when test credentials are absent.

### 3. Tier C Determinism & Diagnostics

- Boot backend on an ephemeral `127.0.0.1:0` port and point the frontend proxy (`E2E_BACKEND_URL`) at it in `vite.config.ts` (`server.proxy` and `preview.proxy`).
- Canonical Tier C frontend build: `vite build --mode development` with `NODE_ENV=development` and `VITE_E2E_BYPASS=1` (keeps `import.meta.env.DEV` true in Vite 8+ so the dev-only bypass UI survives), served via `vite preview` (with `vite dev` fallback).
- Disable cookie caching in E2E server mode so `POST /api/e2e/reset` cleanly logs out the browser between journeys.
- Call `POST /api/e2e/reset` before every journey; use a fresh WebView profile (`localStorage` cleared at boot).
- Collect page `error` and `unhandledrejection` events per navigation and **fail the test** on any unexpected entry.
- On failure, capture a PNG screenshot + DOM HTML dump to `<tmp>/app-e2e/` and log the path.

### 4. Production Safety Gates (Mandatory)

- **Server seam (`server/e2e.ts`)**: throw at module top-level if `NODE_ENV === 'production'`. Mount `/api/e2e/*` in `server/app.ts` **only** when `E2E_AUTH_BYPASS === '1' && NODE_ENV !== 'production'`. Routes 404 by absence in prod.
- **DB guard**: refuse to boot E2E mode if `AUTH_DB_PATH` does not contain `e2e` (never touch `./data/auth.db`).
- **Frontend bypass (`VITE_E2E_BYPASS`)**: gate the bypass UI behind `import.meta.env.DEV && import.meta.env.VITE_E2E_BYPASS === '1'` so production `vite build` dead-code-eliminates every E2E string.

Runner template and checklist: [RUNNER.md](RUNNER.md).
