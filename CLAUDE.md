# CLAUDE.md

Guidance for Claude Code working in this repository.

## Core workflow: plan → execute → verify (loop)

Every task — code, config, docs, infra — follows this loop:

1. **Plan.** State the goal, the change set, and how it will be verified. For non-trivial work, write the plan to `docs/plans/<topic>.md` before touching anything else.
2. **Execute.** Make the smallest change that fulfills the plan.
3. **Verify.** Run the checks defined in the plan (tests, type checks, manual reproduction, doc review). Verification must be objective — "looks right" is not verification.
4. **Loop on failure.** If verify fails, return to step 1 (re-plan if the failure reveals a wrong assumption) or step 2 (fix the implementation). Do not declare a task done until verify passes.

Never skip verify. Never claim success without running it.

## Docs are the single source of truth

**Before any code change or implementation, update the docs first.** Docs in `docs/` describe the intended behavior of the system; code is a derivative artifact that must conform to them.

- New feature → update/add the relevant doc in `docs/` describing the behavior, then plan, then implement.
- Behavior change → update the doc to reflect the new behavior, then change the code.
- Bug fix → if the doc and code disagree, decide which is correct. Update the wrong one first, then reconcile the other.
- Refactor with no behavior change → docs need no update; note this explicitly in the plan.

If you find yourself writing code before the doc is updated, stop and update the doc first. A PR that changes behavior without a corresponding doc update is incomplete.

## Repository layout

- `docs/` — authoritative specifications and plans. `docs/plans/` holds per-task plans.
- `.claude/` — Claude Code configuration (agents, commands, skills, settings).
- `README.md` — entry point for human contributors.

## Conventions

- Keep plans, docs, and commits small and focused. One concern per change.
- Commit messages follow Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`, `test:`).
- Do not introduce abstractions, error handling, or features beyond what the current task requires.
