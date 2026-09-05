# AI Agents Orientation

This repository is governed by the rules in the
[swe-playbook--docs](https://github.com/tibuqx/swe-playbook--docs) repository.
Please read the root `README.md` for architectural context and component boundaries.

## Repository Profile

| Field | Value |
|---|---|
| **Name** | wasmcloud-pipeline--workflow |
| **Description** | Reusable GitHub Actions CI/CD workflow orchestrating ADR-0046 compliant 3-stage environment promotion (integration, staging, production) for wasmCloud applications and components. |
| **Type** | workflow |
| **System** | ci-cd-pipelines |
| **Owner** | group:default/architecture |
| **Lifecycle** | production |

## Stack & Technologies

- **CI/CD Platform**: GitHub Actions reusable workflows (`workflow_call`)
- **WebAssembly & wasmCloud**: `wash-cli` (v0.39+), WADM (OAM application deployment manager), NATS messaging control plane
- **Toolchain**: Rust toolchain (`cargo`, `wasm32-wasip2` target, `rustfmt`, `clippy`)
- **Registry**: GitHub Container Registry (GHCR) OCI distribution
- **Static Analysis & Linting**: `actionlint` with ShellCheck integration

## Key Directories

| Directory | Purpose |
|---|---|
| `.github/workflows/` | Reusable workflow definitions (`wasmcloud-pipeline.yml`) and self-test CI (`actionlint.yml`) |

## Build & Test Commands

| Task | Command |
|---|---|
| Workflow Linting | `actionlint .github/workflows/*.yml` |
| Reusable Workflow Call | Invocation via `uses: tibuqx/wasmcloud-pipeline--workflow/.github/workflows/wasmcloud-pipeline.yml@v1` |

## Conventions & Boundaries

- **Pure GitHub Flow (ADR-0046)**: Branch topology consists strictly of `main` and short-lived `feat/*` branches. No `develop` or `release/*` branches.
- **3-Stage Promotion**: Pull Request triggers automated CI and gates deployment to `integration`. Merge to `main` builds release artifacts and deploys to `staging`. Production deployment is gated via `needs: [deploy-staging]` and requires explicit manual approval via GitHub Actions Environment protection rules on `production`.
- **Caller Single Pipeline Rule (ADR-0026)**: Caller repositories must maintain at most one workflow file (`pipeline.yml`) in `.github/workflows/`.
- **Repository Suffix Policy (ADR-0028)**: Reusable workflow repositories must carry the `--workflow` suffix.
- **Backstage Catalog (ADR-0030)**: In-repository `catalog-info.yaml` with `spec.type: workflow` registered under `system: ci-cd-pipelines`.
- **Automated Semantic Versioning (ADR-0031)**: Production releases delegate to `tibuqx/versioning--workflow/.github/workflows/release.yml@1.0.0` based on Conventional Commits.
- **Security Baseline (ADR-0034)**: Security gates fail closed; unit tests, SDD engine purity checks, and dependency audits are enforced before any deployment.
- **Mandatory Human PR Approval**: Direct pushes to `main` are prohibited; auto-merge is forbidden; explicit human approval is required on all PRs.
- **Zero Secrets**: Secrets are never hardcoded and are isolated per GitHub Environment (`integration`, `staging`, `production`).

## ADR Compliance

- **ADR-0006**: AI-friendly documentation and machine-readable metadata.
- **ADR-0024**: Protected branches and PR-only merges.
- **ADR-0026**: Single `pipeline.yml` in callers delegating to centralized reusable workflows.
- **ADR-0028**: Mandatory repository `--<type>` suffix (`--workflow`).
- **ADR-0030**: Canonical software catalog registration in Backstage (`catalog-info.yaml`).
- **ADR-0031**: Shared versioning workflow integration for post-deploy releases.
- **ADR-0034**: Automated fail-closed security gates in CI baseline.
- **ADR-0035**: Mandatory root `README.md` with Mermaid C4 diagram and `AGENTS.md`.
- **ADR-0040 / ADR-0042**: wasmCloud runtime and WASM component model compilation targets.
- **ADR-0043**: Volatility-Based Decomposition validation in CI.
- **ADR-0046**: Pure GitHub Flow branching model and 3-stage environment promotion.
