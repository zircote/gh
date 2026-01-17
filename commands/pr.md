---
description: Create, update, or manage pull requests (creates drafts by default)
argument-hint: [to-branch] [--ready] [--update] [--web] [--fill]
allowed-tools: Bash(*), Read, Glob, Grep
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
PR(1)                           User Commands                           PR(1)

NAME
    pr - Create, update, or manage pull requests (drafts by default)

SYNOPSIS
    /gh:pr [to-branch] [--ready] [--update] [--web] [--fill] [--no-draft]

DESCRIPTION
    Create, update, or manage pull requests. Creates draft PRs by default
    to encourage iterative development and early feedback. Use --ready to
    convert a draft to ready-for-review, or --update to modify an existing PR.

OPTIONS
    [to-branch]               Target branch (defaults to main or repo default)
    --ready                   Convert existing draft PR to ready for review
    --update                  Update an existing PR (title, body, or commits)
    --web                     Open PR creation in browser
    --fill                    Auto-fill title/body from commit messages
    --no-draft                Create as ready-for-review instead of draft
    --help, -h                Show this help message

EXAMPLES
    /gh:pr                    Create draft PR to main
    /gh:pr develop            Create draft PR to develop branch
    /gh:pr --ready            Convert draft to ready for review
    /gh:pr --update           Update existing PR with new commits
    /gh:pr --no-draft         Create ready-for-review PR (skip draft)
    /gh:pr --fill --web       Auto-fill and open in browser

