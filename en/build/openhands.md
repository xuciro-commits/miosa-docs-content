---
slug: "build/openhands"
title: "Run OpenHands on MIOSA"
description: "OpenHands: Install the open-source software agent; the sandbox is its workspace. Install and run it on a MIOSA sandbox or computer, then wrap it for your team or your customers."
section: "Build"
order: 10016
---

`~15 min` `TypeScript` `Python`

> **What you're building**: Install the open-source software agent; the sandbox is its workspace.
>
> **Primitives you'll use**: Sandbox

## What you're building

an open-source software-engineering agent that works in a sandboxed workspace Modelled on OpenHands, by All Hands AI.

OpenHands runs an agent loop that edits files, runs commands and browses inside a sandboxed runtime, driven by a task description.

Primitives you'll use: Sandbox

- Open-source SWE agent
- Sandboxed runtime for commands and files
- Works on GitHub issues and tasks
- Bring your own model

You install and **run the real OpenHands** on MIOSA. Read the official docs first: [github.com/OpenHands/OpenHands](https://github.com/OpenHands/OpenHands). Every install command below comes from them; if the docs and this guide disagree, the docs win. OpenHands is open source (the Agent Canvas front end and agent server).

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
4. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
5. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)

## Architecture

```mermaid
sequenceDiagram
  participant U as You
  participant M as MIOSA Sandbox
  participant P as Model provider
  participant D as MIOSA Deploy

  U->>M: install the tool from its official docs
  M->>P: calls with your own provider key
  U->>M: run a real task
  M-->>U: output, files and a preview URL
  U->>D: publish it for your team or your customers
```

## How it runs on MIOSA

Where it runs -> a MIOSA sandbox (an isolated Linux workspace)

State and files -> State lives under `~/.openhands`; a persistent sandbox keeps it across pauses. Keep project folders under /workspace.

Model -> your own provider key, never a MIOSA platform key

Its web port 8000 -> a MIOSA preview URL (`miosa preview create`)

Your customers -> one workspace each, isolated from each other

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create openhands-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Install OpenHands

Run these inside the sandbox (for example `miosa exec openhands-task-1 -- bash -lc '<command>'`), in order. The commands are from the official docs.

```bash
node --version   # Node.js 24 or later is required
```

```text
curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash
npm install -g @openhands/agent-canvas
```

Check: the install finishes without errors; the next sections configure and start it.

## Configure

Open the UI, add an LLM profile in Settings, and paste your own provider key. The MIOSA sandbox is already the isolation boundary, so use the "without a Docker sandbox" mode the README describes.

Model: Anthropic (Claude). Calls use my own provider key (entered in the tool's own setup); MIOSA platform keys are never used.

## Run it

Start it and prove it works.

```text
agent-canvas
```

Check: the tool starts without errors and responds.

## Persistence and access

State lives under `~/.openhands`; a persistent sandbox keeps it across pauses. Keep project folders under /workspace.

It listens on port 8000. Run it as a background process so it outlives your shell, then open a preview:

```bash
miosa preview create openhands-task-1 8000 --name web
```

Check: the preview URL answers, and still answers after the machine pauses and resumes (previews wake it on request).

## Make it a product

Run one OpenHands sandbox per customer workspace and put your own auth in front of the preview URL; never expose an unauthenticated agent server.

## Limits and costs

Pin the version you tested and update deliberately; these tools change quickly.

Give the tool only the credentials it needs, as secrets, not baked into files.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

OpenHands runs on a MIOSA sandbox and answers a real task.

State survives a pause and resume.

Each customer has its own workspace and its own instance; nothing is shared between them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
