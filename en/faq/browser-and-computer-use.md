---
slug: "faq/browser-and-computer-use"
title: "Browser and computer use"
description: "Where browser automation runs, what starting a browser returns, CDP, live takeover, and billing."
section: "Help"
order: 16003
---

Short answers on browser automation, the computer-use loop, and live takeover.

## Where does browser automation run?

Inside a sandbox. The browser is part of the sandbox and starts lazily on the first browser request, then survives pause and resume.
See [Browser use](/docs/browser-use).

## What does starting a browser return?

A `stream_url` (live view), a `cdp_url` (Chrome DevTools Protocol), a one-hour `stream_auth` token, and a 1280x720 viewport.
Every sandbox image includes Chromium and Node, so any sandbox works.
See [Browser use quickstart](/docs/browser-use/quickstart).

```bash
curl -X POST https://api.miosa.ai/api/v1/sandboxes/$SANDBOX_ID/browser \
  -H "Authorization: Bearer $MIOSA_API_KEY"
```

## How does an agent attach to the browser?

Connect over CDP to the same browser a human can watch, so cookies and logins are shared.
Detach when you are done rather than calling `browser.close()`.
See [Drive the browser over CDP](/docs/browser-use/cdp).

## Can a human watch or take over a session?

Yes. Share a live viewer link for browser sessions, or take over a Computer desktop directly.
The live viewer needs the sandbox's virtual display, which a SOMA sandbox does not have yet.
See [Live viewer](/docs/browser-use/live-viewer) and [Embed a live desktop](/docs/computers/embedding).

## What is the computer-use loop?

Screenshot, decide, take one action, screenshot again.
Coordinates are screen pixels with the origin at the top left.
See [Computer use quickstart](/docs/computer-use/quickstart).

## How is browser and computer use billed?

Browser use bills as ordinary sandbox usage.
See [Browser use limits](/docs/browser-use/limits) and [Pricing and limits](/docs/pricing).
