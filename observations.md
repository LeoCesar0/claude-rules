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
status: open | in-progress | awaiting-validation | resolved | discarded
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
- **User report** (when user-reported) — a factual restatement of the user's report. Rephrase or omit personal/emotional framing (e.g. "I'm confused", "I don't know what this means") into objective observations. Preserve concrete details, examples, and reproduction info.
- **How to observe or reproduce** — concrete steps, conditions, inputs, or code paths that trigger the behavior

## Where
File(s) and line(s) affected.

## Why it matters
Impact and consequences.

## Suggested approach
Recommended fix or action.

## Progress
Appended during work — test runs (baseline → iterations), key values, decisions. Summarized away on resolve per the lifecycle rules.
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

## Tests as the primary progress tool

Drive observation work with tests / measurable comparison whenever applicable. Extends the bug-fix workflow in `think-ahead.md` to all observation types and adds in-file progress logging.

**Execution Flow:**
1. Understand and investigate the issue, validate the observation's content against the current code, validate it is still relevant and truthful; update the observation if needed
2. Run or write/update tests that capture current behavior; record the baseline output in the obs file
3. For bugs: confirm the bug reproduces; otherwise the observation may be stable and may be discarded
4. Discuss and revalidated solution or already suggested approach based on test results or current behavior
5. Apply changes
6. Re-run tests; record the new output alongside the baseline
7. Iterate until the test passes or the obs moves to `awaiting-validation`

**Logging in the obs file:**
- The obs file is the running log — record runs, values, and decisions there as work happens, not only in chat. Append to the `## Progress` section
- When test artifacts are gitignored (local outputs, fixtures, screenshots), reference the path AND inline the key data (failing assertion, relevant values, summary line). A reader without the artifact must still be able to follow the progress
- Don't paste full logs — extract the meaningful slice (failing assertion, key values, summary line)
- Skip the TDD flow only when not applicable (visual-only UI work, infra inspection, ad-hoc analysis); state in the obs why tests aren't the driver

## Lifecycle

Status transitions: open → in-progress → awaiting-validation → resolved | discarded

`awaiting-validation` is optional — skip it when the fix can be verified purely in-session (tests green, type-check clean, no user-facing behavior to eyeball). Use it when sign-off depends on something only the user can confirm.

**On hand off for validation:**
- Set `status: awaiting-validation`, update `updated`
- Add a `## Pending Validation` section: what was done, what the user needs to check, and how (steps, URLs, commands)
- Do **not** set `resolved-date` yet

**On resolve** (only after the user has explicitly confirmed the observation is resolved — never infer from passing tests or a successful fix alone):
- Ask the user for permission to commit pending changes related to the fix; once committed, record the SHA(s) in `related-commits`
- Summarize the file before marking resolved: keep the final problem statement, root cause, and fix. Remove verbose process notes and self-introduced issues that were created and fixed in-flight. If failed approaches are worth preserving, condense them under a brief `## Things tried that didn't work` subsection — otherwise drop them
- Set `status: resolved`, `resolved-date` to current date, update `updated`
- Add `## Resolution` section describing what was done (may fold in the `## Pending Validation` content)

**On discard:**
- Set `status: discarded`, `discard-reason` with brief explanation, update `updated`
- Keep the file for historical record
