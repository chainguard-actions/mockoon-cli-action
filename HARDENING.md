<!-- markdownlint-disable -->

# Hardening Report: mockoon--cli-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mockoon--cli-action/v3.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates user-controlled inputs via `${{ ... }}` expressions inside shell commands (rule a). Specifically: `${{ inputs.version }}` is concatenated into an npm install command, and `${{ inputs.data-file }}`, `${{ inputs.port }}`, and `${{ inputs.extra-args }}` are interpolated into the mockoon-cli invocation. These values are substituted by the YAML template engine before the shell ever sees them, allowing an attacker to inject arbitrary shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) and execute arbitrary commands. Additionally, none of the resulting shell expansions are double-quoted (rule b), compounding the risk. The fix is to move all inputs into `env:` variables and reference them as double-quoted shell variables: e.g. `env: { VERSION: "${{ inputs.version }}" }` and then `npm install -g "@mockoon/cli@$VERSION"`.

Locations:

- `action.yml:27`
- `action.yml:28`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key, and the only job (`test-cli-action`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal explicit `permissions:` block (e.g. `permissions: {}` or `permissions: contents: read`) should be added at the top level or on the job.

Locations:

- `.github/workflows/ci.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, missing-permissions

**Notes:**

action.yml: Moved all ${{ inputs.* }} expressions (version, data-file, port, extra-args) out of the run: block into an env: map as INPUT_VERSION, INPUT_DATA_FILE, INPUT_PORT, INPUT_EXTRA_ARGS. The shell script now uses double-quoted variable references and xargs-based array tokenization for the list inputs (data-file, port, extra-args) to correctly split space-separated values while preventing shell injection. .github/workflows/ci.yml: Added `permissions: {}` at the top level to explicitly restrict the GITHUB_TOKEN to no permissions.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in action.yml at line 34. Added strict allowlist validation of the `$INPUT_VERSION` variable using a regex pattern (`^[a-zA-Z0-9._+\-^~<>=*]+$`) before it is used in `npm install -g "@mockoon/cli@$INPUT_VERSION"`. The validation rejects any version string containing shell metacharacters (e.g., `$`, `(`, `)`, backticks, `;`, `|`, `&`) that could enable command substitution attacks, while still allowing all legitimate npm version formats and dist-tags.

