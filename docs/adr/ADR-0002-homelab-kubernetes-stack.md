# ADR-0002 — Phase 2 stack: kubeadm cluster, GHCR, Traefik

- **Status:** accepted (recorded after the fact on 2026-11-01; never executed)
- **Date:** 2026-10 (decision), 2026-11-01 (record)
- **Scope:** monolith-to-microservices, phase 2

## Context

Phase 2 deploys the unchanged monolith to Kubernetes. The first draft used k3d.
I already had a kubeadm HA cluster for CKA study in `~/projetos/cka`
(3 control-plane + 2 workers, Kubernetes 1.29, containerd, Flannel), so the
plan moved to that cluster. That cluster has no registry, no ingress
controller and no StorageClass.

## Decision

| Need | Choice | Reason |
|---|---|---|
| Cluster | kubeadm HA cluster from `~/projetos/cka` | Already exists; closer to a real cluster than k3d; reuses CKA practice |
| Image registry | GitHub Container Registry (GHCR), `ghcr.io/ggarrett89/...` | No extra service to run; free for public images; `gh` is already authenticated |
| Ingress | Traefik via Helm, `NodePort` 30080 | My choice; `IngressRoute` CRD is simple to read |
| Storage | Rancher local-path-provisioner (added during review) | Free, local; kubeadm ships without a StorageClass |

## Alternatives considered

| Alternative | Why not |
|---|---|
| k3d | Simpler, but duplicates a cluster I already have |
| In-cluster registry | Extra component to run and secure in a 2 GB-per-node lab |
| NGINX Ingress | Valid (and common in the CKA exam); Traefik was preferred |

## Consequences

- GHCR packages are private by default: either make the package public or use an `imagePullSecret`.
- `local-path` keeps data on one node's disk. If PostgreSQL moves to another node, its data does not follow. Acceptable for a lab.
- The cluster needs 10 GB of RAM; other large VMs must be stopped first.
- Substitution class (see `learning-platform/architecture/principles.md`): the registry and ingress are `configuration-only` swaps; the storage provisioner is `migration` if data must be kept.
