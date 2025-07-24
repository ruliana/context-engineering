# Python UV Development Playbook

## Key Concepts

- **uv**: Extremely fast Python package and project manager written in Rust, designed to replace pip, pip-tools, poetry, pyenv, and virtualenv
- **Project**: A directory containing Python code with `pyproject.toml` and optional `uv.lock` files for dependency management
- **Workspace**: Multi-project setup where multiple Python packages share dependencies and are developed together
- **Lock File**: `uv.lock` file containing exact dependency versions and hashes for reproducible builds
- **Virtual Environment**: Isolated Python environment automatically managed by uv for each project
- **uvx**: Tool runner that executes command-line tools from Python packages in isolated environments
- **PyPI**: Python Package Index where published packages are distributed and installed from
- **Wheel**: Built distribution format (.whl) that's faster to install than source distributions
- **Source Distribution (sdist)**: Archive containing source code and metadata (.tar.gz)

## Project Organization and Structure

### Standard Library Layout
```
my-awesome-lib/
├── pyproject.toml          # Project configuration and dependencies
├── README.md              # Project description and usage
├── LICENSE                # License file
├── .gitignore            # Git ignore patterns
├── uv.lock               # Locked dependency versions
├── .venv/                # Virtual environment (auto-created)
├── src/                  # Source code directory
│   └── my_awesome_lib/   # Package directory (importable)
│       ├── __init__.py   # Package initialization
│       ├── core.py       # Core functionality
│       └── utils.py      # Utility functions
├── tests/                # Test directory
│   ├── __init__.py       # Test package init
│   ├── test_core.py      # Core module tests
│   └── test_utils.py     # Utility tests
└── docs/                 # Documentation
    ├── index.md          # Main documentation
    └── api.md            # API reference
```
**When to use**: Pure Python libraries for import and distribution
**Benefits**: Clean separation, standard packaging, easy testing

### CLI Tool Layout
```
my-cli-tool/
├── pyproject.toml          # Includes [project.scripts] entry points
├── README.md
├── LICENSE
├── uv.lock
├── .venv/
├── src/
│   └── my_cli_tool/
│       ├── __init__.py
│       ├── main.py       # CLI entry point with main() function
│       ├── commands/     # Command modules
│       │   ├── __init__.py
│       │   ├── init.py   # 'init' subcommand
│       │   └── build.py  # 'build' subcommand
│       ├── core.py       # Business logic
│       └── config.py     # Configuration handling
├── tests/
│   ├── test_main.py      # CLI interface tests
│   ├── test_commands/    # Command-specific tests
│   └── fixtures/         # Test data
└── docs/
    ├── usage.md          # Usage examples
    └── commands.md       # Command reference
```
**When to use**: Command-line applications published to PyPI for uvx usage
**Benefits**: Clear CLI structure, testable commands, professional layout

### Mixed Library + CLI Layout
```
awesome-toolkit/
├── pyproject.toml          # Both library dependencies and CLI scripts
├── README.md
├── LICENSE
├── uv.lock
├── .venv/
├── src/
│   └── awesome_toolkit/
│       ├── __init__.py     # Library public API
│       ├── cli/            # CLI-specific code
│       │   ├── __init__.py
│       │   ├── main.py     # CLI entry point
│       │   └── commands.py # CLI commands
│       ├── lib/            # Library functionality
│       │   ├── __init__.py
│       │   ├── core.py     # Core library functions
│       │   └── models.py   # Data models
│       └── shared/         # Shared utilities
│           ├── __init__.py
│           ├── utils.py    # Common utilities
│           └── exceptions.py # Custom exceptions
├── tests/
│   ├── test_lib/           # Library tests
│   ├── test_cli/           # CLI tests
│   └── integration/        # Integration tests
└── docs/
    ├── library/            # Library documentation
    └── cli/                # CLI documentation
```
**When to use**: Projects offering both programmatic API and CLI interface
**Benefits**: Unified codebase, shared functionality, dual distribution

