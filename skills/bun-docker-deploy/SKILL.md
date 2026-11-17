---
name: bun-docker-deploy
description: Production Docker packaging for Bun full-stack apps using multi-stage oven/bun-alpine builds, bun build --compile --bytecode standalone binaries, minimal Alpine runners with tini and su-exec, /healthz probes, and non-root volume permissions. Use when creating or auditing Dockerfile, docker-entrypoint.sh, .dockerignore, or container deploy setup.
---

# Bun + Docker Production Deployment

Multi-stage Docker packaging that compiles a Bun backend into a standalone executable paired with static Vite assets inside a minimal Alpine runtime image.

## Architecture

1. **Stage 1 (`builder`)**: `oven/bun:<pinned>-alpine` matching `packageManager` in `package.json`.
   - Copy `package.json` + `bun.lock` first for layer caching.
   - Install with `bun install --frozen-lockfile --ignore-scripts`.
   - Build frontend (`bun run build`).
   - Compile backend (plus any worker entrypoints) with `bun build ... --compile --minify --sourcemap --bytecode --format=esm --outfile /app/<app>`.
2. **Stage 2 (`runner`)**: minimal `alpine:3.21` (no Bun runtime needed in prod).
   - Install only runtime essentials: `curl`, `tzdata`, `tini`, `su-exec`, `libstdc++`, `libgcc`.
   - Create non-root `bun:bun` (`1000:1000`) user and writable `/app/data` `VOLUME`.
   - `ENTRYPOINT ["/sbin/tini", "--", "./docker-entrypoint.sh"]` + `STOPSIGNAL SIGTERM`.
   - `HEALTHCHECK` against `GET /healthz` (auth-free plain liveness probe; keep `/api/healthz` for detailed JSON readiness checks).

## Templates

### `Dockerfile`

```dockerfile
# Stage 1: Build (pin version to packageManager bun@x in package.json)
FROM oven/bun:1.4.4-alpine AS builder
WORKDIR /app

COPY package.json bun.lock ./
RUN bun install --frozen-lockfile --ignore-scripts

COPY . .
RUN bun run build

RUN bun build server/index.ts \
  --compile --minify --sourcemap --bytecode --format=esm \
  --outfile /app/app

# Stage 2: Production runner (standalone executable only)
FROM alpine:3.21 AS runner
WORKDIR /app

RUN apk add --no-cache curl tzdata tini su-exec libstdc++ libgcc \
  && addgroup -g 1000 bun \
  && adduser -D -u 1000 -G bun bun

ENV NODE_ENV=production \
    PORT=7369 \
    HOST=0.0.0.0 \
    AUTH_DB_PATH=/app/data/auth.db

COPY package.json ./
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/app ./app
RUN chmod +x ./app

COPY docker-entrypoint.sh ./
RUN chmod +x docker-entrypoint.sh

RUN chown -R bun:bun /app && mkdir -p /app/data && chown bun:bun /app/data

VOLUME ["/app/data"]
EXPOSE 7369

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:${PORT}/healthz || exit 1

STOPSIGNAL SIGTERM
ENTRYPOINT ["/sbin/tini", "--", "./docker-entrypoint.sh"]
CMD ["./app"]
```

### `docker-entrypoint.sh`

Handles mounted host volumes owned by root on startup, then drops privileges to `bun`:

```sh
#!/bin/sh
set -e

DB_PATH="${AUTH_DB_PATH:-/app/data/auth.db}"
DB_DIR="$(dirname "$DB_PATH")"
mkdir -p "$DB_DIR"
if [ "$(id -u)" = "0" ]; then
  chown -R bun:bun "$DB_DIR"
  exec su-exec bun "$@"
else
  exec "$@"
fi
```

### `.dockerignore`

Always exclude: `node_modules`, `dist`, `build`, `.git`, `.env`, `data/`, `*.db*`, `coverage`, `e2e`, `perf-reports`.

## Hard Rules

- **Pin `oven/bun` tag** to match `packageManager` in `package.json`.
- **Keep `package.json` in runner** if `/api/version` reads it directly (or ensure `version.ts` embeds it cleanly), and pair with `BUILD_TIME` for redeploy detection.
- **Keep `/healthz` fast and auth-free** — reserve DB/upstream checks for `/api/healthz` so Docker restarts don't thrash on transient upstream slowness.
- **Never copy `.env` or dev secrets** into the image. Pass secrets at runtime (`TRUSTED_ORIGINS`, `BETTER_AUTH_SECRET`, etc.).
- **Verify Windows line endings**: `docker-entrypoint.sh` MUST use LF (`\n`), enforced via `.gitattributes` (`*.sh text eol=lf`).
