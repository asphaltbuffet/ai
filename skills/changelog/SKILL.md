---
name: changelog
description: Create or update CHANGELOG.md following the Common Changelog format (common-changelog.org). Use when user asks to create, update, review, or add entries to a changelog. Also use when detecting release-related work like version bumps, tagging, or preparing release notes.
argument-hint: [version]
allowed-tools: Read, Grep, Glob, Bash(git *)
---

# Changelog

Generate and maintain `CHANGELOG.md` files following the [Common Changelog](https://common-changelog.org/) format by analyzing git history.

## When to Use

- User invokes `/changelog` or `/changelog 1.2.0`
- User asks to update the changelog, add a changelog entry, prepare release notes, or review the changelog
- Release-related work detected (version bumps, tagging)

## Prerequisites

- Project under git version control
- Semantic versioning with git tags (e.g., `v1.0.0` or `1.0.0`)

## Workflow

Follow these steps in order. Present results to the user for review before writing any files.

### Step 1: Detect State

1. Check if `CHANGELOG.md` exists at the repository root
2. List all git tags sorted by semver:
   ```bash
   git tag --list --sort=-v:refname
   ```
3. Determine the mode:
   - **No CHANGELOG.md exists** → creation mode (generate entries for all tagged releases)
   - **CHANGELOG.md exists** → append mode (generate entry for changes since the last tagged release)
4. Identify the target version:
   - If the user provided a version argument (e.g., `/changelog 1.2.0`), use that
   - Otherwise, ask the user what version this release will be

### Step 2: Gather Changes

For each release being generated, collect commits between the relevant tags:

```bash
git log <previous-tag>..<current-tag> --format="%H" --reverse
```

For unreleased changes (append mode):

```bash
git log <latest-tag>..HEAD --format="%H" --reverse
```

For each commit SHA, extract details:

```bash
git log -1 --format="%H%n%an%n%s%n%b" <sha>
```

Extract from each commit:
- **Subject line** (first line of commit message)
- **Body** (remaining lines)
- **Author name** (`%an`)
- **PR/issue references** — look for patterns like `(#123)`, `Fixes #456`, `Closes #789` in subject and body
- **Commit SHA** (short form, first 7 characters) as fallback reference

Determine the remote URL for constructing links:

```bash
git remote get-url origin
```

Convert to HTTPS base URL for linking commits, PRs, and releases.

### Step 3: Draft Entry

For each change, apply curation rules (see Format Rules section below), then:

1. **Categorize** each change into one of: `Changed`, `Added`, `Removed`, `Fixed`
   - Read the commit subject and body to understand the *impact*, not just the prefix
   - Do not rely on conventional commit prefixes (`feat:`, `fix:`) — categorize by actual effect
2. **Rephrase** commit messages for the changelog audience:
   - Do not copy commit messages or PR titles verbatim
   - Focus on consumer impact, not implementation process
   - Align terminology consistently across entries
   - For dependency bumps, use rounded ranges: `Bump `dep` from 2.x to 3.x` (not `2.2.0 to 3.0.1`)
3. **Format** each entry in imperative mood:
   - Start with a present-tense verb: Add, Fix, Remove, Refactor, Bump, Document, Deprecate, Support, Drop, Enable, Prevent, Clarify, Use
   - Each entry must be self-describing — it must read as a complete action independent of its group heading
   - Keep to one line when possible
   - Append references: `([#123](url))` for PRs, `([`​`abc1234`​`](url))` for commits
   - Append author name in parentheses: `(Author Name)`
4. **Sort within each group:**
   - Breaking changes first (prefixed with `**Breaking:**`)
   - Then by importance (user-facing before internal)
   - Then newest-first
5. **Omit empty groups** — only include groups that have entries

Assemble the entry:

```markdown
## [VERSION] - YYYY-MM-DD

### Changed

- Entry here ([#ref](url)) (Author)

### Added

- Entry here ([#ref](url)) (Author)
```

### Step 4: Present for Review

Display the drafted changelog entry to the user in a fenced markdown code block. Ask:

- Are the categories correct?
- Should any entries be reworded, merged, or removed?
- Is anything missing?

Wait for user approval before proceeding. Incorporate any requested changes.

### Step 5: Write

**For new CHANGELOG.md (creation mode):**

Create the file starting with `# Changelog`, followed by all release entries newest-first, with reference-style links at the bottom:

```markdown
# Changelog

## [1.1.0] - 2026-01-15

...

## [1.0.0] - 2025-12-01

_First release._

[1.1.0]: https://github.com/owner/repo/releases/tag/v1.1.0
[1.0.0]: https://github.com/owner/repo/releases/tag/v1.0.0
```

**For existing CHANGELOG.md (append mode):**

Insert the new entry after the `# Changelog` heading and before the first existing `## [` release heading. Add the new reference-style link alongside existing ones at the bottom of the file.

**Reference link format:**

```markdown
[VERSION]: https://github.com/owner/repo/releases/tag/vVERSION
```

If the GitHub release does not yet exist, link to the tag comparison instead:

```markdown
[VERSION]: https://github.com/owner/repo/compare/vPREVIOUS...vVERSION
```

## Format Rules

These rules follow the [Common Changelog](https://common-changelog.org/) specification.

### File Structure

- File starts with a first-level heading: `# Changelog`
- Each release is a second-level heading: `## [VERSION] - YYYY-MM-DD`
  - VERSION: semver-valid, no `v` prefix (even if git tag has one)
  - DATE: ISO 8601 format (`YYYY-MM-DD`)
  - VERSION should be a markdown link using reference-style links
- Releases sorted newest-first
- Each release contains one or more group headings

### Group Order

Use only these third-level headings, in this order. Omit groups with no entries.

1. `### Changed` — changes to existing functionality
2. `### Added` — new functionality
3. `### Removed` — removed functionality
4. `### Fixed` — bug fixes

### Entry Format

Each entry is an unnumbered list item:

```
- Imperative verb description ([references]) (Authors)
```

- **Imperative mood:** start with a present-tense verb: Add, Fix, Remove, Refactor, Bump, Document, Deprecate, Support, Drop, Enable, Prevent, Clarify, Use
- **Self-describing:** each entry must read as a complete action, not a fragment dependent on its group heading
  - Bad: `Support of CentOS` or `\`write()\` method`
  - Good: `Support CentOS` or `Add \`write()\` method`
- **References:** PR links `([#123](url))`, commit links `([`​`abc1234`​`](url))`, or issue links
  - Multiple refs of the same type in one set of parentheses: `(#1, #2)` not `(#1) (#2)`
- **Authors:** after references, in parentheses: `(Alice Meerkat)` or `(Alice, Bob)`
  - Optional for single-contributor projects
- **Breaking changes:** prefix with `**Breaking:**`, list before non-breaking entries in the group

### Curation Rules

**Exclude** (maintenance noise):
- Dotfile changes (`.gitignore`, `.github/`, `.gitlab/`)
- Development-only dependency updates
- Minor code style changes
- Documentation formatting changes

**Include** (consumer-relevant):
- Refactorings (potential unintended side effects)
- Runtime environment changes
- New documentation for previously undocumented features
- Code style changes using new language features

**Rephrase** for consistency and clarity:
- Do not copy `git log` output or PR titles verbatim — curate and contextualize
- Align terminology across entries from different contributors
- Adjust specificity: `Bump \`json-parser\` from 2.x to 3.x` (not exact patch versions)
- Focus on what upgrading will do, not on the development process

**Merge** related multi-commit changes into single entries with combined references.

**Skip** no-op changes where commits negate each other (e.g., a change followed by its revert).

### Notices

A notice is a single italic sentence before any change groups. Maximum one per release.

Use for:
- First release: `_First release._`
- Upgrade guidance: `_If you are upgrading: please see [`UPGRADING.md`](UPGRADING.md)._`
- Yanked releases: `_This release was yanked due to [reason]._`

### Reference Links

Place reference-style link definitions at the bottom of the file:

```markdown
[1.2.0]: https://github.com/owner/repo/releases/tag/v1.2.0
[1.1.0]: https://github.com/owner/repo/releases/tag/v1.1.0
```

## Example

A properly formatted changelog entry:

````markdown
## [1.2.0] - 2026-02-16

### Changed

- **Breaking:** rename `process()` to `run()` for consistency ([#42](https://github.com/owner/repo/pull/42)) (Alice Meerkat)
- Bump `json-parser` from 2.x to 3.x ([#40](https://github.com/owner/repo/pull/40))
- Refactor internal queue to improve throughput ([`a1b2c3d`](https://github.com/owner/repo/commit/a1b2c3d))

### Added

- Add `--dry-run` flag to preview changes ([#38](https://github.com/owner/repo/pull/38)) (Bob Badger)

### Fixed

- Fix crash when input file is empty ([#41](https://github.com/owner/repo/pull/41))

[1.2.0]: https://github.com/owner/repo/releases/tag/v1.2.0
````
