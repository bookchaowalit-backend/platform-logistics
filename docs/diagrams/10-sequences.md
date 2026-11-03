# 10 — Sequences

## 1. Create shipment (idempotent)

```mermaid
sequenceDiagram
  participant C as Caller
  participant Core as Core boundary + require_scope
  participant A as app.create
  participant S as LogisticsStore
  participant K as SandboxCarrier
  participant DB as PostgreSQL
  C->>Core: POST /v1/shipments, Bearer JWT, Idempotency-Key
  Core->>Core: verify RS256, aud, ws, scope shipments:write
  Core->>A: principal with workspace
  A->>S: create_shipment(ws, key, body)
  S->>S: valid_key, validate_shipment
  S->>K: quote(origin, destination, parcel)
  K-->>S: rates, pick service
  S->>DB: BEGIN, set app.workspace_id
  S->>DB: INSERT shipment ON CONFLICT (workspace_id, idempotency_key) DO NOTHING
  S->>DB: SELECT shipment + label, SELECT fingerprint
  alt fingerprint differs
    S-->>A: Conflict
    A-->>C: 409 Idempotency conflict
  else inserted
    A-->>C: 201 shipment
  else replay
    A-->>C: 200 first shipment
  end
```

## 2. Buy label (lease, carrier call, finish)

```mermaid
sequenceDiagram
  participant C as Caller
  participant S as LogisticsStore
  participant DB as PostgreSQL
  participant K as SandboxCarrier
  C->>S: POST /v1/shipments/{id}/label (labels:write)
  S->>DB: get_shipment
  S->>S: carrier(name) or 422
  S->>DB: tx1 expire stale purchasing leases to uncertain, COMMIT
  S->>DB: tx2 lock shipment FOR UPDATE, check label row
  alt label purchased
    S-->>C: 200 replay same tracking number
  else purchasing or uncertain
    S-->>C: 409 in progress or uncertain
  else allowed
    S->>DB: upsert shipment_label purchasing + lease 60 s, shipment label_pending, COMMIT
    S->>K: purchase_label(shipment, "label/id") outside tx
    alt bought
      K-->>S: Label
      S->>DB: finish_label purchased (same token, lease alive), shipment labelled
      S-->>C: 201 shipment with label
    else rejected or unavailable
      S->>DB: finish_label failed, shipment created
      S-->>C: 422 Label rejected or 503 Carrier unavailable
    else uncertain or error
      S->>DB: finish_label uncertain, shipment label_uncertain
      S-->>C: 409 Label outcome uncertain
    end
  end
```

## 3. Reconcile uncertain label

```mermaid
sequenceDiagram
  participant C as Caller
  participant S as LogisticsStore
  participant DB as PostgreSQL
  participant K as SandboxCarrier
  C->>S: POST /v1/shipments/{id}/label/reconcile
  S->>DB: lock shipment, expire leases, read label
  alt label not uncertain
    S-->>C: 200 unchanged
  else uncertain
    S->>K: find_label("label/id")
    alt lookup fails
      S-->>C: 409 Label outcome uncertain
    else found
      S->>DB: label purchased, shipment labelled
      S-->>C: 200
    else none
      S->>DB: label failed, shipment created
      S-->>C: 200 retry is safe
    end
  end
```

## 4. Tracking event (dedup)

```mermaid
sequenceDiagram
  participant B as Carrier-ingest bridge
  participant S as LogisticsStore
  participant DB as PostgreSQL
  B->>S: POST /v1/tracking-events (tracking:write)
  S->>S: validate_event, fingerprint
  S->>DB: find purchased label by carrier + tracking_number
  alt not found
    S-->>B: 404
  else found
    S->>DB: lock shipment FOR UPDATE
    S->>S: stale or graph-invalid gives recorded, else applied
    S->>DB: INSERT tracking_event ON CONFLICT DO NOTHING
    alt conflict, same fingerprint
      S-->>B: 200 duplicate
    else conflict, other fingerprint
      S-->>B: 409
    else applied
      S->>DB: UPDATE shipment status, last_event_at
      S-->>B: 201 applied
    else recorded
      S-->>B: 201 recorded
    end
  end
```

Sources: `src/platform_logistics/app.py`, `src/platform_logistics/store.py`, `src/platform_logistics/carriers.py`, `docs/runtime.md`, `runtime_tests/test_labels.py`, `runtime_tests/test_tracking.py`

หมายเหตุ: "set app.workspace_id" ทำใน `Database.transaction(workspace)` ของ shared core (อนุมานจาก policy ใน `migrations.py` และ docstring ของ `store.py`).
