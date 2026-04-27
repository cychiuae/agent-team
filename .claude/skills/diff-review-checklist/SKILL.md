---
name: diff-review-checklist
description: Categories, severity rubric, and finding format for code review of a feature diff. Use in code-reviewer to produce review.md.
---

# Diff review checklist

## Output structure

```markdown
# Code review: <feature-name>

## Summary
- blocker: N
- major:   N
- minor:   N
- nit:     N

## Findings

### blocker: <short title>
- **Where**: path/to/file.ext:LINE
- **What**: <the issue>
- **Why**:  <impact / risk>
- **Suggest**: <concrete fix>

### major: …
```

Group findings by severity, highest first.

## Severity rubric

- **blocker** — must fix before merge. Correctness bug, security flaw, data loss, broken contract, missing/wrong test for a stated requirement.
- **major** — should fix before merge. Significant design problem, hard-to-read code in a high-traffic path, missing edge-case handling, fragile test.
- **minor** — nice to fix. Small readability / naming, redundant code, awkward structure.
- **nit** — preference, cosmetic. Don't block merge.

## Categories to check

### Correctness
- Off-by-one, null/undefined handling, error paths actually reached.
- Race conditions, transaction boundaries, idempotency where required.
- Behavior matches `docs/` requirements (a passing test suite isn't proof — verify the tests cover what `docs/` says).
- Concurrency: shared state, locking, async ordering.

### Readability
- Names reveal intent; no abbreviations that obscure meaning.
- Functions do one thing at one level of abstraction.
- Reasonable length; deep nesting flagged.
- No dead code, no commented-out code, no debug prints.
- Comments explain WHY when non-obvious; don't restate WHAT.

### Security
- Untrusted input validated/escaped at boundaries.
- Injection vectors parameterized: SQL, shell, template, path traversal, header injection, log injection.
- Secrets: not logged, not committed, not in error messages.
- AuthN vs AuthZ: every protected path has an authorization check, not just authentication.
- Crypto: standard libraries with safe defaults; no homegrown primitives.
- File / path operations: canonicalize, prevent escape from intended directories.
- Deserialization of untrusted data: explicit allow-listed types only.

### Design
- SOLID violations that actually hurt — not theoretical.
- Coupling: would a routine change here force changes elsewhere unnecessarily?
- Premature abstraction or speculative generality (don't add for hypothetical futures).
- Error handling at the right layer (boundaries) — not sprinkled defensively everywhere.
- Layering / dependency direction respected.

### Tests
- Tests check observable behavior, not implementation internals.
- Failure messages would help a future debugger.
- No flakiness, no time / network / filesystem dependence without isolation.
- Edge cases and error modes covered, not just happy path.
- Tests don't duplicate each other; each one earns its keep.

## Out of scope

- Whitespace, line length, import ordering — formatter's job.
- Personal style preferences without a stated rationale.
- Refactors unrelated to the diff.

## Posture

Be specific. "This function is too long" is not a finding; "lines 40–110 mix HTTP parsing and business logic — extract the parse step into its own function" is.
