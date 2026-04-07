---
name: readme
description: Generate or update README.md files across the project. By default uses git diff to find only affected directories; pass --all to re-validate every directory. Leaf folders are written first; parent READMEs reference children without duplicating content. Use when the user asks to add, update, or refresh documentation.
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

Generate or update README.md files for the project.

## 0. Determine mode

Check the invocation arguments for `--all` (or `-a`). This controls how the target set is built in step 1.

- **Default (no flag):** use `git diff` to find only affected directories
- **`--all` flag:** skip the diff check and re-validate every directory in the tree

## 1. Identify the target set

### Default mode — git diff

Run:

```bash
git diff --name-only HEAD
git diff --name-only --cached
```

Collect every unique directory that contains a changed file (non-README source file). Then expand the set upward: for each changed directory, also include every ancestor up to the project root. This ensures parent READMEs are refreshed when a child changes.

If the user also named a specific directory, filter the result to that subtree.

If no source files have changed (clean working tree), say so and exit — nothing to do.

### `--all` mode — full tree

Ignore the diff. Build an ordered list of **all** directories in the project, leaves first. Use:

```bash
find <target_dir> -type d | sort -r
```

### Both modes — apply these exclusions

Skip any directory that:
- Is a `__pycache__`, `.git`, `.pio`, or `managed_components` subtree
- Contains only auto-generated files (`*.generated.*`)
- Has no source files of its own (pass-through with only subdirectories)

Process the final list **leaves first, root last** so parent READMEs are written after their children.

## 2. For each directory — understand before writing

Read all source files in the directory (headers, `.cpp`, `.py`, config files, etc.). Do **not** read into child subdirectories — their READMEs are the authoritative summary of what they contain.

Identify:
- What the directory does / what problem it solves
- Key classes, functions, or scripts it exposes
- Any non-obvious configuration or hardware notes
- Whether an existing README.md is already present (update vs. create)

## 3. Write the README

### Leaf directories (no subdirectories with their own README)

Write a thorough README covering everything a developer needs to understand and use the code:
- One-paragraph description at the top
- Files table (file → description)
- Classes / functions / scripts section with usage examples where helpful
- Configuration section if the code reads config or has build-time inputs
- Hardware notes where relevant

### Parent directories (has child subdirectories)

Write a README that:
- Describes the directory's role in one paragraph
- Has a **Subdirectories** table linking to each child README: `[child/](child/README.md)`
- Briefly (one line) summarises what each child contains — enough to navigate, not enough to duplicate
- Covers only content that lives directly in this directory (files not in any child)
- Does **not** repeat content already in a child README

### Root README (project root)

- Project name and one-paragraph description
- Quick-start (build, flash, configure)
- Top-level directory structure table linking to child READMEs
- HTTP API summary table (endpoints, methods, brief description)
- Links to CLAUDE.md for contributor/build guidance

## 4. Updating an existing README

- Read the existing file first
- Preserve any sections that are still accurate
- Update sections that are stale (wrong API, missing files, outdated config schema, etc.)
- Add sections for anything new that isn't documented
- Remove sections for things that no longer exist

## 5. Quality rules

- Use relative links for all cross-README references (`[child/](child/README.md)`)
- No emojis
- Code blocks for all CLI commands, config snippets, and C++ usage examples
- Keep parent READMEs short — they are navigation aids, not tutorials
- Do not document third-party code (`.pio/`, `managed_components/`) — link to upstream docs if needed
- After writing all READMEs, do a final pass: check that every child README linked from a parent actually exists

## 6. Report

After finishing, output a summary:
- List of files created (NEW) vs updated (UPDATED)
- Any directories skipped and why
