---
description: Autonomously execute a list of tasks (observations, a list inside a spec, or a current-work file) end to end. The session becomes a lean orchestrator that dispatches one fresh subagent per task, makes its own decisions without stopping, skips only on irreversible/external actions, and reports everything done plus every decision at the end. Manual-only.
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion
effort: high
---

# Autopilot

Work a list of tasks autonomously, task by task, while the user is away. The session running this skill is the **orchestrator**: it does not edit code itself — it dispatches one fresh subagent per task, records the result, and moves on. This keeps the orchestrator's context small enough to survive a long queue without compaction.

**Prime directive**: make the best decision you can and keep going. Do not stop to ask. Decide, document the decision, move to the next task. Stop only for the four hard-stops in **Safety** — nothing else.

## Opening interview

Ask all four before doing anything. Use `AskUserQuestion` where the answer is a bounded choice; plain text otherwise. After the answers, present a one-screen recap (parsed task list with numbers, tracking doc, commit mode, scope) and wait for a single confirmation before the loop starts.

1. **Which tasks** — the user names the source and range (loose observations, a list of observations inside a spec, or a `current-work` file; e.g. "tasks 2 to 10"). Read the source, number the items, and list them back so the user confirms the exact set before execution.
2. **Tracking doc** — which file organizes the work (the `current-work`/spec from step 1, usually the same). This is where each task gets its concise record. If no such doc exists, create a lean `report.html` (reuse the HTML shell/CSS from `~/.claude/skills/ahead:visualize/template.html`) and use it as the tracking doc.
3. **Commit mode** — one of:
   - **never-commit**: leave everything unstaged so the user reviews the full diff later.
   - **branch**: create a new branch `auto/YYYY-MM-DD-<slug>` off the current branch and make **one commit per completed task**. Never push (push is a hard-stop). Follow the commit-message rules in `~/.claude/CLAUDE.md` (no `Co-Authored-By`).
4. **Scope & guardrails** — free-text from the user: how far it may go, what it must not touch. This becomes the boundary clause in every subagent briefing.

## Execution loop

Process tasks **sequentially** (later tasks may depend on earlier ones). For each task:

### 1. Brief and dispatch a subagent

Dispatch one subagent (`Agent`, general-purpose, fresh context — no worktree; subagents share the working tree so sequential edits build on each other). Instruct the subagent to, in order:

- **Orient before acting** — orient before building per `~/.claude/rules/think-ahead.md` (read relevant docs/conventions, survey existing code/patterns to reuse, heed documented pitfalls); also read the project's `CLAUDE.local.md` **before** writing anything.
- **Plan before editing** — after orienting and before the first edit, write a brief plan (the steps, the files it will touch, the expected impact/risks) and execute it. Required for every task: articulating the approach up front is the mechanism, not a formality — it stands in for plan mode, which no human will approve here.
- Do the single task with enough context to act without re-reading the whole queue, staying inside the **scope & guardrails** clause from the interview.
- Work **test-first (TDD) by default** — write the test that captures the intended behavior before the implementation, then make it pass, then validate the work with that test. This is the default for **every** task type, not only bugs; for bugs, follow reproduce → fix → verify per `~/.claude/rules/think-ahead.md`. State explicitly when tests cannot verify the real behavior (visual/DOM/external) instead of claiming it works.
- Never perform the **four hard-stops** (below) — return `needs-approval` instead.
- On failure, attempt a different approach up to **3 times**; if still failing, return `blocked` with what was tried and why.
- Write/update any relevant **observations** with full detail, following the global observation rules in `~/.claude/rules/observations.md` (and `observations-html-experiment.md` for new files) — observations carry the rich record; the tracking doc stays terse.

### 2. Require a structured return

The subagent must return a compact summary (not file dumps) with: **what it did**, **decisions taken**, **decisions discarded** (with the why), **before → after**, **test results**, and a **status**: `done` | `blocked` | `needs-approval`. Keep this in the orchestrator's context; push detail into observations and the tracking doc, not into the return.

### 3. Record and advance

- Append a **concise** new section to the tracking doc for this task: done / decisions taken / decisions discarded / before→after. Use today's date from session context — never shell out for it.
- Confirm the subagent fed the relevant observations; if it created `blocked`/`needs-approval` items, ensure an observation captures them per the framework.
- In **branch** mode and status `done`: stage and commit this task as one commit.
- Move to the next task regardless of this task's status — never let one task halt the queue.

## Safety

**CRITICAL** — this skill deliberately **overrides** the *Schema & Contract Changes* gate in `~/.claude/CLAUDE.md` for the duration of the run: schema changes, migrations, and API contract/shape changes are performed autonomously and reported at the end, **not** gated for approval. This override is scoped to this skill only.

**CRITICAL** — the following four actions are **hard-stops** and are never performed autonomously, because no diff review can undo them. On encountering one, the subagent returns `needs-approval`; the orchestrator marks the task, **skips it, and continues** the queue. All hard-stops are surfaced together in the final report.

1. Anything touching **production** (deploy, prod DB ops, cloud CLI targeting prod) — per *Production Safety* in `~/.claude/CLAUDE.md`.
2. **Pushing** to a remote, opening a PR, or triggering CI/CD.
3. **Destructive, irreversible data/work operations** (`rm -rf`, drop/truncate, `git reset --hard` that discards work, force-overwriting files outside the task's own edits).
4. Installing a **new, un-vetted dependency** — the *Dependency Supply-Chain Gate* in `~/.claude/rules/security.md` still applies in full.

A task that requires a hard-stop to complete is `needs-approval`, not `blocked`. A task the subagent could not finish for technical reasons after 3 attempts is `blocked`. Both skip and continue.

**Decisions are not stops.** Architectural forks, multiple viable approaches, ambiguity — the subagent picks the best path, records it (and the discarded alternatives) in its return, and proceeds. Do not pause the run for a decision.

If the user's project has its own `CLAUDE.md`, honor its constraints inside the scope clause — but the four hard-stops above are the floor regardless of project.

## Final report

When the queue is exhausted, generate the report **from the tracking doc**, not from memory. Print to chat using the calibrated-findings structure from `~/.claude/rules/think-ahead.md`:

- **Completed** — tasks finished, one line each; in branch mode, the commit SHA.
- **Blocked** — tasks that failed after 3 attempts, each with the next step needed and its observation.
- **Needs you** — tasks skipped on a hard-stop, each with which hard-stop and what it would take.
- **Decisions** — notable decisions taken and the alternatives discarded, with the why.
- End with where the full record lives (tracking doc path) and, in never-commit mode, a reminder that all changes are unstaged for review.
