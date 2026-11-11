---
slug: "api-reference/databases"
title: "Databases API"
description: "Provision and operate managed databases, backups, restores, branches, replicas, credentials, and placement."
section: "API reference"
order: 12042
---

Managed databases have a lifecycle independent from sandboxes and deployment runtime instances.
Deleting or replacing compute does not implicitly delete an attached database.

Base path: `/api/v1/databases`

The `managed-databases` alias is not available: use `/api/v1/databases`.

## Core lifecycle

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/databases` | List databases visible to the current tenant and workspace scope |
| `POST` | `/databases` | Create a managed database |
| `GET` | `/databases/{id}` | Inspect one database |
| `PATCH` | `/databases/{id}` | Rename it, change its disk size, or change its owner privileges |
| `DELETE` | `/databases/{id}` | Permanently delete one database |
| `POST` | `/databases/{id}/start` | Start a stopped or recoverable database |
| `POST` | `/databases/{id}/stop` | Stop a running database while preserving its volume |
| `POST` | `/databases/{id}/restart` | Restart the database runtime |
| `POST` | `/databases/{id}/wake` | Wake a database that can be resumed |
| `POST` | `/databases/{id}/resize-compute` | Change its compute allocation |
| `POST` | `/databases/{id}/move` | Move the database through the managed placement workflow |

## Data topology and recovery

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/databases/{id}/branch` | Create an independent database branch |
| `POST` | `/databases/{id}/replicas` | Create a replica |
| `POST` | `/databases/{id}/backups` | Create an on-demand backup |
| `GET` | `/databases/{id}/backups` | List backups |
| `GET` | `/databases/{id}/backups/{backup_id}` | Inspect one backup |
| `POST` | `/databases/{id}/restore` | Restore from a selected backup |
| `GET` | `/databases/{id}/restores` | List restore operations |
| `GET` | `/databases/{id}/credentials` | Return the current connection contract |

Logs and metrics use the common observability endpoints:

```http
GET /api/v1/databases/{id}/logs
GET /api/v1/databases/{id}/logs/stream
GET /api/v1/databases/{id}/metrics
```

## Create

```bash
curl -X POST https://api.miosa.ai/api/v1/databases \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Idempotency-Key: $IDEMPOTENCY_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"app-production","engine":"postgresql"}'
```

Creation is asynchronous.
Wait until the returned database reaches `running` before connecting.

```bash
miosa api POST /databases -d '{"name":"app-production","engine":"postgresql"}'
```

## Owner privileges

`owner_privileges` (boolean, default `false`) gives a PostgreSQL 16+ database's owner role `CREATEROLE`, `CREATEDB`, `BYPASSRLS` and membership in a `NOLOGIN` role named `postgres`.
It is never a superuser.
Send it on `POST /databases` to create the database with it, or on `PATCH /databases/{id}` to turn it on or off.
Every database response carries the current value.
See [Postgres: owner privileges](/docs/data/postgres#owner-privileges) for what it allows and when to use it.

```bash
miosa api PATCH /databases/<database-id> -d '{"owner_privileges": true}'
```

Refusals:

- `402 PLAN_LIMIT_EXCEEDED`: turning it on needs a paid plan. Turning it off is always allowed.
- `422 OWNER_PRIVILEGES_UNSUPPORTED`: the database is not PostgreSQL 16 or later.
- `503 OWNER_PRIVILEGES_NOT_APPLIED`: the database is running and its host could not apply the change. Nothing changed; retry.

## Credentials

```bash
curl https://api.miosa.ai/api/v1/databases/$DATABASE_ID/credentials \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Use the returned recommended connection URL.
Do not construct hostnames, ports, usernames, or database names from resource metadata.

## Backup and restore

```bash
miosa db backup <database-id> --json
miosa api POST /databases/<database-id>/restore \
  -d '{"backup_id": "<backup-id>", "allow_overwrite": true}'
```

Restore replaces the target database contents with the selected backup.
Confirm the database ID and backup ID before approving the operation.

> **Note: Application migrations are separate**
>
> Database restore does not change the active application release.
> Application rollback does not reverse database migrations.
> Use backward-compatible expand, migrate, and contract changes when releases must remain rollback-safe.

## Permissions

| Action | Required scope |
|---|---|
| List, inspect, read metrics, or fetch credentials | `databases:read` |
| Create, update, resize, branch, back up, restore, move, or delete | `databases:write` |

## See also

- [Data overview](/docs/data/overview): Choose the correct managed data product and understand its lifecycle.
- [Postgres](/docs/data/postgres): Connect applications and manage schema changes safely.
- [Environments](/docs/docker-host/environments): Bind separate databases to production, staging, and development.
