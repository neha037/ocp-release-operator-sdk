# Performance Guidelines

## Concurrency Defaults

- `--max-concurrent-reconciles` defaults to `runtime.NumCPU()`. On large nodes this can overload the API server. Set it explicitly based on workload, not host capacity. Defined in `internal/helm/flags/flag.go`.

- The Helm operator creates one `controller.Options{MaxConcurrentReconciles: N}` per GVK watch entry. Multiple watched GVKs multiply the effective concurrency. Account for all entries in `watches.yaml` when sizing.

## Mutex and Sync Primitives

- `sync.RWMutex` protects the dependent-resource watch map in `internal/helm/controller/controller.go`. Use `RLock` for the "already watched?" check and promote to `Lock` only to register a new watch. Do not hold the write lock across the `c.Watch(...)` call -- the existing code does this correctly; preserve that pattern when adding new watched kinds.

- `sync.Mutex` in `actionConfigGetter` (`internal/helm/client/actionconfig.go`) guards per-namespace `WatchedSecrets` creation. The lock scope must stay narrow -- acquire, check-or-create, release, then return. Do not put network calls inside this lock.

- `sync.Once` is the standard pattern for lazy one-time initialization (discovery client caching in `restclientgetter.go`, OLM installer client in `installer/manager.go`). Prefer `sync.Once` over double-checked locking for any setup-once resource.

## Retry and Polling Conventions

### retry.RetryOnConflict
Always use `retry.DefaultBackoff` when wrapping `Client.Update` or `Client.Status().Update` calls. This is the established pattern across the codebase for handling Kubernetes conflict (409) errors:
```go
retry.RetryOnConflict(retry.DefaultBackoff, func() error {
    return r.Client.Update(ctx, o)
})
```
Do not use custom backoff parameters for conflict retries; `DefaultBackoff` is consistent across all call sites.

### wait.PollUntilContextCancel
Polling intervals follow this convention by resource type:
- **Pod readiness checks**: 200ms
- **Resource creation/deletion waits**: 100ms-1s depending on expected latency
- **Deployment rollout / CSV phase**: 1s
- **Cache deletion confirmation**: 10ms with a 5s hard timeout

Always pass `false` for the `immediate` parameter (third arg) to avoid running the condition check before the first interval elapses.

## Cache and Informer Configuration

- Namespace-scoped caches use `cache.Config` via `options.Cache.DefaultNamespaces`. When watching specific namespaces, each gets its own `cache.Config{}` entry. When watching all namespaces, a single `metav1.NamespaceAll` entry with `LabelSelector: labels.Everything()` overrides any per-object selectors that would otherwise restrict the cluster-wide watch.

- Per-GVK label selectors are applied through `cache.ByObject` and a global `DefaultLabelSelector` scoped to `helm.sdk.operatorframework.io/chart`. This ensures the shared informer cache only stores objects relevant to the watched charts. Preserve this filtering -- removing it causes the cache to store all cluster objects of those types.

- The Helm secrets informer (`secrets_watch.go`) uses a 30-second resync period and namespace-scoped factories. Each namespace gets its own `SharedInformerFactory` and the factory is started with `wait.NeverStop`. New namespaces lazily create new factories.

## Watch Efficiency

- The `WatchedSecrets` wrapper (`internal/helm/client/secrets_watch.go`) exists specifically to reduce API server load. Helm queries release secrets multiple times per reconciliation. The wrapper intercepts `List` calls matching `owner=helm` and serves them from the informer lister instead of hitting the API server. If a List call includes options beyond a label selector, it falls through to the direct API call. Do not bypass this wrapper.

- Dependent resource watches are deduplicated via a `map[schema.GroupVersionKind]struct{}` guarded by `sync.RWMutex`. A watch is registered at most once per GVK regardless of how many releases include that resource kind.

- Use `predicate.DependentPredicate{}` on all dependent resource watches to filter out events that do not represent meaningful changes.

## Goroutine Patterns

- Scorecard parallel test execution uses `sync.WaitGroup` + buffered channel (`internal/scorecard/scorecard.go`). The channel is pre-allocated to `len(tests)` capacity. Follow this pattern for bounded fan-out: pre-size the channel, launch goroutines, `wg.Wait()`, then close and drain.

- Background streaming (e.g., `storage.go`) launches a goroutine that owns `io.Pipe` writers. Always `defer` closing both `outStream` and `errStream` inside the goroutine to prevent reader hangs.

## Context and Timeout Conventions

- Long-running operations (OLM install/uninstall, scorecard, bundle run) use `context.WithTimeout(context.Background(), timeout)` where timeout defaults to 2 minutes. Always call `defer cancel()` immediately after creating the context.

- Cleanup operations after a primary context expires must use a fresh context: `context.WithTimeout(context.Background(), cleanupTimeout)` with a 30-second ceiling. Do not reuse the expired parent context for cleanup.

- The `waitForDeletion` helper wraps a tight 10ms poll in a 5-second hard timeout. This pattern is appropriate only for cache-backed reads where staleness is the concern, not for waiting on actual API operations.

## Status Update Optimization

- The Helm reconciler compares `status` to `originalStatus` using `reflect.DeepEqual` before issuing a status update. This avoids unnecessary writes when the status has not changed. Apply this pattern in any new reconciler: snapshot the status at the top of `Reconcile`, compare at the end.

## Leader Election

- The resource lock type defaults to `resourcelock.LeasesResourceLock`. Do not change to ConfigMap-based locks as Leases have lower API server overhead and better consistency semantics.

## Logging Performance

- Use `log.V(1).Info(...)` for per-reconcile diagnostics, not `log.Info(...)`. The `V(1)` guard prevents string formatting when debug logging is disabled. The diff output in reconcile.go is additionally guarded by `log.V(1).Enabled()` to avoid computing the diff at all.

- Helm debug logs use a closure (`debugLog` in `actionconfig.go`) that checks `log.Enabled()` before formatting. Follow this pattern when passing log functions to third-party libraries.

## OLM Wait Patterns

- `sync.Once` is used inside `PollUntilContextCancel` callbacks to emit progress logs exactly once per phase transition. This prevents log spam during extended waits (deployment rollout, CSV phase). Use this pattern in any new polling loop that could run for minutes.

- Delete operations use `DeletePropagationBackground` followed by a 100ms poll to confirm actual deletion. This is faster than `Foreground` propagation for bulk teardown and avoids blocking on dependent object deletion.
