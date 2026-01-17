---
allowed-tools: Bash(gh:*), Bash(git:*), Bash(jq:*), Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
argument-hint: "[pr-number] [--auto] [--confidence=95] [--skip-rebase] [--dry-run]"
description: Complete PR remediation workflow - fetch comments, fix findings, rebase, commit, reply, resolve, and push
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
PR_FIX(1)                       User Commands                       PR_FIX(1)

NAME
    pr-fix - Complete PR remediation workflow

SYNOPSIS
    /gh:pr-fix [pr-number] [--auto] [--confidence=95] [--skip-rebase] [--dry-run]

DESCRIPTION
    End-to-end workflow for processing PR review feedback: fetch all
    comments and review feedback, analyze findings with confidence-based
    triage, remediate accepted findings, rebase onto base branch, commit
    changes with attribution, reply to each comment, resolve threads,
    and push updates.

OPTIONS
    [pr-number]               PR number (positional, or infer from branch)
    --auto                    Non-interactive mode - auto-process at/above confidence
    --confidence=N            Confidence threshold 0-100 (default: 95)
    --skip-rebase             Skip rebasing onto base branch
    --dry-run                 Show plan without executing changes
    --force                   Push with --force-with-lease after rebase
    --help, -h                Show this help message

EXAMPLES
    /gh:pr-fix                Process current branch's PR (interactive)
    /gh:pr-fix 123            Process specific PR number
    /gh:pr-fix --auto         Non-interactive with 95% confidence
    /gh:pr-fix --dry-run      Preview actions without executing
    /gh:pr-fix --force        Force push after rebase

