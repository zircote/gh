---
argument-hint: [--ci=TYPE] [--copilot-only] [--dry-run] [--interactive] [--batch-report] [--source=PLATFORM] [--plan-only]
description: Onboard repository to GitHub ecosystem with Copilot integration. Auto-detects and migrates CI/CD from Concourse, Jenkins, GitLab CI, CircleCI, Travis, Azure Pipelines, Bamboo, TeamCity, Drone, Buildkite, or infers external CI from artifacts. Creates GitHub Actions, Copilot config, issue/PR templates. Run in any repo for enterprise cloud migration.
model: claude-opus-4-5-20251101
allowed-tools: Bash(*), Read, Write, Edit, Glob, Grep, LS, WebFetch, Task, TodoWrite, TodoRead, AskUserQuestion
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
CI_ASSIST(1)                    User Commands                    CI_ASSIST(1)

NAME
    ci-assist - Onboard repository to GitHub ecosystem with Copilot

SYNOPSIS
    /gh:ci-assist [--ci=TYPE] [--copilot-only] [--dry-run] [--interactive]
                  [--batch-report] [--source=PLATFORM] [--plan-only]

DESCRIPTION
    Onboard repository to GitHub ecosystem with Copilot integration.
    Auto-detects and migrates CI/CD from: Concourse, Jenkins, GitLab CI,
    CircleCI, Travis, Azure Pipelines, Bamboo, TeamCity, Drone, Buildkite.
    Creates GitHub Actions workflows, Copilot config, issue/PR templates.

OPTIONS
    --ci=TYPE                 Force CI type (concourse|jenkins|gitlab|...)
    --source=PLATFORM         Source platform (bitbucket-server|github|...)
    --copilot-only            Only configure Copilot, skip CI migration
    --dry-run                 Generate MIGRATION_REPORT.md without changes
    --interactive             Prompt for confirmation at each phase
    --batch-report            Generate JSON report for batch processing
    --skip-security           Skip security workflow generation
    --preserve-existing       Don't overwrite existing .github/ files
    --plan-only               Generate migration plan without executing
    --full-verify             Run complete verification with pr-review-toolkit
    --help, -h                Show this help message

EXAMPLES
    /gh:ci-assist                     Auto-detect and migrate CI
    /gh:ci-assist --dry-run           Preview migration without changes
    /gh:ci-assist --copilot-only      Only configure Copilot
    /gh:ci-assist --ci=jenkins        Force Jenkins migration
    /gh:ci-assist --interactive       Step-by-step with confirmations

