---
name: docker-local-dev
description: Default Docker conventions for backend projects — Dockerfile and docker-compose.yml exist primarily to support local development (hot-reload, bind-mounted source, ephemeral databases), NOT to produce production images. Use this skill whenever scaffolding or configuring Docker for a backend service, API, worker, or job queue; whenever the user says "dockerize", "add Docker", "set up docker compose", "run locally with Docker"; or whenever choosing between a dev-oriented and prod-oriented Dockerfile. Trigger even when the user only mentions "I want to run this locally" for a backend project, because Docker is our default local dev platform and other approaches (host installs, manual postgres) should be redirected to Docker unless the user opts out.
---

# Docker for backend local development

## The core principle

**For backend projects, Docker is the default local development platform.** The `Dockerfile` and `docker-compose.yml` at the repo root exist to make `git clone && make run` work on any machine that has Docker installed — nothing more. They are **not** the production build.

This matters because dev and prod Dockerfiles want opposite things:

| Concern             | Dev (default)                            | Prod (separate file when needed)         |
| ------------------- | ---------------------------------------- | ---------------------------------------- |
| Source code         | Bind-mounted from host for hot-reload    | Copied into the image, immutable         |
| Dependencies        | Installed once, cached in a named volume | Pinned, layered for cache efficiency     |
| Build target        | Single stage, dev tools included         | Multi-stage, slim runtime, no dev tools  |
| User                | Often root for convenience                | Non-root for security                    |
| Image size          | Don't care                                | Care a lot                                |

Trying to make one Dockerfile serve both ends up worse at both. Default to a clean dev-only setup; add `Dockerfile.prod` (or a multi-stage `Dockerfile` with explicit `dev` and `prod` targets) only when the project actually needs to publish images.

## When this skill applies

- Backend services (HTTP APIs, gRPC, job workers, cron services).
- Anything with a database, queue, cache, or other infra dependency where local dev means "spin up the whole stack."
- Any project where new contributors should be able to run `make setup && make run` without installing Postgres/Redis/etc. on their host.

It does **not** apply to: static sites, pure libraries/SDKs, CLIs that don't depend on infra, or frontend-only projects (those usually run faster directly on host).

## Standard layout

```
project/
  Dockerfile               # local dev build
  docker-compose.yml       # local dev orchestration (app + infra)
  .dockerignore
  Makefile                 # wraps `docker compose` calls
  .env.example             # references env vars used by compose
  Dockerfile.prod          # only when production images are needed
```

## Dockerfile (local dev)

Optimize for fast iteration, not image size. Install dev tools (test runner, linter, formatter, debugger) so the same image runs `make test` and `make lint` — contributors should never need to install language toolchains on the host.

Pattern (Python/uv example — adapt per language):

```dockerfile
FROM python:3.12-slim

# install uv
RUN pip install --no-cache-dir uv

WORKDIR /app

# install deps first (cached layer when only source changes)
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

# source is bind-mounted in compose; this COPY is a fallback for `docker build` alone
COPY . .

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--reload"]
```

Notes on the pattern:
- The `--reload` (or `nodemon`, `air`, etc.) is what makes hot-reload work alongside the bind mount in compose. A dev image without auto-reload defeats the point.
- Keep it a single stage. Multi-stage builds save image size in prod but slow `docker build` and obscure what's installed during debugging.
- Don't set `USER nonroot` here unless the project specifically requires it — contributors will hit permission issues on bind-mounted files (host UID ≠ container UID) and be confused. Save the hardening for `Dockerfile.prod`.

## docker-compose.yml (local dev)

This is the entry point contributors actually use. Define every service the app talks to so `docker compose up` spins up a complete environment.

Pattern:

```yaml
services:
  app:
    build: .
    command: uv run uvicorn app.main:app --host 0.0.0.0 --reload
    volumes:
      - .:/app                       # bind mount → hot-reload
      - venv:/app/.venv              # named volume → don't clobber container deps with host
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 10

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
  venv:
```

