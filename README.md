# dev-flow Plugin

Comprehensive release management and versioning guidance for Claude Code projects.

## Overview

**dev-flow** provides comprehensive release and git workflow management for teams. It guides semantic versioning with bracket-based freezing, branch strategy coordination, and systematic rebase conflict resolution. Works with single-component projects, multi-component systems, monorepos, and Claude plugins.

## What It Does

### Release Management

The **release-process** skill guides you through:

- **Assessing release scope** — Determine if changes are PATCH, MINOR, or MAJOR
- **Applying bracket system** — Freeze at scope level, jump higher if scope creeps
- **Version bumping** — Correctly increment version numbers across all files
- **Changelog management** — Document user-facing changes only (no version bump documentation)
- **Multi-component coordination** — Manage independent versions with parent version rules
- **Release workflows** — Step-by-step procedures for common scenarios

### Branch Strategy

The **git-flow** and **git-flow-next** skills help you:

- **Choose your workflow** — Traditional git-flow for all platforms, or modern git-flow-next for macOS
- **Set up branching** — Initialize and configure feature/release/hotfix branches
- **Coordinate releases** — Integrate with release-process for semantic versioning
- **Manage hotfixes** — Handle production emergencies without disrupting development
- **Support team workflows** — Multiple concurrent features, explicit dev/prod separation
- **Customize strategies** — git-flow-next supports GitHub Flow, GitLab Flow, and custom presets

### Git Rebase Conflict Resolution

The **rebase-resolver** skill helps you:

- **Resolve conflicts one block at a time** — Systematic approach avoiding bulk operations
- **Verify before committing** — Grep verification prevents incomplete resolutions
- **Leverage git rerere** — Block-by-block resolution enables conflict pattern learning
- **Handle edge cases** — Nested markers, AA conflicts, and shifted line numbers
- **Continue rebases safely** — Ensure all conflicts resolved before proceeding

## Installation

```bash
/plugin marketplace add full-stack-biz/claude-dev-flow
/plugin install dev-flow@dev-flow-marketplace
```

Or directly from GitHub:

```bash
claude plugin install https://github.com/full-stack-biz/claude-dev-flow --scope user
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
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Marketplace listing
├── skills/
│   ├── release-process/
│   │   ├── SKILL.md             # Release management guidance
│   │   └── references/
│   │       ├── bracket-system.md
│   │       ├── changelog-guidelines.md
│   │       ├── project-structures.md
│   │       └── workflows.md
│   ├── git-flow/
│   │   ├── SKILL.md             # Traditional git-flow (cross-platform)
│   │   └── references/
│   │       └── multi-component-coordination.md
│   ├── git-flow-next/
│   │   ├── SKILL.md             # Modern git-flow (macOS only)
│   │   └── references/
│   │       ├── installation.md
│   │       ├── merge-strategies.md
│   │       ├── advanced-configuration.md
│   │       ├── workflows.md
│   │       └── presets-and-configuration.md
│   └── rebase-resolver/
│       ├── SKILL.md             # Rebase conflict resolution
│       └── references/
│           ├── rerere.md
│           └── troubleshooting.md
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

### git-flow

Traditional git-flow branching strategy for multi-feature team development.

**Use when:**
- Setting up explicit feature/release/hotfix branching
- Working with teams on multiple concurrent features
- Maintaining separate development and production branches
- Coordinating releases with semantic versioning
- Cross-platform support needed (macOS, Linux, Windows)

**Includes:**
- Git-flow initialization and setup
- Feature branch workflow (start/finish/delete)
- Release preparation and versioning
- Hotfix management for production issues
- Integration with release-process skill
- Multi-component coordination rules

### git-flow-next

Modern, Go-based git-flow implementation with flexible workflow presets.

**Use when:**
- On macOS and prefer modern tooling
- Need support for multiple workflow styles (Classic GitFlow, GitHub Flow, GitLab Flow)
- Want customizable merge strategies (merge/rebase/squash)
- Managing complex branching configurations
- Need advanced features like base branch auto-update

**Includes:**
- Preset workflows for different team structures
- Customizable branch types and merge strategies
- Configuration management and advanced options
- Support for multiple base branches
- Workflow comparison and strategy guidance

### rebase-resolver

Systematic git rebase conflict resolution with block-by-block verification.

**Use when:**
- Resolving merge conflicts during a rebase
- Continuing an interrupted rebase
- Handling AA conflicts (both added) or other conflict types
- Learning conflict patterns with git rerere
- Ensuring all conflicts resolved before committing

**Includes:**
- Block-by-block resolution methodology
- Mandatory verification rules (grep before staging)
- Git rerere integration and setup
- Edge case handling (nested markers, AA conflicts, line shifts)
- Troubleshooting guide for common issues

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

### Release & Versioning
✅ **Bracket System** — Prevents version thrashing with intelligent scope management
✅ **Multi-Component Support** — Handles monorepos, plugins, and multi-service projects
✅ **Changelog Best Practices** — Critical rules for documenting changes correctly
✅ **Semantic Versioning** — Complete guidance for MAJOR.MINOR.PATCH decisions

### Branch Strategy
✅ **Multiple Workflows** — Traditional git-flow + modern git-flow-next (macOS)
✅ **Cross-Platform Support** — git-flow works on macOS, Linux, and Windows
✅ **Feature/Release/Hotfix** — Complete branching model for team development
✅ **Flexible Configuration** — Customizable branch types, prefixes, and merge strategies
✅ **Workflow Presets** — GitHub Flow, GitLab Flow, and Classic GitFlow templates

### Git Operations
✅ **Systematic Conflict Resolution** — Block-by-block approach prevents bulk operation mistakes
✅ **Verification Rules** — Mandatory grep checks ensure completeness
✅ **Rerere Integration** — Enables conflict pattern learning for future reuses
✅ **Reference Guides** — Comprehensive documentation for complex scenarios
✅ **Team Ready** — Works with all project types and team sizes

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
