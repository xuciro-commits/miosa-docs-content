---
slug: "browser-use/live-viewer"
title: "Browser use live viewer"
description: "Watch a sandbox browser live, take control, share a revocable view-only link, and go full screen."
section: "Sandboxes"
order: 2026
---

The live viewer renders the sandbox browser in a page you can watch.
It needs the sandbox's virtual display (`Xvfb :99`), which ships in the sandbox image, and which a SOMA sandbox does not have yet.
It lives on the Sandbox page in the MIOSA console, and the same protocol serves any client you build.

The viewer is a WebSocket at `wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/stream`: JPEG frames and JSON events come down, JSON input goes up.
It goes through `api.miosa.ai`, never the sandbox's own host.

## Three ways to open it

| Surface | How | Capability |
|---|---|---|
| Sandbox page | Open the sandbox, then its browser panel | Take control |
| Stream URL | `stream_url` from the browser API, with `stream_auth` | Take control |
| View-only share link | Minted for someone else | Watch only |

## Authenticate the stream

Two ways, both on `api.miosa.ai`:

- `Authorization: Bearer <msk_* API key | session JWT>`, or
- `?token=<stream_auth>` for clients that cannot set headers (the `stream_auth` value from `POST`).

API keys used here need the `sandboxes:exec` scope.
A viewer that only watches can pass `?mode=view`, which reaches the view-only listener in the sandbox.

## Take control

A viewer opened on the Sandbox page, or with a control-mode stream, can take control: click, type, scroll, and navigate the live page.
Input goes straight to the browser process, so the page the agent is driving is the page you are steering.

## Share a view-only link

Mint a share link to let someone watch without touching the browser.
No MIOSA login is required to use it.

```bash
curl -X POST "https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/share?ttl_seconds=600" \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{
  "data": {
    "mode": "view",
    "share_token": "bs1.1783000000.0.9f3c...",
    "expires_at": "2026-10-10T13:00:00Z",
    "stream_url": "wss://api.miosa.ai/api/v1/sandboxes/sbx_01j9x/browser/stream"
  }
}
```

| Field | Meaning |
|---|---|
| `mode` | Always `view`. |
| `share_token` | The view-only token; pass it as `?share=<share_token>` on `stream_url`. |
| `expires_at` | When the link stops working. |
| `stream_url` | The stream WebSocket the link connects to. |

`ttl_seconds` is optional: it defaults to `3600` and must be between `60` and `604800` (7 days).
Out of range is `400 INVALID_TTL`.

The share link opens the same stream as a view-only viewer:

```text
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?share=$SHARE_TOKEN
```

A view-only link is **revocable**: revoking it stops every outstanding link for that browser immediately.

```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/share \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{ "data": { "revoked": true, "epoch": 1 } }
```

Revocation bumps the sandbox's share `epoch`; every token minted at an earlier epoch is rejected from then on.

> **Caution: A view-only link cannot drive**
>
> View-only is enforced inside the sandbox's browser process, not in the page. A view-only viewer receives frames; its input is refused on the machine, so it can never type, click, or navigate - whatever mode it asks for.

## Full screen

The viewer has a full-screen control that expands the browser to the whole screen for closer inspection or a kiosk-style display.
Toggle it off to return to the embedded frame.

## Next

- [Drive it over CDP](/docs/browser-use/cdp) - connect an agent to the same browser.
- [Limits, billing, security](/docs/browser-use/limits) - what the viewer can and cannot do.
