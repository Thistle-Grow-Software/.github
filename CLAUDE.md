# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **org-level `.github` repository** for [Thistle-Grow-Software](https://github.com/Thistle-Grow-Software). It provides shared GitHub Actions workflows, composite actions, issue/PR templates, and configuration baselines consumed by all repos in the org.

Changes here propagate to every consuming repo — treat modifications with the same care as a shared library release.

## Repository Layout

- `.github/workflows/` — Reusable workflow definitions (called via `workflow_call`)
- `.github/ISSUE_TEMPLATE/` — Org-default issue templates (bug report, feature request)
- `.github/PULL_REQUEST_TEMPLATE.md` — Org-default PR template
- `actions/setup-python-ci-env/` — Composite action: Python + uv + optional AWS CodeArtifact OIDC auth
- `workflow-templates/` — Starter templates shown in the Actions tab of org repos
- `ruff-base.toml` — Org-wide ruff configuration reference (repos copy settings into their own `pyproject.toml` since ruff `extend` only supports local paths)

## Key Workflows

### `python-quality-gates.yml`
Reusable workflow that runs formatting (`ruff format`), linting (`ruff check` + optional `mypy`), and tests (`pytest`) as three parallel jobs. All jobs use the `setup-python-ci-env` composite action. Key inputs: `python-version`, `uv-sync-args`, `src-paths`, `test-marker`, `mypy-enabled`, `codecov-enabled`, `needs-codeartifact`.

### `publish-codeartifact.yml`
Reusable workflow triggered on `release` events. Verifies the git tag matches the `pyproject.toml` version, builds with `uv build`, and publishes to AWS CodeArtifact. Requires secrets: `aws-role-arn`, `codeartifact-domain`, `codeartifact-domain-owner`, `codeartifact-repo`.

### `setup-python-ci-env` composite action
Sets up Python (via `actions/setup-python`), uv (via `astral-sh/setup-uv`), and optionally authenticates with AWS CodeArtifact via OIDC. When CodeArtifact is configured, it exports `UV_INDEX_PRIVATE_REGISTRY_USERNAME` and `UV_INDEX_PRIVATE_REGISTRY_PASSWORD` environment variables for `uv sync`/`uv install`.

## Development Notes

- There is no build step, test suite, or linter for this repo itself — validation happens in consuming repos.
- Workflow templates in `workflow-templates/` use `$default-branch` as a placeholder that GitHub replaces when a repo adopts the template.
- The ruff baseline requires rules `E`, `F`, `I` across all repos; `B`, `UP`, `SIM`, `RUF`, `C4`, `T20` are recommended additions.
- All CI uses `uv` as the package manager (not pip). Python target is 3.14.
- AWS authentication uses OIDC (`id-token: write` permission), not static credentials.
