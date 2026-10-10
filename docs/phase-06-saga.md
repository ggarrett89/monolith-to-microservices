# Phase 6 — Saga Pattern

> **Status:** 📋 Planned
>
> Complete [Phase 5](phase-05-kafka-events.md) before starting this phase.

---

## Goal

Replace the fragile compensating transaction in `ms-orders` with a proper
**Saga** — a sequence of local transactions coordinated by events, each with
a defined compensating action that runs on failure.

---

## The problem we are solving

In Phase 1 we had:

```python
inventory_service.reserve_stock(...)
try:
    payment_service.process_payment(...)
except:
    inventory_service.release_stock(...)   # manual, fragile
```

If the process crashes between reserve and release, stock is lost forever.
In Phase 5 we distribute this across services, making the partial-failure
problem even harder. Saga solves it with durable, event-driven compensation.

---

## Choreography Saga (what we will build)

```
ms-orders                ms-inventory              ms-payments
    │                         │                         │
    │──── OrderCreated ───────▶│                         │
    │                         │── InventoryReserved ────▶│
    │                         │                         │── PaymentProcessed ──▶ ms-orders
    │◀── OrderConfirmed ───────────────────────────────────────────────────────────│
    │
    │   (on failure)
    │◀── PaymentFailed ────────────────────────────────────────────────────────────│
    │──── CancelInventory ────▶│
    │                         │── InventoryReleased ────▶ ms-orders
    │◀── OrderCancelled ──────────────────────────────────────────────────────────│
```

---

## Planned steps

1. **Review Phase 5 event flow** — identify where failures are not handled
2. **Define the full Saga state machine** for order creation
3. **Add compensating events**
   - `CancelInventoryReservation`
   - `RefundPayment`
4. **Implement compensation in `ms-inventory`** — listen for `CancelInventoryReservation`
5. **Implement compensation in `ms-payments`** — listen for `RefundPayment`
6. **Add `ms-orders` state machine** — track saga steps and trigger compensations
7. **Make all consumers idempotent** — duplicate events must not double-compensate
8. **Test failure scenarios**
   - Payment fails after stock reserved → stock is released
   - ms-orders crashes mid-saga → saga resumes on restart
9. **Add saga state persistence** — store saga progress in orders-db
10. **Commit** — tag all affected services

---

## Key concepts introduced

- Saga pattern (choreography vs orchestration)
- Compensating transactions
- Saga state machine
- Durable saga persistence
- Idempotency keys
- Distributed transaction without 2PC
- Eventual consistency guarantees

---

## Architecture exercise (preview)

Think about these before starting:

- In choreography, no single service knows the full saga state. How do you debug
  a failed saga across 3 services?
- What is the difference between choreography and orchestration Sagas? When would
  you choose orchestration instead?
- If a compensating transaction itself fails, what do you do?

---

## Next phase

[Phase 7 — Observability →](phase-07-observability.md)
