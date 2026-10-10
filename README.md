# Monolith to Microservices — Learning Lab

A hands-on journey from a single FastAPI monolith to a fully distributed,
event-driven system running on Kubernetes and AWS EKS.

Each phase builds on the previous one using the same business domain
(orders, inventory, payments) so the evolution is concrete and visible.

---

## Phases

| # | Phase | Status | Guide |
|---|-------|--------|-------|
| 1 | Modular Monolith | ✅ Done | [phase-01](docs/phase-01-modular-monolith.md) |
| 2 | Kubernetes Deployment | 🔜 Next | [phase-02](docs/phase-02-kubernetes.md) |
| 3 | Microservices + REST | 📋 Planned | [phase-03](docs/phase-03-microservices-rest.md) |
| 4 | Extract All Services | 📋 Planned | [phase-04](docs/phase-04-extract-services.md) |
| 5 | Kafka — Event-Driven | 📋 Planned | [phase-05](docs/phase-05-kafka-events.md) |
| 6 | Saga Pattern | 📋 Planned | [phase-06](docs/phase-06-saga.md) |
| 7 | Observability | 📋 Planned | [phase-07](docs/phase-07-observability.md) |
| 8 | GitOps + Argo CD | 📋 Planned | [phase-08](docs/phase-08-gitops-argocd.md) |
| 9 | AWS EKS | 📋 Planned | [phase-09](docs/phase-09-aws-eks.md) |

---

## Repository map

```
monolith-to-microservices/          ← this repo (lab root)
│
├── docs/                           ← step-by-step guide per phase
│   ├── phase-01-modular-monolith.md
│   ├── phase-02-kubernetes.md
│   └── ...
│
├── monolith-order-system/          ← Phase 1 — modular monolith
│
├── ms-users/                       ← Phase 4+ — users microservice
├── ms-products/                    ← Phase 4+ — products microservice
├── ms-inventory/                   ← Phase 4+ — inventory microservice
├── ms-orders/                      ← Phase 3+ — orders microservice
└── ms-payments/                    ← Phase 4+ — payments microservice
```

GitOps and infrastructure live in separate repositories:

| Repository | Description |
|---|---|
| `gitops-k8s` | Argo CD manifests for all services |
| `infrastructure-lab` | Terraform + cluster configs |

---

## Prerequisites

You need the following tools installed. Each phase guide specifies exactly
which ones are required before you start that phase.

| Tool | Used from |
|---|---|
| Docker + Docker Compose | Phase 1 |
| kubectl | Phase 2 |
| k3d or kubeadm | Phase 2 |
| Helm | Phase 2 |
| Python 3.12+ | Phase 1 (local dev only) |
| git | Phase 1 |
| GitHub CLI (`gh`) | Phase 8 |
| AWS CLI + eksctl | Phase 9 |
| Terraform | Phase 9 |

---

## How to use this lab

1. Read the phase guide before writing any code.
2. Follow each numbered step in order.
3. Complete the **"verify"** checkpoint at the end of each step before moving on.
4. Each phase ends with an **architecture exercise** — answer the questions
   yourself before reading the next phase.

> The goal is not to finish fast. It is to understand *why* each change is made.
