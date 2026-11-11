---
slug: "faq/runtimes"
title: "SOMA and runtimes"
description: "What SOMA is, the default runtime, how to select one, access requests, and availability."
section: "Help"
order: 16005
---

Short answers on the runtime you boot a sandbox on.

## What is SOMA?

SOMA (Secure Optimized Machine Architecture) is MIOSA's own open-source Rust VMM on Linux KVM, giving each sandbox one hardware-isolated VM with its own kernel.
See [SOMA](/docs/soma).

## How do I select a runtime?

Pass `runtime_profile: "standard" | "soma" | "auto"` when creating a sandbox, from the REST API, the SDKs (`runtime`), the CLI (`miosa create --runtime`), or the console.
An invalid value returns `INVALID_SANDBOX_RUNTIME_PROFILE`; there is no silent fallback.
`GET /api/v1/sandboxes/runtimes` lists the runtimes your organization can select.
See [Choosing a runtime](/docs/soma/choosing-a-runtime).

```bash
miosa create --runtime soma --size small
```

## Is SOMA available to everyone?

Yes. SOMA is available to every customer, at `small`, `medium`, and `large`, with live vCPU and memory resize.
`GET /api/v1/sandboxes/runtimes` reports what each runtime supports today.
See [SOMA](/docs/soma).

## What is generally available and what is in preview?

Sandboxes, previews, publishing, data, white-label, SOMA, and the sandbox browser are generally available.
Computers, dynamic deployments, Docker Host, and tenant deployment domains are in partner preview; GPU-backed sandboxes are not productized and remain on the roadmap.
See [Availability and roadmap](/docs/availability).
