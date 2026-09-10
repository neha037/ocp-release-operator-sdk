---
status: Accepted
applies_to: ocp-release-operator-sdk
---
# ADR-0001: Downstream Mirror and Patch Isolation

## Status

Accepted

## Context

This repository is the OpenShift downstream fork of `operator-framework/operator-sdk`. Upstream releases are merged periodically, and downstream-specific adaptations (CI, image references, build adjustments) must survive these merges without causing persistent conflicts.

## Decision

1. **Keep the upstream module path** (`github.com/operator-framework/operator-sdk`) to minimize import churn during merges.
2. **Use commit message prefixes** to classify downstream changes:
   - `UPSTREAM: <carry>:` for changes that must persist across upstream merges.
   - `UPSTREAM: <drop>:` for changes regenerated on each merge (vendor updates, generated code).
3. **Isolate CI adaptations in `patches/`** using numbered `diff -up` format patch files applied before downstream CI builds via `make -f ci/prow.Makefile patch`. This avoids modifying upstream Makefile targets directly.
4. **Separate vendor updates** into their own `UPSTREAM: <drop>:` commits so they can be trivially regenerated after a merge.

## Consequences

- Upstream merges are cleaner because carry commits are clearly identified and re-applicable.
- The patch system introduces a maintenance burden: patches can silently break if the upstream file changes near the patched region.
- Contributors must learn the carry/drop convention and the patch creation workflow.

## Alternatives Considered

- **Maintaining a permanent fork with no upstream tracking:** Rejected because it would prevent adopting upstream fixes and features.
- **Using git rebase instead of merge:** Rejected because rebase rewrites history, complicating collaboration on the downstream branch.
- **Applying downstream changes directly to source files instead of patches:** Rejected because it creates persistent merge conflicts in CI-related Makefile targets.