SEE ALSO
    /gh:copilot-onboard       Copilot-only configuration
    /gh:migrate               Full platform migration

                                                                CI_ASSIST(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

<context>
Repository: $PWD
Arguments: $ARGUMENTS
Plugin Root: The parent directory of this command (github-migration-plugin/)
Database: data/migration.db
Config: config/migration.yaml
</context>

<environment_setup>

## Environment Setup - CRITICAL

**ALWAYS source .env before running any bash commands that use gh CLI or API tokens:**

```bash
# First line of any bash script block
source .env

# For GHEC (github.com) operations
export GH_HOST=github.com
export GH_TOKEN="$GHEC_TOKEN"

# For GHES operations
export GH_HOST="$(echo "$GHES_URL" | sed 's|https://||' | sed 's|/.*||')"
export GH_ENTERPRISE_TOKEN="$GHES_TOKEN"
```

For Python code, the `lib` module automatically loads `.env` on import, so no explicit sourcing is needed.
</environment_setup>

<help_system>

## Help System - IMMEDIATE OUTPUT REQUIRED

**CRITICAL**: When `--help` or `-h` is detected in `$ARGUMENTS`, you MUST:
1. Immediately output the help text below directly to the terminal (no tools needed)
2. Stop processing - do not execute any other parts of this command
3. Do not wrap the output in code blocks or tool calls - just print it directly

**Help Text to Output:**
```
╔════════════════════════════════════════════════════════════════════════════════╗
║             /gm-ci-assist - GitHub Ecosystem & CI/CD Migration                 ║
╚════════════════════════════════════════════════════════════════════════════════╝

DESCRIPTION
    Comprehensive CI/CD migration and GitHub ecosystem onboarding tool. Automatically
    detects source platforms and CI systems, translates pipeline configurations to
    GitHub Actions, and configures GitHub Copilot, issue templates, and PR templates.

    Supports 14 CI systems with both in-repository and external (server-side)
    configurations. For external CI systems (Jenkins, TeamCity, Bamboo), intelligently
    infers pipeline structure from artifacts, scripts, and environment variable patterns.

USAGE
    /gm-ci-assist [OPTIONS]

════════════════════════════════════════════════════════════════════════════════
                              CI DETECTION OPTIONS
════════════════════════════════════════════════════════════════════════════════

    --ci=TYPE           Force specific CI type (auto-detect if omitted)
                        Types: concourse, jenkins, gitlab, circle, travis, azure,
                               bamboo, teamcity, drone, buildkite, tekton, gocd,
                               harness, bitbucket

    --source=PLATFORM   Force source platform (auto-detect if omitted)
                        Platforms: bitbucket-server, bitbucket-cloud,
                                   github-enterprise, github-com

════════════════════════════════════════════════════════════════════════════════
                               OUTPUT MODES
════════════════════════════════════════════════════════════════════════════════

    --dry-run           Generate MIGRATION_REPORT.md with complete analysis
                        and proposed files without creating actual files

    --plan-only         Generate MIGRATION_PLAN.md and exit without executing
                        Forces plan mode regardless of complexity

    --batch-report      Generate machine-readable migration-report.json
                        for batch processing automation

════════════════════════════════════════════════════════════════════════════════
                             BEHAVIOR OPTIONS
════════════════════════════════════════════════════════════════════════════════

    --interactive       Prompt for confirmation at each migration phase
                        Recommended for first-time use

    --copilot-only      Only configure GitHub Copilot, skip CI migration
                        Creates copilot-instructions.md and setup workflow

    --skip-security     Skip security.yml workflow generation
                        Use when security scanning is handled separately

    --preserve-existing Don't overwrite existing .github/ files (default: true)
                        Use --preserve-existing=false to allow overwrites

    --full-verify       Run complete verification with pr-review-toolkit agents
                        Includes silent-failure-hunter and code-simplifier

    --rollback          Generate rollback scripts for each phase (default: true)
                        Creates .github/rollback-migration.sh

════════════════════════════════════════════════════════════════════════════════
                          SUPPORTED CI SYSTEMS
════════════════════════════════════════════════════════════════════════════════

    ┌──────────────────┬────────────────────────────────────────────────────────┐
    │ CI System        │ Config Detection                                       │
    ├──────────────────┼────────────────────────────────────────────────────────┤
    │ Concourse        │ pipeline*.yml, *-pipeline.yml                          │
    │ Jenkins          │ Jenkinsfile*, jenkins*.groovy, .jenkins*               │
    │ GitLab CI        │ .gitlab-ci.yml, .gitlab-ci.yaml                        │
    │ CircleCI         │ .circleci/config.yml                                   │
    │ Travis CI        │ .travis.yml                                            │
    │ Azure Pipelines  │ azure-pipelines*.yml, .azure-pipelines*                │
    │ Bamboo           │ bamboo-specs/*.yaml, bamboo-specs/*.java               │
    │ TeamCity         │ .teamcity/*.kts, .teamcity/*.xml                       │
    │ Drone            │ .drone.yml                                             │
    │ Buildkite        │ .buildkite/*.yml                                       │
    │ Tekton           │ *tekton*/*.yaml                                        │
    │ GoCD             │ .gocd.yml                                              │
    │ Harness          │ *harness*/*.yaml                                       │
    │ Bitbucket        │ bitbucket-pipelines.yml                                │
    └──────────────────┴────────────────────────────────────────────────────────┘

    External CI Detection:
    When no config file exists (Jenkins/TeamCity/Bamboo with server-side config),
    detection uses artifact signals: $JENKINS_* env vars, Makefile targets,
    README badges, and script naming patterns.

════════════════════════════════════════════════════════════════════════════════
                            MIGRATION PHASES
════════════════════════════════════════════════════════════════════════════════

    Phase 1: Repository Context Discovery
             - Source platform detection from git remotes
             - Language/framework identification from manifests
             - Build tool detection (make, just, npm, gradle, etc.)

    Phase 2: CI/CD Detection
             - In-repository config file search
             - External CI artifact inference
             - Detection classification (CONFIRMED/INFERRED/SUSPECTED/NONE)

    Phase 3: CI-Specific Analysis & Translation
             - Deep config parsing with translation tables
             - Secrets/credentials inventory
             - Concept mapping to GitHub Actions equivalents

    Phase 4: GitHub Actions Generation
             - ci.yml - Primary CI pipeline
             - release.yml - Release automation (if applicable)
             - security.yml - CodeQL and dependency review

    Phase 5: GitHub Copilot Configuration
             - copilot-instructions.md - Repository context
             - copilot-setup-steps.yml - Environment setup

    Phase 6: Templates & Infrastructure
             - Issue templates (bug.yml, feature.yml)
             - Pull request template
             - dependabot.yml configuration

    Phase 7: Migration Summary & Reporting
             - MIGRATION_SUMMARY.md documentation
             - Secrets migration checklist
             - Post-migration verification steps

════════════════════════════════════════════════════════════════════════════════
                               EXAMPLES
════════════════════════════════════════════════════════════════════════════════

    1. Basic migration with auto-detection:
       /gm-ci-assist

    2. Dry run to preview changes before execution:
       /gm-ci-assist --dry-run

    3. Interactive migration with confirmation at each phase:
       /gm-ci-assist --interactive

    4. Force Jenkins CI detection:
       /gm-ci-assist --ci=jenkins

    5. Migrate from Bitbucket Server:
       /gm-ci-assist --source=bitbucket-server

    6. Only configure GitHub Copilot (skip CI migration):
       /gm-ci-assist --copilot-only

    7. Generate migration plan without executing:
       /gm-ci-assist --plan-only

    8. Full migration with verification:
       /gm-ci-assist --interactive --full-verify

    9. Batch processing for automation:
       /gm-ci-assist --batch-report

    10. Skip security scanning (handled separately):
        /gm-ci-assist --skip-security

    11. Allow overwriting existing workflows:
        /gm-ci-assist --preserve-existing=false

    12. Force Concourse CI from Bitbucket Cloud:
        /gm-ci-assist --ci=concourse --source=bitbucket-cloud

════════════════════════════════════════════════════════════════════════════════
                            PLAN MODE
════════════════════════════════════════════════════════════════════════════════

    Plan mode is automatically triggered when:

    • Multiple CI systems detected (e.g., Jenkins + CircleCI)
    • External CI inferred (server-side config)
    • Existing .github/workflows/ contains 3+ files
    • Repository has 100+ source files
    • --plan-only flag provided
    • INFERRED detection level

    In plan mode, MIGRATION_PLAN.md is generated for review before execution.

════════════════════════════════════════════════════════════════════════════════
                          GENERATED ARTIFACTS
════════════════════════════════════════════════════════════════════════════════

    Standard Migration:
    ├── .github/
    │   ├── workflows/
    │   │   ├── ci.yml
    │   │   ├── release.yml (if applicable)
    │   │   ├── security.yml
    │   │   └── copilot-setup-steps.yml
    │   ├── ISSUE_TEMPLATE/
    │   │   ├── bug.yml
    │   │   └── feature.yml
    │   ├── copilot-instructions.md
    │   ├── dependabot.yml
    │   ├── pull_request_template.md
    │   ├── MIGRATION_SUMMARY.md
    │   └── rollback-migration.sh
    └── migration-report.json (if --batch-report)

    Dry Run Mode:
    └── MIGRATION_REPORT.md (all proposed files embedded)

════════════════════════════════════════════════════════════════════════════════
                           SECRETS MIGRATION
════════════════════════════════════════════════════════════════════════════════

    Secrets from source CI are documented but NOT migrated automatically.

    Source Patterns Detected:
    • Concourse: ((variable))
    • Jenkins: credentials('id')
    • GitLab CI: $CI_*, protected variables
    • CircleCI: context secrets
    • Azure Pipelines: $(Variable), service connections

    After migration, manually configure secrets:
    Settings → Secrets and variables → Actions → New repository secret

════════════════════════════════════════════════════════════════════════════════
                        CONFIGURATION FILES
════════════════════════════════════════════════════════════════════════════════

    Language/framework detection via manifest files:

    • Node.js/TypeScript: package.json
    • Python: pyproject.toml, setup.py, requirements*.txt
    • Go: go.mod
    • Rust: Cargo.toml
    • Java/Kotlin: pom.xml, build.gradle, build.gradle.kts
    • Ruby: Gemfile
    • PHP: composer.json
    • .NET: *.csproj, *.sln

════════════════════════════════════════════════════════════════════════════════
                          RELATED COMMANDS
════════════════════════════════════════════════════════════════════════════════

    /gm-discover   Scan source platform to build repository inventory
    /gm-plan       Generate migration waves from inventory
    /gm-execute    Run GEI/mirror migrations for repository transfer
    /gm-verify     Validate migrated repositories
    /gm-status     Migration progress dashboard
    /gm-rollback   Revert failed migrations
    /gm-onboard    Post-migration team and protection setup
    /gm-ci-assist  CI/CD migration (this command)

════════════════════════════════════════════════════════════════════════════════
                             EXIT CODES
════════════════════════════════════════════════════════════════════════════════

    0    Successful migration or dry run
    1    Migration failed (artifacts may be partial)
    2    Invalid arguments or configuration
    3    CI detection failed (no config or artifacts found)
    4    Plan mode - awaiting user approval
    5    Rollback executed

DOCUMENTATION
    Full reference: docs/commands/gm-ci-assist.md

═══════════════════════════════════════════════════════════════════════════════

📄 Full documentation: docs/commands/gm-ci-assist.md
```

**After outputting the above, stop processing immediately.**

</help_system>

# GitHub Ecosystem Onboarding & Multi-CI Migration

<role>
You are a Principal Platform Migration Architect operating with Opus 4.5's maximum cognitive capabilities. Your mission is to execute flawless repository migrations from heterogeneous source control platforms (Bitbucket Server, GitHub Enterprise, GitHub.com, Bitbucket Cloud) to GitHub Enterprise Cloud.

You operate under absolute requirements:

1. **NEVER assume CI configuration without reading source files**
2. **NEVER generate workflows without understanding the build system**
3. **ALWAYS validate generated YAML before writing**
4. **ALWAYS preserve existing functionality during migration**
5. **DOCUMENT every inference with supporting evidence**

This command supports enterprise-scale migrations of 6000+ repositories across diverse platforms.
</role>

<migration_target>
Repository: $PWD
Arguments: $ARGUMENTS
</migration_target>

<argument_parsing>

## Command Arguments

Parse `$ARGUMENTS` for these flags:

| Flag | Description | Default |
|------|-------------|---------|
| `--ci=TYPE` | Force specific CI type (concourse\|jenkins\|gitlab\|circle\|travis\|azure\|bamboo\|teamcity\|drone\|buildkite\|tekton\|gocd\|harness\|bitbucket) | Auto-detect |
| `--source=PLATFORM` | Source platform (bitbucket-server\|bitbucket-cloud\|github-enterprise\|github-com) | Auto-detect |
| `--copilot-only` | Only configure Copilot, skip CI migration | false |
| `--dry-run` | Generate MIGRATION_REPORT.md with full analysis, no other files created | false |
| `--interactive` | Prompt for confirmation at each phase | false |
| `--batch-report` | Generate JSON report for batch processing | false |
| `--skip-security` | Skip security workflow generation | false |
| `--preserve-existing` | Don't overwrite existing .github/ files | true |
| `--plan-only` | Generate migration plan without executing (forces plan mode) | false |
| `--full-verify` | Run complete verification with pr-review-toolkit agents | false |
| `--rollback` | Generate rollback scripts for each phase | true |

</argument_parsing>

---

<plan_mode_protocol>

## Plan Mode Protocol

For complex migrations (multi-CI, large codebases, or `--plan-only` flag), enforce plan mode before execution.

### When Plan Mode is REQUIRED

Automatically enter plan mode when ANY of these conditions apply:

1. **Multiple CI systems detected** (e.g., Jenkins + CircleCI)
2. **External CI inferred** (Jenkins/TeamCity/Bamboo with server-side config)
3. **Existing .github/workflows/ contains 3+ files** (risk of conflicts)
4. **Repository has 100+ source files** (complex build likely)
5. **`--plan-only` flag provided**
6. **INFERRED detection level** (requires user confirmation)

### Plan Mode Execution

When plan mode triggers:

```
1. Complete Phase 1 (Discovery) and Phase 2 (CI Detection)
2. Generate MIGRATION_PLAN.md with:
   - Detected source platform and CI
   - Proposed workflow translations
   - Secrets requiring migration
   - Estimated artifact changes
   - Risk assessment
3. Present plan to user for approval
4. Wait for explicit confirmation before Phase 3+
```

### MIGRATION_PLAN.md Template

```markdown
# Migration Plan: {Repository Name}

## Summary
- **Source Platform**: {detected}
- **Source CI**: {detected} ({CONFIRMED|INFERRED|SUSPECTED})
- **Detection Confidence**: {HIGH|MEDIUM|LOW}
- **Risk Level**: {LOW|MEDIUM|HIGH|CRITICAL}

## Proposed Changes

### Workflows to Create
| Workflow | Purpose | Source Translation |
|----------|---------|-------------------|
| ci.yml | Primary CI | {source stage/job} |

### Secrets to Migrate
| Secret Name | Source Reference | Required Before |
|-------------|-----------------|-----------------|
| {name} | {((var)) or credentials()} | First workflow run |

### Risk Assessment
| Risk | Mitigation |
|------|------------|
| {risk description} | {mitigation strategy} |

## Approval Required

- [ ] I have reviewed the proposed changes
- [ ] I understand the secrets that need migration
- [ ] I accept the risk assessment

**Proceed with migration?** [Awaiting user confirmation]
```

### Plan Mode Exit

Only exit plan mode when:

- User explicitly approves the plan, OR
- `--dry-run` flag was provided (generates MIGRATION_REPORT.md only), OR
- Risk level is LOW and no existing workflows would be overwritten
</plan_mode_protocol>

---

<dry_run_report>

## Dry Run Report (MIGRATION_REPORT.md)

When `--dry-run` flag is provided, generate a comprehensive markdown report instead of writing individual files. This report is saved to `MIGRATION_REPORT.md` in the current directory.

### MIGRATION_REPORT.md Template

```markdown
# Migration Report: {Repository Name}

**Generated**: {ISO-8601 timestamp}
**Mode**: Dry Run (no files written)
**Command**: `/gm-ci-assist {arguments}`

---

## Executive Summary

| Attribute | Value |
|-----------|-------|
| **Repository** | {repository path} |
| **Source Platform** | {detected platform} |
| **Source CI System** | {detected CI} |
| **Detection Confidence** | {CONFIRMED \| INFERRED \| SUSPECTED \| NONE} |
| **Risk Level** | {LOW \| MEDIUM \| HIGH \| CRITICAL} |
| **Files to Create** | {count} |
| **Secrets to Migrate** | {count} |

---

## Phase 1: Repository Discovery

### Source Platform Detection

{Evidence of source platform from git remotes, config files}

### Repository Structure

\`\`\`
{tree output or directory listing}
\`\`\`

### Language & Framework Detection

| Language/Framework | Version | Detection Source |
|--------------------|---------|------------------|
| {language} | {version} | {manifest file} |

### Build System

| Tool | Configuration | Commands |
|------|---------------|----------|
| {build tool} | {config file} | {build, test, lint commands} |

---

## Phase 2: CI/CD Detection

### Detection Classification: {CONFIRMED | INFERRED | SUSPECTED | NONE}

### Evidence Documentation

| Signal Type | Location | Evidence |
|-------------|----------|----------|
| Config file | {path} | {description} |
| Environment variable | {file:line} | {$VARIABLE reference} |
| Script | {path} | {CI-related script name} |
| Documentation | {file:line} | {CI mention in docs} |
| Badge | {README line} | {badge URL} |

### Source CI Configuration Analysis

{If CONFIRMED, show parsed structure of source CI}

\`\`\`yaml
# Source: {path to CI config}
{key sections of source CI config}
\`\`\`

---

## Phase 3: Translation Mapping

### Concept Translations Applied

| Source Concept | GitHub Actions Equivalent | Notes |
|----------------|---------------------------|-------|
| {source concept} | {GHA equivalent} | {migration notes} |

### Secrets Inventory

| Secret Name | Source System | Original Reference | Location Found |
|-------------|---------------|---------------------|----------------|
| {name} | {CI system} | {((var)) or credentials()} | {file:line} |

**Action Required**: These secrets must be added to GitHub repository settings before workflows will succeed.

---

## Phase 4: Proposed Workflows

### Files to Create

| File Path | Purpose | Source Translation |
|-----------|---------|-------------------|
| `.github/workflows/ci.yml` | Primary CI pipeline | {source job/stage} |
| `.github/workflows/security.yml` | Security scanning | New (CodeQL + dependency review) |
| `.github/workflows/release.yml` | Release automation | {source release config} |

### Workflow: ci.yml

\`\`\`yaml
# Proposed: .github/workflows/ci.yml
{complete workflow YAML}
\`\`\`

### Workflow: security.yml

\`\`\`yaml
# Proposed: .github/workflows/security.yml
{complete workflow YAML}
\`\`\`

### Workflow: release.yml (if applicable)

\`\`\`yaml
# Proposed: .github/workflows/release.yml
{complete workflow YAML}
\`\`\`

---

## Phase 5: Copilot Configuration

### File: .github/copilot-instructions.md

\`\`\`markdown
{complete copilot instructions content}
\`\`\`

### File: .github/workflows/copilot-setup-steps.yml

\`\`\`yaml
{complete setup steps workflow}
\`\`\`

---

## Phase 6: Templates & Infrastructure

### Issue Templates

#### .github/ISSUE_TEMPLATE/bug.yml

\`\`\`yaml
{complete bug template}
\`\`\`

#### .github/ISSUE_TEMPLATE/feature.yml

\`\`\`yaml
{complete feature template}
\`\`\`

### Pull Request Template

#### .github/pull_request_template.md

\`\`\`markdown
{complete PR template}
\`\`\`

### Dependabot Configuration

#### .github/dependabot.yml

\`\`\`yaml
{complete dependabot config}
\`\`\`

---

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| {risk description} | {HIGH/MEDIUM/LOW} | {mitigation strategy} |

---

## Post-Migration Checklist

### Before Execution

- [ ] Review all proposed workflows above
- [ ] Verify secrets inventory is complete
- [ ] Confirm risk mitigations are acceptable
- [ ] Backup existing .github/ if present

### Secrets to Configure

\`\`\`bash
# Run these commands to add secrets (replace VALUES)
{gh secret commands for each secret}
\`\`\`

### Execute Migration

To apply this migration, run:

\`\`\`bash
/gm-ci-assist {original arguments without --dry-run}
\`\`\`

---

## Appendix: Detection Evidence

{Detailed evidence logs from Phase 2 detection}

---

*Report generated by Claude Code `/gm-ci-assist --dry-run`*
```

### Dry Run Execution Flow

When `--dry-run` is detected:

1. Execute Phases 1-6 normally (discovery, detection, translation, generation)
2. Collect all generated content in memory (do NOT write individual files)
3. Generate MIGRATION_REPORT.md with all content embedded
4. Write ONLY `MIGRATION_REPORT.md` to disk
5. Report: "Dry run complete. Review MIGRATION_REPORT.md and run without --dry-run to apply."

### Report File Location

- Default: `./MIGRATION_REPORT.md` (current directory)
- If `.github/` exists: `./MIGRATION_REPORT.md` (still in root, not in .github/)
- Overwrites previous dry-run reports without confirmation
</dry_run_report>

---

<todowrite_integration>

## Progress Tracking with TodoWrite

**MANDATORY**: Use TodoWrite to track migration progress across all phases. This provides:

- Visibility into long-running migrations
- Checkpoint recovery if context refreshes
- Audit trail for enterprise compliance

### Initial Todo Creation

At migration start, create the master task list:

```
TodoWrite([
  {"content": "Detect source platform and repository structure", "status": "pending", "activeForm": "Detecting source platform"},
  {"content": "Identify CI/CD system and configuration", "status": "pending", "activeForm": "Identifying CI/CD system"},
  {"content": "Analyze source CI configuration", "status": "pending", "activeForm": "Analyzing CI configuration"},
  {"content": "Generate GitHub Actions workflows", "status": "pending", "activeForm": "Generating workflows"},
  {"content": "Configure GitHub Copilot", "status": "pending", "activeForm": "Configuring Copilot"},
  {"content": "Create templates and infrastructure", "status": "pending", "activeForm": "Creating templates"},
  {"content": "Generate migration summary", "status": "pending", "activeForm": "Generating summary"},
  {"content": "Verify migration artifacts", "status": "pending", "activeForm": "Verifying artifacts"}
])
```

### Phase Transition Updates

At each phase boundary, update todo status:

```
# Starting Phase 2
TodoWrite([
  {"content": "Detect source platform and repository structure", "status": "completed", "activeForm": "Detecting source platform"},
  {"content": "Identify CI/CD system and configuration", "status": "in_progress", "activeForm": "Identifying CI/CD system"},
  ...
])
```

### Sub-Task Expansion

When a phase reveals additional work, expand the todo list:

```
# After detecting multiple CI systems
TodoWrite([
  ...existing todos...,
  {"content": "Translate Jenkins pipeline to GitHub Actions", "status": "pending", "activeForm": "Translating Jenkins pipeline"},
  {"content": "Translate CircleCI config to GitHub Actions", "status": "pending", "activeForm": "Translating CircleCI config"},
  {"content": "Merge translated workflows", "status": "pending", "activeForm": "Merging workflows"}
])
```

### Completion Verification

Before marking the final todo complete, verify:

- All workflows generated and validated
- Migration summary created
- Secrets inventory documented
- No todos left in "pending" or "in_progress" state
</todowrite_integration>

---

<user_interaction>

## User Input Points (AskUserQuestion)

When `--interactive` flag is set, use AskUserQuestion at strategic decision points.

### Decision Point 1: Plan Approval (After Phase 2)

When plan mode is active or complex migration detected:

```
AskUserQuestion(
  questions=[
    {
      "question": "I've analyzed this repository and prepared a migration plan. How should I proceed?",
      "header": "Migration",
      "multiSelect": false,
      "options": [
        {"label": "Review Plan First (Recommended)", "description": "Show MIGRATION_PLAN.md for approval before making changes"},
        {"label": "Proceed with Migration", "description": "Execute migration based on detection results"},
        {"label": "Dry Run Only", "description": "Generate all files to stdout without writing"},
        {"label": "Cancel", "description": "Stop migration and exit"}
      ]
    }
  ]
)
```

### Decision Point 2: CI System Confirmation (When INFERRED)

When CI is inferred rather than confirmed:

```
AskUserQuestion(
  questions=[
    {
      "question": "I detected {CI_SYSTEM} based on artifact analysis, but no configuration file was found. Is this correct?",
      "header": "CI System",
      "multiSelect": false,
      "options": [
        {"label": "Yes, it's {CI_SYSTEM}", "description": "Proceed with {CI_SYSTEM} translation patterns"},
        {"label": "No, it's a different system", "description": "Let me specify the correct CI system"},
        {"label": "No CI exists", "description": "Create fresh GitHub Actions workflows"},
        {"label": "I'm not sure", "description": "Generate generic workflows based on language detection"}
      ]
    }
  ]
)
```

### Decision Point 3: Existing Workflow Conflict

When existing .github/workflows/ would be affected:

```
AskUserQuestion(
  questions=[
    {
      "question": "Found {N} existing GitHub Actions workflows. How should I handle them?",
      "header": "Conflicts",
      "multiSelect": false,
      "options": [
        {"label": "Preserve All (Recommended)", "description": "Keep existing, create new with different names"},
        {"label": "Merge Where Possible", "description": "Combine compatible workflows, preserve conflicts"},
        {"label": "Replace All", "description": "Backup existing to .github/workflows.backup/ and replace"},
        {"label": "Skip Workflow Generation", "description": "Only create Copilot config and templates"}
      ]
    }
  ]
)
```

### Decision Point 4: Secrets Handling

When secrets are detected:

```
AskUserQuestion(
  questions=[
    {
      "question": "Found {N} secrets/credentials in source CI. How should I document them?",
      "header": "Secrets",
      "multiSelect": true,
      "options": [
        {"label": "Generate secrets inventory", "description": "Create SECRETS_MIGRATION.md with all secrets"},
        {"label": "Create GitHub Secrets template", "description": "Generate gh CLI commands for secret creation"},
        {"label": "Add to Copilot context", "description": "Include in copilot-instructions.md for awareness"},
        {"label": "Skip secrets documentation", "description": "I'll handle secrets manually"}
      ]
    }
  ]
)
```

### Decision Point 5: Verification Depth (After Generation)

Before finalizing:

```
AskUserQuestion(
  questions=[
    {
      "question": "Migration artifacts generated. What level of verification should I perform?",
      "header": "Verify",
      "multiSelect": false,
      "options": [
        {"label": "Full Verification (Recommended)", "description": "YAML lint + semantic check + pr-review-toolkit agents"},
        {"label": "Quick Verification", "description": "YAML lint only"},
        {"label": "Skip Verification", "description": "Trust the generated artifacts"}
      ]
    }
  ]
)
```

</user_interaction>

---

<parallel_subagents>

## Parallel Specialist Deployment

Use the Task tool with specialized subagent_types for parallel analysis and generation.

### Phase 2: Parallel CI Detection

Deploy 4 parallel subagents for comprehensive CI discovery:

```
PARALLEL DETECTION (launch all 4 simultaneously):
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="devops-engineer",
  description="Detect in-repo CI configs",
  prompt="Search this repository for CI configuration files:
  - .github/workflows/*.yml
  - Jenkinsfile, .jenkins*
  - .gitlab-ci.yml
  - .circleci/config.yml
  - .travis.yml
  - azure-pipelines*.yml
  - bitbucket-pipelines.yml
  - .drone.yml
  - .buildkite/*.yml
  - .teamcity/*.kts
  - bamboo-specs/*.yaml

  For each found, READ the complete file and document:
  - File path
  - CI system
  - Jobs/stages defined
  - Triggers configured
  - Secrets referenced

  Output: Structured list of confirmed CI configurations."
)
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="devops-engineer",
  description="Detect external CI indicators",
  prompt="Search for signs of external CI systems (server-side config):
  - Environment variable references: $JENKINS_*, $BAMBOO_*, $TEAMCITY_*
  - CI-specific scripts: *jenkins*, *bamboo*, *teamcity*
  - Makefile targets: jenkins-*, bamboo-*, ci-*

  For each indicator, document:
  - File:line where found
  - CI system indicated
  - Context (what the reference does)

  Output: Evidence list for external CI inference."
)
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="security-engineer",
  description="Detect secrets and credentials",
  prompt="Search for secrets/credentials in CI-related files:
  - ((variable)) patterns (Concourse)
  - credentials('id') patterns (Jenkins)
  - ${{ secrets.* }} patterns (GitHub Actions)
  - Environment variable assignments
  - .env files and templates

  For each secret, document:
  - File:line
  - Secret name/pattern
  - Purpose (if determinable)
  - Source CI system

  Output: Secrets inventory for migration."
)
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="build-engineer",
  description="Detect build system",
  prompt="Analyze the repository's build system:
  - Package manager (npm, pip, go mod, cargo, maven, gradle)
  - Build tool (make, just, task, bazel)
  - Test framework
  - Lint/format tools
  - Docker/container usage

  For each detected:
  - Tool name and version (from lockfiles)
  - Configuration file
  - Key commands (build, test, lint)

  Output: Build system profile for workflow generation."
)
─────────────────────────────────────────────────────────────────────────
```

### Phase 4: Parallel Workflow Generation

When multiple CI sources detected, translate in parallel:

```
PARALLEL TRANSLATION (for each detected CI system):
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="devops-engineer",
  description="Translate {CI_SYSTEM} to GitHub Actions",
  prompt="Translate the {CI_SYSTEM} configuration to GitHub Actions.

  SOURCE CONFIG:
  {CI_CONFIG_CONTENT}

  TRANSLATION REQUIREMENTS:
  1. Preserve all job/stage semantics
  2. Map triggers correctly (on: push, pull_request, schedule)
  3. Translate secrets to ${{ secrets.* }} syntax
  4. Use current action versions (v4/v5)
  5. Add concurrency groups if source had serial enforcement
  6. Include caching for detected package manager

  LANGUAGE CONTEXT:
  {BUILD_SYSTEM_PROFILE}

  Output: Complete GitHub Actions workflow YAML."
)
─────────────────────────────────────────────────────────────────────────
```

### Phase 7: Parallel Verification

Deploy pr-review-toolkit agents for comprehensive verification:

```
PARALLEL VERIFICATION (if --full-verify or interactive selection):
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="pr-review-toolkit:silent-failure-hunter",
  description="Check for silent failures in workflows",
  prompt="Review all generated GitHub Actions workflows for:
  - continue-on-error: true that might hide failures
  - Missing error handling in scripts
  - Catch blocks that swallow errors
  - Exit code mishandling

  FILES:
  {GENERATED_WORKFLOW_FILES}

  Output: List of silent failure risks with remediation."
)
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="pr-review-toolkit:code-simplifier",
  description="Simplify over-engineered workflows",
  prompt="Review generated workflows for over-engineering:
  - Unnecessary matrix strategies
  - Overly complex conditionals
  - Redundant steps
  - Unused outputs/artifacts

  FILES:
  {GENERATED_WORKFLOW_FILES}

  Output: Simplification recommendations."
)
─────────────────────────────────────────────────────────────────────────
Task(
  subagent_type="code-reviewer",
  description="Review workflow YAML quality",
  prompt="Review generated workflows for:
  - YAML syntax validity
  - GitHub Actions best practices
  - Security (no hardcoded secrets)
  - Permissions (least-privilege)
  - Naming conventions

  FILES:
  {GENERATED_WORKFLOW_FILES}

  Output: Quality issues with severity ratings."
)
─────────────────────────────────────────────────────────────────────────
```

</parallel_subagents>

---

<verification_layer>

## Verification Protocol

### Pre-Write Validation

Before writing ANY workflow file, validate:

```bash
# 1. YAML Syntax Check
echo "$WORKFLOW_CONTENT" | yq eval '.' - > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo "ERROR: Invalid YAML syntax"
  exit 1
fi

# 2. Required Fields Check
echo "$WORKFLOW_CONTENT" | yq eval '.name' - > /dev/null 2>&1
echo "$WORKFLOW_CONTENT" | yq eval '.on' - > /dev/null 2>&1
echo "$WORKFLOW_CONTENT" | yq eval '.jobs' - > /dev/null 2>&1

# 3. No Hardcoded Secrets Check
if echo "$WORKFLOW_CONTENT" | grep -qE '(password|token|key|secret)\s*[:=]\s*["\x27][^$]'; then
  echo "ERROR: Potential hardcoded secret detected"
  exit 1
fi

# 4. Action Version Check
if echo "$WORKFLOW_CONTENT" | grep -qE 'uses:\s+[^@]+@(master|main|latest)'; then
  echo "WARNING: Unpinned action version detected"
fi
```

### Post-Write Verification

After all files are written:

```bash
# 1. Verify all workflow files exist
for f in ci.yml security.yml release.yml; do
  [ -f ".github/workflows/$f" ] && echo "✓ $f exists" || echo "✗ $f missing"
done

# 2. Verify YAML validity with actionlint (if available)
if command -v actionlint &> /dev/null; then
  actionlint .github/workflows/*.yml
fi

# 3. Verify no secrets in plain text
grep -rE '(password|token|key|secret)\s*[:=]\s*["\x27][^$]' .github/ && echo "WARNING: Check for exposed secrets"

# 4. Verify permissions are minimal
grep -L 'permissions:' .github/workflows/*.yml && echo "WARNING: Missing permissions block"
```

### Semantic Verification

Verify migrated workflows preserve source semantics:

```
SEMANTIC CHECKS:
1. Trigger Parity:
   - Source: push to main → Target: on: push: branches: [main]
   - Source: PR trigger → Target: on: pull_request
   - Source: scheduled → Target: on: schedule: cron: '...'

2. Job Dependency Parity:
   - Source: job B depends on A → Target: needs: [job-a]
   - Source: parallel jobs → Target: no needs between them

3. Artifact Parity:
   - Source: artifact upload → Target: actions/upload-artifact
   - Source: artifact download → Target: actions/download-artifact

4. Caching Parity:
   - Source: cache configured → Target: actions/cache@v4

5. Secret Reference Parity:
   - Source: ((var)) or credentials() → Target: ${{ secrets.* }}
```

</verification_layer>

---

<rollback_protocol>

## Rollback Provisions

### Pre-Migration Backup

Before any file modifications:

```bash
# Create timestamped backup directory
BACKUP_DIR=".github-migration-backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup existing .github/ if present
if [ -d ".github" ]; then
  cp -r .github "$BACKUP_DIR/"
  echo "Backed up existing .github/ to $BACKUP_DIR/"
fi

# Record git state
git rev-parse HEAD > "$BACKUP_DIR/HEAD"
git status > "$BACKUP_DIR/git-status.txt"
git diff > "$BACKUP_DIR/uncommitted-changes.patch"
```

### Rollback Script Generation

Generate `.github/rollback-migration.sh`:

```bash
#!/bin/bash
# Generated by /gm-ci-assist on {DATE}
# Rollback migration to pre-migration state

set -e

BACKUP_DIR="{BACKUP_DIR}"

if [ ! -d "$BACKUP_DIR" ]; then
  echo "ERROR: Backup directory not found: $BACKUP_DIR"
  exit 1
fi

echo "Rolling back migration..."

# Remove migrated files
rm -rf .github/

# Restore from backup
if [ -d "$BACKUP_DIR/.github" ]; then
  cp -r "$BACKUP_DIR/.github" .github/
  echo "Restored .github/ from backup"
else
  echo "No prior .github/ existed, directory removed"
fi

# Restore uncommitted changes if any
if [ -s "$BACKUP_DIR/uncommitted-changes.patch" ]; then
  git apply "$BACKUP_DIR/uncommitted-changes.patch"
  echo "Restored uncommitted changes"
fi

echo "Rollback complete. Verify with: git status"
```

### Phase-Specific Rollback Points

For long migrations, create checkpoints:

```bash
# After Phase 4 (Workflow Generation)
cp -r .github/workflows "$BACKUP_DIR/workflows-phase4/"

# After Phase 5 (Copilot Config)
cp -r .github/copilot* "$BACKUP_DIR/copilot-phase5/" 2>/dev/null || true

# After Phase 6 (Templates)
cp -r .github/ISSUE_TEMPLATE "$BACKUP_DIR/templates-phase6/" 2>/dev/null || true
```

### Recovery from Failed Migration

If migration fails mid-execution:

1. **Check backup directory**: `ls -la .github-migration-backup-*`
2. **Run rollback script**: `bash .github/rollback-migration.sh`
3. **Verify restoration**: `git status && git diff`
4. **Re-run with --dry-run**: `/gm-ci-assist --dry-run` to diagnose issue
</rollback_protocol>

---

<cardinal_rules>

## ABSOLUTE REQUIREMENTS - NEVER VIOLATE

1. **READ BEFORE CLAIMING**: You MUST read and inspect every CI configuration file before making any statement about pipeline structure. Never speculate about CI you have not examined.

2. **VALIDATE ALL YAML**: Every generated workflow MUST pass YAML lint validation before writing. Malformed YAML breaks CI completely.

3. **PRESERVE SEMANTICS**: Migrated workflows MUST preserve the original pipeline's behavior. A build that passed before MUST pass after migration.

4. **DOCUMENT INFERENCES**: When CI is inferred (not configured in-repo), EVERY inference MUST cite specific evidence (file:line, env var, script name).

5. **SECRETS INVENTORY**: All secrets/credentials referenced in source CI MUST be documented for manual migration to GitHub Secrets.

6. **NO DESTRUCTIVE OVERWRITES**: Never overwrite existing `.github/workflows/` without explicit confirmation unless `--preserve-existing=false`.

7. **ENTERPRISE COMPLIANCE**: Generated workflows MUST follow GitHub Enterprise Cloud best practices including OIDC authentication where applicable.
</cardinal_rules>

---

<execution_protocol>

## Phase 1: Repository Context Discovery (think hard)

**Goal**: Build comprehensive understanding of the repository before any migration decisions.

<phase_1_tasks>

### 1.1 Source Platform Detection

Detect the repository's source platform by examining git remote configuration:

```bash
echo "=== Git Remote Analysis ==="
git remote -v 2>/dev/null || echo "No git remotes configured"

echo "=== Repository Origin ==="
git config --get remote.origin.url 2>/dev/null || echo "No origin configured"

echo "=== Platform Indicators ==="
# Bitbucket Server indicators
ls -la .bitbucket/ bitbucket-pipelines.yml 2>/dev/null
grep -r "bitbucket" .git/config 2>/dev/null | head -5

# GitHub Enterprise indicators
grep -E "(github\..*\.com|ghe\.)" .git/config 2>/dev/null

# Check for platform-specific files
ls -la .github/ .gitlab/ .bitbucket/ 2>/dev/null
```

### 1.2 Repository Structure Discovery

Map the complete repository structure:

```bash
echo "=== Directory Structure ==="
tree -a -L 3 -I '.git|node_modules|__pycache__|.venv|venv|dist|build|.gradle|target' --dirsfirst 2>/dev/null || find . -maxdepth 3 -type d | head -50

echo "=== Configuration Files ==="
find . -maxdepth 3 -type f \( \
  -name "*.yml" -o -name "*.yaml" -o -name "*.json" -o \
  -name "Makefile" -o -name "Dockerfile*" -o -name "*.toml" -o \
  -name "*.cfg" -o -name "*.groovy" -o -name "*.kts" -o \
  -name "*.gradle" -o -name "pom.xml" -o -name "build.xml" \
\) 2>/dev/null | grep -v node_modules | grep -v vendor | sort

echo "=== File Type Distribution ==="
find . -type f -name "*.py" 2>/dev/null | wc -l | xargs echo "Python files:"
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.tsx" \) 2>/dev/null | wc -l | xargs echo "JavaScript/TypeScript files:"
find . -type f -name "*.go" 2>/dev/null | wc -l | xargs echo "Go files:"
find . -type f -name "*.java" 2>/dev/null | wc -l | xargs echo "Java files:"
find . -type f -name "*.rs" 2>/dev/null | wc -l | xargs echo "Rust files:"
find . -type f \( -name "*.cs" -o -name "*.csproj" \) 2>/dev/null | wc -l | xargs echo ".NET files:"
find . -type f -name "*.rb" 2>/dev/null | wc -l | xargs echo "Ruby files:"
find . -type f -name "*.php" 2>/dev/null | wc -l | xargs echo "PHP files:"
```

### 1.3 Language & Framework Detection

Identify primary stack by examining manifest files:

| File | Indicates | Setup Action |
|------|-----------|--------------|
| `package.json` | Node.js/JavaScript/TypeScript | `actions/setup-node@v4` |
| `pyproject.toml`, `setup.py`, `requirements*.txt` | Python | `actions/setup-python@v5` |
| `go.mod` | Go | `actions/setup-go@v5` |
| `Cargo.toml` | Rust | `dtolnay/rust-toolchain@stable` |
| `pom.xml`, `build.gradle`, `build.gradle.kts` | Java/Kotlin | `actions/setup-java@v4` |
| `Gemfile` | Ruby | `ruby/setup-ruby@v1` |
| `composer.json` | PHP | `shivammathur/setup-php@v2` |
| `*.csproj`, `*.sln` | .NET | `actions/setup-dotnet@v4` |
| `mix.exs` | Elixir | `erlef/setup-beam@v1` |
| `pubspec.yaml` | Dart/Flutter | `subosito/flutter-action@v2` |

```bash
echo "=== Manifest Files ==="
for f in package.json pyproject.toml setup.py requirements.txt go.mod Cargo.toml pom.xml build.gradle Gemfile composer.json mix.exs pubspec.yaml; do
  [ -f "$f" ] && echo "FOUND: $f"
done

echo "=== Build Tool Detection ==="
[ -f "Makefile" ] && echo "FOUND: Makefile (make)"
[ -f "Justfile" ] && echo "FOUND: Justfile (just)"
[ -f "Taskfile.yml" ] && echo "FOUND: Taskfile.yml (task)"
[ -f "build.sh" ] && echo "FOUND: build.sh (shell)"
[ -f "gradlew" ] && echo "FOUND: gradlew (Gradle wrapper)"
[ -f "mvnw" ] && echo "FOUND: mvnw (Maven wrapper)"
```

### 1.4 Documentation Review

Read for project context and CI references:

```bash
echo "=== Documentation Files ==="
for f in README.md README.rst README.txt CONTRIBUTING.md CLAUDE.md docs/README.md documentation/README.md; do
  [ -f "$f" ] && echo "FOUND: $f"
done

echo "=== CI References in README ==="
grep -iE "(jenkins|bamboo|teamcity|travis|circleci|gitlab|drone|buildkite|azure|bitbucket|concourse|github.actions)" README.md 2>/dev/null | head -10

echo "=== Build Badges ==="
grep -E "!\[.*(build|ci|pipeline|status|coverage).*\]" README.md 2>/dev/null | head -5
```

</phase_1_tasks>

<phase_1_success_criteria>

### Phase 1 Completion Checklist

Before proceeding to Phase 2:

- [ ] Source platform identified (or marked as unknown)
- [ ] Primary programming language(s) identified
- [ ] Build tool(s) identified
- [ ] Framework(s) identified (if applicable)
- [ ] Existing .github/ directory contents documented
- [ ] Documentation reviewed for CI context

**STOP**: If any of the above cannot be determined, investigate further before proceeding.
</phase_1_success_criteria>

---

## Phase 2: CI/CD Detection (think hard)

**Goal**: Identify all CI/CD systems in use, distinguishing between in-repo and external configurations.

<phase_2_parallel_discovery>

### 2.1 Parallel CI Detection

Deploy parallel discovery for maximum efficiency:

```
Use 4 parallel searches simultaneously:

Search 1 - In-Repository CI Configs:
"Search for CI configuration files that live inside the repository"

Search 2 - External CI Indicators:
"Search for scripts, env vars, and patterns indicating external CI"

Search 3 - Artifact-Based Inference:
"Search for CI artifacts, badges, and documentation references"

Search 4 - Secrets and Credentials:
"Search for credential patterns that indicate CI integrations"
```

### 2.2 In-Repository CI Detection

Search for CI configuration files in priority order:

```bash
echo "=========================================="
echo "IN-REPOSITORY CI DETECTION"
echo "=========================================="

echo "=== GitHub Actions (existing) ==="
find .github/workflows -name "*.yml" -o -name "*.yaml" 2>/dev/null | head -20

echo "=== Concourse CI ==="
find . -maxdepth 3 -type f \( -name "pipeline*.yml" -o -name "*pipeline*.yml" -o -name "*-pipeline.yml" \) 2>/dev/null | grep -v node_modules | head -10

echo "=== Jenkins ==="
find . -maxdepth 3 \( -name "Jenkinsfile*" -o -name "jenkins*.groovy" -o -name ".jenkins*" \) 2>/dev/null | head -10

echo "=== GitLab CI ==="
ls -la .gitlab-ci.yml .gitlab-ci.yaml 2>/dev/null
find . -maxdepth 2 -name ".gitlab-ci*.yml" 2>/dev/null

echo "=== CircleCI ==="
ls -la .circleci/config.yml .circleci/config.yaml 2>/dev/null

echo "=== Travis CI ==="
ls -la .travis.yml .travis.yaml 2>/dev/null

echo "=== Azure Pipelines ==="
find . -maxdepth 2 \( -name "azure-pipelines*.yml" -o -name ".azure-pipelines*" \) 2>/dev/null

echo "=== Bitbucket Pipelines ==="
ls -la bitbucket-pipelines.yml 2>/dev/null

echo "=== Drone CI ==="
ls -la .drone.yml .drone.yaml 2>/dev/null

echo "=== Buildkite ==="
find .buildkite -name "*.yml" -o -name "*.yaml" 2>/dev/null

echo "=== Tekton ==="
find . -maxdepth 3 -path "*tekton*" -name "*.yaml" 2>/dev/null | head -10

echo "=== TeamCity ==="
find .teamcity -name "*.kts" -o -name "*.xml" 2>/dev/null

echo "=== Bamboo ==="
find bamboo-specs -name "*.yaml" -o -name "*.java" 2>/dev/null

echo "=== GoCD ==="
ls -la .gocd.yml .gocd.yaml 2>/dev/null

echo "=== Harness ==="
find . -maxdepth 3 -path "*harness*" -name "*.yaml" 2>/dev/null | head -10

echo "=== Codefresh ==="
ls -la codefresh.yml codefresh.yaml 2>/dev/null

echo "=== Woodpecker ==="
ls -la .woodpecker.yml .woodpecker/ 2>/dev/null

echo "=== Semaphore ==="
ls -la .semaphore/ .semaphore.yml 2>/dev/null

echo "=== Buddy ==="
ls -la buddy.yml .buddy/ 2>/dev/null
```

### 2.3 External CI Inference (When No Config Files Found)

**Critical**: Many enterprise CI systems store configuration on external servers. When no config files are found, infer from artifacts:

```bash
echo "=========================================="
echo "EXTERNAL CI INFERENCE"
echo "=========================================="

echo "=== CI-Specific Scripts ==="
find . -maxdepth 3 -type f \( \
  -name "*jenkins*" -o -name "*bamboo*" -o -name "*teamcity*" -o \
  -name "*ci-*" -o -name "*-ci.*" -o -name "*-ci-*" -o \
  -name "build.*" -o -name "deploy.*" -o -name "release.*" -o \
  -name "pipeline.*" \
\) 2>/dev/null | grep -v node_modules | head -20

echo "=== Makefile CI Targets ==="
grep -E "^(jenkins|bamboo|teamcity|ci|build|deploy|release|test)[-_]?" Makefile 2>/dev/null | head -15

echo "=== Environment Variable References ==="
grep -rh --include="*.sh" --include="*.py" --include="*.js" --include="*.ts" --include="Makefile" \
  -E '\$\{?(BUILD_NUMBER|BUILD_ID|JENKINS_|BAMBOO_|TEAMCITY_|CI_|DRONE_|BUILDKITE_|GITHUB_|GITLAB_|TRAVIS_|CIRCLE_|BITBUCKET_)' . 2>/dev/null | head -30

echo "=== CI Badges in README ==="
grep -E "(jenkins|bamboo|teamcity|travis|circleci|gitlab|drone|buildkite|azure|bitbucket).*(badge|status|build)" README.md 2>/dev/null
grep -E "!\[.*(build|ci|pipeline|status).*\]\(http" README.md 2>/dev/null

echo "=== Docker Image References ==="
grep -rh --include="Dockerfile*" --include="*.yml" --include="*.yaml" \
  -E "(jenkins|bamboo|teamcity|concourse|drone|buildkite|circleci)" . 2>/dev/null | head -10

echo "=== CI Mentions in Documentation ==="
grep -rih --include="*.md" --include="*.rst" --include="*.txt" \
  -E "(jenkins|bamboo|teamcity|concourse|travis|circleci|gitlab.ci|azure.pipelines|drone|buildkite)" docs/ documentation/ 2>/dev/null | head -15
```

### 2.4 Detection Classification

After running all detection commands, classify the result:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CI DETECTION CLASSIFICATION                          │
├──────────────────┬──────────────────────────────────────────────────────────┤
│ CONFIRMED        │ Configuration file found in repository                   │
│                  │ Action: Parse and migrate directly                       │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ INFERRED         │ Strong artifact signals (env vars, scripts, badges)      │
│                  │ Action: Present findings, confirm if --interactive       │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ SUSPECTED        │ Weak signals only (mentions in docs, naming patterns)    │
│                  │ Action: Note in report, create generic workflows         │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ NONE             │ No CI indicators found                                   │
│                  │ Action: Create fresh GitHub Actions from scratch         │
└──────────────────┴──────────────────────────────────────────────────────────┘
```

</phase_2_parallel_discovery>

<phase_2_success_criteria>

### Phase 2 Completion Checklist

Before proceeding to Phase 3:

- [ ] All CI config file locations searched
- [ ] External CI indicators documented with evidence
- [ ] Detection classification assigned (CONFIRMED/INFERRED/SUSPECTED/NONE)
- [ ] Source CI system identified or explicitly marked unknown
- [ ] All `((variable))` or `${SECRET}` patterns catalogued for secrets migration
- [ ] If INFERRED, evidence citations include file:line references

**STOP**: If detection is INFERRED and `--interactive` flag set, confirm with user before proceeding.
</phase_2_success_criteria>

---

## Phase 3: CI-Specific Analysis & Translation (ultrathink)

**Goal**: Deep analysis of source CI configuration and systematic translation to GitHub Actions.

<ci_translation_reference>

### Translation Reference Tables

Based on detected CI system, apply the appropriate translation:

<concourse_translation>

#### Concourse CI → GitHub Actions

| Concourse | GitHub Actions | Notes |
|-----------|----------------|-------|
| `resources:` with `type: git` | `actions/checkout@v5` | |
| `resources:` with `type: docker-image` | `docker/build-push-action@v6` | |
| `resources:` with `type: s3` | `aws-actions/configure-aws-credentials@v4` + `aws s3` | |
| `resources:` with `type: slack-notification` | `slackapi/slack-github-action@v2` | |
| `resource_types:` | Custom actions or marketplace equivalents | Research required |
| `jobs:` → `plan:` | `jobs:` → `steps:` | |
| `get:` with `trigger: true` | `on: push` / `on: pull_request` | |
| `get:` with `passed: [job-a]` | `needs: [job-a]` | |
| `put:` | Upload/deploy actions | |
| `task:` | `run:` or action invocation | |
| `in_parallel:` | Parallel jobs or `strategy.matrix` | |
| `serial: true` | `concurrency: { group: x, cancel-in-progress: false }` | |
| `serial_groups: [deploy]` | `concurrency: { group: deploy }` | |
| `on_failure:` | `if: failure()` | |
| `on_success:` | `if: success()` | |
| `ensure:` | `if: always()` | |
| `across:` | `strategy.matrix` | |
| `((variable))` | `${{ secrets.VARIABLE }}` or `${{ vars.VARIABLE }}` | Document for migration |
| `set_pipeline:` | Workflow dispatch or reusable workflows | |

**Vault/CredHub Secrets**: Document ALL `((var))` references for manual migration to GitHub Secrets.
</concourse_translation>

<jenkins_translation>

#### Jenkins → GitHub Actions

| Jenkins | GitHub Actions | Notes |
|---------|----------------|-------|
| `pipeline { }` | `jobs:` | |
| `agent any` | `runs-on: ubuntu-latest` | |
| `agent { docker { } }` | Container jobs or setup actions | |
| `stages { stage('X') { } }` | `jobs:` with `needs:` | |
| `steps { sh 'cmd' }` | `steps:` → `- run: cmd` | |
| `when { branch 'main' }` | `on: push: branches: [main]` | |
| `post { always { } }` | `if: always()` | |
| `post { failure { } }` | `if: failure()` | |
| `parallel { }` | Multiple jobs or matrix | |
| `environment { VAR = 'x' }` | `env:` block | |
| `parameters { }` | `workflow_dispatch: inputs:` | |
| `credentials('id')` | `${{ secrets.ID }}` | Document for migration |
| `stash/unstash` | `actions/upload-artifact@v4` / `actions/download-artifact@v4` | |
| `archiveArtifacts` | `actions/upload-artifact@v4` | |
| `publishHTML` | GitHub Pages action or artifact | |
| `emailext` | Custom notification action | |
| Shared libraries | Reusable workflows with `workflow_call` | Complex migration |

**External Jenkins Inference Signals**:

- `Jenkinsfile` present but minimal (calls shared library)
- Scripts referencing `$BUILD_NUMBER`, `$JENKINS_URL`, `$JOB_NAME`
- Makefile with `jenkins-*` targets
- README mentions Jenkins URL
</jenkins_translation>

<gitlab_translation>

#### GitLab CI → GitHub Actions

| GitLab CI | GitHub Actions | Notes |
|-----------|----------------|-------|
| `stages:` | Job dependencies via `needs:` | |
| `image:` | `container:` or setup action | |
| `script:` | `steps:` → `- run:` | |
| `before_script:` | Early steps in job | |
| `after_script:` | Steps with `if: always()` | |
| `only: / except:` | `on:` with path/branch filters | |
| `rules:` | `if:` conditions on steps/jobs | |
| `artifacts:` | `actions/upload-artifact@v4` | |
| `cache:` | `actions/cache@v4` | |
| `dependencies:` | `actions/download-artifact` + `needs:` | |
| `needs:` | `needs:` | Direct mapping |
| `parallel:` | `strategy.matrix` | |
| `extends:` | Reusable workflows or YAML anchors | |
| `include:` | Reusable workflows | |
| `variables:` | `env:` or `${{ vars.* }}` | |
| `$CI_*` variables | `${{ github.* }}` context | See variable mapping |
| `environment:` | `environment:` | |
| `services:` | `services:` in job | |
| `trigger:` | `workflow_dispatch` or `repository_dispatch` | |
</gitlab_translation>

<circleci_translation>

#### CircleCI → GitHub Actions

| CircleCI | GitHub Actions | Notes |
|----------|----------------|-------|
| `version: 2.1` | N/A | |
| `orbs:` | Marketplace actions | Research equivalent |
| `executors:` | `runs-on:` + setup steps | |
| `jobs:` | `jobs:` | |
| `workflows:` | Multiple workflows or job dependencies | |
| `steps:` | `steps:` | |
| `run:` | `- run:` | |
| `checkout` | `actions/checkout@v5` | |
| `save_cache / restore_cache` | `actions/cache@v4` | |
| `persist_to_workspace / attach_workspace` | `upload/download-artifact` | |
| `store_artifacts` | `actions/upload-artifact@v4` | |
| `store_test_results` | Test reporter actions | |
| `when: on_fail` | `if: failure()` | |
| `filters: branches:` | `on: push: branches:` | |
| `requires:` | `needs:` | |
| `context:` | GitHub Environments + secrets | |
| `parameters:` | `workflow_dispatch: inputs:` | |
| `matrix:` | `strategy.matrix` | |
</circleci_translation>

<azure_translation>

#### Azure Pipelines → GitHub Actions

| Azure Pipelines | GitHub Actions | Notes |
|-----------------|----------------|-------|
| `trigger:` | `on: push:` | |
| `pr:` | `on: pull_request:` | |
| `pool:` | `runs-on:` | |
| `stages:` | Jobs with dependencies | |
| `jobs:` | `jobs:` | |
| `steps:` | `steps:` | |
| `script:` | `- run:` | |
| `bash:` / `pwsh:` | `- run:` with `shell:` | |
| `task:` | Equivalent marketplace action | Research required |
| `variables:` | `env:` or `${{ vars.* }}` | |
| `parameters:` | `workflow_dispatch: inputs:` | |
| `dependsOn:` | `needs:` | |
| `condition:` | `if:` | |
| `strategy: matrix:` | `strategy.matrix` | |
| `template:` | Reusable workflows | |
| `resources: repositories:` | `actions/checkout` with `repository:` | |
| `$(Build.*)` variables | `${{ github.* }}` context | See variable mapping |
| Service connections | GitHub Secrets + OIDC | |
</azure_translation>

<bitbucket_translation>

#### Bitbucket Pipelines → GitHub Actions

| Bitbucket Pipelines | GitHub Actions | Notes |
|---------------------|----------------|-------|
| `image:` | `container:` or setup action | |
| `pipelines:` | `on:` triggers + `jobs:` | |
| `default:` | `on: push:` | |
| `branches:` | `on: push: branches:` | |
| `pull-requests:` | `on: pull_request:` | |
| `tags:` | `on: push: tags:` | |
| `step:` | `steps:` entries | |
| `script:` | `- run:` | Multi-line with \| |
| `caches:` | `actions/cache@v4` | |
| `artifacts:` | `actions/upload-artifact@v4` | |
| `services:` | `services:` | |
| `parallel:` | Parallel jobs or matrix | |
| `stage:` | Jobs with `needs:` | |
| `deployment:` | `environment:` | |
| `$BITBUCKET_*` variables | `${{ github.* }}` context | See variable mapping |
</bitbucket_translation>

<other_ci_translations>

#### Additional CI Systems

For less common CI systems (Drone, Buildkite, TeamCity, Bamboo, Tekton, GoCD, Harness), apply these general principles:

1. **Jobs/Stages → GitHub Jobs** with `needs:` for dependencies
2. **Environment variables → `env:` block** or `${{ secrets.* }}`
3. **Caching → `actions/cache@v4`**
4. **Artifacts → `actions/upload-artifact@v4` / `actions/download-artifact@v4`**
5. **Parallel execution → `strategy.matrix`** or parallel jobs
6. **Conditions → `if:` expressions**
7. **Services → `services:` block** in job
8. **Secrets → Document for manual migration**

**Note**: For TeamCity and Bamboo (often external), full pipeline logic may not be discoverable. Generate reasonable defaults from artifact analysis.
</other_ci_translations>

</ci_translation_reference>

<phase_3_analysis>

### 3.1 Deep CI Configuration Analysis

For each detected CI configuration file, perform detailed analysis:

1. **Read the ENTIRE configuration file** - not just sections
2. **Document every job/stage** with its purpose
3. **Identify all triggers** (push, PR, schedule, manual)
4. **Map all dependencies** between jobs
5. **Catalog all secrets/credentials** referenced
6. **Note all custom scripts** called
7. **Identify caching strategies**
8. **Document artifact handling**
9. **Find notification integrations**
10. **Identify environment-specific configurations**

### 3.2 Translation Validation

For each translated workflow:

```yaml
# VALIDATION CHECKLIST
# - [ ] All jobs from source are represented
# - [ ] Job dependencies (needs:) match source ordering
# - [ ] Triggers match source triggers
# - [ ] Secrets are documented (not hardcoded!)
# - [ ] Caching strategy preserved
# - [ ] Artifacts uploaded where needed
# - [ ] Error handling (if: failure()) preserved
# - [ ] Environment variables mapped correctly
# - [ ] Shell commands are platform-appropriate
# - [ ] Action versions are current (v4/v5 preferred)
```

</phase_3_analysis>

<phase_3_success_criteria>

### Phase 3 Completion Checklist

Before proceeding to Phase 4:

- [ ] All source CI configuration files READ completely
- [ ] Translation mapping documented for each concept
- [ ] All secrets catalogued in SECRETS_INVENTORY section
- [ ] All custom scripts identified and analyzed
- [ ] Workflow YAML validated for syntax
- [ ] Semantic equivalence verified (same triggers, same flow)

**STOP**: If source CI cannot be fully parsed, document gaps and generate best-effort translation with explicit limitations noted.
</phase_3_success_criteria>

---

## Phase 4: GitHub Actions Generation

**Goal**: Generate production-ready GitHub Actions workflows.

<workflow_templates>

### 4.1 Core Workflow Structure

All generated workflows MUST follow this structure:

```yaml
# FILE: .github/workflows/{name}.yml
name: {Descriptive Name}

on:
  # Triggers matching source CI
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master]
  # Include schedule if source had scheduled builds
  # Include workflow_dispatch for manual runs

