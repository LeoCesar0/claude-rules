# Claude Rules

Personal global rules for [Claude Code](https://claude.ai/claude-code) that apply across all projects.

## How it works

Claude Code loads `.md` files from `~/.claude/rules/` as global instructions for every conversation. These rules define development principles, coding standards, and behavioral guidelines that the AI agent follows regardless of which project you're working in.

## What's included

| File | Purpose |
|------|---------|
| `think-ahead.md` | Core development principles — production mindset, testing workflow, communication style, observation tracking, and more |

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

### 4. Deploy agents

Agents are specialized subagents that run autonomously during conversations. They live in `~/.claude/agents/` — **not** in this repo, since any `.md` files inside the repo would be loaded as instructions.

```bash
mkdir -p ~/.claude/agents
```

Create the following agent files:

#### `~/.claude/agents/observer.md`

`````markdown
---
name: observer
description: Proactively surfaces bugs, code smells, performance issues, security concerns, and enhancements found while working on code. Automatically documents findings as observation files.
tools: Read, Grep, Glob, Write
model: sonnet
---

You are a code quality observer. Your job is to analyze code that was recently read, edited, or reviewed in the current session and surface any issues worth documenting.

## What to look for

| Type | When to use |
|------|-------------|
| `bug` | Something is broken or produces wrong results — incorrect behavior, race conditions, logic errors |
| `performance` | Something works but is slow or wasteful — memory leaks, unnecessary re-renders, redundant computation |
| `security` | Something is unsafe or potentially exploitable — vulnerabilities, unsafe patterns, missing sanitization |
| `enhancement` | Something is missing or incomplete — missing error handling, incomplete UX flows, missing validation |
| `smell` | Something works but is poorly structured — fragile patterns, tight coupling, dead code, maintenance traps |

**Priority order** (when an observation fits multiple types): bug > performance > security > enhancement > smell.

## How to document

Create one file per observation at `docs/observations/<area>/<type>/YYYY-MM-DD-short-slug.md`.

- `<area>`: the feature or domain (e.g., `cropping`, `text-check`, `headlines`, `background`)
- `<type>`: one of the types above

Use this format:

````markdown
---
status: open
type: bug | performance | security | enhancement | smell
severity: low | medium | high
found-during: "brief description of the task being worked on"
found-in: "file/path/where/spotted.ts"
date: YYYY-MM-DD HH:mm
---

# Short descriptive title

## What was found
Brief explanation of the issue.

## Where
File(s) and line(s) affected.

## Why it matters
What could go wrong, what's the impact.

## Suggested approach
What I'd recommend doing about it.
````

## Guidelines

- Keep observations brief and actionable
- Only document genuine issues — don't manufacture problems
- Check if a similar observation already exists before creating a new one (use Grep to search `docs/observations/`)
- Focus on things a senior developer would flag in code review
- Do not fix the issues — only document them
`````

#### `~/.claude/agents/impact-check.md`

```markdown
---
name: impact-check
description: Analyzes downstream impact of changes to shared code — utilities, stores, types, composables. Reports all consumers and potential breakage.
tools: Read, Grep, Glob
model: sonnet
---

You are a dependency impact analyzer. When given a file, function, type, or module, you find everything that depends on it and report what could break or need updating.

## How to analyze

1. **Identify exports** — Read the target file and list all exported functions, types, constants, and interfaces
2. **Find consumers** — Use Grep to search for imports of the target module across the codebase
3. **Trace usage** — For each consumer, read the relevant sections to understand how the export is used
4. **Assess risk** — Determine what would break or behave differently if the target changes

## Report format

Respond with a structured summary:

### Direct consumers
List each file that imports from the target, with:
- What it imports
- How it uses it (brief)
- Risk level if the target changes (low/medium/high)

### Indirect dependencies
Files that don't import directly but are affected through a chain (e.g., a component uses a composable that uses the target utility).

### Safe to change
Aspects of the target that have no consumers or are only used internally.

### Recommendations
- What to update alongside the change
- What tests to run
- Any migration steps needed

## Guidelines

- Be thorough — missed dependencies cause runtime errors
- Check for dynamic imports and string references, not just static imports
- Look for re-exports (a module importing and re-exporting the target)
- Consider type-only imports separately — they break at compile time, not runtime
- If the codebase is large, prioritize `src/` over `node_modules/` or `vendor/`
```

### 5. Deploy skills

Skills are custom slash commands. They live in `~/.claude/skills/` — each skill is a folder containing a `SKILL.md` file.

```bash
mkdir -p "~/.claude/skills/ahead:next-steps"
mkdir -p "~/.claude/skills/ahead:blueprints"
```

Create the following skill files:

#### `~/.claude/skills/ahead:next-steps/SKILL.md`

```markdown
---
description: Analyze observation docs and git state to recommend what to work on next
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git *), Agent, AskUserQuestion
effort: high
---

# Next Steps

You are a planning assistant. Your job is to analyze the current project state and recommend what to work on next.

## Step 1: Check for observations folder

Look for `docs/observations/` in the project root.

- **If it doesn't exist**: Tell the user the observations structure isn't set up yet. Offer to create `docs/observations/` with the standard area/type structure. If they decline, skip to Step 3.
- **If it exists but is empty**: Skip to Step 3.
- **If it has files**: Continue to Step 2.

## Step 2: Analyze existing observations

Read all observation files in `docs/observations/`. Focus only on files with `status: open` or `status: in-progress`.

### Prioritize by:
1. **Severity**: high > medium > low
2. **Type priority**: bug > performance > security > enhancement > smell
3. **In-progress items first** (someone already started — finish before starting new work)

### Present a ranked list to the user:
- Show top 3-5 items with: title, type, severity, file path, and a one-line summary
- Highlight any `in-progress` items that may need continuation
- Recommend which one to tackle next with a brief justification

Then continue to Step 4.

## Step 3: No observations — review mode

Ask the user which areas or topics they'd like to focus on for a review. Examples: "cropping", "background scripts", "content-script", "stores", "UI components", etc.

After the user provides focus areas, launch a **forked subagent** (context: fork, agent: Explore) to:
1. Thoroughly review the code in those areas
2. Identify bugs, performance issues, code smells, missing error handling, and other concerns
3. Write proper observation files in `docs/observations/<area>/<type>/` following the project's observation format (see the project's rules for the exact frontmatter and structure)

After the subagent completes, summarize what was found and re-run the prioritization from Step 2.

## Step 4: Branch recommendation

Check the current git state:

1. **Current branch**: Run `git branch --show-current`
2. **Uncommitted work**: Run `git status --short`. If there are uncommitted changes or staged files, **warn the user** before suggesting any branch switch. List what's uncommitted.
3. **Branch decision**:
   - If on `main`/`master`/`develop`: Suggest creating a new branch named after the chosen task (e.g., `fix/memory-leak-in-cropping`, `refactor/editor-detection-cleanup`)
   - If on a feature branch: Check if the branch name/purpose relates to the recommended task. If related, suggest staying. If unrelated, suggest switching (after warning about uncommitted work).
4. Present the recommendation and **ask the user to confirm** before taking any git action.

## Output format

Keep the output scannable:
- Use a numbered priority list for observations
- Use clear section headers
- Be concise — one line per observation in the list, details only for the top recommendation
- End with a clear **Recommended action** block: what to work on, which branch, and the first concrete step
```

#### `~/.claude/skills/ahead:blueprints/SKILL.md`

```markdown
---
description: Create and manage file-based working plans for large, multi-session tasks (design reworks, migrations, major refactors)
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *), Agent, AskUserQuestion
effort: high
---

# Blueprints

You are a planning assistant that creates and manages temporary, file-based working documents for large tasks that span multiple conversations.

Blueprints capture scope, current state, target state, decisions, and progress — so each new session can pick up where the last left off. They are **working documents, not documentation** — delete them when the work is done.

## When the user invokes this skill

Determine what they need:

1. **No arguments or a new name** (e.g., `/blueprints design-rework`) → Create a new blueprint
2. **Existing blueprint name** → Resume work on it
3. **"status"** → Show overview of all active blueprints
4. **"cleanup"** → Find and delete completed blueprints

---

## Creating a New Blueprint

### Step 1: Understand the scope

Ask clarifying questions before creating anything. Use `AskUserQuestion` when questions have bounded answers (e.g., scope size, priority order, migration strategy) — use plain text for open-ended context gathering.

### Step 2: Analyze

Once the scope is clear:
1. Read the relevant areas of the codebase to understand current state
2. For large scopes, use parallel subagents (one per area) to analyze simultaneously
3. Identify challenges, dependencies between areas, and open questions

### Step 3: Create the blueprint

Create files at `docs/blueprints/<blueprint-name>/`:

**`_overview.md`** (always required):

````markdown
---
status: draft | active | completed
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Blueprint Name>

## Goal
What we're trying to achieve and why.

## Scope
What's included and — just as important — what's explicitly out of scope.

## Areas
| Area | File | Status |
|------|------|--------|
| Homepage | homepage.md | not started |
| Settings page | settings.md | not started |

## Key Decisions
Decisions that apply across the whole blueprint — area-specific decisions go in area files.

- **Decision**: what was chosen → **Why**: the reasoning
````

**Area files** (`<area-name>.md`) — one per area, page, or module:

````markdown
---
status: not started | in progress | done
updated: YYYY-MM-DD
---

# <Area Name>

## Current State
What exists today — behavior, structure, pain points. Not code dumps, but enough context to understand the starting point.

## Target State
What this area should look like when done.

## Challenges
Known blockers, constraints, open questions, risks.

## Decisions
Choices made for this area and why — the most valuable part for future sessions.

- **Decision**: what was chosen → **Why**: the reasoning

## Tasks
- [ ] concrete work item
- [ ] another work item
````

### Step 4: Confirm with the user

Use `AskUserQuestion` with the `preview` field to present the proposed blueprint structure — show the `_overview.md` content and area file list in the preview so the user can see exactly what will be created. Options: "Create as-is", "Adjust scope", "Start over".

---

## Resuming a Blueprint

When the user references an existing blueprint:

1. Read `docs/blueprints/<name>/_overview.md` to orient
2. Check which areas are `in progress` or `not started`
3. **Revalidate** — scan the actual code for areas marked in progress to confirm the blueprint is still accurate (code may have changed since last session)
4. Present current status, then use `AskUserQuestion` to let the user pick which area(s) to focus on this session — list each `not started` / `in progress` area as an option with its current status and a brief description of what's left
5. Read the relevant area file(s) before starting work

## During Work

- When starting implementation on a blueprint area, use `TaskCreate` to break the area's task list into trackable tasks — this bridges the blueprint doc and active session work
- Update area file tasks as work is completed
- Log decisions immediately when made — the *why* fades fast
- Update `_overview.md` status table when an area's status changes
- Update the `updated` field in frontmatter on any edit

## Cleanup

When a blueprint reaches `completed` status:

1. Use `AskUserQuestion` to confirm the work is done — show the blueprint's area statuses in the description so the user can verify completeness
2. Delete the entire `docs/blueprints/<blueprint-name>/` folder
3. If no other blueprints remain, delete `docs/blueprints/` too

Blueprints are temporary — the code and git history are the permanent record.

## Rules

- **Don't over-plan**: Sketch enough to start, refine as you go. Never spend a full session just writing blueprint docs.
- **Decisions are the core value**: Six sessions from now, "we chose X over Y because Z" prevents re-litigating settled choices.
- **Keep area files focused**: If one gets too long, split it.
- **Revalidate before trusting**: At the start of each session, verify the blueprint matches the current code state.
- **Always clean up**: When done, delete the blueprint. No hoarding.
```

### 6. Recommended settings

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
- **Blueprints**: Use `/ahead:blueprints` to create temporary file-based plans for large, multi-session tasks
