<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v6.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--checkout/v6.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Unpinned references found:
- check-dist.yml: actions/checkout@v6, actions/setup-node@v4, actions/upload-artifact@v4
- codeql-analysis.yml: actions/checkout@v6, github/codeql-action/init@v3, github/codeql-action/analyze@v3
- licensed.yml: actions/checkout@v6
- publish-immutable-actions.yml: actions/checkout@v6, actions/publish-immutable-action@0.0.3
- test.yml: actions/setup-node@v4, actions/checkout@v6 (multiple)
- update-main-version.yml: actions/checkout@v6
- update-test-ubuntu-git.yml: actions/checkout@v6, docker/login-action@v3.3.0, docker/build-push-action@v6.5.0

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:41`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/licensed.yml:9`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:20`
- `.github/workflows/update-main-version.yml:23`
- `.github/workflows/update-test-ubuntu-git.yml:26`
- `.github/workflows/update-test-ubuntu-git.yml:31`
- `.github/workflows/update-test-ubuntu-git.yml:47`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside run: shell command strings, enabling script injection.

(a) update-main-version.yml: workflow_dispatch inputs are interpolated directly into git shell commands without quoting or env-var indirection:
  `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}`
  `run: git push origin ${{ github.event.inputs.major_version }} --force`
An attacker with permission to trigger the workflow could inject arbitrary shell commands via the `target` or `major_version` inputs.

(a) test.yml: step outputs are interpolated directly into a run: shell block:
  `echo "Commit: ${{ steps.checkout.outputs.commit }}"`
  `if [ "${{ steps.checkout.outputs.ref }}" != "test-data/v2/basic" ]; then`
  `if [ "${{ steps.checkout.outputs.commit }}" != "..." ]; then`
Any expression inside ${{ }} is expanded by the template engine before the shell sees it, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/update-main-version.yml:31`
- `.github/workflows/test.yml:218`
- `.github/workflows/test.yml:219`
- `.github/workflows/test.yml:221`
- `.github/workflows/test.yml:226`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (often broad) permissions, violating the principle of least privilege.
- check-dist.yml: no permissions defined at top level or job level
- licensed.yml: no permissions defined at top level or job level
- test.yml: no permissions defined at top level or on any of its jobs (build, test, test-proxy, test-bypass-proxy, test-git-container, test-output)
- update-main-version.yml: no permissions defined at top level or job level

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 7 workflow files:

1. unpinned-uses: Pinned all 8 unique action references to full 40-char commit SHAs with tag comments for readability. Used lookup_action_sha to resolve real SHAs (actions/publish-immutable-action resolved as v0.0.3 since 0.0.3 tag didn't exist).

2. script-injection: In update-main-version.yml, moved github.event.inputs.major_version and github.event.inputs.target out of git tag/push run: commands into env: blocks. In test.yml test-output job, moved steps.checkout.outputs.commit and steps.checkout.outputs.ref into env: block as CHECKOUT_COMMIT and CHECKOUT_REF.

3. missing-permissions: Added top-level permissions blocks to check-dist.yml (contents: read), licensed.yml (contents: read), test.yml (contents: read), and update-main-version.yml (contents: write, needed to push tags). codeql-analysis.yml, publish-immutable-actions.yml, and update-test-ubuntu-git.yml already had job-level permissions blocks.

