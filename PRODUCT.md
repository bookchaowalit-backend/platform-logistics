# Product

<!-- impeccable:product-schema 1 -->

## Platform

contract (Book Platform boundary, `book-platform.contract.v1`) with a workspace-scoped HTTP runtime

## Users

- The owner and the agents that change the Book Platform: they read this boundary before
  touching the `logistics` capability anywhere else.
- Other platform repositories and cells, which may depend on this boundary only through its
  versioned API or event contracts, never its database or private modules.

## Product Purpose

Own the `logistics` capability (carrier adapters, shipment, shipping rates, labels, tracking, fulfillment) as a written, checked contract and a runtime that
serves it per workspace, so the capability can move out of the `solo-empire` control plane
without consumers noticing.

Success: the contract stays valid against the pinned shared schema, the registry entry matches
it, the runtime suite passes against PostgreSQL in CI, and the deployed API answers `/readyz`.

## Positioning

A platform boundary, not an end-user product. It depends on identity, observability; its current
implementation in the parent registry is nothing (a new boundary). Migration gates before any cutover:
carrier sandbox, label replay, tracking deduplication.

## Operating Context

- Contract status: `scaffolded` (no cutover claimed). The runtime runs on Railway
  (`asia-southeast1`, sleeps when idle) at https://logistics-api-production-d3bb.up.railway.app with its own Neon project; it is in
  `repository-catalog/registries/online-systems.yaml` and the uptime worker. Only the sandbox carrier exists; there is no real carrier integration or credential.
- Tokens come from platform-identity (audience `logistics-api`).
- CI: shared `ci-python` on the locked environment (`uv.lock`: Ruff, mypy, the whole pytest
  suite against PostgreSQL 16 under a coverage gate, the wheel build), the offline contract
  check on Python 3.11 and 3.12, the image build (`shared-runtime.yml`) and the shared security
  workflow, on GitHub-hosted runners (public repository). Deploys are Railway CLI uploads.
- Sentry: `src/platform_logistics/_sentry.py` reports errors only, with no free text and no
  request data; DSN in Railway and Infisical `/platform-logistics`.

## Capabilities and Constraints

- `scripts/check.py` validates `contract.json` against the vendored schema;
  `scripts/check_schema_pin.py` checks the vendored schema against the pinned canonical hash.
- The runtime uses `platform-backend-core` at the reviewed commit pinned in `pyproject.toml`
  and `uv.lock`; the Dockerfile builds from the same commit.
- Out of scope: becoming the system of record before the migration gates pass.

## Measures

- CI green on `main` (target: always); coverage at or above the gate in `check.yml`.
- Drift between `contract.json` and the registry entry (target: 0).
- `/readyz` 200 from the uptime worker (target: every hourly check).
