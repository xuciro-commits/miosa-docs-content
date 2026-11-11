---
slug: "cookbook/browser-agent"
title: "Browser agent"
description: "Put an LLM in the loop of a real browser running inside a sandbox. Start the sandbox browser, drive it over CDP with Playwright, and run the screenshot → action → screenshot loop."
section: "Sandboxes"
order: 2034
---

`~12 min` `Python` `TypeScript`

> **Goal**: Run an LLM-driven browser agent inside a MIOSA sandbox. Start the sandbox browser, connect Playwright to it over CDP, and loop: screenshot, ask the model what to do, act on the same page.
>
> **What you'll use**: Sandboxes, the sandbox browser, Playwright over CDP

## What you'll build

A `BrowserAgent` that:

1. Creates a sandbox and starts its browser.
2. Connects Playwright to the same browser the live viewer shows.
3. Enters a screenshot -> LLM -> action loop.
4. Stops when the model signals completion.

The LLM is your choice. The code below uses Anthropic Claude's vision API, but the pattern is model-agnostic.

> **Note: The browser is part of the sandbox**
>
> Browser use runs inside a sandbox, so there is no separate machine to boot and no desktop to wait for. See [Browser use](/docs/browser-use) for the full surface.

## Prerequisites

- A MIOSA workspace API key (`msk_u_*`)  -  see [API Keys](/docs/platform/api-keys)
- An Anthropic API key (or substitute your preferred vision-capable model)
- Node 22+ or Python 3.11+

## Step 1  -  Install and configure

**Python**

```bash
pip install miosa anthropic playwright
playwright install chromium
export MIOSA_API_KEY="msk_u_..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

**TypeScript**

```bash
npm install @miosa/sdk @anthropic-ai/sdk playwright
npx playwright install chromium
export MIOSA_API_KEY="msk_u_..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

Playwright runs on your machine as the driver. The browser it drives runs inside the sandbox.

## Step 2  -  Create a sandbox and start its browser

Create a sandbox, then start its browser. Nothing about the browser is chosen at create time: any template works, as long as its image includes Chromium and Node (the standard images do).

**Python**

```python
import os, time, requests
from miosa import Miosa

client = Miosa(api_key=os.environ["MIOSA_API_KEY"])
API = "https://api.miosa.ai/api/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['MIOSA_API_KEY']}"}

sandbox = client.sandboxes.create(template_id="miosa-sandbox", size="small")
print(f"Sandbox {sandbox.id}")

# Wait for the runtime to accept work.
while sandbox.state != "running":
    if sandbox.state == "error":
        raise RuntimeError(f"Sandbox failed to boot: {sandbox.state}")
    time.sleep(2)
    sandbox = client.sandboxes.get(sandbox.id)

# The browser lives inside the sandbox; ask it to start.
browser_info = requests.post(f"{API}/sandboxes/{sandbox.id}/browser", headers=HEADERS).json()["data"]
cdp_url = f"{browser_info['cdp_url']}?token={browser_info['stream_auth']}"
print(f"Live viewer: {browser_info['stream_url']}")
```

**TypeScript**

```typescript
import { Miosa } from '@miosa/sdk';

const client = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });
const API = 'https://api.miosa.ai/api/v1';
const headers = { Authorization: `Bearer ${process.env.MIOSA_API_KEY!}` };

let sandbox = await client.sandboxes.create({ templateId: 'miosa-sandbox', size: 'small' });
console.log('Sandbox', sandbox.id);

while (sandbox.state !== 'running') {
  if (sandbox.state === 'error') throw new Error('Sandbox failed to boot');
  await new Promise(r => setTimeout(r, 2000));
  sandbox = await client.sandboxes.get(sandbox.id);
}

const started = await fetch(`${API}/sandboxes/${sandbox.id}/browser`, { method: 'POST', headers });
const { data: browser } = await started.json();
const cdpUrl = `${browser.cdp_url}?token=${browser.stream_auth}`;
console.log('Live viewer:', browser.stream_url);
```

> **Note: Endpoints**
>
> `POST /browser` starts the browser lazily and returns `stream_url`, `cdp_url`, and `stream_auth`.
> Both sockets are on `api.miosa.ai`. See [Browser use quickstart](/docs/browser-use/quickstart) for the full set of fields.

## Step 3  -  Connect Playwright over CDP

The sandbox browser speaks the Chrome DevTools Protocol. Connect Playwright to `cdp_url` (with the `stream_auth` token appended as `?token=`) and reuse the context and page that are already open, so you act on the session a viewer would see.

**Python**

```python
from playwright.sync_api import sync_playwright

playwright = sync_playwright().start()
browser = playwright.chromium.connect_over_cdp(cdp_url)
context = browser.contexts[0]
page = context.pages[0] if context.pages else context.new_page()

page.goto("https://example.com")
print("Loaded:", page.title())
```

**TypeScript**

