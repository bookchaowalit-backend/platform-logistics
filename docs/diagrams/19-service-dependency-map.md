# 19 — Service dependency map

```mermaid
flowchart TB
  subgraph Runtime["Runtime dependencies of logistics-api"]
    LOGI["logistics-api"]
    PG[("Neon PostgreSQL - hard: /readyz 503 without it")]
    JWKS["platform-identity JWKS - hard for every authenticated call"]
    SBX["SandboxCarrier in process - hard: serve refuses without a carrier"]
    SEN["Sentry - soft: no-op without SENTRY_DSN"]
  end
  subgraph Inbound["Callers"]
    HUB["Remote MCP hub cell platform-logistics"]
    OWNER["Owner person token (pending Google client id)"]
    UPT["platform-uptime-worker"]
  end
  subgraph Build["Build-time"]
    CORE["platform-backend-core at 21d5f44 (Git)"]
    PYPI["PyPI: uvicorn, sentry-sdk, fastapi via core"]
    CIW["bookchaowalit/platform-ci-workflows v1.1.1"]
  end
  HUB --> LOGI
  OWNER --> LOGI
  UPT --> LOGI
  LOGI --> PG
  LOGI --> JWKS
  LOGI --> SBX
  LOGI -.-> SEN
  CORE --> LOGI
  PYPI --> LOGI
  CIW -.-> LOGI
```

Sources: `pyproject.toml`, `Dockerfile`, `.github/workflows/check.yml`, `src/platform_logistics/__main__.py`, `src/platform_logistics/app.py`, `src/platform_logistics/carriers.py`, `src/platform_logistics/_sentry.py`, `contract.json` (`depends_on: identity, observability`), ops record: book-platforms-status (2026-11-03)

- `contract.json` ระบุ `observability` เป็น dependency แต่ไม่มีโค้ดเรียก observability library; สิ่งที่มีคือ `/metrics` ของ core และ Sentry. ops record: observability เป็น library ไม่ใช่ service.
- ไม่มี service-to-service call ขาออก (ไม่มี `ServiceTokenSource`; core ยัง pin `21d5f44`).
- ไม่มี CDN.

## Open questions

- consumer ในอนาคต (commerce?) จะเรียกผ่าน REST + identity service client ตามแนวทาง "Service↔service stays REST" ใน ops record หรือไม่
