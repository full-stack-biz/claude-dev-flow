# Git-Flow Workflows for Common Scenarios

Step-by-step procedures for realistic development scenarios.

## Workflow 1: Standard Feature Development

**Scenario:** Implementing a new feature over several days with team code review.

```bash
# Step 1: Start feature from develop
git flow feature start user-profile-page

# Step 2: Work on the feature (multiple commits)
# Edit files, commit as needed
git commit -m "feat: add profile page structure"
git commit -m "feat: add profile edit form"
git commit -m "fix: validation for email field"

# Step 3: Push branch to remote so team can see it
git push origin feature/user-profile-page

# Step 4: Create pull request on GitHub/GitLab
# Link to issue, describe changes, request reviewers
# (Team reviews, requests changes)

# Step 5: Address review feedback
git commit -m "fix: reduce padding on form fields"
git push origin feature/user-profile-page

# Step 6: Approval received, finish feature
git flow feature finish user-profile-page

# Result:
# - feature/user-profile-page merged into develop
# - Branch deleted
# - You're on develop, ready for next task
```

**Time to execute:** Minutes (mostly waiting for code review)

## Workflow 2: Release Preparation (Single Component)

**Scenario:** Preparing a release for a single project with version updates and changelog.

```bash
# Prerequisites: Determine version using release-process skill
# Result: Version should be 1.3.0 (minor update)

# Step 1: Ensure develop has all features ready
git checkout develop
git pull origin develop

# Step 2: Start release branch
git flow release start 1.3.0

# Step 3: Update version in all files
# Example for dev-flow plugin:
# - plugin.json: "version": "1.3.0"
# - Skills' SKILL.md: version: 1.3.0

# Step 4: Update CHANGELOG.md
# Add new section at top with user-facing changes
cat CHANGELOG.md
# Edit, then:
git add CHANGELOG.md
git commit -m "docs: update CHANGELOG for 1.3.0"

# Step 5: Verify all version files updated
git log -1 --stat
git diff HEAD~1

# Step 6: Final testing/QA on release branch
# (Run tests, verify everything works)

# Step 7: Finish release
git flow release finish 1.3.0
# Prompt: Enter release tag message (e.g., "Release 1.3.0")
#   → Merges to main
#   → Tags main with v1.3.0
#   → Merges back to develop
#   → Deletes release branch

# Step 8: Push to remote
git push origin main develop --tags

# Result:
# - main is now v1.3.0 (deployable)
# - develop is updated with release changes
# - Tag v1.3.0 exists
```

**Time to execute:** 5-10 minutes (mostly updating files)

## Workflow 3: Release with Multi-Component Versions

**Scenario:** dev-flow has both plugin and skills with separate versions.

Current state:
- plugin.json: 1.2.0
- release-process/SKILL.md: 1.1.0
- git-flow/SKILL.md: 1.0.0

Feature work completed:
- Plugin feature: 1.2.0 → 1.3.0 (minor)
- release-process skill: 1.1.0 → 1.2.0 (minor)
- git-flow skill: 1.0.0 → 1.1.0 (minor)

**Coordination rule:** Plugin must be ≥ highest skill (1.2.0), so plugin → 1.3.0

```bash
# Step 1: Determine versions (using release-process skill)
# Outputs:
# - Plugin: 1.3.0
# - release-process/SKILL.md: 1.2.0
# - git-flow/SKILL.md: 1.1.0

# Step 2: Start release
git flow release start 1.3.0

# Step 3: Update ALL version files
# plugin.json: 1.3.0
sed -i '' 's/"version": "1.2.0"/"version": "1.3.0"/' plugin.json

# skills/release-process/SKILL.md: 1.2.0
# (Edit frontmatter: version: 1.2.0)

# skills/git-flow/SKILL.md: 1.1.0
# (Edit frontmatter: version: 1.1.0)

# Step 4: Verify coordination rule
# Plugin (1.3.0) >= release-process (1.2.0) ✓
# Plugin (1.3.0) >= git-flow (1.1.0) ✓
# Coordination rule satisfied

# Step 5: Update CHANGELOG.md
# Document plugin-level changes only
# (Skills with their own releases have their own changelogs)

git add plugin.json skills/release-process/SKILL.md skills/git-flow/SKILL.md CHANGELOG.md
git commit -m "chore: bump versions for 1.3.0 release"

# Step 6: Finish release
git flow release finish 1.3.0

# Step 7: Push
git push origin main develop --tags

# Result:
# - main tagged v1.3.0
# - All versions updated and in sync
# - develop updated for next cycle
```

**Key:** Update version files systematically to avoid misalignment. Use release-process skill to determine each component version first.

## Workflow 4: Production Hotfix

**Scenario:** Production bug discovered. Latest release is v1.2.0, develop is working on v1.3.0 features.