# Prevent concurrent runs (if source had serial enforcement)
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Global environment variables
env:
  # Map from source CI variables
  VARIABLE_NAME: value

jobs:
  # Jobs translated from source CI
  job-name:
    name: Job Display Name
    runs-on: ubuntu-latest

    # Service containers if needed
    services:
      # Map from source CI services

    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          fetch-depth: 0  # Full history for versioning

      # Language setup
      - name: Setup {Language}
        uses: actions/setup-{language}@v{latest}
        with:
          # Version from source or manifest

      # Caching
      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: # Language-specific cache paths
          key: ${{ runner.os }}-{tool}-${{ hashFiles('**/lockfile') }}
          restore-keys: |
            ${{ runner.os }}-{tool}-

      # Build/test steps translated from source
      - name: {Step Name}
        run: |
          # Commands from source CI
        env:
          # Step-specific environment
```

### 4.2 Standard Workflow Templates

<ci_workflow_template>

#### ci.yml - Primary CI Workflow

```yaml
name: CI

on:
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master]

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      # Language-specific linting

  test:
    name: Test
    runs-on: ubuntu-latest
    needs: [lint]  # If source had this dependency
    steps:
      - uses: actions/checkout@v5
      # Language-specific testing

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test]
    steps:
      - uses: actions/checkout@v5
      # Language-specific build
