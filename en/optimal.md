---
slug: "optimal"
title: "Optimal"
description: "Optimal is MIOSA's built-in agent. Chat with it to answer questions about your workspace, build on your own sandboxes and computers, and investigate problems."
section: "Optimal"
order: 7000
---

Optimal is MIOSA's built-in agent.
You describe what you want in a chat, and Optimal answers from your live workspace, builds on a machine you choose, or investigates a problem.
Open it from **Optimal** in the console sidebar, at `/optimal`.

## What Optimal does

- **Answers** questions about your organization and workspace: what is running, what changed, how something is set up.
- **Builds** on a sandbox or computer: it writes files, runs commands, serves a live preview, and can publish what it built.
- **Investigates** a problem: it traces evidence, such as a failed deployment or an error in your logs, before it reaches a conclusion.
- **Asks before it acts**: steps that change your workspace wait for your approval unless you allow them.

Optimal works only inside your organization and workspace, with your permissions.
See [Isolation and permissions](/docs/optimal/isolation).

## Brains: who does the work

Before the first message you pick the brain that answers the chat.

| Brain | Who pays | What runs |
|---|---|---|
| Optimal | Wallet credits | MIOSA's own model, with a 1M-token context |
| A harness | Your own account | Claude Code, Codex, OSA, and the other harnesses run on a machine |
| A saved agent | Your own account | An agent you built in Agent Studio, with its own instructions and tools |

The brain is locked after the first message.
To change it, start a new chat.
See [Brains and agents](/docs/optimal/brains).

## Ask, Build, Investigate

Every chat is in one of three modes.

| Mode | What it does |
|---|---|
| Ask | Answers from live workspace context, without touching a machine |
| Build | Builds or changes something on a sandbox or computer |
| Investigate | Traces evidence before reaching a conclusion |

Investigate always runs on the Optimal brain.
See [Chats and modes](/docs/optimal/chats).

## Pricing in one line

The Optimal brain costs $1.90 per 1M input tokens and $8.00 per 1M output tokens, paid from wallet credits where 1 credit is 1 cent.
A typical message is about 3 credits.
Machines Optimal uses are billed at their normal rate.
A harness or saved agent bills its model to your own account.
See [Pricing and usage](/docs/optimal/pricing).

## Where to go next

- [Quickstart](/docs/optimal/quickstart): Your first chat, first build, and first API call.
- [Chats and modes](/docs/optimal/chats): The composer, attachments, context, and chat history.
- [Build on your machines](/docs/optimal/machines): Pick a machine, use the side panel, and take over.
- [Approvals](/docs/optimal/approvals): What Optimal asks before it acts, and how to allow it.
- [Optimal API](/docs/optimal/api): Create chats, send messages, and stream replies over HTTP.
- [Troubleshooting](/docs/optimal/troubleshooting): Credits, machines, and errors, with fixes.
