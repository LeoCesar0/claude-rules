---
description: Discuss and craft a complete spec group — _overview.md plus multiple .spec.md files. Used to break a sprint or large feature into small, well-scoped specs.
allowed-tools: Read, Glob, Grep, Bash(git *), Write, Edit, AskUserQuestion, Agent
effort: high
---

# Specs

Design and iterate spec groups. Output is `_overview.md` plus N `<N>-<slug>.spec.md` files under `docs/specs/<group>/`.

See `~/.claude/rules/specs.md` for document formats and lifecycle.

## Invocation

- `/ahead:specs` — list existing groups, ask which to work on (or create new)
- `/ahead:specs <group>` — work on a specific group (create if missing, iterate if exists)

## New Group

When `docs/specs/<name>/` does not exist:

1. Discuss scope — what does this group cover? what's in, what's out?
2. Identify reuse anchors — what existing code/patterns must implementations leverage? Read code superficially to validate.
3. Decompose into individual specs — each spec is one feature, small enough to land in one focused implementation pass
4. Draft `_overview.md` and each `<N>-<slug>.spec.md` as skeletons
5. Present each draft via `AskUserQuestion` with `preview` — options: "Approve", "Iterate", "Discard"
6. On approve: spec `status: approved`, overview `status: active`

## Iterate Existing Group

When the group exists:

1. Read `_overview.md` and all spec files
2. Ask what to work on:
   - Add new specs to the group
   - Iterate an existing draft spec
   - Update Implementation Notes (reuse anchors, constraints)
   - Mark a spec `implemented` (after user confirms code is shipped and tests pass)
3. Apply changes, re-present for approval

## Spec Skeletons

Each spec file is a skeleton, not a fleshed-out plan:

- **Goal** — one paragraph, what the feature enables
- **Why** — motivation, user/business value
- **Requirements** — numbered list of what must be true when done; each names expected behavior, inputs/outputs, and spec-local edge cases
- **Reproduction** — bugs only

Cross-spec context (design decisions, reuse anchors, group-wide edge cases) belongs in `_overview.md`, not in individual specs.

## Bug Specs

When a spec describes a bug fix:

- Add `## Reproduction` section: expected vs actual behavior, conditions to trigger
- Set `reproduction-test: required` in frontmatter
- Focus on observable behavior, not suspected cause — implementation investigates root cause

## Decomposition Guidance

When breaking a large feature into specs:

- One spec = one focused implementation pass — if implementation would touch many unrelated areas, split
- Specs should be independently approvable — if two specs are coupled enough that neither makes sense alone, merge them
- Sequential dependencies are OK — note them in `## Dependencies` in the overview
- Avoid speculative specs — don't write specs for "maybe later" features

## Implementation Notes Section

When creating or updating `_overview.md`, the `## Implementation Notes` section must include:

- The boilerplate reminders (reuse, edge cases, tests first for bugs, read overview before plan mode)
- Group-specific anchors — concrete file paths or patterns implementations must use (e.g., "use `LLMService` in `src/services/llm/`")
- Group-wide edge cases or constraints

Push the user to fill in concrete anchors during discussion — empty Implementation Notes defeats the purpose of the overview.

## Findings

- In-scope issues with requirements → iterate the spec
- Out-of-scope bugs/smells noticed while reading code → create observation files (see `observations.md`)
- Can't produce a coherent spec → flag as blocker to user, stop

## Constraints

- **No implementation details in specs** — describe WHAT, not HOW. No code snippets or step-by-step technical plans.
- **Shallow code reading only** — validate assumptions, name reuse anchors. Don't deep-dive design.
- **Challenge scope** — push back on over-engineering, speculative specs, or specs that should be split
- **Specs over discussion** — capture decisions in files, not in chat
- **Status icons in tables** — every row in the overview's Specs table uses icons alongside the status word (see `~/.claude/rules/specs.md` § Status Icons)