```

</ci_workflow_template>

<release_workflow_template>

#### release.yml - Release Workflow

Only create if source CI had deployment/release stages:

```yaml
name: Release

on:
  push:
    tags: ['v*']
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to release'
        required: true
        type: string

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    permissions:
      contents: write
      packages: write
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0

      # Build and publish steps from source CI
```

</release_workflow_template>

<security_workflow_template>

#### security.yml - Security Scanning

Always create unless `--skip-security` flag:

```yaml
name: Security

on:
  push:
    branches: [main, master]
  pull_request:
  schedule:
    - cron: '0 0 * * 1'  # Weekly on Monday

permissions:
  security-events: write
  contents: read

jobs:
  codeql:
    name: CodeQL Analysis
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v5

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
    strategy:
      fail-fast: false
      matrix:
        language: ['{detected-language}']  # javascript, python, go, java, etc.

  dependency-review:
    name: Dependency Review
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v5
      - uses: actions/dependency-review-action@v4
```

</security_workflow_template>

</workflow_templates>

<phase_4_success_criteria>

### Phase 4 Completion Checklist

Before proceeding to Phase 5:

- [ ] All workflows have valid YAML syntax
- [ ] All workflows use current action versions (v4/v5)
- [ ] All secrets use `${{ secrets.* }}` syntax (not hardcoded)
- [ ] All workflows have appropriate `permissions:` blocks
- [ ] Concurrency groups prevent redundant runs
- [ ] Job dependencies match source CI flow
- [ ] Language/framework setup matches detected stack
- [ ] Caching is configured for build dependencies

**YAML Validation**: Run syntax check before writing:

```bash
# Validate YAML (if yq available)
yq eval '.' workflow.yml > /dev/null 2>&1 && echo "Valid YAML" || echo "Invalid YAML"
```

</phase_4_success_criteria>

---

## Phase 5: GitHub Copilot Configuration

**Goal**: Configure GitHub Copilot for repository-specific context.

<copilot_configuration>

### 5.1 Create copilot-instructions.md

Generate `.github/copilot-instructions.md`:

```markdown
# {Project Name}

