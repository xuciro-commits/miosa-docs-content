---
slug: "build/decagon"
title: "Build a Decagon-style support agent"
description: "Decagon: Resolve tickets across chat and email, with handoff. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10058
---

`~25 min` `TypeScript` `Python`

> **What you're building**: Resolve tickets across chat and email, with handoff.
>
> **Primitives you'll use**: Sandbox, Agent and harness, Postgres, Deployment (Docker Host)

## What you're building

an AI customer-support agent that resolves tickets across channels Modelled on Decagon, by Decagon.

The agent resolves chat and email tickets by following your policies, calling your systems, and escalating with context.

Primitives you'll use: Sandbox · Agent and harness · Postgres · Deployment (App Engine)

- Resolves tickets across channels
- Follows company policies
- Calls real systems
- Escalates with context

Source: [decagon.ai](https://decagon.ai/). Decagon is a trademark of its owner; this guide is not affiliated with or endorsed by them.

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. Postgres - Managed relational data, injected as DATABASE_URL in the machine and in production. - `miosa api POST /databases -d '{"name":"app-db","engine":"postgresql"}'` [Docs](/docs/data/postgres)
4. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
5. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
6. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)
7. Agent and harness - Turns a prompt into work: pick the harness and model a run uses. - `miosa agent harnesses` [Docs](/docs/api-reference/agent-runs)
8. Deployment (App Engine) - Publishes the app to an immutable, versioned URL with rollback. - `miosa deploy create --from-sandbox app-box --name app --wait` [Docs](/docs/docker-host/overview)

## Architecture

```mermaid
sequenceDiagram
  participant U as User
  participant P as Your app
  participant A as Agent sandbox
  participant T as Systems and index
  participant H as Human reviewer

  U->>P: open a case
  P->>A: run the agent with scoped tools
  A->>T: read records and retrieve sources
  T-->>A: facts with their sources
  A-->>P: a proposal with citations
  P->>H: request approval when it matters
  H-->>P: approve or edit
  P->>T: apply the change, write the audit row
```

## How it maps onto MIOSA

The agent that does the domain work -> agent runs (`miosa prompt`, client.runs) streaming events in a sandbox, with the tools it may call defined by you

Domain records and history -> managed Postgres (`DATABASE_URL`) for records, citations and an audit trail

Each customer is isolated -> a workspace per customer (`miosa workspace create`)

Human review and escalation -> agent approvals (`miosa agent approvals`) and a review queue in your product

## Data and storage

Postgres is the system of record with an append-only audit table; nothing the agent decides exists only in a prompt.

Keep sources with every stored field so a human can check them.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create decagon-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

The agent runs inside the sandbox with only the tools you defined. It proposes; your code enforces limits (refund caps, allowed recipients) so the model cannot talk its way past them.

## Steps

1. Create the domain database.
   ```bash
   miosa api POST /databases -d '{"name":"decagon-db","engine":"postgresql"}'
   ```
   ```bash
   miosa db connection decagon-db
   ```
   Check: Domain tables exist: customers, tickets, messages, actions.
2. Create the agent sandbox.
   ```bash
   miosa create decagon-box --template agent-python --wait
   ```
   Check: The sandbox is running with DATABASE_URL set.
3. Load the knowledge base (help center, policies) into a vector index so answers cite policy.
   ```bash
   miosa api POST /databases -d '{"name":"decagon-knowledge","engine":"qdrant"}'
   ```
   Check: A policy, document or article question retrieves the right passage.
4. Define the tools the agent may call on real systems (look up an order, refund within a limit, change an address) as narrow functions with typed arguments.
   ```bash
   miosa prompt --sandbox decagon-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Resolve ticket <id> using only the defined tools and the cited policy; escalate if unsure"
   ```
   Check: The agent completes the case using only those tools.
5. Put a human in the loop where it matters: the agent proposes, a person approves anything irreversible or high-stakes, and low-confidence cases escalate with context.
   ```bash
   miosa agent approvals
   ```
   Check: A refund over the limit, a send, or a filing waits for approval.
6. Write an audit record for every agent action: who, what, inputs, outputs, and the run id.
   ```bash
   miosa audit
   ```
   Check: Any decision can be reconstructed from the audit trail.
7. Publish the agent API and the review UI.
   ```bash
   miosa deploy create --from-sandbox decagon-box --name decagon --dir /workspace --port 3000 --run-command "npm start" --wait
   ```
   Check: The public_url serves the review queue.

## Limits and costs

Customer data is personal data: store only what the task needs, honour deletion requests, and keep an audit trail.

Keep tenant data separate: a workspace per customer, and every query filtered by tenant.

Set `--max-time` on runs and cap tool calls per case.

Prototype: keep it to one machine at the default size, skip replicas and custom hostnames you do not need, and delete everything when you are done.

## Acceptance checks

The agent resolves a representative case end to end using only the defined tools.

A high-stakes action waited for human approval.

The audit trail reconstructs the decision.

Each customer is isolated in its own workspace and usage is metered against them.

Everything it created can be deleted with nothing left running.

## Next steps

- [Agent prompt builder Pick a product shape and hand the brief to your agent.](/docs/build)
- [Run coding agents Run Claude Code, Codex or Cursor in the cloud, or build a product on them.](/docs/build/run-agents)
- [Compare MIOSA How MIOSA compares to other sandbox and agent platforms.](/docs/compare)
- [What you can build Every product pattern MIOSA supports.](/docs/use-cases)
