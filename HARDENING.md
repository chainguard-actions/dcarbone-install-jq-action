<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dcarbone--install-jq-action/v4.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action.yaml directly interpolate `${{ github.action_path }}` inside the shell command string. Any `${{ ... }}` expression directly in a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. The affected steps are 'Install jq - Windows-ish sub-1.7' (`run: ${{ github.action_path }}\scripts\windowsish.ps1`) and 'Install jq - Windows-ish 1.7+' (`run: ${{ github.action_path }}\scripts\windowsish-17.ps1`). The safe alternative is to use the pre-set env var `$Env:GITHUB_ACTION_PATH` instead of the expression, as is already done correctly in the Unix steps (`run: ${GITHUB_ACTION_PATH}/scripts/unixish.sh`).

Locations:

- `action.yaml:72`
- `action.yaml:80`

### github-env-injection (severity: high)

All four install scripts write the inherited process env var `$RUNNER_TOOL_CACHE` (or `$Env:RUNNER_TOOL_CACHE` in PowerShell) to `$GITHUB_PATH` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). `RUNNER_TOOL_CACHE` is not set within the same `run:` block — it is an inherited env var from the runner/calling-workflow environment. Per the composite-action env-injection rule, any inherited process env var written to a special environment file (`$GITHUB_PATH`, `$GITHUB_ENV`, `$GITHUB_OUTPUT`) without sanitization is a violation. Affected lines: `echo "$RUNNER_TOOL_CACHE/jq" >> $GITHUB_PATH` in the shell scripts, and `Add-Content "$Env:GITHUB_PATH" "$Env:RUNNER_TOOL_CACHE\jq\"` in the PowerShell scripts.

Locations:

- `scripts/unixish.sh:88`
- `scripts/unixish-17.sh:118`
- `scripts/windowsish.ps1:52`
- `scripts/windowsish-17.ps1:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two findings: (1) script-injection in action.yaml: replaced `${{ github.action_path }}\scripts\windowsish*.ps1` with `& "$Env:GITHUB_ACTION_PATH\scripts\windowsish*.ps1"` using the PowerShell call operator and the pre-set GITHUB_ACTION_PATH env var. (2) github-env-injection in all four install scripts: sanitized RUNNER_TOOL_CACHE before writing to GITHUB_PATH — in shell scripts using `printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r'`, and in PowerShell scripts using `-replace '[\r\n]', ''`.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed two `run:` steps in action.yaml (the Windows PowerShell install steps) that used `run: & "$Env:GITHUB_ACTION_PATH\scripts\windowsish*.ps1"` as single-line values. The `&` character is a YAML anchor indicator, causing a parse error when used inline. Both were converted to block scalars (`run: |`) so the `& "..."` PowerShell call-operator syntax is treated as literal text. The functionality is preserved — PowerShell's `&` operator is used to invoke the script path stored in a quoted string.

