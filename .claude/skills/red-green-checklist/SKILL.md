---
name: red-green-checklist
description: Discipline gates for the Red and Green phases of TDD. Use in test-writer (Red) and implementer (Green) to confirm the discipline is intact before declaring a phase done.
---

# Red-Green checklist

Strict TDD: a test must fail for the right reason before any production code is written, and production code may only be written to make the failing test pass.

## Red phase (test-writer)

Before declaring red:

- [ ] Test file exists at the path the task specifies.
- [ ] Test file is in a test directory the project's test runner picks up.
- [ ] The test runs (no syntax error, no missing import, no unresolved fixture).
- [ ] The test fails.
- [ ] The failure reason is **missing production behavior**, not:
  - typo / syntax error
  - wrong import path
  - missing fixture
  - test asserting against itself
- [ ] The failure message would help a future debugger see what behavior is missing.
- [ ] No production code was modified to make the test "fail correctly."
- [ ] No existing tests now fail or are skipped.

If a test passes immediately on first run: stop. Either the test is wrong (it tests existing behavior) or the feature already exists. Report rather than "fix" the test to fail.

## Green phase (implementer)

Before declaring green:

- [ ] All tests for this task pass.
- [ ] Full test suite passes (no regressions in unrelated areas).
- [ ] No test was modified to make it pass. If a test seemed wrong, you stopped and reported instead.
- [ ] No test was skipped, xfailed, or disabled.
- [ ] Implementation is within the task's `## Files` and `## API surface` — no scope creep.
- [ ] No new tests were added (those belong to the next task or to test-writer).
- [ ] Code committed only after green confirmed.

## Hard prohibitions (both phases)

- Never `pytest.skip` / `xfail` / `it.skip` / equivalent to dodge a failing test.
- Never delete a test to "fix" it.
- Never add `if TESTING: return` shortcuts in production code.
- Never weaken an assertion to make a test pass — go fix the production code or report the test as wrong.

## Flaky tests

If a test passes sometimes and fails sometimes, that is a finding. Report it; don't retry-loop until green.
