<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dcarbone--install-jq-action/v2.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yaml directly interpolate `${{ github.action_path }}` inside the run: shell command string for the Windows PowerShell install steps. Any `${{ ... }}` expression directly inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Offending lines:
  - `run: ${{ github.action_path }}\scripts\windowsish.ps1` (Install jq - Windows-ish non-1.7 step)
  - `run: ${{ github.action_path }}\scripts\windowsish-17.ps1` (Install jq - Windows-ish 1.7 step)
Fix: use the env var form `$Env:GITHUB_ACTION_PATH\scripts\windowsish.ps1` instead.

Locations:

- `action.yaml:63`
- `action.yaml:70`

### github-env-injection (severity: high)

Both Unix shell scripts write the inherited process env var `$RUNNER_TOOL_CACHE` to `$GITHUB_PATH` without the required sanitization step (`printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r'`). `RUNNER_TOOL_CACHE` is never set within the same `run:` block — it is inherited from the calling workflow environment and must be treated as untrusted per the composite-action rules. A newline injected into this value could add arbitrary entries to `$GITHUB_PATH`.
  - `scripts/unixish.sh`: `echo "$RUNNER_TOOL_CACHE/jq" >> $GITHUB_PATH`
  - `scripts/unixish-17.sh`: `echo "$RUNNER_TOOL_CACHE/jq" >> $GITHUB_PATH`
Fix: sanitize before writing, e.g.:
  `safe=$(printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r')`
  `echo "${safe}/jq" >> "$GITHUB_PATH"`

Locations:

- `scripts/unixish.sh:68`
- `scripts/unixish-17.sh:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two script-injection issues in action.yaml by replacing `${{ github.action_path }}` template expressions in PowerShell run: blocks with the safe `$Env:GITHUB_ACTION_PATH` environment variable reference. Fixed two github-env-injection issues in scripts/unixish.sh and scripts/unixish-17.sh by sanitizing the inherited `$RUNNER_TOOL_CACHE` environment variable with `printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r'` before writing to `$GITHUB_PATH`, preventing newline injection attacks.

