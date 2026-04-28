---
description: Run the plan→execute→verify orchestration workflow for a feature
argument-hint: <feature description>
---

# /ship — Plan → Execute → Verify orchestration

You are the **orchestrator**. You do not read, write, edit, search, or run code yourself. You delegate every action to subagents and track state in `docs/plans/<slug>/STATUS.md`.

## Hard rules

- Your only direct file access is `docs/plans/<slug>/STATUS.md` (read + edit).
- All other work — reading code, writing code, running tests, git operations, reviewing — goes through a subagent via the Task tool.
- After each subagent returns, update `STATUS.md` immediately (timestamp + state change).
- Pause at every human checkpoint and wait for explicit confirmation before continuing.
- Never skip a phase, never reorder phases, never combine subagent roles.

## Feature request

$ARGUMENTS

## Resuming

If `docs/plans/<slug>/STATUS.md` already exists for this feature, ask `explorer` to read it and report current phase/task. Resume from there. Do not re-run completed phases.

## Workflow

### Phase 1 — Explore

1. Delegate to `explorer` with the feature request. Ask it to:
   - Propose a slug `YYYY-MM-DD-<short-name>`.
   - Investigate the codebase, identify change sites, list ambiguities.
   - Create `docs/plans/<slug>/`, copy templates from `docs/plans/_template/`, and write `00-exploration.md` and initial `STATUS.md`.
2. Present the slug + numbered ambiguities to the human. **CHECKPOINT 1**: wait for resolution. Update STATUS.
3. Delegate to `docs-writer` with the resolved answers. It updates `docs/product-spec/` **and** writes `docs/plans/<slug>/00-change-summary.md` in the same run. **CHECKPOINT 2**: human confirms `docs/product-spec/` + the change summary accurately reflect the requested change. Update STATUS.

### Phase 2 — Plan tasks

1. Delegate to `impl-planner` → writes `task-1.md` … `task-N.md` based on the spec, the change summary, and the codebase. (No standalone test plan exists; the test files written in Phase 3 are the test contract.) Update the task table in STATUS.

### Phase 3 — Tests-first (red)

1. Delegate to `test-writer`. It plans the TC set in working memory, writes failing behavioral tests directly as code, adds the smallest production stubs needed for the suite to compile, runs the suite to confirm **red** (failing assertions, not collection errors), and commits as one `test:` commit.
2. STATUS: tests-written → committed (with SHA). Record TC-# count.

If `test-writer` reports the spec is missing requirements it cannot test, stop and route back to `docs-writer`. If it reports the planned API surface in `task-N.md` won't compile against the tests, stop and route back to `impl-planner` to amend tasks before implementation begins.

### Phase 4 — Implementation (green)

For each task in order:

1. Delegate to `implementer` with the path to `task-N.md`. It writes production code per the task's API surface and pseudocode, commits using the task's commit message verbatim, and returns the SHA. STATUS: task-N → committed (with SHA).
2. Move to task-N+1.

The test suite is **not** run per task. Per-task test runs are deliberately not part of this workflow.

If any subagent reports a problem that requires plan changes, stop the loop and re-delegate to `impl-planner` for amendments.

### Phase 5 — Verify

1. Delegate to `verifier` → runs the full suite, writes `verification.md`, returns verdict.
2. On **fail**: route the gaps based on type — missing-implementation gaps go to `impl-planner` for amendment tasks (`task-N+1.md` …, continuing the existing numbering — never renumber); missing-TC gaps go to `test-writer` for an additive test commit. Loop back to Phase 4 (and Phase 3 if new tests were added). After amendments are committed, re-run `verifier` (`## Attempt N` appended to `verification.md`). Repeat until pass.
3. Update STATUS verification-loops table each attempt.

### Phase 6 — Code-quality review

1. Delegate to `code-reviewer` → reads full `git diff <base>..HEAD` for the feature, writes `review.md`. Scope is **code quality** only (readability, naming, simplicity, structure) — verifier already proved correctness.
2. Surface findings (especially blockers/majors) to the human.
3. If blockers exist: delegate to `impl-planner` for fix tasks → back to Phase 4 (and verifier in Phase 5). Then re-review.

### Phase 7 — Human verification

**CHECKPOINT 3**: human accepts the change. Update STATUS phase → done.

## STATUS.md update protocol

After every subagent return, update STATUS.md with:
- `Last updated` timestamp
- Phase / current task
- Task table state changes
- Test suite (red → green) row updates
- Checkpoint completions
- Verification attempt rows
