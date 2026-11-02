# ADR-0001 — Freeze this lab; continue the journey in the CRM-ERP project

- **Status:** accepted
- **Date:** 2026-11-01
- **Scope:** monolith-to-microservices, monolith-order-system

## Context

This lab started the monolith → microservices journey with a small FastAPI
order system (users, products, inventory, orders, payments). By 2026-11-01:

- Phase 1 (modular monolith) was done and validated manually with the curl
  walkthrough, including the payment-failure compensation.
- Phase 2 (Kubernetes) had manifests and a guide, but was never executed.
- Phases 3–9 existed only as plans.

The ecosystem goals were consolidated on 2026-11-01. The journey must run on a
real business application (a CRM-ERP), be grounded in four books, and help me
learn TypeScript. Cloud (phase 9) became optional. These are ecosystem-level
decisions recorded in `learning-platform` (ADR-0001 to ADR-0004 there).

## Decision

1. Freeze this lab. No new phases are implemented here.
2. Keep all code, manifests and guides as a working reference. Nothing is deleted.
3. Mark phase 2 as "not validated" and phases 3–9 as "historical plan".
   Phase 9 is also marked as an optional track.
4. Drop the planned `ms-*`, `gitops-k8s` and `infrastructure-lab` repositories.
   None was ever created.

## Alternatives considered

| Alternative | Why not |
|---|---|
| Continue the journey here | Small didactic app in Python; does not meet the real-application and TypeScript goals |
| Run both journeys in parallel | Duplicates effort and documentation |
| Delete or archive the repositories | Loses a working, validated reference for no gain |

## Consequences

- Readers see the real state of each phase instead of an implied roadmap.
- Ideas from phases 3–8 (REST extraction, Kafka, Saga, observability, GitOps)
  are reconsidered in the CRM-ERP journey. They are not copied over automatically.
- The Phase 2 manifests can still be used to practise Kubernetes on the `cka` cluster.
