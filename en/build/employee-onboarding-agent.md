---
slug: "build/employee-onboarding-agent"
title: "Build an employee onboarding agent"
description: "Employee onboarding agent: Walks a new hire through their first week. Build it on MIOSA with the exact resources, steps and commands."
section: "Build"
order: 10067
---

`~25 min` `TypeScript` `Python`

> **What you're building**: Walks a new hire through their first week.
>
> **Primitives you'll use**: Sandbox, Qdrant vectors, Agent and harness, Deployment (Docker Host)

## What you're building

an agent that walks a new hire through their first week, provisions what they need and answers with citations

It runs a checklist, provisions repos and access in a sandbox, and answers questions from the company docs with citations.

Primitives you'll use: Sandbox · Qdrant vectors · Agent and harness · Deployment (App Engine)

- Runs the onboarding checklist
- Provisions repos and access
- Answers from company docs with citations
- Reports progress to the manager

This is a pattern, not a copy of one product. Start from [the related MIOSA guide](/docs/guides/internal-business-os) and build the smallest version that does the capabilities above.

## What you need on MIOSA

Each row is one thing to create before you start. The number matches the step that uses it.

1. Organization and API key - Scopes every call; a workspace key is all a worker needs. - `miosa api-key create app-key --preset agent` [Docs](/docs/platform/api-keys)
2. Sandbox - The isolated Linux workspace the agent writes code and runs commands in. - `miosa create app-box --template nextjs --wait` [Docs](/docs/develop/sandboxes)
3. Qdrant vectors - Managed vector store for search and embeddings, injected as QDRANT_URL. - `miosa api POST /databases -d '{"name":"app-index","engine":"qdrant"}'` [Docs](/docs/data/qdrant)
4. A workspace per customer - Isolates each customer’s machines, deployments, and data as they onboard. - `miosa workspace create customer-1` [Docs](/docs/platform/white-label)
5. Branding and white-label - Your name and slug on previews, deployments, and the desktop; customers never see MIOSA. - `miosa org` [Docs](/docs/platform/white-label)
6. Usage metering and bill-to - Usage per customer, and which account pays for new machines. - `miosa org bill` [Docs](/docs/platform/usage-and-billing)
7. Agent and harness - Turns a prompt into work: pick the harness and model a run uses. - `miosa agent harnesses` [Docs](/docs/api-reference/agent-runs)
8. Deployment (App Engine) - Publishes the app to an immutable, versioned URL with rollback. - `miosa deploy create --from-sandbox app-box --name app --wait` [Docs](/docs/docker-host/overview)

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

Provisioning in isolation -> a MIOSA sandbox with scoped credentials

Answers from the handbook -> managed Qdrant (`QDRANT_URL`)

Checklist state per hire -> managed Postgres (`DATABASE_URL`)

Org membership -> `miosa org invite`, `miosa member add`

## Data and storage

Keep application data in managed Postgres with a row-level owner (member id or team), and keep an append-only audit table for agent actions.

## Auth and tenancy

Your organization is the platform; customers never get a MIOSA account, they sign into YOUR product. One workspace per customer, created as they onboard, isolates their machines, runs and data.

```bash
miosa workspace create employee-onboarding-agent-customer-1
```

Tag every machine and run with the customer id in `metadata`, and never query across customers. Meter usage per customer with GET /api/v1/usage and set who pays with PUT /api/v1/bill-to (/docs/platform/usage-and-billing).

## Agent loop

Harness: OSA is MIOSA's own harness and works with any model provider you connect, including your own model.

Model: Anthropic (Claude). Calls use my own provider key (`miosa connections add models`); MIOSA platform keys are never used.

Sessions: one chat per project or conversation, so the agent keeps its context. The first `miosa prompt` on a sandbox uses `--new-chat` (a chat id is printed); every later turn passes `--chat <chat-id>`. `--reuse chat` keeps one new machine per chat so files persist.

Streaming: follow a run with `miosa run follow <run-id>` or client.runs.streamEvents(run.id), and steer or stop it with `miosa run steer` and `miosa run interrupt`.

Agents act as members: every run carries the asking member's id in `metadata`, uses credentials scoped to that member or team, and cannot exceed what the member may do.

## Steps

1. Store the checklist and each hire's progress.
   ```bash
   miosa api POST /databases -d '{"name":"employee-onboarding-agent-onboarding","engine":"postgresql"}'
   ```
   Check: A new hire has a checklist with item states.
2. Provision what each item needs in a sandbox with scoped credentials (repos, accounts, access), recording each grant.
   ```bash
   miosa create employee-onboarding-agent-box --template agent-node --wait
   ```
   Check: Every grant is logged and revocable.
3. Answer questions from the handbook with citations.
   ```bash
   miosa api POST /databases -d '{"name":"employee-onboarding-agent-handbook","engine":"qdrant"}'
   ```
   ```bash
   miosa prompt --sandbox employee-onboarding-agent-box --harness osa --model <anthropic-model-id> --chat <chat-id> "Answer from the handbook chunks; cite them"
   ```
   Check: A policy question is answered with a link to the handbook page.
4. Report progress to the manager and escalate blocked items.
   Check: The manager sees what is done and what is stuck.
5. Control access: members and roles, and an audit trail of what agents did.
   ```bash
   miosa member add dana@employee-onboarding-agent.com --role member
   ```
   ```bash
   miosa audit
   ```
   Check: A removed member loses access immediately.
6. Publish the internal UI.
   ```bash
   miosa deploy create --from-sandbox employee-onboarding-agent-box --name employee-onboarding-agent --dir /workspace --port 3000 --run-command "npm start" --wait
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
