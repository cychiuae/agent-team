---
name: red-green-checklist
description: Discipline rules for the test-writer (writes failing tests before implementation) and the implementer (writes production code only, no test runs per task). The /ship flow runs tests once after all implementation tasks land. Use this skill to confirm those boundaries are respected.
---

# Test-writer / implementer discipline checklist

The /ship flow is **red-green at the feature level**, not per task:

1. **Phase 3 — `test-writer`** writes the full set of failing behavioral tests in code, before any implementation lands. Compiling/collecting must succeed; assertions are expected to fail. This is the **red** state.
2. **Phase 4 — `implementer`** lands production code one task per commit. The test suite is **not** run per task.
3. **Phase 5 — `verifier`** runs the full suite once. **Green** = pass.

This skill captures the discipline gates each role must respect.

## Phase 3 — test-writer (single pass, before implementation)

Before declaring tests written:

- [ ] Tests are written in **code** (the project's test framework), not as a markdown plan.
- [ ] Every behavioral test traces to a stable `TC-#` and ≥1 requirement ID (`R-<area>-<n>`) via comment / docstring / `it`-name.
- [ ] Tests exercise **observable behavior**, not internals.
- [ ] Tests cover **business logic only** — no tests for framework behavior, DI wiring, env loading, or trivial scaffolding.
- [ ] Every test file compiles / imports / collects cleanly. If a production symbol doesn't exist yet, add the smallest possible stub in the production module so the test fails on the **assertion**, not at build/collection.
- [ ] Running the full suite produces **red** (failing assertions) — not errors at collection.
- [ ] No tests are skipped, xfailed, or otherwise disabled.
- [ ] Only test files (and the minimal stubs needed to make them compile) were staged. No `git add -A` / `git add .`.
- [ ] One commit was made with a `test:` Conventional Commits subject.

## Phase 4 — implementer (code only, per task)

Before declaring a task done:

- [ ] All production code is within the task's `## Files` and `## API surface` — no scope creep.
- [ ] No test files were created or modified.
- [ ] The test suite was **not** run for this task (per-task test runs are deliberately not part of the workflow — the verifier runs it once at the end).
- [ ] The commit builds / type-checks / lints cleanly on its own.
- [ ] Only files in the task's `## Files` were staged. No `git add -A` / `git add .`.
- [ ] The commit message is the verbatim message from the task's `## Commit message` section.
- [ ] No `--amend`, `--no-verify`, `--no-gpg-sign`, no force-push.

## Hard prohibitions (both roles)

- Never `pytest.skip` / `xfail` / `it.skip` / equivalent to dodge a failing test.
- Never delete a test to "fix" it.
- Never add `if TESTING: return` shortcuts in production code.
- Never weaken an assertion to make a test pass — fix the production code via an amendment task, or report the test as wrong.
- The implementer never writes tests; the test-writer never writes production code (beyond the minimal stubs needed for tests to compile).

## Flaky tests

If a test passes sometimes and fails sometimes, that is a finding. Report it; don't retry-loop until green.
