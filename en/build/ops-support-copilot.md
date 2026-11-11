---
slug: "build/ops-support-copilot"
title: "Build an ops and support copilot"
description: "Ops and support copilot: Triage tickets and run runbooks, with human approval. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10064
---

`~25 min` `TypeScript` `Python`

> **What you're building**: Triage tickets and run runbooks, with human approval.
>
> **Primitives you'll use**: Sandbox, Agent and harness, Deployment (Docker Host)

## What you're building

an internal copilot that reads the queue, runs runbook checks and drafts replies, with a human approving

It reads tickets or alerts, follows a runbook inside a sandbox, drafts the response, and waits for a person to approve before anything changes.

Primitives you'll use: Sandbox · Agent and harness · Deployment (App Engine)

- Reads the support or on-call queue
- Runs runbook checks
- Drafts replies and actions
- Stops for human approval

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

Runbook checks in isolation -> a MIOSA sandbox with read-only credentials

Human approval -> agent approvals (`miosa agent approvals`)

Case history -> managed Postgres (`DATABASE_URL`)

Audit -> `miosa audit`

## Data and storage

Keep application data in managed Postgres with a row-level owner (member id or team), and keep an append-only audit table for agent actions.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create ops-support-copilot-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

Agents act as members: every run carries the asking member's id in `metadata`, uses credentials scoped to that member or team, and cannot exceed what the member may do.

## Steps

1. Connect the queue (tickets or alerts) with read-only credentials, stored with the Secrets API, not in the sandbox `env`.
   Check: The agent can read the queue and cannot write to it.
2. Encode each runbook as steps the agent follows inside a sandbox (checks only, no mutation).
   ```bash
   miosa create ops-support-copilot-box --template agent-python --wait
   ```
   Check: A runbook run produces findings and no side effects.
3. Draft the reply or action, then STOP for approval before anything is sent or changed.
   ```bash
   miosa prompt --sandbox ops-support-copilot-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Follow runbook <name> for ticket <id>; draft the reply; do not send"
   ```
   ```bash
   miosa agent approvals
   ```
   Check: Nothing leaves the building without an approval record.
4. Control access: members and roles, and an audit trail of what agents did.
   ```bash
   miosa member add dana@ops-support-copilot.com --role member
   ```
   ```bash
   miosa audit
   ```
   Check: A removed member loses access immediately.
5. Publish the internal UI.
   ```bash
   miosa deploy create --from-sandbox ops-support-copilot-box --name ops-support-copilot --dir /workspace --port 3000 --run-command "npm start" --wait
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
