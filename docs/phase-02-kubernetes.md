# Phase 2 — Kubernetes Deployment

> Complete [Phase 1](phase-01-modular-monolith.md) before starting this phase.

---

## Goal

Deploy the **same monolith — no application code changes** — onto a real
kubeadm HA cluster. The infrastructure layer changes entirely; the app
doesn't know it's on Kubernetes.

---

## Cluster overview

The cluster lives at `~/projetos/cka` — 5 VirtualBox VMs managed by Vagrant.

```
Host (Ubuntu 24.04)
    │
    ├── cp1   192.168.56.10   control-plane-1 + HAProxy (API LB :6443)
    ├── cp2   192.168.56.11   control-plane-2
    ├── cp3   192.168.56.12   control-plane-3
    ├── node1 192.168.56.20   worker
    └── node2 192.168.56.21   worker

CNI:        Flannel  (pod CIDR 10.244.0.0/16)
Runtime:    containerd
K8s:        1.29
API server: 192.168.56.10:6443  (HAProxy round-robin to all 3 CP nodes)
```

---

## What you will build

```
Host browser / curl
        │
        ▼  :80
 ┌─────────────────────────────────────────────┐
 │  Kubernetes Cluster                         │
 │                                             │
 │  ┌─────────────────────────────────────┐    │
 │  │  Namespace: order-system            │    │
 │  │                                     │    │
 │  │  Traefik IngressRoute               │    │
 │  │       │                             │    │
 │  │       ▼                             │    │
 │  │  orders-app  Deployment (2 pods)    │    │
 │  │       │  ClusterIP Service          │    │
 │  │       │                             │    │
 │  │       ▼                             │    │
 │  │  postgres  StatefulSet (1 pod)      │    │
 │  │            PersistentVolumeClaim    │    │
 │  │            ClusterIP Service        │    │
 │  └─────────────────────────────────────┘    │
 │                                             │
 │  Traefik (kube-system namespace)            │
 └─────────────────────────────────────────────┘
```

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| VirtualBox | **7.2+** | `sudo apt install virtualbox-7.2` |
| Vagrant | 2.4+ | already installed |
| kubectl | any | `sudo apt install kubectl` |
| Helm | 3+ | see Step 4 |
| Docker | 24+ | already installed (for building the image) |
| gh CLI | any | already installed |

---

## Kubernetes manifests structure

```
monolith-order-system/
└── k8s/
    ├── namespace.yaml
    ├── secret.yaml
    ├── configmap.yaml
    ├── postgres/
    │   ├── statefulset.yaml
    │   ├── service.yaml
    │   └── pvc.yaml
    ├── app/
    │   ├── deployment.yaml
    │   └── service.yaml
    └── ingress/
        └── ingressroute.yaml
```

---

## Step 1 — Verify VirtualBox version

The host runs Linux kernel 7.x. VirtualBox 7.0 fails to build kernel modules
against it. You need VirtualBox 7.2.

```bash
VBoxManage --version
# Must show 7.2.x
```

If it shows 7.0.x:

```bash
sudo apt-get install -y virtualbox-7.2
lsmod | grep vboxdrv   # must return a result
VBoxManage --version   # must show 7.2.x
```

---

## Step 2 — Boot the cluster

```bash
cd ~/projetos/cka
vagrant up
```

This takes 5–10 minutes the first time (downloads the box, provisions all nodes).
On subsequent starts it is faster since the VMs already exist.

**Verify all VMs are running:**

```bash
vagrant status
# cp1    running (virtualbox)
# cp2    running (virtualbox)
# cp3    running (virtualbox)
# node1  running (virtualbox)
# node2  running (virtualbox)
```

---

## Step 3 — Bootstrap the cluster (first time only)

> Skip this step if the cluster was already initialized previously.

### 3.1 Initialize cp1

```bash
vagrant ssh cp1
sudo bash /vagrant/scripts/control_plane_init.sh
```

