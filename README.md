# Agent Skills

A collection of reusable AI agent skills for use with GitHub Copilot and other AI coding assistants.

## Overview

This repository contains curated skill definitions that guide AI agents to follow best practices and established patterns when working on specific types of projects or tasks.

## Available Skills

| Skill | Description |
|-------|-------------|
| [react](./react/SKILL.md) | React development best practices including scalable architecture, state management, API layer patterns, and performance optimization |

## Instruction Files

Instruction files (`*.instructions.md`) plug into the GitHub Copilot flow to
automatically inject guidance into the agent's context based on the files being
edited. Unlike skills — which the agent loads on demand — instruction files are
applied automatically whenever a matching file is touched, making them ideal for
enforcing "always do this" rules such as *load the relevant skill before
writing code*.

| Instruction file | Description |
|------------------|-------------|
| [react-frontend.instructions.md](./instructions/react-frontend.instructions.md) | Forces the agent to load the React and design-system skills (and their reference guides) before implementing or modifying frontend code |

### Anatomy of an Instruction File

An instruction file has two parts: a **YAML frontmatter** header and a
**Markdown body** with the actual instructions.

```markdown
---
description: Reference React skill and design system skill before implementing frontend code.
applyTo: 'app/src/**/*.{ts,tsx}'
---

Before implementing or modifying React code:
...
```

#### Frontmatter entries

| Entry | Purpose |
|-------|---------|
| `description` | A short summary of what the instruction file enforces. Shown in tooling and used to give the agent quick context. |
| `applyTo` | A glob pattern that scopes when the instructions apply. The body is automatically added to the agent's context whenever it edits a file matching this pattern. Use `'**'` to apply to every file. |

#### Body sections (from the example)

The `react-frontend.instructions.md` body is organized as a numbered checklist
the agent must work through before writing frontend code:

1. **Load applicable skills** — names the `react` and `design-system` skills the
   agent must read first.
2. **Load skill reference guides** — maps specific tasks (creating a hook,
   adding an API call, optimizing performance, etc.) to the exact reference
   guide to read for that task.
3. **Follow conventions** — points to the project's `AGENTS.md` for
   project-specific rules.
4. **Audit the touched unit** — requires reviewing the whole component/hook/
   function that was changed against the skill, flagging drift, and asking the
   user before refactoring (rather than refactoring unprompted).
5. **Key requirements** — a concise list of non-negotiable rules (use MUI
   components, import colors from theme tokens, no inline hex colors, extract
   page logic into custom hooks).

### Using Instruction Files with GitHub Copilot

1. Place the file in your project's `.github/instructions/` directory (or
   reference this repository's `instructions/` folder).
2. Set the `applyTo` glob to match the files you want the rules to cover.
3. When the agent edits a matching file, the instructions are injected
   automatically — no manual step required.

## How to Use

### With GitHub Copilot

1. Clone this repository or add it as a submodule to your project
2. Reference the skill in your `.github/copilot-instructions.md` or agent configuration
3. The AI agent will follow the patterns defined in the skill files

### Skill Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md           # Main skill definition with YAML frontmatter
└── references/        # Detailed guides and documentation
    └── *.md
```

The `SKILL.md` file contains:
- **YAML frontmatter**: Skill name and trigger description
- **Core principles**: Key guidelines for the skill
- **Quick reference**: Links to detailed documentation
- **Usage guidance**: When to apply specific patterns

## Contributing

To add a new skill:

1. Create a new directory with your skill name
2. Add a `SKILL.md` file with YAML frontmatter
3. Include reference documentation in a `references/` subdirectory
4. Update this README to include the new skill

## License

See [LICENSE](./LICENSE) for details.
