---
slug: "optimal-engine/troubleshooting"
title: "Optimal Engine troubleshooting"
description: "Fixes for common Optimal Engine problems - nothing listening, health checks, keys refused, empty search, hybrid search unavailable, busy store, failed migrations, Docker health, and errors from a managed Engine on MIOSA."
section: "Optimal Engine"
order: 8011
---

Each entry starts with what you see, then the fix.

## Start-up

### "Optimal Engine needs Erlang/OTP 27 or later"

The Engine uses the JSON module that ships with OTP 27.
Install OTP 27 or newer; avoid 28.0 to 28.2.

### The Engine starts but nothing listens on port 4200

The HTTP API is off unless `OPTIMAL_API_ENABLED=true`.
Set it and start again.
`mix optimal.api` alone prints a listening line without turning the API on, so set the variable for it too.

### "OPTIMAL_AUTH_REQUIRED must be true or false"

A setting has a value the Engine does not accept, and the boot stops.
The message names the variable; fix its value.
The same applies to `OPTIMAL_EMBED_URL`, `OPTIMAL_SIZE_PROFILE`, `OPTIMAL_ENGINE_SCOPE`, and the other settings in [Install and configure](/docs/optimal-engine/install#settings).

### "migration ... failed and was rolled back"

An upgrade's migration failed.
The Engine will not open the store until it is fixed.
Restore the backup you took before the upgrade, then start the previous version; see [Operations](/docs/optimal-engine/operations#upgrades).

### The Docker container stays "unhealthy"

The health check calls `/api/health` with `OPTIMAL_ENGINE_API_KEY`, and keys are required in the image.
Set `OPTIMAL_ENGINE_API_KEY` on the container to a valid key.

## Health

`GET /api/health` reports `status` and the checks behind it.

| Warning or error | Fix |
|---|---|
| `store_unreachable` | The database file cannot be opened. Check `OPTIMAL_ENGINE_DB`, the disk, and file permissions. |
| `migrations_pending` | Restart the Engine so it applies them, or run `mix optimal.migrate`. |
| `connector_key_missing` | You set up a connector or a replication peer without `CONNECTOR_KEY`. Set it to 64 hex characters. |
| `auth_open_on_network` | The Engine is bound beyond loopback with keys off. Set `OPTIMAL_AUTH_REQUIRED=true`. |
| `embedder_unreachable`, `embedder_unauthorized`, `embedder_dimension_mismatch` | Shown with `?full=true`. Check `OLLAMA_HOST` or `OPTIMAL_EMBED_URL`, its key, and `OPTIMAL_EMBED_DIMENSIONS`. |

## Requests

### 401 `missing_api_key` on every call, including `/api/health`

Keys are required.
Send `Authorization: Bearer <key>`.
Mint the first key with `mix optimal.auth mint`, or set `OPTIMAL_BOOTSTRAP_KEY`.

### 403 `workspace_scope_denied`

The key does not cover that workspace.
Check its workspaces with `GET /api/auth/whoami`, and mint a key that includes the workspace.

### 400 `workspace_required`

The key is limited to named workspaces and the request named none.
Pass `workspace`, or `-w` in the CLI.

### Search returns nothing for something you just stored

- Use the same full workspace id, such as `default:notes`, in every call.
  The Engine does not look up a bare slug such as `notes`; it treats it as a different workspace id.
- A context package from `assemble` holds reviewed facts only; a new source shows up there after its claims are promoted.
  Use `search` to see sources right away.

### 503 `hybrid_unavailable`

`mode=hybrid` needs an embedding model, and none answered.
Use `mode=auto`, which falls back to full-text, or set up embeddings; see [Install and configure](/docs/optimal-engine/install#embeddings).

### 503 `busy` with `Retry-After: 1`

The store was busy, often right after boot or during a long write.
Retry after the delay.

### 429 `rate_limited`

The key used its budget for the minute.
Wait `retry_after_ms`, or raise the size profile or the key's `rate_limit_per_minute`; see [Operations](/docs/optimal-engine/operations#sizing).

### 426 `client_incompatible`

The `optimal` CLI or MCP server is of a different major version than the Engine.
Update the client from the same release as the Engine.

## On MIOSA

### "Memory could not be started"

Your organization does not have managed Engines enabled, or your role cannot create databases.
See [Availability](/docs/optimal-engine/miosa#availability).

### 422 `ENGINE_NOT_AVAILABLE`

Managed Engines are not enabled for your organization.
Contact support to have them turned on.

### 409 `ENGINE_NOT_RUNNING`

The Engine is stopped or still starting.
Start it from **Databases**, or wait about a minute after creating it.

### 404 `NOT_FOUND` from the engine-api proxy

The route is not one a client key may call, or the workspace has no Engine.
See the allowed routes in [Optimal Engine on MIOSA](/docs/optimal-engine/miosa#call-the-engine-api).
Key, backup, sync, and maintenance routes are managed through MIOSA instead.

### A client key stopped working

Client keys expire, after 24 hours by default.
Run `miosa engine connect <engine>` again, and restart the MCP server, which reads its key once at start.
