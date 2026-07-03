---
description: Write and review instruction files for Claude — CLAUDE.md, rules, agents, skills, and Claude-facing docs
allowed-tools: Read, Write, Edit, Glob, Grep, AskUserQuestion, Bash(git status, git log *, git diff *, git show *)
effort: high
---

# Write Instructions

Write and review files whose audience is Claude — rules, CLAUDE.md, agents, skills, and docs that guide Claude's behavior.

## Modes

1. `review` arg or reviewing existing files → Review mode
2. Everything else → Write mode

## Write Mode

### CRITICAL

- **No tutorials** — state the rule. Don't explain how tools, libraries, or patterns work. Claude has training knowledge. Only document what Claude can't infer from code or general knowledge.
- **No defaults** — don't instruct behavior Claude already exhibits. Before writing a rule, ask: "Would Claude do this anyway?" If yes, skip it. Only add rules that override defaults, enforce non-obvious constraints, or prevent specific past mistakes.
- **No redundancy** — check all existing instruction files before writing. Never duplicate a rule unless it's critical enough to reinforce across files. If it exists, reference it — don't restate it.

### Hierarchy

Rules have weight. Mark critical rules explicitly (`**CRITICAL**`, `**IMPORTANT**`). Don't mark everything — if everything is critical, nothing is.

Priority order for instruction content:
1. **Constraints** — what NOT to do (highest value — prevents damage)
2. **Overrides** — behavior that differs from Claude's defaults
3. **Conventions** — project-specific patterns Claude can't infer from code
4. **Workflows** — step sequences for complex tasks

### Writing rules

- Empirical language — direct, observable, verifiable. No "be careful", "consider", "try to". Say what to do.
- One rule per bullet. No compound instructions.
- Every rule must be testable — you can look at output and say "followed" or "violated"
- Examples only when the correct behavior isn't obvious from the rule itself
- Scope each file to one purpose. Rules ≠ agents ≠ skills ≠ CLAUDE.md.
- Frontmatter `description` field in agents/skills is the relevance signal — make it specific and concrete, not generic

### Placement

- Global behavior → `~/.claude/rules/` (referenced from `~/.claude/CLAUDE.md`)
- Project-specific → project's `CLAUDE.md` or `.claude/` directory
- Agent definition → `~/.claude/agents/<name>.md`
- Skill definition → `~/.claude/skills/<namespace>:<name>/SKILL.md`

### Workflow

1. Read all existing instruction files in scope — identify gaps, overlaps, contradictions
2. Draft the instruction applying the rules above
3. Present draft to user before writing to disk
4. After writing to `~/.claude/rules/`, flag that it needs commit+push to the rules repo

## Review Mode

Read target files (or full instruction set if none specified: `~/.claude/CLAUDE.md`, `~/.claude/rules/`, `~/.claude/agents/`, `~/.claude/skills/`, project CLAUDE.md).

### Check for

**High priority:**
- Redundancy — rules duplicated across files
- Contradictions — rules that conflict
- Tutorial creep — explanations of how things work instead of what to do
- Default instructions — rules that tell Claude to do what it already does

**Medium priority:**
- Vague rules — not testable, not specific ("be careful", "follow best practices")
- Scope creep — files doing too much, should be split
- Dead references — rules pointing to files, functions, or patterns that no longer exist

**Low priority:**
- Ordering — related rules not grouped
- Frontmatter quality — generic descriptions that won't match relevance queries

### Output

Group findings by file. Each finding: what's wrong, where, concrete fix.
Use `AskUserQuestion` with `multiSelect: true` so the user picks which to apply.
