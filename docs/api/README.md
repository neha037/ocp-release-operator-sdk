# API Surface

The Operator SDK's product surface consists of CLI binaries (`operator-sdk` and `helm-operator`) and Go packages under `internal/` (not exported as a library). There is no HTTP API, REST service, or OpenAPI/Swagger specification.

A stub OpenAPI 3.1 spec is provided at [openapi.yaml](openapi.yaml) documenting this intentional absence. If the project adds an HTTP API surface in the future, that file should be replaced with a real specification.
