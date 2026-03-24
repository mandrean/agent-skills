# agent-skills

Shared repository for AI agent skills, usable by Claude Code, Codex, and other compatible tools.

## Layout

```
skills/<skill-name>/
  SKILL.md              # Required. YAML frontmatter + markdown instructions.
  references/           # Optional. Supporting docs referenced from SKILL.md.
  agents/               # Optional. Agent-specific configs (e.g. openai.yaml).
  scripts/              # Optional. Helper scripts the agent can execute.
  assets/               # Optional. Static files the skill depends on.
```

## Conventions

- Keep one skill per subdirectory under `skills/`.
- Keep reusable references, scripts, and assets inside each skill directory.
- Document tool-specific setup close to the skill that needs it.

## Included Skills

- `skills/gh-cli`: GitHub CLI skill for `gh`-based repository, PR, issue, workflow, and API tasks
- `skills/harvest`: Fill out Harvest timesheets — copies last week's entries or interactively picks projects and tasks
