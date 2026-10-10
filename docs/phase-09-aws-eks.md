# Phase 9 — AWS EKS

> **Status:** 📋 Planned
>
> Complete [Phase 8](phase-08-gitops-argocd.md) before starting this phase.

---

## Goal

Move the entire system from a local k3d cluster to **AWS EKS**. The
application code and GitOps manifests stay the same — only the infrastructure
layer changes.

---

## What you will build

```
  ┌──────────────────────────────────────────────────────────┐
  │  AWS                                                     │
  │                                                          │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │  EKS Cluster                                       │  │
  │  │                                                    │  │
  │  │  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │  │
  │  │  │ms-orders │  │ms-inventory│  │  ms-payments   │  │  │
  │  │  └──────────┘  └───────────┘  └────────────────┘  │  │
  │  │                                                    │  │
  │  │  ┌──────────┐  ┌───────────────────────────────┐  │  │
  │  │  │  Kafka   │  │  Prometheus + Grafana + Jaeger │  │  │
  │  │  │  (MSK)   │  └───────────────────────────────┘  │  │
  │  │  └──────────┘                                     │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  ┌──────────┐  ┌──────────────┐  ┌───────────────────┐  │
  │  │  RDS     │  │     ECR      │  │   Route 53 / ALB  │  │
  │  │(postgres)│  │(image registry│  │   (ingress)       │  │
  │  └──────────┘  └──────────────┘  └───────────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

---

## Planned steps

1. **Set up AWS prerequisites**
   - IAM user with least-privilege policy
   - Configure AWS CLI
2. **Write Terraform for EKS**
   - VPC + subnets
   - EKS cluster + managed node groups
   - IAM roles for service accounts (IRSA)
3. **Provision RDS PostgreSQL** (one instance per service or Aurora Serverless)
4. **Set up ECR** (Elastic Container Registry) — push all service images
5. **Update GitHub Actions** — push to ECR instead of local registry
6. **Connect Argo CD** to the EKS cluster
7. **Create `environments/aws/`** in `gitops-k8s` — override values for AWS
8. **Deploy all services** — same GitOps flow as Phase 8
9. **Set up MSK** (Managed Kafka) or migrate to Amazon MSK
10. **Configure ALB Ingress Controller** — expose services via AWS Load Balancer
11. **Set up Route 53** — custom domain
12. **Enable cluster autoscaler** — nodes scale with load
13. **Load test** — verify the system handles real traffic
14. **Cost review** — understand what each component costs

---

## Terraform structure (planned)

```
infrastructure-lab/
├── modules/
│   ├── eks/
│   ├── rds/
│   ├── msk/
│   └── ecr/
└── environments/
    ├── homelab/
    └── aws/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## Key concepts introduced

- EKS — managed control plane, node groups, Fargate profiles
- IRSA (IAM Roles for Service Accounts)
- RDS — managed PostgreSQL, connection pooling with PgBouncer
- ECR — private container registry
- ALB Ingress Controller
- MSK — managed Kafka
- Cluster Autoscaler + Karpenter
- AWS cost optimization basics
- Terraform for EKS (state in S3 + DynamoDB lock)

---

## Architecture exercise (preview)

Think about these before starting:

- Your monolith from Phase 1 had one database. You now have 5 RDS instances.
  How does the operational complexity compare? Was it worth it?
- A node in EKS becomes unhealthy. What happens to the pods on it?
  How is this different from Phase 1?
- Which AWS services could replace Kafka, Prometheus, and Jaeger natively?
  When would you choose the managed versions over self-hosted?

---

## Congratulations

If you have reached this phase, you have built the same system at three levels:

| Phase | Architecture | Deployment |
|---|---|---|
| 1 | Monolith | Docker Compose |
| 2 | Monolith | Kubernetes (local) |
| 3–4 | Microservices + REST | Kubernetes (local) |
| 5–6 | Event-driven + Saga | Kubernetes (local) |
| 7 | + Observability | Kubernetes (local) |
| 8 | + GitOps + Rollouts | Kubernetes (local) |
| 9 | Full system | AWS EKS |

You now understand not just *how* to run microservices, but *why* each
architectural decision was made and what problems it solves.
