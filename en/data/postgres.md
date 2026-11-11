---
slug: "data/postgres"
title: "Postgres"
description: "Managed Postgres per project with lifecycle controls, backup, restore, and DATABASE_URL injection."
section: "Databases & storage"
order: 4001
---

> **Tip: Available**
>
> Managed Postgres supports dedicated instances, `DATABASE_URL` injection, persistent storage, lifecycle operations, on-demand backup, and restore through the CLI and API.

## Provision

When a project is created with `data: { postgres: true }`, MIOSA provisions a logical database in a managed Postgres cluster, generates credentials, and injects them as env vars.

```mermaid
graph LR
    PC[Project create] -->|data.postgres=true| MP[MIOSA provisioner]
    MP --> PG[(Managed Postgres cluster)]
    PG -->|credentials generated| ENV[DATABASE_URL env var]
    ENV --> SB[Sandbox]
    ENV --> RI[Runtime Instance]
```
```typescript
const project = await miosa.projects.create({
  name: "Team Portal",
  data: { postgres: true },
})

// Sandboxes and runtime instances under this project boot with:
// DATABASE_URL = postgresql://user:pass@host:port/proj_xxx
```

You can also add Postgres to an existing project:
```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "postgres",
  size: "small",  // small | medium | large
})
```

## Create it your way: API, CLI, or SDK

You don't need an SDK. The same managed database can be created and used three ways. All of these hit the same API.

**Raw API (curl):**
```bash
# 1. Create a Postgres database
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-db", "engine": "postgresql"}'
# → { "id": "db_...", "state": "provisioning", ... }   (reaches "running" in ~30-60s)

# 2. Get the connection string once it's running
curl https://api.miosa.ai/api/v1/databases/DB_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
# → { "data": { "recommended_url": "postgresql://user:pass@host:port/miosa_db", ... } }

# 3. Connect with anything that speaks Postgres
psql "postgresql://user:pass@host:port/miosa_db"
```
Other engines: pass `"engine": "mysql"`, `"redis"`, or `"qdrant"`.

**CLI:**
```bash
miosa api POST /databases -d '{"name": "my-db", "engine": "postgresql"}'  # create
miosa db list                     # see them + their state
miosa db connection <id>                    # open a shell to it
miosa db logs <id>                       # tail logs
miosa db backup <id>                     # snapshot
miosa api POST /databases/<id>/restore \
  -d '{"backup_id": "<backup-id>"}'          # restore
```

**SDK** (Python example below; TypeScript/Go/Elixir SDKs have the same methods).

## Connection management

Use a bounded application connection pool and size it for the database plan.
Treat the connection URL returned by the credentials endpoint as authoritative.
Do not assume a separate session or transaction pooler unless the returned database configuration explicitly advertises one.

## Where you can connect from

The connection URL points at a private MIOSA address.
It is reachable from sandboxes, computers and deployments in your organization.
It is not reachable from the public internet, so `psql` on your laptop cannot use it directly.
To work with the database from your own machine, run `psql` inside a sandbox, or open a tunnel through one.

## Schema management is yours

MIOSA doesn't run migrations for you. The recommended workflow:

1. Use your framework's migration tool (Prisma, Drizzle, Ecto, Alembic, etc.).
2. Run migrations as part of your build or startup, or as a separate one-off:

```typescript
await sandbox.exec({ cmd: "npx", args: ["prisma", "migrate", "deploy"] })
```

3. Make migrations backward-compatible (expand → migrate → contract). This keeps rollback working  -  see [Rollback](/docs/docker-host/rollback).

## Owner privileges

The user in your connection URL is the database's owner role.
By default it owns the database and everything it creates, and nothing more: it cannot create roles or databases, and row-level security applies to it.

Some migrations need more.
Migrations written for Supabase or Railway often create their own roles, grant to a role named `postgres`, run `ALTER DEFAULT PRIVILEGES FOR ROLE postgres`, or rely on the owner reading past row-level security.
For those, turn on the **elevated** owner:

| | Standard (default) | Elevated |
|---|---|---|
| Owns the database and its objects | Yes | Yes |
| `CREATEROLE` (create and grant your own roles) | No | Yes |
| `CREATEDB` | No | Yes |
| `BYPASSRLS` (reads past row-level security) | No | Yes |
| Member of a `NOLOGIN` role named `postgres` | No | Yes |
| Superuser | No | No |

