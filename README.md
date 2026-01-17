# gh Plugin

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-blueviolet)](https://github.com/anthropics/claude-code)
[![CI](https://github.com/zircote/gh/actions/workflows/ci.yml/badge.svg)](https://github.com/zircote/gh/actions/workflows/ci.yml)
[![Version](https://img.shields.io/badge/version-0.5.0-green.svg)](https://github.com/zircote/gh/releases)

GitHub ecosystem integration with streamlined git workflows, Copilot coding agent configuration, and multi-CI migration tools.

## Installation

```bash
claude plugin install zircote/gh
```

## Verify Installation

After installing, verify the commands and agent are available:

```bash
# Test git workflow command (dry run)
claude "/gh:prune"

# Verify gh CLI is authenticated
gh auth status

# Test Copilot onboarding (shows help without modifying files)
claude "What does the /gh:copilot-onboard command do?"
```

You should see the prune command show stale branches (if any) and confirmation that gh CLI is authenticated.

## Contents

| Component | Count | Description |
|-----------|-------|-------------|
| Commands | 12 | Git workflow and GitHub integration commands |
| Agents | 1 | Copilot onboarding specialist |
| Skills | 1 | GitHub ecosystem configuration |

## Commands

### Git Workflow Commands

| Command | Description | Arguments |
|---------|-------------|-----------|
| `/gh:cp` | Stage, commit, and push all changes | - |
| `/gh:pr` | Create a pull request via `gh` CLI | `[to-branch] [from-branch]` |
| `/gh:fr` | Fetch and rebase onto remote branch | `[remote] [branch]` |
| `/gh:sync` | Full sync: fetch, rebase, push | `[remote] [branch]` |
| `/gh:ff` | Fast-forward merge only | `[remote] [branch]` |
| `/gh:prune` | Clean up stale local branches | `[--force]` |

### Copilot, Migration & Review Commands

| Command | Description | Arguments |
|---------|-------------|-----------|
| `/gh:copilot-onboard` | Configure repo for GitHub Copilot coding agent | `[repository-path]` |
| `/gh:onboard` | Alias for copilot-onboard | `[repository-path]` |
| `/gh:migrate` | Migrate multi-CI to GitHub Actions | `[--ci=TYPE]` |
| `/gh:ci-assist` | Onboard repository to GitHub ecosystem | - |
| `/gh:pr-fix` | Complete PR remediation workflow | `[pr-number]` |
| `/gh:review-comments` | Process PR review comments with assessment & remediation | `[pr-number] [--auto\|--interactive]` |

## Command Details

### `/gh:cp` - Commit and Push

Stages, commits, and pushes all changes with smart commit message generation:

```bash
/gh:cp
```

Features:
- Reviews all changes for sensitive data (API keys, credentials)
- Generates conventional commit messages (`feat:`, `fix:`, `refactor:`, etc.)
- Splits new files and modifications into separate commits
- Never adds AI attribution signatures

### `/gh:pr` - Create Pull Request

Creates a pull request using the GitHub CLI:

```bash
/gh:pr                    # PR to main from current branch
/gh:pr develop            # PR to develop from current branch
/gh:pr main feature/auth  # PR to main from feature/auth
```

Requires: `gh` CLI installed and authenticated

### `/gh:fr` - Fetch and Rebase

Fetches from remote and rebases current branch:

```bash
/gh:fr                    # Rebase onto origin/current-upstream
/gh:fr upstream           # Rebase onto upstream/...
/gh:fr origin main        # Rebase onto origin/main
```

Includes conflict resolution guidance if rebase fails.

### `/gh:sync` - Full Sync Cycle

Complete synchronization: fetch, rebase, and push with confirmation:

```bash
/gh:sync
/gh:sync origin main
```

Features:
- Pre-flight checks for uncommitted changes
- Shows incoming commits before rebasing
- Confirms before pushing
- Conflict resolution assistance

### `/gh:ff` - Fast-Forward Only

Updates branch via fast-forward merge (no history rewriting):

```bash
/gh:ff
/gh:ff origin main
```

Fails gracefully if fast-forward isn't possible, offering alternatives.

### `/gh:prune` - Clean Up Branches

Removes stale local branches that have been merged or deleted on remote:

```bash
/gh:prune          # Dry run - shows what would be deleted
/gh:prune --force  # Actually delete stale branches
```

### `/gh:copilot-onboard` - Copilot Configuration

Configures a repository for GitHub Copilot coding agent:

```bash
/gh:copilot-onboard
/gh:copilot-onboard /path/to/repo
```

Generates:
- `.github/copilot-instructions.md` - Repository-wide instructions
- `.github/workflows/copilot-setup-steps.yml` - Environment setup
- `.github/instructions/*.instructions.md` - Scoped instructions

Cross-references with existing `CLAUDE.md` to avoid duplication.

### `/gh:migrate` - CI Migration

Migrates from various CI systems to GitHub Actions:

```bash
/gh:migrate              # Auto-detect CI system
/gh:migrate --ci=jenkins # Migrate from Jenkins
```

Supports: Jenkins, CircleCI, GitLab CI, Travis CI, Azure Pipelines, Bitbucket Pipelines, Concourse, Drone, TeamCity

### `/gh:review-comments` - Process PR Review Comments

Processes GitHub PR review comments with validity assessment, remediation, and response generation:

```bash
/gh:review-comments              # Current branch's PR, interactive mode
/gh:review-comments 123          # Specific PR, interactive mode
/gh:review-comments 123 --auto   # Auto-process with default thresholds
/gh:review-comments --dry-run    # Preview actions without executing
```

**Flags:**
- `--auto` - Non-interactive: auto-accept findings with >=85% confidence
- `--interactive` - Prompt at each decision (default)
- `--confidence=N` - Set auto-accept threshold (0-100, default: 85)
- `--dry-run` - Show proposed actions without executing

**Workflow:**

1. **Fetches all review comments** from the PR
2. **Categorizes** into: Code Review, Questions, Suggestions, Blockers, Approvals
3. **Assesses validity** with confidence scoring (0-100%)
4. **Prompts for decisions** (in interactive mode) or auto-processes
5. **Remediates accepted findings** using appropriate specialist agents
6. **Posts responses** to all comments with explanations
7. **Resolves conversations** where appropriate

## Agent

### copilot-assistant

GitHub Copilot coding agent onboarding specialist.

**When to use:**
- Configuring repositories for GitHub Copilot
- Creating `copilot-instructions.md`
- Setting up `copilot-setup-steps.yml` workflows
- Aligning Copilot and Claude Code configurations

**Invocation:**
- Via `/gh:copilot-onboard` command
- Direct agent call when discussing Copilot setup

## Skill

### github-ecosystem

Generates comprehensive GitHub repository configuration.

**Components generated:**
- CI/CD workflows (ci.yml, release.yml, docker.yml)
- Issue templates (bug report, feature request)
- PR template with checklist
- CODEOWNERS file
- Dependabot configuration
- Copilot instructions

**Supported languages:**
- Python (pyproject.toml detection)
- Go (go.mod detection)
- TypeScript (package.json + tsconfig.json detection)

**Usage:**
```bash
# In conversation
"Set up GitHub ecosystem for this Python project"

# Or via trigger phrases
"Add GitHub Actions to this repo"
"Create issue templates"
"Configure dependabot"
```

## File Structure

```
gh/
├── .claude-plugin/
│   └── plugin.json
├── agents/
│   └── copilot-assistant.md
├── commands/
│   ├── ci-assist.md
│   ├── copilot-onboard.md
│   ├── cp.md
│   ├── ff.md
│   ├── fr.md
│   ├── migrate.md
│   ├── onboard.md
│   ├── pr-fix.md
│   ├── pr.md
│   ├── prune.md
│   ├── review-comments.md
│   └── sync.md
├── skills/
│   └── ecosystem/
│       ├── SKILL.md
│       ├── references/
│       └── scripts/
├── CHANGELOG.md
├── LICENSE
└── README.md
```

## Troubleshooting

### `gh` CLI Not Found

Install and authenticate the GitHub CLI:

```bash
# macOS
brew install gh
gh auth login

# Linux
sudo apt install gh
gh auth login

# Windows
winget install GitHub.cli
gh auth login
```

### Rebase Conflicts

If `/gh:sync` or `/gh:fr` encounters conflicts:

1. Resolve conflicts in each file (remove `<<<<<<<`, `=======`, `>>>>>>>` markers)
2. Stage resolved files: `git add <file>`
3. Continue: `git rebase --continue`
4. Or abort: `git rebase --abort`

### Push Rejected

If push fails after rebase:

1. Run `/gh:sync` again to incorporate new remote changes
2. Or use `git push --force-with-lease` (only if you understand the implications)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

MIT
