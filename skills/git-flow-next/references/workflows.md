# Git-Flow-Next Workflows for Real-World Scenarios

## Table of Contents
- [Workflow 1: Standard Feature Development](#workflow-1-standard-feature-development)
- [Workflow 2: Release with Semantic Versioning](#workflow-2-release-with-semantic-versioning)
- [Workflow 3: Production Hotfix](#workflow-3-production-hotfix)
- [Workflow 4: Setting Up for Team](#workflow-4-setting-up-for-team)
- [Workflow 5: Custom Branch Types](#workflow-5-custom-branch-types)
- [Workflow 6: Configuring Merge Strategies](#workflow-6-configuring-merge-strategies)

---

## Workflow 1: Standard Feature Development

**Scenario:** Developing a feature over several days with code review before merging.

```bash
# Step 1: Start feature from develop
git-flow feature start user-authentication

# Step 2: Work on the feature (multiple commits)
git commit -m "feat: add login form component"
git commit -m "feat: add password validation"
git commit -m "fix: handle edge case in auth flow"

# Step 3: Push branch to remote for code review
git push origin feature/user-authentication

# Step 4: Create pull request on GitHub/GitLab
# - Link to issue
# - Request reviewers
# - Discuss changes

# Step 5: Address review feedback
git commit -m "fix: improve form accessibility"
git push origin feature/user-authentication

# Step 6: Approval received, finish feature
git-flow feature finish user-authentication

# Step 7: Deploy to develop
git push origin develop
```

**Result:**
- Feature merged into develop
- Branch deleted locally and on remote
- Team can now start next feature from updated develop

---

## Workflow 2: Release with Semantic Versioning

**Scenario:** Preparing a release with version bumps and changelog updates.

```bash
# Prerequisites: Determine version using release-process skill
# Decision: Version should be 1.3.0 (minor update)

# Step 1: Ensure develop has all features ready
git checkout develop
git pull origin develop

# Step 2: Start release branch
git-flow release start 1.3.0

# Step 3: Update version files
# - plugin.json: "version": "1.3.0"
# - SKILL.md: version: 1.3.0
# - package.json: "version": "1.3.0"

# Step 4: Update CHANGELOG.md
# Add new section documenting user-facing changes
cat CHANGELOG.md
# Edit, then:
git add CHANGELOG.md
git commit -m "docs: update CHANGELOG for 1.3.0"

# Step 5: Verify all version files
git diff HEAD~1 --stat

# Step 6: Final testing/QA on release branch
# (Run tests, verify everything works)

# Step 7: Finish release
git-flow release finish 1.3.0
# Prompt: Enter release tag message
#   → Creates tag v1.3.0
#   → Merges to main
#   → Merges back to develop
#   → Deletes release branch

# Step 8: Push everything
git push origin main develop --tags

# Step 9: Deploy main to production
# (CI/CD pipeline or manual deployment)
```

**Result:**
- main is now v1.3.0 (deployable)
- develop updated with release changes
- Tag v1.3.0 exists for reference

---

## Workflow 3: Production Hotfix

**Scenario:** Critical bug discovered in production (v1.2.0). Develop is working on v1.3.0 features.

```bash
# Step 1: Ensure you're on main with latest production
git checkout main
git pull origin main

# Step 2: Start hotfix (version = last release + patch)
# Last release: v1.2.0 → Hotfix: v1.2.1
git-flow hotfix start 1.2.1

# Step 3: Fix the critical bug
# Edit the affected files
git add .
git commit -m "fix: resolve payment processing error"

# Step 4: Update version and changelog
# - Update version files to 1.2.1
# - Add entry to CHANGELOG.md
git commit -m "chore: bump to 1.2.1"

# Step 5: Finish hotfix
git-flow hotfix finish 1.2.1
# Prompt: Enter hotfix tag message
#   → Creates tag v1.2.1
#   → Merges to main
#   → Merges back to develop
#   → Deletes hotfix branch

# Step 6: Deploy immediately
git push origin main develop --tags
# Deploy main branch to production NOW

# Result:
# - Production running v1.2.1 with fix
# - develop has the fix (ready for v1.3.0)
# - No time wasted on release process
```

---

## Workflow 4: Setting Up for Team

**Scenario:** Team of 5 developers, need to establish branching strategy and enforce standards.

```bash
# Step 1: Decide on workflow preset
# For teams: Classic GitFlow recommended
git-flow init --preset=classic --defaults

# Step 2: Verify configuration
git-flow config list

# Output shows:
# Base branches:
#   main (production)
#   develop (integration)
# Topic branches:
#   feature/ (from develop)
#   release/ (from develop, merges to main)
#   hotfix/ (from main)

# Step 3: Set merge strategy for features (team preference)
git-flow config edit topic feature --upstream-strategy=rebase
# Result: Clean linear history, no merge commits

# Step 4: Optional - Add bugfix as alternative to feature
git-flow config add topic bugfix develop --prefix=bugfix/

# Step 5: Push configuration to remote
git push origin develop  # Config is in .git/config

# Step 6: Document team standards
# Create CONTRIBUTING.md with:
# - When to use feature vs bugfix
# - Naming conventions (lowercase, hyphens)
# - Code review requirements
# - Release process

# Team members now initialize same way:
git clone <repo>
git-flow init --preset=classic --defaults
```

**Team practices:**
- All features start from develop
- Code review before finish
- Release lead manages releases
- Hotfixes are emergency-only

---

## Workflow 5: Custom Branch Types

**Scenario:** Team wants both features and bugfixes, with different handling.

```bash
# Step 1: Initialize classic
git-flow init --preset=classic --defaults

# Step 2: Add bugfix as separate branch type
git-flow config add topic bugfix develop \
  --prefix=bugfix/ \
  --upstream-strategy=rebase

# Step 3: Verify configuration
git-flow config list

# Step 4: Use cases
# Use FEATURE for new functionality:
git-flow feature start new-dashboard

# Use BUGFIX for bug fixes:
git-flow bugfix start form-validation

# Both merge to develop but signal intent clearly

# Step 5: Edit bugfix settings (example)
git-flow config edit topic bugfix --upstream-strategy=squash

# Now bugfixes squash into develop, features rebase
```

**Why custom branches?**
- Clear intent (feature vs bugfix)
- Different handling if needed
- Documentation in branch name

---

## Workflow 6: Configuring Merge Strategies

**Scenario:** Production-grade setup with different strategies per branch type.

```bash
# Scenario: Clean main, fast-moving develop, team collaboration

# Step 1: Initialize
git-flow init --preset=classic --defaults

# Step 2: Configure strategies
# Features: rebase (clean develop history)
git-flow config edit topic feature --upstream-strategy=rebase

# Releases: squash (simplified main history)
git-flow config edit topic release --upstream-strategy=squash

# Step 3: Verify
git-flow config list

# Output shows:
# feature:
#   upstream-strategy: rebase
# release:
#   upstream-strategy: squash

# Step 4: Usage
git-flow feature start api-v2
# ... work ...
git-flow feature finish api-v2
# Result: Commits replayed on develop, clean history

git-flow release start 2.0.0
# ... version bump ...
git-flow release finish 2.0.0
# Result: All release commits squashed into main, tagged v2.0.0

# Step 5: main branch is now simple:
git log main
# v2.0.0: All release changes
# v1.5.0: Previous release
# v1.4.0: Previous release
# (No merge commits, no feature noise)
```

**Strategy benefits:**
- **rebase:** Linear develop history, easy to understand feature progression
- **squash:** Simplified main history, one commit per feature/release
- **merge:** Full history preservation (use for open source)

---

## Team Onboarding Checklist

For new team members:

```bash
# 1. Clone the repo
git clone <repo>

# 2. Initialize git-flow-next
git-flow init --preset=classic --defaults

# 3. Verify team config
git-flow config list

# 4. Create a test feature
git-flow feature start test-setup

# 5. Make a commit
git commit --allow-empty -m "test: verify git-flow setup"

# 6. Finish the test feature
git-flow feature finish test-setup

# 7. Check that everything merged correctly
git log develop --oneline -5

# Ready to start real work!
```
