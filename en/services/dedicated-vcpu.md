---
slug: "services/dedicated-vcpu"
title: "Dedicated vCPU"
description: "Run a service on vCPUs that are one hardware thread each and never shared: sizes, prices, and how to select one."
section: "Services & projects"
order: 1015
---

A dedicated size gives a service vCPUs it owns outright.
Each dedicated vCPU is one hardware thread of the host, pinned to your service and never shared with another workload.
Use it when you need steady CPU with no noisy neighbors.

Dedicated sizes are available on Business and above.

## Sizes and prices

Memory is 4 GiB per vCPU.

| Size | vCPU | Memory | Price per month |
|---|---|---|---|
| `dedicated-2` | 2 | 8 GiB | $153.37 |
| `dedicated-4` | 4 | 16 GiB | $306.74 |
| `dedicated-8` | 8 | 32 GiB | $613.48 |
| `dedicated-16` | 16 | 64 GiB | $1,226.95 |

- A dedicated vCPU costs $32.8725 a month ($0.04503 an hour), and memory costs the normal $10.953 per GiB-month.
- A dedicated service is billed for its full size for every second a replica runs, whether the CPU is busy or idle.
- A stopped or sleeping service is not billed for compute.
- The same vCPUs and memory on a shared size cost about 17 percent less, without the guarantee.

## Select a size

Set `shape` in the service settings.

```bash
curl -X PATCH https://api.miosa.ai/api/v1/projects/{id}/services/{sid} \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"settings": {"shape": "dedicated-4"}}'
```

- The size applies to the next deployment.
- `replicas` multiplies it: three replicas of `dedicated-4` use 12 dedicated vCPUs.
- If you also set `limits`, they must fit inside the size.
- Check `GET /api/v1/projects/{id}/services/setting-capabilities`: the `shape` entry reads `enforced` when dedicated sizes are available to place.

## Capacity

Dedicated vCPUs come from a fixed pool of 16 threads on each host that carries one.
A deployment is placed only on such a host, and it is refused with `no_capacity` when that host's pool has too few free threads.
A deployment of 16 vCPUs takes a whole host pool.
Free threads return to the pool when a replica stops.
