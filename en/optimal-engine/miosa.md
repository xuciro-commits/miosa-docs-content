---
slug: "optimal-engine/miosa"
title: "Optimal Engine on MIOSA"
description: "Run a managed Optimal Engine on MIOSA - workspace Memory in the console, how Optimal chat uses it, the miosa engine commands, client keys for the optimal CLI and MCP server, the API, sizes, backups, and how to get it enabled."
section: "Optimal Engine"
order: 8010
---

On MIOSA, an Optimal Engine runs as a managed database with the engine `optimal`.
MIOSA creates and runs it for you: you never install it, hold its admin key, or take its backups by hand.

## Availability

Managed Engines are enabled per organization.
Until yours is enabled:

- the console's create flow under **Databases** does not offer Optimal Engine;
- **Start Memory** answers "Memory could not be started. Your role may not allow it, or Memory is not set up for this organization yet.";
- creating one over the API or CLI answers `422` with the code `ENGINE_NOT_AVAILABLE`.

To have it turned on, open a ticket from the console or email [support@miosa.ai](mailto:support@miosa.ai) with your organization's name.
See [Support](/docs/faq/support).
Self-hosting needs no enablement; see the [quickstart](/docs/optimal-engine/quickstart).

## What you get

| Part | Detail |
|---|---|
| Machine | A private machine in your workspace's network, reachable only inside MIOSA and through the MIOSA API |
| Data | A durable data disk that survives restarts and upgrades |
| Keys | MIOSA holds the Engine's admin key and never shows it; you mint short-lived client keys |
| Workspaces | An Engine workspace named after each MIOSA workspace attached to it |
| Backups | Scheduled backups of the whole data disk, and backups on demand |
| Health | Health checks reported by the Engine itself |
| Search | Full-text search. Managed Engines do not include an embedding model, so vector search is not available on MIOSA. |

It is billed like an always-on managed database while it runs, at the hourly price of its size.
The console shows that price, and the price for a full month, before it creates anything.

## Workspace Memory in the console

**Optimal > Memory** (`/optimal/memory`) is the simplest way in.
Memory belongs to one workspace: pick a workspace in the switcher first.

1. Press **Start Memory**.
   - If your organization already has a running Engine, Memory attaches this workspace to it at no new cost.
   - Otherwise the console offers to create one with 1 vCPU, 2 GiB of memory, and the smallest disk, and shows its price. Press **Create and start Memory**.
2. "Memory is starting. This takes about a minute."
3. When it is running, the page has a search box over everything the workspace knows, with each result's title, location, and an excerpt.

To add to Memory, ask Optimal in a chat to save a note.

## How Optimal chat uses Memory

When the workspace has Memory, [Optimal](/docs/optimal) can:

- **search it** before answering about the workspace's people, projects, clients, or history, and say so when Memory has nothing;
- **save** a note, a decision, or a transcript you ask it to keep;
- **start** Memory for the workspace when you ask.

In an **All workspaces** chat, Optimal searches every workspace you can open and names the workspace each result came from.

Each chat also has a Memory panel:

- **Save to memory** stores the chat as a session.
- **Sources** lists what Optimal read, grouped by workspace in an organization chat.
- **Learned in this chat** lists the claims the chat produced; accept or reject each one.

Optimal reaches Memory through MIOSA as you, with your permissions.
It holds no Engine key and has no network path to the Engine.
What Memory returns is passed to the model as untrusted content.

## From a terminal

The [MIOSA CLI](/docs/cli) has a `miosa engine` command group, version 2.3.19 or newer.

```bash
miosa engine create brain --workspace notes --for workspace --wait
miosa engine list
miosa engine status notes
```

`create` returns the existing Engine when one with that name exists, so scripts can call it every time.
`--for` picks a starting size: `agent` (tiny, 2 GiB disk), `workspace` (xs, 10 GiB), or `business` (small, 25 GiB).
`--size tiny|xs|small|medium|large` and `--disk <GiB>` override it.
An Engine created for a workspace attaches to that workspace when it first runs.

| Command | What it does |
|---|---|
| `miosa engine attach <workspace> --to <engine>` | Attach another workspace to an Engine |
| `miosa engine detach <workspace>` | Detach a workspace and revoke its Engine key |
| `miosa engine status [workspace]` | Which Engine serves a workspace, or every Engine and its workspaces |
| `miosa engine connect <engine>` | Mint a client key and write an `optimal` CLI profile |
| `miosa engine shell <engine> [-- command]` | Run a shell or a command with a key that is revoked when it exits |
| `miosa engine backup <engine>`, `miosa engine backups <engine>` | Take a backup now, or list backups |
| `miosa engine recover <engine>` | Restore a backup over the Engine, or with `--to-new <name>` into a new one |
| `miosa engine upgrade <engine> --to <version>`, `miosa engine rollback <engine>` | Change the Engine version |
| `miosa engine rm <engine>` | Delete the Engine and its data disk |

### Use the optimal CLI and MCP server

`miosa engine connect` mints a read/write key for one of the Engine's workspaces and writes `~/.config/optimal/profile` with mode `0600`.
The [`optimal` CLI and MCP server](/docs/optimal-engine/cli-sdks-mcp) read that profile, so they reach the managed Engine with no other setup.

```bash
miosa engine connect brain                       # key lasts 24 hours
miosa engine connect brain --workspace notes --profile notes --ttl 72h
optimal boot
optimal search "renewal"
miosa engine shell brain -- optimal brief        # one command, key revoked after
```

