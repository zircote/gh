---
description: Onboard a repository for GitHub Copilot coding agent with configuration aligned to Claude Code
argument-hint: "[repository-path]"
allowed-tools: Read, Write, Glob, Grep, Bash
---

<help_check>
## Help Check

If `$ARGUMENTS` contains `--help` or `-h`:

**Output this help and HALT (do not proceed further):**

<help_output>
```
COPILOT_ONBOARD(1)              User Commands              COPILOT_ONBOARD(1)

NAME
    copilot-onboard - Configure repository for GitHub Copilot agent

SYNOPSIS
    /gh:copilot-onboard [repository-path]

DESCRIPTION
    Onboard a repository for GitHub Copilot coding agent with configuration
    aligned to Claude Code. Discovers existing CLAUDE.md, .claude/ directory,
    and creates .github/copilot-instructions.md with consistent settings.

OPTIONS
    [repository-path]         Path to repository (default: current directory)
    --help, -h                Show this help message

EXAMPLES
    /gh:copilot-onboard                 Onboard current repository
    /gh:copilot-onboard ~/projects/app  Onboard specific repository

SEE ALSO
    /gh:migrate         Full GitHub ecosystem onboarding
    /gh:ci-assist       CI/CD migration with Copilot

                                                        COPILOT_ONBOARD(1)
```
</help_output>

**After outputting help, HALT immediately. Do not proceed with command execution.**
</help_check>

---

Analyze this repository and configure it for GitHub Copilot coding agent, ensuring alignment with any existing Claude Code configuration.

## Task

1. **Discover existing configuration**:
   - Check for: `CLAUDE.md`, `.claude/`, `AGENTS.md`, `.github/copilot-instructions.md`
   - Identify: package manager, test runner, linter, formatter
   - Review: `.github/workflows/` for CI patterns

2. **Generate Copilot configuration files**:

### `.github/copilot-instructions.md`
Repository-wide instructions including:
- Project summary and tech stack
- Dependency installation command
- Test command
- Lint/format commands
- Code style requirements
- Key architectural patterns

### `.github/workflows/copilot-setup-steps.yml`
Environment setup with job named `copilot-setup-steps`:
```yaml
name: Copilot Setup Steps
on:
  workflow_dispatch:
  push:
    paths: [.github/workflows/copilot-setup-steps.yml]
jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Language runtime setup
      # Package manager setup  
      # Dependency installation
      # Tool verification
```

### `.github/instructions/*.instructions.md` (optional)
Scoped instructions with frontmatter:
```yaml
---
applyTo: "**/*.py"
---
```

3. **For Python projects with uv**, use:
   - `astral-sh/setup-uv@v4` action
   - `uv sync --all-extras --dev` for deps
   - `uv run ruff check .` and `uv run pytest` for quality

4. **Cross-reference with CLAUDE.md**: Don't duplicate—Copilot reads CLAUDE.md too. Add complementary info only.

5. **Output summary**: What was detected, files created, any manual steps needed.

$ARGUMENTS
