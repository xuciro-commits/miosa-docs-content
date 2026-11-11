---
slug: "computer-use/api-reference"
title: "Computer use API reference"
description: "Every MIOSA Computer desktop control endpoint - screenshot, mouse, keyboard, scroll, clipboard, windows, accessibility tree, and the one-shot window-state read."
section: "Computers"
order: 3009
---

Desktop control endpoints proxy authenticated commands to the agent inside the Computer.
All of them require the Computer to be `running`; a stopped or provisioning Computer returns `409 COMPUTER_NOT_RUNNING`.

Base path: `/api/v1/computers/{id}/desktop`.

> **Note: Two hosts, one contract**
>
> Full surface is on `api.miosa.ai`.
> The narrower developer host `sandboxes.miosa.ai` exposes the read and common-action endpoints only: screenshot, screenshot/region, click, type, key, scroll, drag, double-click, wait, windows, cursor, screen-size, accessibility-tree, window/focus, launch, and window-state.
> The low-level endpoints listed below - hotkey, key-down/key-up, mouse-down/mouse-up, move, clipboard, wallpaper, environment, and per-window geometry - are on `api.miosa.ai`.

## Observation

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/screenshot` | Full desktop as PNG (`Content-Type: image/png`) |
| `POST` | `/screenshot/region` | Rectangular region as PNG; body `{ x, y, width, height }` |
| `GET` | `/windows` | Open windows: `id`, `title`, `class`, `x`, `y`, `width`, `height`, `focused` |
| `GET` | `/cursor` | Cursor position `{ x, y }` in screen pixels |
| `GET` | `/screen-size` | Display resolution `{ width, height }` |
| `GET` | `/environment` | Desktop environment name and version (for example `xfce4`) |
| `GET` | `/accessibility-tree` | AT-SPI element tree as nested JSON |
| `GET` | `/window-state` | Screenshot **and** accessibility tree in one round trip |

### `GET /desktop/window-state`

Runs the screenshot and the accessibility tree in parallel and returns them together from the same moment, so an agent pays one round trip instead of two.

```json
{
  "data": {
    "screenshot": {
      "content_type": "image/png",
      "encoding": "base64",
      "bytes": 214733,
      "data": "iVBORw0KGgo..."
    },
    "accessibility_tree": { "role": "frame", "name": "Firefox", "children": [ ] },
    "captured_at": "2026-10-10T12:00:00.000000Z"
  }
}
```

A part that fails does not fail the request: it is `null` and named under an `errors` object, so the caller can still use the other half.
Only when both parts fail is the request a `502 AGENT_UNAVAILABLE`.

## Mouse

| Method | Path | Body |
|---|---|---|
| `POST` | `/click` | `x`, `y`, `button` (`left`\|`right`\|`middle`, default `left`) |
| `POST` | `/double-click` | `x`, `y` |
| `POST` | `/move` | `x`, `y` (move the pointer, no click) |
| `POST` | `/drag` | `from_x`, `from_y`, `to_x`, `to_y` |
| `POST` | `/mouse-down` | `x`, `y`, `button` (default `left`) |
| `POST` | `/mouse-up` | `x`, `y`, `button` (default `left`) |
| `POST` | `/scroll` | `x`, `y`, `direction` (`up`\|`down`\|`left`\|`right`), `clicks` |

> **Caution: mouse-down / mouse-up naming gap**
>
> The public endpoints are `/desktop/mouse-down` and `/desktop/mouse-up`.
> Inside the Computer they map to the lower-level actions named `mouse-press` and `mouse-release`.
> The names differ between the public contract and the on-machine action; the pair-and-release behavior is the same.
> For a one-shot click, use `POST /desktop/click` - do not assume the request field is named `press`.

`mouse-down` / `mouse-up` hold and release a button with no click between them, so you can move the pointer mid-drag (`move` then `mouse-up`) or implement a long press.

## Keyboard

| Method | Path | Body |
|---|---|---|
| `POST` | `/type` | `text`, optional `delay` (ms between keys) |
| `POST` | `/key` | `key` - a single key or a `+` chord, for example `Return` or `ctrl+a` |
| `POST` | `/hotkey` | `keys` - a simultaneous combo, for example `["ctrl", "shift", "t"]` |
| `POST` | `/key-down` | `key` - press and hold |
| `POST` | `/key-up` | `key` - release |

Key names follow the X11 keysym convention: `Return`, `Tab`, `BackSpace`, `Delete`, `Home`, `End`, `Page_Up`, `Page_Down`, `Up`, `Down`, `Left`, `Right`, `F1`-`F12`, and the modifiers `super`, `ctrl`, `alt`, `shift`.
`type` types literal text and does not interpret key names; use `key` for those.

## Clipboard, windows, and desktop

| Method | Path | Body |
|---|---|---|
| `GET` | `/clipboard` | Read clipboard text |
| `POST` | `/clipboard` | `text` - write clipboard text |
| `POST` | `/window/focus` | `window_id` |
| `POST` | `/launch` | `command` - launch an app, runs in the background |
| `GET` | `/window/{window_id}/size` | Read a window's `{ width, height }` |
| `GET` | `/window/{window_id}/position` | Read a window's `{ x, y }` |
| `POST` | `/window/{window_id}/resize` | `width`, `height` |
| `POST` | `/window/{window_id}/move` | `x`, `y` |
| `POST` | `/window/{window_id}/maximize` | - |
| `POST` | `/window/{window_id}/minimize` | - |
| `POST` | `/window/{window_id}/close` | - |
| `POST` | `/wallpaper` | `path` - set the background from a file inside the VM |

## Timing

| Method | Path | Body |
|---|---|---|
| `POST` | `/wait` | `seconds` - sleep inside the Computer, capped at 30, to let UI animations settle |

## Related endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/v1/computers/{id}/screenshot` | Alias for the full-desktop screenshot |
| `GET` | `/api/v1/computers/{id}/urls` | Signed desktop, VNC, terminal, and entry URLs |
| `POST` | `/api/v1/computers/{id}/stream-token` | Mint a short-lived viewer stream token |
| `GET` | `/api/v1/computers/{id}/vnc-credentials` | VNC credentials for the desktop |
| `GET` | `/api/v1/computers/{id}/viewer-password` | Read the raw external viewer password |
| `POST` | `/api/v1/computers/{id}/viewer-password/rotate` | Rotate the raw external viewer password |

## Coordinates

Coordinates are screen pixels with the origin at the top-left, matching what `screen-size` and `cursor` report.
A vision model may be shown a scaled-down screenshot; map its output back to screen pixels before sending an action.

## Errors

| Status | Code | Cause |
|---|---|---|
| `409` | `COMPUTER_NOT_RUNNING` | The Computer is stopped, provisioning, or recovering |
| `502` | `AGENT_UNAVAILABLE` | The agent inside the Computer did not answer |
| `429` | - | Rate limited; honor `X-RateLimit-*` and `Retry-After` |

See also: [Desktop API](/docs/api-reference/desktop) for the field-by-field request and response shapes.
