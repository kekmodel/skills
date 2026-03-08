---
name: uv-env-setup
description: |
  Automate Python virtual environment setup using uv. Handles project detection (pyproject.toml, requirements.txt, setup.py, empty directory), venv creation, dependency installation, and dev tooling setup. Trigger this skill when the user asks to: set up a Python environment or project, create a virtual environment (venv), install Python dependencies, configure uv, troubleshoot uv errors (sync failures, resolution errors), or handle uninstallable packages (flash-attn, megatron-core) for code review. Also trigger for Korean phrases like "환경설정", "가상환경", "개발환경 세팅", "파이썬 환경 잡아줘", "의존성 설치". Even if the user just says "set up this project" or "install deps" in a Python context, use this skill. Do NOT trigger for: non-Python projects (React, Node.js), writing tests, editing config files, Docker setup, CI/CD pipelines, or debugging Python code.
---

# uv Virtual Environment Setup

Automates Python development environment setup using `uv` — the extremely fast Python package and project manager written in Rust.

## Key Concepts

uv has **two modes**:

| Mode | When | Commands | Lockfile |
|------|------|----------|----------|
| **Project mode** | `pyproject.toml` with `[project]` exists or can be created | `uv add`, `uv sync`, `uv run`, `uv lock` | Yes (`uv.lock`) |
| **pip-compatible mode** | Legacy projects with `requirements.txt`, `setup.py`, etc. | `uv pip install`, `uv pip compile` | No |

Project mode is preferred — it provides reproducible builds via lockfile. Use pip-compatible mode only for legacy projects that can't migrate.

**uv automatically finds `.venv`** in the working directory or parent directories — explicit activation is not needed when using `uv run`.

## Setup Steps

### Step 1: Check uv Installation

```bash
uv --version
```

If not found, install it:
- **Windows**: `powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"`
- **macOS/Linux**: `curl -LsSf https://astral.sh/uv/install.sh | sh`

### Step 2: Detect Project Type

Scan the project root and classify:

**Scenario A — uv-managed project:**
- `pyproject.toml` with `[project]` section exists
- → Use **Project mode**

**Scenario B — Legacy project:**
- Has `requirements.txt`, `setup.py`, `setup.cfg`, or `Pipfile` but no `pyproject.toml`
- → Ask user: migrate (`uv init` + `uv add -r requirements.txt`) or stay in pip-compatible mode?

**Scenario C — New/empty project:**
- No dependency files found
- → Use `uv init` to create a new project

**Python version**: Use `.python-version` file or `requires-python` from `pyproject.toml`. If neither exists, default to **Python 3.12**.

Report the detected scenario to the user before proceeding.

### Step 3: Create Virtual Environment

Check if `.venv` already exists:
- **If exists**: Ask the user whether to **recreate** or **reuse**. (`uv venv` overwrites without warning.)
- **If not**: Proceed to create.

For **new projects** (Scenario C), first run `uv init` — this creates `pyproject.toml`, `.python-version`, `.gitignore`, `README.md`, and `main.py`.

Then create the venv:
```bash
uv venv --python <version> --seed
```
- `--python <version>`: From `.python-version` or default `3.12`. uv **auto-downloads** Python if not installed.
- `--seed`: Installs `pip` into the venv — required for Jupyter `%pip install` compatibility.

### Step 4: Install Dependencies

#### Project mode

```bash
uv sync
```
This reads `uv.lock` (creating it if needed) and installs all dependencies including the `dev` group.

To migrate from `requirements.txt`:
```bash
uv add -r requirements.txt
uv add --dev -r requirements-dev.txt   # if exists
```

#### pip-compatible mode

```bash
uv pip install -r requirements.txt                          # requirements.txt
uv pip install -e .                                         # setup.py / setup.cfg
uv pip install -e ".[dev]" 2>/dev/null || uv pip install -e .  # pyproject.toml (setuptools)
```

Also install from `requirements-dev.txt` or `requirements/dev.txt` if they exist.

### Step 5: Install Essential Dev Packages

