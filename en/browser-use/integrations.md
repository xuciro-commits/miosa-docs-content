---
slug: "browser-use/integrations"
title: "Browser use integrations"
description: "Connect a Chrome DevTools Protocol client to a MIOSA sandbox browser. Only clients verified end to end are listed."
section: "Sandboxes"
order: 2028
---

Any client that speaks the Chrome DevTools Protocol can drive a sandbox browser.
Only clients verified end to end on MIOSA are listed here; more will be added as they are proven.

## Compatibility

Each client below was run against a live sandbox browser: connect, navigate, click, type, screenshot, and confirm a live viewer sees the same page.

| Client | Language | Version | Status |
|---|---|---|---|
| Playwright | JavaScript | 1.64.0 | **Tested** |
| Playwright | Python | 1.63.0 | **Tested** |
| Puppeteer | JavaScript | 25.13.0 | **Tested** |
| Raw CDP | Node (no library) | Node 24 | **Tested** |
| Stagehand | JavaScript | 3.7.3 | **Tested** |
| browser-use | Python | 0.13.11 | **Tested** |
| Computer use (screenshot + actions) | Any HTTP client | - | **Tested** |

## Connect

Every CDP client connects to the same endpoint on `api.miosa.ai`:

```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp
```

`POST /browser` starts the browser and returns `cdp_url` and `stream_auth`.
Authenticate the socket one of two ways:

- **Token in the URL.** Append the `stream_auth` token as `?token=`:

  ```text
  wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp?token=<stream_auth>
  ```

- **Authorization header.** `Authorization: Bearer <msk_* API key>` (the key needs the `sandboxes:exec` scope), for clients that can set headers on the WebSocket.

Both go through `api.miosa.ai`; nothing is exposed on the sandbox's own host.
The endpoint is control only: a view-only share token is refused here.

`POST /browser` also returns `cdp_ws_url` and `cdp_http_url`, which carry the token in the path for clients that cannot set headers or append a query string.
`cdp_http_url` answers `/json/version`, `/json/list`, `/json/new`, and `/json/protocol`, and rewrites every advertised `webSocketDebuggerUrl` to itself, so clients that follow discovery stay authenticated.

> **Note: Reuse the open context**
>
> The browser is already running with a page. Use the existing context and page instead of launching a new browser, so you act on the session the live viewer shows.

## Client examples

**Playwright (JavaScript)**

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

Or authenticate with a header, avoiding the token in the URL:

```javascript
const browserConn = await chromium.connectOverCDP(browser.cdp_url, {
  headers: { Authorization: `Bearer ${process.env.MIOSA_API_KEY}` },
})
```

**Playwright (Python)**

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

**Puppeteer**

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

Puppeteer can also discover the socket for you: pass `browserURL: cdp_http_url`, and Puppeteer reads `/json/version` and follows the rewritten `webSocketDebuggerUrl`.

**Raw CDP (WebSocket)**

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

**Stagehand 3.x**

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

**browser-use**

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

**Computer use (no CDP)**

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

Action types are `click`, `double_click`, `triple_click`, `move`, `drag`, `scroll`, `type`, `key` (for example `"ctrl+a"` or `"Enter"`), `navigate`, `back`, `forward`, `reload`, and `wait`.
`GET /browser/screenshot` takes `format=png|jpeg`, `encoding=base64`, and `tab=ID`.

## Stagehand 4.x

Stagehand 4.x is not supported: it requires pipe-only CDP.
Use Stagehand 3.x.

> **Caution: Detach, do not kill**
>
> `browser.close()` in Playwright and `browser.disconnect()` in Puppeteer detach your driver. They do not stop the sandbox browser, so a viewer stays connected.
> To stop the browser and free its memory, `DELETE /browser` - see [Limits](/docs/browser-use/limits).

## See also

- [Drive it over CDP](/docs/browser-use/cdp) - the endpoint, auth, and close codes in full.
- [Live viewer](/docs/browser-use/live-viewer) - watch and take control of the same browser.
