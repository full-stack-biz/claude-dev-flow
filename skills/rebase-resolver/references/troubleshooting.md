# Troubleshooting Rebase Conflicts

## Table of Contents

- [Nested/Corrupted Markers](#nestedcorrupted-markers)
- [AA Conflicts (Both Added)](#aa-conflicts-both-added)
- [Orphaned Content After Partial Edit](#orphaned-content-after-partial-edit)
- [Line Numbers Shifted](#line-numbers-shifted)
- [Rerere Not Recording](#rerere-not-recording)
- [Wrong Resolution Applied](#wrong-resolution-applied)

## Nested/Corrupted Markers

**Symptom:** grep shows unbalanced markers (multiple `<<<<<<` without matching `======`)

**Cause:** Previous failed resolution left partial edits

**Fix:**
```bash
git checkout --conflict=merge <file>
```
This resets the file to its original conflict state, allowing fresh resolution.

## AA Conflicts (Both Added)

**Symptom:** `git status` shows `AA` (both added) for a file

**Cause:** File was created in both branches independently

**Resolution options:**
1. Keep HEAD version: Read HEAD content, delete entire file, Write HEAD content back
2. Keep incoming: Use `git show :3:<file>` to get incoming version
3. Manual merge: Combine both versions manually

After resolving:
```bash
git add <file>
```

## Orphaned Content After Partial Edit

**Symptom:** grep shows 0 markers but file has duplicate/orphaned code (e.g., two class definitions, orphaned `use` statements, duplicate methods)

**Cause:** When editing large conflict blocks piece by piece:
1. Removed opening marker (`<<<<<<< HEAD`) but didn't remove full incoming section
2. Removed separator (`=======`) and end marker (`>>>>>>>`) but left incoming content between them
3. Content from incoming version remains in file without any markers

**Detection:** After grep shows 0 markers, verify file structure:
```bash
# Check line count - should match expected HEAD version size
wc -l <file>

# Look for duplicate definitions
grep -n "^class \|^namespace \|^use " <file>

# Check file ends correctly
tail -20 <file>
```

**Prevention - CRITICAL:** When resolving a conflict block, ALWAYS include the FULL block in old_string:
```
old_string: From <<<<<<< HEAD through >>>>>>> commit (including ALL content between)
new_string: Only the resolved content (no markers)
```

**Fix if orphaned content exists:**
1. Identify where HEAD version should end (check structure, closing braces)
2. Use Edit tool to remove orphaned content block by block
3. Verify file compiles/parses correctly before staging

**Example of bad partial edit:**
```php
    }
}           // HEAD's closing brace
use App\... // Orphaned incoming content - NO MARKERS!
class Foo { // Duplicate class from incoming
```

## Line Numbers Shifted

**Symptom:** Grep shows markers at different lines than expected after editing

**Cause:** Previous edits changed line numbers

**Fix:** Always re-run grep after each edit to get current positions:
```bash
grep -n "^<<<<<<\|^======\|^>>>>>>" <file>
```

Process conflicts top-to-bottom to minimize shifting issues.

## Rerere Not Recording

**Symptom:** Same conflict appears again without auto-resolution

**Cause:**
- rerere.enabled is false
- Bulk operations were used (bypass rerere)
- Commit didn't happen after resolution

**Verify rerere is working:**
```bash
git config --get rerere.enabled  # Should return "true"
git rerere status                # Shows recorded resolutions
ls .git/rr-cache/               # Should have hash directories
```

**Fix:** Enable rerere and resolve conflicts block-by-block:
```bash
git config --global rerere.enabled true
```

## Wrong Resolution Applied

**Symptom:** Rerere auto-applies incorrect resolution

**Fix:**
```bash
git rerere forget <file>           # Forget the bad resolution
git checkout --conflict=merge <file>  # Reset to conflict state
# Now re-resolve correctly
```
