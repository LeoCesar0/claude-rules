---
description: Reflect on the current session (or a bounded work arc) for reusable self-improvements — mistakes made and fixed, bypassed conventions, something missed and noticed later, a project doc or global instruction that got in the way or had a real gap, a costly decision, "X works better than Y" patterns, or a surfaced user preference. Reports every finding as a summary card, then walks through apply/skip/adjust via the decision protocol before ever writing anything. Use when the user asks to review the session for self-improvement, "auto improvement", "auto-melhoria", or a retrospective.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *), AskUserQuestion
effort: high
---

# Self Improve

Reflect on the current session (or a bounded arc within it) to surface reusable lessons — habits, rules, harness, docs, or memories that would make the next similar task go better. Manual-only: invoked directly by the user, or by Claude suggesting it after a complex task and the user agreeing. Never fires on its own.

This is a **retrospective**, not a fix-it pass — it never edits code to resolve a bug, and it never applies a finding before the user approves it.

## Arguments

- **No args** → reflect on the whole current session.
- **A scope** (e.g., "just the auth refactor", "the last three tasks") → bound the reflection to that arc, skip the rest of the session.
- **Free text** → additional constraints or reminders folded into the reflection.

## Scope

Look for patterns like:
- a mistake made and later corrected
- a convention bypassed, then caught and fixed
- something that slipped through and was noticed only afterward
- a project doc that got in the way of the work
- a global instruction that got in the way, or had a real gap that would have helped
- a decision that cost more than it saved
- "solution X works better than Y" — a pattern worth generalizing
- a user preference surfaced, project-specific or general

Always abstract the idea before proposing it for the global tier — never write a global instruction tied to the specific app or project just worked on. Project-bound lessons stay in that project's scope (CLAUDE.md/CLAUDE.local.md, project docs); only the generalizable shape of a lesson goes global.

## Flow

### 1. Reflect

Gather signal from:
- conversation memory of the session — the primary narrative (what actually happened: mistakes, corrections, decisions)
- `git diff` / `git log` / `git status` — anchors real code changes; don't credit a fix that isn't in the diff
- observations (`docs/observations/`) touched or created this session — an existing signal of a noticed pattern
- existing memory files (`MEMORY.md` + entries) and relevant rules/skills/agents — check for duplication, and for cases where an existing instruction only partially covers what was observed

### 2. Filter

Report a finding only if **all** hold:
- it's recurring/generalizable — not an isolated slip with no pattern (e.g., a one-off typo)
- it's actionable — resolves into a concrete rule, comment, memory, or doc, not a vague feeling
- it isn't already covered, **or** is only partially covered and has room for real improvement (not cosmetic rewording)
- the expected gain outweighs the cost of one more permanent rule/memory

If nothing clears the bar, say so plainly: "No valid findings to keep from this session." Do not pad the report with weak items to have something to show.

### 3. Report + decide

Present each surviving finding as a summary card — **not** the final drafted text, only the summary. Fields:

- **Finding** — short title
- **What it is** — 1–2 sentences: the mistake, correction, gap, or preference observed
- **Category / where it lives** — one of the five destinations below, plus the specific target file when known
- **Why it matters** — the concrete cost/error/gain observed
- **Expected behavior after** — what should change in practice, in similar future sessions/tasks

Destinations:
1. **Harness & global rule** (merged) — `~/.claude/rules`, `~/.claude/agents`, `~/.claude/skills` — any new/adjusted agent, skill, workflow, or rule. Follow the conventions already established in that repo (mirror files under `~/.claude/rules/_meta/`, the `agents-and-skills.md` index) — reviewed at execution time via `ahead:instructions`.
2. **Project CLAUDE.md or CLAUDE.local.md** — CLAUDE.md when the lesson is a team-wide convention, CLAUDE.local.md when it's the user's personal preference/workflow.
3. **Code comment** — a non-obvious constraint discovered during the work, following the "Comment the Why" rule in `think-ahead.md`.
4. **Project reference doc** — review the project's existing documentation conventions first (may be an ADR, `docs/references/`, etc.); use `ahead:reference-docs` at execution time.
5. **Memory** (`user`/`feedback`/`project`/`reference`) — written directly in the memory system's format (entry file + `MEMORY.md` index).

Walk findings one at a time using the decision protocol (`~/.claude/rules/decision-protocol.md`): each finding is a point (🔹 Finding N/M), the card fields are its 📌, and the question (❓) is apply / skip / adjust. **CRITICAL**: never apply a finding before this question is answered — the summary card is not permission to write anything yet.

### 4. Execute (only the approved findings, one at a time, in sequence)

Once the full list is decided, draft the real text per approved finding and apply it:
- Harness/global rule → invoke `ahead:instructions` before writing
- Project reference doc → invoke `ahead:reference-docs` before writing
- CLAUDE.md/CLAUDE.local.md, code comment → write/edit directly
- Memory → write directly per the memory system format

Process approved findings one at a time, in the order they were confirmed — don't batch unrelated destinations into a single edit.

**CRITICAL**: never commit any of these edits, at any destination — leave everything unstaged for the user to review and commit later.

## Output

Conversational only — this skill does not write its own log/report file. The only artifacts left behind are the destinations from step 4 that the user approved.

## What this skill is not

- Not a bug-fix pass — it doesn't resolve the mistakes it finds, only proposes the lesson to keep.
- Not automatic — never self-triggers; Claude may suggest running it after a complex task, but only the user's go-ahead invokes it.
- Not `ahead:resolve` — that closes out observations already fixed in-session; this looks for the meta-lesson behind the session's work, fixed or not.