Wait for the completion message. It saves join commands to `/vagrant/kubeadm-init.log`.

### 3.2 Join cp2

```bash
vagrant ssh cp2
# Paste the --control-plane join command from kubeadm-init.log
# Add --apiserver-advertise-address 192.168.56.11 at the end
```

### 3.3 Join cp3

```bash
vagrant ssh cp3
# Paste the --control-plane join command from kubeadm-init.log
# Add --apiserver-advertise-address 192.168.56.12 at the end
```

### 3.4 Join workers

```bash
vagrant ssh node1
# Paste the worker join command from kubeadm-init.log

vagrant ssh node2
# Paste the same worker join command
```

### 3.5 Verify all nodes ready

```bash
vagrant ssh cp1 -c "kubectl get nodes"
# NAME    STATUS   ROLES           AGE
# cp1     Ready    control-plane   Xm
# cp2     Ready    control-plane   Xm
# cp3     Ready    control-plane   Xm
# node1   Ready    <none>          Xm
# node2   Ready    <none>          Xm
```

---

## Step 4 — Copy kubeconfig to host

```bash
cd ~/projetos/cka

# Create local .kube dir if it doesn't exist
mkdir -p ~/.kube

# Copy admin.conf from cp1 to the host
vagrant ssh cp1 -c "sudo cat /etc/kubernetes/admin.conf" > ~/.kube/config

# Replace the internal API server address with the HAProxy address
# (already set correctly — 192.168.56.10:6443)
chmod 600 ~/.kube/config
```

**Verify kubectl works from host:**

```bash
kubectl get nodes
kubectl get pods -A
```

---

## Step 5 — Install Helm on the host

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

---

## Step 6 — Build and push image to GHCR

```bash
cd ~/projetos/monolith_to_microservices/monolith-order-system

# Login to GHCR (uses your existing gh auth token)
echo $(gh auth token) | docker login ghcr.io -u ggarrett89 --password-stdin

# Build for linux/amd64 (the VMs are amd64)
docker build --platform linux/amd64 -t ghcr.io/ggarrett89/monolith-order-system:0.2.0 .

# Push
docker push ghcr.io/ggarrett89/monolith-order-system:0.2.0
```

> By default GHCR packages are private. Make the package public at:
> https://github.com/users/ggarrett89/packages/container/monolith-order-system/settings
>
> Or use an imagePullSecret (see Troubleshooting below).

---

## Step 7 — Create Kubernetes manifests

Apply the manifests from `monolith-order-system/k8s/`:

```bash
cd ~/projetos/monolith_to_microservices/monolith-order-system
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/postgres/
kubectl apply -f k8s/app/
```

**Verify:**

```bash
kubectl get all -n order-system
# NAME                              READY   STATUS    RESTARTS
# pod/postgres-0                    1/1     Running   0
# pod/orders-app-xxxxxxxxx-xxxxx    1/1     Running   0
# pod/orders-app-xxxxxxxxx-xxxxx    1/1     Running   0
```

---

## Step 8 — Install Traefik ingress controller

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update

helm install traefik traefik/traefik \
  --namespace kube-system \
  --set ports.web.nodePort=30080 \
  --set service.type=NodePort
```

**Verify:**

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=traefik
# NAME                       READY   STATUS    RESTARTS
# traefik-xxxxxxxxxx-xxxxx   1/1     Running   0
```

---

## Step 9 — Apply IngressRoute

```bash
kubectl apply -f k8s/ingress/ingressroute.yaml
```

**Test from host:**

```bash
curl http://192.168.56.20:30080/health
# {"status":"ok"}
```

