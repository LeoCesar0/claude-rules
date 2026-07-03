---
description: Decide between competing solution paths without anchoring on inflated effort estimates, phantom urgency, or a bias toward compromise. Use when about to present multiple viable options for the same problem, or when weighing a quick fix against a deeper fix.
allowed-tools: Read, Glob, Grep, Bash(git *)
effort: medium
---

# Decision

Counter Claude's defaults when a problem has multiple viable solutions. Output is conversational — no file written. The number and shape of options vary per problem; this skill does not assume any fixed structure.

## When this skill applies

- Presenting more than one viable option for the same problem
- Weighing a quick fix against a deeper fix
- Estimating effort in "days" or "weeks"
- Recommending a workaround "for now"

If only one viable path exists, this skill does not apply — proceed normally.

## Motivation (illustrative)

A recurring failure mode: Claude presents options like "the solid rewrite (2 weeks), a compromise (2 days), or a quick patch (half day)" and tends to recommend the compromise or the patch — partly because the estimates inherit human-developer norms (which inflate AI-assisted work), partly because Claude assumes urgency that may not exist (e.g., "users may be hitting this" when the feature isn't deployed). This skill exists to break those defaults. The 3-option shape is just one example — real decisions can be 2 options, 4 options, or peers with no fragility axis at all.

## Defaults to override

### 1. Effort estimates are wrong by default

Human-developer estimates do not apply to AI-assisted work. Rough re-anchor:

- "2 weeks" of human work → ~1–2 days with Claude
- "1 day" of human work → ~30–60 minutes with Claude
- "1 hour" → minutes

Re-derive per task — boilerplate compresses harder than novel architecture, and verification time compresses less than authoring time. Never quote a human-norm estimate without re-deriving it for the AI workflow.

### 2. Urgency is unverified by default

Before invoking time pressure ("users may be hitting this in production", "we need a quick fix to unblock"), verify:

- Is the affected code deployed? Check release tags, version files, deployment configs, recent merges to main/release branches.
- Has the feature shipped to real users? Ask the user — there is no telemetry inside the repo.
- Is there an actual deadline? Ask, do not infer.

If any of these are unverifiable, **do not use urgency as justification**. Say "urgency unconfirmed" explicitly.

### 3. Default to the correct path, not the compromise

The default recommendation is the option that best fits the problem — the right-sized, structurally sound solution. Do not gravitate to a middle option for its own sake, and do not gravitate to the fastest option without verified urgency.

"Correct" means **right-fit**, not maximalist:
- Don't rewrite code that already works to satisfy an aesthetic preference
- Don't propose extra abstraction for hypothetical future requirements
- Don't expand scope beyond the actual problem

A quick or partial fix is the correct answer only when:
- Verified deadline AND verified usage AND a follow-up cleanup is explicitly tracked
- The fuller path is genuinely disproportionate to the problem size

## Workflow

1. **State the options** — describe each viable path in plain language. Do not invent options to fill a slot; use however many real paths exist (often two).

2. **Re-derive estimates** — give an AI-adjusted estimate per option. Use the smallest realistic unit (minutes / hours / days). Avoid "weeks" unless scope is genuinely novel.

3. **Audit urgency claims** — for any option justified by speed, name the premise:
   - "This is faster — relevant only if [premise]"
   - Mark each premise verified or unverified

4. **Recommend the right-fit path** — explain why in one or two sentences. The recommendation is the path that fits the problem, not the most ambitious and not the fastest.

5. **Present via the decision protocol** — deliver the options using the walkthrough format in `~/.claude/rules/decision-protocol.md` (one point per message, template per point, AI-adjusted estimate, recommendation tagged with its *reason*). Do not use `AskUserQuestion`.

## What this skill is not

- Not a planner — does not produce a plan. After the decision, scope new specs via `/ahead:specs` or implement directly using plan mode.
- Not a reviewer — does not audit existing code quality.
- Not for one-path problems — if the answer is obvious, do not invoke.
- Not a template — does not impose a fixed number of options or a fragility axis.
