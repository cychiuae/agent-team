---
name: golang-prefer-stack
description: Preferred tooling stack and conventions for Go projects — Go modules for dependency management, gorilla/mux for HTTP routing, Cobra for command-line apps, Viper for config, sqlc for type-safe database access, golang-migrate for migrations, go-playground/validator for input validation, golangci-lint for linting, gofumpt for formatting, the standard testing package with testify for assertions, slog for structured logging, asynq for background jobs, and Sentry for error reporting. Use this skill whenever scaffolding, configuring, or making tooling decisions for a Go project — including picking a router, database access layer, migration tool, validator, linter, formatter, test runner, logger, or job queue, or when the user asks "what should I use for X" in a Go context.
---

# Go preferred stack

The opinionated defaults below. Don't relitigate these per-project unless there's a concrete reason the default doesn't fit.

## Core stack

| Concern                       | Tool                                      |
| ----------------------------- | ----------------------------------------- |
| Dependency / project mgmt     | **Go modules** (`go mod`)                 |
| HTTP router                   | **gorilla/mux** (`github.com/gorilla/mux`) |
| CLI framework                 | **Cobra** (`github.com/spf13/cobra`) — for command-line apps |
| DB access                     | **sqlc** (generated type-safe queries)    |
| DB driver                     | **pgx** (`github.com/jackc/pgx/v5`)       |
| DB migrations                 | **golang-migrate**                        |
| Input validation              | **go-playground/validator/v10**           |
| Linter (aggregator)           | **golangci-lint**                         |
| Formatter                     | **gofumpt** (stricter `gofmt`)            |
| Test runner                   | **`go test`** + **testify** (`require`)   |
| Structured logging            | **`log/slog`** (stdlib)                   |
| Background jobs               | **asynq** (Redis-backed) when needed      |
| Error reporting               | **Sentry** (`sentry-go`) when deployed    |
| Config                        | **Viper** (`github.com/spf13/viper`)      |

## Versions

- Go: latest stable (1.23+). Pin in `go.mod` via the `go` directive and (optionally) `toolchain`.
- Use Go modules exclusively. No `GOPATH`-style projects, no `dep`, no vendoring unless air-gapped builds force it.

## Project layout

Standard Go layout — keep it flat unless the project genuinely needs subdivision:

```
project/
  cmd/
    <binary-name>/
      main.go           # entrypoint only; no business logic
  internal/
    <package>/          # core packages — not importable outside this module
      ...
  migrations/           # if the project uses a database
    0001_init.up.sql
    0001_init.down.sql
  sqlc.yaml             # if using sqlc
  go.mod
  go.sum
  Makefile
  README.md
```

`internal/` is enforced by the toolchain — packages there can't be imported by other modules. Use it for everything that isn't a deliberately public API.

## Hello world

`cmd/<binary>/main.go` with a `main()` that initializes slog and logs a startup line. Wire `go build -o bin/<binary> ./cmd/<binary>` into the Makefile so a fresh clone builds on day one.

For an HTTP service, add a `mux.NewRouter()` with `GET /healthz` returning `{"status":"ok"}` and serve via a configured `*http.Server` (read/write timeouts set — never use the zero-value defaults in production).

For a CLI, scaffold with Cobra (see "CLI framework — Cobra" below): `main.go` calls `cmd.Execute()`, the root command lives in `internal/cmd/root.go`, and one trivial subcommand (e.g. `version`) is wired up so `make run -- version` works on day one.

## Linter — golangci-lint

Use **golangci-lint** as the single linting entrypoint. It bundles `govet`, `staticcheck`, `errcheck`, `ineffassign`, `revive`, etc., with one config and one cache.

`.golangci.yml` starter:

```yaml
run:
  timeout: 5m
linters:
  enable:
    - govet
    - staticcheck
    - errcheck
    - ineffassign
    - revive
    - gosimple
    - unused
    - gofumpt
    - misspell
    - bodyclose
    - errorlint
```

`make lint` → `golangci-lint run ./...`
`make format` → `gofumpt -w .` and `goimports -w .`

## Formatter — gofumpt

Use **gofumpt** instead of plain `gofmt`. It's a strict superset — every gofumpt-formatted file is also gofmt-formatted, but it adds a few extra rules (no empty lines at start of blocks, consistent composite literal style). Run it via the linter and as `make format`.

## Web framework — gorilla/mux + validator

