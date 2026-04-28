# Verification: <feature-name>

## Attempt 1 — <YYYY-MM-DD HH:MM>

### Test suite output
```
<full output of the suite run>
```

### Coverage matrix
| Requirement   | Test case | Actual test                  | Result | Evidence |
|---------------|-----------|------------------------------|--------|----------|
| R-<area>-<n>  | TC-1      | <test name / file:line>      | pass   | …        |

### Gaps
Requirements with no passing evidence in this run.

- …

### Dead test cases
TC-#s referenced in trace blocks that point at no actual requirement, or duplicates.

- …

### Verdict
**pass** | **fail**

(On fail, the gap list above must be specific enough that `impl-planner`
can produce amendment tasks without re-investigating.)

---

## Attempt 2 — <YYYY-MM-DD HH:MM>
(append after each amendment loop — never overwrite prior attempts)
