---
slug: "optimal/chats"
title: "Chats and modes"
description: "How an Optimal chat works - the composer, Ask, Build and Investigate, attachments, queued messages, the context meter, and renaming or deleting a chat."
section: "Optimal"
order: 7002
---

A chat is one conversation with one brain, in one workspace.
Open **Optimal** in the sidebar to reopen your most recent chat in the current workspace, or press **New** to start a fresh one.

## Pick a workspace first

Optimal works in one workspace at a time.
With a single workspace selected, the chat can read and change resources in that workspace only.

If **All workspaces** is selected, the chat reads every workspace you belong to.
It asks which workspace to use before it changes anything.

## The composer

The empty chat shows a headline for the current mode, three starter suggestions, and the composer.
From left to right, the composer toolbar has:

| Control | What it does |
|---|---|
| **+** | Adds files or MIOSA resources to the chat as context |
| Mode menu | Switches between Ask, Build, and Investigate |
| Brain picker | Chooses who answers; see [Brains and agents](/docs/optimal/brains) |
| Machine menu | Chooses where Optimal works; shown in Build mode or once a machine is involved |
| **Send message** | Sends; while Optimal is working it becomes **Queue message** |
| **Stop** | Cancels the running reply |

Under the input, the cost strip shows what the chat has cost so far.
See [Pricing and usage](/docs/optimal/pricing).

## Modes

| Mode | Use it to | Starter suggestions |
|---|---|---|
| Ask | Get answers from live workspace context | "What needs my attention in this workspace?", "Summarize what changed in the last day" |
| Build | Build or change something on a sandbox or computer | "Build a landing page and publish it", "Build a small API with a database" |
| Investigate | Trace evidence before reaching a conclusion | "Why did my latest deployment fail?", "Trace the most recent error in my logs" |

You can change the mode between messages.
After an answer in Ask mode, **Switch to Build and run this** turns the answer into work on a machine.

Investigate always runs on the Optimal brain.
With a harness or saved agent as the brain, Investigate is not offered; start a new chat to use it.

## Add context

Press **+** to open **Add context**.
You can:

- attach files: images, documents, or any file, up to 10 MB each and 25 MB per chat;
- drop or paste files straight into the composer;
- point Optimal at MIOSA resources in your workspace.

Optimal takes three kinds of file:

| Kind | Examples | What happens |
|---|---|---|
| Image | PNG, JPEG, WebP, GIF | The model sees it |
| Document | PDF, Word, Excel, CSV, Markdown, JSON, code | Its text is read into the message |
| Any other file | Archives, binaries | It is written to the chat's machine, under `/workspace/uploads` |

Any other file needs a machine bound to the chat; pick a sandbox or computer, then send again.
When the chat has a machine, images and documents are copied to it too.

## Send while Optimal works

You can keep typing while Optimal works.
**Queue message** sends your message right after the current reply finishes.
A queued message can be removed until it starts.

## Stop

**Stop** cancels the reply in progress.
The chat shows "Stopped", and what Optimal finished before you stopped it stays in the chat.

## Context and compaction

Every chat has a 1M-token context.
The context meter in the chat header shows how much is used, for example "312K of 1M context".
It turns amber at 70% and red at 90%.

When the context fills, Optimal folds the older part of the chat into a short summary on its own.
The meter shows "Compacting context..." while it does.
Nothing is needed from you.

## Rename, share, delete

The **...** menu in the chat header (tooltip "Chat options") has:

- **Rename**: gives the chat a new title.
- **Allowed in this chat**: lists the steps you allowed without asking, with **Remove** on each; see [Approvals](/docs/optimal/approvals).
- **Delete**: deletes the chat and its messages for good.
  Machines the chat used are not touched.

To share a chat with your team, use **Share with team** from its menu in **My chats**.
See [Team chats and sharing](/docs/optimal/team-chats).

## Ask Optimal from any page

On project, product, deployment, agent, workflow, computer, sandbox, database, and storage pages, the sidebar shows **Ask Optimal**.
It opens Optimal next to the page, with that page as context.
The shortcut is `Alt` + `Shift` + `O`.

## Find a chat again

**History** in the Optimal sidebar lists every chat in the workspace, with search and **Load older chats**.
Recent chats also sit in the sidebar under **Mine** and **Team**.
How long chats stay is set in **Preferences**: 30 days, 90 days, or 1 year.
See [Tasks, artifacts and history](/docs/optimal/workspace).

> **Note: A chat keeps its brain**
>
> The brain is fixed after the first message.
> Picking a different brain in a started chat starts a new chat.

## Related

- [Build on your machines](/docs/optimal/machines): the machine menu and the side panel.
- [Optimal API](/docs/optimal/api): the same chats over HTTP.
