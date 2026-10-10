# Phase 4 — Extract All Services

> **Status:** 📋 Planned
>
> Complete [Phase 3](phase-03-microservices-rest.md) before starting this phase.

---

## Goal

Extract every remaining domain from the monolith into its own service.
By the end of this phase the monolith is retired and replaced by five
independent microservices, each with its own database.

---

## What you will build

```
                         ┌─────────────┐
                         │ API Gateway │
                         └──────┬──────┘
                                │
       ┌──────────┬─────────────┼──────────────┬──────────┐
       │          │             │              │          │
 ┌─────▼──┐ ┌─────▼──┐  ┌──────▼────┐  ┌──────▼──┐ ┌─────▼──────┐
 │ms-users│ │ms-prod.│  │ ms-orders │  │ms-inv.  │ │ms-payments │
 └────┬───┘ └────┬───┘  └──────┬────┘  └──────┬──┘ └─────┬──────┘
      │          │             │              │           │
     DB         DB            DB             DB          DB
```

---

## Planned steps

1. **Extract `ms-users`** — users domain → own FastAPI app + own DB
2. **Extract `ms-products`** — products domain → own FastAPI app + own DB
3. **Extract `ms-inventory`** — inventory domain → own FastAPI app + own DB
4. **Extract `ms-payments`** — payments domain → own FastAPI app + own DB
5. **Update `ms-orders`** to call all four services via HTTP
6. **Add an API Gateway** — nginx or Traefik as single entry point
7. **Write K8s manifests** for all services
8. **Deploy all services** to the cluster
9. **Retire the monolith** — remove `monolith-order-system` from the cluster
10. **End-to-end test** — the same curl walkthrough from Phase 1 must still work
11. **Tag all repos** — `v1.0.0`

---

## Repository structure after this phase

```
github.com/<you>/
├── monolith-order-system   (archived — kept for reference)
├── ms-users                v1.0.0
├── ms-products             v1.0.0
├── ms-inventory            v1.0.0
├── ms-orders               v1.0.0
└── ms-payments             v1.0.0
```

---

## Key concepts introduced

- Database per service pattern
- Data ownership — each service owns its own tables
- Cross-service data consistency without shared DB
- API contracts between services
- Independent versioning and deployment
- Service-to-service authentication (basic API key)

---

## Architecture exercise (preview)

Think about these before starting:

- `ms-orders` now makes 4 HTTP calls per order. What is the total failure
  probability if each service has 99% uptime? (`0.99^4 = 96%`)
- How do you look up a product's name in `ms-orders` when products live in `ms-products`?
  Do you call the service every time, or cache the result?
- What happens if `ms-inventory` goes down during a peak sale?

---

## Next phase

[Phase 5 — Kafka: Event-Driven Communication →](phase-05-kafka-events.md)
