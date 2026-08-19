<!-- markdownlint-disable -->

# Hardening Report: mockoon--cli-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mockoon--cli-action/v2.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates GitHub Actions expressions for user-controlled inputs into shell commands without any env-var indirection or quoting. Specifically: `npm install -g @mockoon/cli@${{ inputs.version }}` and `mockoon-cli start --data ${{ inputs.data-file }} --port ${{ inputs.port }} &`. An attacker calling this composite action can supply values containing shell metacharacters (e.g. `; malicious-command #`) to achieve arbitrary command execution on the runner. Sub-rule (a): direct expression interpolation inside a `run:` block.

Locations:

- `action.yml:21`

### unpinned-uses (severity: high)

The workflow file .github/workflows/ci.yml references actions using mutable version tags instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved. Unpinned references: `actions/checkout@v3`, `actions/setup-node@v3`, `mockoon/cli-action@v2`.

Locations:

- `.github/workflows/ci.yml:13`
- `.github/workflows/ci.yml:15`
- `.github/workflows/ci.yml:18`

### missing-permissions (severity: medium)

The workflow file .github/workflows/ci.yml has no top-level `permissions:` key and the single job `test-cli-action` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (typically `write` for all scopes on private repos, or the organization default), granting broader access than necessary.

Locations:

- `.github/workflows/ci.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:24`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.data-file }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.port }}" appears directly in run: block of step "Run Mockoon CLI"; move to env: map

Locations:

- `action.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed action.yml: moved all three ${{ inputs.* }} expressions (version, data-file, port) out of the run: block into an env: block as MOCKOON_VERSION, MOCKOON_DATA_FILE, and MOCKOON_PORT; shell script now references them as quoted env vars. Fixed ci.yml: pinned actions/checkout@v3 → SHA a37ce91..., actions/setup-node@v3 → SHA 3235b87..., mockoon/cli-action@v2 → SHA 971589e... with original tags in comments; added top-level `permissions: {}` to enforce least privilege.

