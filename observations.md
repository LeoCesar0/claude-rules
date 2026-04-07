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

## File format

```markdown
---
status: open | in-progress | resolved | discarded
type: bug | performance | security | enhancement | smell
severity: low | medium | high
found-during: "brief description of the task being worked on"
found-in: "file/path/where/spotted.ts"
working-branch: "branch-being-worked-on-when-spotted"
found-in-branch: "branch-where-the-issue-originates"
date: YYYY-MM-DD
updated: YYYY-MM-DD
resolved-date:
discard-reason:
deferred:
---

# Short descriptive title

## What was found
Brief explanation of the issue.

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
- `working-branch` — branch being worked on when spotted
- `found-in-branch` — branch where the issue originates (may differ from working-branch)
- `deferred` — when `true`, postponed intentionally; don't re-surface unless user asks

## Before working on an observation

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

After resolving or discarding, use `AskUserQuestion` to ask whether to keep the file or delete it.
