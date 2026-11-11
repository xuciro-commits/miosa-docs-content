---
slug: "computers/overview"
title: "Computers (Desktop)"
description: "Durable cloud desktops for browser agents, human takeover, GUI automation, and long-running computer-use workflows."
section: "Computers"
order: 3000
---

A **Computer** is a durable cloud desktop.
It gives each user, agent, or workflow a real Linux GUI with browser state, files, shell access, desktop streaming, and programmatic control.
Use it when the work depends on pixels, sessions, browser logins, desktop apps, accessibility trees, or a human taking over the same machine an agent was using.

For pure code execution, builds, and preview servers, [Sandboxes](/docs/develop/sandboxes) are cheaper and faster.
For a 24/7 production web app, use [Docker Host](/docs/docker-host/docker-deploy).
Still deciding? See [Computers vs sandbox](/docs/computers/vs-sandbox).

## 1. Create a Computer

**CLI**

```bash
miosa computer create --size small
miosa computer list
```

**TypeScript**

```typescript
import { Miosa } from "@miosa/sdk"

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! })

const computer = await miosa.computers.create({
  name: "agent-1",
  template_type: "miosa-desktop",
  size: "small",
})
await computer.start()
```

**Python**

```python
import miosa, os

client = miosa.Miosa(api_key=os.environ["MIOSA_API_KEY"])

computer = client.computers.create(
    name="agent-1",
    template_type="miosa-desktop",
    size="small",
)
computer.start()
```

Pass `external_workspace_id` to attribute the Computer to one of your tenants.
See [Lifecycle and sizing](/docs/computers/lifecycle) for states, sizes, and workspace scoping.

## 2. Drive the desktop

**CLI**

```bash
miosa api GET /computers/cmp_123/desktop/screenshot > screen.png
```

**TypeScript**

```typescript
const png = await computer.screenshot()
await computer.leftClick(640, 480)
await computer.type("hello world")
await computer.key("Return")
await computer.bash("firefox &")
```

**Python**

```python
png_bytes = computer.screenshot()
computer.left_click(640, 480)
computer.type("hello world")
computer.key("Return")
computer.bash("firefox &")
```

Start each loop with an observation, act once, then observe again.
The full action set is in [Computer use](/docs/computer-use).

## 3. Watch it live

A Computer is also a live pixel stream.
Open it from the MIOSA console, or embed the same stream in your app with a short-lived URL minted on your backend.

```typescript
const urls = await computer.urls()
console.log(urls.desktop_url)
```

See [Embedding and streaming](/docs/computers/embedding) for the token and iframe flow.

## Pick what you need

| You want to | Start here |
|---|---|
| Create, size, and manage the machine | [Lifecycle and sizing](/docs/computers/lifecycle) |
| Choose a Computer or a Sandbox | [Computers vs sandbox](/docs/computers/vs-sandbox) |
| See what ships in the box | [Inside a Computer](/docs/computers/anatomy) |
| Drive the desktop from code | [Computer use](/docs/computer-use) |
| Embed the live desktop in your UI | [Embedding and streaming](/docs/computers/embedding) |
| Ship it safely | [Production checklist](/docs/computers/production-checklist) |
| Measure startup | [Fast start and readiness](/docs/computers/fast-start) |

## Next

- [Computers vs sandbox](/docs/computers/vs-sandbox): Pick the right machine for the workload, with the rule of thumb.
- [Inside a Computer](/docs/computers/anatomy): What ships in the box and how the desktop, disk, stream, and API fit together.
- [Lifecycle and sizing](/docs/computers/lifecycle): Fast startup, the state machine, resource sizes, and workspace-scoped creation.
- [Embedding & streaming](/docs/computers/embedding): Access model, the operator workflow, and embedding the live desktop in your UI.
- [Production checklist](/docs/computers/production-checklist): The defaults to verify before you rely on a Computer in production.
- [Computer use](/docs/computer-use): Full action reference: screenshot, click, type, key, scroll, drag, windows, accessibility tree.
