---
slug: "platform/agents-in-your-product"
title: "Agents in your product"
description: "Give each of your users a machine with an agent or harness working inside it, and show the result in your own UI."
section: "Platform builders"
order: 9003
---

A platform makes a machine for each of your users, then puts an agent or harness inside it.
The agent writes files, runs commands, and opens ports.
Your product shows the result: a preview URL, a terminal, or a screenshot.

## One machine per user

One sandbox or computer per end user, never shared across customers.

Each machine has its own filesystem, network, and process tree.
Pick sandboxes for code and previews, and computers when an agent needs a desktop or browser: [Agent device selection](/docs/guides/agent-device-selection).

A code agent gets a sandbox, and a browsing agent gets a computer.
Both get their own filesystem and network.

## Let your user bring their own model account

An agent run thinks with an account that belongs to its owner.
Your customer can connect their own Claude plan, ChatGPT plan or model key from inside your product, with no console visit.
Every run on their machines then uses it, before the workspace's.

```typescript
const signin = await miosa.connections.connectSubscription({ provider: "claude", externalUserId: "cus_123" })
// show signin.verificationUrl in your UI, then: await signin.submitCode(pastedCode)
```

See [Connect agent accounts](/docs/guides/connect-agent-accounts).

## Show the result to your user

The browser only ever sees preview URLs and scoped, expiring tokens.
Mint short-lived [browser tokens](/docs/platform/browser-tokens) for previews and terminals.

To show a preview inside your product, mint a share token on the server and send only the URL to the browser:

```typescript
// GET /embed-url on your backend
const share = await sandbox.previews.share(preview.id, { expiresInSec: 3600 })
return { embedUrl: share.share_url }
```

The token is bound to that one preview and expires.

Never put an `msk_*` key in a browser.
Treat it like a Stripe secret key: server-only, rotated on a schedule, audited.

Always display the returned `public_url` instead of building hostnames yourself.

## Next

- [Agent operating guide](/docs/guides/agent-operating-guide): Run an AI agent against a sandbox or computer.
- [Agent Runs](/docs/api-reference/agent-runs): Dispatch work, stream events, and collect artifacts.
- [Connect agent accounts](/docs/guides/connect-agent-accounts): Keys and subscription plans for your workspace or for each end user.
- [Browser tokens](/docs/platform/browser-tokens): Short-lived tokens for previews and terminals in the end user's browser.
- [Agent device selection](/docs/guides/agent-device-selection): Choose a sandbox or a computer for each agent.
