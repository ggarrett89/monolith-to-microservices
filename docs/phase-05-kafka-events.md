# Phase 5 — Kafka: Event-Driven Communication

> **Status:** 📋 Planned
>
> Complete [Phase 4](phase-04-extract-services.md) before starting this phase.

---

## Goal

Replace synchronous HTTP calls between services with **asynchronous events**
published to Kafka. Services become loosely coupled — a producer does not
know or care who consumes its events.

---

## What you will build

```
  ┌───────────┐
  │ ms-orders │
  └─────┬─────┘
        │  OrderCreated event
        ▼
  ┌─────────────┐
  │    Kafka    │
  │  (broker)   │
  └──┬────┬─────┘
     │    │
     │    └──────────────────┐
     ▼                       ▼
┌────────────┐         ┌─────────────┐
│ms-inventory│         │ ms-payments │
│ consumes   │         │  consumes   │
│ OrderCreated│        │ OrderCreated│
└────────────┘         └─────────────┘
     │                       │
     │ InventoryReserved      │ PaymentProcessed
     │ or InventoryFailed     │ or PaymentFailed
     ▼                       ▼
  ┌─────────────────────────────┐
  │           Kafka             │
  └─────────────────────────────┘
```

---

## Planned steps

1. **Add Kafka to the cluster** — deploy via Helm (Bitnami Kafka chart)
2. **Design the event schema** — define event types and payloads
   - `OrderCreated`
   - `InventoryReserved` / `InventoryFailed`
   - `PaymentProcessed` / `PaymentFailed`
   - `OrderConfirmed` / `OrderCancelled`
3. **Add Kafka producer to `ms-orders`** — publish `OrderCreated` on POST /orders
4. **Add Kafka consumer to `ms-inventory`** — listen for `OrderCreated`, reserve stock
5. **Add Kafka consumer to `ms-payments`** — listen for `InventoryReserved`, process payment
6. **Add Kafka consumer back to `ms-orders`** — update order status on final events
7. **Handle delivery semantics** — at-least-once, idempotent consumers
8. **Test the async flow** — observe eventual consistency
9. **Observe the new problems** — ordering, duplicate messages, consumer lag
10. **Commit** — tag all affected services

---

## Key concepts introduced

- Event-driven architecture
- Topics, partitions, consumer groups
- Producer / consumer patterns with `confluent-kafka-python`
- At-least-once vs exactly-once delivery
- Idempotent consumers (deduplication)
- Message ordering guarantees
- Consumer lag monitoring
- Eventual consistency vs strong consistency

---

## Event schema (planned)

```json
// OrderCreated
{
  "event_type": "OrderCreated",
  "order_id": 1,
  "user_id": 1,
  "product_id": 1,
  "quantity": 2,
  "total": "199.80",
  "timestamp": "2026-01-01T00:00:00Z"
}
```

---

## Architecture exercise (preview)

Think about these before starting:

- `ms-orders` publishes `OrderCreated` and returns `202 Accepted` immediately.
  How does the client know when the order is actually confirmed?
- If `ms-inventory` crashes and restarts, how does it know which events it already processed?
- If the same `OrderCreated` event is delivered twice, what prevents inventory
  from being decremented twice?

---

## Next phase

[Phase 6 — Saga Pattern →](phase-06-saga.md)
