<!-- markdownlint-disable -->

# Hardening Report: josStorer--get-current-time/v2.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **josStorer--get-current-time/v2.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is updated maliciously.

.github/workflows/main.yml:
  - uses: josStorer/get-current-time@master  (line 11) — branch ref
  - uses: actions/checkout@v3  (line 28) — tag ref
  - uses: codecov/codecov-action@v3  (line 32) — tag ref

.github/workflows/pr-check.yml:
  - uses: actions/checkout@v3  (line 10) — tag ref
  - uses: actions/checkout@v3  (line 17) — tag ref

All of these should be replaced with their full 40-character SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:11`
- `.github/workflows/main.yml:28`
- `.github/workflows/main.yml:32`
- `.github/workflows/pr-check.yml:10`
- `.github/workflows/pr-check.yml:17`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no individual job within them defines a `permissions:` block either. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often broad write access), violating the principle of least privilege. Both files should declare `permissions: {}` at the top level (or minimal required scopes) to restrict the GITHUB_TOKEN.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/pr-check.yml:1`

### script-injection (severity: high)

Rule (b) violation: In .github/workflows/main.yml, the `run:` block at line 24 expands multiple env vars without double-quoting them: `echo $TIME $R_TIME $F_TIME $YEAR $DAY`. These env vars are sourced directly from `steps.current-time.outputs.*` (a workflow-controllable context). Unquoted shell variable expansions allow the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, glob chars, whitespace) out of the values, enabling command injection if the action's outputs contain shell metacharacters. The fix is to double-quote every expansion: `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"`.

Locations:

- `.github/workflows/main.yml:24`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses** — Pinned all 5 action references to full 40-char SHAs:
   - `josStorer/get-current-time@master` → `@49693c19176a68a5ecb39f75dae82364aee49a9a # master`
   - `actions/checkout@v3` (×3) → `@f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3`
   - `codecov/codecov-action@v3` → `@ab904c41d6ece82784817410c45d8b8c02684457 # v3`

2. **missing-permissions** — Added `permissions: {}` at the top level of both `main.yml` and `pr-check.yml`.

3. **script-injection** — Fixed the unquoted variable expansions in `main.yml` line 24: changed `echo $TIME $R_TIME $F_TIME $YEAR $DAY` to `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"` to prevent shell metacharacter interpretation.

