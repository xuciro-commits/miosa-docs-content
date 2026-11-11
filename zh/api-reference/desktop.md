---
slug: "api-reference/desktop"
title: "桌面API"
description: "所有桌面控制端点的 API 参考 - 屏幕截图、鼠标、键盘、剪贴板、窗口、辅助功能树。"
section: "API reference"
order: 12030
---

桌面控制端点通过 MIOSA 控制平面将命令智能体到计算机 VM 内的计算机命令服务。所有坐标均以屏幕像素为单位（0,0 = 左上角）。

基本路径：`/api/v1/computers/{id}/desktop`

> **注意**
>
> 所有桌面端点都要求计算机处于 `"running"` 状态。如果计算机已停止或配置，则返回 `409 COMPUTER_NOT_RUNNING`。

> **注意**
>
> 速率限制因密钥、租户策略和端点系列而异。尊重 `X-RateLimit-*` 和 `Retry-After` 响应标头。单个桌面操作速度很快，但高频 AI 智能体应尽可能批量操作。

---

## 截图

**`GET /api/v1/computers/{id}/desktop/screenshot`**

将完整桌面捕获为 PNG 图像。

### 响应 - `200 OK`

带有 `Content-Type: image/png` 的二进制 PNG 数据。
```bash
curl https://api.miosa.ai/api/v1/computers/{id}/desktop/screenshot \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  --output screenshot.png
```

---

## 截图区域

**`POST /api/v1/computers/{id}/desktop/screenshot/region`**

捕获屏幕的矩形区域。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 |左边缘X坐标|
| `y` |整数 |是的 |顶边 Y 坐标 |
| `width` |整数 |是的 |宽度（以像素为单位）|
| `height` |整数 |是的 |高度（以像素为单位）|

### 响应 - `200 OK`

二进制 PNG 数据。
```bash
curl -X POST https://api.miosa.ai/api/v1/computers/{id}/desktop/screenshot/region \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"x": 0, "y": 0, "width": 800, "height": 600}' \
  --output region.png
```

---

## 点击

**`POST /api/v1/computers/{id}/desktop/click`**

在指定坐标处执行鼠标单击。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 | X坐标|
| `y` |整数 |是的 | Y坐标|
| `button` |字符串|没有 | `"left"`（默认）、`"right"`、`"middle"` |

### 响应 - `200 OK`
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

## 双击

**`POST /api/v1/computers/{id}/desktop/double-click`**

在指定坐标处执行双击。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 | X坐标|
| `y` |整数 |是的 | Y坐标|

### 响应 - `200 OK`
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

## 输入文本

**`POST /api/v1/computers/{id}/desktop/type`**

键入文本字符串作为键盘输入。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `text` |字符串|是的 |要输入的文本 |

### 响应 - `200 OK`
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

## 按键

**`POST /api/v1/computers/{id}/desktop/key`**

按下一个键或组合键。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `key` |字符串|是的 |按键名称或组合|

组合键使用 `+` 作为分隔符：`"ctrl+c"`、`"alt+Tab"`、`"ctrl+shift+t"`。

### 常用键

|关键|价值|
|-----|--------|
|输入| `Return` |
|选项卡| `Tab` |
|逃亡| `Escape` |
|退格 | `BackSpace` |
|删除| `Delete` |
|空间| `space` |
|箭头| `Up`、`Down`、`Left`、`Right` |
|功能| `F1`...`F12` |
|首页/完| `Home`、`End` |
|向上/向下翻页| `Page_Up`、`Page_Down` |

### 响应 - `200 OK`
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

## 滚动

**`POST /api/v1/computers/{id}/desktop/scroll`**

在指定位置滚动鼠标滚轮。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 | X坐标|
| `y` |整数 |是的 | Y坐标|
| `direction` |字符串|是的 | `"up"` 或 `"down"` |
| `amount` |整数 |没有 |滚动步数（默认：3）|

### 响应 - `200 OK`
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

## 拖动

**`POST /api/v1/computers/{id}/desktop/drag`**

执行从一点到另一点的单击并拖动操作。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `start_x` |整数 |是的 |起始 X 坐标 |
| `start_y` |整数 |是的 |起始 Y 坐标 |
| `end_x` |整数 |是的 |结束 X 坐标 |
| `end_y` |整数 |是的 |结束 Y 坐标 |

### 响应 - `200 OK`
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

## 等等

**`POST /api/v1/computers/{id}/desktop/wait`**服务器端暂停。在响应之前将连接保持指定的持续时间。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `seconds` |数量 |没有 |等待秒数（默认值：1，最大值：30）|

### 响应 - `200 OK`
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

## 列出窗口

**`GET /api/v1/computers/{id}/desktop/windows`**

返回桌面上打开的窗口的列表。

### 响应 - `200 OK`
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

## 焦点窗口

**`POST /api/v1/computers/{id}/desktop/window/focus`**

将一个窗口带到前台。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `window_id` |字符串|是的 |窗口响应列表中的窗口 ID |

### 响应 - `200 OK`
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

## 启动应用程序

**`POST /api/v1/computers/{id}/desktop/launch`**

通过名称或命令启动应用程序。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `app` |字符串|是的 |应用程序名称或命令（例如，`"firefox"`、`"xfce4-terminal"`）|

