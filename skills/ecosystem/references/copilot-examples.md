# Example Generated Files

This shows what the onboarding command typically generates for a Python/uv project.

---

## `.github/copilot-instructions.md`

```markdown
# Copilot Instructions

## Project Overview

[Auto-generated from README or pyproject.toml description]

## Tech Stack

- Python 3.11+
- Package manager: uv (astral)
- Testing: pytest
- Linting: ruff
- Type checking: mypy (if present)

## Development Commands

### Install dependencies
```bash
uv sync --all-extras --dev
```

### Run tests
```bash
uv run pytest
```

### Run linting
```bash
uv run ruff check .
uv run ruff format --check .
```

### Fix linting issues
```bash
uv run ruff check . --fix
uv run ruff format .
```

## Code Standards

- All code must pass ruff checks before committing
- All new functionality requires tests
- Use type hints for public functions and methods
- Follow existing patterns in the codebase

## Before Submitting

1. Run `uv run ruff check . --fix && uv run ruff format .`
2. Run `uv run pytest`
3. Ensure all CI checks would pass

## Additional Context

See `CLAUDE.md` for architectural decisions and detailed coding guidelines.
```

---

## `.github/workflows/copilot-setup-steps.yml`

```yaml
name: Copilot Setup Steps

on:
  workflow_dispatch:
  push:
    paths:
      - .github/workflows/copilot-setup-steps.yml
  pull_request:
    paths:
      - .github/workflows/copilot-setup-steps.yml

jobs:
  copilot-setup-steps:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          enable-cache: true
          cache-dependency-glob: "uv.lock"

      - name: Set up Python
        run: uv python install

      - name: Install dependencies
        run: uv sync --all-extras --dev

      - name: Verify Python environment
        run: |
          echo "Python version:"
          uv run python --version
          echo ""
          echo "Installed packages:"
          uv pip list

      - name: Verify development tools
        run: |
          echo "Ruff version:"
          uv run ruff --version
          echo ""
          echo "Pytest version:"
          uv run pytest --version

      - name: Verify tests can run
        run: uv run pytest --collect-only -q
```

---

## `.github/instructions/python.instructions.md`

```yaml
---
applyTo: "**/*.py"
---
```

```markdown
## Python Code Guidelines

- Use type hints for all function parameters and return types
- Prefer f-strings over .format() or % formatting
- Use pathlib.Path instead of os.path
- Imports should be sorted (ruff handles this)
- Maximum line length: 88 characters (ruff default)

## Error Handling

- Use specific exception types, not bare `except:`
- Log exceptions with context before re-raising
- Use `raise ... from err` for exception chaining

## Testing

- Test files: `test_*.py` or `*_test.py`
- Use pytest fixtures for shared setup
- Aim for single assertion per test when practical
- Use `pytest.mark.parametrize` for data-driven tests
```

---

## `.github/instructions/tests.instructions.md`

```yaml
---
applyTo: "**/test_*.py,**/*_test.py,**/tests/**/*.py"
---
```

```markdown
## Test Guidelines

- Each test should be independent and isolated
- Use descriptive test names: `test_<function>_<scenario>_<expected>`
- Use fixtures for common setup, conftest.py for shared fixtures
- Mock external dependencies (network, filesystem, time)
- Include both happy path and error cases

## Assertions

- Use plain `assert` statements (pytest rewrites them)
- For exceptions: `with pytest.raises(ExceptionType):`
- For approximate floats: `pytest.approx()`

## Running Tests

```bash
uv run pytest                    # all tests
uv run pytest -x                 # stop on first failure
uv run pytest -k "test_name"     # run specific test
uv run pytest --cov              # with coverage
```
```

---

## Usage Notes

After these files are created:

1. **Commit and push** to your default branch
2. **Test the setup workflow**: Go to Actions tab → "Copilot Setup Steps" → Run workflow
3. **Assign an issue to Copilot**: On any issue, set assignee to `@copilot`
4. **Monitor**: Watch the agent session logs to see Copilot working

### If you have secrets needed for tests

Add them to the `copilot` environment in GitHub:
1. Settings → Environments → New environment → Name it `copilot`
2. Add secrets/variables that your tests need
