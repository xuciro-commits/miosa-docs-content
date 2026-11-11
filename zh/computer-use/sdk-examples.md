---
slug: "computer-use/sdk-examples"
title: "电脑端使用SDK示例"
description: "用于驱动 MIOSA 计算机桌面的 Python 和 TypeScript 方法名称，以及屏幕截图、输入、窗口、辅助功能和 shell 的可运行示例。"
section: "Computers"
order: 3010
---

Python 和 TypeScript SDK 公开相同的桌面合约。
在 TypeScript 中，桌面方法也存在于 `computer.desktop` 上。
方法名称区分大小写。

## 方法参考

|集团|蟒蛇 |打字稿 |目的|
|---|---|---|---|
| **屏幕** | `screenshot()` | `screenshot()` | PNG 字节的完整桌面 |
| | `screenshot_base64()` | `screenshotBase64()` |同样，base64，准备好视觉模型 |
| | `screenshot_region(x, y, w, h)` | `screenshotRegion(x, y, w, h)` |区域为 PNG 字节 |
| **点击** | `click(x, y, button="left")` | `click(x, y, button)` |单击像素；按钮 `left`\|`right`\|`middle` |
| | `left_click(x, y)` | `leftClick(x, y)` |左键单击 |
| | `right_click(x, y)` | `rightClick(x, y)` |右键单击|
| | - | `middleClick(x, y)` |单击中键 |
| | `double_click(x, y)` | `doubleClick(x, y)` |双击 |
| **鼠标** | `move_cursor(x, y)` | `moveMouse(x, y)` |移动指针而不单击 |
| | `mouse_down(x, y, button="left")` | `mouseDown(x, y, button)` |按住按钮 |
| | `mouse_up(x, y, button="left")` | `mouseUp(x, y, button)` |释放按住的按钮|
| | `drag(from_x, from_y, to_x, to_y)` | `drag(fromX, fromY, toX, toY)` |在两点之间单击并拖动 |
| **键盘** | `type(text)` | `type(text)` |在焦点处键入文字 |
| | `key(key)` | `key(key)` |按一个键或 `+` 和弦 |
| | `hotkey(*keys)` | `hotkey(...keys)` |同时组合|
| | `key_down(key)` / `key_up(key)` | `keyDown(key)` / `keyUp(key)` |按住或松开键 |
| **滚动** | `scroll(direction, clicks)` | `scroll(direction, clicks)` |将 `up`\|`down`\|`left`\|`right` 滚动 N 格 |
| | `scroll_up/down/left/right(clicks)` | - |便利卷轴|
| **剪贴板** | `get_clipboard()` / `set_clipboard(text)` | `getClipboard()` / `setClipboard(text)` |读取或写入剪贴板文本 |
| **屏幕信息** | `get_screen_size()` | `screenSize()` |分辨率 `{ width, height }` |
| | `get_cursor_position()` | `cursor()` |屏幕像素中的光标 `{ x, y }` |
| **Windows** | `windows()` | `windows()` |列出打开的窗口 |
| | `launch(app)` | `launch(appName)` |打开已安装的应用程序 |
| | `focus_window(id)` | `focusWindow(id)` |把窗户放在前面|
| | `get_window_size(id)` / `set_window_size(id, w, h)` | `windowSize(id)` / `resizeWindow(id, w, h)` |读取或设置窗口大小 |
| | `get_window_position(id)` / `set_window_position(id, x, y)` | `windowPosition(id)` / `moveWindow(id, x, y)` |读取或设置窗口位置 |
| | `maximize_window(id)` / `minimize_window(id)` / `close_window(id)` | `maximizeWindow(id)` / `minimizeWindow(id)` / `closeWindow(id)` |更改窗口状态 |
| **环境** | `get_desktop_environment()` | `environment()` | DE 名称和版本 |
| | `set_wallpaper(path)` | `setWallpaper(path)` |从 VM 文件设置背景 |
| | `get_accessibility_tree()` | `accessibilityTree()` | AT-SPI 元素树 |
| **外壳和文件** | `bash(cmd)` / `python(code)` | `bash(cmd)` / `python(code)` |在虚拟机内运行 |
| | `write_file(path, content)` / `read_file(path)` | `writeFile(path, content)` / `readFile(path)` |读取或写入 VM 文件 |

## 第一个循环

**Python**

```python
png = computer.screenshot()

# Decide from the image, or read the accessibility tree for structured elements.
tree = computer.get_accessibility_tree()

computer.left_click(500, 500)
after = computer.screenshot()
```

**打字稿**

```typescript
const png = await computer.screenshot()
const tree = await computer.desktop.accessibilityTree()

await computer.leftClick(500, 500)
const after = await computer.screenshot()
```

## 将屏幕截图提供给视觉模型

`Screenshot_base64` 返回一个可以直接作为图像数据传递的字符串。

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

## 键盘和鼠标

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

**打字稿**

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

## 剪贴板、滚动条和窗口

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

**打字稿**

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

## Shell、Python 和文件

它们在虚拟机内运行，无需 GUI 交互，通常是执行多步骤工作的最快方法。

```python
print(computer.bash("ls /workspace/Desktop"))
print(computer.python("print(1 + 1)"))

computer.write_file("/workspace/script.py", "print('hello')")
computer.bash("python3 /workspace/script.py")
print(computer.read_file("/workspace/.bashrc"))
```

## 命令行界面

```bash
miosa api GET "/computers/$COMPUTER_ID/desktop/screenshot" > before.png
miosa api GET "/computers/$COMPUTER_ID/desktop/accessibility-tree"
miosa api POST "/computers/$COMPUTER_ID/desktop/click" -d '{"x": 500, "y": 500}'
miosa api GET "/computers/$COMPUTER_ID/desktop/screenshot" > after.png
```

## 另请参阅

- [API 参考](/docs/computer-use/api-reference) - 通过 REST 进行相同的操作。
- [桌面 API](/docs/api-reference/desktop) - 请求和响应形状。
