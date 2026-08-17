<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v3.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dcarbone--install-jq-action/v3.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action.yaml directly interpolate `${{ github.action_path }}` inside the run: shell command string. Although `github.action_path` is not attacker-controlled, any `${{ ... }}` expression directly inside a `run:` block is a script-injection finding per the check rules. The offending lines are:
  `run: ${{ github.action_path }}\scripts\windowsish.ps1`
  `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`
These should use the `$GITHUB_ACTION_PATH` environment variable instead (as the Unix steps already do correctly with `${GITHUB_ACTION_PATH}/scripts/unixish.sh`).

Locations:

- `action.yaml:62`
- `action.yaml:70`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag-based refs instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved:
- `uses: actions/checkout@v4` (tests.yaml, twice)
- `uses: dcarbone/install-jq-action@v3` (example-linux.yaml, example-macos.yaml, example-windows.yaml)
All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/tests.yaml:97`
- `.github/workflows/tests.yaml:222`
- `.github/workflows/example-linux.yaml:21`
- `.github/workflows/example-macos.yaml:21`
- `.github/workflows/example-windows.yaml:21`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and no individual job within them defines a `permissions:` block either. Without explicit permissions, workflows inherit the default repository permissions (which may include `contents: write` and other broad scopes). Each workflow should declare minimal required permissions, e.g. `permissions: {}` or `permissions: contents: read`.

Locations:

- `.github/workflows/tests.yaml:1`
- `.github/workflows/example-linux.yaml:1`
- `.github/workflows/example-macos.yaml:1`
- `.github/workflows/example-windows.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in action.yaml: replaced `${{ github.action_path }}\scripts\windowsish.ps1` and `${{ github.action_path }}\scripts\windowsish-17.ps1` with `$Env:GITHUB_ACTION_PATH\scripts\windowsish.ps1` and `$Env:GITHUB_ACTION_PATH\scripts\windowsish-17.ps1` to use the environment variable instead of a template expression in the run shell string; (2) unpinned-uses: pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262` in tests.yaml (both occurrences), and pinned `dcarbone/install-jq-action@v3` to SHA `b7ef57d46ece78760b4019dbc4080a1ba2a40b45` in all three example workflow files; (3) missing-permissions: added `permissions: contents: read` top-level block to all four workflow files.

