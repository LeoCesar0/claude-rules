---
description: Run the observation resolve ritual for observations handled this session — validate against the diff, present a plan, then trim/close/commit each. Closing ceremony only; never writes fix code.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *), AskUserQuestion
effort: medium
---

# Resolve Observations

Execute the resolve ritual from `~/.claude/rules/observations.md` for observations whose fix was already implemented and verified this session. This is a **closing ceremony** — it never writes fix code.

The ritual itself (retention-based trim, `## Resolution`, status/date fields, `promote` follow-up) lives in `observations.md` under **Lifecycle → On resolve**. Follow it there; this skill orchestrates *which* observations, *in what order*, and *with what safety gate*.

## Arguments

- **No args** → discover candidates automatically (default flow below).
- **Paths/slugs** (e.g. `cropping/bug/2026-05-29-foo`) → treat as an explicit scope filter; resolve only those, skip auto-discovery.
- **Free-text** (natural language) → additional constraints/reminders folded into the flow.
- **CRITICAL**: no argument disables the gate or the plan-approval step — not even "resolve tudo automático". Both are non-negotiable.

## Flow

### 1. Discover candidates

- Start from observations handled this session.
- Cross-check with `git status` and `git diff` (staged + unstaged) — **the diff is the source of truth** for what actually changed, not conversation memory.
- Detect **old** observations (not the session's focus) that the current changes touched, resolve, or turn into discards.
- Both formats count: `.md` and `.html` observation files.

### 2. Validate each candidate (the gate)

Re-read the affected code per `observations.md` → **Before working on an observation**, then classify:

- **CLEAN** — the fix is present in the diff and the observation still confers. Default verdict. Ready to close.
- **BLOCKED** — only when the code itself proves something required is missing. Two cases:
  - *Fix absent / issue still reproduces*: the change the observation calls for is not in the diff, or re-reading the code shows the broken behavior is still there. May mean the work wasn't done, or the observation should become a **discard**.
  - *A required task in the observation is unaddressed*: the observation enumerates a fundamental sub-task or section that the diff does not cover.

**Invoking this skill asserts the user has already validated everything outside the code's reach** — manual/browser checks, visual QA, `Pending Validation` steps. Do **not** block on `awaiting-validation`, unchecked `Pending Validation`, or any human-only confirmation; treat those as done. Block only on what the diff and code make verifiably absent. Schema/contract approval and unanswered `needs-input` are upstream gates, not gates here.

### 3. Present the plan (always, before any write or commit)

Stop and present, even when nothing is blocked:

- **To resolve** — ordered list (order is your judgment; respect obvious `related-observations` dependencies). Per item: retention trim plan, target commit message, and `## Follow-up` target when `retention: promote`.
- **To discard** — observations the gate found are no longer valid, each with proposed `discard-reason`.
- **Blocked** — each with the reason and a **concrete suggestion** for how to proceed.

Decide commit composition and messages yourself — present them as decisions in the plan, not as questions. Surface a question **only** for genuine ambiguity the code can't settle: e.g. unrelated changes in the working tree where it's unclear whether a file belongs in the observation's commit. Never ask about obvious grouping or message wording.

Wait for user approval. Approving the plan covers both the manual's "explicit user confirmation to resolve" and its "permission to commit". Execute nothing before approval.

### 4. Execute (after approval) — one commit per observation

In planned order, for each observation:

- Apply the `observations.md` resolve ritual (or discard ritual for discards). For `.html` files, keep the `<meta obs-*>` tags in sync with the `<dl class="frontmatter">` per `observations-html-experiment.md`.
- Set `resolved-date` and `updated` to today's date (today is provided in session context — never shell out for it).
- Update related docs: `related-observations` cross-links, and any doc named by a `promote` follow-up (surface the follow-up to the user before closing).
- Commit the fix changes **and** the observation file together.

### 5. Wrap up

Report with the calibrated findings format from `think-ahead.md`:

- **Resolved** — each with its commit SHA.
- **Discarded** — each with reason.
- **Still blocked** — each with the next step needed.
