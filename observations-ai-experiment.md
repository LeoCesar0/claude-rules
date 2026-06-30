# Observations · AI-First Content (EXPERIMENT)

**Status:** experimental. Overrides **only the authoring style and content emphasis** of new observations — not the file format, frontmatter fields, statuses, or lifecycle. New observations stay `.md`.

**To toggle off:** remove the matching `@`-import line from `~/.claude/CLAUDE.md` **and** add this file to `claudeMdExcludes` in `~/.claude/settings.json` (the directory scan loads it otherwise). The file stays on disk; reverse both steps to re-enable. **To revert permanently:** also delete this file.

This overrides `~/.claude/rules/observations.md` on content only. Everything else there still applies unchanged: directory path (`docs/observations/<area>/<type>/YYYY-MM-DD-short-slug.md`), the type set and priority order, every frontmatter field, statuses, the full lifecycle (retention trims, Pending Validation, Resolution), before-/after-work rules, and tests-as-progress.

## Audience

- Write each observation for Claude reading it later — not for a human with no prior exposure.
- Claude re-reads the code. Do not re-narrate what the code already states. Document what the code **cannot** tell: root cause, the reasoning behind it, expected-vs-actual intent, trade-offs, and (for `security`) the threat model.
- Point at code with exact coordinates instead of describing it in prose.

## Content overrides

These replace the human-oriented guidance in `observations.md`'s `## Context` block.

### Summary line
- The first body line is a single machine-scannable summary: `<what> → <impact>`. Relevance must be judgeable from the frontmatter plus this line alone, without reading the rest.

### Context / What
- Replace the "reader with no prior exposure" onboarding prose with: claim → impact → anchor.
- Keep self-contained only for the non-obvious — the parts not readable from the code. Drop explanations of what a feature or function does.

### Anchors
- Anchor every claim to code as `anchor: path:symbol:line`. Always include the symbol name and the exact condition/string — line numbers drift.
- For bugs, state `expected:` and `actual:` explicitly instead of describing the gap in prose.

### Reproduction
- Write repro as deterministic numbered steps with exact inputs, commands, or values — never prose. A reader must replay it without interpretation.

### Cross-links
- Link related observations inline as `[[YYYY-MM-DD-short-slug]]` (the related file's slug). Build the graph; do not restate the linked observation's content.

## Skeleton

Frontmatter uses the fields and values defined in `observations.md`, plus one marker field this experiment adds:

- `experiment: ai-first` — set on every observation written under this experiment. Absent on normal observations. Harvest the cohort for evaluation with `grep -rl "experiment: ai-first" docs/observations/`.

The body:

```markdown
# Short descriptive title

<one-line machine summary: what → impact>

## What
<claim, 1–2 lines>
anchor: path:symbol:line
expected: <...>
actual: <...>

## Root cause
<the part not visible in the code; link [[related-slug]] when relevant>

## Repro
1. <deterministic step with exact input/command>
2. ...

## Where
path:line(s)

## Why it matters
<impact / consequence>

## Suggested approach
<fix or action>

## Progress
<appended during work, per observations.md>
```

- Drop `## Root cause` and `expected`/`actual` for non-bug types where they don't apply; keep the type-appropriate content from `observations.md` (threat model for `security`; current-vs-desired for `enhancement`/`performance`).
- `## Open Questions`, `## Pending Validation`, and `## Resolution` appear under the same conditions as in `observations.md`.
