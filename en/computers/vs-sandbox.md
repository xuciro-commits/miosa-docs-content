---
slug: "computers/vs-sandbox"
title: "Computers vs sandbox"
description: "Choose a Computer or a Sandbox for a workload, with the rule of thumb and a side-by-side table."
section: "Computers"
order: 3002
---

> **Note: The rule**
>
> If the user or agent needs to see and operate a real screen, use a Computer.
> If the workload only needs a shell, filesystem, process, or preview port, use a Sandbox.

## Pick by workload

| Workload | Pick | Why |
|---|---|---|
| Run tests, install packages, build code, expose a dev server | **Sandbox** | Faster and cheaper shell-first compute |
| Computer-use agent that must click through a website | **Computer** | Needs pixels, browser state, and input events |
| Logged-in browser session that must survive across tasks | **Computer** | Browser profile persists with the VM |
| Headless Playwright, API tests, CLI workflows | **Sandbox** | No visible desktop needed |
| Screenshot-based QA or visual inspection | **Computer** | Uses the rendered screen as truth |
| Desktop apps, extensions, file pickers, or manual takeover | **Computer** | Needs a full GUI and streamable session |
| Public 24/7 app URL for customers | **Docker Host** | Built for durable deployments and custom domains |

## What a Computer is for

| Use | What it gives you |
|---|---|
| Agent browsers | Run a browser session that stays logged in, exposes pixels to agents, and can be resumed by a human. |
| Human handoff | Let an operator open the same live desktop, inspect the state, and continue from where automation stopped. |
| GUI automation | Drive desktop apps with mouse, keyboard, clipboard, windows, accessibility, and shell commands. |
| Persistent workspaces | Keep files, browser profiles, installed tools, and local state across stop/start cycles. |

## See also

- **Computer use**: Full action reference: screenshot, click, type, key, scroll, drag, hotkey, windows, accessibility tree, and more.
  [Open](/docs/computer-use)
- **Embedding & Streaming**: Embed the live pixel stream in your own UI using short-lived stream tokens.
  [Open](/docs/computers/embedding)
- **BYOC / OpenComputers**: Bring your own hardware (Mac, Linux, Windows).
  One command registers your machine as a MIOSA computer.
  [Open](/docs/opencomputers)
