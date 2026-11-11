---
slug: "browser-use/integrations"
title: "浏览器使用集成"
description: "将 Chrome DevTools 协议客户端连接到 MIOSA 沙箱浏览器。仅列出经过端到端验证的客户端。"
section: "Sandboxes"
order: 2028
---

任何使用 Chrome DevTools 协议的客户端都可以驱动沙箱浏览器。
此处仅列出在 MIOSA 上经过端到端验证的客户端；当它们被证明时将添加更多。

## 兼容性

下面的每个客户端都针对实时沙箱浏览器运行：连接、导航、单击、键入、屏幕截图并确认实时查看者看到相同的页面。

|客户|语言 |版本 |状态 |
|---|---|---|---|
|Playwright | JavaScript | 1.64.0 | **经过测试** |
|Playwright |蟒蛇 | 1.63.0 | **经过测试** |
|傀儡师 | JavaScript | 25.13.0 | **经过测试** |
|原始 CDP |节点（无库）|节点 24 | **经过测试** |
|舞台工作人员| JavaScript | 3.7.3 | **经过测试** |
| browser-use | Python | 0.13.11 | **已测试** |
|电脑使用（截图+动作）|任何 HTTP 客户端 | - | **经过测试** |

## 连接

每个 CDP 客户端都连接到 `api.miosa.ai` 上的同一端点：

```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp
```

`POST /browser` 启动浏览器并返回 `cdp_url` 和 `stream_auth`。
通过以下两种方式之一验证套接字：

- **URL 中的令牌。** 将 `stream_auth` 令牌附加为 `?token=`：

  ```text
  wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp?token=<stream_auth>
  ```

- **授权标头。** `Authorization: Bearer <msk_* API key>`（密钥需要 `sandboxes:exec` 范围），适用于可以在 WebSocket 上设置标头的客户端。

两者都经过 `api.miosa.ai`;沙箱自己的主机上没有暴露任何内容。
该端点仅用于控制：此处拒绝仅查看共享令牌。

`POST /browser` 还返回 `cdp_ws_url` 和 `cdp_http_url`，它们在无法设置标头或附加查询字符串的客户端的路径中携带令牌。
`cdp_http_url` 应答 `/json/version`、`/json/list`、`/json/new` 和 `/json/protocol`，并将每个通告的 `webSocketDebuggerUrl` 重写为自身，以便遵循发现的客户端保持身份验证。

> **注意：重用开放上下文**
>
> 浏览器已经在运行一个页面。使用现有的上下文和页面，而不是启动新的浏览器，因此您可以对实时查看器显示的会话进行操作。

## 客户端示例

**Playwright（JavaScript）**

```javascript
import { chromium } from "playwright"

const cdpUrl = `${browser.cdp_url}?token=${browser.stream_auth}`

const browserConn = await chromium.connectOverCDP(cdpUrl)
const context = browserConn.contexts()[0]
const page = context.pages()[0] ?? (await context.newPage())

await page.goto("https://example.com")
console.log(await page.title())
await page.screenshot({ path: "page.png" })

await browserConn.close()   // detaches; it does not stop the sandbox browser
```

或者使用标头进行身份验证，避免 URL 中的令牌：

```javascript
const browserConn = await chromium.connectOverCDP(browser.cdp_url, {
  headers: { Authorization: `Bearer ${process.env.MIOSA_API_KEY}` },
})
```

**Playwright（Python）**

```python
from playwright.sync_api import sync_playwright

cdp_url = f"{browser['cdp_url']}?token={browser['stream_auth']}"

with sync_playwright() as p:
    browser_conn = p.chromium.connect_over_cdp(cdp_url)
    context = browser_conn.contexts[0]
    page = context.pages[0] if context.pages else context.new_page()

    page.goto("https://example.com")
    print(page.title())
    page.screenshot(path="page.png")

    browser_conn.close()   # detaches; it does not stop the sandbox browser
```

**傀儡师**

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

