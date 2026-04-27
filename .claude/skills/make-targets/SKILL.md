---
name: make-targets
description: Implement the standard Makefile target contract for all projects including setup, lint, format, test, audit, and deploy targets. Use when creating or updating a Makefile, setting up a new project's build system, configuring linting with cognitive complexity checks, or when the user asks about make targets, project build conventions, or CI pipeline commands.
---

# Make Targets Contract

All projects must implement this standard set of make targets.

## Required targets

```makefile
make secret   # decrypt secrets, set up credentials
make setup    # install dependencies, initial project setup
make audit    # check dependencies for security vulnerabilities
make lint     # lint + type check + cognitive complexity check
make format   # auto-format code per .editorconfig
make test     # run tests (end-to-end / integration)
make ci       # runs audit + lint + format + test (for CI pipeline)
make deploy   # deploy the application
```

Push to the main branch must trigger: `audit`, `lint`, `format`, `test`.

## Secrets

Copy https://github.com/oursky/devsecops-secret into the repo. There must be no static passwords, even in development.

## Lint

Includes: basic linting, type checking, and **cognitive complexity check** (mandatory, default threshold: 10).

If no cognitive complexity tool exists for the language, use cyclomatic complexity instead.

| Language | Tool |
|----------|------|
| Go | [gocognit](https://github.com/uudashr/gocognit) |
| .NET (C#) | SonarAnalyzer.CSharp rule S3776 |
| Dart | dart_cognitive_complexity |
| Python | flake8 `max_cognitive_complexity` plugin |
| JavaScript/TypeScript | eslint-plugin-sonarjs |

## Format

Auto-formats code according to `.editorconfig`. Should be non-destructive and idempotent.

## Test

- End-to-end and unit tests.
- No fixed coverage percentage required.
- A normal-flow test case must accompany every PR.