The app is now reachable at `http://192.168.56.20:30080` (node1's IP + Traefik NodePort).

---

## Step 10 — Verify the full walkthrough

```bash
BASE="http://192.168.56.20:30080"

# Create user
curl -s -X POST $BASE/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}' | python3 -m json.tool

# Create product
curl -s -X POST $BASE/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Kubernetes Course", "price": "99.90"}' | python3 -m json.tool

# Create inventory
curl -s -X POST $BASE/inventory \
  -H "Content-Type: application/json" \
  -d '{"product_id": 1, "quantity": 20}' | python3 -m json.tool

# Place order
curl -s -X POST $BASE/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "product_id": 1, "quantity": 2}' | python3 -m json.tool
```

---

## Step 11 — Scale the deployment and observe

```bash
# Scale to 3 replicas
kubectl scale deployment orders-app -n order-system --replicas=3

# Watch pods come up
kubectl get pods -n order-system -w
```

Now place multiple orders and observe:
- Requests are distributed across 3 pods
- All pods share the same PostgreSQL — the DB is the bottleneck, not the app
- You cannot scale PostgreSQL the same way (it's a StatefulSet with one replica)

This is the key lesson: **horizontal scaling helps the stateless app tier but
the shared database remains a single point of contention**.

---

## Step 12 — Tag and push git

```bash
cd ~/projetos/monolith_to_microservices/monolith-order-system
git add k8s/
git commit -m "feat: phase 2 — kubernetes manifests"
git push
```

---

## Architecture exercise

**Q1.** You scaled the app to 3 replicas. What happens to the database?
Nothing — PostgreSQL is a `StatefulSet` with 1 replica. It does not scale
with the app. All 3 app pods write to the same single database pod.

**Q2.** A worker node goes down. What happens to the pods on it?
The scheduler detects the node is `NotReady` and reschedules the pods onto the
remaining healthy node — within ~5 minutes (default tolerations).

**Q3.** The PostgreSQL pod is deleted. What happens to the data?
The `PersistentVolumeClaim` keeps the data. When the StatefulSet recreates the
pod, it reattaches to the same PVC and data is preserved.

**Q4.** You have 2 replicas of orders-app. A `POST /orders` request hits replica
A, which reserves stock. The payment call hangs. Another request hits replica B
for the same product. What happens?
Both calls run `reserve_stock()` against the same DB concurrently. Without
row-level locking, you can oversell. This is a **race condition** that becomes
visible under real concurrency — another reason to rethink the design.

**Q5.** What is the difference between a `Deployment` and a `StatefulSet`?
`Deployment` pods are interchangeable — they get random names and can be
rescheduled anywhere. `StatefulSet` pods have stable identities (`postgres-0`),
stable DNS names, and ordered startup/shutdown. PostgreSQL needs a StatefulSet
because it requires a stable volume attachment and predictable pod identity.

---

## Troubleshooting

### Image pull errors (ErrImagePull / ImagePullBackOff)

The GHCR image is private by default. Either:

1. Make it public at: `https://github.com/users/ggarrett89/packages`

2. Or create an imagePullSecret on each node:

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=ggarrett89 \
  --docker-password=$(gh auth token) \
  --namespace order-system
```

Then add to `k8s/app/deployment.yaml`:
```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: ghcr-secret
```

### Pod stuck in Pending

```bash
kubectl describe pod <pod-name> -n order-system
# Look for Events section — usually a resource or scheduling issue
```

### App CrashLoopBackOff

```bash
kubectl logs <pod-name> -n order-system
# Usually a DATABASE_URL misconfiguration or postgres not ready yet
```

---

## What we still deliberately have wrong

| Problem | Where | Future solution |
|---|---|---|
| Single PostgreSQL pod | `postgres/statefulset.yaml` | One DB per service (Phase 4) |
| All domains in one container | `app/deployment.yaml` | Separate services (Phase 3+) |
| Shared DB across all domains | `secret.yaml` | DB per service (Phase 4) |
| No resource limits on app | `app/deployment.yaml` | Will add in Phase 7 |
| Race condition on stock reservation | `orders/service.py` | Saga pattern (Phase 6) |

---

## Next phase

[Phase 3 — Microservices + REST →](phase-03-microservices-rest.md)