SEE ALSO
    /gh:pr             Create/update pull requests
    /gh:review-comments  Detailed comment review workflow
    /gh:cp              Stage, commit, and push

                                                                    PR_FIX(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

# PR Fix - Complete Remediation Workflow

End-to-end workflow for processing PR review feedback:
1. **Fetch** all comments and review feedback
2. **Analyze** findings with confidence-based triage
3. **Ask** user questions when confidence < 95% (or custom threshold)
4. **Remediate** accepted findings
5. **Rebase** onto base branch (unless `--skip-rebase`)
6. **Commit** changes with attribution
7. **Reply** to each comment with resolution
8. **Resolve** comment threads
9. **Push** updates to remote

## Arguments

Parse `$ARGUMENTS` for:

| Flag | Purpose | Default |
|------|---------|---------|
| `[pr-number]` | PR number (positional) | Current branch's PR |
| `--auto` | Non-interactive mode - auto-process at/above confidence | Off |
| `--confidence=N` | Confidence threshold (0-100) for auto-action | **95** |
| `--skip-rebase` | Skip rebasing onto base branch | Off |
| `--dry-run` | Show plan without executing changes | Off |
| `--force` | Push with `--force-with-lease` after rebase | Off |

---

<phase number="1" name="Initialization">
# PHASE 1: INITIALIZATION

## 1.1 Pre-flight Checks

```bash
# Verify gh CLI
command -v gh >/dev/null || { echo "ERROR: gh CLI not found. Install: brew install gh"; exit 1; }

# Verify authentication
gh auth status || { echo "ERROR: Not authenticated. Run: gh auth login"; exit 1; }

# Get repo info
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
OWNER=$(echo "$REPO" | cut -d'/' -f1)
REPO_NAME=$(echo "$REPO" | cut -d'/' -f2)

echo "Repository: $REPO"
```

## 1.2 Determine PR

```bash
if [ -n "$PR_NUMBER_ARG" ]; then
  PR_NUMBER="$PR_NUMBER_ARG"
else
  PR_NUMBER=$(gh pr view --json number -q '.number' 2>/dev/null)
fi

if [ -z "$PR_NUMBER" ]; then
  echo "ERROR: No PR found. Specify PR number: /gh:pr-fix 123"
  exit 1
fi

echo "Processing PR #$PR_NUMBER"
```

## 1.3 Get PR Metadata

```bash
PR_INFO=$(gh pr view "$PR_NUMBER" --json number,title,author,headRefName,baseRefName,state,url,isDraft)
echo "$PR_INFO" | jq .

BASE_BRANCH=$(echo "$PR_INFO" | jq -r '.baseRefName')
HEAD_BRANCH=$(echo "$PR_INFO" | jq -r '.headRefName')
PR_STATE=$(echo "$PR_INFO" | jq -r '.state')

# Verify PR is open
if [ "$PR_STATE" != "OPEN" ]; then
  echo "WARNING: PR is $PR_STATE, not OPEN"
fi
```

## 1.4 Sync Local Branch

```bash
# Ensure we're on the right branch
CURRENT_BRANCH=$(git branch --show-current)
if [ "$CURRENT_BRANCH" != "$HEAD_BRANCH" ]; then
  echo "Switching to PR branch: $HEAD_BRANCH"
  git checkout "$HEAD_BRANCH"
fi

# Fetch latest
git fetch origin "$HEAD_BRANCH" "$BASE_BRANCH"
```
</phase>

---

<phase number="2" name="Fetch All Feedback">
# PHASE 2: FETCH ALL FEEDBACK

## 2.1 Code Review Comments (Inline)

```bash
gh api repos/$OWNER/$REPO_NAME/pulls/$PR_NUMBER/comments --paginate | jq -r '
  .[] | select(.body != null) | {
    id: .id,
    path: .path,
    line: (.line // .original_line),
    side: .side,
    body: .body,
    user: .user.login,
    created_at: .created_at,
    in_reply_to_id: .in_reply_to_id,
    html_url: .html_url,
    diff_hunk: .diff_hunk,
    commit_id: .commit_id
  }
' > /tmp/pr_review_comments.json
```

## 2.2 Issue-style Comments

```bash
gh pr view "$PR_NUMBER" --json comments --jq '.comments[] | {
  id: .id,
  author: .author.login,
  body: .body,
  createdAt: .createdAt,
  url: .url
}' > /tmp/pr_issue_comments.json
```

## 2.3 Review Requests & Status

```bash
gh pr view "$PR_NUMBER" --json reviews --jq '.reviews[] | {
  author: .author.login,
  state: .state,
  body: .body,
  submittedAt: .submittedAt
}' > /tmp/pr_reviews.json
```

## 2.4 Categorize Feedback

Organize into actionable categories:

| Category | Detection | Priority |
|----------|-----------|----------|
| **Blocking** | Contains "must", "required", "blocker", "NACK" | P0 - Critical |
| **Bug/Issue** | Contains "bug", "error", "wrong", "incorrect" | P1 - High |
| **Suggestion** | Contains "consider", "might", "could", "nit" | P2 - Medium |
| **Question** | Contains "?", "why", "how", "what" | P3 - Respond |
| **Approval** | Contains "LGTM", "looks good", "approved" | Info only |
</phase>

---

<phase number="3" name="Confidence-Based Triage">
# PHASE 3: CONFIDENCE-BASED TRIAGE

For each actionable comment, calculate confidence:

## 3.1 Confidence Dimensions

| Dimension | Weight | Assessment |
|-----------|--------|------------|
| **Technical Accuracy** | 35% | Is the feedback technically correct? |
| **Code Evidence** | 30% | Does reading the code confirm the issue? |
| **Clear Remediation** | 20% | Is the fix obvious and safe? |
| **Scope Impact** | 15% | Is change localized without side effects? |

**Confidence Score** = Weighted sum (0-100)

## 3.2 Threshold Behavior (default: 95%)

<conditional trigger="confidence score determines action">
| Confidence | Action |
|------------|--------|
| >= threshold | **Auto-accept**: Remediate without prompting |
| 70-threshold | **Prompt**: Use AskUserQuestion for decision |
| 50-69% | **Detailed prompt**: Show evidence, recommend action |
| < 50% | **Skeptical prompt**: Present counter-evidence |
</conditional>

## 3.3 User Decision Points

<conditional trigger="confidence < threshold">
When confidence < threshold, use AskUserQuestion:

<ask_user_question>
```
header: "PR Feedback"
question: "How should we handle this comment from @{reviewer}?"
multiSelect: false
options:
  - label: "Accept & Fix"
    description: "Implement the suggested change"
  - label: "Accept Modified"
    description: "Fix the issue but implement differently"
  - label: "Reject"
    description: "Decline with explanation"
  - label: "Discuss"
    description: "Need clarification before deciding"
  - label: "Skip"
    description: "Defer to later"
```
</ask_user_question>
</conditional>
</phase>

---

<phase number="4" name="Remediation">
# PHASE 4: REMEDIATION

## 4.1 Read Before Edit

**CRITICAL**: Always read the full file before making changes.

```bash
# For each file needing changes
git show HEAD:"$FILE_PATH" > /tmp/original_file
```

## 4.2 Apply Fixes

For each accepted finding:
1. Read the file completely
2. Understand surrounding context
3. Implement minimal fix
4. Verify fix doesn't break related code

## 4.3 Specialist Agent Routing

Route complex fixes to appropriate specialists using Task tool:

| Issue Type | Agent | Purpose |
|------------|-------|---------|
| Security | `zircote:security-engineer` | Secure implementation |
| Performance | `zircote:performance-engineer` | Optimized fix |
| Type errors | `zircote:typescript-pro` | Type-safe resolution |
| Architecture | `zircote:refactoring-specialist` | Pattern-aligned refactor |
| Tests | `zircote:test-automator` | Add/fix tests |

## 4.4 Track Changes

Maintain remediation log:

```markdown
| Comment ID | File:Line | Fix Applied | Status |
|------------|-----------|-------------|--------|
| 12345 | src/api.ts:42 | Added null check | Done |
```
</phase>

---

<phase number="5" name="Rebase">
# PHASE 5: REBASE (unless --skip-rebase)

<conditional trigger="--skip-rebase flag NOT set">
## 5.1 Fetch Latest Base

```bash
git fetch origin "$BASE_BRANCH"
```

## 5.2 Rebase onto Base

```bash
echo "Rebasing $HEAD_BRANCH onto origin/$BASE_BRANCH..."
git rebase "origin/$BASE_BRANCH"
```

## 5.3 Handle Conflicts

<conditional trigger="rebase conflicts occur">
If conflicts occur:

```bash
# Show conflicts
git diff --name-only --diff-filter=U

# For each conflicted file, read and resolve
# Use AskUserQuestion if resolution is unclear
```

<ask_user_question>
Conflict resolution options:
- **Ours**: Keep PR's version
- **Theirs**: Take base branch's version
- **Manual**: Let me resolve it
- **Abort**: Cancel rebase, user handles it
</ask_user_question>
</conditional>

## 5.4 Verify Rebase

```bash
# Ensure all commits are intact
git log --oneline "origin/$BASE_BRANCH..HEAD"

# Run quick sanity check
npm test 2>/dev/null || yarn test 2>/dev/null || pytest 2>/dev/null || echo "No test command found"
```
</conditional>
</phase>

---

<phase number="6" name="Commit Changes">
# PHASE 6: COMMIT CHANGES

## 6.1 Stage Remediation Changes

```bash
git add -A
git status
```

## 6.2 Create Commit

```bash
git commit -m "fix: address PR review feedback

Remediations:
$(cat /tmp/remediation_log.md)

Reviewers: $(cat /tmp/reviewers.txt | tr '\n' ', ')
"
```

## 6.3 Commit Message Format

```
fix: address PR review feedback

- [file:line] Brief description of fix
- [file:line] Brief description of fix

Co-authored-by: @reviewer1
Addresses feedback from: @reviewer2, @reviewer3
```
</phase>

---

<phase number="7" name="Reply to Comments">
# PHASE 7: REPLY TO COMMENTS

## 7.1 Response Templates

### Fixed Comment
```markdown
**Fixed**

{brief_description}

Changes in commit `{short_sha}`.
```

### Fixed with Modification
```markdown
**Addressed (different approach)**

{description_of_alternative}

**Rationale**: {why_alternative}
```

### Rejected Comment
```markdown
**Considered but not changed**

{reasoning}

{evidence_or_code_reference}

Happy to discuss further!
```

### Question Response
```markdown
**Response**

{answer}
```

## 7.2 Post Replies

For inline comments (reply to thread):
```bash
gh api repos/$OWNER/$REPO_NAME/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies \
  -f body="$RESPONSE_BODY"
```

For issue-style comments:
```bash
gh pr comment "$PR_NUMBER" --body "$RESPONSE_BODY"
```
</phase>

---

<phase number="8" name="Resolve Threads">
# PHASE 8: RESOLVE THREADS

## 8.1 Get Thread IDs

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $pr: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 1) {
            nodes { body author { login } }
          }
        }
      }
    }
  }
}' -f owner="$OWNER" -f repo="$REPO_NAME" -F pr="$PR_NUMBER"
```

## 8.2 Resolve Fixed Threads

For each thread corresponding to a fixed comment:

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}' -f threadId="$THREAD_ID"
```
</phase>

