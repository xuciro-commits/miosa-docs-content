---
slug: "agents/agent-studio"
title: "Agent Studio"
description: "Chat with an agent in a dedicated workspace, with one warm sandbox per chat and honest status and errors."
section: "Agents"
order: 6007
---

Agent Studio is the chat-first way to use a saved agent.
The chat is the main surface, and the canvas folds into a slim strip you can reopen.

## One sandbox per chat

A chat gets one warm sandbox, and follow-ups reuse it.
A paused sandbox is woken with its files.
Only a destroyed sandbox is replaced, and the chat says so.

## Honest status and errors

The status line shows what is happening: starting the sandbox, starting the agent, thinking.
A failed message shows the platform's own message, its code, and one hint, shown once on the turn.

## The workspace panel

A workspace panel sits beside the chat, with the same tabs as Optimal:

- Preview
- Files
- Code
- Terminal
- Machine
- Logs

It opens on the chat's first activity and shows the chat's own sandbox.

## A clean conversation

Harness wording such as Overdrive warnings and the stdin banner never reaches the chat.
Overdrive is one quiet chip beside the composer with a tooltip.
Tool rows stay inside the chat: a long argument can no longer widen it, and commands, arguments, and JSON output are readable.

Past chats are kept per agent in both Agent Studio and the agent page.

## Related

- [Chat with an agent on a machine](/docs/agents/chat-on-a-machine): the chat on a computer or sandbox page.
- [Configure an agent](/docs/guides/configure-agent): instructions, model, tools, and permissions.
