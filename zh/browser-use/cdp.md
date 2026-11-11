---
slug: "browser-use/cdp"
title: "通过 CDP 驱动沙箱浏览器"
description: "通过 CDP 将 Playwright 和 Puppeteer 连接到实时沙箱浏览器，并驱动观众正在观看的同一会话。"
section: "Sandboxes"
order: 2027
---

沙盒浏览器使用 Chrome DevTools 协议。
将 Playwright 或 Puppeteer 指向浏览器的 CDP 端点，您的智能体将驱动实时查看器显示的**相同**浏览器 - 相同的选项卡、相同的登录、相同的页面。

CDP 端点是 `api.miosa.ai` 上的 WebSocket：

```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp
```

这只是控制。此处拒绝仅供查看的共享令牌。

## 获取端点

`POST /browser` 启动浏览器并返回 `cdp_url` 和 `stream_auth`：

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## 验证 CDP 套接字

套接字采用与流相同的凭据：

- `?token=<stream_auth>` - `POST` 返回的令牌（1 小时）。对于智能体来说最简单。
- `Authorization: Bearer <msk_* API key>` - API 密钥需要 `sandboxes:exec` 范围；浏览器可以访问沙箱可以访问的任何内容，因此范围是经过深思熟虑的。

## Playwright

> **注意：已经打开的上下文**
>
> 浏览器已经在运行一个页面。使用现有的上下文和页面，而不是启动新的浏览器，这样您就可以对查看者看到的会话进行操作。

**Python**

```python
from playwright.sync_api import sync_playwright

cdp_url = f"{browser['cdp_url']}?token={browser['stream_auth']}"

with sync_playwright() as p:
    browser_conn = p.chromium.connect_over_cdp(cdp_url)
    context = browser_conn.contexts[0]
    page = context.pages[0] if context.pages else context.new_page()

    page.goto("https://example.com")
    print(page.title())

    browser_conn.close()   # detaches; it does not stop the sandbox browser
```

**JavaScript**

```javascript
import { chromium } from "playwright"

const cdpUrl = `${browser.cdp_url}?token=${browser.stream_auth}`

const browserConn = await chromium.connectOverCDP(cdpUrl)
const context = browserConn.contexts()[0]
const page = context.pages()[0] ?? (await context.newPage())

await page.goto("https://example.com")
console.log(await page.title())

await browserConn.close()   // detaches; it does not stop the sandbox browser
```

当您的客户端支持时，您可以将令牌作为标头而不是查询参数传递，例如 Playwright 的 `connect_over_cdp(cdp_url, headers={"Authorization": "Bearer " + api_key})`。

## Puppeteer

**JavaScript**

```javascript
import puppeteer from "puppeteer"

const browser = await puppeteer.connect({
  browserWSEndpoint: `${browserInfo.cdp_url}?token=${browserInfo.stream_auth}`,
})

const [page] = await browser.pages()
await page.goto("https://example.com")
console.log(await page.title())

await browser.disconnect()   // detaches; it does not stop the sandbox browser
```

> **注意：断开连接，不要杀人**
>
> Playwright 中的 `browser.close()` 和 Puppeteer 中的 `browser.disconnect()` 分离智能体。
> 它们不会停止沙盒浏览器，因此观看者可以保持连接。
> 要停止浏览器并释放其内存，`DELETE /browser` - 请参阅[限制](/docs/browser-use/limits)。

## 一个浏览器，两个驱动程序

因为查看者和智能体共享一个浏览器：

- 一个人可以在智能体运行时打开实时查看器并观看智能体工作。
- 如果有人控制，智能体的下一步行动就会出现在同一页面上。
- Cookie 和登录信息在沙箱的暂停和恢复期间持续存在。

仅在自动化空闲时进行控制，以防止智能体和人类同时采取行动。

## 关闭代码

如果套接字在准备好之前关闭，关闭代码会说明原因：

|代码|意义|
|---|---|
| `4001` |未经授权 |
| `4003` |禁止（密钥缺少 `sandboxes:exec`） |
| `4004` |未找到沙箱 |
| `4009` |沙盒未运行 |
| `4012` |浏览器未启动 - 首先调用 `POST /browser` |
| `4013` |已达到租户流限制 |
| `4014` |分享链接已过期或已撤销 |
| `1011` |上游故障|

## 下一步

- [实时查看器](/docs/browser-use/live-viewer) - 观看并掌控。
- [限制、计费、安全性](/docs/browser-use/limits) - 生命周期、计费以及如何强制执行访问。
