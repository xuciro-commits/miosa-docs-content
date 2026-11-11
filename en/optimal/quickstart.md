---
slug: "optimal/quickstart"
title: "Optimal quickstart"
description: "Ask Optimal a question, build and preview a small app on a sandbox, then send your first message over the Optimal API."
section: "Optimal"
order: 7001
---

This walkthrough takes about five minutes.
You ask Optimal a question, have it build a page on a sandbox, and then send the same kind of message from your own code.

## Before you start

- A MIOSA account with a workspace.
- Wallet credits, or a paid plan.
  On the free plan with no credits left, the Optimal brain is disabled until you add credits.

## 1. Ask a question

1. Open **Optimal** in the console sidebar.
2. Pick a single workspace in the workspace switcher.
3. Leave the mode on **Ask** and choose the starter "What needs my attention in this workspace?", or type your own question.
4. Press **Send message**.

Optimal reads your live workspace and answers in the chat.
Ask mode never touches a machine.

## 2. Build something

1. Open the mode menu in the composer and pick **Build**.
2. Type "Build a landing page and publish it", then send.
3. Optimal asks where to build.
   In the machine menu, pick **New sandbox**, or pick a machine you already have.
4. The side panel opens.
   Watch **Progress** for the steps, and open **Preview** when the page is served.

Steps that change your workspace wait for your approval.
Press **Approve** on the card above the composer, or **Allow for this chat** to stop being asked for that kind of step in this chat.

When the page looks right, press **Publish** on the preview card to put it on a public URL.
See [Build on your machines](/docs/optimal/machines).

## 3. Send a message from code

Optimal has an HTTP API at `https://optimal.miosa.ai/api/v1`.
It takes a short-lived Optimal token, which your API key mints.

Create an API key with the **Chat with Optimal and start runs** permission (`osa:write`), then:

```bash
export MIOSA_API_KEY=msk_your_key
export WORKSPACE=your_workspace_id

# 1. Mint a 10-minute Optimal token for one workspace.
TOKEN=$(curl -s https://api.miosa.ai/api/v1/optimal/workspace-token \
  -H "Authorization: Bearer $MIOSA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"workspace_id\": \"$WORKSPACE\"}" | jq -r .token)

# 2. Start a chat.
CHAT=$(curl -s https://optimal.miosa.ai/api/v1/sessions \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"resume": false}' | jq -r .id)

# 3. Open the stream in the background, so you see the turn from its start.
curl -sN "https://optimal.miosa.ai/api/v1/sessions/$CHAT/stream" \
  -H "Authorization: Bearer $TOKEN" &

# 4. Send a message. This call returns at once with 202; the reply arrives on the stream.
curl -s https://optimal.miosa.ai/api/v1/sessions/$CHAT/message \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Summarize what changed in the last day", "mode": "ask"}'
```

The stream sends `streaming_token` events while the reply is written and an `agent_response` event with the full reply.
`GET /sessions/$CHAT/messages` returns the finished transcript at any time.
The chat also appears in **History** in the console.
See the [Optimal API](/docs/optimal/api) for every endpoint.

> **Note: Tokens expire after 10 minutes**
>
> Mint a new Optimal token when a call answers `401`.
> The chat keeps going; only the token is new.

## Next

- [Chats and modes](/docs/optimal/chats): attachments, queued messages, and context.
- [Approvals](/docs/optimal/approvals): choose what Optimal asks before.
- [Pricing and usage](/docs/optimal/pricing): what a chat costs and where to see it.
