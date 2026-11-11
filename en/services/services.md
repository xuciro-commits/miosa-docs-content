---
slug: "services/services"
title: "Services and types"
description: "The source, settings, address and limits of a web service, worker, private service, static site, cron job and database service."
section: "Services & projects"
order: 1006
---

A service's `kind` is what the API stores.
It is one of `server`, `worker`, `cron`, `static` or `database`.
The product types map onto it:

| Product type | `kind` | Public address |
|---|---|---|
| Web service | `server` | Yes, when enabled |
| Private service | `server` | No |
| Worker | `worker` | No |
| Static site | `static` | Yes, when enabled |
| Cron job | `cron` | No |
| Database service | `database` | No (use `connection`) |

There is no `web` or `private` kind.
The CLI and both SDKs take `type: web` or `type: private`, and store `kind: server`.
A private service is a server service with no public address.

## Source

A service builds from one source.

| `source.type` | What it needs | Notes |
|---|---|---|
| `git` | `ref` (repository URL), optional `branch` and `root_dir` | Built on a Services build cell |
| `image` | `ref` (OCI image), optional `port`, optional `registry_secret` | The image is run as-is |
| `upload` | `upload_id` | PUT a `.tar.gz`, `.tar.zst`, `.tar` or `.zip` archive first |
| `forge` | `ref` | A repository hosted by MIOSA Forge |

Dockerfile builds run on the Services build cells and are detected in the source.
While the build cells are off, a Dockerfile service does not build; use a git build or an image source.

Uploading a directory is one command:

```bash
miosa service deploy shop API --dir . --wait
```

The CLI packs the directory (skipping `.git`, `node_modules` and `.env*`), records the archive with `POST /uploads`, PUTs it to the returned URL and deploys `{source: {type: "upload", upload_id}}`.
Archives are single use and expire after an hour.

## Settings

Settings are validated strictly: an unknown key is refused, so a typo never silently stays.

| Setting | Applies to | Meaning |
|---|---|---|
| `start_command` | all | Replaces the image entrypoint when a replica starts. |
| `pre_deploy_command` | all | Runs once in the new replica before the app starts. A non-zero exit fails the release. |
| `healthcheck_path` | `server` | The path probed before a new release becomes active. |
| `healthcheck_timeout_s` | `server` | How long a release may take to pass. |
| `restart_policy` | all | `always`, `on_failure` or `never`. |
| `max_retries` | all | Restarts before an `on_failure` replica is left stopped. |
| `replicas` | all | 1 to 50. A writable volume forces 1. |
| `region` | all | `us-west`, `us-east` or `us-mia`. |
| `limits` | all | `{vcpu, memory_mb}`. Above 4 vCPU or 8192 MB the release fails. |
| `sleep` | services with a public address | `{enabled, idle_s}`. The next request wakes the service. |
| `cron_schedule` | `cron` | Required. Five-field cron, UTC. |
| `overlap_s` | all | Extra time before the previous release is stopped. |
| `terminate_grace_s` | all | Extra time before the previous release is stopped, like `overlap_s`. |
| `volumes` | `server`, `worker` | Up to four `{volume, mount_path, read_only}`. |

Read what each setting actually does on your platform:

```bash
miosa service settings shop
```

Each entry is `enforced` (it takes effect), `stored` (kept but not applied yet) or `unsupported`.
Never present a `stored` setting as if it took effect.

## Create and change

```bash
miosa service create shop API --type web --repo https://github.com/acme/api --replicas 2
miosa service update shop API --replicas 4
miosa service scale  shop API --replicas 4        # also queues a release
```

`update` merges settings: only the flags you pass change.
It sends the `lock_version` it read; a stale value is refused with `STALE_SERVICE` so two editors cannot overwrite each other.
`scale` changes replicas, region or limits and queues a release that applies them.

## Cron jobs

```bash
miosa service create shop Nightly --type schedule \
  --repo https://github.com/acme/api \
  --start-command "node jobs/nightly.js" \
  --schedule "0 3 * * *"
miosa service runs shop Nightly --output
```

`runs` shows the newest 20 runs with state, exit code, times and output.
A run is limited to five minutes.

## Database services

A database service is a managed Postgres or Redis bound to the project.
Apply a template to create one with its variables wired, or attach an existing managed database.
The connection string is only available through `connection`, which is audited:

```bash
miosa service connection shop Postgres
```

## Limits and errors

| Error | Meaning |
|---|---|
| `SERVICE_NOT_FOUND` | No service with that id in the project. The API resolves services by id only. |
| `STALE_SERVICE` | The `lock_version` you sent is older than the stored one. Re-read and retry. |
| `SERVICE_CONFLICT` | `start` on a service that is already running or starting. |
| `SERVICES_NOT_STOPPED` | `stop` could not stop a replica; the service stays running. |
| `INGRESS_DISABLED` | Public addresses are off for this platform; the service runs but has no address to reach it on. |
| `PLAN_LIMIT_REACHED` | The plan's service limit is reached. |
