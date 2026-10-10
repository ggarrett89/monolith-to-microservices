# Phase 2 — Kubernetes Deployment

> **Status:** 🔜 Coming next
>
> Complete [Phase 1](phase-01-modular-monolith.md) before starting this phase.

---

## Goal

Deploy the **same monolith** — no code changes — onto a local Kubernetes
cluster. The application code doesn't know it's running on K8s. Only the
infrastructure layer changes.

---

## What you will build

```
             ┌─────────────────────────────┐
             │       Kubernetes Cluster     │
             │                             │
             │  ┌──────────┐               │
             │  │ Ingress  │ ← localhost   │
             │  └────┬─────┘               │
             │       │                     │
             │  ┌────▼──────────────┐      │
             │  │ orders-app        │      │
             │  │ Deployment (1 pod)│      │
             │  └────┬──────────────┘      │
             │       │                     │
             │  ┌────▼──────────────┐      │
             │  │ postgres          │      │
             │  │ StatefulSet       │      │
             │  │ PersistentVolume  │      │
             │  └───────────────────┘      │
             └─────────────────────────────┘
```

---

## Planned steps

1. **Install k3d** — lightweight K8s in Docker
2. **Create a local cluster** — `k3d cluster create`
3. **Build and push image** — to a local registry inside k3d
4. **Write Kubernetes manifests**
   - `Namespace`
   - `Secret` for database credentials
   - `ConfigMap` for non-secret config
   - PostgreSQL `StatefulSet` + `PersistentVolumeClaim` + `Service`
   - App `Deployment` + `Service`
   - `Ingress`
5. **Apply manifests** — `kubectl apply -f k8s/`
6. **Verify deployment** — `kubectl get pods`, `kubectl logs`
7. **Test the app** — same curl commands as Phase 1
8. **Scale the deployment** — `kubectl scale deployment orders-app --replicas=3`
9. **Observe the problem** — all 3 replicas share one DB, scaling helps HTTP but not the DB bottleneck
10. **Commit manifests** — tag `v0.2.0`

---

## Prerequisites for this phase

| Tool | Install guide |
|---|---|
| k3d | https://k3d.io |
| kubectl | https://kubernetes.io/docs/tasks/tools |
| Helm | https://helm.sh/docs/intro/install |

---

## Key concepts introduced

- Pod, Deployment, ReplicaSet
- Service (ClusterIP, NodePort)
- Ingress + IngressController
- StatefulSet vs Deployment
- PersistentVolume + PersistentVolumeClaim
- ConfigMap and Secret
- `kubectl` fundamentals

---

## Architecture exercise (preview)

Think about these before starting:

- If you scale the app to 3 replicas, what happens to the database?
- What happens when a pod is killed and rescheduled on a different node?
- What is the difference between a Deployment and a StatefulSet, and why does PostgreSQL need a StatefulSet?

---

## Next phase

[Phase 3 — Microservices + REST →](phase-03-microservices-rest.md)
