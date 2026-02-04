# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dev-flow** is a Claude Code plugin that provides comprehensive release management and semantic versioning guidance. It helps teams manage releases using bracket-based version freezing for single-component projects, multi-component systems, monorepos, and Claude plugins.

The project consists of:
- **release-process skill** — Main guidance for planning and executing semantic versioning releases
- **Reference guides** — Detailed documentation for bracket system, changelog best practices, multi-component projects, and workflows

## Key Architecture

### Skill-Based Plugin Structure

This is a Claude plugin with a single skill (`release-process`) that guides users through release workflows. The skill:

1. **Prevents version thrashing** — Uses bracket-based freezing to prevent multiple version bumps in one cycle
2. **Supports multiple project types** — Single projects, monorepos, multi-service systems, and plugins
3. **Enforces changelog best practices** — Critical rules about what to document (user-facing changes only, never version bumps)
4. **Coordinates multi-component versions** — Parent version must be ≥ highest child version

### Bracket System (Core Concept)

The bracket system freezes version numbers at scope level:

- **PATCH bracket (Z)** — Bug fixes, documentation, wording only → 1.2.3 → 1.2.4
- **MINOR bracket (Y)** — New features (backward compatible) → 1.2.3 → 1.3.0
- **MAJOR bracket (X)** — Breaking changes → 1.5.2 → 2.0.0

**Critical principle:** Once a version is bumped to a bracket level, it stays frozen at that level for the entire release cycle. Can jump UP to higher brackets if scope demands it, but never down. After commit, the bracket resets for the next cycle.

### Version Freezing in Action

```
Start: 1.0.1 (fresh, no bracket frozen)
Discover: Bug fix → 1.0.2 (patch frozen)
Discover: New feature → 1.1.0 (jump to minor, patch reset to 0)
Discover: Breaking change → 2.0.0 (jump to major, minor and patch reset)
More work: Bug fixes → 2.0.0 (major frozen, don't bump again)
Commit: 2.0.0 (all changes under one version)
```

### Multi-Component Projects

In monorepos and plugins with multiple components:
- Each component has its own version
- Only update components that changed
- Parent version must be ≥ highest child version

Example: If plugin has skill-a and skill-b, and skill-a gets a minor update (1.0.0 → 1.1.0), the plugin must jump from 1.2.0 → 1.3.0 to stay ≥ 1.1.0.

## Working with Release Process Skill

### Quick Release Workflow (6 Steps)

1. **Determine scope** — Assess changes: PATCH (fixes/docs), MINOR (features), or MAJOR (breaking)
2. **Find current version** — Check `plugin.json`, skill `SKILL.md` frontmatter, etc.
3. **Apply bracket rules** — Use the table in SKILL.md to determine new version
4. **Update ALL version files** — Search for and update every occurrence of the version number
5. **Update CHANGELOG.md** — Document user-facing changes only in new section at top
6. **Commit and tag** — Commit with descriptive message, tag the release, push

### Common Version Locations in This Project

- `.claude-plugin/plugin.json` — Root plugin version in `"version"` field
- `skills/release-process/SKILL.md` — Skill version in frontmatter `version:` field

### Changelog Rules (Non-Negotiable)

1. **Only document changes users see** — Skip internal refactoring, code cleanup, optimization
2. **Never document version bumps** — NO "Bumped to 1.2.3", NO "Updated version from 1.0.0 to 1.1.0"
3. **Verify against git diff** — Run `git diff` before writing changelog, don't rely on memory
4. **Be specific** — "Reduced search from 500ms to 50ms" not just "Improved performance"

**Changelog format:**
```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New feature description

### Changed
- Existing behavior modification

### Fixed
- Bug fix description

### Removed
- Feature that's no longer available
```

## Project Files

- **README.md** — Public overview, installation instructions, quick start guide
- **CHANGELOG.md** — Semantic versioning changelog with bracket system examples
- **plugin.json** — Plugin manifest with name, version, description, keywords
- **skills/release-process/SKILL.md** — Main skill guidance (6-step process, bracket rules, changelog rules)
- **skills/release-process/references/** — Deep reference guides:
  - `bracket-system.md` — State transitions, scope creep examples
  - `changelog-guidelines.md` — Detailed changelog best practices and mistakes
  - `project-structures.md` — Examples for different project types
  - `workflows.md` — Step-by-step procedures for complex scenarios

## Development Guidelines

### When Making Changes

1. **Update relevant documentation** — If changing release guidance, update SKILL.md and/or reference guides
2. **Follow your own advice** — Use the release-process skill when releasing a new version
3. **Version coordinates** — Keep plugin.json and skill SKILL.md versions in sync (or follow multi-component rules)
4. **Test references** — Ensure real-world examples and scenario walkthroughs still match the guidance

### Git Practices

- **NEVER commit anything.** Wait for explicit operator request for each commit.
- When operator requests a commit, use descriptive messages that match the changelog
- Tag releases with format: `vX.Y.Z` (only on operator request)
- Ensure changelog is committed with version files in same commit (only on operator request)

### Content Style

- Reference guides should be detailed with real-world examples
- Skill guidance should be step-by-step and actionable
- Critical rules should be clearly marked (⚠️ symbol, bold text, tables)
- Include state transition examples for complex concepts like scope creep
