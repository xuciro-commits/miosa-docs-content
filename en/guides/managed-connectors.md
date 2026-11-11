---
slug: "guides/managed-connectors"
title: "Managed connectors for sandbox agents"
description: "Attach MIOSA-managed tools like Refero to sandbox or computer agents without client-supplied vendor tokens."
section: "Sandboxes"
order: 2020
---

MIOSA agents should build inside MIOSA sandboxes or computers. The local CLI or
MCP client should orchestrate the work; the sandbox/computer should be where the
agent writes files, installs packages, runs commands, previews apps, and
produces artifacts.

Managed connectors give those in-VM agents tools without asking every client to
bring vendor credentials.

## Connectors, egress, and MCP

These three surfaces work together, but they are not interchangeable:

| Surface | Owns | Example |
|---|---|---|
| Connector | Identity and credential grant for a provider or tool | `refero/design-research`, `anthropic/workspace-claude`, a tenant OAuth install |
| Egress | Network policy and token brokering from the sandbox/computer to approved endpoints | Swap `miosa-tok-*` only when calling `https://api.refero.design/mcp` |
| MCP | Tool protocol the agent runtime speaks inside the sandbox, computer, editor, or hosted endpoint | Claude Code calls a Refero MCP server or the MIOSA hosted MCP server |

A connector says **what account/tool is granted**. Egress says **where network
traffic may go and when placeholders can be exchanged**. MCP says **how an
agent discovers and invokes tools**. A product may use all three in one run:
attach a connector, enforce egress to the provider endpoint, and configure the
agent runtime to call that provider through MCP.

> **Tip: Built-in provider**
>
> `refero/design-research` is MIOSA-managed. Tenants can attach it to a sandbox,
> but MIOSA does not return the Refero token to clients.

## Default flow

Create a sandbox with enough time for active agent work:

```bash
miosa create builder \
  --template nextjs \
  --ttl 1h \
  --wait \
  --json
```

Attach Refero Design Research:

```bash
miosa connections add tools \
  --name refero-design-research \
  --url https://api.refero.design/mcp \
  --header-env Authorization=REFERO_MCP_TOKEN \
  --json
```

Run the agent inside `/workspace`:

```bash
miosa prompt --sandbox <sandbox-id> \
  --harness claude \
  --json \
  "Research product references, build the page, run tests, and leave files in /workspace"
```

For direct tool use, the same sandbox can be edited with file and command
commands:

```bash
miosa files cp ./page.jsx <sandbox-id>:/workspace/app/page.jsx --json
miosa exec <sandbox-id> --cwd /workspace --json -- npm run build
miosa wait <sandbox-id> --timeout 180s --json
```

## How the token works

The sandbox receives a brokered placeholder:

```bash
REFERO_MCP_TOKEN         # from the connector page
```

The real provider token stays encrypted in MIOSA-controlled infrastructure. The
egress proxy swaps the placeholder only for the provider endpoint it belongs to,
for Refero:

```text
https://api.refero.design/mcp
```

That means:

- clients do not paste the Refero token,
- the token is not shown by `miosa connections list`,
- `miosa connections test refero-design-research` does not return the token,
- sandbox code can still use normal environment-variable based MCP config.

## MCP config inside the sandbox

Use the brokered env var in the agent's MCP configuration:

```toml
[mcp_servers.refero]
enabled = true
url = "https://api.refero.design/mcp"

[mcp_servers.refero.http_headers]
Authorization = "Bearer ${REFERO_MCP_TOKEN}"
```

## Tenant-provided connectors

When MIOSA does not provide a connector, tenants can still bring their own:

```bash
printf '%s' "$ANTHROPIC_API_KEY" \
  | miosa connections add models anthropic \
      --label workspace-claude \
      --key-stdin \
      --json
```

## Which runtime should I use?

- [Sandbox](/docs/develop/sandboxes): Agent coding, generated files, tests, package installs, live preview, and publish prep.
- [Computer](/docs/computers/overview): Persistent desktop or VM workflows, browser/GUI tasks, SSH, and long-running operator sessions.
- [Docker Host](/docs/docker-host/docker-deploy): Durable app containers on the workspace Docker Host host after the sandbox-built app is ready.
