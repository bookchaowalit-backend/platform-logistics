# 06 — C4 Container

```mermaid
C4Container
  title Container view - logistics-api
  Person(caller, "Caller", "JWT with ws + scopes")
  System_Ext(gw, "Edge API gateway", "Cloudflare + Railway gateway, route /v1/platform-logistics")
  System_Ext(identity, "platform-identity", "JWKS")
  System_Ext(sentry, "Sentry", "platform-logistics project")
  Container_Boundary(railway, "Railway service logistics-api, asia-southeast1, sleep on") {
    Container(api, "API process", "Python 3.12, FastAPI on platform-backend-core, uvicorn", "python -m platform_logistics serve, PORT 8080")
    Container(sandbox, "SandboxCarrier", "In-process Python object", "Deterministic quotes, label ledger in memory")
  }
  Container(migrate, "Migrate one-off", "Same image, command override", "python -m platform_logistics migrate with DDL role")
  ContainerDb(db, "Neon PostgreSQL 16", "Tables shipment, shipment_label, tracking_event, platform_schema_migrations", "Forced RLS on app.workspace_id")
  Rel(caller, gw, "HTTPS JSON (MCP hub path)")
  Rel(gw, api, "HTTPS JSON, token passed through")
  Rel(caller, api, "HTTPS JSON direct Railway URL (still open)")
  Rel(api, sandbox, "quote, purchase_label, find_label")
  Rel(api, db, "psycopg pool 8, runtime role NOSUPERUSER NOBYPASSRLS")
  Rel(migrate, db, "DDL role")
  Rel(api, identity, "JWKS fetch")
  Rel(api, sentry, "Scrubbed error events")
```

Sources: ops record: api-gateway-edge (2026-11-03); `bookchaowalit-backend/platform-api-gateway@51c4711` `routes.json`; `bookchaowalit/platform-mcp-remote-worker@7d127c2` `wrangler.toml` `CELLS` + `src/worker.ts` `PLATFORM_CELLS`, `Dockerfile`, `.railway/railway.ts`, `src/platform_logistics/__main__.py`, `src/platform_logistics/app.py`, `src/platform_logistics/carriers.py`, `src/platform_logistics/migrations.py`, `docs/runtime.md`, `pyproject.toml`, ops record: book-platforms-status (2026-11-03) (region, sleep, Neon project, Sentry project)

## Open questions

- Neon project `platform-logistics` มี compute/branch และ region ใด — ops record ระบุเพียงรูปแบบชื่อ `platform-<svc>`
- `migrate` ถูกรันอย่างไรบน Railway (one-off service? local?) — ไม่มีบันทึก
