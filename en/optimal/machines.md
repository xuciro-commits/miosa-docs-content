---
slug: "optimal/machines"
title: "Build on your machines"
description: "Choose the sandbox or computer Optimal works on, use the side panel to watch and steer it, publish a preview, and take over from the agent at any time."
section: "Optimal"
order: 7004
---

Optimal does its building on a machine: a sandbox or a computer in your workspace.
You choose which one, and you can watch, steer, or take over at any time.

## Choose a machine

The machine menu sits in the composer toolbar.
It shows in Build mode, or once the chat has a machine.
Its label is **Auto** until you pick one, then the machine's name.

When you send a Build message and the chat has no machine, the menu opens on its own.
You can:

- pick **New sandbox**, with a size and its hourly price;
- pick **New computer**, with its hourly price;
- pick a machine you already have, under **Other machines in this workspace**.

A new sandbox has an idle timeout of 30 minutes.

A harness or saved agent always asks where to work, in any mode, because each message is a run on a machine.

## Machines in this chat

Under **This chat** the menu lists the chat's machines.
For each machine you can **Open** it, choose **Run next message here**, pause or resume it, stop it, or delete it.
**Add a sandbox** and **Add a computer** add another.

At the bottom of the menu:

| Action | What it does |
|---|---|
| **Pause all** | Pauses every machine in the chat |
| **Remove all** | Deletes every machine of the chat for good, after a confirm; their files are lost unless you took snapshots |
| **See all** | Opens the full machine list |
| **Show the preview** | Opens the Preview tab |
| **Detach from this chat** | Unbinds the machine; it keeps running |

If Optimal wants to move the chat to a different machine, it asks "Move this chat to ...?" with **Keep current** and **Move it**.

## The side panel

When work starts, the side panel opens on the right and follows the chat's machine.
Toggle it with **Open side panel** and **Close side panel**, and drag its edge to resize it.

| Tab | Shows |
|---|---|
| Preview | Your app, live from the machine |
| Browser | The live browser Optimal is using, from a sandbox |
| Desktop | The computer's live desktop |
| Machine | The sandbox or computer itself: its state, size, cost, and controls |
| Progress | What the agent is doing now and what it has finished |
| Terminal | A shell on the machine |
| Files | Browse and edit what is on the machine |
| Code | Every file changed in this chat, with its changes |
| Artifacts | Everything Optimal has produced for you |
| Deployments | Your sites and apps, and whether they are live |
| Logs | What the machine and its apps print |

A tab that does not apply yet shows what to do, for example "No computer in this chat" with **Add a computer**.

## Preview and publish

When Optimal serves an app, a preview card appears in the chat.
**Open preview** opens it; **Publish** puts it on a public URL.

The publish sheet asks for:

- **Update this site** or **New site**;
- the site name;
- the folder to publish, `/workspace` by default.

**Publish** creates the site and **Publish update** updates it.
The machine has to be running to publish.
The sheet also links to **Manage**, the custom domain, and recent versions.

## Stop, take over, hand back

- **Stop** cancels the running reply.
- **Take over**, on the Desktop and Browser tabs, pauses the agent and gives you the controls.
  The chat shows "You're in control".
- **Hand back** returns control, and the agent picks up from the machine's current state.

While you are in control, the agent cannot click or type.
If the agent asked you to take over, for example to finish a sign-in, it continues from where you leave the machine.

> **Tip: Watching without taking over**
>
> Clicking the desktop while you are only watching asks first.
> Choose **Keep watching** to stay hands-off.

## When the machine is gone

If the chat's machine is deleted, the chat says "This chat's machine was deleted. Pick another, or let Optimal choose."
Pick a machine in the machine menu and send again.

## Related

- [Chats and modes](/docs/optimal/chats): the composer and modes.
- [Isolation and permissions](/docs/optimal/isolation): what Optimal can touch on a machine.
- [Chat with an agent on a machine](/docs/agents/chat-on-a-machine): the same chat from a computer or sandbox page.
