<!-- markdownlint-disable -->

# Hardening Report: josStorer--get-current-time/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **josStorer--get-current-time/v2.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable branch or tag refs instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced ref is overwritten.

In `.github/workflows/main.yml`:
- Line 11: `uses: josStorer/get-current-time@master` (branch ref)
- Line 27: `uses: actions/checkout@v3` (tag ref)
- Line 32: `uses: codecov/codecov-action@v3` (tag ref)

In `.github/workflows/pr-check.yml`:
- Line 9: `uses: actions/checkout@v3` (tag ref)
- Line 16: `uses: actions/checkout@v3` (tag ref)

All should be pinned to full 40-character hex commit SHAs, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/main.yml:11`
- `.github/workflows/main.yml:27`
- `.github/workflows/main.yml:32`
- `.github/workflows/pr-check.yml:9`
- `.github/workflows/pr-check.yml:16`

### permissions (severity: medium)

Neither `.github/workflows/main.yml` nor `.github/workflows/pr-check.yml` has a top-level `permissions:` key, and none of their individual jobs define a `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` access to contents and pull requests). A minimal `permissions:` block should be added at the top level or per-job.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/pr-check.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: In `.github/workflows/main.yml`, the `run:` step at line 23 expands five environment variables (`$TIME`, `$R_TIME`, `$F_TIME`, `$YEAR`, `$DAY`) without double-quoting them. These variables are sourced from `steps.current-time.outputs.*` (workflow-controllable data). Unquoted shell variable expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace) out of the values, enabling command injection.

Offending line:
```
run: echo $TIME $R_TIME $F_TIME $YEAR $DAY
```

Fix: quote every expansion:
```
run: echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"
```

Locations:

- `.github/workflows/main.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses** (high): Pinned all 5 action references to full 40-char commit SHAs:
   - `josStorer/get-current-time@master` → `@49693c19176a68a5ecb39f75dae82364aee49a9a # master`
   - `actions/checkout@v3` (×3) → `@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3`
   - `codecov/codecov-action@v3` → `@ab904c41d6ece82784817410c45d8b8c02684457 # v3`

2. **permissions** (medium): Added `permissions: {}` at the top level of both `main.yml` and `pr-check.yml`, restricting the GITHUB_TOKEN to no permissions by default.

3. **script-injection** (high): Fixed the unquoted variable expansions in `main.yml` line 23: changed `echo $TIME $R_TIME $F_TIME $YEAR $DAY` to `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"` to prevent shell metacharacter injection.

