# AI Agent Skills

Community-maintained skills for AI agents.

This repo is agent-first: skills are designed for any capable AI runtime, with [Automaton](https://github.com/Conway-Research/automaton) as the primary reference implementation.

## How skills work

A skill is one folder with one `SKILL.md`:

```text
<skill-name>/
  SKILL.md
```

`SKILL.md` uses YAML frontmatter + markdown instructions:

```md
---
name: my-skill
description: One-line summary
auto-activate: true
---

# Instructions
...
```

## Runtime notes

- **Automaton** loads skills from `~/.automaton/skills/<skill-name>/SKILL.md`.
- **Other agents** can use the same `SKILL.md` pattern, but loading paths and activation behavior may differ by runtime.

## Contributing

PRs are welcome.

When adding a skill:

- Create `<skill-name>/SKILL.md`.
- Keep instructions concrete, concise, and safe.
- Include assumptions (for example, what is already provisioned).
- Add the skill to [SKILLS.md](./SKILLS.md).
