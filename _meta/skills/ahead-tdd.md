---
description: Red-green-refactor test-first procedure. Mandatory when fixing a bug (Bug Fix Workflow); for features, use when tests were requested or agreed. Covers verify-RED discipline, minimal GREEN, and mock anti-patterns.
---

# Test-Driven Development

Scope: bug fixes always start test-first, per the Bug Fix Workflow in think-ahead.md. For features, apply this cycle when tests were requested or agreed via Testing Mindset. The E2E/eval approval gates in think-ahead.md apply unchanged.

## Cycle

### RED — write a failing test
- One behavior per test; the name describes the behavior
- Test real code; mock only when unavoidable

### Verify RED — watch it fail (never skip)
- Run the test and read the failure
- It must FAIL, not error — an exception or typo is not a valid RED; fix and re-run
- It must fail for the expected reason: the feature is missing or the bug is present
- Passes immediately → it exercises existing behavior and proves nothing; rewrite it

### GREEN — minimal code
- Simplest code that passes the test; no options, parameters, or features beyond it

### Verify GREEN — watch it pass
- Run again: this test passes, all other tests still pass, output clean
- Failing → fix the code, not the test

### REFACTOR
- Only after green: remove duplication, improve names, extract helpers
- No new behavior; tests stay green

## Test quality
- "and" in a test name → split into two tests
- The test demonstrates the desired API — hard to write means hard to use; simplify the interface, not the test
- Huge setup or mocking everything → coupling problem; consider dependency injection or an integration test

## Mock anti-patterns
1. Never assert on a mock's existence or behavior — test what the code does, not what the mock does
2. Never add test-only methods to production classes — cleanup belongs in test utilities
3. Never mock a method whose side effects the test depends on — mock at the lower level (the slow or external operation); if unsure what the test depends on, run it with the real implementation first and observe
4. Mock the complete real data structure, not only the fields the test touches — omitted fields fail silently downstream

Red flags: assertions on `*-mock` test IDs; mock setup longer than the test logic; "mock it to be safe"; test breaks when the mock is removed.
