# Claude Skills

Store Claude Code-compatible skills here.

Create one subdirectory per skill under `claude/skills/`. Each Claude skill must include a `SKILL.md` file and may include `references/`, `scripts/`, and `assets/`.

## Structure

```
claude/skills/<skill-name>/
├── SKILL.md          # Required. YAML frontmatter + markdown instructions.
├── references/       # Optional. Supporting docs referenced from SKILL.md.
├── scripts/          # Optional. Helper scripts Claude can execute.
└── assets/           # Optional. Static files the skill depends on.
```

### SKILL.md frontmatter

```yaml
---
name: skill-name           # Becomes the /slash-command name
description: …             # Used by Claude to decide when to auto-invoke
argument-hint: […]         # Shown in autocomplete (optional)
allowed-tools: Bash(gh *), Read  # Scope no-prompt shell access narrowly (optional)
---
```

## Current Skills

- `gh-cli`: Use `gh` for GitHub repository, PR, issue, Actions, and `gh api` workflows
