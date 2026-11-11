---
slug: "mcp"
title: "MCP overview"
description: "Connect OSA, Claude Code, Codex, Cursor, Hermes or Claude Desktop to the MIOSA MCP server for sandboxes, computers, deployments, storage and 151 tools across the platform."
section: "CLI, SDKs & MCP"
order: 11022
---

CLI & MCP

Give any AI client full MIOSA tools: OSA, Claude Code, Codex, Cursor, Hermes.

`{ "mcpServers": { "miosa": { "url": "https://api.miosa.ai/api/v1/mcp", "headers": { "Authorization": "Bearer msk_u_..." } } } }`

Every MIOSA tool inside OSA, our open-source agent, as mcp__miosa__* tools.

Save as ~/.osa/mcp.json, or run /mcp add inside OSA.

- [Install and authenticate](/docs/mcp/install): Hosted HTTP or local stdio, keys and organization context.
- [Use from agents](/docs/mcp#osa): Connect OSA, Claude Code, Codex, Cursor or Hermes.
- [Tools reference](/docs/mcp/tools): All 151 tools, grouped by family.
- [CLI or MCP](/docs/cli/cli-vs-mcp): Which one to give an agent, and why both work.

MIOSA ships a public **Model Context Protocol** server. Point any MCP-aware client at it with an `msk_u_*` API key and you get the MIOSA toolbelt. [OSA](/docs/mcp#osa), MIOSA's open-source agent, is the primary client, and Claude Code, Codex, Cursor and Hermes work the same way. The toolbelt: sandboxes, full GUI computers, snapshots, deployments, databases, storage, volumes, cron, functions, webhooks, API keys, workspaces, computer services, and regions.

This page is the server your own agent calls. For the other direction, the official tool servers you add to your agents, see the [tool server catalog](/docs/mcp/catalog).

Hosted endpoint:

```text
https://api.miosa.ai/api/v1/mcp
```

The server speaks MCP 2025-03-26 Streamable HTTP, single-POST JSON-RPC. No SSE listen stream is required.

## OSA

OSA is MIOSA's open-source agent. It runs on your machine or on a MIOSA computer, and works with your Claude or ChatGPT subscription. It is an MCP client, so it takes the MIOSA server as another set of tools next to its built-ins; they appear as `mcp__miosa__*`.

Install OSA (macOS, Linux and Windows), then start it once to run the setup wizard:

```bash
curl -fsSL https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.sh | sh
osa
```

On Windows use `irm https://raw.githubusercontent.com/Miosa-osa/OSA/main/scripts/install.ps1 | iex`. Homebrew installs the `osagent` binary instead: `brew install Miosa-osa/homebrew-tap/osagent`.

Add MIOSA to `~/.osa/mcp.json` (user scope). The file uses the same `mcpServers` shape as Claude Desktop, and a `url` selects the HTTP transport:

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

Or add it from inside OSA with the slash command, which writes the same file. OSA does not expand environment variables in `headers`, so put the key itself in the value:

```text
/mcp add miosa https://api.miosa.ai/api/v1/mcp -t http -s user -H 'Authorization: Bearer msk_u_...'
```

Run `/mcp` to list servers and their connection status. A project `.mcp.json` also works; OSA asks for approval before it starts a project-scoped server.

## Claude Code

```bash
claude mcp add --transport http miosa \
  https://api.miosa.ai/api/v1/mcp \
  --header "Authorization: Bearer msk_u_..."
```

Add `--scope user` to make it available across all your projects. Verify with `claude mcp list`.

Equivalently, in `.mcp.json` (project) or `~/.claude.json` (user):

```json
{
  "mcpServers": {
    "miosa": {
      "type": "http",
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

## Codex

`~/.codex/config.toml`:

```toml
[mcp_servers.miosa]
url = "https://api.miosa.ai/api/v1/mcp"
bearer_token_env_var = "MIOSA_API_KEY"
```

`bearer_token_env_var` names the environment variable holding the key, not the key itself. The variable must be present in the environment that launches Codex. The one-line form:

```bash
codex mcp add miosa --url https://api.miosa.ai/api/v1/mcp --bearer-token-env-var=MIOSA_API_KEY
```

## Cursor

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per project):

```json
{
  "mcpServers": {
    "miosa": {
      "url": "https://api.miosa.ai/api/v1/mcp",
      "headers": { "Authorization": "Bearer msk_u_..." }
    }
  }
}
```

## Hermes

[Hermes Agent](https://github.com/NousResearch/hermes-agent) is Nous Research's open-source agent. MCP support ships with the standard install. Add MIOSA to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  miosa:
    url: "https://api.miosa.ai/api/v1/mcp"
    headers:
      Authorization: "Bearer ${MIOSA_API_KEY}"
```

Hermes resolves `${MIOSA_API_KEY}` in `url` and `headers` when it connects, from the environment and from `~/.hermes/.env`, so the key stays out of the config file. Put `MIOSA_API_KEY=msk_u_...` in `~/.hermes/.env`, then start `hermes chat`.

## Claude Desktop

Claude Desktop's `claude_desktop_config.json` is stdio only. Use the `mcp-remote` bridge, which launches as a stdio server and proxies to the hosted endpoint.

`~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "miosa": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.miosa.ai/api/v1/mcp",
        "--header",
        "Authorization: Bearer ${MIOSA_API_KEY}"
      ],
      "env": { "MIOSA_API_KEY": "msk_u_..." }
    }
  }
}
```

Restart the app (not just reload) after editing. Alternatively, add the hosted URL under Settings, Connectors, Add custom connector and set the Authorization header there.

> **Note**
>
> Prefer the local stdio server (Python package) when you need to wrap the MCP layer with custom local logic, or your client does not support remote MCP. Both modes hit the same MIOSA REST API. See [Install and authenticate](/docs/mcp/install).

## Action authority

Governed MCP tools ask MIOSA's control-plane action authority immediately before dispatch. The control plane owns capability versions, grants, one-time approvals, revocation and receipts. The MCP server only maps tool names to canonical capabilities from a generated contract, and never copies policy locally.

Every registered tool has exactly one canonical capability. A tool with no mapping fails closed before any dispatch. Pending approval, denial, a missing catalog capability, or an unavailable authority all fail closed before the SDK or REST operation runs.

## Active computer

All computer tools accept an optional `computer_id`. When omitted, the server uses the most recently created or accessed computer, so you can `computer_create` once and omit the ID for the rest of the session.

## Errors

Failed tool calls use MCP `isError=true` and include a structured error object that preserves the backend error code, HTTP status, request ID, response details, tool name and a human-readable message. Clients can branch on stable codes such as `INVALID_TENANT_CONTEXT` without parsing display text.

- [Install and authenticate](/docs/mcp/install): Hosted HTTP and local stdio, keys, and tenant context.
- [Tools reference](/docs/mcp/tools): All 151 tools, grouped by family.
