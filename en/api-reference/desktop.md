---
slug: "api-reference/desktop"
title: "Desktop API"
description: "API reference for all desktop control endpoints - screenshot, mouse, keyboard, clipboard, windows, accessibility tree."
section: "API reference"
order: 12030
---

Desktop control endpoints proxy commands to the computer command service inside the computer VM via the MIOSA control plane. All coordinates are in screen pixels (0,0 = top-left).

Base path: `/api/v1/computers/{id}/desktop`

> **Caution**
>
> All desktop endpoints require the computer to be in `"running"` status. Returns `409 COMPUTER_NOT_RUNNING` if the computer is stopped or provisioning.

> **Caution**
>
> Rate limits vary by key, tenant policy, and endpoint family. Honor `X-RateLimit-*` and `Retry-After` response headers. Individual desktop actions are fast but high-frequency AI agents should batch operations where possible.

---

## Screenshot

**`GET /api/v1/computers/{id}/desktop/screenshot`**

Captures the full desktop as a PNG image.

### Response  -  `200 OK`

Binary PNG data with `Content-Type: image/png`.
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/desktop/screenshot \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output screenshot.png
```

---

## Screenshot Region

**`POST /api/v1/computers/{id}/desktop/screenshot/region`**

Captures a rectangular region of the screen.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | Left edge X coordinate |
| `y` | integer | Yes | Top edge Y coordinate |
| `width` | integer | Yes | Width in pixels |
| `height` | integer | Yes | Height in pixels |

### Response  -  `200 OK`

Binary PNG data.
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/screenshot/region \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x": 0, "y": 0, "width": 800, "height": 600}' \
  --output region.png
```

---

## Click

**`POST /api/v1/computers/{id}/desktop/click`**

Performs a mouse click at the specified coordinates.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | X coordinate |
| `y` | integer | Yes | Y coordinate |
| `button` | string | No | `"left"` (default), `"right"`, `"middle"` |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/click \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300}'
```

---

## Double-Click

**`POST /api/v1/computers/{id}/desktop/double-click`**

Performs a double-click at the specified coordinates.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | X coordinate |
| `y` | integer | Yes | Y coordinate |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/double-click \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300}'
```

---

## Type Text

**`POST /api/v1/computers/{id}/desktop/type`**

Types a string of text as keyboard input.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | Yes | Text to type |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/type \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello, World!"}'
```

---

## Key Press

**`POST /api/v1/computers/{id}/desktop/key`**

Presses a key or key combination.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | Yes | Key name or combination |

Key combinations use `+` as separator: `"ctrl+c"`, `"alt+Tab"`, `"ctrl+shift+t"`.

### Common Keys

| Key | Value |
|-----|-------|
| Enter | `Return` |
| Tab | `Tab` |
| Escape | `Escape` |
| Backspace | `BackSpace` |
| Delete | `Delete` |
| Space | `space` |
| Arrows | `Up`, `Down`, `Left`, `Right` |
| Function | `F1`...`F12` |
| Home/End | `Home`, `End` |
| Page Up/Down | `Page_Up`, `Page_Down` |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/key \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"key": "ctrl+c"}'
```

---

## Scroll

**`POST /api/v1/computers/{id}/desktop/scroll`**

Scrolls the mouse wheel at the specified position.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | X coordinate |
| `y` | integer | Yes | Y coordinate |
| `direction` | string | Yes | `"up"` or `"down"` |
| `amount` | integer | No | Number of scroll steps (default: 3) |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/scroll \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x": 500, "y": 300, "direction": "down", "amount": 5}'
```

---

## Drag

**`POST /api/v1/computers/{id}/desktop/drag`**

Performs a click-and-drag from one point to another.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `start_x` | integer | Yes | Starting X coordinate |
| `start_y` | integer | Yes | Starting Y coordinate |
| `end_x` | integer | Yes | Ending X coordinate |
| `end_y` | integer | Yes | Ending Y coordinate |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/drag \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"start_x": 100, "start_y": 200, "end_x": 400, "end_y": 500}'
```

---

## Wait

**`POST /api/v1/computers/{id}/desktop/wait`**

Server-side pause. Holds the connection for the specified duration before responding.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `seconds` | number | No | Seconds to wait (default: 1, max: 30) |

### Response  -  `200 OK`
```json
{
  "success": true,
  "waited_seconds": 2
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/wait \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"seconds": 2}'
```

---

## List Windows

**`GET /api/v1/computers/{id}/desktop/windows`**

Returns a list of open windows on the desktop.

### Response  -  `200 OK`
```json
{
  "windows": [
    {
      "id": "0x2200003",
      "title": "Terminal - user@computer",
      "class": "Xfce4-terminal",
      "x": 100,
      "y": 50,
      "width": 800,
      "height": 600
    }
  ]
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/desktop/windows \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Focus Window

**`POST /api/v1/computers/{id}/desktop/window/focus`**

Brings a window to the foreground.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `window_id` | string | Yes | Window ID from the list windows response |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/window/focus \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"window_id": "0x2200003"}'
```

---

## Launch Application

**`POST /api/v1/computers/{id}/desktop/launch`**

Launches an application by name or command.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `app` | string | Yes | Application name or command (e.g., `"firefox"`, `"xfce4-terminal"`) |

