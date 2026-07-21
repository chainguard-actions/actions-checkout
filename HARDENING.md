<!-- markdownlint-disable -->

# Hardening Report: actions--checkout/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--checkout/v7.0.0** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ github.event.inputs.major_version }} and ${{ github.event.inputs.target }} are directly interpolated into run: shell commands. These are workflow_dispatch inputs that an authorized user controls and they flow through YAML template substitution before the shell sees them, enabling shell metacharacter injection. Offending lines: `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}` and `run: git push origin ${{ github.event.inputs.major_version }} --force`.

Locations:

- `.github/workflows/update-main-version.yml:30`
- `.github/workflows/update-main-version.yml:32`

### script-injection (severity: high)

Rule (a) violation: ${{ steps.checkout.outputs.ref }} and ${{ steps.checkout.outputs.commit }} are directly interpolated inside a run: shell block. Step outputs are workflow-controllable context values that flow through YAML template substitution before the shell sees them. Offending lines include: `echo "Commit: ${{ steps.checkout.outputs.commit }}"`, `echo "Ref: ${{ steps.checkout.outputs.ref }}"`, and conditional comparisons using the same expressions.

Locations:

- `.github/workflows/test.yml:248`

### unpinned-uses (severity: high)

All uses: references in check-dist.yml use mutable tags instead of full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced tags are moved. Unpinned references: actions/checkout@v6, actions/setup-node@v4, actions/upload-artifact@v4.

Locations:

- `.github/workflows/check-dist.yml:21`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:40`

### unpinned-uses (severity: high)

All uses: references in codeql-analysis.yml use mutable tags instead of full 40-character SHA digests. Unpinned references: actions/checkout@v6, github/codeql-action/init@v3, github/codeql-action/analyze@v3.

Locations:

- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:47`

### unpinned-uses (severity: high)

The uses: reference in licensed.yml uses a mutable tag instead of a full 40-character SHA digest. Unpinned reference: actions/checkout@v6.

Locations:

- `.github/workflows/licensed.yml:11`

### unpinned-uses (severity: high)

All uses: references in publish-immutable-actions.yml use mutable tags instead of full 40-character SHA digests. Unpinned references: actions/checkout@v6, actions/publish-immutable-action@v0.0.4.

Locations:

- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`

### unpinned-uses (severity: high)

Multiple uses: references in test.yml use mutable tags instead of full 40-character SHA digests. Unpinned references: actions/setup-node@v4, actions/checkout@v6 (used in multiple jobs: build, test, test-proxy, test-bypass-proxy, test-git-container, test-output).

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:36`

### unpinned-uses (severity: high)

The uses: reference in update-main-version.yml uses a mutable tag instead of a full 40-character SHA digest. Unpinned reference: actions/checkout@v6.

Locations:

- `.github/workflows/update-main-version.yml:24`

### unpinned-uses (severity: high)

All uses: references in update-test-ubuntu-git.yml use mutable tags instead of full 40-character SHA digests. Unpinned references: actions/checkout@v6, docker/login-action@v3.3.0, docker/build-push-action@v6.5.0.

Locations:

- `.github/workflows/update-test-ubuntu-git.yml:29`
- `.github/workflows/update-test-ubuntu-git.yml:34`
- `.github/workflows/update-test-ubuntu-git.yml:55`

### missing-permissions (severity: medium)

check-dist.yml has no top-level permissions: block and its only job (check-dist) also has no job-level permissions: block. Without explicit permissions, the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

licensed.yml has no top-level permissions: block and its only job (test) also has no job-level permissions: block. Without explicit permissions, the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/licensed.yml:1`

### missing-permissions (severity: medium)

test.yml has no top-level permissions: block and none of its jobs (build, test, test-proxy, test-bypass-proxy, test-git-container, test-output) have job-level permissions: blocks. Without explicit permissions, all jobs run with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

update-main-version.yml has no top-level permissions: block and its only job (tag) also has no job-level permissions: block. Without explicit permissions, the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/update-main-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 13 findings across 7 workflow files:

1. script-injection (update-main-version.yml): Moved github.event.inputs.major_version and github.event.inputs.target into env: blocks, referenced as $MAJOR_VERSION and $TARGET in shell.

2. script-injection (test.yml): Moved steps.checkout.outputs.commit and steps.checkout.outputs.ref into env: block (CHECKOUT_COMMIT, CHECKOUT_REF), referenced as plain env vars in shell.

3. unpinned-uses: Pinned all mutable tag references to full 40-char SHAs:
   - actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v4 → 49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/upload-artifact@v4 → ea165f8d65b6e75b540449e92b4886f43607fa02
   - github/codeql-action/init@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - github/codeql-action/analyze@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - actions/publish-immutable-action@v0.0.4 → 4bc8754ffc40f27910afb20287dbbbb675a4e978
   - docker/login-action@v3.3.0 → 9780b0c442fbb1117ed29e0efdff1e18412f7567
   - docker/build-push-action@v6.5.0 → 5176d81f87c23d6fc96624dfdbcd9f3830bbe445

4. missing-permissions: Added top-level permissions blocks:
   - check-dist.yml: contents: read
   - licensed.yml: contents: read
   - test.yml: contents: read
   - update-main-version.yml: contents: write (needed to push tags)

