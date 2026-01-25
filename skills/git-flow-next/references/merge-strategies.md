# Merge Strategies in Git-Flow-Next

## Table of Contents
- [Understanding Merge Strategies](#understanding-merge-strategies)
- [Merge Strategy](#merge-strategy)
- [Rebase Strategy](#rebase-strategy)
- [Squash Strategy](#squash-strategy)
- [Strategy Comparison](#strategy-comparison)
- [Setting Strategies](#setting-strategies)

---

## Understanding Merge Strategies

Git-flow-next supports three ways to integrate branches:

1. **merge** — Combines branches with merge commit (preserves history)
2. **rebase** — Replays commits on parent branch (linear history)
3. **squash** — Combines all commits into one (simplified history)

Each has tradeoffs in history clarity, debugging, and cleanup.

---

## Merge Strategy

The traditional three-way merge. Creates a merge commit that links both branch histories.

### How It Works

```
Before:
  main:     A - B - C
  feature:        D - E

After git-flow feature finish (merge):
  main:     A - B - C - M (merge commit)
                    \ /
  feature:        D - E

git log main:
  M - Merge branch 'feature/x' into main
  C - previous work
  B - previous work
  A - initial commit
```

### Visual Impact

```
Feature history visible:
  Each feature has a merge commit
  Branch points and merges are explicit
  Full commit history for each feature
  Easier to revert entire feature (one commit)
```

### Use Cases

- **Open source projects** — Preserve contributor history
- **Compliance/audit** — Full history required
- **Large teams** — See who merged what and when
- **Long-lived features** — Need to understand feature integration point

### Configuration

```bash
# Set merge for features
git-flow config edit topic feature --upstream-strategy=merge

# Set merge for releases
git-flow config edit topic release --upstream-strategy=merge
```

### Example

```bash
git-flow feature start user-dashboard
git commit -m "feat: add dashboard component"
git commit -m "feat: add dashboard styling"
git-flow feature finish user-dashboard

# Result in main:
# M - Merge pull request #42: User Dashboard
# D - feat: add dashboard styling
# C - feat: add dashboard component
# B - previous commit
```

---

## Rebase Strategy

Replays feature commits directly onto the parent branch. No merge commit. Results in linear history.

### How It Works

```
Before:
  main:     A - B - C
  feature:        D - E

After git-flow feature finish (rebase):
  main:     A - B - C - D' - E'

git log main:
  E - feat: add dashboard styling
  D - feat: add dashboard component
  C - previous work
  B - previous work
  A - initial commit
```

### Visual Impact

```
Linear history:
  No merge commits clutter
  Clean, easy-to-follow timeline
  Feature commits appear as if written on main
  Easier to bisect (find bad commit)
  Harder to revert entire feature (multiple commits)
```

### Use Cases

- **Development team** — Clean, readable history (recommended)
- **Fast-moving projects** — Keep main simple
- **Code review** — Easy to review individual commits
- **Debugging** — git bisect works best with linear history

### Configuration

```bash
# Set rebase for features
git-flow config edit topic feature --upstream-strategy=rebase

# Set rebase for releases
git-flow config edit topic release --upstream-strategy=rebase
```

### Example

```bash
git-flow feature start api-auth
git commit -m "feat: add JWT authentication"
git commit -m "test: add auth tests"
git-flow feature finish api-auth

# Result in main:
# E - test: add auth tests
# D - feat: add JWT authentication
# C - previous commit
# B - previous commit
# A - initial commit

# No merge commit, commits appear inline
```

### Handling Conflicts During Rebase

```bash
git-flow feature finish api-auth

# If conflict occurs:
# CONFLICT (content merge conflict) in src/auth.js
# Automatic merge failed; fix conflicts and then continue the rebase.

# Step 1: Fix conflicts in editor
# Edit src/auth.js, remove conflict markers

# Step 2: Mark as resolved
git add src/auth.js

# Step 3: Continue rebase
git rebase --continue
# (or git-flow feature finish api-auth again)
```

---

## Squash Strategy

Combines all feature commits into a single commit. Simplifies history, one commit = one feature.

### How It Works

```
Before:
  main:     A - B - C
  feature:        D - E - F - G

After git-flow feature finish (squash):
  main:     A - B - C - S (squash commit)

S contains all changes from D, E, F, G combined
But is a single commit

git log main:
  S - User authentication (all auth work in one commit)
  C - previous work
  B - previous work
  A - initial commit
```

### Visual Impact

```
Simplified history:
  One commit per feature/release
  Very clean main branch
  Easy to revert entire feature (one commit)
  Hard to understand individual changes
  Not ideal for debugging individual feature parts
```

### Use Cases

- **Production main branch** — Simplified history
- **Release branches** — One commit per release
- **Monorepos with releases** — Clear version markers
- **Marketing/release notes** — One line per release

### Configuration

```bash
# Set squash for releases (main stays simple)
git-flow config edit topic release --upstream-strategy=squash

# Set squash for features (if very simple)
git-flow config edit topic feature --upstream-strategy=squash
```

### Example

```bash
git-flow release start 2.0.0
# ... version bump commits ...
git commit -m "bump: version to 2.0.0"
git commit -m "docs: update CHANGELOG"
git commit -m "docs: update README"
git-flow release finish 2.0.0

# Result in main:
# S - Release 2.0.0 (all 3 commits squashed)
# P - Previous release (from commit S)
# O - Previous release

# main looks like:
# Release 2.0.0
# Release 1.9.0
# Release 1.8.0
# Very clean!
```

---

## Strategy Comparison

| Aspect | Merge | Rebase | Squash |
|---|---|---|---|
| **Merge commits** | ✓ Yes | ✗ No | ✗ No |
| **Individual commits visible** | ✓ Yes | ✓ Yes | ✗ No |
| **History complexity** | Higher | Lower | Lowest |
| **Debugging (git bisect)** | Hard | Good | Hard |
| **Revert one feature** | Easy (1 commit) | Hard (multiple) | Easy (1 commit) |
| **Code review clarity** | Good | Excellent | Poor |
| **History readability** | Moderate | Excellent | Good |
| **Best for teams** | ✓ Open source | ✓ Development | ✗ Not usual |
| **Best for production** | ✗ Noisy | ✗ Too detailed | ✓ Clean |

---

## Setting Strategies

### For Individual Branch Types

```bash
# Features use rebase
git-flow config edit topic feature --upstream-strategy=rebase

# Releases use squash
git-flow config edit topic release --upstream-strategy=squash

# Hotfixes use merge
git-flow config edit topic hotfix --upstream-strategy=merge
```

### Per-Command Override

```bash
# Use different strategy just this once
git-flow feature finish --rebase my-feature
git-flow release finish --squash 1.5.0
git-flow hotfix finish --merge 1.5.1
```

### View Current Strategies

```bash
git-flow config list

# Output:
# Branch: feature
#   upstream-strategy: rebase
# Branch: release
#   upstream-strategy: squash
# Branch: hotfix
#   upstream-strategy: merge
```

---

## Recommended Configurations

### Team Development (Clean History)

```bash
git-flow init --preset=classic --defaults

# Features: rebase for clean development history
git-flow config edit topic feature --upstream-strategy=rebase

# Releases: squash for simple main
git-flow config edit topic release --upstream-strategy=squash

# Hotfixes: merge (quick, preserve context)
git-flow config edit topic hotfix --upstream-strategy=merge
```

Result: Clean feature development, simplified releases, quick hotfixes.

### Open Source (Full History)

```bash
git-flow init --preset=classic --defaults

# All branches: merge (preserve contributor history)
git-flow config edit topic feature --upstream-strategy=merge
git-flow config edit topic release --upstream-strategy=merge
git-flow config edit topic hotfix --upstream-strategy=merge
```

Result: Every commit visible, full contributor history, clear merge points.

### Production-Grade (Optimized)

```bash
git-flow init --preset=classic --defaults

# Features: rebase (clean integration)
git-flow config edit topic feature --upstream-strategy=rebase

# Releases: squash (one commit per version)
git-flow config edit topic release --upstream-strategy=squash

# Hotfixes: rebase (minimize commit noise)
git-flow config edit topic hotfix --upstream-strategy=rebase
```

Result: Clean develop, simple main, fast debugging.

---

## Handling Strategy Changes Mid-Project

If you need to change strategies after starting:

```bash
# Current setup
git-flow config list

# Change strategy
git-flow config edit topic feature --upstream-strategy=squash

# Future features will squash
# Already-merged features keep their merge style
# Can't change past merges
```

**Note:** Changing strategies doesn't affect already-merged branches, only new ones.
