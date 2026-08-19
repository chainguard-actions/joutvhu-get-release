<!-- markdownlint-disable -->

# Hardening Report: joutvhu--get-release/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **joutvhu--get-release/v1.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In `.github/workflows/update-tag.yml` line 22, `${{ steps.current_release.outputs.tag_name }}` is embedded directly in the shell command `echo "::set-output name=version::$(echo ${{ steps.current_release.outputs.tag_name }} | cut -f 1 -d .)"`. The `steps.*.outputs.*` value flows through YAML template substitution before the shell ever sees it, allowing an attacker who controls the release tag name to inject arbitrary shell commands.

Locations:

- `.github/workflows/update-tag.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- `.github/workflows/auto-build.yml`: `actions/checkout@v4` (line 15), `actions/setup-node@v4` (line 17)
- `.github/workflows/latest-testing.yml`: `actions/checkout@v4` (line 13)
- `.github/workflows/release-testing.yml`: `actions/checkout@v4` (line 15)
- `.github/workflows/update-tag.yml`: `joutvhu/get-release@v1` (line 15), `joutvhu/create-tag@v1` (line 25)

Locations:

- `.github/workflows/auto-build.yml:15`
- `.github/workflows/auto-build.yml:17`
- `.github/workflows/latest-testing.yml:13`
- `.github/workflows/release-testing.yml:15`
- `.github/workflows/update-tag.yml:15`
- `.github/workflows/update-tag.yml:25`

### missing-permissions (severity: medium)

Two workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. `latest-testing.yml` and `release-testing.yml` both lack any permissions declaration.

Locations:

- `.github/workflows/latest-testing.yml:1`
- `.github/workflows/release-testing.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files: (1) script-injection in update-tag.yml: moved `${{ steps.current_release.outputs.tag_name }}` into an env: block as TAG_NAME and referenced it as "$TAG_NAME" in the shell run command; (2) unpinned-uses: pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262, actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, joutvhu/get-release@v1 to SHA 8c3531bba0d249f5e7c3d7e1954d34db9edf2a90, and joutvhu/create-tag@v1 to SHA e5e6b59bb6ad5604c09de0409f1798edea88ae22 across auto-build.yml, latest-testing.yml, release-testing.yml, and update-tag.yml; (3) missing-permissions: added `permissions: {}` top-level blocks to latest-testing.yml and release-testing.yml.