**Project mode:**
```bash
uv add --dev ipykernel ipywidgets pytest pytest-cov ruff pyright debugpy pre-commit
```

**pip-compatible mode:**
```bash
uv pip install ipykernel ipywidgets pytest pytest-cov ruff pyright debugpy pre-commit
```

| Package | Purpose |
|---------|---------|
| ipykernel | Jupyter kernel — run notebooks in the venv |
| ipywidgets | Interactive Jupyter widgets |
| pytest | Testing framework |
| pytest-cov | Coverage reporting for pytest |
| ruff | Fast linter + formatter (replaces Flake8, Black, isort) |
| pyright | Static type checker (powers VS Code Pylance) |
| debugpy | VS Code debug protocol |
| pre-commit | Git hooks — auto-runs linting/formatting before commits |

Skip packages already in the project's dependencies.

### Step 6: Configure Session

After setup, **remember the project path and mode** for all subsequent operations.

#### Project mode — use `uv run` for everything

```bash
uv run python script.py       # Run scripts
uv run pytest                  # Run tools
uv add <package>               # Install packages (updates pyproject.toml + uv.lock)
uv add --dev <package>         # Install dev packages
```

#### pip-compatible mode — reference the venv explicitly

```bash
.venv/Scripts/python script.py  # Windows
.venv/bin/python script.py      # macOS/Linux
uv pip install <package>        # Install packages (uv auto-detects .venv)
```

### Completion Summary

Tell the user:
- Python version in use
- Project mode (project vs pip-compatible)
- venv location (absolute path)
- How subsequent commands work (`uv run` vs explicit venv path)
- Number of packages installed

## Quick Reference

| Task | Project Mode | pip-compatible Mode |
|------|-------------|-------------------|
| Run script | `uv run python script.py` | `.venv/bin/python script.py` |
| Install package | `uv add <pkg>` | `uv pip install <pkg>` |
| Install dev package | `uv add --dev <pkg>` | `uv pip install <pkg>` |
| Run tool | `uv run pytest` | `.venv/bin/pytest` |
| Sync environment | `uv sync` | N/A |
| Update all deps | `uv lock --upgrade` | N/A |
| Run with temp dep | `uv run --with <pkg> python script.py` | N/A |

## Handling Installation Failures

When a package fails to install, read `references/troubleshooting.md` for the full diagnostic flow. Key scenarios:

- **PyTorch/CUDA**: uv has first-class support — use `[tool.uv.sources]` with environment markers or `--extra-index-url` to auto-select CPU vs CUDA builds
- **Fundamentally impossible** (e.g., `flash-attn`, `megatron-core`, `apex`): These require NVIDIA GPU + CUDA and cannot be installed on CPU machines. Ask the user's intent — most commonly they want **code review, not execution**
- **Code Review Mode**: Clone source to `libs/`, add to pyright `extraPaths`, set `reportMissingImports = "warning"`. This enables Go to Definition, type info, and autocomplete without installing the package. See `references/troubleshooting.md` for details.
- **Platform incompatibility**: Use environment markers (e.g., `; sys_platform == 'linux'`)
- **Build failures**: Install platform build tools (VS Build Tools / xcode-select / build-essential)

Never silently skip a failed package — always report what failed, why, and what was done about it.

## Additional References

- **Flag reference**: Read `references/flags.md` for detailed flags of `uv venv`, `uv add`, `uv sync`, `uv run`, `uv pip install`, `uv lock`
- **Troubleshooting**: Read `references/troubleshooting.md` for installation failure diagnosis, PyTorch/CUDA setup, Code Review Mode, and platform incompatibility fixes

## Important Reminders

- Prefer **Project mode** whenever possible — lockfile-based reproducibility
- In Project mode, **always use `uv add`** not `uv pip install` — keeps lockfile in sync
- uv **auto-downloads Python** — no need to install Python separately
- `uv run` syncs the environment before every execution — safest way to run code
- `--seed` flag + `ipykernel` = Jupyter notebooks work correctly in the venv
