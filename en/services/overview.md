---
slug: "services/overview"
title: "Services"
description: "Run web services, workers, private services and static sites in a project, with releases, variables, domains and rollback."
section: "Services & projects"
order: 1004
---

Services are the apps a project runs.
A service is one of four common types, plus cron jobs and database services:

| Type | What it is |
|---|---|
| Web service | A process that serves HTTP on a public address. |
| Worker | A process with no port and no address. It runs until it stops. |
| Private service | A process with no public address. Private networking between services is not available yet, so nothing else reaches it by name; it can reach managed databases through their connection variables. |
| Static site | Pre-built files served from the edge. |
| Cron job | A command on a five-field cron schedule, with run history and output. |
| Database service | A managed Postgres or Redis bound to the project. |

Every service belongs to a project and an environment.
A project starts with a `production` environment; add `staging` or anything else you need.

## What a service has

- **Releases.** Every deploy is an immutable release with a number, a state and a reason when it fails.
- **Variables.** Sealed at rest, masked on read, and able to reference another service, a shared variable or a generated secret.
- **An address.** Every service is assigned a private name. A web service or static site can also have a public address.
- **Settings.** Replicas, region, start command, health check, restart policy, volumes and idle sleep.

## The lifecycle

1. Create the service with a source: a git repository, a container image or an uploaded archive.
2. Deploy it. MIOSA builds it and starts a release.
3. The release becomes `active` when its replicas pass their health checks.
4. Deploy again to change it, or roll back to an earlier release. Rollback serves the earlier built image, so nothing is rebuilt.

## Where to go next

- [Quick start](/docs/services/quickstart): a repository to a live URL, then Postgres, a worker, a domain and a rollback.
- [Services and types](/docs/services/services): the source, settings and address of each type.
- [Variables](/docs/services/variables): references between services and generated secrets.
- [Releases](/docs/services/releases): deploy, promote, roll back and cancel.
- [CLI](/docs/services/cli) and [SDKs](/docs/services/sdk): the same operations from a terminal or code.

## Migrating from Railway

The model maps directly:

| Railway | MIOSA |
|---|---|
| Project | Project |
| Service | Service (web service, worker, private service or static site) |
| Environment | Environment |
| Deployment | Release |
| Variables and shared variables | Variables and shared variables |
| Custom domain | Public address |

`miosa service` operates Services.
Services use `/projects/:id/services` and `/services/:id`; the separate Deploy product keeps its own `/deployments` routes.
