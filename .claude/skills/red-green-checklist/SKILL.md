---
name: red-green-checklist
description: Discipline rules for the implementer (Phase 3, code-only) and test-writer (Phase 4, single test pass). The /ship flow no longer runs per-task TDD — tests are planned in Phase 2, implementation tasks land in Phase 3 without tests, and tests are written and run once in Phase 4. Use this skill to confirm those boundaries are respected.
---

# Implementer / test-writer discipline checklist

The /ship flow does **not** practice per-task red-green TDD. Tests for the feature are planned in `01-test-plan.md` (Phase 2), implementation tasks are committed without tests (Phase 3), and tests are written and run once after all tasks are done (Phase 4).

This skill captures the discipline gates each role must respect.

## Phase 3 — implementer (code only)

Before declaring a task done:

- [ ] All production code is within the task's `## Files` and `## API surface` — no scope creep.
- [ ] No test files were created or modified.
- [ ] The test suite was **not** run for this task (per-task test runs are deliberately not part of the workflow).
- [ ] The commit builds / type-checks / lints cleanly on its own.
- [ ] Only files in the task's `## Files` were staged. No `git add -A` / `git add .`.
- [ ] The commit message is the verbatim message from the task's `## Commit message` section.
- [ ] No `--amend`, `--no-verify`, `--no-gpg-sign`, no force-push.

## Phase 4 — test-writer (single pass)

Before declaring tests written:

- [ ] Every in-scope TC in `01-test-plan.md` has a corresponding test, and each test traces to its TC-# (test name, docstring, or comment).
- [ ] Tests exercise observable behavior, not internals.
- [ ] Every test executes (no syntax / import / missing-fixture errors). Whether each one *passes* is for the verifier to determine — not a gate here.
- [ ] No production code was modified, even if a test reveals what looks like a bug. Such findings are reported to the orchestrator instead.
- [ ] No tests were skipped, xfailed, or otherwise disabled.
- [ ] No assertions were weakened to make a test pass.
- [ ] Only test files were staged. No `git add -A` / `git add .`.
- [ ] One commit was made containing only the test files, with a `test:` Conventional Commits subject.

## Hard prohibitions (both roles)

- Never `pytest.skip` / `xfail` / `it.skip` / equivalent to dodge a failing test.
- Never delete a test to "fix" it.
- Never add `if TESTING: return` shortcuts in production code.
- Never weaken an assertion to make a test pass — fix the production code via an amendment task, or report the test as wrong.
- The implementer never writes tests; the test-writer never writes production code.

## Flaky tests

If a test passes sometimes and fails sometimes, that is a finding. Report it; don't retry-loop until green.
