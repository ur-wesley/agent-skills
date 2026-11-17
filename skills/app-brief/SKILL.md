---
name: app-brief
description: Interview the user until the app picture is complete, then write idea/scope, stack, and brand files. Use when starting a new app, "app brief", "scope this idea", "brand this app", or the user wants docs/IDEA.md, docs/STACK.md, and docs/BRAND.md together. Do not use on an existing codebase — that is app-existing.
---

# App Brief

Orchestrator. Interview first. Writers: `app-scope`, `app-stack`, `app-brand`.

If the target already has an app → stop, use `app-existing`.

Write files in the **target app repo** (`docs/IDEA.md`, `docs/STACK.md`, `docs/BRAND.md`), not in this skills repo. Language of the files = language the user used in the interview.

## Hard gate

Do **not** write any of the three files until the user confirms the picture. No "I'll draft them while we talk." No scaffolding the app.

## Interview

Grilling: **one question at a time**. Offer a recommended answer. Wait. Skip a question if already answered in chat.

Order:

1. Problem in one sentence
2. Who uses it, in what situation
3. Happy-path workflow
4. MVP features vs later
5. Non-goals
6. Constraints (local-first, offline, must-integrate-with)
7. Platform: **desktop / web / web-fullstack / cli**
8. Naming **theme field** (stars, minerals, ships, birds, …)
9. Visual personality: **calm / dense / playful / serious**

Do not invent answers. If they shrug, give 2–3 concrete options and a pick.

## Picture

After Q9, dump a short picture: idea, MVP vs later, non-goals, constraints, platform, theme field, vibe. Ask: confirm or correct?

If they correct, update the picture and ask again.

## After yes

Same session, in order:

1. Follow **app-scope** — write `docs/IDEA.md`
2. Follow **app-stack** — write `docs/STACK.md`
3. Follow **app-brand** — propose names, wait for pick, write `docs/BRAND.md`

Do not scaffold the app. Stack/UI skills do that later.

## Solo writers

`app-scope`, `app-stack`, `app-brand` may run alone (rerun brand, fix stack). If `docs/IDEA.md` exists, they read it and skip answered questions. This skill is the full brief.
