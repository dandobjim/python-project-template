---
status: accepted
date: 2026-10-19
decision-makers: [Daniel Doblas]
consulted: []
informed: []
---

# Offer pyrefly as an optional type checker

## Context and Problem Statement

[ADR-0001](0001-use-uv-and-ruff-for-tooling.md) made mypy in strict mode the type checker of every generated project. Since then [pyrefly](https://pyrefly.org/), Meta's type checker written in Rust, has reached a stable release (1.x on PyPI) with a strict preset, `pyproject.toml` configuration and a built-in language server.

Should generated projects be able to use pyrefly instead of mypy, and if so, how without breaking projects that already use mypy?

## Decision Drivers

- **Speed**: type checking runs on every commit (pre-commit) and in CI.
- **Editor/CI consistency**: the errors shown in the editor should match the ones that fail CI.
- **Framework support**: some projects depend on mypy plugins (Django, Pydantic, SQLAlchemy).
- **Safe updates**: `copier update` must not change the type checker of existing projects.
- **Maintenance cost**: every option is one more configuration and one more CI combination to keep green.

## Considered Options

- Keep mypy only
- Offer a choice: mypy (default) or pyrefly
- Switch the default to pyrefly
- Offer ty (Astral)

## Decision Outcome

Chosen option: **offer a choice, mypy (default) or pyrefly**, through a `type_checker` Copier question. Projects that need speed or a matching language server can opt in to pyrefly, projects that need mypy plugins keep mypy, and existing projects keep mypy on `copier update` because a new question gets its default.

Both options use the same pattern as ADR-0001: a dev dependency in the project environment, a `repo: local` pre-commit hook (`uv run mypy` / `uv run pyrefly check`) that sees the real dependencies, and checking of `src/` and `tests/` in strict mode.

### Consequences

- Good, because pyrefly is much faster than mypy and ships its own language server, so the editor and CI report the same errors.
- Good, because pyrefly is independent of Astral, so it does not add to the vendor concentration noted in ADR-0001.
- Good, because the default does not change: existing projects and users who do not care get exactly what they had.
- Neutral, because `[tool.pyrefly]` uses `preset = "strict"` and `search-path = ["src"]`; no interpreter path is needed because `uv run` exposes the project environment.
- Bad, because the template CI matrix doubles (4 Python versions × 2 type checkers = 8 jobs).
- Bad, because the template keeps two configurations and two sets of docs in sync through Jinja conditionals.
- Bad, because the strict modes are not equivalent: pyrefly infers return types instead of requiring annotations on them, and its error codes differ from mypy's, so switching an existing project can surface or hide errors.
- Bad, because pyrefly does not support mypy plugins; projects that rely on them must stay on mypy.

### Confirmation

`Template CI` renders the template for every Python version (3.11–3.15) and every `type_checker` value, then runs `uv lock`, `uv sync --locked`, `pytest` and every pre-commit hook in the generated project. A failure in either type checker fails the matrix.

## Pros and Cons of the Options

### Keep mypy only

- Good, because there is a single configuration and the CI matrix stays small.
- Bad, because projects that want faster checks or a language server that matches CI have to change the tooling by hand and then diverge from the template.

### Offer a choice: mypy (default) or pyrefly

- Good, because each project picks what fits it, and the default keeps updates safe.
- Bad, because of the extra maintenance and CI cost described above.

### Switch the default to pyrefly

- Good, because new projects get the faster tool without opting in.
- Bad, because `copier update` would move existing projects to pyrefly unless they re-answer the question, a breaking change with differing strictness.
- Bad, because projects that need mypy plugins would have to opt out explicitly.

### Offer ty (Astral)

- Good, because it is fast and from the same vendor as uv and Ruff, with a similar configuration style.
- Bad, because it adds to the vendor concentration ADR-0001 already flags.
- Bad, because ADR-0001 deferred it until it is stable; this decision does not change that.

## More Information

- Revisit this decision if pyrefly gains plugin support or full mypy-strict parity (then consider making it the default in a major template release), if ty becomes stable, or if keeping both configurations green becomes a burden.
- [pyrefly configuration](https://pyrefly.org/en/docs/configuration/) · [pyrefly on PyPI](https://pypi.org/project/pyrefly/)
