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
   - If the user provided a version via `/changelog` or argument, use that
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
   - Use the commit subject and body to determine the category
   - `feat`/`add` → Added, `fix` → Fixed, `remove`/`deprecate` → Removed, everything else → Changed
2. **Format** each entry in imperative mood:
   - Start with a verb: Add, Fix, Remove, Refactor, Bump, Update, etc.
   - Keep to one line when possible
   - Append references: `([#123](url))` for PRs, `([`​`abc1234`​`](url))` for commits
   - Append author name in parentheses: `(Author Name)`
3. **Sort within each group:**
   - Breaking changes first (prefixed with `**Breaking:**`)
   - Then by importance (user-facing before internal)
   - Then newest-first
4. **Omit empty groups** — only include groups that have entries

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

- **Imperative mood:** start with Add, Fix, Remove, Refactor, Bump, Update, etc.
- **Self-describing:** entry should make sense without reading the group heading
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
- Refactor internal queue to improve throughput ([`a1b2c3d`](https://github.com/owner/repo/commit/a1b2c3d))

### Added

- Add `--dry-run` flag to preview changes ([#38](https://github.com/owner/repo/pull/38)) (Bob Badger)

### Fixed

- Fix crash when input file is empty ([#41](https://github.com/owner/repo/pull/41))

[1.2.0]: https://github.com/owner/repo/releases/tag/v1.2.0
````
