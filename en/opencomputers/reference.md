---
slug: "opencomputers/reference"
title: "API, CLI, and SDK reference"
description: "Every OpenComputers endpoint group, the CLI command families that call them, and the SDK namespaces that wrap them."
section: "Computers"
order: 3025
---

Everything OpenComputers is reachable three ways: the REST API, the `miosa` CLI, and the SDKs.
This page is the map. Field-level detail for hosts, jobs, files, tunnels, agents, and clusters lives in the [OpenComputers API reference](/docs/api-reference/open-computers).

## REST API

Base path: `/api/v1/opencomputers`. All calls need `Authorization: Bearer <platform-api-key>`.

| Group | Endpoints |
|---|---|
| Hosts | `GET /hosts`, `POST /hosts`, `GET /hosts/:id`, `DELETE /hosts/:id` |
| Host tags | `PATCH /hosts/:id/tags`, `POST /hosts/:id/tags/:tag`, `DELETE /hosts/:id/tags/:tag`, `GET /tags` |
| Host runtime | `GET /hosts/:id/runtime`, `POST /hosts/:id/runtime`, `POST /hosts/:id/runtime/certify`, `POST /hosts/:id/runtime/enrollment` |
| Host info | `GET /hosts/:id/metrics`, `GET /hosts/:id/services`, `GET /hosts/:id/jobs` |
| Groups | `GET /groups`, `POST /groups`, `GET /groups/:id`, `PATCH /groups/:id`, `DELETE /groups/:id`, `POST /groups/:id/members`, `POST /groups/:id/members/:host_id`, `DELETE /groups/:id/members/:host_id`, `POST /groups/:id/exec`, `POST /groups/:id/apps/:app_id/install` |
| Jobs | `POST /hosts/:id/exec`, `GET /hosts/:id/exec`, `POST /hosts/:id/jobs`, `GET /hosts/:id/exec/:job_id`, `DELETE /hosts/:id/exec/:job_id` |
| Files | `GET /hosts/:id/fs`, `GET /hosts/:id/fs/stat`, `GET /hosts/:id/fs/download`, `POST /hosts/:id/fs/upload`, `DELETE /hosts/:id/fs`, `POST /hosts/:id/fs/mkdir` |
| Terminal and desktop | `POST /hosts/:id/terminal/ticket`, `POST /hosts/:id/desktop/ticket` |
| Containers | `GET, POST /hosts/:host_id/containers`, `GET, PATCH, DELETE /hosts/:host_id/containers/:id`, `POST .../:id/start`, `POST .../:id/stop`, `POST .../:id/restart` |
| Compose | `GET, POST /hosts/:host_id/compose-projects`, `GET, DELETE /hosts/:host_id/compose-projects/:id`, `POST .../up`, `POST .../down` |
| IDE sessions | `GET, POST /hosts/:host_id/ide-sessions`, `GET, DELETE /hosts/:host_id/ide-sessions/:id`, `POST .../reveal-password` |
| GitHub Actions runners | `GET, POST /hosts/:id/gha-runners`, `GET, DELETE /hosts/:id/gha-runners/:runner_id`, `POST .../refresh` |
| Meshes | `GET, POST /meshes`, `GET, DELETE /meshes/:id`, `POST /meshes/:id/hosts/:host_id`, `DELETE /meshes/:id/hosts/:host_id` |
| Backups | `GET, POST /hosts/:host_id/backup-configs`, `PATCH, DELETE /hosts/:host_id/backup-configs/:config_id`, `POST .../pause`, `POST .../resume`, `POST .../trigger`, `GET .../snapshots` |
| Snapshots | `GET /snapshots/:id`, `DELETE /snapshots/:id`, `POST /snapshots/:id/restore` |
| Tunnels | `GET, POST /hosts/:host_id/tunnels`, `GET, PATCH, DELETE /hosts/:host_id/tunnels/:id` |
| Tunnel domains | `GET, POST /tunnels/:tunnel_id/domains`, `GET, DELETE /tunnels/:tunnel_id/domains/:id`, `POST .../verify` |
| Secrets | `GET, POST /secrets`, `GET, POST /hosts/:id/secrets`, `PATCH, DELETE /hosts/:id/secrets/:secret_id`, `POST .../reveal` |
| SSH keys | `GET, POST /hosts/:id/ssh-keys`, `DELETE /hosts/:id/ssh-keys/:key_id`, `POST /hosts/:id/ssh-keys/import/github` |
| Clipboard | `POST /hosts/:id/clipboard`, `GET /hosts/:id/clipboard` |
| Apps | `GET /apps`, `GET /hosts/:id/apps`, `POST /hosts/:id/apps/:app_id/install`, `GET /hosts/:id/apps/installs/:install_id`, `DELETE /hosts/:id/apps/:app_id`, `POST .../:app_id/start`, `POST .../:app_id/health` |
| Workspaces | `GET /workspaces`, `GET, POST /hosts/:host_id/workspaces`, `GET, PATCH, DELETE /hosts/:host_id/workspaces/:wid`, `POST .../pull`, `POST .../open-terminal`, `POST .../run`, `POST .../expose` |
| Schedules | `GET, POST /hosts/:host_id/schedules`, `GET, PATCH, DELETE /hosts/:host_id/schedules/:id`, `POST .../pause`, `POST .../resume`, `POST .../trigger`, `GET .../runs` |
| Shares | `GET, POST /hosts/:id/shares`, `PATCH, DELETE /hosts/:id/shares/:share_id` |
| Alerts | `GET, POST /alerts/rules`, `PATCH, DELETE /alerts/rules/:id`, `POST .../fire-test`, `GET, POST /alerts/channels`, `PATCH, DELETE /alerts/channels/:id`, `POST .../verify`, `GET /alerts/fires`, `POST /alerts/fires/:id/ack` |
| Webhooks | `GET, POST /webhooks`, `PATCH, DELETE /webhooks/:id`, `POST .../test`, `GET .../deliveries` |
| Audit | `GET /audit`, `GET /audit/export`, `GET /audit/verify`, `GET /audit/:id` |
| Clusters | `GET, POST /clusters`, `GET, DELETE /clusters/:id`, `POST /clusters/:id/start`, `POST /clusters/:id/stop` |

