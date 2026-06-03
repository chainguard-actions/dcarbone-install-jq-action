# Hardening Report: dcarbone--install-jq-action/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dcarbone--install-jq-action/v3.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in action.yaml directly interpolate the `${{ github.action_path }}` GitHub Actions expression inside the shell command string, rather than referencing it via an environment variable. The 'Install jq - Windows-ish sub-1.7' step uses `run: ${{ github.action_path }}\scripts\windowsish.ps1` and the 'Install jq - Windows-ish 1.7+' step uses `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`. This is a `github.*` context expression interpolated directly in a `run:` block. The Unix equivalents correctly use the shell environment variable `${GITHUB_ACTION_PATH}` instead. The fix is to use `$Env:GITHUB_ACTION_PATH` (PowerShell env var) in the run block instead of the `${{ github.action_path }}` expression.

Locations:

- `action.yaml:68`
- `action.yaml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yaml at lines 68 and 76. The 'Install jq - Windows-ish sub-1.7' and 'Install jq - Windows-ish 1.7+' steps previously used `run: ${{ github.action_path }}\scripts\windowsish.ps1` and `run: ${{ github.action_path }}\scripts\windowsish-17.ps1` respectively, directly interpolating the GitHub Actions expression in the shell command string. Both have been replaced with `$Env:GITHUB_ACTION_PATH\scripts\windowsish.ps1` and `$Env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1`, using the PowerShell environment variable form — consistent with how the Unix steps already use `${GITHUB_ACTION_PATH}`.

