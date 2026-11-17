# Bun + Hono Reference Patterns

Code patterns for `bun-hono-fullstack`.

## 1. Chained App & Request-ID Middleware (`server/app.ts`)

```ts
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { newRequestId, logApi } from './logger';
import { rateLimit, securityHeaders } from './middleware';
import { itemsRoutes } from './routes/items';

const app = new Hono()
  .use('*', async (c, next) => {
    const requestId = newRequestId();
    const started = performance.now();
    await next();
    const ms = Math.max(0, Math.round(performance.now() - started));
    c.header('x-request-id', requestId);
    const line = `${c.req.method} ${c.req.path} -> ${c.res.status} ${ms}ms id=${requestId}`;
    if (c.res.status >= 400) logApi.warn(line);
    else logApi.debug(line);
  })
  .get('/healthz', (c) => c.text('OK'))
  .get('/api/healthz', async (c) => c.json(await getDetailedHealth()))
  .get('/api/version', (c) => c.json(getBuildInfo()))
  .use('*', securityHeaders())
  .use(
    '*',
    cors({
      origin: (origin) => (isAllowedOrigin(origin) ? origin : null),
      allowHeaders: ['Content-Type', 'Authorization'],
      allowMethods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
      credentials: true,
    }),
  )
  .use('/api/auth/*', rateLimit())
  .route('/api/items', itemsRoutes);

export type AppType = typeof app;
export default app;
```

## 2. Typed RPC Client (`src/lib/api.ts`)

```ts
import { hc } from 'hono/client';
import type { AppType } from '../../server/app';

export const api = hc<AppType>('/');
```

## 3. Timing-Safe Token Comparison (`server/timing-safe.ts`)

```ts
import { createHash, timingSafeEqual } from 'node:crypto';

function sha256(value: string): Buffer {
  return createHash('sha256').update(value, 'utf8').digest();
}

export function timingSafeTokenEqual(a: string, b: string): boolean {
  if (!a || !b) return false;
  return timingSafeEqual(sha256(a), sha256(b));
}
```

## 4. Production-Safe E2E Mount Guard (`server/e2e.ts` & `server/app.ts`)

Mount `/api/e2e/*` **only** when `E2E_AUTH_BYPASS === '1' && NODE_ENV !== 'production'`:

```ts
// server/e2e.ts — top-level guard
if (process.env.NODE_ENV === 'production') {
  throw new Error('server/e2e.ts must never be imported in production');
}
```

Unset in production, `/api/e2e/*` returns 404 by route absence.

## 5. Runtime-Agnostic Env Reader (`src/lib/runtime-env.ts`)

Shared server/browser/Vite modules (such as `server/logger.ts` or `vite.config.ts`) avoid raw `Bun.env` crashes in browser bundles by checking `globalThis` cleanly:

```ts
export function readRuntimeEnv(source: unknown = globalThis): Record<string, string | undefined> {
  if (typeof source !== 'object' || source === null) return {};
  const bunObj = Reflect.get(source, 'Bun');
  if (typeof bunObj === 'object' && bunObj !== null) {
    const bunEnv = Reflect.get(bunObj, 'env');
    if (typeof bunEnv === 'object' && bunEnv !== null) {
      return bunEnv as Record<string, string | undefined>;
    }
  }
  const procObj = Reflect.get(source, 'process');
  if (typeof procObj === 'object' && procObj !== null) {
    const procEnv = Reflect.get(procObj, 'env');
    if (typeof procEnv === 'object' && procEnv !== null) {
      return procEnv as Record<string, string | undefined>;
    }
  }
  return {};
}
```

## 6. Vite `build-info.json` + Dev Proxy Parity (`vite.config.ts`)

- Emit `build-info.json` (`{ version, buildTime }`) from `package.json` version + `BUILD_TIME` env (or build timestamp) so `/api/version` and static assets stay aligned.
- Proxy `/api/*` using `E2E_BACKEND_URL || 'http://localhost:7370'` in both `server.proxy` and `preview.proxy` so Tier C E2E tests work against `vite preview` with zero code changes.
- Any dev CORS proxy middleware must check same-origin (`Origin`/`Referer`) and run the shared `isBlockedFetchUrl` + DNS-pinned SSRF guard, disabled completely when `NODE_ENV === 'production'`.
