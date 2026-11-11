---
slug: "compare/fly"
title: "MIOSA vs Fly.io"
description: "A factual comparison of MIOSA and Fly.io for running apps and machines, using Fly.io's public documentation as of 2026-10-23."
section: "Optional"
order: 17016
---

Compare

Fly.io runs apps as machines close to your users. MIOSA runs services, computers and sandboxes on one platform. This page compares what each documents.

Every competitor statement is from Fly.io's own public documentation ([fly.io/docs](https://fly.io/docs), as of 2026-10-23). MIOSA statements link to the page that documents them. Where MIOSA has no equivalent, the row says so.

## Workloads

| Capability | MIOSA | Fly.io |
|---|---|---|
| Long-running app from an image or repository | Yes, Services run modes `server`, `worker`, `cron`, `static` | Yes, apps on Machines |
| Direct control of a VM | Yes, Computers with a full Linux desktop | Yes, Machines |
| Ephemeral code execution | Yes, Sandboxes | Not the documented model; Machines are long-lived by default |
| Config as code | `miosa.json` / `miosa.app.yml` | `fly.toml` |
| Multiple regions | `us-east`, `us-west`, `us-mia` | A wider published region list |

Sources: [services](/docs/services/overview), [computers](/docs/computers/overview), [sandboxes](/docs/develop/sandboxes), and Fly.io's [Machines](https://fly.io/docs/machines/) and [fly.toml reference](https://fly.io/docs/reference/configuration/).

## Networking and data

| Capability | MIOSA | Fly.io |
|---|---|---|
| Private networking between a project's services | Same host today; wider private networking is not generally available | Documented private networking across a region |
| Public address per service | Yes, `<label>.<org>.miosa.app`, plus custom domains | Yes, `*.fly.dev` plus certificates |
| Managed Postgres, MySQL, Redis, MongoDB | Yes | Managed Postgres is documented; other engines are run by you |
| Public database endpoint with an allowlist | Yes, TLS only | Connections are documented through your own proxy or app |

Sources: [custom domains](/docs/docker-host/domains), [databases](/docs/data/overview), [public endpoint](/docs/data/public-endpoint).

## What MIOSA does not have

- **Fly.io's region coverage.** Three regions, named above; nothing else is claimed.
- **Cross-host private networking between services.** Same-host private networking works today.

## When to choose each

- **Choose Fly.io** if you need its region footprint or you already run your networking around it.
- **Choose MIOSA** if your product also needs sandboxes and agent workloads, a console for customers of your own, or one platform for machines, services and data.

> **Note: Check before you decide**
>
> Both products change. The links above are what this page was written from on 2026-10-23.
