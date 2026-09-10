---
paths:
  - ".github/workflows/**"
  - "ci/**"
  - "Makefile"
---
# CI Rules

## Two CI Systems

- **GitHub Actions** (`.github/workflows/`): upstream-style checks. The primary required gate is `quality-gate.yml` which runs `make setup`, `make build`, `make test-static`.
- **OpenShift Prow** (`ci/prow.Makefile`): downstream CI. Always applies `patches/` before running builds or tests.

## Quality Gate

The single required PR check is the `quality-gate` job. It exercises the same commands documented in AGENTS.md:

```bash
make setup       # bootstrap tools
make build       # compile binaries
make test-static # test-sanity + test-unit + test-docs
```

## Security Scanning

`security.yml` runs `govulncheck`, OSV scanning, and Trivy Dockerfile scanning. Exceptions must be documented in `docs/security-exceptions.md`.

## Prow Patches

Downstream Prow CI applies patches from `patches/` before any build step. If you modify a Makefile target that Prow uses, check whether a patch modifies that same target. See [docs/patterns/downstream-patches.md](../docs/patterns/downstream-patches.md).
