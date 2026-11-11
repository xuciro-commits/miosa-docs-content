---
slug: "api-reference/snapshots"
title: "Snapshots API"
description: "API reference for creating, listing, and restoring microVM snapshots (checkpoints)."
section: "API reference"
order: 12025
---

Snapshots capture the full in-memory state of a running computer  -  CPU state, RAM, and filesystem  -  and store it durably. Restore a snapshot to spin up an identical computer in seconds.

> **Note: Looking for sandbox snapshots?**
>
> This page documents desktop Computer snapshots only.
> Canonical sandbox V1 does not expose direct snapshot CRUD; use [Persistence, pause, and forks](/docs/develop/snapshots) and the [fork endpoint](/docs/api-reference/sandboxes#fork-a-sandbox).

Base path: `/api/v1/computers/{id}/snapshots`

> **Note**
>
> Snapshots require the computer to be **running**. Creating a snapshot does not stop the computer.

---

## Quick Start
```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa();

// Create a snapshot
const snap = await client.snapshots.create(computerId, {
  comment: 'before-deploy-v2',
});

// Poll until ready
let status = snap.status;
while (status !== 'ready') {
  await new Promise(r => setTimeout(r, 3000));
  const updated = await client.snapshots.get(computerId, snap.id);
  status = updated.status;
}

// Restore  -  creates a new computer from the snapshot
const restored = await client.snapshots.restore(computerId, snap.id);
console.log(`Restored computer: ${restored.id}`);
```
```bash
# Create a snapshot
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"comment": "before-deploy-v2"}'
```

---

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/computers/{id}/snapshots` | Create a snapshot |
| `GET` | `/computers/{id}/snapshots` | List snapshots for a computer |
| `GET` | `/computers/{id}/snapshots/{snap_id}` | Get a snapshot |
| `DELETE` | `/computers/{id}/snapshots/{snap_id}` | Delete a snapshot |
| `POST` | `/computers/{id}/restore/{snap_id}` | Restore from snapshot |
| `GET` | `/computers/{id}/snapshots/{snap_id}/events` | SSE progress stream |

---

## Create a Snapshot

**`POST /api/v1/computers/{id}/snapshots`**

Initiates an asynchronous snapshot. The returned status will be `"creating"`. Poll `GET /snapshots/{snap_id}` or subscribe to SSE events for progress.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `comment` | string | No | Human-readable label (max 500 chars) |

### Response  -  `201 Created`
```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "computer_id": "...",
    "tenant_id": "...",
    "comment": "before-deploy-v2",
    "status": "creating",
    "state_size_bytes": null,
    "memory_size_bytes": null,
    "rootfs_size_bytes": null,
    "compressed_size_bytes": null,
    "s3_bucket": null,
    "s3_prefix": null,
    "error": null,
    "created_at": "2026-04-11T00:00:00Z",
    "updated_at": "2026-04-11T00:00:00Z"
  }
}
```

### Status State Machine

creating goes to uploading; uploading goes to ready.

ready goes to deleted on delete.

any state goes to failed on failure.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 404 | `computer not found` | Computer does not exist or wrong tenant |
| 409 | `computer is not running` | Snapshot requires running VM |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"comment": "checkpoint-before-risky-op"}'
```

---

## List Snapshots

**`GET /api/v1/computers/{id}/snapshots`**

### Response  -  `200 OK`
```json
{
  "data": [
    {
      "id": "...",
      "comment": "before-deploy-v2",
      "status": "ready",
      "compressed_size_bytes": 524288000,
      "created_at": "2026-04-11T00:00:00Z"
    }
  ]
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/snapshots \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Get a Snapshot

**`GET /api/v1/computers/{id}/snapshots/{snap_id}`**

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | UUID | Computer ID |
| `snap_id` | UUID | Snapshot ID |

### Response  -  `200 OK`

Full snapshot object (same shape as create response).

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 403 | `forbidden` | Snapshot belongs to a different tenant |
| 404 | `snapshot not found` | Does not exist |
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/snapshots/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Delete a Snapshot

**`DELETE /api/v1/computers/{id}/snapshots/{snap_id}`**

Soft-deletes the snapshot (transitions to `"deleted"` status). S3 objects are cleaned up asynchronously.

### Response  -  `200 OK`
```json
{
  "data": {
    "id": "...",
    "status": "deleted"
  }
}
```

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 404 | `snapshot not found` | Does not exist |
| 409 | `snapshot is not in a deletable state` | Snapshot is being restored |
```bash
curl -X DELETE https://api.miosa.ai/api/v1/computers/{id}/snapshots/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Restore from Snapshot

**`POST /api/v1/computers/{id}/restore/{snap_id}`**

Creates a new computer with the state from the snapshot. The original computer is unchanged.

### Response  -  `201 Created`
```json
{
  "data": {
    "id": "new-computer-uuid",
    "name": "my-computer-restored",
    "status": "provisioning",
    "size": "small"
  },
  "snapshot": {
    "id": "snap-uuid",
    "status": "restoring"
  }
}
```

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 404 | `snapshot not found` | Does not exist |
| 409 | `snapshot is not ready` | Snapshot must be in `ready` state |
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/restore/{snap_id} \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## SSE Progress Stream

**`GET /api/v1/computers/{id}/snapshots/{snap_id}/events`**

Subscribe to real-time snapshot progress. Requires a short-lived ticket rather than a Bearer token (browsers cannot set Authorization on EventSource connections).

### Event Types

| Event | Description |
|-------|-------------|
| `snapshot_creating` | VM writing memory and VM state to disk |
| `snapshot_uploading` | Compressing and uploading to object storage |
| `snapshot_ready` | Upload complete; snapshot is usable |
| `snapshot_failed` | Unrecoverable error  -  see `error` field |

---

## Common Recipes

### Automated checkpoint before risky operations
```python
from miosa import Miosa
import time

client = Miosa()

snap = client.snapshots.create(computer_id, comment="pre-migration")
while snap.status not in ("ready", "failed"):
    time.sleep(2)
    snap = client.snapshots.get(computer_id, snap.id)

if snap.status == "failed":
    raise RuntimeError(f"Snapshot failed: {snap.error}")

# Proceed with risky operation knowing you can restore
do_migration(computer_id)
```

### Snapshot-based parallelism (clone a baseline)
```typescript
// Build a baseline environment once, clone it N times for parallel jobs
const baseline = await client.snapshots.create(setupComputerId, {
  comment: 'baseline-env',
});

// Each restore creates an independent computer from the same state
const workers = await Promise.all(
  Array.from({ length: 5 }, () =>
    client.snapshots.restore(setupComputerId, baseline.id)
  )
);
```

---

## See also

- [Computers API](/docs/api-reference/computers)  -  computer lifecycle
- [Events (SSE)](/docs/api-reference/events)  -  stream snapshot progress events
- [Error Codes](/docs/api-reference/errors)  -  snapshot-specific error codes