Puppeteer 还可以为您发现套接字：传递 `browserURL: cdp_http_url`，Puppeteer 读取 `/json/version` 并遵循重写的 `webSocketDebuggerUrl`。

**原始 CDP (WebSocket)**

```javascript
// Plain WebSocket, no client library.
const url = `${browser.cdp_url}?token=${browser.stream_auth}`

const ws = new WebSocket(url)
const pending = new Map()
let nextId = 1

ws.addEventListener("message", (event) => {
  const message = JSON.parse(event.data)
  pending.get(message.id)?.(message)
  pending.delete(message.id)
})

const send = (method, params = {}) =>
  new Promise((resolve) => {
    const id = nextId++
    pending.set(id, resolve)
    ws.send(JSON.stringify({ id, method, params }))
  })

await new Promise((resolve) => ws.addEventListener("open", resolve))
console.log((await send("Browser.getVersion")).result.product)

// Page-level work: discover a page target through /json/list,
// then open a second socket to its page-level URL.
const list = await (await fetch(`${browser.cdp_http_url}/json/list`)).json()
console.log(list[0].webSocketDebuggerUrl)
```

**舞台工作人员 3.x**

```javascript
import { Stagehand } from "@browserbasehq/stagehand"

const stagehand = new Stagehand({
  env: "LOCAL",
  localBrowserLaunchOptions: {
    cdpUrl: `${browser.cdp_url}?token=${browser.stream_auth}`,
  },
})

await stagehand.init()
const page = (await stagehand.context.pages())[0]
await page.goto("https://example.com")
console.log(await page.title())
```

**浏览器使用**

```python
import asyncio
from browser_use import Browser

async def main():
    # `browser` is the POST /browser response: { "cdp_url": ..., "stream_auth": ... }
    cdp_url = f"{browser['cdp_url']}?token={browser['stream_auth']}"

    agent = Browser(cdp_url=cdp_url, is_local=True)
    await agent.start()
    page = await agent.get_current_page()
    await page.goto("https://example.com")
    print(await page.get_title())
    # browser-use evaluates with the function form:
    print(await page.evaluate("(...a) => document.title"))
    await agent.stop()

asyncio.run(main())
```

**计算机使用（无 CDP）**

```javascript
// OpenAI/Anthropic computer-use style loop: screenshots plus coordinate actions.
// No CDP in the client at all.
const base = `https://api.miosa.ai/api/v1/sandboxes/${sandboxId}/browser`
const headers = { Authorization: `Bearer ${apiKey}` }

// A screenshot of the current page
const shot = await fetch(`${base}/screenshot?format=png`, { headers })

// Navigate, type, and click by coordinate
await fetch(`${base}/actions`, {
  method: "POST",
  headers: { ...headers, "content-type": "application/json" },
  body: JSON.stringify({
    actions: [
      { type: "navigate", url: "https://example.com" },
      { type: "wait", ms: 600 },
      { type: "type", text: "miosa" },
      { type: "click", x: 140, y: 270 },
    ],
  }),
})
```

操作类型为 `click`、`double_click`、`triple_click`、`move`、`drag`、`scroll`、`type`、`key`（例如 `"ctrl+a"` 或 `"Enter"`）， `navigate`、`back`、`forward`、`reload` 和 `wait`。
`GET /browser/screenshot` 采用 `format=png|jpeg`、`encoding=base64` 和 `tab=ID`。

## 舞台工作人员 4.x

不支持 Stagehand 4.x：它需要仅管道 CDP。
使用 Stagehand 3.x。

> **注意：分离，不要杀死**
>
> Playwright 中的 `browser.close()` 和 Puppeteer 中的 `browser.disconnect()` 分离您的驱动程序。它们不会停止沙盒浏览器，因此观看者保持连接。
> 要停止浏览器并释放其内存，`DELETE /browser` - 请参阅[限制](/docs/browser-use/limits)。

## 另请参阅

- [通过 CDP](/docs/browser-use/cdp) 驱动它 - 完整的端点、身份验证和关闭代码。
- [实时查看器](/docs/browser-use/live-viewer) - 观看并控制同一浏览器。
