# Helm Operator

## Overview

`helm-operator` is a runtime binary deployed inside a container that reconciles Helm-based operators. It uses raw Cobra and controller-runtime directly (not kubebuilder's CLI framework).

Entry point: `cmd/helm-operator/main.go` -> `internal/cmd/helm-operator/run/`

## How It Works

1. Reads `watches.yaml` to discover which GVKs to watch and which Helm charts to reconcile.
2. Sets up a controller-runtime manager.
3. Registers a per-GVK controller using `internal/helm/controller/`.
4. Each controller uses `internal/helm/release/` to manage Helm releases.

## Key Packages

| Package | Purpose |
|---|---|
| `internal/cmd/helm-operator/run/` | CLI entry point, manager setup |
| `internal/helm/controller/` | Reconciler: watches CRs, manages Helm releases |
| `internal/helm/release/` | Helm release management (install, upgrade, uninstall) |
| `internal/helm/watches/` | `watches.yaml` parsing and validation |
| `internal/helm/client/` | Helm action client factory |

## Logging

The helm-operator uses `sigs.k8s.io/controller-runtime/pkg/log` (logr interface). Do not use `github.com/sirupsen/logrus` in controller/runtime code.

## watches.yaml

Defines which GVK maps to which Helm chart:

```yaml
- group: example.com
  version: v1alpha1
  kind: MyApp
  chart: helm-charts/myapp
```

The operator watches CRs of this GVK and manages corresponding Helm releases.