### Streaming endpoints

These answer with Server-Sent Events and keep the connection open:

| Stream | Endpoint |
|---|---|
| Job output | `GET /hosts/:id/exec/:job_id/stream` |
| Host lifecycle | `GET /hosts/events` |
| Container logs | `GET /hosts/:host_id/containers/:id/logs` |
| Container stats | `GET /hosts/:host_id/containers/:id/stats` |
| Compose events | `GET /hosts/:host_id/compose-projects/:id/events` |
| Mesh events | `GET /meshes/:id/events` |
| Runner events | `GET /hosts/:id/gha-runners/:runner_id/events` |
| Agent session events | `GET /hosts/:id/agent/sessions/:session_id/events` |
| App install events | `GET /hosts/:id/apps/installs/:install_id/events` |
| Cluster events | `GET /clusters/:id/events` |
| Workspace events | `GET /hosts/:host_id/workspaces/:wid/events` |
| Audit feed | `GET /audit/live` |

`GET /hosts/events` uses a ticket rather than a bearer token: mint one with `POST /api/v1/opencomputers/hosts/events/sse-ticket`, then connect with `?ticket=`.

### Inference

An active cluster serves an OpenAI-compatible endpoint:

| Endpoint | Purpose |
|---|---|
| `POST /inference/:slug/v1/chat/completions` | Chat completions |
| `POST /inference/:slug/v1/completions` | Text completions |
| `GET /inference/:slug/v1/models` | Model list |

### Unauthenticated endpoints

