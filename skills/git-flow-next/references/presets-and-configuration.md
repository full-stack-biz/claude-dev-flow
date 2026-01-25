# Git-Flow-Next Presets and Configuration

## Table of Contents
- [Preset Models](#preset-models)
- [Classic GitFlow Preset](#classic-gitflow-preset)
- [GitHub Flow Preset](#github-flow-preset)
- [GitLab Flow Preset](#gitlab-flow-preset)
- [Custom Configuration](#custom-configuration)
- [Configuration Tuning](#configuration-tuning)

---

## Preset Models

Git-flow-next includes three pre-built workflow presets, each optimized for different development practices.

| Preset | Best For | Main Feature | Branches |
|---|---|---|---|
| **classic** | Teams, semantic versioning | Multiple long-lived branches | main, develop, feature/, release/, hotfix/ |
| **github** | Continuous deployment, simple | Single integration branch | main, feature/ |
| **gitlab** | Staged deployments | Progressive environment promotion | main, develop, staging, production, feature/ |

---

## Classic GitFlow Preset

The original git-flow model, best for teams using semantic versioning with explicit dev/prod separation.

### Structure

```
main (production, always deployable)
  ↑ merges from release/ and hotfix/
  ↓ tags (v1.0.0, v1.1.0, etc.)

develop (integration, pre-release testing)
  ↑ merges from feature/, release/, and hotfix/
  ↓ source for new features

feature/ (individual features)
develop-flow-next← starts from develop, merges back to develop

release/ (release preparation)
develop ← starts from develop
main → merges to main (with tag), merges back to develop

hotfix/ (production fixes)
main ← starts from main
main → merges to main (with tag), merges back to develop
```

### Initialize Classic GitFlow

```bash
# Option 1: Interactive
git-flow init
# Choose "classic" when prompted

# Option 2: Direct
git-flow init --preset=classic --defaults

# Option 3: With custom branch names
git-flow init --preset=classic --defaults \
  --main=main \
  --develop=develop \
  --feature=feature/ \
  --release=release/ \
  --hotfix=hotfix/ \
  --tag=v
```

### Command Reference

```bash
# Features
git-flow feature start auth-module
git-flow feature finish auth-module
git-flow feature list

# Releases
git-flow release start 1.2.0
git-flow release finish 1.2.0
git-flow release list

# Hotfixes
git-flow hotfix start 1.2.1
git-flow hotfix finish 1.2.1
git-flow hotfix list
```

### Use Cases

**When Classic GitFlow is right:**
- Team with 3+ developers
- Releases planned in advance
- Semantic versioning important
- Need distinct dev and production
- CI/CD pipeline exists

**Team workflow example:**
```
Feature developers: Start from develop, finish when PR approved
Release lead: Manages release/ branches every 2-3 weeks
Hotfix responder: Emergency patches from main
```

---

## GitHub Flow Preset

A simpler model focused on continuous deployment. Main branch is always deployable, features are PR-based.

### Structure

```
main (production, continuously deployable)
  ↑ merges from feature/ PRs
  ↓ deploys immediately

feature/* (short-lived branches)
  main ← starts from main
  main → merges via PR, deletes immediately
```

### Initialize GitHub Flow

```bash
# Option 1: Interactive
git-flow init
# Choose "github" when prompted

# Option 2: Direct
git-flow init --preset=github --defaults

# Option 3: Custom
git-flow init --preset=github --defaults --main=main
```

### Command Reference

```bash
# Features only (no release/hotfix)
git-flow feature start dashboard-redesign
git-flow feature finish dashboard-redesign

# Check active features
git-flow feature list

# Note: GitHub Flow has no separate release or hotfix branches
# Hotfixes are treated as regular features (just marked urgent)
```

### Use Cases

**When GitHub Flow is right:**
- Small team or solo development
- Continuous deployment (deploy main every push)
- Simple codebase, fast iteration
- No need for staging environment
- Frequent, small releases

**Workflow example:**
```
Every commit to feature/ → PR → review → merge to main → deploy
No separate release or hotfix branches
```

---

## GitLab Flow Preset

A staged deployment model with separate environments (develop → staging → production).

### Structure

```
production (live, only stable releases)
  ↑ merges from staging
  ↓ customer-facing

staging (pre-production, final testing)
  ↑ merges from develop
  ↓ team testing before release

develop (integration, features and fixes)
  ↑ merges from feature/
  ↓ source for next staging push

feature/* (individual features)
develop ← starts from develop, merges back to develop
```

### Initialize GitLab Flow

```bash
# Option 1: Interactive
git-flow init
# Choose "gitlab" when prompted

# Option 2: Direct
git-flow init --preset=gitlab --defaults

# Option 3: Custom
git-flow init --preset=gitlab --defaults \
  --develop=develop \
  --staging=staging \
  --production=production
```

### Command Reference

```bash
# Start features from develop
git-flow feature start search-indexing

# Finish features (merge to develop)
git-flow feature finish search-indexing

# Promote to staging manually
git checkout staging
git merge develop
git push origin staging

# Final test, then promote to production
git checkout production
git merge staging
git push origin production
# Deploy production branch
```

### Use Cases

**When GitLab Flow is right:**
- Teams with formal QA/staging process
- Need environment progression (dev → staging → prod)
- Compliance or quality gates required
- Separate deployment timelines per environment

**Workflow example:**
```
Feature → develop → staging (test) → production (deploy)
Each environment has own branch and deployment
```

---

## Custom Configuration

Build a workflow tailored to your team's specific needs.

### Initialize with Custom Config

```bash
# Start custom
git-flow init --custom

# Or add to existing config
git-flow config add base develop
git-flow config add topic feature develop --prefix=feature/
```

### Add Custom Branch Types

**Example: Separate bugfix and feature branches**

```bash
# Initialize classic first
git-flow init --preset=classic --defaults

# Add bugfix as distinct from feature
git-flow config add topic bugfix develop --prefix=bugfix/

# Verify
git-flow config list

# Usage:
git-flow feature start new-dashboard
git-flow bugfix start form-validation
# Both merge to develop, but intent is clear
```

**Example: Documentation branches**

```bash
git-flow config add topic docs main --prefix=docs/ --tag=true

# Usage:
git-flow docs start api-guide
git-flow docs finish api-guide
# Creates docs tag on main when finished
```

### Configure Base Branches

**Add staging environment:**

```bash
# staging is based on production
git-flow config add base staging main \
  --upstream-strategy=merge \
  --auto-update=false

# Now you have: main → staging → develop progression
```

**Enable auto-update:**

```bash
# Staging automatically updates from main
git-flow config add base staging main --auto-update=true

# When releasing to main, staging auto-updates
# Prevents staging from drifting
```

---

## Configuration Tuning

### Merge Strategy Selection

**For feature branches:**
```bash
# Rebase: Clean develop history (recommended for teams)
git-flow config edit topic feature --upstream-strategy=rebase

# Merge: Preserve full history (open source, collaboration)
git-flow config edit topic feature --upstream-strategy=merge

# Squash: One commit per feature (fast-moving projects)
git-flow config edit topic feature --upstream-strategy=squash
```

**For release branches:**
```bash
# Squash: Simplified main (production-ready)
git-flow config edit topic release --upstream-strategy=squash

# Merge: Full release history (compliance, audits)
git-flow config edit topic release --upstream-strategy=merge
```

### Team-Wide Configuration

**Option 1: Document in README**
```markdown
## Git-Flow Setup

Initialize with:
```bash
git-flow init --preset=classic --defaults
git-flow config edit topic feature --upstream-strategy=rebase
```
```

**Option 2: Store custom init script**
```bash
#!/bin/bash
# scripts/setup-gitflow.sh
git-flow init --preset=classic --defaults
git-flow config edit topic feature --upstream-strategy=rebase
git-flow config edit topic release --upstream-strategy=squash
git-flow config add topic bugfix develop --prefix=bugfix/
echo "Git-flow configured for team standards"
```

Team members run once:
```bash
./scripts/setup-gitflow.sh
```

**Option 3: Pre-configured template**
```bash
# Clone repo, it already has team config in .git/config
git clone <repo>
git-flow feature list
# Ready to go, no setup needed
```

### Configuration Verification

```bash
# See current setup
git-flow config list

# Sample output:
# Base Branches:
#   main (main)
#   develop (develop)
#
# Topic Branches:
#   feature (develop -> main, rebase, no tag)
#   release (develop -> main, squash, tag=yes)
#   hotfix (main -> main, merge, tag=yes)
#   bugfix (develop -> develop, rebase, no tag)
```

### Changing Configuration Mid-Project

```bash
# Existing config
git-flow config list

# Edit an entry
git-flow config edit topic feature --upstream-strategy=rebase

# Delete custom branch type
git-flow config delete topic bugfix

# Verify change
git-flow config list
```

---

## Decision Matrix: Which Preset?

| Question | Classic | GitHub | GitLab |
|---|---|---|---|
| Team size >3? | ✓ | ✓ | ✓ |
| Semantic versioning? | ✓ | ✗ | ✓ |
| Planned releases? | ✓ | ✗ | ✓ |
| Continuous deployment? | ✓ | ✓ | ✗ |
| Staging environment? | ✗ | ✗ | ✓ |
| Multiple environments? | ✗ | ✗ | ✓ |
| Simple, fast iteration? | ✗ | ✓ | ✗ |

**Decision flowchart:**
```
Do you deploy constantly (every push)?
  YES → GitHub Flow (simplest)
  NO ↓

Do you need staged environments?
  YES → GitLab Flow (production-grade)
  NO ↓

Do you plan releases in advance?
  YES → Classic GitFlow (most common)
  NO → GitHub Flow
```
