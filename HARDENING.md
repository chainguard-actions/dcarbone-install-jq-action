# Hardening Report: dcarbone--install-jq-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **dcarbone--install-jq-action/v2.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two Windows run steps in action.yaml directly interpolate the `${{ github.action_path }}` expression inside the `run:` shell command string, rather than using the pre-set environment variable `$GITHUB_ACTION_PATH` (as the Unix steps correctly do). This constitutes direct expression interpolation in a run block. The 'Install jq - Windows-ish non-1.7' step uses `run: ${{ github.action_path }}\scripts\windowsish.ps1` and the 'Install jq - Windows-ish 1.7' step uses `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`. The fix is to use the `$Env:GITHUB_ACTION_PATH` environment variable instead, consistent with how the Unix steps use `${GITHUB_ACTION_PATH}`.

Locations:

- `action.yaml:69`
- `action.yaml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yaml:
1. 'Install jq - Windows-ish non-1.7' step (line 69): replaced `${{ github.action_path }}\scripts\windowsish.ps1` with `$Env:GITHUB_ACTION_PATH\scripts\windowsish.ps1`
2. 'Install jq - Windows-ish 1.7' step (line 77): replaced `${{ github.action_path }}\scripts\windowsish-17.ps1` with `$Env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1`

Both Windows steps now use the pre-set `$Env:GITHUB_ACTION_PATH` environment variable, consistent with how the Unix steps use `${GITHUB_ACTION_PATH}`, eliminating direct expression interpolation in run blocks.