---

<phase number="9" name="Push Updates">
# PHASE 9: PUSH UPDATES

## 9.1 Normal Push (no rebase)

```bash
git push origin "$HEAD_BRANCH"
```

## 9.2 Force Push (after rebase, with --force flag)

<conditional trigger="rebase was performed">
```bash
if [ "$FORCE_FLAG" = "true" ]; then
  git push --force-with-lease origin "$HEAD_BRANCH"
else
  echo ""
  echo "Rebase complete. Changes require force push."
  echo "Run: git push --force-with-lease origin $HEAD_BRANCH"
  echo "Or use --force flag with this command."
fi
```
</conditional>

## 9.3 Verify Push

```bash
gh pr view "$PR_NUMBER" --json commits --jq '.commits | length'
echo "Commits pushed successfully"
```
</phase>

---

<phase number="10" name="Summary">
# PHASE 10: SUMMARY

Generate completion report:

```markdown
# PR #${PR_NUMBER} Remediation Complete

## Statistics

| Metric | Count |
|--------|-------|
| Comments Processed | {total} |
| Fixes Applied | {fixes} |
| Rejected | {rejected} |
| Questions Answered | {questions} |
| Threads Resolved | {resolved} |

## Changes Made

| File | Changes | Description |
|------|---------|-------------|
| {file} | +{add}/-{del} | {desc} |

## Replies Posted

| Comment | Response Type |
|---------|---------------|
| @{reviewer}: "{summary}" | {type} |

## Actions Taken

- [x] Fetched {n} review comments
- [x] Analyzed with {confidence}% threshold
- [x] Applied {n} remediations
- [x] Rebased onto origin/{base}
- [x] Committed changes
- [x] Posted {n} replies
- [x] Resolved {n} threads
- [x] Pushed to origin/{head}

## Next Steps

- [ ] Wait for CI checks
- [ ] Request re-review if needed: `gh pr review --request`
- [ ] Mark ready if draft: `/gh:pr --ready`
```
</phase>

