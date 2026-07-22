<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v4.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--checkout/v4.3.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the action is compromised.

Failing references:
- check-dist.yml: actions/checkout@v4.1.6, actions/setup-node@v4, actions/upload-artifact@v4
- codeql-analysis.yml: actions/checkout@v4.1.6, github/codeql-action/init@v3, github/codeql-action/analyze@v3
- licensed.yml: actions/checkout@v4.1.6
- publish-immutable-actions.yml: actions/checkout@v4, actions/publish-immutable-action@0.0.3
- test.yml: actions/setup-node@v4, actions/checkout@v4.1.6 (multiple occurrences)
- update-main-version.yml: actions/checkout@v4.1.6
- update-test-ubuntu-git.yml: actions/checkout@v4, docker/login-action@v3.3.0, docker/build-push-action@v6.5.0

Locations:

- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:40`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:39`
- `.github/workflows/codeql-analysis.yml:50`
- `.github/workflows/licensed.yml:10`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:26`
- `.github/workflows/update-main-version.yml:24`
- `.github/workflows/update-test-ubuntu-git.yml:27`
- `.github/workflows/update-test-ubuntu-git.yml:33`
- `.github/workflows/update-test-ubuntu-git.yml:50`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the default (often write) token permissions, violating the principle of least privilege.

- check-dist.yml: no permissions at top-level or on the `check-dist` job
- licensed.yml: no permissions at top-level or on the `test` job
- test.yml: no permissions at top-level or on any of the jobs (build, test, test-proxy, test-bypass-proxy, test-git-container, test-output)
- update-main-version.yml: no permissions at top-level or on the `tag` job

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-main-version.yml:1`

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

1. update-main-version.yml — The `Tag new target` step interpolates `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` (both user-supplied `workflow_dispatch` inputs) directly into `git tag` and `git push` commands:
   `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}`
   `run: git push origin ${{ github.event.inputs.major_version }} --force`
   An attacker with permission to trigger the workflow can inject shell metacharacters.

2. test.yml — The `Verify output` step interpolates `${{ steps.checkout.outputs.ref }}` and `${{ steps.checkout.outputs.commit }}` (step outputs, a workflow-controllable context) directly inside a `run:` block:
   `echo "Commit: ${{ steps.checkout.outputs.commit }}"`
   `if [ "${{ steps.checkout.outputs.ref }}" != "test-data/v2/basic" ]; then`
   These values should be passed via `env:` variables and referenced as quoted shell variables instead.

Locations:

- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/update-main-version.yml:32`
- `.github/workflows/test.yml:222`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 7 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs in check-dist.yml, codeql-analysis.yml, licensed.yml, publish-immutable-actions.yml, test.yml, update-main-version.yml, and update-test-ubuntu-git.yml. SHAs were resolved via lookup_action_sha for each tag/version.

2. missing-permissions: Added top-level `permissions: contents: read` to check-dist.yml, licensed.yml, and test.yml. Added `permissions: contents: write` to update-main-version.yml (required for git push/tag operations). codeql-analysis.yml and publish-immutable-actions.yml already had job-level permissions.

3. script-injection: In update-main-version.yml, moved github.event.inputs.major_version and github.event.inputs.target into env: blocks and referenced as shell variables $MAJOR_VERSION and $TARGET. In test.yml Verify output step, moved steps.checkout.outputs.ref and steps.checkout.outputs.commit into env: block and referenced as $CHECKOUT_REF and $CHECKOUT_COMMIT.

