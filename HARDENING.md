<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--checkout/v6.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions and Docker images using mutable tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- check-dist.yml: actions/checkout@v6, actions/setup-node@v4, actions/upload-artifact@v4
- codeql-analysis.yml: actions/checkout@v6, github/codeql-action/init@v3, github/codeql-action/analyze@v3
- licensed.yml: actions/checkout@v6
- publish-immutable-actions.yml: actions/checkout@v6, actions/publish-immutable-action@0.0.3
- test.yml: actions/setup-node@v4, actions/checkout@v6, docker://bitnami/git:latest (mutable image tag)
- update-main-version.yml: actions/checkout@v6
- update-test-ubuntu-git.yml: actions/checkout@v6, docker/login-action@v3.3.0, docker/build-push-action@v6.5.0

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/licensed.yml:8`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-main-version.yml:24`
- `.github/workflows/update-test-ubuntu-git.yml:28`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions.
- check-dist.yml: jobs `check-dist` has no permissions
- licensed.yml: job `test` has no permissions
- test.yml: jobs `build`, `test`, `test-proxy`, `test-bypass-proxy`, `test-git-container`, `test-output` all have no permissions
- update-main-version.yml: job `tag` has no permissions

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-main-version.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing injection of arbitrary shell commands.

1. update-main-version.yml — The `Tag new target` step interpolates `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` (attacker-controlled `workflow_dispatch` inputs) directly into shell commands:
   `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}`
   `run: git push origin ${{ github.event.inputs.major_version }} --force`
   An attacker with write access could inject arbitrary git or shell commands via these inputs.

2. test.yml — The `Verify output` step interpolates `${{ steps.checkout.outputs.commit }}` and `${{ steps.checkout.outputs.ref }}` (step outputs, which flow through YAML template substitution before the shell sees them) directly into a `run:` block:
   `echo "Commit: ${{ steps.checkout.outputs.commit }}"`
   `if [ "${{ steps.checkout.outputs.ref }}" != "test-data/v2/basic" ]; then`

Locations:

- `.github/workflows/update-main-version.yml:28`
- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/test.yml:222`
- `.github/workflows/test.yml:223`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments. Also pinned bitnami/git:latest container image to its sha256 digest in both docker:// uses and container: job field.

2. missing-permissions: Added top-level `permissions: contents: read` to check-dist.yml, licensed.yml, and test.yml. Added `permissions: contents: write` to update-main-version.yml (required for git push/tag operations). Files that already had job-level permissions (codeql-analysis.yml, publish-immutable-actions.yml, update-test-ubuntu-git.yml) were left with their existing job-level blocks.

3. script-injection: In update-main-version.yml, moved ${{ github.event.inputs.major_version }} and ${{ github.event.inputs.target }} into step env: blocks and referenced them as $MAJOR_VERSION and $TARGET in shell. In test.yml Verify output step, moved ${{ steps.checkout.outputs.commit }} and ${{ steps.checkout.outputs.ref }} into env: block as CHECKOUT_COMMIT and CHECKOUT_REF, referenced as plain env vars in shell script.

Note: actions/publish-immutable-action@0.0.3 was resolved as v0.0.3 (with 'v' prefix) → SHA 4b1aa5c1cde5fedc80d52746c9546cb5560e5f53.