The elevated owner is never a superuser.
`COPY ... TO PROGRAM`, untrusted languages such as `plpython3u`, `pg_read_server_files`, `ALTER SYSTEM` and extensions outside the [allowed list](/docs/data/postgres#extensions) stay refused.
A migration that changes the `SUPERUSER` attribute, for example `ALTER ROLE app NOSUPERUSER`, still fails: remove that clause, since roles you create are not superusers anyway.

It is available on PostgreSQL 16 or later, on paid plans (Developer and up), and is off unless you turn it on for a database.

**Turn it on when you create the database:**

```bash
miosa db create app-db --version 16 --owner-privileges elevated
```

```bash
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "app-db", "engine": "postgresql", "owner_privileges": true}'
```

**Or change an existing database:**

```bash
miosa db update app-db --owner-privileges elevated
miosa db update app-db --owner-privileges standard
```

```bash
curl -X PATCH https://api.miosa.ai/api/v1/databases/DB_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"owner_privileges": true}'
```

On a running database the change is applied before the request returns.
On a stopped database it is saved and applied the next time the database starts.
`miosa db get app-db` and the API's `owner_privileges` field show the current setting.

Returning to standard removes the attributes and the `postgres` membership again.
It does not drop roles or databases created while the owner was elevated: drop those yourself if you no longer want them.
Every change made from your organization is recorded in its audit log as `database.owner_privileges.enabled` or `database.owner_privileges.disabled`, with the member or API key that made it.

> **Caution: Who can use it**
>
> Anyone with the owner's connection URL gets these privileges: they can create roles that can sign in, and read rows that row-level security would hide.
> Give your application a separate, less privileged role if it does not need them.

## Extensions

These extensions can be used in a database: `vector`, `pg_trgm`, `pgcrypto`, `uuid-ossp`, `citext`, `hstore`, `btree_gin`, `btree_gist` and `pg_stat_statements`. Ask for the ones you want when you create the database:

```bash
miosa db create app-db --extensions vector,pg_trgm
```

Anything else (for example `adminpack` or `file_fdw`) is refused.

### pg_stat_statements is on by default

Every **new** Postgres database has `pg_stat_statements` turned on, so the console's top-queries view and your own `SELECT ... FROM pg_stat_statements` work from the first minute. It is added to the list you ask for, and `pg_stat_statements.track` is the Postgres default (`top`).

To create a database without it, send `"pg_stat_statements": false` when you create it:

```bash
miosa api POST /databases -d '{"name":"app-db","engine":"postgres","pg_stat_statements":false}'
```

Databases that already exist are not changed. To turn it on for one of them, enable the extension on that database; it takes effect the next time you restart it.

## Backups

MIOSA supports scheduled backups and on-demand backups stored outside the database runtime.
Retention depends on the database plan.

Create an on-demand backup:

```bash
miosa db backup <database-id> --json
```

Restore from a selected backup:

```bash
miosa api POST /databases/<database-id>/restore \
  -d '{"backup_id": "<backup-id>"}'
```

Restore replaces database contents.
Review the target database and backup ID before confirming the operation.

## Lifecycle and recovery

create creates provisioning; provisioning goes to running on ready; running goes to stopped on stop, and back to running on start.

running goes to restarting on restart or maintenance; restarting goes to running on verified.

restarting goes to error on recovery fails.

| State | Meaning |
|---|---|
| `provisioning` | MIOSA is creating storage, credentials, and the database runtime. |
| `running` | The database accepts connections. |
| `stopped` | The database was intentionally stopped. Persistent data remains attached. |
| `restarting` | MIOSA is restoring the desired-running database after maintenance or a restart request. |
| `error` | Recovery did not complete and operator action is required. |

During host maintenance, MIOSA preserves the database ID, credentials, persistent volume, bindings, and desired state.
A database that was running is restarted and verified before its host returns to active service.
An intentionally stopped database remains stopped.

## Sandboxed development with the same DB

When you create a sandbox under a project that has Postgres, the sandbox also gets `DATABASE_URL`  -  pointing at the same managed database. Two options:

1. **Shared dev DB.** Sandboxes and production both point at the production database. Fast iteration, risk of dev work affecting prod data.
2. **Per-environment DB.** Production gets one DB, staging/dev get another. Configured via [Environments](/docs/docker-host/environments).

Default is (2) once per-environment isolation lands. Until then, sandboxes share the production DB; use schema namespacing or be careful with destructive queries.

## Connecting from a sandbox
```typescript
await sandbox.exec({
  cmd: "psql",
  args: [process.env.DATABASE_URL, "-c", "SELECT 1"],
})
```

The sandbox's `DATABASE_URL` is the same shape the runtime would see in production. Whatever code works in the sandbox works in the deployment.

## Limits

| Tier | Storage | Connections (PgBouncer) | Memory |
|---|---|---|---|
| small | 1 GB | 100 | 512 MB |
| medium | 10 GB | 200 | 2 GB |
| large | 100 GB | 500 | 8 GB |

Storage, connection, and memory limits depend on the database plan.
Inspect the database response and metrics rather than assuming automatic storage growth.

## Costs

Postgres pricing is in two parts:

- **Compute**  -  per-hour while the DB is running (always-on by default for production; can be paused for sandboxed dev DBs).
- **Storage**  -  GB-month for data plus WAL.

See [Usage & Billing](/docs/platform/usage-and-billing). Per-database metering carries [attribution](/docs/platform/attribution), so platforms can charge back per end-customer.

## Bring your own Postgres?

If you'd rather use an external Postgres, just set `DATABASE_URL` as an env var on the deployment and skip the managed offering:
```typescript
await miosa.deployments.env.set(deploymentId, {
  environment: "production",
  vars: { DATABASE_URL: "postgresql://...your-external-db..." },
})
```

MIOSA's runtime injects whatever you set. The managed Postgres is convenience; bringing your own is supported.

## Python SDK

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Provision a managed database
db = miosa.databases.create(
    name="team-portal-db",
    engine="postgres",
    size="small",
)
print(db["id"])  # db_...

# Get connection credentials
creds = miosa.databases.credentials(db["id"])
print(creds["url"])  # postgresql://user:pass@host:port/dbname

# Lifecycle operations
miosa.databases.stop(db["id"])
miosa.databases.start(db["id"])
miosa.databases.restart(db["id"])

# Stream logs (returns a generator of SSE events)
for event in miosa.databases.stream_logs(db["id"]):
    print(event)

# Teardown
miosa.databases.delete(db["id"])
```

## See also

- [Overview](/docs/data/overview): the broader data plane
- [MySQL](/docs/data/mysql): if you need MySQL compatibility instead
- [Environments](/docs/docker-host/environments): per-env credentials
- [Rollback](/docs/docker-host/rollback): why backward-compatible migrations matter
