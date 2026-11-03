# ADR-0001 — Python 3.12 FastAPI service on platform-backend-core pinned by commit

- Status: accepted (retro-documented)
- Date of evidence: 2026-10-29 (runtime pilot), core pin `21d5f44`

## Context

ทุก platform service ของ Book Platform ใช้ runtime กลางร่วมกัน แต่ "each repo owns its code". Service นี้ต้องมี auth, workspace transaction, readiness และ error boundary แบบเดียวกับ service อื่น.

## Decision

- ใช้ `platform_backend_core.create_app('logistics', ...)` และ `require_scope` (FastAPI + uvicorn) บน Python ≥3.12.
- ติดตั้ง `platform-backend-core` จาก Git commit `21d5f44d26220080cbed43b9807979dc84a8533a` ที่ review แล้ว ไม่ใช่จาก PyPI (`[tool.uv.sources]`) เพื่อกัน dependency confusion; commit เดียวกันใน `Dockerfile`, `uv.lock` และ CI.
- `tests/test_build_hygiene.py` fail ถ้า commit ใน Dockerfile กับ workflow ไม่ตรงกัน.

## Evidence

- `pyproject.toml` (dependencies, `[tool.uv.sources]`)
- `Dockerfile` (`pip wheel 'platform-backend-core @ git+...@21d5f44...'`)
- `src/platform_logistics/app.py` (`base_app`, `require_scope`)
- `tests/test_build_hygiene.py` (`test_core_comes_from_the_ci_pinned_commit`)
- `docs/runtime.md` (Container image)

## Consequences

- อัปเกรด core ต้องแก้ 3 ที่พร้อมกัน (workflow, Dockerfile, `[tool.uv.sources]`).
- ops record: book-platforms-status (2026-11-03): "Other services still pin core 21d5f44" — logistics ยังไม่ได้ core ที่มี `ServiceTokenSource` (dc1fa7d) แต่ service นี้ไม่ได้เรียก service อื่นอยู่แล้ว.
