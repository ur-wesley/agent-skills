---
name: app-existing
description: Auto-discover idea, stack, and brand from an existing codebase, confirm the picture, then write docs/IDEA.md, docs/STACK.md, and docs/BRAND.md. Use when the repo already has an app, "document this app", "existing project brief", "capture this repo", or the user wants those docs for code that already exists. Do not use for a new or empty app — that is app-brief.
---

# App Existing

Orchestrator for a repo that already has code. Discover first. Writers: `app-scope`, `app-stack`, `app-brand`.

If the target is empty / no app yet → stop, use `app-brief`.

Write files in the **target app repo** (`docs/IDEA.md`, `docs/STACK.md`, `docs/BRAND.md`), not in this skills repo. Language of the files = language the user used.

## Hard gate

Do **not** write any of the three files until the user confirms the picture. No "I'll draft them while we talk." No scaffolding the app.

## Discover

Read. Do not invent:

- `README.md`, `DESIGN.md`, existing `docs/IDEA.md` / `docs/STACK.md` / `docs/BRAND.md`
- `package.json` / lockfile, `mise.toml`, `.oxlintrc.json`, `knip.json`, `Cargo.toml`, `tauri.conf.*`
- `server/`, `e2e/`, `Dockerfile` (to distinguish `web` SPA vs `web-fullstack`)
- routes, `features/*`, `components/ui/*`, `components/*`, app entry
- CSS variables / `@theme inline` tokens for palette
- package name, window title, current product name

Infer: problem, audience if stated, happy-path, features that exist, platform (`desktop` / `web` / `web-fullstack` / `cli`), constraints, name.

## Gaps

Grill: **one question at a time**. Offer a recommended answer from discovery. Wait. Skip if already in code or chat.

- Later vs now — **shipped code = MVP** unless they say otherwise
- Non-goals
- Constraints (local-first, offline, must-integrate-with)
- Naming **theme field** — only if unnamed / no brand
- Visual personality — only if no brand: **calm / dense / playful / serious**
- If actual stack ≠ prescribed for the detected platform: **record actual**, or write prescribed as the target

Do not re-ask platform or existing features. Do not invent answers. If they shrug, give 2–3 concrete options and a pick.

## Picture

Dump a short picture: idea, MVP vs later, non-goals, constraints, platform, stack (prescribed or actual if they diverged), name if known, theme field, vibe. Ask: confirm or correct?

If they correct, update the picture and ask again.

## After yes

Same session, in order:

1. Follow **app-scope** — write `docs/IDEA.md`
2. Follow **app-stack** — write `docs/STACK.md`
3. Follow **app-brand** — keep existing name unless they asked to rename; extract palette from theme if present; propose 3–5 names only on rename / unnamed; write `docs/BRAND.md`

**Stack:** follow `app-stack` (prescribed tables, no invented stack). Platform and actual-vs-prescribed from the confirmed picture.

Do not scaffold the app. Stack/UI skills do that later.

## Solo writers

`app-scope`, `app-stack`, `app-brand` may run alone. If `docs/IDEA.md` exists, they read it and skip answered questions. This skill is the existing-repo brief.
