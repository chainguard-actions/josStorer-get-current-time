<!-- markdownlint-disable -->

# Hardening Report: josStorer--get-current-time/v2.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **josStorer--get-current-time/v2.1.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is moved or overwritten.

Failing references in .github/workflows/main.yml:
- `josStorer/get-current-time@master` (line 11) — branch ref
- `actions/checkout@v4` (line 30) — tag ref
- `codecov/codecov-action@v3` (line 35) — tag ref

Failing references in .github/workflows/pr-check.yml:
- `actions/checkout@v4` (line 10) — tag ref
- `actions/checkout@v4` (line 17) — tag ref

All should be replaced with full SHA pins, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:11`
- `.github/workflows/main.yml:30`
- `.github/workflows/main.yml:35`
- `.github/workflows/pr-check.yml:10`
- `.github/workflows/pr-check.yml:17`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `write` access to repository contents and other resources depending on the organization/repository settings), violating the principle of least privilege. Both files should declare `permissions: {}` or specific minimal scopes at the top level or per-job.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/pr-check.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: The 'Use current time' step in main.yml sets env vars (`TIME`, `R_TIME`, `F_TIME`, `YEAR`, `DAY`) from `steps.current-time.outputs.*` (a workflow-controllable context) and then expands them **unquoted** in the `run:` shell command: `echo $TIME $R_TIME $F_TIME $YEAR $DAY`. Unquoted shell variable expansions allow the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace) out of the values, enabling command injection if any output contains shell metacharacters. The fix is to double-quote every expansion: `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"`.

Locations:

- `.github/workflows/main.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses** (main.yml + pr-check.yml): Replaced all 5 mutable tag/branch references with full 40-character SHA pins:
   - `josStorer/get-current-time@master` → `@49693c19176a68a5ecb39f75dae82364aee49a9a # master`
   - `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4` (3 occurrences)
   - `codecov/codecov-action@v3` → `@ab904c41d6ece82784817410c45d8b8c02684457 # v3`

2. **missing-permissions** (main.yml + pr-check.yml): Added `permissions: {}` at the top level of both workflow files to enforce least-privilege.

3. **script-injection** (main.yml line 25): Quoted all shell variable expansions in the `echo` command — changed `echo $TIME $R_TIME $F_TIME $YEAR $DAY` to `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"` to prevent shell metacharacter interpretation.

