---
slug: "opencomputers/containers"
title: "Containers on a host"
description: "Run Docker or Podman containers on one of your hosts, stream their logs and resource usage, and keep docker-compose projects running."
section: "Computers"
order: 3016
---

A host runs its own container runtime, and MIOSA manages containers on it: create, start, stop, restart, delete, and stream logs and resource usage.
Everything is addressed by the host ID and the container ID, and every operation is tenant-scoped.

MIOSA does not build the image for you. Your image must already be pullable by the host's runtime.

## Run a container

**miosa CLI**

```bash
miosa host container run <host> --image nginx:alpine --name web --set PORT=8080
miosa host container list <host>
miosa host container get <host> <container>
```

`miosa host container run` takes `--image`, `--name`, `--cmd` (repeatable), `--set KEY=VALUE` (repeatable), and `--restart`.
Aliases: `miosa host containers`, `miosa host container create`.

**REST API**

```bash
curl --fail-with-body -sS \
  -X POST "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers" \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"web","image":"nginx:alpine","env":{"PORT":"8080"},"ports":[{"host_port":8080,"container_port":8080}]}'
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | 1 to 63 characters, starting with a letter or digit |
| `image` | string | Yes | Image reference the host's runtime can pull |
| `command` | string[] | No | Command and arguments; default `[]` (the image default) |
| `env` | object | No | Environment variables as a map; default `{}` |
| `ports` | object[] | No | `{host_port, container_port, protocol}`; `protocol` defaults to `tcp` |
| `volumes` | object[] | No | `{source, target, readonly}`; `readonly` defaults to `false` |
| `restart_policy` | string | No | `no`, `on-failure`, `always`, or `unless-stopped` (default) |
| `runtime` | string | No | `auto` (default), `docker`, or `podman` |

The response is `{"container": {...}}` with `id`, `name`, `image`, `command`, `env`, `ports`, `volumes`, `restart_policy`, `runtime`, `state`, `runtime_id`, `runtime_name`, `exit_code`, `started_at`, `stopped_at`, `last_error`, `last_cpu_percent`, `last_mem_mb`, and `last_stats_at`.

Container states are `provisioning`, `running`, `stopped`, `failed`, `removing`, and `removed`.

> **Note: Check state, do not assume it**
>
> `create` returns as soon as the host accepts the request. Read `state` before you rely on the container: `provisioning` is not `running`, and `last_error` is where a failed start explains itself.

## Logs

**miosa CLI**

```bash
miosa host container logs <host> <container>
miosa host container logs <host> <container> --json
```

Streams until the connection ends or you press Ctrl-C. With `--json`, each event is one JSON object per line.

**REST API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers/$CONTAINER_ID/logs" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

An SSE stream of `{stream, line, ts}` frames, where `stream` is the source stream on the container.

## Stats

**miosa CLI**

```bash
miosa host container stats <host> <container>
miosa host container stats <host> <container> --json
```

**REST API**

```bash
curl -N --fail-with-body -sS \
  "https://api.miosa.ai/api/v1/opencomputers/hosts/$HOST_ID/containers/$CONTAINER_ID/stats" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

An SSE stream that ticks about every 10 seconds with `{container_id, cpu_percent, mem_mb, stats_at}`, and sends a final `{state, container_id}` frame when the container stops.

The latest sample is also kept on the container record as `last_cpu_percent`, `last_mem_mb`, and `last_stats_at`.

## Lifecycle

```bash
miosa host container start <host> <container>
miosa host container stop <host> <container>
miosa host container restart <host> <container>
miosa host container rm <host> <container> --force --yes
```

| Action | REST | Notes |
|---|---|---|
| Start | `POST /opencomputers/hosts/{host_id}/containers/{id}/start` | Only from `stopped` or `failed`; returns `409` otherwise |
| Stop | `POST .../containers/{id}/stop` | Only from `running`; accepts a `timeout_s` grace period, default 10 |
| Restart | `POST .../containers/{id}/restart` | Allowed from `running`, `stopped`, or `failed` |
| Update | `PATCH .../containers/{id}` | `image`, `command`, `env`, `ports`, `volumes`, `restart_policy`, `runtime` |
| Delete | `DELETE .../containers/{id}?force=true` | `force` deletes a running container; returns `202` |

Deleting a container does not delete its volumes for you unless you asked for the volume to be removed with the container.

## Compose projects

A compose project is a docker-compose file the host keeps running.

```bash
miosa host compose list <host>
miosa host compose up <host> app --build --pull
miosa host compose down <host> app --remove-volumes
miosa host compose rm <host> app --yes
```

| Action | REST |
|---|---|
| List | `GET /opencomputers/hosts/{host_id}/compose-projects` |
| Create | `POST .../compose-projects` with `name`, `yaml` (required), `env`, `pull`, `build` |
| Up | `POST .../compose-projects/{id}/up` |
| Down | `POST .../compose-projects/{id}/down` with `remove_volumes` |
| Delete | `DELETE .../compose-projects/{id}` |
| Events | `GET .../compose-projects/{id}/events` (SSE) |

Compose project states are `creating`, `running`, `partial`, `stopped`, `failed`, and `removing`.
The events stream carries `{type, payload}` frames where `type` is `progress`, `log`, or `error`.

`--remove-volumes` destroys the project's volumes. Confirm what you intend to lose before using it on a stateful project.

## Plan limits

The number of container slots a tenant can use depends on the plan, and a compose project counts its services against the same budget.

| Plan | Containers and compose service slots |
|---|---|
| `free` | 2 |
| `starter` | 10 |
| `pro` | 50 |
| `team` | 250 |
| `enterprise` | 1000 |

Exceeding the limit returns `402` with `error: "plan_limit_reached"`, plus the `limit` and `current` values.
See [Billing and limits](/docs/opencomputers/billing) for the other plan-scoped limits.

> **Caution: A container on a host is not a sandbox**
>
> Containers share the host's kernel. They are not Firecracker microVMs, and they do not give you the isolation or the disposable lifecycle of a [sandbox](/docs/develop/sandboxes).
> Use containers when you want a long-lived service on a machine you own.
