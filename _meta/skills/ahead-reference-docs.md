---
name: reference-docs
description: Use when writing, porting, or editing a project's persistent reference documentation — architecture/design descriptions, references/, workflows/, or any doc that describes how the system currently works (not changelogs, ADRs, or dev notes). Enforces a timeless present-state reading for a stranger or an LLM with zero conversation context: strips process/sprint jargon, transition narrative, and personal voice; keeps permanent rationale and durable anchors; forbids links to observations and dev meta-docs.
---

# Reference Docs

Write and edit persistent reference documentation so it states the system's
present as timeless fact — readable by a stranger, or by an LLM that has only
the code and this doc, never the conversation that produced it. The audience is
both: a third-party human and Claude in a later session.

Applies to reference/architecture/workflow docs describing how the system
works. Does not apply to intentionally temporal docs — changelogs, ADRs,
migration guides, dev meta-docs, current-work — which keep their timeline.

Specializes `~/.claude/rules/objective-writing.md` for this doc type — the
timeline-stripping rules below are its enforcement checklist for reference
docs specifically.

## The test

Every sentence must be true and understandable to a reader who has seen only
the code and this doc, with zero knowledge of the project's timeline. If a
sentence needs the timeline to make sense, the fact is in the wrong place —
rewrite it to the permanent fact or move it out.

## Strip — remove directly, no approval

- Phase/sprint/process jargon — "Fase 5", "S5", "P1b", "a frente trocou",
  "resposta ao revisor". Marks of the process, not facts of the system.
- Transition narrative — "desde a Fase 5", "mudamos X para Y", "não mais",
  "agora passou a", "antes vs depois". State the current state, not the change.
- Personal/conversational voice — "nós decidimos", "a gente optou", "acho que".
  State the fact, not who chose it.
- Cosmetic, redundant, or process-narration filler.

Examples:
- "Desde a Fase 5, a detecção é uma onda única, não mais duas." →
  "A detecção é uma onda única."
- "A deferência não some — moveu do prompt para a reconciliação." →
  "A deferência é garantida na reconciliação pós-hoc."

## Preserve — do not over-clean

- Design rationale that is a permanent property — "o engine bypassa o juiz
  porque o juiz tende a remover engine válido". Atemporal cause stays.
- Durable anchors — commit SHA, symbol/function name, file path, reference to
  another published doc. Traceability, not process jargon.

Line: rationale-as-permanent-fact stays; rationale-as-historical-episode
("desde que observamos na Fase 4…") moves out.

## Load-bearing rationale — signal, don't delete

- When stripping the why/when would drop a non-obvious decision or trade-off,
  do not delete silently. Flag it and ask whether to preserve it elsewhere.
  Remove only after the user decides.
- A doc that fuses a timeless reference with an investigation/study
  (measurements, dated decisions): signal the split to the user. Do not split
  or move automatically.

## Links and meta-docs

- **CRITICAL**: No personal/non-shared artifact links — see
  `~/.claude/rules/objective-writing.md`. When porting or editing, remove such
  links that already exist.
- Reference another published doc instead of restating — prefer a link over
  duplication.
- A permanent fact whose only record is a meta-doc: absorb the fact inline; do
  not link the meta-doc.
- Data from a gitignored or personal meta-source (e.g. eval/test results): state
  that the study was done and give a short summary. Do not link the gitignored
  path. Do not paste the full data — unless it is small, then inline it.

## Language

- New doc: match the language of the neighboring docs in the same directory.
- Editing an existing doc: keep the language already used in that doc.
- Do not impose a fixed language.

## Stay project-agnostic

- Do not assume a fixed docs path, an ADR directory, or any specific project
  structure. Apply only the flexible rules here.
- Do not prescribe where moved-out history should live — that is the user's call.

## On finish

- Report what changed and why.
- Surface any note or worth-noting; use the findings format in
  `~/.claude/CLAUDE.md` when there are caveats.

## Final checklist

Scan the doc and resolve before calling it done, matching terms to the doc's
language:

- Phase/sprint names.
- Transition words — "desde"/"since", "não mais"/"no longer", "passou a"/"used
  to", "antes"/"before", "mudamos"/"we changed".
- Personal voice — "nós"/"we", "a gente", "acho"/"I think".
- Links to observations or dev meta-docs.
