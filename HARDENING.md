<!-- markdownlint-disable -->

# Hardening Report: joutvhu--get-release/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **joutvhu--get-release/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: `${{ steps.current_release.outputs.tag_name }}` is interpolated directly inside a `run:` shell command string. A release tag name is attacker-controlled (anyone who creates a release can set the tag), so this allows arbitrary shell command injection. The offending line is: `echo "::set-output name=version::$(echo ${{ steps.current_release.outputs.tag_name }} | cut -f 1 -d .)"`. The value should be passed via an `env:` variable and double-quoted in the shell script instead.

Locations:

- `.github/workflows/update-tag.yml:19`

### unpinned-uses (severity: high)

All `uses:` references in workflow files use mutable version tags instead of full 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: auto-build.yml: `actions/checkout@v4`, `actions/setup-node@v4`; latest-testing.yml: `actions/checkout@v4`; release-testing.yml: `actions/checkout@v4`; update-tag.yml: `joutvhu/get-release@v1`, `joutvhu/create-tag@v1`.

Locations:

- `.github/workflows/auto-build.yml:12`
- `.github/workflows/auto-build.yml:14`
- `.github/workflows/latest-testing.yml:11`
- `.github/workflows/release-testing.yml:12`
- `.github/workflows/update-tag.yml:11`
- `.github/workflows/update-tag.yml:24`

### missing-permissions (severity: medium)

These workflow files have no `permissions:` key at the top level and no `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), granting unnecessarily broad access. A minimal `permissions:` block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/latest-testing.yml:1`
- `.github/workflows/release-testing.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:
1. script-injection (update-tag.yml line 19): Moved `${{ steps.current_release.outputs.tag_name }}` into an `env:` block as `TAG_NAME` and referenced it as `"$TAG_NAME"` in the shell script.
2. unpinned-uses: Pinned all 6 action references to full SHAs — actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262, actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020, joutvhu/get-release@v1 → @8c3531bba0d249f5e7c3d7e1954d34db9edf2a90, joutvhu/create-tag@v1 → @e5e6b59bb6ad5604c09de0409f1798edea88ae22.
3. missing-permissions: Added `permissions: contents: read` at the top level of latest-testing.yml and release-testing.yml.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/update-tag.yml line 22. The original code used `echo "::set-output name=version::$(echo "$TAG_NAME" | cut -f 1 -d .)"` which wrote a value derived from the untrusted `tag_name` output without stripping newlines. The fix replaces this with `safe=$(printf '%s' "$TAG_NAME" | tr -d '\n\r' | cut -f 1 -d .)` followed by `echo "version=$safe" >> "$GITHUB_OUTPUT"`, which sanitizes the value by removing newline and carriage-return characters before writing to the output, and also modernizes from the deprecated `::set-output` syntax to `$GITHUB_OUTPUT`.

