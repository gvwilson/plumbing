# Packages and Virtual Environments

## The Problem

-   Python ships with a standard library, but most real work needs third-party packages
-   Installing packages globally causes [dependency conflicts](g:dependency_conflict):
    -   Project A needs `requests==2.28`; project B needs `requests==2.31`
    -   Only one version can live in the global `site-packages` directory at a time
-   Solution: give each project its own isolated Python environment

## `uv`

-   [`uv`][uv] is a fast Python package and project manager written in Rust
-   Replaces `pip`, `pip-tools`, `virtualenv`, and parts of `conda` with a single tool
-   Install once per machine:

```{data-file="install_uv.sh"}
curl -LsSf https://astral.sh/uv/install.sh | sh
```

-   Verify:

```{data-file="uv_version.sh"}
uv --version
```
```{data-file="uv_version.out"}
uv 0.4.18
```

## Creating a Project

-   `uv init` creates a new project in the current directory (or a named subdirectory)

```{data-file="uv_init.sh"}
uv init birdcount
cd birdcount
ls -a
```
```{data-file="uv_init.out"}
.
..
.python-version
README.md
hello.py
pyproject.toml
```

-   `uv init` also runs `git init` and creates a `.gitignore`
-   `.python-version` pins the Python version for this project

## `pyproject.toml`

-   `pyproject.toml` is the standard way to describe a Python project
-   `uv init` generates a minimal version:

```{data-file="pyproject_initial.toml"}
[project]
name = "birdcount"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
```

-   `[project]` table holds metadata
-   `dependencies` lists the packages this project needs (empty for now)

## Adding Dependencies

-   Use `uv add` to add a package:

```{data-file="uv_add.sh"}
uv add requests
```
```{data-file="uv_add.out"}
Resolved 5 packages in 234ms
Prepared 5 packages in 1.23s
Installed 5 packages in 89ms
 + certifi==2024.8.30
 + charset-normalizer==3.3.2
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

-   `uv add` updates `pyproject.toml` and creates (or updates) `uv.lock`

```{data-file="pyproject_with_dep.toml"}
[project]
name = "birdcount"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32.3",
]
```

## The Lock File

-   `uv.lock` records the exact version of every installed package (including transitive dependencies)
-   Commit `uv.lock` to version control so teammates get identical environments
-   `pyproject.toml` says "I need `requests` 2.32 or newer"; `uv.lock` says "use `requests` 2.32.3 exactly"

## Running Code

-   Use `uv run` to execute a script inside the project's environment:

```{data-file="fetch_birds.py"}
import requests

response = requests.get("https://example.com/birds.json")
print(f"status: {response.status_code}")
```

```{data-file="uv_run.sh"}
uv run python fetch_birds.py
```
```{data-file="uv_run.out"}
status: 200
```

-   `uv run` creates the virtual environment in `.venv/` on the first run if it does not exist
-   Never activate the virtual environment manually: `uv run` handles it

## How the Virtual Environment Works

-   `.venv/` contains a minimal Python installation and the project's installed packages
-   `uv run` prepends `.venv/bin` to `PATH` before executing the script
    -   The Python interpreter and installed packages in `.venv/bin` shadow the system versions
-   This is the same mechanism described in the [Shell Variables](@/var/) chapter

```{data-file="show_venv.sh"}
uv run python -c "import sys; print(sys.prefix)"
```
```{data-file="show_venv.out"}
/Users/tut/birdcount/.venv
```

## Reproducing an Environment

-   A new collaborator clones the repository and runs:

```{data-file="uv_sync.sh"}
uv sync
```
```{data-file="uv_sync.out"}
Resolved 5 packages in 12ms
Installed 5 packages in 341ms
 + certifi==2024.8.30
 + charset-normalizer==3.3.2
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

-   `uv sync` reads `uv.lock` and installs exactly those versions
-   No "works on my machine" surprises

## Removing and Upgrading Dependencies

-   Remove a package: `uv remove requests`
-   Upgrade a package to the latest allowed version: `uv add --upgrade requests`
-   Upgrade all packages: `uv lock --upgrade`

<section class="exercise" markdown="1">

## Exercise: First Project

1.  Create a new project called `weather` using `uv init`.

1.  Add `httpx` as a dependency and write a script that fetches any URL and prints its status code.

1.  Open `uv.lock` and find `httpx`.
    How many packages were installed to satisfy that one dependency?

1.  What is in `.venv/bin`?
    Why is there more than just Python there?

</section>
