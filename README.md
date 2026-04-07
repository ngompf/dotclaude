# dotclaude

Shared Claude Code skills for ngompf projects.

## Skills

- **commit** — Stage and commit changes with a descriptive message
- **readme** — Generate or update README.md files across a project
- **upload** — Build and upload ESP32 firmware over WiFi (OTA)

## Adding to a project

```bash
git submodule add https://github.com/ngompf/dotclaude.git .claude/skills
git add .gitmodules .claude/skills
git commit -m "chore: add ngompf/dotclaude skills submodule"
git push
```

## After cloning a project that uses this submodule

```bash
git submodule update --init
```

Or clone with submodules from the start:

```bash
git clone --recurse-submodules <repo-url>
```

## Updating skills

To pull the latest skills into a project:

```bash
git submodule update --remote .claude/skills
git add .claude/skills
git commit -m "chore: update dotclaude skills submodule"
```
