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
