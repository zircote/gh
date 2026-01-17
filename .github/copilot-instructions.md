# Copilot Instructions

You are working in the gh plugin repository for Claude Code.

## Project Overview

This is a Claude Code plugin providing GitHub ecosystem integration with:
- 12 git workflow and GitHub integration commands
- 1 Copilot onboarding agent
- 1 GitHub ecosystem configuration skill

## Plugin Structure

```
.claude-plugin/plugin.json  # Plugin manifest
agents/                     # Copilot assistant agent
commands/                   # 12 workflow commands
skills/                     # GitHub ecosystem skill
```

## Development Guidelines

1. Follow Claude Code plugin standards
2. Keep changes focused and reviewable
3. Update CHANGELOG.md for user-facing changes
4. Commands must follow markdown frontmatter format

## Command Format

```yaml
---
name: command-name
description: Brief description
allowed-tools: Bash, Read, Write
---

[Command instructions]
```

## Skill Format

Skills use SKILL.md files with YAML frontmatter containing `name`, `description`, and optional `triggerPhrases`.
