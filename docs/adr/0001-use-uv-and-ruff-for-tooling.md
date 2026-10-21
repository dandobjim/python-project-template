---
status: accepted
date: 2026-10-15
decision-makers: [Daniel Doblas]
consulted: []
informed: []
---

# Use uv, Ruff and mypy as the default tooling of generated projects

## Context and Problem Statement

Every project generated from this template inherits its toolchain: dependency and environment management, linting/formatting and static type checking. Changing it later means touching every generated project through `copier update`, so the choice has a long tail.

Which tools should the template ship by default for (1) dependency management, (2) linting and formatting and (3) type checking?

## Decision Drivers

- **Speed**: local feedback loops and CI minutes (cold installs, pre-commit on every commit).
- **Number of tools to maintain**: every tool is one more config, one more version to bump and one more thing to learn.
- **Cross-platform lockfile**: a single lockfile that resolves the same way on macOS, Linux and Windows and on every supported Python version in the CI matrix.
- **Build backend support**: projects must build a wheel without an extra tool and with standard `pyproject.toml` metadata (PEP 621).
- **Standards and exit cost**: configuration in `pyproject.toml` and standard metadata, so the tool can be replaced without rewriting projects.

## Considered Options

Dependency management:

- uv
- Poetry
- pip-tools (+ venv + a separate build backend)

Linting and formatting:

- Ruff (linter + formatter)
- black + isort + flake8

Type checking:

- mypy
- pyright

## Decision Outcome

Chosen options: **uv** (with the `uv_build` backend), **Ruff** and **mypy** in strict mode, because uv and Ruff are the only options that score well on speed and number of tools at the same time, uv is the only candidate with a universal lockfile *and* Python version management in one binary, and mypy runs from PyPI inside the project environment without a Node.js runtime.

### Consequences

- Good, because one binary (`uv`) replaces pip, venv, pip-tools, pyenv and a build frontend, and one (`ruff`) replaces black, isort, flake8 and several flake8 plugins: fewer configs and fewer Dependabot PRs.
- Good, because `uv.lock` is a universal lockfile, so `uv sync --locked` gives the same resolution on every OS and Python version of the CI matrix.
- Good, because everything is configured in `pyproject.toml` (PEP 621 metadata, PEP 735 dependency groups, `[tool.ruff]`, `[tool.mypy]`).
- Bad, because **two of the three tools depend on a single vendor, Astral**. A change in Astral's priorities, licensing or maintenance affects packaging *and* linting at once. This risk grew when OpenAI announced on 2026-03-19 that it would acquire Astral: both sides have stated the tools stay open source, but their roadmap is now tied to a company whose priorities are not Python packaging.
- Bad, because **uv is still 0.x** (0.11 at the time of writing): its versioning policy allows breaking changes in minor releases, so lockfile format, CLI flags or defaults can change under us. `uv_build` follows the same policy, which is why the template pins it with an upper bound (`uv_build>=0.9,<1.0`).
- Bad, because `uv.lock` is a uv-specific format. Leaving uv requires exporting it (`uv export --format pylock.toml` for the PEP 751 standard, or `requirements.txt`).
- Bad, because `uv_build` only supports pure Python packages: a project that needs C/Rust extensions must switch backend (e.g. hatchling, maturin).
- Bad, because uv defaults differ from pip in subtle ways (for example, the `first-index` index strategy), which surprises people used to pip with private indexes.

### Confirmation

- `Template CI` renders the template on Python 3.11–3.15 and runs `uv lock`, `uv sync --locked`, `pytest` and every pre-commit hook (ruff check, ruff format, mypy) in the generated project; it fails if any of the chosen tools stops working.
- Generated projects contain no configuration for black, isort, flake8, Poetry or pip-tools; reviews of template changes reject reintroducing them without a superseding ADR.
- Dependabot keeps uv, Ruff and mypy versions current, so breaking changes in 0.x releases show up as a failing PR instead of a surprise.

## Pros and Cons of the Options

### uv

Rust-based package and project manager from Astral: resolver, installer, virtualenvs, Python version management, lockfile and build backend (`uv_build`).

- Good, because it is much faster than pip/Poetry for resolution and installation, especially on cold CI caches.
- Good, because `uv.lock` is universal (multi-platform, multi-Python) and `--locked` makes CI fail on drift.
- Good, because it installs and pins Python itself (`.python-version`), removing pyenv from the setup.
- Good, because it uses standard metadata (PEP 621, PEP 735) and can export to PEP 751 `pylock.toml`.
- Neutral, because `uv_build` is new; the template could switch to hatchling with a one-line change in `[build-system]`.
- Bad, because it is 0.x and allows breaking changes in minor releases.
- Bad, because it adds to the single-vendor concentration (Astral, now being acquired by OpenAI).
- Bad, because the lockfile format is proprietary and contributors cannot patch a Rust binary as easily as a Python tool.
- Bad, because `uv_build` cannot build extension modules.

