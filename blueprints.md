# Blueprints Framework

Structured development workflow: **Spec → Plan → Execute → Validate**. Each stage has a dedicated skill under `ahead:*`.

## Directory Structure

```
docs/blueprints/<blueprint-name>/
  _overview.md
  <N>-<slug>.spec.md
  <N>-<slug>.plan.md
  <N>-<slug>.validation.md
```

- **Blueprint** = feature grouping (e.g., `mvp-enhancements`)
- **Feature** = numbered item within a blueprint (e.g., `1-component-foundation`)
- Execution (stage 3) produces code and commits, not a document

## Document Formats

### `_overview.md`

```markdown
---
status: draft | active | completed
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Blueprint Name

## Goal

## Features
| # | Feature | Spec | Plan | Execute | Validate |
|---|---------|------|------|---------|----------|

## Design Decisions
- **Decision**: what was chosen → **Why**: reasoning

## Dependencies
```

### `<N>-<slug>.spec.md`

```markdown
---
status: draft | approved | changed
feature: Feature Name
reproduction-test: required | optional | none
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Feature Name

## Goal
## Why
## Dependencies
## Design Decisions
## Requirements
### 1. Requirement name
```

- `approved` = reviewed by user, ready for plan
- `changed` = altered after plan exists — plan may be stale
- `reproduction-test`: `required` for bugs, `optional` for complex logic, `none` default

Bug specs add a `## Reproduction` section: expected vs actual behavior, conditions to trigger.

### `<N>-<slug>.plan.md`

```markdown
---
status: draft | approved | in-progress | done
feature: Feature Name
spec: ./<N>-<slug>.spec.md
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Feature Name — Implementation Plan

## Analysis
## Reuse Requirements
## Test Strategy
## Steps
### Step N: Name
- [ ] sub-task
## Files Created
## Files Modified
## Execution Order
## Risks / Open Questions
## Deviations
```

`## Reuse Requirements` lists existing patterns/services/utilities that MUST be reused — hard items. `## Deviations` is populated by the executor when a hard item in the plan cannot be followed as written; soft adaptations don't go here.

### `<N>-<slug>.validation.md`

```markdown
---
status: pass | partial | fail
feature: Feature Name
spec: ./<N>-<slug>.spec.md
plan: ./<N>-<slug>.plan.md
created: YYYY-MM-DD
---

# Feature Name — Validation

## Requirements Check
| # | Requirement | Status | Notes |
|---|-------------|--------|-------|

## Plan Adherence
## Issues Found
## Blockers
## Verdict
```

`## Blockers` is only populated when verdict is `fail` — it lists critical failures that prevent the plan from moving to `status: done`.

## Lifecycle

### Status transitions

- Spec: draft → approved → changed (if edited after plan exists)
- Plan: draft → approved → in-progress → done
- Overview: draft → active → completed

### Stage gates

- Plan requires spec `status: approved`
- Execute requires plan `status: approved`
- Validate requires plan `status: in-progress | done`

### Separation rule

- Spec contains no implementation details — WHAT, not HOW
- Plan does not alter requirements — flag problems, don't fix them
- Execute follows the plan — document deviations in the plan file

## Document Updates

Each stage updates specific files. On any edit, also bump the `updated` field in the file's frontmatter to today's date.

| File | Updated by | When |
|------|------------|------|
| `_overview.md` feature table | spec, plan, execute, validate | Any status transition on a feature |
| `_overview.md` decisions | spec, plan | Cross-feature decision emerges |
| `.spec.md` | spec | Create; iterate; mark `changed` if edited after plan exists |
| `.plan.md` (main content) | plan | Create; re-plan when spec changes |
| `.plan.md` checkboxes | execute | As each sub-task completes |
| `.plan.md` `## Deviations` | execute; plan (on revision) | Hard deviation found; plan edited after validation failure |
| `.plan.md` status | execute (→ in-progress), validate (→ done) | Stage transitions |
| `.validation.md` | validate | Create once per validation run |
| Observation files | any stage | Out-of-scope issue found — see `observations.md` |

