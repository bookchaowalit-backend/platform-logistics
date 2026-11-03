# 05 — C4 Context

```mermaid
C4Context
  title System Context - logistics-api
  Person(owner, "Owner", "Person token from identity, not yet active")
  System_Ext(hub, "Remote MCP hub", "Service client mcp-hub, cell platform-logistics")
  System_Ext(bridge, "Carrier-ingest bridge", "Planned holder of tracking:write; none exists")
  System(logi, "logistics-api", "Rates, shipments, labels, tracking per workspace")
  System_Ext(identity, "platform-identity", "Issues RS256 JWT, publishes JWKS")
  SystemDb_Ext(neon, "Neon PostgreSQL", "Own project per service")
  System_Ext(sentry, "Sentry", "Errors only, no free text")
  System_Ext(uptime, "platform-uptime-worker", "Hourly GET /readyz")
  Rel(owner, logi, "HTTPS + Bearer JWT")
  Rel(hub, logi, "HTTPS + client_credentials JWT")
  Rel(bridge, logi, "POST /v1/tracking-events")
  Rel(logi, identity, "Fetch JWKS")
  Rel(logi, neon, "SQL over TLS, runtime role")
  Rel(logi, sentry, "Error events")
  Rel(uptime, logi, "GET /readyz")
```

Sources: `src/platform_logistics/__main__.py` (`JWT_JWKS_URL`, `DATABASE_URL`), `src/platform_logistics/_sentry.py`, `docs/runtime.md`, `PRODUCT.md` (Operating Context), ops record: book-platforms-status (2026-11-03)

Carrier จริงไม่อยู่ใน context เพราะ `SandboxCarrier` ทำงานใน process (`carriers.py`: "no network, no credentials").

Central architecture: [C4-DIAGRAMS.md](https://github.com/bookchaowalit/architecture-docs/blob/main/docs/architecture/C4-DIAGRAMS.md) ไม่ได้กล่าวถึง logistics (ตรวจด้วย grep)

## Open questions

- ผ่าน edge gateway หรือไม่ — ops record (book-platforms-status) ไม่ระบุ gateway route ของ logistics