Key conventions:

- **Bind-mount the source, named-volume the deps.** The bind mount gives hot-reload; the named volume on `node_modules` / `.venv` / `vendor/` prevents the host's empty/stale dependency directory from shadowing the container's installed deps. This pair-of-mounts pattern is the single most common cause of "it works in build, breaks on `up`" — get it right.
- **Expose database/cache ports to the host** (5432, 6379, etc.) so contributors can connect with their preferred GUI tool (TablePlus, DBeaver) without `docker exec`.
- **Use healthchecks + `depends_on: condition: service_healthy`** for stateful services. Without healthchecks, the app starts before Postgres is accepting connections and the first request fails — the kind of papercut that makes new contributors think the project is broken.
- **Default credentials are fine** (`app/app/app`). This is a local dev compose file; it will never see production. Document this clearly in the README so no one is tempted to copy it to prod.
- **Don't add only the services this project uses.** If there's no Redis dependency, don't include Redis. Compose files accumulate unused services and they all start on `docker compose up`.

## Makefile integration

The Makefile is the stable interface. Contributors and CI both call `make <target>`; they don't call `docker compose` directly. This means changing how things run later (e.g. switching to devcontainers or Tilt) only requires editing the Makefile.

Targets shell out to compose:

```make
setup:        ## build images and pull infra
	docker compose build
	docker compose pull

run:          ## start the full local stack
	docker compose up

test:         ## run tests inside the app container
	docker compose run --rm app uv run pytest

lint:         ## run linters inside the app container
	docker compose run --rm app uv run ruff check .

format:
	docker compose run --rm app uv run ruff format .

shell:        ## drop into the app container
	docker compose run --rm app bash

down:         ## stop everything
	docker compose down

clean:        ## stop and remove volumes (DESTROYS local db data)
	docker compose down -v
```

Use `docker compose run --rm` (not `exec`) for one-off commands so contributors don't need to remember to start the stack first. `--rm` cleans up the throwaway container.

## .dockerignore

Always include one. Without it, `docker build` ships your `.git`, `node_modules`, `.venv`, build artifacts, and secrets into the build context — slow at best, leaky at worst.

Minimum:

```
.git
.gitignore
.env
.env.*
!.env.example
node_modules
.venv
__pycache__
*.pyc
dist
build
.coverage
.pytest_cache
```

## When the project needs production images

Don't bolt prod concerns onto the dev `Dockerfile`. Two clean options:

1. **Separate file**: `Dockerfile.prod` with multi-stage build, slim runtime base, non-root user, no dev deps. Build with `docker build -f Dockerfile.prod -t app:prod .`.
2. **Multi-stage `Dockerfile` with named targets**: `dev` and `prod` stages in one file. The compose file builds with `target: dev`; the prod pipeline builds with `--target prod`. Use this when the dev and prod images share most of their layers.

Either way, the README should be explicit: "the default Dockerfile/compose is for local dev. Production builds use X."

## Anti-patterns to avoid

- **One Dockerfile trying to be both dev and prod via env-var hacks.** The two have different goals; combining them produces an image that's bad at both and confusing to debug.
- **Skipping the named volume for dependencies.** Bind-mounting source over `/app` will also bind-mount over `/app/node_modules` (or `.venv`), shadowing whatever the build installed. Symptom: app runs in `docker build`, breaks on `docker compose up`.
- **No healthchecks on databases.** App starts faster than Postgres; first request 500s; new contributor thinks the app is broken.
- **Putting `make` targets that bypass Docker.** Once Docker is the default, `make test` running on the host violates "works on a clean machine with only Docker." If a contributor wants to run on host, they can call the underlying tool directly.
- **Committing a `.env` with real values.** `.env.example` is checked in; `.env` is gitignored. Compose reads from `.env` automatically.