```typescript
import { chromium } from 'playwright';

const browser = await chromium.connectOverCDP(cdpUrl);
const context = browser.contexts()[0];
const page = context.pages()[0] ?? (await context.newPage());

await page.goto('https://example.com');
console.log('Loaded:', await page.title());
```

> **Caution: Detach, do not kill**
>
> `browser.close()` in Playwright detaches your driver. It does not stop the sandbox browser, so a viewer stays connected. To stop the browser and free its memory, use the browser API.

## Step 4  -  Screenshot, decide, act

The loop is the same as any vision agent: capture a screenshot, ask the model what to do next, act, repeat. The model returns a JSON action; your code executes it on the page.

**Python**

```python
import anthropic, base64, json

ai = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

SYSTEM_PROMPT = """\
You control a browser page. You receive a screenshot and a task.
Respond with ONLY a JSON object  -  no explanation, no markdown fences.

Action format:
  {"type": "click", "x": <int 0-1000>, "y": <int 0-1000>}
  {"type": "type", "text": "<string>"}
  {"type": "key", "combo": "<combo>"}      e.g. "Enter", "Control+A", "Tab"
  {"type": "scroll", "direction": "up"|"down"}
  {"type": "goto", "url": "<url>"}
  {"type": "done", "result": "<summary>"}

Coordinates are 0-1000 on both axes regardless of the page size.
"""

def next_action(task, png_b64, history):
    messages = history + [{
        "role": "user",
        "content": [
            {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": png_b64}},
            {"type": "text", "text": f"Task: {task}"},
        ],
    }]
    msg = ai.messages.create(model="claude-opus-4-5", max_tokens=256, system=SYSTEM_PROMPT, messages=messages)
    return json.loads(msg.content[0].text.strip())
```

**TypeScript**

```typescript
import Anthropic from '@anthropic-ai/sdk';

const ai = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! });

const SYSTEM_PROMPT = `
You control a browser page. You receive a screenshot and a task.
Respond with ONLY a JSON object  -  no explanation, no markdown fences.

Action format:
  {"type": "click", "x": <int 0-1000>, "y": <int 0-1000>}
  {"type": "type", "text": "<string>"}
  {"type": "key", "combo": "<combo>"}
  {"type": "scroll", "direction": "up"|"down"}
  {"type": "goto", "url": "<url>"}
  {"type": "done", "result": "<summary>"}

Coordinates are 0-1000 on both axes regardless of the page size.
`.trim();

async function nextAction(task: string, pngB64: string, history: Anthropic.MessageParam[]) {
  const messages: Anthropic.MessageParam[] = [
    ...history,
    {
      role: 'user',
      content: [
        { type: 'image', source: { type: 'base64', media_type: 'image/png', data: pngB64 } },
        { type: 'text', text: `Task: ${task}` },
      ],
    },
  ];
  const msg = await ai.messages.create({ model: 'claude-opus-4-5', max_tokens: 256, system: SYSTEM_PROMPT, messages });
  const block = msg.content[0];
  return JSON.parse(block.type === 'text' ? block.text.trim() : '{}');
}
```

## Step 5  -  Execute actions on the page

Playwright works in pixels; the model works in a 0-1000 space. Scale on the way in.

**Python**

```python
def screenshot_b64(page):
    return base64.b64encode(page.screenshot()).decode()

def to_pixels(page, x, y):
    size = page.evaluate("() => ({ w: window.innerWidth, h: window.innerHeight })")
    return x / 1000 * size["w"], y / 1000 * size["h"]

def execute(page, action):
    t = action["type"]
    if t == "click":
        px, py = to_pixels(page, action["x"], action["y"])
        page.mouse.click(px, py)
    elif t == "type":
        page.keyboard.type(action["text"])
    elif t == "key":
        page.keyboard.press(action["combo"])
    elif t == "scroll":
        page.mouse.wheel(0, 400 if action["direction"] == "down" else -400)
    elif t == "goto":
        page.goto(action["url"])
    elif t == "done":
        return None

    page.wait_for_timeout(500)   # let the UI settle
    return page.screenshot()
```

**TypeScript**

```typescript
import type { Page } from 'playwright';

const screenshotB64 = async (page: Page) =>
  (await page.screenshot()).toString('base64');

async function toPixels(page: Page, x: number, y: number) {
  const size = await page.evaluate(() => ({ w: window.innerWidth, h: window.innerHeight }));
  return { px: (x / 1000) * size.w, py: (y / 1000) * size.h };
}

async function execute(page: Page, action: Record<string, any>): Promise<Buffer | null> {
  switch (action.type) {
    case 'click': {
      const { px, py } = await toPixels(page, action.x, action.y);
      await page.mouse.click(px, py);
      break;
    }
    case 'type':    await page.keyboard.type(action.text); break;
    case 'key':     await page.keyboard.press(action.combo); break;
    case 'scroll':  await page.mouse.wheel(0, action.direction === 'down' ? 400 : -400); break;
    case 'goto':    await page.goto(action.url); break;
    case 'done':    return null;
  }
  await page.waitForTimeout(500);
  return page.screenshot();
}
```

