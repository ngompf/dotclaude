---
name: commit
description: Stage all changes and commit them with a descriptive message. Use when the user asks to commit changes.
allowed-tools: [Bash, Read, Glob, Grep]
---

Follow these steps to commit all pending changes.

## 1. Understand what changed

Run these in parallel:

```bash
git status --short
git diff
git log --oneline -5
```

Read the diff carefully. Identify:
- Which files changed and why
- Whether any new files should be included (untracked `??` entries)
- The overall theme of the changes (new feature, refactor, bug fix, etc.)

Do NOT commit files that look sensitive (`.env`, secrets, credentials).

## 2. Assess logical groupings

Before staging anything, consider whether the changes naturally split into **distinct, independent concerns** — for example:

- A new feature in `src/` alongside unrelated config tooling changes
- A refactor of one module alongside a bug fix in a completely different module
- Test files for a feature that was already committed separately

**A grouping is worth proposing only if:**
- The groups would make sense as separate entries in `git log` (a reader would benefit from them being separate)
- Each group could stand alone without the other (no file in group A depends on a change in group B)
- There are at least 2 clearly distinct concerns

**Do not propose a split if:**
- All changes are part of one coherent feature or fix
- The only difference is file type or directory (e.g. `.h` + `.cpp` for the same feature)
- The split would be arbitrary or cosmetic

If a meaningful split exists, present it to the user before doing anything:

> These changes look like they cover two distinct areas:
> - **Commit 1**: `<short description>` — `file_a.cpp`, `file_b.h`
> - **Commit 2**: `<short description>` — `config/x.yaml`, `scripts/y.py`
>
> Should I commit them separately, or as one commit?

Wait for the user's answer before proceeding. If they say one commit (or if no split was identified), continue to step 3 with all files.

## 3. Stage files

Stage modified files by name (never `git add -A` or `git add .`):

```bash
git add <file1> <file2> ...
```

For untracked files that belong to the change, include them too. If doing multiple commits, stage and commit each group in sequence.

## 4. Write the commit message

Draft a concise message:
- **Subject line**: `<type>: <what and why>` — keep it under 72 characters
  - Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Body** (optional): bullet points for non-obvious details
- Always end with the co-author trailer

## 5. Commit

```bash
git commit -m "$(cat <<'EOF'
<type>: <subject>

- <optional detail 1>
- <optional detail 2>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

## 6. Verify

```bash
git status
git log --oneline -3
```

Confirm the working tree is clean and the commit(s) appear at the top of the log.

## 7. Push

After a successful commit, push to origin unless the user explicitly says not to:

```bash
git push origin main
```

Report success or any error.

## Rules

- Prefer one commit for a cohesive set of changes; don't split arbitrarily
- Never use `--no-verify` or `--amend` on already-pushed commits
- Never force-push unless the user explicitly asks
- If a pre-commit hook fails, fix the issue and create a **new** commit — do not amend
