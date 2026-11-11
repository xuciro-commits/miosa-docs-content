---
slug: "browser-use/live-viewer"
title: "浏览器使用实时查看器"
description: "实时观看沙盒浏览器、控制、共享可撤销的仅供查看的链接，然后全屏显示。"
section: "Sandboxes"
order: 2026
---

实时查看器将沙盒浏览器呈现在您可以观看的页面中。
它需要沙箱的虚拟显示 (`Xvfb :99`)，该虚拟显示包含在沙箱映像中，而 SOMA 沙箱尚不具备。
它位于 MIOSA 控制台的沙箱页面上，并且相同的协议为您构建的任何客户端提供服务。

查看器是 `wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/stream` 处的 WebSocket：JPEG 帧和 JSON 事件下降，JSON 输入上升。
它通过 `api.miosa.ai`，而不是沙箱自己的主机。

## 三种打开方式

|表面|如何|能力|
|---|---|---|
|沙盒页面 |打开沙箱，然后打开其浏览器面板 |掌控|
|直播网址 | `stream_url` 来自浏览器 API，带有 `stream_auth` |掌控|
|仅供查看的分享链接 |为别人铸造|仅限观看 |

## 验证流

两种方式，都在 `api.miosa.ai` 上：

- `Authorization: Bearer <msk_* API key | session JWT>`，或
- `?token=<stream_auth>` 对于无法设置标头的客户端（`stream_auth` 值来自 `POST`）。

此处使用的 API 密钥需要 `sandboxes:exec` 范围。
仅观看的查看器可以传递 `?mode=view`，它到达沙箱中的仅查看侦听器。

## 掌控一切

在沙盒页面上打开的查看器或使用控制模式流打开的查看器可以进行控制：单击、键入、滚动和导航实时页面。
输入直接进入浏览器进程，因此智能体正在驱动的页面就是您正在操纵的页面。

## 分享仅供查看的链接

创建共享链接，让其他人无需接触浏览器即可观看。
使用它不需要 MIOSA 登录。

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

|领域|意义|
|---|---|
| `mode` |始终`view`。 |
| `share_token` |仅供查看的令牌；将其作为 `?share=<share_token>` 传递到 `stream_url`。 |
| `expires_at` |当链接停止工作时。 |
| `stream_url` |链接连接到的 WebSocket 流。 |

`ttl_seconds` 是可选的：默认为 `3600`，并且必须介于 `60` 和 `604800` 之间（7 天）。
`400 INVALID_TTL` 超出范围。

共享链接将打开与仅供查看的查看者相同的流：

```text
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?share=$SHARE_TOKEN
```

仅供查看的链接是**可撤销的**：撤销它会立即停止该浏览器的每个未完成的链接。

```bash
curl -X DELETE https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/share \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

```json
{ "data": { "revoked": true, "epoch": 1 } }
```

撤销会增加沙箱的份额`epoch`；从那时起，在较早时期铸造的每个代币都会被拒绝。

> **警告：仅查看链接无法驱动**
>
> 仅查看是在沙箱的浏览器进程内强制执行的，而不是在页面中执行。仅供查看的查看器接收帧；它的输入在机器上被拒绝，因此它永远无法键入、单击或导航——无论它要求什么模式。

## 全屏

查看器具有全屏控件，可将浏览器扩展到整个屏幕以进行更仔细的检查或信息亭式显示。
将其关闭可返回到嵌入框架。

## 下一步

- [通过 CDP](/docs/browser-use/cdp) 驱动它 - 将智能体连接到同一浏览器。
- [限制、计费、安全性](/docs/browser-use/limits) - 观看者可以做什么和不能做什么。
