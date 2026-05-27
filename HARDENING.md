# Hardening Report: dcarbone--install-jq-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dcarbone--install-jq-action/v2.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in action.yaml directly interpolate the `${{ github.action_path }}` expression inside the shell command string, rather than assigning it to an environment variable first. While `github.action_path` is system-controlled, this pattern violates the safe-usage rule that requires all `github.*` expressions to be passed via `env:` before use in `run:` blocks. The affected steps are 'Install jq - Windows-ish non-1.7' and 'Install jq - Windows-ish 1.7', both using `run: ${{ github.action_path }}\scripts\windowsish*.ps1`.

Locations:

- `action.yaml:68`
- `action.yaml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yaml at lines 68 and 76. For both 'Install jq - Windows-ish non-1.7' and 'Install jq - Windows-ish 1.7' steps, moved `${{ github.action_path }}` out of the `run:` shell string and into the step's `env:` block as `ACTION_PATH: '${{ github.action_path }}'`. The PowerShell `run:` commands now use `$env:ACTION_PATH\scripts\windowsish*.ps1` instead of directly interpolating the expression.

