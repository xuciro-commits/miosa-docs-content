---
slug: "build/uizard"
title: "Build a sketch-to-mockup tool"
description: "Uizard: A sketch or screenshot becomes editable UI code. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10049
---

`~25 min` `TypeScript` `Python`

> **What you're building**: A sketch or screenshot becomes editable UI code.
>
> **Primitives you'll use**: Sandbox, Agent and harness, Deployment (Docker Host)

## What you're building

a tool that turns a sketch or a screenshot into editable UI Modelled on Uizard, by Uizard.

The user supplies a hand-drawn sketch or a screenshot; the tool reads the layout and produces editable screens or code for web and mobile apps.

Primitives you'll use: Sandbox · Agent and harness · Deployment (App Engine)

- Sketch or screenshot to editable UI
- Screens you can adjust
- Code you can export
- Share the result

Source: [uizard.io](https://uizard.io/). Uizard is a trademark of its owner; this guide is not affiliated with or endorsed by them.

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
4. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
5. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)
6. Agent and harness - Turns a prompt into work: pick the harness and model a run uses. - `miosa agent harnesses` [Docs](/docs/api-reference/agent-runs)
7. Deployment (App Engine) - Publishes the app to an immutable, versioned URL with rollback. - `miosa deploy create --from-sandbox app-box --name app --wait` [Docs](/docs/docker-host/overview)

## Architecture

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your tool
  participant S as Project sandbox
  participant A as Agent
  participant D as MIOSA Deploy

  U->>P: prompt plus a design or a sketch
  P->>S: store the reference under /workspace
  P->>A: build the UI as code
  A->>S: write components and assets
  S-->>P: preview URL
  P->>P: screenshot and compare with the reference
  U->>P: publish or export
  P->>D: publish the page or hand back a bundle
```

## How it maps onto MIOSA

Prompt (and reference design) to a working UI -> agent runs (`miosa prompt`, client.runs) streaming events that write the UI as code inside the project sandbox

Live preview -> a sandbox preview URL (`miosa preview create`)

Projects and assets -> managed Postgres (`DATABASE_URL`) for project metadata; design files and exports under the sandbox's /workspace

Share or export the result -> a MIOSA deployment (`miosa deploy create`), immutable and versioned for a published page, or a downloaded bundle from the sandbox

## Data and storage

Project metadata is small; the design itself is code and assets in the project sandbox's /workspace.

Assets are content-addressed by hash so identical inputs are never regenerated.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create uizard-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

The harness writes the UI as code inside the project sandbox so it can build and screenshot its own work and iterate against the reference.

## Steps

1. Create the project sandbox that holds the generated design as code and serves the preview.
   ```bash
   miosa create uizard-box --template nextjs --wait
   ```
   Check: The sandbox is running.
2. Take the input image: store the sketch or screenshot under /workspace/design and describe it first (screens, elements, hierarchy) so the model works from a structure rather than pixels.
   Check: The image is present and the agent lists the screens and elements it sees.
3. Generate editable UI code: dispatch the harness to turn the described structure into screens (and the code behind them) the user can edit.
   ```bash
   miosa prompt --sandbox uizard-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Build the screens described from /workspace/design; write editable components and the code behind them"
   ```
   Check: The project builds and matches the reference within visual tolerance.
4. Preview and compare: render the preview, take a screenshot headlessly and compare it with the reference.
   ```bash
   miosa preview create uizard-box 3000 --name web
   ```
   Check: A screenshot of the preview exists next to the reference image.
5. Export: package the result as code (zip of /workspace) or rendered assets; no deployment is needed to hand a file back.
   ```text
   sandbox.files.download("/workspace/out")
   ```
   Check: The exported bundle opens on a clean machine.

## Limits and costs

Previews are cheap; keep one sandbox per active project and pause idle ones.

Provider usage (generation, speech) is billed by those providers; store your own job-level cost.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

The generated design builds and previews.

The result can be exported.

Re-running with the same inputs gives the same result structure.

Each customer is isolated in its own workspace and usage is metered against them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
