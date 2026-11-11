---
slug: "optimal/api"
title: "Optimal API"
description: "HTTP reference for Optimal - mint an Optimal token, create chats, send messages, stream replies, bind a machine, upload attachments, decide approvals, and read a chat's cost."
section: "Optimal"
order: 7010
---

The Optimal API is the same API the console uses.
Everything a chat in the console does starts from these endpoints, and chats you create here show up in the console's **History**.

| | |
|---|---|
| Base URL | `https://optimal.miosa.ai/api/v1` |
| Auth | `Authorization: Bearer <Optimal token>` |
| Body | JSON with `Content-Type: application/json`, except attachment uploads |
| Errors | `{"error": "<code>", "details": "<message>"}` |

## Authentication

Optimal does not take your API key directly.
Your API key mints a short-lived **Optimal token** on the MIOSA API, and every Optimal call uses that token.

The API key needs the **Chat with Optimal and start runs** permission (`osa:write`).
To approve or reject Optimal's changes with the token, the key also needs **Approve or reject Optimal's changes** (`osa:approve`).
A key bound to one workspace can mint a token for that workspace only.

### Mint a workspace token

`POST https://api.miosa.ai/api/v1/optimal/workspace-token`

```bash
curl -s https://api.miosa.ai/api/v1/optimal/workspace-token \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workspace_id": "your_workspace_id"}'
```

```json
{
  "token": "eyJhbGciOi...",
  "expires_at": 1792771200,
  "can_decide_approvals": false,
  "scope": {
    "user_id": "...",
    "tenant_id": "...",
    "workspace_id": "your_workspace_id",
    "workspace_role": "admin"
  }
}
```

- The token lasts 10 minutes; `expires_at` is a Unix time in seconds.
  Mint a new one when a call answers `401`.
- Chats belong to the key's user and to the workspace in the token.

| Status | Code | When |
|---|---|---|
| 401 | `UNAUTHORIZED` | The API key is missing or revoked |
| 403 | `FORBIDDEN` | The key does not have `osa:write` |
| 403 | `WORKSPACE_FORBIDDEN` | The key's user is not a member of the workspace |
| 403 | `API_KEY_WORKSPACE_BOUND` | The key is bound to a different workspace |
| 403 | `API_KEY_NOT_ELIGIBLE` | The key is bound to a project or an end user |
| 422 | `WORKSPACE_REQUIRED` | `workspace_id` is missing |

### Mint an organization token

`POST https://api.miosa.ai/api/v1/optimal/org-token` takes no body.
It mints a token for an **All workspaces** chat, which reads every workspace the user belongs to and asks which one to use before changing anything.
A key bound to one workspace cannot mint it.

## Chats

A chat is called a session in the API.

### Create a chat

`POST /sessions`

```bash
curl -s https://optimal.miosa.ai/api/v1/sessions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"resume": false}'
```

```json
{
  "id": "your_chat_id",
  "status": "created",
  "machine": null,
  "workspace": { "id": "your_workspace_id", "rule": "This chat belongs to the workspace it was started in and never moves; its machines, memory and spend are that workspace's." },
  "pending_workspace": null,
  "turn": { "state": "idle", "turn_id": null, "started_at": null, "last_event_seq": 0 }
}
```

| Field | Type | Description |
|---|---|---|
| `resume` | boolean | Send `false` to always start a fresh chat |
| `client_key` | string | Your own key for this chat; creating again with the same key resumes it (`status: "resumed"`) |
| `model` | string | Optional; the only value is `optimal` |

### List chats

`GET /sessions`

| Query | Description |
|---|---|
| `page`, `per_page` | Pagination; `per_page` is 20 by default, at most 100 |
| `view` | `mine` (default), `team` for shared chats, or `members` for owners and admins |

```json
{
  "sessions": [
    { "id": "your_chat_id", "title": "Landing page", "message_count": 6, "created_at": "2026-10-23T14:02:11Z", "visibility": "personal" }
  ],
  "count": 1,
  "page": 1,
  "per_page": 20
}
```

