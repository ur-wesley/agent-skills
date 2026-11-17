---
name: bun-hono-fullstack
description: Full-stack Bun + Hono backend, SQLite, better-auth, Zod v4 validation, structured logging, and security hardening (CSP, DNS-pinned SSRF guard, timing-safe tokens, health/version probes). Use when building or auditing Bun HTTP APIs, server/routes/*, server/db/*, auth, or full-stack SolidJS web apps.
---

# Bun + Hono Full-Stack Backend

Canonical server architecture for full-stack Bun + SolidJS web apps (modeled on `calplan`). Always pair with `ts-styleguide` (toolchain, `knip`, `no-console`) and `solidjs-ui` (frontend).

> **Alternative stack note:** When a project uses **TanStack Solid Start + Nitro + Drizzle** (`cairn` / `streamplan` style) instead of Hono + `bun:sqlite`, keep the same logging, Zod validation, `better-auth`, CSP/SSRF, and health/version principles while adapting routing to Nitro/server functions and schema to Drizzle.

## Architecture at a Glance

```
server/
  index.ts            # Bun.serve entrypoint + graceful shutdown + TTL sweeper boot
  app.ts              # Hono app wiring ONLY (chained routes for hono/client RPC types)
  auth.ts             # better-auth dynamic per-origin instances + SESSION_TTL_HOURS
  logger.ts           # @ur-wesley/ts-prelude/log tagged loggers + LOG_LEVEL + redaction
  middleware.ts       # requireSession, rateLimit, securityHeaders (CSP), DNS-pinned SSRF
  health.ts           # /api/healthz readiness checks (DB writable, upstream status)
  version.ts          # package.json version + BUILD_TIME (single source of truth)
  timing-safe.ts      # sha256 + crypto.timingSafeEqual for bearer/share tokens
  e2e.ts              # dev/test-only /api/e2e/* seam (throws on NODE_ENV=production)
  routes/*.ts         # modular Hono route factories mounted onto server/app.ts
  validators/*.ts     # Zod v4 schemas with explicit .max() bounds
  db/
    init.ts           # bun:sqlite WAL + 0600 perms + migrations runner
    migrations/       # ordered SQL/TS schema migrations
src/lib/
  ssrf.ts             # shared sync URL/host blocklist (imported by server & vite.config.ts)
  runtime-env.ts      # Bun.env-first, process.env-fallback, browser-safe env reader
  api.ts              # typed hc<AppType>() client
```

## Non-negotiable Rules

### 1. Chained Hono Wiring (`hono/client` RPC)

- Keep `server/app.ts` as wiring only under the `max-lines` budget.
- Chain `.route(...)` / `.get(...)` calls on `const app = new Hono()` in a single expression so `export type AppType = typeof app` preserves complete RPC inference for `hono/client`.

### 2. Structured Logging (Zero `console.*`)

- `no-console: "error"` applies everywhere. Use `@ur-wesley/ts-prelude/log` (`logger.withTag(...)`).
- Default `LOG_LEVEL` to `debug` in dev and `warn` in `NODE_ENV === 'production'`.
- First middleware in `server/app.ts` assigns `x-request-id` (`@paralleldrive/cuid2`), times the request, and logs `METHOD path -> status ms id=...` (`warn` for `>= 400`, `debug` otherwise).
- **Never log** headers, cookies, request bodies, tokens, secrets, or full emails — redact metadata and log opaque IDs/counts only.

### 3. Validation & Error Shapes

- Validate every request body/query with **Zod v4** schemas in `server/validators/*.ts`.
- Put explicit `.max(...)` bounds on every string and array (`ingredients.max(100)`, `tags.max(20)`, etc.).
- On validation failure, return `400` with `{ error, fields }` only — never leak raw stack traces or schema internals in production.
- Avoid existence oracles on auth/resource errors: `401` and `403` return identical generic `{ error }` shapes without revealing whether an ID exists.

### 4. Auth (`better-auth`) & Sessions

- Resolve per-request origin dynamically (`getAuthForOrigin`) sharing one DB handle so `localhost`, LAN, and Tailscale (`*.ts.net`) origins receive valid OAuth redirects and cookies.
- Configure `SESSION_TTL_HOURS` (with sane bounds and fallback) + sliding `updateAge`.
- Rate-limit `/api/auth/*` (e.g., 60 req/min/IP).
- In production (`NODE_ENV === 'production'`), throw at boot if `TRUSTED_ORIGINS` is missing; never allow wildcard CORS with `credentials: true`.

### 5. SQLite (`bun:sqlite`) & Hygiene

- Enable `PRAGMA journal_mode = WAL;` and best-effort `chmod 0600` on the DB file.
- Run bounded batch TTL sweepers (`LIMIT 500`) on an interval in `server/index.ts` (skip when `NODE_ENV === 'test'`).
- Never copy `-wal` files alone for backups; quiesce or snapshot atomically.

### 6. Security Hardening

- **CSP**: `script-src 'self'` only — **never** `unsafe-inline` scripts. Lock down `object-src 'none'`, `frame-ancestors 'none'`, `base-uri 'self'`, `form-action 'self'`, and mirror external `connect-src` origins with the PWA service worker allowlist (`src/lib/pwa-allowlist.ts`).
- **SSRF Guard**: share `isBlockedFetchUrl` from `src/lib/ssrf.ts` between Hono and Vite's dev proxy. On every fetch/redirect hop (capped at `MAX_PROXY_HOPS = 3`), DNS-resolve via `Bun.dns.lookup`, reject private/loopback/link-local IPs, and pin resolved IPs across redirects.
- **Timing-safe secrets**: compare share/webhook/API tokens via `sha256` + `timingSafeEqual` — never `===`.

### 7. Health & Version Probes

- `GET /healthz`: auth-free plain `200 OK` liveness probe (for Docker `HEALTHCHECK`).
- `GET /api/healthz`: auth-free JSON readiness (`{ ok, version, buildTime, uptime, dbWritable, ... }`).
- `GET /api/version`: reads `package.json` version (single source of truth) + `BUILD_TIME`; match with Vite's emitted `build-info.json`.

Patterns and code templates: [REFERENCE.md](REFERENCE.md).
