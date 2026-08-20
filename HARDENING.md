<!-- markdownlint-disable -->

# Hardening Report: peter-evans--commit-comment/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--commit-comment/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In update-major-version.yml, two run: steps directly interpolate ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} into shell commands. These are workflow_dispatch inputs that can be supplied by any user with dispatch access, and the values are passed directly to git tag and git push without any quoting or sanitization. This allows shell metacharacter injection. Offending lines:
  - `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` (rule a)
  - `run: git push origin ${{ github.event.inputs.main_version }} --force` (rule a)

Locations:

- `.github/workflows/update-major-version.yml:28`
- `.github/workflows/update-major-version.yml:30`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Failing references include:
  - automerge-dependabot.yml: peter-evans/enable-pull-request-automerge@v3
  - ci.yml: actions/checkout@v5, actions/setup-node@v5, actions/upload-artifact@v4 (×2), actions/download-artifact@v5 (×2), peter-evans/create-pull-request@v7
  - slash-command-dispatch.yml: peter-evans/slash-command-dispatch@v4
  - update-major-version.yml: actions/checkout@v5

Locations:

- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:22`
- `.github/workflows/ci.yml:29`
- `.github/workflows/ci.yml:31`
- `.github/workflows/ci.yml:44`
- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:95`
- `.github/workflows/ci.yml:97`
- `.github/workflows/ci.yml:100`
- `.github/workflows/slash-command-dispatch.yml:9`
- `.github/workflows/update-major-version.yml:19`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: block and no job-level permissions: blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be read/write for contents), violating the principle of least privilege.
  - automerge-dependabot.yml: no permissions declared
  - slash-command-dispatch.yml: no permissions declared
  - update-major-version.yml: no permissions declared

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (update-major-version.yml): Moved ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} into env: blocks as MAIN_VERSION and TARGET, then referenced them as double-quoted shell variables in the run: steps.

2. unpinned-uses: Pinned all 9 action references to full 40-character commit SHAs with tag comments preserved: actions/checkout@v5 (×4), actions/setup-node@v5 (×1), actions/upload-artifact@v4 (×2), actions/download-artifact@v5 (×3), peter-evans/create-pull-request@v7 (×1), peter-evans/enable-pull-request-automerge@v3 (×1), peter-evans/slash-command-dispatch@v4 (×1).

3. missing-permissions: Added top-level permissions blocks to automerge-dependabot.yml (pull-requests: write), slash-command-dispatch.yml (issues: read, pull-requests: read), and update-major-version.yml (contents: write). ci.yml already had a permissions block.

