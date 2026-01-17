---
description: Stage, commit and push all code in the current branch
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
CP(1)                       User Commands                       CP(1)

NAME
    cp - Stage, commit and push all code in the current branch

SYNOPSIS
    /gh:cp [options]

DESCRIPTION
    Stage, commit and push all code in the current branch

OPTIONS
    --help, -h               Show this help message


EXAMPLES
    /gh:cp
    /gh:cp --help

SEE ALSO
    /gh:* for related commands

                                                                    CP(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

<workflow name="commit_push">
Stage, commit and push all code in the current branch:

<step number="1">
Review all modified files and their changes

<error_handling>
<error type="confidential_data" severity="HIGH">
**DO NOT** commit and push any confidential information (such as dotenv files, API keys, database credentials, etc.) to git repository!
</error>
</error_handling>
</step>

<step number="2">
Generate a clear, descriptive commit message summarizing the changes.
    Follow convention commit rules (eg. `fix`, `feat`, `perf`, `refactor`, `docs`, `style`, `ci`, `chore`, `build`, `test`)

<conditional>
    Any changes related to Markdown files in `.claude/` should be using `perf:` (instead of `docs:`)
    New files in `.claude/` directory should be using `feat:` (instead of `docs:` or `perf:`)
    Commit title should be less than 70 characters.
    If there are new files and file changes at the same time, split them into separate commits.
    Commit body should be a summarized list of key changes.
</conditional>

<error_handling>
<error type="ai_attribution" severity="MEDIUM">
    NEVER automatically add AI attribution signatures like:
    - "🤖 Generated with [Claude Code]"
    - "Co-Authored-By: Claude noreply@anthropic.com"
    - Any AI tool attribution or signature
    - Create clean, professional commit messages without AI references.
</error>
</error_handling>
</step>

<step number="3">
Stage all modified files using git add & commit the changes using: git commit -m "commit_message".
    - Split files into separate commits to reflect the changes.
</step>

<step number="4">
Confirm the commit was successful and display the resulting commit hash and message.
</step>

<step number="5">
Push the commit to the origin remote on the current branch.
</step>
</workflow>