---
name: diff-review-checklist
description: Code-quality checklist and finding format for the brief code review at the end of /ship. Focused on readability, naming, simplicity, and design — not full security/correctness audit. Use in code-reviewer to produce review.md.
---

# Diff review checklist — code quality

Scope of this review is **code quality**: is the code clear, simple, and well-shaped for the next reader to maintain. Correctness has already been checked by the verifier (full behavioral test suite green). Don't re-do verification work here.

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

- **blocker** — must fix before merge. Code is unreadable or actively misleading; an obvious simpler form exists; a name lies about what the code does.
- **major** — should fix before merge. Significant clarity / structure problem in code a maintainer will touch often.
- **minor** — nice to fix. Small readability / naming, redundant code, awkward structure.
- **nit** — preference, cosmetic. Don't block merge.

## Categories to check

### Readability
- Names reveal intent. No abbreviations that obscure meaning. No names that lie about behavior.
- Functions do one thing at one level of abstraction.
- Reasonable length; deep nesting flagged.
- No dead code, no commented-out code, no debug prints, no TODOs left in.
- Comments explain WHY when non-obvious; don't restate WHAT.

### Simplicity
- The simplest form that satisfies the tests. Flag clever code that a junior reader can't follow.
- No premature abstraction or speculative generality (no abstractions added "for the future").
- No defensive error handling for cases that can't happen.
- No backward-compat shims or feature flags that aren't needed.
- Three-line repetition is fine; a wrong abstraction is worse.

### Structure
- Coupling: would a routine change here force changes elsewhere unnecessarily?
- Layering / dependency direction respected (e.g. domain doesn't import HTTP / DB drivers directly if the project layers that way).
- Error handling sits at the right boundary — not sprinkled defensively at every call site.
- Public surface area is the minimum needed. Don't export internals.

## Out of scope

- Whitespace, line length, import ordering — formatter's job.
- Personal style preferences without a stated rationale.
- Refactors unrelated to the diff.
- Security audits — only flag obvious code-quality-adjacent issues (e.g. a SQL string concatenation that's also unreadable). Deep security review is a separate pass.
- Test coverage / behavior correctness — verifier owns those. Only flag tests for **code quality** (unclear test names, copy-pasted setup that should be a fixture, etc.).

## Posture

Be specific. "This function is too long" is not a finding; "lines 40–110 mix HTTP parsing and business logic — extract the parse step into its own function" is.

If you have nothing severity ≥ minor to say, the review is short and that's correct. Do not invent findings to fill space.
