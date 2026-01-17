---
allowed-tools: Bash(*), Read, Write, Edit, Glob, Grep, LS, WebFetch
argument-hint: [--ci=TYPE] [--copilot-only] [--dry-run] [--interactive]
description: Onboard repository to GitHub ecosystem with Copilot integration. Auto-detects and migrates CI/CD from Concourse, Jenkins, GitLab CI, CircleCI, Travis, Azure Pipelines, Bamboo, TeamCity, Drone, Buildkite, or infers external CI from artifacts. Creates GitHub Actions, Copilot config, issue/PR templates. Run in any repo for enterprise cloud migration.
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
MIGRATE(1)                      User Commands                      MIGRATE(1)

NAME
    migrate - Onboard repository to GitHub ecosystem with CI migration

SYNOPSIS
    /gh:migrate [--ci=TYPE] [--copilot-only] [--dry-run] [--interactive]

DESCRIPTION
    Onboard repository to GitHub ecosystem with Copilot integration.
    Auto-detects and migrates CI/CD from: Concourse, Jenkins, GitLab CI,
    CircleCI, Travis, Azure Pipelines, Bamboo, TeamCity, Drone, Buildkite.
    Creates GitHub Actions workflows, Copilot config, issue/PR templates.

OPTIONS
    --ci=TYPE                 Force CI type (concourse|jenkins|gitlab|...)
    --copilot-only            Only configure Copilot, skip CI migration
    --dry-run                 Generate plan without creating files
    --interactive             Prompt for confirmation at each phase
    --help, -h                Show this help message

EXAMPLES
    /gh:migrate                       Auto-detect and migrate CI
    /gh:migrate --dry-run             Preview without changes
    /gh:migrate --copilot-only        Only configure Copilot
    /gh:migrate --ci=jenkins          Force Jenkins migration
    /gh:migrate --interactive         Step-by-step with confirmations

