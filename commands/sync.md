---
description: Full sync - fetch, rebase onto remote branch, and push
argument-hint: "[remote] [branch]"
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
SYNC(1)                         User Commands                         SYNC(1)

NAME
    sync - Full sync: fetch, rebase, and push

SYNOPSIS
    /gh:sync [remote] [branch]

DESCRIPTION
    Perform a full sync cycle: fetch from remote, rebase current branch
    onto the remote branch, and push the updated branch. Handles conflicts
    interactively if they occur.

OPTIONS
    [remote]                  Remote name (default: origin)
    [branch]                  Branch to sync with (default: upstream)
    --help, -h                Show this help message

EXAMPLES
    /gh:sync                  Sync with origin (tracking branch)
    /gh:sync upstream         Sync with upstream remote
    /gh:sync origin main      Sync with origin/main

SEE ALSO
    /gh:fr              Fetch and rebase (without push)
    /gh:ff              Fast-forward only
    /gh:cp              Stage, commit, and push

                                                                      SYNC(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

Perform a full sync cycle: fetch from remote, rebase current branch, and push:

## Variables

- **REMOTE**: First argument, defaults to `origin`
- **BRANCH**: Second argument, defaults to current branch's upstream tracking branch

If user previously specified a different remote in this session, use that as the default instead of `origin`.

<preflight_checks>
## Pre-flight Checks

<check>
1. **Check for uncommitted changes**:
   ```bash
   git status --porcelain
   ```

   <conditional>
   If there are uncommitted changes, display a warning with the file list and ask the user:
   > "You have uncommitted changes. How would you like to proceed?
   > A) Stash changes, proceed with sync, then pop stash
   > B) Abort and let me commit/stash manually first
   > C) Proceed anyway (risky - changes may conflict)"

   Wait for user choice before continuing.
   </conditional>
</check>

<check>
2. **Check for existing rebase in progress**:
   ```bash
   ls .git/rebase-merge .git/rebase-apply 2>/dev/null
   ```

   <error_handling>
   <error type="rebase_in_progress" severity="HIGH">
   If rebase is in progress, inform user and offer:
   > "A rebase is already in progress. Options:
   > - `git rebase --continue` (after resolving conflicts)
   > - `git rebase --abort` (to cancel and restore original state)"
   </error>
   </error_handling>
</check>

<check>
3. **Determine target branch**:
   - If BRANCH not specified, get upstream:
     ```bash
     git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null
     ```
   - If no upstream tracking branch, ask user which branch to sync with
</check>
</preflight_checks>

<workflow name="full_sync">
## Workflow

<step number="1">
**Fetch from remote**:
   ```bash
   git fetch ${REMOTE}
   ```
   Display fetch output to show what was retrieved.
</step>

<step number="2">
**Show divergence** (informational):
   ```bash
   git log --oneline HEAD..${REMOTE}/${BRANCH} | head -10
   ```
   Display incoming commits (if any).
</step>

<step number="3">
**Rebase onto remote branch**:
   ```bash
   git rebase ${REMOTE}/${BRANCH}
   ```

   <conditional>
   If conflicts occur, stop and provide conflict resolution guidance (see below).
   </conditional>
</step>

<step number="4">
**Confirm before push**:
   After successful rebase, show what will be pushed:
   ```bash
   git log --oneline ${REMOTE}/${BRANCH}..HEAD
   ```

   Ask user:
   > "Ready to push these commits to ${REMOTE}/${BRANCH}? (y/n)"

   Only proceed with push if user confirms.
</step>

<step number="5">
**Push to remote**:
   ```bash
   git push ${REMOTE} HEAD
   ```

   <error_handling>
   <error type="push_rejected" severity="HIGH">
   If push fails due to non-fast-forward:
   > "Push rejected - remote has new commits. This can happen if someone else pushed while you were rebasing.
   > Options:
   > - Run `/git:sync` again to incorporate new remote changes
   > - Use `git push --force-with-lease` (only if you're certain)"
   </error>
   </error_handling>
</step>

<step number="6">
**Report success**:
   > "Successfully synced with ${REMOTE}/${BRANCH}"

   Show final status:
   ```bash
   git status
   ```
</step>
</workflow>

<error_handling>
## Conflict Resolution

<error type="rebase_conflict" severity="HIGH">
If the rebase encounters conflicts:

> **Rebase Conflict Detected**
>
> The rebase stopped due to conflicts. To resolve:
>
> 1. Open each conflicted file and resolve the conflicts
>    - Look for `<<<<<<<`, `=======`, and `>>>>>>>` markers
>    - Keep the changes you want, remove the markers
>
> 2. After resolving each file:
>    ```bash
>    git add <resolved-file>
>    ```
>
> 3. Continue the rebase:
>    ```bash
>    git rebase --continue
>    ```
>
> 4. After rebase completes, I'll continue with the push step.
>
> **To abort and restore original state:**
> ```bash
> git rebase --abort
> ```
>
> Tell me which file you need help with and I'll analyze the conflicting changes.
</error>
</error_handling>

## Notes

- If user specifies a custom REMOTE, remember it for subsequent `/git:fr` or `/git:sync` calls in this session
- Always confirm before pushing to avoid accidental pushes to shared branches
- **DO NOT** use `--force` push unless explicitly requested by user
- This command performs a full sync cycle; use `/git:fr` if you only want fetch+rebase without push
