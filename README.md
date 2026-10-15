# python-project-template

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A [Copier](https://copier.readthedocs.io/en/stable/) template for standard Python projects, ready to code from minute one.

## What you get

| Area            | Tooling                                                                                         |
| --------------- | ----------------------------------------------------------------------------------------------- |
| Packaging       | [uv](https://docs.astral.sh/uv/) + `uv_build` backend, PEP 621 `pyproject.toml`, `src/` layout   |
| Linting/format  | [Ruff](https://docs.astral.sh/ruff/) (pycodestyle, pyflakes, isort, bugbear, pyupgrade, …)      |
| Git hooks       | [pre-commit](https://pre-commit.com/): ruff check/format, file hygiene hooks, `uv-lock`        |
| Tests           | [pytest](https://docs.pytest.org/) + [pytest-cov](https://pytest-cov.readthedocs.io/), branch coverage and a configurable `fail_under` |
| CI/CD           | GitHub Actions: lint job + test matrix from the minimum Python version up to 3.14              |
| License         | MIT, generated with the author name and current year                                           |

## Requirements

- [uv](https://docs.astral.sh/uv/getting-started/installation/) (Copier runs through `uvx`, no global install needed)
- Git

## Usage

Generate a new project from GitHub:

```bash
uvx copier copy gh:<your-user>/python-project-template my-project
```

Or from a local clone:

```bash
uvx copier copy /path/to/python-project-template my-project
```

Then:

```bash
cd my-project
git init
uv sync
uv run pre-commit install
uv run pytest
```

### Questions

| Question              | Description                                                        | Default                       |
| --------------------- | ------------------------------------------------------------------ | ----------------------------- |
| `project_name`        | Distribution name (e.g. `my-project`)                              | —                             |
| `package_name`        | Import package name, snake_case                                    | derived from `project_name`   |
| `project_description` | Short description                                                  | `A Python project`            |
| `author_name`         | Author full name (also used in the license)                        | —                             |
| `author_email`        | Author email                                                       | —                             |
| `github_username`     | GitHub user/org hosting the repo (badges and URLs)                 | —                             |
| `python_version`      | Minimum Python version (`3.11`–`3.14`); defines the CI matrix      | `3.12`                        |
| `coverage_threshold`  | Minimum coverage percentage before pytest fails                    | `80`                          |

Answers can be passed non-interactively with `--data key=value` (and `--defaults` for the rest).

### Updating an existing project

Projects keep their answers in `.copier-answers.yml`, so they can pull in template changes:

```bash
cd my-project
uvx copier update
```

## Generated project layout

```
my-project/
├── .github/workflows/ci.yml
├── src/my_project/
│   ├── __init__.py
│   └── py.typed
├── tests/
│   ├── conftest.py
│   └── test_my_project.py
├── .copier-answers.yml
├── .gitignore
├── .pre-commit-config.yaml
├── .python-version
├── LICENSE
├── README.md
└── pyproject.toml
```

## Template repository layout

```
python-project-template/
├── copier.yml     # questions and Copier settings
├── template/      # files rendered into the generated project (*.jinja are templated)
├── LICENSE
└── README.md
```

## Testing the template locally

```bash
uvx copier copy --defaults \
  --data project_name=demo-app \
  --data author_name="Jane Doe" \
  --data author_email=jane@example.com \
  --data github_username=jane \
  . /tmp/demo-app

cd /tmp/demo-app && git init && uv sync && uv run pytest && uv run pre-commit run --all-files
```

## License

[MIT](LICENSE) © 2026 Daniel Doblas
