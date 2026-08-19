<!-- markdownlint-disable -->

# Hardening Report: josStorer--get-current-time/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **josStorer--get-current-time/v2.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable branch or tag refs instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks. Failing references: `josStorer/get-current-time@master` (branch ref), `actions/checkout@v3` (tag ref), `codecov/codecov-action@v3` (tag ref).

Locations:

- `.github/workflows/main.yml:11`
- `.github/workflows/main.yml:29`
- `.github/workflows/main.yml:33`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tag refs instead of full 40-character commit SHAs. Failing references: `actions/checkout@v3` (tag ref, appears twice).

Locations:

- `.github/workflows/pr-check.yml:10`
- `.github/workflows/pr-check.yml:17`

### script-injection (severity: high)

Rule (b) violation: The `run:` block expands shell variables `$TIME`, `$R_TIME`, `$F_TIME`, `$YEAR`, and `$DAY` without double-quoting them. These variables are populated from `steps.current-time.outputs.*` (workflow-controllable data) via the `env:` block. Unquoted expansion allows shell metacharacters in the output values to be interpreted by the shell. Offending line: `run: echo $TIME $R_TIME $F_TIME $YEAR $DAY`

Locations:

- `.github/workflows/main.yml:24`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents).

Locations:

- `.github/workflows/main.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. This workflow is triggered by `pull_request` events, making overly broad permissions especially risky.

Locations:

- `.github/workflows/pr-check.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all 5 findings across 2 workflow files:

.github/workflows/main.yml:
- Pinned josStorer/get-current-time@master → @49693c19176a68a5ecb39f75dae82364aee49a9a # master
- Pinned actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
- Pinned codecov/codecov-action@v3 → @ab904c41d6ece82784817410c45d8b8c02684457 # v3
- Fixed script injection: added double-quotes around $TIME, $R_TIME, $F_TIME, $YEAR, $DAY in the echo command
- Added top-level `permissions: {}` block

.github/workflows/pr-check.yml:
- Pinned both occurrences of actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
- Added top-level `permissions: {}` block

