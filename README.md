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

## Key highlights

- **Test-first bug fixes**: Reproduce the bug with a failing test before fixing, then verify
- **Observation tracking**: Document bugs, smells, and issues found during any code work
- **Constructive pushback**: The agent challenges assumptions when it spots problems
- **Production safety**: No commands that could affect production environments
- **Comment the "why"**: Only comment non-obvious decisions, not what the code does
