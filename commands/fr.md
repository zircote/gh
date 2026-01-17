---
description: Fetch from remote and rebase current branch onto the remote branch
argument-hint: "[remote] [branch]"
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
FR(1)                           User Commands                           FR(1)

NAME
    fr - Fetch and rebase current branch onto remote

SYNOPSIS
    /gh:fr [remote] [branch]

DESCRIPTION
    Fetch from remote and rebase the current branch onto the remote branch.
    Handles conflicts interactively if they occur. Stash uncommitted changes
    before running if working directory is dirty.

OPTIONS
    [remote]                  Remote name (default: origin)
    [branch]                  Branch to rebase onto (default: upstream)
    --help, -h                Show this help message

EXAMPLES
    /gh:fr                    Fetch and rebase from origin (tracking branch)
    /gh:fr upstream           Fetch and rebase from upstream remote
    /gh:fr origin main        Fetch and rebase onto origin/main

SEE ALSO
    /gh:ff              Fast-forward only (no rebase)
    /gh:sync            Full sync with push after rebase

                                                                        FR(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

Fetch and rebase the current branch onto the specified remote branch:

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
   > A) Stash changes, proceed with rebase, then pop stash
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

<workflow name="fetch_rebase">
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
</step>

<step number="4">
**Report success**:
   <conditional>
   If rebase completed successfully:
   > "Successfully rebased onto ${REMOTE}/${BRANCH}"
   </conditional>

   Show current status:
   ```bash
   git log --oneline -5
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
- **DO NOT** force push after rebase unless explicitly requested
- This command only fetches and rebases—it does not push (use `/git:sync` for full cycle)
