# 15 — CI/CD pipeline

สอง workflow; ทั้งคู่ trigger `push` (ทุก branch), `pull_request`, `workflow_dispatch`; `permissions: contents: read`; concurrency ต่อ ref แบบ cancel-in-progress. ไม่มี job deploy.

```mermaid
flowchart TD
  T["push / pull_request / workflow_dispatch"] --> CHK["check.yml"]
  T --> SR["shared-runtime.yml"]
  subgraph CHK["check.yml (group check-ref)"]
    C1["contract: matrix Python 3.11, 3.12 - ubuntu-latest, 10 min"]
    C1a["check_schema_pin.py"]
    C1b["scripts/check.sh: check.py, schema pin, unittest tests/"]
    C2["python: reusable ci-python.yml at 87dc5e1 v1.1.1"]
    C2a["uv sync --frozen, Ruff, mypy, pytest on PostgreSQL 16, coverage gate 95"]
    C2b["uv build --wheel"]
    C3["security: reusable security.yml at 87dc5e1 v1.1.1"]
    C1 --> C1a --> C1b
    C2 --> C2a --> C2b
  end
  subgraph SR["shared-runtime.yml (group shared-runtime-ref)"]
    I1["image: docker build --pull, no push - 15 min"]
    I2["docker run: import service, id -u is not 0"]
    I1 --> I2
  end
  CHK -. "no deploy job" .-> DEP["Manual: railway up --ci from git archive main"]
  SR -.-> DEP
  DEP --> RW["Railway build from Dockerfile, healthcheck /readyz"]
```

Sources: `.github/workflows/check.yml`, `.github/workflows/shared-runtime.yml`, `scripts/check.sh`, `railway.toml`, `PRODUCT.md` (CI, "Deploys are Railway CLI uploads"), ops record: book-platforms-status (2026-11-03) (deploy recipe)

- Jobs ทั้ง 3 ใน `check.yml` ไม่มี `needs:` ต่อกัน จึงรันขนานกัน.
- Runner: GitHub-hosted (`ubuntu-latest`) เพราะ repo public (`PRODUCT.md`).
- `tests/test_build_hygiene.py` บังคับว่าทุก `uses:` ต้อง pin 40-hex SHA และ CI ต้อง build image.

## Open questions

- ขั้นตอนภายใน `ci-python.yml` / `security.yml` อยู่ใน repo `bookchaowalit/platform-ci-workflows` (ไม่ได้อ่าน); รายละเอียด security scan (เช่น gitleaks/pip-audit) ไม่ได้ยืนยัน
- มี branch protection / required checks บน `main` หรือไม่ — ไม่มีข้อมูลใน repo