### Workspace Layout (Multiple Packages)
```
my-workspace/
├── pyproject.toml          # Workspace configuration
├── README.md
├── uv.lock                 # Shared lock file
├── packages/               # Individual packages
│   ├── core/               # Core library
│   │   ├── pyproject.toml
│   │   └── src/core/
│   ├── cli/                # CLI tool using core
│   │   ├── pyproject.toml
│   │   └── src/cli_tool/
│   └── plugins/            # Plugin system
│       ├── pyproject.toml
│       └── src/plugins/
├── tools/                  # Development tools
│   └── scripts/            # Build/maintenance scripts
├── tests/                  # Workspace-wide tests
│   ├── unit/               # Unit tests per package
│   └── integration/        # Cross-package tests
└── docs/                   # Unified documentation
    ├── packages/           # Per-package docs
    └── guides/             # Usage guides
```
**When to use**: Multiple related packages in single repository
**Benefits**: Shared dependencies, coordinated releases, unified development

### Configuration Files Structure
```
project/
├── pyproject.toml          # Primary configuration
├── .python-version         # Python version (uv managed)
├── .gitignore             # Git ignore patterns
├── .pre-commit-config.yaml # Pre-commit hooks
├── tox.ini                # Multi-environment testing
├── .github/               # GitHub Actions
│   └── workflows/
│       ├── test.yml       # Test workflow
│       └── publish.yml    # PyPI publishing
```

### File Naming Conventions
- **Package names**: `my_package` (underscore, lowercase)
- **Module names**: `core.py`, `utils.py` (lowercase, descriptive)
- **Class names**: `MyClass` (PascalCase)
- **Function names**: `my_function` (lowercase, underscore)
- **Constants**: `MY_CONSTANT` (uppercase, underscore)
- **Test files**: `test_module.py` (prefix with `test_`)

## Common Patterns

### Project Initialization
```bash
# Create new project with standard structure
uv init myproject
cd myproject

# Create library project with src/ layout
uv init --lib mylib
cd mylib

# Initialize in existing directory
cd existing-project
uv init
```
**When to use**: Starting new Python libraries or converting existing projects to uv
**Expected output**: Creates `pyproject.toml`, `README.md`, `src/` directory structure

### Dependency Management
```bash
# Add production dependency
uv add requests

# Add development dependency  
uv add --dev pytest black ruff

# Add optional dependency group
uv add --optional-group docs sphinx

# Install exact version
uv add "fastapi==0.104.1"

# Install with version constraints
uv add "django>=4.0,<5.0"
```
**When to use**: Adding new dependencies to project
**Expected output**: Updates `pyproject.toml` and `uv.lock`, installs packages

### Environment and Execution
```bash
# Run command in project environment
uv run python script.py
uv run pytest
uv run black .

# Run Python REPL in project environment
uv run python

# Sync environment with lock file
uv sync

# Install project in development mode
uv sync --dev
```
**When to use**: Running code, tests, or tools within project environment
**Expected output**: Commands execute in isolated virtual environment

### Workspace Management
```bash
# Create workspace root
mkdir my-workspace
cd my-workspace

# Initialize workspace
uv init --workspace

# Add member projects
mkdir packages/core packages/api
uv init --lib packages/core
uv init packages/api

# Add local dependency between workspace members
cd packages/api
uv add --editable ../core
```
**When to use**: Managing multiple related packages in single repository
**Expected output**: Workspace with shared dependencies and cross-references

### Tool Execution with uvx
```bash
# Run tool temporarily
uvx black --check .
uvx ruff check src/

# Install tool persistently
uvx install black
uvx install --with black-jupyter black

# Run tool from local wheel file
uvx --from ./dist/my-tool-1.0.0-py3-none-any.whl mytool

# Run tool from PyPI with version
uvx --from "mytool==1.0.0" mytool

# List installed tools
uvx list

# Uninstall tool
uvx uninstall black
```
**When to use**: Running Python CLI tools without installing them globally
**Expected output**: Tools execute in isolated environments

## Implementation Details

### Setting Up New Library Project
1. Create project structure:
```bash
uv init --lib myawesome-lib
cd myawesome-lib
```

