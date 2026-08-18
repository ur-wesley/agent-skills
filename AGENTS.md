# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository overview

A collection of [Agent Skills](https://agentskills.io/) for TypeScript/SolidJS/Tauri development and Fusion multi-agent workflows. Skills are packaged instructions that extend agent capabilities.

## Creating or editing a skill

### Directory structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # required: skill definition
    scripts/              # optional: executable scripts
    references/           # optional: supporting docs loaded on demand
```

### Naming conventions

- **Skill directory**: `kebab-case` (must match `name` in frontmatter)
- **SKILL.md**: always uppercase, always this exact filename
- **Scripts**: `kebab-case.sh`, `kebab-case.mjs`, or `kebab-case.js`

### SKILL.md format

```markdown
---
name: {skill-name}
description: What the skill does and when to use it. Include trigger phrases.
---

# {Skill Title}

{Brief description and instructions}
```

### Best practices

- Keep `SKILL.md` under 500 lines; move heavy reference material to separate files
- Write specific descriptions so agents know when to activate the skill
- Use progressive disclosure — link supporting files read only when needed
- Prefer scripts over inline code for deterministic operations
- Keep file references one level deep from `SKILL.md`

## Installation

```bash
npx skills add ur-wesley/agent-skills
npx skills add ur-wesley/agent-skills --skill ts-styleguide
```
