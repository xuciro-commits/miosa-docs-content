---
slug: "api-reference/services"
title: "Services API"
description: "API reference for managing long-running systemd services inside MIOSA computers."
section: "API reference"
order: 12033
---

Services are persistent processes managed by systemd inside a computer. Use them for web servers, background workers, databases, or any daemon that should survive across exec calls and restart automatically on failure.

Base path: `/api/v1/computers/{id}/services`

> **Note**
>
> Services map directly to systemd unit files (`/etc/systemd/system/miosa-{name}.service`) managed by the computer command service. The computer must be **running** to interact with services.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Register and start a web server
const svc = await client.services.create(computerId, {
  name: 'webserver',
  command: 'python3 -m http.server 8080',
  cwd: '/workspace/app',
  restart: 'always',
});

console.log(svc.status); // "starting"

// Check status later
const updated = await client.services.get(computerId, svc.id);
console.log(updated.status); // "running"
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "webserver",
    "command": "python3 -m http.server 8080",
    "cwd": "/workspace/app",
    "restart": "always"
  }'
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/computers/{id}/services` | Create and start a service |
| `GET` | `/computers/{id}/services` | List all services on a computer |
| `GET` | `/computers/{id}/services/{name}` | Get a service by name |
| `POST` | `/computers/{id}/services/{name}/start` | Start a stopped service |
| `POST` | `/computers/{id}/services/{name}/stop` | Stop a running service |
| `POST` | `/computers/{id}/services/{name}/restart` | Restart a service |
| `DELETE` | `/computers/{id}/services/{name}` | Remove a service |

---

## Create a Service

**`POST /api/v1/computers/{id}/services`**

Writes the systemd unit file and starts the service.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Service name. Pattern: `[a-z][a-z0-9-]{0,62}` |
| `command` | string | Yes | Full command to run |
| `env` | object | No | Environment variables as `{ "KEY": "value" }` |
| `cwd` | string | No | Working directory (default: `/workspace`) |
| `restart` | string | No | `"always"`, `"on-failure"` (default), or `"no"` |

### Response  -  `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "tenant_id": "...",
    "name": "webserver",
    "command": "python3 -m http.server 8080",
    "env": {},
    "cwd": "/workspace/app",
    "restart": "always",
    "status": "starting",
    "pid": null,
    "exit_code": null,
    "last_started_at": null,
    "last_exited_at": null,
    "inserted_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### Status Values

| Status | Description |
|--------|-------------|
| `stopped` | Registered but not started |
| `starting` | Start dispatched to systemd |
| `running` | systemd reports `ActiveState=active` |
| `failed` | systemd reports `ActiveState=failed` |
| `removed` | Unit file deleted; record kept for audit |

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `a service with this name already exists` | Name must be unique per computer |
| 422 | Validation error | Invalid name format or restart policy |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "api-server",
    "command": "/workspace/app/server",
    "env": {"PORT": "3000", "NODE_ENV": "production"},
    "cwd": "/workspace/app",
    "restart": "on-failure"
  }'
```

---

## List Services

**`GET /api/v1/computers/{id}/services`**

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "name": "webserver",
      "status": "running",
      "pid": 12345,
      "restart": "always",
      "inserted_at": "2026-04-11T00:00:00Z"
    }
  ]
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/services \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get a Service

**`GET /api/v1/computers/{id}/services/{name}`**

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Computer ID |
| `name` | string | Service name |

### Response  -  `200 OK`

Full service object (same as create response).

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 404 | `service not found` | Does not exist on this computer |

---

## Start, Stop, Restart

**`POST /api/v1/computers/{id}/services/{name}/start`**
**`POST /api/v1/computers/{id}/services/{name}/stop`**
**`POST /api/v1/computers/{id}/services/{name}/restart`**

All return a `200 OK` with the updated service object.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 409 | `service is already running` | Cannot start a running service |
| 409 | `service is not running` | Cannot stop a non-running service |
| 502 | `AGENT_UNAVAILABLE` | In-VM agent unreachable |
```bash
# Stop a service
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services/webserver/stop \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Restart it
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/services/webserver/restart \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Remove a Service

**`DELETE /api/v1/computers/{id}/services/{name}`**

Stops the service and removes the systemd unit file. The database record transitions to `"removed"` status.

### Response  -  `200 OK`
```json
{
  "data": { "deleted": true }
}
```
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/services/webserver \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Common Recipes

### Deploy a Node.js app as a service
```typescript
// Upload app files first
await client.files.write(computerId, {
  path: '/workspace/app/server.js',
  content: serverJsContent,
});

// Register and start
const svc = await client.services.create(computerId, {
  name: 'node-app',
  command: 'node /workspace/app/server.js',
  env: { PORT: '3000' },
  restart: 'on-failure',
});
```

### Wait until a service is running
```python
import time

svc = client.services.create(computer_id, name="worker", command="/usr/bin/worker")
while svc.status == "starting":
    time.sleep(1)
    svc = client.services.get(computer_id, svc.name)

if svc.status != "running":
    raise RuntimeError(f"Service failed to start: {svc.status}")
```

### Check for failed services in a fleet
```typescript
const computers = await client.computers.list();
for (const computer of computers.data) {
  if (computer.status !== 'running') continue;
  const services = await client.services.list(computer.id);
  const failed = services.data.filter(s => s.status === 'failed');
  if (failed.length > 0) {
    console.log(`${computer.name}: ${failed.length} failed services`);
  }
}
```
