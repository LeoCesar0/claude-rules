# Specs Framework

Lightweight spec-driven workflow. Features are captured as small spec files, grouped under an overview that holds shared context, reuse anchors, and the dashboard of what's done.

Implementation happens in Claude Code's built-in plan mode after a spec is approved. Validation is automated tests + user manual review — no separate validation document.

## Directory Structure

```
docs/specs/<group>/
  _overview.md
  <N>-<slug>.spec.md
```

- **Group** = feature grouping (e.g., `mvp-enhancements`, `auth-rewrite`)
- **Spec** = numbered item within a group (e.g., `1-component-foundation`)
- Always grouped — even single-spec features get an overview

## `_overview.md`

```markdown
---
status: draft | active | completed
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Group Name

## Goal

## Specs
| # | Spec | Status | Notes |
|---|------|--------|-------|

## Design Decisions
- **Decision**: what was chosen → **Why**: reasoning

## Implementation Notes

When planning any spec in this group:
- Reuse existing code, patterns, and utilities — search before creating new
- Account for edge cases (per-spec + group-wide listed below)
- Tests first for bugs; suggest tests for new features
- Read this overview before entering plan mode

### Group-specific anchors / constraints

(concrete reuse anchors, integration points, edge cases that span specs)

## Dependencies
```

The `## Implementation Notes` boilerplate carries the planning rigor; the group-specific block names concrete anchors (e.g., "use `LLMService` in `src/services/llm/`").

## `<N>-<slug>.spec.md`

```markdown
---
status: draft | approved | implemented
feature: Feature Name
reproduction-test: required | optional | none
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# Feature Name

## Goal
## Why
## Requirements

### 1. Requirement name
- expected behavior
- inputs / outputs
- spec-local edge cases

## Reproduction   ← bugs only
```

- `approved` = reviewed by user, ready for implementation
- `implemented` = code shipped and verified
- `reproduction-test`: `required` for bugs, `optional` for complex logic, `none` otherwise
- Cross-spec design decisions and reuse anchors live in `_overview.md`, not here

Bug specs include a `## Reproduction` section: expected vs actual behavior, conditions to trigger.

## Lifecycle

- Spec: `draft → approved → implemented`
- Overview: `draft → active → completed`
- Implementation requires spec `status: approved`
- Spec moves to `implemented` only after code is shipped and tests pass
- On any edit, bump the file's `updated` field to today's date

### Status Icons

Use icons in the overview's Specs table for quick visual scan. Icons go alongside the status word, not in the frontmatter `status:` field:

- 📝 `draft` (spec or overview) — not yet ready
- 🚧 `approved` (spec) / `active` (overview) — work in progress
- ✅ `implemented` (spec) / `completed` (overview) — done

Example row: `| 1 | component-foundation | 🚧 approved | first to ship |`

## Implementation Flow

Once a spec is `approved`:

1. Read the group's `_overview.md` first — Implementation Notes live there
2. Enter Claude Code's built-in plan mode for the spec
3. Implement following the plan
4. Validate via automated tests + user manual review
5. Mark spec `status: implemented`, update overview's Specs table

For bug specs with `reproduction-test: required`, write the failing test first, commit it as `test: reproduce <description>`, then implement the fix and verify the test passes.

## Issue Disposition

- **Within scope** → iterate the spec or address during implementation
- **Out of scope** → create an observation file (see `observations.md`)
- **Blocker** → stop, report to user with concrete blocker description

## Skills

- `/ahead:specs` — design and iterate spec groups (creates overview + specs through discussion)

## Migration from `docs/blueprints/`

Move feature folders to `docs/specs/<group>/`, keep `.spec.md` files (drop `.plan.md` and `.validation.md`), and ensure each group has an `_overview.md` with Specs table and Implementation Notes.
