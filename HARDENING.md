<!-- markdownlint-disable -->

# Hardening Report: joutvhu--get-release/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **joutvhu--get-release/v1.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v2`, which is a mutable tag rather than a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved or the upstream repository is compromised.

Locations:

- `.github/workflows/latest-testing.yml:13`
- `.github/workflows/release-testing.yml:16`

### missing-permissions (severity: medium)

Neither `.github/workflows/latest-testing.yml` nor `.github/workflows/release-testing.yml` declares a top-level `permissions:` block, and no job in either file has a job-level `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/latest-testing.yml:1`
- `.github/workflows/release-testing.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed both workflow files (.github/workflows/latest-testing.yml and .github/workflows/release-testing.yml):
1. Pinned `actions/checkout@v2` to its full commit SHA `0717577d45739eb3c851188b29f50ed6c0b2194e` with a `# v2` comment for readability.
2. Added `permissions: {}` top-level block to both files to enforce least privilege, as neither workflow requires any GitHub token permissions beyond what is explicitly passed via `secrets.GITHUB_TOKEN` in env blocks.

