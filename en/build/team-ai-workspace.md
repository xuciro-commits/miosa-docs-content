---
slug: "build/team-ai-workspace"
title: "Build an internal AI workspace"
description: "Internal AI workspace: One place for your team's agents, apps, and data. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10063
---

`~25 min` `TypeScript` `Python`

> **What you're building**: One place for your team's agents, apps, and data.
>
> **Primitives you'll use**: Sandbox, Agent and harness, Deployment (Docker Host)

## What you're building

the one place a team goes to ask, build and ship with agents

Everyone is a member of a single organization; each team has its own workspace with its machines, deployments and data, and agents act with the team's permissions.

Primitives you'll use: Sandbox · Agent and harness · Deployment (App Engine)

- Chat with agents that do company work
- A workspace per team
- Agents that build and run internal tools
- Access control and an audit log

This is a pattern, not a copy of one product. Start from [the related MIOSA guide](/docs/guides/internal-business-os) and build the smallest version that does the capabilities above.

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
  participant M as Teammate
  participant O as One organization
  participant W as Team workspace
  participant S as Sandbox
  participant A as Agent run

  M->>O: sign in with a role
  O->>W: scope machines and data to the team
  M->>A: ask for something
  A->>S: work inside the team sandbox
  A-->>M: a result, a draft or a built tool
  M->>M: approve anything consequential
```

## How it maps onto MIOSA

One organization for the company -> the MIOSA organization, with teammates as members (`miosa member add`)

A workspace per team -> `miosa workspace create`

Agents that do work -> agent runs (`miosa prompt`, client.runs) streaming events in a sandbox

Audit -> `miosa audit`

## Data and storage

Keep application data in managed Postgres with a row-level owner (member id or team), and keep an append-only audit table for agent actions.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create team-ai-workspace-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

Agents act as members: every run carries the asking member's id in `metadata`, uses credentials scoped to that member or team, and cannot exceed what the member may do.

## Steps

1. Create a workspace per team inside the one organization, and invite members.
   ```bash
   miosa workspace create team-ai-workspace-ops
   ```
   ```bash
   miosa org invite dana@team-ai-workspace.com
   ```
   Check: A member of one team cannot see another team's machines.
2. Run the chat backend and agent runner in a sandbox in each team workspace.
   ```bash
   miosa create team-ai-workspace-box --template agent-node --wait
   ```
   Check: Each team has its own backend.
3. Dispatch agent work from the chat, on the team's sandbox, with the member's id in run metadata.
   ```bash
   miosa prompt --sandbox team-ai-workspace-box --harness osa --model <anthropic-model-id> --chat <chat-id> "<request from the member>"
   ```
   Check: Every run shows who asked and which team it ran for.
4. Control access: members and roles, and an audit trail of what agents did.
   ```bash
   miosa member add dana@team-ai-workspace.com --role member
   ```
   ```bash
   miosa audit
   ```
   Check: A removed member loses access immediately.
5. Publish the internal UI.
   ```bash
   miosa deploy create --from-sandbox team-ai-workspace-box --name team-ai-workspace --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   Check: Members can open it and nobody else can.

## Limits and costs

Connectors are the real work: budget time for each source system's auth, rate limits and data shape.

Set `idle_timeout_sec` so unused team sandboxes pause.

Prefer read-only access by default; grant writes per task.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

Members see only their team's data and tools.

Every agent action is attributable to a member in the audit log.

Nothing consequential happens without the approval the guide describes.

Each customer is isolated in its own workspace and usage is metered against them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
