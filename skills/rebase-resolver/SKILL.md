---
name: rebase-resolver
description: >-
  Resolve git rebase conflicts one block at a time with verification. Use when rebasing,
  resolving merge conflicts, or continuing a rebase. Works with git rerere. Always keeps
  HEAD by default, verifies with grep, never uses bulk operations. Delegates file edits to subagents.
allowed-tools: Read,Bash(git:*,grep:*),Task
---

# Rebase Conflict Resolver

Resolve git rebase conflicts systematically, one block at a time, with full verification.
Delegates individual file resolution to subagents for reduced context usage.

## Quick Reference

```bash
# 1. Find conflicted files
git status --porcelain | grep "^UU\|^AA\|^DD"

# 2. For each file, spawn a subagent (see Subagent Delegation below)

# 3. After all files resolved, continue rebase
git rebase --continue
```

## Mandatory Rules

1. **NEVER bulk operations** — No `git checkout --ours/--theirs`, no loops, no multi-file commands
2. **One conflict at a time** — Resolve each `<<<<<<<` to `>>>>>>>` block independently
3. **ALWAYS grep before staging** — `git add` FORBIDDEN until grep confirms 0 markers
4. **Respect rerere** — Block-by-block resolution enables rerere learning
5. **HEAD by default** — Keep HEAD unless operator explicitly specifies otherwise
6. **Read before edit** — Always read the conflict block first
7. **No time commentary** — Never comment on duration, tedium, or effort
8. **No alternative offers** — Never ask "would you prefer a different approach?" or suggest shortcuts
9. **Focus on current block only** — Don't project ahead about total remaining work
10. **Delegate to subagents** — Use Task tool for file-level conflict resolution

## Whole-File Conflicts (Optimization)

When a conflict spans the **entire file**, use `git show :2:` to extract HEAD directly.
This avoids token-expensive Edit operations on large files.

### Detection

```bash
# Whole-file = first line is <<<<<<<, last line is >>>>>>>, exactly 3 markers
first=$(head -1 {FILE} | grep -c "^<<<<<<<")
last=$(tail -1 {FILE} | grep -c "^>>>>>>>")
markers=$(grep -c "^<<<<<<\|^======\|^>>>>>>" {FILE})

[ "$first" -eq 1 ] && [ "$last" -eq 1 ] && [ "$markers" -eq 3 ] && echo "whole-file"
```

### Resolution (No Subagent Needed)

```bash
# Stage 2 = HEAD version during rebase
git show :2:{FILE} > {FILE}
git add {FILE}
```

**Benefits:** Zero tokens for resolution. Works for any file size.

## Subagent Delegation

For files with **partial conflicts** (not whole-file), spawn a **general-purpose** subagent.

### Subagent Prompt Template

```
Resolve git conflicts in: {FILE_PATH}

RESOLUTION STRATEGY: Always keep HEAD version (between <<<<<<< HEAD and =======)

WORKFLOW:
1. Run: grep -n "^<<<<<<\|^======\|^>>>>>>" {FILE_PATH}
2. Read the file around each conflict
3. For EACH conflict block (top to bottom):
   - Use Edit tool with old_string = ENTIRE block from <<<<<<< HEAD through >>>>>>> commit
   - new_string = HEAD content only (no markers)
4. After ALL conflicts resolved, verify: grep -c "^<<<<<<\|^======\|^>>>>>>" {FILE_PATH} || echo "0 markers"
5. If 0 markers, stage: git add {FILE_PATH}
6. Report: "Resolved {N} conflicts in {FILE_PATH}, staged successfully" or any errors

RULES:
- One conflict at a time, top to bottom
- Never use bulk operations (no git checkout --ours/--theirs)
- Include FULL block in Edit (all 3 markers + content) to avoid orphans
- Do NOT continue rebase - just resolve and stage this file
```

### Orchestrator Workflow

1. **Identify conflicts:**
   ```bash
   git status --porcelain | grep "^UU\|^AA\|^DD"
   ```

2. **For each file, detect whole-file vs partial:**
   ```bash
   first=$(head -1 {FILE} | grep -c "^<<<<<<<")
   last=$(tail -1 {FILE} | grep -c "^>>>>>>>")
   markers=$(grep -c "^<<<<<<\|^======\|^>>>>>>" {FILE})
   [ "$first" -eq 1 ] && [ "$last" -eq 1 ] && [ "$markers" -eq 3 ] && echo "whole-file"
   ```

3. **If whole-file:** Resolve directly (no subagent):
   ```bash
   git show :2:{FILE} > {FILE}
   git add {FILE}
   ```

4. **If partial conflicts:** Spawn subagent:
   ```
   Task tool:
     subagent_type: general-purpose
     prompt: [use template above with actual file path]
     description: "Resolve conflicts in {filename}"
   ```

5. **Parallel execution:** Run whole-file resolutions in parallel with Bash.
   Spawn subagents in parallel for partial conflicts.

6. **After all complete**, verify overall status:
   ```bash
   git status --porcelain | grep "^UU\|^AA\|^DD" || echo "All resolved"
   ```

7. **Continue rebase:**
   ```bash
   git rebase --continue
   ```

## AA Conflicts (Both Added)

For AA conflicts where both versions added the same file:
- Often identical content (rerere may auto-resolve)
- Subagent should compare versions: if identical, keep HEAD
- If different, keep HEAD unless operator specifies otherwise

## Prerequisites: Git Rerere

Ensure rerere is enabled (remembers resolutions for reuse):

```bash
git config --get rerere.enabled      # Check status
git config --global rerere.enabled true  # Enable if needed
```

See `references/rerere.md` for detailed documentation.

## Manual Resolution (Without Subagents)

If needed, resolve directly following these steps:

### Step 1: Find Conflict Markers

```bash
grep -n "^<<<<<<\|^======\|^>>>>>>" <file>
```

Process markers **one at a time, top to bottom**.

### Step 2: Resolve ONE Block

**2a. Read:** Use Read tool with offset/limit around conflict lines

**2b. Conflict structure:**
```
<<<<<<< HEAD
(HEAD version - current branch)
=======
(Incoming version - commit being rebased)
>>>>>>> commit-hash
```

**2c. Edit:** Use Edit tool — CRITICAL: include FULL block
```
old_string: ENTIRE conflict from <<<<<<< HEAD through >>>>>>> commit
           (all three markers AND all content between them)
new_string: resolved content only (no markers)
```

**WARNING:** Partial edits leave orphaned content without markers. Always include the complete block from opening marker to closing marker in a single Edit operation.

**2d. Repeat** for each conflict in file

### Step 3: Verify — MANDATORY

```bash
grep -c "^<<<<<<\|^======\|^>>>>>>" <file> || echo "0 markers"
```

**STOP if not 0. Do NOT proceed to staging.**

### Step 4: Stage

```bash
git add <file>
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
