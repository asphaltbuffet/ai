# AGENTS.md

## Project Overview

A repository of AI skills and plugins. Currently contains a **changelog skill** for Claude Code that generates and maintains `CHANGELOG.md` files following the [Common Changelog](https://common-changelog.org/) format.

This is a **documentation-only repo** — no build system, no runtime code, no compiled artifacts. All content is Markdown.

## Version Control

- **Git + Jujutsu (jj)**: The repo is colocated — both `.git/` and `.jj/` exist. Commits have historically been made with `jj` (see the implementation plan for `jj describe` / `jj new` patterns).
- **No tags** are currently defined.
- **Branches**: `main` is the primary branch; feature work uses `push-*` branches.
- **Conventional commits**: Messages follow `type: description` format (e.g., `feat:`, `fix:`).

## Repository Structure

```
.
├── AGENTS.md                  # This file
├── README.md                  # Brief project description
├── .gitignore                 # Editor, OS, and env ignores
├── .claude/
│   └── settings.local.json    # Claude Code local settings (web fetch permissions)
├── docs/
│   └── plans/                 # Design documents and implementation plans
│       ├── *-design.md        # High-level design decisions
│       └── *-plan.md          # Step-by-step implementation plans
└── skills/
    └── changelog/
        └── SKILL.md           # Claude Code skill definition
```

### Key Directories

- **`skills/`** — Claude Code skills. Each skill lives in its own subdirectory with a `SKILL.md` file containing YAML frontmatter and Markdown instructions.
- **`docs/plans/`** — Design and planning documents. Named with date prefix: `YYYY-MM-DD-<topic>-{design,plan}.md`.

## Skills Architecture

Skills are self-contained Markdown files (`SKILL.md`) that instruct Claude Code how to perform a specific task.

### SKILL.md Format

```yaml
---
name: <skill-name>
description: <when Claude should activate this skill>
argument-hint: <optional argument placeholder>
allowed-tools: <comma-separated list of permitted tools>
---
```

Followed by Markdown body with:
1. Overview and trigger conditions (`## When to Use`)
2. Prerequisites
3. Step-by-step workflow
4. Format rules / specifications
5. Concrete examples

### Conventions Observed in Existing Skills

- **Imperative mood** for action descriptions
- **Strict verb vocabulary** — the changelog skill enforces a specific set of approved verbs (Add, Bump, Clarify, Deprecate, Document, Drop, Enable, Fix, Prevent, Refactor, Remove, Support, Use)
- **Git CLI for data gathering** — skills use `git log`, `git tag`, `git remote` etc. via `Bash(git *)` tool permission
- **Present-then-write pattern** — draft output is shown to the user for review before writing files
- **No external dependencies** — skills are pure Markdown instruction, no scripts or binaries

## Commands

There are no build, test, lint, or deploy commands. This is a pure Markdown documentation repo.

### Useful Commands for Working in This Repo

```bash
# View git history
git log --oneline

# Check skill file
cat skills/changelog/SKILL.md

# If using jj for commits:
jj describe -m "type: description"
jj new
```

## Claude Code Configuration

`.claude/settings.local.json` grants web fetch permission for `common-changelog.org` — this is used by the changelog skill to reference the spec.

## Development Workflow

1. **Design first**: Create a design doc in `docs/plans/YYYY-MM-DD-<topic>-design.md`
2. **Plan tasks**: Create an implementation plan in `docs/plans/YYYY-MM-DD-<topic>-plan.md` with numbered tasks
3. **Implement incrementally**: Each task gets its own commit
4. **Iterate on correctness**: Refine with `fix:` commits as needed (the changelog skill went through several rounds of tightening its rules)

## Naming Conventions

- **Files**: lowercase with hyphens (`changelog-skill-design.md`)
- **Directories**: lowercase (`skills/`, `docs/plans/`)
- **Skill directories**: named after the skill (`skills/changelog/`)
- **Plan files**: date-prefixed (`2026-02-16-changelog-skill-plan.md`)
- **Commits**: conventional format (`feat:`, `fix:`)

## Gotchas

- **Jujutsu colocated repo**: The `.jj/` directory means this repo uses Jujutsu alongside Git. Standard `git` commands work, but the project's commit workflow uses `jj` commands. Either works; be aware both VCS states exist.
- **No branch checked out**: `git branch` may show `* (no branch)` — this is normal for jj-managed repos. The working copy is managed by jj.
- **Skills are instructions, not code**: `SKILL.md` files don't execute — they're prompts that Claude Code follows. "Testing" a skill means invoking it in Claude Code and verifying the output.
- **Common Changelog specifics**: The changelog skill is strict about format. If modifying it, read the [Common Changelog spec](https://common-changelog.org/) to ensure compliance. Key constraints: no `v` prefix on versions, specific group ordering (Changed → Added → Removed → Fixed), reference-style links at file bottom.
