# Agent Skills

A collection of reusable AI agent skills for use with GitHub Copilot and other AI coding assistants.

## Overview

This repository contains curated skill definitions that guide AI agents to follow best practices and established patterns when working on specific types of projects or tasks.

## Available Skills

| Skill | Description |
|-------|-------------|
| [react](./react/SKILL.md) | React development best practices including scalable architecture, state management, API layer patterns, and performance optimization |

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
