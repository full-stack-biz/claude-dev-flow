# Git Rerere — Reuse Recorded Resolution

Reference: https://git-scm.com/book/en/v2/Git-Tools-Rerere

## Table of Contents

- [What is Rerere?](#what-is-rerere)
- [Why Use Rerere?](#why-use-rerere)
- [Enabling Rerere](#enabling-rerere)
- [How It Works](#how-it-works)
- [Rerere Commands](#rerere-commands)
- [Block-by-Block Resolution & Rerere](#block-by-block-resolution--rerere)
- [Example: Rerere in Action](#example-rerere-in-action)
- [Verifying Rerere Is Working](#verifying-rerere-is-working)
- [Troubleshooting](#troubleshooting)

## What is Rerere?

Rerere stands for "reuse recorded resolution." It allows Git to remember how you resolved a hunk conflict so that the next time it sees the same conflict, Git can resolve it automatically.

## Why Use Rerere?

1. **Long-running rebases** — When rebasing many commits, the same conflict may appear multiple times. Rerere records your resolution once and applies it automatically afterward.

2. **Branch maintenance** — When maintaining topic branches that need periodic merging with main, rerere remembers resolutions across merge attempts.

3. **Testing merges** — You can do a test merge, resolve conflicts, then undo it. Later when you do the real merge, rerere applies your resolutions automatically.

## Enabling Rerere

**Check current status:**
```bash
git config --get rerere.enabled
```

**Enable globally (recommended):**
```bash
git config --global rerere.enabled true
```

**Enable for current repo only:**
```bash
git config rerere.enabled true
```

## How It Works

1. **Recording:** When you resolve a merge conflict and commit, rerere records:
   - The "before" state (the conflict)
   - The "after" state (your resolution)

2. **Replaying:** When the same conflict appears again, rerere:
   - Recognizes the conflict pattern
   - Automatically applies your previous resolution
   - Shows `Resolved 'filename' using previous resolution.`

3. **Storage:** Resolutions are stored in `.git/rr-cache/`

## Rerere Commands

**See what rerere has recorded:**
```bash
git rerere status
```

**See the current state of resolution:**
```bash
git rerere diff
```

**Forget a specific resolution:**
```bash
git rerere forget <pathspec>
```

**Clear all recorded resolutions:**
```bash
git rerere gc
```

## Block-by-Block Resolution & Rerere

**Why block-by-block matters for rerere:**

When you resolve conflicts one block at a time using Edit tool:
- Rerere records each distinct conflict pattern
- The same pattern in another file gets auto-resolved
- Similar patterns in future rebases get auto-resolved

When you use bulk operations like `git checkout --ours`:
- Rerere does NOT learn the resolution
- Future conflicts require manual resolution again
- No pattern recognition occurs

## Example: Rerere in Action

```
# First encounter - manual resolution
CONFLICT (content): Merge conflict in config.php
# You resolve block by block with Edit tool
git add config.php
git commit

# Second encounter - same conflict pattern
Auto-merging config.php
Resolved 'config.php' using previous resolution.
# No manual work needed!
```

## Verifying Rerere Is Working

After resolving a conflict, check that rerere recorded it:
```bash
git rerere status
# Should show the file you just resolved

ls .git/rr-cache/
# Should show hash directories with recorded resolutions
```

## Troubleshooting

**Rerere not recording:**
- Ensure `rerere.enabled` is `true`
- Check that you committed after resolving (rerere records on commit)

**Wrong resolution applied:**
```bash
git rerere forget <file>  # Forget the bad resolution
git checkout --conflict=merge <file>  # Reset to conflict state
# Now re-resolve correctly
```

**Partial auto-resolution:**
Sometimes rerere resolves some hunks but not others. Remaining conflicts need manual resolution — this is normal when conflicts partially match.