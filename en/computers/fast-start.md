---
slug: "computers/fast-start"
title: "Fast start & readiness"
description: "How MIOSA starts managed desktop Computers quickly, defines browser readiness, and exposes failures without false Live states."
section: "Computers"
order: 3003
---

MIOSA Computers use ready capacity on the managed fleet so a new desktop can be assigned without waiting for a full cold boot.
The product target is simple: click **Launch**, then receive an interactive desktop in about 800 to 900 ms on the warm path.

## Verified result

The production canary measured on 2026-08-06 completed **10 of 10** launches and reached a browser-ready desktop in a median **740 ms**.

| Phase | Time |
|---|---:|
| Successful launches | `10 / 10` |
| Create response median | `348 ms` |
| Create request to browser-ready desktop median | **`740 ms`** |
| Create request to browser-ready desktop p90 | **`809 ms`** |
| Slowest sample | `942 ms` |

Every canary used the warm path and did not require the restart fallback.
The browser used the bundled direct noVNC client, and `Live` required its real RFB connection event.
The launch path warmed the future viewer transport concurrently with the create request and retained bounded retry for route-registration races.

> **Note: How to read this number**
>
> The 10-sample result is a dated production canary, not a universal latency guarantee.
> Region, fleet capacity, network latency, image choice, and cold-path fallback can change startup time.

## What browser-ready means

MIOSA does not call a Computer ready merely because its database record exists or its lifecycle says `Running`.
Browser readiness requires the complete viewing path to work.

1. The create request claims ready desktop capacity.
2. The stream API returns a short-lived authenticated viewer URL.
3. The bundled viewer client connects directly to the authenticated VNC WebSocket.
4. The VNC WebSocket upgrades with the stream token.
5. The browser renders a usable desktop frame.
6. Opening the Computer workspace keeps the desktop mounted and connects its terminal.

The managed fleet keeps separate reserves on every host: 20 ready desktop slots and 20 ready sandbox slots.
Desktop slots enter the ready pool only after the default MIOSA wallpaper is visible.
Tenant wallpaper and desktop branding are applied during claim, before the Computer is handed to the user.

- **No false Live state**: A browser error document or failed iframe navigation must not be labeled as a live desktop.
- **Authenticated handoff**: The stream URL is short-lived and scoped to the selected Computer.
- **Workspace continuity**: The gallery-to-workspace transition preserves the viewer handoff instead of requesting a conflicting session.

## Application integration

Request the Computer stream URL from your backend and pass only the short-lived URL to the browser.
Embed the authenticated viewer URL returned by the API instead of rebuilding the desktop URL from the Computer slug.

Your UI should represent these states separately:

| State | Meaning | Recommended UI |
|---|---|---|
| `provisioning` | Capacity is being assigned or the guest is becoming ready | Show bounded progress |
| `running` with viewer ready | Desktop and stream path are usable | Show `Live` and reveal the viewer |
| `paused` | Durable state exists but the active desktop is stopped | Offer Resume |
| viewer unavailable | Lifecycle and viewer reachability disagree | Show an actionable error and retry control |
| failed | Provisioning or startup failed | Show the backend error and recovery action |

Do not promote an iframe to `Live` from its generic `load` event alone.
Browsers also fire that event for their own network-error documents.

## Troubleshooting

### The card says Running but the viewer says Host unavailable

Treat viewer reachability as authoritative for the user-visible state.
Refresh the Computer lifecycle, then retry the stream request.
If the mismatch persists, contact support with the Computer ID, region, and approximate creation time.

### The viewer page returns 503

The public viewer gateway cannot reach the active compute proxy or assigned host.
This is a platform routing failure rather than proof that your files or Computer record were lost.
Retry once, then contact support if it persists.

### The stream request returns 409

The Computer is not currently in a streamable lifecycle state.
Refresh its status and resume it when the API reports `paused`.
Do not retry indefinitely without re-reading the lifecycle.

### The iframe loads but no desktop appears

Confirm that you embedded the authenticated viewer URL returned by the API.
Do not substitute a raw desktop entry URL or wait for an undocumented cross-frame event.

## Related documentation

- [Computers overview](/docs/computers/overview)
- [Embedding & Streaming](/docs/computers/embedding)
- [Computer use](/docs/computer-use)
- [Computers API](/docs/api-reference/computers)