2. Configure `pyproject.toml` for PyPI publishing:
```toml
[project]
name = "myawesome-lib"
version = "0.1.0"
description = "An awesome Python library"
authors = [
    {name = "Your Name", email = "you@example.com"}
]
readme = "README.md"
license = {file = "LICENSE"}
requires-python = ">=3.8"
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]
dependencies = []

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "black>=23.0",
    "ruff>=0.1.0",
    "mypy>=1.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[dependency-groups]
dev = [
    "pytest>=7.0",
    "black>=23.0", 
    "ruff>=0.1.0",
    "mypy>=1.0",
]
```

3. Add development dependencies:
```bash
uv add --dev pytest black ruff mypy
```

**Validation**: Project structure contains `src/myawesome_lib/`, `pyproject.toml`, `uv.lock`, `.venv/` directory

### Creating CLI Tools for uvx
1. Create project with CLI entry point:
```bash
uv init my-cli-tool
cd my-cli-tool
```

2. Add CLI script in `src/my_cli_tool/main.py`:
```python
#!/usr/bin/env python3
"""CLI tool entry point."""
import argparse
import sys


def main():
    """Main CLI function."""
    parser = argparse.ArgumentParser(description="My CLI tool")
    parser.add_argument("input", help="Input value")
    parser.add_argument("--verbose", "-v", action="store_true", help="Verbose output")
    
    args = parser.parse_args()
    
    if args.verbose:
        print(f"Processing: {args.input}")
    
    # Your CLI logic here
    print(f"Result: {args.input.upper()}")
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

3. Configure entry points in `pyproject.toml`:
```toml
[project]
name = "my-cli-tool"
version = "0.1.0"
description = "My awesome CLI tool"
authors = [
    {name = "Your Name", email = "you@example.com"}
]
readme = "README.md"
requires-python = ">=3.8"
dependencies = []

