# 03 — User flows (API call flows per caller)

service นี้ไม่มี UI. "user flow" ในที่นี้คือลำดับ API call ของแต่ละ caller ตาม route ใน `app.py`.

## Flow A — Fulfillment caller (owner token หรือ consumer ในอนาคต)

```mermaid
flowchart TD
  S(["Get token from identity: aud logistics-api, ws, scopes"]) --> R["POST /v1/rates"]
  R --> C["POST /v1/shipments with Idempotency-Key"]
  C -- "201 new or 200 replay" --> L["POST /v1/shipments/{id}/label"]
  C -- "409 changed payload" --> X1["Fix body or use a new key"]
  C -- "422 Invalid request" --> X2["Fix input: address, parcel, carrier, service"]
  L -- "201 bought / 200 replay" --> G["GET /v1/shipments/{id}"]
  L -- "409 Label purchase in progress" --> W["Wait, retry later"]
  L -- "409 Label outcome uncertain" --> RC["POST /v1/shipments/{id}/label/reconcile"]
  L -- "422 Label rejected / 503 Carrier unavailable" --> RT["Retry label: new attempt"]
  RC -- "labelled" --> G
  RC -- "created (none found)" --> RT
  RC -- "409 still uncertain" --> RC
  RT --> L
  G --> EV["GET /v1/shipments/{id}/events"]
  C --> CN["POST /v1/shipments/{id}/cancel (only from created)"]
```

Sources: `src/platform_logistics/app.py`, `src/platform_logistics/store.py` (`purchase_label`, `reconcile_label`, `cancel`), `docs/runtime.md` (API, Label purchase)

## Flow B — Carrier-ingest bridge (`tracking:write`)

```mermaid
flowchart TD
  A(["Carrier event arrives at bridge"]) --> B["POST /v1/tracking-events: carrier, carrier_event_id, tracking_number, status, occurred_at"]
  B -- "404" --> N["Tracking number not a purchased label in this workspace"]
  B -- "201 applied" --> AP["Shipment status moved"]
  B -- "201 recorded" --> RE["Stored only: older or graph-invalid"]
  B -- "200 duplicate" --> DU["No change"]
  B -- "409" --> CF["Same event id, other content"]
  B -- "422" --> IV["Bad status, time zone missing, too far in future, before 2000"]
```

Sources: `src/platform_logistics/app.py` (`tracking`), `src/platform_logistics/store.py` (`validate_event`, `record_tracking_event`), `runtime_tests/test_tracking.py`

## Flow C — MCP hub (service client `mcp-hub`)

```mermaid
flowchart LR
  A["MCP client call_cell platform-logistics"] --> B["Hub gets client_credentials token per audience, caches it"]
  B --> C["HTTPS call to logistics-api route"]
  C --> D["Response returned to MCP client"]
```

Sources: ops record: book-platforms-status (2026-11-03) (MCP hub paragraph)

## Flow D — Operator lifecycle

```mermaid
flowchart LR
  M["python -m platform_logistics migrate (DDL role)"] --> S["python -m platform_logistics serve (runtime role)"]
  S --> H["GET /healthz"]
  S --> Y["GET /readyz: migrations, RLS, grants, carrier"]
```

Sources: `src/platform_logistics/__main__.py`, `docs/runtime.md` (Installation and lifecycle)

## Open questions

- Flow B: ยังไม่มี bridge จริง; ใครจะ build และอยู่ที่ไหน?
- Flow C: tool ใดของ hub ที่ map ไป route ใดของ logistics — อยู่ใน `platform-mcp-remote-worker` ซึ่งไม่ได้อ่านในงานนี้
