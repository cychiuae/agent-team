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
3. Delegate to `docs-writer` with the resolved answers → updates `docs/product-spec/`. **CHECKPOINT 2**: human confirms `docs/product-spec/` accurately reflects requirements. Update STATUS.

### Phase 2 — Plan

1. Delegate to `test-planner` → writes `01-test-plan.md`.
2. Delegate to `spec-verifier` (independent) → writes `02-spec-verification.md`. If verdict = fail, loop back to step 1 (or to `docs-writer` if `docs/product-spec/` is the gap). Do not proceed on a fail verdict.
3. Delegate to `impl-planner` → writes `task-1.md` … `task-N.md`. Update the task table in STATUS.
4. Delegate to `plan-verifier` (independent) → writes `03-plan-verification.md`. If verdict = fail, loop back to step 3 (or to `test-planner` / `docs-writer` if the gap is upstream). Do not proceed on a fail verdict.

### Phase 3 — Implementation

For each task in order:

1. Delegate to `test-writer` with the path to `task-N.md`. It writes failing tests, confirms red. STATUS: task-N → red.
2. Delegate to `implementer` with the path to `task-N.md`. It writes code, confirms green, commits using the task's commit message verbatim, returns SHA. STATUS: task-N → committed (with SHA).
3. Move to task-N+1.

If any subagent reports a problem that requires plan changes, stop the loop and re-delegate to `impl-planner` for amendments.

### Phase 4 — Verify

1. Delegate to `verifier` → writes `verification.md`, returns verdict.
2. On **fail**: delegate to `impl-planner` for amendment tasks (`task-N+1.md` …, continuing the existing numbering — never renumber). Loop back to Phase 3 for the new tasks. Then re-verify (`## Attempt N` appended to `verification.md`). Repeat until pass.
3. Update STATUS verification-loops table each attempt.

### Phase 5 — Code review

1. Delegate to `code-reviewer` → reads full `git diff <base>..HEAD` for the feature, writes `review.md`.
2. Surface findings (especially blockers/majors) to the human.
3. If blockers exist: delegate to `impl-planner` for fix tasks → back to Phase 3. Then re-review.

### Phase 6 — Human verification

**CHECKPOINT 3**: human accepts the change. Update STATUS phase → done.

## STATUS.md update protocol

After every subagent return, update STATUS.md with:
- `Last updated` timestamp
- Phase / current task
- Task table state changes
- Checkpoint completions
- Verification attempt rows
