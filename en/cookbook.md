---
slug: "cookbook"
title: "Cookbook"
description: "Every MIOSA recipe in one place, grouped by area. Sandbox procedures and platform-builder walkthroughs, each with a working code path."
section: "Sandboxes"
order: 2030
---

Practical walkthroughs for the things customers build on MIOSA. Each recipe starts with a product goal and ends with a running result, and every one is grounded in the current SDK and API.

The recipes live next to the product they belong to in the sidebar, under **Sandboxes > Recipes** and **Platform builders > Recipes**. This page is the index of all of them.

## Sandbox recipes

For running code, agents, and browsers inside a sandbox.

- [Agent builds an app](/docs/cookbook/agent-builds-app): The full lifecycle: user prompt to files, dev server, live preview, feedback loop, publish, and a stable production URL.
- [Agent with secrets](/docs/cookbook/agent-with-secrets): Inject API keys and OAuth credentials into a sandbox without exposing them in code, logs, or the browser.
- [Streaming code execution](/docs/cookbook/streaming-code-execution): Run code and stream stdout to your UI in real time: graders, workflow builders, data jobs, and AI tool execution.
- [Browser agent](/docs/cookbook/browser-agent): Drive the sandbox browser with an LLM over CDP: screenshot, decide, act, and watch the same session live.

## Platform builder recipes

For embedding MIOSA into your own product and charging your customers for it.

- [Multi-tenant SaaS](/docs/cookbook/multi-tenant-saas): Tag every sandbox with your own workspace and user IDs, query usage per tenant, and hand the browser a short-lived preview share token.
- [Per-user quotas and billing](/docs/cookbook/per-user-quotas-billing): Cap end-users with max_sandboxes and credit limits, pull monthly rollups keyed by external_user_id, and wire the totals into Stripe.
- [Embed a sandbox in an iframe](/docs/cookbook/embed-sandbox-iframe): Your server mints a preview token; your frontend drops it into an iframe. React, Vue, Svelte, and vanilla variants, with TTL refresh.
- [Realtime sandbox events](/docs/cookbook/sandbox-events-realtime): React to sandbox lifecycle events through HMAC-signed webhooks or the SSE stream, with idempotency patterns.
- [Build a virtual device product](/docs/cookbook/build-virtual-device-product): A product blueprint for exposing a device per user: MIOSA mapping, architecture, data model, and jobs.

## Before you start

All recipes share the same prerequisites:

- A MIOSA workspace API key (`msk_u_*`)  -  see [API Keys](/docs/platform/api-keys)
- Node 22+ **or** Python 3.11+ installed locally
- The MIOSA SDK installed for your language:

```bash
# Python
pip install miosa

# TypeScript / Node
npm install @miosa/sdk
```

Set the key in your environment once  -  every recipe reads it from there:

```bash
export MIOSA_API_KEY="msk_u_..."
```
