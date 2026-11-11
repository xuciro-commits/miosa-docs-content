---
slug: "optimal/approvals"
title: "Approvals"
description: "Decide which Optimal steps wait for you, approve or decline them in the chat, allow a step for the rest of a chat, and let an API key approve."
section: "Optimal"
order: 7005
---

Optimal asks before it acts.
When a step needs your decision, the chat pauses and an approval card appears above the composer.

## Approve or decline

An approval card names the step Optimal wants to take.

- **Approve** lets that one step run.
- **Decline** refuses it, and Optimal continues another way or tells you what it could not do.
- **Allow for this chat** approves it and lets the same kind of step run without asking for the rest of the chat.
  It shows only for steps that can be remembered.

When several cards are waiting, the newest shows first with "N more waiting".

## Choose what Optimal asks before

In **Preferences**, under **Approvals**, **Ask before** sets which steps wait for you in the workspace:

| Setting | Steps that wait |
|---|---|
| Every step | Every step that uses a tool |
| Changes only | Steps that change something |
| Deletions only | Steps that delete something |

The default is **Changes only**.
The setting is yours: it applies to your chats in this workspace, on every device.

## Auto-approve

The **Auto-approve** switch sits in the chat header of an Optimal chat.
When it is on, Optimal runs every tool it asks for in this workspace without stopping.
Turn it off to go back to the **Ask before** setting.

On an empty chat, the same switch sits under the composer.
Auto-approve is offered for the Optimal brain only.

## Allowed in this chat

Every step you allowed with **Allow for this chat** is listed under **...** > **Allowed in this chat (N)** in the chat header.
**Remove** puts a step back to asking.
When nothing is allowed, the list reads "Nothing is allowed without asking in this chat."

## Every pending approval

**Approvals** in the Optimal sidebar lists approvals across your chats in the workspace.
Each row says which chat it came from, and you can search the list.
A pending approval expires when the turn that asked for it ends.

## Signing in to a site

When Optimal needs to sign in to a site in its browser, it shows a secure sign-in card instead of asking in the chat.
What you type goes straight to the sign-in page and is not saved in the chat.
Tick **Save for next time** to keep the sign-in; saved sign-ins are listed in **Preferences** under **Sign-ins**, with a masked name only.
**Not now** skips it.

## Approvals over the API

An Optimal token minted from an API key can approve or reject only if the key has the **Approve or reject Optimal's changes** permission (`osa:approve`).
The token's `can_decide_approvals` field says whether it can.
Without it, the decision routes answer `403 approval_scope_required`, so a script cannot approve its own changes unless you allowed that.

```bash
# Pending approvals in one chat
curl -s "https://optimal.miosa.ai/api/v1/approvals?session_id=$CHAT&status=pending" \
  -H "Authorization: Bearer $TOKEN"

# Approve one
curl -s -X POST https://optimal.miosa.ai/api/v1/approvals/$APPROVAL/approve \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reason": "Looks right"}'
```

`/reject` declines.
Deciding twice answers `409 already_decided`.
See the [Optimal API](/docs/optimal/api#approvals).

> **Caution: Auto-approve follows you**
>
> Auto-approve and **Ask before** apply to your Optimal chats in the workspace, on every device.
> Platform policy still applies on top.

## Related

- [Isolation and permissions](/docs/optimal/isolation): what Optimal can reach at all.
- [Tasks, artifacts and history](/docs/optimal/workspace): the other Optimal sidebar pages.
