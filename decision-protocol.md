# Decision Protocol

How to present any decision to the user. Replaces `AskUserQuestion` for
decisions: every choice among paths or options is delivered as text in this
format. Applies always, down to a single decision point.

## Scope

- Applies to every decision among options or paths — architecture forks,
  quick-fix vs. root-cause, competing approaches, any "which way do we go".
- **Do not use `AskUserQuestion` to present a decision.** The user answers in
  free text.
- Open-ended clarification with no bounded set of paths (e.g. "what should this
  route return?") stays a plain prose question — this protocol does not apply.
- Render all field labels in the conversation's language.

## Icons

Prefix each structural element with its fixed icon, every time. Do not reuse
any other glyph for these slots, and do not add icons to sub-lines (e.g.
examples). Never reuse the `current-work` status glyphs (`🔄 ⬜ ⏸️ ✅ ▶`).

| Element | Icon |
|---------|------|
| Agenda (point list + live updates) | 🧭 |
| Topic header (with `N/M`) | 🔹 |
| What it is | 📌 |
| Before vs. after (field) | 🔀 |
| Before state (inside 🔀) | 🔴 |
| After state (inside 🔀) | 🟢 |
| Recommendation | ⭐ |
| Not recommended | ⚠️ |
| Question | ❓ |

## Phase 1 — Agenda (only when 2+ points)

- Before discussing any point, list every open point under a 🧭 line, one line
  each, standardized and brief.
- Skip this phase entirely when there is a single point.

## Phase 2 — Walkthrough

- Discuss one point per message. Never bundle multiple points into one message.
- Head each point with a counter `N/M` when there are 2+ points. Omit the
  counter for a single point.
- Each point uses this template, in order (icon per field from the legend):
  1. 🔹 **Title** (+ counter when 2+ points)
  2. 📌 **What it is** + a concrete example — always
  3. 🔀 **Before vs. after** — put each state on its own line, `🔴 <before>` and
     `🟢 <after>`, with no connecting arrow; only when the change alters existing
     behavior or values, else omit
  4. ⭐ **Recommendation** + why — always; the recommendation is the right-fit
     path (see Recommendation discipline)
  5. ⚠️ **Not recommended** + why — only when a real trap exists: a path that
     causes a bug, regression, data loss, or contradicts a decision already
     made. Omit the field entirely when no path is worth warning against —
     never fabricate one
  6. ❓ **Question** — the actual ask

## Agenda is live

- The agenda may change mid-walkthrough: drop a point an earlier answer made
  moot, or add a point that surfaced.
- **Announce every change under a 🧭 line and restate the counter** when it
  happens — e.g. "your answer drops point 4; agenda is now 4 points, going to
  3/4". Never add or remove a point silently.

## Recommendation discipline

- The recommendation is the right-fit path — not the compromise for its own
  sake, not the fastest without verified urgency.
- Estimate effort AI-adjusted, never at human-developer norms.
- Do not invoke urgency ("users may be hitting this") unless deployment and
  real usage are verified; otherwise state "urgency unconfirmed".
- For the detailed effort re-anchoring and urgency checklist, see the
  `ahead:decision` skill.