### Poetry

Mature, Python-based dependency manager with its own lockfile and build backend (`poetry-core`).

- Good, because it is stable (post-1.0) with a long track record and a large user base.
- Good, because `poetry.lock` is cross-platform.
- Good, because it is maintained independently of Astral, spreading vendor risk.
- Neutral, because Poetry 2 supports standard `[project]` metadata, but some features still live in `[tool.poetry]`.
- Bad, because resolution and installation are noticeably slower than uv.
- Bad, because it does not manage Python installations by itself (pyenv or similar is still needed), so it is one more tool in the setup.

### pip-tools (+ venv + build backend)

`pip-compile` produces pinned `requirements.txt` files; environments are managed with `venv` and builds with a separate backend.

- Good, because it relies only on pip and standard formats; there is no vendor lock-in at all.
- Good, because it is simple and well understood.
- Bad, because compiled requirements are specific to the platform and Python version they were generated on, so a cross-platform CI matrix needs one lockfile per combination or loses reproducibility.
- Bad, because it needs several tools (pip-tools, venv, a build frontend, pyenv) and scripts to glue them together.
- Bad, because it is the slowest option.

### Ruff (linter + formatter)

Rust-based linter that reimplements hundreds of rules from flake8, its plugins, isort, pyupgrade and others, plus a Black-compatible formatter.

- Good, because one tool and one config section replace black, isort, flake8 and several plugins.
- Good, because it is fast enough to run on every save and every commit.
- Good, because it autofixes many rules (imports, pyupgrade, simplifications).
- Bad, because it has no plugin system: custom rules or niche flake8 plugins that Ruff has not reimplemented are not available.
- Bad, because the formatter is Black-*compatible*, not Black: there are documented deviations, and the style can change between Ruff minor releases (also 0.x).
- Bad, because new Ruff releases can add or change rules and produce new violations, which is why the version is pinned in pre-commit.
- Bad, because it is the second critical tool from the same vendor as uv.

### black + isort + flake8

The traditional trio: formatter, import sorter and linter, each from a different maintainer.

- Good, because each tool is mature and has a stable, well-documented behaviour (Black's stable style policy).
- Good, because flake8's plugin ecosystem allows custom and niche rules.
- Good, because maintenance is spread across independent projects.
- Bad, because there are three tools, three versions and three configs; flake8 does not read `pyproject.toml` natively.
- Bad, because black and isort need extra configuration to agree on import formatting.
- Bad, because they are slower, which matters for pre-commit on every commit.

### mypy

The reference Python type checker, written in Python (compiled with mypyc).

- Good, because it is the de facto reference implementation of the typing PEPs, with mature plugins (e.g. Pydantic, Django stubs).
- Good, because it installs from PyPI into the project environment and runs as a `uv run mypy` pre-commit hook that sees the real dependencies.
- Good, because it is independent of Astral, so it does not add to the vendor concentration.
- Bad, because it is slower than pyright on large codebases.
- Bad, because its inference is less precise than pyright's in some cases, and editor integration is weaker (most editors use pyright/Pylance for live feedback, so local and CI type errors can differ).
- Bad, because `strict = true` on `tests/` adds friction (annotating fixtures and test helpers).

### pyright

Microsoft's type checker, written in TypeScript; the engine behind Pylance in VS Code.

- Good, because it is fast and has strong inference, and it matches what most editors show.
- Good, because it is actively maintained by Microsoft.
- Neutral, because it supports a strict mode comparable to mypy's.
- Bad, because it needs a Node.js runtime; the PyPI wrapper downloads one, which adds weight and a non-Python dependency to CI.
- Bad, because it has no plugin system for frameworks that rely on mypy plugins.

## More Information

- [ADR-0002](0002-offer-pyrefly-as-optional-type-checker.md) adds pyrefly as an opt-in alternative; mypy stays the default chosen here.

- Revisit this decision if: uv or Ruff change license or stop being maintained as open source after the acquisition; uv reaches 1.0 (re-evaluate the `uv_build` upper bound); a generated project needs extension modules; or Astral's type checker `ty` becomes stable (consider it, weighing it against even more vendor concentration).
- [OpenAI to acquire Astral: The New Stack](https://thenewstack.io/openai-astral-acquisition/) · [Simon Willison: thoughts on OpenAI acquiring Astral](https://simonw.substack.com/p/thoughts-on-openai-acquiring-astral)
- [uv build backend documentation](https://docs.astral.sh/uv/concepts/build-backend/) · [PEP 751: lockfile format](https://peps.python.org/pep-0751/)