### Rename or share a chat

`PATCH /sessions/{id}` with any of:

| Field | Values |
|---|---|
| `title` | Text |
| `mode` | `ask`, `build`, `investigate` |
| `visibility` | `personal`, `team` |

### Delete a chat

`DELETE /sessions/{id}` deletes the chat and its messages for good, and answers `{"status": "deleted", "session_id": "..."}`.
Machines the chat used are not touched.

## Messages

### Send a message

`POST /sessions/{id}/message`

```bash
curl -s https://optimal.miosa.ai/api/v1/sessions/$CHAT/message \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Why did my latest deployment fail?", "mode": "investigate", "client_message_id": "msg-001"}'
```

The call returns at once with `202`; the reply arrives on the [stream](/docs/optimal/api#streaming).

```json
{
  "status": "processing",
  "session_id": "your_chat_id",
  "turn_id": "msg-001",
  "client_message_id": "msg-001",
  "queued": false
}
```

| Field | Type | Description |
|---|---|---|
| `message` | string | Required. The text to send |
| `mode` | string | `ask`, `build`, or `investigate` |
| `client_message_id` | string | 1 to 128 characters of letters, digits, `_`, `.`, `:`, or `-`. Sending the same id again answers `200` with `status: "duplicate"` instead of a second turn |
| `machine` | object | `{"resource_type": "sandbox" \| "computer", "resource_id": "..."}`: run this message on that machine |
| `confirm_switch` | boolean | `true` to move a chat that is already working on another machine |
| `attachments` | array | Attachment ids from [Attachments](/docs/optimal/api#attachments) |

A message sent while Optimal is working waits behind the current turn, and `queued` is `true`.
`DELETE /sessions/{id}/queued/{client_message_id}` removes it until it starts; after that the call answers `409 not_queued`.

### Read the transcript

`GET /sessions/{id}/messages`

```json
{
  "messages": [
    { "role": "user", "content": "Why did my latest deployment fail?", "timestamp": "2026-10-23T14:02:11Z" },
    { "role": "assistant", "content": "The build failed at npm install...", "timestamp": "2026-10-23T14:02:40Z" }
  ],
  "count": 2,
  "queued": [],
  "owner": { "id": "...", "label": "Ada" }
}
```

### Stop a reply

`POST /sessions/{id}/cancel` answers `{"status": "cancel_requested"}`.
When nothing is running it answers `404 not_running`.

## Streaming

`GET /sessions/{id}/stream` is a Server-Sent Events stream.

```bash
curl -N https://optimal.miosa.ai/api/v1/sessions/$CHAT/stream \
  -H "Authorization: Bearer $TOKEN"
```

- A browser `EventSource` cannot set headers, so this endpoint also takes the token as `?token=`.
  No other endpoint accepts a token in the URL.
- Every event has a numbered `id:`.
  Reconnect with the `Last-Event-ID` header, or `?last_event_id=`, to replay what you missed.
- A new reader with no id gets the turn in flight from its start; finished turns come from `GET /sessions/{id}/messages`.
- If the events you missed are gone, the stream sends `replay_gap`; read the transcript, then keep streaming.

The main events:

| Event | Data | Meaning |
|---|---|---|
| `connected` | `session_id` | The stream is open |
| `turn_start` | `turn_id` | Optimal started on a message |
| `streaming_token` | `text`, `message_id` | The next piece of the reply |
| `tool_call` | `name`, `phase` (`start` or `end`) | Optimal is using a tool |
| `permission_required` | the step that waits | A step needs approval; see [Approvals](/docs/optimal/api#approvals) |
| `agent_response` | `response` | The finished reply |
| `turn_billed` | `turn_credits`, `session_credits`, `balance` | What the turn cost, once the ledger settles |
| `replay_gap` | `session_id`, `last_seq` | Events you asked to replay are gone |

Ignore events you do not use; new events can be added.

## Machines

A chat can work on one sandbox or computer in your workspace at a time.

`POST /sessions/{id}/machine`

```json
{ "resource_type": "sandbox", "resource_id": "your_sandbox_id" }
```

Answers `{"session_id": "...", "machine": {"resource_type": "sandbox", "resource_id": "..."}}`.
`DELETE /sessions/{id}/machine` unbinds it; the machine keeps running.

| Status | Code | When |
|---|---|---|
| 400 | `invalid_machine` | `resource_type` is not `sandbox` or `computer`, or `resource_id` is missing |
| 404 | `machine_not_found` | The machine is not in this workspace |
| 409 | `machine_unavailable` | The machine is not running or cannot be reached |
| 409 | `machine_switch_requires_confirmation` | The chat is working on another machine; send `confirm_switch: true` |

## Attachments

`POST /sessions/{id}/attachments?name={file_name}` takes the raw file bytes as the body.

```bash
curl -s "https://optimal.miosa.ai/api/v1/sessions/$CHAT/attachments?name=error.log" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/octet-stream" \
  --data-binary @error.log
```

```json
{ "id": "att_...", "name": "error.log", "mime": "text/plain", "bytes": 2048, "path": "/workspace/uploads/error.log", "vision": false }
```

Pass the `id` in the next message's `attachments`.
Files are up to 10 MB each and 25 MB per chat.
A file that is neither an image nor a document needs a machine bound to the chat, or the call answers `409 no_machine`.

## Approvals

`GET /approvals` lists approvals; filter with `session_id` and `status` (`pending`, `approved`, `rejected`).

```json
{ "approvals": [ { "id": "...", "status": "pending", "session_id": "your_chat_id", "title": "Delete sandbox api-old" } ], "count": 1 }
```

- `POST /approvals/{id}/approve` approves; `POST /approvals/{id}/reject` declines.
  Both take an optional `reason`.
- A token whose `can_decide_approvals` is `false` gets `403 approval_scope_required`.
- Deciding twice answers `409 already_decided`.

See [Approvals](/docs/optimal/approvals).

## Models

`GET /models` returns one model, `optimal`, with its live price:

```json
{
  "models": [
    {
      "id": "optimal",
      "name": "Optimal",
      "provider": "MIOSA",
      "context_window": 1048576,
      "default": true,
      "credits_per_1m": { "input": 150, "output": 600 },
      "pricing": {
        "unit": "wallet credits (1 credit = 1 cent)",
        "input_credits_per_1m": 150,
        "output_credits_per_1m": 600
      }
    }
  ],
  "current": "optimal",
  "default": "optimal",
  "currency": "wallet credits (1 credit = 1 cent)"
}
```

Any other `model` value answers `400 MODEL_NOT_ALLOWED`.

## A chat's cost

`GET https://api.miosa.ai/api/v1/credits/session/{chat_id}` is on the MIOSA API and takes your API key.

```json
{
  "session_id": "your_chat_id",
  "model_credits": 12,
  "machine_credits": 7,
  "total_credits": 19,
  "attributed": true,
  "settled": true,
  "as_of": "2026-10-23T14:10:00Z"
}
```

## Errors

| Status | Code | Meaning |
|---|---|---|
| 400 | `missing or empty message` | `message` is empty |
| 401 | `MISSING_TOKEN`, or an expired token | Mint a new Optimal token |
| 402 | `OPTIMAL_CREDITS_REQUIRED` | No credits left; add credits or upgrade |
| 403 | `ownership_denied` | The machine is not in your organization |
| 404 | `session_not_found` | The chat does not exist, or is not yours |
| 415 | `unsupported_media_type` | A JSON body without `Content-Type: application/json` |
| 429 | `rate_limited` | Too many requests; wait for `Retry-After` |
| 429 | `session_ceiling` | Too many chats running at once for your organization |

A chat in another organization or workspace answers `404`, the same as one that does not exist.

> **Note: Only what a chat needs**
>
> The Optimal token reaches chats, history, artifacts, investigations, approvals, cost, and settings.
> It cannot run tools directly or reach MIOSA's own servers.
> See [Isolation and permissions](/docs/optimal/isolation).
