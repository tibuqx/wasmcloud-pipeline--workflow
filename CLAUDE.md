# CLAUDE.md — Instructions for AI Developers

This repository contains **`wasmcloud-pipeline--workflow`**, the organizational reusable GitHub Actions CI/CD workflow for wasmCloud applications and WebAssembly components.

## Repository Mission & Scope

This repository provides:
1. `.github/workflows/wasmcloud-pipeline.yml`: The canonical reusable workflow (`workflow_call`) implementing ADR-0046 pure GitHub Flow with a 3-stage promotion lifecycle (`integration`, `staging`, `production`).
2. `.github/workflows/actionlint.yml`: The CI validation workflow that enforces syntax and expression validity on all workflows.
3. `catalog-info.yaml`: Backstage catalog metadata (`kind: Component`, `spec.type: workflow`).
4. Comprehensive architectural documentation (`README.md` with C4 diagram per ADR-0035, `AGENTS.md` per ADR-0006).

## Key Architectural Principles

1. **ADR-0046 (GitHub Flow)**:
   - Only `main` and short-lived `feat/*` branches exist.
   - Pull Requests to `main` trigger CI baseline verification and automatic deployment to the `integration` environment upon passing.
   - Merging to `main` builds release WASM artifacts, publishes them to GHCR, and deploys to `staging`.
   - Production deployment is strictly sequenced after staging (`needs: [deploy-staging]`) and gated by GitHub Actions Environment protection rules on `production` (requiring manual review and approval).
2. **ADR-0026 (Single Pipeline Rule)**:
   - Caller repositories (`tx-functions--microservice`, etc.) maintain a single `.github/workflows/pipeline.yml` that delegates CI/CD lifecycle tasks to this reusable workflow.
3. **Fail-Closed Security (ADR-0034)**:
   - Baseline CI runs formatting checks, clippy linting, SDD engine purity checks, unit tests, and live database integration tests. Any failure immediately cancels deployment.
4. **Environment Secrets Isolation**:
   - Environment-scoped credentials (e.g. lattice NATS credentials, OCI registry tokens) are segregated by GitHub Environment (`integration`, `staging`, `production`).

## Validation Commands

Before committing any workflow modifications, run actionlint:
```powershell
& "C:\Users\Luis Manuel\go\bin\actionlint.exe" d:/tibuqx/repos/wasmcloud-pipeline--workflow/.github/workflows/*.yml
```
Ensure that 0 errors and 0 warnings are emitted.

## Interface Contract Summary

Callers invoke this reusable workflow via:
```yaml
jobs:
  wasmcloud-pipeline:
    uses: tibuqx/wasmcloud-pipeline--workflow/.github/workflows/wasmcloud-pipeline.yml@v1
    with:
      rust-version: "stable"
      wasm-target: "wasm32-wasip2"
      wadm-manifest-path: "deploy/wadm.yaml"
      database-compose-file: "deploy/docker-compose.db.yml"
      run-database-tests: true
      health-endpoint-staging: "https://staging-api.tibuqx.com/health"
      health-endpoint-production: "https://api.tibuqx.com/health"
    secrets: inherit
```
