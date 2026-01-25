# Branch Lifecycle in Git-Flow

This guide tracks the complete lifecycle of each branch type and when they exist in your repository.

## Feature Branches

**Timeline:** Local development → Shared PR review → Merged to develop → Deleted

```
START: developer creates from develop
  ↓
feature/user-auth ← Branch exists here (local and remote)
  ↓ (developers commit, push)
SHARED: Code review on GitHub/GitLab
  ↓
Approved, ready to merge
  ↓
FINISH: git flow feature finish
  ├─ Merged into develop
  ├─ Branch deleted
  └─ Developer switches to develop
```

**Key points:**
- Never merge feature branches directly to `main`
- Always finish through `develop`
- Safe to delete after finishing (history preserved in `develop`)

## Release Branches

**Timeline:** Preparation from develop → Testing → Merged to main & develop → Tagged

```
START: Developer ready to release
  ├─ git flow release start 1.2.0
  └─ Branched from: develop

release/1.2.0 ← Branch exists here
  ↓
VERSION UPDATES: Update version in all files
  ├─ plugin.json
  ├─ SKILL.md frontmatter
  ├─ CHANGELOG.md
  └─ Any other versioned files

TESTING: QA tests, final verification
  ↓
FINISH: git flow release finish 1.2.0
  ├─ Merged into main ✓
  ├─ Tagged main with v1.2.0 ✓
  ├─ Merged back into develop ✓
  ├─ Branch deleted
  └─ Developer switches to develop

DEPLOY: git push origin main develop --tags
  └─ main branch deployed to production
```

**Critical:** Release branches always merge to BOTH `main` and `develop` to keep them in sync.

## Hotfix Branches

**Timeline:** Critical bug fix from main → Merged to main & develop → Tagged

```
START: Production bug discovered
  ├─ git flow hotfix start 1.2.1
  └─ Branched from: main (NOT develop!)

hotfix/1.2.1 ← Branch exists here
  ↓
BUG FIX: Make minimal fix
  ├─ Only fix the bug (no features)
  ├─ Update version: 1.2.0 → 1.2.1
  ├─ Update CHANGELOG.md
  └─ Commit changes

FINISH: git flow hotfix finish 1.2.1
  ├─ Merged into main ✓
  ├─ Tagged main with v1.2.1 ✓
  ├─ Merged back into develop ✓
  ├─ Branch deleted
  └─ Developer switches to develop

DEPLOY: git push origin main develop --tags
  └─ main branch deployed immediately
```

**Critical:** Hotfixes branch from `main`, not `develop`. This prevents untested features from reaching production.

## Develop Branch

**Timeline:** Always exists, receives merges from features, releases, and hotfixes

```
develop ← Always exists and is protected
  ↑
  ├─ Feature merges (feature/X → develop)
  ├─ Release merges (release/X.Y.Z → develop)
  └─ Hotfix merges (hotfix/X.Y.Z → develop)

Long-lived branch:
  • Developers create features FROM develop
  • Integrate work through develop
  • Source for next release
```

**Role:** Integration branch. `develop` is where features come together before release.

## Main Branch

**Timeline:** Always exists, receives merges from releases and hotfixes, always deployable

```
main ← Always exists and is protected
  ↑
  ├─ Release merges (release/X.Y.Z → main)
  └─ Hotfix merges (hotfix/X.Y.Z → main)

Long-lived branch:
  • Tags mark releases: v1.0.0, v1.1.0, v1.2.0, v1.2.1...
  • Always production-ready
  • Deploy directly from this branch
```

**Rule:** Only releases and hotfixes merge to `main`. Never merge features directly.

## Branch Existence Checklist

| Branch Type | Always Exists? | Protection | Source | Destinations |
|---|---|---|---|---|
| `main` | ✓ Yes | Protected | Release, Hotfix | (None—terminal) |
| `develop` | ✓ Yes | Protected | Features, Release, Hotfix | (None—terminal for merges) |
| `feature/*` | ✗ Temporary | No | `develop` | `develop` (merged) |
| `release/*` | ✗ Temporary | Suggested | `develop` | `main` + `develop` |
| `hotfix/*` | ✗ Temporary | Suggested | `main` | `main` + `develop` |

## Concurrent Operations

Multiple branches can exist simultaneously:

```
Scenario: Team of 3 working in parallel

develop ← Base integration branch
  ├─ feature/user-auth (developer A)
  ├─ feature/payment (developer B)
  ├─ feature/notifications (developer C)
  └─ release/1.3.0 (developer A, preparing next release)

main ← Production (currently v1.2.0)
```

**Operations:**
- Developer A: Can finish `feature/user-auth`, start working on `release/1.3.0`
- Developer B: Continues on `feature/payment`, creates PR when ready
- Developer C: Working on `feature/notifications`
- Release lead: Manages `release/1.3.0` preparation, version updates

**Note:** A developer can work on at most one git-flow branch at a time (can only be checked out once). New work requires finishing the previous branch first.

## State Transitions

```
Fresh repository (after git flow init)
  ├─ main exists (empty history)
  └─ develop exists (empty history)

First release prep
  develop (v0.0.0) → release/1.0.0 → main (tagged v1.0.0)

Steady state (production running)
  main (v1.2.0) ← Tagged release point
  develop (v1.3.0-dev) ← Next features integrate here
  feature/X, feature/Y, feature/Z ← Parallel work

Production bug
  hotfix/1.2.1 ← From main (urgent fix)
  → main (tagged v1.2.1)
  → develop (synced)

Resume feature work
  develop ← Hotfix now available
  feature/X continues
```
