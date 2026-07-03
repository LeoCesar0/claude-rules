---
description: Build a standalone, portable HTML report for an EXTERNAL audience — a code reviewer, MR/PR approver, stakeholder, or anyone outside the project's daily dev context — to present a decision, a change summary, or context they must act on. Use when the user asks for a "report"/"relatório" aimed at a reviewer, revisor, someone external, or to accompany an MR/PR. NOT for internal working docs (current-work, observations) — those keep the project dashboard style.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git *)
effort: medium
---

# Reviewer Report

Produce a self-contained HTML report for someone **outside** the project's daily context. The reader is deciding or reviewing, not developing — they lack the internal vocabulary, cannot open your local files, and will judge from this page alone.

Distinct from internal working docs. The `current-work` dashboard and observations use the project's shared `report.css` (dense, dashboard-style, linked stylesheet) — that standard lives in `~/.claude/rules/current-work.md`. This skill is the opposite audience: a **presentational, embedded-CSS** page meant to travel.

## Output

- Copy `reviewer-report-template.html` (same folder) as the starting point. Its CSS is the standard — do not restyle it; fill the body.
- **Embedded CSS only.** Self-contained, single file — it will be sent or opened elsewhere. Never link an external stylesheet.
- Location: a gitignored working dir if one exists (`docs/tmp/` when present and gitignored), else project root. Descriptive filename (e.g. `report-<assunto>.html`).
- Match the language the user is working in (default pt-BR in most projects).

## CRITICAL content rules

These are the learnings the skill exists to enforce. Violating them defeats the report.

- **No dev-internal or broken references.** Never link or cite anything the reader cannot open or should not see: local observations, `docs/tmp/` paths, ADR/spec files, tracker/dashboard files, gitignored docs, internal file paths. If the reader needs the rationale, **explain it inline** in the report. Before finishing, grep the file for `docs/`, `observations`, `.md`, and internal path fragments — the only links that survive are external URLs the reader is meant to follow (MR/PR/ticket).
- **Translate every internal term.** Never use a task ID, batch name, code alias, or project jargon without explaining it in the report itself. When you reference a named group of work (e.g. an internal batch like "Grupo A"), add a one-line explanation **and a mapping table** of what it contains. Assume zero prior exposure to the project's vocabulary.
- **First person for the author's voice.** When the report carries the user's opinion or recommendation, write it in first-person singular as if the user wrote it ("removi", "minha preferência"). Do not slip into "we"/third person for the author's own actions and stance.
- **One emphasis mechanism.** Use the `.hl` class (semibold) for ALL keyword emphasis. Never mix `.hl` with `<b>`/`<strong>` in the same document. Cap emphasis at **1–2 highlights per block** — a wall of bold reads as no emphasis.
- **Objective, not verbose.** Every block earns its place. Cut process narration; state substance.

## Structure

- **Header** — eyebrow (contextual label) + h1 (the question or subject) + subtitle (1–2 sentences framing the read) + the single anchor link the reader needs (MR/PR/ticket) in the `.mr` chip. Omit the chip if there is no such link.
- **Context** — what the reader must know before the body. This is where jargon gets translated and named work batches get their mapping table.
- **Item cards** — one `.card` per thing under discussion: what it is, a concrete `<pre>` example, status `.badge`s, and a "what X means" line.
- **Opinion** (`.opinion`, optional) — the author's stance, first person.
- **Ask** (`.ask`) — the objective question/CTA to the reader.
- **Sticky decision rail** (`.rail` + `.decision-card`) — the **default for decision reports**: the options live in a card fixed on the right, visible while the reader scrolls the left column. It collapses to one column under 920px and scrolls internally if taller than the viewport. **Omit it for pure summaries** (no options to choose) — then remove the `.layout`/`.rail` wrappers and use a single column, per the comments in the template.

## Interactive decision capture (optional)

For a **decision report where the reviewer answers per item**, the template ships an optional widget: each question gets choice buttons + an optional reason field, and a sticky **Copiar decisões** button emits a markdown block the reviewer pastes into the MR/PR. Use it when the report asks the reader to pick, per item, from a fixed set of options. Omit it (and delete its CSS block + `<script>`) for pure summaries or open-ended asks.

The engine is **data-driven — never edit the `<script>`**. Configure only via HTML:

- One `.decide[data-item][data-title]` block per question, inside its card. `data-item` is the unique key; `data-title` is the full title used in the copied text.
- Each `.choice` button: `data-opt` (machine key), visible text (short label for the live summary), optional `data-label` (full label for the copied text — falls back to the visible text).
- **Color is decoupled from meaning** — assign any `-opt-{danger,good,accent,warn,neutral}` palette class to any option.
- One `.sum-row` per question in the rail, its `data-sum` matching the card's `data-item`.
- Copy heading via `data-copy-heading` on `[data-decisions]`; line labels/status strings via `data-decision-label` / `data-motive-label` / `data-ready-text` / `data-pending-text` (all defaulted).

Rules:
- **Reason is optional** — the copy button enables once every question has a choice, regardless of reason fields; an empty reason is omitted from the copied text.
- **Clipboard has a `file://` fallback** — the script tries the async Clipboard API, then `execCommand('copy')`, since a report opened as a local file is not a secure context.

## Workflow

1. Identify **audience, purpose, and the one link** the reader acts on. Confirm it is an external/reviewer report — if it is an internal working doc, use the `current-work` standard instead.
2. Gather the substance from the conversation, diff, or task list. For every internal term, decide its inline translation.
3. Copy the template. Fill the header, context (+ mapping table when a named batch is referenced), item cards, opinion, ask. Keep the sticky decision rail for decisions; drop it for summaries. Wire the interactive decision capture (above) when the reviewer answers per item; otherwise delete its `.decide` blocks, the `.summary` block, its CSS, and the `<script>`.
4. Apply emphasis with `.hl` only, 1–2 per block.
5. **Verify before done**: grep the file for internal/broken references (see CRITICAL rules); confirm every internal term is explained; confirm the author's voice is first person; confirm spans are balanced.
6. Save to the gitignored working dir; report the path. Do not commit unless asked.

## What this skill is not

- Not for internal dashboards or trackers — that is `current-work.md`.
- Not a data-viz tool — for charts/graphs use the dataviz skill.
- Not `ahead:visualize` — that renders a conversation to explain it to yourself; this builds a structured deliverable for an external reader.
