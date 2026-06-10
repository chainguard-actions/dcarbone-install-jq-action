<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **dcarbone--install-jq-action/v3.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yaml directly interpolate `${{ github.action_path }}` into the shell command string. Although `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection finding per the check rules — the value flows through YAML template substitution before the shell ever sees it. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable (as the Unix steps already do correctly). Offending lines:
- `run: ${{ github.action_path }}\scripts\windowsish.ps1`
- `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`

Locations:

- `action.yaml:75`
- `action.yaml:82`

### github-env-injection (severity: high)

All four install scripts write the inherited process environment variable `$RUNNER_TOOL_CACHE` (or `$Env:RUNNER_TOOL_CACHE`) to `$GITHUB_PATH` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). In a composite action, `RUNNER_TOOL_CACHE` is inherited from the calling workflow's environment and is therefore workflow-controlled/untrusted for the purposes of this check. Writing it unsanitized to `$GITHUB_PATH` allows a newline-injection attack that could add arbitrary entries to the runner's PATH. The pattern `echo "$RUNNER_TOOL_CACHE/jq" >> $GITHUB_PATH` (and the PowerShell equivalent `Add-Content "$Env:GITHUB_PATH" "$Env:RUNNER_TOOL_CACHE\jq\"`) must be preceded by a sanitization step.

Locations:

- `scripts/unixish.sh:76`
- `scripts/unixish-17.sh:67`
- `scripts/windowsish.ps1:47`
- `scripts/windowsish-17.ps1:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 2 findings across 5 files:

1. script-injection (action.yaml): Replaced `${{ github.action_path }}\scripts\windowsish.ps1` and `${{ github.action_path }}\scripts\windowsish-17.ps1` with `$Env:GITHUB_ACTION_PATH\scripts\windowsish.ps1` and `$Env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1`. This uses the PowerShell environment variable equivalent instead of YAML template interpolation.

2. github-env-injection (4 scripts): Added sanitization of RUNNER_TOOL_CACHE before writing to GITHUB_PATH:
   - unixish.sh and unixish-17.sh: Used `printf '%s' "$RUNNER_TOOL_CACHE" | tr -d '\n\r'` to strip newlines before writing to $GITHUB_PATH.
   - windowsish.ps1 and windowsish-17.ps1: Used PowerShell `-replace` operator to strip CR and LF characters from RUNNER_TOOL_CACHE before writing to GITHUB_PATH.

