<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dcarbone--install-jq-action/v4.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` blocks for the 'Install jq - Windows-ish sub-1.7' and 'Install jq - Windows-ish 1.7+' steps directly interpolate `${{ github.action_path }}` inside the shell command string. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. The offending lines are: `run: ${{ github.action_path }}\scripts\windowsish.ps1` and `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`. The safe alternative is to use the `$env:GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yaml:79`
- `action.yaml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection vulnerabilities in action.yaml at lines 79 and 86. The 'Install jq - Windows-ish sub-1.7' and 'Install jq - Windows-ish 1.7+' steps used `${{ github.action_path }}\scripts\windowsish.ps1` and `${{ github.action_path }}\scripts\windowsish-17.ps1` directly in their `run:` blocks. These were replaced with `$env:GITHUB_ACTION_PATH\scripts\windowsish.ps1` and `$env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1` respectively, using the PowerShell environment variable instead of the YAML template expression to avoid template injection.