SEE ALSO
    /gh:ci-assist         Extended CI migration with batch support
    /gh:copilot-onboard   Copilot-only configuration

                                                                  MIGRATE(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

# GitHub Ecosystem Onboarding & Multi-CI Migration

You are performing comprehensive onboarding of this repository into the GitHub ecosystem. This command:

1. Deeply explores the codebase to understand its structure
2. Detects CI/CD systems (in-repo or inferred from artifacts)
3. Translates existing pipelines to GitHub Actions
4. Configures GitHub Copilot with repository-specific context
5. Establishes testing infrastructure and quality gates
6. Creates templates optimized for AI-assisted development
7. Generates migration summary for enterprise cloud submission

## Arguments

Parse `$ARGUMENTS` for flags:
- `--ci=TYPE`: Force specific CI type (concourse|jenkins|gitlab|circle|travis|azure|bamboo|teamcity|drone|buildkite|tekton|gocd|harness|bitbucket)
- `--copilot-only`: Only configure Copilot, skip CI migration
- `--dry-run`: Generate plan without creating files
- `--interactive`: Prompt for confirmation at each phase

---

<phase number="1" name="Codebase Exploration">
# PHASE 1: CODEBASE EXPLORATION

**Goal**: Build comprehensive understanding before making any changes.

## 1.1 Repository Structure Discovery

```bash
pwd && ls -la
find . -maxdepth 3 -type f \( -name "*.yml" -o -name "*.yaml" -o -name "*.json" -o -name "Makefile" -o -name "Dockerfile*" -o -name "*.toml" -o -name "*.cfg" -o -name "*.groovy" -o -name "*.kts" \) 2>/dev/null | grep -v node_modules | grep -v vendor | head -60
```

## 1.2 Language & Framework Detection

Examine these files to determine primary stack:

| File | Indicates |
|------|-----------|
| `package.json` | Node.js/JavaScript/TypeScript |
| `pyproject.toml`, `setup.py`, `requirements*.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml`, `build.gradle`, `build.gradle.kts` | Java/Kotlin |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `*.csproj`, `*.sln` | .NET |
| `mix.exs` | Elixir |
| `pubspec.yaml` | Dart/Flutter |

## 1.3 Documentation Review

Read for project context and CI references:
- README.md (especially badges and "How to build" sections)
- CONTRIBUTING.md
- docs/ or documentation/
- CLAUDE.md (if exists)
</phase>

---

<phase number="2" name="CI/CD Detection">
# PHASE 2: CI/CD DETECTION

## 2.1 In-Repository CI Detection

Search for CI configuration files in priority order:

```bash
echo "=== GitHub Actions (existing) ===" && find .github/workflows -name "*.yml" -o -name "*.yaml" 2>/dev/null
echo "=== Concourse CI ===" && find . -type f \( -name "pipeline*.yml" -o -name "*pipeline*.yml" \) 2>/dev/null | grep -v node_modules
echo "=== Jenkins ===" && find . -name "Jenkinsfile*" -o -name "jenkins*.groovy" 2>/dev/null
echo "=== GitLab CI ===" && ls -la .gitlab-ci.yml 2>/dev/null
echo "=== CircleCI ===" && ls -la .circleci/config.yml 2>/dev/null
echo "=== Travis CI ===" && ls -la .travis.yml 2>/dev/null
echo "=== Azure Pipelines ===" && find . -name "azure-pipelines*.yml" -o -name ".azure-pipelines*" 2>/dev/null
echo "=== Bitbucket Pipelines ===" && ls -la bitbucket-pipelines.yml 2>/dev/null
echo "=== Drone CI ===" && ls -la .drone.yml 2>/dev/null
echo "=== Buildkite ===" && find .buildkite -name "*.yml" 2>/dev/null
echo "=== Tekton ===" && find . -path "*tekton*" -name "*.yaml" 2>/dev/null | head -10
echo "=== TeamCity ===" && find .teamcity -name "*.kts" -o -name "*.xml" 2>/dev/null
echo "=== Bamboo ===" && find bamboo-specs -name "*.yaml" -o -name "*.java" 2>/dev/null
echo "=== GoCD ===" && ls -la .gocd.yml 2>/dev/null
echo "=== Harness ===" && find . -path "*harness*" -name "*.yaml" 2>/dev/null
echo "=== Codefresh ===" && ls -la codefresh.yml 2>/dev/null
echo "=== Woodpecker ===" && ls -la .woodpecker.yml .woodpecker/ 2>/dev/null
```

## 2.2 External CI Inference

**Many CI systems store configuration externally.** When no config files found, infer from artifacts:

### 2.2.1 Script Pattern Analysis

```bash
echo "=== CI-specific scripts ==="
find . -type f \( -name "*jenkins*" -o -name "*bamboo*" -o -name "*teamcity*" -o -name "*ci-*" -o -name "*-ci.*" -o -name "build.*" -o -name "deploy.*" \) 2>/dev/null | grep -v node_modules | head -20

echo "=== Makefile CI targets ==="
grep -E "^(jenkins|bamboo|teamcity|ci|build|deploy|release)[-_]?" Makefile 2>/dev/null | head -10
```

### 2.2.2 Environment Variable Patterns

```bash
echo "=== CI env var references ==="
grep -rh --include="*.sh" --include="*.py" --include="*.js" --include="Makefile" -E '\$\{?(BUILD_NUMBER|BUILD_ID|JENKINS_|BAMBOO_|TEAMCITY_|CI_|DRONE_|BUILDKITE_|GITHUB_|GITLAB_|TRAVIS_|CIRCLE_)' . 2>/dev/null | head -20
```

### 2.2.3 README Badge Analysis

```bash
echo "=== CI badges in README ==="
grep -E "(jenkins|bamboo|teamcity|travis|circleci|gitlab|drone|buildkite|azure|bitbucket).*(badge|status|build)" README.md 2>/dev/null
grep -E "!\[.*(build|ci|pipeline|status).*\]" README.md 2>/dev/null
```

### 2.2.4 Docker Image References

```bash
echo "=== CI-related Docker images ==="
grep -rh --include="Dockerfile*" --include="*.yml" --include="*.yaml" -E "(jenkins|bamboo|teamcity|concourse|drone|buildkite)" . 2>/dev/null | head -10
```

### 2.2.5 Documentation References

```bash
echo "=== CI mentions in docs ==="
grep -rih --include="*.md" -E "(jenkins|bamboo|teamcity|concourse|travis|circleci|gitlab.ci|azure.pipelines|drone|buildkite)" . 2>/dev/null | head -15
```

## 2.3 CI Detection Summary

<conditional trigger="CI detection complete">
After running detection, classify as one of:

| Detection Level | Meaning | Action |
|-----------------|---------|--------|
| **CONFIRMED** | Config file found in repo | Parse and migrate directly |
| **INFERRED** | Strong artifact signals | Present findings, ask for confirmation if `--interactive` |
| **SUSPECTED** | Weak signals | Note in migration report, create generic workflows |
| **NONE** | No CI detected | Create fresh GitHub Actions from scratch |
</conditional>
</phase>

---

<phase number="3" name="CI-Specific Analysis and Translation">
# PHASE 3: CI-SPECIFIC ANALYSIS & TRANSLATION

Based on detected CI system, apply the appropriate translation guide.

## 3.1 Concourse CI

**Config locations**: `pipeline.yml`, `ci/pipeline.yml`, `*-pipeline.yml`

| Concourse | GitHub Actions |
|-----------|----------------|
| `resources:` with `type: git` | `actions/checkout@v5` |
| `resources:` with `type: docker-image` | `docker/build-push-action@v6` |
| `resources:` with `type: s3` | `aws-actions/configure-aws-credentials@v4` + `aws s3` |
| `resources:` with `type: slack-notification` | `slackapi/slack-github-action@v2` |
| `resource_types:` | Custom actions or marketplace equivalents |
| `jobs:` -> `plan:` | `jobs:` -> `steps:` |
| `get:` with `trigger: true` | `on: push` / `on: pull_request` |
| `get:` with `passed: [job-a]` | `needs: [job-a]` |
| `put:` | Upload/deploy actions |
| `task:` | `run:` or action invocation |
| `in_parallel:` | Parallel jobs or `strategy.matrix` |
| `serial: true` | `concurrency: { group: x, cancel-in-progress: false }` |
| `serial_groups: [deploy]` | `concurrency: { group: deploy }` |
| `on_failure:` | `if: failure()` |
| `on_success:` | `if: success()` |
| `ensure:` | `if: always()` |
| `across:` | `strategy.matrix` |
| `((variable))` | `${{ secrets.VARIABLE }}` or `${{ vars.VARIABLE }}` |
| `set_pipeline:` | Workflow dispatch or reusable workflows |

**Vault/CredHub secrets**: Document all `((var))` references for manual migration to GitHub Secrets.

## 3.2 Jenkins

**Config locations**: `Jenkinsfile`, `jenkins/*.groovy`, or **external** (Jenkins server)

| Jenkins | GitHub Actions |
|---------|----------------|
| `pipeline { }` | `jobs:` |
| `agent any` / `agent { docker { } }` | `runs-on: ubuntu-latest` / container jobs |
| `stages { stage('X') { } }` | `jobs:` with `needs:` |
| `steps { sh 'cmd' }` | `steps:` -> `- run: cmd` |
| `when { branch 'main' }` | `on: push: branches: [main]` |
| `post { always { } }` | `if: always()` |
| `post { failure { } }` | `if: failure()` |
| `parallel { }` | Multiple jobs or matrix |
| `environment { VAR = 'x' }` | `env:` block |
| `parameters { }` | `workflow_dispatch: inputs:` |
| `credentials('id')` | `${{ secrets.ID }}` |
| `stash/unstash` | `actions/upload-artifact` / `actions/download-artifact` |
| `archiveArtifacts` | `actions/upload-artifact@v4` |
| `publishHTML` | GitHub Pages action or artifact |
| `emailext` | Custom notification action |
| Shared libraries | Reusable workflows (`.github/workflows/*.yml` with `workflow_call`) |

<conditional trigger="Jenkins detected but minimal Jenkinsfile">
**External Jenkins inference signals**:
- `Jenkinsfile` present but minimal (calls shared library)
- Scripts referencing `$BUILD_NUMBER`, `$JENKINS_URL`, `$JOB_NAME`
- Makefile with `jenkins-*` targets
- README mentions Jenkins URL
</conditional>

## 3.3 GitLab CI

**Config location**: `.gitlab-ci.yml`

| GitLab CI | GitHub Actions |
|-----------|----------------|
| `stages:` | Job dependencies via `needs:` |
| `image:` | `container:` or setup action |
| `script:` | `steps:` -> `- run:` |
| `before_script:` | Early steps in job |
| `after_script:` | Steps with `if: always()` |
| `only: / except:` | `on:` with path/branch filters |
| `rules:` | `if:` conditions on steps/jobs |
| `artifacts:` | `actions/upload-artifact@v4` |
| `cache:` | `actions/cache@v4` |
| `dependencies:` | `actions/download-artifact` + `needs:` |
| `needs:` | `needs:` |
| `parallel:` | `strategy.matrix` |
| `extends:` | Reusable workflows or YAML anchors |
| `include:` | Reusable workflows |
| `variables:` | `env:` or `${{ vars.* }}` |
| `$CI_*` variables | `${{ github.* }}` context |
| `environment:` | `environment:` |
| `services:` | `services:` in job |
| `trigger:` | `workflow_dispatch` or `repository_dispatch` |

## 3.4 CircleCI

**Config location**: `.circleci/config.yml`

| CircleCI | GitHub Actions |
|----------|----------------|
| `version: 2.1` | N/A |
| `orbs:` | Marketplace actions |
| `executors:` | `runs-on:` + setup steps |
| `jobs:` | `jobs:` |
| `workflows:` | Multiple workflows or job dependencies |
| `steps:` | `steps:` |
| `run:` | `- run:` |
| `checkout` | `actions/checkout@v5` |
| `save_cache / restore_cache` | `actions/cache@v4` |
| `persist_to_workspace / attach_workspace` | `actions/upload-artifact` / `actions/download-artifact` |
| `store_artifacts` | `actions/upload-artifact@v4` |
| `store_test_results` | Test reporter actions |
| `when: on_fail` | `if: failure()` |
| `filters: branches:` | `on: push: branches:` |
| `requires:` | `needs:` |
| `context:` | GitHub Environments + secrets |
| `parameters:` | `workflow_dispatch: inputs:` |
| `matrix:` | `strategy.matrix` |

## 3.5 Travis CI

**Config location**: `.travis.yml`

| Travis CI | GitHub Actions |
|-----------|----------------|
| `language:` | Setup action (e.g., `actions/setup-node@v4`) |
| `os:` / `dist:` | `runs-on:` |
| `env:` | `env:` |
| `matrix:` / `jobs:` | `strategy.matrix` |
| `before_install:` | Early steps |
| `install:` | Dependency installation steps |
| `before_script:` | Setup steps |
| `script:` | Main `- run:` steps |
| `after_success:` | `if: success()` steps |
| `after_failure:` | `if: failure()` steps |
| `deploy:` | Deployment steps/jobs |
| `cache:` | `actions/cache@v4` |
| `services:` | `services:` |
| `branches:` | `on: push: branches:` |
| `stages:` | Jobs with `needs:` |
| `$TRAVIS_*` variables | `${{ github.* }}` context |

## 3.6 Azure Pipelines

**Config location**: `azure-pipelines.yml`, `.azure-pipelines/*.yml`

| Azure Pipelines | GitHub Actions |
|-----------------|----------------|
| `trigger:` | `on: push:` |
| `pr:` | `on: pull_request:` |
| `pool:` | `runs-on:` |
| `stages:` | Jobs with dependencies |
| `jobs:` | `jobs:` |
| `steps:` | `steps:` |
| `script:` | `- run:` |
| `bash:` / `pwsh:` | `- run:` with `shell:` |
| `task:` | Equivalent marketplace action |
| `variables:` | `env:` or `${{ vars.* }}` |
| `parameters:` | `workflow_dispatch: inputs:` |
| `dependsOn:` | `needs:` |
| `condition:` | `if:` |
| `strategy: matrix:` | `strategy.matrix` |
| `template:` | Reusable workflows |
| `resources: repositories:` | `actions/checkout` with `repository:` |
| `$(Build.*)` variables | `${{ github.* }}` context |
| Service connections | GitHub Secrets + OIDC |

## 3.7 Drone CI

**Config location**: `.drone.yml`

| Drone | GitHub Actions |
|-------|----------------|
| `kind: pipeline` | Workflow file |
| `type: docker` | Container job or setup actions |
| `steps:` | `steps:` |
| `image:` | `container:` or setup action |
| `commands:` | `- run:` |
| `environment:` | `env:` |
| `when:` | `if:` conditions |
| `depends_on:` | `needs:` |
| `trigger:` | `on:` |
| `volumes:` | N/A (use actions/cache) |
| `services:` | `services:` |
| `secrets:` | `${{ secrets.* }}` |
| `from_secret:` | `${{ secrets.* }}` |

## 3.8 Buildkite

**Config location**: `.buildkite/pipeline.yml`

| Buildkite | GitHub Actions |
|-----------|----------------|
| `steps:` | `jobs:` -> `steps:` |
| `command:` | `- run:` |
| `plugins:` | Marketplace actions |
| `agents:` | `runs-on:` with labels |
| `env:` | `env:` |
| `depends_on:` | `needs:` |
| `if:` | `if:` |
| `parallelism:` | `strategy.matrix` |
| `wait` | Job dependencies with `needs:` |
| `block` | `environment:` with required reviewers |
| `trigger` | `workflow_dispatch` |
| `artifact_paths` | `actions/upload-artifact@v4` |
| `$BUILDKITE_*` variables | `${{ github.* }}` context |

## 3.9 TeamCity (Often External)

**Config location**: `.teamcity/*.kts` (Kotlin DSL) or **external server**

| TeamCity | GitHub Actions |
|----------|----------------|
| Build configurations | Workflow files |
| Build steps | `steps:` |
| VCS roots | `actions/checkout@v5` |
| Build features | Various actions |
| Dependencies | `needs:` |
| Parameters | `env:`, `inputs:`, `secrets` |
| Agent requirements | `runs-on:` |
| Artifact paths | `actions/upload-artifact@v4` |

<conditional trigger="TeamCity detected via external signals">
**External TeamCity inference signals**:
- Scripts referencing `$TEAMCITY_*` env vars
- `.teamcity/` directory with minimal config (references server)
- README/docs mentioning TeamCity server URL
- Build status badges pointing to TeamCity
</conditional>

## 3.10 Bamboo (Often External)

**Config location**: `bamboo-specs/` or **external server**

| Bamboo | GitHub Actions |
|--------|----------------|
| Plans | Workflow files |
| Stages | Jobs with `needs:` |
| Jobs | Jobs |
| Tasks | Steps |
| Artifacts | `actions/upload-artifact@v4` |
| Variables | `env:`, `secrets` |
| Triggers | `on:` events |
| Deployment projects | Deployment workflows |

<conditional trigger="Bamboo detected via external signals">
**External Bamboo inference signals**:
- Scripts referencing `$bamboo_*` or `$BAMBOO_*` variables
- Makefile targets like `bamboo-build`, `bamboo-deploy`
- README mentioning Bamboo
- `bamboo-specs/` with Java/YAML specs that reference plan keys
</conditional>

## 3.11 Tekton

**Config location**: `tekton/`, `.tekton/`, `**/tekton/*.yaml`

| Tekton | GitHub Actions |
|--------|----------------|
| `Pipeline` | Workflow file |
| `PipelineRun` | Workflow trigger |
| `Task` | Job or reusable workflow |
| `TaskRun` | Job execution |
| `steps:` in Task | `steps:` |
| `params:` | `inputs:` or `env:` |
| `workspaces:` | Artifacts or checkout |
| `results:` | Job outputs |
| `finally:` | `if: always()` steps |

## 3.12 GoCD

**Config location**: `.gocd.yml` or **external server**

| GoCD | GitHub Actions |
|------|----------------|
| Pipelines | Workflow files |
| Stages | Jobs with `needs:` |
| Jobs | Jobs (parallel by default in GoCD stages) |
| Tasks | Steps |
| Materials | `actions/checkout` or triggers |
| Artifacts | `actions/upload-artifact@v4` |
| Environment variables | `env:` |

## 3.13 Harness

**Config location**: `.harness/`, `harness/`

| Harness | GitHub Actions |
|---------|----------------|
| Pipelines | Workflow files |
| Stages | Jobs with dependencies |
| Steps | Steps |
| Services | Deployment targets |
| Environments | `environment:` |
| Connectors | Secrets + OIDC |
| Input sets | `workflow_dispatch: inputs:` |
</phase>

---

<phase number="4" name="GitHub Actions Generation">
# PHASE 4: GITHUB ACTIONS GENERATION

## 4.1 Workflow Structure

Create `.github/workflows/` with:

### ci.yml (Primary CI)

```yaml
name: CI

on:
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Translate from detected CI structure
```

### release.yml (If deployment detected)

```yaml
name: Release

on:
  push:
    tags: ['v*']
  workflow_dispatch:

# Translate deployment stages
```

### security.yml (Always create)

```yaml
name: Security

on:
  push:
    branches: [main, master]
  pull_request:
  schedule:
    - cron: '0 0 * * 1'  # Weekly

jobs:
  codeql:
    # If supported language detected

  dependency-review:
    # For PRs
```

## 4.2 Language-Specific Patterns

Apply appropriate setup and caching:

| Language | Setup Action | Cache |
|----------|--------------|-------|
| Node.js | `actions/setup-node@v4` | `cache: 'npm'` or `'pnpm'` or `'yarn'` |
| Python | `actions/setup-python@v5` | `cache: 'pip'` or `'pipenv'` or `'poetry'` |
| Go | `actions/setup-go@v5` | `cache: true` |
| Rust | `dtolnay/rust-toolchain@stable` | `Swatinem/rust-cache@v2` |
| Java | `actions/setup-java@v4` | `cache: 'maven'` or `'gradle'` |
| Ruby | `ruby/setup-ruby@v1` | `bundler-cache: true` |
| .NET | `actions/setup-dotnet@v4` | `actions/cache` for NuGet |
| PHP | `shivammathur/setup-php@v2` | `actions/cache` for Composer |
| Elixir | `erlef/setup-beam@v1` | `actions/cache` for mix |
</phase>

---

<phase number="5" name="GitHub Copilot Configuration">
# PHASE 5: GITHUB COPILOT CONFIGURATION

## 5.1 Create copilot-instructions.md

Create `.github/copilot-instructions.md`:

```markdown
# [Project Name]

[Description from README]

## Tech Stack
- [Primary language and version]
- [Framework(s)]
- [Build tool]
- CI/CD: GitHub Actions (migrated from [detected CI])

## Coding Guidelines
[Infer from linter configs, .editorconfig]

## Project Structure
[Key directories from exploration]

## Development Commands
- Build: [detected command]
- Test: [detected command]
- Lint: [detected command]

## CI/CD Notes
[Any migration-specific context]
```

## 5.2 Copilot Setup Steps

Create `.github/workflows/copilot-setup-steps.yml`:

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
      # Language-specific setup from detection
```
</phase>

---

<phase number="6" name="Templates and Infrastructure">
# PHASE 6: TEMPLATES & INFRASTRUCTURE

## 6.1 Issue Templates

Create `.github/ISSUE_TEMPLATE/bug.yml` and `.github/ISSUE_TEMPLATE/feature.yml` with YAML-based forms.

## 6.2 PR Template

Create `.github/pull_request_template.md` with sections for description, type, testing, and related issues.

## 6.3 Pre-commit Configuration

Create `.pre-commit-config.yaml` with language-appropriate hooks.

## 6.4 Dependabot

Create `.github/dependabot.yml` for detected package ecosystems.
</phase>

---

<phase number="7" name="Migration Summary">
# PHASE 7: MIGRATION SUMMARY

Create `.github/MIGRATION_SUMMARY.md`:

```markdown
# GitHub Ecosystem Migration Summary

**Date**: [Current date]
**Repository**: [Repo name]
**Tool**: Claude Code `/github-onboard`

## Source CI System

| Attribute | Value |
|-----------|-------|
| **CI System** | [Detected system] |
| **Detection** | [CONFIRMED / INFERRED / SUSPECTED / NONE] |
| **Config Location** | [Path or "External (server-side)"] |
| **Evidence** | [What indicated this CI system] |

### Inference Evidence (if applicable)

If CI was inferred rather than confirmed, document the signals:

| Signal Type | Evidence Found |
|-------------|----------------|
| Environment variables | [e.g., `$JENKINS_URL` in deploy.sh] |
| Script naming | [e.g., `jenkins-build.sh`] |
| Documentation | [e.g., README mentions "Jenkins pipeline"] |
| Badges | [e.g., Jenkins badge URL] |
| Makefile targets | [e.g., `make bamboo-deploy`] |

## Migration Scope

### Workflows Created
| Workflow | Source | Purpose |
|----------|--------|---------|
| ci.yml | [Source pipeline/stage] | Main CI |
| release.yml | [Source] | Release automation |
| security.yml | New | Security scanning |

### Concept Translation Applied
| Original Concept | GitHub Actions Equivalent |
|------------------|---------------------------|
| [e.g., Concourse `serial: true`] | [e.g., `concurrency` group] |
| [e.g., Jenkins `stash/unstash`] | [e.g., `upload/download-artifact`] |

### Copilot Integration
- [x] `.github/copilot-instructions.md`
- [x] `.github/workflows/copilot-setup-steps.yml`

### Templates
- [x] Bug report template
- [x] Feature request template
- [x] Pull request template

## Secrets Migration Required

| GitHub Secret | Source System | Original Reference | Description |
|---------------|---------------|---------------------|-------------|
| [SECRET_NAME] | [Jenkins/Concourse/etc] | [credentials('x') / ((x))] | [What it's for] |

## External CI Decommissioning

<conditional trigger="migrating from external CI system">
If migrating from external CI (Jenkins, TeamCity, Bamboo, etc.):

### Server-Side Actions Required
1. [ ] Export pipeline configuration for archival
2. [ ] Disable webhooks from [CI system] to this repository
3. [ ] Remove or archive job/pipeline definitions
4. [ ] Revoke service account credentials used by old CI
5. [ ] Update any deployment targets that expect old CI

### Repository Actions Required
1. [ ] Remove CI-specific scripts if no longer needed
2. [ ] Update README badges to GitHub Actions
3. [ ] Update CONTRIBUTING.md with new CI instructions
4. [ ] Remove any CI-specific configuration files
</conditional>

## Manual Steps

1. [ ] Add secrets to GitHub repository settings
2. [ ] Enable Copilot code review (Settings -> Rules -> Rulesets)
3. [ ] Configure branch protection rules
4. [ ] Verify workflows pass on test branch
5. [ ] Notify team of CI migration

## Enterprise Cloud Readiness Checklist

- [ ] All CI/CD runs on GitHub Actions
- [ ] No external CI dependencies for build/test/deploy
- [ ] Secrets documented and migrated
- [ ] Copilot configured
- [ ] Branch protection configured
- [ ] External CI decommissioned or archived
```
</phase>

---

<error_handling>
# ERROR HANDLING

| Error | Response |
|-------|----------|
| Permission denied | Verify repository write access |
| File exists | Prompt before overwriting if `--interactive` |
| Invalid YAML | Validate syntax before writing |
| Unknown CI type | Fall back to generic workflow templates |
| Missing dependencies | Note in migration summary for manual resolution |
| API rate limit | Pause and retry with exponential backoff |

<conditional trigger="error occurs during migration">
When an error occurs:
1. Log the error with phase and file context
2. Determine if error is blocking or recoverable
3. If recoverable, continue with remaining phases
4. If blocking, generate partial migration summary
5. Include all errors and incomplete items in MIGRATION_SUMMARY.md
</conditional>
</error_handling>

---

# EXECUTION PROTOCOL

1. **Complete Phase 1-2 before any file creation**
2. **Report CI detection findings with confidence level before proceeding**
3. <conditional trigger="detection level is INFERRED or SUSPECTED">If INFERRED or SUSPECTED, explain evidence found</conditional>
4. <conditional trigger="--interactive flag is set">If `--interactive`, confirm CI type with user</conditional>
5. <conditional trigger="--dry-run flag is set">If `--dry-run`, output all files to stdout with `--- FILE: path ---` headers</conditional>
6. **Validate all YAML before writing**
7. <conditional trigger="existing workflows found">Never overwrite existing `.github/workflows/` without confirmation</conditional>
8. <conditional trigger="external CI detected">For external CI, note that full pipeline logic may not be discoverable</conditional>
9. **Commit incrementally with descriptive messages**

## Handling External CI Limitations

<conditional trigger="CI configuration lives on external server">
When CI configuration lives on an external server (Jenkins, TeamCity, Bamboo, etc.):

1. **Acknowledge the limitation**: State that full pipeline cannot be reverse-engineered
2. **Use available signals**: Build workflows from scripts, Makefile targets, documentation
3. **Create interview questions**: If `--interactive`, ask user about:
   - Build steps and their order
   - Test commands
   - Deployment targets
   - Required credentials/secrets
   - Artifact handling
4. **Generate reasonable defaults**: Based on language detection and common patterns
5. **Document gaps**: Note in MIGRATION_SUMMARY what couldn't be determined
</conditional>

Begin with Phase 1 exploration now.