---

<error_handling>
# ERROR HANDLING

| Error | Response |
|-------|----------|
| `gh: command not found` | Install: `brew install gh` |
| Not authenticated | Run: `gh auth login` |
| PR not found | Verify PR number or create one first |
| Merge conflict | Show conflicts, offer resolution options |
| Push rejected | Suggest `--force` if rebased |
| Rate limited | Wait, show remaining limits |
| Thread already resolved | Skip, note as "already resolved" |
| File not found | Comment may be on deleted file; note in response |

<conditional trigger="error occurs during any phase">
When an error occurs:
1. Identify the phase and operation that failed
2. Log error details for debugging
3. Determine if the workflow can continue
4. If recoverable, skip the failed item and proceed
5. If critical (e.g., rebase conflict), pause and prompt user
6. Include all errors in the final summary report
</conditional>
</error_handling>

---

<conditional trigger="--dry-run flag is set">
# DRY RUN MODE

With `--dry-run`, output all planned actions without executing:

```markdown
# Dry Run Report

## Would Process

{n} comments from {reviewers}

## Would Apply

{list of fixes with file:line and description}

## Would Reply

{list of responses}

## Would Resolve

{list of thread IDs}

## Commands That Would Run

{git and gh commands}
```
</conditional>

---

# EXAMPLES

```bash
# Process current branch's PR with 95% confidence (default)
/gh:pr-fix

# Process specific PR
/gh:pr-fix 123

# Lower confidence threshold (more prompts)
/gh:pr-fix --confidence=80

# Fully automatic mode
/gh:pr-fix --auto --confidence=90

# Skip rebase (useful if already rebased)
/gh:pr-fix --skip-rebase

# Force push after rebase
/gh:pr-fix --force

# Preview without making changes
/gh:pr-fix --dry-run
```

---

Begin with Phase 1: Initialization.
