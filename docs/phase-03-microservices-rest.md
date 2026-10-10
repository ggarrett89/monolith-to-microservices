# Phase 3 — Microservices + REST Communication

> **Status:** 📋 Planned
>
> Complete [Phase 2](phase-02-kubernetes.md) before starting this phase.

---

## Goal

Extract **ms-orders** as the first independent microservice.
Orders will communicate with inventory and payments via **HTTP REST**
instead of direct function calls.

---

## What you will build

```
                    ┌─────────────┐
                    │ API Gateway │
                    └──────┬──────┘
                           │
          ┌────────────────┼──────────────┐
          │                │              │
    ┌─────▼─────┐   ┌──────▼────┐  ┌─────▼──────┐
    │ ms-orders │   │ ms-users  │  │ ms-products│
    └─────┬─────┘   └───────────┘  └────────────┘
          │
    ┌─────▼──────────────────┐
    │  HTTP calls (REST)     │
    └──────┬──────────────┬──┘
           │              │
    ┌──────▼────┐   ┌──────▼─────┐
    │ms-inventory│  │ms-payments │
    └────────────┘  └────────────┘
```

---

## Planned steps

1. **Create `ms-orders` repository** — new Python + FastAPI project
2. **Copy orders domain code** — from the monolith
3. **Replace direct service imports** with HTTP clients using `httpx`
4. **Add environment variables** for service URLs (`INVENTORY_SERVICE_URL`, `PAYMENTS_SERVICE_URL`)
5. **Create `ms-orders` Dockerfile**
6. **Create Docker Compose** with all services
7. **Write Kubernetes manifests** for ms-orders
8. **Deploy and test** — verify orders still work end-to-end
9. **Observe the new problems introduced**
   - Network latency between services
   - What happens when ms-inventory is down?
   - Retry logic and timeouts
10. **Add basic resilience** — timeout + retry with `tenacity`
11. **Commit** — tag `ms-orders v1.0.0`

---

## Prerequisites for this phase

- Phase 2 completed and running on Kubernetes
- Understanding of HTTP clients in Python (`httpx`)

---

## Key concepts introduced

- Service decomposition — when and how to split
- Synchronous REST communication between services
- Service discovery in Kubernetes (DNS-based)
- Timeout and retry patterns
- Cascading failures — what happens when a downstream service is slow
- Independent deployment of a single service

---

## Architecture exercise (preview)

Think about these before starting:

- If `ms-payments` takes 30 seconds to respond, what happens to `ms-orders`?
- If `ms-inventory` is completely down, should `ms-orders` fail immediately or retry?
- You now have two databases (orders-db and inventory-db). What happens if an order
  is confirmed but the inventory update fails halfway through?

---

## Next phase

[Phase 4 — Extract All Services →](phase-04-extract-services.md)