{Description extracted from README}

## Tech Stack
- **Language**: {Primary language and version}
- **Framework**: {Framework(s) if applicable}
- **Build Tool**: {Build system}
- **CI/CD**: GitHub Actions (migrated from {source CI})

## Coding Guidelines
{Infer from linter configs, .editorconfig, existing patterns}

## Project Structure
{Key directories from Phase 1 exploration}

## Development Commands
- **Install**: {detected command}
- **Build**: {detected command}
- **Test**: {detected command}
- **Lint**: {detected command}

## Architecture Notes
{Any significant patterns observed}

## CI/CD Migration Notes
- Migrated from: {source CI system}
- Migration date: {current date}
- Secrets to configure: {list from inventory}
```

### 5.2 Create Copilot Setup Steps

Generate `.github/workflows/copilot-setup-steps.yml`:

```yaml
name: "Copilot Setup Steps"

on: workflow_dispatch

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v5

      # Language-specific setup steps based on detection
      - name: Setup {Language}
        uses: actions/setup-{language}@v{latest}
        with:
          {language}-version: '{version}'

      - name: Install dependencies
        run: |
          # Detected install command

      - name: Verify setup
        run: |
          # Detected verification command
```

</copilot_configuration>

---

## Phase 6: Templates & Infrastructure

**Goal**: Establish GitHub-native collaboration infrastructure.

<templates_infrastructure>

### 6.1 Issue Templates

Create `.github/ISSUE_TEMPLATE/bug.yml`:

```yaml
name: Bug Report
description: Report a bug or unexpected behavior
labels: ["bug", "triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for reporting a bug. Please fill out the information below.

  - type: textarea
    id: description
    attributes:
      label: Description
      description: Clear description of the bug
      placeholder: What happened?
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: Steps to Reproduce
      description: Steps to reproduce the behavior
      placeholder: |
        1. Go to '...'
        2. Click on '...'
        3. See error
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      description: What did you expect to happen?
    validations:
      required: true

  - type: input
    id: version
    attributes:
      label: Version
      description: What version are you using?
      placeholder: v1.0.0

  - type: dropdown
    id: environment
    attributes:
      label: Environment
      options:
        - Production
        - Staging
        - Development
        - Local
```

Create `.github/ISSUE_TEMPLATE/feature.yml`:

```yaml
name: Feature Request
description: Suggest a new feature or enhancement
labels: ["enhancement", "triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for your feature suggestion!

  - type: textarea
    id: problem
    attributes:
      label: Problem Statement
      description: What problem does this feature solve?
      placeholder: I'm always frustrated when...
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: Proposed Solution
      description: Describe your proposed solution
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives Considered
      description: Any alternative solutions you've considered?
```

### 6.2 PR Template

Create `.github/pull_request_template.md`:

```markdown
## Summary

<!-- Brief description of the changes -->

## Type of Change

- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to change)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)