| Endpoint | Purpose |
|---|---|
| `POST /opencomputers/hosts/registration-status` | Check whether a host key is valid and connected |
| `POST /opencomputers/schedules/trigger/:token` | Trigger a schedule from a webhook, using its one-time token |
| `GET /opencomputers/osa/latest` | The latest agent release manifest |
| `ANY /t/:slug/*path` | The public tunnel proxy; the tunnel's own access mode decides who gets through |

## CLI

The CLI exposes OpenComputers as `miosa host`, with `miosa host` as an alias.

**Hosts and work**

```bash
miosa host register <name> --platform linux|macos|windows
miosa host list --tag <tag>
miosa host get <host>
miosa host metrics <host>
miosa host runtime <host>
miosa host services <host>
miosa host jobs <host>
miosa host job list|get|cancel <host> [job]
miosa host fs ls|stat|mkdir|rm <host> --path <path>
miosa host tag list|add|rm
miosa host group list|get|new|add|remove|exec|rm
miosa host secret list|set|rm
miosa host ssh-key list|add|import|rm
miosa host app catalog|list|install|start|health|uninstall
miosa host agent runs|run|cancel
miosa host rm <host> --yes
```

**Containers and networking**

```bash
miosa host container list|get|run|start|stop|restart|rm
miosa host container logs|stats <host> <container>
miosa host compose list|up|down|rm
miosa host mesh list|new|get|add-host|remove-host|events|rm
miosa host tunnel list|get|new|rm
miosa host gha-runner list|new|get|events|refresh|rm
miosa host backup list|new|update|trigger|pause|resume|rm
miosa host backup snapshot list|get|fork|rm
miosa host cluster list|get|start|stop|rm
```

**Any runtime kind**

```bash
miosa device list
miosa device get|capabilities <device>
miosa device exec <device> -- <command>
miosa device files|read|write <device> --path <path>
miosa device expose <device> <port>
miosa device browser <device>
miosa device extend|pause|resume|stop|rm <device>
```

`miosa host` and `miosa device` both accept the global flags (`--json`, `--org`, `--profile`, `--timeout`, and others). See the [CLI reference](/docs/cli/platform#your-own-hosts) for the full command and flag list.

## SDKs

| SDK | Namespace |
|---|---|
| [TypeScript](/docs/sdks/typescript) | `client.openComputers` |
| [Python](/docs/sdks/python) | `client.open_computers` |
| [Go](/docs/sdks/go) | `client.OpenComputers` |
| [Elixir](/docs/sdks/elixir) | `Miosa.OpenComputers.*` modules |

```typescript
import Miosa from '@miosa/sdk';

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY });

const host = await miosa.openComputers.hosts.create({ name: 'studio', platform: 'linux' });
console.log(host.host_key);                       // shown once

const job = await miosa.openComputers.jobs.start(host.id, { command: 'uname -a' });
console.log(job.links.stream);                   // stream this job
```

```python
import miosa

client = miosa.Miosa(api_key="msk_...")

for host in client.open_computers.hosts.list().data:
    print(host.id, host.status)
```

> **Note: The SDKs do not cover the same surface**
>
> Coverage differs by language. The Python and Elixir SDKs wrap the widest surface - containers, compose, meshes, backups, runner provisioning, schedules, shares, alerts, audit, and webhooks. The TypeScript and Go SDKs cover hosts, jobs, files, tunnels, agents, clusters, and secrets.
> When your language does not wrap a resource, call the REST endpoint directly.

> **Caution: Some methods raise by design**
>
> Host rename is not available on the tenant API, so SDK `update` methods for hosts raise an unsupported error rather than sending a request. Rename is possible only on the admin surface. Use tags to group hosts instead.

- [OpenComputers API reference](/docs/api-reference/open-computers): Request fields and response shapes for hosts, jobs, files, tunnels, agents, clusters, and secrets.

- [CLI reference](/docs/cli/platform#your-own-hosts): Every miosa host and miosa device command.

- [Devices](/docs/opencomputers/devices): The id-addressed device surface.
