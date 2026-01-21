---
name: release-process
description: >-
  Plan and execute semantic versioning releases using bracket-based freezing. Use when: planning a release, bumping versions, writing changelogs, coordinating multi-component projects, handling scope creep, or releasing plugins. Works for single projects, monorepos, plugins, and multi-component systems.
version: 1.1.0
allowed-tools: Read,Write,Edit,Bash(git:*),Grep,Glob
---

# Release Process Guidance

Execute semantic versioning releases with bracket-based version freezing. This skill helps prevent version thrashing and coordinate releases across single projects, monorepos, plugins, and multi-component systems.

## Quick Start: 6-Step Release Process

Follow these steps in order. Use references for complex scenarios (scope creep, multi-component, breaking changes).

### Step 1: Determine Scope

What's changing? Pick ONE:

- **Bug fixes, docs, wording only** → PATCH (bump Z: 1.0.2 → 1.0.3)
- **New features (backward compatible)** → MINOR (bump Y, reset Z: 1.0.5 → 1.1.0)
- **Breaking changes (incompatible)** → MAJOR (bump X, reset Y,Z: 1.5.2 → 2.0.0)

⚠️ **If uncertain:** Run `git diff` to see exact changes. Don't rely on memory.

### Step 2: Find Current Version

```bash
# Single-component (pick one):
grep '"version"' package.json
grep 'version:' SKILL.md frontmatter
grep 'version =' Cargo.toml

# Multi-component:
grep -r '"version"' .
```

### Step 3: Apply Bracket Rules

**Core principle:** Version freezes at scope level until commit. Can jump up, never down.

| If Currently At | And Discover | Do This | Example |
|---|---|---|---|
| 1.0.1 (fresh) | Patch work | Bump Z | 1.0.1 → 1.0.2 |
| 1.0.1 (fresh) | Minor work | Jump to Y+1, Z=0 | 1.0.1 → 1.1.0 |
| 1.0.1 (fresh) | Major work | Jump to X+1, Y=0, Z=0 | 1.0.1 → 2.0.0 |
| 1.0.2 (patch frozen) | Discover minor | Jump to minor | 1.0.2 → 1.1.0 |
| 1.1.0 (minor frozen) | Discover major | Jump to major | 1.1.0 → 2.0.0 |
| 2.0.0 (major frozen) | More work | **Don't bump again** | 2.0.0 (stays) |

**Why frozen?** Prevents version thrashing. One release = one version bump.

### Step 4: Update All Version Files

Find and update EVERY file containing the version:

```bash
# Find all version locations
grep -r "version" . --include="*.json" --include="*.toml" --include="*.py" --include="*.md"
```

**Common locations:**
- `package.json` (Node.js) — `"version": "X.Y.Z"`
- `Cargo.toml` (Rust) — `version = "X.Y.Z"`
- `SKILL.md` (Claude) — `version: X.Y.Z` in frontmatter
- Root `plugin.json` (plugins) — `"version": "X.Y.Z"`

**Multi-component rule:** If your project has children (skills, services), parent version ≥ highest child version.

⚠️ **Error prevention:** After updating, run `grep -r "OLD_VERSION" .` to catch any you missed.

### Step 5: Update CHANGELOG.md

Add entry at the TOP (under `## [Unreleased]`):

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New feature users can access

### Changed
- Existing behavior modified (backward compatible)

### Fixed
- Bug that was broken

### Removed
- Feature that's gone
```

**Four critical rules (non-negotiable):**

1. ⚠️ **Only document changes users see** — Skip internal refactoring, code cleanup, optimization
2. ⚠️ **NEVER say "bumped version"** — NO "Bumped to 1.2.3", NO "Updated version", NO "skill-x: 1.0.0→1.0.1"
3. ⚠️ **Verify each change exists** — Run `git diff` and only document what's actually there
4. ⚠️ **Be specific** — "Reduced search from 500ms→50ms" not "Improved performance"

### Step 6: Commit and Tag

```bash
# Final verification
git status           # See what's staged
git diff             # See exact changes
head -20 CHANGELOG.md # Verify entries

