# agent-rules

A curated collection of AI agent rules and skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and [Cursor](https://cursor.com).

Rules are defined once and symlinked into your projects. Both agents pick them up automatically through their native conventions — no manual copying.

## Usage options

### 1. Use the installer (recommended)

The installer clones this repository to a local directory and creates symlinks into your projects:

```sh
curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh \
  | sh -s -- ~/code/project-a ~/code/project-b
```

You get the rules from this repo, applied consistently across all listed projects. Re-run the same command to pull updates and reapply.

### 2. Fork and customise

Fork this repository, add your own rules and skills to `agents/rules/` and `agents/skills/`, then point the installer at your fork:

```sh
REPO_URL="https://github.com/yourname/agent-rules.git" \
  curl -fsSL https://raw.githubusercontent.com/yourname/agent-rules/main/install.sh \
  | sh -s -- ~/code/project-a ~/code/project-b
```

To pull in upstream changes later, merge from the original repo into your fork using standard git workflow.

This is the recommended approach if you want to maintain your own rules alongside the ones provided here.

### 3. Download and manage manually

Browse the [`agents/rules/`](agents/rules) directory, download the files you want, and place them directly into your project's `.claude/rules/` or `.cursor/rules/` directory. No installer needed — but no automated updates either.

> **Do not add custom rules to `~/.local/share/agent-rules/`.** The installer runs `git reset --hard` on updates, which will delete any local additions.

## How it works

The installer clones this repository to `~/.local/share/agent-rules` (configurable) and creates symlinks from each target project into that shared copy:

| Agent | Symlink target | Behaviour |
|---|---|---|
| **Claude Code** | `.claude/rules/<name>.md` | Auto-loaded into every session |
| **Cursor** | `.cursor/rules/<name>.mdc` | Applied as project rules (`alwaysApply` via frontmatter) |

Skills (multi-file directories) are symlinked to both `.claude/skills/<name>/` and `.cursor/skills/<name>/`, following the [Agent Skills](https://agentskills.io) open standard.

All symlinks point back to the shared copy. Editing a rule inside a target project modifies the source — this is by design.

## Installation

### Quick start

```sh
curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh \
  | sh -s -- ~/code/project-a ~/code/project-b
```

### Using an environment variable

```sh
PROJECTS="~/code/project-a ~/code/project-b" \
  curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh | sh
```

### Prerequisites

- **git** (preferred) — the installer clones the repo with `--depth 1`
- **curl + tar** (fallback) — used automatically if git is not available

## Configuration

All configuration is via environment variables, set before running the installer.

| Variable | Default | Description |
|---|---|---|
| `REPO_URL` | `https://github.com/pecodez/agent-rules.git` | Git URL of the rules repository |
| `REPO_BRANCH` | `main` | Branch to install |
| `INSTALL_DIR` | `~/.local/share/agent-rules` | Where the shared copy is stored locally |
| `PROJECTS` | *(none)* | Space-separated list of project directories (alternative to passing as arguments) |

### Examples

Use a custom local directory:

```sh
INSTALL_DIR="$HOME/.agent-rules" \
  curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh \
  | sh -s -- ~/code/my-project
```

Install from a specific branch:

```sh
REPO_BRANCH="experimental" \
  curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh \
  | sh -s -- ~/code/my-project
```

## Updating

Re-run the install command. There is no separate update step — installing and updating are the same operation.

1. The installer pulls the latest changes from the configured repo (`git fetch` + `git reset --hard`)
2. Symlinks are removed and recreated, picking up any new, renamed, or deleted rules

```sh
curl -fsSL https://raw.githubusercontent.com/pecodez/agent-rules/main/install.sh \
  | sh -s -- ~/code/project-a ~/code/project-b
```

## What gets installed

For each target project, the installer creates:

```
your-project/
├── .claude/
│   ├── rules/
│   │   ├── always-ask-agent-mode.md  → symlink
│   │   └── confidence-rating.md      → symlink
│   └── skills/
│       └── <skill-name>/             → symlink (if skills exist)
└── .cursor/
    ├── rules/
    │   ├── always-ask-agent-mode.mdc → symlink
    │   └── confidence-rating.mdc     → symlink
    └── skills/
        └── <skill-name>/             → symlink (if skills exist)
```

## Included rules

### `always-ask-agent-mode`

Requires the agent to ask for explicit user confirmation before making any changes. The agent must pause and confirm at every step — not just at the start of a task. Clarifying questions are never interpreted as permission to proceed.

### `confidence-rating`

Mandates a structured confidence footer on every response with a percentage rating (High / Medium / Low / Uncertain), justification, and sources. Defines mode-specific requirements for Ask, Plan, and Agent modes.

## Repository structure

```
agents/
  rules/              *.md or *.mdc rule files
  skills/             skill directories (each contains SKILL.md + supporting files)
install.sh            installer script
AGENTS.md             project documentation for AI agents
LICENSE               MIT
README.md             this file
```

## Adding rules and skills (fork workflow)

After forking this repo, you can add your own rules and skills.

### Rules

Create a new `.mdc` file in `agents/rules/`:

```yaml
---
description: Brief description of what the rule does
globs:
alwaysApply: true
---
```

Write the rule content in markdown below the frontmatter, then re-run the installer on your projects.

### Skills

1. Create a directory under `agents/skills/<skill-name>/`.
2. Add a `SKILL.md` file (required).
3. Add any supporting files alongside it.
4. Re-run the installer.

Both Claude Code and Cursor get the full skill directory at `.claude/skills/<name>/` and `.cursor/skills/<name>/` respectively.

## Important notes

- **Symlinks, not copies.** Editing a rule inside a target project modifies the shared source.
- **Re-run after changes.** After adding, renaming, or removing rules in your fork, re-run the installer on each target project.
- **Updates are destructive to local changes.** The installer runs `git reset --hard` when updating, so any files added directly to the local install directory will be lost. Use a fork to maintain custom rules.
- **POSIX sh.** The installer is written for `/bin/sh` compatibility. No Bash required.
- **Shallow clone.** The repo is cloned with `--depth 1` by default, so full git history is not available in the local copy.
- **`.gitignore` the symlinks.** You may want to add `.claude/` and `.cursor/rules/` to your project's `.gitignore` so the symlinks aren't committed.

## License

MIT
