# Operator SDK CLI

## Overview

`operator-sdk` is the developer CLI for scaffolding, building, validating, and running Kubernetes operators. It wraps kubebuilder's CLI framework (`cli.New()`) and injects SDK-specific commands and plugins.

Entry point: `cmd/operator-sdk/main.go` -> `internal/cmd/operator-sdk/cli/cli.go`

## Plugin System

The CLI uses kubebuilder's plugin architecture. SDK-specific plugins live in `internal/plugins/` and use the `.sdk.operatorframework.io` naming suffix. Plugins implement kubebuilder interfaces with compile-time assertions.

| Plugin | Path | Purpose |
|---|---|---|
| Helm v1 | `internal/plugins/helm/v1/` | Scaffold Helm-based operators |
| Manifests v2 | `internal/plugins/manifests/v2/` | Generate OLM bundle manifests |
| Scorecard v2 | `internal/plugins/scorecard/v2/` | Scaffold scorecard test configs |

## Command Tree

Extra commands (not plugin-based) are added via `WithExtraCommands`:

| Command | Package | Purpose |
|---|---|---|
| `bundle` | `internal/cmd/operator-sdk/bundle/` | Validate, create OLM bundles |
| `run` | `internal/cmd/operator-sdk/run/` | Run operators (bundle, bundle-upgrade) |
| `cleanup` | `internal/cmd/operator-sdk/cleanup/` | Remove OLM-managed operators |
| `olm` | `internal/cmd/operator-sdk/olm/` | Install, uninstall, status of OLM |
| `generate` | `internal/cmd/operator-sdk/generate/` | Generate bundle, kustomize manifests |
| `scorecard` | `internal/cmd/operator-sdk/scorecard/` | Run conformance tests |
| `pkgmantobundle` | `internal/cmd/operator-sdk/pkgmantobundle/` | Migrate package manifests to bundles |

## Command Function Pattern

All commands follow the `NewCmd() *cobra.Command` convention:

```go
func NewCmd() *cobra.Command {
    c := cmdState{}
    cmd := &cobra.Command{
        Use:   "subcommand",
        Short: "Short description",
        RunE:  c.run,
    }
    cmd.Flags().StringVar(&c.flagName, "flag-name", "", "Flag description")
    return cmd
}
```

Flag state lives in a private struct constructed inside `NewCmd()`. Do not use global variables for flags.

## Logging

CLI commands use `github.com/sirupsen/logrus`. Do not use `sigs.k8s.io/controller-runtime/pkg/log` (logr) in CLI code.
