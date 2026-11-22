# sing-box-patch

A patch-maintained fork of [SagerNet/sing-box](https://github.com/SagerNet/sing-box), preserving the upstream source code and module identity while maintaining targeted modifications for integration with [V2bX](https://github.com/wyx2685/V2bX).

## Overview

`sing-box-patch` is designed to maintain a small, controlled set of patches on top of upstream sing-box releases.

Instead of duplicating individual packages or types into V2bX, this project keeps the complete sing-box source tree and applies custom patches directly to it. This allows V2bX and sing-box to share the same Go type identities, avoiding compatibility issues caused by duplicated named types across different import paths.

The goal is to keep customizations minimal, traceable, and easy to rebase or migrate onto future upstream releases.

## Goals

- Preserve the complete upstream sing-box source tree and its history.
- Retain the original upstream Go module path to maintain type identity and compatibility.
- Maintain V2bX integration patches separately from upstream code.
- Keep custom changes small, focused, and easy to review.
- Make upstream version upgrades and patch migration predictable.
- Provide a local development workflow using Go's `replace` directive.
- Avoid unnecessary divergence from upstream.

## Upstream

| Item                | Value                                                     |
| ------------------- | --------------------------------------------------------- |
| Upstream repository | [SagerNet/sing-box](https://github.com/SagerNet/sing-box) |
| Upstream module     | `github.com/sagernet/sing-box`                            |
| Initial baseline    | `v1.13.0`                                                 |
| License             | See upstream license files                                |

This repository is based on upstream sing-box. The upstream module path and relevant licensing notices are retained.

## Branch Strategy

This project uses a lightweight branch model. The `main` branch is dedicated to project documentation and maintenance guidelines; it does not represent a code release branch.

| Branch         | Purpose                                                             |
| -------------- | ------------------------------------------------------------------- |
| `main`         | Project documentation, maintenance policies, and development guides |
| `patch/vX.Y.Z` | Upstream-version-specific patch branch                              |
| `feature/*`    | Temporary branches for larger changes                               |
| `fix/*`        | Temporary branches for isolated bug fixes                           |

### Patch Branches

Each patch branch is based on a specific upstream release tag.

For example:

- `patch/v1.13.0` is based on upstream `v1.13.0`.
- `patch/v1.14.0` is based on upstream `v1.14.0`.

Patch branches contain the upstream source plus only the modifications required by this project.

Different upstream versions are maintained independently. A new patch branch is created when adopting a new upstream release, rather than merging the entire previous patch branch into it.

## Patch Principles

All custom modifications should follow these principles:

1. **Upstream first** — Avoid modifying code that does not need customization.
2. **Minimal changes** — Keep patches as small and focused as practical.
3. **One concern per commit** — Each commit should represent a single logical change.
4. **Traceability** — Clearly identify why a patch exists and what it changes.
5. **Compatibility** — Preserve upstream behavior unless a change is explicitly required.
6. **Portability** — Design patches to be easy to migrate to newer upstream versions.
7. **No unnecessary vendoring or type duplication** — Prefer sharing upstream packages and types.

## Development Workflow

All code changes must be submitted to the corresponding `patch/vX.Y.Z` branch through Pull Requests. Direct commits to patch branches are not permitted.

This applies to all types of changes:

- **New patches** — changes developed from scratch.
- **Migrated patches** — changes cherry-picked or ported from an earlier upstream-based patch branch.
- **Bug fixes** — fixes and refinements to existing patches.
- **Maintenance changes** — updates or adjustments to previously integrated patches.

The goal is to keep every change entering a patch branch reviewable, traceable, and documented through the GitHub Pull Request history.

### 1. Development Branches

Create a temporary branch from the target patch branch for each change.

```bash
git switch patch/v1.13.0
git pull --ff-only origin patch/v1.13.0

git switch -c feature/my-change
```

Use appropriate branch prefixes:

- `feature/*` — new features or patches
- `fix/*` — bug fixes
- `maintenance/*` — maintenance or refactoring

### 2. Make and Commit Changes

Implement the change and commit it on the temporary branch. Each commit should represent a logical, self-contained change.

```bash
git add .
git commit -m "feat: add custom integration support"
```

For migrated patches, cherry-pick the required commit(s) onto the temporary branch created from the target patch branch, review the changes, and adjust them as necessary.

```bash
git cherry-pick <commit>
```

Cherry-picking changes from another branch does not bypass the Pull Request workflow.

### 3. Submit a Pull Request

Push the temporary branch and open a Pull Request targeting the corresponding patch branch.

```bash
git push -u origin feature/my-change
```

All changes must go through a PR, regardless of whether they are newly developed, cherry-picked from another version, or bug fixes to existing patches.

Each PR should clearly describe:

- The purpose and motivation of the change.
- What upstream files or components are affected.
- The actual modifications introduced.
- Any relevant upstream commits or PRs, if the change was migrated.
- Testing performed and potential compatibility implications.

### 4. Review and Merge

Review the PR to ensure the patch is necessary, minimal, compatible with the target upstream version, and does not introduce unrelated modifications.

Once approved and checks have passed, merge the PR into the target patch branch.

The patch branch should only receive changes through reviewed PRs. Do not push commits directly to `patch/vX.Y.Z`.

### 5. Bug Fixes and Follow-up Changes

Bug fixes or improvements to an existing patch must follow the same workflow.

Create a new `fix/*` branch from the current patch branch, implement the fix, and submit a PR. Avoid silently modifying existing patches directly on the patch branch.

This ensures that the history reflects not only the original implementation, but also subsequent fixes and refinements.

### 6. Upstream Version Migration

When upgrading to a new upstream version:

1. Create a new `patch/vX.Y.Z` branch from the official upstream release tag.
2. Identify patches that are still required.
3. Cherry-pick or port the required patches from the previous patch branch onto temporary migration branches.
4. Review and resolve any conflicts or compatibility changes.
5. Submit PRs targeting the new patch branch.
6. Merge each PR after review and testing.

Do not merge the previous patch branch wholesale into the new upstream branch. Each migrated patch should be independently reviewable, making it easier to identify which customizations are retained, modified, replaced, or dropped.

### Workflow Summary

All changes follow the same contribution path:

`New change / Cherry-pick / Bug fix / Maintenance`
→ `Temporary branch`
→ `Commit`
→ `Push`
→ `Pull Request`
→ `Review & Testing`
→ `Merge into patch/vX.Y.Z`

The patch branch is the maintained integration baseline, while Pull Requests serve as the audit trail for everything added, modified, or removed from that baseline.

## Release Workflow

Releases are created from tested `patch/vX.Y.Z` branches. A release represents a verified snapshot of the upstream version together with the patches maintained by this project.

### 1. Release Requirements

Before creating a release:

- All intended changes must have been merged through Pull Requests.
- The target patch branch must be in a stable state.
- Relevant build and test checks must pass.
- The patch inventory and documentation must reflect the changes included in the release.
- No uncommitted or unreviewed changes should be included.

Releases must be created explicitly. Merging a PR into a patch branch does not automatically create a release.

### 2. Release Versioning

Use the following format for release tags:

`v<upstream-version>-patch.<revision>`

Examples:

- `v1.13.0-patch.1` — first release based on upstream `v1.13.0`.
- `v1.13.0-patch.2` — subsequent release containing additional fixes or patches.
- `v1.14.0-patch.1` — first release based on upstream `v1.14.0`.

The upstream version identifies the base sing-box release. The patch revision identifies the release iteration maintained by this project.

Patch revisions are incremented when a new release is published for the same upstream version.

### 3. Prepare a Release

Check out the corresponding patch branch and ensure it is up to date.

```bash
git switch patch/v1.13.0
git pull --ff-only origin patch/v1.13.0
```

Review the changes included since the previous release:

```bash
git log --oneline --decorate
git diff <previous-release-tag>..HEAD
```

Verify the upstream base version, patch inventory, build status, and test results before proceeding.

### 4. Create a Release Tag

Create an annotated tag on the commit intended for release.

```bash
git tag -a v1.13.0-patch.1 -m "Release v1.13.0-patch.1"
git push origin v1.13.0-patch.1
```

Release tags must point to the exact commit that was tested. Published release tags should not be moved or overwritten.

### 5. Publish GitHub Release

Create a GitHub Release using the pushed tag.

Release notes should include:

- Upstream sing-box version.
- Release version.
- Summary of included patches and changes.
- Bug fixes and improvements.
- Compatibility notes or known issues, if applicable.
- Relevant Pull Request references.

Keep the release notes concise and focused on what changed from the previous release.

### 6. Subsequent Releases

When additional patches or bug fixes are merged into the same upstream patch branch, publish a new patch revision after validation.

For example:

`v1.13.0-patch.1` → `v1.13.0-patch.2`

Do not modify or reuse an existing release tag.

### 7. New Upstream Versions

When moving to a newer upstream version, create and maintain a new patch branch based on the official upstream release.

Once the new branch has been migrated, reviewed, and tested, publish its first release independently.

For example:

`patch/v1.13.0` → `v1.13.0-patch.1`

`patch/v1.14.0` → `v1.14.0-patch.1`

Releases from different upstream versions are maintained independently. The latest release is not necessarily the numerically newest patch revision across all upstream versions.

## Documentation

Project documentation is maintained on the `main` branch.

Documentation may include:

- Architecture and design decisions
- Development environment setup
- Patch inventory and rationale
- Contribution and commit guidelines
- Upstream upgrade procedures
- V2bX integration notes

Documentation changes should not be mixed into upstream code patch commits unless the documentation specifically belongs to the patched source itself.

## License & Attribution

This repository is derived from sing-box and retains upstream source code, copyright notices, and applicable license files.

Please refer to the upstream repository and the license files included in this repository for licensing terms.

All custom modifications should respect the upstream license and applicable attribution requirements.
