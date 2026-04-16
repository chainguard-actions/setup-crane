# Hardening Report: imjasonh--setup-crane/v0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **imjasonh--setup-crane/v0.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions into shell commands without first assigning them to environment variables. Specifically:
- Line 22: `case ${{ inputs.version }} in` — the user-supplied `inputs.version` is injected directly into a shell `case` statement. A malicious value (e.g. containing `)` or `;`) could break out of the case construct and execute arbitrary commands.
- Line 30: `url="https://github.com/.../download/${{ inputs.version }}"` — same `inputs.version` injected into a shell string.
- Line 33: `os=${{ runner.os }}` — `runner.os` context value injected directly into a shell variable assignment.
The safe pattern is to assign these values to environment variables via `env:` and then reference them as `$ENV_VAR` in the shell script.

Locations:

- `action.yml:22`
- `action.yml:30`
- `action.yml:33`

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

Fixed all script injection vulnerabilities in action.yml by adding an env: block to the step and moving all GitHub Actions expressions out of the run: block:
- `${{ inputs.version }}` → env var `CRANE_VERSION` (used in case statement and URL construction)
- `${{ runner.os }}` → env var `RUNNER_OS` (used in OS detection)
- `${{ github.token }}` → env var `GITHUB_TOKEN` (used for crane auth login)
All shell references now use plain `$VAR_NAME` syntax, preventing shell injection attacks.

