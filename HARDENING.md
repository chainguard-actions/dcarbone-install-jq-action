<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v3.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dcarbone--install-jq-action/v3.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yaml directly interpolate `${{ github.action_path }}` into the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the expression is substituted by the Actions template engine before the shell ever sees the string. The safe alternative already used by the Unix steps is the pre-set env var `$GITHUB_ACTION_PATH` (no expression interpolation). Offending lines:
- `run: ${{ github.action_path }}\scripts\windowsish.ps1` (Install jq - Windows-ish non-1.7 step)
- `run: ${{ github.action_path }}\scripts\windowsish-17.ps1` (Install jq - Windows-ish 1.7 step)

Locations:

- `action.yaml:74`
- `action.yaml:81`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yaml. Both PowerShell steps (lines 74 and 81) that used `${{ github.action_path }}\scripts\windowsish.ps1` and `${{ github.action_path }}\scripts\windowsish-17.ps1` were updated to use `& "$env:GITHUB_ACTION_PATH\scripts\windowsish.ps1"` and `& "$env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1"` respectively. This replaces the Actions template expression interpolation with the pre-set `GITHUB_ACTION_PATH` environment variable, consistent with how the Unix steps already handle this. The `& "..."` call operator is the correct PowerShell syntax for invoking a script whose path is in a quoted string.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed two `run:` values in action.yaml that used PowerShell's `& "$env:GITHUB_ACTION_PATH\scripts\..."` syntax as plain YAML scalars. The `&` character is a YAML anchor indicator and caused a parse error at line 79. Both occurrences were converted to block scalars (`run: |`) so the `&` is treated as literal text within the block, preserving the correct PowerShell invocation syntax.