SEE ALSO
    /gh:pr-fix          Complete PR remediation workflow
    /gh:review-comments PR comment review
    /gh:cp              Stage, commit, and push

                                                                        PR(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

# Pull Request Management

Create, update, or manage pull requests. **Creates draft PRs by default** to encourage code review workflows.

## Arguments

Parse `$ARGUMENTS` for:
- **TO_BRANCH**: Target branch (defaults to `main` or repo default)
- **`--ready`**: Convert existing draft PR to ready for review
- **`--update`**: Update an existing PR (title, body, or add commits)
- **`--web`**: Open PR creation in browser
- **`--fill`**: Auto-fill title/body from commit messages
- **`--no-draft`**: Create as ready-for-review instead of draft

<preflight_checks>
## Pre-flight Checks

<check>
### 1. Verify `gh` CLI is available

```bash
which gh
```

<error_handling>
<error type="gh_not_found" severity="HIGH">
If not found:
> **GitHub CLI Required**
>
> Install the `gh` command:
> - macOS: `brew install gh`
> - Linux: `sudo apt install gh` or see https://github.com/cli/cli#installation
> - Windows: `winget install GitHub.cli`
>
> Then authenticate: `gh auth login`
</error>
</error_handling>
</check>

<check>
### 2. Verify authentication

```bash
gh auth status
```

<error_handling>
<error type="not_authenticated" severity="HIGH">
If not authenticated:
> **GitHub CLI Not Authenticated**
>
> Run `gh auth login` to authenticate with GitHub.
</error>
</error_handling>
</check>

<check>
### 3. Get current branch and remote info

```bash
CURRENT_BRANCH=$(git branch --show-current)
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
echo "Branch: $CURRENT_BRANCH"
echo "Repo: $REPO"
```
</check>

<check>
### 4. Check for existing PR

```bash
EXISTING_PR=$(gh pr list --head "$CURRENT_BRANCH" --json number,state,isDraft,url --jq '.[0]')
if [ -n "$EXISTING_PR" ]; then
  echo "Existing PR found: $EXISTING_PR"
fi
```
</check>
</preflight_checks>

---

<workflow name="ready_conversion">
# WORKFLOW: --ready (Convert Draft to Ready)

<conditional>
If `--ready` flag is provided:
</conditional>

<step number="1">
```bash
# Get current PR for this branch
PR_NUMBER=$(gh pr list --head "$(git branch --show-current)" --json number -q '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "ERROR: No PR found for current branch"
  exit 1
fi

# Check if it's a draft
IS_DRAFT=$(gh pr view "$PR_NUMBER" --json isDraft -q '.isDraft')

if [ "$IS_DRAFT" = "true" ]; then
  echo "Converting PR #$PR_NUMBER from draft to ready for review..."
  gh pr ready "$PR_NUMBER"
  echo ""
  gh pr view "$PR_NUMBER" --web
else
  echo "PR #$PR_NUMBER is already marked as ready for review"
  gh pr view "$PR_NUMBER"
fi
```

<error_handling>
<error type="no_pr_found" severity="HIGH">
If no PR exists for current branch, display error and exit
</error>
</error_handling>
</step>
</workflow>

---

<workflow name="update_pr">
# WORKFLOW: --update (Update Existing PR)

<conditional>
If `--update` flag is provided:
</conditional>

<step number="1">
## 1. Verify PR exists

```bash
PR_NUMBER=$(gh pr list --head "$(git branch --show-current)" --json number -q '.[0].number')

if [ -z "$PR_NUMBER" ]; then
  echo "ERROR: No PR found for current branch. Create one first with /gh:pr"
  exit 1
fi

echo "Found PR #$PR_NUMBER"
gh pr view "$PR_NUMBER" --json title,body,state,isDraft
```

<error_handling>
<error type="no_pr_found" severity="HIGH">
If no PR exists for current branch, display error and exit
</error>
</error_handling>
</step>

<step number="2">
## 2. Check for new commits

```bash
echo ""
echo "=== Commits not yet in PR ==="
git log origin/$(git branch --show-current)..HEAD --oneline 2>/dev/null || echo "All commits are pushed"
```
</step>

<step number="3">
## 3. Push new commits (if any)

```bash
git push origin $(git branch --show-current)
```
</step>

<step number="4">
## 4. Optionally update title/body

Use AskUserQuestion:
- question: "What would you like to update on PR #$PR_NUMBER?"
- header: "Update"
- multiSelect: true
- options:
  - label: "Push commits only"
    description: "Just push new commits, don't change title/body"
  - label: "Update title"
    description: "Change the PR title"
  - label: "Update body"
    description: "Regenerate PR description from commits"
  - label: "Add comment"
    description: "Add a comment to the PR"

<conditional>
If updating title:
</conditional>
```bash
# Generate new title from latest commit or prompt
NEW_TITLE=$(git log -1 --format="%s")
gh pr edit "$PR_NUMBER" --title "$NEW_TITLE"
```

<conditional>
If updating body:
</conditional>
```bash
# Regenerate body from all commits in PR
gh pr edit "$PR_NUMBER" --body "$(git log origin/main..HEAD --format='- %s' 2>/dev/null)"
```
</step>
</workflow>

---

<workflow name="create_pr">
# WORKFLOW: Create New PR (Default)

<step number="1">
## 1. Check for uncommitted changes

```bash
if [ -n "$(git status --porcelain)" ]; then
  echo "WARNING: You have uncommitted changes"
  git status --short
  echo ""
  echo "Consider committing first with /git:cp or /commit-commands:commit"
fi
```
</step>

<step number="2">
## 2. Check if branch is pushed

```bash
if ! git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null; then
  echo "Pushing branch to origin..."
  git push -u origin "$(git branch --show-current)"
fi
```
</step>

<step number="3">
## 3. Check for existing PR

```bash
EXISTING_PR=$(gh pr list --head "$(git branch --show-current)" --json number,url,isDraft -q '.[0]')
if [ -n "$EXISTING_PR" ]; then
  echo "PR already exists for this branch!"
  gh pr view "$(git branch --show-current)"
  echo ""
  echo "Use --update to modify it, or --ready to mark it ready for review"
  exit 0
fi
```

<error_handling>
<error type="pr_already_exists" severity="MEDIUM">
If PR already exists, display current PR and suggest using --update or --ready flags
</error>
</error_handling>
</step>

<step number="4">
## 4. Gather PR information

Get commits for title/body generation:

```bash
# Get all commits between base and head
BASE_BRANCH="${TO_BRANCH:-main}"
COMMITS=$(git log "origin/$BASE_BRANCH..HEAD" --format="%s" 2>/dev/null)
COMMIT_COUNT=$(git rev-list --count "origin/$BASE_BRANCH..HEAD" 2>/dev/null || echo "0")

echo "Creating PR with $COMMIT_COUNT commits"
echo "Base: $BASE_BRANCH"
echo "Head: $(git branch --show-current)"
```
</step>

<step number="5">
## 5. Generate PR title and body

<conditional>
If `--fill` is specified, auto-generate. Otherwise:
</conditional>

**Title**: Use first commit message or prompt user
**Body**: Generate from commit list with template:

```markdown
## Summary

[Auto-generated from commits or user input]

## Changes

[Commit list]

## Testing

- [ ] Tests pass locally
- [ ] Linting passes
- [ ] Manual testing completed

## Checklist

- [ ] Code follows project conventions
- [ ] Documentation updated (if applicable)
- [ ] No sensitive data exposed
```
</step>

<step number="6">
## 6. Create the PR

```bash
# Default: Create as DRAFT
# Use --no-draft to create as ready

if [ "$NO_DRAFT" = "true" ]; then
  gh pr create \
    --base "${TO_BRANCH:-main}" \
    --title "$PR_TITLE" \
    --body "$PR_BODY"
else
  gh pr create \
    --base "${TO_BRANCH:-main}" \
    --title "$PR_TITLE" \
    --body "$PR_BODY" \
    --draft
fi
```
</step>

<step number="7">
## 7. Report success

```bash
echo ""
echo "=========================================="
echo "  PR Created Successfully (Draft)"
echo "=========================================="
echo ""
gh pr view "$(git branch --show-current)"
echo ""
echo "Next steps:"
echo "  - Push more commits as needed"
echo "  - Use '/gh:pr --ready' to mark ready for review"
echo "  - Use '/gh:pr --update' to modify title/body"
```
</step>
</workflow>

---

<error_handling>
# ERROR HANDLING

<error type="gh_not_found" severity="HIGH">
`gh: command not found` - Install GitHub CLI: `brew install gh`
</error>

<error type="auth_failed" severity="HIGH">
`gh auth status` fails - Run `gh auth login`
</error>

<error type="no_commits" severity="MEDIUM">
No commits between branches - "No changes to create PR for"
</error>

<error type="branch_not_pushed" severity="MEDIUM">
Branch not pushed - Auto-push with `git push -u origin`
</error>

<error type="pr_exists" severity="MEDIUM">
PR already exists - Show existing PR, suggest `--update` or `--ready`
</error>

<error type="on_main_branch" severity="MEDIUM">
Not on a feature branch - Warn if on main/master
</error>
</error_handling>

---

# EXAMPLES

```bash
# Create draft PR to main (default)
/gh:pr

# Create draft PR to develop branch
/gh:pr develop

# Create PR and open in browser
/gh:pr --web

# Create ready-for-review PR (skip draft)
/gh:pr --no-draft

# Auto-fill title/body from commits
/gh:pr --fill

# Convert draft to ready for review
/gh:pr --ready

# Update existing PR
/gh:pr --update
```

---

# NOTES

- **Draft PRs are default** because they encourage iterative development:
  - Push early, push often
  - Get early feedback before formal review
  - Mark ready when confident

- Use `gh pr view --web` to open any PR in browser
- Use `gh pr checks` to see CI status
- Use `gh pr merge` when ready to merge (after review)
