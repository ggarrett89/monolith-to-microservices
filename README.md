# Monolith to Microservices — Learning Lab

A hands-on preparatory lab: a small FastAPI order system built as a modular
monolith, with plans for evolving it into microservices.

## Status: frozen since 2026-11-01

This lab is no longer evolved. The monolith → microservices → distributed
systems journey continues in a separate **CRM-ERP** project, based on four books:
*Monolith to Microservices* (Newman, 1st ed.), *Building Microservices*
(Newman, 2nd ed.), *Building Event-Driven Microservices* (Bellemare, 1st ed.)
and *Designing Distributed Systems* (Burns, 1st ed.).

Why, and what is kept: [ADR-0001](docs/adr/ADR-0001-freeze-preparatory-lab.md).
Everything below is preserved as a working reference.

---

## Phases

| # | Phase | Status | Guide |
|---|-------|--------|-------|
| 1 | Modular Monolith | ✅ Done, validated manually | [phase-01](docs/phase-01-modular-monolith.md) |
| 2 | Kubernetes Deployment | ⚠️ Written, never executed | [phase-02](docs/phase-02-kubernetes.md) |
| 3 | Microservices + REST | 📋 Historical plan | [phase-03](docs/phase-03-microservices-rest.md) |
| 4 | Extract All Services | 📋 Historical plan | [phase-04](docs/phase-04-extract-services.md) |
| 5 | Kafka — Event-Driven | 📋 Historical plan | [phase-05](docs/phase-05-kafka-events.md) |
| 6 | Saga Pattern | 📋 Historical plan | [phase-06](docs/phase-06-saga.md) |
| 7 | Observability | 📋 Historical plan | [phase-07](docs/phase-07-observability.md) |
| 8 | GitOps + Argo CD | 📋 Historical plan | [phase-08](docs/phase-08-gitops-argocd.md) |
| 9 | AWS EKS | 📋 Historical plan, optional track | [phase-09](docs/phase-09-aws-eks.md) |

## Decisions

| ADR | Title |
|---|---|
| [0001](docs/adr/ADR-0001-freeze-preparatory-lab.md) | Freeze this lab; continue the journey in the CRM-ERP project |
| [0002](docs/adr/ADR-0002-homelab-kubernetes-stack.md) | Phase 2 stack: kubeadm cluster, GHCR, Traefik |

---

## Repository map

```
monolith-to-microservices/          ← this repo (lab root)
│
├── docs/
│   ├── adr/                        ← decisions of this lab
│   ├── phase-01-modular-monolith.md
│   ├── phase-02-kubernetes.md
│   └── ...                         ← phases 3–9: historical plans
│
└── monolith-order-system/          ← git submodule: the application + k8s/ manifests
```

Clone with the application included:

```bash
git clone --recurse-submodules git@github.com:ggarrett89/monolith-to-microservices.git
```

The `ms-*`, `gitops-k8s` and `infrastructure-lab` repositories from early plans
were never created and will not be.

---

## Prerequisites

You need the following tools installed. Each phase guide specifies exactly
which ones are required before you start that phase.

| Tool | Used from |
|---|---|
| Docker + Docker Compose | Phase 1 |
| kubectl | Phase 2 |
| kubeadm cluster (`~/projetos/cka`) | Phase 2 |
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
