---
slug: "browser-use/quickstart"
title: "浏览器使用快速入门"
description: "在任何 MIOSA 沙箱中启动实时浏览器，读取其流和 CDP 端点，并在完成后停止它。"
section: "Sandboxes"
order: 2025
---

浏览器使用在沙箱内运行，在每个模板上运行，无需额外设置。
您像往常一样创建一个沙箱，然后启动其浏览器。

浏览器在第一个 `POST` 上延迟启动。从不使用浏览器的沙箱无需支付任何费用，并且沙箱启动速度仍与今天一样快。

## 1.启动沙箱

任何沙箱都可以在任何模板上运行。创建时不会选择有关浏览器的任何内容。
实时观看者需要沙箱的虚拟显示，而 SOMA 沙箱尚不具备。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "template_id": "miosa-sandbox", "size": "small" }'
```

等待沙箱 `state` 为 `running`。

## 2.启动浏览器

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

响应包含您需要的一切：

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

|领域|意义|
|---|---|
| `state` | `running` 当浏览器启动时。 |
| `epoch` |共享链接时代。碰撞它会撤销每个未完成的共享链接。 |
| `viewport` |浏览器视口，`1280` by `720`。 |
| `stream_url` |用于实时视图的 WebSocket (`wss://api.miosa.ai/...`)。 |
| `cdp_url` |智能体的 WebSocket Chrome DevTools 协议端点。 |
| `stream_auth` |查看器或 CDP 套接字的短期令牌。 |
| `stream_auth_expires_at` | `stream_auth` 过期时的 Unix 秒数（1 小时）。 |

`POST` 是幂等的：在正在运行的浏览器上再次调用它会返回相同的端点，而无需重新启动它。

> **注：范围**
>
> `GET` 需要 `sandboxes:read` 范围。
> `POST` 和 `DELETE` 需要 `sandboxes:write`。
> 当您使用 API 密钥进行身份验证时，WebSocket 端点需要 `sandboxes:exec`。

## 3. 打开查看器

在 MIOSA 控制台中，打开沙箱并使用其浏览器面板。
要自行打开查看器，请连接到 `stream_url` 并进行身份验证。

```bash
# The viewer protocol is a WebSocket: JPEG frames and JSON events down, JSON input up.
# Pass the token as a query parameter...
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?token=$STREAM_AUTH
# ...or as a header: Authorization: Bearer $STREAM_AUTH
```

从那里，查看者可以**控制**浏览器或共享**仅供查看的链接**。
请参阅[实时查看器](/docs/browser-use/live-viewer)。

## 4. 让智能体驾驶它

人们正在观看的浏览器与智能体通过 CDP 连接到的浏览器相同。
使用 Playwright 或 Puppeteer 连接到 `cdp_url` - 请参阅[通过 CDP](/docs/browser-use/cdp) 驱动它。

## 5.在不启动的情况下检查状态

`GET` 仅报告状态；它永远不会启动浏览器。

```bash
curl https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{
  "data": { "state": "running", "epoch": 0, "viewport": { "width": 1280, "height": 720 } }
}
```

对于未运行的沙箱，`state` 为 `sandbox_not_running` 并且不会触及任何来宾：

```json
{ "data": { "state": "sandbox_not_running", "epoch": 0 } }
```

## 6. 停止浏览器

```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{
  "data": { "state": "stopped", "epoch": 0, "viewport": { "width": 1280, "height": 720 } }
}
```

停止浏览器会释放其内存，同时沙箱继续运行。
浏览器在沙箱的**暂停和恢复**中仍然存在；停止它就是释放内存。

## 错误

|状态 |代码|原因 |
|---|---|---|
| `400` | `INVALID_ID` |沙箱 ID 不是 UUID |
| `404` | `NOT_FOUND` |未知的沙箱，或由另一租户拥有的沙箱 |
| `401`/`403` | - |凭证缺失或不足 |
| `409` | `SANDBOX_NOT_RUNNING` |沙箱必须正在运行才能启动或停止浏览器 |
| `501` | `BROWSER_NOT_AVAILABLE` |沙箱镜像没有 Chromium 或 Node |
| `502` | `BROWSER_START_FAILED` |浏览器守护进程没有及时启动 |

`501` 命名了一个包含浏览器先决条件的模板：沙箱映像包括 Chromium 和 Node.js。

## 下一步

- [实时查看器](/docs/browser-use/live-viewer) - 控制、共享、全屏。
- [通过 CDP](/docs/browser-use/cdp) 驱动它 - Playwright和Puppeteer。
- [限制、计费、安全性](/docs/browser-use/limits) - 最小大小和执行。