### 响应 - `200 OK`
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

## 光标位置

**`GET /api/v1/computers/{id}/desktop/cursor`**

返回当前鼠标光标坐标。

### 响应 - `200 OK`
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

## 热键

**`POST /api/v1/computers/{id}/desktop/hotkey`**

按热键组合。功能上等同于 `key` 但对命名组合使用专用端点。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `keys` |字符串[] |是的 |同时按下的按键的有序列表（例如 `["ctrl", "shift", "t"]`）|

### 响应 - `200 OK`
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

## 按下键/按下键

**`POST /api/v1/computers/{id}/desktop/key-down`**

**`POST /api/v1/computers/{id}/desktop/key-up`**

发送单独的按键按下或按键弹起事件。当按住修饰键并执行其他操作时，可以使用它们进行精确控制。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `key` |字符串|是的 |键名称（与 `key` 端点格式相同）|

### 响应 - `200 OK`
```json
{"success": true}
```

---

## 鼠标按下/鼠标向上

**`POST /api/v1/computers/{id}/desktop/mouse-down`**

**`POST /api/v1/computers/{id}/desktop/mouse-up`**

发送单独的鼠标按钮按下或鼠标按钮松开事件。用于自定义拖动序列或长按交互。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 | X坐标|
| `y` |整数 |是的 | Y坐标|
| `button` |字符串|没有 | `"left"`（默认）、`"right"`、`"middle"` |

### 响应 - `200 OK`
```json
{"success": true}
```

---

## 移动鼠标

**`POST /api/v1/computers/{id}/desktop/move`**

将鼠标光标移动到指定位置而不单击。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 |目标 X 坐标 |
| `y` |整数 |是的 |目标 Y 坐标 |

### 响应 - `200 OK`
```json
{"success": true}
```

---

## 剪贴板

### 获取剪贴板

**`GET /api/v1/computers/{id}/desktop/clipboard`**

返回当前剪贴板内容。

### 响应 - `200 OK`
```json
{"text": "Hello, World!"}
```

### 设置剪贴板

**`POST /api/v1/computers/{id}/desktop/clipboard`**

设置剪贴板内容。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `text` |字符串|是的 |要放置在剪贴板中的文本 |

### 响应 - `200 OK`
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

## 屏幕尺寸

**`GET /api/v1/computers/{id}/desktop/screen-size`**

返回当前桌面分辨率。

### 响应 - `200 OK`
```json
{"width": 1280, "height": 720}
```

---

## 环境

**`GET /api/v1/computers/{id}/desktop/environment`**

返回桌面会话元数据 - 显示服务器、会话类型、活动用户和桌面环境名称。

---

## 设置壁纸

**`POST /api/v1/computers/{id}/desktop/wallpaper`**

设置桌面壁纸。支持每个租户的白标品牌。

### 请求正文

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `url` |字符串|是的 |设置为壁纸的图像的可公开访问的 URL |

---

## 辅助功能树

**`GET /api/v1/computers/{id}/desktop/accessibility-tree`**返回当前桌面状态的 AT-SPI 可访问性树。该树描述了所有可见的 UI 元素及其角色、标签、状态和边界框。当确切的坐标未知时，使用它来发现元素。

### 响应 - `200 OK`
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

## 窗口管理

### 列出窗口

请参阅上面的[列出 Windows](/docs/api-reference/desktop#list-windows)。

### 窗口大小

**`GET /api/v1/computers/{id}/desktop/window/{window_id}/size`**

返回特定窗口的宽度和高度。

### 窗口位置

**`GET /api/v1/computers/{id}/desktop/window/{window_id}/position`**

返回特定窗口的 x,y 位置。

### 调整窗口大小

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/resize`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `width` |整数 |是的 |新宽度（以像素为单位）|
| `height` |整数 |是的 |新高度（以像素为单位）|

### 移动窗口

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/move`**

|领域|类型 |必填 |描述 |
|--------|------|----------|-------------|
| `x` |整数 |是的 |新的 X 位置 |
| `y` |整数 |是的 |新 Y 位置 |

### 最大化窗口

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/maximize`**

### 最小化窗口

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/minimize`**

### 关闭窗口

**`POST /api/v1/computers/{id}/desktop/window/{window_id}/close`**

所有窗口管理端点都在 `200 OK` 上返回 `{"success": true}`。

---

## 常见错误

所有桌面端点共享这些错误响应：

|状态 |代码|描述 |
|--------|------|-------------|
| 400 | `invalid computer id` | UUID 格式无效 |
| 403 | `FORBIDDEN`|不是这台计算机的成员 |
| 404 | `NOT_FOUND`|计算机不存在 |
| 409 | `COMPUTER_NOT_RUNNING`|计算机已停止或配置 |
| 502 | `AGENT_UNAVAILABLE`|无法访问计算机命令服务 |

---

## 另请参阅

- [Computers API](/docs/api-reference/computers) - 创建、启动、停止计算机
- [Exec API](/docs/api-reference/exec) - 无需桌面交互即可运行 bash/Python
- [Files API](/docs/api-reference/files) - 在虚拟机中读写文件
- [错误代码](/docs/api-reference/errors) - 完整的错误代码参考
