---
description: Systematic debugging for any bug, test failure, or unexpected behavior — root-cause investigation before any fix. Use before proposing fixes, especially under time pressure, after a failed fix attempt, or when a quick fix seems obvious.
---

# Systematic Debugging

Root cause before any fix. A fix proposed before the cause is understood is a symptom patch — a failure even when the error disappears.

## Phase 1 — Root cause
1. Read the complete error message and stack trace — file, line, error code
2. Reproduce consistently; not reproducible → gather more data, never guess
3. Check recent changes: git diff, commits, dependencies, config
4. Multi-component flows (API → service → DB; CI → build → deploy): log what enters and exits each boundary, run once, locate WHERE it breaks before theorizing why
5. Error deep in the call stack: trace the bad value backward, caller by caller, to its origin; fix at the source, never where the error surfaces

## Phase 2 — Pattern
- Find similar working code in the same codebase; list every difference to the broken code, however small
- Following a reference implementation: read it completely before adapting

## Phase 3 — Hypothesis
- State one specific hypothesis: "X is the root cause because Y"
- Test it with the smallest possible change — one variable at a time
- Refuted → new hypothesis; never stack another fix on top of a failed one
- Something not understood → say so and keep investigating; no fixes from partial understanding

## Phase 4 — Fix
- Write the failing test that reproduces the bug first — Bug Fix Workflow in think-ahead.md; red-green mechanics in the ahead:tdd skill
- One fix, at the root cause; no bundled refactoring or "while I'm here" improvements
- Verify: reproduction test passes and no other test broke

## Rule of three
After 3 failed fix attempts, stop — do not attempt #4. Each fix revealing a new problem elsewhere, or demanding ever-larger refactors, indicates an architectural problem: present it to the user as an architecture question, not another patch.

## Red flags — stop and return to Phase 1
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- Changing several things at once
- Proposing fixes before tracing the data flow
- "One more attempt" after 2+ failures

## No root cause found
When investigation truly ends at environmental, timing, or external causes: document what was ruled out, implement handling (retry, timeout, clear error), add logging for the next occurrence. Most "no root cause" conclusions are incomplete investigation.

## Techniques

### Backward tracing
- From the symptom, ask "what called this, with what value?" repeatedly until the origin of the bad value
- When manual tracing stalls, instrument: log the suspect value plus `new Error().stack` immediately before the failing operation; in tests use `console.error` (loggers may be suppressed)
- Test pollution of unknown origin: bisect — run tests one by one until the polluter appears

### Defense in depth
- After fixing at the source, add validation at every layer the bad data passed through: entry point (reject invalid input), business logic (assert preconditions), environment guard (refuse dangerous operations in the wrong context, e.g. writes outside tmpdir during tests), debug logging before the dangerous operation
- Different layers catch bypasses the others miss: alternate code paths, mocks, platform edge cases

### Condition-based waiting
- Flaky test sleeping an arbitrary duration → replace with polling for the actual condition (event present, state reached, file exists), with a timeout and descriptive error
- Poll ~every 10ms; read fresh state inside the loop, never cache it before
- An arbitrary delay is valid only when timing itself is the behavior under test (debounce, throttle) — justified in a comment
