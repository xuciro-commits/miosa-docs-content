---
slug: "optimal/troubleshooting"
title: "Optimal troubleshooting"
description: "Fixes for common Optimal problems - no credits, no workspace, a missing machine, attachments, harness sign-in, taking over, and API errors."
section: "Optimal"
order: 7011
---

Each entry starts with what you see, then the fix.

## In the console

### "Optimal runs on credits"

Optimal needs a positive credit balance, and yours is empty.
Press **Add credits**, or **Upgrade** for a plan with credits, then send again.
In the brain picker the same state reads "Add credits to use Optimal".

A harness or saved agent does not use credits for its model, so you can still use one of those.

### "Choose or create a workspace to start an isolated Optimal chat."

No workspace is selected.
Pick one in the workspace switcher.
Under **All workspaces**, Optimal reads every workspace but asks which one to use before it changes anything.

### Investigate is greyed out

Investigate always runs on the Optimal brain.
The chat's brain is a harness or a saved agent, and a chat keeps its brain.
Start a new chat with **New** and pick Optimal.

### The brain picker will not change

The brain is fixed after the first message.
Picking a different brain starts a new chat.

### A harness shows "Not connected" or "Needs signing in again"

Press **Connect** or **Reconnect** on its row in the brain picker, sign in, and you return to Optimal.
See [Harnesses and auth](/docs/agents/harnesses).

### "This chat's machine was deleted. Pick another, or let Optimal choose."

The sandbox or computer bound to the chat no longer exists.
Pick a machine in the machine menu, or send in Build mode and choose **New sandbox**.

### "... is kept on this chat's machine: pick a sandbox or computer, then send again."

The file is not an image or a document, so it is written to the chat's machine, and the chat has none.
Pick a machine in the machine menu, then send the file again.
Files are limited to 10 MB each and 25 MB per chat.

### "Interrupted. The reply stopped early."

The reply was cut off before it finished.
Press **Try again** next to the note; the chat keeps everything before it.

### "Couldn't take over. Try again." or "Couldn't hand it back. Try again."

The machine did not answer in time.
Wait a moment and press **Take over** or **Hand back** again.
Check the **Machine** tab to see that the machine is running.

### Publish is not available

The machine has to be running to publish.
Resume it from the **Machine** tab or the machine menu, then press **Publish** again.

### The context meter is red

The chat has used most of its 1M-token context.
Nothing is needed: Optimal folds the older part into a summary on its own and shows "Compacting context..." while it does.

## Over the API

| You get | Fix |
|---|---|
| `401` with `MISSING_TOKEN` | Send `Authorization: Bearer <Optimal token>`, not your API key |
| `401` on a call that worked before | The Optimal token expired after 10 minutes; mint a new one |
| `403 FORBIDDEN` when minting | Give the API key the `osa:write` permission |
| `403 API_KEY_WORKSPACE_BOUND` | Mint the token for the workspace the key is bound to |
| `403 approval_scope_required` | Give the API key the `osa:approve` permission, then mint a new token |
| `402 OPTIMAL_CREDITS_REQUIRED` | Add credits or upgrade |
| `404 session_not_found` | The chat id is wrong, deleted, or in another workspace; check the token's workspace |
| `409 machine_switch_requires_confirmation` | Send the message again with `"confirm_switch": true` |
| `409 machine_unavailable` | Start the machine, then retry |
| `409 no_machine` on an upload | Bind a machine with `POST /sessions/{id}/machine` first |
| `415 unsupported_media_type` | Add `Content-Type: application/json` |
| `429 rate_limited` | Wait for the `Retry-After` seconds |
| A stream that shows nothing | Open the stream before you send, or read finished turns from `GET /sessions/{id}/messages` |

See the [Optimal API](/docs/optimal/api) for every endpoint.

## Related

- [Pricing and usage](/docs/optimal/pricing): credits and what a chat costs.
- [Build on your machines](/docs/optimal/machines): the machine menu and side panel.
- [Troubleshooting](/docs/troubleshooting): problems outside Optimal.
