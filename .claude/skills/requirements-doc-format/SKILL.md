---
name: requirements-doc-format
description: Canonical structure for top-level docs/ files capturing business requirements. Use when writing or verifying docs/ — covers file organization, required sections, requirement IDs, and deprecation rules.
---

# Requirements doc format

`docs/product-spec/` is the single source of truth for **current** business requirements. WHAT and WHY only — never HOW.

## Where docs live

`docs/` is organized by purpose. Each category has its own subdirectory:

| Subdirectory          | Contents                                                |
|-----------------------|---------------------------------------------------------|
| `docs/product-spec/`  | Business requirements (this skill governs the format)   |
| `docs/architecture/`  | System architecture, design decisions, ADRs            |
| `docs/plans/`         | Per-feature planning artifacts (orchestrated by /ship) |
| `docs/<other>/`       | New categories get their own dedicated subdirectory    |

Never put requirements outside `docs/product-spec/`. Never mix architecture or how-it-works content into product-spec.

## File organization within `docs/product-spec/`

- `docs/product-spec/<area>.md` per business domain or feature area (e.g. `docs/product-spec/billing.md`, `docs/product-spec/auth.md`, `docs/product-spec/notifications.md`).
- One feature can update multiple files; one file can hold many features. Don't create a file per feature.
- Keep file names lowercase, kebab-case if multi-word.

## Required structure per file

```markdown
# <Area name>

## Purpose
One paragraph: what this area delivers and to whom.

## Requirements

### R-<area>-<n>: <short title>
- **Behavior**: <statement of observable behavior in business terms>
- **Rationale**: <why this is required>
- **Acceptance**: <observable outcome that proves it works>
```

## Rules

- **Testable**: every requirement is a statement of observable behavior. If you can't write a Given/When/Then for it, rewrite it.
- **Stable IDs**: `R-<area>-<n>` (e.g. `R-billing-3`). Once assigned, never reuse, never renumber.
- **No implementation**: no API names, no file paths, no class names, no DB tables.
- **No prose-only sections**: every claim about behavior lives under a numbered requirement so it can be referenced.
- **Changes**: when a requirement's meaning changes, deprecate the old ID (`R-billing-3 [DEPRECATED <date> — replaced by R-billing-12]`) and add a new one. Don't silently mutate existing IDs.
- **Removals**: keep the heading with a removal marker (`R-billing-3 [REMOVED <date> — reason]`) instead of deleting. Preserves traceability for old commits and tests.
- **Scope**: only include what is currently true (or is being added in this change). Future ideas live elsewhere (e.g. an internal roadmap), not here.

## What does NOT belong in `docs/product-spec/`

- Implementation plans → `docs/plans/<slug>/`
- Architecture / design / how-it-works → `docs/architecture/`
- Sprint planning, owner assignment → not in `docs/` at all
- Code style guides → not in `docs/product-spec/`
- Anything that would rot if the team rewrote the implementation