### Response  -  `200 OK`
```json
{
  "success": true
}
```
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/launch \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"app": "firefox"}'
```

---

## Cursor Position

**`GET /api/v1/computers/{id}/desktop/cursor`**

Returns the current mouse cursor coordinates.

### Response  -  `200 OK`
```json
{
  "x": 500,
  "y": 300
}
```
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/desktop/cursor \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

---

## Hotkey

**`POST /api/v1/computers/{id}/desktop/hotkey`**

Presses a hotkey combination. Functionally equivalent to `key` but uses a dedicated endpoint for named combinations.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `keys` | string[] | Yes | Ordered list of keys to press simultaneously (e.g. `["ctrl", "shift", "t"]`) |

### Response  -  `200 OK`
```json
{"success": true}
```

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/hotkey \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"keys": ["ctrl", "shift", "t"]}'
```

---

## Key Down / Key Up

**`POST /api/v1/computers/{id}/desktop/key-down`**

**`POST /api/v1/computers/{id}/desktop/key-up`**

Send individual key-down or key-up events. Use these for precise control when holding a modifier key while performing another action.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | string | Yes | Key name (same format as `key` endpoint) |

### Response  -  `200 OK`
```json
{"success": true}
```

---

## Mouse Down / Mouse Up

**`POST /api/v1/computers/{id}/desktop/mouse-down`**

**`POST /api/v1/computers/{id}/desktop/mouse-up`**

Send individual mouse-button-down or mouse-button-up events. Use for custom drag sequences or long-press interactions.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | X coordinate |
| `y` | integer | Yes | Y coordinate |
| `button` | string | No | `"left"` (default), `"right"`, `"middle"` |

### Response  -  `200 OK`
```json
{"success": true}
```

---

## Move Mouse

**`POST /api/v1/computers/{id}/desktop/move`**

Moves the mouse cursor to the specified position without clicking.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | Target X coordinate |
| `y` | integer | Yes | Target Y coordinate |

### Response  -  `200 OK`
```json
{"success": true}
```

---

## Clipboard

### Get Clipboard

**`GET /api/v1/computers/{id}/desktop/clipboard`**

Returns the current clipboard contents.

### Response  -  `200 OK`
```json
{"text": "Hello, World!"}
```

### Set Clipboard

**`POST /api/v1/computers/{id}/desktop/clipboard`**

Sets the clipboard contents.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | Yes | Text to place in the clipboard |

### Response  -  `200 OK`
```json
{"success": true}
```

```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/clipboard \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Hello, World!"}'
```

---

## Screen Size

**`GET /api/v1/computers/{id}/desktop/screen-size`**

Returns the current desktop resolution.

### Response  -  `200 OK`
```json
{"width": 1280, "height": 720}
```

---

## Environment

**`GET /api/v1/computers/{id}/desktop/environment`**

Returns desktop session metadata  -  display server, session type, active user, and desktop environment name.

---

## Set Wallpaper

**`POST /api/v1/computers/{id}/desktop/wallpaper`**

Sets the desktop wallpaper. Supports per-tenant white-label branding.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Publicly accessible URL of the image to set as wallpaper |

---

## Accessibility Tree

**`GET /api/v1/computers/{id}/desktop/accessibility-tree`**

Returns the AT-SPI accessibility tree for the current desktop state. The tree describes all visible UI elements with their roles, labels, states, and bounding boxes. Use this for element discovery when exact coordinates are unknown.

### Response  -  `200 OK`
```json
{
  "tree": {
    "role": "application",
    "name": "Firefox",
    "children": [
      {
        "role": "button",
        "name": "Close",
        "x": 1260,
        "y": 10,
        "width": 20,
        "height": 20
      }
    ]
  }
}
```

---

## Window Management

### List Windows

See [List Windows](/docs/api-reference/desktop#list-windows) above.

### Window Size

**`GET /api/v1/computers/{id}/desktop/window/{window_id}/size`**

Returns the width and height of a specific window.

### Window Position

**`GET /api/v1/computers/{id}/desktop/window/{window_id}/position`**

Returns the x,y position of a specific window.

### Resize Window

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/resize`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `width` | integer | Yes | New width in pixels |
| `height` | integer | Yes | New height in pixels |

### Move Window

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/move`**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | integer | Yes | New X position |
| `y` | integer | Yes | New Y position |

### Maximize Window

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/maximize`**

### Minimize Window

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/minimize`**

### Close Window

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/close`**

All window management endpoints return `{"success": true}` on `200 OK`.

---

## Common Errors

All desktop endpoints share these error responses:

| Status | Code | Description |
|--------|------|-------------|
| 400 | `invalid computer id` | UUID format invalid |
| 403 | `FORBIDDEN` | Not a member of this computer |
| 404 | `NOT_FOUND` | Computer does not exist |
| 409 | `COMPUTER_NOT_RUNNING` | Computer is stopped or provisioning |
| 502 | `AGENT_UNAVAILABLE` | Cannot reach the computer command service |

---

## See also

- [Computers API](/docs/api-reference/computers)  -  create, start, stop computers
- [Exec API](/docs/api-reference/exec)  -  run bash/Python without desktop interaction
- [Files API](/docs/api-reference/files)  -  read and write files in the VM
- [Error Codes](/docs/api-reference/errors)  -  complete error code reference
