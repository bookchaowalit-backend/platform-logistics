# 07 — C4 Component (API process)

```mermaid
C4Component
  title Components inside logistics-api
  Container_Boundary(api, "API process") {
    Component(core, "platform_backend_core.create_app", "Shared core", "Boundary: body limit 16 KiB, timeout, redacted 500, /healthz /readyz /metrics")
    Component(auth, "require_scope + TokenVerifier", "Shared core", "RS256, iss/aud, ws claim, scopes; 401/403")
    Component(capture, "CaptureErrors", "_sentry.py", "Innermost ASGI layer, reports unhandled errors")
    Component(routes, "Routes + pydantic models", "app.py", "Strict bodies, generic exception handlers")
    Component(store, "LogisticsStore", "store.py", "Idempotency, label lease, reconcile, tracking dedup, readiness")
    Component(rules, "rules", "rules.py", "Parcel/address validation, billable weight, TRANSITIONS")
    Component(carriers, "carriers", "carriers.py", "Carrier protocol, SandboxCarrier, configured_carriers")
    Component(errors, "InvalidInput", "errors.py", "Only domain error mapped to 422")
    Component(mig, "MIGRATIONS 0001", "migrations.py", "Tables + forced RLS policies")
  }
  ComponentDb(db, "PostgreSQL", "Neon")
  Rel(core, capture, "wraps")
  Rel(capture, routes, "wraps")
  Rel(routes, auth, "Depends")
  Rel(routes, store, "calls")
  Rel(store, rules, "validates")
  Rel(store, carriers, "calls adapter")
  Rel(store, errors, "raises")
  Rel(store, db, "Database.transaction(workspace)")
  Rel(store, mig, "ready() compares checksums")
```

Sources: `src/platform_logistics/app.py`, `src/platform_logistics/store.py`, `src/platform_logistics/rules.py`, `src/platform_logistics/carriers.py`, `src/platform_logistics/errors.py`, `src/platform_logistics/migrations.py`, `src/platform_logistics/_sentry.py`, `docs/runtime.md` (shared core limits)

หมายเหตุ: ลำดับ middleware มาจากคอมเมนต์ใน `app.py` ("Last in the list = innermost: inside the core boundary that turns errors into a 500").
