---
status: Accepted
applies_to: ocp-release-operator-sdk
---
# ADR-0002: Vendored Dependency Policy

## Status

Accepted

## Context

The upstream `operator-sdk` repository does not vendor dependencies (uses module proxy at build time). The downstream OpenShift build infrastructure requires vendored dependencies for reproducible, air-gapped builds.

## Decision

1. **Commit the `vendor/` directory** to the repository. This diverges from upstream but is required by the downstream build system.
2. **Vendor updates are separate commits** with the prefix `UPSTREAM: <drop>: Update vendor directory`. These commits are regenerated on each upstream merge.
3. **After any `go.mod` change**, always run `go mod tidy` followed by `go mod vendor` and commit vendor changes separately.
4. **Do not add `vendor/` to `.gitignore`.**

## Consequences

- Repository size is significantly larger due to vendored code.
- Builds are reproducible without network access.
- Dependency updates require two commits: one for `go.mod`/`go.sum`, one for `vendor/`.
- Linters and checks must exclude `vendor/` from analysis.

## Alternatives Considered

- **Module proxy caching:** Would reduce repo size but does not meet the air-gapped build requirement of the downstream CI environment.
- **Vendoring only in CI:** Would reduce the burden on local development but create a divergence between local and CI builds that leads to subtle failures.
