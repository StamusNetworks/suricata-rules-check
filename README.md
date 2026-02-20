# Validate Suricata Rules

A GitHub Action that validates Suricata `.rules` files and reports diagnostics as GitHub annotations on your pull requests and workflow runs.

## Usage

```yaml
- uses: StamusNetworks/suricata-rules-check@v1
```

By default, the action finds all `*.rules` files in the repository and validates each one using `suricata-language-server`.

## Example Workflows

### Validate on every pull request

```yaml
name: Validate Suricata Rules

on:
  pull_request:
  push:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: StamusNetworks/suricata-rules-check@v1
```

### Fail on warnings and use a pinned Suricata version

```yaml
- uses: StamusNetworks/suricata-rules-check@v1
  with:
    fail-on-warnings: 'true'
    suricata-image: 'jasonish/suricata:7.0'
```

### Fast validation (skip engine analysis)

Engine analysis catches more issues but requires pulling a Docker image and running Suricata. Skip it for faster feedback on syntax-only checks:

```yaml
- uses: StamusNetworks/suricata-rules-check@v1
  with:
    engine-analysis: 'false'
```

### Use outputs to gate a deployment

```yaml
- uses: StamusNetworks/suricata-rules-check@v1
  id: check
- name: Summarize results
  run: |
    echo "Errors: ${{ steps.check.outputs.error-count }}"
    echo "Warnings: ${{ steps.check.outputs.warning-count }}"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `rules-path` | No | `.` | Directory to search for rules files |
| `rules-pattern` | No | `*.rules` | `find -name` pattern used to locate rule files in the repository |
| `suricata-image` | No | `jasonish/suricata:latest` | Suricata Docker image used for engine analysis |
| `fail-on-warnings` | No | `false` | Exit with a failure status if any warnings are found, in addition to errors |
| `engine-analysis` | No | `true` | Run Suricata engine analysis. Set to `false` to validate syntax only and speed up the action |

## Outputs

| Output | Description |
|--------|-------------|
| `error-count` | Total number of error-severity diagnostics found across all files |
| `warning-count` | Total number of warning-severity diagnostics found across all files |

## How It Works

For each matching rules file, the action calls `suricata-language-server` to produce structured diagnostics. Those diagnostics are converted into [GitHub workflow annotations](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/workflow-commands-for-github-actions#setting-a-notice-message) (errors, warnings, and notices) that appear inline in pull request file diffs and on the workflow summary page.

All files are always checked before the action exits, so you see the full picture in a single run even when multiple files have issues.

The action exits with a non-zero status if any file produces error-severity diagnostics, or if `fail-on-warnings` is enabled and any warnings are found.

## Requirements

The runner must have Docker available. The action installs Python and `suricata-language-server` automatically.
