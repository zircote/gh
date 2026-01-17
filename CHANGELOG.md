# Changelog

All notable changes to the gh plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.5.0] - 2025-01-17

### Changed
- Migrated to standalone repository (zircote/gh)
- Added GitHub Actions CI for plugin validation
- Updated README with badges and installation instructions
- Added repository field to plugin.json

## [0.4.2] - 2025-12-24

## [0.4.1] - 2025-12-24

## [0.4.0] - 2025-12-21

### Added
- `review-comments` command for processing GitHub PR review comments
  - Validity assessment with confidence scoring (0-100%)
  - Interactive mode (default) with AskUserQuestion prompts
  - Auto mode (`--auto`) for batch processing with configurable thresholds
  - Specialist agent routing for complex remediations (security, performance, etc.)
  - Response generation with templates for accept/reject/question cases
  - Dry-run support (`--dry-run`) for previewing actions
  - Conversation resolution via GitHub GraphQL API

## [0.3.2] - 2025-12-20

## [0.3.1] - 2025-12-20

## [0.3.0] - 2025-12-20

### Added
- Comprehensive error handling to pr.md command
- Pre-flight checks for gh CLI, authentication, and branch state

### Changed
- Rewrote README.md to document all 9 commands, agent, and skill
- Updated plugin.json description and keywords

## [0.2.0] - 2025-12-19

### Added
- copilot-onboard command (merged from copilot plugin)
- copilot-assistant agent
- migrate command for multi-CI migration
- github-ecosystem skill for comprehensive repo setup

### Changed
- Merged copilot plugin functionality into gh plugin

## [0.1.0] - 2025-12-18

### Added
- Initial release with git workflow commands
- cp, pr, fr, sync, ff, prune commands
