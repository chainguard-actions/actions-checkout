<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v7.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--checkout/v7.0.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: workflow_dispatch inputs are directly interpolated into run: shell commands without going through env: variables. In 'Tag new target', `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` are passed directly to `git tag -f`, and in 'Push new tag', `${{ github.event.inputs.major_version }}` is passed directly to `git push origin`. An attacker with write access (or a maintainer tricked into running the workflow) could inject arbitrary shell commands via these inputs.

Locations:

- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/update-main-version.yml:32`

### script-injection (severity: high)

Rule (a) violation: step outputs are directly interpolated into a run: shell block. In the 'Verify output' step, `${{ steps.checkout.outputs.commit }}` and `${{ steps.checkout.outputs.ref }}` are interpolated directly into shell commands (echo and if-condition strings). The `steps.*.outputs.*` context flows through YAML template substitution before the shell sees it, making it a script-injection risk.

Locations:

- `.github/workflows/test.yml:248`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or compromised. Failing references include: check-dist.yml: actions/checkout@v7, actions/setup-node@v6, actions/upload-artifact@v7; codeql-analysis.yml: actions/checkout@v7, github/codeql-action/init@v4, github/codeql-action/analyze@v4; licensed.yml: actions/checkout@v7; publish-immutable-actions.yml: actions/checkout@v7, actions/publish-immutable-action@v0.0.4; test.yml: actions/setup-node@v6, actions/checkout@v7; update-main-version.yml: actions/checkout@v7; update-test-ubuntu-git.yml: actions/checkout@v7, docker/login-action@v4.4.0, docker/build-push-action@v7.3.0.

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:38`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/codeql-analysis.yml:48`
- `.github/workflows/licensed.yml:10`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:18`
- `.github/workflows/update-main-version.yml:26`
- `.github/workflows/update-test-ubuntu-git.yml:28`
- `.github/workflows/update-test-ubuntu-git.yml:33`
- `.github/workflows/update-test-ubuntu-git.yml:50`

### missing-permissions (severity: medium)

check-dist.yml has no top-level `permissions:` key and the single job 'check-dist' has no job-level `permissions:` key. Without explicit permissions, the workflow runs with the default GITHUB_TOKEN permissions which may be overly broad (write access to contents and packages by default on some repository configurations).

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

licensed.yml has no top-level `permissions:` key and the single job 'test' has no job-level `permissions:` key. Without explicit permissions, the workflow runs with default GITHUB_TOKEN permissions which may be overly broad.

Locations:

- `.github/workflows/licensed.yml:1`

### missing-permissions (severity: medium)

test.yml has no top-level `permissions:` key and none of its jobs (build, test, test-proxy, test-bypass-proxy, test-git-container, test-output) have job-level `permissions:` keys. Without explicit permissions, all jobs run with default GITHUB_TOKEN permissions which may be overly broad.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

update-main-version.yml has no top-level `permissions:` key and the single job 'tag' has no job-level `permissions:` key. This workflow pushes tags to the repository and should explicitly declare the minimum required permissions (e.g., `contents: write`).

Locations:

- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across 6 workflow files:

1. script-injection in update-main-version.yml: Moved github.event.inputs.major_version and github.event.inputs.target from direct shell interpolation into env: blocks.

2. script-injection in test.yml: Moved steps.checkout.outputs.commit and steps.checkout.outputs.ref from direct shell interpolation into an env: block.

3. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v7, actions/setup-node@v6, actions/upload-artifact@v7, github/codeql-action/init@v4, github/codeql-action/analyze@v4, actions/publish-immutable-action@v0.0.4, docker/login-action@v4.4.0, docker/build-push-action@v7.3.0.

4. missing-permissions: Added permissions blocks to check-dist.yml (contents: read), licensed.yml (contents: read), test.yml (contents: read at top level), and update-main-version.yml (contents: write, required for pushing tags). codeql-analysis.yml and publish-immutable-actions.yml already had job-level permissions.

