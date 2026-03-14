# .github

Org-level shared configuration for [Thistle-Grow-Software](https://github.com/Thistle-Grow-Software).

## What's Here

| Path | Purpose |
|---|---|
| `.github/workflows/` | Reusable GitHub Actions workflows |
| `.github/ISSUE_TEMPLATE/` | Org-default issue templates |
| `.github/PULL_REQUEST_TEMPLATE.md` | Org-default PR template |
| `actions/setup-python-ci-env/` | Composite action: Python + uv + optional CodeArtifact |
| `workflow-templates/` | Starter workflow templates (shown in repo Actions tab) |
| `ruff-base.toml` | Reference ruff configuration (org baseline) |

## Reusable Workflows

### Python Quality Gates

Runs formatting (ruff format), linting (ruff check), type checking (mypy), and unit tests (pytest) in parallel.

```yaml
# .github/workflows/quality-gates.yml
name: Quality Gates
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality-gates:
    uses: Thistle-Grow-Software/.github/.github/workflows/python-quality-gates.yml@main
    with:
      # All inputs have sensible defaults. Override as needed:
      # python-version: "3.14"
      # uv-sync-args: "--group dev"
      # src-paths: "src/"
      # test-marker: "unit"
      # mypy-enabled: true
      # mypy-continue-on-error: true
      # codecov-enabled: false
      # codecov-slug: "Thistle-Grow-Software/my-repo"
      # needs-codeartifact: false
    secrets: inherit
```

For repos that depend on private packages from CodeArtifact (e.g. `griddy-archive-manager`), set `needs-codeartifact: true` and configure these repo secrets: `aws-role-arn`, `codeartifact-domain`, `codeartifact-domain-owner`.

### Publish to CodeArtifact

Verifies the release tag matches `pyproject.toml` version, builds with `uv build`, and publishes to CodeArtifact.

```yaml
# .github/workflows/publish.yml
name: Publish to CodeArtifact
on:
  release:
    types: [published]

jobs:
  publish:
    uses: Thistle-Grow-Software/.github/.github/workflows/publish-codeartifact.yml@main
    secrets: inherit
```

Required repo secrets: `aws-role-arn`, `codeartifact-domain`, `codeartifact-domain-owner`, `codeartifact-repo`.

## Composite Action: `setup-python-ci-env`

Sets up Python, uv, and optionally authenticates with AWS CodeArtifact via OIDC.

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: Thistle-Grow-Software/.github/actions/setup-python-ci-env@main
    with:
      python-version: "3.14"          # optional, default: 3.14
      aws-role-to-assume: ${{ secrets.AWS_ROLE_ARN }}  # omit to skip AWS
      codeartifact-domain: ${{ secrets.CODEARTIFACT_DOMAIN }}
      codeartifact-domain-owner: ${{ secrets.AWS_ACCOUNT_ID }}
```

When CodeArtifact is configured, the action exports `UV_INDEX_PRIVATE_REGISTRY_USERNAME` and `UV_INDEX_PRIVATE_REGISTRY_PASSWORD` so subsequent `uv sync` / `uv install` commands can resolve private packages.

## Ruff Baseline

See [`ruff-base.toml`](ruff-base.toml) for the org-wide ruff configuration baseline. Since ruff doesn't support remote `extend`, repos should copy the relevant settings into their `pyproject.toml` and stay aligned.

**Required rules:** `E` (pycodestyle errors), `F` (pyflakes), `I` (isort)

**Recommended additions:** `B` (bugbear), `UP` (pyupgrade), `SIM` (simplify), `RUF` (ruff-specific), `C4` (comprehensions), `T20` (print)

## Templates

Issue and PR templates in `.github/` are org-level defaults. They apply to any repo in the org that doesn't define its own templates. Repos can override by adding their own `.github/ISSUE_TEMPLATE/` or `.github/PULL_REQUEST_TEMPLATE.md`.
