# Multi-Component Coordination for Monorepos and Plugins

How to manage versions and releases when multiple components have separate version numbers.

## Core Principle: Parent ≥ Children

In systems with multiple versioned components (plugin + skills, monorepo services), the parent version must always be greater than or equal to the highest child version.

**Example: dev-flow plugin structure**
```
dev-flow (parent)
├─ release-process skill v1.2.0
├─ git-flow skill v1.1.0
└─ plugin.json v1.3.0 (parent version)
```

**Rule:** Plugin version (1.3.0) ≥ max(1.2.0, 1.1.0) = 1.2.0 ✓

## Version Coordination Matrix

| Scenario | Action | Result |
|---|---|---|
| Parent 1.0.0, Child 1.0.0 | Child bumps to 1.1.0 | Parent must bump to 1.1.0 |
| Parent 1.1.0, Child 1.0.0 | Child bumps to 1.2.0 | Parent must bump to 1.2.0 |
| Parent 1.5.0, Child 1.5.0 | Only parent changes | Parent bumps, child stays (e.g., 1.5.0 → 1.5.1) |
| Parent 2.0.0, Child 1.9.0 | Parent already ahead | No coordination needed |

**Decision rule:**
1. Determine version for EACH child (using release-process bracket rules)
2. Parent version = MAX(children versions) if any child bumped
3. If no children bumped but parent has changes → Parent bumps (follows same bracket rules)

## Step-by-Step Release Process for Multi-Component

### Phase 1: Assessment (Before Release)

```bash
# Step 1: Determine what changed
git log --oneline origin/develop..HEAD

# Output:
# feat: add hotfix skill capability (skill-a)
# feat: improve UI (skill-b)
# fix: resolve config loading bug (main plugin)

# Step 2: Map changes to components
# Components changed:
# - skill-a: MINOR (new feature)
# - skill-b: MINOR (new feature)
# - plugin: PATCH (bug fix)

# Step 3: Look up current versions
cat plugin.json | grep version
# "version": "1.2.0"

cat skills/skill-a/SKILL.md | head -20
# version: 1.0.0

cat skills/skill-b/SKILL.md | head -20
# version: 1.1.0

# Step 4: Calculate new versions (using release-process bracket rules)
# skill-a: 1.0.0 → 1.1.0 (minor)
# skill-b: 1.1.0 → 1.2.0 (minor)
# plugin: 1.2.0 → ??? (must be >= max(1.1.0, 1.2.0) = 1.2.0)

# Since skill-b jumped to 1.2.0, plugin must jump at least to 1.2.0
# But plugin itself has changes (bug fix = PATCH)
# Coordination rule: 1.2.0 already >= 1.2.0, but plugin PATCH would be 1.2.1
# Resolution: Plugin jumps to 1.3.0 to accommodate skill-b's minor

# Final versions:
# - skill-a: 1.1.0
# - skill-b: 1.2.0
# - plugin: 1.3.0 (MAJOR coordination bump to stay >= 1.2.0)
```

**Key insight:** A child's version bump can force parent to bump more than its own changes would require.

### Phase 2: Release Preparation

```bash
# Step 1: Start release branch
git flow release start 1.3.0

# Step 2: Update ALL version files in correct order
# (Update child versions first, then parent)

# Update skill-a version
# File: skills/skill-a/SKILL.md
# Change frontmatter: version: 1.1.0

# Update skill-b version
# File: skills/skill-b/SKILL.md
# Change frontmatter: version: 1.2.0

# Update plugin version
# File: plugin.json
# Change: "version": "1.3.0"

# Step 3: Verify coordination
echo "Checking versions..."
grep version plugin.json                           # 1.3.0
grep "^version:" skills/skill-a/SKILL.md          # 1.1.0
grep "^version:" skills/skill-b/SKILL.md          # 1.2.0

# Verify: 1.3.0 >= max(1.1.0, 1.2.0) = 1.2.0 ✓

# Step 4: Update CHANGELOG.md (plugin level only)
# Document plugin changes, not child changes
# (Each skill has its own changelog if desired)

# Step 5: Commit all version updates
git add plugin.json skills/skill-a/SKILL.md skills/skill-b/SKILL.md CHANGELOG.md
git commit -m "chore: bump versions for 1.3.0 release"
```

### Phase 3: Testing and Finish

```bash
# Step 1: Final verification
git diff HEAD~1 --stat
# Should show only version files and CHANGELOG

# Step 2: Finish release
git flow release finish 1.3.0

# Step 3: Push and tag
git push origin main develop --tags

# Result:
# - main: v1.3.0 (tagged)
# - All versions updated
# - Child versions 1.1.0 and 1.2.0 now part of parent 1.3.0 release
```

