---
slug: "browser-use"
title: "Browser use"
description: "A live browser inside any MIOSA sandbox - watch it, take control, share a view-only link, and let agents drive the same browser over CDP."
section: "Sandboxes"
order: 2023
---

**Browser use** gives a sandbox a real, interactive browser.
It is available on **every sandbox**, on any template, with nothing to choose at create time: the browser is part of the sandbox, not a separate machine. The live viewer is the one exception on SOMA today, noted below.

Inside the sandbox the browser runs on a virtual display (`Xvfb :99`), so it needs no desktop session.
That display ships in the sandbox image.
One exception today: a SOMA sandbox ships Docker and Chromium but not the virtual display the live viewer renders, so the live viewer is not available on SOMA yet.

A person can watch it, take control, or share a view-only link.
An agent connects to the **same** browser over CDP, so automation and a human share one session, one set of cookies, and one page.

> **Note: Browser use or computer use?**
>
> Browser use is a single browser inside a sandbox and bills as ordinary sandbox usage.
> If a task needs a full desktop, other apps, or takeover of the whole screen, use [Computer use](/docs/computer-use) instead.
> See [Which to pick](/docs/computer-use/vs-browser-use).

## 1. Create a sandbox

Any sandbox works, on any template.

**CLI**

```bash
miosa create my-box --wait
```

**TypeScript**

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

Wait until the sandbox `state` is `running`.

## 2. Start the browser

The browser starts lazily on the first `POST`.

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

The response carries `state`, `viewport`, `stream_url`, `cdp_url`, and a short-lived `stream_auth` token.
See [Quickstart](/docs/browser-use/quickstart) for every field.

## 3. Open it or drive it

A person opens the viewer, or an agent connects over CDP.

**Open the viewer**

```bash
# The viewer is a WebSocket on api.miosa.ai, authenticated with the stream token.
wss://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser/stream?token=$STREAM_AUTH
```

Take control or share a view-only link in the [Live viewer](/docs/browser-use/live-viewer).

**Playwright (TypeScript)**

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

    browser_conn.close()   # detaches; it does not stop the sandbox browser
```

Stopping the browser with `DELETE /browser` frees its memory while the sandbox keeps running.
The browser survives **pause and resume**.

## Pick what you need

| You want to | Start here |
|---|---|
| Start and stop a browser | [Quickstart](/docs/browser-use/quickstart) |
| Watch and take control | [Live viewer](/docs/browser-use/live-viewer) |
| Drive it with an agent | [Drive it over CDP](/docs/browser-use/cdp) |
| Use a verified client | [Integrations](/docs/browser-use/integrations) |
| Check limits and billing | [Limits, billing, and security](/docs/browser-use/limits) |
| Understand the model | [How browser use works](/docs/browser-use/how-it-works) |

## Next

- [Quickstart](/docs/browser-use/quickstart): Start a live browser in any sandbox and open the viewer (not on SOMA yet).
- [Live viewer](/docs/browser-use/live-viewer): Take control, share a view-only link, and go full screen.
- [Drive it over CDP](/docs/browser-use/cdp): Playwright and Puppeteer connecting to the same browser.
- [Integrations](/docs/browser-use/integrations): Connect a Chrome DevTools Protocol client that was verified end to end.
- [Limits, billing, and security](/docs/browser-use/limits): Minimum size, lifecycle, billing, and how access is enforced.
- [How browser use works](/docs/browser-use/how-it-works): The parts of a sandbox browser and the three surfaces that see it.
