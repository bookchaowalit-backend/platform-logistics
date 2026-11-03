# ADR-0003 — Own Neon PostgreSQL project with forced RLS and split DDL/runtime roles

- Status: accepted (retro-documented)
- Date of evidence: 2026-10-29

## Context

Service เป็น multi-tenant ต่อ workspace และ Neon free quota เป็นราย project (`empire-platform` เคยเต็ม).

## Decision

- ฐานข้อมูลเฉพาะของ service (Neon project `platform-<svc>` ตาม ops record), PostgreSQL ≥16.
- Migration `0001` สร้าง 3 ตาราง และ `ENABLE` + `FORCE ROW LEVEL SECURITY` พร้อม policy `workspace` ที่ใช้ `current_setting('app.workspace_id')` ทั้ง `USING` และ `WITH CHECK`.
- `migrate` ใช้ DDL role; `serve` ใช้ runtime role NOSUPERUSER NOBYPASSRLS ที่มีเฉพาะ grant: SELECT/INSERT/UPDATE บน `shipment`, `shipment_label`; SELECT/INSERT บน `tracking_event`; SELECT บน `platform_schema_migrations`; USAGE บน schema. ไม่มี DELETE/TRUNCATE.
- API ไม่ migrate ตอน startup; `/readyz` ตรวจ checksum migration, forced RLS และแต่ละ privilege แยกกัน.

## Evidence

- `src/platform_logistics/migrations.py`
- `src/platform_logistics/store.py` (`ready`)
- `src/platform_logistics/__main__.py`
- `runtime_tests/test_http.py` (`test_readiness_requires_each_effective_privilege`), `runtime_tests/test_shipments.py` (`test_workspaces_are_isolated_and_rls_rejects_foreign_rows`)
- `docs/runtime.md` (Runtime grants)
- ops record: book-platforms-status (2026-11-03) ("Each service has its own free Neon project `platform-<svc>`", "Database roles")

## Consequences

- ไม่มี DELETE จึงไม่มี purge/retention job (ระบุใน `docs/runtime.md` Deliberate omissions).
- Restore ต้องทำลง database ใหม่แยก (`docs/runtime.md`: "Restore only into a new isolated database").
