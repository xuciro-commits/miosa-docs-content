---
slug: "data/overview"
title: "Data"
description: "Managed databases, object storage, auth, and volumes with explicit lifecycle, credentials, and recovery operations."
section: "Databases & storage"
order: 4000
---

MIOSA Data services give your sandboxes and production runtime instances access to persistent storage without any external account setup. When you attach a data service to a project, MIOSA provisions the resource and injects its credentials at boot  -  your code reads `process.env.DATABASE_URL` and connects. That is the entire integration.

Do it now

Create your first database

A managed Postgres instance with credentials you can inject as DATABASE_URL.

- [Create database](https://miosa.ai/databases/new): (opens the MIOSA console in a new tab)

miosa db create postgres --name my-db --wait

Next [Managed Postgres](/docs/data/postgres)

## Phase status

> **Note: Current service status**
>
> Data services ship incrementally. This table reflects verified platform state:
>
> | Service | Status |
> |---|---|
> | Object Storage | **Available** - buckets, objects, presigned URLs, quota, per-tenant |
> | Cron Jobs | **Available** - CLI, MCP, SDK, and backend all wired |
> | Managed Postgres | **Generally available** - dedicated instances, credential injection, lifecycle operations, backup, and restore |
> | Managed MySQL | **Generally available** - dedicated instances, credential injection, and lifecycle operations |
> | Managed Redis | **Generally available** - per-project instances and `REDIS_URL` injection |
> | Managed MongoDB | **Generally available** - per-project document databases and `MONGODB_URI` injection |
> | Managed Qdrant (vector) | **Generally available** - per-project vector databases and `QDRANT_URL` injection |
> | Auth (per-project end-user auth) | **Generally available** - opt-in signup, login, JWT, password reset, and email confirmation |
> | Volumes (persistent block storage) | **Generally available** - create, attach, detach, delete, and meter host-local volumes |
>
> Per-page callouts indicate any in-progress edge cases.

## Mental model

Each service fills a different role. Pick the right one for your data shape:

| Service | Use when you need | Persistence |
|---|---|---|
| **Postgres** | Relational data, joins, transactions, migrations | Durable - survives deploys and sandbox destruction |
| **MySQL** | MySQL-compatible apps, existing MySQL schemas | Durable - survives deploys and sandbox destruction |
| **Redis** | Cache, sessions, pub/sub, rate limiting | Durable (AOF/RDB) but treated as cache-class |
| **MongoDB** | Document data, flexible schemas | Durable - survives deploys and sandbox destruction |
| **Qdrant** | Vector search, semantic similarity, embeddings | Durable - collection data persists across VMs |
| **Storage** | File uploads, generated assets, binary blobs | Durable - content-addressed object store |
| **Auth** | Per-project end-user signup, login, and JWT issuance | Durable - user records outlive any VM |
| **Volumes** | Large or fast filesystem state that does not fit in Postgres | Attached to one runtime instance at a time |

## Data persists past your VMs

Sandboxes are persistent by default and can be paused, resumed, snapshotted, or destroyed.
Deployment runtime instances are replaceable.
Managed data has an independent lifecycle and is not deleted when either compute surface is replaced.

Destroying a sandbox removes the VM  -  the connected Postgres database is untouched. Publishing a new version does not migrate your schema. Rolling back a deployment does not roll back your database. Data services are managed independently of the compute that connects to them.

This is intentional. It is also why production stays available during deploys: the new version connects to the same database the old version was using.

## How credentials are injected

MIOSA can bind different database credentials to different deployment environments:
```text

production runtime  →  DATABASE_URL = postgres://prod-db...
staging runtime     →  DATABASE_URL = postgres://staging-db...
```

You ship one environment variable name and MIOSA resolves the binding for that environment.
Environment isolation must be configured explicitly.
Until separate development and production databases are attached, a sandbox may share the production database.
See [Environments](/docs/docker-host/environments).

## Quick example

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

# Create a managed Postgres database
db = miosa.databases.create(
    engine="postgres",
    engine_version="15",
    name="my-app-db",
)

print(db["id"])          # db_...
print(db["status"])      # provisioning | running

# Fetch connection credentials once ready
creds = miosa.databases.credentials(db["id"])
print(creds["url"])      # postgresql://user:pass@host:5432/dbname
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

// Create a managed Postgres database
const db = await miosa.databases.create({
  engine: "postgres",
  engineVersion: "15",
  name: "my-app-db",
})

console.log(db.id)       // db_...
console.log(db.status)   // provisioning | running

// Fetch connection credentials once ready
const creds = await miosa.databases.credentials(db.id)
console.log(creds.url)   // postgresql://user:pass@host:5432/dbname
```

**cURL**

```bash
# Create a managed Postgres database
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"engine":"postgres","engine_version":"15","name":"my-app-db"}'

