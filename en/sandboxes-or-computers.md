---
slug: "sandboxes-or-computers"
title: "Sandboxes or Computers?"
description: "Pick where your app or agent runs on MIOSA: a Sandbox for code, previews and a built-in browser, a Computer for a full desktop, Docker Host to publish."
section: "Get started"
order: 6
---

Most work starts in a **Sandbox**.
Reach for a **Computer** when someone has to see and use a real screen.
When the result should stay online for your users, publish it with **Docker Host**.

- [Sandbox](/docs/develop/sandboxes): A fast, isolated machine for code. Run commands, edit files, install packages, open a live preview URL, and drive the built-in browser. Pauses when idle and wakes on the next request.
- [Computer](/docs/computers/overview): A durable cloud desktop. A real Linux screen with a browser, desktop apps, mouse and keyboard control, a live view a person can take over, and state that stays between sessions.

**Ready to go live?** [Docker Host](/docs/docker-host/docker-deploy) keeps what you built online at a stable URL, with your own domain, versions and rollback.

## Pick by what you need

| You want to | Use |
|---|---|
| Run code, build, test, or install packages | Sandbox |
| Show a live preview of an app while you build it | Sandbox |
| Let an agent browse and fill forms in a browser | Sandbox, with its [built-in browser](/docs/browser-use) |
| Use desktop apps, or anything that needs a full screen | Computer |
| Keep logins, files and a desktop around for days | Computer |
| Let a person watch and take over an agent's screen | Either: the sandbox browser or the computer desktop both have a live view |
| Keep an app online for your users | Docker Host |

> **Tip: Not sure? Start with a Sandbox**
>
> Sandboxes start fastest and cost the least.
> Move to a Computer only when the work truly needs a desktop.

## Create one

**CLI**

```bash
# A sandbox for code and previews
miosa create

# A computer with a full desktop
miosa computer create --size small
```

**TypeScript**

```ts
import { Miosa } from "@miosa/sdk";

const miosa = new Miosa({ apiKey: process.env.MIOSA_API_KEY! });

const sandbox = await miosa.sandboxes.create({ templateId: "miosa-sandbox" });
const computer = await miosa.computers.create({ name: "research-desk", size: "small" });
```

## Often you use both

A typical agent product writes and runs code in a Sandbox, uses a Computer when a task needs a real desktop, and publishes the finished app with Docker Host.
See [Agent device selection](/docs/guides/agent-device-selection) for the full decision guide, including when an agent needs no machine at all.

## Next steps

- [Quickstart](/docs/quickstart): create your first sandbox in minutes.
- [Computers](/docs/computers/overview): lifecycle, streaming and embedding.
- [Build an app like...](/docs/build): pick a product and get a step-by-step guide.
