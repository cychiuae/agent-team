---
name: readme-maintenance
description: Keep README.md practical for new contributors by documenting runtime tech stack versions, local setup and startup steps, and project-specific onboarding sections. Use this skill whenever the user asks to update, rewrite, or improve a README; whenever onboarding is unclear or incomplete; whenever runtime versions are missing or stale; or whenever startup commands are undocumented. Trigger even if the user phrases the request indirectly ("the README is out of date", "document how to run this", "what does a new dev need to know") — the goal is a README that lets a new contributor go from clone to running in minutes.
---

# README Maintenance

## Purpose

Keep `README.md` practical for new contributors by documenting:
- runtime tech stack versions
- local setup and startup steps
- any requested project-specific onboarding sections

## When To Use

Use this skill when:
- the user asks to update or rewrite `README.md`
- project onboarding is unclear or incomplete
- runtime versions are missing
- startup commands are undocumented

## Update Workflow

Copy and track this checklist:

```md
README update progress
- [ ] Inspect existing README structure and tone
- [ ] Detect runtime versions (Node.js, Python, Go, etc.)
- [ ] Add or update "Tech Stack" section
- [ ] Add or update "Quickstart" section
- [ ] Add any user-requested extra section(s)
- [ ] Verify commands work or flag assumptions
- [ ] Keep README concise and skimmable
```

## Step 1: Inspect Existing Project Signals

Use repository files and toolchain configs to infer accurate versions:
- Node.js: `.nvmrc`, `.node-version`, `package.json` (`engines.node`)
- Python: `.python-version`, `pyproject.toml`, `requirements*`, `runtime.txt`
- Go: `go.mod` (`go` directive)
- containerized stacks: `Dockerfile`, `docker-compose.yml`

Rules:
- prefer explicit version declarations over inferred versions
- if exact version is unavailable, document supported range and source
- do not invent versions

## Step 2: Document Tech Stack

Add a `## Tech Stack` section with concise bullets.

Template:

```md
## Tech Stack

- Node.js: `vX.Y.Z` (from `.nvmrc`)
- Python: `X.Y` (from `.python-version`)
- Go: `X.Y` (from `go.mod`)
```

If a runtime is not used by the project, omit it.

## Step 3: Document Quickstart

Add a `## Quickstart` section with copy-paste commands.

Template:

````md
## Quickstart

```bash
npm run install
docker compose up
npm run dev
```
````

Guidelines:
- prefer the project's actual command conventions (`npm`, `pnpm`, `yarn`, `make`, etc.)
- keep command order logical: install -> infrastructure -> app start
- include only minimum commands needed for first successful run

## Step 4: Add User-Requested Extra Sections

When the user asks for additional sections (for example item "3"), add them after `Quickstart` unless the user specifies placement.

Common extras:
- `## Prerequisites`
- `## Environment Variables`
- `## Troubleshooting`
- `## Development Commands`

If the user indicates an extra section but leaves details incomplete, ask one focused clarifying question.

## Quality Bar

Before finishing:
- headings are clear and non-duplicative
- commands are fenced and executable
- version claims cite a concrete source file
- wording is concise (no long prose for simple setup)

## Guardrails

- do not add secrets or real credentials to README examples
- do not commit or push unless explicitly requested
- align with existing repository terminology and command style
