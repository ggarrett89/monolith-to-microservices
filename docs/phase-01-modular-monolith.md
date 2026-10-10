# Phase 1 — Modular Monolith

## Goal

Build a working order management system as a **modular monolith**:
one process, one database, five business domains separated by directory.

This is the intentional starting point. The coupling between domains is
real and visible — we keep it that way so later phases have concrete
problems to solve.

---

## Prerequisites

| Tool | Version |
|---|---|
| Docker | 24+ |
| Docker Compose | v2 (bundled with Docker Desktop) |
| Python | 3.12+ (local dev only) |
| git | any |

Verify:

```bash
docker --version
docker compose version
python3 --version
git --version
```

---

## Architecture overview

```
         Browser / curl
               │
               ▼
       ┌───────────────┐
       │    FastAPI    │
       │   MONOLITH    │
       │               │
       │  users/       │
       │  products/    │
       │  inventory/   │
       │  orders/      │
       │  payments/    │
       └──────┬────────┘
              │
              ▼
       ┌───────────────┐
       │  PostgreSQL   │
       └───────────────┘
```

Order creation flow:

```
POST /orders
    │
    ├── 1. validate user          (users domain)
    ├── 2. validate product       (products domain)
    ├── 3. reserve_stock()        (inventory domain)
    ├── 4. process_payment()      (payments domain)
    │         └── REJECTED? → release_stock()   ← compensating tx
    └── 5. persist Order (status=CONFIRMED)
```

---

## Project structure

```
monolith-order-system/
├── app/
│   ├── __init__.py
│   ├── main.py               # FastAPI app entry point
│   ├── database.py           # Shared SQLAlchemy engine + Base
│   ├── users/
│   │   ├── __init__.py
│   │   ├── models.py         # SQLAlchemy ORM model
│   │   ├── schemas.py        # Pydantic request/response schemas
│   │   ├── service.py        # Business logic
│   │   └── router.py         # FastAPI router
│   ├── products/             # same structure
│   ├── inventory/            # same structure
│   ├── orders/               # same structure
│   └── payments/             # same structure
├── tests/
│   ├── users/
│   ├── products/
│   ├── inventory/
│   ├── orders/
│   └── payments/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── .gitignore
```

---

## Step 1 — Clone or create the project

If you are following this lab from scratch, the code already lives in
`monolith-order-system/` at the root of this repository.

```bash
git clone <your-repo-url>
cd monolith-to-microservices/monolith-order-system
```

**Verify:**

```bash
ls app/
# users  products  inventory  orders  payments  database.py  main.py  __init__.py
```

---

## Step 2 — Review the dependencies

Open `requirements.txt`:

```
fastapi==0.115.0
uvicorn[standard]==0.30.6
sqlalchemy==2.0.35
psycopg2-binary==2.9.9
pydantic==2.9.2
pydantic-settings==2.5.2
python-dotenv==1.0.1
```

Key points:
- All versions are **pinned** — avoids surprise breakage when packages update.
- `psycopg2-binary` is the PostgreSQL driver for SQLAlchemy.
- `pydantic[email]` is needed for `EmailStr` in the users schema.

---

## Step 3 — Understand the database module

Open `app/database.py`:

```python
DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://orders:orders@localhost:5432/orders",
)
```

Two things to notice:

1. The URL comes from an **environment variable** — this is what lets Docker
   Compose and later Kubernetes inject different values without changing code.
2. There is a single shared `Base = declarative_base()`.
   Every domain model imports this same `Base`, so
   `Base.metadata.create_all()` in `main.py` creates all tables at once.

---

## Step 4 — Understand the domain structure

Each domain follows the same four-file pattern:

| File | Responsibility |
|---|---|
| `models.py` | SQLAlchemy table definition |
| `schemas.py` | Pydantic input/output shapes |
| `service.py` | Business logic (no HTTP knowledge) |
| `router.py` | HTTP endpoints (calls service, raises HTTPException) |

This separation is intentional. When we extract a domain into a
microservice, `service.py` becomes the core of the new service and
`router.py` barely changes.

---

## Step 5 — Notice the cross-domain coupling in orders

Open `app/orders/service.py` and read `create_order()`.

```python
from app.inventory import service as inventory_service
from app.payments import service as payment_service
from app.products import service as product_service
from app.users import service as user_service
```

This is the coupling we are keeping intentionally.

Ask yourself:
- What happens if the payments service is slow?
- What happens if inventory is unavailable?
- Can we deploy orders without deploying inventory?

Write down your answers. We will revisit them in Phase 3.

---

## Step 6 — Notice the compensating transaction

Still in `create_order()`:

```python
try:
    payment_service.process_payment(db, user_id=data.user_id, amount=total)
except ValueError:
    # Compensating transaction: release the reserved stock
    inventory_service.release_stock(db, data.product_id, data.quantity)
    raise
```

Stock is reserved **before** payment is attempted. If payment fails,
stock must be manually released. This is the embryonic form of the
**Saga pattern** we will implement properly in Phase 6.

---

## Step 7 — Run with Docker Compose

```bash
docker compose up --build
```

Expected output (last lines):

```
orders-db   | database system is ready to accept connections
orders-app  | INFO:     Application startup complete.
orders-app  | INFO:     Uvicorn running on http://0.0.0.0:8000
```

