---
name: cursor-fusion
description: >
  Cursor Fusion workflow (Devin Fusion-style). Orchestrates model split:
  Grok 4.5 plans/reviews, Composer edits, explore/cavecrew for search.
  Use when user says "use fusion", "delegate sidekick", "cursor fusion",
  or when starting multi-file feature work.
---

# Cursor Fusion

Pragmatic Fusion-style workflow inside Cursor. Main agent plans and judges; Composer executes larger edits.

## Model cheat sheet

| Role | Model | How |
|------|-------|-----|
| Main / Plan | Grok 4.5 | User selects in model picker |
| Sidekick (edits) | composer-2.5 | Task with `model: composer-2.5` |
| Search | explore or cavecrew-investigator | Task subagent_type explore, or custom agent |
| Review | cavecrew-reviewer or code-reviewer | Task after edits |

## Decision table

| Situation | Action |
|-----------|--------|
| Multi-step feature, unclear scope | Plan mode first; brainstorm + writing-plans skills |
| "Where is X" / "what calls Y" | cavecrew-investigator or explore Task |
| Obvious edit, <=2 files | Main edits directly |
| >2 files, refactor, mechanical sweep | sidekick via Task, model composer-2.5 |
| Tiny mechanical 1-2 file fix | cavecrew-builder via Task, model composer-2.5 |
| After meaningful edits | cavecrew-reviewer before claiming done |
| Ambiguous requirements | Main decides; do not guess via sidekick |

## Chaining pattern

1. cavecrew-investigator → site list
2. Main picks 1-2 sites OR hands full spec to sidekick
3. sidekick executes multi-file work
4. cavecrew-reviewer audits diff

## Token stack (always on)

- **Ponytail** — minimum LOC (decision ladder in ponytail rule)
- **Caveman** — terse prose
- **Cavecrew** — compressed subagent returns

## Launching sidekick

```
Task tool:
  subagent_type: generalPurpose (or invoke sidekick custom agent)
  model: composer-2.5
  prompt: <precise edit spec with file paths, expected changes, verification commands>
```

For cavecrew agents, invoke by name in Task description or ask main to use the custom agent.

## What main must not do

- Dump 50+ grep lines into context (delegate search)
- Edit 5 files inline when sidekick should run
- Claim done without review on non-trivial changes
- Use npm run / npm build (user rule)