```bash
# Step 1: Ensure you're on main with latest production code
git checkout main
git pull origin main

# Step 2: Start hotfix (version = current version + patch)
# Current: v1.2.0 → Hotfix: v1.2.1
git flow hotfix start 1.2.1

# Step 3: Fix the bug
# Edit the problematic file(s)
git add .
git commit -m "fix: resolve payment processing error"

# Step 4: Update version to 1.2.1
# plugin.json: "version": "1.2.1"
# CHANGELOG.md: Add new section for 1.2.1

git add plugin.json CHANGELOG.md
git commit -m "chore: bump to 1.2.1"

# Step 5: Finish hotfix
git flow hotfix finish 1.2.1
# Prompt: Enter hotfix tag message (e.g., "Hotfix 1.2.1")
#   → Merges to main (now v1.2.1)
#   → Tags main with v1.2.1
#   → Merges back to develop (develop now has the fix)
#   → Deletes hotfix branch

# Step 6: Deploy immediately
git push origin main develop --tags
# Deploy main branch to production immediately

# Result:
# - Production running v1.2.1 with fix
# - develop has the fix integrated
# - When v1.3.0 releases, it includes this fix
```

**Critical timing:** Hotfix deploys directly after finishing. No intermediate QA.

## Workflow 5: Cancel a Feature (No Merge)

**Scenario:** Feature development started but scope changed. Need to cancel without merging.

```bash
# Step 1: You were on feature/experimental-api
git branch

# Output:
# * feature/experimental-api
#   develop
#   main

# Step 2: Delete without merging
git flow feature delete experimental-api

# Result:
# - Branch is deleted
# - Changes are discarded (work is lost)
# - You're on develop

# Alternative: If you want to keep the work
git checkout develop
# The branch still exists, can be recreated or recovered if needed
```

**Use case:** Keep the feature branch around if work might resume; delete if confirmed it's not needed.

## Workflow 6: Handling Merge Conflicts During Release Finish

**Scenario:** Release branch modified both `main` and `develop` differently, causing conflicts.

```bash
# Step 1: Start finishing release
git flow release finish 1.3.0

# Output (if conflict):
# CONFLICT (content merge conflict) in plugin.json
# Auto-merging plugin.json
# CONFLICT (add/add): Merge conflict in plugin.json
# Automatic merge failed; fix conflicts and then commit the result.

# Step 2: Resolve conflicts
git status

# Shows conflicted file(s)
# plugin.json: both main and develop modified version field

# Step 3: Edit conflicted file manually
# Look for conflict markers:
# <<<<<<< HEAD
# "version": "1.3.0"  (main version)
# =======
# "version": "1.3.0"  (develop version)
# >>>>>>> feature/branch

# If versions are the same: Remove conflict markers, keep one copy
# If versions differ: Coordinate with team on which to keep

# Step 4: Mark as resolved
git add plugin.json

# Step 5: Complete the merge
git commit -m "Resolve merge conflicts during release finish"

# Step 6: Now finish completes and continues
# Continue with finish workflow as normal
```

**Prevention:** Avoid modifying the same files on both `main` and `develop` outside of git-flow commands. Most conflicts happen when manual edits cause misalignment.

## Workflow 7: Syncing Feature Branch With Upstream Develop

**Scenario:** You're on `feature/user-profile` and `develop` has new commits from teammates.

```bash
# Step 1: Check current status
git status
# On branch feature/user-profile

# Step 2: Fetch latest from remote
git fetch origin

# Step 3: Rebase on develop (preferred) or merge (simpler)
# Option A: Rebase (cleaner history)
git rebase origin/develop

# If conflicts: resolve, then git add . && git rebase --continue

# Option B: Merge (safer if history matters)
git merge origin/develop

# Step 4: Push updated branch
git push origin feature/user-profile

# Result:
# - Your feature branch has latest develop changes
# - Continue work with updated codebase
```

**Team coordination:** Before starting major work, sync with remote to minimize conflicts later.

## Workflow 8: Debugging When Branches Are Out of Sync

**Scenario:** It's unclear whether `main`, `develop`, or a branch is ahead/behind.

```bash
# Step 1: Fetch latest
git fetch origin

# Step 2: Check branch status
git branch -vv

# Output shows tracking branches:
# develop        abc1234 [origin/develop] Last commit message
# feature/page   def5678 [origin/feature/page: ahead 2, behind 1] ...
# main           ghi9012 [origin/main] Last commit message

# Interpretation:
# - develop: In sync with remote
# - feature/page: 2 commits ahead (local), 1 behind (remote has changes)
# - main: In sync with remote

# Step 3: View differences
git log origin/develop..feature/page
# Shows commits on feature but not on develop

git log feature/page..origin/develop
# Shows commits on develop but not on feature

# Step 4: Decide action
# If feature needs to catch up: git rebase origin/develop
# If you're ready to finish: git flow feature finish
```

**Use regularly:** Prevents surprise conflicts and keeps team in sync.
