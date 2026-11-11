---
slug: "data/volumes"
title: "Volumes"
description: "Persistent block storage attached to a runtime instance. State that doesn't fit in Postgres."
section: "Databases & storage"
order: 4007
---

> **Tip: Available**
>
> Volumes are live.
> Create a persistent block device, attach it to a compatible computer on the same host, and preserve writes independently of the computer lifecycle.
> Attachment to a replacement computer is explicit.
> Most workloads should use [Postgres](/docs/data/postgres) or [Storage](/docs/data/storage) because local volumes are host-bound.

A **Volume** is a persistent block device attached to a compatible computer.
It survives ordinary computer restarts while the host-local volume remains healthy.
It does not automatically move across hosts.
Useful cases include:

- SQLite databases that need to be local to the runtime.
- Caches you can't afford to lose on restart.
- LLM model weights downloaded once and shared across instances.
- Workloads that legitimately need filesystem state (not just key-value).

Volumes are NOT useful for:

- Stateful databases  -  use [Postgres](/docs/data/postgres) instead. It scales better, backs up automatically, and survives host failures.
- User-uploaded files  -  use [Storage / Buckets](/docs/data/storage). Cheaper, browser-uploadable, geo-replicable.
- Shared state across multiple replicas  -  volumes are typically attached to ONE runtime instance.

## Create

**TypeScript**

```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "volume",
  name: "model-weights",
  sizeGb: 50,
})
```

**Python**

```python
from miosa import Miosa

miosa = Miosa(api_key="msk_u_...")

# Create a 50 GB volume
vol = miosa.volumes.create(name="model-weights", size_gb=50)
print(vol["id"])  # vol_...

# List all volumes
for v in miosa.volumes.list():
    print(v["name"], v["size_gb"], "GB")

# Delete a volume
miosa.volumes.delete(vol["id"])
```

## Use it your way: API, CLI, or SDK

No SDK required.

**Raw API (curl):** the raw API takes `size_bytes` and a `backend` (use `local`).
```bash
# Create a 50 GB volume
curl -X POST https://api.miosa.ai/api/v1/volumes \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"name": "model-weights", "size_bytes": 53687091200, "backend": "local"}'

# Attach it to a running computer at a mount path
curl -X POST https://api.miosa.ai/api/v1/computers/COMPUTER_ID/volumes \
  -H "Authorization: Bearer $MIOSA_API_KEY" -H "Content-Type: application/json" \
  -d '{"volume_id": "VOL_ID", "mount_path": "/mnt/data"}'

# Detach (by attachment id)
curl -X DELETE https://api.miosa.ai/api/v1/computers/COMPUTER_ID/volumes/ATTACHMENT_ID \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

**CLI:**
```bash
miosa volume create model-weights --size-gb 50
miosa volume list
```

Attach and detach are per-computer; drive them with `miosa api`:

```bash
# Attach the volume to a running computer at a mount path
miosa api POST /computers/<computer-id>/volumes \
  -d '{"volume_id": "<volume-id>", "mount_path": "/mnt/data"}'

# Detach by attachment id
miosa api DELETE /computers/<computer-id>/volumes/<attachment-id>
```

Delete a volume (it must be detached first):

```bash
miosa volume rm <volume-id>
```

> A local-backend volume lives on one host, so it attaches to computers on that same host. Cross-host attach returns a clear error rather than failing silently.

## Mount

A volume is mounted into the runtime instance's filesystem at a path:
```typescript
await miosa.services.update(serviceId, {
  volumes: [
    { dataServiceId: volumeId, mountPath: "/var/models" },
  ],
})
```

At boot, `/var/models` is the volume's contents. Writes persist.

## Replicas and volumes

Most volumes attach to a single runtime instance. If you scale a service to N replicas, by default each replica gets the **same volume mount** if it's marked `shared: true`, or its own volume if `shared: false`.

Shared volumes have all the usual concurrency caveats (filesystem locks, file-corruption risk if multiple writers touch the same file). Use Postgres for shared state unless you really know what you're doing.

## Limits

| Limit | Default |
|---|---|
| Max volume size | 500 GB |
| Max volumes per project | 10 |
| Filesystem | ext4 |
| Host placement | Must match the computer host |

## When to use a volume

Realistic short list:

1. You're shipping a self-hosted SQLite-backed app to keep deployment simple.
2. You're running an LLM inference service and don't want to re-download model weights on every instance start.
3. You have a writable filesystem cache that's expensive to rebuild.

Anything else, use Postgres or Storage.

## See also

- [Overview](/docs/data/overview)  -  the broader data plane
- [Postgres](/docs/data/postgres)  -  for relational data (almost always the right answer)
- [Storage](/docs/data/storage)  -  for user-uploaded files
