---
slug: "regions"
title: "Regions"
description: "Which regions MIOSA runs in, what is region-local, how to pin a sandbox or computer to one, and the cross-region story."
section: "API reference"
order: 12008
---

Every Computer and every Sandbox is created in exactly one region and stays there for its lifetime. A resource does not span regions, and there is no live migration between them today.

## Available regions

| Slug | Location | City | Status |
|---|---|---|---|
| `us-west` | US West | Los Angeles, CA | Live |
| `us-east` | US East | New York, NY | Live |
| `us-mia` | US East 2 | Miami | Live |

All three are generally available and none is restricted. A Computer defaults to your workspace's default region, or `us-mia` when none is set; a Sandbox has no default region and is placed wherever capacity allows.

## Pin a resource to a region

Pass `region` on create; the same field works on Computers and Sandboxes.

**Python**

```python
computer = client.computers.create(name="ny-build-box", size="M", region="us-east")
print(computer.region)   # "us-east"
```

**TypeScript**

```typescript
const computer = await client.computers.create({ name: 'ny-build-box', size: 'M', region: 'us-east' });
```

If you omit `region`, a Computer uses your workspace's default region (falling back to `us-mia`), while a Sandbox is placed wherever capacity allows; the response includes the resolved region when one is set. You can change your workspace's default region in dashboard settings; the override applies to Computer creation that omits the field.

| Status | Error | Cause |
|---|---|---|
| 400 | `invalid_region` | Slug not in the list above |
| 503 | `REGION_UNAVAILABLE` | Transient capacity issue; retry or pick another |

## List regions

**`GET /api/v1/regions`** returns the regions available to your tenant, so a region picker in your UI does not need a hardcoded list.

```bash
curl https://api.miosa.ai/api/v1/regions \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

Each entry has `id` (the region slug), `label`, `active_nodes`, `available`, `small_slots_free`, and a `headroom` object with free slots per size and the region's total and used memory. See the [Regions API](/docs/api-reference/regions) for the full response shape.

## What is region-local

- **Compute.** The VM runs in one region. All VM sizes are available in every region for both Computers and Sandboxes.
- **Data.** Data services and volumes live in the region of the resource that uses them.
- **Latency.** The three US sites are roughly equidistant for the median US user, so compute performance is effectively identical. Pick by data residency first; otherwise pick by proximity to your users or CI runners, which lowers RTT for desktop streaming and terminal round-trips.

## Cross-region

There is no live migration between regions. To run work in more than one region, create one resource per region, and federate bring-your-own hosts with [OpenComputers](/docs/api-reference/open-computers) if you want a single logical workspace across them.

EU and APAC regions are on the roadmap and will appear in `GET /api/v1/regions` alongside the current three.

> **Note: Regions are not runner endpoints**
>
> A resource's region (`us-west`, `us-east`, `us-mia`) is separate from the opt-in runner endpoint that serves fast create and exec calls. Region is about where the VM lives; the runner endpoint is about how your request reaches MIOSA. See [regions and the runner](/docs/develop/sandboxes/sizing) for that distinction.

- [Regions API](/docs/api-reference/regions): Request and response shapes for region listing and errors.
- [Sizing and limits](/docs/develop/sandboxes/sizing): Named resource contracts and the opt-in runner fast path.
