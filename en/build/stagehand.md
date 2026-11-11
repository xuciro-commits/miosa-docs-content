---
slug: "build/stagehand"
title: "Run Stagehand on MIOSA"
description: "Stagehand: Install the framework and run reliable browser automations on a computer. Install and run it on a MIOSA sandbox or computer, then wrap it for your team or your customers."
section: "Build"
order: 10026
---

`~15 min` `TypeScript` `Python`

> **What you're building**: Install the framework and run reliable browser automations on a computer.
>
> **Primitives you'll use**: Computer

> **Tip: Run theirs, or build your own**
>
> This guide runs **Stagehand** on MIOSA. You don't have to use it.
> You can build your own browser agent instead, on MIOSA alone: drive the [live browser inside a sandbox](/docs/browser-use) over CDP, or a full [computer](/docs/computer-use) with screenshots, clicks and typing.
> You own the loop, the prompts, the model and the product.
> See [Build a browser and computer-use agent](/docs/build/chatgpt-agent).

## What you're building

an AI browser-automation framework with act, extract and observe primitives Modelled on Stagehand, by Browserbase.

Stagehand mixes code with natural-language steps: you call act/extract/observe against a page and it uses a model to find elements and perform actions reliably.

Primitives you'll use: Computer

- Natural-language act, extract and observe
- Mixes with normal automation code
- Reliable, repeatable browser flows
- Works against cloud or local browsers

You install and **run the real Stagehand** on MIOSA. Read the official docs first: [docs.stagehand.dev/v4/first-steps/quickstart](https://docs.stagehand.dev/v4/first-steps/quickstart). Every install command below comes from them; if the docs and this guide disagree, the docs win. Stagehand is open source (Browserbase).

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Computer - A persistent Linux desktop for browser, mouse, and screenshot work. - `miosa computer create app-desktop` [Docs](/docs/computers/overview)
3. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
4. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
5. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)

## Architecture

```mermaid
sequenceDiagram
  participant U as You
  participant M as MIOSA Computer
  participant P as Model provider
  participant D as MIOSA Deploy

  U->>M: install the tool from its official docs
  M->>P: calls with your own provider key
  U->>M: run a real task
  M-->>U: output, files
  U->>D: publish it for your team or your customers
```

## How it runs on MIOSA

Where it runs -> a MIOSA computer (a persistent desktop with a browser)

State and files -> Scripts live in /workspace on the computer.

Model -> your own provider key, never a MIOSA platform key

Your customers -> one workspace each, isolated from each other

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create stagehand-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Install Stagehand

Run these inside the computer (its terminal), in order. The commands are from the official docs.

```bash
pnpm add @browserbasehq/stagehand 'zod@~4.4.3'   # Python: pip install stagehand
```

Check: the install finishes without errors; the next sections configure and start it.

## Configure

Local runs need Chrome, which a MIOSA computer has. Export your own model key as the quickstart describes.

Model: Anthropic (Claude). Calls use my own provider key (ANTHROPIC_API_KEY); MIOSA platform keys are never used.

```bash
export ANTHROPIC_API_KEY="..."   # set it in the process environment, or the Secrets API; never write it into files you commit
```

## Run it

Start it and prove it works.

```bash
pnpm tsx automation.ts   # your script using act, extract and observe
```

Check: the tool starts without errors and responds.

## Persistence and access

Scripts live in /workspace on the computer.

Check: after the machine pauses and resumes, the tool starts again with its state intact.

## Make it a product

Run each customer's automation on its own computer in the customer's workspace.

## Limits and costs

Pin the version you tested and update deliberately; these tools change quickly.

Give the tool only the credentials it needs, as secrets, not baked into files.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

Stagehand runs on a MIOSA computer and answers a real task.

State survives a pause and resume.

Each customer has its own workspace and its own instance; nothing is shared between them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
