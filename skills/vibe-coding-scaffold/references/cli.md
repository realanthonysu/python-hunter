# CLI Tool Project Templates

Exact file contents for generating a command-line tool project (distributable via PyPI).

## pyproject.toml

```toml
[project]
name = "{package_name}"
version = "0.1.0"
description = "{description}"
readme = "README.md"
license = "Apache-2.0"
requires-python = ">={python_version}"
authors = [
    { name = "{author_name}", email = "{author_email}" },
]
classifiers = [
    "Development Status :: 3 - Alpha",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: {python_version}",
    "License :: OSI Approved :: Apache Software License",
    "Operating System :: OS Independent",
]
dependencies = [
    "click>=8.1",
]

[project.scripts]
{cli_command} = "{package_name}.cli:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/{package_name}"]

# ── uv ──
[tool.uv]
dev-dependencies = [
    "pytest>=8.0",
    "coverage>=7.0",
    "mypy>=1.15",
    "ty>=0.0",
    "ruff>=0.11",
]

# ── ruff ──
[tool.ruff]
target-version = "py{python_version_short}"
line-length = 120

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "UP",  # pyupgrade
    "B",   # flake8-bugbear
    "SIM", # flake8-simplify
    "TCH", # flake8-type-checking
    "RUF", # ruff-specific
]

# ── mypy ──
[tool.mypy]
python_version = "{python_version}"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

# ── ty ──
[tool.ty]
python-version = "{python_version}"

# ── pytest + coverage ──
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["src"]

[tool.coverage.report]
show_missing = true
fail_under = 80
```

## src/{package_name}/__init__.py

```python
"""{description}"""

__version__ = "0.1.0"
```

## src/{package_name}/cli.py

```python
"""Command-line interface entry point."""

import click

from {package_name} import __version__


@click.group()
@click.version_option(version=__version__, prog_name="{package_name}")
def main() -> None:
    """{description}"""


@main.command()
@click.option("--name", default="World", help="Name to greet.")
def hello(name: str) -> None:
    """Print a greeting."""
    click.echo(f"Hello, {name}!")


if __name__ == "__main__":
    main()
```

## tests/test_example.py

```python
"""Example test file — replace with real tests."""

from click.testing import CliRunner

from {package_name}.cli import main
from {package_name} import __version__


def test_version():
    assert __version__ == "0.1.0"


def test_hello_default():
    runner = CliRunner()
    result = runner.invoke(main, ["hello"])
    assert result.exit_code == 0
    assert "Hello, World!" in result.output


def test_hello_custom_name():
    runner = CliRunner()
    result = runner.invoke(main, ["hello", "--name", "Alice"])
    assert result.exit_code == 0
    assert "Hello, Alice!" in result.output


def test_help():
    runner = CliRunner()
    result = runner.invoke(main, ["--help"])
    assert result.exit_code == 0
    assert "hello" in result.output  # subcommand listed
```

## .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: {RUFF_PRE_COMMIT_LATEST}
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: {MYPY_PRE_COMMIT_LATEST}  # 查看: https://github.com/pre-commit/mirrors-mypy/releases
    hooks:
      - id: mypy
        additional_dependencies:
          - click

  - repo: https://github.com/astral-sh/ty-pre-commit
    rev: {TY_PRE_COMMIT_LATEST}
    hooks:
      - id: ty
```

> **生成时必须查询最新版本**: 用 `git ls-remote --tags` 或 GitHub API 查询上述三个仓库的最新 release tag，替换 `{*_LATEST}` 占位符。

## .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so

# Distribution / packaging
dist/
build/
*.egg-info/
*.egg

# Virtual environments
.venv/
venv/
ENV/

# IDE
.idea/
.vscode/
*.swp
*.swo

# Testing
.pytest_cache/
htmlcov/
.coverage
.coverage.*
coverage.xml

# Type checking
.mypy_cache/
.ty/

# Ruff
.ruff_cache/

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db
```

## README.md

```markdown
# {package_name}

{description}

## Installation

```bash
pip install {package_name}
```

## Usage

```bash
# Show help
{cli_command} --help

# Run the hello command
{cli_command} hello

# Greet a specific name
{cli_command} hello --name Alice
```

## Development

```bash
# Install dependencies (editable mode for CLI entry points)
uv sync

# Run CLI directly from source
uv run python -m {package_name}.cli

# Run tests
uv run pytest

# Run linter
uv run ruff check .

# Run type checker
uv run mypy src/
uv run ty check src/
```

## License

Apache-2.0
```
