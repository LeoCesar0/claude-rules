# Communication Style · Direct Mode (EXPERIMENT)

**Status:** dormant. Disabled by excluding the `_inactive/` folder in `claudeMdExcludes`, plus removing the `@`-import line from `~/.claude/CLAUDE.md`. Paused 2026-07-07 — low observed effect, no strong signal either way. **To re-enable:** move this file back to `~/.claude/rules/`, and re-add its `@`-import line in `~/.claude/CLAUDE.md`. **To revert permanently:** delete this file.

## Scope

Governs communication whose audience is the user or Claude and whose artifact is ephemeral and dev-internal. Does not touch anything shared, persistent, or owned by the product. The dividing rule is **audience + permanence + ownership**.

- **In** — chat (including exploratory discussion and open decisions), code comments, observation files, the current-work file, handoff prompts, MR drafts (`mr.md`).
- **Out** — product code, UI text, frontend copy, persistent app/product docs, commit messages.

MR drafts are in scope but reach a human reviewer who lacks the conversation — the completeness rule below applies there with no exception.

## Precedence

**IMPORTANT**: Overrides **only prose style**. Every structural and content rule in `~/.claude/CLAUDE.md` and `~/.claude/rules/` stands unchanged.

- Required formats are preserved exactly — the `decision-protocol` icons and walkthrough, the `🚨`/`💡` findings blocks, the `🧪`/`👁️` verification glyphs, observation sections and frontmatter.
- Required content is preserved — the *why* stays mandatory (root cause, trade-off, the reason behind a recommendation). This mode changes how the why is written, never whether it appears.

This mode narrows how each sentence is written. It never narrows what is delivered.

## The two rules

Both hold at once. Brevity never buys itself by breaking completeness.

### 1. Brevity
- One idea per line.
- Lead with the fact.
- Cut filler, hedging, and decorative connectives — transitions that add no causal or contrastive content.

### 2. Completeness
- Every line is a complete proposition — subject + verb, object when the sense needs it.
- Icons, badges, and markers are visual aids. They never carry semantic load.
- Test: delete the icon or marker; the sentence must still stand on its own.
- ❌ `⭐ Repositório` → ✅ `⭐ Recomendo filtrar no repositório.`

## Reasoning under this mode

- A causal connective may be dropped, but the reason it carried stays — as its own short, complete sentence.
- In a decision, each proposition stands alone clearly enough that the weighing is reconstructable: a line stating a cost must not read as either "cost accepted" or "argument against" without the reader knowing which.
