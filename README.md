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

| Item | Value |
|---|---|
| Upstream repository | [SagerNet/sing-box](https://github.com/SagerNet/sing-box) |
| Upstream module | `github.com/sagernet/sing-box` |
| Initial baseline | `v1.13.0` |
| License | See upstream license files |

This repository is based on upstream sing-box. The upstream module path and relevant licensing notices are retained.

## Branch Strategy

This project uses a lightweight branch model. The `main` branch is dedicated to project documentation and maintenance guidelines; it does not represent a code release branch.

| Branch | Purpose |
|---|---|
| `main` | Project documentation, maintenance policies, and development guides |
| `patch/vX.Y.Z` | Upstream-version-specific patch branch |
| `feature/*` | Temporary branches for larger changes |
| `fix/*` | Temporary branches for isolated bug fixes |

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

### 1. Clone the Repository

```bash
git clone https://github.com/proxynav/sing-box-patch.git
cd sing-box-patch
```

### 2. Select an Upstream Baseline

Fetch upstream tags:

```bash
git fetch upstream --tags
```

Check available versions:

```bash
git tag --list 'v1.13*'
```

### 3. Create a Patch Branch

Create a branch from the desired upstream release:

```bash
git switch -c patch/v1.13.0 v1.13.0
```

### 4. Implement Changes

Make the required changes directly on the patch branch, or use a temporary feature branch for larger work.

Commit each logical change separately:

```bash
git add .
git commit -m "fix: resolve V2bX type compatibility"
```

### 5. Build and Test

Run the relevant checks before considering a patch complete:

```bash
go build ./...
go test ./...
```

Additional integration and runtime tests may be required depending on the patch.

### 6. Push the Patch Branch

```bash
git push -u origin patch/v1.13.0
```

### 7. Tag Tested Patch Releases

Create tags only for tested and usable patch snapshots.

```bash
git tag -a v1.13.0-patch.1 -m "sing-box v1.13.0 with custom patches"
git push origin v1.13.0-patch.1
```

## Upstream Upgrade & Patch Migration

When a new upstream release becomes available, create a new patch branch from that release.

For example, upgrading from `v1.13.0` to `v1.14.0`:

### 1. Fetch Upstream

```bash
git fetch upstream --tags
```

### 2. Create the New Patch Branch

```bash
git switch -c patch/v1.14.0 v1.14.0
```

The new branch starts from the original upstream release and does not contain patches from the previous branch.

### 3. Review Existing Patches

List the custom commits on the previous patch branch:

```bash
git log --reverse --oneline v1.13.0..patch/v1.13.0
```

### 4. Migrate Required Patches

Apply patches individually in chronological order:

```bash
git cherry-pick <commit-id>
```

Review each patch during migration. Some patches may already be implemented upstream, may no longer be needed, or may require adjustments due to API and internal changes.

### 5. Resolve Conflicts

If conflicts occur:

```bash
git status
```

Resolve the affected files, then continue:

```bash
git add <resolved-files>
git cherry-pick --continue
```

To abort the current migration:

```bash
git cherry-pick --abort
```

### 6. Validate the New Branch

Run build, tests, and integration checks again. Do not assume a patch that worked on a previous upstream version remains valid.

### 7. Publish the New Patch Branch

```bash
git push -u origin patch/v1.14.0
```

The previous patch branch remains available for maintenance and reference.

## Integrating with V2bX

During local development, V2bX can use this repository through a local Go module replacement.

Assuming both repositories are checked out in the same parent directory:

```text
~/proj/
├── V2bX/
└── sing-box-patch/
```

Add the following to V2bX's `go.mod`:

```go
replace github.com/sagernet/sing-box => ../sing-box-patch
```

The replacement target must match the upstream module path declared in sing-box's `go.mod`.

This local replacement allows V2bX to use the modified sing-box source directly without publishing a module version for every development change.

For production or CI builds, use a controlled and reproducible dependency configuration, pinning the intended patch revision.

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
