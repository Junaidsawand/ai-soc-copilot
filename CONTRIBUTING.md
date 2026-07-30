# Contributing to AI SOC Copilot

First off, thank you for considering contributing to AI SOC Copilot! It's people like you that make AI SOC Copilot a great platform for the cybersecurity community.

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

## Contribution Workflow

We utilize a strict **Trunk-Based Development** workflow via Pull Requests (PRs).

1. **Fork the repository** on GitHub.
2. **Clone your fork** locally.
3. **Create a branch** for your feature or bug fix.
4. **Develop and test** your changes locally.
5. **Commit your changes** following our conventions.
6. **Push the branch** to your fork.
7. **Submit a Pull Request** against the `main` branch of the upstream repository.

## Branch Naming Conventions

Branches must be descriptive and categorized by prefix:
- `feat/` for new features (e.g., `feat/add-pdf-export`)
- `fix/` for bug fixes (e.g., `fix/parser-crash-on-null-ip`)
- `docs/` for documentation changes (e.g., `docs/api-spec-update`)
- `chore/` for routine tasks, dependencies, etc. (e.g., `chore/update-dependencies`)

## Commit Conventions

We strictly follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. This allows us to automate changelog generation.

**Format:** `<type>(<scope>): <description>`

**Examples:**
- `feat(api): add endpoint for pdf export`
- `fix(parser): gracefully handle missing agent name`
- `docs(readme): update local setup instructions`

## Pull Request Process

1. **Fill out the PR template completely.** Ensure you detail *why* the change is necessary, not just *what* it is.
2. **Ensure tests pass.** Your PR will trigger GitHub Actions CI. If CI fails, the PR cannot be merged.
3. **Request Review.** At least one peer review approval from a core maintainer is required.
4. **Squash & Merge.** Maintainers will squash and merge your PR to keep the main branch history clean.

## Coding Standards

### Backend (Python)
- **Formatting:** We use `Black`.
- **Linting:** We use `Ruff`.
- **Typing:** Strict Python type hints are mandatory.
- **Validation:** All inputs must be validated via Pydantic.

### Frontend (Dart/Flutter)
- **Linting:** Must pass `flutter_lints` with zero warnings.
- **State:** Use the established state management provider. Avoid massive StatefulWidgets.

## Development Setup

For instructions on setting up your local environment (Docker, Flutter, environment variables), please read the [Developer Guide](docs/10-Developer-Guide/Developer_Guide.md).

## Issue Reporting

If you find a bug or have a feature request:
1. Check the existing issues to ensure it hasn't already been reported.
2. Open a new issue using the provided GitHub Issue Templates.
3. Provide as much context as possible, including OS, logs, and steps to reproduce.
