# dev-flow Plugin

Comprehensive release management and versioning guidance for Claude Code projects.

## Overview

**dev-flow** helps teams manage project releases using semantic versioning with bracket-based version freezing. Works with single-component projects, multi-component systems, monorepos, and Claude plugins.

## What It Does

The **release-process** skill guides you through:

- **Assessing release scope** — Determine if changes are PATCH, MINOR, or MAJOR
- **Applying bracket system** — Freeze at scope level, jump higher if scope creeps
- **Version bumping** — Correctly increment version numbers across all files
- **Changelog management** — Document user-facing changes only (no version bump documentation)
- **Multi-component coordination** — Manage independent versions with parent version rules
- **Release workflows** — Step-by-step procedures for common scenarios

## Installation

```bash
claude plugin install dev-flow@marketplace
```

Or install from GitHub:

```bash
claude plugin install full-stack-biz/claude-dev-flow
```

For local development:

```bash
claude --plugin-dir /path/to/claude-dev-flow
```

## Quick Start

When planning a release, ask Claude:

> "Help me release version X.Y.Z for my project"

or

> "Guide me through the release process using the bracket system"

The plugin will:
1. Help assess the scope of changes
2. Apply bracket-based version freezing rules
3. Guide you through updating version files
4. Ensure your changelog follows best practices
5. Help you commit and tag the release

## Key Concepts

### Bracket System

Prevents version thrashing by freezing at scope level:

- **PATCH bracket** (Z) — Bug fixes, docs, wording
- **MINOR bracket** (Y) — New features, backward compatible
- **MAJOR bracket** (X) — Breaking changes

**Core principle:** Version freezes at bracket level until commit. Can jump to higher bracket if scope demands it. After commit, reset for next cycle.

### Version Freezing

```
Last commit: 1.2.0
Start patch work → 1.2.1 (patch frozen)
Discover new feature → 1.3.0 (minor frozen, jumped brackets)
Discover breaking change → 2.0.0 (major frozen, jumped brackets)
More work discovered → 2.0.0 (major, don't bump further)
Commit: 2.0.0
```

### Changelog Rules

1. **Document user-facing changes only** — Skip internal refactoring, cleanup
2. **Never document version bumps** — NO "Bumped to 1.2.3"
3. **Verify against git diff** — Run `git diff` before writing changelog
4. **Be specific** — "Reduced search time from 500ms to 50ms" not "Improved performance"

### Multi-Component Projects

In monorepos and plugins:
- Each component versions independently
- Parent version ≥ highest child version
- Only update components that changed

## Project Structure

```
dev-flow/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   └── release-process/
│       ├── SKILL.md             # Main skill guidance
│       └── references/
│           ├── bracket-system.md
│           ├── changelog-guidelines.md
│           ├── project-structures.md
│           └── workflows.md
├── CHANGELOG.md
├── README.md
└── RELEASE_PROCESS.md           # Original documentation
```

## Skills

### release-process

Comprehensive guide to semantic versioning releases with bracket-based freezing.

**Use when:**
- Planning or executing a project release
- Bumping version numbers
- Writing or reviewing changelogs
- Managing multi-component project versions
- Handling scope creep during development

**Includes:**
- Quick start (6-step release workflow)
- Bracket system explanation with state transitions
- Changelog best practices and common mistakes
- Multi-component version coordination
- Real-world scenario examples
- Detailed reference guides

## Common Scenarios

### Simple Patch Release

```bash
# Bug fix + doc update
# Current: 1.2.3 → New: 1.2.4

Ask: "Help me release a patch version with bug fix and documentation update"
```

### Feature Release

```bash
# New feature
# Current: 1.2.3 → New: 1.3.0 (MINOR, reset Z)

Ask: "I added a new feature, help me release 1.3.0"
```

### Breaking Change Release

```bash
# API changed incompatibly
# Current: 1.5.2 → New: 2.0.0 (MAJOR, reset Y,Z)

Ask: "I made breaking changes to the API, help me release 2.0.0"
```

### Scope Creep

```bash
# Multiple bracket jumps in one cycle
# 1.2.0 → patch (1.2.1) → minor (1.3.0) → major (2.0.0)

Ask: "I found multiple issues while working. Guide me through releasing with scope creep"
```

## Key Features

✅ **Bracket System** — Prevents version thrashing with intelligent scope management
✅ **Multi-Component Support** — Handles monorepos, plugins, and multi-service projects
✅ **Changelog Best Practices** — Critical rules for documenting changes correctly
✅ **Reference Guides** — Comprehensive documentation for complex scenarios
✅ **Real-World Examples** — Common patterns with copy-paste solutions
✅ **Git Integration** — Verifies changes before documenting
✅ **Team Ready** — Supports team coordination and selective updates

## Troubleshooting

### "My version numbers don't align"

Check the bracket rules — Parent version should be ≥ highest child version.

### "I documented a version bump in changelog"

Remove it! Changelogs document **changes**, not **version management**. The version is in the header: `## [1.2.3] - 2026-01-21`

### "I forgot to update a version file"

Don't amend the commit. Create a new commit to fix it.

### "Should I release a pre-release version?"

Use pre-releases (`2.0.0-alpha.1`, `2.0.0-beta.1`, `2.0.0-rc.1`) when testing major changes with team before stable release.

## Contributing

To improve this plugin:

1. Follow the release process documented in the skill
2. Update version in `.claude-plugin/plugin.json`
3. Update `CHANGELOG.md` with your changes
4. Create a commit with the version bump
5. Tag the release: `git tag v1.0.0`

## License

MIT

## Support

For issues, questions, or suggestions, please open an issue or contact the development team.

---

**Made with ❤️ for better release management in Claude Code**
