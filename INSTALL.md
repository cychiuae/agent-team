# INSTALL

## If you are a human

Paste the following prompt into your Claude Code (or other Claude-powered agent) session, with the working directory set to the project you want to install this into:

```
Read https://raw.githubusercontent.com/cychiuae/agent-team/refs/heads/main/INSTALL.md
Install it per its INSTALL.md
```

The agent will take it from here.

---

## If you are an LLM agent

You are installing the `agent-team` toolkit into the user's **existing project directory** (your current working directory). Follow these steps exactly.

### 1. Clone the source repository to a temporary location

Clone `https://github.com/cychiuae/agent-team` into a temporary directory **outside** the user's project. Do not clone into the user's project directory.

```
git clone https://github.com/cychiuae/agent-team /tmp/agent-team-install
```

### 2. Files to copy into the user's project

Copy the following from the cloned repo into the root of the user's current project directory:

- `.claude/` (the entire directory and its contents, recursively)
- `CLAUDE.md`
- `docs/` (the entire directory and its contents, recursively)

Do **not** copy anything else (no `README.md`, no `INSTALL.md`, no `.git`, no other files).

### 3. Conflict resolution

For **each** file you are about to write, check whether a file already exists at the destination path.

If a destination file already exists, **stop and ask the user** which resolution to apply for that file. Present these three options verbatim:

> A file already exists at `<path>`. How would you like to resolve this?
> 1. **Replace** — overwrite the existing file with the incoming file.
> 2. **Skip** — keep the existing file unchanged; do not copy the incoming file.
> 3. **Merge** — analyze both files and produce a merged version that preserves the user's existing content while incorporating the incoming additions. Show the merge result before writing.

Apply the user's chosen resolution before moving to the next conflicting file. Ask per file — do not assume a single answer applies to all conflicts unless the user explicitly says so (e.g., "replace all", "skip all").

If a destination file does **not** exist, copy it directly without prompting.

### 4. After installation

- Remove the temporary clone (e.g., `rm -rf /tmp/agent-team-install`).
- Report a summary to the user: which files were created, replaced, skipped, or merged.
- Do not commit anything. Leave staging and committing to the user.