[project.scripts]
mycli = "my_cli_tool.main:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/my_cli_tool"]
```

4. Install and test locally:
```bash
uv sync
uv run mycli "test input" --verbose
```

**Validation**: CLI script runs with `uv run mycli` and shows expected output

### Publishing to PyPI
1. Install build tools:
```bash
uv add --dev build twine
```

2. Build distribution packages:
```bash
uv run python -m build
```

3. Test upload to TestPyPI:
```bash
uv run twine upload --repository testpypi dist/*
```

4. Upload to PyPI:
```bash
uv run twine upload dist/*
```

**Validation**: Package appears on PyPI and can be installed with `pip install myawesome-lib`

### Workspace Configuration
1. Create workspace structure:
```bash
mkdir monorepo && cd monorepo
uv init --workspace
mkdir packages/core packages/utils packages/cli
```

2. Initialize member projects:
```bash
uv init --lib packages/core
uv init --lib packages/utils  
uv init packages/cli
```

3. Configure workspace root `pyproject.toml`:
```toml
[tool.uv.workspace]
members = ["packages/*"]

[tool.uv.sources]
core = { workspace = true }
utils = { workspace = true }
```

4. Add cross-dependencies:
```bash
cd packages/cli
uv add --editable ../core --editable ../utils
```

**Validation**: `uv sync` resolves all workspace dependencies correctly

### Advanced Dependency Resolution
1. Lock specific platforms:
```bash
uv lock --resolution-strategy highest
uv lock --resolution-strategy lowest-direct
```

2. Update specific dependency:
```bash
uv lock --upgrade-package requests
```

3. Generate requirements.txt:
```bash
uv export --format requirements-txt > requirements.txt
uv export --dev --format requirements-txt > requirements-dev.txt
```

**Validation**: Lock file contains expected dependency versions and platform markers

## Validation Methods

### Project Health Checks
- **Structure**: `ls -la` shows `pyproject.toml`, `uv.lock`, `src/` directory
- **Dependencies**: `uv tree` displays dependency graph without conflicts  
- **Environment**: `uv run python -c "import sys; print(sys.path)"` shows project paths
- **Lock file**: `uv lock --check` verifies lock file is up-to-date
- **Virtual environment**: `.venv/` directory exists and contains Python installation

### Publishing Readiness
- **Build**: `uv run python -m build` creates `dist/` with `.whl` and `.tar.gz`
- **Package check**: `uv run twine check dist/*` validates package metadata
- **Local install**: `pip install dist/*.whl` in clean environment succeeds
- **Import test**: `python -c "import mypackage"` works after installation

### Tool Execution Validation  
- **uvx list**: Shows installed tools and their versions
- **Tool runs**: `uvx tool --version` returns expected version number
- **Isolation**: Tools don't conflict with project dependencies
- **CLI scripts**: `uvx --from ./dist/package.whl command` executes CLI tools
- **Entry points**: Published tools work with `uvx install package-name`

### Common Error Patterns
- **Import error**: Check if package installed with `uv sync` or `uv add`
- **Lock conflicts**: Run `uv lock --refresh` to regenerate lock file
- **Missing tools**: Install with `uv add --dev tool-name` for project tools
- **Workspace issues**: Verify `members` list in root `pyproject.toml`

## Troubleshooting

### Dependency Resolution Failures
**Symptoms**: `uv add` fails with "No solution found" error
**Cause**: Incompatible version constraints between dependencies
**Fix**: 
```bash
uv add package --resolution-strategy lowest-direct
# OR
uv add "package>=1.0,<2.0" --resolution-strategy highest
```

### Lock File Out of Sync
**Symptoms**: `uv sync` shows "lock file is out of date" warning  
**Cause**: `pyproject.toml` changed but lock file not updated
**Fix**: 
```bash
uv lock
uv sync
```

### Virtual Environment Issues
**Symptoms**: Import errors when running `uv run python`
**Cause**: Virtual environment corrupted or dependencies not installed
**Fix**:
```bash
rm -rf .venv
uv sync --reinstall
```

### Publishing Authentication Errors
**Symptoms**: `twine upload` fails with 403 Forbidden
**Cause**: Missing or incorrect PyPI API token
**Fix**:
```bash
# Set up API token
export TWINE_USERNAME=__token__
export TWINE_PASSWORD=pypi-your-api-token-here
uv run twine upload dist/*
```

### Workspace Member Not Found
**Symptoms**: `uv add --editable ../package` fails with "package not found"
**Cause**: Package not listed in workspace members or incorrect path
**Fix**:
```toml
# In workspace root pyproject.toml
[tool.uv.workspace]
members = ["packages/*", "tools/*"]
```

### Tool Installation Conflicts
**Symptoms**: `uvx install` fails with dependency conflicts
**Cause**: Tool requirements conflict with existing installations
**Fix**:
```bash
uvx uninstall conflicting-tool
uvx install --force new-tool
# OR
uvx install --with dependency==version new-tool
```

### CLI Script Entry Points Not Working
**Symptoms**: `uv run command` fails with "command not found"
**Cause**: Missing `[project.scripts]` configuration or build system
**Fix**:
```toml
# Add to pyproject.toml
[project.scripts]
mycommand = "package.module:function"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/package"]
```

### Package Not Found for uvx
**Symptoms**: `uvx --from package command` fails with package not found
**Cause**: Package not built or wrong file path
**Fix**:
```bash
uv run python -m build  # Build first
uvx --from ./dist/package-version-py3-none-any.whl command
```

## Authoritative References

- [uv Documentation](https://docs.astral.sh/uv/) - Official uv documentation and guides
- [Python Packaging User Guide](https://packaging.python.org/) - Official Python packaging standards
- [PyPI Publishing Tutorial](https://packaging.python.org/en/latest/tutorials/packaging-projects/) - Step-by-step PyPI publishing guide
- [PEP 621](https://peps.python.org/pep-0621/) - Storing project metadata in pyproject.toml
- [PEP 517](https://peps.python.org/pep-0517/) - Build system interface specification
- [Python Package Index](https://pypi.org/) - Official package repository
- [TestPyPI](https://test.pypi.org/) - Test environment for package publishing
