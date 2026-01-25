---
name: git-flow-next
description: >-
  Manage git-flow branching workflows. Use when working with feature branches, release branches, hotfixes, or configuring branching strategy. Supports Classic GitFlow, GitHub Flow, GitLab Flow, and custom configurations.
version: 1.0.1
allowed-tools: Bash(git-flow:*)
---

# Git-Flow-Next Skill

Git-flow-next is a modern, Go-based implementation of the git-flow branching model with support for preset workflows (Classic GitFlow, GitHub Flow, GitLab Flow) and fully customizable branch configurations.

## Quick Start

**Prerequisites:** git-flow-next must be installed. See `references/installation.md` for setup instructions.

**Initialize your repository:**
```bash
# Interactive (choose preset: classic, github, gitlab)
git-flow init

# With preset (no prompts)
git-flow init --preset=classic --defaults

# Custom configuration
git-flow init --custom
```

**Common branch operations:**
```bash
# Start and finish a feature
git-flow feature start my-feature
git-flow feature finish my-feature

# Start and finish a release
git-flow release start 1.2.0
git-flow release finish 1.2.0

# Emergency production fix
git-flow hotfix start 1.2.1
git-flow hotfix finish 1.2.1
```

## Branch Management

### Feature Branches

**Start a feature:**
```bash
git-flow feature start authentication
```
Creates `feature/authentication` from `develop`.

**Finish a feature:**
```bash
git-flow feature finish authentication
```
- Merges into `develop`
- Deletes feature branch
- Returns to `develop`

**List active features:**
```bash
git-flow feature list
```

**Delete without merging:**
```bash
git-flow feature delete authentication
```

### Release Branches

**Start a release:**
```bash
git-flow release start 1.2.0
```
Creates `release/1.2.0` from `develop`.

**Finish a release:**
```bash
git-flow release finish 1.2.0
```
- Merges into `main`
- Tags `main` with `v1.2.0`
- Merges back into `develop`
- Deletes release branch

**List pending releases:**
```bash
git-flow release list
```

### Hotfix Branches

**Start a hotfix:**
```bash
git-flow hotfix start 1.2.1
```
Creates `hotfix/1.2.1` from `main` (critical: from production, not develop).

**Finish a hotfix:**
```bash
git-flow hotfix finish 1.2.1
```
- Merges into `main`
- Tags `main` with `v1.2.1`
- Merges back into `develop`
- Deletes hotfix branch

**List active hotfixes:**
```bash
git-flow hotfix list
```

## Configuration Management

### View Current Configuration

```bash
git-flow config list
```

Output shows your branch structure, prefixes, and merge strategies.

### Add a Base Branch

```bash
# Add production trunk
git-flow config add base production

# Add staging with auto-update
git-flow config add base staging production --auto-update=true
```

**Options:**
- `--upstream-strategy=merge|rebase|squash` — Merge strategy when finishing
- `--downstream-strategy=merge|rebase` — Merge strategy when updating from parent
- `--auto-update=true|false` — Auto-update from parent on finish (default: false)

### Add a Topic Branch Type

```bash
# Add custom feature branch
git-flow config add topic feature develop --prefix=feat/

# Release with tagging
git-flow config add topic release main --starting-point=develop --tag=true

# Bugfix branch (alternative to feature)
git-flow config add topic bugfix develop --prefix=bugfix/
```

**Topic Options:**
- `--prefix=prefix` — Branch name prefix (default: *name*/)`
- `--starting-point=branch` — Create from this branch (default: parent)
- `--upstream-strategy=merge|rebase|squash` — Merge strategy
- `--downstream-strategy=merge|rebase` — Update strategy
- `--tag=true|false` — Create tags on finish (default: false)

### Edit Configuration

```bash
# Change merge strategy for features
git-flow config edit topic feature --upstream-strategy=rebase

# Disable auto-update on staging
git-flow config edit base staging --auto-update=false
```

### Delete Configuration

```bash
# Remove custom branch type
git-flow config delete topic bugfix

# Remove base branch from git-flow (keeps Git branch)
git-flow config delete base staging
```

## Merge Strategies

Git-flow-next supports merge, rebase, and squash strategies. See `references/merge-strategies.md` for detailed comparison and examples.

**Quick example:**
```bash
git-flow config edit topic feature --upstream-strategy=rebase
```

## Workflow Examples

### Classic GitFlow (Recommended for Teams)

**Setup:**
```bash
git-flow init --preset=classic --defaults
```

**Workflow:**
```bash
# Feature development
git-flow feature start user-auth
# ... commit, push, create PR ...
git-flow feature finish user-auth

# Release preparation
git-flow release start 1.3.0
# ... update version files, changelog ...
git-flow release finish 1.3.0
git push origin main develop --tags

# Production emergency
git-flow hotfix start 1.3.1
# ... fix bug ...
git-flow hotfix finish 1.3.1
git push origin main develop --tags
```

### GitHub Flow (Continuous Deployment)

**Setup:**
```bash
git-flow init --preset=github --defaults
```

**Workflow:**
```bash
# Simpler than classic
git-flow feature start api-endpoint
git-flow feature finish api-endpoint

# Immediately deployable from main
git push origin main
```

### GitLab Flow (Staging/Production)

**Setup:**
```bash
git-flow init --preset=gitlab --defaults
```

**Workflow:**
```bash
# Features as normal
git-flow feature start dashboard
git-flow feature finish dashboard

# Deploy to staging first
git-flow release start 2.0.0
git-flow release finish 2.0.0
# Merge to staging for testing
# Then promote to production
```

## Common Patterns & Troubleshooting

### Push Feature for Code Review

```bash
git-flow feature start new-feature
# ... work, commit ...
git push origin feature/new-feature
# Create PR, get reviews
git-flow feature finish new-feature
```

### Sync Feature With Upstream Develop

```bash
git fetch origin
git rebase origin/develop
# Fix conflicts if needed
git push origin feature/my-feature
```

### Check Branch Status

```bash
git-flow overview
# Shows all active branches and their status
```

### Show Help for Any Command

```bash
git-flow init --help
git-flow feature --help
git-flow config --help
```

### Enable Verbose Output

```bash
git-flow -v feature start my-feature
# Shows detailed operation information
```

## Error Handling

**If git-flow command is not found:**
- Ensure git-flow-next is installed (see `references/installation.md`)
- Verify it's in your PATH: `which git-flow`
- On macOS with Homebrew: `brew install gittower/tap/git-flow-next`

**If merge/rebase conflicts occur:**
- Resolve conflicts in files
- Continue: `git add .` then run the `git-flow` command again with `--continue` flag
- Abort: Most commands support `--abort` to cancel operation

## Key Notes

**Branch prefixes:** Customizable via `--prefix` option in `git-flow init` or `git-flow config add topic`

**Merge strategies matter:**
- Team projects: Use `rebase` for clean history
- Production: Use `squash` for simplified main branch
- Open source: Use `merge` to preserve contributor history

**Configuration stored locally:** Config stays in `.git/config` (checked into repo by default)

**Presets provide starter templates:**
- Classic: Traditional feature/develop/main model
- GitHub: Single main branch with features
- GitLab: Develop → staging → production progression

**Always version your releases:** Tag with semantic versioning (v1.2.3) when finishing releases

**Team coordination:** Announce releases and hotfixes so team knows deployment timing

## References

- `references/installation.md` — Install and initialize git-flow-next
- `references/advanced-configuration.md` — Custom workflows, multi-base branches
- `references/merge-strategies.md` — Strategy comparison and examples
- `references/workflows.md` — Detailed workflow examples
- `references/presets-and-configuration.md` — Preset details and configuration options
