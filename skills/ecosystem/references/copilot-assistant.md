# GitHub Copilot Coding Agent Onboarding Kit

Tools for Claude Code to configure existing projects for GitHub Copilot coding agent, maintaining alignment with your Claude Code standards.

## Quick Start

### Option 1: Slash Command (Recommended)

Copy the command to your global Claude commands:

```bash
cp .claude/commands/copilot-onboard.md ~/.claude/commands/
```

Then in any project:

```
/copilot-onboard
```

### Option 2: Direct Prompt

In Claude Code, paste the contents of `copilot-onboard.md` or reference it:

```
@copilot-onboard.md analyze this project
```

## What It Does

1. **Analyzes** your existing project structure:
   - Detects CLAUDE.md, package manager, test framework, linters
   - Reviews existing CI workflows for patterns

2. **Generates** Copilot configuration:
   - `.github/copilot-instructions.md` — repository-wide agent instructions
   - `.github/workflows/copilot-setup-steps.yml` — environment setup
   - `.github/instructions/*.instructions.md` — scoped instructions (optional)

3. **Aligns** with existing Claude Code config:
   - Reads CLAUDE.md (Copilot supports this natively)
   - Doesn't duplicate—adds complementary info

## Files in This Kit

```
copilot-onboard-kit/
├── README.md                      # This file
├── copilot-onboard.md             # Full prompt with detailed instructions
├── copilot-onboard-examples.md    # Example generated files for reference
└── .claude/
    └── commands/
        └── copilot-onboard.md     # Slash command version
```

## Python/uv Projects

The kit defaults to uv-first Python configuration:

- Uses `astral-sh/setup-uv@v4` action
- Runs `uv sync --all-extras --dev`
- Verifies `ruff` and `pytest` availability

## After Running

1. Review generated files
2. Commit to default branch
3. Test: Actions → "Copilot Setup Steps" → Run workflow
4. Assign an issue to `@copilot`

## Requirements

- GitHub Copilot Pro+ or Copilot Enterprise with coding agent enabled
- Repository on GitHub (coding agent runs in GitHub Actions)

## Cross-Compatibility

Copilot coding agent reads these instruction formats:
- `.github/copilot-instructions.md`
- `.github/instructions/**/*.instructions.md`
- `CLAUDE.md` ✓
- `AGENTS.md`
- `GEMINI.md`

Your existing CLAUDE.md works for both tools.
