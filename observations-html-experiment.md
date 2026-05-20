# Observations · HTML Format (EXPERIMENT)

**Status:** experimental. To revert: delete this file, delete `~/.claude/rules/observation-template.html`, and remove the matching `@`-import line from `~/.claude/CLAUDE.md`. No edits needed in `observations.md`.

This file **overrides only the file format** of new observations. Everything else in `~/.claude/rules/observations.md` (types, frontmatter fields, statuses, lifecycle, before-/after-work rules, tests-as-progress, Pending Validation / Resolution semantics) still applies unchanged.

## Format

- New observation files: write as **HTML** at `docs/observations/<area>/<type>/YYYY-MM-DD-short-slug.html`
- Existing `.md` observation files: **do not convert, do not touch** — they remain valid in their original format
- Copy the template at `~/.claude/rules/observation-template.html` as the starting point. Keep the `<style>` block intact so each file renders standalone in a browser.

## Frontmatter encoding

- Render the frontmatter as a visible `<dl class="frontmatter">` immediately after `<header class="title-block">` and before the Summary section
- Use the **same field names** as `observations.md`: `status`, `type`, `severity`, `found-during`, `found-in`, `working-branch`, `found-in-branch`, `date`, `updated`, `resolved-date`, `discard-reason`, `deferred`, `deferred-reason`, `related-commits`, `related-observations`
- **Omit empty fields entirely** — drop the `<dt>/<dd>` pair rather than emitting an empty `<dd>`
- Render `status`, `type`, and `severity` values as `<span class="verdict X">…</span>` inside their `<dd>` using the badge map below — gives at-a-glance scan
- Render file/branch paths inside `<span class="path">…</span>` (mono styling)
- Render list-valued fields (`related-commits`, `related-observations`) as comma-separated `<code>` chips inside the `<dd>`

### Badge color map

| Field    | Value                | Class                |
|----------|----------------------|----------------------|
| status   | `open`               | `verdict warn`       |
| status   | `in-progress`        | `verdict accent`     |
| status   | `awaiting-validation`| `verdict warn`       |
| status   | `resolved`           | `verdict good`       |
| status   | `discarded`          | `verdict neutral`    |
| severity | `high`               | `verdict bad`        |
| severity | `medium`             | `verdict warn`       |
| severity | `low`                | `verdict neutral`    |
| type     | `bug`                | `verdict bad`        |
| type     | `security`           | `verdict bad`        |
| type     | `performance`        | `verdict warn`       |
| type     | `enhancement`        | `verdict accent`     |
| type     | `smell`              | `verdict neutral`    |

## Required section order

1. `<header class="title-block">` — eyebrow, `<h1>` title, optional `<p class="subtitle">`
2. `<dl class="frontmatter">` — all populated fields
3. `<section id="summary">` — `<p class="lead">` (1–2 sentences) + one `<div class="callout">` containing the type/severity/status verdict badges and a one-line "where / what flows" line
4. `<section id="context">` — Context prose; subsections `Examples`, `User report` (when user-reported), `How to observe or reproduce`
5. `<section id="where">` — files / lines affected
6. `<section id="why">` — Why it matters
7. `<section id="approach">` — Suggested approach
8. `<section id="progress">` — appended live during work; one `<div class="progress-entry">` per run/decision
9. `<section id="pending-validation">` — **conditional**, only when `status: awaiting-validation`
10. `<section id="resolution">` — **conditional**, only when `status: resolved` (may include an `<h3>Things tried that didn't work</h3>` block)
11. `<footer class="footer">` — area, type, date, branch

The Summary section is **top-level** here (it was nested inside Context in the markdown format). The long-form "Summary" subsection from `observations.md` Context still lives inside the Context prose — the top-level Summary is a 1–2 sentence headline plus the verdict callout, not a replacement.

## Reusable building blocks (from the template)

- `<div class="callout {good|warn|bad}">` — highlighted note inside any section
- `<div class="recs">` with `<div class="rec-card p{1|2|3}">` — when Suggested approach has multiple ranked options
- `.table-wrap > table.eval` — structured examples / comparisons
- `<pre class="code">` and `<code class="inline">` — code samples
- `<nav class="toc">` — optional, add only when the observation is long enough to warrant it (~6+ main sections or many subsections)
- `<div class="findings">` with `.finding-list.must` / `.finding-list.note` — when wrapping up with a calibrated findings split

## Editing an existing HTML observation

- Use the `Edit` tool with exact strings — never rewrite the file
- When updating a frontmatter value, edit only the inner content of that specific `<dd>`
- When appending to Progress, insert a new `<div class="progress-entry">` immediately before `</section>` of `#progress` — never rewrite the section
- Bumping `updated`: edit only the `<dd>` for the `updated` field

## Tooling

Treat HTML observations like prose: `Read` + `Edit`. No build step, no preview tool — files render directly in a browser if needed.
