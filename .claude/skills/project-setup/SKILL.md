---
name: project-setup
description: Scaffold a new project from zero with the standard baseline — hello-world entry point, linter, formatter, test runner, structured logging, i18n (for frontend), optional Sentry, README with prerequisites and quickstart, and a Makefile interface for local dev and CI. Use this skill whenever the user is starting a new project, bootstrapping a repo, asking to "set up" or "initialize" a codebase, creating a new service or app, or asking what tooling a fresh project should have — even when they don't list every component. Trigger for greenfield work in any language (Node/TypeScript, Python, Go, etc.); pick the right stack-specific tooling from the references.
---

# Project setup

Use this skill when bootstrapping a new project. The goal is a repo that a new developer can clone and run in one command (`make setup && make run`), with quality gates (lint, format, test) wired up the same way locally and in CI.

## Workflow

1. **Identify the stack and project type.** Ask the user (or infer from context) the language/runtime, whether it's a frontend app, backend service, library, or CLI, and whether it ships to production users (affects Sentry decision). Don't proceed on assumptions if it's ambiguous — one clarifying question is cheaper than redoing the scaffold.

2. **Pin tool versions up front.** Before generating files, decide concrete versions (Node 20.x, Python 3.12, Go 1.22, etc.) and capture them in a version file the ecosystem already supports (`.nvmrc`, `.tool-versions`, `.python-version`, `go.mod`). The README's prerequisites section must match these exactly — drift between README and version files is a top onboarding paper-cut.

3. **Invoke the relevant language preferred-stack skill.** Each language has its own sibling skill that encodes the opinionated tooling defaults so you don't have to relitigate "ruff vs flake8" each time. Invoke the matching skill and follow its conventions:
   - Python → `python-prefer-stack`
   - Go → `golang-prefer-stack`
   - Other languages aren't covered yet. If the user is starting a project in an uncovered language, pick the most boring, widely-used tools in that ecosystem, note in the README why they were chosen, and consider creating a new `<lang>-prefer-stack` skill under `.claude/skills/languages/<lang>/` so the next project benefits.

4. **Scaffold in this order.** Each step builds on the previous, and stopping partway still leaves a working repo:
   1. Hello-world entry point that actually runs (`make run` should print something).
   2. Formatter + linter configured and passing on the hello-world.
   3. Test runner with one trivial passing test.
   4. Structured logging (see "Logging" below).
   5. i18n if it's a frontend project (see "i18n").
   6. Sentry if it ships to users (see "Sentry").
   7. README with prerequisites + quickstart.
   8. Makefile — defer to the `make-targets` skill for the target contract; do not redefine it here.

5. **Verify end-to-end.** Run `make setup`, `make lint`, `make format`, `make test`, `make run` yourself before reporting done. A scaffold that doesn't actually pass its own gates is worse than no scaffold.

## Logging (structured)

Structured logging means every log line is a key-value record (usually JSON in production), not a printf string. This is non-negotiable for any service that will be deployed — grep-on-prose doesn't scale, and downstream tooling (Datadog, Loki, CloudWatch Insights) needs structured fields.

- **Node/TS**: `pino` (fast, JSON by default). For frontend, log to console with structured fields and forward via Sentry breadcrumbs.
- **Python**: `structlog` configured to emit JSON in production and pretty key=value locally. Avoid the stdlib `logging` module's default formatter — it's prose-shaped.
- **Go**: `log/slog` (stdlib, since 1.21) with `JSONHandler` in prod, `TextHandler` locally.

Always include: timestamp, level, message, and a request/correlation ID field (even if the project doesn't use it yet — wiring it later is annoying). Don't log secrets; if the project handles tokens or PII, add a redaction note in the README.

## i18n (frontend only)

Set this up from day one even if there's only one language. Retrofitting i18n into a codebase full of hardcoded strings is a multi-week project; doing it now costs an hour.

- React/Next.js: `react-i18next` or `next-intl` (next-intl for App Router projects).
- Vue: `vue-i18n`.
- Plain JS/other: `i18next` directly.

Create `locales/en.json` with the hello-world string, wire up a provider/hook, and use it in the hello-world component so the pattern is established. Backend services and CLIs don't need i18n unless explicitly asked.

## Sentry (when needed)

Add Sentry when the project will be deployed to real users in production. Skip it for libraries, internal tools where logs are sufficient, throwaway prototypes, or when the user says they don't want it.

If adding it: install the SDK, initialize as early as possible in the entry point, gate the DSN behind an env var (`SENTRY_DSN`) so local dev runs without it, and document the env var in the README. Don't hardcode a DSN.

## README structure

Use this exact skeleton — onboarding suffers when READMEs hide the run command three scrolls down:

```markdown
# <Project name>

<One-sentence description.>

## Prerequisites

- <Runtime> <exact version>  (e.g. Node 20.11.x — see `.nvmrc`)
- <Package manager> <version>
- <Any other tools: docker, make, etc.>

Recommended: install via `mise` / `asdf` so versions match `.tool-versions`.

## Quickstart

```bash
git clone <repo>
cd <repo>
make setup    # install deps, set up env
make test     # verify it works
make run      # start the app
```

## Common tasks

- `make lint` — run the linter
- `make format` — auto-format code
- `make test` — run tests
- `make audit` — check dependencies for vulnerabilities
- See `Makefile` for the full list.

## Configuration

<Env vars the app reads, with example values. Reference `.env.example`.>

## Project layout

<2–6 lines describing top-level directories.>
```

The prerequisites section must list **exact** versions (or version ranges with a clear "tested on" note), and they must match what's in `.nvmrc` / `.tool-versions` / `go.mod` / `pyproject.toml`. A new developer should be able to follow Quickstart top-to-bottom without alt-tabbing to ask questions.

Also include a `.env.example` with every env var the app reads (with placeholder values), and reference it from the Configuration section.

## Makefile

Use the `make-targets` skill for the target contract (`setup`, `lint`, `format`, `test`, `audit`, `run`, etc.). The Makefile is the stable interface — local dev and CI both call the same `make <target>` so they can't drift. Don't reimplement the contract here; if `make-targets` is unavailable, at minimum provide: `setup`, `lint`, `format`, `test`, `run`, and a default `help` target that lists them.

## Anti-patterns to avoid

- **Adding tooling without wiring it into Make and CI.** A linter that nobody runs is worse than no linter — it creates the illusion of quality.
- **Multiple sources of truth for versions.** Pick one (`.tool-versions` is good across ecosystems) and have the README point to it rather than restating versions that will drift.
- **Skipping the hello-world verification.** Always run `make setup && make test && make run` before declaring done. If any of those fail, the scaffold isn't finished.
- **Over-scaffolding.** Don't add Docker, k8s manifests, GitHub Actions, pre-commit hooks, etc. unless the user asked. The 10 items in this skill are the floor; everything else is opt-in.
- **Generating a 200-line README.** Onboarding READMEs should be scannable. Detail belongs in `docs/` or per-module READMEs.
