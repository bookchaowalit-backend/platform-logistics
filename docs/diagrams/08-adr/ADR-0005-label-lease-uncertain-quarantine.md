# ADR-0005 — Label purchase: persisted lease before the carrier call, uncertain quarantine, explicit reconcile

- Status: accepted (retro-documented)
- Date of evidence: 2026-10-29 (runtime pilot + review fixes)

## Context

การซื้อ label มีผลข้างเคียงภายนอก (เงิน) และ carrier อาจ timeout หลังส่งคำขอแล้ว. ต้องไม่ซื้อซ้ำ (migration gate "label replay").

## Decision

- `shipment_label` PK `(workspace_id, shipment_id)` — หนึ่ง label ต่อ shipment.
- `begin_label` commit แถว `purchasing` พร้อม lease 60 s ก่อนเรียก carrier; carrier key คงที่ `label/<shipment id>`.
- `finish_label` บันทึกผลได้เฉพาะ lease token เดิมที่ยังไม่หมดอายุ.
- ผล `uncertain` (timeout หลังส่ง, คำตอบผิดรูป, exception อื่น, lease หมด) → quarantine; ไม่เรียก carrier ซ้ำจนกว่าจะ `POST .../label/reconcile` ซึ่งถาม `find_label`.
- carrier ต้องถูก resolve ก่อนเอา lease (review fix): carrier ที่ไม่ได้ configure เป็น 422 ธรรมดา.

## Evidence

- `src/platform_logistics/store.py` (`begin_label`, `finish_label`, `purchase_label`, `reconcile_label`, `_expire_leases`)
- `src/platform_logistics/migrations.py` (`shipment_label`)
- `runtime_tests/test_labels.py`, `runtime_tests/test_review_fixes.py`, `runtime_tests/test_http.py`
- `docs/runtime.md` (Label purchase, Review fixes)

## Consequences

- ไม่มี background worker: lease ที่หมดถูก settle แบบ lazy; reconcile ต้องสั่งเอง.
- adapter จริงต้อง idempotent บน key, timeout ภายใน 60 s และ `find_label` ต้อง authoritative (`docs/runtime.md`).
