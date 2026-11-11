---
slug: "browser-use"
title: "浏览器使用"
description: "任何 MIOSA 沙箱内的实时浏览器 - 监视、控制、共享仅供查看的链接，并让智能体通过 CDP 驱动同一浏览器。"
section: "Sandboxes"
order: 2023
---

**浏览器使用**为沙箱提供了一个真正的交互式浏览器。
它可在**每个沙箱**、任何模板上使用，在创建时无需选择任何内容：浏览器是沙箱的一部分，而不是单独的计算机。现场观众是今天 SOMA 上的一个例外，如下所述。

在沙箱内，浏览器在虚拟显示器 (`Xvfb :99`) 上运行，因此不需要桌面会话。
该显示位于沙箱图像中。
今天有一个例外：SOMA 沙箱附带了 Docker 和 Chromium，但不包含实时查看器渲染的虚拟显示，因此实时查看器在 SOMA 上尚不可用。

一个人可以观看、控制或共享仅供查看的链接。
智能体通过 CDP 连接到**同一**浏览器，因此自动化和人类共享一个会话、一组 cookie 和一个页面。

> **注意：浏览器使用还是计算机使用？**
>
> 浏览器使用是沙箱内的单个浏览器，并按普通沙箱使用计费。
> 如果任务需要完整桌面、其他应用程序或接管整个屏幕，请改用 [计算机使用](/docs/computer-use)。
> 请参阅[选择哪个](/docs/computer-use/vs-browser-use)。

## 1.创建沙箱

任何沙箱都可以在任何模板上运行。

**命令行界面**

```bash
miosa create my-box --wait
```

**打字稿**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const sandbox = await miosa.sandboxes.create({
  templateId: "miosa-sandbox",
})
console.log(sandbox.id, sandbox.state)
```

**Python**

```python
import os
from miosa import Miosa

miosa = Miosa(api_key=os.environ["MIOSA_API_KEY"])

sandbox = miosa.sandboxes.create(template_id="miosa-sandbox")
print(sandbox.id, sandbox.state)
```

等待沙箱 `state` 为 `running`。

## 2.启动浏览器

浏览器在第一个 `POST` 上延迟启动。

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

响应携带 `state`、`viewport`、`stream_url`、`cdp_url` 和一个短暂的 `stream_auth` 令牌。
请参阅[快速入门](/docs/browser-use/quickstart)了解每个字段。

## 3. 打开或驱动它

一个人打开查看器，或者一个智能体通过 CDP 连接。

**打开查看器**

```bash
# The viewer is a WebSocket on api.miosa.ai, authenticated with the stream token.
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?token=$STREAM_AUTH
```

在[实时查看器](/docs/browser-use/live-viewer)中控制或共享仅供查看的链接。

**Playwright（TypeScript）**

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

    browser_conn.close()   # detaches; it does not stop the sandbox browser
```

使用 `DELETE /browser` 停止浏览器会释放其内存，同时沙箱继续运行。
浏览器在**暂停和恢复**中仍然存在。

## 选择你需要的

|你想要|从这里开始 |
|---|---|
|启动和停止浏览器 | [快速入门](/docs/browser-use/quickstart) |
|观察并掌控| [实时查看器](/docs/browser-use/live-viewer) |
| 与智能体一起驾驶 | [通过 CDP](/docs/browser-use/cdp) 驱动它 |
|使用经过验证的客户端 | [集成](/docs/browser-use/integrations) |
|检查限额和计费 | [限制、计费和安全性](/docs/browser-use/limits) |
|了解模型 | [浏览器的使用方式](/docs/browser-use/how-it-works) |

## 下一步

- [快速入门](/docs/browser-use/quickstart)：在任何沙箱中启动实时浏览器并打开查看器（尚未在 SOMA 上）。
- [实时查看器](/docs/browser-use/live-viewer)：控制、共享仅供查看的链接并全屏显示。
- [通过 CDP](/docs/browser-use/cdp) 驱动它：Playwright 和 Puppeteer 连接到同一浏览器。
- [Integrations](/docs/browser-use/integrations)：连接经过端到端验证的 Chrome DevTools 协议客户端。
- [限制、计费和安全性](/docs/browser-use/limits)：最小大小、生命周期、计费以及如何强制执行访问。
- [浏览器的使用原理](/docs/browser-use/how-it-works)：沙盒浏览器的组成部分以及看到它的三个表面。
