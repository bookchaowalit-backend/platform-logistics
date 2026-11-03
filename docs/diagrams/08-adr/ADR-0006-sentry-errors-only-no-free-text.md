# ADR-0006 — Sentry errors only, no free text, innermost CaptureErrors

- Status: accepted (retro-documented)
- Date of evidence: 2026-11-03 (PR #12, main `501601f`)

## Context

PR #2–#11 พยายาม scrub free text ด้วย pattern แต่ review พบทางเลี่ยงเรื่อย ๆ (ดู commit log 2026-11-02/03). Shared core แปลง exception เป็น 500 ที่ redacted ก่อนถึง integration ชั้นนอกของ Sentry.

## Decision

- `init_sentry()` เป็น no-op ถ้าไม่มี `SENTRY_DSN`; `traces_sample_rate=None`, `trace_propagation_targets=[]`, `send_default_pii=False`, `include_local_variables=False`, `max_request_body_size="never"`, ปิด logging integration.
- `_scrub_event` ทิ้ง exception value, message, logentry, breadcrumbs, extra, user, dynamic sampling context; request เหลือแค่ method.
- `CaptureErrors` เป็น user middleware ชั้นในสุด รายงาน `handled=False` แล้ว re-raise; ไม่สร้าง event ถ้า client ไม่ active.

## Evidence

- `src/platform_logistics/_sentry.py`, `src/platform_logistics/__init__.py`, `src/platform_logistics/app.py`
- `runtime_tests/test_sentry.py` (`test_no_exception_text_leaves_the_process`, `test_a_server_500_reaches_sentry_scrubbed`)
- `git log` 1f05d6c…3942a64
- ops record: book-platforms-status (2026-11-03) (logistics 501601f, Sentry project `platform-<svc>`)

## Consequences

- debug ต้องอาศัย type + stack + route; ไม่มีข้อความ exception ใน Sentry (owner revert ได้ตาม ops record).
