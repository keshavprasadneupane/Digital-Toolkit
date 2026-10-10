# `uv` Python Package Manager — Complete Cheat Sheet

`uv` is a fast Python package and project manager written in Rust. It combines Python version management, virtual environment creation, dependency resolution, project management, script execution, and CLI tool management in one tool.

It provides workflows that overlap with `pip`, `pip-tools`, `virtualenv`, `pipx`, `poetry`, and `pyenv`, although it does not reproduce every feature of those tools.

---

## 🚀 Quick Reference: Initialize Without a `src/` Layout

**Use `--no-package` when you want a project without Python package-building configuration or an automatically generated `src/` package structure.** This is useful for reinforcement learning (RL), machine learning (ML), experiments, notebooks, and standalone applications.

### Initialize the current directory with Python 3.12

```bash
uv init --no-package --python 3.12
```

### Create a new project directory

```bash
uv init --no-package --python 3.12 my_project
```

### Initialize a minimal project

```bash
uv init --bare --python 3.12
```

`--bare` minimizes generated project files and metadata. Unlike `--no-package`, it is intended for minimal initialization and can be combined with package-related options.

### Understand the flags

| Flag | Purpose |
|---|---|
| `--no-package` | Disable package-building configuration. |
| `--python 3.12` | Select Python 3.12 when initializing the project. |
| `--bare` | Generate minimal project metadata and files. |
| `--app` | Explicitly select an application project template. |
| `--package` | Enable package-building configuration. |

For a flat application project, use:

```bash
uv init --app --no-package --python 3.12
```

