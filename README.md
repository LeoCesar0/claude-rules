# Claude Rules

Personal global rules for [Claude Code](https://claude.ai/claude-code) that apply across all projects.

## How it works

Claude Code loads files from `~/.claude/rules/` as global instructions for every conversation. These rules define development principles, coding standards, and behavioral guidelines that the AI agent follows regardless of which project you're working in.

## Setup

1. Clone this repo into your Claude config directory:

```bash
git clone https://github.com/LeoCesar0/claude-rules.git ~/.claude/rules
```

2. Reference rules from `~/.claude/CLAUDE.md` (your global instructions file):

```markdown
# Global Instructions

## Core Principles
See @~/.claude/rules/think-ahead.md for proactive development approach
```

Claude Code will automatically load the referenced files into every conversation.

## Structure

| File | Purpose |
|------|---------|
| `think-ahead.md` | Core development principles — production mindset, testing workflow, communication style, observation tracking, and more |

## Adding new rules

1. Create a new `.md` file in this directory
2. Reference it from `~/.claude/CLAUDE.md` using `See @~/.claude/rules/<filename>`
3. Commit and push

## Setting up global skills

Claude Code discovers custom slash commands (skills) from `~/.claude/skills/`. Each skill is a folder containing a `SKILL.md` file.

### Quick setup

```bash
mkdir -p ~/.claude/skills
```

### Creating a skill

```bash
mkdir ~/.claude/skills/my-command
```

Then create `~/.claude/skills/my-command/SKILL.md`:

```markdown
---
description: What this command does
disable-model-invocation: true
---

Instructions for Claude when /my-command is invoked.
Use $ARGUMENTS to capture user input.
```

### Key frontmatter options

| Field | Purpose |
|-------|---------|
| `description` | What the skill does (used for auto-invocation matching) |
| `disable-model-invocation: true` | Only manual `/command` invocation, not auto-triggered |
| `allowed-tools` | Tools Claude can use without asking (e.g., `Read, Grep, Bash(git *)`) |
| `effort` | Effort level: `low`, `medium`, `high` |
| `context: fork` | Run in an isolated subagent |
| `agent` | Subagent type to use (e.g., `Explore`, `Plan`) |

### Skill locations and scope

| Location | Scope | Invocation |
|----------|-------|------------|
| `~/.claude/skills/` | All projects (global) | `/skill-name` |
| `<project>/.claude/skills/` | Single project | `/skill-name` |
| Plugin `skills/` | Where plugin is enabled | `/plugin-name:skill-name` |

For more details, see the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code).

## Key highlights

- **Test-first bug fixes**: Reproduce the bug with a failing test before fixing, then verify
- **Observation tracking**: Document bugs, smells, and issues found during any code work
- **Constructive pushback**: The agent challenges assumptions when it spots problems
- **Production safety**: No commands that could affect production environments
- **Comment the "why"**: Only comment non-obvious decisions, not what the code does
