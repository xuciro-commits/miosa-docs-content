---
slug: "api-reference/cloud"
title: "Cloud and BYOC API"
description: "Organization-scoped AWS and GCP accounts, regions, pools, nodes, preflight, and host provisioning."
section: "API reference"
order: 12032
---

Base path: `/api/v1/cloud`

All resources belong to the active organization, which is returned as both `organization_id` and the compatibility field `tenant_id`.
Cloud reads require `cloud:read` and mutations require `cloud:write`.
Organization members can read inventory.
Organization owners and admins can mutate it.
Workspace-scoped credentials cannot mutate organization cloud capacity.

AWS and GCP BYOC are gated private previews.
Provider connection and preflight do not make workload placement selectable.
Each exact organization, region, provider, and workload combination needs a current verification receipt from a real accepted host.

## Accounts

- `GET /cloud/accounts` lists organization cloud accounts.
- `POST /cloud/accounts` creates an AWS or GCP customer BYOC trust boundary.
- `POST /cloud/accounts/{id}/aws-role` attaches an AWS AssumeRole ARN and generated external ID.
- `POST /cloud/accounts/{id}/gcp-workload-identity` attaches the GCP project, customer actuator service account, and separate least-privilege worker service account without storing a service-account key.

## Regions

- `GET /cloud/regions?cloud_account_id={id}` lists configured regions.
- `POST /cloud/regions` records provider region, zone, network, host image, and exact immutable artifact configuration.
- `PATCH /cloud/regions/{id}` updates that configuration and clears stale preflight readiness.

Every launchable region must provide an HTTPS manifest URI, lowercase SHA-256 digest, version bound into the URI, and positive artifact generation.

## Pools

- `GET /cloud/pools?cloud_region_id={id}` lists desired host pools.
- `POST /cloud/pools` creates a pool configuration.
- `PATCH /cloud/pools/{id}` updates organization capacity and placement guardrails.
- `POST /cloud/pools/{pool_id}/provision` explicitly launches between 1 and 50 hosts.

```json
{
  "count": 1
}
```

Provisioning returns one result per requested host.
A successful launch starts in `provider_pending` and is not schedulable until host acceptance passes.

## Nodes

- `GET /cloud/nodes?cloud_pool_id={id}` lists provider and acceptance state for a pool.

Important node states include `provider_pending`, `acceptance_running`, `active`, `draining`, `terminated`, and `failed`.
Only `active` nodes with a durable organization cell and accepted host session are eligible for placement.

## Preflight

- `GET /cloud/preflights` lists account and region preflight evidence.
- `POST /cloud/preflights` runs server-side account or region verification and records immutable evidence.

Preflight and host acceptance are different gates.
Preflight validates AWS or GCP identity, real provider API access, zone and machine availability, configured network and image references, and the immutable artifact contract before launch.
Host acceptance validates the actual launched machine and promoted MIOSA runtime before placement.
Sandbox, Computer, and Deployment placement are certified separately because one working runtime does not prove another workload contract.

## SDK

```typescript
const result = await miosa.cloud.provisionPool(poolId, 1);
const nodes = await miosa.cloud.listNodes({ cloudPoolId: poolId });
```

```python
result = client.cloud.provision_pool(pool_id, count=1)
nodes = client.cloud.list_nodes(cloud_pool_id=pool_id)
```

Sandbox creation does not take a pool ID.
An active BYOC organization is automatically and exclusively placed into its configured scheduling cell.
