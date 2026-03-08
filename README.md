# 🌱 Gemini Skills Garden

A curated collection of reusable [Gemini CLI Skills](https://geminicli.com/docs/cli/skills/) — plug-and-play expertise modules that extend what Gemini can do in your terminal.

## What are Gemini Skills?

Skills are on-demand expertise packages for [Gemini CLI](https://github.com/google-gemini/gemini-cli). Unlike general context files (`GEMINI.md`), skills represent **specialized capabilities** that Gemini activates only when needed — keeping your context window clean while unlocking powerful workflows like PR reviews, security audits, cloud deployments, and more.

Each skill is a self-contained folder with a `SKILL.md` instruction file and optional bundled resources (scripts, templates, examples).

## Skills

| Skill | Description |
|-------|-------------|
| [Generate PR Summary](/.agents/skills/generate-pr-summary/) | Generates a human-readable PR review summary in Markdown from a GitHub compare URL (tags, branches, or commits) using only the `gh` CLI. |

## Installation

### As Workspace Skills (recommended for teams)

Clone this repo into your project and skills will be auto-discovered by Gemini CLI:

```bash
# Clone into your project root
git clone https://github.com/chocoobar/gemini-skills-garden.git .agents/skills-garden

# Or add as a git submodule
git submodule add https://github.com/chocoobar/gemini-skills-garden.git .agents/skills-garden
```

Then copy the skills you want into your `.agents/skills/` directory:

```bash
cp -r .agents/skills-garden/.agents/skills/* .agents/skills/
```

### As User Skills (personal, available across all workspaces)

```bash
# Copy skills to your user-level skills directory
git clone https://github.com/chocoobar/gemini-skills-garden.git /tmp/skills-garden
cp -r /tmp/skills-garden/.agents/skills/* ~/.agents/skills/
```

## How Skills Work

1. **Discovery** — Gemini CLI scans `.agents/skills/` (or `.gemini/skills/`) at session start and loads each skill's name + description.
2. **Activation** — When your request matches a skill's description, Gemini activates it automatically.
3. **Execution** — The full `SKILL.md` instructions and bundled resources are injected into context, and Gemini follows the skill's workflow.

> Skills use progressive disclosure — only metadata is loaded initially. Full instructions are pulled in on activation, saving context tokens.

## Contributing

Have a useful Gemini CLI workflow? Turn it into a skill and submit a PR!

Each skill should live in its own folder under `.agents/skills/` and include:

```
.agents/skills/<skill-name>/
├── SKILL.md          # Required — YAML frontmatter (name, description) + instructions
├── scripts/          # Optional — helper scripts
├── examples/         # Optional — reference implementations
└── resources/        # Optional — templates, assets, etc.
```

### `SKILL.md` Format

```yaml
---
name: My Skill Name
description: A short description of what this skill does.
---

# My Skill Name

Detailed instructions for Gemini to follow when this skill is activated.
```

## License

MIT
