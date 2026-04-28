---
name: task-file-format
description: Canonical task-N.md template and rules for sizing, dependencies, API surface, pseudocode, and Conventional Commits messages. Use in impl-planner (writing) and in implementer (reading). Tasks describe production code only — failing tests are authored separately by test-writer before implementation begins.
---

# task-N.md format

Each `task-N.md` describes exactly **one** git-commit-sized unit of **production-code** work. Tests are not part of any task. Failing behavioral tests for the whole feature are written once, by `test-writer`, **before** any implementation task lands. The verifier runs the suite once, after every task is committed.

## Template

````markdown
# Task N: <short title>

## Scope
One paragraph: what & why. This task is a single git commit of production code.

## Requirements
- R-<area>-<n>[, R-<area>-<m>]

## Test cases addressed
- TC-<i>[, TC-<j>]
(Traceability only — these TCs already exist as failing tests written by test-writer.
This task does not write or run tests.)

## Files
- path/to/source.ext  (new | modify | delete)
(Production-code paths only. No test paths.)

## API surface
Concrete signatures of new or changed functions, classes, endpoints, types.
Use the language's natural signature form; be concrete enough that two
implementers would produce compatible interfaces.

## Pseudocode
Per function in the API surface, step-level pseudocode — enough that the
implementer writes real code without redesigning the algorithm.

function foo(x):
    validate x
    look up y from store
    if y is empty: return Err(NOT_FOUND)
    compute z from x and y
    return Ok(z)

## Dependencies
- Depends on: task-<k>  (or "none")
- Blocks:     task-<l>  (or "none")

## Risks / notes
Anything tricky the implementer should know — concurrency, ordering,
fixtures, data migrations, env vars, feature flags, etc.

## Commit message
```
<type>(<scope>): <subject under 72 chars, imperative mood>

<body — wrap at 72 cols. Explain the WHY of this change.
The diff already shows the WHAT.>
```
````

## Sizing rules

- **One concern per task.** If you describe it with "and", split it.
- **< ~300 lines diff** is a healthy ceiling. Larger only with explicit justification under `## Risks / notes`.
- **Independently committable**: the commit must build / type-check / lint cleanly on its own. The test suite is **not** run per task — the verifier runs it once after every task is committed — so "suite green" is not a per-task gate.
- **Forward dependencies only**: `Depends on:` may reference earlier tasks; `Blocks:` may reference later tasks. No cycles.

## API surface rules

- Be concrete. `def charge(order_id: str, amount: Money) -> ChargeResult` is fine; "a charge function" is not.
- Specify types where the language has them.
- For endpoints: method, path, request shape, response shape, status codes.
- For data shapes: field names and types, not prose descriptions.
- Match the API surface that `test-writer`'s tests expect — if a test imports `service.Apply(...)`, the task's surface must define it. Tests are the contract.

## Pseudocode rules

- Step-level, not statement-level. Implementer should not have to redesign control flow but is free to choose names, helpers, and idioms.
- Cover the unhappy paths (validation failures, missing data, conflicts).
- Don't write the language's actual syntax — that's the implementer's job.

## Commit message rules

- **Conventional Commits** types: `feat | fix | refactor | docs | test | chore | perf | build | ci | style`.
- Subject in imperative mood ("add X", not "added X" or "adds X").
- Subject ≤ 72 chars. Body wraps at 72 cols.
- Body answers "why now / why this approach", not "what does the diff contain".
- Reference requirement IDs in the body when they help context (`Implements R-billing-3.`).
- No tool / agent attribution lines unless the project convention requires them.

## Amendment tasks

When `impl-planner` produces tasks in response to a verification failure or review blocker:

- Continue the existing numbering (`task-N+1`, `task-N+2`, …). Never renumber prior tasks.
- Under `## Scope`, name the gap or finding being addressed (e.g. "Addresses verification.md attempt-1 gap: R-billing-7 not exercised").
