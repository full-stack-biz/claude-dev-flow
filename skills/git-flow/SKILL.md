---
name: git-flow
description: >-
  Initialize git-flow, manage feature/release/hotfix branches, and coordinate multi-component releases. Use when setting up branching strategy, starting/finishing branches, or releasing with semantic versioning. Integrates with release-process skill.
version: 1.0.0
allowed-tools: Bash(git:*),Read,Write
---

# Git-Flow Branching Skill

Git-flow is a structured branching model that organizes parallel development with dedicated branch types. This skill guides setup, branch operations, and integration with semantic versioning.

## Quick Start

**Branch types:**
- `main` — Production, always deployable
- `develop` — Integration, pre-release
- `feature/*` — New features (from develop)
- `release/*` — Release prep (version bump)
- `hotfix/*` — Production fixes (from main)

**When to use:** Team with multiple concurrent features, explicit dev/prod separation, or semantic versioning.

## Setup: Initialize Git-Flow

### Step 1: Install git-flow (if not installed)

**macOS (Homebrew):**
```bash
brew install git-flow
```

**Linux (apt):**
```bash
sudo apt-get install git-flow
```

**Windows (Git Bash):**
```bash
# Option A: Download installer from github.com/nvie/gitflow
# Option B: Via Chocolatey
choco install gitflow
```

**Verify installation:**
```bash
git flow version
```

### Step 2: Initialize git-flow in your repository

```bash
git flow init
```

You'll be prompted to confirm branch names:
- Production branch (default: `main`)
- Development branch (default: `develop`)
- Feature branch prefix (default: `feature/`)
- Release branch prefix (default: `release/`)
- Hotfix branch prefix (default: `hotfix/`)

Press Enter to accept defaults unless your team has different conventions.

### Step 3: Verify initialization

```bash
# Check branches were created
git branch -a

# Expected output:
# * develop
#   main
```

## Branch Operations: Step-by-Step

### Starting a Feature

```bash
git flow feature start my-feature-name
```

This creates and checks out `feature/my-feature-name` from `develop`.

**What happens:**
- Branch created from `develop`
- You're automatically on the new branch
- Make commits as normal
- Other developers can see the branch and understand its purpose

**Convention:** Use lowercase hyphen-separated names: `feature/user-authentication`, `feature/payment-integration`

### Finishing a Feature

```bash
git flow feature finish my-feature-name
```

This:
1. Merges `feature/my-feature-name` into `develop`
2. Deletes the feature branch
3. Switches back to `develop`
4. You're ready for the next feature or release

**Before finishing:**
- Push feature branch so team can review: `git push origin feature/my-feature-name`
- Create pull request, get reviews, fix issues
- After approval, run finish command

### Starting a Release

Use this when you're ready to prepare a release version.

```bash
git flow release start 1.2.0
```

This creates `release/1.2.0` from `develop`.

**In the release branch:**
- Update version numbers in all files
- Update CHANGELOG.md
- Fix only critical bugs (no new features)
- Commit all release-preparation changes

**Coordinate with release-process skill:**
- Run `release-process` skill first to determine version (MAJOR.MINOR.PATCH)
- Use that version when creating release branch: `git flow release start X.Y.Z`

### Finishing a Release

```bash
git flow release finish 1.2.0
```

This:
1. Merges `release/1.2.0` into `main`
2. Tags `main` with version number (`v1.2.0`)
3. Merges `release/1.2.0` back into `develop` (keeps versions in sync)
4. Deletes the release branch
5. Switches to `develop`

**After finishing:**
- Push all branches and tags: `git push origin main develop --tags`
- Deploy `main` branch to production
- Verify deployment succeeded

### Starting a Hotfix

Use for urgent production bugs.

```bash
git flow hotfix start 1.2.1
```

This creates `hotfix/1.2.1` from `main` (not `develop`—critical!).

**In the hotfix branch:**
- Fix the bug
- Update version to next PATCH: 1.2.0 → 1.2.1
- Update CHANGELOG.md
- Commit changes

**Version rule:** Hotfix version = last release version + PATCH bump
- Last release: 1.2.0 → Hotfix: 1.2.1
- Last release: 2.3.5 → Hotfix: 2.3.6

### Finishing a Hotfix

```bash
git flow hotfix finish 1.2.1
```

This:
1. Merges `hotfix/1.2.1` into `main`
2. Tags `main` with version (`v1.2.1`)
3. Merges `hotfix/1.2.1` back into `develop` (keeps versions in sync)
4. Deletes the hotfix branch

**After finishing:**
- Push branches and tags: `git push origin main develop --tags`
- Deploy `main` immediately to production
- The hotfix is now integrated into `develop` for next release

## Multi-Component Projects

For projects with multiple components (plugin + skills, monorepos):

**Rule:** Parent version ≥ max(child versions)

See `references/multi-component-coordination.md` for detailed release strategies, version coordination matrices, and multi-component examples.

## Common Patterns & Troubleshooting

### Push Feature Before Finishing

Let your team review the branch before merging:

```bash
git push origin feature/my-feature-name
# Create pull request on GitHub
# After approval, run:
git flow feature finish my-feature-name
```

### Cancel a Feature (Delete Without Merging)

```bash
git flow feature delete my-feature-name
```

### List All Branches

```bash
git flow feature list     # In-progress features
git flow release list     # Pending releases
git flow hotfix list      # Active hotfixes
```

### If Release Conflicts During Merge

```bash
# Resolve conflicts in editor
git add .
git commit -m "Resolve merge conflicts"
# Finish command will complete the merge
git flow release finish 1.2.0
```

### Sync With Team

If teammates updated `develop` while you were on a feature:

```bash
git fetch origin
git rebase origin/develop
# (Fix any conflicts, then continue work)
```

## Integration with Release-Process Skill

This skill coordinates with `release-process` skill for semantic versioning:

1. **Before starting release:** Run `release-process` skill to determine version bump (PATCH/MINOR/MAJOR)
2. **Use that version:** `git flow release start X.Y.Z` with the determined version
3. **Follow changelog rules:** Both skills follow same changelog best practices (user-facing changes only)
4. **Coordinate versions:** In multi-component projects, ensure parent ≥ children (same rule as release-process)

**Example workflow:**
```bash
# Step 1: Use release-process to determine version
# → Returns: "Bump to 1.3.0 (minor update)"

# Step 2: Start release with that version
git flow release start 1.3.0

# Step 3: Update version files (guided by release-process)
# → Update plugin.json, SKILL.md, etc.

# Step 4: Update CHANGELOG.md (using changelog rules)
# → Document user-facing changes

# Step 5: Finish and tag
git flow release finish 1.3.0
git push origin main develop --tags
```

## Key Notes

**Branch protection:** Most teams protect `main` and `develop`:
- Require pull requests before merge
- Require status checks (tests pass)
- Require code review

**Naming conventions:**
- Features: `feature/user-auth`, `feature/api-v2` (lowercase, hyphens)
- Releases: `release/1.2.0` (semantic version)
- Hotfixes: `hotfix/1.2.1` (semantic version)

**Commit messages:** Use conventional commits when working with automated changelog generation:
```
feat: add user authentication
fix: resolve payment processing bug
docs: update installation guide
```

**Team communication:** Announce when starting/finishing releases or hotfixes so team knows deployment is coming.

**Version numbers follow semantic versioning:** See `release-process` skill for detailed bracket rules and version decisions.