## Monorepo Example: Multiple Services

**Scenario:** Monorepo with 3 services that version independently.

```
monorepo/
├─ package.json (root) v2.0.0
├─ services/
│  ├─ auth-service/ v1.8.0
│  ├─ api-service/ v2.1.0
│  └─ worker-service/ v1.5.0
```

**Release workflow:**
1. Assess changes for each service
2. Calculate each service version (using bracket rules)
3. Root version = MAX(service versions) if any service bumped
4. Update all package.json versions (root + service-specific)
5. Start release from root version: `git flow release start X.Y.Z`
6. Commit all version changes
7. Finish release

**Example:**
```bash
# Changes assessed:
# - auth-service: 1.8.0 → 1.9.0 (minor: new OAuth provider)
# - api-service: 2.1.0 → 2.1.1 (patch: fix timeout)
# - worker-service: 1.5.0 → 1.5.0 (no changes)

# Current root: 2.0.0
# Max service: max(1.9.0, 2.1.1, 1.5.0) = 2.1.1
# Root must be >= 2.1.1, so root: 2.0.0 → 2.2.0

# Release version: 2.2.0
git flow release start 2.2.0

# Update all package.json files:
# - root package.json: 2.2.0
# - services/auth-service/package.json: 1.9.0
# - services/api-service/package.json: 2.1.1
# - services/worker-service/package.json: 1.5.0
```

## Handling Version Drift

**Problem:** Someone manually updated a version without coordinating, now versions don't follow parent ≥ children rule.

**Detection:**
```bash
# Check if versions are aligned
PARENT_VERSION=$(grep version plugin.json | head -1 | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")

for skill in skills/*/SKILL.md; do
  SKILL_VERSION=$(grep "^version:" "$skill" | grep -oE "[0-9]+\.[0-9]+\.[0-9]+")
  echo "Checking $skill: $SKILL_VERSION vs parent $PARENT_VERSION"
done
```

**Resolution:**
1. Identify which versions are misaligned
2. Correct them before the next release
3. Follow the multi-component release process to get back in sync
4. Document what caused drift (training opportunity for team)

## Child-Level Releases (Individual Components)

In some projects, individual components release independently. **Recommendation:** Use single release (coordinated) to prevent version confusion.

If you do allow independent releases:
- Each child maintains its own CHANGELOG
- Parent still coordinates versions at release time
- CI/CD deploys child versions separately
- Parent version is ceremonial but follows ≥ rule

**Warning:** Independent child releases + parent coordination creates complexity. Simple coordinated releases are easier to maintain.

## Git-Flow Commands in Multi-Component Context

```bash
# Release that spans multiple components
git flow release start 2.2.0
# ← Single release for entire project

# Not recommended: separate releases per component
# (Would require manual version coordination and cause version confusion)

# After finishing multi-component release
git flow feature start my-feature
# ← Features can modify any component
# ← When finished, all components' changes integrate to develop together
```

## Common Mistakes and Fixes

| Mistake | Impact | Fix |
|---|---|---|
| Update parent version, forget children | Drift develops, children older than parent | Next release: review all versions, realign |
| Update children versions, forget parent | Drift develops, parent older than children | Next release: parent version must jump to coordinate |
| Release child independently | Versioning chaos, deployment confusion | Stick to coordinated multi-component releases |
| CHANGELOG documents child changes | Parent CHANGELOG becomes cluttered | Child changes go in child CHANGELOG only |
| Merge child version commits outside release | Versions updated without release tag | All version updates should happen in release branch |

## Decision Tree: Should I Release?

```
Are there changes in any component?
├─ YES: Determine version for each (use bracket rules)
│   ├─ Calculate parent version (≥ max children)
│   └─ Start release with parent version
│       └─ Update all versions
│           └─ Finish and tag
└─ NO: No release needed
    └─ Continue development on develop
```

## Coordination Checklist

Before finishing a multi-component release, verify:

- [ ] Each changed component has new version
- [ ] Unchanged components keep current version
- [ ] Parent version ≥ max(all children versions)
- [ ] CHANGELOG.md documents parent-level changes only
- [ ] All version files are updated (plugin.json, SKILL.md frontmatter, etc.)
- [ ] git diff shows ONLY version files and CHANGELOG
- [ ] Release branch has no unrelated changes
- [ ] Team aware of new version for coordination
