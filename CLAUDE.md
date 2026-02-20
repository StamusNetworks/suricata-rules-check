# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A GitHub composite Action that validates Suricata IDS/IPS `.rules` files using `suricata-language-server`. It has no build system, package manager, or test suite — the action itself is the deliverable.

## Repository Structure

```
action.yml              # GitHub Action definition (inputs, outputs, composite steps)
actions/
  Dockerfile            # Docker image: jasonish/suricata + suricata-language-server
  entrypoint.sh         # Placeholder entrypoint for the Docker image (stub)
```

## Action Architecture

The action runs as a **composite action** (not Docker-based at the action level):

1. **Setup**: Installs Python 3.12 and `suricata-language-server` via pip on the runner.
2. **Validation loop**: Uses `find` to locate all files matching `rules-pattern`, then calls `suricata-language-server --container --batch-file <file>` on each.
3. **Diagnostic parsing**: Each line of output is a JSON object with `severity`, `message`, and `range.start.{line,column}` fields. Severity codes: `1`=error, `2`=warning, `4`=notice.
4. **GitHub annotations**: Converts diagnostics to `::error`, `::warning`, `::notice` workflow commands with file/line/col metadata.
5. **Outputs**: Sets `error-count` and `warning-count` to `$GITHUB_OUTPUT`. Exits non-zero on any file failure.

The `actions/Dockerfile` and `actions/entrypoint.sh` are for an alternative Docker-based usage pattern; the main validation logic lives entirely in `action.yml`.

## Action Inputs / Outputs

| Input | Default | Description |
|-------|---------|-------------|
| `rules-pattern` | `*.rules` | `find -name` pattern for locating rule files |
| `suricata-image` | `jasonish/suricata:latest` | Docker image passed to `suricata-language-server --image` |
| `fail-on-warnings` | `false` | Also fail on warning-severity diagnostics |
| `engine-analysis` | `true` | Run Suricata engine analysis (set `false` to speed up) |

Outputs: `error-count`, `warning-count`.

## Key Implementation Details

- `set +e` is intentional: all files are checked before deciding on overall exit code.
- Line numbers from `suricata-language-server` are 0-indexed; the script adds `+1` for GitHub annotations (which are 1-indexed).
- The `--container` flag on `suricata-language-server` is what causes it to pull and use the Suricata Docker image for engine analysis.
- `$FAIL_ON_WARNINGS_FLAG` and `$NO_ENGINE_FLAG` are passed directly to `suricata-language-server`, not handled in the shell script itself.
