---
slug: "computers/lifecycle"
title: "Lifecycle and sizing"
description: "Computer fast startup, the desktop state machine, resource sizes, and workspace-scoped creation for per-tenant attribution."
section: "Computers"
order: 3005
---

A Computer is a long-lived machine. This page covers how fast it starts, the states it moves through, the sizes it runs on, and how to scope it to a workspace.

## Fast startup

MIOSA keeps ready desktop capacity on the managed fleet so a new Computer can be assigned instead of cold-booted from scratch.
The production acceptance test measures the complete user-visible path from creation to the first usable desktop frame.

The verified 2026-08-06 canary completed **10 of 10** warm launches and reached a browser-ready desktop in a median **740 ms**:

| Phase | Time |
|---|---:|
| Create response median | `348 ms` |
| Create request to browser-ready desktop median | **`740 ms`** |
| Create request to browser-ready desktop p90 | **`809 ms`** |
| Slowest sample | `942 ms` |

This is a dated warm-path canary, not a guarantee that every region and capacity condition will complete inside the sampled range.
See [Fast start & readiness](/docs/computers/fast-start) for the readiness contract, measurement boundary, and troubleshooting guidance.

## Lifecycle

create creates provisioning; provisioning goes to running on ready; running goes to stopped on stop, and back to running on start.

running goes to recovering on host maintenance; recovering goes to running on restored.

recovering goes to error on recovery fails.

running or stopped goes to deleted on delete.

| State | Description |
|---|---|
| `provisioning` | The VM is being claimed, restored, and prepared |
| `running` | Desktop is up, streamable, and accepting API actions |
| `stopped` | Compute is stopped and filesystem state is preserved |
| `recovering` | MIOSA is restoring a desired-running Computer after host maintenance |
| `error` | Recovery did not complete; the preserved disk or snapshot remains available for retry |
| `deleted` | Permanent deletion; not reversible |

A stopped computer is not billed for compute; its disk is billed at $0.018 per GiB-month while it is stopped. Restarting resumes from the snapshot.

During host maintenance, MIOSA preserves Computer identity, disk state, snapshots, metadata, and desired state.
Computers that were running are restored and verified before the host returns to active service.
Intentionally stopped Computers remain stopped.

If a local resume fails, MIOSA preserves the disk and snapshot instead of replacing the Computer with a blank machine.
Use `delete` only when the state should be permanently removed.

## Sizing

| Size | vCPU | RAM | Disk | Typical use |
|---|---|---|---|---|
| `xs` | 1 | 2 GB | 10 GB | Lightweight headless tasks |
| `small` | 2 | 4 GB | 20 GB | **Default** for a single agent and basic browsing |
| `medium` | 4 | 8 GB | 50 GB | Heavier desktop apps, faster page loads |
| `large` | 8 | 16 GB | 100 GB | Multi-app, large-context AI workloads |
| `xl` | 16 | 32 GB | 200 GB | Intensive parallelism, large model weights |

GPU is not productized: the public API does not currently schedule GPU-backed desktops.

> **Note: OSA agent**
>
> Computers ship with the OSA agent installed but disabled by default.
> Activate it to give your computer a desktop AI assistant that can run multi-step desktop tasks autonomously.
> OSA is controlled through documented MIOSA APIs.

## Workspace-scoped creation

Computers can be created inside a named workspace so that resources are isolated and billed separately. Pass `external_workspace_id` to attribute the computer to a tenant in your platform.

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

computer = client.computers.create(
    name="agent-1",
    template_type="miosa-desktop",
    size="small",
    external_workspace_id="acme-corp",      # your tenant identifier
    metadata={"agent_run": "run-2026-05-17"},
)
computer.start()
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const computer = await miosa.computers.create({
  name: "agent-1",
  template_type: "miosa-desktop",
  size: "small",
  externalWorkspaceId: "acme-corp",
  metadata: { agentRun: "run-2026-05-17" },
})
await computer.start()
```

`external_workspace_id` is a free-form string you control.
Use your own tenant, org, customer, or workspace identifier.
Usage is tracked per workspace in the billing dashboard so you can attribute compute costs to individual customers.
