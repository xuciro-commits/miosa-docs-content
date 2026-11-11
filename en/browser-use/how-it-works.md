---
slug: "browser-use/how-it-works"
title: "How browser use works"
description: "The parts of a sandbox browser, the three surfaces that see it, and how the live view and CDP sockets are exposed."
section: "Sandboxes"
order: 2024
---

## What you get

- **A live browser**: Start it in any running sandbox and watch the page render in real time on the Sandbox page.
- **Take control**: A viewer can take control and use the browser directly, or share a revocable view-only link.
- **Agent access over CDP**: Playwright and Puppeteer connect over CDP and drive the same browser the viewer is watching.
- **Sandbox-priced**: It runs inside the sandbox you already have, so it bills as plain sandbox usage.

## How it fits together

- **Start.** Ask the sandbox to start its browser. It starts lazily on the first request, stays up across pause and resume, and stops when you stop it.
- **Watch and drive.** The Sandbox page shows the live browser. A viewer can take control; a view-only link can watch but never drive.
- **Automate.** Playwright or Puppeteer connect to the live browser over CDP and act on the same session a person sees.

Because the viewer and the automation share one browser, a person can open the page mid-run, watch, and take control without losing cookies, logins, or page state.

## The three viewers of one browser

| Surface | Who | What they can do |
|---|---|---|
| Live viewer on the Sandbox page | The signed-in owner | Watch the browser, take control |
| View-only share link | Anyone with the link | Watch only; cannot drive the browser |
| CDP endpoint | An agent (Playwright, Puppeteer) | Drive the browser over the Chrome DevTools Protocol |

> **Caution: View-only means view-only**
>
> A view-only share link cannot type, click, or navigate. Input is rejected inside the browser process in the sandbox, not just hidden in the UI.

> **Note: Everything goes through api.miosa.ai**
>
> Both sockets - the live-view stream and the CDP endpoint - are on `api.miosa.ai`, with normal API authentication.
> Nothing is exposed on the sandbox's own host or on a template port.

## See also

- **Drive it over CDP**: Playwright and Puppeteer connecting to the same browser.
  [Open](/docs/browser-use/cdp)
- **Limits, billing, security**: Minimum size, lifecycle, billing, and how access is enforced.
  [Open](/docs/browser-use/limits)
