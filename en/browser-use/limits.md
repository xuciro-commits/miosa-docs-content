---
slug: "browser-use/limits"
title: "Browser use limits, billing, and security"
description: "Availability and size, lifecycle across pause and resume, how browser use is billed, and how access is enforced."
section: "Sandboxes"
order: 2029
---

## Availability

Browser use is available on every running sandbox, on any template.
One exception today: a SOMA sandbox ships Docker and Chromium but not the virtual display the live viewer renders, so the live viewer is not available on SOMA yet.
The browser starts lazily on the first `POST /browser`, so sandboxes that never use it pay nothing and boot as fast as before.

The sandbox image must include a Chromium-family browser and Node.
Every MIOSA sandbox image ships them; a custom image that removed them returns `501 BROWSER_NOT_AVAILABLE`.

No template publish is needed to use the browser: the daemon ships with the control plane and is written into the sandbox on start, on every launchable template.

## Size

A real Chromium needs room. `small` (2 vCPU, 4 GB) or larger is recommended for a comfortable browser.

| Size | vCPU | RAM | Browser use |
|---|---|---|---|
| `xs` | 1 | 2 GB | Tight; a single light page at most |
| `small` | 2 | 4 GB | **Recommended minimum** |
| `medium` | 4 | 8 GB | Comfortable |
| `large` | 8 | 16 GB | Heavy pages and parallel tabs |

## Lifecycle

| Action | Effect on the browser |
|---|---|
| `POST /browser` | Starts the browser lazily (idempotent) |
| Pause sandbox | Browser state is preserved; the session survives resume |
| Resume sandbox | The same browser comes back, with its tabs and logins |
| `DELETE /browser` | Stops the browser and frees its memory; the sandbox keeps running |
| Destroy sandbox | Removes everything, including browser state |

Pause and resume keep the browser; stopping the browser is what releases its memory.

## Saved website sign-ins

The platform stores a person's saved website sign-ins so the sandbox browser can fill them without the agent ever seeing the password.
`GET /api/v1/signins` lists them, `PUT /api/v1/signins/{site}` saves one, and `DELETE /api/v1/signins/{site}` removes one.
Send `workspace_id`; a request without it answers `422 WORKSPACE_REQUIRED`.
Responses are masked: they carry a username preview and `last_used_at`, never the password.
`POST /api/v1/sandboxes/{id}/browser/signin` with `{"site": "..."}` fills the saved sign-in into the open tab, only on your own running sandbox and only on an HTTPS page of that site.

The browser is also a process inside the sandbox, so if the sandbox is restarted the browser must be started again. `GET /browser` reports the current state (`running`, `stopped`, `unavailable`, or `sandbox_not_running`) without starting anything.

## Billing

Browser use bills as **plain sandbox usage**.
There is no separate browser line item: while the sandbox is running, you pay the sandbox rate you already pay for that size, and the browser runs inside it.
Stopping the browser frees memory but does not by itself stop the sandbox.

## Security

- Nothing is exposed publicly. Both sockets - the live-view stream and the CDP endpoint - go through `api.miosa.ai` with normal API authentication, never through the sandbox's own host.
- The control plane connects to the browser daemon on the sandbox's loopback only, and hands it a 60-second token each time. Callers never see that token.
- An **API key on a socket needs `sandboxes:exec`**, because a browser can reach anything the sandbox can.
- Share tokens are **view-only**. Their input is refused inside the sandbox's browser process, not merely hidden in the UI.
- Share links are **revocable**: revoke one link by its `share_id`, or revoke every share and CDP link at once. Revoking bumps the browser's share epoch, so an outstanding link stops working immediately.
- Live-view and CDP sockets share a per-tenant stream limit; exceeding it closes the socket with `4013`.

> **Caution: Never expose the API key**
>
> Mint viewer and CDP access on your backend.
> Send only the short-lived `stream_auth` token or a share link to the browser or the agent.
> Never ship a MIOSA API key to the frontend.

## Next

- [Quickstart](/docs/browser-use/quickstart) - start and stop a browser.
- [Live viewer](/docs/browser-use/live-viewer) - take control and share.
- [Drive it over CDP](/docs/browser-use/cdp) - connect an agent.
