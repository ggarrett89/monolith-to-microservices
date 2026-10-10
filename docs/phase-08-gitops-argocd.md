# Phase 8 — GitOps + Argo CD

> **Status:** 📋 Planned
>
> Complete [Phase 7](phase-07-observability.md) before starting this phase.

---

## Goal

Replace manual `kubectl apply` with a **GitOps workflow**: the cluster state
is always derived from Git. Argo CD watches the `gitops-k8s` repository and
automatically reconciles any drift.

Then use **Argo Rollouts** to deploy services with zero downtime using
canary and blue/green strategies.

---

## What you will build

```
  Developer                  Git                    Cluster
  ──────────────────         ─────────────────      ───────────────────────
  push image tag     ──────▶ gitops-k8s repo  ──── Argo CD watches repo
  open PR            ──────▶ (values updated)  │    reconciles manifests
  merge PR                                     └──▶ deploys new version

                                                    Argo Rollouts
                                                    ┌─────────────────┐
                                                    │ canary: 10%     │
                                                    │ analysis: ok?   │
                                                    │ canary: 50%     │
                                                    │ analysis: ok?   │
                                                    │ full rollout    │
                                                    └─────────────────┘
```

---

## Planned steps

1. **Create `gitops-k8s` repository** — separate from application repos
2. **Organize manifests** by app and environment
   ```
   gitops-k8s/
   ├── apps/
   │   ├── ms-orders/
   │   ├── ms-inventory/
   │   └── ms-payments/
   ├── infrastructure/
   │   ├── argocd/
   │   ├── kafka/
   │   └── prometheus/
   └── environments/
       ├── homelab/
       └── aws/
   ```
3. **Install Argo CD** via Helm
4. **Connect Argo CD to `gitops-k8s`** — create Application resources
5. **Test reconciliation** — manually delete a pod, watch Argo CD restore it
6. **Add GitHub Actions CI pipeline**
   - On push to `ms-orders`: build image, push to registry, update image tag in `gitops-k8s`
7. **Install Argo Rollouts**
8. **Convert `ms-orders` Deployment to Rollout**
9. **Deploy a canary release** — 10% traffic to new version
10. **Add analysis** — automatic promotion/rollback based on error rate
11. **Test rollback** — deploy a broken version, watch it auto-rollback

---

## Repository structure

| Repository | Role |
|---|---|
| `ms-orders` | Application code + Dockerfile |
| `ms-inventory` | Application code + Dockerfile |
| `ms-payments` | Application code + Dockerfile |
| `gitops-k8s` | K8s manifests — source of truth for cluster state |

The application repos **never** contain Kubernetes manifests. The gitops
repo **never** contains application code.

---

## Key concepts introduced

- GitOps principles (Git as single source of truth)
- Argo CD — Application, AppProject, sync policies
- Drift detection and reconciliation
- Argo Rollouts — canary, blue/green, analysis templates
- Progressive delivery
- GitHub Actions CI → image build → tag update → Argo CD sync
- Separate application and infrastructure repositories

---

## Architecture exercise (preview)

Think about these before starting:

- A developer pushes code that breaks `ms-payments`. With GitOps, how do you
  roll back — do you revert the code or revert the manifest?
- What is the difference between Argo CD sync and Argo Rollouts promotion?
- In a canary deployment at 10%, a critical bug is found. How does the
  rollback work and who is affected?

---

## Next phase

[Phase 9 — AWS EKS →](phase-09-aws-eks.md)