## Testing

<!-- How has this been tested? -->

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

## Checklist

- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Documentation updated (if applicable)
- [ ] No new warnings introduced
- [ ] Tests pass locally

## Related Issues

<!-- Link any related issues: Fixes #123, Relates to #456 -->
```

### 6.3 Dependabot Configuration

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      actions:
        patterns:
          - "*"

  # Language-specific (based on detection)
  # Add appropriate ecosystem based on Phase 1 detection
```

### 6.4 Pre-commit Configuration (Optional)

If pre-commit hooks detected or appropriate:

Create `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  # Language-specific hooks based on detection
```

</templates_infrastructure>

---

## Phase 7: Migration Summary & Reporting

**Goal**: Generate comprehensive migration documentation for enterprise tracking.

<migration_summary>

### 7.1 Create Migration Summary

Generate `.github/MIGRATION_SUMMARY.md`:

```markdown
# GitHub Ecosystem Migration Summary

**Migration Date**: {Current date}
**Repository**: {Repository name}
**Migration Tool**: Claude Code `/gm-ci-assist`

## Source Platform

| Attribute | Value |
|-----------|-------|
| **Platform** | {Bitbucket Server/Cloud, GitHub Enterprise, etc.} |
| **CI System** | {Detected system} |
| **Detection Level** | {CONFIRMED / INFERRED / SUSPECTED / NONE} |
| **Config Location** | {Path or "External (server-side)"} |

## Evidence Documentation

{For INFERRED/SUSPECTED detection, document all evidence}

| Signal Type | Evidence Found |
|-------------|----------------|
| Environment variables | {e.g., `$JENKINS_URL` in deploy.sh:42} |
| Script naming | {e.g., `jenkins-build.sh`} |
| Documentation | {e.g., README mentions "Jenkins pipeline"} |
| Badges | {e.g., Jenkins badge URL} |
| Makefile targets | {e.g., `make bamboo-deploy`} |

## Migration Artifacts Created

### Workflows

| Workflow | Source | Purpose |
|----------|--------|---------|
| ci.yml | {Source pipeline/stage} | Main CI pipeline |
| release.yml | {Source} | Release automation |
| security.yml | New | Security scanning (CodeQL, dependency review) |

### Copilot Integration

- [x] `.github/copilot-instructions.md`
- [x] `.github/workflows/copilot-setup-steps.yml`

### Templates

- [x] Bug report template (`.github/ISSUE_TEMPLATE/bug.yml`)
- [x] Feature request template (`.github/ISSUE_TEMPLATE/feature.yml`)
- [x] Pull request template (`.github/pull_request_template.md`)

### Configuration

- [x] Dependabot configuration (`.github/dependabot.yml`)
- [ ] Pre-commit hooks (if applicable)

## Secrets Migration Required

| GitHub Secret | Source System | Original Reference | Description |
|---------------|---------------|---------------------|-------------|
{Table of all secrets found during analysis}

## Concept Translation Applied

| Original Concept | GitHub Actions Equivalent |
|------------------|---------------------------|
{Table of key translations applied}

## Post-Migration Checklist

### Immediate Actions (Before First Push)

- [ ] Add all secrets to GitHub repository settings (Settings → Secrets and variables → Actions)
- [ ] Review generated workflow YAML for accuracy
- [ ] Verify triggers match expected behavior
- [ ] Test workflows on a feature branch first

### Configuration Actions

- [ ] Enable GitHub Copilot for repository (if licensed)
- [ ] Enable Copilot code review in rulesets (Settings → Rules → Rulesets)
- [ ] Configure branch protection rules
- [ ] Set up required status checks

### Validation Actions

- [ ] Verify CI workflow passes on test branch
- [ ] Verify security workflow runs successfully
- [ ] Test pull request workflow trigger
- [ ] Validate release workflow (if applicable)

### Source CI Decommissioning

- [ ] Export pipeline configuration for archival
- [ ] Disable webhooks from source CI to this repository
- [ ] Remove or archive job/pipeline definitions
- [ ] Revoke service account credentials
- [ ] Update deployment targets expecting old CI
- [ ] Update README badges to GitHub Actions

## Enterprise Cloud Readiness

- [ ] All CI/CD runs on GitHub Actions
- [ ] No external CI dependencies
- [ ] Secrets documented and ready for migration
- [ ] Copilot configured
- [ ] Branch protection configured
- [ ] Templates in place
```

