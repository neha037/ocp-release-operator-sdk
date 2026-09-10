---
status: Accepted
applies_to: ocp-release-operator-sdk
---
# ADR-0003: Release Signing and Build Automation

## Status

Accepted

## Context

The Operator SDK produces two CLI binaries and several container images. Releases must be verifiable and reproducible. The downstream build uses OpenShift Prow CI, while upstream uses GitHub Actions with goreleaser.

## Decision

1. **Use goreleaser for upstream-style releases** with checksums for all binary artifacts. The goreleaser configuration lives in `release/`.
2. **Inject version information via ldflags** at build time into `internal/version/`. Five variables (`Version`, `GitVersion`, `GitCommit`, `KubernetesVersion`, `ImageVersion`) are set in the Makefile.
3. **Downstream releases override the version** to append `-ocp` via `patches/03-setversion.patch`, so downstream binaries are clearly distinguishable from upstream.
4. **Container images are built with `CGO_ENABLED=0`** for static binaries. Dockerfiles live in `images/`.
5. **Release commits update `IMAGE_VERSION`** in the Makefile and must pass `make prerelease` validation.

## Consequences

- Release artifacts have checksums for integrity verification.
- Downstream builds are clearly versioned with the `-ocp` suffix.
- The version patch must be updated on each upstream merge to reference the new version.
- goreleaser is fetched at build time, introducing a supply-chain dependency (mitigated by version pinning in `tools/scripts/fetch`).

## Alternatives Considered

- **Manual release process:** Rejected for lack of reproducibility and auditability.
- **Using `go install` for distribution:** Does not produce container images or cross-compiled binaries.
- **Embedding version at source level:** Would cause merge conflicts on every upstream sync; ldflags injection avoids this.
