---
slug: "browser-use/quickstart"
title: "Browser use quickstart"
description: "Start a live browser in any MIOSA sandbox, read its stream and CDP endpoints, and stop it when you are done."
section: "Sandboxes"
order: 2025
---

Browser use runs inside a sandbox, on every template, with no extra setup.
You create a sandbox as usual, then start its browser.

The browser starts lazily on the first `POST`. Sandboxes that never use a browser pay nothing, and sandbox boot stays as fast as it is today.

## 1. Start a sandbox

Any sandbox works, on any template. Nothing about the browser is chosen at create time.
The live viewer needs the sandbox's virtual display, which a SOMA sandbox does not have yet.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "template_id": "miosa-sandbox", "size": "small" }'
```

Wait until the sandbox `state` is `running`.

## 2. Start the browser

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

The response carries everything you need:

```json
{
  "data": {
    "state": "running",
    "epoch": 0,
    "viewport": { "width": 1280, "height": 720 },
    "stream_url": "wss://api.miosa.ai/api/v1/sandboxes/sbx_01j9x/browser/stream",
    "cdp_url": "wss://api.miosa.ai/api/v1/sandboxes/sbx_01j9x/browser/cdp",
    "stream_auth": "miosa_stream_...",
    "stream_auth_expires_at": 1783000000
  }
}
```

| Field | Meaning |
|---|---|
| `state` | `running` when the browser is up. |
| `epoch` | Share-link epoch. Bumping it revokes every outstanding share link. |
| `viewport` | Browser viewport, `1280` by `720`. |
| `stream_url` | WebSocket for the live view (`wss://api.miosa.ai/...`). |
| `cdp_url` | WebSocket Chrome DevTools Protocol endpoint for agents. |
| `stream_auth` | A short-lived token for the viewer or CDP socket. |
| `stream_auth_expires_at` | Unix seconds when `stream_auth` expires (1 hour). |

`POST` is idempotent: calling it again on a running browser returns the same endpoints without restarting it.

> **Note: Scope**
>
> `GET` needs the `sandboxes:read` scope.
> `POST` and `DELETE` need `sandboxes:write`.
> The WebSocket endpoints need `sandboxes:exec` when you authenticate with an API key.

## 3. Open the viewer

In the MIOSA console, open the sandbox and use its browser panel.
To open the viewer yourself, connect to `stream_url` and authenticate.

```bash
# The viewer protocol is a WebSocket: JPEG frames and JSON events down, JSON input up.
# Pass the token as a query parameter...
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?token=$STREAM_AUTH
# ...or as a header: Authorization: Bearer $STREAM_AUTH
```

From there a viewer can **take control** of the browser or share a **view-only link**.
See [Live viewer](/docs/browser-use/live-viewer).

## 4. Let an agent drive it

The browser a person is watching is the same browser an agent connects to over CDP.
Connect to `cdp_url` with Playwright or Puppeteer - see [Drive it over CDP](/docs/browser-use/cdp).

## 5. Check the state without starting it

`GET` only reports state; it never starts the browser.

```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{
  "data": { "state": "running", "epoch": 0, "viewport": { "width": 1280, "height": 720 } }
}
```

For a sandbox that is not running, `state` is `sandbox_not_running` and no guest is touched:

```json
{ "data": { "state": "sandbox_not_running", "epoch": 0 } }
```

## 6. Stop the browser

```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{
  "data": { "state": "stopped", "epoch": 0, "viewport": { "width": 1280, "height": 720 } }
}
```

Stopping the browser frees its memory while the sandbox keeps running.
The browser survives **pause and resume** of the sandbox; stopping it is what releases the memory.

## Errors

| Status | Code | Cause |
|---|---|---|
| `400` | `INVALID_ID` | The sandbox id is not a UUID |
| `404` | `NOT_FOUND` | Unknown sandbox, or one owned by another tenant |
| `401`/`403` | - | Missing or insufficient credentials |
| `409` | `SANDBOX_NOT_RUNNING` | The sandbox must be running to start or stop the browser |
| `501` | `BROWSER_NOT_AVAILABLE` | The sandbox image has no Chromium or Node |
| `502` | `BROWSER_START_FAILED` | The browser daemon did not come up in time |

`501` names a template that ships the browser prerequisites: the sandbox image includes Chromium and Node.

## Next

- [Live viewer](/docs/browser-use/live-viewer) - take control, share, full screen.
- [Drive it over CDP](/docs/browser-use/cdp) - Playwright and Puppeteer.
- [Limits, billing, security](/docs/browser-use/limits) - minimum size and enforcement.
