---
slug: "develop/sandboxes"
title: "Sandboxes"
description: "Fast, isolated Linux VMs for code execution, builds, tests, and dev servers, with live preview URLs and snapshots."
section: "Sandboxes"
order: 2000
---

A **Sandbox** is MIOSA's fast code execution environment.
Your agent writes files into it, executes commands, exposes ports as live preview URLs, and snapshots state for branching or rollback.
Use it for build loops, tests, scripts, package installs, code generation, preview servers, and short-lived automation.

Sandboxes are persistent by default, which means they survive a stop.
When you stop one or its timeout expires, MIOSA preserves the disk and moves the sandbox to `stopped` so it can resume later.
Running processes do not survive a stop.
Persistent does not mean the sandbox survives the loss of its host; see [Sandbox lifecycle](/docs/develop/sandboxes/lifecycle).
SOMA does not support persistence yet.
Destroy only when the saved state should be removed.

## 1. Create a sandbox

**CLI**

```bash
miosa create my-box --wait     # provision a sandbox and wait until it is ready
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create()
console.log(sandbox.id, sandbox.state)
// sbx_a1b2c3d4e5  running
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create()
print(sandbox.id, sandbox.state)
# sbx_a1b2c3d4e5  running
```

## 2. Run a command

**CLI**

```bash
miosa exec my-box -- echo "hello from MIOSA"
```

**TypeScript**

```typescript
const result = await sandbox.exec("echo hello from MIOSA")
console.log(result.stdout)
// hello from MIOSA
```

**Python**

```python
result = sandbox.exec("echo hello from MIOSA")
print(result.stdout)
# hello from MIOSA
```

## 3. Write a file and open a preview

**CLI**

```bash
echo "<h1>Hello</h1>" | miosa files write my-box:/workspace/index.html
miosa preview create my-box 3000 --visibility public
```

**TypeScript**

```typescript
await sandbox.files.write("/workspace/index.html", "<h1>Hello</h1>")
const preview = await sandbox.previews.create(3000)
console.log(preview.url)
// https://3000-abc12345.sandbox.miosa.ai
```

**Python**

```python
sandbox.files.write("/workspace/index.html", "<h1>Hello</h1>")
preview = sandbox.previews.create(port=3000)
print(preview.url)
# https://5173-abc12345.sandbox.miosa.ai
```

Open the preview URL MIOSA returns. Only the port you exposed is reachable, and only through the MIOSA proxy.

> **Note: The rule**
>
> Use a Sandbox for code and processes.
> Use a [Computer](/docs/computers/overview) for a visible GUI, browser login, or human takeover.
> Use [Docker Host](/docs/docker-host/docker-deploy) when the result should be a durable 24/7 deployment URL with custom domains.
> Every sandbox boots the same main image, which ships Docker with the `docker compose` and `docker buildx` plugins, and a live browser with CDP on a virtual display (`Xvfb :99`), so you can build and run containers, and drive a browser, while you develop.
> A SOMA sandbox does not have that display yet, so the live viewer is not available there.

## Image, template, and runtime

Three different things decide what a sandbox boots with, and they are not the same choice.

| | What it is | Where you set it |
|---|---|---|
| **Image** | The one main image, with Node, Python, Docker Engine 29.9 with Compose and Buildx, a live browser with CDP, and the code interpreter already inside. | Nothing to choose: every sandbox boots it. |
| **Template** | Optional content staged inside that image at create time: files, a scaffold, packages, and the start command. | `template_id`, for example `nextjs`, `static-html`, or a custom template. |
| **Runtime** | Which virtual machine monitor runs the sandbox. | `runtime`: `firecracker` (the default), `soma`, or `auto`. |

A template is never a separate image.
An older image id passed as `template_id` still works and boots the main image; see the [Sandbox Templates API](/docs/api-reference/sandbox-templates).
See [Templates](/docs/develop/templates) for the app starters, and [Choosing a runtime](/docs/soma/choosing-a-runtime) for the runtime option.

## Pick what you need

| Need | Product | Result |
|---|---|---|
| Run commands, edit files, install packages, test code | **Sandbox** | Isolated runtime with exec, files, snapshots, and preview URLs |
| Use a browser login, desktop app, screenshots, mouse, keyboard | **Computer** | Durable GUI machine with streaming and desktop control |
| Publish a 24/7 app for users | **Docker Host** | Always-on deployment URL with custom domain support |

Sandboxes are excellent for building and previewing.
They are not the final production hosting layer.
When a preview becomes a user-facing app, publish it through Docker Host so it gets the right routing, domains, lifecycle, and billing behavior.

## Next

- [Create and connect](/docs/develop/sandboxes/create): Fused run, full create, context managers, and connecting to an existing sandbox.
- [Lifecycle](/docs/develop/sandboxes/lifecycle): The state machine, what each state means, and how to pause and resume.
- [Sizing and limits](/docs/develop/sandboxes/sizing): Named resource contracts, billing notes, and the opt-in runner fast path.
- [Timeouts and auto-stop](/docs/develop/sandboxes/timeouts): Default TTL, extending it, and how usage is measured.
- [Sandbox events](/docs/develop/sandboxes/events): Subscribe to lifecycle and activity events over SSE.
- [Production checklist](/docs/develop/sandboxes/production-checklist): The defaults to set before relying on sandboxes in production.
- [Environment variables](/docs/develop/sandboxes/environments): Set and read env vars at create time, and where secrets should live instead.
- [Capabilities](/docs/develop/sandboxes/capabilities): What sandboxes are for, what one is, and the core capabilities.

- [Docker in sandboxes](/docs/develop/sandboxes/docker): Docker Engine, Compose, and Buildx inside the sandbox.
- [Connecting to a running sandbox](/docs/develop/sandboxes/connect): SDK, CLI, SSH over a tunnel, and preview URLs.
- [Internet access and egress](/docs/develop/sandboxes/internet-access): Default network behavior and allowlists.
- [Long-running tasks](/docs/develop/sandboxes/long-running-tasks): Background processes and always-on sandboxes.
- [Metadata and labels](/docs/develop/sandboxes/metadata): Attach and read arbitrary metadata.

## Related

- [Files & Exec](/docs/develop/files-and-exec): Full filesystem and process API: recursive operations, watch, detached jobs.
- [Previews](/docs/develop/previews): Live HTTPS URLs proxied to a sandbox port with share tokens and visibility controls.
- [Templates](/docs/develop/templates): Built-in images and custom BuildSpec reference.
- [Snapshots & persistence](/docs/develop/snapshots): Pause, resume, fork, and the current public snapshot boundary.
