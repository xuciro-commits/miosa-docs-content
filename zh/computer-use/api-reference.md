---
slug: "computer-use/api-reference"
title: "电脑使用API​​参考"
description: "每个 MIOSA 计算机桌面控制端点 - 屏幕截图、鼠标、键盘、滚动、剪贴板、窗口、辅助功能树和一次性窗口状态读取。"
section: "Computers"
order: 3009
---

桌面控制端点将经过身份验证的命令智能体到计算机内的智能体。
所有这些都要求计算机为`running`；停止或配置的计算机返回 `409 COMPUTER_NOT_RUNNING`。

基本路径：`/api/v1/computers/{id}/desktop`。

> **注：两个主机，一份合约**
>
> 整个表面位于 `api.miosa.ai` 上。
> 较窄的开发者主机 `sandboxes.miosa.ai` 仅公开读取和常见操作端点：屏幕截图、屏幕截图/区域、单击、键入、按键、滚动、拖动、双击、等待、窗口、光标、屏幕大小、可访问性树、窗口/焦点、启动和窗口状态。
> 下面列出的低级端点 - 热键、键按下/键弹起、鼠标按下/鼠标弹起、移动、剪贴板、壁纸、环境和每个窗口几何 - 位于 `api.miosa.ai` 上。

## 观察

|方法|路径|目的|
|---|---|---|
| `GET` | `/screenshot` |完整桌面 PNG (`Content-Type: image/png`) |
| `POST` | `/screenshot/region` | PNG 矩形区域；正文 `{ x, y, width, height }` |
| `GET` | `/windows` |打开窗口： `id`、`title`、`class`、`x`、`y`、`width`、`height`、`focused` |
| `GET` | `/cursor` |屏幕像素中的光标位置 `{ x, y }` |
| `GET` | `/screen-size` |显示分辨率 `{ width, height }` |
| `GET` | `/environment` |桌面环境名称和版本（例如 `xfce4`）|
| `GET` | `/accessibility-tree` |作为嵌套 JSON 的 AT-SPI 元素树 |
| `GET` | `/window-state` |屏幕截图 **和** 一次往返中的可访问性树 |

### `GET /desktop/window-state`

并行运行屏幕截图和可访问性树，并从同一时刻将它们一起返回，因此智能体只需支付一次往返费用，而不是两次。

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

失败的部分不会导致请求失败：它是 `null` 并在 `errors` 对象下命名，因此调用者仍然可以使用另一半。
仅当两个部分都失败时，才会请求 `502 AGENT_UNAVAILABLE`。

## 鼠标

|方法|路径|身体|
|---|---|---|
| `POST` | `/click` | `x`、`y`、`button` (`left`\|`right`\|`middle`，默认 `left`) |
| `POST` | `/double-click` | `x`、`y` |
| `POST` | `/move` | `x`、`y`（移动指针，不单击）|
| `POST` | `/drag` | `from_x`、`from_y`、`to_x`、`to_y` |
| `POST` | `/mouse-down` | `x`、`y`、`button`（默认 `left`）|
| `POST` | `/mouse-up` | `x`、`y`、`button`（默认 `left`）|
| `POST` | `/scroll` | `x`、`y`、`direction` (`up`\|`down`\|`left`\|`right`)、`clicks` |

> **注意：鼠标按下/鼠标向上命名间隙**
>
> 公共端点是 `/desktop/mouse-down` 和 `/desktop/mouse-up`。
> 在计算机内部，它们映射到名为 `mouse-press` 和 `mouse-release` 的较低级别操作。
> 公共合约和机上操作的名称不同；配对和释放行为是相同的。
> 对于一次性点击，请使用 `POST /desktop/click` - 不要假设请求字段名为 `press`。

`mouse-down` / `mouse-up` 按住并释放按钮，中间不要单击，这样您就可以在中间拖动时移动指针（`move`，然后 `mouse-up`）或实现长按。

## 键盘

|方法|路径|身体|
|---|---|---|
| `POST` | `/type` | `text`，可选 `delay`（按键之间的毫秒）|
| `POST` | `/key` | `key` - 单个调或 `+` 和弦，例如 `Return` 或 `ctrl+a` |
| `POST` | `/hotkey` | `keys` - 同时组合，例如 `["ctrl", "shift", "t"]` |
| `POST` | `/key-down` | `key` - 按住 |
| `POST` | `/key-up` | `key` - 发布 |

键名称遵循 X11 键符号约定： `Return`、`Tab`、`BackSpace`、`Delete`、`Home`、`End`、`Page_Up`、`Page_Down`、 `Up`、`Down`、`Left`、`Right`、`F1`-`F12` 以及修饰符 `super`、`ctrl`、`alt`、 `shift`。
`type` 键入文字文本并且不解释键名称；使用 `key` 来实现这些。

## 剪贴板、窗口和桌面

|方法|路径|身体|
|---|---|---|
| `GET` | `/clipboard` |读取剪贴板文本 |
| `POST` | `/clipboard` | `text` - 写入剪贴板文本 |
| `POST` | `/window/focus` | `window_id` |
| `POST` | `/launch` | `command` - 启动应用程序，在后台运行 |
| `GET` | `/window/{window_id}/size` |读取窗口的 `{ width, height }` |
| `GET` | `/window/{window_id}/position` |读取窗口的 `{ x, y }` |
| `POST` | `/window/{window_id}/resize` | `width`, `height` |
| `POST` | `/window/{window_id}/move` | `x`, `y` |
| `POST` | `/window/{window_id}/maximize` | - |
| `POST` | `/window/{window_id}/minimize` | - |
| `POST` | `/window/{window_id}/close` | - |
| `POST` | `/wallpaper` | `path` - 从虚拟机内的文件设置背景 |

## 时间安排

|方法|路径|身体|
|---|---|---|
| `POST` | `/wait` | `seconds` - 在计算机内休眠，上限为 30，让 UI 动画稳定下来 |

## 相关端点

|方法|路径|目的|
|---|---|---|
| `GET` | `/api/v1/computers/{id}/screenshot` |全桌面截图的别名 |
| `GET` | `/api/v1/computers/{id}/urls` |签名桌面、VNC、终端和入口 URL |
| `POST` | `/api/v1/computers/{id}/stream-token` |铸造一个短暂的观众流令牌 |
| `GET` | `/api/v1/computers/{id}/vnc-credentials` |桌面的 VNC 凭据 |
| `GET` | `/api/v1/computers/{id}/viewer-password` |读取原始外部查看器密码 |
| `POST` | `/api/v1/computers/{id}/viewer-password/rotate` |轮换原始外部查看器密码 |

## 坐标

坐标是原点位于左上角的屏幕像素，与 `screen-size` 和 `cursor` 报告的内容匹配。
视觉模型可能会显示按比例缩小的屏幕截图；在发送操作之前将其输出映射回屏幕像素。

## 错误

|状态 |代码|原因 |
|---|---|---|
| `409` | `COMPUTER_NOT_RUNNING` |计算机已停止、配置或恢复 |
| `502` | `AGENT_UNAVAILABLE` |电脑内的智能体没有回答 |
| `429` | - |速率有限；荣誉 `X-RateLimit-*` 和 `Retry-After` |

另请参阅：[Desktop API](/docs/api-reference/desktop) 了解逐字段请求和响应形状。
