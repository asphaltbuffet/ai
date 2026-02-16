# Changelog Skill Design

## Overview

A Claude Code skill that creates and updates `CHANGELOG.md` files following the [Common Changelog](https://common-changelog.org/) format. It analyzes git history to draft changelog entries, curates them per the spec, and presents them for review before writing.

## Trigger Conditions

- Explicit invocation via `/changelog [version]`
- Natural language: "update the changelog", "add a changelog entry", "prepare release notes", "review the changelog"
- Auto-detection of release-related work (version bumps, tagging)

## Approach

Single self-contained `SKILL.md` with embedded Common Changelog rules. No external scripts or reference files. Claude uses its standard tools (Bash for git, Read/Edit/Write for files) to implement the workflow.

## Workflow

1. **Detect state** — Check if `CHANGELOG.md` exists, list git tags (semver-sorted), determine creation vs append mode
2. **Gather changes** — Get commits between last tag and HEAD, extract subject/body/PR refs/authors, filter maintenance noise
3. **Draft entry** — Categorize into Changed/Added/Removed/Fixed, format in imperative mood with refs and authors, sort breaking-first
4. **Present for review** — Show draft to user, allow edits before writing
5. **Write** — Insert entry at correct position, use reference-style links

## Common Changelog Rules (Embedded)

### Structure
- File starts with `# Changelog`
- Releases: `## [VERSION] - YYYY-MM-DD` (no "v" prefix, ISO date, linked)
- Groups: `### Changed`, `### Added`, `### Removed`, `### Fixed` (this order only)
- Releases sorted newest-first

### Entry Format
- Imperative mood: "Add X", "Fix Y", "Remove Z"
- Format: `- Change text ([#ref](url)) (Author Name)`
- Breaking: `**Breaking:** prefix`, listed first in group
- Multiple refs same type: `(#1, #2)` not `(#1) (#2)`
- Authors optional for single-contributor projects

### Curation
- Exclude: dotfiles, dev-only deps, minor style, doc formatting
- Include: refactors, runtime env changes, new docs
- Merge related commits into single entries
- Skip no-op changes (reverts canceling out)

### Special Cases
- First release: `_First release._` notice
- Yanked releases: keep entry + explanatory notice
- Notices: single italic sentence before groups, max one per release

## File Location

```
/home/grue/dev/ai/
└── skills/
    └── changelog/
        └── SKILL.md
```

Developed in this repo, installable globally via Claude config or symlink.

## Change Source

Git tags and commit history only. No manual input mode. Works with standard git tags in any repo (including jj colocated).
