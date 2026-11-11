---
slug: "concepts/data-and-bindings"
title: "Data and bindings"
description: "Managed Postgres, Redis, object storage, volumes, and auth - the durable resources a Deployment attaches to, and how bindings inject credentials."
section: "Optional"
order: 17011
---

Sandboxes and Runtime Instances are replaceable.
Data Services are durable resources with an independent lifecycle.
Replacing a Sandbox, publishing a Release, or rolling back a Version does not delete the Deployment's database or object storage.

| Service | What it is |
|---------|-----------|
| Managed Postgres | Fully managed PostgreSQL database. Schema migrations are yours to run; MIOSA provides the connection string. |
| Managed Redis | Persistent Redis cluster for caching, queues, pub/sub. |
| Object Storage | S3-compatible bucket. Useful for uploads, assets, generated files. |
| Volumes | Block storage mounted into Runtime Instances. Persists across restarts. |
| Auth | JWT-based signup/login as a service. One env var gives your project a full auth system. |

Bindings connect a Sandbox or Deployment to those durable resources.
MIOSA resolves the binding and injects the appropriate credentials at runtime.

```bash
DATABASE_URL=postgresql://user:pass@host/dbname
REDIS_URL=redis://host:6379
STORAGE_BUCKET=miosa-ws123-uploads
AUTH_URL=https://auth.miosa.app/proj_abc
AUTH_JWT_SECRET          # injected by MIOSA for this project
```

Destroying a Sandbox does not destroy its database.
Publishing a Version does not automatically make an incompatible schema safe.
Use backward-compatible expand, migrate, and contract changes so the active and previous Versions can both operate during promotion or rollback.

> **Note: Phase status**
>
> Managed Postgres and Auth are part of the platform Phase 6-8 rollout. The doc surface is published now so you can plan integration; some endpoints are still being wired in production.

```bash
miosa api POST /databases -d '{"name":"app-production","engine":"postgresql"}'
```

[Data overview →](/docs/data/overview)

## Next

- [The publish pipeline](/docs/concepts/publish-pipeline): how a Release attaches to these resources.
- [Data overview](/docs/data/overview): the full data-services reference.
- [Core Concepts](/docs/concepts): the one-screen overview.
