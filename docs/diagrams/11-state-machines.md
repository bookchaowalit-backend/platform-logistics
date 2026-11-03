# 11 — State machines

## shipment.status

ตรงกับ `rules.TRANSITIONS` ทุกเส้น. `delivered`, `returned`, `cancelled` เป็น terminal.

```mermaid
stateDiagram-v2
  [*] --> created
  created --> label_pending : begin_label
  created --> cancelled : POST cancel
  label_pending --> labelled : label purchased
  label_pending --> label_uncertain : uncertain or lease expired
  label_pending --> created : rejected or unavailable
  label_uncertain --> labelled : reconcile found
  label_uncertain --> created : reconcile none
  labelled --> in_transit : event
  labelled --> out_for_delivery : event
  labelled --> delivered : event
  labelled --> exception : event
  in_transit --> out_for_delivery : event
  in_transit --> delivered : event
  in_transit --> exception : event
  in_transit --> returned : event
  out_for_delivery --> in_transit : event
  out_for_delivery --> delivered : event
  out_for_delivery --> exception : event
  exception --> in_transit : event
  exception --> out_for_delivery : event
  exception --> delivered : event
  exception --> returned : event
  delivered --> [*]
  returned --> [*]
  cancelled --> [*]
```

Sources: `src/platform_logistics/rules.py` (`TRANSITIONS`), `src/platform_logistics/store.py`, `src/platform_logistics/migrations.py` (CHECK on status), `runtime_tests/test_rates_and_transitions.py` (`test_transitions_follow_the_fulfillment_graph`)

## shipment_label.status

```mermaid
stateDiagram-v2
  [*] --> purchasing : begin_label inserts row, attempts 1
  purchasing --> purchased : finish_label with valid Label
  purchasing --> failed : rejected or unavailable
  purchasing --> uncertain : uncertain, malformed label, error
  purchasing --> uncertain : lease_until passed (lazy _expire_leases)
  failed --> purchasing : new attempt, attempts + 1
  uncertain --> purchased : reconcile found
  uncertain --> failed : reconcile none
  purchased --> [*]
```

Sources: `src/platform_logistics/store.py` (`begin_label`, `finish_label`, `_expire_leases`, `reconcile_label`), `src/platform_logistics/migrations.py`, `runtime_tests/test_labels.py`

## tracking_event.outcome

แถวเกิดครั้งเดียวและไม่เปลี่ยน (`applied` หรือ `recorded`); `duplicate` เป็นผลตอบกลับ HTTP ไม่ได้เก็บ.

```mermaid
stateDiagram-v2
  [*] --> applied : newer event and allowed move
  [*] --> recorded : older than last_event_at or move not allowed
  applied --> [*]
  recorded --> [*]
```

Sources: `src/platform_logistics/store.py` (`record_tracking_event`), `src/platform_logistics/migrations.py` (CHECK on outcome)
