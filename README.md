# python-project-template

[![Template CI](https://github.com/dandobjim/python-project-template/actions/workflows/template-ci.yml/badge.svg)](.github/workflows/template-ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A [Copier](https://copier.readthedocs.io/en/stable/) template for standard Python projects, ready to code from minute one.

## What you get

| Area            | Tooling                                                                                         |
| --------------- | ----------------------------------------------------------------------------------------------- |
| Packaging       | [uv](https://docs.astral.sh/uv/) + `uv_build` backend, PEP 621 `pyproject.toml`, `src/` layout   |
| Linting/format  | [Ruff](https://docs.astral.sh/ruff/) (pycodestyle, pyflakes, isort, bugbear, pyupgrade, bandit, pathlib, …) |
| Type checking   | [mypy](https://mypy.readthedocs.io/) in `strict` mode over `src/` and `tests/`                  |
| Git hooks       | [pre-commit](https://pre-commit.com/): ruff check/format, mypy, file hygiene hooks, `uv-lock`  |
| Tests           | [pytest](https://docs.pytest.org/) + [pytest-cov](https://pytest-cov.readthedocs.io/), branch coverage and a configurable `fail_under` |
| CI/CD           | GitHub Actions: lint job + test matrix from the minimum Python version up to 3.14, `uv sync --locked`, coverage to [Codecov](https://codecov.io), actions pinned by commit SHA |
| Documentation   | README (badges, overview, quickstart, configuration, development, testing, architecture), [MADR](https://adr.github.io/madr/) ADR template, CONTRIBUTING (branch naming, Conventional Commits, PR checklist), CHANGELOG ([Keep a Changelog](https://keepachangelog.com/en/1.1.0/) + SemVer), SECURITY policy and PR template |
| Dependency bots | [Dependabot](https://docs.github.com/en/code-security/dependabot) weekly updates for actions, uv dependencies and pre-commit hooks |
| License         | MIT, generated with the author name and year                                                    |

## Requirements

- [uv](https://docs.astral.sh/uv/getting-started/installation/) (Copier runs through `uvx`, no global install needed)
- Git

## Usage

Generate a new project from GitHub (uses the latest released tag):

```bash
uvx copier copy gh:dandobjim/python-project-template my-project
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

Commit the generated `uv.lock`: the generated CI installs with `--locked` and fails if it is missing or stale.

After pushing the new project to GitHub:

1. Connect the repository on [codecov.io](https://codecov.io) and add the `CODECOV_TOKEN` repository secret. Until then the coverage upload is skipped without failing CI.
2. Enable **Settings → Security → Private vulnerability reporting** so the link in `SECURITY.md` works.

### Questions

| Question              | Description                                                        | Default                       |
| --------------------- | ------------------------------------------------------------------ | ----------------------------- |
| `project_name`        | Distribution name (e.g. `my-project`)                              | —                             |
| `package_name`        | Import package name, snake_case                                    | derived from `project_name`   |
| `project_description` | Short description                                                  | `A Python project`            |
| `author_name`         | Author full name (also used in the license)                        | —                             |
| `author_email`        | Author email                                                       | —                             |
| `github_username`     | GitHub user/org hosting the repo (badges and URLs)                 | —                             |
| `python_version`      | Minimum Python version (`3.11`–`3.14`); defines the CI matrix      | `3.14`                        |
| `coverage_threshold`  | Minimum coverage percentage before pytest fails                    | `80`                          |
| `copyright_year`      | Year in the MIT license (stored, so it never changes on update)    | current year                  |

Answers can be passed non-interactively with `--data key=value` (and `--defaults` for the rest).

### Updating an existing project

Projects keep their answers (and the template commit) in `.copier-answers.yml`, so they can pull in template changes:

```bash
cd my-project
uvx copier update
```

## Versioning

The template is versioned with [SemVer](https://semver.org/) git tags (`v0.1.0`, …) and every release is described in [CHANGELOG.md](CHANGELOG.md). Copier uses the latest tag **on the remote** for `copy` and `update`, which is what lets `copier update` compute a clean diff between template versions. To release a change:

1. Move the `[Unreleased]` entries in `CHANGELOG.md` to a new `[X.Y.Z] - YYYY-MM-DD` section.
2. Commit, create an **annotated** tag and push the commit together with the tag:

   ```bash
   git tag -a vX.Y.Z -m "vX.Y.Z"
   git push --follow-tags
   ```

   `--follow-tags` only pushes annotated tags; a lightweight tag (`git tag vX.Y.Z`) stays local unless you run `git push --tags`.

Without tags on the remote Copier falls back to `HEAD` and warns `No git tags found in template`.

Design decisions about the template itself are recorded as ADRs in [`docs/adr/`](docs/adr/README.md).

## Template repository layout

```
python-project-template/
├── .github/
│   ├── dependabot.yml             # keeps the template's own CI actions updated
│   └── workflows/template-ci.yml  # renders the template and tests the result
├── docs/adr/                      # decisions about the template itself (MADR 4)
├── copier.yml                     # questions and Copier settings
├── template/                      # files rendered into the generated project (*.jinja are templated)
├── CHANGELOG.md                   # template releases (Keep a Changelog + SemVer)
├── LICENSE
└── README.md
```

## Generated project layout

```
my-project/
├── .github/
│   ├── dependabot.yml
│   ├── pull_request_template.md
│   └── workflows/ci.yml
├── docs/adr/
│   ├── 0000-template.md
│   └── README.md
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
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
└── pyproject.toml
```

## Testing the template

`Template CI` runs on every push and pull request: it renders the template for each Python version from 3.11 to 3.14 and, inside the generated project, runs `uv lock`, `uv sync --locked`, `pytest` and every pre-commit hook (ruff, mypy, …).

To do the same locally (`--vcs-ref HEAD` renders your working commit instead of the latest tag; uncommitted changes are included as a dirty copy):

```bash
uvx copier copy --defaults --vcs-ref HEAD \
  --data project_name=demo-app \
  --data author_name="Jane Doe" \
  --data author_email=jane@example.com \
  --data github_username=jane \
  . /tmp/demo-app

cd /tmp/demo-app && git init && git add -A \
  && uv sync && uv run pytest && uv run pre-commit run --all-files
```

## License

[MIT](LICENSE) © 2026 Daniel Doblas
