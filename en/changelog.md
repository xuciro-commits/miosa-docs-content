---
slug: "changelog"
title: "Changelog"
description: "Customer-facing updates to MIOSA products, platform capabilities, SDKs, CLI tools, and documentation."
section: "Optional"
order: 17028
---

This page highlights meaningful improvements after they are available to customers.

Entries focus on the product result, affected workflow, and any action customers should take.

Internal infrastructure details, private identifiers, security-sensitive implementation notes, and routine maintenance are intentionally excluded.

Planned work belongs in [Availability & Roadmap](/docs/availability), not in the changelog.

> **Note: No action usually required**
>
> Unless an entry includes an action notice, existing projects and integrations continue working without changes.

---

## Rolling out

In progress

Merged and rolling out. Each bullet moves under its date once it is live.

- Flat pricing: one flat hourly price per machine size for sandboxes and computers, with apps and databases billed on use up to that size's price.
- A deployment guest is isolated per customer rather than by the host session.
- A sandbox copy cold boots the scrubbed disk when memory is withheld.
- Docker Engine 29.9 with the `docker compose` and `docker buildx` plugins in the default `miosa-sandbox` template and the templates built on it.
- SOMA at `small`, `medium` and `large`, with live vCPU and memory resize.
- SSH certificates scoped to a single sandbox, alongside the existing authorized keys.

## October 2026

### October 18, 2026

New

- Deployments: a Next.js standalone build ships and runs as built - only the top-level dependency folder is stripped from the artifact, and the standalone server is started directly, so an app whose build output carries its own dependencies runs in a guest with no package-registry access.
- Deployments: `PORT` is set for the app unless the app sets its own.

Improved

- Deployments on managed hosts: a release starts, stays up and serves its app on the port it listens on, and readiness is checked inside the host against that port, so an app that reported listening is no longer left unreported.
- Deployments: a managed instance records the shape its guest actually got.

Fixed

- Deployments on managed hosts: the platform's own sweepers no longer kill a deployment's guest while it is still starting, and a guest survives a restart of the host agent.
- Deployments on managed hosts: a guest is published at its host's mesh address, so a deployment's hostname reaches the app.
- Deployments: a release that cannot install, cannot launch, or never answers on its port fails the launch with a named reason - `release_install_failed`, `app_launch_failed` or `app_not_listening` - instead of leaving a guest that cannot work.
- Deployments: a Next.js release that produced no application build is refused with `build_output_missing` naming the missing output, instead of a guest that restarts forever.
- Databases: backup retention restamps completed backups only, and only when the plan's date is at least a minute later, so a completed backup keeps its full window.
