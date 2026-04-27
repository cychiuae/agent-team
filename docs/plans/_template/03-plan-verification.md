# Plan verification: <feature-name>

Independent check that the task plan (`task-1.md` … `task-N.md`), when implemented, will fulfill `docs/product-spec/` requirements and the TCs in `01-test-plan.md`.

## Requirement → Task matrix
| Requirement | Fulfilled by | Notes |
|-------------|--------------|-------|
| R-<area>-<n> | task-1, task-3 |       |
| R-<area>-<m> | task-2         |       |

## TC → Task matrix
| TC  | Made green by | Notes |
|-----|---------------|-------|
| TC-1 | task-1       |       |
| TC-2 | task-2       |       |

## Gaps
Requirements or TCs not addressed by any task.

- …

## Over-reach
Tasks (or parts of tasks) not traceable to any requirement or TC.

- …

## Feasibility risks
Pseudocode/API choices that look unlikely to fulfill the requirement.

- …

## Ordering issues
Dependency chains that break the "suite green at every boundary" rule.

- …

## Verdict
**pass** | **fail**

Reasons: …