The `--no-package` option is the important part when you want to avoid package-building configuration. See the [official uv project initialization documentation](https://docs.astral.sh/uv/concepts/projects/init/).

### Example RL project structure

```text
my_project/
├── .python-version
├── pyproject.toml
├── uv.lock
├── README.md
├── train.py
├── evaluate.py
├── agents/
├── environments/
├── experiments/
└── models/
```

`uv.lock` is normally created when the project dependencies are resolved. The directories are examples, not requirements imposed by `uv`.

**Important:** A flat project layout does not mean the project has no environment management. You can still use `pyproject.toml`, `uv.lock`, `.venv`, `uv add`, `uv sync`, and `uv run` without packaging your application.

---

## 1. Project Initialization and Setup

| Command | Description |
|---|---|
| `uv init` | Initialize a project in the current directory. |
| `uv init --no-package` | Initialize without package-building configuration. |
| `uv init --no-package --python 3.12` | Initialize a flat project using Python 3.12. |
| `uv init --bare` | Generate minimal project files and metadata. |
| `uv init --app --no-package` | Initialize an unpackaged application project. |
| `uv init --package` | Initialize a package-oriented project. |
| `uv init --lib` | Initialize a library project. |
| `uv init my_project --no-package` | Create a new unpackaged project directory. |
| `uv init --script main.py` | Initialize a standalone Python script with inline dependency metadata. |
| `uv init --help` | Display initialization options. |

### Typical project initialization

```bash
mkdir my_project
cd my_project

uv init --no-package --python 3.12

# Add dependencies
uv add numpy pandas matplotlib

# Run the application
uv run python main.py
```

---

## 2. Managing Python Versions (`uv python`)

`uv` can download and manage Python versions without requiring a separate version manager.

| Command | Description |
|---|---|
| `uv python install 3.12` | Install a managed Python 3.12 interpreter. |
| `uv python install 3.11 3.12 3.13` | Install multiple Python versions. |
| `uv python list` | List available and installed Python versions. |
| `uv python find` | Find a suitable Python interpreter. |
| `uv python find 3.12` | Find a Python 3.12 interpreter. |
| `uv python pin 3.12` | Pin the project to Python 3.12 in `.python-version`. |
| `uv python uninstall 3.10` | Uninstall a managed Python version. |
| `uv python upgrade 3.12` | Upgrade a managed Python 3.12 installation where supported. |
| `uv python dir` | Display the managed Python installation directory. |

### Example

```bash
# Install Python
uv python install 3.12

# Pin the project version
uv python pin 3.12

# Create the environment and synchronize dependencies
uv sync
```

**Note:** `uv` can also discover compatible system Python installations. You do not necessarily need to install a separate managed interpreter.

---

## 3. Virtual Environment Management (`uv venv`)

`uv` can create virtual environments directly without requiring manual `virtualenv` or `venv` commands.

| Command | Description |
|---|---|
| `uv venv` | Create a virtual environment at `.venv`. |
| `uv venv --python 3.12` | Create `.venv` using Python 3.12. |
| `uv venv custom_env` | Create an environment at `custom_env`. |
| `uv venv --seed` | Seed the environment with tools such as `pip`. |
| `source .venv/bin/activate` | Activate the environment on Linux/macOS. |
| `source .venv/bin/activate.fish` | Activate the environment in Fish shell. |
| `.venv\Scripts\Activate.ps1` | Activate the environment in Windows PowerShell. |
| `deactivate` | Deactivate an activated environment. |

### Recommended workflow

```bash
uv init --no-package --python 3.12

uv add numpy pandas

uv run python main.py
```

You usually do not need to run `uv venv` manually for a project. Commands such as `uv sync` create the project environment when needed, and `uv run` uses the project's environment.

Manual activation is optional.

---

## 4. Dependency and Package Management (`uv add` / `uv remove`)

Project dependencies are declared in `pyproject.toml` and resolved into `uv.lock`.

| Command | Description |
|---|---|
| `uv add pandas numpy` | Add project dependencies. |
| `uv add "requests>=2.28.0"` | Add a dependency with a version constraint. |
| `uv add --dev pytest ruff` | Add development dependencies. |
| `uv add --optional gui pyqt6` | Add an optional dependency under an extra. |
| `uv remove pandas` | Remove a project dependency. |
| `uv remove --dev pytest` | Remove a development dependency. |
| `uv sync` | Synchronize the project environment with the lockfile and selected dependency groups. |
| `uv sync --locked` | Synchronize without allowing the lockfile to change. |
| `uv sync --frozen` | Synchronize using the existing lockfile without checking whether it needs updating. |
| `uv sync --no-dev` | Synchronize without development dependencies. |
| `uv lock` | Resolve dependencies and update the lockfile. |
| `uv lock --upgrade` | Upgrade dependencies within the allowed constraints. |
| `uv lock --upgrade-package pandas` | Upgrade a specific dependency within its constraints. |
| `uv tree` | Display the project's dependency tree. |

### Example: Set up an RL environment

```bash
uv add gymnasium numpy matplotlib
uv add torch
```

Install `stable-baselines3` if you want its preimplemented RL algorithms:

```bash
uv add stable-baselines3
```

For ML projects, keep expensive or hardware-specific dependencies in mind. For example, PyTorch installation options can depend on whether you need CPU execution, CUDA, or a specific platform configuration.

### `uv.lock` vs. `pyproject.toml`

- `pyproject.toml` declares project metadata and dependency requirements.
- `uv.lock` records the resolved dependency versions and related resolution information.
- `.venv` contains the project's installed environment.

Commit `pyproject.toml` and `uv.lock` to version control when reproducible dependency installation matters. Do not commit `.venv`.

---

## 5. Running Code and Commands (`uv run`)

`uv run` executes commands in the project's environment and normally checks that the environment is synchronized before execution.

| Command | Description |
|---|---|
| `uv run train.py` | Run a Python script. |
| `uv run python evaluate.py` | Run a Python file through the selected interpreter. |
| `uv run pytest` | Run tests. |
| `uv run ruff check .` | Run Ruff if it is available in the project environment. |
| `uv run python -c "print('Hello')"` | Execute inline Python code. |
| `uv run --python 3.11 script.py` | Run a script using Python 3.11. |
| `uv run --with requests script.py` | Run with a temporary dependency. |
| `uv run --with duckdb python -c "import duckdb; print(duckdb.__version__)"` | Execute an inline command with DuckDB temporarily available. |
| `uv run --no-sync script.py` | Run without synchronizing the project environment first. |

### Example

```bash
uv run python train.py
uv run python evaluate.py
uv run pytest
```

You do not need to activate `.venv` manually for these commands.

**Important:** `uv run` is not limited to Python scripts. It can execute other commands using the project's environment, including command-line tools installed as dependencies.

---

## 6. Standalone Script Dependencies (PEP 723)

`uv` supports declaring dependencies directly inside individual Python scripts. This is useful for small utilities, isolated experiments, and scripts that do not need a full project configuration.

### Add dependencies to a script

```bash
uv add --script script.py requests
```

### Example script

```python
# /// script
# dependencies = [
#     "requests>=2.31.0",
# ]
# ///

import requests

response = requests.get("https://httpbin.org/ip", timeout=10)
response.raise_for_status()

print(response.json())
```

Run it with:

```bash
uv run script.py
```

| Command | Description |
|---|---|
| `uv init --script main.py` | Initialize a script with dependency metadata. |
| `uv add --script script.py requests` | Add a script dependency. |
| `uv remove --script script.py requests` | Remove a script dependency. |
| `uv run script.py` | Run the script with its declared dependencies. |
| `uv run --script script.py` | Explicitly run in script mode. |

Standalone script dependencies are separate from the project's normal dependency list. This is convenient when a utility should be reproducible without adding its dependencies to an entire application.

---

## 7. Global CLI Tool Management (`uv tool`)

`uv tool` manages command-line utilities in isolated environments, similar to `pipx`.

| Command | Description |
|---|---|
| `uv tool install ruff` | Install Ruff as a persistent CLI tool. |
| `uv tool install black` | Install Black. |
| `uv tool run ruff check .` | Run Ruff in an isolated tool environment. |
| `uvx ruff check .` | Short form for `uv tool run`. |
| `uv tool list` | List installed tools. |
| `uv tool upgrade --all` | Upgrade installed tools. |
| `uv tool upgrade ruff` | Upgrade Ruff. |
| `uv tool uninstall ruff` | Uninstall Ruff. |
| `uv tool dir` | Display the tool installation directory. |
| `uv tool update-shell` | Update shell configuration for tool executables where needed. |

### When to use `uvx` vs. `uv add`

```bash
# Run a tool without adding it to the project
uvx ruff check .

# Add Ruff to the project's development dependencies
uv add --dev ruff

# Run the project-managed Ruff installation
uv run ruff check .
```

Use `uvx` for occasional tools and `uv add --dev` for tools that should be available consistently to everyone working on the project.

---

## 8. Legacy `pip` Replacement Interface (`uv pip`)

Use this interface when working with traditional virtual environments and `requirements.txt` workflows.

| Command | Description |
|---|---|
| `uv pip install -r requirements.txt` | Install requirements into the detected environment. |
| `uv pip install pandas numpy` | Install packages into the detected environment. |
| `uv pip install -e .` | Install the current project in editable mode. |
| `uv pip compile requirements.in -o requirements.txt` | Compile dependency requirements. |
| `uv pip compile pyproject.toml -o requirements.txt` | Compile requirements from project metadata where supported. |
| `uv pip sync requirements.txt` | Synchronize an environment to match a requirements file. |
| `uv pip freeze` | List installed packages in requirements format. |
| `uv pip list` | List installed packages. |
| `uv pip show pandas` | Show information about an installed package. |
| `uv pip check` | Check for incompatible installed dependencies. |
| `uv pip uninstall pandas` | Uninstall a package. |
| `uv pip tree` | Display the installed environment's dependency tree. |

### Example: Traditional environment

```bash
uv venv
source .venv/bin/activate

uv pip install -r requirements.txt
uv pip freeze
```

The `uv pip` interface manages packages in an environment; it does not replace the project-oriented workflow of `uv add`, `uv lock`, and `uv sync`.

---

## 9. Cache and Maintenance (`uv cache`)

`uv` uses a global cache to reduce repeated downloads and builds. This can make dependency installation significantly faster across projects.

| Command | Description |
|---|---|
| `uv cache dir` | Display the cache directory. |
| `uv cache clean` | Clear cached package artifacts. |
| `uv cache clean pandas` | Clear cached data associated with a particular package. |
| `uv cache prune` | Remove outdated cache entries and centralized project environments. |
| `uv self update` | Update the `uv` executable where supported. |
| `uv --version` | Display the installed `uv` version. |
| `uv help` | Display general help. |
| `uv help sync` | Display help for a specific command. |

**Important:** Clearing the cache is not the same as deleting a project's `.venv`. Cache cleanup can also remove data that would otherwise speed up future installations.

---

## 10. Useful Project and Environment Commands

| Command | Description |
|---|---|
| `uv version` | Display the version of the project itself, where configured. |
| `uv build` | Build project distribution archives when packaging is configured. |
| `uv publish` | Publish built distributions to a package index. |
| `uv export --format requirements-txt` | Export the lockfile into a requirements file. |
| `uv sync --all-extras` | Synchronize with all optional dependency extras. |
| `uv sync --all-groups` | Synchronize with all dependency groups. |
| `uv lock --check` | Check whether the lockfile is up to date with project metadata. |

Some commands are relevant only when the project has the corresponding configuration. For example, `uv build` is intended for building distributions, while an unpackaged project can still use `uv run` and `uv sync` without building itself.

---

## 11. Common Workflows

### A. Start a new flat-layout project

```bash
uv init --no-package --python 3.12 my_project
cd my_project

uv add numpy pandas matplotlib
uv add --dev pytest ruff

uv run python main.py
```

### B. Clone an existing project

```bash
git clone <repository_url>
cd <project_directory>

uv sync --locked
uv run python main.py
```

Use `uv sync --locked` when you expect the committed lockfile to be current and do not want it modified automatically.

### C. Add a dependency

```bash
uv add scikit-learn
```

### D. Update dependencies

```bash
# Update the lockfile within existing constraints
uv lock --upgrade

# Synchronize the environment
uv sync
```

### E. Run tests and linting

```bash
uv run pytest
uv run ruff check .
```

### F. Rebuild the project environment

If the environment is corrupted, you can remove the local `.venv` and recreate it:

```bash
rm -rf .venv
uv sync
```

This removes the project's virtual environment, not its source files or global cache. Ensure that `.venv` is the intended environment directory before running the deletion command.

---

## 12. Best Practices

1. **Use `uv run` by default.** It avoids unnecessary manual activation and runs commands in the project's environment.
2. **Use `uv add` for project dependencies.** Keep dependency declarations in `pyproject.toml`.
3. **Commit `uv.lock`.** It helps reproduce dependency resolutions across development machines.
4. **Pin Python when necessary.** Use `uv python pin 3.12` to record a project-level Python version.
5. **Use `uvx` for occasional CLI tools.** Use project development dependencies for tools that should be reproducible across a team.
6. **Use temporary dependencies for experiments.** Try `uv run --with duckdb script.py` before adding a dependency permanently.
7. **Choose your project layout deliberately.** Use `--no-package` for a flat project when package-building features are unnecessary; use a package-oriented layout when your application needs distribution or installation as a package.
8. **Keep RL experiments organized.** Separate training scripts, environments, checkpoints, evaluation code, and experiment results as appropriate.
9. **Avoid unnecessary cache deletion.** The cache is one of the reasons repeated installations can be fast.
10. **Understand synchronization flags.** `--locked` checks lockfile consistency; `--frozen` uses the lockfile without checking whether project metadata would require an update.
11. **Keep dependency groups intentional.** Separate runtime dependencies from development-only tools such as test runners and linters.
12. **Check the installed CLI version.** If a command or option differs from this reference, use `uv --version` and `uv help <command>`.

---

## 13. Official Documentation

- [uv Documentation](https://docs.astral.sh/uv/)
- [Project Initialization](https://docs.astral.sh/uv/concepts/projects/init/)
- [Managing Dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/)
- [Python Version Management](https://docs.astral.sh/uv/guides/install-python/)
- [Running Commands and Scripts](https://docs.astral.sh/uv/guides/projects/)
- [Standalone Scripts](https://docs.astral.sh/uv/guides/scripts/)
- [CLI Tools](https://docs.astral.sh/uv/guides/tools/)
- [The `uv pip` Interface](https://docs.astral.sh/uv/pip/)
- [Command Reference](https://docs.astral.sh/uv/reference/cli/)

---

## Quick Reference

The commands most frequently used in everyday development:

```bash
# Initialize a flat project
uv init --no-package --python 3.12

# Add dependencies
uv add numpy pandas

# Add development dependencies
uv add --dev pytest ruff

# Synchronize dependencies
uv sync

# Run a script
uv run python main.py

# Run tests
uv run pytest

# Run a temporary tool
uvx ruff check .

# Pin Python
uv python pin 3.12

# Inspect dependencies
uv tree

# Update dependencies
uv lock --upgrade
uv sync
```

**Remember:** `uv` brings project management, dependency resolution, Python version management, environment creation, and command execution into one workflow. Choose between a flat application and an installable package based on your project's needs, then use `uv add`, `uv sync`, and `uv run` for everyday development.