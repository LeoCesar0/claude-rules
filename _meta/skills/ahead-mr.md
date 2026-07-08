---
description: Create mr.md — a Merge Request draft (title + description) from the current branch's net changes, filtering out self-introduced-and-fixed noise
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *, gh *), AskUserQuestion
effort: medium
---

# Merge Request Draft

Create a temporary `mr.md` file (title + description) from the current branch's net changes against its base branch.

## Default behavior

On invocation, create `mr.md` immediately. Only ask questions if the base branch cannot be determined or if the branch has zero net changes.

## Language

Write the title and description in **Brazilian Portuguese (pt-BR)** by default. Do not ask.

## File location

1. `docs/tmp/mr.md` — if `docs/tmp/` exists AND `git check-ignore docs/tmp` confirms it is gitignored
2. Otherwise → `mr.md` at the project root

Overwrite existing `mr.md` without prompting. It is a working file.

## File format

```markdown
fix: Título curto no imperativo

<um ou dois parágrafos de descrição>

## Mudanças
- Um bullet por mudança lógica
- Agrupadas por área quando fizer sentido
```

- First line = title with conventional-commits prefix (`fix:`, `feat:`, `refactor:`, `chore:`, `docs:`, `test:`, `perf:`)
- Infer the prefix from the dominant type of net change

## Determining the base branch

Resolve `$BASE` in order, stop at first hit:

1. `gh pr view --json baseRefName 2>/dev/null` → use `baseRefName`
2. `AskUserQuestion` with options: `main`, `master`, `develop`, custom text. Include the current branch name in the question.

## Analyzing the branch

Read all of the following before writing `mr.md`:

1. `git log $BASE..HEAD --no-merges --format="%h %s%n%b%n---"` — commit narrative (intent only, not source of truth)
2. `git diff $BASE...HEAD --stat` — files and line counts
3. `git diff $BASE...HEAD` — net diff (the authoritative "what changed")
4. Observation files where `working-branch` matches the current branch — glob `docs/observations/**/*.md`, grep frontmatter. Input only, see Observation handling below

Describe the net diff, not the commit log. Use commits only to understand *why*.

## Self-fix filtering

**CRITICAL**: If a bug was introduced AND fixed within this branch, do not mention either in the MR — unless the fix also modifies code that existed before the branch diverged from `$BASE`.

## Observation handling

Observations are a personal dev artifact, not shared with the team — see `~/.claude/rules/objective-writing.md`. Never link, cite, or name an observation file in `mr.md`.

For each observation with matching `working-branch` whose issue the diff addresses, fold its resolution into the relevant `## Mudanças` bullet, described in plain language — not as a separate section, not by filename.

Do not modify observation files from this skill.

## Empty-branch case

If `git diff $BASE...HEAD` is empty, report that to the user and do not create `mr.md`.

## Rules

- Describe changes in terms of behavior and impact, not implementation steps
- No preamble ("Este MR faz..."), no emoji, no marketing language
- Skip incidental refactors unless they are the dominant change
- One bullet per logical change — not one per file and not one per commit
- No personal/session jargon or internal artifact references — see `~/.claude/rules/objective-writing.md`