# Get credentials
curl https://api.miosa.ai/api/v1/databases/$DATABASE_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## How data services attach
```mermaid
graph TB
    DEP[Deployment / Sandbox]
    DEP -->|DATABASE_URL injected| PG[(Postgres)]
    DEP -->|DATABASE_URL injected| MY[(MySQL)]
    DEP -->|REDIS_URL injected| RD[(Redis)]
    DEP -->|QDRANT_URL injected| QD[(Qdrant)]
    DEP -->|STORAGE_BUCKET + STORAGE_ENDPOINT + keys injected| ST[(Object Storage)]
    DEP -->|AUTH_URL + AUTH_JWT_SECRET injected| AU[Auth Service]
    DEP -->|mount path injected| VL[Volume]
```

## How it fits together
```mermaid
flowchart LR
  P[Project] --> S["Sandbox<br/>persistent-by-default development compute"]
  P --> D["Data services (durable)<br/>Postgres, MySQL, Redis, MongoDB, Qdrant, Storage, Auth (opt-in), Volumes"]
  D -->|"connection env vars injected at boot"| S
```

Your application code reads standard environment variables and opens normal client connections.
The application remains responsible for schema migrations, backward compatibility, and safe handling of destructive operations.

## Managed database lifecycle

Managed databases expose explicit lifecycle and recovery operations:

create creates provisioning; provisioning goes to running on ready; running goes to stopped on stop, and back to running on start.

running goes to restarting on restart or maintenance; restarting goes to running on verified.

restarting goes to error on recovery fails.

The API supports start, stop, restart, wake, resize, branch, replica creation, backup, restore, move, credentials, logs, and metrics.
The current CLI exposes the common create, inspect, lifecycle, connect, backup, restore, logs, and metrics operations.
See [Databases API](/docs/api-reference/databases) for the exact endpoints.

## Related

- [Postgres](/docs/data/postgres): Managed Postgres per project with lifecycle operations, backup, restore, and DATABASE_URL injection.
- [MySQL](/docs/data/mysql): Managed MySQL per project. DATABASE_URL injection, dedicated instances, data persists across VM destroy.
- [Redis](/docs/data/redis): Managed Redis per project. Cache, session store, and pub/sub. REDIS_URL injected at boot.
- [MongoDB](/docs/data/mongodb): Managed MongoDB per project. Document data. MONGODB_URI injected at boot.
- [Qdrant (Vector)](/docs/data/qdrant): Per-project vector database. Semantic search and embedding storage. QDRANT_URL injected at boot.
- [Storage](/docs/data/storage): Object storage buckets per project. Signed-URL uploads and downloads, no credentials in the browser.
- [Auth](/docs/data/auth): Opt-in per-project end-user auth. Signup, login, JWT. `AUTH_URL` + `AUTH_JWT_SECRET` injected at boot.
- [Transactional Email](/docs/data/email): Resend-backed auth email for password reset and email confirmation. Operator-level setup, not per-project.
- [Volumes](/docs/data/volumes): Persistent block storage attached to a runtime instance. Data survives VM destroy and reattaches.
- [Fractal Computing](/docs/data/fractal): Enterprise data partner pattern for client-wide structured data, high-volume calculations, reconciliation, and private data-plane deployments.

## Next steps

- [Connect Postgres](/docs/data/postgres): Credentials, DATABASE_URL injection, backup and restore.
- [Add object storage](/docs/data/storage): Buckets and signed-URL uploads with no keys in the browser.
- [Attach a volume](/docs/data/volumes): Persistent block storage that survives VM destroy.