`--print` prints shell exports instead of writing a profile.
Keys last 24 hours by default and at most 30 days.

## Machines in the workspace

Sandboxes, computers, apps, and agents in a workspace attached to an Engine start with two variables:

| Variable | Value |
|---|---|
| `OPTIMAL_ENGINE_URL` | The Engine's private address, on port 4200 |
| `OPTIMAL_ENGINE_API_KEY` | A key limited to that workspace |

The `optimal` CLI and the MCP server read both, so an agent on the machine can call `optimal boot` straight away.
A machine still starts when the Engine is down.

## Call the Engine API

From outside the workspace, call the Engine through MIOSA with a client key.

1. Mint a client key. Only an organization owner or admin can mint one.

   ```bash
   curl -s -X POST https://api.miosa.ai/api/v1/databases/$ENGINE_ID/client-keys \
     -H "Authorization: Bearer $MIOSA_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"workspace_id": "your_workspace_id", "ttl_seconds": 86400}'
   ```

   `201` returns `data.key` once, with its `id` and `expires_at`.
   The workspace must be attached to the Engine.
   `GET /databases/:id/client-keys` lists keys without their secrets, and `DELETE /databases/:id/client-keys/:key_id` revokes one.

2. Call the Engine with it:

   ```bash
   curl -s "https://api.miosa.ai/api/v1/engines/$ENGINE_ID/engine-api/api/search?q=renewal" \
     -H "Authorization: Bearer $ENGINE_CLIENT_KEY"
   ```

The proxy forwards only the routes an agent needs, and answers `404` for everything else, including key, sync, backup, and maintenance routes.

| Method | Routes under `/api/` |
|---|---|
| `GET` | `boot`, `brief`, `protocol`, `operating-projection`, `rhythm/current`, `loops`, `search`, `grep`, `read`, `memory`, `memory-core/claims`, `memory-core/claim-review`, `sessions`, `sessions/:id`, `org-policy`, `review/routing`, `routing/rules`, `authority/effective`, `viability`, `viability/history`, `analyses`, `jobs`, `workspaces`, `status`, `health`, `version`, `profile`, `recall/:kind`, `wiki/:slug`, `render/specs`, `agent-actions/:id` |
| `POST` | `rag`, `assemble`, `reconstruct`, `ingest`, `memory`, `memory/remember`, `memory/:id/forget`, `loops`, `loops/extract`, `loops/:id/:action`, `sessions/commit`, `routing/classify`, `signals/check`, `authority/decide`, `authority/requests`, `agent-actions`, `render/context` |

| Status | Code | When |
|---|---|---|
| 404 | `NOT_FOUND` | The route is not on the list |
| 409 | `ENGINE_NOT_RUNNING` | The Engine is stopped or starting |
| 413 | `REQUEST_TOO_LARGE` | The body is over 1 MiB, or the query string over 8 KiB |
| 502 | `ENGINE_UNAVAILABLE`, `ENGINE_RESPONSE_TOO_LARGE` | The Engine did not answer, or answered too much |

### Workspace Memory routes

These use your MIOSA API key, not a client key, and reach the Engine that serves the workspace.

All paths start with `https://api.miosa.ai/api/v1/workspaces/:workspace_id/engine`.

| Route | What it does | Permission |
|---|---|---|
| `GET .../engine` | Which Engine serves the workspace | |
| `GET .../engine/search?q=&limit=` | Search the workspace's Memory | `databases:read` |
| `GET .../engine/boot` | The workspace's boot briefing | `databases:read` |
| `POST .../engine/remember` with `{"content", "metadata"}` | Save a memory | `databases:write` |
| `POST .../engine/sessions` with `{"correlation_id", "messages", "title"}` | Save a chat as a session | `databases:write` |
| `GET .../engine/sessions/:session_id` | Read a saved chat | `databases:read` |
| `POST .../engine/claims/:claim_id/accept` or `/reject` | Review a claim | `databases:write` |

A workspace with no Engine answers `404 NOT_FOUND`; an Engine that is not running answers `409 ENGINE_NOT_RUNNING`.

## Backups, upgrades, and size

Open the Engine under **Databases** in the console.
Its page has an **Overview** tab with the connection, health, and usage against the size's limits, a **Backups** tab, a **Resize** button, and, when the Engine reports them, **Brief**, **Loops**, **Workspaces**, and **Keys** tabs.

| Size | vCPU | Memory | Disk |
|---|---|---|---|
| `agent` | 1 | 1 GiB | 2 GiB |
| `workspace` | 1 | 2 GiB | 10 GiB |
| `business` | 2 | 4 GiB | 25 GiB |
| `platform` | 4 | 8 GiB | 100 GiB |
| `scale` | 8 | 16 GiB | 250 GiB |

Resizing moves between `workspace`, `business`, `platform`, and `scale`; `agent` is a starting size only.
A resize refuses a size whose disk or memory cannot hold what the Engine already uses, and restarts a running Engine.
Over the API, `POST /api/v1/databases/:id/resize` takes `{"size": "business"}`, and `"dry_run": true` returns the plan without changing anything.

`miosa engine recover --to-new` restores a backup into a new Engine and never touches the original.
Restoring over the Engine keeps the previous data aside; `GET /api/v1/databases/:id/restore-copies` lists those copies.

> **Note: Self-hosted and managed together**
>
> A managed Engine is the same software as a self-hosted one, so the [API](/docs/optimal-engine/api), the [CLI](/docs/optimal-engine/cli-sdks-mcp), and the [concepts](/docs/optimal-engine/concepts) apply to both.
