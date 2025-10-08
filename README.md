# bfs_pipeline

## Code Quality and Standards

This project enforces consistent code style, linting, and type checking using:

- Ruff (lint + formatter)
- Mypy (static type checking)
- Pre-commit hooks (local guardrails)
- GitHub Actions CI (lint, type, test on push/PR)

Below are the configurations and how to use them.

---

## Ruff configuration (`ruff.toml`)

Ruff is configured for Python >= 3.7 with strong code quality rules and formatting options.

- Target version: `py37`
- Line length: `100`
- Docstrings: Google convention via Pydocstyle
- Imports: isort (via Ruff) with first-party `bfs_executor`
- Quotes: double quotes enforced
- Complexity: McCabe max-complexity = 10
- Tests: relaxed rules for asserts/prints/docstrings
- Formatter: enabled in `ruff.toml` under `[format]`

Key sections:

```toml
target-version = "py37"
line-length = 100

[lint.pydocstyle]
convention = "google"

[lint.isort]
known-first-party = ["bfs_executor"]
combine-as-imports = true
force-sort-within-sections = true

[lint.flake8-quotes]
inline-quotes = "double"
multiline-quotes = "double"
docstring-quotes = "double"
avoid-escape = true

[lint.mccabe]
max-complexity = 10

[lint.per-file-ignores]
"bfs_executor/tests/**" = ["S101", "D", "T20"]

lint.preview = true

[format]
quote-style = "double"
indent-style = "space"
line-ending = "auto"
skip-magic-trailing-comma = false
docstring-code-format = true
docstring-code-line-length = 100
preview = true
```

Usage:

```powershell
# Format code
ruff format .

# Lint (and optionally fix) code
ruff check .
ruff check --fix .
```

VS Code tip: set Ruff as default formatter and format on save.

---

## Ruff formatter in `pyproject.toml`

Ruff formatter options are also defined in `pyproject.toml` so tools that read
PEP 621-style metadata can discover the formatter configuration.

```toml
[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "auto"
skip-magic-trailing-comma = false
docstring-code-format = true
docstring-code-line-length = 100
preview = true
```

---

## Mypy configuration (`mypy.ini`)

Targets Python 3.7 with strict defaults in source code and looser requirements for tests.

```ini
[mypy]
python_version = 3.7
mypy_path = bfs_executor/src
warn_unused_configs = True
warn_return_any = True
warn_redundant_casts = True
warn_unused_ignores = True
disallow_untyped_defs = True
disallow_incomplete_defs = True
no_implicit_optional = True
check_untyped_defs = True
show_error_codes = True
pretty = True
exclude = (?x)(^venv/|^\.venv/|^build/|^dist/|^\.mypy_cache/)

[mypy-bfs_executor.tests.*]
disallow_untyped_defs = False
```

Usage:

```powershell
mypy --config-file mypy.ini bfs_executor
```

---

## Pre-commit hooks (`.pre-commit-config.yaml`)

Local guardrails to keep code clean before it’s committed.

```yaml
repos:
	- repo: https://github.com/pre-commit/pre-commit-hooks
		rev: v4.6.0
		hooks:
			- id: end-of-file-fixer
			- id: trailing-whitespace
			- id: check-added-large-files

	- repo: https://github.com/astral-sh/ruff-pre-commit
		rev: v0.6.9
		hooks:
			- id: ruff
				args: ["--fix"]
			- id: ruff-format

	- repo: https://github.com/pre-commit/mirrors-mypy
		rev: v1.11.2
		hooks:
			- id: mypy
				args: ["--config-file", "mypy.ini"]
```

Usage:

```powershell
pip install pre-commit
pre-commit install
# Run on all files
pre-commit run --all-files
```

---

## GitHub Actions CI (`.github/workflows/ci.yml`)

Runs on every push and pull request: Ruff format check, Ruff lint, mypy, and pytest.

```yaml
name: ci

on:
	push:
	pull_request:

jobs:
	lint-type-test:
		runs-on: ubuntu-latest
		steps:
			- name: Checkout
				uses: actions/checkout@v4

			- name: Set up Python
				uses: actions/setup-python@v5
				with:
					python-version: '3.11'

			- name: Set PYTHONPATH for local imports
				run: echo "PYTHONPATH=$(pwd)/bfs_executor/src" >> $GITHUB_ENV

			- name: Install tools
				run: |
					python -m pip install --upgrade pip
					pip install ruff mypy pytest

			- name: Ruff format check
				run: ruff format --check .

			- name: Ruff lint
				run: ruff check .

			- name: Type check (mypy)
				run: mypy --config-file mypy.ini bfs_executor

			- name: Run tests (pytest)
				run: pytest -q
```

---

## VS Code settings (recommended)

The workspace includes `.vscode/settings.json` with sensible defaults:

```jsonc
{
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit",
    "source.organizeImports": "explicit"
  },
  "editor.formatOnSave": true,
  "ruff.enable": true,
  "ruff.organizeImports": true,
  "python.analysis.extraPaths": ["${workspaceFolder}/bfs_executor/src"],
  "python.testing.pytestArgs": ["bfs_executor"],
  "python.testing.unittestEnabled": false,
  "python.testing.pytestEnabled": true
}
```

Optional: set Ruff as the default formatter in VS Code for the repo by adding:

```jsonc
"editor.defaultFormatter": "charliermarsh.ruff"
```

---

## Quick start for contributors

```powershell
# 1) Create/activate your venv
python -m venv .venv; .\.venv\Scripts\Activate.ps1

# 2) Install dev tools
pip install -U ruff mypy pytest pre-commit

# 3) Install pre-commit hooks
pre-commit install

# 4) Check and format before commit
ruff format .
ruff check --fix .
mypy --config-file mypy.ini bfs_executor
pytest -q
```

This setup keeps the codebase consistent and high-quality with minimal friction. If you want stricter or looser rules, propose changes in `ruff.toml` or `mypy.ini` via PR.