## Step 6  -  Run the full loop

**Python**

```python
TASK = "Find the 'More information...' link on example.com, click it, then report the URL you land on."
MAX_TURNS = 20

history = []
png = page.screenshot()
png_b64 = base64.b64encode(png).decode()

try:
    for turn in range(1, MAX_TURNS + 1):
        action = next_action(TASK, png_b64, history)
        print(f"Turn {turn}: {action}")
        history.append({"role": "assistant", "content": [{"type": "text", "text": json.dumps(action)}]})

        if action["type"] == "done":
            print("Done:", action.get("result"))
            break

        nxt = execute(page, action)
        if nxt is None:
            break
        png_b64 = base64.b64encode(nxt).decode()
    else:
        print(f"Reached max turns ({MAX_TURNS}) without completion.")
finally:
    browser.close()      # detach the driver
    playwright.stop()
    # Free the sandbox browser's memory, then destroy the sandbox.
    requests.delete(f"{API}/sandboxes/{sandbox.id}/browser", headers=HEADERS)
    sandbox.destroy()
```

**TypeScript**

```typescript
const TASK =
  "Find the 'More information...' link on example.com, click it, then report the URL you land on.";
const MAX_TURNS = 20;

const history: Anthropic.MessageParam[] = [];
let pngB64 = await screenshotB64(page);

try {
  for (let turn = 1; turn <= MAX_TURNS; turn++) {
    const action = await nextAction(TASK, pngB64, history);
    console.log(`Turn ${turn}:`, action);
    history.push({ role: 'assistant', content: [{ type: 'text', text: JSON.stringify(action) }] });

    if (action.type === 'done') { console.log('Done:', action.result); break; }

    const next = await execute(page, action);
    if (next === null) break;
    pngB64 = next.toString('base64');
  }
} finally {
  await browser.close();     // detach the driver
  // Free the sandbox browser's memory, then destroy the sandbox.
  await fetch(`${API}/sandboxes/${sandbox.id}/browser`, { method: 'DELETE', headers });
  await sandbox.destroy();
}
```

## Tips for reliable browser agents

- **Wait after every action.** Pages animate and reflow. `page.wait_for_timeout(500)` after each step is cheap insurance against acting on a stale screenshot.
- **Save every screenshot.** The debug loop is: screenshot N shows the state the model reasoned about to produce action N. When the agent gets stuck, inspect the saved frames.
- **Cap `MAX_TURNS` tightly (10-25).** A runaway loop burns credits. Surface an incomplete-task status instead of retrying forever.
- **Prefer `goto` over clicking through navigation** when the goal is simply to reach a URL.
- **Watch it live.** The live viewer shows the same browser your agent is driving, so you can take over when the model stalls instead of restarting the sandbox.
- **Snapshot after login.** If the task needs authenticating, snapshot immediately after login succeeds, then restore for later tasks to skip the login step. See [Snapshots and persistence](/docs/develop/snapshots).

## When to use a Computer instead

The sandbox browser is the default for web automation: it is one browser, it bills as ordinary sandbox usage, and an agent and a person can share it.

Reach for a [Computer](/docs/computers/overview) when the task is not a web page:

- **Full desktop applications.** A native app, an installer, a file manager, or anything that is not a browser tab.
- **Non-browser GUIs.** Windows apps, IDE windows, or a desktop that needs a real window manager.
- **Whole-screen computer use.** When the model must see and control the entire desktop, not one page. See [Computer use](/docs/computer-use).
- **Human takeover of the machine.** When a person needs to drive a real desktop, not just a page.

Computer use drives screenshots, clicks, and key presses against a full desktop VM. The action loop is the same shape as above, but the primitives are `computer.screenshot()`, `computer.left_click()`, `computer.type()`, and `computer.key()`. See [Computer use API reference](/docs/computer-use/api-reference). See also [Computer use vs browser use](/docs/computer-use/vs-browser-use).

## What you learned

- The browser is part of the sandbox; there is no separate machine to boot.
- Playwright connects over CDP to the same browser a viewer can watch, so automation and a human share one session.
- The agent loop is yours: the browser executes actions, your code decides what to do next.
- Coordinates are 0-1000 in the model's view; scale to pixels with the page's inner dimensions.
- Detaching the driver (`browser.close()`) does not stop the sandbox browser; use the browser API to free its memory.

## Next

- [Browser use](/docs/browser-use): The live viewer, CDP, and limits for the sandbox browser.
- [Drive it over CDP](/docs/browser-use/cdp): Playwright and Puppeteer against the sandbox browser.
- [Agent builds an app](/docs/cookbook/agent-builds-app): The full lifecycle: prompt -> files -> dev server -> preview -> production URL.
