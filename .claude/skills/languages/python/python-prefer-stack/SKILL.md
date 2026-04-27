---
name: python-prefer-stack
description: Preferred tooling stack and conventions for Python projects — uv for package management, FastAPI for HTTP APIs, SQLAlchemy + Alembic for database access and migrations, Pydantic for models/schemas, Ruff for lint+format, ty (Astral) for static type checking, pytest for tests, structlog for structured logging, Celery for background jobs, and Sentry for error reporting. Use this skill whenever scaffolding, configuring, or making tooling decisions for a Python project — including picking a package manager, web framework, ORM, migration tool, validation library, linter, formatter, type checker, test runner, logger, or job queue, or when the user asks "what should I use for X" in a Python context.
---

# Python preferred stack

The opinionated defaults below. Don't relitigate these per-project unless there's a concrete reason the default doesn't fit.

## Core stack

| Concern                       | Tool                          |
| ----------------------------- | ----------------------------- |
| Package / project management  | **uv**                        |
| API / HTTP server             | **FastAPI**                   |
| ORM / DB access               | **SQLAlchemy** (2.x style)    |
| DB migrations                 | **Alembic**                   |
| Models & schemas (validation) | **Pydantic** (v2)             |
| Linter + formatter            | **Ruff**                      |
| Static type checker           | **ty** (Astral)               |
| Test runner                   | **pytest**                    |
| Structured logging            | **structlog**                 |
| Background jobs               | **Celery** (when needed)      |
| Error reporting               | **Sentry** (when deployed)    |

## Versions

- Python: 3.12+ (3.11 if a dependency forces it). Pin in `.python-version` and `pyproject.toml`'s `requires-python`.
- Package / project manager: **`uv`**. It handles venvs, lockfiles, and Python installs in one tool — replaces pip+pip-tools+virtualenv+pyenv. Don't use `poetry` or `pip + requirements.txt` for new projects.

## Project layout

Use the `src/` layout — it prevents accidental imports of the working tree and forces an editable install:

```
project/
  src/
    <package_name>/
      __init__.py
      main.py
  tests/
    test_main.py
  alembic/                  # if the project uses a database
    versions/
    env.py
  pyproject.toml
  alembic.ini               # if the project uses a database
  README.md
  Makefile
```

## Hello world

`src/<package>/main.py` with a `main()` that logs (using structlog) and an `if __name__ == "__main__": main()` guard. Wire a `[project.scripts]` entry in `pyproject.toml` so it installs as a CLI command.

For a FastAPI service, add `src/<package>/app.py` with a minimal `app = FastAPI()` and a `GET /healthz` route returning `{"status": "ok"}`. Run via `uv run uvicorn <package>.app:app --reload` in dev.

## Linter + formatter — Ruff

Use **`ruff`** for both linting and formatting. It replaces flake8, isort, pyupgrade, black, and a dozen plugins, runs ~100x faster, and has one config block.

In `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "N", "RUF"]
ignore = []
```

`make lint` → `ruff check .` and `ruff format --check .`
`make format` → `ruff format .` and `ruff check --fix .`

## Type checking — ty (Astral)

Use **`ty`**, Astral's static type checker. It's fast (Rust-based, like ruff), integrates cleanly with the same toolchain, and is the intended successor to mypy/pyright in Astral-flavored projects.

Run it as part of `make lint`: `uv run ty check src/`. Configure in `pyproject.toml` under `[tool.ty]` if needed; defaults are sensible. Start permissive on existing codebases and tighten over time — landing strict mode on day one in a non-trivial project causes more friction than value.

Note: `ty` is newer than mypy/pyright. If a project must integrate with tooling that only speaks mypy (some IDE setups, certain CI plugins), fall back to mypy and document the reason in the README.

## Web framework — FastAPI + Pydantic

FastAPI for any HTTP service. Define request/response schemas as Pydantic v2 models — never hand-roll dict validation. Pydantic models double as OpenAPI schema sources, so you get docs at `/docs` for free.

Patterns:
- One `routers/` package, one router per resource. Mount on the app in `app.py`.
- Dependencies (DB session, current user) declared via FastAPI's `Depends` — don't import a global session.
- Settings via `pydantic-settings` (`BaseSettings`), reading from env / `.env`. Never read `os.environ` ad-hoc inside business logic.

## ORM — SQLAlchemy 2.x

