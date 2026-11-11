---
slug: "cookbook/browser-agent"
title: "浏览器智能体"
description: "将 LLM 放入在沙箱内运行的真实浏览器的循环中。启动沙箱浏览器，使用 Playwright 通过 CDP 驱动它，然后运行屏幕截图 → 操作 → 屏幕截图循环。"
section: "Sandboxes"
order: 2034
---

`~12 min` `Python` `TypeScript`

> **目标**：在 MIOSA 沙箱内运行 LLM 驱动的浏览器智能体。启动沙盒浏览器，通过 CDP 将 Playwright 连接到它，然后循环：屏幕截图，询问模型要做什么，在同一页面上执行操作。
>
> **您将使用什么**：沙盒、沙盒浏览器、Playwright over CDP

## 你将构建什么

`BrowserAgent` 表示：

1. 创建沙箱并启动其浏览器。
2. 将 Playwright 连接到实时查看器显示的同一浏览器。
3.进入截图->LLM->动作循环。
4. 当模型发出完成信号时停止。

LLM 是您的选择。下面的代码使用 Anthropic Claude 的视觉 API，但该模式与模型无关。

> **注意：浏览器是沙箱的一部分**
>
> 浏览器使用在沙箱内运行，因此无需启动单独的计算机，也无需等待桌面。请参阅[浏览器使用](/docs/browser-use)了解完整内容。

## 先决条件

- MIOSA 工作区 API 密钥 (`msk_u_*`) - 请参阅 [API 密钥](/docs/platform/api-keys)
- Anthropic API 密钥（或替换您首选的具有视觉功能的模型）
- 节点 22+ 或 Python 3.11+

## 步骤 1 - 安装和配置

**Python**

```bash
pip install miosa anthropic playwright
playwright install chromium
export MIOSA_API_KEY="msk_u_..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

**打字稿**

```bash
npm install @miosa/sdk @anthropic-ai/sdk playwright
npx playwright install chromium
export MIOSA_API_KEY="msk_u_..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

Playwright 作为驱动程序在您的机器上运行。它驱动的浏览器在沙箱内运行。

## 步骤 2 - 创建沙箱并启动其浏览器

创建一个沙箱，然后启动其浏览器。创建时不会选择有关浏览器的任何内容：任何模板都可以工作，只要其图像包含 Chromium 和 Node（标准图像即可）。

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

**打字稿**

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

> **注意：端点**
>
> `POST /browser` 延迟启动浏览器并返回 `stream_url`、`cdp_url` 和 `stream_auth`。
> 两个套接字均位于 `api.miosa.ai` 上。有关完整字段集，请参阅[浏览器使用快速入门](/docs/browser-use/quickstart)。

## 步骤 3 - 通过 CDP 连接 Playwright

沙盒浏览器使用 Chrome DevTools 协议。将 Playwright 连接到 `cdp_url`（将 `stream_auth` 标记附加为 `?token=`）并重用已打开的上下文和页面，以便您对查看者将看到的会话进行操作。

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

**打字稿**

```typescript
import { chromium } from 'playwright';

const browser = await chromium.connectOverCDP(cdpUrl);
const context = browser.contexts()[0];
const page = context.pages()[0] ?? (await context.newPage());

await page.goto('https://example.com');
console.log('Loaded:', await page.title());
```

> **注意：分离，不要杀死**
>
> Playwright 中的 `browser.close()` 会分离您的驱动程序。它不会停止沙箱浏览器，因此查看者保持连接。要停止浏览器并释放其内存，请使用浏览器 API。

## 第 4 步 - 截图、决定、行动

该循环与任何视觉智能体相同：捕获屏幕截图，询问模型下一步要做什么，采取行动，重复。模型返回一个 JSON 操作；您的代码在页面上执行它。

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

**打字稿**

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

## 步骤 5 - 在页面上执行操作

Playwright以像素为单位进行创作；该模型在 0-1000 的空间内工作。在进来的路上规模。

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

**打字稿**

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

## 步骤 6 - 运行完整循环

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

**打字稿**

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

## 可靠浏览器智能体的提示

- **每次操作后等待。** 页面动画并重排。 `page.wait_for_timeout(500)` 在每一步之后都是廉价的保险，可以防止对过时的屏幕截图进行操作。
- **保存每个屏幕截图。** 调试循环是：屏幕截图 N 显示模型推理产生动作 N 的状态。当智能体卡住时，检查保存的帧。
- **紧紧盖住`MAX_TURNS` (10-25)。** 失控循环会消耗积分。显示未完成的任务状态，而不是永远重试。
- **当目标只是到达某个 URL 时，首选 `goto` 而不是点击导航**。
- **实时观看。** 实时查看器显示您的智能体正在驾驶的同一浏览器，因此您可以在模型停止时接管而不是重新启动沙箱。
- **登录后快照。** 如果任务需要身份验证，则登录成功后立即快照，然后恢复以供后续任务跳过登录步骤。请参阅[快照和持久性](/docs/develop/snapshots)。

## 何时使用计算机

沙箱浏览器是 Web 自动化的默认设置：它是一个浏览器，按普通沙箱使用计费，智能体和个人可以共享它。

当任务不是网页时，请访问 [计算机](/docs/computers/overview)：- **完整的桌面应用程序。** 本机应用程序、安装程序、文件管理器或浏览器选项卡以外的任何内容。
- **非浏览器 GUI。** Windows 应用程序、IDE 窗口或需要真正窗口管理器的桌面。
- **全屏计算机使用。** 当模型必须查看和控制整个桌面而不是一页时。请参阅[计算机使用](/docs/computer-use)。
- **人类接管机器。** 当一个人需要驱动一个真正的桌面，而不仅仅是一个页面时。

计算机的使用驱动了针对完整桌面虚拟机的屏幕截图、点击和按键操作。动作循环的形状与上面相同，但基元是 `computer.screenshot()`、`computer.left_click()`、`computer.type()` 和 `computer.key()`。请参阅[计算机使用 API 参考](/docs/computer-use/api-reference)。另请参阅[计算机使用与浏览器使用](/docs/computer-use/vs-browser-use)。

## 你学到了什么

- 浏览器是沙箱的一部分；没有单独的机器可以启动。
- Playwright通过 CDP 连接到观众可以观看的同一浏览器，因此自动化和人类共享一个会话。
- 智能体循环是你的：浏览器执行操作，你的代码决定下一步做什么。
- 模型视图中的坐标为0-1000；根据页面的内部尺寸缩放到像素。
- 分离驱动程序（`browser.close()`）不会停止沙箱浏览器；使用浏览器 API 释放其内存。

## 下一步

- [浏览器使用](/docs/browser-use)：实时查看器、CDP 和沙箱浏览器的限制。
- [通过 CDP](/docs/browser-use/cdp) 驱动它：Playwright 和 Puppeteer 对抗沙盒浏览器。
- [Agent 构建应用程序](/docs/cookbook/agent-builds-app)：完整生命周期：提示 -> 文件 -> 开发服务器 -> 预览 -> 生产 URL。