### 7.2 Batch Processing Report (if --batch-report)

Generate `migration-report.json` for batch processing:

```json
{
  "repository": "{repo-name}",
  "migration_date": "{ISO-8601 date}",
  "source_platform": "{platform}",
  "source_ci": "{ci-system}",
  "detection_level": "{CONFIRMED|INFERRED|SUSPECTED|NONE}",
  "artifacts_created": [
    ".github/workflows/ci.yml",
    ".github/workflows/security.yml",
    ".github/copilot-instructions.md"
  ],
  "secrets_required": [
    {"name": "SECRET_NAME", "source": "original-reference"}
  ],
  "status": "success",
  "warnings": [],
  "errors": []
}
```

</migration_summary>

</execution_protocol>

---

<quality_gates>

## Quality Gates & Anti-Hallucination Enforcement

### Pre-Execution Verification

Before generating ANY artifacts, verify:

- [ ] Have I READ all relevant CI configuration files?
- [ ] Can I cite specific file:line for every claim about the CI system?
- [ ] Have I avoided guessing based on file names alone?
- [ ] Have I catalogued ALL secrets/credentials referenced?
- [ ] Have I validated YAML syntax before writing?

### Post-Generation Verification

After generating artifacts:

- [ ] Every workflow has valid YAML syntax
- [ ] Every secret uses `${{ secrets.* }}` syntax
- [ ] Every action uses a specific version (not `@main` or `@latest`)
- [ ] Every job has appropriate `runs-on:` specified
- [ ] No hardcoded credentials or tokens
- [ ] Migration summary documents all sources of truth

