# 09 — ERD

จาก migration `0001` ใน `src/platform_logistics/migrations.py` (namespace `logistics`). ตาราง `platform_schema_migrations` เป็นของ shared core `MigrationRunner` (คอลัมน์ที่เห็นในโค้ดนี้: `namespace`, `version`, `checksum`).

```mermaid
erDiagram
  shipment ||--o| shipment_label : "has at most one"
  shipment ||--o{ tracking_event : "receives"
  shipment {
    uuid workspace_id PK
    uuid id PK
    text idempotency_key "UNIQUE with workspace_id"
    text fingerprint "SHA-256 of canonical body"
    text reference "1..64, caller order id"
    text recipient_ref "1..128, opaque pointer"
    text carrier
    text service
    text origin_country
    text origin_postal_code
    text destination_country
    text destination_postal_code
    int weight_grams "1..30000"
    int length_mm "1..1500"
    int width_mm "1..1500"
    int height_mm "1..1500"
    int billable_weight_grams "gt 0"
    bigint rate_amount_minor "ge 0"
    text currency
    text status "10 values, default created"
    timestamptz last_event_at
    timestamptz created_at
    timestamptz updated_at
  }
  shipment_label {
    uuid workspace_id PK
    uuid shipment_id PK
    text carrier
    text status "purchasing, purchased, failed, uncertain"
    int attempts "ge 1"
    uuid lease_token
    timestamptz lease_until
    text tracking_number "UNIQUE with workspace_id, carrier"
    text label_ref
    timestamptz created_at
    timestamptz updated_at
  }
  tracking_event {
    uuid workspace_id PK
    text carrier PK
    text carrier_event_id PK
    uuid shipment_id FK
    text status
    timestamptz occurred_at
    text fingerprint
    text outcome "applied or recorded"
    timestamptz received_at
  }
  platform_schema_migrations {
    text namespace
    text version
    text checksum
  }
```

Sources: `src/platform_logistics/migrations.py`, `src/platform_logistics/store.py` (`ready`, `fingerprint`)

## Constraints และ RLS

- FK แบบ composite: `shipment_label(workspace_id, shipment_id)` และ `tracking_event(workspace_id, shipment_id)` → `shipment(workspace_id, id)`.
- CHECK: `(status='purchased') = (tracking_number IS NOT NULL AND label_ref IS NOT NULL)`.
- Index: `tracking_event(workspace_id, shipment_id, occurred_at)`.
- ทั้ง 3 ตาราง: `ENABLE` + `FORCE ROW LEVEL SECURITY`, policy `workspace` `USING`/`WITH CHECK` `workspace_id = nullif(current_setting('app.workspace_id', true), '')::uuid`.
- Runtime role: NOSUPERUSER NOBYPASSRLS; grants ตาม ADR-0003; ไม่มี DELETE/TRUNCATE.
- ข้อมูลส่วนบุคคล: ไม่มีชื่อ ที่อยู่ถนน เบอร์โทร อีเมล; `recipient_ref` regex `[A-Za-z0-9_.:-]{1,128}` ปฏิเสธช่องว่างจึงกันชื่อ/ที่อยู่ (`store.RECIPIENT_REF`).

Sources: `src/platform_logistics/migrations.py`, `src/platform_logistics/store.py`, `docs/runtime.md` (Runtime grants, Data held)

Central: [ERD-DIAGRAMS.md](https://github.com/bookchaowalit/architecture-docs/blob/main/docs/architecture/ERD-DIAGRAMS.md) ไม่มี logistics.

## Open questions

- ชนิดคอลัมน์ของ `platform_schema_migrations` อยู่ใน shared core; ด้านบนแสดงเป็น `text` ตามที่ใช้ใน query เท่านั้น
