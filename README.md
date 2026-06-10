# GitHub Workflows

A collection of reusable GitHub Actions workflows for platform-wide security and CI standards.

## Usage

Reference workflows from this repo using the `uses` key with a pinned ref:

```yaml
jobs:
  security:
    uses: your-org/platform-github-workflows/.github/workflows/security-baseline.yml@main
```

Pin to a tag or SHA in production to avoid unexpected changes:

```yaml
    uses: your-org/platform-github-workflows/.github/workflows/security-baseline.yml@v1.0.0
```

---

## Workflows

### `security-baseline.yml` — Security Checks

Runs secret scanning (Gitleaks) and vulnerability/IaC scanning (Trivy) against the calling repo.

#### Inputs

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `severity` | `string` | `CRITICAL,HIGH` | Comma-separated Trivy severity levels that fail the scan. Valid values: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `UNKNOWN`. |

#### Secrets

| Name | Required | Description |
|------|----------|-------------|
| `GITLEAKS_LICENSE` | Yes (org repos) | Gitleaks license key. Not required for personal-account repos. Store as an org or repo secret named `GITLEAKS_LICENSE`. |

#### Permissions required in the caller

The workflow sets its own least-privilege permissions, but the calling workflow must grant:

```yaml
permissions:
  contents: read
  security-events: write  # needed for SARIF upload to GitHub Code Scanning
  actions: read
```

#### Example — minimal

```yaml
name: Security

on:
  pull_request:
  push:
    branches: [main]

jobs:
  security:
    uses: your-org/platform-github-workflows/.github/workflows/security-baseline.yml@main
    permissions:
      contents: read
      security-events: write
      actions: read
    secrets:
      GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}
```

#### Example — custom severity threshold

```yaml
jobs:
  security:
    uses: your-org/platform-github-workflows/.github/workflows/security-baseline.yml@main
    permissions:
      contents: read
      security-events: write
      actions: read
    with:
      severity: CRITICAL,HIGH,MEDIUM
    secrets:
      GITLEAKS_LICENSE: ${{ secrets.GITLEAKS_LICENSE }}
```

#### What it scans

| Tool | Scope | Fails PR on |
|------|-------|-------------|
| [Gitleaks](https://github.com/gitleaks/gitleaks) | Full git history | Any detected secret |
| [Trivy](https://github.com/aquasecurity/trivy) | Filesystem (deps + IaC) | Findings at or above `severity` threshold |

Trivy results are uploaded to [GitHub Code Scanning](https://docs.github.com/en/code-security/code-scanning) as SARIF, even when the scan fails, so findings are always visible in the Security tab.

---

## Contributing

### Adding a new workflow

1. Add the workflow file under `.github/workflows/`
2. Include `workflow_call:` in the `on:` block so it is callable
3. Document it in this README under the **Workflows** section
4. Pin all action dependencies to a full commit SHA and add a version comment

### Updating action versions

Dependabot is configured to open weekly PRs for action version bumps. Review and merge these to keep SHA pins current.
