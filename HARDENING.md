<!-- markdownlint-disable -->

# Hardening Report: dcarbone--install-jq-action/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dcarbone--install-jq-action/v2.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` steps in action.yaml directly interpolate `${{ github.action_path }}` into the shell command string. Before the shell executes the command, GitHub Actions performs YAML template substitution, meaning any attacker-controlled or unexpected value in `github.action_path` is injected raw into the PowerShell command. Offending lines:
  - `run: ${{ github.action_path }}\scripts\windowsish.ps1`
  - `run: ${{ github.action_path }}\scripts\windowsish-17.ps1`
Fix: Use the pre-set env var `$Env:GITHUB_ACTION_PATH` instead (as the Unix steps already do with `${GITHUB_ACTION_PATH}`).

Locations:

- `action.yaml:68`
- `action.yaml:75`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in tests.yaml directly interpolate GitHub Actions expressions into shell command strings, including `${{ matrix.version }}`, `${{ matrix.force }}`, `${{ steps.install-jq.outputs.installed }}`, and `${{ steps.install-jq.outputs.found }}`. These values flow through YAML template substitution before the shell parses them, enabling script injection if any value contains shell metacharacters. Examples:
  - `if [[ "${_vers}" != 'jq-${{ matrix.version }}' ]]; then`
  - `if [[ '${{ matrix.force }}' == 'true' ]]; then`
  - `if [[ '${{ steps.install-jq.outputs.installed }}' != 'true' ]]; then`
Fix: Move these values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/tests.yaml:57`
- `.github/workflows/tests.yaml:63`
- `.github/workflows/tests.yaml:70`
- `.github/workflows/tests.yaml:75`
- `.github/workflows/tests.yaml:80`
- `.github/workflows/tests.yaml:85`

### unpinned-uses (severity: high)

All four workflow files reference actions by mutable tag rather than a full 40-character commit SHA. Mutable tags can be silently moved to point to different (potentially malicious) commits, enabling supply-chain attacks.
  - `.github/workflows/tests.yaml`: `uses: actions/checkout@v4` (appears twice)
  - `.github/workflows/example-linux.yaml`: `uses: dcarbone/install-jq-action@v2`
  - `.github/workflows/example-macos.yaml`: `uses: dcarbone/install-jq-action@v2`
  - `.github/workflows/example-windows.yaml`: `uses: dcarbone/install-jq-action@v2`
Fix: Pin each `uses:` to a full SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/tests.yaml:44`
- `.github/workflows/tests.yaml:163`
- `.github/workflows/example-linux.yaml:21`
- `.github/workflows/example-macos.yaml:21`
- `.github/workflows/example-windows.yaml:21`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `contents: write` on some repository configurations), violating the principle of least privilege.
  - `.github/workflows/tests.yaml`
  - `.github/workflows/example-linux.yaml`
  - `.github/workflows/example-macos.yaml`
  - `.github/workflows/example-windows.yaml`
Fix: Add `permissions: {}` or a minimal set of required permissions (e.g. `contents: read`) at the top level of each workflow file.

Locations:

- `.github/workflows/tests.yaml:1`
- `.github/workflows/example-linux.yaml:1`
- `.github/workflows/example-macos.yaml:1`
- `.github/workflows/example-windows.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings:
1. script-injection in action.yaml: Replaced `${{ github.action_path }}` with `$Env:GITHUB_ACTION_PATH` in both Windows PowerShell run steps.
2. script-injection in tests.yaml: Moved all ${{ matrix.version }}, ${{ matrix.force }}, ${{ steps.install-jq.outputs.installed }}, and ${{ steps.install-jq.outputs.found }} expressions from run: shell blocks into env: blocks, referencing them as environment variables in both bash and PowerShell steps across all affected steps in both test-linux and test-container jobs.
3. unpinned-uses: Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 (both occurrences in tests.yaml) and dcarbone/install-jq-action@v2 to SHA 8867ddb4788346d7c22b72ea2e2ffe4d514c7bcb (in all three example workflow files).
4. missing-permissions: Added `permissions: {}` at the top level of all four workflow files (tests.yaml, example-linux.yaml, example-macos.yaml, example-windows.yaml).

