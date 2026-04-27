---
name: behavioral-test-format
description: Given/When/Then template, naming, traceability, and coverage rules for behavioral test plans. Use in 01-test-plan.md, when verifying the plan, and when writing tests from it.
---

# Behavioral test format

## File structure

```markdown
# Test plan: <feature-name>

## TC-1: <short title>
- **Requirement**: R-<area>-<n>[, R-<area>-<m>]
- **Category**: happy | edge | error | security
- **Given**: <preconditions / system state / fixtures>
- **When**: <single action under test>
- **Then**: <observable outcome — what an external caller can see>
- **Notes**: <optional — fixture data shape, env constraints>
```

## Scope: business logic only

Behavioral test cases exist to pin down **business logic** — decisions the code makes that the product cares about. Write TCs for:

- Branching / conditional rules ("if X then Y else Z").
- Validation, authorization, and error semantics.
- Calculations, transformations, and state transitions.
- Domain invariants and security-relevant behavior.

Do NOT write TCs for things with no business decision in them:

- Framework / library behavior (e.g. "the ORM persists rows", "the HTTP server returns 404 for unknown routes").
- Pure configuration, DI wiring, env loading, module registration.
- Trivial scaffolding: getters/setters, DTO field passthrough, no-logic adapters.
- Build / infra concerns ("uses Postgres", "deployed to ECS").

If a `docs/product-spec/` requirement is non-business-logic, list it in the plan's **Out of scope (no behavioral test)** section with a one-line reason. The spec-verifier treats those as covered, not as gaps. Inventing a TC just to satisfy a coverage matrix is over-reach.

## Rules

- **Stable IDs**: `TC-1`, `TC-2`, … Never renumber. New cases append.
- **Traceability**: every TC lists ≥1 requirement ID. A TC that traces to nothing is over-reach.
- **Behavior, not internals**: Given/When/Then describe what an external caller observes. Don't name private methods, internal classes, or DB tables unless they are part of the observable contract (e.g. a public API response shape).
- **One When per TC**: if you need multiple When steps, that's two test cases.
- **Coverage minimum** per requirement:
  - One **happy** path.
  - One **edge** per input boundary (empty, max, min, type-edge cases that are reachable).
  - **Error** modes the requirement implies (invalid input, missing dependency, conflict states).
  - **Security** cases for any requirement touching auth, untrusted input, or sensitive data (authz checks, injection vectors, secret handling).
- **Concrete, not vague**: "Then the response is correct" is not a check. "Then the response status is 200 and `body.amount == 1500`" is.
- **No assertion-on-mocks**: don't write checks like "Then `service.foo()` was called once" — that's a unit-test design choice the implementer makes, not a behavioral guarantee.

## Categories

| Category | Trigger |
|---|---|
| `happy` | Normal valid input, normal state, success outcome |
| `edge` | Boundary values, unusual but valid inputs, race-adjacent timing |
| `error` | Invalid input, conflicting state, dependency failure |
| `security` | Authn/authz, untrusted input, secrets, injection vectors, rate limits |

## Anti-patterns

- TC depending on the order of other TCs running.
- TC whose Then is "and the system is in a good state" (unverifiable).
- TC that is really an assertion about the test's own setup.
- TC that re-states a unit test's internals as a "behavioral" check.
