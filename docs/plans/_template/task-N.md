# Task N: <short title>

## Scope
One paragraph: what & why. This task is exactly one git commit.

## Requirements
- R-<area>-<n>

## Test cases addressed
- TC-<i>
(Traceability only — TCs already exist as failing tests committed by test-writer.
This task does not write or run tests.)

## Files
- path/to/source.ext  (new | modify | delete)
(Production-code paths only. No test paths.)

## API surface
Concrete signatures of new or changed functions, classes, endpoints, types.

## Pseudocode
Per function in the API surface, step-level pseudocode.

```
function foo(x):
    validate x
    look up y from store
    if y is empty: return Err(NOT_FOUND)
    compute z from x and y
    return Ok(z)
```

## Dependencies
- Depends on: task-<k>  (or "none")
- Blocks:     task-<l>  (or "none")

## Risks / notes
Anything tricky the implementer should know.

## Commit message
```
<type>(<scope>): <subject under 72 chars, imperative mood>

<body — wrap at 72 cols. Explain WHY, not WHAT.
Reference requirement IDs when they help context.>
```