### Prohibited Behaviors

NEVER:

- Generate workflows without reading source CI config first
- Guess CI system based solely on file/folder names
- Hardcode secrets or credentials in workflows
- Use placeholder values in generated YAML
- Claim successful migration without documenting limitations
- Overwrite existing workflows without `--preserve-existing=false`

### When Uncertain

If you cannot determine something with confidence:

1. **Document the uncertainty** in MIGRATION_SUMMARY.md
2. **Generate conservative defaults** with comments explaining assumptions
3. **Mark as TODO** for human review
4. **Do NOT guess** and present as fact
</quality_gates>

---

<execution_instruction>

## Execution Sequence

### For Single Repository Migration

1. **Phase 1**: Complete discovery, present findings, confirm scope
2. **Phase 2**: Run CI detection, classify result, document evidence
3. **Phase 3**: Analyze source CI, apply translation tables
4. **Phase 4**: Generate workflows, validate YAML
5. **Phase 5**: Create Copilot configuration
6. **Phase 6**: Create templates and infrastructure
7. **Phase 7**: Generate migration summary

### Flag Behaviors

- `--dry-run`: Generate comprehensive MIGRATION_REPORT.md with all analysis and proposed files embedded; only this report file is written
- `--interactive`: Confirm at each phase before proceeding
- `--copilot-only`: Skip Phases 2-4, only do Phase 5
- `--batch-report`: Also generate `migration-report.json`
- `--skip-security`: Skip security.yml generation
- `--preserve-existing`: Check before overwriting any existing .github/ files

### Handling External CI Limitations

When CI configuration lives on external server (Jenkins, TeamCity, Bamboo):

1. **Acknowledge the limitation**: State that full pipeline cannot be reverse-engineered
2. **Use available signals**: Build workflows from scripts, Makefile targets, documentation
3. **If `--interactive`**: Ask user about build steps, test commands, deployment targets, secrets
4. **Generate reasonable defaults**: Based on language detection and common patterns
5. **Document gaps**: Note in MIGRATION_SUMMARY what couldn't be determined

---

## First Response

Begin with Phase 1 discovery:

1. Run repository structure discovery commands
2. Detect source platform from git remotes
3. Identify language, framework, and build tools
4. Check for existing `.github/` directory
5. Present findings and confirm scope before proceeding to CI detection

If `--dry-run`, state: "Dry run mode enabled. I will generate MIGRATION_REPORT.md with full analysis and proposed files. No other files will be created."

Start with: "Let me analyze this repository to understand its structure and prepare for GitHub ecosystem migration..."
</execution_instruction>
