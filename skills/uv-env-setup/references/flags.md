# uv Flag Reference

## `uv venv`
| Flag | Purpose |
|------|---------|
| `--python <version>` | Specify Python version (e.g., `3.12`, `3.11.5`, `pypy@3.10`). Auto-downloads if not installed. |
| `--seed` | Install `pip` into the venv. Required for Jupyter `%pip install` compatibility. |
| `--relocatable` | Make the venv relocatable (relative paths instead of absolute). |

## `uv add`
| Flag | Purpose |
|------|---------|
| `--dev` | Add to the `dev` dependency group. Alias for `--group dev`. |
| `--group <name>` | Add to a named dependency group (e.g., `--group lint`, `--group test`). |
| `--optional <extra>` | Add as an optional dependency under `[project.optional-dependencies.<extra>]`. |
| `-r <file>` | Import dependencies from a requirements file (e.g., `uv add -r requirements.txt`). |

## `uv sync`
| Flag | Purpose |
|------|---------|
| `--no-dev` | Exclude the `dev` dependency group. |
| `--only-dev` | Sync only the `dev` group. |
| `--group <name>` | Include a specific dependency group. |
| `--no-group <name>` | Exclude a specific dependency group. |
| `--all-groups` | Include all dependency groups. |
| `--frozen` | Use lockfile as-is without updating it. Error if lockfile is missing. |
| `--locked` | Assert lockfile is up-to-date without modifying it. Error if out of sync. |
| `--no-install-project` | Install only dependencies, not the project itself. Useful for Docker layer caching. |
| `--no-editable` | Install the project non-editably. Useful for production Docker images. |

## `uv run`
| Flag | Purpose |
|------|---------|
| `--with <pkg>` | Temporarily add a package for this run only. Does not modify lockfile. |
| `--no-project` | Run without project context (ignore `pyproject.toml`). |
| `--frozen` | Don't update lockfile before running. |
| `--locked` | Assert lockfile is up-to-date before running. |

## `uv pip install`
| Flag | Purpose |
|------|---------|
| `--system` | Install into system Python instead of venv. **Use only in CI/Docker — never in local development.** |
| `--python <path>` | Target a specific Python interpreter. |
| `--target <dir>` | Install into a specific directory instead of the environment. |
| `-e .` | Install current project in editable/development mode. |
| `--all-extras` | Install all optional dependency groups from `pyproject.toml`. |
| `--extra-index-url <url>` | Additional package index (e.g., PyTorch wheels). |

## `uv lock`
| Flag | Purpose |
|------|---------|
| `--upgrade` | Upgrade all packages to latest versions within constraints. |
| `--upgrade-package <pkg>` | Upgrade a specific package (e.g., `--upgrade-package flask==3.0`). |

## Key Distinctions

- `--frozen` vs `--locked`: `--frozen` silently uses the existing lockfile; `--locked` checks that it's up-to-date and errors if not. Use `--locked` in CI for safety, `--frozen` when you explicitly want to skip resolution.
- `--system`: Never use in local development — it modifies the system Python. Designed for CI/Docker only.
- `--dev` is an alias for `--group dev`. The `dev` group is synced by default.
