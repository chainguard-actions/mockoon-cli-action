# Hardening Report: mockoon--cli-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **mockoon--cli-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run Mockoon CLI' step in action.yml directly interpolates three attacker-controlled inputs expressions into the shell run: block without first assigning them to environment variables: `${{ inputs.version }}` is appended to the npm install command, and `${{ inputs.data-file }}` and `${{ inputs.port }}` are passed directly to mockoon-cli. An attacker who controls these input values (e.g. via workflow_dispatch or a calling workflow) can inject arbitrary shell commands. The fix is to assign each input to an env: variable and reference it as $VAR_NAME in the run: block.

Locations:

- `action.yml:22`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four script injection findings in action.yml by adding an env: block to the 'Run Mockoon CLI' step with three variables: MOCKOON_VERSION (${{ inputs.version }}), MOCKOON_DATA_FILE (${{ inputs.data-file }}), and MOCKOON_PORT (${{ inputs.port }}). The run: block now references these as plain shell environment variables (${MOCKOON_VERSION}, ${MOCKOON_DATA_FILE}, ${MOCKOON_PORT}) with proper double-quoting to prevent word splitting and shell injection.