[`gorilla/mux`](https://github.com/gorilla/mux) for HTTP routing. It's stdlib-shaped (`http.Handler`-compatible), supports path variables (`/users/{id}`), method/host/header matchers, and subrouters with per-group middleware. Don't introduce Gin/Echo/Fiber for new projects — `net/http` + `gorilla/mux` is enough.

Patterns:
- One `internal/handlers/` package; one file per resource. Wire routes in a `Routes(r *mux.Router)` function called from `main.go`.
- Use subrouters for versioned APIs: `api := r.PathPrefix("/api/v1").Subrouter()`. Attach middleware via `api.Use(...)`.
- Read path variables with `mux.Vars(r)["id"]` inside handlers.
- Decode request bodies into a struct, then run `validator.Struct(&req)` — never validate fields ad-hoc. Tag with `validate:"required,email"` etc.
- Pass dependencies (db, logger, config) via a small struct on the handler — no package-level globals.
- Load config via Viper into a typed `Config` struct (see "Config — Viper" below). Never call `os.Getenv` inside business logic.

```go
r := mux.NewRouter()
r.HandleFunc("/healthz", healthz).Methods(http.MethodGet)
r.HandleFunc("/users/{id}", getUser).Methods(http.MethodGet)
```

Note: gorilla/mux was archived for a period and revived under the gorilla org; it remains the most widely-used stdlib-style router and is actively maintained again.

## CLI framework — Cobra

For command-line applications (anything where the binary's primary interface is `<binary> <subcommand> [flags]`), use [`github.com/spf13/cobra`](https://github.com/spf13/cobra). Cobra pairs naturally with Viper — both are from the same author, and Cobra flags can be bound directly into Viper so flag/env/config precedence "just works".

When **not** to reach for Cobra:
- Pure HTTP services with no subcommands — keep `main.go` simple, parse a couple of flags with the stdlib `flag` package or skip flags entirely.
- Trivial single-purpose tools with one or two flags — `flag` is fine and ships with the stdlib.

Layout:

```
cmd/
  <binary>/
    main.go          # calls cmd.Execute()
internal/
  cmd/
    root.go          # rootCmd with persistent flags (--config, --verbose)
    serve.go         # `<binary> serve`
    migrate.go       # `<binary> migrate up|down`
```

Patterns:
- One file per subcommand under `internal/cmd/`. Each file declares its `*cobra.Command` and registers it in an `init()` via `rootCmd.AddCommand(...)`.
- Persistent flags (apply to all subcommands) live on `rootCmd`; local flags live on the specific subcommand.
- Bind flags into Viper with `viper.BindPFlag(...)` in the root command so config precedence is: flag > env > config file > default.
- Keep `RunE` (not `Run`) so subcommands can return errors and Cobra surfaces them with a non-zero exit code.
- Don't put business logic in `cmd/*.go`. Subcommand files parse flags and call into `internal/<package>` — keeps the surface testable without spinning up Cobra.

Skeleton:

```go
// internal/cmd/root.go
var rootCmd = &cobra.Command{
    Use:   "myapp",
    Short: "Short description",
}

func Execute() {
    if err := rootCmd.Execute(); err != nil {
        os.Exit(1)
    }
}

func init() {
    rootCmd.PersistentFlags().String("config", "", "config file path")
    _ = viper.BindPFlag("config", rootCmd.PersistentFlags().Lookup("config"))
}
```

```go
// cmd/myapp/main.go
package main

import "github.com/<org>/<repo>/internal/cmd"

func main() { cmd.Execute() }
```

## DB access — sqlc + pgx

Use **sqlc** to generate type-safe Go code from `.sql` files. You write SQL; sqlc emits typed Go functions. This avoids the runtime reflection cost and footguns of GORM/ent and keeps SQL legible in source control.

- Driver: **pgx** (Postgres). Configure sqlc with `sql_package: "pgx/v5"`.
- Layout: queries live in `internal/db/queries/*.sql`, generated code in `internal/db/`.
- Regenerate via `make gen` (`sqlc generate`) — commit the generated code so reviewers can see what the compiler sees.
- For non-Postgres databases (MySQL, SQLite), sqlc still works; swap the driver and dialect in `sqlc.yaml`.

Avoid GORM and ent in new projects. GORM hides SQL behind reflection (slow, hard to debug); ent is powerful but heavyweight. sqlc keeps you close to the database with full type safety.

## Migrations — golang-migrate

Every schema change ships as a numbered pair of `.up.sql` / `.down.sql` files in `migrations/`. No `AutoMigrate`-style schema syncing.

- Filename convention: `NNNN_short_description.up.sql` / `.down.sql` (zero-padded sequence).
- Apply via `migrate -path migrations -database $DATABASE_URL up` — wire into `make migrate-up` / `make migrate-down`.
- Migrations must be reversible (real `down.sql`) unless the change is genuinely one-way; if so, leave `down.sql` empty with a SQL comment explaining why.
- Run migrations as a separate step in deploy — don't run them from app startup. App boot races make that brittle in multi-replica deploys.

## Test runner — `go test` + testify

Use the standard `go test` with the `testify/require` package for ergonomic assertions. Avoid Ginkgo/Gomega — they fight Go's idiom and make stack traces noisy.

- One `_test.go` file per source file, same package (white-box) by default. Use `package foo_test` (black-box) when you need to verify only the public API.
- Table-driven tests (`tests := []struct{...}{...}`) are the default shape for anything with multiple cases.
- For DB-backed tests, use a real Postgres (testcontainers-go or a CI-managed instance). Don't mock `*sql.DB` or sqlc-generated interfaces — those tests pass while real queries fail.
- Coverage: `go test -cover ./...` baseline, `-coverprofile=coverage.out` for reports.

One trivial passing test in `internal/<pkg>/main_test.go` so `make test` is green from day one.

## Structured logging — log/slog

Use the **stdlib `log/slog`** — added in Go 1.21, JSON or text handler, structured by default. No third-party logger needed for new projects.

```go
import (
    "log/slog"
    "os"
)

func main() {
    h := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{Level: slog.LevelInfo})
    slog.SetDefault(slog.New(h))
    slog.Info("hello", "who", "world")
}
```

Use `slog.NewTextHandler` for local dev (readable) and `slog.NewJSONHandler` in deployed environments. Pass a `*slog.Logger` through dependency injection — don't reach for the global except in `main`.

For HTTP: write a small middleware that puts a `request_id` into the `context.Context` and creates a per-request logger via `logger.With("request_id", id)`. Pull it out in handlers with a helper.

Avoid logrus/zap/zerolog in new projects unless you have a measured perf reason. slog is enough and stdlib means no dependency drift.

## Background jobs — asynq

Add background jobs only when the project actually has work that should run outside the request cycle. Don't preemptively add it.

When adding:
- **asynq** (Redis-backed) is the default. It's well-maintained, has scheduling, retries, and a web UI.
- One `internal/tasks/` package; one file per logical job category.
- Task payloads are small and JSON-serializable — pass IDs, not full objects. The worker re-fetches from the DB.
- Run workers via a separate `cmd/worker/main.go` binary so the API and worker scale independently.

Alternatives: River (Postgres-backed, newer) is a fine pick if the project already has Postgres and wants to avoid Redis. Don't roll your own queue.

## Sentry

`sentry-go` with the `sentryhttp` middleware when there's an HTTP service. Initialize in `main` before starting the server:

```go
import "github.com/getsentry/sentry-go"

if dsn := os.Getenv("SENTRY_DSN"); dsn != "" {
    _ = sentry.Init(sentry.ClientOptions{Dsn: dsn, TracesSampleRate: 0.1})
    defer sentry.Flush(2 * time.Second)
}
```

DSN comes from env var; no-op when unset so local dev runs without Sentry traffic.

## Error handling

- Wrap errors with `fmt.Errorf("doing X: %w", err)` — always include context, always use `%w` so `errors.Is` / `errors.As` work upstream.
- Define sentinel errors (`var ErrNotFound = errors.New("not found")`) at the package level for conditions callers need to branch on.
- Don't `panic` in library code. `panic` is for truly unrecoverable invariants (nil pointer in init, etc.). HTTP handlers should recover via middleware and return 500.

## Config — Viper

Use [`github.com/spf13/viper`](https://github.com/spf13/viper) to load config from env vars (and optionally `config.yaml` / `.env` for local dev), then `Unmarshal` into a typed `Config` struct. Bind the struct in `main` once and pass it down — don't sprinkle `viper.GetString(...)` across business logic, and don't call `os.Getenv` directly.

```go
type Config struct {
    Port        int    `mapstructure:"PORT"`
    DatabaseURL string `mapstructure:"DATABASE_URL"`
    SentryDSN   string `mapstructure:"SENTRY_DSN"`
}

func Load() (Config, error) {
    v := viper.New()
    v.SetEnvPrefix("APP")
    v.AutomaticEnv()
    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))

    v.SetDefault("PORT", 8080)

    // Optional: load a local config file if present (don't fail if it isn't).
    v.SetConfigName("config")
    v.SetConfigType("yaml")
    v.AddConfigPath(".")
    _ = v.ReadInConfig()

    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return cfg, fmt.Errorf("unmarshal config: %w", err)
    }
    if cfg.DatabaseURL == "" {
        return cfg, errors.New("DATABASE_URL is required")
    }
    return cfg, nil
}
```

Notes:
- Env vars win over the config file (Viper's precedence: explicit `Set` > flag > env > config file > default).
- Use `mapstructure` tags on the struct (Viper unmarshals via `mapstructure`, not `encoding/json`).
- Validate required fields explicitly after `Unmarshal` — Viper doesn't enforce "required" via tags.
- Don't commit a populated `config.yaml`; ship a `config.example.yaml` and gitignore the real one.

## go.mod starter

```
module github.com/<org>/<repo>

go 1.23

require (
    github.com/gorilla/mux v1.x
    github.com/jackc/pgx/v5 v5.x
    github.com/go-playground/validator/v10 v10.x
    github.com/spf13/viper v1.x
    github.com/spf13/cobra v1.x
    github.com/stretchr/testify v1.x
)
```

Drop dependencies the project doesn't actually use (e.g., omit pgx for a pure CLI, omit gorilla/mux if there's no HTTP server, omit cobra if there are no subcommands).

## Make targets shell out to

- `go mod download` for setup
- `go test ./...` for test (add `-race` in CI)
- `golangci-lint run ./...` for lint
- `gofumpt -w .` + `goimports -w .` for format
- `go build -o bin/<binary> ./cmd/<binary>` for build
- `go run ./cmd/<binary>` for run
- `sqlc generate` for gen (when using sqlc)
- `migrate -path migrations -database $DATABASE_URL up` for migrate-up
