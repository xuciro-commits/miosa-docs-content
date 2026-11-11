---
slug: "api-reference/regions"
title: "Regions API"
description: "List available regions and pin Computers and Sandboxes to a specific region."
section: "API reference"
order: 12013
---

MIOSA Computers and Sandboxes run in geographic regions. Every Computer (and every Sandbox, since a Sandbox is a lightweight Computer flavor) is created in exactly one region and stays there for its lifetime.

Base path: `/api/v1/regions`

---

## Overview

- **3 regions are live today.** All have equal status  -  none are preview, none are restricted.
- **Default region** when you don't specify one: `us-west`.
- **All VM sizes** (XS, S, M, L, XL) are available in every region for both Computers and Sandboxes.
- **Latency** is roughly equidistant for US users  -  pick by data residency or proximity preference rather than performance.
- **No multi-region resources.** A Computer lives in exactly one region. To run workloads in multiple regions, create multiple Computers.

---

## Available Regions

| Slug             | Location          | City             | Status |
|------------------|-------------------|------------------|--------|
| `us-west`     | US West           | Los Angeles, CA  | Live   |
| `us-east`     | US East           | New York, NY     | Live   |
| `us-mia`  | US East 2        | Miami         | Live   |

The region slug is what you pass on create and what comes back on read  -  every Computer and Sandbox response includes a `region` field, e.g. `"region": "us-west"`.

---

## Specify a region on create

Pass `region` in the request body when creating a Computer or Sandbox. If omitted, the default (`us-west`) is used.

**Python**

```python
  computer = client.computers.create(
      name="ny-build-box",
      size="M",
      region="us-east",
  )
  print(computer.region)  # "us-east"
```

**TypeScript**

```typescript
  const computer = await client.computers.create({
    name: 'ny-build-box',
    size: 'M',
    region: 'us-east',
  });
  console.log(computer.region); // "us-east"
```

**curl**

```bash
  curl -X POST https://api.miosa.ai/api/v1/computers \
    -H "Authorization: Bearer $MIOSA_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "name": "ny-build-box",
      "size": "M",
      "region": "us-east"
    }'
```

The same `region` field works on `POST /api/v1/sandboxes`.

### Errors

| Status | Error                | Cause                                          |
|--------|----------------------|------------------------------------------------|
| 400    | `invalid_region`     | Slug not in the list above                     |
| 409    | `region_unavailable` | Transient capacity issue  -  retry or pick another |

---

## Default region behavior

If `region` is omitted from the create request:

- The resource resolves to your tenant's default region (`us-west` by default; change it in dashboard settings  -  that override applies to API calls that omit `region`).
- The response still includes the resolved `region` so you always know where the resource lives.

---

## List regions

**`GET /api/v1/regions`**

Returns the regions available to your tenant. Useful for region pickers in UIs.

### Response  -  `200 OK`
```json
{
  "regions": [
    {
      "id": "us-west",
      "label": "US-West (Los Angeles)",
      "active_nodes": 2,
      "available": true,
      "small_slots_free": 70,
      "headroom": {
        "xs_slots_free": 250,
        "small_slots_free": 70,
        "medium_slots_free": 14,
        "large_slots_free": 7,
        "total_memory_mb": 524288,
        "used_memory_mb": 12288
      }
    },
    {
      "id": "us-east",
      "label": "US-East (New York)",
      "active_nodes": 1,
      "available": true,
      "small_slots_free": 40,
      "headroom": { "xs_slots_free": 120, "small_slots_free": 40, "medium_slots_free": 8, "large_slots_free": 4, "total_memory_mb": 262144, "used_memory_mb": 6144 }
    },
    {
      "id": "us-mia",
      "label": "US-East (Miami)",
      "active_nodes": 1,
      "available": true,
      "small_slots_free": 30,
      "headroom": { "xs_slots_free": 90, "small_slots_free": 30, "medium_slots_free": 6, "large_slots_free": 3, "total_memory_mb": 262144, "used_memory_mb": 4096 }
    }
  ]
}
```

### Response Fields

| Field               | Type    | Description                                                        |
|---------------------|---------|--------------------------------------------------------------------|
| `id`                | string  | Stable region slug  -  pass this as `region` on create               |
| `label`             | string  | Human-readable label (safe to render in UIs)                       |
| `active_nodes`      | integer | Hosts registered in the region right now                           |
| `available`         | boolean | `true` when the region has capacity for at least one `small` sandbox |
| `small_slots_free`  | integer | Free slots at the `small` shape                                    |
| `headroom`          | object  | Free slots per size, plus total and used memory for the region     |
```bash
curl https://api.miosa.ai/api/v1/regions \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

> **Tip**
>
> The list endpoint is unauthenticated-friendly  -  you can use a short-lived ticket or a public token to populate region pickers without exposing your `msk_*` key.

---

## FAQ

### Why these 3 regions?

We picked Los Angeles, New York, and Miami to cover the US west coast, east coast, and central corridor. The three sites are roughly equidistant for the median US user, and they each sit on independent network and power footprints  -  so a regional outage at one doesn't take the others down.

### How do I pick a region?

Pick by **data residency** first. If you have a customer or compliance requirement that pins data to a coast, that decides for you.

If you have no residency requirement, pick by **proximity to your users or your CI runners**  -  shorter physical distance means lower RTT for VNC streaming and PTY round-trips. Compute performance itself is identical across regions.

### Can a Computer span multiple regions?

No. A Computer (or Sandbox) lives in exactly one region for its lifetime. To run workloads in multiple regions:

- Create one Computer per region.
- Use OpenComputers federation if you want a single logical workspace across them (see [OpenComputers](/docs/api-reference/open-computers)).

There's no live migration between regions today.

### Will more regions ship?

Yes  -  EU and APAC regions are on the roadmap. When they ship they'll appear in `GET /api/v1/regions` alongside the current three. Watch the [changelog](/docs/changelog) for announcements.

---

## See also

- [Computers API](/docs/api-reference/computers)  -  pass `region` on create
- [Sandboxes API](/docs/api-reference/sandboxes)  -  sandbox region support
- [OpenComputers](/docs/api-reference/open-computers)  -  federate BYOC hosts across regions
