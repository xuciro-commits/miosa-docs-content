---
slug: "integrations/external-computers"
title: "External computer providers"
description: "Which kinds of compute MIOSA can run on today, and why connecting a third-party sandbox or desktop provider is not offered."
section: "Self-hosting"
order: 15004
---

MIOSA runs your workloads on its own cloud by default, and it can also run them on hardware you own or operate yourself.
Two bring-your-own paths are supported today.
Connecting a third-party sandbox or desktop provider such as Orgo, E2B, or Daytona is **not** supported today: those adapters are not part of the product.

## Where compute can run

| Source | Status | Where it runs |
|---|---|---|
| MIOSA cloud | Available | MIOSA-managed sandboxes and computers |
| OpenComputers | Available | Hardware you own or rent, connected over an outbound WebSocket |
| Bring your own cloud | Available, by onboarding | Worker capacity inside your own AWS or Google Cloud account |
| Third-party provider adapter | Not offered | - |

Sandboxes run on Firecracker by default.
SOMA is available to every customer, and `auto` lets the platform choose.
See [Choosing a runtime](/docs/soma/choosing-a-runtime).

## Run on hardware you own

[OpenComputers](/docs/opencomputers) is MIOSA's bring-your-own-hardware path.
You run a small host agent on a machine you control, and it enrolls with MIOSA over an outbound WebSocket.
The machine then appears to your application with the same interface as a MIOSA computer, so your product drives both without branching on the host.
Start with [Connect a host](/docs/opencomputers/connect).

## Run in your own cloud account

For worker capacity inside your own AWS or Google Cloud account, MIOSA has a separate onboarding path.
See [Bring your own cloud](/docs/infrastructure/byoc/overview).

## What is not offered

MIOSA does not currently provide adapters that connect a third-party compute provider.
There is no supported way to drive an Orgo, E2B, Daytona, or similar machine through the MIOSA API today, and you cannot connect a third-party API key to run machines inside MIOSA.

> **Note: Use a supported path instead**
>
> If you need capacity outside the MIOSA cloud, use [OpenComputers](/docs/opencomputers) on hardware you control, or [Bring your own cloud](/docs/infrastructure/byoc/overview) in your own cloud account.

## See also

- [Computers overview](/docs/computers/overview)  -  the unified computer surface
- [Sandboxes](/docs/develop/sandboxes)  -  isolated, disposable code execution
- [OpenComputers](/docs/opencomputers)  -  bring your own hardware
- [Bring your own cloud](/docs/infrastructure/byoc/overview)  -  worker capacity in your own cloud account
- [Compute provider comparison](/docs/integrations/comparison)  -  how MIOSA's rates compare