Use SQLAlchemy 2.x style (the `Mapped[...]` annotations + `select()` API). Avoid the legacy 1.x `Query` API in new code — it's confusing alongside async patterns and the type checker doesn't model it as well.

- Define ORM models in `<package>/models/`. Keep them separate from Pydantic schemas — ORM models map rows; Pydantic models validate I/O. Mixing them creates pain when the table shape and the API shape need to diverge.
- Provide a session factory and a FastAPI dependency that yields a session per request and closes it on exit.
- Use async SQLAlchemy (`AsyncSession`) only if the project is end-to-end async; otherwise sync sessions are simpler and fine.

## Migrations — Alembic

Every schema change ships as an Alembic migration. No exceptions — `Base.metadata.create_all` is fine for tests, never for production.

- Generate with `uv run alembic revision --autogenerate -m "<description>"` after editing models, then review the generated script before committing — autogenerate misses things (enum changes, server defaults, indexes on existing columns).
- One logical change per migration. Don't bundle unrelated schema changes.
- Migrations must be reversible (write a real `downgrade()`) unless the change is genuinely one-way (e.g. dropping a column with data) — note the reason in the migration docstring.

## Test runner — pytest

`pytest` with `pytest-cov` for coverage. Configure in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra --strict-markers"
```

For DB-backed tests, use a real database (SQLite for unit tests, Postgres in CI for anything using Postgres-specific features) — don't mock SQLAlchemy. Mock-the-DB tests pass while real queries fail.

One trivial passing test in `tests/test_main.py` so `make test` is green from day one.

## Structured logging — structlog

```python
import structlog

structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),  # swap for ConsoleRenderer in dev
    ],
)
log = structlog.get_logger()
log.info("hello", who="world")
```

Use `ConsoleRenderer` when `sys.stderr.isatty()` for readable local output, `JSONRenderer` otherwise. Route stdlib logs (libraries, SQLAlchemy, uvicorn) through structlog with `structlog.stdlib.ProcessorFormatter` so everything ends up structured.

For FastAPI: bind a `request_id` into structlog's contextvars at the start of each request via middleware so every log line during that request includes it.

## Background jobs — Celery

Add Celery only when the project actually has work that should run outside the request cycle (long-running jobs, scheduled tasks, fan-out work). Don't preemptively add it.

When adding it:
- Broker: Redis for most cases. RabbitMQ if the team already runs it or the workload needs advanced routing.
- Result backend: Redis (or skip if results aren't needed — most fire-and-forget jobs don't need a result store).
- One `<package>/tasks/` package, one module per logical job category. Use `@shared_task` so tasks aren't tied to a specific app instance.
- Keep task signatures small and serializable — pass IDs, not ORM objects. The worker should re-fetch from the DB.
- Run workers via `uv run celery -A <package>.celery_app worker` — wire up via `make` so it's the same command everywhere.

## Sentry

`sentry-sdk` with the FastAPI / Celery integrations enabled when those are in use. Initialize in the entry point before any other imports that might raise:

```python
import os
import sentry_sdk

if dsn := os.getenv("SENTRY_DSN"):
    sentry_sdk.init(dsn=dsn, traces_sample_rate=0.1)
```

DSN comes from env var; no-op when unset so local dev runs without Sentry traffic.

## i18n

Backend Python services rarely need i18n. Skip unless explicitly required. If the project serves rendered templates (Django, Flask + Jinja), use the framework's built-in i18n (`gettext` / `Flask-Babel`).

## pyproject.toml starter

```toml
[project]
name = "<package_name>"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi",
    "uvicorn[standard]",
    "pydantic>=2",
    "pydantic-settings",
    "sqlalchemy>=2",
    "alembic",
    "structlog>=24",
]

[project.optional-dependencies]
dev = ["pytest>=8", "pytest-cov", "ruff", "ty"]
celery = ["celery[redis]"]

[project.scripts]
<cli-name> = "<package_name>.main:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Drop dependencies the project doesn't actually use (e.g., omit `fastapi` for a pure CLI, omit `sqlalchemy`/`alembic` if there's no database).

## Make targets shell out to

- `uv sync` for setup (also runs Alembic migrations on local dev if a DB is present — `uv run alembic upgrade head`)
- `uv run pytest` for test
- `uv run ruff check .` + `uv run ty check src/` for lint
- `uv run ruff format .` + `uv run ruff check --fix .` for format
- `uv run uvicorn <package>.app:app --reload` for run (FastAPI services)
- `uv run python -m <package>` or the installed script for run (CLIs)
