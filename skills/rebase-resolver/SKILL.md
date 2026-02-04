---
name: rebase-resolver
description: >-
  Resolve git rebase conflicts one block at a time with verification. Use when rebasing,
  resolving merge conflicts, or continuing a rebase. Works with git rerere. Always keeps
  HEAD by default, verifies with grep, never uses bulk operations.
allowed-tools: Read,Edit,Bash(git:*,grep:*)
---

# Rebase Conflict Resolver

Resolve git rebase conflicts systematically, one block at a time, with full verification.

## Quick Reference

```bash
# 1. Find conflicted files
git status --porcelain | grep "^UU\|^AA\|^DD"

# 2. Find markers in file
grep -n "^<<<<<<\|^======\|^>>>>>>" <file>

# 3. Resolve each block with Edit tool (one at a time)

# 4. MANDATORY: Verify before staging
grep -c "^<<<<<<\|^======\|^>>>>>>" <file> || echo "0 markers"

# 5. Stage only if grep returns 0
git add <file>

# 6. Continue rebase when all files resolved
git rebase --continue
```

## Mandatory Rules

1. **NEVER bulk operations** — No `git checkout --ours/--theirs`, no loops, no multi-file commands
2. **One conflict at a time** — Resolve each `<<<<<<<` to `>>>>>>>` block independently
3. **ALWAYS grep before staging** — `git add` FORBIDDEN until grep confirms 0 markers
4. **Respect rerere** — Block-by-block resolution enables rerere learning
5. **HEAD by default** — Keep HEAD unless operator explicitly specifies otherwise
6. **Read before edit** — Always read the conflict block first

## Prerequisites: Git Rerere

Ensure rerere is enabled (remembers resolutions for reuse):

```bash
git config --get rerere.enabled      # Check status
git config --global rerere.enabled true  # Enable if needed
```

See `references/rerere.md` for detailed documentation.

## Core Workflow

### Step 1: Identify Conflicted Files

```bash
git status --porcelain | grep "^UU\|^AA\|^DD"
```

- `UU` = Both modified
- `AA` = Both added
- `DD` = Both deleted

### Step 2: Find Conflict Markers

```bash
grep -n "^<<<<<<\|^======\|^>>>>>>" <file>
```

Process markers **one at a time, top to bottom**.

### Step 3: Resolve ONE Block

**3a. Read:** Use Read tool with offset/limit around conflict lines

**3b. Conflict structure:**
```
<<<<<<< HEAD
(HEAD version - current branch)
=======
(Incoming version - commit being rebased)
>>>>>>> commit-hash
```

**3c. Edit:** Use Edit tool — CRITICAL: include FULL block
```
old_string: ENTIRE conflict from <<<<<<< HEAD through >>>>>>> commit
           (all three markers AND all content between them)
new_string: resolved content only (no markers)
```

**WARNING:** Partial edits leave orphaned content without markers. Always include the complete block from opening marker to closing marker in a single Edit operation.

**3d. Repeat** for each conflict in file

### Step 4: Verify — MANDATORY

```bash
grep -c "^<<<<<<\|^======\|^>>>>>>" <file> || echo "0 markers"
```

**STOP if not 0. Do NOT proceed to Step 5.**

### Step 5: Stage

```bash
git add <file>
```

### Step 6: Continue

Repeat Steps 2-5 for all files, then:
```bash
git rebase --continue
```

## Resolution Patterns

### Keep HEAD (Default)
```
old_string: Full block from <<<<<<< HEAD through >>>>>>> commit
new_string: Only HEAD content (between <<<<<<< HEAD and =======)
```

### Keep Incoming
```
old_string: Full block from <<<<<<< HEAD through >>>>>>> commit
new_string: Only incoming content (between ======= and >>>>>>>)
```

### Manual Merge
Combine both versions, remove all markers.

## References

- `references/rerere.md` — Git rerere documentation
- `references/troubleshooting.md` — Common issues and fixes
