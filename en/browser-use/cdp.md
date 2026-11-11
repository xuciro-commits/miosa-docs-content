---
slug: "browser-use/cdp"
title: "Drive a sandbox browser over CDP"
description: "Connect Playwright and Puppeteer to a live sandbox browser over CDP and drive the same session a viewer is watching."
section: "Sandboxes"
order: 2027
---

The sandbox browser speaks the Chrome DevTools Protocol.
Point Playwright or Puppeteer at the browser's CDP endpoint and your agent drives the **same** browser the live viewer shows - same tabs, same logins, same page.

The CDP endpoint is a WebSocket on `api.miosa.ai`:

```text
wss://api.miosa.ai/api/v1/sandboxes/{id}/browser/cdp
```

It is control only. A view-only share token is refused here.

## Get the endpoint

`POST /browser` starts the browser and returns `cdp_url` and `stream_auth`:

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## Authenticate the CDP socket

The socket takes the same credentials as the stream:

- `?token=<stream_auth>` - the token returned by `POST` (1 hour). Simplest for an agent.
- `Authorization: Bearer <msk_* API key>` - API keys need the `sandboxes:exec` scope; a browser can reach anything the sandbox can, so the scope is deliberate.

## Playwright

> **Note: Already-open context**
>
> The browser is already running with a page. Use the existing context and page instead of launching a new browser, so you act on the session the viewer sees.

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

You can pass the token as a header instead of a query parameter when your client supports it, for example Playwright's `connect_over_cdp(cdp_url, headers={"Authorization": "Bearer " + api_key})`.

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

> **Caution: Disconnect, do not kill**
>
> `browser.close()` in Playwright and `browser.disconnect()` in Puppeteer detach the agent.
> They do not stop the sandbox browser, so a viewer stays connected.
> To stop the browser and free its memory, `DELETE /browser` - see [Limits](/docs/browser-use/limits).

## One browser, two drivers

Because the viewer and the agent share one browser:

- A person can open the live viewer while an agent is running and watch the agent work.
- If a person takes control, the agent's next action lands on the same page.
- Cookies and logins persist across both, and across pause and resume of the sandbox.

Keep the agent and the human from acting at the same instant by taking control only when the automation is idle.

## Close codes

If the socket closes before it is ready, the close code says why:

| Code | Meaning |
|---|---|
| `4001` | Unauthorized |
| `4003` | Forbidden (the key lacks `sandboxes:exec`) |
| `4004` | Sandbox not found |
| `4009` | Sandbox not running |
| `4012` | Browser not started - call `POST /browser` first |
| `4013` | Tenant stream limit reached |
| `4014` | Share link expired or revoked |
| `1011` | Upstream failure |

## Next

- [Live viewer](/docs/browser-use/live-viewer) - watch and take control.
- [Limits, billing, security](/docs/browser-use/limits) - lifecycle, billing, and how access is enforced.
