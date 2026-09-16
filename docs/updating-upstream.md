# Updating from upstream

This repository is an Alpheya-owned mirror of the Envoy AI Gateway controller,
upstream [`theagentrouter/agent-router`](https://github.com/theagentrouter/agent-router)
(formerly `envoyproxy/ai-gateway`). It is not a GitHub fork: upstream history is
pushed in and Alpheya changes are carried as commits on top of an upstream
release tag. The Go module path stays `github.com/envoyproxy/ai-gateway` so
upstream commits cherry-pick cleanly.

The sibling repo `quantum-wealth/ai-gateway` is the ext_proc policy brain and is
unrelated code; it is not affected by syncs here.

## How to sync

```sh
git remote add upstream https://github.com/theagentrouter/agent-router.git
git fetch upstream --tags
git checkout -b chore/sync-vX.Y.Z main
git merge --no-ff vX.Y.Z        # or cherry-pick a range
```

Resolve conflicts per the list below, run `make build.controller` and
`go test ./internal/controller/...`, then open a PR.

## Alpheya differences

Keep these on conflict; do not let upstream overwrite them.

### Azure managed-identity token provider

Upstream PR theagentrouter/agent-router#2378, not yet merged. Adds
`managedIdentity` under `BackendSecurityPolicy.spec.azureCredentials` so the
controller mints Azure tokens through workload identity instead of a client
secret. Files:

- `api/v1alpha1/backendsecurity_policy.go`, `api/v1beta1/backendsecurity_policy.go` and their `zz_generated.deepcopy.go`
- `internal/controller/backend_security_policy.go` (+ test)
- `internal/controller/tokenprovider/azure_managed_identity_token_provider.go` (+ test)
- `manifests/charts/ai-gateway-crds-helm/templates/aigateway.envoyproxy.io_backendsecuritypolicies.yaml`
- `tests/crdcel/main_test.go` and `tests/crdcel/testdata/backendsecuritypolicies/azure_managed_identity_*.yaml`
- `examples/basic/azure_openai_managed_identity.yaml`, `examples/basic/README.md`, `site/docs/api/api.mdx`

Once the upstream PR ships in a release, sync to that release and drop this
section; the mirror can then be retired in favour of the released image.

### `.github/`

Upstream workflows, issue templates, PR template and codecov config are
removed. Only files that exist here are updated on sync; if upstream re-adds a
deleted file, `git rm` it. CI is wired through `quantum-wealth/alpheya-shared-ci`.

### `CODEOWNERS`

Points at `@quantum-wealth/alpheya-devsecops-codeowners`, not the upstream
maintainers team.

## History

| Upstream tag | Released | Synced | Notes |
|---|---|---|---|
| v1.1.0 | 2026-08-21 | initial seed | Baseline plus the managed-identity patch |
