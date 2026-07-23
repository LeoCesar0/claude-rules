# Current-Work File Standard

Structure for the current-work file — the lean, human-facing hub tracking what
is being worked on now. The session-start read trigger lives in
`think-ahead.md` ("Current Work Context"); this file defines how to write,
structure, and reset it.

Apply when creating, resetting, restructuring, or updating the current-work
file. Does not apply to a one-off read.

## Nature

- It is a **summary and a hub**, never the source of truth — the detail of each
  item lives in its linked observation or spec. Never paste content that already
  lives in an observation; link to it.
- **IMPORTANT**: Current-work records *what* is being worked on, never the detail
  of the work.
- Progress notes, decisions, changed values, findings, trade-offs, and
  specifics live in the linked observation or spec — never in a current-work
  row, and never accreted into a row as the task evolves.
- Write all content in **Portuguese-BR**. Only technical tokens (paths, branch
  names, commit SHAs, identifiers) stay as-is.
- **IMPORTANT**: For `.html`, link an external stylesheet
  (`<link rel="stylesheet">`) and never embed a `<style>` block. The stylesheet
  path is per-project — keep whatever the project already uses; do not
  standardize or move it.

## Structure

For `.html`, required element order inside a single `div.container`:

1. `header.title-block` — `eyebrow` (`<projeto> · <branch> · atualizado
   YYYY-MM-DD`) + `h1` + `subtitle` (objetivo do arco atual)
2. `callout` with `id="proximo"` — the single next action ("▶ Próximo"); one
   short paragraph
3. `section#sprint` — "Sprint atual": tasks being worked on now
4. `section#curto-prazo` — "Curto prazo": tasks to pick up next
5. `section#feitas` — "Feitas recentemente" (optional; trimmed on reset)
6. `section#hub` — links to the specs, observations, and reference docs in play
7. `footer.footer` — `atualizado YYYY-MM-DD · branch · fonte`

Add `nav.toc` after the header only when the file grows past ~5 sections.

Use the project's shared report-stylesheet classes (`container`,
`title-block`/`eyebrow`/`subtitle`, `callout`, `table-wrap`+`table.eval`,
`verdict`, `code.inline`, `footer`). Do not invent classes or inline styles.

For a `.md` current-work file, mirror the same sections and tables in Markdown.

## Tasks

- Represent tasks as **rows in `table.eval`**, not cards.
- Board columns (`#sprint`, `#curto-prazo`): `ID` · `Status` · `Task` ·
  `Resumo` · `Obs`.
- "Feitas recentemente" columns: `ID` · `Item` · `Resultado · commit`.
- **`ID`** — a short, stable identifier (e.g. `T17`). It survives across boards:
  the same task keeps its ID as it moves `Curto prazo → Sprint atual → Feitas`.
  Never renumber on reset.
- **`Status`** — a single `verdict` badge with one fixed emoji per state,
  exactly: `verdict accent` 🔄 (em andamento), `verdict neutral` ⬜ (aberto),
  `verdict warn` ⏸️ (aguardando validação — trabalho implementado, falta o
  usuário validar/aprovar), `verdict bad` 🔒 (bloqueada — não iniciada,
  aguardando decisão ou input do usuário antes de poder começar), `verdict
  good` ✅ (feito). One emoji per verdict — never vary the emoji used for a
  given verdict. **Never conflate ⏸️ and 🔒** — ⏸️ means the work is done and
  waiting on the user; 🔒 means the work has not started because it is
  waiting on the user first.
- **`Task`** — the task name as a short noun phrase; one line. Not a place for
  progress or approach.
- **`Resumo`** — one to two lines stating what the task is about; a brief
  description only, never more than two lines.
- **IMPORTANT**: `Resumo` never accretes. Do not append progress notes,
  decisions, changed values, findings, or specifics to it as the task evolves —
  those belong in the linked observation or spec.
- On every update, re-point or shorten `Resumo`; never grow it past two lines. If
  the task needs more than two lines to describe, that detail belongs in its
  observation or spec, and `Resumo` links there via the `Obs` column.
- **`Obs`** — a relative, clickable `<a href>` to the observation file
  (`../observations/<area>/<type>/YYYY-MM-DD-slug.html|md`), link text = short
  label. Use `—` when there is no observation.

## Hub

- `section#hub` links each spec group and reference doc in play, as relative
  `<a href>`. One-line label per link; never restate the linked content.

## Reset / refactor

- On finishing a session or sprint, trim the file: drop or shorten "Feitas
  recentemente", remove resolved rows from the boards, and re-point `#proximo`.
- Preserve the stable `ID` of tasks still in flight; only drop IDs of tasks
  fully done and trimmed out.
