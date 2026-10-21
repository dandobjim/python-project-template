# Changelog

All notable changes to this template will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
Generated projects pick up these versions with `uvx copier update`.

## [Unreleased]

### Added

- Python 3.15 as an option for `python_version`; generated CI matrices and `pyproject.toml` classifiers now run up to 3.15.
- `Template CI` renders and tests the template on Python 3.15.

### Changed

- Default `python_version` is now `3.15` (was `3.14`).

## [0.4.0] - 2026-10-19

### Added

- `type_checker` question: generated projects use mypy (default) or pyrefly, both in strict mode, with the matching dev dependency, `pyproject.toml` section, pre-commit hook and docs.
- Template CI renders and tests every combination of Python version and type checker.
- ADR-0002 documenting pyrefly as an optional type checker.

## [0.3.0] - 2026-10-15

### Added

- `CHANGELOG.md` for the template repository.
- ADR-0001 documenting the choice of uv, Ruff and mypy (`docs/adr/`).
- Guidance in the `python_version` question: latest for services, oldest supported for libraries.

### Changed

- ADR template upgraded to MADR 4: YAML front matter (`status`, `date`, `decision-makers`, `consulted`, `informed`) and a `Confirmation` section.
- Generated `CHANGELOG.md` starts with the scaffold under `[Unreleased]` and no empty subsections, instead of a `[0.1.0]` entry linking to a tag that does not exist.
- Generated `CONTRIBUTING.md` links to the pull request template instead of duplicating its checklist.

## [0.2.0] - 2026-10-15

### Added

- Documentation templates in generated projects: structured README with CI, Codecov and Python badges; MADR ADR template and index; `CONTRIBUTING.md`; `CHANGELOG.md`; `SECURITY.md` using GitHub private vulnerability reporting; pull request template.

### Changed

- Generated CI uploads coverage to Codecov instead of a build artifact.

## [0.1.0] - 2026-10-15

### Added

- Copier template with uv (`uv_build`, `src/` layout, dependency groups), Ruff, mypy `--strict`, pytest + pytest-cov with a configurable coverage threshold and pre-commit.
- Generated GitHub Actions CI: lint job and test matrix from the minimum Python version to 3.14, `uv sync --locked`, actions pinned by commit SHA.
- Dependabot for GitHub Actions, uv and pre-commit in generated projects.
- `Template CI` workflow that renders the template on Python 3.11–3.14 and runs the full toolchain.
- MIT license.

[Unreleased]: https://github.com/dandobjim/python-project-template/compare/v0.4.0...HEAD
[0.4.0]: https://github.com/dandobjim/python-project-template/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/dandobjim/python-project-template/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/dandobjim/python-project-template/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/dandobjim/python-project-template/releases/tag/v0.1.0
