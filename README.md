# CongoCC Documentation

This repository holds the source for the [CongoCC](https://github.com/congo-cc/congo-parser-generator)
documentation. The pages are written in [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html)
and built with [Sphinx](https://www.sphinx-doc.org/); the published site is
hosted on [Read the Docs](https://about.readthedocs.com/).

This README is for **contributors editing the documentation locally**. It walks
through setting up a build environment and previewing your changes live in a
browser as you edit.

## Repository layout

```
source/                 Sphinx sources
├── conf.py             Sphinx configuration
├── index.rst           top-level landing page (the three guides)
└── docs/
    ├── reference/      Reference Manual
    ├── userguide/      User Guide
    └── targets/        Target Language Guide
build/                  generated output (not tracked in git)
tools/                  maintenance tools (e.g. settings-extractor)
```

The three guides are wired together through ``toctree`` directives: every page
is listed in the landing page of its section (for example
``source/docs/reference/reference.rst``). When you add a page, add it to the
nearest ``toctree`` or Sphinx will warn that the document is not included.

## Prerequisites

- **Git**
- **Python 3.14** — the version is pinned in `.python-version`
- **[uv](https://docs.astral.sh/uv/)** — used to manage Python and the
  virtual environment

## Set up a build environment

### 1. Install uv

uv is a single self-contained binary. Install it with the official installer:

```sh
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

(Alternatively, `brew install uv` or `pipx install uv`.)

### 2. Install Python 3.14

Let uv install the pinned interpreter:

```sh
uv python install 3.14
```

### 3. Clone the repository

```sh
git clone https://github.com/richcar58/congocc-docs.git
cd congocc-docs
```

### 4. Create and activate a virtual environment

`uv venv` creates a `.venv` using the pinned Python from `.python-version`:

```sh
uv venv

# Activate it:
source .venv/bin/activate        # macOS / Linux
# .venv\Scripts\activate         # Windows (PowerShell/cmd)
```

### 5. Install Sphinx and sphinx-autobuild

Both are declared as dependencies, so installing them into the active
environment is one command:

```sh
uv pip install sphinx sphinx-autobuild
```

(Equivalently, `uv sync` installs the exact, locked versions from
`pyproject.toml` / `uv.lock`.)

## Preview the docs live while editing

Run `sphinx-autobuild` to build the site, serve it, and rebuild automatically
whenever you save a source file:

```sh
sphinx-autobuild source build/html
```

It prints a local URL — by default <http://127.0.0.1:8000> — open it in a
browser. Edit any `.rst` file under `source/`, save, and the page reloads on its
own. Press `Ctrl-C` to stop the server. Add `--open-browser` to open the page
automatically, or `--port N` to use a different port.

> **Tip:** `make livehtml` runs the same command.

If you did not activate the virtual environment in step 4, prefix commands with
`uv run` instead — for example `uv run sphinx-autobuild source build/html`.

## Build the docs once

To produce the HTML without the live server:

```sh
make html
# or, equivalently:
uv run sphinx-build -M html source build
```

The output is written to `build/html/`; open `build/html/index.html`.

## Editing conventions

- Keep the build **warning-free**. A broken cross-reference or a malformed table
  shows up as a Sphinx warning — treat warnings as errors.
- Because CongoCC has no numbered releases, mark a feature's availability with
  an ISO date using the standard directives, e.g. `.. versionadded:: 2026-06-19`
  (these are configured to render as "Added 2026-06-19").
- The Settings Reference is kept in sync with the CongoCC source by a small
  helper — see [`tools/settings-extractor/`](tools/settings-extractor/).

## Publishing

The site builds on Read the Docs from `.readthedocs.yaml`, which installs the
locked dependencies with uv and runs `sphinx-build`. Merging to `main` triggers
a rebuild of the published documentation.

## Acknowledgments

Anthropic's Claude Opus 4.8 was used to generate the code and documentation in this project.