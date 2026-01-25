# Advanced Configuration

## Custom Workflow: Multiple Hotfix Types

Add a bugfix branch type (different from feature):

```bash
git-flow config add topic bugfix develop --prefix=bugfix/
```

Bugfix is now available:
```bash
git-flow bugfix start form-validation
git-flow bugfix finish form-validation
```

Use case: When you want to distinguish quick bug fixes from feature development.

## Production-Ready: Rebase + Squash

Configure features to rebase and releases to squash:

```bash
# Features rebase on develop
git-flow config edit topic feature --upstream-strategy=rebase

# Releases squash into main
git-flow config edit topic release --upstream-strategy=squash
```

Result: Clean feature history, simplified main branch history.

## Override Defaults Per Command

Apply strategy just for a single operation:

```bash
# Use rebase just this once (not stored)
git-flow feature finish --rebase my-feature

# Squash this release
git-flow release finish --squash 1.5.0
```

## Multi-Base Branch Workflows

Add additional base branches for staging and production environments:

```bash
# Add staging as intermediate base
git-flow config add base staging develop --auto-update=true

# Add production as final base
git-flow config add base production staging --auto-update=false
```

Options:
- `--upstream-strategy=merge|rebase|squash` — Merge strategy when finishing
- `--downstream-strategy=merge|rebase` — Merge strategy when updating from parent
- `--auto-update=true|false` — Auto-update from parent on finish

## View Configuration

```bash
git-flow config list
```

Shows all configured branches, topics, and their merge strategies.

## Reset to Preset

If custom configuration becomes complex, reset to a preset:

```bash
# View current config
git-flow config list

# Edit if needed
git-flow config edit topic feature --upstream-strategy=rebase

# Or remove and reinitialize with preset
rm -rf .git/config  # Back up first!
git-flow init --preset=classic --defaults
```
