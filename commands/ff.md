---
description: Fast-forward merge only - update branch without rebase or merge commits
argument-hint: "[remote] [branch]"
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
FF(1)                           User Commands                           FF(1)

NAME
    ff - Fast-forward merge only (no rebase, no merge commits)

SYNOPSIS
    /gh:ff [remote] [branch]

DESCRIPTION
    Attempt a fast-forward merge from the remote branch. If fast-forward
    is not possible (diverged history), the command fails rather than
    creating a merge commit. Use /gh:fr for rebase-based updates.

OPTIONS
    [remote]                  Remote name (default: origin)
    [branch]                  Branch to merge from (default: upstream)
    --help, -h                Show this help message

EXAMPLES
    /gh:ff                    Fast-forward from origin (tracking branch)
    /gh:ff upstream           Fast-forward from upstream remote
    /gh:ff origin main        Fast-forward from origin/main

SEE ALSO
    /gh:fr              Fetch and rebase (for diverged branches)
    /gh:sync            Full sync with push

                                                                        FF(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

Attempt a fast-forward merge from the remote branch (no rebase, no merge commits):

## Variables

- **REMOTE**: First argument, defaults to `origin`
- **BRANCH**: Second argument, defaults to current branch's upstream tracking branch

<preflight_checks>
## Pre-flight Checks

<check>
1. **Check for uncommitted changes**:
   ```bash
   git status --porcelain
   ```

   <error_handling>
   <error type="uncommitted_changes" severity="HIGH">
   If there are uncommitted changes:
   > "You have uncommitted changes. Fast-forward merge requires a clean working directory.
   > Please commit or stash your changes first."

   Do not proceed until working directory is clean.
   </error>
   </error_handling>
</check>

<check>
2. **Determine target branch**:
   - If BRANCH not specified, get upstream:
     ```bash
     git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null
     ```
   - If no upstream tracking branch, ask user which branch to merge from
</check>
</preflight_checks>

<workflow name="fast_forward">
## Workflow

<step number="1">
**Fetch from remote**:
   ```bash
   git fetch ${REMOTE}
   ```
</step>

<step number="2">
**Check if fast-forward is possible**:
   ```bash
   git merge-base --is-ancestor HEAD ${REMOTE}/${BRANCH}
   ```

   <conditional>
   If exit code is 0: fast-forward is possible
   If exit code is non-0: fast-forward is NOT possible
   </conditional>
</step>

<step number="3">
**If fast-forward possible, perform merge**:
   <conditional>
   ```bash
   git merge --ff-only ${REMOTE}/${BRANCH}
   ```

   Report success:
   > "Successfully fast-forwarded to ${REMOTE}/${BRANCH}"

   Show new commits:
   ```bash
   git log --oneline -10
   ```
   </conditional>
</step>

<step number="4">
<error_handling>
<error type="ff_not_possible" severity="MEDIUM">
**If fast-forward NOT possible**:
   > **Fast-forward not possible**
   >
   > Your branch has diverged from ${REMOTE}/${BRANCH}. This means you have local commits that aren't on the remote.
   >
   > Your options:
   >
   > 1. **Rebase** (recommended for feature branches):
   >    ```bash
   >    /git:fr
   >    ```
   >    This replays your commits on top of the remote branch.
   >
   > 2. **Merge** (creates a merge commit):
   >    ```bash
   >    git merge ${REMOTE}/${BRANCH}
   >    ```
   >    This preserves both histories with a merge commit.
   >
   > 3. **Reset** (discard local commits - destructive!):
   >    ```bash
   >    git reset --hard ${REMOTE}/${BRANCH}
   >    ```
   >    **Warning**: This discards your local commits permanently.
</error>
</error_handling>
</step>
</workflow>

## Notes

- Fast-forward is the safest way to update a branch—no history rewriting, no merge commits
- Ideal for pulling updates on branches you haven't modified locally
- If you need to incorporate changes from a branch you've worked on, use `/git:fr` instead
- This command never creates merge commits or rewrites history
