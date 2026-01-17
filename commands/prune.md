---
description: Clean up local branches whose remote tracking branch is gone. Dry-run by default
argument-hint: "[--force]"
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
PRUNE(1)                        User Commands                        PRUNE(1)

NAME
    prune - Clean up stale local branches (dry-run by default)

SYNOPSIS
    /gh:prune [--force]

DESCRIPTION
    Find and optionally delete local branches whose remote tracking branch
    no longer exists (marked as [gone]). Runs in dry-run mode by default,
    listing branches that would be deleted without making changes.

OPTIONS
    --force                   Actually delete branches (default: dry-run)
    --help, -h                Show this help message

EXAMPLES
    /gh:prune                 List stale branches (dry-run)
    /gh:prune --force         Delete stale branches after confirmation

SEE ALSO
    /gh:sync            Full sync with remote
    /gh:fr              Fetch and rebase

                                                                    PRUNE(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

Find and optionally delete local branches whose remote tracking branch no longer exists:

## Variables

- **MODE**: If `--force` is provided, delete branches. Otherwise, dry-run (list only).

## Protected Branches

The following branches are NEVER deleted, even with `--force`:
- `main`
- `master`
- `develop`
- `development`
- The current branch (cannot delete checked-out branch)

<workflow name="branch_pruning">
## Workflow

<step number="1">
**Fetch with prune to update remote tracking info**:
   ```bash
   git fetch --prune
   ```

   This removes remote-tracking references that no longer exist on the remote.
</step>

<step number="2">
**Find branches with "gone" upstream**:
   ```bash
   git branch -vv | grep ': gone]'
   ```

   Parse the output to extract branch names.
</step>

<step number="3">
**Filter out protected branches**:
   Remove `main`, `master`, `develop`, `development`, and current branch from the list.
</step>

<step number="4">
**Display stale branches**:
   <conditional>
   If stale branches found:
   > **Stale branches found:**
   > - `feature/old-feature` (upstream was `origin/feature/old-feature`)
   > - `bugfix/resolved-issue` (upstream was `origin/bugfix/resolved-issue`)
   > ...

   If no stale branches:
   > "No stale branches found. Your local branches are in sync with remote."
   Stop here.
   </conditional>
</step>

<step number="5">
**Handle based on mode**:

   <conditional>
   **If dry-run (no `--force`):**
   > "These branches would be deleted. Run `/git:prune --force` to delete them."

   Stop here—do not delete anything.

   **If `--force` provided:**
   > "You are about to delete these ${N} branches. This cannot be undone.
   > Proceed? (y/n)"

   Wait for explicit user confirmation.
   </conditional>
</step>

<step number="6">
**Delete branches (only if --force AND user confirmed)**:
   For each branch:
   ```bash
   git branch -d ${BRANCH_NAME}
   ```

   <error_handling>
   <error type="not_fully_merged" severity="MEDIUM">
   If `-d` fails (branch not fully merged), report:
   > "Branch `${BRANCH_NAME}` is not fully merged. Use `git branch -D ${BRANCH_NAME}` to force delete."

   Ask user if they want to force delete unmerged branches.
   </error>
   </error_handling>
</step>

<step number="7">
**Report results**:
   > **Cleanup complete:**
   > - Deleted: `feature/old-feature`, `bugfix/resolved-issue`
   > - Skipped (not fully merged): `feature/wip-changes`
   > - Protected (not touched): `main`, `develop`
</step>
</workflow>

<error_handling>
## Edge Cases

<error type="no_stale_branches" severity="MEDIUM">
### No stale branches
> "No stale branches found. Your local branches are in sync with remote."
</error>

<error type="all_protected" severity="MEDIUM">
### All stale branches are protected
> "Found stale branches, but they are all protected (main/master/develop). No action taken."
</error>

<error type="unmerged_commits" severity="HIGH">
### Branch not fully merged
When using `git branch -d` and it fails:
> "Branch `${BRANCH}` has unmerged commits. These commits exist only on this branch:
> ```
> git log main..${BRANCH} --oneline
> ```
> To force delete anyway: `git branch -D ${BRANCH}`
> Warning: This will permanently lose these commits."
</error>
</error_handling>

## Notes

- **Dry-run is the default** - this command will NEVER delete branches without `--force`
- Even with `--force`, you must confirm before any deletion
- Protected branches are never deleted regardless of flags
- Use this command after merging PRs to clean up feature branches
- This only affects LOCAL branches—it does not delete anything on the remote
