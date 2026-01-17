---
allowed-tools: Bash(gh:*), Bash(git:*), Bash(jq:*), Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
argument-hint: "[pr-number] [--auto | --interactive] [--confidence=N] [--dry-run]"
description: Review GitHub PR comments, assess validity, remediate accepted findings, and respond to all comments with explanations
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
REVIEW_COMMENTS(1)              User Commands              REVIEW_COMMENTS(1)

NAME
    review-comments - Review GitHub PR comments and remediate findings

SYNOPSIS
    /gh:review-comments [pr-number] [--auto | --interactive] [--confidence=N] [--dry-run]

DESCRIPTION
    Process code review comments on a GitHub pull request. For each
    comment: assess validity with confidence scoring, accept or reject
    based on evidence, remediate accepted findings with code changes,
    and respond to all comments with explanations.

OPTIONS
    [pr-number]               PR number (positional, or infer from branch)
    --auto                    Non-interactive mode - auto-process based on confidence
    --interactive             Interactive mode - prompt at each decision (default)
    --confidence=N            Minimum confidence 0-100 to auto-accept (default: 85)
    --dry-run                 Show proposed actions without executing
    --help, -h                Show this help message

CONFIDENCE THRESHOLDS
    --confidence=95           Very conservative - only obvious fixes
    --confidence=85           Balanced (default)
    --confidence=75           More aggressive auto-acceptance
    --confidence=50           Accept most reasonable suggestions

EXAMPLES
    /gh:review-comments            Process current branch's PR (interactive)
    /gh:review-comments 123        Process specific PR number
    /gh:review-comments --auto     Non-interactive with 85% confidence
    /gh:review-comments --dry-run  Preview actions without executing

