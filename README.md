# Claude Ahead Rules

Personal global rules for [Claude Code](https://claude.ai/claude-code) that apply across all projects.

## How it works

Claude Code loads `.md` files from `~/.claude/rules/` as global instructions for every conversation. These rules define development principles, coding standards, and behavioral guidelines that the AI agent follows regardless of which project you're working in.

## What's included

| File | Purpose |
|------|---------|
| `think-ahead.md` | Core development principles — production mindset, testing workflow, communication style, and more |
| `security.md` | Security & data protection — threat modeling, autonomous vs. authorized fixes, supply-chain gate, LGPD/TIPA compliance |
| `observations.md` | Observation framework — types, file format, frontmatter fields, and lifecycle |
| `specs.md` | Specs framework — lightweight spec-driven workflow (group + spec files, plan mode for implementation) |

This README also serves as the setup guide for deploying the full environment: rules, agents, and skills.

## Setup

> **For AI agents**: If a user asks you to set up this repo, follow every step below. Adjust paths to match the user's home directory (replace `/home/<user>/` as needed).

### 1. Clone the repo

```bash
git clone https://github.com/LeoCesar0/claude-rules.git ~/.claude/rules
```

### 2. Set up global instructions

Create or update `~/.claude/CLAUDE.md`:

```markdown
# Global Instructions

## Core Principles
See @~/.claude/rules/think-ahead.md for proactive development approach
See @~/.claude/rules/security.md for security & data protection (CRITICAL)
See @~/.claude/rules/observations.md for observation framework (types, format, lifecycle)
```

Claude Code will automatically load the referenced files into every conversation.

### 3. Exclude this README from instructions

Since Claude Code loads all `.md` files in `~/.claude/rules/` as instructions, this README must be excluded. Add `claudeMdExcludes` to `~/.claude/settings.json`:

```json
{
  "claudeMdExcludes": [
    "/home/<user>/.claude/rules/README.md"
  ]
}
```

**Important**: Replace `/home/<user>/` with the actual absolute path to your home directory (e.g., `/home/leonardo/`, `/Users/john/`). The pattern matches against absolute paths, so `~` won't work.

If `settings.json` already exists, merge the `claudeMdExcludes` array into it.

### 4. Deploy agents & skills

Agent and skill definitions are kept verbatim in [`agents-and-skills.md`](agents-and-skills.md) — the source of truth for deploying them on a new machine. Read that file only when creating, editing, or recreating an agent or skill; it is intentionally kept out of Claude's default context.

### 5. Recommended settings

Add `effortLevel` to `~/.claude/settings.json` (the same file where `claudeMdExcludes` was added in Step 3):

```json
{
  "effortLevel": "high"
}
```

This sets Claude Code to use high effort by default. Merge into your existing `settings.json` if it already has other fields.

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
- **Impact analysis**: Check downstream dependencies before modifying shared code
- **Next steps planning**: Use `/ahead:next-steps` to get prioritized recommendations based on observations
- **Specs workflow**: Lightweight spec-driven workflow — discuss and craft a spec group via `/ahead:specs`, then implement each spec with Claude Code's built-in plan mode + tests
- **Instruction writing**: Use `/ahead:instructions` to write or review Claude-facing instruction files
- **Merge Request drafts**: Use `/ahead:mr` to generate a `mr.md` draft from the current branch's net changes
- **Session handoff**: Use `/ahead:handoff` to generate a paste-ready prompt that carries the current work into a fresh Claude Code window
- **Decision-making**: Use `/ahead:decision` to weigh multi-option tradeoffs without inflated effort estimates or phantom urgency
- **Session recall**: Use `/ahead:recall` to summarize the last 2 chats of the current project and suggest where to pick up
- **Visualize & explain**: Use `/ahead:visualize` to render the current response (or a focused part of the conversation) as a styled, didactically enriched HTML page that opens in the browser — for when you want to *understand*, not just *read*
- **Resolve observations**: Use `/ahead:resolve` to run the observation closing ceremony for items handled in the session — validates each against the git diff, presents a plan, then trims/closes/commits one commit per observation
- **Autonomous task runs**: Use `/ahead:autopilot` to work a list of tasks end to end while away — a lean orchestrator dispatches one fresh subagent per task, decides without stopping, skips only on irreversible/external actions (prod, push, destructive ops, new deps), and reports everything done plus every decision at the end
