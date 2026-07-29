<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--checkout/v6** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of user-controlled `workflow_dispatch` inputs inside `run:` shell commands. In `update-main-version.yml`, `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}` are interpolated directly into `git tag` and `git push` commands, allowing an attacker with workflow_dispatch access to inject arbitrary shell commands. Example offending lines: `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}` and `run: git push origin ${{ github.event.inputs.major_version }} --force`.

Locations:

- `.github/workflows/update-main-version.yml:31`
- `.github/workflows/update-main-version.yml:33`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of step outputs inside a `run:` shell command. In `test.yml`, `${{ steps.checkout.outputs.commit }}` and `${{ steps.checkout.outputs.ref }}` are interpolated directly into shell commands in the 'Verify output' step. Step outputs are workflow-controllable data and must not be interpolated directly into run: blocks. Offending lines include: `echo "Commit: ${{ steps.checkout.outputs.commit }}"`, `echo "Ref: ${{ steps.checkout.outputs.ref }}"`, and comparisons using `${{ steps.checkout.outputs.ref }}` and `${{ steps.checkout.outputs.commit }}`.

Locations:

- `.github/workflows/test.yml:243`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of full 40-character SHA digests. Unpinned references are vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised. Failing references: `actions/checkout@v6`, `actions/setup-node@v4`, `actions/upload-artifact@v4`.

Locations:

- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:27`
- `.github/workflows/check-dist.yml:44`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of full 40-character SHA digests. Failing references: `actions/checkout@v6`, `github/codeql-action/init@v3`, `github/codeql-action/analyze@v3`.

Locations:

- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:54`

### unpinned-uses (severity: high)

Workflow file references an action using a mutable tag instead of a full 40-character SHA digest. Failing reference: `actions/checkout@v6`.

Locations:

- `.github/workflows/licensed.yml:10`

### unpinned-uses (severity: high)

Workflow file references actions using mutable tags or version strings instead of full 40-character SHA digests. Failing references: `actions/checkout@v6`, `actions/publish-immutable-action@0.0.3`.

Locations:

- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`

### unpinned-uses (severity: high)

Workflow file references actions using mutable tags instead of full 40-character SHA digests. Failing references include `actions/setup-node@v4` and `actions/checkout@v6` (used many times throughout the file).

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:20`

### unpinned-uses (severity: high)

Workflow file references an action using a mutable tag instead of a full 40-character SHA digest. Failing reference: `actions/checkout@v6`.

Locations:

- `.github/workflows/update-main-version.yml:24`

### unpinned-uses (severity: high)

Workflow file references actions using mutable tags or version strings instead of full 40-character SHA digests. Failing references: `actions/checkout@v6`, `docker/login-action@v3.3.0`, `docker/build-push-action@v6.5.0`.

Locations:

- `.github/workflows/update-test-ubuntu-git.yml:31`
- `.github/workflows/update-test-ubuntu-git.yml:35`
- `.github/workflows/update-test-ubuntu-git.yml:52`

### missing-permissions (severity: medium)

Workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/licensed.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (build, test, test-proxy, test-bypass-proxy, test-git-container, test-output). Without explicit permissions, all jobs inherit the repository's default token permissions.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level `permissions:` key and no job-level `permissions:` key on the `tag` job. Without explicit permissions, the job inherits the repository's default token permissions, which may be overly broad — especially concerning given this workflow pushes tags.

Locations:

- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across 6 workflow files:

1. script-injection (update-main-version.yml): Moved github.event.inputs.major_version and github.event.inputs.target from direct run: interpolation into env: blocks.

2. script-injection (test.yml): Moved steps.checkout.outputs.commit and steps.checkout.outputs.ref from direct run: interpolation into an env: block (CHECKOUT_COMMIT, CHECKOUT_REF).

3. unpinned-uses: Pinned all action references to full SHA digests:
   - actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v4 → 49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/upload-artifact@v4 → ea165f8d65b6e75b540449e92b4886f43607fa02
   - github/codeql-action/init@v3 → 4187e74d05793876e9989daffde9c3e66b4acd07
   - github/codeql-action/analyze@v3 → 4187e74d05793876e9989daffde9c3e66b4acd07
   - actions/publish-immutable-action@0.0.3 → 4b1aa5c1cde5fedc80d52746c9546cb5560e5f53 (resolved via v0.0.3)
   - docker/login-action@v3.3.0 → 9780b0c442fbb1117ed29e0efdff1e18412f7567
   - docker/build-push-action@v6.5.0 → 5176d81f87c23d6fc96624dfdbcd9f3830bbe445

4. missing-permissions: Added permissions blocks:
   - check-dist.yml: contents: read (top-level)
   - licensed.yml: contents: read (top-level)
   - test.yml: contents: read (top-level)
   - update-main-version.yml: contents: write (top-level, needed for pushing tags)

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned `docker://bitnami/git:latest` to `docker://bitnami/git:latest@sha256:c02bdac0976e9edc06f5f4b262686b7c93192431d6eeffb7484ec664467a0c2f` in `.github/workflows/test.yml` at line 148. The `docker://` scheme and `:latest` tag are preserved inline with the digest appended, following the required format for container action references.

