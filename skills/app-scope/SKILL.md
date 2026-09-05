---
name: app-scope
description: Write docs/IDEA.md — app idea, audience, MVP scope, features, non-goals. Use when scoping an app, listing features, or the user asks for an idea/scope file. Also used after app-brief or app-existing confirms the picture.
---

# App Scope

Writes `docs/IDEA.md` in the **target app repo**. Create `docs/` if missing. Language = interview language.

If `docs/IDEA.md` already exists and the user did not ask to replace it, update in place — do not fork a second file.

## Inputs

Read `docs/IDEA.md` if present. Skip questions already answered there or in chat.

If run **solo** (no confirmed app-brief / app-existing picture): ask only these, **one at a time**, recommended answer, wait:

1. Problem in one sentence
2. Who uses it, in what situation
3. Happy-path workflow
4. MVP features vs later
5. Non-goals
6. Constraints (local-first, offline, must-integrate-with)

If run **after app-brief or app-existing**: do not re-interview. Write from the confirmed picture.

## File

```markdown
# {working title or name if known}

## Idea

{1–2 sentences}

## Problem

{what hurts}

## Audience

{who, in what situation}

## Scope

MVP: {what ships}

Later: {explicitly not now}

## Features

- **{Name}** — {what it does, not how}
- **{Name}** — {what it does, not how}

## Non-goals

- {out of scope}

## Constraints

- {local-first, offline, must-integrate-with, none}

## Success

{how we know MVP worked}
```

Rules:

- Features: named, one line each, user-visible behavior. No stack, no folder trees.
- MVP vs later must disagree — if everything is MVP, push back once.
- No fluff headings. No implementation plan.
- **Success**: write from confirmed MVP + happy-path. If too vague, ask once. Do not invent a metric.
