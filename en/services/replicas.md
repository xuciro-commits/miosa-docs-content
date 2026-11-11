---
slug: "services/replicas"
title: "Services replicas and scaling"
description: "Set how many replicas a service runs, where, and how large they are, and what a scale change does to the running release."
section: "Services & projects"
order: 1014
---

A replica is one running copy of a service.
A service runs from 1 to 50 replicas.

## Change replicas, region and size

```bash
miosa service update shop API --replicas 4
miosa service scale  shop API --replicas 4
```

`update` changes the setting.
`scale` changes replicas, region or limits and also queues a release that applies them.
The running release is not touched until that release is active, so a scale change never interrupts traffic part way.

| Setting | Meaning |
|---|---|
| `replicas` | 1 to 50. A writable volume forces 1. |
| `region` | `us-west`, `us-east` or `us-mia`. |
| `limits` | `{vcpu, memory_mb}`. Above 4 vCPU or 8192 MB the release fails. |
| `restart_policy` | `always`, `on_failure` or `never`. |
| `healthcheck_path` | The path a web service must answer before a new release becomes active. |

## What a release does to replicas

A new release starts its replicas first.
Once they pass the health check, the release becomes `active` and the earlier release drains and stops.
`overlap_s` and `terminate_grace_s` add time before the earlier release stops.

## Volumes

A volume is attached to a service, never added on its own.
A writable volume allows one replica.
Mount it read-only if you need more.
A service can mount up to four volumes, and a path or a volume cannot repeat.

## Check what takes effect

```bash
miosa service settings shop
```

Each setting is `enforced`, `stored` or `unsupported`.
A `stored` setting is kept but not applied yet, so do not rely on it.

## Where to go next

- [Run modes and types](/docs/services/services): every setting and which types it applies to.
- [Releases and rollback](/docs/services/releases): how a scale change becomes a release.
- [Logs](/docs/services/logs): what each replica prints.
