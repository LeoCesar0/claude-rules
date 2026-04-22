# Observation Framework

Document issues found during any code work in `docs/observations/<area>/<type>/YYYY-MM-DD-short-slug.md`.

- `<area>`: feature or domain (e.g., `cropping`, `text-check`, `headlines`, `background`)
- `<type>`: one of the types below

## Types

Priority order (highest first): **bug > performance > security > enhancement > smell**
When an observation fits multiple types, use the higher-priority one.

- `bug` — broken or wrong results
- `performance` — works but slow or wasteful
- `security` — unsafe or exploitable
- `enhancement` — missing or incomplete functionality
- `smell` — works but poorly structured

## Before writing a new observation

- Ask the user which language to use for observation files (typically English or Portuguese-BR). Apply that choice to every observation file created during the session unless the user changes it.

## File format

```markdown
---
status: open | in-progress | resolved | discarded
type: bug | performance | security | enhancement | smell
severity: low | medium | high
found-during: "brief description of the task being worked on"
found-in: "file/path/where/spotted.ts"
working-branch: "branch-where-fix-will-be-executed"
found-in-branch: "branch-where-spotted"
date: YYYY-MM-DD
updated: YYYY-MM-DD
resolved-date:
discard-reason:
deferred:
deferred-reason:
related-commits:
related-observations:
---

# Short descriptive title

## Context
Self-contained description — a reader with no prior exposure must be able to understand and reproduce what's being observed. Always start by explaining *what* the problem or improvement is in prose, before any step-by-step. Do not summarize away detail.

Include, in this order:
- **Summary** — 1–3 paragraphs explaining, in prose, what was observed and why it matters to a reader who has never heard of it. For `bug`/`smell`: describe current vs. expected behavior. For `enhancement`/`performance`: describe the current gap vs. the desired state. For `security`: describe the exposure and the threat model. Name the feature/area in plain language before introducing function names or file paths. This subsection is required — reproduction steps never come first.
- **Examples** — actual inputs/outputs, log snippets, values, or scenarios (whenever possible)
- **User report** (when user-reported) — the user's original description of the problem or pain, rendered objectively and scientifically, without omitting details or replacing the user's words with interpretation
- **How to observe or reproduce** — concrete steps, conditions, inputs, or code paths that trigger the behavior

## Where
File(s) and line(s) affected.

## Why it matters
Impact and consequences.

## Suggested approach
Recommended fix or action.
```

## Frontmatter fields

- `date` — when first discovered
- `updated` — last change to any field or content
- `resolved-date` — when closed (empty until resolved)
- `discard-reason` — why discarded (empty until discarded)
- `working-branch` — branch where the fix will be executed; leave empty if unknown
- `found-in-branch` — branch where the issue was spotted (usually current branch at discovery)
- `deferred` — when `true`, postponed intentionally; don't re-surface unless user asks
- `deferred-reason` — brief explanation of why it was deferred (empty until deferred)
- `related-commits` — list of commit SHAs that contributed to or addressed this observation; optional
- `related-observations` — list of paths to other observation files with direct impact on this one; optional

## Before working on an observation

Observations are investigation tools, not source of truth — they are a snapshot of what was true when written. The code is the source of truth.

- Re-read the affected code — the codebase may have changed since the observation was written
- Verify the issue still exists and the suggested approach still applies
- If stale, update the observation before starting the fix

## Lifecycle

Status transitions: open → in-progress → resolved | discarded

**On resolve:**
- Set `status: resolved`, `resolved-date` to current date, update `updated`
- Add `## Resolution` section describing what was done

**On discard:**
- Set `status: discarded`, `discard-reason` with brief explanation, update `updated`
- Keep the file for historical record
