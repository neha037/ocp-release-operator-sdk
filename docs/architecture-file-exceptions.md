# File Size Exceptions

Files listed here are exempt from the 600-line limit enforced by `hack/check-file-size.sh`. Each exception requires justification and a tracking issue or plan for eventual refactoring.

## Active Exceptions

| File | Lines | Justification |
|---|---|---|
| `internal/generate/clusterserviceversion/clusterserviceversion_updaters.go` | ~695 | CSV update logic is tightly coupled; splitting would fragment the update transaction boundary. Upstream sync risk. |
| `internal/olm/operator/registry/index_image.go` | ~680 | Registry index operations form a single workflow; refactor deferred to avoid upstream merge conflicts. |
| `hack/generate/samples/internal/go/memcached-with-customization/memcached_with_customization.go` | ~1313 | Sample generator with embedded Go templates; splitting would fragment template coherence and break `make generate`. |