SEE ALSO
    /gh:pr-fix          Complete remediation with rebase
    /gh:pr              Create/update pull requests

                                                        REVIEW_COMMENTS(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

# GitHub Code Review Comment Processor

You are processing code review comments on a GitHub pull request. For each comment, you will:

1. **Assess validity** with confidence scoring
2. **Accept or reject** based on evidence
3. **Remediate** accepted findings with code changes
4. **Respond** to all comments with explanations and resolutions

## Arguments

Parse `$ARGUMENTS` for:

| Flag | Purpose | Default |
|------|---------|---------|
| `[pr-number]` | PR number (positional, or infer from current branch) | Current branch's PR |
| `--auto` | Non-interactive mode - auto-process based on confidence | Off |
| `--interactive` | Interactive mode - prompt at each decision | **Default** |
| `--confidence=N` | Minimum confidence (0-100) to auto-accept in `--auto` mode | 85 |
| `--dry-run` | Show proposed actions without executing | Off |

**Mode behavior:**
- **Interactive (default)**: Use `AskUserQuestion` at each finding for user decision
- **Auto (`--auto`)**: Process findings automatically; only prompt on low-confidence items

---

<phase number="1" name="Context Gathering">
# PHASE 1: CONTEXT GATHERING

## 1.1 Pre-flight Checks

1. **Verify `gh` CLI is available and authenticated**:
   ```bash
   which gh && gh auth status
   ```

   <conditional trigger="gh CLI not available or not authenticated">
   If not available or authenticated, display:
   > **GitHub CLI Required**
   > Install: `brew install gh` (macOS) | `apt install gh` (Linux)
   > Authenticate: `gh auth login`
   </conditional>

2. **Determine PR number**:

   <conditional trigger="PR number provided as argument">
   If PR number provided as argument, use it. Otherwise:
   ```bash
   gh pr view --json number --jq '.number' 2>/dev/null
   ```
   </conditional>

   <conditional trigger="no PR found">
   If no PR found:
   > "No PR found for current branch. Please specify a PR number:
   > `/gh:review-comments 123`"
   </conditional>

3. **Get PR metadata**:
   ```bash
   gh pr view ${PR_NUMBER} --json number,title,author,headRefName,baseRefName,state,url
   ```

## 1.2 Fetch Review Comments

Retrieve all review comments via REST API:

```bash
gh api repos/{owner}/{repo}/pulls/${PR_NUMBER}/comments --paginate | jq -r '
  .[] | {
    id: .id,
    path: .path,
    line: .line,
    original_line: .original_line,
    side: .side,
    body: .body,
    user: .user.login,
    created_at: .created_at,
    in_reply_to_id: .in_reply_to_id,
    html_url: .html_url,
    diff_hunk: .diff_hunk
  }
'
```

**IMPORTANT: Fetch GraphQL Thread IDs for Resolution**

The REST API comment IDs cannot be used to resolve threads. You MUST fetch the GraphQL node IDs for review threads:

```bash
# IMPORTANT: GraphQL variables ($owner, $repo, $pr) must be escaped with backslashes
# to prevent shell expansion while allowing -F flags to pass values
gh api graphql \
  -F owner="${OWNER}" \
  -F repo="${REPO}" \
  -F pr="${PR_NUMBER}" \
  -f query="query(\$owner: String!, \$repo: String!, \$pr: Int!) { repository(owner: \$owner, name: \$repo) { pullRequest(number: \$pr) { reviewThreads(first: 100) { nodes { id isResolved isOutdated path line comments(first: 1) { nodes { id databaseId body author { login } } } } } } } }"
```

<graphql_escaping_rules>
**Critical escaping rules for `gh api graphql`:**

1. **Use double quotes** around the query (not single quotes)
2. **Escape GraphQL variables** with backslashes: `\$owner`, `\$repo`, `\$pr`
3. **Shell variables expand normally**: `${OWNER}`, `${REPO}`, `${PR_NUMBER}`
4. **Pass values via -F (numeric) or -f (string) flags**

**Why this works:**
- Double quotes allow shell variable expansion (`${OWNER}`)
- Backslash escaping prevents shell from consuming GraphQL's `$` markers
- GraphQL receives the proper variable declarations
</graphql_escaping_rules>

**Store a mapping of comment `databaseId` to thread `id`** for use in Phase 6 resolution.

<thread_mapping_example>
Build a lookup table:
```
THREAD_MAP = {
  comment_database_id_1: "thread_node_id_1",
  comment_database_id_2: "thread_node_id_2",
  ...
}
```
Where `databaseId` corresponds to the REST API `id` field.
</thread_mapping_example>

Also get general PR comments (issue-style):

```bash
gh pr view ${PR_NUMBER} --json comments --jq '.comments[] | {id: .id, author: .author.login, body: .body, createdAt: .createdAt}'
```

## 1.3 Categorize Comments

Organize comments into:

| Category | Description | Action Required |
|----------|-------------|-----------------|
| **Code Review** | Specific feedback on code changes | Assess & remediate |
| **Questions** | Clarification requests | Answer with explanation |
| **Suggestions** | Optional improvements | Assess validity |
| **Blockers** | Must-fix before merge | High priority remediation |
| **Approvals** | Positive feedback | Acknowledge only |
| **Conversations** | Threaded discussions | Check if resolved |
</phase>

---

<phase number="2" name="Validity Assessment">
# PHASE 2: VALIDITY ASSESSMENT

For each actionable comment (Code Review, Suggestions, Blockers), perform:

## 2.1 Evidence Gathering

1. **Read the referenced file**:
   ```bash
   git show HEAD:${FILE_PATH}
   ```

2. **Understand the diff context**:
   - Use the `diff_hunk` from the comment
   - Read surrounding code for full context

3. **Check related code**:
   - Search for related patterns/usages
   - Review tests if applicable

## 2.2 Validity Scoring

Score each comment on these dimensions (0-100):

| Dimension | Weight | Criteria |
|-----------|--------|----------|
| **Technical Accuracy** | 40% | Is the concern factually correct? |
| **Relevance** | 25% | Does it apply to this specific code? |
| **Impact** | 20% | Would fixing it meaningfully improve the code? |
| **Feasibility** | 15% | Is the suggested fix reasonable to implement? |

**Confidence Score** = Weighted average of all dimensions

## 2.3 Classification

<conditional trigger="confidence score determines action">
| Confidence | Classification | Action |
|------------|----------------|--------|
| >=90% | **Strong Accept** | Auto-remediate (even in interactive mode, just inform) |
| 75-89% | **Accept** | Remediate (prompt in interactive mode) |
| 50-74% | **Uncertain** | Always prompt user for decision |
| 25-49% | **Likely Reject** | Present counter-evidence, recommend rejection |
| <25% | **Strong Reject** | Auto-reject with explanation |
</conditional>
</phase>

---

<phase number="3" name="Decision Workflow">
# PHASE 3: DECISION WORKFLOW

## 3.1 Interactive Mode (Default)

<conditional trigger="confidence < 90% AND mode is interactive">
For each finding with confidence < 90%, use `AskUserQuestion`:

<ask_user_question>
```
## Comment from @{reviewer}:
"{comment_body}"

File: {file_path}:{line}

### Assessment
- **Confidence**: {score}%
- **Classification**: {classification}
- **Evidence**: {evidence_summary}

### Recommendation
{recommendation}

**How would you like to proceed?**
```

Options:
- **Accept & Remediate**: Implement the suggested fix
- **Accept & Modify**: Accept but implement differently
- **Reject with Explanation**: Decline with reasoning
- **Defer**: Skip for now, revisit later
- **Discuss**: Need more context before deciding
</ask_user_question>
</conditional>

## 3.2 Auto Mode (`--auto`)

<conditional trigger="mode is auto">
Process based on confidence thresholds:

| Confidence | Auto Action |
|------------|-------------|
| >= `--confidence` threshold | Auto-accept and remediate |
| 50% to threshold | Queue for batch user review |
| < 50% | Auto-reject with explanation |

After auto-processing, present batch summary:

> **Auto-processed {N} comments:**
> - Accepted: {accepted_count}
> - Rejected: {rejected_count}
> - Needs Review: {uncertain_count}
>
> Review the uncertain items? (y/n)
</conditional>
</phase>

---

<phase number="4" name="Remediation">
# PHASE 4: REMEDIATION

## 4.1 Remediation Strategy

For each accepted finding:

1. **Read the full file** to understand context
2. **Design the fix** considering:
   - Minimal change principle
   - Consistency with codebase patterns
   - Test coverage implications
3. **Implement the change** using `Edit` tool
4. **Verify the fix**:
   - Syntax check (language-appropriate)
   - Run related tests if identifiable

## 4.2 Specialist Agent Routing

For complex remediations, use the Task tool with appropriate specialists:

| Finding Type | Agent | Purpose |
|--------------|-------|---------|
| Security vulnerability | `zircote:security-engineer` | Secure fix implementation |
| Performance issue | `zircote:performance-engineer` | Optimized solution |
| Type/interface issue | `zircote:typescript-pro` / appropriate language | Type-safe fix |
| Architecture concern | `zircote:refactoring-specialist` | Pattern-aligned refactor |
| Test coverage gap | `zircote:test-automator` | Add missing tests |

## 4.3 Change Tracking

Maintain a remediation log:

```markdown
## Remediation Log

| Comment ID | Thread ID | File | Change | Status |
|------------|-----------|------|--------|--------|
| {id} | {thread_id} | {path}:{line} | {description} | Fixed |
```
</phase>

---

<phase number="5" name="Response Generation">
# PHASE 5: RESPONSE GENERATION

## 5.1 Response Templates

### For Accepted Findings (Remediated)

```markdown
**Addressed in this PR**

{brief_description_of_fix}

**Changes made:**
- {change_1}
- {change_2}

Thanks for catching this, @{reviewer}!
```

### For Accepted with Modification

```markdown
**Addressed with modification**

I agree with the concern. Instead of the suggested approach, I implemented:

{description_of_alternative}

**Rationale:**
{why_alternative_is_better}

Let me know if you'd like to discuss further.
```

### For Rejected Findings

```markdown
**Considered but not implemented**

Thank you for the suggestion. After analysis, I believe the current approach is preferable:

**Reasoning:**
{detailed_reasoning}

**Evidence:**
{citations_or_code_references}

{optional: alternative_consideration}

Happy to discuss if you see something I'm missing!
```

### For Questions

```markdown
**Response:**

{answer_to_question}

{code_examples_if_helpful}
```

## 5.2 Posting Responses

For each response:

```bash
gh api repos/{owner}/{repo}/pulls/${PR_NUMBER}/comments \
  -f body="${RESPONSE}" \
  -f commit_id="${COMMIT_SHA}" \
  -f path="${FILE_PATH}" \
  -f line=${LINE_NUMBER} \
  -f side="${SIDE}"
```

For replies to existing threads:

```bash
gh api repos/{owner}/{repo}/pulls/${PR_NUMBER}/comments/${COMMENT_ID}/replies \
  -f body="${RESPONSE}"
```
</phase>

---

<phase number="6" name="Resolution and Summary">
# PHASE 6: RESOLUTION & SUMMARY

<mandatory_resolution>
**⚠️ MANDATORY: Thread Resolution Required**

You MUST resolve all addressed comment threads before completing this command. This is NOT optional.

**Failure mode to avoid:** Responding to comments but leaving threads unresolved. This defeats the purpose of the command.

**Verification requirement:** After resolution attempts, you MUST run a verification query to confirm all threads are resolved.
</mandatory_resolution>

## 6.1 Mark Conversations Resolved

After addressing a comment thread, resolve it using the GraphQL API.

<resolution_requirements>
**Prerequisites:**
- You MUST have the GraphQL thread node ID (not the REST API comment ID)
- Thread IDs start with a prefix like `PRRT_` and are base64-encoded
- Use the thread mapping created in Phase 1.2
</resolution_requirements>

<resolution_command>
**Correct GraphQL mutation syntax:**

```bash
# Using variables (recommended - avoids escaping issues)
gh api graphql \
  -f query='
    mutation($threadId: ID!) {
      resolveReviewThread(input: {threadId: $threadId}) {
        thread {
          id
          isResolved
        }
      }
    }
  ' \
  -f threadId="${THREAD_NODE_ID}"
```

**Alternative inline syntax (for simple cases):**

```bash
# Note: Thread ID must be properly escaped in the query
gh api graphql -f query="
  mutation {
    resolveReviewThread(input: {threadId: \"${THREAD_NODE_ID}\"}) {
      thread {
        id
        isResolved
      }
    }
  }
"
```
</resolution_command>

<resolution_error_handling>
**Error Handling for Thread Resolution:**

| Error | Cause | Resolution |
|-------|-------|------------|
| `Could not resolve to a node` | Invalid thread ID | Verify thread ID from Phase 1.2 GraphQL query |
| `Resource not accessible` | Permission denied | User may not have write access to the PR |
| `Thread is already resolved` | Previously resolved | Skip with note (not an error) |
| `Variable $threadId was provided invalid value` | Malformed ID | Ensure ID is the full GraphQL node ID |

```bash
# Resolution with error handling
resolve_thread() {
  local thread_id="$1"
  local result

  result=$(gh api graphql \
    -f query='
      mutation($threadId: ID!) {
        resolveReviewThread(input: {threadId: $threadId}) {
          thread {
            id
            isResolved
          }
        }
      }
    ' \
    -f threadId="${thread_id}" 2>&1)

  if echo "$result" | grep -q '"isResolved":true'; then
    echo "SUCCESS: Thread ${thread_id} resolved"
    return 0
  elif echo "$result" | grep -q "already resolved"; then
    echo "SKIPPED: Thread ${thread_id} was already resolved"
    return 0
  else
    echo "FAILED: Could not resolve thread ${thread_id}"
    echo "Error: $result"
    return 1
  fi
}
```
</resolution_error_handling>

<resolution_workflow>
**Complete Resolution Workflow:**

1. **Look up thread ID** from the mapping created in Phase 1.2
2. **Skip if already resolved** (check `isResolved` from Phase 1.2 data)
3. **Execute the mutation** with proper error handling
4. **Log the result** in the remediation log

```bash
# For each addressed comment
COMMENT_DB_ID="12345678"  # REST API comment ID
THREAD_NODE_ID="${THREAD_MAP[$COMMENT_DB_ID]}"  # GraphQL node ID from Phase 1.2

if [ -n "$THREAD_NODE_ID" ]; then
  gh api graphql \
    -f query='
      mutation($threadId: ID!) {
        resolveReviewThread(input: {threadId: $threadId}) {
          thread { id isResolved }
        }
      }
    ' \
    -f threadId="$THREAD_NODE_ID"
else
  echo "Warning: No thread ID found for comment $COMMENT_DB_ID"
fi
```
</resolution_workflow>

<resolution_verification>
## 6.1.1 MANDATORY: Verify All Threads Resolved

**After all resolution attempts, you MUST verify that threads are actually resolved:**

```bash
# Verify all threads are resolved
gh api graphql \
  -F owner="${OWNER}" \
  -F repo="${REPO}" \
  -F pr="${PR_NUMBER}" \
  -f query='
    query($owner: String!, $repo: String!, $pr: Int!) {
      repository(owner: $owner, name: $repo) {
        pullRequest(number: $pr) {
          reviewThreads(first: 100) {
            nodes {
              id
              isResolved
              path
              line
            }
          }
        }
      }
    }
  ' | jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

**If any unresolved threads remain that you addressed:**
1. Retry resolution for those specific threads
2. If retry fails, report the failure explicitly to the user
3. Do NOT mark the command as complete until all addressed threads are resolved or explicitly failed

**Success criteria:** The verification query returns empty (no unresolved threads for comments you addressed).
</resolution_verification>

## 6.2 Generate Summary Report

Create a summary of all actions taken:

```markdown
# Code Review Response Summary

**PR**: #{pr_number} - {title}
**Processed**: {timestamp}
**Mode**: {interactive|auto}

## Statistics

| Metric | Count |
|--------|-------|
| Total Comments | {total} |
| Accepted & Fixed | {accepted_fixed} |
| Accepted & Modified | {accepted_modified} |
| Rejected | {rejected} |
| Questions Answered | {questions} |
| Deferred | {deferred} |
| **Threads Resolved** | {resolved_count} |
| Resolution Failures | {resolution_failures} |

## Changes Made

| File | Lines Changed | Description |
|------|---------------|-------------|
| {file} | +{add}/-{del} | {description} |

## Responses Posted

| Comment | Reviewer | Response Type | Thread Resolved |
|---------|----------|---------------|-----------------|
| {summary} | @{user} | {type} | {yes/no/failed} |

## Next Steps

- [ ] Review deferred items: {count}
- [ ] Run full test suite
- [ ] Request re-review if significant changes
- [ ] Verify all threads show as resolved in GitHub UI
```

## 6.3 Dry Run Output

<conditional trigger="--dry-run flag is set">
If `--dry-run` specified, output all proposed actions without executing:

```markdown
# Dry Run Report

## Proposed Remediations
{list_of_changes}

## Proposed Responses
{list_of_responses}

## Threads to Resolve
| Thread ID | Comment | Path | Line |
|-----------|---------|------|------|
| {thread_id} | {comment_preview} | {path} | {line} |

## Commands That Would Execute
{list_of_gh_commands}

### Resolution Commands
```bash
# Thread 1
gh api graphql -f query='mutation($threadId: ID!) { resolveReviewThread(input: {threadId: $threadId}) { thread { isResolved } } }' -f threadId="PRRT_xxx..."

# Thread 2
gh api graphql -f query='mutation($threadId: ID!) { resolveReviewThread(input: {threadId: $threadId}) { thread { isResolved } } }' -f threadId="PRRT_yyy..."
```
```
</conditional>
</phase>

---

<error_handling>
# ERROR HANDLING

| Error | Response |
|-------|----------|
| `gh: command not found` | Install GitHub CLI first |
| API rate limit | Wait and retry, or use `--dry-run` |
| File not found in repo | Comment may be on deleted/moved file; note in response |
| Merge conflict during edit | Abort remediation, notify user |
| Permission denied on PR | Verify repository access with `gh auth status` |
| Comment already resolved | Skip, note as "previously addressed" |
| Thread resolution failed | Log error, continue with other threads, report in summary |
| Invalid thread ID | Verify thread mapping from Phase 1.2; may need to re-fetch |

<conditional trigger="error occurs during execution">
When an error occurs:
1. Log the error with context (phase, comment ID, file path)
2. Determine if error is recoverable
3. If recoverable, attempt retry with backoff
4. If not recoverable, add to deferred list and continue with remaining comments
5. Include all errors in final summary report
</conditional>
</error_handling>

---

# EXECUTION PROTOCOL

1. **Always read before modifying** - Never edit a file you haven't read
2. **One file at a time** - Complete all changes to a file before moving to next
3. **Test after each remediation** - Verify changes don't break existing functionality
4. **Preserve reviewer attribution** - Always credit the reviewer in responses
5. **Never force-push** - If commits are needed, use normal push
6. **Commit incrementally** - Group related fixes into logical commits
7. **⚠️ MANDATORY: Resolve ALL threads** - Every addressed comment thread MUST be resolved via GraphQL mutation
8. **⚠️ MANDATORY: Verify resolution** - Run verification query to confirm threads are resolved before completing

<execution_checklist>
**Before marking this command complete, verify ALL of the following:**

- [ ] All comments have been assessed
- [ ] All accepted findings have been remediated
- [ ] All responses have been posted
- [ ] **All addressed threads have been resolved via GraphQL mutation**
- [ ] **Verification query confirms no unresolved threads remain**
- [ ] Summary report has been generated

**If thread resolution fails:** Do NOT silently continue. Report the failure explicitly.
</execution_checklist>

## Commit Strategy

After remediations:

```bash
git add -A
git commit -m "fix: address code review feedback

- {change_1}
- {change_2}

Addresses review comments from @{reviewer1}, @{reviewer2}
"
```

---

# CONFIDENCE THRESHOLDS & OVERRIDES

The `--confidence` flag sets the auto-accept threshold in `--auto` mode:

| Setting | Behavior |
|---------|----------|
| `--confidence=95` | Very conservative - only obvious fixes |
| `--confidence=85` | Balanced (default) |
| `--confidence=75` | More aggressive auto-acceptance |
| `--confidence=50` | Accept most reasonable suggestions |

<conditional trigger="interactive mode with high confidence findings">
**Override in interactive mode:**
Even in interactive mode, findings with >=95% confidence are auto-accepted with notification (no prompt). Use `--confidence=100` to require confirmation for everything.
</conditional>

---

<final_reminder>
**⚠️ CRITICAL REMINDER: Thread Resolution is MANDATORY**

Before completing this command:
1. You MUST resolve all addressed comment threads using GraphQL mutations
2. You MUST verify resolution with a verification query
3. You MUST report any resolution failures explicitly

**DO NOT** end this command with unresolved threads that you addressed. This is the #1 failure mode to avoid.
</final_reminder>

Begin with Phase 1: Context Gathering.
