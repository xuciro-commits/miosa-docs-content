---
slug: "faq/agents"
title: "Agents"
description: "Supported harnesses, creating and running an agent, OSA, model keys, and where agents run."
section: "Help"
order: 16004
---

Short answers on agent harnesses, model keys, and where agents run.

## Which agent harnesses are supported?

`claude-code`, `codex`, `osa`, and `custom`, set with `miosa agent new --harness`.
Agent Run providers also include `claude`, `pi`, and `hermes`.
See [Agents CLI](/docs/cli/agents) and [Configure an agent](/docs/guides/configure-agent).

## How do I create and run an agent?

```bash
miosa agent new reviewer --harness claude-code --instructions-file review.md
miosa agent run reviewer "review PR 42" --sandbox my-box
miosa run list                 # follow, stop
```

See [Agents CLI](/docs/cli/agents).

## What is OSA?

OSA is MIOSA's open-source agent.
It installs on your machine, runs as an MCP client, and runs headless with `osa run "..."` when you do not want the terminal UI.
See [MCP](/docs/mcp) and the [CLI reference](/docs/cli).

## Do agents use MIOSA platform keys?

No. Agents use your own model connections, never MIOSA platform keys.
Create an agent-scoped key and add a model connection before the first run.
See [Agents CLI](/docs/cli/agents).

## Where do agents run?

Agents work against a device: a Sandbox Worker, a Computer, or a local/BYOC device.
Docker Host is a hosting target, not an agent device.
See [Agent devices](/docs/guides/agent-devices) and [Choose an agent device](/docs/guides/agent-device-selection).

## Can I drive MIOSA from my own agent loop?

Yes. Call the primitives from your own backend, or start with the [Agent Development Kit (ADK)](/docs/adk).
See [Integrating an AI agent](/docs/guides/agent).
