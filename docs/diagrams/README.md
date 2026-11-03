# Core 22 SDLC diagram pack — platform-logistics

ชุดภาพ 22 รายการตาม SDLC Diagram Catalog v2 วาดจากสิ่งที่โค้ดและ config ทำจริง ณ `origin/main` `96f23af` (2026-11-03). เอกสารเดิมที่ยังใช้: [runtime.md](../runtime.md) (API และ operating guide), [CONTRACT-SURFACE.md](../CONTRACT-SURFACE.md), [UPGRADE-PLAN.md](../UPGRADE-PLAN.md), [PRODUCT.md](../../PRODUCT.md).

ข้อควรรู้: `docs/runtime.md` ยังเขียนว่า "Nothing has been deployed" แต่ `PRODUCT.md` และ ops record (book-platforms-status, 2026-11-03) ระบุว่า service live บน Railway แล้ว. ภาพชุดนี้ยึดสถานะ live.

| # | Item | Status | File | Sources read |
|---|---|---|---|---|
| 01 | Business process (As-Is / To-Be) | done | [01-business-process.md](01-business-process.md) | `app.py`, `store.py`, `rules.py`, `carriers.py`, `PRODUCT.md`, `docs/UPGRADE-PLAN.md`, `docs/runtime.md` |
| 02 | Use cases | done | [02-use-cases.md](02-use-cases.md) | `app.py`, `runtime_tests/conftest.py`, `runtime_tests/test_http.py`, ops record |
| 03 | User flows (API call flows) | done | [03-user-flows.md](03-user-flows.md) | `app.py`, `store.py`, `__main__.py`, ops record |
| 04 | Traceability (RTM) | done | [04-traceability.md](04-traceability.md) | `PRODUCT.md`, `contract.json`, `docs/runtime.md`, workflows, all tests |
| 05 | C4 Context | done | [05-c4-context.md](05-c4-context.md) | `__main__.py`, `_sentry.py`, `PRODUCT.md`, ops record |
| 06 | C4 Container | done | [06-c4-container.md](06-c4-container.md) | `Dockerfile`, `railway.toml`, `migrations.py`, `pyproject.toml` |
| 07 | C4 Component | done | [07-c4-component.md](07-c4-component.md) | all `src/platform_logistics/*.py` |
| 08 | ADRs | done (6, retro-documented) | [08-adr/](08-adr/) | `pyproject.toml`, `Dockerfile`, `railway.toml`, `migrations.py`, `store.py`, `_sentry.py`, git log, ops record |
| 09 | ERD | done | [09-erd.md](09-erd.md) | `migrations.py`, `store.py` |
| 10 | Sequences | done | [10-sequences.md](10-sequences.md) | `app.py`, `store.py`, `carriers.py` |
| 11 | State machines | done | [11-state-machines.md](11-state-machines.md) | `rules.py`, `store.py`, `migrations.py` |
| 12 | API and integrations | done (reuses `docs/runtime.md` API table) | [12-api-integration.md](12-api-integration.md) | `app.py`, `store.py`, `docs/runtime.md`, core `http.py` @ 21d5f44 |
| 13 | Threat model | done | [13-threat-model.md](13-threat-model.md) | `app.py`, `_sentry.py`, `migrations.py`, `Dockerfile`, core `http.py`/`auth.py` @ 21d5f44 |
| 14 | Backup and DR | done (backup unverified) | [14-backup-dr.md](14-backup-dr.md) | `migrations.py`, `carriers.py`, `docs/runtime.md`, central ADR-0002 |
| 15 | CI/CD pipeline | done | [15-cicd-pipeline.md](15-cicd-pipeline.md) | `.github/workflows/check.yml`, `.github/workflows/shared-runtime.yml`, `scripts/check.sh` |
| 16 | Deployment | done | [16-deployment.md](16-deployment.md) | `railway.toml`, `Dockerfile`, `__main__.py`, `PRODUCT.md`, ops record, uptime `targets.ts` |
| 17 | Test coverage matrix | done | [17-test-coverage-matrix.md](17-test-coverage-matrix.md) | `tests/*.py`, `runtime_tests/*.py`, `check.yml` |
| 18 | Git / release workflow | done | [18-git-release-workflow.md](18-git-release-workflow.md) | `git log`, `gh pr list`, workflows, ops record |
| 19 | Service dependency map | done | [19-service-dependency-map.md](19-service-dependency-map.md) | `pyproject.toml`, `contract.json`, `__main__.py`, ops record |
| 20 | Observability | done | [20-observability.md](20-observability.md) | `_sentry.py`, core `http.py`, `PRODUCT.md`, ops record, uptime `targets.ts`, central ADR-0002 |
| 21 | Incident response | done | [21-incident-response.md](21-incident-response.md) | `docs/runtime.md`, `store.py`, ops record, central ADR-0002 |
| 22 | DORA | done (deploy-based values not computable) | [22-dora.md](22-dora.md) | `git log`, `gh run list`, `gh pr list`, ops record |

"ops record" = `ops record: book-platforms-status (2026-11-03)`. Central architecture: https://github.com/bookchaowalit/architecture-docs/blob/main/docs/architecture/ (C4, ERD, attack-path docs ไม่มี logistics; ADR-0002 ใช้สำหรับ backup/monitoring).

## Open questions รวม (ต้องการ owner)

1. consumer แรกและ carrier จริงตัวไหน (01, 19)
2. scope ของ logistics ที่ hub `mcp-hub` และ owner grant ได้ (02)
3. logistics-api อยู่หลัง edge gateway หรือเปิดตรง `*.up.railway.app` (05, 13)
4. Neon `platform-logistics` อยู่ใน nightly backup + restore drill หรือไม่; RTO/RPO (14)
5. `/metrics`, `/docs`, `/openapi.json` เปิดสาธารณะ — ตั้งใจหรือไม่ (12, 13)
6. Sentry alert rule และช่องทาง (20, 21)
7. commit ที่ deploy อยู่ตอนนี้ และจะบันทึก deploy (tag / Railway API) เพื่อวัด DORA หรือไม่ (16, 22)
8. ปิด PR #1 ที่ถูกแทนที่ได้หรือไม่ (18)
9. อัปเดต `docs/runtime.md` ส่วน "Nothing has been deployed" / Staging plan ให้ตรงสถานะ live