**Verify:**

```bash
curl -s http://localhost:8000/health
# {"status":"ok"}
```

Open the interactive docs: [http://localhost:8000/docs](http://localhost:8000/docs)

---

## Step 8 — Create test data

Run these commands in order. Each depends on the previous one.

### 8.1 Create a user

```bash
curl -s -X POST http://localhost:8000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}' | python3 -m json.tool
```

Expected:

```json
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
}
```

### 8.2 Create a product

```bash
curl -s -X POST http://localhost:8000/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Kubernetes Course", "description": "K8s from zero", "price": "99.90"}' | python3 -m json.tool
```

Expected:

```json
{
    "id": 1,
    "name": "Kubernetes Course",
    "description": "K8s from zero",
    "price": "99.90"
}
```

### 8.3 Create an inventory record

Stock lives in the inventory domain, not in the product.

```bash
curl -s -X POST http://localhost:8000/inventory \
  -H "Content-Type: application/json" \
  -d '{"product_id": 1, "quantity": 20}' | python3 -m json.tool
```

Expected:

```json
{
    "id": 1,
    "product_id": 1,
    "quantity": 20
}
```

### 8.4 Place an order

```bash
curl -s -X POST http://localhost:8000/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "product_id": 1, "quantity": 2}' | python3 -m json.tool
```

Expected (payment approved):

```json
{
    "id": 1,
    "user_id": 1,
    "product_id": 1,
    "quantity": 2,
    "total": "199.80",
    "status": "CONFIRMED"
}
```

If you get `400 Payment rejected` — that is the **10% simulated failure
rate** working correctly. Retry the request.

---

## Step 9 — Verify side effects

### 9.1 Check inventory was decremented

```bash
curl -s http://localhost:8000/inventory/1 | python3 -m json.tool
# quantity should be 18 (20 - 2)
```

### 9.2 Check payment was recorded

```bash
curl -s http://localhost:8000/payments/user/1 | python3 -m json.tool
# status: "APPROVED"
```

### 9.3 Cancel the order and verify stock is restored

```bash
curl -s -X POST http://localhost:8000/orders/1/cancel | python3 -m json.tool
# status: "CANCELLED"

curl -s http://localhost:8000/inventory/1 | python3 -m json.tool
# quantity should be 20 again
```

---

## Step 10 — Trigger a payment failure

The payment service rejects ~10% of requests. To force a failure for
testing, temporarily edit `app/payments/service.py`:

```python
PAYMENT_FAILURE_RATE = 1.0   # always reject
```

Because Docker Compose mounts the source directory with `--reload`,
the change takes effect immediately without rebuilding.

Place an order:

```bash
curl -s -X POST http://localhost:8000/orders \
  -H "Content-Type: application/json" \
  -d '{"user_id": 1, "product_id": 1, "quantity": 2}' | python3 -m json.tool
```

Expected: `400 Payment rejected: Simulated gateway rejection`

Check inventory — it should **not** have decreased:

```bash
curl -s http://localhost:8000/inventory/1 | python3 -m json.tool
# quantity unchanged — compensating transaction worked
```

Reset the failure rate back to `0.1` when done.

---

## Step 11 — First Git commit

```bash
cd monolith-order-system

git init
git add .
git commit -m "feat: phase 1 — modular monolith"
```

Push to GitHub (create the repo first if needed):

```bash
gh repo create monolith-order-system --public --source=. --remote=origin --push
```

Or with plain git:

```bash
git remote add origin git@github.com:<your-username>/monolith-order-system.git
git push -u origin main
```

---

## Architecture exercise

Before moving to Phase 2, answer these questions yourself.

**Q1.** Where does inventory live?  
`app/inventory/` — but it is still in the same process and database as orders.

**Q2.** What happens if we want to deploy orders without deploying payments?  
We can't. They are the same application.

**Q3.** What happens if inventory needs to scale 10× but users doesn't?  
We have to scale the entire monolith.

**Q4.** What happens if `create_order()` takes 5 seconds due to a slow payment gateway?  
Every other request to the same process is affected.

**Q5.** What happens if we need to change the payments database schema?  
It is the same database as orders, users, products, and inventory.
Any migration touches all domains.

**Q6.** The compensating transaction in `orders/service.py` releases stock
when payment fails. What happens if the application crashes **between**
`reserve_stock()` and `release_stock()`?  
Stock is permanently decremented even though no order was confirmed.
This is the **partial failure problem** that Saga solves in Phase 6.

---

## What we built deliberately wrong

| Coupling | Location | Why it's a problem |
|---|---|---|
| Orders imports inventory directly | `orders/service.py` | Can't deploy independently |
| Orders imports payments directly | `orders/service.py` | Slow payment blocks all orders |
| Single shared database | `database.py` | Schema changes affect all domains |
| Manual compensating transaction | `orders/service.py` | Not durable — crashes lose state |
| One process for everything | `main.py` | Can't scale domains independently |

These are not bugs. They are the **architecture problems** we will solve
step by step in the phases ahead.

---

## Next phase

[Phase 2 — Kubernetes Deployment →](phase-02-kubernetes.md)

Deploy the same monolith on a local Kubernetes cluster using k3d,
write the Deployment and Service manifests, and expose the app with
an Ingress.
