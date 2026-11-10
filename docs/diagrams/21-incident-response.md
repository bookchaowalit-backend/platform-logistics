# 21 — Incident response

ไม่มี RUNBOOK ใน repo. Flow ด้านล่างประกอบจากช่องทาง alert ที่มีจริงและขั้นตอน rollback/restore ที่ `docs/runtime.md` ระบุ.

```mermaid
flowchart TD
  A1["Uptime worker: /readyz down - Telegram"] --> T
  A2["Sentry issue in platform-logistics"] --> T
  A3["Caller reports 409 Label outcome uncertain"] --> T
  T{"Triage"}
  T -- "503 readyz" --> R1["Check readiness parts: DB reachable, migration checksums, forced RLS, grants, LOGISTICS_CARRIERS"]
  T -- "500 in Sentry" --> R2["Read type + stack + route (no message text)"]
  T -- "uncertain label" --> R3["POST /v1/shipments/{id}/label/reconcile"]
  R1 --> F1["Fix variable or grant on Railway / Neon, redeploy with railway up --ci"]
  R2 --> F2["Fix on branch, TDD, PR, CI green, merge to main, railway up --ci"]
  R3 --> F3["labelled, or created and safe to retry, or still uncertain"]
  F1 --> V["Verify /readyz 200"]
  F2 --> V
  F3 --> V
  V --> RB{"Need rollback?"}
  RB -- "yes" --> RBA["Stop the optional runtime; never downgrade SQL; restore only into a new database"]
  RB -- "no" --> PM["Postmortem: not defined in repo"]
  RBA --> PM
```

Sources: `docs/runtime.md` (readiness, Rollback, Label purchase reconcile), `src/platform_logistics/store.py` (`ready`, `reconcile_label`), `src/platform_logistics/_sentry.py`, `.railway/railway.ts`, `PRODUCT.md`, ops record: book-platforms-status (2026-11-03) (uptime worker, Sentry, deploy recipe); central [ADR-0002](https://github.com/bookchaowalit/architecture-docs/blob/main/docs/architecture/decisions/ADR-0002-production-baseline-monitoring-backups.md) (Telegram)

## Open questions

- ไม่มี postmortem template / severity ใน repo; ใช้ของกลางที่ใด?
- Sentry alert ไปช่องทางใด (Telegram? email?)
- ใครเป็น on-call เมื่อมี consumer จริง
