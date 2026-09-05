---
name: app-brand
description: Write docs/BRAND.md — name from a theme field, corporate design, concise logo prompt. Use when naming an app, defining brand/CD, or after app-brief or app-existing confirms the picture.
---

# App Brand

Writes `docs/BRAND.md` in the **target app repo**. Create `docs/` if missing. Language = interview language.

If `docs/BRAND.md` already exists and the user did not ask to replace it, update in place.

Do **not** generate a logo image unless the user asks later. The logo field is a prompt only.

## Inputs

Read `docs/IDEA.md` if present. Skip questions already answered there or in chat.

If run **after app-brief**: do not re-ask theme or vibe. Still propose names and wait for pick.

If run **after app-existing**: do not re-ask theme or vibe if discovered. Keep the existing name unless they asked to rename. Propose 3–5 names only on rename / unnamed. Extract palette from theme if present.

Need:

- App concept (from IDEA or a one-liner)
- Naming **theme field** if unknown — ask once (stars, minerals, ships, birds, …). Wait. Skip if already in the confirmed picture.
- Visual personality if unknown — **calm / dense / playful / serious**. Wait. Skip if already in the confirmed picture.

## Names

If keeping an existing name (after **app-existing**, no rename): skip this section. Write `docs/BRAND.md` with that name.

Otherwise propose **3–5 names** from that field. Each name must **map to the concept**: what that thing does ≈ what the app does. Reject cool-but-unrelated picks.

Show: name — one-line metaphor. Recommend one. Wait for pick or veto.

Veto → new list, same rules. Do not write `docs/BRAND.md` until a name is picked.

## File

```markdown
# {Name}

## Name

{Name} — {one-line why it fits}

Theme: {field}

## Voice

{tone in a few adjectives + one sentence of how copy should sound}

## Palette

| Role | Hex |
| --- | --- |
| bg | # |
| fg | # |
| accent | # |
| {optional muted/danger} | # |

## Type

{one sentence: weight, vibe, not a font catalog}

## Logo prompt

{one tight paragraph, image-gen ready}
```

### Logo prompt rules

- Mark only. No wordmark, no app name in the mark, unless the user asked for letters.
- Square-safe, flat vector / geometric.
- Colors from the palette (name them + hex).
- One paragraph. No bullet lists inside the prompt. No camera/film jargon.
- Concise: subject, composition, style, colors, constraints (no text, no photorealism).
