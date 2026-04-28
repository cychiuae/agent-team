---
name: behavioral-test-format
description: Rules for writing failing behavioral tests in code (not markdown) before implementation. Covers business-logic scope, naming, traceability, Given/When/Then structure, and coverage. Use in test-writer when authoring tests under docs/plans/<slug>/.
---

# Behavioral test format

The test artifact is **real test code**, in the project's test framework, written **before** implementation. There is no markdown test plan. Each test is a behavioral specification: name it after the requirement it pins down, structure its body as Given / When / Then, and trace it to a stable `TC-#` ID.

For concrete code examples per language, see:

- `references/golang.md` — Go (`testing` + table-driven, `t.Run`)
- `references/python.md` — Python (`pytest`)
- `references/typescript.md` — TypeScript (`vitest` / `jest`)

## Scope: business logic only

Behavioral tests exist to pin down **business logic** — decisions the code makes that the product cares about. Write tests for:

- Branching / conditional rules ("if X then Y else Z").
- Validation, authorization, and error semantics.
- Calculations, transformations, and state transitions.
- Domain invariants and security-relevant behavior.

Do NOT write tests for things with no business decision in them:

- Framework / library behavior (e.g. "the ORM persists rows", "the HTTP server returns 404 for unknown routes").
- Pure configuration, DI wiring, env loading, module registration.
- Trivial scaffolding: getters/setters, DTO field passthrough, no-logic adapters.
- Build / infra concerns ("uses Postgres", "deployed to ECS").

If a `docs/product-spec/` requirement is non-business-logic, list it in the test file's **Out of scope** comment block (top of file) with a one-line reason. Inventing a test just to satisfy a coverage matrix is over-reach.

## Test case IDs

- Stable IDs: `TC-1`, `TC-2`, … Never renumber. New cases append.
- Every test traces to ≥1 requirement ID (`R-<area>-<n>`) — record both `TC-#` and requirement IDs in a comment or docstring directly above the test. A test that traces to nothing is over-reach.

## Test name

A reader of the name alone should know what behavior is being pinned. Use the project's idiomatic case (e.g. `Test_<Subject>_<Behavior>` in Go, `test_<subject>_<behavior>` in Python). One behavior per name. Avoid generic names like `test_happy_path`.

## Structure: Given / When / Then

Each test body has three blocks, separated by blank lines or comments:

- **Given** — preconditions, fixtures, system state.
- **When** — the **single** action under test.
- **Then** — observable assertions an external caller can see.

If a test needs multiple When steps, that's two tests. See per-language references for idiomatic phrasing (table-driven for Go, parametrize for pytest, `describe`/`it` for TS).

## Coverage minimum per requirement

- One **happy** path.
- One **edge** per input boundary that is reachable (empty, max, min, type-edge).
- **Error** modes the requirement implies (invalid input, missing dependency, conflict states).
- **Security** cases for any requirement touching auth, untrusted input, or sensitive data (authz checks, injection vectors, secret handling).

Tag each test's category in the trace comment: `happy | edge | error | security`.

## Rules

- **Behavior, not internals.** Given/When/Then describe what an external caller observes. Don't assert on private methods, internal classes, or DB tables unless they're part of the observable contract.
- **One When per test.**
- **Concrete assertions.** "response is correct" is not a check. `status == 200 && body.amount == 1500` is.
- **No assertion-on-mocks.** Don't assert "service.foo() was called once" — that's a unit-test design choice the implementer makes, not a behavioral guarantee.
- **Tests must run.** Even before implementation exists, the test file must compile / import / collect cleanly. Failing assertions are expected (red); errors at collection are not.
- **Never skip / xfail / disable** tests to dodge red. Red is the point.

## Categories

| Category | Trigger |
|---|---|
| `happy` | Normal valid input, normal state, success outcome |
| `edge` | Boundary values, unusual but valid inputs, race-adjacent timing |
| `error` | Invalid input, conflicting state, dependency failure |
| `security` | Authn/authz, untrusted input, secrets, injection vectors, rate limits |

## Anti-patterns

- Test depending on the order other tests run in.
- Test whose Then is "and the system is in a good state" (unverifiable).
- Test that is really an assertion about the test's own setup.
- Test that re-states a unit test's internals as a "behavioral" check.
- Weakening an assertion to make a test pass — fix the production code instead.
