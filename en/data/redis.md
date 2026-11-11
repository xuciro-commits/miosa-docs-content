---
slug: "data/redis"
title: "Redis / Cache"
description: "Managed Redis per project for caching, sessions, and pub/sub."
section: "Databases & storage"
order: 4003
---

> **Tip: Available**
>
> Managed Redis is live.
> Per-project instances and `REDIS_URL` injection are available.
> Redis is provisioned or attached explicitly and should not be assumed to exist in every sandbox template.

## What you get

Per project: a managed Redis instance, with credentials injected as `REDIS_URL`:
```bash

REDIS_URL=redis://default:pass@host:port/0
```

Standard Redis clients work without modification:
```typescript
import { createClient } from "redis"
const redis = createClient({ url: process.env.REDIS_URL })
await redis.connect()

await redis.set("user:123:last-seen", new Date().toISOString())
const value = await redis.get("user:123:last-seen")
```

## Use cases

- **Cache**  -  short-lived computed results.
- **Sessions**  -  express-session / Phoenix sessions / similar.
- **Pub/sub**  -  agent coordination, real-time fan-out.
- **Rate limiting**  -  atomic INCR + TTL.
- **Job queues**  -  Bull, BullMQ, Sidekiq, Oban-light alternatives.

For full job-queue workloads at scale, MIOSA has a [Background Jobs](/docs/docker-host/runtime-instances) primitive that's a better fit than rolling your own on Redis.

## Provision
```typescript
await miosa.projects.dataServices.create(projectId, {
  type: "redis",
  size: "small",  // small | medium | large
})
```

## Limits and durability

Memory, connection, and persistence settings depend on the returned managed-database plan.
Inspect the database response and metrics for the selected instance.
Treat Redis as cache-class unless the selected plan explicitly guarantees the durability required by your application.

## Pub/sub considerations

If you're using Redis pub/sub for cross-runtime coordination, multiple runtime instances of the same service can all subscribe  -  MIOSA does not deduplicate or proxy pub/sub. For higher-volume real-time fan-out, consider SSE through MIOSA's [Events](/docs/api-reference/events) or a dedicated message broker.

## Bring your own Redis?

Yes. Set `REDIS_URL` as an env var to whatever external Redis you prefer (Upstash, AWS ElastiCache, your own).

## See also

- [Overview](/docs/data/overview)  -  the broader data plane
- [Runtime Instances](/docs/docker-host/runtime-instances)  -  for background-job workloads
