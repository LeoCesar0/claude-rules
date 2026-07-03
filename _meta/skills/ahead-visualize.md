---
description: Render the current conversation (or a focused part of it) as a richly formatted, didactically enhanced HTML page that opens automatically in the browser. For when the user wants to deeply understand something — not just read it.
allowed-tools: Read, Write, Bash(date *), Bash(xdg-open *), Bash(mkdir *)
effort: medium
---

# Visualize & Explain

Generate an HTML rendering of the current conversation (or a part of it) that goes **beyond formatting** — it actively helps the user understand. The output opens in the browser automatically.

## Purpose

Long Claude Code responses are hard to digest in monospace chat: hierarchy collapses, examples mix with prose, terms-of-art appear without explanation, decisions arrive without their reasoning surfaced. This skill produces a reading-mode HTML where:

- Structure is visually obvious (proper h1/h2/h3, callouts for key insights)
- Implicit context is made explicit (terms defined, jargon expanded, abbreviations spelled out)
- Decisions show their *why*, not just their *what*
- Comparisons live in tables
- Code is properly highlighted, not inline-collapsed
- Warnings and trade-offs are surfaced as callouts rather than buried in prose

This is **not** a typesetter — you are allowed and encouraged to enrich, explain, and reorganize for clarity. You are not allowed to invent facts or change conclusions.

## Invocation

- `/ahead:visualize` — render the **last substantive assistant message** in the conversation
- `/ahead:visualize <focus>` — render content related to the focus, drawing from the conversation as needed (e.g. `/ahead:visualize the hook architecture we just discussed`, `/ahead:visualize all the decisions about cropping`)

## Steps

1. **Identify content to render**:
   - No argument → use the last substantive assistant message (skip yours-just-now if the user invoked `/ahead:visualize` immediately; pick the previous substantive answer)
   - With argument → identify the slice of conversation matching the focus. Could span multiple messages. Use your conversation memory directly — do not re-read files unless necessary.

2. **Read the template** at `~/.claude/skills/ahead:visualize/template.html`. It has 4 placeholders: `{{TITLE}}`, `{{SUBTITLE}}`, `{{EYEBROW}}`, `{{BODY}}`, `{{TIMESTAMP}}`.

3. **Compose the rendering** — see "How to render" below for the augmentation pattern.

4. **Generate output path**: `/tmp/claude-render-$(date +%Y%m%d-%H%M%S).html`. Get the timestamp via `date +%Y%m%d-%H%M%S` (Bash). Each invocation creates a new file — past renders persist until reboot.

5. **Write the file** using Write. Substitute the placeholders inline.

6. **Open in browser**: `xdg-open /tmp/claude-render-<timestamp>.html &` (Bash, background so the command returns immediately).

7. **Confirm in chat**: one short line stating the file path. No long summary — the HTML *is* the summary.

## How to render — the augmentation pattern

### Header

- `{{EYEBROW}}` — a short tag like "Discussão · Arquitetura", "Análise", "Resumo de decisão". Lowercase contextual label.
- `{{TITLE}}` — the main subject in one phrase (e.g. "HTML Render Hook: por que falhou e o que veio depois")
- `{{SUBTITLE}}` — italic one-liner that frames the read (e.g. "Race condition + drift conversacional levaram a um pivot para skill")
- `{{TIMESTAMP}}` — `$(date '+%Y-%m-%d %H:%M')` in the local timezone

### Body — the augmentation rules

Write HTML directly into `{{BODY}}`. No `<html>`/`<head>`/`<body>` wrappers — those are in the template.

**Always start** with a `<p class="lead">` paragraph (1-3 sentences) that answers: *what is this page about, and what's the single most important takeaway?* This is the TL;DR — written so the user can stop reading after the lead if pressed for time.

**Then enrich** as you compose the rest. The enrichment moves (apply where useful, not everywhere):

| Move | Example |
|---|---|
| **Define on first use** | First mention of "race condition", "Stop hook", "transcript JSONL" — give a one-line definition inline or in a `<dl class="glossary">` block. |
| **Make the *why* explicit** | "Escolhemos X" → "Escolhemos X porque Y; a alternativa Z teria custado W." |
| **Lift warnings into callouts** | A caveat buried in prose → `<div class="callout warn">` with `<h4>` label. |
| **Recommendations stand out** | "Recomendo A" → `<div class="callout good"><h4>Recomendação</h4>...` |
| **Trade-offs in tables** | Comparing options → `<div class="table-wrap"><table>` with columns like "Opção", "Prós", "Contras". |
| **Expand abbreviations** | "CC" → "Claude Code (CC)" on first use. |
| **Add file paths as `<span class="path">`** | Mentions of files → render as path chips, not bare text. |
| **Show a glossary when terms are dense** | If 3+ specialized terms appear, add a `<dl class="glossary">` block early in the body listing them with short definitions. |

### What NOT to do

- Don't invent facts, numbers, file paths, or quotes
- Don't change recommendations or conclusions
- Don't omit substantive information just to make the page shorter
- Don't translate — preserve the language the user has been using (typically PT-BR in this project)
- Don't make up a glossary entry if the term isn't actually in the source content
- Don't include greetings, sign-offs, or meta-commentary ("Aqui está o resumo…")

### CSS classes available (from the template)

**Callouts**: `<div class="callout">`, `<div class="callout good">`, `<div class="callout warn">`, `<div class="callout bad">`. Inside, use `<h4>` for a label and `<p>` for content.

**Verdict badges**: `<span class="verdict good|warn|bad|neutral|accent">label</span>` — inline pills for status/labels.

**Tables**: always wrap in `<div class="table-wrap"><table>...</table></div>`. The CSS handles the styling.

**Glossary**: `<dl class="glossary"><dt>term</dt><dd>definition</dd>...</dl>` — for definition lists.

**Paths**: `<span class="path">~/.claude/skills/...</span>` — for file/folder references.

**Code**: `<pre><code>multi-line</code></pre>` for blocks, `<code>inline</code>` for inline.

**Lead**: `<p class="lead">...</p>` — for the TL;DR paragraph at the top.

## When the input is sparse

If the source content is very short (one sentence, a confirmation), still produce a useful page: explain the surrounding context, define any term that appeared, give the user something to read. Don't refuse — the user invoked the skill because they wanted understanding, not just transcription.

If the user's `<focus>` argument is vague, interpret broadly and produce something coherent; do not stop to ask for clarification.

## Output to chat

After writing and opening, respond with a single short line in chat:

```
✓ /tmp/claude-render-<timestamp>.html (aberto no navegador)
```

No long summary. The page *is* the deliverable.
