---
slug: "computer-use/sdk-examples"
title: "Computer use SDK examples"
description: "Python and TypeScript method names for driving a MIOSA Computer desktop, with runnable examples for screenshots, input, windows, accessibility, and shell."
section: "Computers"
order: 3010
---

The Python and TypeScript SDKs expose the same desktop contract.
In TypeScript the desktop methods also live on `computer.desktop`.
Method names are case-sensitive.

## Method reference

| Group | Python | TypeScript | Purpose |
|---|---|---|---|
| **Screen** | `screenshot()` | `screenshot()` | Full desktop as PNG bytes |
| | `screenshot_base64()` | `screenshotBase64()` | Same, base64, ready for a vision model |
| | `screenshot_region(x, y, w, h)` | `screenshotRegion(x, y, w, h)` | Region as PNG bytes |
| **Click** | `click(x, y, button="left")` | `click(x, y, button)` | Click at pixels; button `left`\|`right`\|`middle` |
| | `left_click(x, y)` | `leftClick(x, y)` | Left-button click |
| | `right_click(x, y)` | `rightClick(x, y)` | Right-button click |
| | - | `middleClick(x, y)` | Middle-button click |
| | `double_click(x, y)` | `doubleClick(x, y)` | Double-click |
| **Mouse** | `move_cursor(x, y)` | `moveMouse(x, y)` | Move the pointer without clicking |
| | `mouse_down(x, y, button="left")` | `mouseDown(x, y, button)` | Press and hold a button |
| | `mouse_up(x, y, button="left")` | `mouseUp(x, y, button)` | Release a held button |
| | `drag(from_x, from_y, to_x, to_y)` | `drag(fromX, fromY, toX, toY)` | Click-drag between two points |
| **Keyboard** | `type(text)` | `type(text)` | Type literal text at the focus |
| | `key(key)` | `key(key)` | Press one key or a `+` chord |
| | `hotkey(*keys)` | `hotkey(...keys)` | Simultaneous combo |
| | `key_down(key)` / `key_up(key)` | `keyDown(key)` / `keyUp(key)` | Hold or release a key |
| **Scroll** | `scroll(direction, clicks)` | `scroll(direction, clicks)` | Scroll `up`\|`down`\|`left`\|`right` by N notches |
| | `scroll_up/down/left/right(clicks)` | - | Convenience scrolls |
| **Clipboard** | `get_clipboard()` / `set_clipboard(text)` | `getClipboard()` / `setClipboard(text)` | Read or write clipboard text |
| **Screen info** | `get_screen_size()` | `screenSize()` | Resolution `{ width, height }` |
| | `get_cursor_position()` | `cursor()` | Cursor `{ x, y }` in screen pixels |
| **Windows** | `windows()` | `windows()` | List open windows |
| | `launch(app)` | `launch(appName)` | Open an installed app |
| | `focus_window(id)` | `focusWindow(id)` | Bring a window to the front |
| | `get_window_size(id)` / `set_window_size(id, w, h)` | `windowSize(id)` / `resizeWindow(id, w, h)` | Read or set window size |
| | `get_window_position(id)` / `set_window_position(id, x, y)` | `windowPosition(id)` / `moveWindow(id, x, y)` | Read or set window position |
| | `maximize_window(id)` / `minimize_window(id)` / `close_window(id)` | `maximizeWindow(id)` / `minimizeWindow(id)` / `closeWindow(id)` | Change window state |
| **Environment** | `get_desktop_environment()` | `environment()` | DE name and version |
| | `set_wallpaper(path)` | `setWallpaper(path)` | Set the background from a VM file |
| | `get_accessibility_tree()` | `accessibilityTree()` | AT-SPI element tree |
| **Shell and files** | `bash(cmd)` / `python(code)` | `bash(cmd)` / `python(code)` | Run inside the VM |
| | `write_file(path, content)` / `read_file(path)` | `writeFile(path, content)` / `readFile(path)` | Read or write a VM file |

## First loop

**Python**

```python
png = computer.screenshot()

# Decide from the image, or read the accessibility tree for structured elements.
tree = computer.get_accessibility_tree()

computer.left_click(500, 500)
after = computer.screenshot()
```

**TypeScript**

```typescript
const png = await computer.screenshot()
const tree = await computer.desktop.accessibilityTree()

await computer.leftClick(500, 500)
const after = await computer.screenshot()
```

## Feed a screenshot to a vision model

`Screenshot_base64` returns a string you can pass straight through as image data.

```python
b64 = computer.screenshot_base64()

import anthropic
client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-opus-4-5",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": b64}},
            {"type": "text", "text": "What is on screen, and where should I click?"},
        ],
    }],
)
```

## Keyboard and mouse

**Python**

```python
computer.type("hello, world")     # literal text
computer.key("Return")            # one key
computer.key("ctrl+a")            # a chord
computer.hotkey("ctrl", "shift", "t")

computer.key_down("shift")
computer.left_click(800, 200)     # shift-click to extend a selection
computer.key_up("shift")

computer.mouse_down(200, 200)
computer.move_cursor(400, 400)    # move while held
computer.mouse_up(400, 400)
```

**TypeScript**

```typescript
await computer.type("hello, world")
await computer.key("Return")
await computer.desktop.hotkey("ctrl", "shift", "t")

await computer.desktop.keyDown("shift")
await computer.leftClick(800, 200)
await computer.desktop.keyUp("shift")

await computer.desktop.mouseDown(200, 200)
await computer.desktop.moveMouse(400, 400)
await computer.desktop.mouseUp(400, 400)
```

## Clipboard, scroll, and windows

**Python**

```python
computer.set_clipboard("text to paste")
computer.hotkey("ctrl", "v")

computer.scroll("down", clicks=3)
computer.scroll_up(clicks=5)

w = computer.windows()[0]
computer.focus_window(w["id"])
computer.set_window_size(w["id"], 1280, 800)
computer.set_window_position(w["id"], 100, 50)
computer.maximize_window(w["id"])
```

**TypeScript**

```typescript
await computer.desktop.setClipboard("text to paste")
await computer.desktop.hotkey("ctrl", "v")

await computer.scroll("down", 3)

const w = (await computer.windows())[0]
await computer.focusWindow(w.id)
await computer.desktop.resizeWindow(w.id, 1280, 800)
await computer.desktop.moveWindow(w.id, 100, 50)
await computer.desktop.maximizeWindow(w.id)
```

## Shell, Python, and files

These run inside the VM with no GUI interaction and are usually the fastest way to do multi-step work.

```python
print(computer.bash("ls /workspace/Desktop"))
print(computer.python("print(1 + 1)"))

computer.write_file("/workspace/script.py", "print('hello')")
computer.bash("python3 /workspace/script.py")
print(computer.read_file("/workspace/.bashrc"))
```

## CLI

```bash
miosa api GET "/computers/$COMPUTER_ID/desktop/screenshot" > before.png
miosa api GET "/computers/$COMPUTER_ID/desktop/accessibility-tree"
miosa api POST "/computers/$COMPUTER_ID/desktop/click" -d '{"x": 500, "y": 500}'
miosa api GET "/computers/$COMPUTER_ID/desktop/screenshot" > after.png
```

## See also

- [API reference](/docs/computer-use/api-reference) - the same operations over REST.
- [Desktop API](/docs/api-reference/desktop) - request and response shapes.
