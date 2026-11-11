---
slug: "computer-use/vs-browser-use"
title: "Computer use vs browser use: which to pick"
description: "A short comparison of MIOSA browser use inside a sandbox and full computer use on a durable desktop, to pick the right one for a task."
section: "Computers"
order: 3012
---

Both drive a real browser. They differ in how much machine you rent.

> **Note: The rule**
>
> If the task is a web page an agent or a person should watch, use [Browser use](/docs/browser-use) - it runs in any sandbox and bills as ordinary sandbox usage.
> If the task needs a full desktop, other apps, or takeover of the whole screen, use [Computer use](/docs/computer-use).

| | Browser use | Computer use |
|---|---|---|
| What runs | A live browser inside a sandbox | A full Linux desktop on a Computer |
| Where | Any sandbox, any template | A Computer you create and start |
| Setup | On by default; nothing to choose at create time | Create a Computer, then start it |
| Live viewer | On the Sandbox page: take control, view-only share links, full screen | A streamable desktop, embeddable with a short-lived URL |
| Agent access | Over CDP - Playwright, Puppeteer | Screenshot and control APIs: mouse, keyboard, windows, accessibility |
| Human takeover | The browser view | The whole desktop |
| Other apps | No - browser only | Yes - any installed Linux app |
| Persistence | Survives pause and resume | Survives stop and start |
| Billing | Plain sandbox usage | Computer compute while running |
| Best for | Web automation, scraping, logged-in browser work that should be watched | GUI apps, desktop workflows, screenshot QA, full human handoff |

## Pick browser use when

- The task lives entirely on web pages.
- You want the cheapest isolated environment: a sandbox you already use for code.
- A person should be able to watch, take control, or share a view-only link.
- Playwright or Puppeteer is your driver.

## Pick computer use when

- The task needs desktop apps, file pickers, extensions, or multiple windows.
- The agent works from pixels and the accessibility tree across the whole screen.
- A human must take over the exact machine an agent was using.
- You need a durable desktop that keeps its browser profile and installed tools across restarts.

## Next

- [Browser use](/docs/browser-use) - start a live browser in any sandbox.
- [Computer use](/docs/computer-use) - the desktop perception-action loop.
