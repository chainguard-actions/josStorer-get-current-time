<!-- markdownlint-disable -->

# Hardening Report: josStorer--get-current-time/v2.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **josStorer--get-current-time/v2.1.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or branch names instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced action is compromised or its tag is moved.

In `.github/workflows/main.yml`:
- Line 11: `uses: josStorer/get-current-time@master` (branch ref — mutable)
- Line 29: `uses: actions/checkout@v4` (tag ref — mutable)
- Line 33: `uses: codecov/codecov-action@v3` (tag ref — mutable)

In `.github/workflows/pr-check.yml`:
- Line 10: `uses: actions/checkout@v4` (tag ref — mutable)
- Line 17: `uses: actions/checkout@v4` (tag ref — mutable)

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/main.yml:11`
- `.github/workflows/main.yml:29`
- `.github/workflows/main.yml:33`
- `.github/workflows/pr-check.yml:10`
- `.github/workflows/pr-check.yml:17`

### missing-permissions (severity: medium)

Neither `.github/workflows/main.yml` nor `.github/workflows/pr-check.yml` declares a top-level `permissions:` block, and no individual job within either file declares its own `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` access to contents). Every workflow should declare minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/pr-check.yml:1`

### script-injection (severity: high)

Rule (b) violation in `.github/workflows/main.yml` at line 25: the `run:` block expands five shell variables (`$TIME`, `$R_TIME`, `$F_TIME`, `$YEAR`, `$DAY`) without double-quoting them. All five are sourced from `steps.current-time.outputs.*` (workflow-controllable step outputs) via the `env:` block. Unquoted shell expansion allows shell metacharacters embedded in those values (`;`, `|`, `&`, `$(...)`, etc.) to be interpreted by the shell, enabling command injection.

Offending line:
```
run: echo $TIME $R_TIME $F_TIME $YEAR $DAY
```

Fix: quote every variable — `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"`.

Locations:

- `.github/workflows/main.yml:25`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses** (main.yml + pr-check.yml): Pinned all 5 action references to full 40-char SHAs:
   - `josStorer/get-current-time@master` → `@49693c19176a68a5ecb39f75dae82364aee49a9a # master`
   - `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4` (3 occurrences)
   - `codecov/codecov-action@v3` → `@ab904c41d6ece82784817410c45d8b8c02684457 # v3`

2. **missing-permissions** (main.yml + pr-check.yml): Added `permissions: {}` top-level block to both workflow files, granting no permissions beyond what is strictly required.

3. **script-injection** (main.yml line 25): Quoted all five shell variables in the `run:` step — changed `echo $TIME $R_TIME $F_TIME $YEAR $DAY` to `echo "$TIME" "$R_TIME" "$F_TIME" "$YEAR" "$DAY"` to prevent shell metacharacter interpretation.

