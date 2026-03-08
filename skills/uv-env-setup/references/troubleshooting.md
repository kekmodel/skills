# Troubleshooting: Installation Failures

When dependency installation fails, follow this diagnostic flow.

## Step 1: Collect Environment Info

```bash
uv --version
python --version
uname -m 2>/dev/null || echo "Windows"
nvidia-smi 2>/dev/null || echo "No NVIDIA GPU detected"
```

## Step 2: Diagnose the Error

| Error Pattern | Cause | Solution |
|--------------|-------|----------|
| `No matching distribution found` | OS/arch/Python incompatible | See Platform Incompatibility |
| `Failed to build` / `subprocess-exited-with-error` | Missing compiler or build deps | See Build Failures |
| `CUDA not available` | No GPU or wrong CUDA version | See PyTorch/CUDA |
| `ResolutionImpossible` / `conflict` | Version conflicts | `uv lock --upgrade` or relax constraints |
| `requires-python` mismatch | Wrong Python version | Adjust Python or find compatible version |

## Step 3: Apply the Fix

### PyTorch / CUDA

uv has first-class PyTorch support. Check GPU availability first, then choose the right index.

**Project mode** — auto-select CPU vs CUDA via environment markers:
```toml
[tool.uv.sources]
torch = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu128", marker = "sys_platform == 'linux'" },
]
torchvision = [
  { index = "pytorch-cpu", marker = "sys_platform != 'linux'" },
  { index = "pytorch-cu128", marker = "sys_platform == 'linux'" },
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[[tool.uv.index]]
name = "pytorch-cu128"
url = "https://download.pytorch.org/whl/cu128"
explicit = true
```

**pip-compatible mode**:
```bash
uv pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cpu     # CPU
uv pip install torch torchvision --extra-index-url https://download.pytorch.org/whl/cu128   # CUDA 12.8
```

### Fundamentally Impossible Installations

Some packages cannot be installed on certain environments due to hard hardware/OS requirements:

**GPU/CUDA-only** (require NVIDIA GPU + CUDA toolkit):
- `flash-attn` — requires CUDA GPU, no CPU/AMD support
- `megatron-core` — requires NVIDIA GPU cluster
- `apex` — NVIDIA-specific mixed precision
- `triton` on Windows — Linux/CUDA only
- `bitsandbytes` on Windows — limited support

**Linux-only**: `uvloop` (uses epoll), packages requiring `/proc`

**Architecture-specific**: Some packages lack ARM/Apple Silicon wheels

**When this happens:**
1. Stop and inform the user of the hard constraint
2. Ask the user's intent — usually they want to **read/review code**, not run it
3. Suggest runtime alternatives if they exist:
   - `flash-attn` → `torch.nn.functional.scaled_dot_product_attention`
   - `apex` → `torch.amp`
   - `bitsandbytes` on Windows → WSL2
4. If user wants code review → set up **Code Review Mode** (below)
5. Document skipped packages in the completion summary

### Code Review Mode

When a package can't be installed but the user needs to read and navigate its source code (Go to Definition, type info, autocomplete), clone the source and configure pyright.

**Step 1: Clone source**
```bash
mkdir -p libs
git clone --depth 1 <repo-url> libs/<package-name>
```

`--depth 1` fetches only the latest commit to save space. Find the repo URL from PyPI or GitHub.

**Step 2: Configure pyright/Pylance**

Pylance resolves imports independently from the runtime Python — it searches `site-packages` first, then `extraPaths`. So even without installing the package, Go to Definition, hover type info, and autocomplete all work.

**Important:** `extraPaths` must point to the **parent** of the package directory. Example: if `libs/flash-attention/` contains `flash_attn/` (the importable package), point to `libs/flash-attention`, NOT `libs/flash-attention/flash_attn`. For `src/` layout repos, use `libs/<repo>/src`.

Add to `pyproject.toml`:
```toml
[tool.pyright]
extraPaths = [
  "libs/flash-attention",
  "libs/Megatron-LM",
]
reportMissingImports = "warning"
```

Setting `reportMissingImports` to `"warning"` (not `"error"`) prevents the editor from being flooded with red underlines for uninstallable packages while still catching real missing imports.

**Step 3: Add to .gitignore**
```bash
echo "libs/" >> .gitignore
```

**What works:** Go to Definition, type info, autocomplete, code structure exploration, `git pull` to update source.

**What doesn't work:** Actually running/importing the package at runtime — `uv run` or `python` will raise ImportError. Native code (CUDA kernels, C extensions) cannot execute.

### Platform Incompatibility

Use environment markers to conditionally include platform-specific packages:
```toml
dependencies = [
  "uvloop>=0.19; sys_platform == 'linux'",
]
```

### Build Failures (Missing Compiler)

- **Windows**: Install Visual Studio Build Tools or use pre-built wheels
- **macOS**: `xcode-select --install`
- **Linux**: `sudo apt install build-essential python3-dev`

### Version Conflicts

```bash
uv lock --upgrade                              # Upgrade all
uv lock --upgrade-package <conflicting-pkg>    # Upgrade specific package
```

## Step 4: Report to User

After any failure, always report:
- **What failed** — which package(s) and the error type
- **Why** — root cause (no GPU, wrong OS, version conflict)
- **What was done** — fix applied, alternative installed, or Code Review Mode set up
- **What's missing** — functionality that won't be available

Never silently skip a failed package.
