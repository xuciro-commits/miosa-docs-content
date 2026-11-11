---
slug: "soma/choosing-a-runtime"
title: "Choosing a runtime"
description: "Choose Firecracker, SOMA or auto when you create a sandbox with the runtime option. List the runtimes your organization offers and see what each runtime reports it supports today."
section: "Sandboxes"
order: 2022
---

A sandbox runs on one of two engines: **Firecracker**, MIOSA's long-standing default, or **SOMA**, MIOSA's own open-source VMM.
Select one with the `runtime` option, which travels on the wire as the `runtime_profile` field.

> **Note: The default is Firecracker**
>
> If you never select a runtime, nothing changes. You get Firecracker (`standard`).
> Organization and workspace runtime settings, with SOMA as the default, are coming.
> SOMA is available to every customer.

## The runtimes

| Runtime | `runtime_profile` | What it is |
|---|---|---|
| Firecracker | `standard` | Full machine. The default for every size and template. |
| SOMA | `soma` | Fastest start for supported sandbox images, offered at `small` (2 vCPU / 4 GiB), `medium` (4 / 8 GiB), and `large` (8 / 16 GiB), with live vCPU and memory resize up and down. |
| Auto | `auto` | Let the platform pick the best runtime for the request. |

`runtime` is the name the SDKs and the CLI use; the REST field is `runtime_profile`. The mapping is `firecracker` to `standard`, `soma` to `soma`, and `auto` to `auto`.
Omitting the field is treated exactly as `standard`. A value outside those three is refused with `422 INVALID_SANDBOX_RUNTIME_PROFILE`, and the error names the allowed set.

## List the runtimes your organization offers

`GET /api/v1/sandboxes/runtimes` reports the runtimes available to your organization and what each runtime reports it supports today.

```http
GET /api/v1/sandboxes/runtimes
Authorization: Bearer msk_u_...
```

```json
{
  "data": [
    {
      "id": "firecracker",
      "runtime_profile": "standard",
      "label": "Firecracker",
      "description": "Full machine. The default for every size and template.",
      "default": true,
      "available": true,
      "unavailable_reason": null,
      "capabilities": {
        "browser": true,
        "network_egress": true,
        "docker": true,
        "persistence": true,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 16, "memory_mb": 32768, "disk_size_mb": 163840 }
      }
    },
    {
      "id": "soma",
      "runtime_profile": "soma",
      "label": "SOMA",
      "description": "Fastest start for supported sandbox images.",
      "default": false,
      "available": false,
      "unavailable_reason": "GLOBAL_GATE_CLOSED",
      "capabilities": {
        "browser": false,
        "network_egress": false,
        "persistence": false,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 8, "memory_mb": 16384, "disk_size_mb": 40960 }
      }
    },
    {
      "id": "auto",
      "runtime_profile": "auto",
      "label": "Auto",
      "description": "Let the platform pick the best runtime for the request.",
      "default": false,
      "available": true,
      "unavailable_reason": null,
      "capabilities": {
        "browser": true,
        "network_egress": true,
        "docker": true,
        "persistence": true,
        "pause": true,
        "gpu": false,
        "max_shape": { "cpu_count": 16, "memory_mb": 32768, "disk_size_mb": 163840 }
      }
    }
  ]
}
```

Read `available` before you offer a runtime. When it is `false`, `unavailable_reason` says why:

- `GLOBAL_GATE_CLOSED` - the platform has not enabled SOMA for customer traffic yet.
- `ORG_RESTRICTED` - SOMA is turned off for your organization by policy.
- `NOT_CONFIGURED` - SOMA is switched on but has no default capacity configured yet.

The `capabilities` flags state what a runtime does today. They come from this endpoint, so read the entry for the runtime you plan to use instead of inferring a capability from its name.

## Choose a runtime on create

**REST**

```bash
# List the runtimes your organization can select
curl https://api.miosa.ai/api/v1/sandboxes/runtimes \
  -H "Authorization: Bearer $MIOSA_API_KEY"

# Create a SOMA sandbox
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "runtime_profile": "soma", "size": "small", "template_id": "miosa-sandbox" }'
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const runtimes = await miosa.runtimes.list();
const sandbox = await miosa.sandboxes.create({ runtime: "soma", size: "small" });
```

**Python**

```python
import os
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])

runtimes = client.runtimes.list()
sandbox = client.sandboxes.create(runtime="soma", size="small")
```

**CLI**

