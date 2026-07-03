---
description: Produce a self-contained prompt the user can paste into a fresh Claude Code window to continue the current investigation or work without losing context
allowed-tools: Read, Glob, Grep, Bash(git *), Write
effort: medium
---

# Session Handoff

Write a prompt the user copies into a new Claude Code window so the next agent picks up this work cold.

## Invocation

- `/ahead:handoff` — print the prompt to chat in a fenced block
- `/ahead:handoff --save` — also write to a file (see "Saving" below)

## Output format

One-line intro, then a single ```text fenced block containing the full prompt. Nothing after the block.

The prompt must include these sections, in this order. Omit a section only if it would be empty:

1. **Objective** — one paragraph: what the user is trying to accomplish and why
2. **Current state** — branch, uncommitted files, files open mid-edit, last commit hash
3. **Progress so far** — concrete work completed, with `file:line` references
4. **Ruled out** — approaches tried and why they failed
5. **Open decisions** — questions pending user input, with options discussed
6. **Immediate next step** — the single next action the next agent should take
7. **Required reading** — absolute paths the next agent must read first: observations, specs, related code

## Rules

- Write from conversation memory — the value is synthesis, not re-derivation
- Concrete over abstract: "tried X in auth.ts:42, failed because Y" — not "tried a few approaches"
- Use absolute paths — the next session may start in a different cwd
- Capture session-only context explicitly: verbal scope decisions, user preferences surfaced, rejected alternatives
- Target one screen. Longer means noise.

## Anchoring current state

Run once before drafting:
- `git rev-parse --abbrev-ref HEAD`
- `git status --short`
- `git log -1 --format="%h %s"`

Summarize the output. Do not paste it raw into the prompt.

## Saving

When invoked with `--save`, also write the prompt to the first path that applies:
1. `docs/tmp/handoff.md` if `docs/tmp/` exists AND `git check-ignore docs/tmp` confirms it is gitignored
2. `handoff.md` at repo root

Overwrite without prompting.
