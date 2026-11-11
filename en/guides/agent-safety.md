---
slug: "guides/agent-safety"
title: "Agent safety and status"
description: "How agents interpret deployment health, preserve durable resources, and respect current capability boundaries."
section: "Agents"
order: 6003
---

An agent must not treat a single signal as proof that production is healthy, and must not replace durable resources on a guess.
This page covers status interpretation, durable-resource handling, and the current capability boundaries.

## Interpret status correctly

> **Caution: Healthy is a composite result**
>
> A deployment is not proven healthy from one database row, one HTTP response, or one host heartbeat.
> The active release, running artifact, public route, data bindings, required capabilities, and runtime placement must agree.

| Signal | What it proves |
|---|---|
| Release status is `ready` | The immutable artifact can be considered for promotion |
| Build status is `succeeded` | The build step completed for this release |
| `active_release_id` matches | The control plane selected the expected release |
| The public route responds | Users can reach the deployed application surface |
| Runtime logs show healthy startup | The process bound its port and is serving |

## Preserve durable resources

A sandbox is a mutable development workspace.

A deployment is the durable production identity.

A managed database and its volume are durable data resources.

An Docker Host host can run multiple deployments and should not be replaced for an application-level failure without host-level evidence.

Pause, resume, recover, and reconcile existing resources before creating replacements.

Create a replacement only when the original resource is proven terminal and its state, bindings, and ownership have been preserved.

## Capability boundaries

MIOSA currently supports sandboxes, computers, previews, immutable application releases, managed data, workspace attribution, domains, integrations, and OpenComputers.

Specialized model APIs can be connected today.

CPU-compatible open-source applications can be self-hosted when their runtime requirements fit the selected compute.

Managed GPU execution and built-in managed agent orchestration remain planned until the availability page marks them shipped.

AWS and Google Cloud customer-region packages exist, but production activation remains gated on provider canary and host-acceptance evidence.

## See also

- [Agent CLI workflow](/docs/guides/agent-cli-workflow): Discovery, scope, sandbox development, publish, promote, and rollback.
- [Agent operating guide](/docs/guides/agent-operating-guide): The three-step entry path for agents.
- [Availability](/docs/availability): Check whether a capability is available, integrated, self-hostable, gated, or planned.
