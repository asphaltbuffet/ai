# Changelog Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a Claude Code skill that generates and updates CHANGELOG.md files following the Common Changelog format, sourcing changes from git history.

**Architecture:** Single self-contained SKILL.md with embedded Common Changelog rules and a step-by-step workflow. No external scripts or reference files. Claude uses Bash (git), Read, Edit, and Write tools to implement the workflow at invocation time.

**Tech Stack:** Markdown (SKILL.md), git CLI for history analysis

---

### Task 1: Create skill directory structure

**Files:**
- Create: `skills/changelog/SKILL.md`

**Step 1: Create the directory**

```bash
mkdir -p skills/changelog
```

**Step 2: Create SKILL.md with frontmatter**

Create `skills/changelog/SKILL.md` with:

```yaml
---
name: changelog
description: Create or update CHANGELOG.md following the Common Changelog format (common-changelog.org). Use when user asks to create, update, review, or add entries to a changelog. Also use when detecting release-related work like version bumps, tagging, or preparing release notes.
argument-hint: [version]
allowed-tools: Read, Grep, Glob, Bash(git *)
---
```

**Step 3: Commit**

```bash
jj describe -m "feat: add changelog skill frontmatter"
jj new
```

---

### Task 2: Write the skill overview and trigger section

**Files:**
- Modify: `skills/changelog/SKILL.md`

**Step 1: Add overview and when-to-use section**

After the frontmatter, add:

```markdown
# Changelog

Generate and maintain CHANGELOG.md files following the [Common Changelog](https://common-changelog.org/) format by analyzing git history.

## When to Use

- User invokes `/changelog` or `/changelog 1.2.0`
- User asks to "update the changelog", "add a changelog entry", "prepare release notes", or "review the changelog for missing content"
- Release-related work detected (version bumps, tagging)

## Prerequisites

- Project under git version control
- Semantic versioning with git tags (e.g., `v1.0.0` or `1.0.0`)
```

**Step 2: Commit**

```bash
jj describe -m "feat: add changelog skill overview section"
jj new
```

---

### Task 3: Write the workflow section

**Files:**
- Modify: `skills/changelog/SKILL.md`

**Step 1: Add the workflow**

This is the core procedural section. Add the step-by-step workflow that Claude follows when the skill is invoked. The workflow has 5 phases: detect state, gather changes, draft entry, present for review, write.

Each step includes the exact git commands to run and the logic for processing results. Key details:

- **Detect state:** Check for existing CHANGELOG.md, list tags with `git tag --list --sort=-v:refname`, determine creation vs append mode
- **Gather changes:** Use `git log <last-tag>..HEAD --format="%H %ae %s" --reverse` to get commits, then for each commit get full details with `git log -1 --format="%H%n%ae%n%an%n%s%n%b" <sha>` and extract PR refs from subjects/bodies
- **Draft entry:** Categorize each change into Changed/Added/Removed/Fixed groups, format in imperative mood, add references and authors, sort breaking-first then by importance
- **Present for review:** Show the drafted entry to the user in a fenced code block, ask for approval/edits before writing
- **Write:** For new files create with `# Changelog` header; for existing files insert after the header line and before existing entries; add reference-style links at the bottom of the file

**Step 2: Commit**

```bash
jj describe -m "feat: add changelog skill workflow"
jj new
```

---

### Task 4: Write the Common Changelog format rules section

**Files:**
- Modify: `skills/changelog/SKILL.md`

**Step 1: Add the format rules**

Embed a concise but complete reference of Common Changelog rules. This section is what Claude checks against when formatting entries. Cover:

- **File structure:** Starts with `# Changelog`, releases as `## [VERSION] - YYYY-MM-DD`, groups as `### Changed | Added | Removed | Fixed` (in that order)
- **Entry format:** Imperative mood, `- Change text ([#ref](url)) (Author Name)`, breaking prefix `**Breaking:**`, multiple refs as `(#1, #2)`
- **Curation rules:** What to exclude (dotfiles, dev-only deps, minor style, doc formatting), what to include (refactors, runtime env changes, new docs), merge related commits, skip no-ops
- **Special cases:** First release notice `_First release._`, yanked releases, notices (single italic sentence before groups, max one per release)
- **Reference links:** Use reference-style `[VERSION]: url` at file bottom, link releases to GitHub release or tag comparison URLs

**Step 2: Commit**

```bash
jj describe -m "feat: add Common Changelog format rules to skill"
jj new
```

---

### Task 5: Write example output section

**Files:**
- Modify: `skills/changelog/SKILL.md`

**Step 1: Add a concrete example**

Include a short example showing what a properly formatted changelog entry looks like. This helps Claude (and users reading the skill) see the target format clearly:

```markdown
## Example Output

A properly formatted entry:

​```markdown
## [1.2.0] - 2026-02-16

### Changed

- **Breaking:** rename `process()` to `run()` for consistency ([#42](https://github.com/owner/repo/pull/42)) (Alice Meerkat)
- Refactor internal queue to improve throughput ([`a1b2c3d`](https://github.com/owner/repo/commit/a1b2c3d))

### Added

- Add `--dry-run` flag to preview changes ([#38](https://github.com/owner/repo/pull/38)) (Bob Badger)

### Fixed

- Fix crash when input file is empty ([#41](https://github.com/owner/repo/pull/41))

[1.2.0]: https://github.com/owner/repo/releases/tag/v1.2.0
​```
```

**Step 2: Commit**

```bash
jj describe -m "feat: add example output to changelog skill"
jj new
```

---

### Task 6: Final review and validation

**Step 1: Read the complete SKILL.md and verify**

- Frontmatter is valid YAML with correct fields
- Under 500 lines total
- No absolute paths or personal info
- Consistent terminology throughout
- All workflow steps are clear and actionable
- Format rules match the Common Changelog spec
- Example output follows all stated rules

**Step 2: Verify the skill loads**

Test that Claude Code recognizes the skill (if installed locally).

**Step 3: Final commit with all tasks complete**

```bash
jj describe -m "feat: complete changelog skill"
jj new
```

---

## Execution Notes

- Tasks 2-5 all modify the same file (`skills/changelog/SKILL.md`), building it up incrementally
- Each task adds a distinct section — they must be done in order
- The skill is pure markdown instruction — no scripts to test, no code to compile
- Validation in Task 6 is a manual review pass against the Common Changelog spec and Anthropic's skill best practices
