---
slug: "optimal-engine/api"
title: "Optimal Engine API reference"
description: "The Optimal Engine HTTP API - base URL, keys, scopes, paging, rate limits, errors, and the main routes for workspaces, ingest, search, ask, memory, claims, and operations, with curl examples."
section: "Optimal Engine"
order: 8008
---

| | |
|---|---|
| Base URL | `http://127.0.0.1:4200/api` on a self-hosted Engine. `/v1/...` is an alias of `/api/...`. |
| On MIOSA | `https://api.miosa.ai/api/v1/engines/<engine id>/engine-api/api/...` with an Engine client key; see [Optimal Engine on MIOSA](/docs/optimal-engine/miosa#call-the-engine-api) |
| API version | `v1` |
| Auth | `Authorization: Bearer <key>` or `X-API-Key: <key>` |
| Body | JSON with `Content-Type: application/json` |
| Errors | `{"error": "<code>"}`, sometimes with a `detail` |

The Engine has about 300 routes.
This page covers the ones most apps and agents use.
The complete list, with the grant each route needs, is generated from the router into [`docs/reference/api.md`](https://github.com/Miosa-osa/OptimalEngine/blob/main/docs/reference/api.md), and a test fails whenever the two disagree.

```bash
E=http://127.0.0.1:4200
K="Authorization: Bearer $OPTIMAL_ENGINE_API_KEY"
```

## Keys and errors

When `OPTIMAL_AUTH_REQUIRED=true`, every route needs a key, including `/api/health`.

| Status | `error` | When |
|---|---|---|
| 400 | `invalid_json`, `invalid_parameter`, `workspace_required` | The request is malformed, or a key limited to named workspaces did not name one |
| 401 | `missing_api_key`, `invalid_api_key`, `api_key_revoked`, `api_key_expired` | The key is missing or not usable |
| 403 | `workspace_scope_denied` | The key does not cover that workspace |
| 403 | `permission_scope_denied` | The key does not have the scope the route needs |
| 404 | `not found`, `workspace not found` | No such route, or a workspace of another tenant |
| 413 | `request_too_large` | The body is over 8 MB |
| 426 | `client_incompatible` | An `optimal` CLI or MCP server of another major version |
| 429 | `rate_limited` | Over the key's rate limit; the body has `retry_after_ms` |
| 503 | `busy` | The store is busy, for example right after boot; retry after `Retry-After` |

### Rate limits

Each key has its own budget per minute, set by the [size profile](/docs/optimal-engine/operations#sizing): 1,200 requests a minute on the default `business` profile.
Requests without a valid key share their IP address's budget.
Every response carries `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset`.
`/api/health`, `/api/status`, and `/api/metrics/prometheus` are not counted.

### Paging

List routes take `offset` and `limit` (1 to 200, default 50) and answer with:

```json
"pagination": {"offset": 0, "limit": 50, "total": 132, "has_more": true}
```

## Engine

| Route | Returns |
|---|---|
| `GET /api/health` | `status` (`up`, `degraded`, `down`), `checks`, `warnings`, and the release. `?full=true` runs ten store diagnostics for one workspace. |
| `GET /api/status` | The same, plus `api_version` |
| `GET /api/version` | `version`, `git_sha`, `build_timestamp`, `release_channel`, `api_version`, `expected_migration`, `capabilities` |
| `GET /api/metrics` | JSON counters, latencies, and size gauges |
| `GET /api/metrics/prometheus` | Prometheus text format |
| `GET /api/auth/whoami` | The key's own scopes and workspaces |

## Workspaces and nodes

### Create a workspace

`POST /api/workspaces`

```bash
curl -s -X POST $E/api/workspaces -H "$K" -H "Content-Type: application/json" \
  -d '{"slug": "sales", "name": "Sales"}'
```

| Field | Required | Meaning |
|---|---|---|
| `slug` | yes | Lowercase name, unique in the tenant |
| `name` | yes | Display name |
| `description` | no | |
| `external_id` | no | Your own id for this workspace, unique in the tenant |

`201` returns the workspace: `id` (`default:sales`), `tenant_id`, `slug`, `name`, `status`, `created_at`.
Errors: `400` for a missing or invalid slug or name, `409 slug_taken`, `409 external_id_taken`.

| Route | What it does |
|---|---|
| `GET /api/workspaces` | List workspaces, paged |
| `GET /api/workspaces/:id`, `PATCH /api/workspaces/:id` | Read or rename one |
| `POST /api/workspaces/:id/archive` | Archive it; its data is kept |
| `GET /api/workspaces/:id/config`, `PATCH ...` | Workspace settings |

### Create a node

`POST /api/nodes` with `{"name", "kind", "slug", "workspace", "description"}` answers `{"ok": true, "id", "slug", "kind"}`.
`kind` is one of the node kinds in [Concepts](/docs/optimal-engine/concepts#topology).

## Ingest

`POST /api/ingest` stores text as a source.
See [Sources and ingestion](/docs/optimal-engine/ingestion) for the fields, files, chat sessions, Notion, and connectors.

```bash
curl -s -X POST $E/api/ingest -H "$K" -H "Content-Type: application/json" \
  -d '{"workspace": "default:sales", "title": "Auth decision", "text": "Decided: refresh-token rotation ships in v2."}'
```

```json
{"ok": true, "signal_id": "...", "genre": "...", "type": "...", "entities": [], "source_package_id": "..."}
```

## Search

`GET /api/search?q=&workspace=&limit=&offset=&mode=&type=&as_of=`

```bash
curl -s "$E/api/search?q=refresh%20token&workspace=default:sales&limit=5" -H "$K"
```

```json
{
  "query": "refresh token",
  "results": [
    {
      "id": "...",
      "title": "Auth decision",
      "uri": "optimal://...",
      "content": "Decided: refresh-token rotation ships in v2.",
      "l0_abstract": "...",
      "score": 0.82,
      "workspace_id": "default:sales",
      "inherited": false
    }
  ],
  "pagination": {"offset": 0, "limit": 5, "total": 1, "has_more": false}
}
```

`mode=hybrid` answers `503 hybrid_unavailable` when no embedding model answers.
A bad `as_of` answers `400`.

| Route | What it does |
|---|---|
| `GET /api/grep?q=` | Matching chunks, with `intent`, `scale`, `modality`, `literal`, and `path` filters |
| `GET /api/read?uri=&tier=` | One item by its `optimal://` URI; `tier` is `l0`, `l1`, or `full` |
| `GET /api/recall/who`, `when`, `where`, `owns`, `actions` | Targeted recall over people, time, places, ownership, and actions |
| `GET /api/profile` | A workspace profile: ground truth, recent activity, and top entities |
| `GET /api/graph`, `/api/graph/hubs`, `/api/graph/clusters` | The knowledge graph |

## Ask and assemble

`POST /api/rag` returns context for one question; `GET /api/rag/stream` streams it as server-sent events.
`POST /api/assemble` returns a tiered context package for an agent.
See [Memory and retrieval](/docs/optimal-engine/memory-and-retrieval#ask) for fields and responses.

```bash
curl -s -X POST $E/api/rag -H "$K" -H "Content-Type: application/json" \
  -d '{"query": "What did we decide about auth?", "workspace": "default:sales", "format": "markdown", "bandwidth": "small"}'

curl -N "$E/api/rag/stream?query=auth%20decision&workspace=default:sales" -H "$K"
```

## Memory

| Route | What it does |
|---|---|
| `POST /api/memory` | Store a memory: `{"content", "workspace", "is_static", "audience", "citation_uri", "metadata"}`. Answers `201` with the memory, its `version`, and `was_existing`. |
| `POST /api/memory/remember` | Store through the gate; answers `{"action", "memory", "pending_claim_id", "gate"}` |
| `GET /api/memory` | List; `q`, `audience`, `include_forgotten`, paging |
| `GET /api/memory/:id` | One memory |
| `GET /api/memory/:id/versions`, `/relations` | Its history and links |
| `POST /api/memory/:id/update`, `/extend`, `/derive` | A new linked version |
| `POST /api/memory/:id/forget` | Hide it |
| `DELETE /api/memory/:id` | Delete it |

## Claims

| Route | Scope |
|---|---|
| `GET /api/memory-core/claim-review` | `read` |
| `GET /api/memory-core/claims`, `/claims/:id` | `read` |
| `POST /api/memory-core/claims/:id/promote` | `claims:review` |
| `POST /api/memory-core/claims/:id/reject` | `claims:review` |

## Chat sessions

| Route | What it does |
|---|---|
| `POST /api/sessions/commit` | Store a chat; see [Sources and ingestion](/docs/optimal-engine/ingestion#chat-sessions) |
| `GET /api/sessions` | Committed chats, newest first |
| `GET /api/sessions/:id` | One chat and the review state of what it learned |

## Operations

These need an `admin` key.

| Route | What it does |
|---|---|
| `POST /api/auth/keys`, `GET /api/auth/keys` | Mint and list keys |
| `POST /api/auth/keys/:id/revoke`, `DELETE /api/auth/keys/:id` | Revoke or delete a key |
| `POST /api/backup` | Take a backup now |
| `GET /api/backup/offsite` | Offsite backups |
| `POST /api/workspaces/:id/bundle`, `POST /api/bundles/import` | Export or import a workspace bundle |
| `GET /api/stores`, `GET /api/storage` | Stores, providers, and usage against the size profile |
| `/api/sync/*` | Replication; see [Operations](/docs/optimal-engine/operations#replication) |

> **Note: App data**
>
> `/api/data` is a separate, smaller API for app records: collections, records, queries, and a change stream.
> A key made for it can call nothing else.
> The repository guide `docs/guides/app-data-api.md` describes it.