## Plan Revisions

When the plan needs to change (validator failed, execute hit blockers, approach proved wrong):

- **Edit the plan in place** — keep it enxuto and current. Rewrite the affected steps to reflect the new approach.
- **Log the learning** in `## Deviations` — one entry per significant iteration. Entry format:
  - Date
  - What was originally attempted
  - Why it didn't work
  - What's being done instead

Example entry:

```
### 2026-04-12: Tried direct SQL, switched to Prisma
Step 3 originally used raw SQL for perf. Query complexity made it
unmaintainable. Switched to Prisma with indexes on foo/bar — perf is
acceptable.
```

Do not keep superseded steps visible — git preserves history, the plan stays lean. The `## Deviations` log is the human-readable "why the plan evolved" record.

## Issue Disposition

When any stage finds an issue, classify and route it:

| Category | Example | Destination | Effect |
|----------|---------|-------------|--------|
| **In-scope caveat** | "Step 3 adjusted because component X already had pattern Y" | Stage's doc (`.plan.md`, `.validation.md`) | Log only |
| **In-scope blocker** | Requirement can't be met / critical test fails | Stage's doc `## Blockers` section | **Blocks progression** |
| **Out-of-scope finding** | Bug in unrelated code noticed while working | Observation file (see `observations.md`) | Persists beyond blueprint |

### Decision rule

- Related to the feature being worked on? → stage's doc
- Noticed but unrelated? → observation file
- Prevents the stage from completing? → blocker

### Blocker protocol

When a stage cannot proceed:

1. Document under `## Blockers` in the stage's doc (what, where, why it blocks)
2. Do **not** transition status forward (spec stays `draft`, plan stays `approved`, etc.)
3. Present to user via `AskUserQuestion`:
   - **Fix now** — enter exploratory/fix mode to resolve
   - **Defer as observation** — create observation, mark stage as blocked
   - **Override** — user accepts risk, document justification in `## Blockers`, proceed

### Out-of-scope findings

For issues unrelated to the current feature, create observation files per `observations.md`. Set `working-branch` to the feature's branch if the user plans to address it there; otherwise leave empty.

## Reproduction Tests

When spec has `reproduction-test: required`:

1. The **plan** stage writes and runs the reproduction test — must fail
2. The test result informs the technical plan
3. Plan records the test as Step 0 (already completed), with commit reference
4. Execute starts from Step 1 — the failing test already exists
5. Validate checks that the Step 0 test now passes

See `think-ahead.md` § Bug Fix Workflow for the underlying principle.

## Test Strategy

The **plan** stage owns test strategy for the feature.

Every `.plan.md` includes a `## Test Strategy` section specifying:
- What will be tested, at what level (unit / integration / manual / E2E)
- What won't be tested and why (environment-specific, visual, external)
- Whether `chaos-agent` subagent will be invoked for adversarial coverage

Steps in the plan include **test sub-tasks alongside implementation sub-tasks**. Test sub-tasks the plan wrote itself are marked complete; the rest are written by execute.

The plan stage may:
- Write tests as part of verification (not just reproduction tests for bugs)
- Fix issues discovered while writing tests — scoped to what the tests reveal, not general implementation
- Spawn `chaos-agent` (see `~/.claude/agents/chaos-agent.md`) when the feature has many edge cases, complex state transitions, or high-risk data handling

## Skills

| Skill | Stage | Purpose |
|-------|-------|---------|
| `/ahead:spec` | 1 | Create/iterate feature specs |
| `/ahead:plan` | 2 | Produce technical implementation plans |
| `/ahead:execute` | 3 | Implement following spec + plan |
| `/ahead:validate` | 4 | Verify implementation against spec + plan |
| `/ahead:blueprints` | — | Status dashboard, next actions |
