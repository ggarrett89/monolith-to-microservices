# Phase 7 — Observability

> **Status:** 📋 Planned
>
> Complete [Phase 6](phase-06-saga.md) before starting this phase.

---

## Goal

Add the three pillars of observability — **metrics, logs, and traces** —
so you can understand what is happening inside the distributed system
without touching the code or connecting to a server.

---

## What you will build

```
  Services                  Collectors              Dashboards
  ─────────────────────     ───────────────────     ──────────────
  ms-orders     ──────────▶ Prometheus         ──▶  Grafana
  ms-inventory  ──────────▶ (scrapes /metrics)
  ms-payments   ──────────▶

  ms-orders     ──────────▶ OpenTelemetry      ──▶  Jaeger
  ms-inventory  ──────────▶ Collector               (traces)
  ms-payments   ──────────▶

  All services  ──────────▶ stdout/JSON        ──▶  Loki + Grafana
                             (structured logs)
```

---

## Planned steps

1. **Add Prometheus metrics** to all services
   - `prometheus-fastapi-instrumentator`
   - Custom business metrics: orders per minute, payment failure rate
2. **Deploy Prometheus** to the cluster via Helm
3. **Deploy Grafana** and import dashboards
4. **Create dashboards**
   - Request rate, error rate, latency (RED method)
   - Kafka consumer lag
   - Saga completion rate
5. **Add structured logging** — JSON logs with correlation IDs
6. **Deploy Loki** — log aggregation
7. **Add distributed tracing** — instrument with OpenTelemetry
8. **Deploy Jaeger** — trace visualization
9. **Trace a full order saga** end-to-end across 3 services
10. **Set up alerts** — Prometheus alerting rules for error rate and latency

---

## Key concepts introduced

- RED method (Rate, Errors, Duration)
- USE method (Utilization, Saturation, Errors) for infrastructure
- Prometheus data model: counters, gauges, histograms, summaries
- PromQL basics
- Distributed tracing — spans, traces, context propagation
- Correlation IDs across services
- Structured logging (JSON)
- Log aggregation with Loki

---

## Architecture exercise (preview)

Think about these before starting:

- An order is failing in production. You have logs, metrics, and traces.
  In what order do you look at them and why?
- A Kafka consumer is falling behind. What metric tells you this, and what
  are the possible causes?
- You have a 5-second latency spike on `POST /orders`. How do you use traces
  to find which service caused it?

---

## Next phase

[Phase 8 — GitOps + Argo CD →](phase-08-gitops-argocd.md)
