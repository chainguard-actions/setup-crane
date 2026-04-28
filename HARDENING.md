# Hardening Report: imjasonh--setup-crane/v0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **imjasonh--setup-crane/v0.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` step in action.yml directly interpolates attacker-controlled GitHub Actions expressions into the shell script without first assigning them to environment variables:
- Line 22: `case ${{ inputs.version }} in` — `inputs.version` is interpolated directly into a shell `case` statement. A malicious value (e.g. containing `; malicious_cmd ;`) could execute arbitrary commands.
- Line 30: `url=".../${{ inputs.version }}"` — same `inputs.version` interpolated into a string assignment.
- Line 33: `os=${{ runner.os }}` — `github.runner.os` context value interpolated directly.
- Line 61: `echo "${{ github.token }}" | crane auth login ...` — `github.token` interpolated directly into the shell command.
All of these should be assigned to environment variables via `env:` and referenced as `$ENV_VAR` in the run block instead.

Locations:

- `action.yml:22`
- `action.yml:30`
- `action.yml:33`
- `action.yml:61`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml by adding an `env:` block to the composite step and moving all ${{ }} expressions out of the `run:` shell script:
- `${{ inputs.version }}` → `CRANE_VERSION` env var (referenced in `case $CRANE_VERSION` and URL construction)
- `${{ runner.os }}` → `RUNNER_OS` env var (referenced as `os=$RUNNER_OS`)
- `${{ github.token }}` → `GITHUB_TOKEN` env var (referenced as `echo "$GITHUB_TOKEN" | crane auth login ...`)
All three locations (lines 22, 30/31, 33, 61) are now safe from shell injection attacks.