```bash
miosa runtimes list
miosa create --runtime soma --size small
```

`miosa runtimes list` prints each runtime with its availability and reason, so you can see whether SOMA is offered to your organization before selecting it.

If a runtime cannot serve the requested shape, create is refused with `422 SOMA_RUNTIME_UNSUPPORTED_REQUEST`, naming the runtime's maximum shape, before any machine is admitted.
A sandbox response carries `runtime`: `"firecracker"` or `"soma"`, the engine it actually runs on, so an `auto` request that fell back reports `"firecracker"`.

## What happens when you ask for SOMA

`soma` and `auto` are checked against a trusted runtime policy for your organization before any host work starts.
If the request cannot be honoured, create fails closed with a typed error rather than silently running on the other engine. There is never a silent fallback.

Once your organization is enabled, a SOMA create needs two things to succeed:

1. **An enabled runtime policy** for your organization that carries the SOMA requirements.
2. **Qualified capacity.** A host profile that is approved, has a fresh host-truth observation, and holds a certified Generation for the requested shape.

### Select it

Create sandboxes with `runtime_profile: "soma"`, or `"auto"` to let the platform choose.
Read `available` from `GET /api/v1/sandboxes/runtimes` first; when it is `false`, `unavailable_reason` says why, and a shape with no certified Generation for your request is refused before any machine is admitted.

> **Note: Not the same as the SOMA one-hop runner**
>
> Do not confuse this with the **SOMA one-hop runner**, which the SDKs expose through `config.runner.enabled` (or the `MIOSA_RUNNER_MODE=1` environment variable).
> That option is a transport fast path, not a runtime selector: it sends `create`, `exec`, and `destroy` straight to the runner fleet instead of through the control plane first. It says nothing about whether the sandbox runs on SOMA or Firecracker, and every other client path is unaffected either way.
> The [Sizing and limits](/docs/develop/sandboxes/sizing) page documents that runner path in full, including its request shape and its `Soma-Runner-Url` header.

## Feature parity and gaps

> **Caution: Know the gaps before you switch**
>
> Pick SOMA for a workload once you have confirmed it does not depend on a gap below.

What is the same on both engines:

- The sandbox API contract, response shapes, and error codes.
- Named resource contracts. SOMA serves `small`, `medium`, and `large`; Firecracker serves every size, `xs` through `xl`.
- `exec`, streaming output, and the file API: read, write, list, mkdir, and remove.
- Create, extend, and destroy. Pause, persistence, snapshots, and fork are Firecracker only for now.
- Pricing and billing, which accrue per second at the price of the size regardless of engine.

What to confirm before you switch:

- **No live browser viewer yet.** A SOMA sandbox ships Docker and Chromium, but not the virtual display the live viewer renders, so the live viewer is not available on SOMA yet.
- **No pause, snapshot, or fork yet.** A SOMA sandbox cannot be paused, snapshotted, or forked.
  These arrive in their own release.
- **Long commands on SOMA.** The executable and each argument are limited to 4096 bytes, and a longer shell command fails today with a `502 AGENT_UNAVAILABLE` that looks retryable.
  Send it as a file, or on standard input, instead.

- **SOMA sandboxes are not persistent yet.** State does not survive a stop or a timeout, so a SOMA create defaults to `persistent: false`, and `persistent: true` with SOMA is refused with a `422`. A disposable SOMA sandbox whose timeout expires is deleted. Keep durable state in a data service, or use Firecracker when you need a persistent sandbox.
- **Only sandboxes take a runtime.** There is no runtime choice for desktop Computers, which run on Firecracker.
- **Check the capability flags.** Read the runtime's `capabilities` from `GET /api/v1/sandboxes/runtimes` and confirm the features your workload needs are enabled before you switch.
- **The shape ceiling is real.** A request above a runtime's `max_shape` is refused with `422 SOMA_RUNTIME_UNSUPPORTED_REQUEST`, so stay within the advertised shape.
- **SOMA capacity is limited.** A SOMA create needs a certified Generation for the exact shape you asked for, on a host that is approved and freshly observed.
- **SOMA is alpha software.** Its own repository labels the engine `1.0.0-alpha.1` and keeps a claim ledger of what is proven and what is still only designed.

## See also

- [SOMA](/docs/soma) for what the engine is and how it works.
- [Benchmarks](/docs/benchmarks) for MIOSA's independent burst numbers.
- [Sizing and limits](/docs/develop/sandboxes/sizing) for resource contracts and the one-hop runner.