# Stage only version files and changelog
git add package.json Cargo.toml SKILL.md plugin.json CHANGELOG.md

# Commit (DO NOT use --amend unless asked)
git commit -m "Release version X.Y.Z

- Added: What users can now do
- Fixed: What was broken and is fixed
- Changed: What works differently
"

# Tag and push
git tag vX.Y.Z
git push origin vX.Y.Z  # if using remote
```

⚠️ **Error prevention:** If you realize you forgot a file AFTER committing, create a NEW commit (don't amend).

## Multi-Component Projects (Monorepos, Plugins)

**Each component versions independently. Parent version ≥ highest child.**

Quick rule: Only update components that changed.

**Example:**
```
Before: plugin 1.2.0, skill-a 1.0.0, skill-b 1.0.0
Change: skill-a gets new feature only
After:  plugin 1.3.0, skill-a 1.1.0, skill-b 1.0.0 (unchanged)

Why plugin jumped to 1.3.0? Because skill-a is now 1.1.0,
and parent must be ≥ highest child.
```

See `references/project-structures.md` for detailed multi-component examples.

## Common Release Patterns

### Pattern 1: Patch Release (Bug Fix + Docs)

```
Current: 1.2.3 → New: 1.2.4
Changes: Fix bug, update docs
Type: Patch (Z only)

CHANGELOG:
## [1.2.4] - 2026-01-21
### Fixed
- Fix timeout in database queries
- Fix typo in README
```

### Pattern 2: Feature Release (New Capability)

```
Current: 1.2.3 → New: 1.3.0
Changes: New encryption support
Type: Minor (Y+1, Z→0)

CHANGELOG:
## [1.3.0] - 2026-01-22
### Added
- Support for encrypted documents
```

### Pattern 3: Breaking Release (API Change)

```
Current: 1.5.2 → New: 2.0.0
Changes: Rename command (breaking)
Type: Major (X+1, Y→0, Z→0)

CHANGELOG:
## [2.0.0] - 2026-01-23
### Changed
- **BREAKING**: Renamed 'validate' to 'verify'
### Removed
- Old 'validate' command (use 'verify')
```

### Pattern 4: Scope Creep (Multiple Jumps)

```
Discoveries:
1. Bug fix detected → 1.2.1 (patch frozen)
2. Need new feature → 1.3.0 (jump to minor)
3. Need breaking change → 2.0.0 (jump to major)
4. Find another bug → 2.0.0 (stay major)

Final: 2.0.0 with all changes documented
See references/workflows.md for details
```

## Pre-Release Versions

Use format `X.Y.Z-IDENTIFIER` for testing before stable release:

- `2.0.0-alpha.1` — Early testing
- `2.0.0-beta.1` — More stable
- `2.0.0-rc.1` — Release candidate
- `2.0.0` — Stable

## Critical Rules (Don't Violate These)

| Rule | Why | Example |
|------|-----|---------|
| **One bump per cycle** | Prevents version thrashing | Don't go 1.0.1 → 1.0.2 → 1.1.0 in same release |
| **Never amend releases** | Clear git history for teams | Create new commit if you forgot a file |
| **Don't document version bumps** | Changelogs are for users, not admins | ❌ "Bumped to 1.2.3" ✅ "Fixed timeout bug" |
| **Verify against git diff** | Don't document from memory | Run `git diff` before writing CHANGELOG |
| **Parent ≥ child versions** | Prevents version confusion | If child is 1.1.0, parent must be ≥1.1.0 |

## When to Use References

- **Bracket System Deep Dive:** `references/bracket-system.md` — Understand state transitions and scope creep
- **Changelog Best Practices:** `references/changelog-guidelines.md` — Learn all sections and common mistakes
- **Project Patterns:** `references/project-structures.md` — Single-component to complex monorepos
- **Step-by-Step Workflows:** `references/workflows.md` — Detailed procedures for all scenarios
